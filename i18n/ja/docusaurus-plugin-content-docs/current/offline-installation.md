---
title: Install Chaos Mesh Offline
---

import PickVersion from '@site/src/components/PickVersion'

import PickHelmVersion from '@site/src/components/PickHelmVersion'

import VerifyInstallation from './common/\_verify-installation.md'

import QuickRun from './common/\_quick-run.md'

このドキュメントでは、Chaos Meshをオフライン環境にインストールする方法について説明します。

## 前提条件

Chaos Meshをインストールする前に、オフライン環境にDockerがインストールされており、Kubernetesクラスターがデプロイされていることを確認してください。環境が準備されていない場合は、以下のドキュメントを参照してDockerのインストールとKubernetesクラスターのデプロイを行ってください：

- [Docker](https://www.docker.com/get-started)
- [Kubernetes](https://kubernetes.io/docs/setup/)

## インストールファイルの準備

Chaos Meshをオフラインでインストールする前に、外部ネットワークに接続されたマシンからすべてのChaos Meshイメージとリポジトリの圧縮パッケージをダウンロードし、ダウンロードしたファイルをオフライン環境にコピーする必要があります。

### バージョン番号の指定

外部ネットワークに接続されたマシンで、Chaos Meshのバージョン番号を環境変数として設定します：

<PickVersion>
{`export CHAOS_MESH_VERSION=latest`}
</PickVersion>

### Chaos Meshイメージのダウンロード

外部ネットワークに接続されたマシンで、設定済みのバージョン番号を使用してイメージをプルします：

```bash
docker pull ghcr.io/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION}
docker pull ghcr.io/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION}
docker pull ghcr.io/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION}
```

イメージをtarパッケージとして保存します：

```bash
docker save ghcr.io/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION} > image-chaos-mesh.tar
docker save ghcr.io/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION} > image-chaos-daemon.tar
docker save ghcr.io/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION} > image-chaos-dashboard.tar
```

:::note

DNS障害をシミュレートする場合（例えば、DNS応答がランダムな誤ったIPアドレスを返すようにする）、追加で[`pingcap/coredns`](https://hub.docker.com/r/pingcap/coredns)イメージをプルする必要があります。

:::

### Chaos Meshリポジトリ圧縮パッケージのダウンロード

外部ネットワークに接続されたマシンで、Chaos Meshのzipパッケージをダウンロードします：

<PickVersion isArchive replaced="refs/heads/master">
{`curl -fsSL -o chaos-mesh.zip https://github.com/chaos-mesh/chaos-mesh/archive/refs/heads/master.zip`}
</PickVersion>

### ファイルのコピー

インストールに必要なすべてのファイルをダウンロードした後、これらのファイルをオフライン環境にコピーする必要があります：

- `image-chaos-mesh.tar`
- `image-chaos-daemon.tar`
- `image-chaos-dashboard.tar`
- `chaos-mesh.zip`

## Chaos Meshのインストール

Chaos Meshイメージのtarパッケージとリポジトリのzipパッケージをオフライン環境にコピーした後、以下の手順に従ってChaos Meshをインストールします。

### ステップ1. Chaos Meshイメージのロード

tarパッケージからイメージをロードします：

```bash
docker load < image-chaos-mesh.tar
docker load < image-chaos-daemon.tar
docker load < image-chaos-dashboard.tar
```

### ステップ2. イメージをレジストリにプッシュ

:::note

イメージをレジストリにプッシュする前に、オフライン環境にレジストリがデプロイされていることを確認してください。レジストリがデプロイされていない場合は、[Docker Registry](https://docs.docker.com/registry/)を参照してデプロイ方法を確認してください。

:::

Chaos Meshのバージョンとレジストリのアドレスを環境変数として設定します：

<PickVersion>
{`export CHAOS_MESH_VERSION=latest; export DOCKER_REGISTRY=localhost:5000`}
</PickVersion>

イメージをタグ付けして、イメージがレジストリを指すようにします：

```bash
export CHAOS_MESH_IMAGE=$DOCKER_REGISTRY/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION}
export CHAOS_DAEMON_IMAGE=$DOCKER_REGISTRY/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION}
export CHAOS_DASHBOARD_IMAGE=$DOCKER_REGISTRY/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION}
docker image tag ghcr.io/chaos-mesh/chaos-mesh:${CHAOS_MESH_VERSION} $CHAOS_MESH_IMAGE
docker image tag ghcr.io/chaos-mesh/chaos-daemon:${CHAOS_MESH_VERSION} $CHAOS_DAEMON_IMAGE
docker image tag ghcr.io/chaos-mesh/chaos-dashboard:${CHAOS_MESH_VERSION} $CHAOS_DASHBOARD_IMAGE
```

イメージをレジストリにプッシュします：

```bash
docker push $CHAOS_MESH_IMAGE
docker push $CHAOS_DAEMON_IMAGE
docker push $CHAOS_DASHBOARD_IMAGE
```

### ステップ3. Helmを使用したChaos Meshのインストール

Chaos Meshのzipパッケージを解凍します：

```bash
unzip chaos-mesh.zip -d chaos-mesh && cd chaos-mesh
```

:::note

Kubernetes v1.15（またはそれ以前のバージョン）にChaos Meshをインストールする場合、`kubectl create -f manifests/crd-v1beta1.yaml`を使用して手動でCRDをインストールする必要があります。詳細については、[FAQ](./faqs.md#failed-to-install-chaos-mesh-with-the-message-no-matches-for-kind-customresourcedefinition-in-version-apiextensionsk8siov1)を参照してください。

:::

名前空間を作成します：

```bash
kubectl create ns chaos-mesh
```

インストールコマンドを実行します。インストールコマンドを実行する際には、Chaos Meshの名前空間と各コンポーネントのイメージ値を指定する必要があります：

```bash
helm install chaos-mesh helm/chaos-mesh -n=chaos-mesh --set images.registry=$DOCKER_REGISTRY
```

## インストールの検証

<VerifyInstallation />

## Chaos実験の実行

<QuickRun />