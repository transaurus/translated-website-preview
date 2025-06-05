---
title: Serial and Parallel Experiments
---

Chaos Mesh Workflowでは、実験のスケジューリングに2つの方法を提供しています: シリアル（直列）とパラレル（並列）です。必要に応じて複数の実験を設定し、スケジュールすることができます。

- 複数のカオス実験を順番にスケジュールしたい場合は、シリアルノードを使用します。
- 複数のカオス実験を同時に実行したい場合は、パラレルノードを使用します。

Chaos Meshはシリアルノードとパラレルノードを設計する際に[コンポジットパターン](https://en.wikipedia.org/wiki/Composite_pattern)を使用しています。これにより、異なるタイプの複数のノードを含むことができ、特定のモードでコンポジットノードを実行できます。これはまた、シリアルノードとパラレルノードをネストして複雑なスケジューリングを実現できることも意味します。

## シリアル実験

Workflowで`templates`を作成する際、`templateType: Serial`を使用してシリアルノードを宣言します。

シリアルノードで必要なもう1つのフィールドは`children`です。その型は`[]string`で、値は他の`template`の名前です。例:

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: Workflow
metadata:
  name: try-workflow-serial
spec:
  entry: serial-of-3-node
  templates:
    - name: serial-of-3-node
      templateType: Serial
      deadline: 240s
      children:
        - workflow-stress-chaos
        - suspending
        - workflow-network-chaos
    - name: suspending
      templateType: Suspend
      deadline: 10s
    - name: workflow-network-chaos
      templateType: NetworkChaos
      deadline: 20s
      networkChaos:
        direction: to
        action: delay
        mode: all
        selector:
          labelSelectors:
            'app': 'hello-kubernetes'
        delay:
          latency: '90ms'
          correlation: '25'
          jitter: '90ms'
    - name: workflow-stress-chaos
      templateType: StressChaos
      deadline: 20s
      stressChaos:
        mode: one
        selector:
          labelSelectors:
            'app': 'hello-kubernetes'
        stressors:
          cpu:
            workers: 1
            load: 20
            options: ['--cpu 1', '--timeout 600']
```

上記のコマンドは`serial-of-3-node`という名前のシリアルノードを宣言しています。これは、Chaos Meshが`workflow-stress-chaos`、`suspending`、`workflow-network-chaos`を順番に実行することを意味します。すべてのタスクが完了すると、シリアルノードは完了としてマークされます。

Chaos Meshがシリアルノードを実行する際、`children`で宣言されたタスクは順番に実行され、同時に実行されるタスクは1つだけであることが保証されます。

シリアルノードの`deadline`フィールドはオプションで、シリアルプロセス全体の最大実行時間を制限します。この時間が経過すると、サブノードは停止され、まだ実行されていないノードは実行されません。すべてのサブノードが`deadline`時間前に作業を完了した場合、シリアルノードはすぐに完了としてマークされ、`deadline`は影響を受けません。

## パラレル実験

Workflowで`templates`を作成する際、`templateType: Parallel`を使用してパラレルノードを宣言します。

パラレルノードで必要なもう1つのフィールドは`children`です。その型は`[]string`で、値は他の`template`の名前です。例:

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: Workflow
metadata:
  name: try-workflow-parallel
spec:
  entry: parallel-of-2-chaos
  templates:
    - name: parallel-of-2-chaos
      templateType: Parallel
      deadline: 240s
      children:
        - workflow-stress-chaos
        - workflow-network-chaos
    - name: workflow-network-chaos
      templateType: NetworkChaos
      deadline: 20s
      networkChaos:
        direction: to
        action: delay
        mode: all
        selector:
          labelSelectors:
            'app': 'hello-kubernetes'
        delay:
          latency: '90ms'
          correlation: '25'
          jitter: '90ms'
    - name: workflow-stress-chaos
      templateType: StressChaos
      deadline: 20s
      stressChaos:
        mode: one
        selector:
          labelSelectors:
            'app': 'hello-kubernetes'
        stressors:
          cpu:
            workers: 1
            load: 20
            options: ['--cpu 1', '--timeout 600']
```

上記のコマンドは`parallel-of-2-chaos`という名前のパラレルノードを宣言しています。これは、Chaos Meshが`workflow-stress-chaos`と`workflow-network-chaos`を同時に実行することを意味します。すべてのタスクが完了すると、パラレルノードは完了としてマークされます。

Chaos Meshがパラレルノードを実行する際、`children`で宣言されたすべてのタスクは同時に実行されます。

シリアルノードと同様に、パラレルノードでもオプションの`deadline`フィールドを使用でき、パラレルプロセス全体の最大実行時間を制限します。この時間に達すると、サブノードは停止されます。すべてのサブノードが`deadline`時間前に作業を完了した場合、パラレルノードはすぐに完了としてマークされ、`deadline`は影響を受けません。

## Chaos Dashboardを使用してシリアルまたはパラレルノードを含むワークフローを作成

### シリアルノードの作成

Chaos Dashboardは`entry`という名前の事前定義されたシリアルノードを作成します。そのため、Chaos Dashboardを使用してシリアルノードを含むワークフローを作成する場合、ワークフローはデフォルトで`entry`の下に作成されます。

![Create Serial Node On Dashboard](./img/create-serial-node-on-dashboard.png)

### パラレルノードの作成

`Parallel`というパラレルノードを作成し、`Parallel`の下にサブノードを作成できます。

![Create Parallel Node on Dashboard](./img/create-parallel-node-on-dashboard.png)

### シリアルノードとパラレルノードのネスト

シリアルノードとパラレルノードを一緒にネストすることで、より複雑なプロセスを作成できます。

![Nested Serial And Parallel Node](./img/nested-serial-and-parallel.png)