---
title: Simulate Network Faults
---

このドキュメントでは、Chaos MeshのNetworkChaosを使用してネットワーク障害をシミュレートする方法について説明します。

## NetworkChaosの概要

NetworkChaosはChaos Meshの障害タイプの一つです。NetworkChaos実験を作成することで、クラスタに対してネットワーク障害シナリオをシミュレートできます。現在、NetworkChaosは以下の障害タイプをサポートしています：

- **Partition**: ネットワークの切断および分断
- **Net Emulation**: 高遅延、高パケット損失率、パケット順序変更などの悪いネットワーク状態
- **Bandwidth**: ノード間の通信帯域幅の制限

## 注意事項

NetworkChaos実験を作成する前に、以下の点を確認してください：

1. ネットワークインジェクション処理中、Controller ManagerとChaos Daemon間の接続が機能していることを確認してください。そうでない場合、NetworkChaosを回復できなくなる可能性があります。
2. Net Emulation障害をシミュレートする場合、LinuxカーネルにNET_SCH_NETEMモジュールがインストールされていることを確認してください。CentOSを使用している場合、kernel-modules-extraパッケージを通じてこのモジュールをインストールできます。他のほとんどのLinuxディストリビューションでは、このモジュールはデフォルトでインストールされています。

## Chaos Dashboardを使用した実験の作成

1. Chaos Dashboardを開き、ページ上の**NEW EXPERIMENT**をクリックして新しい実験を作成します：

   ![実験の作成](./img/create-new-exp.png)

