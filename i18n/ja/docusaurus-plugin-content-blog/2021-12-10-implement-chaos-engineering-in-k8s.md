---
slug: /implement-chaos-engineering-in-k8s
title: 'Implementing Chaos Engineering in K8s: Chaos Mesh Principle Analysis and Control Plane Development'
authors: mayocream
image: /img/blog/implement-chaos-engineering-in-k8s.png
tags: [Chaos Mesh, Chaos Engineering]
---

![Kubernetesにおけるカオスエンジニアリングの実装](/img/blog/implement-chaos-engineering-in-k8s.png)

[Chaos Mesh](https://chaos-mesh.org/docs/)は、Kubernetes（K8s）のカスタムリソース定義（CRD）上に構築されたオープンソースのクラウドネイティブなカオスエンジニアリングプラットフォームです。Chaos Meshはさまざまな種類の障害をシミュレートでき、障害シナリオをオーケストレーションする巨大な能力を持っています。開発、テスト、本番環境で発生する可能性のあるさまざまな異常を簡単にシミュレートし、システム内の潜在的な問題を見つけることができます。

<!--truncate-->

この記事では、Kubernetesクラスターにおけるカオスエンジニアリングの実践を探り、ソースコードの分析を通じてChaos Meshの重要な機能について議論し、コード例を用いてChaos Meshのコントロールプレーンの開発方法を説明します。

Chaos Meshに慣れていない場合は、[Chaos Meshのドキュメント](https://chaos-mesh.org/docs/#architecture-overview)を確認して、Chaos Meshのアーキテクチャに関する基本的な知識を取得してください。

この記事のテストコードについては、GitHubの[mayocream/chaos-mesh-controlpanel-demo](https://github.com/mayocream/chaos-mesh-controlpanel-demo)リポジトリを参照してください。

## Chaos Meshがカオスを生み出す方法

Chaos Meshは、Kubernetes上でカオスエンジニアリングを実装するためのスイスアーミーナイフです。このセクションでは、その仕組みを紹介します。

### 特権モード

Chaos Meshは、Kubernetesで特権コンテナを実行して障害を作り出します。Chaos DaemonのPodは`DaemonSet`として実行され、Podのセキュリティコンテキストを介してPodのコンテナランタイムに追加の[capabilities](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#capabilities)を追加します。

```yaml
apiVersion: apps/v1
kind: DaemonSet
spec:
 template:
   metadata: ...
   spec:
     containers:
       - name: chaos-daemon
         securityContext:
           {{- if .Values.chaosDaemon.privileged }}
           privileged: true
           capabilities:
             add:
               - SYS_PTRACE
           {{- else }}
           capabilities:
             add:
               - SYS_PTRACE
               - NET_ADMIN
               - MKNOD
               - SYS_CHROOT
               - SYS_ADMIN
               - KILL
               # CAP_IPC_LOCK is used to lock memory
               - IPC_LOCK
           {{- end }}
```

Linuxのcapabilitiesは、コンテナに`/dev/fuse` Filesystem in Userspace（FUSE）パイプを作成およびアクセスする特権を付与します。FUSEはLinuxのユーザースペースファイルシステムインターフェースです。これにより、非特権ユーザーがカーネルコードを編集することなく独自のファイルシステムを作成できます。

GitHubの[プルリクエスト #1109](https://github.com/chaos-mesh/chaos-mesh/pull/1109)によると、`DaemonSet`プログラムはcgoを使用してLinuxの`makedev`関数を呼び出し、FUSEパイプを作成します。

```go
// #include <sys/sysmacros.h>
// #include <sys/types.h>
// // makedev is a macro, so a wrapper is needed
// dev_t Makedev(unsigned int maj, unsigned int min) {
//   return makedev(maj, min);
// }
// EnsureFuseDev ensures /dev/fuse exists. If not, it will create one

func EnsureFuseDev() {
   if _, err := os.Open("/dev/fuse"); os.IsNotExist(err) {
       // 10, 229 according to https://www.kernel.org/doc/Documentation/admin-guide/devices.txt
       fuse := C.Makedev(10, 229)
       syscall.Mknod("/dev/fuse", 0o666|syscall.S_IFCHR, int(fuse))
   }
}
```

[プルリクエスト #1453](https://github.com/chaos-mesh/chaos-mesh/pull/1453)では、Chaos Daemonはデフォルトで特権モードを有効にしています。つまり、コンテナの`SecurityContext`に`privileged: true`を設定しています。

### Podの強制終了

`PodKill`、`PodFailure`、`ContainerKill`は`PodChaos`カテゴリに属します。`PodKill`はランダムにPodを強制終了します。APIサーバーを呼び出して強制終了コマンドを送信します。

```go
import (
   "context"
   v1 "k8s.io/api/core/v1"
   "sigs.k8s.io/controller-runtime/pkg/client"
)

type Impl struct {
   client.Client
}

func (impl *Impl) Apply(ctx context.Context, index int, records []*v1alpha1.Record, obj v1alpha1.InnerObject) (v1alpha1.Phase, error) {
   ...
   err = impl.Get(ctx, namespacedName, &pod)
   if err != nil {
       // TODO: handle this error
       return v1alpha1.NotInjected, err
   }
   err = impl.Delete(ctx, &pod, &client.DeleteOptions{
       GracePeriodSeconds: &podchaos.Spec.GracePeriod, // PeriodSeconds has to be set specifically
   })
   ...
   return v1alpha1.Injected, nil
}
```

`GracePeriodSeconds`パラメータを使用すると、Kubernetesは[Podを強制的に終了](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-forced)できます。たとえば、Podを即座に削除する必要がある場合は、`kubectl delete pod --grace-period=0 --force`コマンドを使用します。

`PodFailure`はPodオブジェクトリソースにパッチを適用し、Pod内のイメージを誤ったものに置き換えます。Chaosは`containers`と`initContainers`の`image`フィールドのみを変更します。これは、Podに関するメタデータの大部分が不変であるためです。詳細については、[Podの更新と置換](https://kubernetes.io/docs/concepts/workloads/pods/#pod-update-and-replacement)を参照してください。

```go
func (impl *Impl) Apply(ctx context.Context, index int, records []*v1alpha1.Record, obj v1alpha1.InnerObject) (v1alpha1.Phase, error) {
   ...
   pod := origin.DeepCopy()
   for index := range pod.Spec.Containers {
       originImage := pod.Spec.Containers[index].Image
       name := pod.Spec.Containers[index].Name
       key := annotation.GenKeyForImage(podchaos, name, false)
       if pod.Annotations == nil {
           pod.Annotations = make(map[string]string)
       }
       // If the annotation is already existed, we could skip the reconcile for this container
       if _, ok := pod.Annotations[key]; ok {
           continue
       }
       pod.Annotations[key] = originImage
       pod.Spec.Containers[index].Image = config.ControllerCfg.PodFailurePauseImage
   }
   for index := range pod.Spec.InitContainers {
       originImage := pod.Spec.InitContainers[index].Image
       name := pod.Spec.InitContainers[index].Name
       key := annotation.GenKeyForImage(podchaos, name, true)
       if pod.Annotations == nil {
           pod.Annotations = make(map[string]string)
       }
       // If the annotation is already existed, we could skip the reconcile for this container
       if _, ok := pod.Annotations[key]; ok {
           continue
       }
       pod.Annotations[key] = originImage
       pod.Spec.InitContainers[index].Image = config.ControllerCfg.PodFailurePauseImage
   }
   err = impl.Patch(ctx, pod, client.MergeFrom(&origin))
   if err != nil {
       // TODO: handle this error
       return v1alpha1.NotInjected, err
   }
   return v1alpha1.Injected, nil
}
```

障害を引き起こすデフォルトのコンテナイメージは`gcr.io/google-containers/pause:latest`です。

`PodKill`と`PodFailure`はKubernetes APIサーバーを介してPodのライフサイクルを制御します。しかし、`ContainerKill`はクラスターノード上で実行されるChaos Daemonを介してこれを行います。`ContainerKill`はChaos Controller Managerを使用してクライアントを実行し、Chaos DaemonへのgRPC呼び出しを開始します。

```go
func (b *ChaosDaemonClientBuilder) Build(ctx context.Context, pod *v1.Pod) (chaosdaemonclient.ChaosDaemonClientInterface, error) {
   ...
   daemonIP, err := b.FindDaemonIP(ctx, pod)
   if err != nil {
       return nil, err
   }
   builder := grpcUtils.Builder(daemonIP, config.ControllerCfg.ChaosDaemonPort).WithDefaultTimeout()
   if config.ControllerCfg.TLSConfig.ChaosMeshCACert != "" {
       builder.TLSFromFile(config.ControllerCfg.TLSConfig.ChaosMeshCACert, config.ControllerCfg.TLSConfig.ChaosDaemonClientCert, config.ControllerCfg.TLSConfig.ChaosDaemonClientKey)
   } else {
       builder.Insecure()
   }
   cc, err := builder.Build()
   if err != nil {
       return nil, err
   }
   return chaosdaemonclient.New(cc), nil
}
```

Chaos Controller ManagerがChaos Daemonにコマンドを送信する際、Pod情報に基づいて対応するクライアントを作成します。例えば、ノード上のPodを制御する場合、Podが配置されているノードの`ClusterIP`を取得してクライアントを作成します。Transport Layer Security（TLS）証明書の設定が存在する場合、Controller ManagerはクライアントにTLS証明書を追加します。

Chaos Daemon起動時、TLS証明書を持っている場合、gRPCSを有効化するために証明書を添付します。TLS設定オプション`RequireAndVerifyClientCert`は相互TLS（mTLS）認証を有効にするかどうかを示します。

```go
func newGRPCServer(containerRuntime string, reg prometheus.Registerer, tlsConf tlsConfig) (*grpc.Server, error) {
   ...
   if tlsConf != (tlsConfig{}) {
       caCert, err := ioutil.ReadFile(tlsConf.CaCert)
       if err != nil {
           return nil, err
       }
       caCertPool := x509.NewCertPool()
       caCertPool.AppendCertsFromPEM(caCert)
       serverCert, err := tls.LoadX509KeyPair(tlsConf.Cert, tlsConf.Key)
       if err != nil {
           return nil, err
       }
       creds := credentials.NewTLS(&tls.Config{
           Certificates: []tls.Certificate{serverCert},
           ClientCAs:    caCertPool,
           ClientAuth:   tls.RequireAndVerifyClientCert,
       })
       grpcOpts = append(grpcOpts, grpc.Creds(creds))
   }
   s := grpc.NewServer(grpcOpts...)
   grpcMetrics.InitializeMetrics(s)
   pb.RegisterChaosDaemonServer(s, ds)
   reflection.Register(s)
   return s, nil
}
```

Chaos Daemonは呼び出し可能な以下のgRPCインターフェースを提供します：

```go
// ChaosDaemonClient is the client API for ChaosDaemon service.
//
// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://godoc.org/google.golang.org/grpc#ClientConn.NewStream.

type ChaosDaemonClient interface {

   SetTcs(ctx context.Context, in *TcsRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   FlushIPSets(ctx context.Context, in *IPSetsRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   SetIptablesChains(ctx context.Context, in *IptablesChainsRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   SetTimeOffset(ctx context.Context, in *TimeRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   RecoverTimeOffset(ctx context.Context, in *TimeRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   ContainerKill(ctx context.Context, in *ContainerRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   ContainerGetPid(ctx context.Context, in *ContainerRequest, opts ...grpc.CallOption) (*ContainerResponse, error)
   ExecStressors(ctx context.Context, in *ExecStressRequest, opts ...grpc.CallOption) (*ExecStressResponse, error)
   CancelStressors(ctx context.Context, in *CancelStressRequest, opts ...grpc.CallOption) (*empty.Empty, error)
   ApplyIOChaos(ctx context.Context, in *ApplyIOChaosRequest, opts ...grpc.CallOption) (*ApplyIOChaosResponse, error)
   ApplyHttpChaos(ctx context.Context, in *ApplyHttpChaosRequest, opts ...grpc.CallOption) (*ApplyHttpChaosResponse, error)
   SetDNSServer(ctx context.Context, in *SetDNSServerRequest, opts ...grpc.CallOption) (*empty.Empty, error)
}
```

### ネットワーク障害の注入

[プルリクエスト #41](https://github.com/chaos-mesh/chaos-mesh/pull/41)から、Chaos Meshがネットワーク障害を注入する方法がわかります：`pbClient.SetNetem`を呼び出してパラメータをリクエストにカプセル化し、そのリクエストをノード上のChaos Daemonに送信して処理させます。

2019年時点でのネットワーク障害注入コードを以下に示します。プロジェクトの進化に伴い、機能は複数のファイルに分散されました。

```go
func (r *Reconciler) applyPod(ctx context.Context, pod *v1.Pod, networkchaos *v1alpha1.NetworkChaos) error {
   ...
   pbClient := pb.NewChaosDaemonClient(c)
   containerId := pod.Status.ContainerStatuses[0].ContainerID
   netem, err := spec.ToNetem()
   if err != nil {
       return err
   }
   _, err = pbClient.SetNetem(ctx, &pb.NetemRequest{
       ContainerId: containerId,
       Netem:       netem,
   })
   return err
}
```

`pkg/chaosdaemon`パッケージでは、Chaos Daemonがリクエストを処理する方法を確認できます。

```go
func (s *Server) SetNetem(ctx context.Context, in *pb.NetemRequest) (*empty.Empty, error) {
   log.Info("Set netem", "Request", in)
   pid, err := s.crClient.GetPidFromContainerID(ctx, in.ContainerId)
   if err != nil {
       return nil, status.Errorf(codes.Internal, "get pid from containerID error: %v", err)
   }
   if err := Apply(in.Netem, pid); err != nil {
       return nil, status.Errorf(codes.Internal, "netem apply error: %v", err)
   }
   return &empty.Empty{}, nil
}

// Apply applies a netem on eth0 in pid related namespace

func Apply(netem *pb.Netem, pid uint32) error {
   log.Info("Apply netem on PID", "pid", pid)
   ns, err := netns.GetFromPath(GenNetnsPath(pid))
   if err != nil {
       log.Error(err, "failed to find network namespace", "pid", pid)
       return errors.Trace(err)
   }
   defer ns.Close()
   handle, err := netlink.NewHandleAt(ns)
   if err != nil {
       log.Error(err, "failed to get handle at network namespace", "network namespace", ns)
       return err
   }
   link, err := handle.LinkByName("eth0") // TODO: check whether interface name is eth0
   if err != nil {
       log.Error(err, "failed to find eth0 interface")
       return errors.Trace(err)
   }
   netemQdisc := netlink.NewNetem(netlink.QdiscAttrs{
       LinkIndex: link.Attrs().Index,
       Handle:    netlink.MakeHandle(1, 0),
       Parent:    netlink.HANDLE_ROOT,
   }, ToNetlinkNetemAttrs(netem))
   if err = handle.QdiscAdd(netemQdisc); err != nil {
       if !strings.Contains(err.Error(), "file exists") {
           log.Error(err, "failed to add Qdisc")
           return errors.Trace(err)
       }
   }
   return nil
}
```

最終的に、[`vishvananda/netlink`ライブラリ](https://github.com/vishvananda/netlink)がLinuxネットワークインターフェースを操作して処理を完了させます。

ここから、`NetworkChaos`はLinuxホストネットワークを操作してカオスを発生させます。iptablesやipsetなどのツールが含まれます。

Chaos DaemonのDockerfileでは、依存しているLinuxツールチェーンを確認できます：

```dockerfile
RUN apt-get update && \
   apt-get install -y tzdata iptables ipset stress-ng iproute2 fuse util-linux procps curl && \
   rm -rf /var/lib/apt/lists/*
```

### ストレステスト

Chaos Daemonは`StressChaos`も実装しています。Controller Managerがルールを計算した後、タスクを特定の`Daemon`に送信します。組み立てられたパラメータを以下に示します。これらはコマンド実行パラメータに結合され、`stress-ng`コマンドに追加されて実行されます。

```go
// Normalize the stressors to comply with stress-ng
func (in *Stressors) Normalize() (string, error) {
   stressors := ""
   if in.MemoryStressor != nil && in.MemoryStressor.Workers != 0 {
       stressors += fmt.Sprintf(" --vm %d --vm-keep", in.MemoryStressor.Workers)
       if len(in.MemoryStressor.Size) != 0 {
           if in.MemoryStressor.Size[len(in.MemoryStressor.Size)-1] != '%' {
               size, err := units.FromHumanSize(string(in.MemoryStressor.Size))
               if err != nil {
                   return "", err
               }
               stressors += fmt.Sprintf(" --vm-bytes %d", size)
           } else {
               stressors += fmt.Sprintf(" --vm-bytes %s",
                   in.MemoryStressor.Size)
           }
       }
       if in.MemoryStressor.Options != nil {
           for _, v := range in.MemoryStressor.Options {
               stressors += fmt.Sprintf(" %v ", v)
           }
       }
   }
   if in.CPUStressor != nil && in.CPUStressor.Workers != 0 {
       stressors += fmt.Sprintf(" --cpu %d", in.CPUStressor.Workers)
       if in.CPUStressor.Load != nil {
           stressors += fmt.Sprintf(" --cpu-load %d",
               *in.CPUStressor.Load)
       }
       if in.CPUStressor.Options != nil {
           for _, v := range in.CPUStressor.Options {
               stressors += fmt.Sprintf(" %v ", v)
           }
       }
   }
   return stressors, nil
}
```

Chaos Daemonサーバー側では、関数の実行コマンドを処理するために公式Goパッケージ`os/exec`を呼び出します。詳細は[`pkg/chaosdaemon/stress_server_linux.go`](https://github.com/chaos-mesh/chaos-mesh/blob/98af3a0e7832a4971d6b133a32069539d982ef0a/pkg/chaosdaemon/stress_server_linux.go#L33)ファイルを参照してください。darwinで終わる同名のファイルもあります。`*_darwin`ファイルは、プログラムがmacOSで実行される際の潜在的なエラーを防ぎます。

このコードは[`shirou/gopsutil`](https://github.com/shirou/gopsutil)パッケージを使用してPIDプロセスステータスを取得し、stdoutとstderrの標準出力を読み取ります。この処理モードは[`hashicorp/go-plugin`](https://github.com/hashicorp/go-plugin)でも見られ、go-pluginの方が優れています。

### I/O障害の注入

[プルリクエスト #826](https://github.com/chaos-mesh/chaos-mesh/pull/826)では、サイドカー注入を使用しない新しいIOChaosの実装が導入されています。Chaos Daemonを使用して、[runc](https://github.com/opencontainers/runc)コンテナの低レベルコマンドを通じてLinuxネームスペースを直接操作し、Rustで開発された[chaos-mesh/toda](https://github.com/chaos-mesh/toda) FUSEプログラムを実行してコンテナI/Oカオスを注入します。[JSON-RPC 2.0](https://pkg.go.dev/github.com/ethereum/go-ethereum/rpc)プロトコルがtodaとコントロールプレーン間の通信に使用されます。

新しいIOChaosの実装では、Podリソースを変更しません。IOChaosのカオス実験を定義する際、selectorフィールドでフィルタリングされた各Podに対して、対応するPodIOChaosリソースが作成されます。PodIOChaosの[オーナーリファレンス](https://kubernetes.io/docs/concepts/overview/working-with-objects/owners-dependents/)はPodになります。同時に、PodIOChaosが削除される前にリソースを解放するため、[ファイナライザー](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/)のセットがPodIOChaosに追加されます。

```go
// Apply implements the reconciler.InnerReconciler.Apply

func (r *Reconciler) Apply(ctx context.Context, req ctrl.Request, chaos v1alpha1.InnerObject) error {
   iochaos, ok := chaos.(*v1alpha1.IoChaos)
   if !ok {
       err := errors.New("chaos is not IoChaos")
       r.Log.Error(err, "chaos is not IoChaos", "chaos", chaos)
       return err
   }
   source := iochaos.Namespace + "/" + iochaos.Name
   m := podiochaosmanager.New(source, r.Log, r.Client)
   pods, err := utils.SelectAndFilterPods(ctx, r.Client, r.Reader, &iochaos.Spec)
   if err != nil {
       r.Log.Error(err, "failed to select and filter pods")
       return err
   }
   r.Log.Info("applying iochaos", "iochaos", iochaos)
   for _, pod := range pods {
       t := m.WithInit(types.NamespacedName{
           Name:      pod.Name,
           Namespace: pod.Namespace,
       })

       // TODO: support chaos on multiple volume

       t.SetVolumePath(iochaos.Spec.VolumePath)
       t.Append(v1alpha1.IoChaosAction{
           Type: iochaos.Spec.Action,
           Filter: v1alpha1.Filter{
               Path:    iochaos.Spec.Path,
               Percent: iochaos.Spec.Percent,
               Methods: iochaos.Spec.Methods,
           },
           Faults: []v1alpha1.IoFault{
               {
                   Errno:  iochaos.Spec.Errno,
                   Weight: 1,
               },
           },
           Latency:          iochaos.Spec.Delay,
           AttrOverrideSpec: iochaos.Spec.Attr,
           Source:           m.Source,
       })
       key, err := cache.MetaNamespaceKeyFunc(&pod)
       if err != nil {
           return err
       }
       iochaos.Finalizers = utils.InsertFinalizer(iochaos.Finalizers, key)
   }
   r.Log.Info("commiting updates of podiochaos")
   err = m.Commit(ctx)
   if err != nil {
       r.Log.Error(err, "fail to commit")
       return err
   }
   r.Event(iochaos, v1.EventTypeNormal, utils.EventChaosInjected, "")
   return nil
}
```

PodIOChaosリソースのコントローラーでは、Controller Managerがリソースをパラメータにカプセル化し、Chaos Daemonインターフェースを呼び出して処理します。

```go
// Apply flushes io configuration on pod

func (h *Handler) Apply(ctx context.Context, chaos *v1alpha1.PodIoChaos) error {
   h.Log.Info("updating io chaos", "pod", chaos.Namespace+"/"+chaos.Name, "spec", chaos.Spec)
   ...
   res, err := pbClient.ApplyIoChaos(ctx, &pb.ApplyIoChaosRequest{
       Actions:     input,
       Volume:      chaos.Spec.VolumeMountPath,
       ContainerId: containerID,
       Instance:  chaos.Spec.Pid,
       StartTime: chaos.Spec.StartTime,
   })
   if err != nil {
       return err
   }
   chaos.Spec.Pid = res.Instance
   chaos.Spec.StartTime = res.StartTime
   chaos.OwnerReferences = []metav1.OwnerReference{
       {
           APIVersion: pod.APIVersion,
           Kind:       pod.Kind,
           Name:       pod.Name,
           UID:        pod.UID,
       },
   }
   return nil
}
```

`pkg/chaosdaemon/iochaos_server.go`ファイルではIOChaosを処理します。このファイルでは、コンテナにFUSEプログラムを注入する必要があります。GitHubのissue [#2305](https://github.com/chaos-mesh/chaos-mesh/issues/2305)で議論されているように、`/usr/local/bin/nsexec -l- p /proc/119186/ns/pid -m /proc/119186/ns/mnt - /usr/local/bin/toda --path /tmp --verbose info`コマンドを実行し、Podと同じnamespaceでtodaプログラムを実行します。

```go
func (s *DaemonServer) ApplyIOChaos(ctx context.Context, in *pb.ApplyIOChaosRequest) (*pb.ApplyIOChaosResponse, error) {
   ...
   pid, err := s.crClient.GetPidFromContainerID(ctx, in.ContainerId)
   if err != nil {
       log.Error(err, "error while getting PID")
       return nil, err
   }
   args := fmt.Sprintf("--path %s --verbose info", in.Volume)
   log.Info("executing", "cmd", todaBin+" "+args)
   processBuilder := bpm.DefaultProcessBuilder(todaBin, strings.Split(args, " ")...).
       EnableLocalMnt().
       SetIdentifier(in.ContainerId)
   if in.EnterNS {
       processBuilder = processBuilder.SetNS(pid, bpm.MountNS).SetNS(pid, bpm.PidNS)
   }
   ...

   // Calls JSON RPC

   client, err := jrpc.DialIO(ctx, receiver, caller)
   if err != nil {
       return nil, err
   }
   cmd := processBuilder.Build()
   procState, err := s.backgroundProcessManager.StartProcess(cmd)
   if err != nil {
       return nil, err
   }
   ...
}
```

以下のコードサンプルは、実行コマンドを構築します。これらのコマンドは、runcの基盤となるnamespace隔離の実装です：

```go
// GetNsPath returns corresponding namespace path

func GetNsPath(pid uint32, typ NsType) string {
   return fmt.Sprintf("%s/%d/ns/%s", DefaultProcPrefix, pid, string(typ))
}

// SetNS sets the namespace of the process

func (b *ProcessBuilder) SetNS(pid uint32, typ NsType) *ProcessBuilder {
   return b.SetNSOpt([]nsOption{{
       Typ:  typ,
       Path: GetNsPath(pid, typ),
   }})
}

// Build builds the process

func (b *ProcessBuilder) Build() *ManagedProcess {
   args := b.args
   cmd := b.cmd
   if len(b.nsOptions) > 0 {
       args = append([]string{"--", cmd}, args...)
       for _, option := range b.nsOptions {
           args = append([]string{"-" + nsArgMap[option.Typ], option.Path}, args...)
       }
       if b.localMnt {
           args = append([]string{"-l"}, args...)
       }
       cmd = nsexecPath
   }
   ...
}
```

## コントロールプレーン

Chaos MeshはApache 2.0ライセンスのオープンソースカオスエンジニアリングシステムです。前述のように、豊富な機能と良好なエコシステムを備えています。メンテナンスチームは、カオスシステムに基づいて[`chaos-mesh/toda`](https://github.com/chaos-mesh/toda) FUSE、[`chaos-mesh/k8s_dns_chaos`](https://github.com/chaos-mesh/k8s_dns_chaos) CoreDNSカオスプラグイン、およびBerkeley Packet Filter (BPF)ベースのカーネルエラーインジェクション[`chaos-mesh/bpfki`](https://github.com/chaos-mesh/bpfki)を開発しました。

ここでは、エンドユーザー向けのカオスエンジニアリングプラットフォームを構築するために必要なサーバーサイドコードについて説明します。この実装はあくまで例であり、必ずしも最良の例ではありません。実際のプラットフォームでの開発実践を見たい場合は、Chaos Meshの[Dashboard](https://github.com/chaos-mesh/chaos-mesh/tree/master/pkg/dashboard)を参照してください。これは[`uber-go/fx`](https://github.com/uber-go/fx)依存性注入フレームワークと、コントローラーランタイムのマネージャーモードを使用しています。

### Chaos Meshの主要機能

以下のChaos Meshのワークフローに示すように、YAMLをKubernetes APIに送信するサーバーを実装する必要があります。Chaos Controller Managerは、複雑なルール検証とChaos Daemonへのルール配信を実装します。独自のプラットフォームでChaos Meshを使用する場合は、CRDリソースを作成するプロセスに接続するだけで済みます。

![Chaos Meshの基本的なワークフロー](/img/blog/chaos-mesh-basic-workflow.png)

Chaos Meshウェブサイトの例を見てみましょう：

```go
import (
   "context"
   "github.com/pingcap/chaos-mesh/api/v1alpha1"
   "sigs.k8s.io/controller-runtime/pkg/client"
)

func main() {
   ...
   delay := &chaosv1alpha1.NetworkChaos{
       Spec: chaosv1alpha1.NetworkChaosSpec{...},
   }
   k8sClient := client.New(conf, client.Options{ Scheme: scheme.Scheme })
   k8sClient.Create(context.TODO(), delay)
   k8sClient.Delete(context.TODO(), delay)
}
```

Chaos Meshは、すべてのCRDに対応するAPIを提供します。Kubernetes [API Machinery SIG](https://github.com/kubernetes/community/tree/master/sig-api-machinery)が開発した[controller-runtime](https://github.com/kubernetes-sigs/controller-runtime)を使用して、Kubernetes APIとのやり取りを簡素化します。

### カオスの注入

プログラムを呼び出して`PodKill`リソースを作成したい場合を考えます。リソースがKubernetes APIサーバーに送信されると、Chaos Controller Managerの[validating admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)を通過してデータが検証されます。カオス実験を作成する際、admission controllerが入力データの検証に失敗すると、クライアントにエラーが返されます。具体的なパラメータについては、[YAML設定ファイルを使用した実験の作成](https://chaos-mesh.org/docs/simulate-pod-chaos-on-kubernetes/#create-experiments-using-yaml-configuration-files)を参照してください。

`NewClient`はKubernetes APIクライアントを作成します。この例を参照できます：

```go
package main

import (
   "context"
   "controlpanel"
   "log"
   "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
   "github.com/pkg/errors"
   metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func applyPodKill(name, namespace string, labels map[string]string) error {
   cli, err := controlpanel.NewClient()
   if err != nil {
       return errors.Wrap(err, "create client")
   }
   cr := &v1alpha1.PodChaos{
       ObjectMeta: metav1.ObjectMeta{
           GenerateName: name,
           Namespace:    namespace,
       },
       Spec: v1alpha1.PodChaosSpec{
           Action: v1alpha1.PodKillAction,
           ContainerSelector: v1alpha1.ContainerSelector{
               PodSelector: v1alpha1.PodSelector{
                   Mode: v1alpha1.OnePodMode,
                   Selector: v1alpha1.PodSelectorSpec{
                       Namespaces:     []string{namespace},
                       LabelSelectors: labels,
                   },
               },
           },
       },
   }

   if err := cli.Create(context.Background(), cr); err != nil {
       return errors.Wrap(err, "create podkill")
   }
   return nil
}
```

実行プログラムのログ出力は以下の通りです：

```bash
I1021 00:51:55.225502   23781 request.go:665] Waited for 1.033116256s due to client-side throttling, not priority and fairness, request: GET:https://***
2021/10/21 00:51:56 apply podkill
```

kubectlを使用して`PodKill`リソースのステータスを確認します：

```bash
$ k describe podchaos.chaos-mesh.org -n dev podkillvjn77
Name:         podkillvjn77
Namespace:    dev
Labels:       <none>
Annotations:  <none>
API Version:  chaos-mesh.org/v1alpha1
Kind:         PodChaos

Metadata:
 Creation Timestamp:  2021-10-20T16:51:56Z
 Finalizers:
   chaos-mesh/records
 Generate Name:     podkill
 Generation:        7
 Resource Version:  938921488
 Self Link:         /apis/chaos-mesh.org/v1alpha1/namespaces/dev/podchaos/podkillvjn77
 UID:               afbb40b3-ade8-48ba-89db-04918d89fd0b

Spec:
 Action:        pod-kill
 Grace Period:  0
 Mode:          one
 Selector:
   Label Selectors:
     app:  nginx
   Namespaces:
     dev

Status:
 Conditions:
   Reason:
   Status:  False
   Type:    Paused
   Reason:
   Status:  True
   Type:    Selected
   Reason:
   Status:  True
   Type:    AllInjected
   Reason:
   Status:  False
   Type:    AllRecovered

 Experiment:
   Container Records:
     Id:            dev/nginx
     Phase:         Injected
     Selector Key:  .
   Desired Phase:   Run

Events:
 Type    Reason           Age    From          Message
 ----    ------           ----   ----          -------
 Normal  FinalizerInited  6m35s  finalizer     Finalizer has been inited
 Normal  Updated          6m35s  finalizer     Successfully update finalizer of resource
 Normal  Updated          6m35s  records       Successfully update records of resource
 Normal  Updated          6m35s  desiredphase  Successfully update desiredPhase of resource
 Normal  Applied          6m35s  records       Successfully apply chaos for dev/nginx
 Normal  Updated          6m35s  records       Successfully update records of resource
```

コントロールプレーンはまた、Chaosリソースをクエリおよび取得する必要があります。これにより、プラットフォームユーザーはすべてのカオス実験の実施状況を確認し、管理できます。これを実現するために、`REST` APIを呼び出して`Get`または`List`リクエストを送信できます。しかし実際には、細部に注意が必要です。当社では、コントローラーが毎回リソースデータの全量をリクエストするたびに、Kubernetes APIサーバーの負荷が増加することが観察されています。

[controller-runtimeクライアントの使用方法](https://zoetrope.github.io/kubebuilder-training/controller-runtime/client.html)（日本語）のチュートリアルを読むことをお勧めします。日本語が理解できなくても、ソースコードを読むことで多くのことを学べます。このチュートリアルには多くの詳細が含まれています。例えば、デフォルトではcontroller-runtimeはkubeconfig、フラグ、環境変数、およびPodに自動的にマウントされたサービスアカウントを複数の場所から読み取ります。[`armosec/kubescape`](https://github.com/armosec/kubescape)の[Pull request #21](https://github.com/armosec/kubescape/pull/21)はこの機能を使用しています。このチュートリアルには、ページネーション、オブジェクトの更新や上書きなど、一般的な操作も含まれています。これほど詳細な英語のチュートリアルは見たことがありません。

以下は`Get`および`List`リクエストの例です：

```go
package controlpanel

import (
   "context"
   "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
   "github.com/pkg/errors"
   "sigs.k8s.io/controller-runtime/pkg/client"
)

func GetPodChaos(name, namespace string) (*v1alpha1.PodChaos, error) {
   cli := mgr.GetClient()
   item := new(v1alpha1.PodChaos)
   if err := cli.Get(context.Background(), client.ObjectKey{Name: name, Namespace: namespace}, item); err != nil {
       return nil, errors.Wrap(err, "get cr")
   }
   return item, nil
}

func ListPodChaos(namespace string, labels map[string]string) ([]v1alpha1.PodChaos, error) {
   cli := mgr.GetClient()
   list := new(v1alpha1.PodChaosList)
   if err := cli.List(context.Background(), list, client.InNamespace(namespace), client.MatchingLabels(labels)); err != nil {
       return nil, err
   }
   return list.Items, nil
}
```

この例ではマネージャーを使用しています。このモードでは、キャッシュメカニズムが大量のデータを繰り返し取得するのを防ぎます。以下の[図](https://zoetrope.github.io/kubebuilder-training/controller-runtime/client.html)にワークフローを示します：

1. Podを取得する。

2. `List`リクエストの全データを初めて取得する。

3. watchデータが変更されたときにキャッシュを更新する。

![Listリクエスト](/img/blog/list-request.png)

### カオスのオーケストレーション

CRI（Container Runtime Interface）ランタイムは、コンテナの安定した動作をサポートする強力な基盤分離機能を提供します。しかし、より複雑でスケーラブルなシナリオでは、コンテナオーケストレーションが必要です。Chaos Meshはまた、[`Schedule`](https://chaos-mesh.org/docs/define-scheduling-rules/)と[`Workflow`](https://chaos-mesh.org/docs/create-chaos-mesh-workflow/)機能を提供します。設定された`Cron`時間に基づいて、`Schedule`は定期的かつ間隔を空けて障害をトリガーできます。`Workflow`はArgo Workflowsのように複数の障害テストをスケジュールできます。

Chaos Controller Managerが大部分の作業を行ってくれます。コントロールプレーンは主にこれらのYAMLリソースを管理します。エンドユーザーに提供したい機能だけを考慮すればよいのです。

### プラットフォーム機能

以下の図はChaos Meshダッシュボードを示しています。エンドユーザーに提供すべき機能を考慮する必要があります。

![Chaos Meshダッシュボード](/img/blog/chaos-mesh-dashboard-k8s.png)

ダッシュボードから、プラットフォームには以下の機能があることがわかります：

- カオスインジェクション
- Podクラッシュ
- ネットワーク障害
- 負荷テスト
- I/O障害
- イベント追跡
- 関連アラーム
- タイミングテレメトリー

Chaos Meshに興味があり、改善に貢献したい場合は、[Slackチャンネル](https://slack.cncf.io/)（#project-chaos-mesh）に参加するか、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやイシューを提出してください。