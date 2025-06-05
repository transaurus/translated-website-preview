---
title: Extend Chaos Daemon Interface
---

import PickHelmVersion from '@site/src/components/PickHelmVersion'

[新しいカオス実験タイプを追加する](add-new-chaos-experiment-type.md)では、Chaos Controller Managerのログに`Hello world!`を出力する`HelloWorldChaos`を追加しました。

`HelloWorldChaos`がターゲットPodに障害を注入できるようにするには、Chaos Daemonインターフェースを拡張する必要があります。

:::tip

先に進む前に、[Chaos Meshのアーキテクチャ](./overview.md#architecture-overview)を読むことをお勧めします。

:::

このドキュメントでは以下を扱います:

- [セレクター](#selector)
- [gRPCインターフェースの実装](#implement-the-grpc-interface)
- [HelloWorldChaosの出力を検証](#verify-the-output-of-helloworldchaos)
- [次のステップ](#next-steps)

## セレクター

`api/v1alpha1/helloworldchaos_type.go`では、`ContainerSelector`を含む`HelloWorldSpec`を定義しました:

```go
// HelloWorldChaosSpec defines the desired state of HelloWorldChaos
type HelloWorldChaosSpec struct {
        // ContainerSelector specifies the target for injection
        ContainerSelector `json:",inline"`

        // Duration represents the duration of the chaos
        // +optional
        Duration *string `json:"duration,omitempty"`

        // RemoteCluster represents the remote cluster where the chaos will be deployed
        // +optional
        RemoteCluster string `json:"remoteCluster,omitempty"`
}

//...

// GetSelectorSpecs is a getter for selectors
func (obj *HelloWorldChaos) GetSelectorSpecs() map[string]interface{} {
        return map[string]interface{}{
                ".": &obj.Spec.ContainerSelector,
        }
}
```

Chaos Meshでは、セレクターはカオス実験のスコープ、ターゲットのネームスペース、アノテーション、ラベルなどを定義するために使用されます。

セレクターは、より具体的な値（例：`AWSChaos`の`AWSSelector`）になることもあります。通常、各カオス実験には1つのセレクターのみが必要ですが、`NetworkChaos`のようにネットワーク分割のために2つのオブジェクトとして2つのセレクターが必要な場合など例外もあります。

セレクターに関する詳細は、[カオス実験のスコープを定義する](./define-chaos-experiment-scope.md)を参照してください。

## gRPCインターフェースの実装

Chaos DaemonがChaos Controller Managerからのリクエストを受け入れるようにするには、新しいgRPCインターフェースを実装する必要があります。

1. `pkg/chaosdaemon/pb/chaosdaemon.proto` にRPCを追加します:

   ```proto
   service ChaosDaemon {
      ...

      rpc ExecHelloWorldChaos(ExecHelloWorldRequest) returns (google.protobuf.Empty) {}
   }

   message ExecHelloWorldRequest {
      string container_id = 1;
   }
   ```

   その後、以下のコマンドを実行して関連する `chaosdaemon.pb.go` ファイルを更新する必要があります:

   ```bash
   make proto
   ```

2. Chaos DaemonでgRPCサービスを実装します。

   `pkg/chaosdaemon` ディレクトリに `helloworld_server.go` というファイルを作成し、以下の内容を記述します:

   ```go
   package chaosdaemon

   import (
           "context"

           "github.com/golang/protobuf/ptypes/empty"

           "github.com/chaos-mesh/chaos-mesh/pkg/bpm"
           "github.com/chaos-mesh/chaos-mesh/pkg/chaosdaemon/pb"
   )

   func (s *DaemonServer) ExecHelloWorldChaos(ctx context.Context, req *pb.ExecHelloWorldRequest) (*empty.Empty, error) {
           log := s.getLoggerFromContext(ctx)
           log.Info("ExecHelloWorldChaos", "request", req)

           pid, err := s.crClient.GetPidFromContainerID(ctx, req.ContainerId)
           if err != nil {
                   return nil, err
           }

           cmd := bpm.DefaultProcessBuilder("sh", "-c", "ps aux").
                   SetContext(ctx).
                   SetNS(pid, bpm.MountNS).
                   Build(ctx)
           out, err := cmd.Output()
           if err != nil {
                   return nil, err
           }
           if len(out) != 0 {
                   log.Info("cmd output", "output", string(out))
           }

           return &empty.Empty{}, nil
   }
   ```

   `chaos-daemon` が `ExecHelloWorldChaos` リクエストを受信すると、現在のコンテナ内のプロセス一覧を確認できます。

3. カオス実験を適用する際にgRPCリクエストを送信します。

   すべてのカオス実験にはライフサイクルがあります: `apply` してから `recover` します。ただし、デフォルトでは回復できないカオス実験もあります（例えば、PodChaosのPodKillやHelloWorldChaosなど）。これらはOneShot実験と呼ばれます。`HelloWorldChaos` スキーマで定義した `+chaos-mesh:oneshot=true` を見つけることができます。

   カオスコントローラーマネージャーは、`HelloWorldChaos` が `apply` フェーズにあるときに、カオスデーモンにリクエストを送信する必要があります。これは `controllers/chaosimpl/helloworldchaos/types.go` を更新することで実現します:

   ```go
   func (impl *Impl) Apply(ctx context.Context, index int, records []*v1alpha1.Record, obj v1alpha1.InnerObject) (v1alpha1.Phase, error) {
           impl.Log.Info("Apply helloworld chaos")

           decodedContainer, err := impl.decoder.DecodeContainerRecord(ctx, records[index], obj)
           if err != nil {
                   return v1alpha1.NotInjected, err
           }

           pbClient := decodedContainer.PbClient
           containerId := decodedContainer.ContainerId

           _, err = pbClient.ExecHelloWorldChaos(ctx, &pb.ExecHelloWorldRequest{
                   ContainerId: containerId,
           })
           if err != nil {
                   return v1alpha1.NotInjected, err
           }

           return v1alpha1.Injected, nil
   }

   func (impl *Impl) Recover(ctx context.Context, index int, records []*v1alpha1.Record, obj v1alpha1.InnerObject) (v1alpha1.Phase, error) {
           impl.Log.Info("Recover helloworld chaos")
           return v1alpha1.NotInjected, nil
   }
   ```

   :::info

   `HelloWorldChaos` は **OneShot** 実験であるため、回復する必要はありません。開発するカオス実験のタイプに応じて、必要に応じて回復関数のロジックを実装できます。

   :::

## HelloWorldChaosの出力を検証

これで `HelloWorldChaos` の出力を検証できます:

1. [新しいカオス実験タイプの追加](add-new-chaos-experiment-type.md#step-4-build-docker-images)で説明した通りにDockerイメージをビルドし、クラスタにロードします。

   :::note

   minikubeを使用している場合、一部のバージョンでは同じタグの既存イメージを上書きできません。新しいイメージをロードする前に既存のイメージを削除する必要があるかもしれません。

   :::

2. Chaos Meshを更新します:

   <PickHelmVersion>{`helm upgrade chaos-mesh helm/chaos-mesh -n=chaos-mesh --set controllerManager.leaderElection.enabled=false,dashboard.securityMode=false`}</PickHelmVersion>

3. テスト用のPodをデプロイします:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/chaos-mesh/apps/master/ping/busybox-statefulset.yaml
   ```

4. 以下の内容で`hello-busybox.yaml`ファイルを作成します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: HelloWorldChaos
   metadata:
     name: hello-busybox
     namespace: chaos-mesh
   spec:
     selector:
       namespaces:
         - busybox
     mode: all
     duration: 1h
   ```

5. 実行します:

   ```bash
   kubectl apply -f hello-busybox.yaml
   # helloworldchaos.chaos-mesh.org/hello-busybox created
   ```

   - これで`chaos-controller-manager`のログに`Apply helloworld chaos`が表示されるか確認できます:

     ```bash
     kubectl logs -n chaos-mesh chaos-controller-manager-xxx
     ```

     出力例:

     ```log
     2023-07-16T08:20:46.823Z INFO records records/controller.go:149 apply chaos {"id": "busybox/busybox-0/busybox"}
     2023-07-16T08:20:46.823Z INFO helloworldchaos helloworldchaos/types.go:27 Apply helloworld chaos
     ```

   - Chaos Daemonのログを確認します:

     ```bash
     kubectl logs -n chaos-mesh chaos-daemon-xxx
     ```

     出力例:

     ```log
     2023-07-16T08:20:46.833Z INFO chaos-daemon.daemon-server chaosdaemon/server.go:187 ExecHelloWorldChaos {"namespacedName": "chaos-mesh/hello-busybox", "request": "container_id:\"docker://5e01e76efdec6aa0934afc15bb80e121d58b43c529a6696a01a242f7ac68f201\""}
     2023-07-16T08:20:46.834Z INFO chaos-daemon.daemon-server.background-process-manager.process-builder pb/chaosdaemon.pb.go:4568 build command {"namespacedName": "chaos-mesh/hello-busybox", "command": "/usr/local/bin/nsexec -m /proc/104710/ns/mnt -- sh -c ps aux"}
     2023-07-16T08:20:46.841Z INFO chaos-daemon.daemon-server chaosdaemon/server.go:187 cmd output {"namespacedName": "chaos-mesh/hello-busybox", "output": "PID   USER     TIME  COMMAND\n    1 root      0:00 sh -c echo Container is Running ; sleep 3600\n"}
     2023-07-16T08:20:46.856Z INFO chaos-daemon.daemon-server chaosdaemon/server.go:187 ExecHelloWorldChaos {"namespacedName": "chaos-mesh/hello-busybox", "request": "container_id:\"docker://bab4f632a0358529f7d72d35e014b8c2ce57438102d99d6174dd9df52d093e99\""}
     2023-07-16T08:20:46.864Z INFO chaos-daemon.daemon-server.background-process-manager.process-builder pb/chaosdaemon.pb.go:4568 build command {"namespacedName": "chaos-mesh/hello-busybox", "command": "/usr/local/bin/nsexec -m /proc/104841/ns/mnt -- sh -c ps aux"}
     2023-07-16T08:20:46.867Z INFO chaos-daemon.daemon-server chaosdaemon/server.go:187 cmd output {"namespacedName": "chaos-mesh/hello-busybox", "output": "PID   USER     TIME  COMMAND\n    1 root      0:00 sh -c echo Container is Running ; sleep 3600\n"}
     ```

   2つの異なるPodに対応する`ps aux`の出力が2行表示されます。

## 次のステップ

このプロセスで問題が発生した場合は、Chaos Meshリポジトリに[issue](https://github.com/chaos-mesh/chaos-mesh/issues)を作成してください。

この仕組みに興味がある方は、[controllers/README.md](https://github.com/chaos-mesh/chaos-mesh/blob/master/controllers/README.md)や各種コントローラーのコードを読んでみてください。

これでChaos Meshの開発者になる準備が整いました！[Chaos Mesh issues](https://github.com/chaos-mesh/chaos-mesh/issues)を訪れて、[good first issue](https://github.com/chaos-mesh/chaos-mesh/labels/good%20first%20issue)から始めてみましょう！