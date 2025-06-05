---
title: Simulate Process Faults
---

このドキュメントでは、Chaosdを使用してプロセス障害をシミュレートする方法について説明します。プロセス障害は、`kill`コマンドのGolangインターフェースを使用して、プロセスが強制終了または停止されるシナリオをシミュレートします。コマンドラインモードまたはサービスモードのいずれかで実験を作成できます。

## コマンドラインモードを使用した実験の作成

実験を作成する前に、次のコマンドを実行して、Chaosdがサポートするプロセス障害のタイプを確認できます：

```bash
chaosd attack process -h
```

結果は次のとおりです：

```bash
Process attack related commands

Usage:
  chaosd attack process [command]

Available Commands:
  kill        kill process, default signal 9
  stop        stop process, this action will stop the process with SIGSTOP

Flags:
  -h, --help   help for process

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'

Use "chaosd attack process [command] --help" for more information about a command.
```

現在、Chaosdはプロセスの強制終了または停止をシミュレートすることをサポートしています。

### コマンドラインモードを使用したプロセスの強制終了

#### プロセスを強制終了するためのコマンド

```bash
chaosd attack process kill -h
```

結果は次のとおりです：

```bash
kill process, default signal 9

Usage:
  chaosd attack process kill [flags]

Flags:
  -h, --help                 help for kill
  -p, --process string       The process name or the process ID
  -r, --recover-cmd string   The command to be run when recovering experiment
  -s, --signal int           The signal number to send (default 9)

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### プロセス強制終了の設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `process` | p | The name or the identifier of the process to be injected faults | string; the default value is `""`. |
| `recover-cmd` | r | The command to be run when recovering experiment | string; the default value is `""`. |
| `signal` | s | The provided value of the process signal | int; the default value is `9`. Currently, only `SIGKILL`, `SIGTERM`, and `SIGSTOP` are supported. |

#### プロセス強制終了の例

```bash
chaosd attack process kill -p python
```

結果は次のとおりです：

```bash
Attack process python successfully, uid: 10e633ac-0a37-41ba-8b4a-cd5ab92099f9
```

:::note

`signal`が`SIGSTOP`である実験のみが復旧可能です。

:::

### コマンドラインモードを使用したプロセスの停止

#### プロセスを停止するためのコマンド

```bash
chaosd attack process stop -h
```

結果は次のとおりです：

```bash
stop process, this action will stop the process with SIGSTOP

Usage:
  chaosd attack process stop [flags]

Flags:
  -h, --help             help for stop
  -p, --process string   The process name or the process ID

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### プロセス停止の設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `process` | p | The name or the identifier of the process to be stopped | string; the default value is `""`. |

#### プロセス停止の例

```bash
chaosd attack process stop -p python
```

結果は次のとおりです：

```bash
Attack process python successfully, uid: 9cb6b3be-4f5b-4ecb-ae05-51050fcd0010
```

## サービスモードを使用した実験の作成

サービスモードを使用して実験を作成するには、以下の手順に従ってください：

1. Chaosdをサービスモードで実行します：

   ```bash
   chaosd server --port 31767
   ```

2. Chaosdサービスの`/api/attack/process`パスに`POST` HTTPリクエストを送信します。

   ```bash
   curl -X POST 172.16.112.130:31767/api/attack/process -H "Content-Type:application/json" -d '{fault-configuration}'
   ```

   上記のコマンドでは、障害タイプに応じて`fault-configuration`を設定する必要があります。対応するパラメータについては、以下のセクションの各障害タイプのパラメータと例を参照してください。

:::note

実験を実行する際は、実験のUIDを記録しておいてください。UIDに対応する実験を終了する場合は、Chaosdサービスの`/api/attack/{uid}`パスに`DELETE` HTTPリクエストを送信する必要があります。

:::

### サービスモードを使用したプロセス障害のシミュレーション

#### プロセス障害をシミュレートするためのパラメータ

| Parameter | Description                                                     | Value                              |
| :-------- | :-------------------------------------------------------------- | :--------------------------------- |
| `process` | The name or the identifier of the process to be injected faults | string; the default value is `""`. |
| `signal`  | The provided value of the process signal                        | int; the default value is `9`      |

#### サービスモードを使用したプロセス障害シミュレーションの例

##### プロセスの終了

```bash
curl -X POST 172.16.112.130:31767/api/attack/process -H "Content-Type:application/json" -d '{"process":"12345","signal":15}'
```

結果は次のとおりです：

```bash
{"status":200,"message":"attack successfully","uid":"c3c519bf-819a-4a7b-97fb-e3d0814481fa"}
```

##### プロセスの停止

```bash
curl -X POST 172.16.112.130:31767/api/attack/process -H "Content-Type:application/json" -d '{"process":"12345","signal":19}'
```

結果は次のとおりです：

```bash
{"status":200,"message":"attack successfully","uid":"a00cca2b-eba7-4716-86b3-3e66f94880f7"}
```

:::note

`signal`が`SIGSTOP`である実験のみが復旧可能です。

:::