2. **Choose a Target**エリアで、**NETWORK ATTACK**を選択し、**LOSS**などの特定の動作を選択します。その後、具体的な設定を入力します。

   ![NetworkChaos実験](./img/networkchaos-exp.png)

   具体的な設定フィールドの詳細については、[フィールド説明](#field-description)を参照してください。

3. 実験情報を入力し、実験範囲と予定された実験期間を指定します。

   ![実験情報](./img/exp-info.png)

4. 実験情報を送信します。

## YAMLファイルを使用した実験の作成

### 遅延の例

1. 以下のように実験設定を`network-delay.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: NetworkChaos
   metadata:
     name: delay
   spec:
     action: delay
     mode: one
     selector:
       namespaces:
         - default
       labelSelectors:
         'app': 'web-show'
     delay:
       latency: '10ms'
       correlation: '100'
       jitter: '0ms'
   ```

   この設定は、対象Podのネットワーク接続に10ミリ秒の遅延を発生させます。遅延注入に加えて、Chaos Meshはパケット損失やパケット順序変更の注入もサポートしています。詳細は[フィールド説明](#field-description)を参照してください。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f ./network-delay.yaml
   ```

### 分断の例

1. 以下のように実験設定を`network-partition.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: NetworkChaos
   metadata:
     name: partition
   spec:
     action: partition
     mode: all
     selector:
       namespaces:
         - default
       labelSelectors:
         'app': 'app1'
     direction: to
     target:
       mode: all
       selector:
         namespaces:
           - default
         labelSelectors:
           'app': 'app2'
   ```

   この設定は、`app1`から`app2`への接続をブロックします。`direction`フィールドの値は`to`、`from`または`both`を指定できます。詳細は[フィールド説明](#field-description)を参照してください。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f ./network-partition.yaml
   ```

### 帯域幅の例

1. 以下のように、実験設定を `network-bandwidth.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: NetworkChaos
   metadata:
     name: bandwidth
   spec:
     action: bandwidth
     mode: all
     selector:
       namespaces:
         - default
       labelSelectors:
         'app': 'app1'
     bandwidth:
       rate: '1mbps'
       limit: 20971520
       buffer: 10000
   ```

   この設定は、`app1` の帯域幅を 1 mbps に制限します。

2. 設定ファイルの準備ができたら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f ./network-bandwidth.yaml
   ```

### ネットワークエミュレーションの例

1. 以下のように、実験設定を `netem.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: NetworkChaos
   metadata:
     name: network-emulation
   spec:
     action: netem
     mode: all
     selector:
       namespaces:
         - default
       labelSelectors:
         'app': 'web-show'
     delay:
       latency: '10ms'
       correlation: '100'
       jitter: '0ms'
     rate:
       rate: '10mbps'
   ```

   この設定は、対象の Pod のネットワーク接続に 10 ミリ秒の遅延と 10mbps の帯域幅制限を発生させます。遅延と帯域幅制限に加えて、`netem` アクションはパケット損失、順序変更、破損もサポートしています。

2. 設定ファイルの準備ができたら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f ./netem.yaml
   ```

## フィールド説明

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| action | string | Indicates the specific fault type. Available types include: `netem`, `delay` (network delay), `loss` (packet loss), `duplicate` (packet duplicating), `corrupt` (packet corrupt), `partition` (network partition), and `bandwidth` (network bandwidth limit). After you specify `action` field, refer to [Description for `action`-related fields](#description-for-action-related-fields) for other necessary field configuration. | None | Yes | Partition |
| target | Selector | Used in combination with direction, making Chaos only effective for some packets. | None | No |  |
| direction | enum | Indicates the direction of `target` packets. Available values include `from` (the packets from `target`), `to` (the packets to `target`), and `both` ( the packets from or to `target`). This parameter makes Chaos only take effect for a specific direction of packets. | to | No | both |
| mode | string | Specifies the mode of the experiment. The mode options include `one` (selecting a random Pod), `all` (selecting all eligible Pods), `fixed` (selecting a specified number of eligible Pods), `fixed-percent` (selecting a specified percentage of Pods from the eligible Pods), and `random-max-percent` (selecting the maximum percentage of Pods from the eligible Pods). | None | Yes | `one` |
| value | string | Provides a parameter for the `mode` configuration, depending on `mode`. For example, when `mode` is set to `fixed-percent`, `value` specifies the percentage of Pods. | None | No | 1 |
| selector | struct | Specifies the target Pod. For details, refer to [Define the experiment scope](./define-chaos-experiment-scope.md). | None | Yes |  |
| externalTargets | []string | Indicates the network targets except for Kubernetes, which can be IPv4 addresses or domains. This parameter only works with `direction: to`. | None | No | 1.1.1.1, google.com |
| device | string | Specifies the affected network interface | None | No | "eth0" |

## `action` 関連フィールドの説明

ネットワークエミュレーションと帯域幅の障害タイプについては、以下の説明に従って `action` 関連のパラメータをさらに設定できます。

- ネットワークエミュレーションタイプ: `delay`, `loss`, `duplicated`, `corrupt`, `rate`
- 帯域幅タイプ: `bandwidth`

### delay

`action` を `delay` に設定すると、ネットワーク遅延障害をシミュレートします。以下のパラメータも設定できます。

| Parameter | Type | Description | Required | Required | Example |
| --- | --- | --- | --- | --- | --- |
| latency | string | Indicates the network latency | No | No | 2ms |
| correlation | string | Indicates the correlation between the current latency and the previous one. Range of value: [0, 100] | No | No | 50 |
| jitter | string | Indicates the range of the network latency | No | No | 1ms |
| reorder | Reorder(#Reorder) | Indicates the status of network packet reordering |  | No |  |

`correlation` の計算モデルは以下の通りです:

1. 前の値に関連した分布を持つ乱数を生成します:

   ```c
   rnd = value * (1-corr) + last_rnd * corr
   ```

   `rnd` は乱数です。`corr` は前に記入した `correlation` です。

2. この乱数を使用して、現在のパケットの遅延を決定します:

   ```c
   ((rnd % (2 * sigma)) + mu) - sigma
   ```

   上記のコマンドで、`sigma` は `jitter`、`mu` は `latency` です。

### reorder

`action` を `reorder` に設定すると、ネットワークパケットの順序変更障害をシミュレートします。以下のパラメータも設定できます。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| reorder | string | Indicates the probability to reorder | 0 | No | 0.5 |
| correlation | string | Indicates the correlation between this time's length of delay time and the previous time's length of delay time. Range of value: [0, 100] | 0 | No | 50 |
| gap | int | Indicates the gap before and after packet reordering | 0 | No | 5 |

### loss

`action` を `loss` に設定すると、パケット損失障害をシミュレートします。以下のパラメータも設定できます。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| loss | string | Indicates the probability of packet loss. Range of value: [0, 100] | 0 | No | 50 |
| correlation | string | Indicates the correlation between the probability of current packet loss and the previous time's packet loss. Range of value: [0, 100] | 0 | No | 50 |

### duplicate

`action` を `duplicate` に設定すると、パケットの重複をシミュレートします。この時、以下のパラメータも設定できます。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| duplicate | string | Indicates the probability of packet duplicating. Range of value: [0, 100] | 0 | No | 50 |
| correlation | string | Indicates the correlation between the probability of current packet duplicating and the previous time's packet duplicating. Range of value: [0, 100] | 0 | No | 50 |

### corrupt

`action` を `corrupt` に設定すると、パケット破損障害をシミュレートします。以下のパラメータも設定できます。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| corrupt | string | Indicates the probability of packet corruption. Range of value: [0, 100] | 0 | No | 50 |
| correlation | string | Indicates the correlation between the probability of current packet corruption and the previous time's packet corruption. Range of value: [0, 100] | 0 | No | 50 |

`reorder`、`loss`、`duplicate`、`corrupt`などの偶発的なイベントの場合、`correlation`の計算はより複雑です。具体的なモデルの説明については、[NetemCLG](http://web.archive.org/web/20200120162102/http://netgroup.uniroma2.it/twiki/bin/view.cgi/Main/NetemCLG)を参照してください。

### rate

`action`を`rate`に設定すると、帯域幅レートの障害をシミュレートします。このアクションは以下の[bandwidth/rate](#bandwidth)と似ていますが、**重要な違いは、このアクションが上記の他の`netem`アクションと組み合わせることができる点です**。ただし、バッファサイズの制限など、帯域幅シミュレーションをより詳細に制御する必要がある場合は、[bandwidth](#bandwidth)アクションを確認してください。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| rate | string | Indicates the rate of bandwidth limit. Allows bit, kbit, mbit, gbit, tbit, bps, kbps, mbps, gbps, tbps unit. bps means bytes per second |  | Yes | 1mbps |

### bandwidth

`action`を`bandwidth`に設定すると、帯域幅制限の障害をシミュレートします。また、以下のパラメータを設定する必要があります。

:::info

このアクションは、上記で定義された`netem`アクションとは排他的です。帯域幅レートと破損などの他のネットワーク障害を同時に注入する必要がある場合は、代わりに[rate](#rate)アクションを使用してください。

:::

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| rate | string | Indicates the rate of bandwidth limit. Allows bit, kbit, mbit, gbit, tbit, bps, kbps, mbps, gbps, tbps unit. bps means bytes per second |  | Yes | 1mbps |
| limit | uint32 | Indicates the number of bytes waiting in queue |  | Yes | 1 |
| buffer | uint32 | Indicates the maximum number of bytes that can be sent instantaneously |  | Yes | 1 |
| peakrate | uint64 | Indicates the maximum consumption of `bucket` (usually not set) |  | No | 1 |
| minburst | uint32 | Indicates the size of `peakrate bucket` (usually not set) |  | No | 1 |

これらのフィールドの詳細については、[tc-tbfドキュメント](https://man7.org/linux/man-pages/man8/tc-tbf.8.html)を参照してください。`limit`は少なくとも`2 * rate * latency`に設定することを推奨します。ここで、`latency`は送信元と送信先間の推定遅延であり、`ping`コマンドで推定できます。`limit`が小さすぎると、高い損失率が発生し、TCP接続のスループットに影響を与える可能性があります。