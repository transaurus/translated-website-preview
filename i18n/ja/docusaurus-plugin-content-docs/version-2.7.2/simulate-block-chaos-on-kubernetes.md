---
title: Simulate Block Device Incidents
---

## BlockChaosの紹介

Chaos MeshはBlockChaos実験タイプを提供しています。この実験タイプを使用して、ブロックデバイスの遅延やフリーズのシナリオをシミュレートできます。このドキュメントでは、BlockChaos実験の依存関係のインストール方法とBlockChaosの作成方法について説明します。

:::note

BlockChaosは初期段階です。インストールと設定の体験は継続的に改善されます。問題を見つけた場合は、[chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)でイシューを開いて報告してください。

:::

:::note

BlockChaosの`freeze`アクションは、ブロックデバイスを使用するすべてのプロセスに影響を与えます。対象のコンテナだけでなく、すべてのプロセスが影響を受けます。

:::

## カーネルモジュールのインストール

BlockChaosの`delay`アクションは[chaos-driver](https://github.com/chaos-mesh/chaos-driver)カーネルモジュールに依存しています。このモジュールがインストールされているマシンでのみ注入できます。現在のところ、このモジュールを手動でコンパイルしてインストールする必要があります。

1. 以下のコマンドを使用してモジュールのソースコードをダウンロードします:

   ```bash
   curl -fsSL -o chaos-driver-v0.2.1.tar.gz https://github.com/chaos-mesh/chaos-driver/archive/refs/tags/v0.2.1.tar.gz
   ```

2. `chaos-driver-v0.2.1.tar.gz`ファイルを解凍します:

   ```bash
   tar xvf chaos-driver-v0.2.1.tar.gz
   ```

3. 現在のカーネルのヘッダーを準備します。CentOS/Fedoraを使用している場合は、`yum`でカーネルヘッダーをインストールできます:

   ```bash
   yum install kernel-devel-$(uname -r)
   ```

   Ubuntu/Debianを使用している場合は、`apt`でカーネルヘッダーをインストールできます:

   ```bash
   apt install linux-headers-$(uname -r)
   ```

4. モジュールをコンパイルします:

   ```bash
   cd chaos-driver-v0.2.1
   make driver/chaos_driver.ko
   ```

5. カーネルモジュールをインストールします:

   ```bash
   insmod ./driver/chaos_driver.ko
   ```

`chaos_driver`モジュールは、再起動するたびにインストールする必要があります。モジュールを自動的にロードするには、モジュールを`/lib/modules/$(uname -r)/kernel/drivers`のサブディレクトリにコピーし、`depmod -a`を実行してから、`/etc/modules`に`chaos_driver`を追加します。

カーネルをアップグレードした場合、モジュールを再コンパイルする必要があります。

:::note

DKMSまたはakmodを使用してカーネルモジュールの自動コンパイルやロードを行うことを推奨します。インストール体験を改善したい場合は、DKMSまたはakmodパッケージを作成してさまざまなディストリビューションのリポジトリに提出することが大歓迎です。

:::

## YAMLファイルを使用した実験の作成

1. 実験設定をYAML設定ファイルに記述します。以下は`block-latency.yaml`ファイルの例です。

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: BlockChaos
   metadata:
     name: hostpath-example-delay
   spec:
     selector:
       labelSelectors:
         app: hostpath-example
     mode: all
     volumeName: hostpath-example
     action: delay
     delay:
       latency: 1s
   ```

   :::note

   hostpathまたはlocalvolumeのみがサポートされています。

   :::

2. `kubectl`を使用して実験を作成します:

   ```bash
   kubectl apply -f block-latency.yaml
   ```

以下のような魔法が起こります:

1. ボリュームのエレベーターが`ioem`または`ioem-mq`に変更されます。`cat /sys/block/<device>/queue/scheduler`で確認できます。
2. `ioem`または`ioem-mq`スケジューラが遅延リクエストを受け取り、指定された時間だけリクエストを遅延させます。

YAML設定ファイルのフィールドは以下の表で説明されています:

| Parameter | Type | Note | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| `mode` | string | Specifies the mode of the experiment. The mode options include `one` (selecting a random Pod), `all` (selecting all eligible Pods), `fixed` (selecting a specified number of eligible Pods), `fixed-percent` (selecting a specified percentage of Pods from the eligible Pods), and `random-max-percent` (selecting the maximum percentage of Pods from the eligible Pods). | None | Yes | `one` |
| `value` | string | Provides parameters for the `mode` configuration, depending on `mode`. For example, when `mode` is set to `fixed-percent`, `value` specifies the percentage of Pods. | None | No | `1` |
| `selector` | struct | Specifies the target Pod. For details, refer to [Define the experiment scope](./define-chaos-experiment-scope.md). | None | Yes |  |
| `volumeName` | string | Specifies the volume to inject in the target pods. There should be a corresponding entry in the pods' `.spec.volumes`. | None | Yes | `hostpath-example` |
| `action` | string | Indicates the specific type of faults. The available fault types include `delay` and `freeze`. `delay` will simulate the latency of block devices, and `freeze` will simulate that the block device cannot handle any requests | None | Yes | `delay` |
| `delay.latency` | string | Specifies the latency of the block device. | None | Yes (if `action` is `delay`) | `500ms` |