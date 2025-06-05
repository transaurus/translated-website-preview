---
title: Uninstall Chaos Mesh
---

このドキュメントでは、Chaos Meshのアンインストール方法について説明します。Helmを使用したアンインストール方法と手動でのアンインストール方法が含まれます。また、KubernetesクラスターからChaos Meshのインストールを手動で完全に削除する必要がある場合にも非常に役立ちます。

## Helmを使用したChaos Meshのアンインストール

### ステップ1: Chaos Experimentのクリーンアップ

Chaos Meshをアンインストールする前に、すべてのChaos Experimentが削除されていることを確認してください。以下のコマンドを実行して、Chaos関連のオブジェクトを一覧表示できます:

```shell
for i in $(kubectl api-resources | grep chaos-mesh | awk '{print $1}'); do kubectl get $i -A; done
```

すべてのChaos Experimentが削除されていることを確認したら、Chaos Meshをアンインストールできます。

### ステップ2: Helmリリースの一覧表示

以下のコマンドを実行して、インストールされているHelmリリースを一覧表示できます:

```shell
helm list -A
```

出力は以下のようになります:

```text
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
chaos-mesh-playground   chaos-mesh      1               2021-12-01 22:58:18.037052401 +0800 CST deployed        chaos-mesh-2.1.0        2.1.0
```

これは、Chaos Meshが`chaos-mesh`ネームスペースに`chaos-mesh-playground`という名前のHelmリリースとしてインストールされていることを意味します。これがアンインストール対象のリリースです。

### ステップ3: Helmリリースの削除

対象のHelmリリースを特定したら、以下のコマンドを実行して削除できます:

```shell
helm uninstall chaos-mesh-playground -n chaos-mesh
```

### ステップ4: CRDの削除

`helm uninstall`ではCRDは削除されないため、以下のコマンドを実行して手動で削除できます:

```shell
kubectl delete crd $(kubectl get crd | grep 'chaos-mesh.org' | awk '{print $1}')
```

## Chaos Meshの手動アンインストール

Chaos Meshを`install.sh`スクリプトでインストールした場合、またはインストール後に設定やコンポーネントを変更した場合、またはChaos Meshのアンインストール中に問題が発生した場合、以下の手順で手動でChaos Meshをアンインストールできます。

### ステップ1: Chaos Experimentのクリーンアップ

Chaos Meshをアンインストールする前に、すべてのChaos Experimentが削除されていることを確認してください。以下のコマンドを実行して、Chaos関連のオブジェクトを一覧表示できます:

```shell
for i in $(kubectl api-resources | grep chaos-mesh | awk '{print $1}'); do kubectl get $i -A; done
```

すべてのChaos Experimentが削除されていることを確認したら、Chaos Meshをアンインストールできます。

### ステップ2: Chaos Meshワークロードの削除

Chaos Meshをインストールすると、通常以下のようなコンポーネントが作成されます:

- `chaos-controller-manager`という名前の`Deployment`。Chaos Meshのコントローラー/リコンサイラーです。
- `chaos-daemon`という名前の`DaemonSet`。各Kubernetesワーカーノード上のChaos Meshエージェントです。
- `chaos-dashboard`という名前の`Deployment`。Chaos MeshのWebUIです。
- `chaos-dns-server`という名前の`Deployment`。DNSプロキシサーバーで、DNSChaos機能を有効にした場合にのみ作成されます。

これらのワークロードオブジェクトを削除してください。

次に、対応する`Service`を削除します:

- chaos-daemon
- chaos-dashboard
- chaos-mesh-controller-manager
- chaos-mesh-dns-server

### ステップ3: Chaos Mesh RBACオブジェクトの削除

Chaos Meshをインストールすると、以下のようなRBACオブジェクトが作成されます:

- ClusterRoleBinding
  - chaos-mesh-playground-chaos-controller-manager-cluster-level
  - chaos-mesh-playground-chaos-controller-manager-target-namespace
  - chaos-mesh-playground-chaos-dns-server-cluster-level
  - chaos-mesh-playground-chaos-dns-server-target-namespace
- ClusterRole
  - chaos-mesh-playground-chaos-controller-manager-cluster-level
  - chaos-mesh-playground-chaos-controller-manager-target-namespace
  - chaos-mesh-playground-chaos-dns-server
  - chaos-mesh-playground-chaos-dns-server-cluster-level
- RoleBinding
  - chaos-mesh-playground-chaos-controller-manager-control-plane
  - chaos-mesh-playground-chaos-dns-server-control-plane
- Role
  - chaos-mesh-playground-chaos-controller-manager-control-plane
  - chaos-mesh-playground-chaos-dns-server-control-plane
- ServiceAccount
  - chaos-controller-manager
  - chaos-daemon
  - chaos-dns-server

これらのRBACオブジェクトを削除してください。

### ステップ4: ConfigMapとSecretを削除

Chaos Meshのインストール時に作成されるConfigMapとSecretは以下の通りです:

- ConfigMap
  - chaos-mesh
  - dns-server-config
- Secret
  - chaos-mesh-webhook-certs

これらのConfigMapとSecretオブジェクトを削除してください。

### ステップ5: Webhookを削除

Chaos Meshのインストール時に作成されるWebhookは以下の通りです:

- ValidatingWebhookConfigurations
  - chaos-mesh-validation
  - chaos-mesh-validate-auth
- MutatingWebhookConfigurations
  - chaos-mesh-mutation

これらのWebhookを削除してください。

### ステップ6: CRDを削除

最後に、以下のコマンドを実行してCRDを削除できます:

```shell
kubectl delete crd $(kubectl get crd | grep 'chaos-mesh.org' | awk '{print $1}')
```