---
title: Quick Start
---

import Tabs from '@theme/Tabs'

import TabItem from '@theme/TabItem'

import PickVersion from '@site/src/components/PickVersion'

import VerifyInstallation from './common/\_verify-installation.md'

import QuickRun from './common/\_quick-run.md'

このドキュメントでは、テスト環境やローカル環境でChaos Meshを迅速に開始する方法について説明します。

:::caution

**このドキュメントでは、簡易的な試用のためにスクリプト経由でChaos Meshをインストールします。**

本番環境やその他の厳格な非テストシナリオでChaos Meshをインストールする必要がある場合は、[Helm](https://helm.sh/)を使用することを推奨します。詳細については、[Helmを使用したインストール](production-installation-using-helm.md)を参照してください。

:::

## 環境の準備

試用前に、環境にKubernetesクラスターがデプロイされていることを確認してください。Kubernetesクラスターがまだデプロイされていない場合は、以下のリンクを参照してデプロイを完了させてください：

- [Kubernetes](https://kubernetes.io/docs/setup/)
- [minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [K3s](https://rancher.com/docs/k3s/latest/en/quick-start/)
- [Microk8s](https://microk8s.io/)

## クイックインストール

テスト環境にChaos Meshをインストールするには、以下のスクリプトを実行してください：

<!-- prettier-ignore -->

<Tabs defaultValue="k8s" values={[
  {label: 'K8s', value: 'k8s'},
  {label: 'kind', value: 'kind'},
  {label: 'K3s', value: 'k3s'},
  {label: 'MicroK8s', value: 'microk8s'}
]}>
  <TabItem value="k8s">
    <PickVersion>
    {`curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash`}
    </PickVersion>
  </TabItem>
  <TabItem value="kind">
    <PickVersion>{`curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --local kind`}</PickVersion>

    If you want to specify a `kind` version, add the `--kind-version xx` parameter at the end of the script, for example:

    <PickVersion>
    {`curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --local kind --kind-version v0.20.0`}
    </PickVersion>

  </TabItem>
  <TabItem value="k3s">
    <PickVersion>
    {`curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --k3s`}
    </PickVersion>
  </TabItem>
  <TabItem value="microk8s">
    <PickVersion>
    {`curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --microk8s`}
    </PickVersion>
  </TabItem>
</Tabs>

実行後、Chaos Meshは適切なバージョンのCustomResourceDefinitionsと必要なコンポーネントを自動的にインストールします。

インストールの詳細については、[`install.sh`](https://github.com/chaos-mesh/chaos-mesh/blob/master/install.sh)のソースコードを参照してください。

## インストールの確認

<VerifyInstallation />

## Chaos実験の実行

<QuickRun />

## Chaos Meshのアンインストール

Chaos Meshをアンインストールするには、以下のコマンドを実行してください：

<PickVersion>
{`curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash -s -- --template | kubectl delete -f -`}
</PickVersion>

また、`chaos-mesh`ネームスペースを削除することで直接Chaos Meshをアンインストールすることもできます：

```sh
kubectl delete ns chaos-mesh
```

## よくある質問

### インストール後にルートディレクトリに`local`ディレクトリが作成されるのはなぜですか？

既存の環境に`kind`がインストールされておらず、インストールコマンドを実行する際に`--local kind`パラメータを使用した場合、`install.sh`スクリプトはルートディレクトリの`local`ディレクトリに`kind`を自動的にインストールします。