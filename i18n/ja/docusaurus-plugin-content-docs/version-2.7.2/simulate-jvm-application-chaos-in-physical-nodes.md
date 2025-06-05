---
title: Simulate JVM Application Faults
---

Chaosdは[Byteman](https://github.com/chaos-mesh/byteman)を使用してJVMアプリケーションの障害をシミュレートします。サポートされている障害タイプは以下の通りです：

- カスタム例外のスロー
- ガベージコレクションのトリガー
- メソッドレイテンシの増加
- メソッドの戻り値の変更
- Byteman設定ファイルによる障害のトリガー
- JVM負荷の増加

このドキュメントでは、Chaosdを使用して上記のJVM実験の障害タイプを作成する方法について説明します。

## コマンドラインモードを使用した実験の作成

このセクションでは、コマンドラインモードを使用してJVMアプリケーション障害の実験を作成する方法を紹介します。

実験を作成する前に、以下のコマンドラインを実行してChaosdがサポートするJVMアプリケーション障害のタイプを確認できます：

```bash
chaosd attack jvm -h
```

結果は以下の通りです：

```bash
JVM attack related commands

Usage:
  chaosd attack jvm [command]

Available Commands:
  exception   throw specified exception for specified method
  gc          trigger GC for JVM
  latency     inject latency to specified method
  return      return specified value for specified method
  rule-file   inject fault with configured byteman rule file
  stress      inject stress to JVM

Flags:
  -h, --help       help for jvm
      --pid int    the pid of Java process which needs to attach
      --port int   the port of agent server (default 9288)

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID

Use "chaosd attack jvm [command] --help" for more information about a command.
```

### コマンドラインモードを使用したカスタム例外のスロー

#### カスタム例外スロー用コマンド

カスタム例外をスローするコマンドの使用方法と設定項目を確認するには、以下のコマンドを実行します：

```bash
chaosd attack jvm exception --help
```

結果は以下の通りです：

```bash
throw specified exception for specified method

Usage:
  chaosd attack jvm exception [options] [flags]

Flags:
  -c, --class string       Java class name
      --exception string   the exception which needs to throw for action 'exception'
  -h, --help               help for exception
  -m, --method string      the method name in Java class

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --pid int            the pid of Java process which needs to attach
      --port int           the port of agent server (default 9288)
      --uid string         the experiment ID
```

#### カスタム例外スローの設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `class` | `c` | The name of the Java class | string type, required |
| `exception` | None | The thrown custom exception | string type, required |
| `method` | `m` | The name of the method | string type, required to be configured |
| `pid` | None | The Java process ID where the fault is to be injected | int type, required |
| `port` | None | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | None | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### カスタム例外スローの例

```bash
chaosd attack jvm exception -c Main -m sayhello --exception 'java.io.IOException("BOOM")' --pid 30045
```

結果は以下の通りです：

```bash
[2021/08/05 02:39:39.106 +00:00] [INFO] [jvm.go:208] ["byteman rule"] [rule="\nRULE Main-sayhello-exception-q6nd0\nCLASS Main\nMETHOD sayhello\nAT ENTRY\nIF true\nDO \n\tthrow new java.io.IOException(\"BOOM\");\nENDRULE\n"] [file=/tmp/rule.btm296930759]
Attack jvm successfully, uid: 26a45ae2-d395-46f5-a126-2b2c6c85ae9d
```

### コマンドラインモードを使用したガベージコレクションのトリガー

#### ガベージコレクション用コマンド

ガベージコレクションをトリガーするコマンドの使用方法と設定項目を確認するには、以下のコマンドを実行します：

```bash
chaosd attack jvm gc --help
```

```bash
trigger GC for JVM

Usage:
  chaosd attack jvm gc [flags]

Flags:
  -h, --help   help for gc

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --pid int            the pid of Java process which needs to attach
      --port int           the port of agent server (default 9288)
      --uid string         the experiment ID
```

#### ガベージコレクションの設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `pid` | None | The Java process ID where the fault is to be injected | int type, required |
| `port` | None | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | None | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### ガベージコレクションの例

```bash
chaosd attack jvm gc --pid 89345
```

結果は以下の通りです：

```bash
[2021/08/05 02:49:47.850 +00:00] [INFO] [jvm.go:208] ["byteman rule"] [rule="\nRULE --gc-u0mlf\nGC\nENDRULE\n"] [file=/tmp/rule.btm012481052]
Attack jvm successfully, uid: f360e70a-5359-49b6-8526-d7e0a3c6f696
```

ガベージコレクションのトリガーは1回限りの操作であり、実験の復旧は不要です。

### コマンドラインモードを使用したメソッドレイテンシの増加

#### メソッドレイテンシ増加用コマンド

メソッドレイテンシを増加させるコマンドの使用方法と設定項目を確認するには、以下のコマンドを実行します：

```bash
chaosd attack jvm latency --help
```

結果は以下の通りです：

```bash
inject latency to specified method

Usage:
  chaosd attack jvm latency [options] [flags]

Flags:
  -c, --class string    Java class name
  -h, --help            help for latency
      --latency int     the latency duration, unit ms
  -m, --method string   the method name in Java class

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --pid int            the pid of Java process which needs to attach
      --port int           the port of agent server (default 9288)
      --uid string         the experiment ID
```

#### メソッドレイテンシ増加の設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `class` | `c` | The name of the Java class | string type, required |
| `latency` | None | The duration of increasing method latency | int type, required. The unit is millisecond. |
| `method` | `m` | The name of the method | string type, required |
| `pid` | None | The Java process ID where the fault is to be injected | int type, required |
| `port` | None | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | None | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### メソッドレイテンシ増加の例

```bash
chaosd attack jvm latency --class Main --method sayhello --latency 5000 --pid 100840
```

結果は以下の通りです：

```bash
[2021/08/05 03:08:50.716 +00:00] [INFO] [jvm.go:208] ["byteman rule"] [rule="\nRULE Main-sayhello-latency-hlib2\nCLASS Main\nMETHOD sayhello\nAT ENTRY\nIF true\nDO \n\tThread.sleep(5000);\nENDRULE\n"] [file=/tmp/rule.btm359997255]
[2021/08/05 03:08:51.155 +00:00] [INFO] [jvm.go:94] ["submit rules"] [output="install rule Main-sayhello-latency-hlib2\n\n"]
Attack jvm successfully, uid: bbe00c57-ac9d-4113-bf0c-2a6f184be261
```

### コマンドラインモードを使用したメソッドの戻り値の変更

#### メソッド戻り値変更用コマンド

メソッドの戻り値を変更するコマンドの使用方法と設定項目を確認するには、以下のコマンドを実行します：

```bash
chaosd attack jvm return --help
```

```bash
return specified value for specified method

Usage:
  chaosd attack jvm return [options] [flags]

Flags:
  -c, --class string    Java class name
  -h, --help            help for return
  -m, --method string   the method name in Java class
      --value string    the return value for action 'return'. Only supports number and string types.

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --pid int            the pid of Java process which needs to attach
      --port int           the port of agent server (default 9288)
      --uid string         the experiment ID
```

#### メソッド戻り値変更の設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| class | c | The name of the Java class | string type, required to be configured |
| method | `m` | The name of the method | string type, required to be configured |
| value | None | Specifies the return value of the method | string type, required to be configured. Currently, the item can be numeric and string types. If the item (return value) is string, double quotes are required, like "chaos". |
| pid | None | The Java process ID where the fault is needed to be injected | int type, required to be configured |
| port | None | The port number attached to the Java process agent. The faults is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | None | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### メソッド戻り値変更のシナリオ例

```bash
chaosd attack jvm return --class Main --method getnum --value 999 --pid 112694
```

結果は以下の通りです：

```bash
[2021/08/05 03:35:10.603 +00:00] [INFO] [jvm.go:208] ["byteman rule"] [rule="\nRULE Main-getnum-return-i6gb7\nCLASS Main\nMETHOD getnum\nAT ENTRY\nIF true\nDO \n\treturn 999;\nENDRULE\n"] [file=/tmp/rule.btm051982059]
[2021/08/05 03:35:10.820 +00:00] [INFO] [jvm.go:94] ["submit rules"] [output="install rule Main-getnum-return-i6gb7\n\n"]
Attack jvm successfully, uid: e2f204f6-4bed-4d92-aade-2b4a47b02e5d
```

### コマンドラインモードを使用したByteman設定ファイルによる障害のトリガー

Bytemanルール設定ファイルに障害ルールを設定し、Chaosdを使用して設定ファイルのパスを指定することで障害を注入できます。Bytemanルール設定については、[byteman-rule-language](https://downloads.jboss.org/byteman/4.0.16/byteman-programmers-guide.html#the-byteman-rule-language)を参照してください。

#### Byteman設定ファイルを使用した障害発生コマンド

Byteman設定ファイルを使用して障害を発生させるコマンドの使用方法と設定項目を確認するには、次のコマンドを実行します：

```bash
chaosd attack jvm rule-file --help
```

結果は次のようになります：

```bash
inject fault with configured byteman rule file

Usage:
  chaosd attack jvm rule-file [options] [flags]

Flags:
  -h, --help          help for rule-file
  -p, --path string   the path of configured byteman rule file

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
      --pid int            the pid of Java process which needs to attach
      --port int           the port of agent server (default 9288)
      --uid string         the experiment ID
```

#### Byteman設定ファイルを使用した障害発生の設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `path` | None | Specifies the path of the Byteman configuration file | string type, required |
| `pid` | None | The Java process ID where the fault is to be injected | int type, required |
| `port` | None | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | None | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### Byteman設定ファイルを使用した障害発生の例

まず、特定のJavaプログラムに基づき、[Bytemanルール言語](https://downloads.jboss.org/byteman/4.0.16/byteman-programmers-guide.html#the-byteman-rule-language)を参照してルール設定ファイルを作成します。例：

```txt
RULE modify return value
CLASS Main
METHOD getnum
AT ENTRY
IF true
DO
    return 9999
ENDRULE
```

次に、設定ファイルを`return.btm`ファイルに保存します。その後、次のコマンドを実行して障害を注入します。

```bash
chaosd attack jvm rule-file -p ./return.btm --pid 112694
```

結果は次のようになります：

```bash
[2021/08/05 03:45:40.757 +00:00] [INFO] [jvm.go:152] ["rule file data:RULE modify return value\nCLASS Main\nMETHOD getnum\nAT ENTRY\nIF true\nDO\n    return 9999\nENDRULE\n"]
[2021/08/05 03:45:41.011 +00:00] [INFO] [jvm.go:94] ["submit rules"] [output="install rule modify return value\n\n"]
Attack jvm successfully, uid: 5ca2e06d-a7c6-421d-bb67-0c9908bac17a
```

#### コマンドラインモードでのJVM負荷増加

#### JVM負荷増加コマンド

JVM負荷を増加させるコマンドの使用方法と設定項目を確認するには、次のコマンドを実行します：

```bash
chaosd attack jvm stress --help
```

結果は次のようになります：

```bash
inject stress to JVM

Usage:
  chaosd attack jvm stress [options] [flags]

Flags:
      --cpu-count int   the CPU core number
  -h, --help            help for stress
      --mem-type int    the memory type to be allocated. The value can be 'stack' or 'heap'.

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --pid int            the pid of Java process which needs to attach
      --port int           the port of agent server (default 9288)
      --uid string         the experiment ID
```

#### JVM負荷増加の設定説明

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| `cpu-count` | None | The number of CPU cores used for increasing JVM stress | int type. You must configure one item between `cpu-count` and `mem-type`. |
| `mem-type` | None | The type of OOM | string type. Currently, both 'stack' and 'heap' OOM types are supported. You must configure one item between `cpu-count` and `mem-type`. |
| `pid` | None | The Java process ID where the fault is to be injected | int type, required |
| `port` | None | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | None | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### JVM負荷増加の例

```bash
chaosd attack jvm stress --cpu-count 2 --pid 123546
```

結果は次のようになります：

```bash
[2021/08/05 03:59:51.256 +00:00] [INFO] [jvm.go:208] ["byteman rule"] [rule="\nRULE --stress-jfeiu\nSTRESS CPU\nCPUCOUNT 2\nENDRULE\n"] [file=/tmp/rule.btm773062009]
[2021/08/05 03:59:51.613 +00:00] [INFO] [jvm.go:94] ["submit rules"] [output="install rule --stress-jfeiu\n\n"]
Attack jvm successfully, uid: b9b997b5-0a0d-4f1f-9081-d52a32318b84
```

## サービスモードでの実験作成

以下の手順に従って、サービスモードで実験を作成できます。

1. Chaosdをサービスモードで実行します：

   ```bash
   chaosd server --port 31767
   ```

2. Chaosdサービスの`/api/attack/{uid}`パスにHTTP POSTリクエストを送信します。

   ```bash
   curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{fault-configuration}'
   ```

   上記コマンドの`fault-configuration`部分は、障害タイプに応じて設定する必要があります。対応するパラメータについては、以下のセクションの各障害タイプのパラメータと例を参照してください。

:::note

実験を実行する際は、実験のUID情報を保存することを忘れないでください。UIDに対応する実験を終了する場合は、Chaosdサービスの`/api/attack/{uid}`パスにHTTP DELETEリクエストを送信する必要があります。

:::

#### サービスモードでのカスタム例外発生

#### カスタム例外発生のパラメータ

| Parameter | Description | Value |
| :-- | :-- | :-- |
| `action` | The action of the experiment | Set to "exception" |
| `class` | The name of the Java class | string type, required |
| `exception` | The thrown custom exception | string type, required |
| `method` | The name of the method | string type, required |
| `pid` | The Java process ID where the fault is to be injected | int type, required |
| `port` | The port number attached to the Java process agent. The faults is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### サービスモードでのカスタム例外発生の例

```bash
curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{"action":"exception","class":"Main","method":"sayhello","exception":"java.io.IOException(\"BOOM\")","pid":1828622}'
```

結果は次のようになります：

```bash
{"status":200,"message":"attack successfully","uid":"c3c519bf-819a-4a7b-97fb-e3d0814481fa"}
```

#### サービスモードでのガベージコレクションのトリガー

#### ガベージコレクションのトリガーパラメータ

| Parameter | Description | Value |
| :-- | :-- | :-- |
| `action` | The action of the experiment | Set to "gc" |
| `pid` | The Java process ID where the fault is to be injected | int type, required |
| `port` | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### サービスモードでのガベージコレクションのトリガー例

```bash
curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{"action":"gc","pid":1828622}'
```

結果は次のようになります：

```bash
{"status":200,"message":"attack successfully","uid":"c3c519bf-819a-4a7b-97fb-e3d0814481fa"}
```

ガベージコレクションのトリガーは1回限りの操作です。この実験ではリカバリーは不要です。

### サービスモードを使用してメソッドの遅延を増加させる

#### メソッドの遅延を増加させるためのパラメータ

| Parameter | Description | Value |
| :-- | :-- | :-- |
| `action` | The action of the experiment | Set to "latency" |
| `class` | The name of the Java class | string type, required |
| `latency` | The duration of increasing method latency | int type, required. The unit is millisecond. |
| `method` | The name of the method | string type, required |
| `pid` | The Java process ID where the fault is to be injected | int type, required |
| `port` | The Java process ID where the fault is needed to be injected | int type, required |
| `uid` | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### サービスモードを使用してメソッドの遅延を増加させる例

```bash
curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{"action":"latency","class":"Main","method":"sayhello","latency":5000,"pid":1828622}'
```

結果は以下の通りです：

```bash
{"status":200,"message":"attack successfully","uid":"a551206c-960d-4ac5-9056-518e512d4d0d"}
```

### サービスモードを使用してメソッドの戻り値を変更する

#### メソッドの戻り値を変更するためのパラメータ

| Parameter | Description | Value |
| :-- | :-- | :-- |
| `action` | The action of the experiment | Set to "return" |
| `class` | The name of the Java class | string type, required |
| `method` | The name of the method | string type, required |
| `value` | Specifies the return value of the method | string type, required. Currently, the item can be numeric and string types. If the item (return value) is string, double quotes are required, like "chaos". |
| `pid` | The Java process ID where the fault is to be injected | int type, required |
| `port` | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### サービスモードを使用してメソッドの戻り値を変更する例

```bash
curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{"action":"return","class":"Main","method":"getnum","value":"999","pid":1828622}'
```

結果は以下の通りです：

```bash
{"status":200,"message":"attack successfully","uid":"a551206c-960d-4ac5-9056-518e512d4d0d"}
```

### サービスモードを使用してByteman設定ファイルを設定して障害をトリガーする

Bytemanルール設定に従って障害ルールを設定できます。Bytemanルール設定の詳細については、[byteman-rule-language](https://downloads.jboss.org/byteman/4.0.16/byteman-programmers-guide.html#the-byteman-rule-language)を参照してください。

#### Byteman設定ファイルを設定して障害をトリガーするためのパラメータ

| Parameter | Description | Value |
| :-- | :-- | :-- |
| `action` | The action of the experiment | Set to "rule-data" |
| `rule-data` | Specifies the Byteman configuration data | string type, required |
| `pid` | The Java process ID where the fault is to be injected | int type, required |
| `port` | The port number attached to the Java process agent. The fault is injected into the Java process through this port number. | int type. The default value is `9288`. |
| `uid` | The experiment ID | string type. This item is not required to be configured, because Chaosd randomly creates one. |

#### サービスモードを使用してByteman設定ファイルを設定して障害をトリガーする例

まず、特定のJavaプログラムに基づき、[Bytemanルール言語](https://downloads.jboss.org/byteman/4.0.16/byteman-programmers-guide.html#the-byteman-rule-language)を参照してルール設定ファイルを作成します。例：

```txt
RULE modify return value
CLASS Main
METHOD getnum
AT ENTRY
IF true
DO
    return 9999
ENDRULE
```

次に、設定ファイルの改行を改行文字"\n"にエスケープし、エスケープされたテキストを"rule-data"の値として使用します。以下のコマンドを実行します：

```bash
curl -X POST 127.0.0.1:31767/api/attack/jvm -H "Content-Type:application/json" -d '{"action":"rule-data","pid":30045,"rule-data":"\nRULE modify return value\nCLASS Main\nMETHOD getnum\nAT ENTRY\nIF true\nDO return 9999\nENDRULE\n"}'
```

結果は以下の通りです：

```bash
{"status":200,"message":"attack successfully","uid":"a551206c-960d-4ac5-9056-518e512d4d0d"}
```

### サービスモードを使用してJVMストレスを増加させる

#### JVMストレスを増加させるためのパラメータ

| パラメータ | 説明 | 値 |
| :-- | :-- | :-- | --- |
| `action` | 実験のアクション | "stress"に設定 |
| `cpu-count` | CPUストレスを増加させるために使用するCPUコア数 | int型。`cpu-count`と`mem-type`のいずれかを設定する必要があります。 |
| `mem-type` | OOMのタイプ | string型。現在は'stack'と'heap'のOOMタイプがサポートされています。`cpu-count`と`mem-type`のいずれかを設定する必要があります。 |
| `pid` | なし | 障害を注入するJavaプロセスID | int型、必須 |
| `port` | なし | Javaプロセスエージェントに接続するポート番号。このポート番号を通じてJavaプロセスに障害が注入されます。 | int型。デフォルト値は`9288`。 |
| `uid` | なし | 実験ID | string型。Chaosdがランダムに生成するため、設定は不要です。 |

#### サービスモードを使用してJVMストレスを増加させる例

```bash
curl -X POST 172.16.112.130:31767/api/attack/jvm -H "Content-Type:application/json" -d '{"action":"stress","cpu-count":1,"pid":1828622}'
```

結果は以下の通りです：

```bash
{"status":200,"message":"attack successfully","uid":"a551206c-960d-4ac5-9056-518e512d4d0d"}
```