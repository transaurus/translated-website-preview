---
title: Configure the Development Environment
---

このドキュメントでは、Chaos Meshのローカル開発環境を設定する方法について説明します。

Chaos Meshのほとんどのコンポーネントは**Linux専用に設計されている**ため、開発環境もLinux上で動作するように設定することを推奨します。例えば、仮想マシンやWSL 2を使用し、VSCode Remoteをエディタとして利用します。

このドキュメントでは、特定のLinuxディストリビューションに依存せず、Linuxを使用していることを前提としています。Windows/MacOSを利用する場合、動作させるためにいくつかの工夫が必要になる場合があります（例えば、環境によってはmakeターゲットが失敗する可能性があります）。

## 設定要件

設定前に、以下の開発ツールをインストールすることを推奨します。これらの多くは既に環境にインストールされている可能性があります：

- [make](https://www.gnu.org/software/make/)
- [docker](https://docs.docker.com/install/)
- [golang](https://go.dev/doc/install)、`v1.18`以降のバージョン
- [gcc](https://gcc.gnu.org/)
- [helm](https://helm.sh/)、`v3.9.0`以降のバージョン
- [minikube](https://minikube.sigs.k8s.io/docs/start/)

オプション：

- [nodejs](https://nodejs.org/en/)と[pnpm](https://pnpm.io/)、Chaos Dashboardの開発用

## Chaos Meshのコンパイル

インストール後、以下の手順に従ってChaos Meshをコンパイルします。

1. Chaos Meshリポジトリをローカルサーバーにクローンします：

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh.git
   cd chaos-mesh
   ```

2. [Docker](https://docs.docker.com/install/)がインストールされ、実行されていることを確認してください。

   :::info

   Chaos MeshはコンテナイメージのビルドにDockerを利用しており、これは本番環境との一貫性を保つためです。

   :::

3. Chaos Meshをコンパイルします：

   ```bash
   UI=1 make image
   ```

   :::tip

   `UI=1`はChaos Dashboardのユーザーインターフェースをコンパイルすることを意味します。不要な場合はこの環境変数を省略できます。

   :::

   :::tip

   イメージのタグを指定したい場合は、`UI=1 make IMAGE_TAG=dev image`を使用できます。

   :::

   コンパイル後、以下のコンテナイメージが生成されます：

   - `ghcr.io/chaos-mesh/chaos-dashboard:latest`
   - `ghcr.io/chaos-mesh/chaos-mesh:latest`
   - `ghcr.io/chaos-mesh/chaos-daemon:latest`

## ローカルのminikube KubernetesクラスターでChaos Meshを実行

コンパイル後、ローカルのKubernetesクラスターでChaos Meshを実行できます。

1. minikubeでローカルKubernetesクラスターを起動します：

   ```bash
   minikube start
   ```

2. コンテナイメージをminikubeにロードします：

   ```bash
   minikube image load ghcr.io/chaos-mesh/chaos-dashboard:latest
   minikube image load ghcr.io/chaos-mesh/chaos-mesh:latest
   minikube image load ghcr.io/chaos-mesh/chaos-daemon:latest
   ```

3. Helmを使用してChaos Meshをインストールします：

   ```bash
   helm upgrade --install chaos-mesh-debug ./helm/chaos-mesh -n=chaos-mesh-debug --create-namespace
   ```

:::tip

`minikube image load`は多くの時間を要するため、開発中に何度もイメージをロードすることを避けるための工夫があります。ホストのDockerではなく、minikubeノードのDockerを使用します：

```bash
minikube start --mount --mount-string "$(pwd):$(pwd)"
eval $(minikube -p minikube docker-env)
UI=1 make image
```

:::

## ローカル環境でのChaos Meshのデバッグ

ローカル環境でChaos Meshをデバッグするために、[delve](https://github.com/go-delve/delve)とリモートデバッグを利用できます。

1. `DEBUGGER=1` を指定して Chaos Mesh をコンパイル:

   ```bash
   UI=1 DEBUGGER=1 make image
   ```

2. コンテナイメージを minikube にロード:

   ```bash
   minikube image load ghcr.io/chaos-mesh/chaos-mesh:latest
   minikube image load ghcr.io/chaos-mesh/chaos-daemon:latest
   minikube image load ghcr.io/chaos-mesh/chaos-dashboard:latest
   ```

3. Chaos Mesh をインストールしリモートデバッグを有効化:

   ```bash
   helm upgrade --install chaos-mesh-debug ./helm/chaos-mesh -n=chaos-mesh-debug --create-namespace --set chaosDlv.enable=true --set controllerManager.leaderElection.enabled=false
   ```

   :::note

   高可用性を確保するため、Chaos Mesh はデフォルトで `leader-election` 機能を有効にしており、`chaos-controller-manager` のレプリカを3つ作成します。デバッグを容易にするため、`controllerManager.leaderElection.enabled=false` を設定して Chaos Mesh が `chaos-controller-manager` のインスタンスを1つだけ作成するようにします。

   詳細は [異なる環境での Chaos Mesh インストール](production-installation-using-helm.md#step-4-install-chaos-mesh-in-different-environments) を参照してください。

   :::

4. ポートフォワーディングの設定とIDEでのリモートデバッガ接続:

   `kubectl port-forward` を使用して、delve デバッグサーバーをローカルポートに転送できます。

   例えば、`chaos-controller-manger` をデバッグする場合、以下のコマンドを実行します:

   ```bash
   kubectl -n chaos-mesh-debug port-forward chaos-controller-manager-766dc8488d-7n5bq 58000:8000
   ```

   これにより、リモートの delve デバッガサーバーに `127.0.0.1:58000` でアクセス可能になります。

   :::info

   pod 内では delve デバッグサーバー用に常に `8000` ポートを使用します。これは規約であり、helm テンプレートファイルで確認できます。

   :::

   IDE を設定してリモートデバッガに接続します。以下は設定例です:

   - Goland の場合: [Attach to running Go processes with the debugger#Attach to a process on a remote machine](https://www.jetbrains.com/help/go/attach-to-running-go-processes-with-debugger.html#attach-to-a-process-on-a-remote-machine) を参照

   - VSCode の場合: [vscode-go - Debugging#Remote Debugging](https://github.com/golang/vscode-go/blob/master/docs/debugging.md#remote-debugging) を参照

詳細については [chaos-dlv コンテナイメージの README.md](https://github.com/chaos-mesh/chaos-mesh/blob/master/images/chaos-dlv/README.md) を参照してください。

## 次のステップ

上記の準備が完了したら、[新しい Chaos 実験タイプの追加](add-new-chaos-experiment-type.md) を試すことができます。

## FAQ

### MacOS で `make` 実行時に `error obtaining VCS status: exit status 128` が発生する

この原因は https://github.blog/2022-04-12-git-security-vulnerability-announced/ に関連しています。まずこの記事を読むことを推奨します。

Chaos Mesh は現在のユーザー（`make` を実行したユーザー）でコンテナ（`dev-env` または `build-env`）を起動します。[get_env_shell.py#L81C10-L81C10](https://github.com/chaos-mesh/chaos-mesh/blob/813b650c02e0b065ae5c4707725c346929ab1847/build/get_env_shell.py#L81C10-L81C10) で適切な `--user` フラグを確認できます。Git がトップレベルの `.git` ディレクトリを探す際、ディレクトリ走査中に現在のユーザーから所有権が変更されると停止します。

現在の暫定対策として、`--user` の行をコメントアウトすることができます。