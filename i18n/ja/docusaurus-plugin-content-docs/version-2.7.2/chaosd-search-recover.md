---
title: Search and Recover Experiments of Chaosd
summary: Describes how to search and recover the experiments of Chaosd, and provide related examples.
---

Chaosdを使用して条件に基づいて実験を検索し、指定したUIDに対応する実験を回復することができます。このドキュメントでは、Chaosdの実験を検索および回復する方法と関連する例を説明します。

## Chaosdの実験を検索する

このセクションでは、コマンドラインモードとサービスモードを使用してChaosdの実験を検索する方法を紹介します。

### コマンドラインモードで実験を検索する

以下のコマンドを実行することで、`search`コマンドがサポートする設定を確認できます：

```bash
$ chaosd search --help
Search chaos attack, you can search attacks through the uid or the state of the attack

Usage:
  chaosd search UID [flags]

Flags:
  -A, --all             list all chaos attacks
      --asc             order by CreateTime, default value is false that means order by CreateTime desc
  -h, --help            help for search
  -k, --kind string     attack kind, supported value: network, process, stress, disk, host, jvm
  -l, --limit uint32    limit the count of attacks
  -o, --offset uint32   starting to search attacks from offset
  -s, --status string   attack status, supported value: created, success, error, destroyed, revoked

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### 設定の説明

| Configuration item | Abbreviation | Description | Type |
| :-- | :-- | :-- | :-- |
| `all` | A | Lists all experiments | bool |
| `asc` | None | Sorts the experiments in ascending order of the creation time. The default value is `false`. | bool |
| `kind` | k | Lists experiments of the specified kind | string. The supported kinds are as follows: `network`, `process`, `stress`, `disk`, `host`, `jvm` |
| `limit` | l | The number of listed experiments | int |
| `offset` | o | Searches from the specified offset | int |
| `status` | s | Lists experiments with the specified status | string. The supported types are as follows: `created`, `success`, `error`, `destroyed`, `revoked` |

#### 例

```bash
./chaosd search --kind network --status destroyed --limit 1
```

このコマンドを実行することで、`network`タイプでステータスが`destroyed`（実験が回復済みであることを示す）の実験を検索できます。

コマンド実行後、結果には1行のデータのみが出力されます：

```bash
                  UID                     KIND     ACTION    STATUS            CREATE TIME                                                                                                                  CONFIGURATION
--------------------------------------- --------- -------- ----------- --------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  1f6c1253-522a-43d9-83f8-42607102b3b9   network   delay    destroyed   2021-11-02T15:14:07+08:00   {"schedule":"","duration":"","action":"delay","kind":"network","uid":"1f6c1253-522a-43d9-83f8-42607102b3b9","latency":"2s","jitter":"0ms","correlation":"0","device":"eth0","ip-address":"220.181.38.251","ip-protocol":"all"}
```

### サービスモードで実験を検索する

現在、サービスモードではすべての実験を検索することのみがサポートされています。Chaosdサービスの`/api/experiments/`パスにアクセスすることでデータを取得できます。

#### 例

```bash
curl -X GET 127.0.0.1:31767/api/experiments/
```

結果は以下のようになります：

```bash
[{"id":1,"uid":"ddc5ca81-b677-4595-b691-0ce57bedb156","created_at":"2021-10-18T16:01:18.563542491+08:00","updated_at":"2021-10-18T16:07:27.87111393+08:00","status":"success","kind":"stress","action":"mem","recover_command":"{\"schedule\":\"\",\"duration\":\"\",\"action\":\"mem\",\"kind\":\"stress\",\"uid\":\"ddc5ca81-b677-4595-b691-0ce57bedb156\",\"Load\":0,\"Workers\":0,\"Size\":\"100MB\",\"Options\":null,\"StressngPid\":0}","launch_mode":"svr"}]
```

## Chaosdの実験を回復する

実験を作成した後、実験によって引き起こされた影響を取り消したい場合は、実験の回復機能を使用できます。

### コマンドラインモードで実験を回復する

Chaosdの`recover`コマンドを使用して実験を回復できます。

以下の例は、コマンドラインモードを使用して実験を回復する方法を示しています。

1. Chaosdを使用してCPU負荷実験を作成します：

   ```bash
   chaosd attack stress cpu --workers 2 --load 10
   ```

   結果は以下のようになります：

   ```bash
   [2021/05/12 03:38:33.698 +00:00] [INFO] [stress.go:66] ["stressors normalize"] [arguments=" --cpu 2 --cpu-load 10"]
   [2021/05/12 03:38:33.702 +00:00] [INFO] [stress.go:82] ["Start stress-ng process successfully"] [command="/usr/bin/stress-ng --cpu 2 --cpu-load 10"] [Pid=27483]
   Attack stress cpu successfully, uid: 4f33b2d4-aee6-43ca-9c43-0f12867e5c9c
   ```

   後で使用するために実験のUIDを保存してください。

2. CPU負荷シナリオをシミュレートする必要がなくなったら、`recover`コマンドを使用してUIDに対応する実験を回復します：

   ```bash
   chaosd recover 4f33b2d4-aee6-43ca-9c43-0f12867e5c9c
   ```

### サービスモードで実験を回復する

Chaosdサービスの`/api/attack/{uid}`パスに`DELETE` HTTPリクエストを送信することで実験を回復できます。

以下の例は、サービスモードを使用して実験を回復する方法を示しています。

1. Chaosdサービスに`POST` HTTPリクエストを送信し、CPU負荷実験を作成します:

   ```bash
   curl -X POST 172.16.112.130:31767/api/attack/stress -H "Content-Type:application/json" -d '{"load":10, "action":"cpu","workers":1}'
   ```

   結果は以下のようになります:

   ```bash
   {"status":200,"message":"attack successfully","uid":"c3c519bf-819a-4a7b-97fb-e3d0814481fa"}
   ```

   後で使用するために実験のUIDを保存してください。

2. CPU負荷シナリオのシミュレーションが不要になった場合、以下のコマンドを実行して該当するUIDの実験を回復します:

   ```bash
   curl -X DELETE 172.16.112.130:31767/api/attack/c3c519bf-819a-4a7b-97fb-e3d0814481fa
   ```