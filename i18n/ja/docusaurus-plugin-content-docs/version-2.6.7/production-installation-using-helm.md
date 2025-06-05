---
title: Install Chaos Mesh using Helm
---

import Tabs from '@theme/Tabs'

import TabItem from '@theme/TabItem'

import PickVersion from '@site/src/components/PickVersion'

import PickHelmVersion from '@site/src/components/PickHelmVersion'

import VerifyInstallation from './common/\_verify-installation.md'

import QuickRun from './common/\_quick-run.md'

このドキュメントでは、本番環境にChaos Meshをインストールする方法について説明します。

## 前提条件

Chaos Meshをインストールする前に、環境に[Helm](https://helm.sh/docs/intro/install/)がインストールされていることを確認してください。

Helmがインストールされているかどうかを確認するには、次のコマンドを実行します：

```bash
helm version
```

期待される出力は以下の通りです：

```bash
version.BuildInfo{Version:"v3.5.4", GitCommit:"1b5edb69df3d3a08df77c9902dc17af864ff05d1", GitTreeState:"dirty", GoVersion: "go1.16.3"}
```

実際の出力が、`Version`、`GitCommit`、`GitTreeState`、`GoVersion`を含む期待される出力と類似している場合、Helmが正常にインストールされていることを意味します。

:::note

このドキュメントでは、Chaos Meshの操作を行うためにHelm v3を使用しています。環境でHelm v2を使用している場合は、[Helm v2からv3への移行](https://helm.sh/docs/topics/v2_v3_migration/)を参照するか、Helmのバージョンをv2形式に変更してください。

:::

## Helmを使用してChaos Meshをインストール

### ステップ1: Chaos Meshリポジトリを追加

HelmリポジトリにChaos Meshリポジトリを追加します：

```bash
helm repo add chaos-mesh https://charts.chaos-mesh.org
```

### ステップ2: インストール可能なChaos Meshのバージョンを確認

インストール可能なチャートを確認するには、次のコマンドを実行します：

```bash
helm search repo chaos-mesh
```

:::note

上記のコマンドは、最新リリースのチャートを出力します。過去のバージョンをインストールしたい場合は、次のコマンドを実行してすべてのリリース済みバージョンを確認してください：

```bash
helm search repo chaos-mesh -l
```

:::

上記のコマンドが完了すると、Chaos Meshのインストールを開始できます。

### ステップ3: Chaos Meshをインストールする名前空間を作成

Chaos Meshは`chaos-mesh`名前空間にインストールすることを推奨しますが、任意の名前空間を指定してインストールすることも可能です：

```bash
kubectl create ns chaos-mesh
```

### ステップ4: 異なる環境にChaos Meshをインストール

:::note

Kubernetes v1.15（またはそれ以前のバージョン）にChaos Meshをインストールする場合、CRDを手動でインストールする必要があります。詳細については、[FAQ](./faqs.md#failed-to-install-chaos-mesh-with-the-message-no-matches-for-kind-customresourcedefinition-in-version-apiextensionsk8siov1)を参照してください。

:::

異なるコンテナランタイムのデーモンは異なるソケットパスでリッスンしているため、インストール時に適切な値を設定する必要があります。以下のインストールコマンドを環境に応じて実行してください。

<!-- prettier-ignore -->

<Tabs defaultValue="docker" values={[
  {label: 'Docker', value: 'docker'},
  {label: 'Containerd', value: 'containerd'},
  {label: 'K3s', value: 'k3s'},
  {label: 'MicroK8s', value: 'microk8s'},
  {label: 'CRI-O', value: 'cri-o'}
]}>
  <TabItem value="docker">
    <PickHelmVersion>{`\# Default to /var/run/docker.sock\nhelm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version latest`}</PickHelmVersion>
  </TabItem>
  <TabItem value="containerd">
    <PickHelmVersion>{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock --version latest`}</PickHelmVersion>
  </TabItem>
  <TabItem value="k3s">
    <PickHelmVersion>{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/k3s/containerd/containerd.sock --version latest`}</PickHelmVersion>
  </TabItem>
  <TabItem value="microk8s">
    <PickHelmVersion>{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/var/snap/microk8s/common/run/containerd.sock --version latest`}</PickHelmVersion>
  </TabItem>
  <TabItem value="cri-o">
    <PickHelmVersion>{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=crio --set chaosDaemon.socketPath=/var/run/crio/crio.sock --version latest`}</PickHelmVersion>
  </TabItem>
</Tabs>

:::info

特定のバージョンのChaos Meshをインストールするには、`helm install`の後に`--version x.y.z`パラメータを追加します。例：`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version 2.1.0`。

:::

:::tip

高可用性を確保するため、Chaos Meshはデフォルトで`leader-election`機能を有効にしています。この機能を使用する必要がない場合は、`--set controllerManager.leaderElection.enabled=false`で手動で無効にできます。

> バージョン`<2.6.1`の場合、コントローラマネージャーのレプリカ数を1つに減らすために`--set controllerManager.replicaCount=1`を設定する必要があります。

:::

## インストールの確認

<VerifyInstallation />

## Chaos実験を実行

<QuickRun />

## Chaos Meshのアップグレード

Chaos Meshをアップグレードするには、次のコマンドを実行します：

```bash
helm upgrade chaos-mesh chaos-mesh/chaos-mesh
```

:::info

特定のバージョンのChaos Meshにアップグレードするには、`helm upgrade`の後に`--version x.y.z`パラメータを追加します。例: `helm upgrade chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version 2.1.0`

:::

:::note

非Docker環境でChaos Meshをアップグレードした場合、[Step 4: 異なる環境でのChaos Meshインストール](#step-4-install-chaos-mesh-in-different-environments)で説明されている適切なパラメータを追加する必要があります。

:::

設定を変更するには、必要に応じて異なる値を設定します。例えば、以下のコマンドを実行して`chaos-dashboard`をアップグレードおよびアンインストールします:

<PickHelmVersion>{`helm upgrade chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version latest --set dashboard.create=false`}</PickHelmVersion>

:::note

すべての値とその使用方法については、[all values](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/values.yaml)を参照してください。

:::

:::caution

現在、Helmのアップグレード時に最新のCustomResourceDefinition (CRD)が適用されないため、エラーが発生する可能性があります。この状況を回避するには、最新のCRDを手動で適用できます:

<PickVersion>
{`curl -sSL https://mirrors.chaos-mesh.org/latest/crd.yaml | kubectl create -f -`}
</PickVersion>

:::

## Chaos Meshのアンインストール

Chaos Meshをアンインストールするには、以下のコマンドを実行します:

```bash
helm uninstall chaos-mesh -n chaos-mesh
```

## よくある質問

### Chaos Meshの最新バージョンをインストールするには？

`helm/chaos-mesh/values.yaml`ファイルには最新バージョン（masterブランチ）のイメージが定義されています。Chaos Meshの最新バージョンをインストールするには、以下のコマンドを実行します:

```bash
# Clone repository
git clone https://github.com/chaos-mesh/chaos-mesh.git
cd chaos-mesh

helm install chaos-mesh helm/chaos-mesh -n=chaos-mesh
```

### セーフモードを無効にするには？

セーフモードでは、Chaos Meshダッシュボードへの認証を無効にできますが、非本番環境でのみ使用してください。セーフモードはデフォルトで**有効**になっています。セーフモードを無効にするには、インストールまたはアップグレード時に`dashboard.securityMode`を`false`に指定します:

<PickHelmVersion>
{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set dashboard.securityMode=false --version latest`}
</PickHelmVersion>

### Chaos Dashboardデータを永続化するには

Chaos Dashboardはデフォルトのデータベースエンジンとして`SQLite`を使用します。[PV (Persistent Volumes)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)が無効になっている場合、再起動が発生するとChaos Dashboardのデータは失われます。データ損失を防ぐには、[Chaos Dashboardデータの永続化](persistence-dashboard.md)ドキュメントを参照して、Chaos Dashboard用にPVを有効にするか、`MySQL`および`PostgreSQL`をデータベースエンジンとして設定できます。