---
title: Configure namespace for Chaos experiments
---

import PickHelmVersion from '@site/src/components/PickHelmVersion'

この章では、Chaos実験が指定された名前空間でのみ効果を発揮し、他の指定されていない名前空間を故障注入から保護するように設定する方法について説明します。

## Chaos実験が効果を発揮する範囲の制御

Chaos Meshは、Chaos実験が効果を発揮する範囲を制御する2つの方法を提供します：

- Chaos実験を指定された名前空間でのみ有効にするには、FilterNamespace機能（デフォルトでは無効）を有効にする必要があります。この機能はグローバルスコープで有効になります。この機能を有効にした後、Chaos実験が許可される名前空間にアノテーションを追加できます。アノテーションのない他の名前空間は故障注入から保護されます。
- 単一のChaos実験が効果を発揮する範囲を指定するには、[Chaos実験のスコープを定義する](define-chaos-experiment-scope.md)を参照してください。

## FilterNamespaceの有効化

Chaos Meshをまだインストールしていない場合、Helmを使用してインストールする際に、コマンドに`--set controllerManager.enableFilterNamespace=true`を追加することでこの機能を有効にできます。以下はDockerコンテナ内でのコマンド例です：

<PickHelmVersion>{`helm install chaos-mesh chaos-mesh/chaos-mesh -n chaos-mesh --set controllerManager.enableFilterNamespace=true --version latest`}</PickHelmVersion>

:::note

Helmを使用してインストールする場合、コンテナによってコマンドとパラメータが異なります。詳細は[Helmを使用したChaos Meshのインストール](production-installation-using-helm.md)を参照してください。

:::

Helmを使用してChaos Meshを既にインストールしている場合、`helm upgrade`コマンドで設定を更新することでこの機能を有効にできます。例：

<PickHelmVersion>{`helm upgrade chaos-mesh chaos-mesh/chaos-mesh -n chaos-mesh --version latest --set controllerManager.enableFilterNamespace=true`}</PickHelmVersion>

`helm upgrade`では、コマンドに複数の`--set`を追加することで複数のパラメータを設定できます。後の設定は前の設定を上書きします。例えば、コマンドに`--set controllerManager.enableFilterNamespace=false -set controllerManager.enableFilterNamespace=true`を追加しても、この機能は有効になります。

また、`-f`パラメータを使用してYAMLファイルを指定し、設定を記述することもできます。詳細は[Helm upgrade](https://helm.sh/zh/docs/helm/helm_upgrade/#%E7%AE%80%E4%BB%8B)を参照してください。

## Chaos実験が許可される名前空間へのアノテーション追加

FilterNamespaceが有効になっている場合、Chaos Meshは`chaos-mesh.org/inject=enabled`というアノテーションを含む名前空間にのみ故障を注入します。したがって、Chaos実験を開始する前に、Chaos実験が許可される名前空間にこのアノテーションを追加する必要があります。他の名前空間は故障注入から保護されます。

以下の`kubectl`コマンドを使用して、`namespace`にアノテーションを追加できます：

```bash
kubectl annotate ns $NAMESPACE chaos-mesh.org/inject=enabled
```

上記のコマンドで、`$NAMESPACE`は名前空間の名前（例：`default`）を指します。アノテーションが正常に追加されると、出力は以下のようになります：

```bash
namespace/$NAMESPACE annotated
```

このアノテーションを削除したい場合は、以下のコマンドを使用できます：

```bash
kubectl annotate ns $NAMESPACE chaos-mesh.org/inject-
```

アノテーションが正常に削除されると、出力は以下のようになります：

```bash
namespace/$NAMESPACE annotated
```

## Chaos実験が許可されるすべての名前空間の確認

以下のコマンドを使用して、Chaos実験が許可されるすべての名前空間をリストアップできます：

```bash
kubectl get ns -o jsonpath='{.items[?(@.metadata.annotations.chaos-mesh\.org/inject=="enabled")].metadata.name}'
```

このコマンドは、`chaos-mesh.org/inject=enabled`アノテーションを持つすべての名前空間を出力します。例：

```bash
default
```