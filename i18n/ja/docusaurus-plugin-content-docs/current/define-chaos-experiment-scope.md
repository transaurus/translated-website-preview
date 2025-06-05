---
title: Define the Scope of Chaos Experiments
---

このドキュメントでは、単一のChaos実験のスコープを定義する方法について説明します。これにより、障害の影響範囲を正確に制御することが可能になります。

## 実験スコープの概要

Chaos Meshでは、セレクタを指定することで単一のChaos実験のスコープを定義できます。

異なるタイプのセレクタは異なるフィルタリングルールに対応しています。実験のスコープを定義するために、1つ以上のセレクタをChaos実験で指定できます。複数のセレクタが同時に指定された場合、現在の実験ターゲットはすべての指定されたセレクタのルールを同時に満たす必要があります。

Chaos実験を作成する際、Chaos Meshは以下の方法で実験スコープを定義することをサポートしています。必要に応じて以下のいずれかの方法を選択できます：

- YAML設定ファイルで実験スコープを定義する
- Chaos Dashboardで実験スコープを定義する

## YAML設定ファイルで実験スコープを定義する

このセクションでは、異なるセレクタタイプの意味と使用方法を紹介し、YAMLファイルでの設定例を提供します。YAMLファイルで実験スコープを定義する際、スコープフィルタリングの必要性に応じて1つ以上のセレクタを指定できます。

### ネームスペースセレクタ

- 実験対象のPodのネームスペースを指定します。
- データ型: 文字列配列型。
- このセレクタが空または指定されていない場合、Chaos Meshは現在のChaos実験のネームスペースに設定します。

YAMLファイルを使用して実験を作成する際、セレクタを設定する必要があります。例：

```yaml
spec:
  selector:
    namespaces:
      - 'app-ns'
```

### ラベルセレクタ

- 実験対象のPodが持つ必要がある[ラベル](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)を指定します。
- データ型: キーと値のペア。
- 複数のラベルが指定された場合、実験対象はこのセレクタで指定されたすべてのラベルを持っている必要があります。

YAMLファイルを使用して実験を作成する際、セレクタを設定する必要があります。例：

```yaml
spec:
  selector:
    labelSelectors:
      'app.kubernetes.io/component': 'tikv'
```

### 式セレクタ

- 実験対象のPodを指定するためのラベルのルールを定義する[式](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements)のセットを指定します。
- このセレクタを使用して、特定のラベルを満たさない実験対象のPodを設定できます。

YAMLファイルを使用して実験を作成する際、セレクタを設定する必要があります。例：

```yaml
spec:
  selector:
    expressionSelectors:
      - { key: tier, operator: In, values: [cache] }
      - { key: environment, operator: NotIn, values: [dev] }
```

### アノテーションセレクタ

- 実験対象のPodが持つ必要がある[アノテーション](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)を指定します。
- データ型: キーと値のペア。
- 複数のアノテーションが指定された場合、実験対象はこのセレクタで指定されたすべてのアノテーションを持っている必要があります。

YAMLファイルを使用して実験を作成する際、セレクタを設定する必要があります。例：

```yaml
spec:
  selector:
    annotationSelectors:
      'example-annotation': 'group-a'
```

### フィールドセレクタ

- 実験対象のPodの[フィールド](https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/)を指定します。
- データ型: キーと値のペア。
- 複数のフィールドが指定された場合、実験対象はこのセレクタで設定されたすべてのフィールドを持っている必要があります。

YAMLファイルを使用して実験を作成する際、セレクタを設定する必要があります。例：

```yaml
spec:
  selector:
    fieldSelectors:
      'metadata.name': 'my-pod'
```

### Podフェーズセレクタ

- 実験対象のPodのフェーズを指定します。
- データ型: 文字列配列型。
- サポートされているフェーズ: `Pending`、`Running`、`Succeeded`、`Failed`、`Unknown`。
- このオプションはデフォルトで空であり、対象Podのフェーズは制限されません。

