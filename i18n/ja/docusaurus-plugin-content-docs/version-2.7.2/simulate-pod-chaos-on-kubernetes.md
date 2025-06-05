---
title: Simulate Pod Faults
---

このドキュメントでは、Chaos Meshを使用してKubernetes Podに障害を注入し、Podやコンテナの障害をシミュレートする方法について説明します。Chaos DashboardとYAMLファイルを使用してPodChaos実験を作成する手順を提供します。

## PodChaosの概要

PodChaosはChaos Meshの障害タイプの一つです。PodChaos実験を作成することで、指定したPodやコンテナの障害シナリオをシミュレートできます。現在、PodChaosは以下の障害タイプをサポートしています：

- **Pod Failure**: 指定したPodに障害を注入し、一定期間Podを利用不可にします。
- **Pod Kill**: 指定したPodを強制終了します。Podが正常に再起動できるようにするためには、ReplicaSetまたは類似のメカニズムを設定する必要があります。
- **Container Kill**: 対象Pod内の指定したコンテナを強制終了します。

## 使用上の制限

Chaos Meshは、Deployment、StatefulSet、DaemonSetなどのコントローラーにバインドされているかどうかに関係なく、任意のPodにPodChaosを注入できます。ただし、独立したPodにPodChaosを注入する場合、異なる状況が発生する可能性があります。例えば、独立したPodに「pod-kill」障害を注入した場合、Chaos Meshはアプリケーションが障害から回復することを保証できません。

## 注意事項

PodChaos実験を作成する前に、以下の点を確認してください：

- 対象Pod上でChaos MeshのControl Managerが実行されていないこと。
- 障害タイプがPod Killの場合、Podが自動的に再起動できるようにreplicaSetまたは類似のメカニズムが設定されていること。

## Chaos Dashboardを使用した実験の作成

:::note

Chaos Dashboardを使用して実験を作成する前に、以下の点を確認してください：

- Chaos Dashboardがインストールされていること。
- Chaos Dashboardが既にインストールされている場合、`kubectl port-forward`を実行してDashboardにアクセスできます：`bash kubectl port-forward -n chaos-mesh svc/chaos-dashboard 2333:2333`。その後、[`http://localhost:2333`](http://localhost:2333)にアクセスしてChaos Dashboardを利用できます。

:::

1. Chaos Dashboardを開き、ページ上の**NEW EXPERIMENT**をクリックして新しい実験を作成します。

   ![新しい実験の作成](./img/create-new-exp.png)

2. **Choose a Target**エリアで、**POD FAULT**を選択し、**POD FAILURE**などの具体的な動作を選択します。

3. 実験情報を入力し、実験範囲と予定された実験期間を指定します。

4. 実験情報を送信します。

## YAML設定ファイルを使用した実験の作成

### pod-failureの例

1. 実験設定を`pod-failure.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: PodChaos
   metadata:
     name: pod-failure-example
     namespace: chaos-mesh
   spec:
     action: pod-failure
     mode: one
     duration: '30s'
     selector:
       labelSelectors:
         'app.kubernetes.io/component': 'tikv'
   ```

   この例に基づき、Chaos Meshは指定したPodに`pod-failure`を注入し、30秒間Podを利用不可にします。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f ./pod-failure.yaml
   ```

### pod-killの例

1. 実験設定を`pod-kill.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: PodChaos
   metadata:
     name: pod-kill-example
     namespace: chaos-mesh
   spec:
     action: pod-kill
     mode: one
     selector:
       namespaces:
         - tidb-cluster-demo
       labelSelectors:
         'app.kubernetes.io/component': 'tikv'
   ```

   この例に基づき、Chaos Meshは指定したPodに`pod-kill`を注入し、Podを1回強制終了します。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f ./pod-kill.yaml
   ```

### container-killの例

1. 実験設定を`container-kill.yaml`ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: PodChaos
   metadata:
     name: container-kill-example
     namespace: chaos-mesh
   spec:
     action: container-kill
     mode: one
     containerNames: ['prometheus']
     selector:
       labelSelectors:
         'app.kubernetes.io/component': 'monitor'
   ```

   この例に基づき、Chaos Meshは指定されたコンテナに`container-kill`を注入し、コンテナを1回強制終了します。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します:

   ```bash
   kubectl apply -f ./container-kill.yaml
   ```

### フィールド説明

以下の表はYAML設定ファイルのフィールドについて説明しています。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| action | string | Specifies the fault type to inject. The supported types include `pod-failure`, `pod-kill`, and `container-kill`. | None | Yes | `pod-kill` |
| mode | string | Specifies the mode of the experiment. The mode options include `one` (selecting a random Pod), `all` (selecting all eligible Pods), `fixed` (selecting a specified number of eligible Pods), `fixed-percent` (selecting a specified percentage of Pods from the eligible Pods), and `random-max-percent` (selecting the maximum percentage of Pods from the eligible Pods). | None | Yes | `one` |
| value | string | Provides parameters for the `mode` configuration, depending on `mode`.For example, when `mode` is set to `fixed-percent`, `value` specifies the percentage of Pods. | None | No | 1 |
| selector | struct | Specifies the target Pod. For details, refer to [Define the experiment scope](./define-chaos-experiment-scope.md). | None | Yes |  |
| containerNames | []string | When you configure `action` to `container-kill`, this configuration is mandatory to specify the target container name for injecting faults. | None | No | ['prometheus'] |
| gracePeriod | int64 | When you configure `action` to `pod-kill`, this configuration is mandatory to specify the duration before deleting Pod. | 0 | No | 0 |
| duration | string | Specifies the duration of the experiment. | None | Yes | 30s |

## 「Pod Failure」カオス実験に関する注意点

要約すると、「Pod Failure」カオス実験を使用する際の推奨事項は以下の通りです:

- エアギャップ環境のKubernetesクラスターを操作する場合は、利用可能な「pause image」に変更してください
- コンテナに`livenessProbe`と`readinessProbe`を設定してください

Pod Failureカオス実験は、ターゲットPod内の各コンテナの`image`を「pause image」（何も操作を行わない特殊なイメージ）に変更します。デフォルトでは「pause image」として`gcr.io/google-containers/pause:latest`を使用しており、helmのvalues設定`controllerManager.podChaos.podFailure.pauseImage`で任意のイメージに変更可能です。

「pause image」のダウンロードには時間がかかり、その時間は実験期間に含まれます。そのため、「実際の影響期間」が設定した期間より短くなる場合があります。これが利用可能な「pause image」を設定することを推奨するもう一つの理由です。

もう一つの注意点として、「pause image」はコンテナに`command`が設定されていない場合でも「一応正常に」動作します。そのため、コンテナに`command`、`livenessProbe`、`readinessProbe`が設定されていない場合、コンテナは`Running`かつ`Ready`と判定されますが、実際には「pause image」に変更されており、通常の機能を提供していない（または利用不可の）状態になります。このため、コンテナに`livenessProbe`と`readinessProbe`を設定することを強く推奨します。