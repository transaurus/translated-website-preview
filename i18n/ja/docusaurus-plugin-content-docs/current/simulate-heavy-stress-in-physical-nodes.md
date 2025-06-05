---
title: Simulate Stress Scenarios
---

このドキュメントでは、Chaosdを使用してストレスシナリオをシミュレートする方法について説明します。この機能は、[stress-ng](https://wiki.ubuntu.com/Kernel/Reference/stress-ng)を使用してホスト上でCPUまたはメモリのストレスを生成します。ストレス実験は、コマンドラインまたはサービスモードのいずれかで作成できます。

## コマンドラインモードを使用してストレス実験を作成する

このセクションでは、コマンドラインモードを使用してストレス実験を作成する方法について説明します。

ストレス実験を作成する前に、以下のコマンドを実行してChaosdがサポートするストレス実験のタイプを確認できます：

```bash
chaosd attack stress --help
```

結果は以下の通りです：

```bash
Stress attack related commands

Usage:
  chaosd attack stress [command]

Available Commands:
  cpu         continuously stress CPU out
  mem         continuously stress virtual memory out

Flags:
  -h, --help   help for stress

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'

Use "chaosd attack stress [command] --help" for more information about a command.
```

現在、ChaosdはCPUストレス実験とメモリストレス実験の作成をサポートしています。

### コマンドラインモードを使用してCPUストレスをシミュレートする

#### CPUストレスシミュレーションのコマンド

CPUストレスシミュレーションでサポートされている設定項目を確認するには、以下のコマンドを実行します：

```bash
chaosd attack stress cpu --help
```

結果は以下の通りです：

```bash
continuously stress CPU out

Usage:
  chaosd attack stress cpu [options] [flags]

Flags:
  -h, --help              help for cpu
  -l, --load int          Load specifies P percent loading per CPU worker. 0 is effectively a sleep (no load) and 100 is full loading. (default 10)
  -o, --options strings   extend stress-ng options.
  -w, --workers int       Workers specifies N workers to apply the stressor. (default 1)

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### CPUストレスシミュレーションの設定説明

| Configuration item | Abbreviation | Description | Type | Value |
| :-- | :-- | :-- | :-- | :-- |
| `load` | l | Specifies the percentage of CPU load per CPU worker. `0` means no CPU utilization, and `100` means full CPU utilization. | int | Range: `0` to `100`; Default value: `10`. |
| `workers` | w | Specifies the number of workers used to create CPU stress. | int | Default value: 1. |
| `options` | o | The extended parameter of stress-ng, usually not configured. | string | Default value: "". |

#### CPUストレスシミュレーションの例

```bash
chaosd attack stress cpu --workers 2 --load 10
```

結果は以下の通りです：

```bash
[2021/05/12 03:38:33.698 +00:00] [INFO] [stress.go:66] ["stressors normalize"] [arguments=" --cpu 2 --cpu-load 10"]
[2021/05/12 03:38:33.702 +00:00] [INFO] [stress.go:82] ["Start stress-ng process successfully"] [command="/usr/bin/stress-ng --cpu 2 --cpu-load 10"] [Pid=27483]
Attack stress cpu successfully, uid: 4f33b2d4-aee6-43ca-9c43-0f12867e5c9c
```

### コマンドラインモードを使用してメモリストレスをシミュレートする

#### メモリストレスシミュレーションのコマンド

メモリストレスシミュレーションでサポートされている設定項目を確認するには、以下のコマンドを実行します：

```bash
chaosd attack stress mem --help
```

結果は以下の通りです：

```bash
continuously stress virtual memory out

Usage:
  chaosd attack stress mem [options] [flags]

Flags:
  -h, --help              help for mem
  -o, --options strings   extend stress-ng options.
  -s, --size string       Size specifies N bytes consumed per vm worker, default is the total available memory. One can specify the size as % of total available memory or in units of B, KB/KiB, MB/MiB, GB/GiB, TB/TiB..

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### メモリストレスシミュレーションの設定説明

| Configuration item | Abbreviation | Description | Type | Value |
| :-- | :-- | :-- | :-- | :-- |
| `size` | s | Specifies the size of memory per VM worker. | string | The memory size in B, KB/KiB, MB/MiB, GB/GiB, TB/TiB. If the size is not set, all available memory is used by default. |
| `options` | o | The extended parameter of stress-ng, usually not configured. | string | Default value: "". |

#### メモリストレスシミュレーションの例

```bash
chaosd attack stress mem --workers 2 --size 100M
```

結果は以下の通りです：

```bash
[2021/05/12 03:37:19.643 +00:00] [INFO] [stress.go:66] ["stressors normalize"] [arguments=" --vm 2 --vm-keep --vm-bytes 100000000"]
[2021/05/12 03:37:19.654 +00:00] [INFO] [stress.go:82] ["Start stress-ng process successfully"] [command="/usr/bin/stress-ng --vm 2 --vm-keep --vm-bytes 100000000"] [Pid=26799]
Attack stress mem successfully, uid: c2bff2f5-3aac-4ace-b7a6-322946ae6f13
```

実験を実行する際には、実験のuid情報を保存する必要があります。ストレスシミュレーションが不要になった場合、`recover`を使用してuidに関連する実験を終了できます：

```bash
chaosd recover c2bff2f5-3aac-4ace-b7a6-322946ae6f13
```

結果は以下の通りです：

```bash
Recover c2bff2f5-3aac-4ace-b7a6-322946ae6f13 successfully
```

## サービスモードを使用してストレス実験を作成する

サービスモードを使用して実験を作成するには、以下の手順に従ってください：

1. Chaosdをサービスモードで実行します：

   ```bash
   chaosd server --port 31767
   ```

2. Chaosdサービスの`/api/attack/{uid}`パスに`POST` HTTPリクエストを送信します。

   ```bash
   curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{fault-configuration}'
   ```

   上記のコマンドの`fault-configuration`部分は、障害タイプに応じて設定する必要があります。対応するパラメータについては、以下のセクションの各障害タイプのパラメータと例を参照してください。

:::note

実験を実行する際には、実験のUID情報を保存することを忘れないでください。UIDに対応する実験を終了したい場合、Chaosdサービスの`/api/attack/{uid}`パスに`DELETE` HTTPリクエストを送信する必要があります。

:::

### サービスモードを使用してCPUストレスをシミュレートする

#### CPUストレスシミュレーションのパラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Actions of the experiment |  | Set to "cpu" |
| `load` | Specifies the percentage of CPU load per CPU worker. `0` means no CPU utilization, and `100` means full CPU utilization. | int | Range: `0` to `100`; Default value: `10` |
| `workers` | Specifies the number of workers used to create CPU stress | int | Default value: `1` |
| `options` | The extended parameter of stress-ng, usually not configured. | string | Default value: "" |

#### サービスモードを使用したCPUストレスシミュレーションの例

```bash
curl -X POST 172.16.112.130:31767/api/attack/stress -H "Content-Type:application/json" -d '{"load":10, "action":"cpu","workers":1}'
```

結果は以下の通りです：

```bash
{"status":200,"message":"attack successfully","uid":"c3c519bf-819a-4a7b-97fb-e3d0814481fa"}
```

### サービスモードを使用してメモリストレスをシミュレートする

#### メモリストレスのシミュレーション用パラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Actions of the experiment |  | Set to "mem" |
| `size` | Specifies the size of memory per VM worker | string | the memory size in B, KB/KiB, MB/MiB, GB/GiB, TB/TiB. If the size is not set, all available memory is used by default. |
| `options` | The extended parameter of stress-ng, usually not configured. | string | Default value: "" |

#### サービスモードを使用したメモリストレスのシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/stress -H "Content-Type:application/json" -d '{"size":"100M", "action":"mem"}'
```

結果は以下の通りです：

```bash
{"status":200,"message":"attack successfully","uid":"a551206c-960d-4ac5-9056-518e512d4d0d"}
```