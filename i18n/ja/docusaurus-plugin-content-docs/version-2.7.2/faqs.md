---
title: FAQs
---

import PickHelmVersion from '@site/src/components/PickHelmVersion'

### Kubernetesクラスタをデプロイしていない場合、Chaos Meshを使用してカオス実験を作成できますか？

いいえ。代わりに、[`chaosd`](https://github.com/chaos-mesh/chaosd/)を使用してKubernetesなしで障害を注入できます。

### Chaos Meshをデプロイし、PodChaos実験は正常に作成できましたが、NetworkChaos/TimeChaos実験の作成に失敗しました。ログは以下の通りです：

```console
2020-06-18T02:49:15.160Z ERROR controllers.TimeChaos failed to apply chaos on all pods {"reconciler": "timechaos", "error": "rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial tcp xx.xx.xx.xx:xxxx: connect: connection refused\""}
```

この原因は、`chaos-controller-manager`が`chaos-daemon`に接続できなかったためです。まずPodのネットワークとその[ポリシー](https://kubernetes.io/docs/concepts/services-networking/network-policies/)を確認してください。

すべてが正常であれば、以下のように`hostNetwork`パラメータを使用してこの問題を解決できる可能性があります：

<PickHelmVersion>{`helm upgrade chaos-mesh chaos-mesh/chaos-mesh -n chaos-mesh --version latest --set chaosDaemon.hostNetwork=true`}</PickHelmVersion>

参考: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#hostport-services-do-not-work

### デフォルトのGoogle Cloud管理者アカウントではカオス実験の作成が禁止されています。どうすれば修正できますか？

デフォルトのGoogle Cloud管理者ユーザーは`AdmissionReview`でチェックされません。管理者ロールを作成し、そのロールをアカウントに割り当てて、カオス実験を作成する権限を付与する必要があります。例：

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: role-cluster-manager-pdmas
rules:
  - apiGroups: ['']
    resources: ['pods', 'namespaces']
    verbs: ['get', 'watch', 'list']
  - apiGroups:
      - chaos-mesh.org
    resources: ['*']
    verbs: ['get', 'list', 'watch', 'create', 'delete', 'patch', 'update']
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-manager-binding
  namespace: chaos-mesh
subjects:
  # Google Cloud user account
  - kind: User
    name: USER_ACCOUNT
roleRef:
  kind: ClusterRole
  name: role-cluster-manager-pdmas
  apiGroup: rbac.authorization.k8s.io
```

上記の`USER_ACCOUNT`はGoogle Cloudユーザーのメールアドレスに置き換えてください。

### Daemonが`version 1.41 is too new. The maximum supported API version is 1.39`に類似したエラーをスローする

これは、Dockerデーモンが受け入れ可能な最大APIバージョンが`1.39`であるのに対し、`chaos-daemon`のクライアントがデフォルトで`1.41`を使用していることを示しています。この問題を解決するには以下のオプションを選択できます：

1. Dockerを新しいバージョンにアップグレードする。
2. Helm install/upgrade時に`--set chaosDaemon.env.DOCKER_API_VERSION=1.39`を追加する。

## DNSChaos

### OpenShiftでDNSChaosを実行しようとした際、認可に関する問題でプロセスがブロックされました

エラーメッセージが以下のような場合：

```bash
Error creating: pods "chaos-dns-server-123aa56123-" is forbidden: unable to validate against any security context constraint: [spec.containers[0].securityContext.capabilities.add: Invalid value: "NET_BIND_SERVICE": capability may not be added]
```

`chaos-dns-server`にprivileged Security Context Constraints (SCC)を追加する必要があります。

```bash
oc adm policy add-scc-to-user privileged -n chaos-mesh -z chaos-dns-server
```

## インストール

### OpenShiftにChaos Meshをインストールしようとした際、認可に関する問題でインストールプロセスがブロックされました

エラーメッセージが以下のような場合：

```bash
Error creating: pods "chaos-daemon-" is forbidden: unable
 to validate against any security context constraint: [spec.securityContext.hostNetwork:
 Invalid value: true: Host network is not allowed to be used spec.securityContext.hostPID:
 Invalid value: true: Host PID is not allowed to be used spec.securityContext.hostIPC:
 Invalid value: true: Host IPC is not allowed to be used securityContext.runAsUser:
 Invalid value: "hostPath": hostPath volumes are not allowed to be used spec.containers[0].securityContext.volumes[1]:
 Invalid value: true: Host network is not allowed to be used spec.containers[0].securityContext.containers[0].hostPort:
 Invalid value: 31767: Host ports are not allowed to be used spec.containers[0].securityContext.hostPID:
 Invalid value: true: Host PID is not allowed to be used spec.containers[0].securityContext.hostIPC:
......]
```

defaultにprivileged sccを追加する必要があります。

```bash
oc adm policy add-scc-to-user privileged -n chaos-mesh -z chaos-daemon
```

### インストール時に「no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1"」というメッセージで失敗しました

この問題は、Kubernetes v1.15以前のバージョンにChaos Meshをインストールする際に発生します。デフォルトで`apiextensions.k8s.io/v1`を使用していますが、これは2019年9月19日にKubernetes v1.16で導入されました。

Kubernetes v1.16未満にChaos Meshをインストールする場合は、以下の手順に従ってください：

1. `https://mirrors.chaos-mesh.org/<chaos-mesh-version>/crd-v1beta1.yaml` を使用して手動でCRDを作成します。
2. `--validate=false` を追加します。この設定を追加しない場合、CRDとの互換性問題が発生する可能性があります。例: `kubectl create -f https://mirrors.chaos-mesh.org/v2.1.0/crd-v1beta1.yaml --validate=false`
3. Helmを使用してインストールの残りのプロセスを完了し、`helm install` コマンドに `--skip-crds` を追加します。

Kubernetesクラスターのアップグレードを検討することをお勧めします。詳細はKubernetesの[Version Skew Policy](https://kubernetes.io/releases/version-skew-policy/)を参照してください。