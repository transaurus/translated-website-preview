---
title: Upgrade from 2.1 to 2.2
---

Helm Charts 2.2.0リリースではいくつかの変更が行われています。このドキュメントでは、2.1.xから2.2.0への移行方法を説明します。

## Helmを使用したアップグレード

### ステップ1: Chaos Mesh Helmリポジトリの追加/更新

HelmリポジトリにChaos Meshリポジトリを追加し、更新します:

```bash
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm repo update
```

### ステップ2: `values.yaml`ファイルの移行

特定の`values.yaml`を使用してChaos Meshをインストールした場合、カスタマイズした設定をChaos Mesh 2.2.0の`values.yaml`に適用することを推奨します。

以下のコマンドでデフォルトの`values.yaml`を取得できます:

```bash
helm show values chaos-mesh/chaos-mesh --version 2.2.0 > values.yaml
```

変更された設定が分からない場合、その特定の機能に依存していない可能性が高く、通常は無視しても問題ありません。

以下はHelm Chartの変更点リストです:

- 新しい設定: `chaosDaemon.mtls.enabled`はchaos-controller-managerとchaos-daemon間のmtls使用を表します。
- 新しい設定: `webhook.caBundlePEM`はWebhookサーバーで使用されるCAバンドルを表します。
- 変更された値: `dashboard.serviceAccount`が`chaos-controller-manager`から`chaos-dashboard`に変更
- 変更された値: `webhook.FailurePolicy`が`Ignore`から`Fail`に変更

:::note

詳細な説明については、[README](https://github.com/chaos-mesh/chaos-mesh/blob/v2.2.0/helm/chaos-mesh/README.md)を参照してください。

:::

### ステップ3: CRDの更新

`Kubernetes >= 1.16`の場合、以下のコマンドを実行して最新のCRDを適用できます:

```bash
kubectl replace -f https://mirrors.chaos-mesh.org/v2.2.0/crd.yaml
```

`Kubernetes <= 1.15`の場合、以下のコマンドを実行して最新のCRDを適用できます:

```bash
kubectl replace -f https://mirrors.chaos-mesh.org/v2.2.0/crd-v1beta1.yaml
```

:::caution

Chaos Mesh 2.2.xはKubernetes < 1.19をサポートする最後のリリースシリーズとなります。

:::

### ステップ4: `helm upgrade`によるChaos Meshのアップグレード

以下のコマンドを実行してChaos Meshを2.2.0にアップグレードできます:

```bash
helm upgrade <release-name> chaos-mesh/chaos-mesh --namespace=<namespace> --version=2.2.0 <--other-required-flags>
```

## コミュニティへの質問

Chaos Meshのアップグレードに関する質問がある場合は、[Slackチャンネル](https://cloud-native.slack.com/archives/C0193VAV272)、GitHubの[Issues](https://github.com/chaos-mesh/chaos-mesh/issues/new?assignees=&labels=&template=question.md)、[Discussions](https://github.com/chaos-mesh/chaos-mesh/discussions/new)でお気軽にお問い合わせください。