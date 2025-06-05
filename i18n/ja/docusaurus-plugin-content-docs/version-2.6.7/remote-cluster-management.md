---
title: Remote Cluster Management
---

## リモートクラスタの概要

Chaos Meshは、リモートKubernetesクラスタを管理し、障害を注入するためのクラスタスコープの`RemoteCluster`リソースを提供します。このドキュメントでは、`RemoteCluster`オブジェクトの作成方法と、それを使用した障害注入の方法について説明します。

:::note

`RemoteCluster`は現在初期段階です。設定や機能（設定移行、バージョン管理、認証など）は今後改善されていきます。問題が発生した場合は、[chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)にイシューを開いて報告してください。

:::

## リモートクラスタの登録

現在のクラスタにインストールされたChaos Meshにリモートクラスタを登録するには、`RemoteCluster`リソースを作成する必要があります。このリソースを作成すると、リモートクラスタに必要なコンポーネントが自動的にインストールされます。以下は`RemoteCluster`リソースの例です：

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: RemoteCluster
metadata:
  name: cluster-xxx
spec:
  namespace: chaos-mesh
  version: 2.6.2
  kubeConfig:
    secretRef:
      name: remote-chaos-mesh.kubeconfig
      namespace: chaos-mesh
      key: kubeconfig
  # configOverride:
  #   dashboard:
  #     create: true
```

これは、`.spec.kubeConfig`フィールドで提供された`KUBECONFIG`を使用して、指定された名前空間に`chaos-mesh` Helmチャートをインストールします。

### フィールドの説明

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| namespace | string | Represent the namespace to install Chaos Mesh components in the remote cluster | None | Yes | chaos-mesh |
| version | string | The version of Chaos Mesh to install in the remote cluster | None | Yes | 2.6.2 |
| kubeConfig.secretRef.name | string | The name of the secret, which is used to store the kubeconfig of remote cluster. This kubeconfig will be used to install chaos-mesh components and inject errors | None | Yes | `remote-chaos-mesh.kubeconfig` |
| kubeConfig.secretRef.namespace | string | The name of the namespace of the kubeconfig secret. | None | Yes | `default` |
| kubeConfig.secretRef.key | string | The key of the kubeconfig in the secret. | None | Yes | `kubeconfig` |
| configOverride | string | Passing helm values during install or upgrade | None | No | `{"dashboard":{"create":true}}` |

## リモートクラスタへの障害注入

登録済みの`RemoteCluster`を使用してリモートクラスタに障害を注入するには、各種Chaosタイプの`.spec`内の`remoteCluster`フィールドを使用します。例：

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: burn-cpu
spec:
  remoteCluster: cluster-xxx
  mode: one
  selector:
    labelSelectors:
      'app.kubernetes.io/component': 'tikv'
  stressors:
    cpu:
      workers: 1
      load: 100
      options: ['--cpu 2', '--timeout 600', '--hdd 1']
  duration: '30s'
```

Chaos Meshは、`cluster-xxx`という名前の`RemoteCluster`に登録されたkubeconfigを使用してリモートクラスタに障害を注入します。対応する`StressChaos`はリモートクラスタに自動的に作成され、ステータスは現在のクラスタに同期されます。これにより、単一のKubernetesクラスタで複数の異なるクラスタに対する障害注入を管理できます。