YAMLファイルを使用して実験を作成する際、セレクタを設定する必要があります。例：

```yaml
spec:
  selector:
    podPhaseSelectors:
      - 'Running'
```

### ノードセレクター

- 実験対象のPodが属する[ノードラベル](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/)を指定します。
- データ型: キーと値のペア。
- 複数のノードラベルが指定されている場合、実験対象のPodが属するノードは、このセレクターで指定されたすべてのラベルを持っている必要があります。

YAMLファイルを使用して実験を作成する場合、セレクターを設定する必要があります。例:

```yaml
spec:
  selector:
    nodeSelectors:
      'node-label': 'label-one'
```

### ノードリストセレクター

- 実験対象のPodが属するノードを指定します。
- データ型: 文字列配列型。
- 対象のPodは、設定されたノードリスト内の1つのノードにのみ属することができます。

YAMLファイルを使用して実験を作成する場合、セレクターを設定する必要があります。例:

```yaml
spec:
  selector:
    nodes:
      - node1
      - node2
```

### Podリストセレクター

- 実験対象の`Pods`の名前空間とリストを指定します。
- データ型: キーと値のペア。キーは対象の`Pod`の名前空間で、値は対象の`Pod`リストです。
- このセレクターを指定した場合、Chaos Meshは他の設定されたセレクターを**無視**します。

YAMLファイルを使用して実験を作成する場合、セレクターを設定する必要があります。例:

```yaml
spec:
  selector:
    pods:
      tidb-cluster: # namespace of the target pods
        - basic-tidb-0
        - basic-pd-0
        - basic-tikv-0
        - basic-tikv-1
```

### 物理マシンリストセレクター

- 実験対象の`PhysicalMachines`の名前空間とリストを指定します。
- データ型: キーと値のペア。キーは対象の`PhysicalMachine`の名前空間で、値は対象の`PhysicalMachine`リストです。
- このセレクターを指定した場合、Chaos Meshは他の設定されたセレクターを**無視**します。

:::note

`PhysicalMachine`は物理マシンを表すCRD（CustomResourcesDefinition）です。`PhysicalMachine`を作成するために、Chaos Meshは[Chaosctl](chaosctl-tool.md#generate-tls-certificates-for-chaosd)を使用します。

:::

YAMLファイルを使用して実験を作成する場合、セレクターを設定する必要があります。例:

```yaml
spec:
  selector:
    physicalMachines:
      default: # namespace of the target PhysicalMachines
        - physical-machine-a
        - physical-machine-b
```

## Chaos Dashboardで実験範囲を定義

Chaos Dashboardを使用してChaos実験を作成する場合、実験情報を入力する際に実験範囲を設定できます。

現在、Chaos Dashboardで利用可能なセレクターは以下の通りです。実験範囲のフィルタリング要件に応じて、1つ以上のセレクターを指定できます:

- 名前空間セレクター
- ラベルセレクター
- アノテーションセレクター
- フェーズセレクター

セレクターを設定する際、ダッシュボードで実験対象の実際の範囲をリアルタイムで確認でき、セレクターによってフィルタリングされた対象Podの範囲を直接変更することもできます。

![ダッシュボードセレクター](img/dashboard_selectors_en.png)

## 互換性マトリックス

| Type                           | Support Kubernetes | Support physical machine |
| :----------------------------- | :----------------- | :----------------------- |
| Namespace Selectors            | Yes                | Yes                      |
| Label Selectors                | Yes                | Yes                      |
| Expression Selectors           | Yes                | Yes                      |
| Annotation Selectors           | Yes                | Yes                      |
| Field Selectors                | Yes                | Yes                      |
| PodPhase Selectors             | Yes                | No                       |
| Node Selectors                 | Yes                | No                       |
| Node List Selectors            | Yes                | No                       |
| Pod List Selectors             | Yes                | No                       |
| PhysicalMachine List Selectors | No                 | Yes                      |