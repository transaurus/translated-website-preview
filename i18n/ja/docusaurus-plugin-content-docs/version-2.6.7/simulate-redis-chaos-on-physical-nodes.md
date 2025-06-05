---
title: Simulate Redis Faults
---

このドキュメントでは、Chaosdを使用してRedisの障害をシミュレートする方法を紹介します。この機能は`go-redis`パッケージのGolangインターフェースと`redis-server`コマンドラインツールを使用しています。コマンドラインモードまたはサービスモードのいずれかで実験を作成できます。

## コマンドラインモードを使用した実験の作成

実験を作成する前に、以下のコマンドを実行してChaosdがサポートするRedis障害タイプを確認できます：

```bash
chaosd attack redis -h
```

結果は以下の通りです：

```bash
Redis attack related commands

Usage:
  chaosd attack redis [command]

Available Commands:
  cache-expiration  expire keys in Redis
  cache-limit       set maxmemory of Redis
  cache-penetration penetrate cache
  sentinel-restart  restart sentinel
  sentinel-stop     stop sentinel

Flags:
  -h, --help   help for redis

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID

Use "chaosd attack redis [command] --help" for more information about a command.
```

現在、Chaosdはキャッシュの有効期限切れ、キャッシュペネトレーション、キャッシュ制限、センチネルの再起動、センチネルの停止をシミュレートできます。

### コマンドラインモードを使用したキャッシュ有効期限切れのシミュレート

このコマンドの意味はRedisのEXPIREと同じです。詳細については、[Redis公式ドキュメント](https://redis.io/commands/expire/)を参照してください。

:::note

現在、Chaosdは`cache-expiration`を実行したキーの復旧をサポートしていないため、復旧が必要な場合は事前にバックアップを取ってください。

:::

#### キャッシュ有効期限切れのコマンド

```bash
chaosd attack redis cache-expiration -h
```

結果は以下の通りです：

```bash
expire keys in Redis

Usage:
  chaosd attack redis cache-expiration [flags]

Flags:
  -a, --addr string         The address of redis server
      --expiration string   The expiration of the key. A expiration string should be able to be converted to a time duration, such as "5s" or "30m" (default "0")
  -h, --help                help for cache-expiration
  -k, --key string          The key to be set a expiration, default expire all keys
      --option string       The additional options of expiration, only NX, XX, GT, LT supported
  -p, --password string     The password of server

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### キャッシュ有効期限切れの設定説明

| Configuration item | Abbreviation | Type | Description | Value |
| :-- | :-- | :-- | :-- | :-- |
| `addr` | a | string | The address and port of Redis server to be injected into the fault, for example `127.0.0.1:6379` | Default value: `""` |
| `expiration` | None | string | The specified key will be expired after `expiration` arrives | Default value: `"0"`. Make sure that the string is in the format supported by `time.Duration` |
| `key` | k | string | The key to be expired | Default value: `""`, which means the expiration is set for all keys |
| `option` | None | string | Additional options for `expiration`. **Only versions of Redis after 7.0.0 support this flag** | Default value: `""`. Only NX, XX, GT, and LT are supported |
| `password` | p | string | The password to log in to the server | Default value: `""` |

#### キャッシュ有効期限切れのシミュレート例

```bash
chaosd attack redis cache-expiration -a 127.0.0.1:6379 --option GT --expiration 1m
```

### コマンドラインモードを使用したキャッシュ制限のシミュレート

#### キャッシュ制限のコマンド

```bash
chaosd attack redis cache-limit -h
```

結果は以下の通りです：

```bash
set maxmemory of Redis

Usage:
  chaosd attack redis cache-limit [flags]

Flags:
  -a, --addr string       The address of redis server
  -h, --help              help for cache-limit
  -p, --password string   The password of server
      --percent string    The percentage of maxmemory
  -s, --size string       The size of cache (default "0")

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### キャッシュ制限の設定説明

| Configuration item | Abbreviation | Type | Description | Value |
| :-- | :-- | :-- | :-- | :-- |
| `addr` | a | string | The address and port of Redis server to be injected into the fault, such as `127.0.0.1:6379` | Default value: `""` |
| `password` | p | string | The password to log in to the server | Default value: `""` |
| `percent` | None | string | Specifies `maxmemory` as a percentage of the original value | Default value: `""` |
| `size` | s | string | Specifies the size of `maxmemory` | Default `0`, which means no limitation of memory |

#### キャッシュ制限のシミュレート例

```bash
chaosd attack redis cache-limit -a 127.0.0.1:6379 -s 256M
```

### コマンドラインモードを使用したキャッシュペネトレーションのシミュレート

このコマンドは、Redis Pipelineを使用して指定された数の`GET`リクエストをRedisサーバーに可能な限り迅速に送信します。リクエストされたキーはRedisサーバー上に存在しないため、これらのリクエストはキャッシュペネトレーション現象を引き起こします。

#### キャッシュペネトレーションのコマンド

```bash
chaosd attack redis cache-penetration -h
```

結果は以下の通りです：

```bash
penetrate cache

Usage:
  chaosd attack redis cache-penetration [flags]

Flags:
  -a, --addr string       The address of redis server
  -h, --help              help for cache-penetration
  -p, --password string   The password of server
      --request-num int   The number of requests

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### キャッシュペネトレーションの設定説明

| Configuration item | Abbreviation | Type | Description | Value |
| :-- | :-- | :-- | :-- | :-- |
| `addr` | a | string | The address and port of Redis server to be injected into the fault, such as `127.0.0.1:6379` | Default value: `""` |
| `password` | p | string | The password to log in to the server | Default value: `""` |
| `request-num` | None | int | Specifies the number of requests to be sent to the Redis server | Default value: `0` |

#### キャッシュペネトレーションのシミュレート例

```bash
chaosd attack redis cache-penetration -a 127.0.0.1:6379 --request-num 100000
```

### コマンドラインモードを使用したセンチネル再起動のシミュレート

#### センチネル再起動のコマンド

```bash
chaosd attack redis sentinel-restart -h
```

結果は以下の通りです：

```bash
restart sentinel

Usage:
  chaosd attack redis sentinel-restart [flags]

Flags:
  -a, --addr string         The address of redis server
  -c, --conf string         The config of Redis server
      --flush-config         Force Sentinel to rewrite its configuration on disk (default true)
  -h, --help                help for sentinel-restart
  -p, --password string     The password of server
      --redis-path string   The path of the redis-server command

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### センチネル再起動の設定説明

| Configuration item | Abbreviation | Type | Description | Value |
| :-- | :-- | :-- | :-- | :-- |
| `addr` | a | string | The address and port of Sentinel to be injected into the fault, such as `127.0.0.1:26379` | Default value: `""` |
| `conf` | c | string | Specifies the path of Sentinel config file, this file will be used to revover the Sentinel | Default value: `""` |
| `flush-config` | None | bool | Forces Sentinel to rewrite its configuration on disk, including the current Sentinel state | Default value: `true` |
| `password` | p | string | The password to log in to the server | Default value: `""` |
| `redis-path` | None | string | Specifies the path of `redis-server` command-line tool | Default value: `""` |

#### センチネル再起動のシミュレート例

```bash
chaosd attack redis sentinel-restart -a 127.0.0.1:26379 --conf /home/redis-test/sentinel-26379.conf
```

### コマンドラインモードを使用したセンチネル停止のシミュレート

#### センチネル停止のコマンド

```bash
chaosd attack redis sentinel-stop -h
```

結果は以下の通りです：

```bash
stop sentinel

Usage:
  chaosd attack redis sentinel-stop [flags]

Flags:
  -a, --addr string         The address of redis server
  -c, --conf string         The config path of Redis server
      --flush-config        Force Sentinel to rewrite its configuration on disk (default true)
  -h, --help                help for sentinel-stop
  -p, --password string     The password of server
      --redis-path string   The path of the redis-server command

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### センチネル停止の設定説明

| Configuration item | Abbreviation | Type | Description | Value |
| :-- | :-- | :-- | :-- | :-- |
| `addr` | a | string | The address and port of Sentinel to be injected into the fault, such as `127.0.0.1:26379` | Default value: `""` |
| `conf` | c | string | Specifies the path of Sentinel configuration file, which is used to recover the Sentinel | Default value: `""` |
| `flush-config` | None | bool | Forces Sentinel to rewrite its configuration on disk, including the current Sentinel state | Default value: `true` |
| `password` | p | string | The password to log in to the server | Default value: `""` |
| `redis-path` | None | string | Specifies the path of `redis-server` command-line tool | Default value: `""` |

#### センチネル再起動のシミュレート例

```bash
chaosd attack redis sentinel-stop -a 127.0.0.1:26379 --conf /home/redis-test/sentinel-26379.conf
```

## サービスモードを使用したRedis障害実験の作成

サービスモードを使用して実験を作成するには、以下の手順に従ってください：

1. Chaosdをサービスモードで実行します:

   ```bash
   chaosd server --port 31767
   ```

2. Chaosdサービスの`/api/attack/redis`パスに`POST` HTTPリクエストを送信します:

   ```bash
   curl -X POST 127.0.0.1:31767/api/attack/redis -H "Content-Type:application/json" -d '{fault-configuration}'
   ```

   上記のコマンドでは、障害タイプに応じて`fault-configuration`を設定する必要があります。対応するパラメータについては、以下のセクションで各障害タイプのパラメータと例を参照してください。

:::note

実験を実行する際は、実験のUIDを記録しておいてください。UIDに対応する実験を終了する場合は、Chaosdサービスの`/api/attack/{uid}`パスに`DELETE` HTTPリクエストを送信する必要があります。

:::

### サービスモードを使用してキャッシュ期限切れをシミュレート

#### キャッシュ期限切れシミュレーションのパラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Action of the experiment | string | set to "corrupt" |
| `addr` | The address and port of Redis server to be injected into the fault, such as `127.0.0.1:6379` | string | Default value: `""` |
| `expiration` | The specified key will be expired after `expiration` arrives | string | Default value: `"0"`. Make sure that the string is in the format supported by `time.Duration` |
| `key` | The key to be expired | string | Default value: `""`, which means the expiration is set for all keys |
| `option` | Additional options for `expiration`. **Only versions of Redis after 7.0.0 support this flag** | string | Default value: `""`. Only NX, XX, GT, and LT are supported |
| `password` | The password to log in to the server | Default value: `""` |

#### サービスモードを使用したキャッシュ期限切れシミュレーションの例

```bash
curl -X POST 127.0.0.1:31767/api/attack/redis -H "Content-Type:application/json" -d '{"action":"expiration", "expiration":"1m","addr":"127.0.0.1:6379"}'
```

### サービスモードを使用してキャッシュ制限をシミュレート

#### キャッシュ制限シミュレーションのパラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Action of the experiment | string | set to "cacheLimit" |
| `addr` | The address and port of Redis server to be injected into the fault, such as `127.0.0.1:6379` | string | Default value: `""` |
| `password` | The password to log in to the server | string | Default value: `""` |
| `percent` | Specifies `maxmemory` as a percentage of the original value | string | Default value: `""` |
| `size` | Specifies the size of `maxmemory` | string | Default `0`, which means no limitation of memory |

#### サービスモードを使用したキャッシュ制限シミュレーションの例

```bash
curl -X POST 127.0.0.1:31767/api/attack/redis -H "Content-Type:application/json" -d '{"action":"cacheLimit", ""addr":"127.0.0.1:6379", "percent":"50%"}'
```

### サービスモードを使用してキャッシュペネトレーションをシミュレート

#### キャッシュペネトレーションシミュレーションのパラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Action of the experiment | string | set to "penetration" |
| `addr` | The address and port of Redis server to be injected into the fault, such as `127.0.0.1:6379` | string | Default value: `""` |
| `password` | The password to log in to the server | string | Default value: `""` |
| `request-num` | Specifies the number of requests to be sent to the Redis server | int | Default value: `0` |

#### サービスモードを使用したキャッシュペネトレーションシミュレーションの例

```bash
curl -X POST 127.0.0.1:31767/api/attack/redis -H "Content-Type:application/json" -d '{"action":"penetration", ""addr":"127.0.0.1:6379", "request-num":"10000"}'
```

### サービスモードを使用してSentinel再起動をシミュレート

#### Sentinel再起動シミュレーションのパラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Action of the experiment | string | set to "restart" |
| `addr` | The address and port of Sentinel to be injected into the fault, such as `127.0.0.1:26379` | string | Default value: `""` |
| `conf` | Specifies the path of Sentinel configuration file, which is used to recover the Sentinel | string | Default value: `""` |
| `flush-config` | Forces Sentinel to rewrite its configuration on disk, including the current Sentinel state | bool | Default value: `true` |
| `password` | The password to log in to the server | string | Default value: `""` |
| `redis-path` | Specifies the path of `redis-server` command-line tool | string | Default value: `""` |

#### サービスモードを使用したSentinel再起動シミュレーションの例

```bash
curl -X POST 127.0.0.1:31767/api/attack/redis -H "Content-Type:application/json" -d '{"action":"restart", ""addr":"127.0.0.1:26379", "conf":"/home/redis-test/sentinel-26379.conf"}'
```

### サービスモードを使用してSentinel停止をシミュレート

#### Sentinel停止シミュレーションのパラメータ

| Parameter | Description | Type | Value |
| :-- | :-- | :-- | :-- |
| `action` | Action of the experiment | string | set to "stop" |
| `addr` | The address and port of Sentinel to be injected into the fault, such as `127.0.0.1:26379` | string | Default value: `""` |
| `conf` | Specifies the path of Sentinel configuration file, which is used to recover the Sentinel | string | Default value: `""` |
| `flush-config` | Forces Sentinel to rewrite its configuration on disk, including the current Sentinel state | bool | Default value: `true` |
| `password` | The password to log in to the server | string | Default value: `""` |
| `redis-path` | Specifies the path of `redis-server` command-line tool | string | Default value: `""` |

#### サービスモードを使用したSentinel停止シミュレーションの例

```bash
curl -X POST 127.0.0.1:31767/api/attack/redis -H "Content-Type:application/json" -d '{"action":"stop", ""addr":"127.0.0.1:26379", "conf":"/home/redis-test/sentinel-26379.conf"}'
```