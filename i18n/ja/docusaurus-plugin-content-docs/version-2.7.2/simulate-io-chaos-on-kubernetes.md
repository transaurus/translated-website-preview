---
title: Simulate File I/O Faults
---

このドキュメントでは、Chaos MeshでIOChaos実験を作成する方法について説明します。

## IOChaosの概要

IOChaosはChaos Meshの障害タイプの1つです。IOChaos実験を作成することで、ファイルシステム障害のシナリオをシミュレートできます。現在、IOChaosは以下の障害タイプをサポートしています：

- `latency`: ファイルシステムコールを遅延させる
- `fault`: ファイルシステムコールに対してエラーを返す
- `attrOverride`: ファイルの属性を変更する
- `mistake`: ファイルの読み書きで誤った値を返す

具体的な機能については、[YAMLファイルを使用した実験の作成](#create-experiments-using-the-yaml-files)を参照してください。

## 注意事項

1. IOChaos実験を作成する前に、対象のPodでChaos MeshのControl Managerが実行されていないことを確認してください。

2. IOChaosはデータを破損する可能性があります。本番環境では**注意して**使用してください。

## Chaos Dashboardを使用した実験の作成

1. Chaos Dashboardを開き、ページ上の**NEW EXPERIMENT**をクリックして新しい実験を作成します：

   ![新規実験の作成](./img/create-new-exp.png)

2. **Choose a Target**エリアで、**FILE SYSTEM INJECTION**を選択し、**LATENCY**などの特定の障害タイプを選択します。

   ![ioChaos実験](./img/iochaos-exp.png)

3. 実験情報を入力し、実験範囲と予定された実験期間を指定します。

   ![実験情報](./img/exp-info.png)

4. 実験情報を送信します。

## YAMLファイルを使用した実験の作成

### 遅延の例

1. 以下のように、実験設定を`io-latency.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: IOChaos
   metadata:
     name: io-latency-example
     namespace: chaos-mesh
   spec:
     action: latency
     mode: one
     selector:
       labelSelectors:
         app: etcd
     volumePath: /var/run/etcd
     path: '/var/run/etcd/**/*'
     delay: '100ms'
     percent: 50
     duration: '400s'
   ```

   この設定例では、Chaos Meshは`/var/run/etcd`ディレクトリに遅延を注入し、このディレクトリ内のすべてのファイルシステム操作（読み取り、書き込み、内容のリスト表示など）に100ミリ秒の遅延を引き起こします。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f ./io-latency.yaml
   ```

### 障害の例

1. 以下のように、実験設定を`io-fault.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: IOChaos
   metadata:
     name: io-fault-example
     namespace: chaos-mesh
   spec:
     action: fault
     mode: one
     selector:
       labelSelectors:
         app: etcd
     volumePath: /var/run/etcd
     path: /var/run/etcd/**/*
     errno: 5
     percent: 50
     duration: '400s'
   ```

   この例では、Chaos Meshは`/var/run/etcd`ディレクトリにファイル障害を注入し、このディレクトリ内のすべてのファイルシステム操作に50%の確率で失敗し、エラーコード5（Input/output error）を返します。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f ./io-fault.yaml
   ```

### attrOverrideの例

1. 実験設定を `io-attr.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: IOChaos
   metadata:
     name: io-attr-example
     namespace: chaos-mesh
   spec:
     action: attrOverride
     mode: one
     selector:
       labelSelectors:
         app: etcd
     volumePath: /var/run/etcd
     path: /var/run/etcd/**/*
     attr:
       perm: 72
     percent: 10
     duration: '400s'
   ```

   この設定例では、Chaos Mesh は `/var/run/etcd` ディレクトリに `attrOverride` 障害を注入し、このディレクトリ内のすべてのファイルシステム操作に対して10%の確率で対象ファイルのパーミッションを72（8進数で110）に変更します。これにより、ファイルは所有者とそのグループのみが実行可能となり、他の操作は許可されません。

2. 設定ファイルの準備ができたら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f ./io-attr.yaml
   ```

### Mistake の例

1. 実験設定を `io-mistake.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: IOChaos
   metadata:
     name: io-mistake-example
     namespace: chaos-mesh
   spec:
     action: mistake
     mode: one
     selector:
       labelSelectors:
         app: etcd
     volumePath: /var/run/etcd
     path: /var/run/etcd/**/*
     mistake:
       filling: zero
       maxOccurrences: 1
       maxLength: 10
     methods:
       - READ
       - WRITE
     percent: 10
     duration: '400s'
   ```

   この設定例では、Chaos Mesh は `/var/run/etcd` ディレクトリに読み書き障害を注入し、このディレクトリ内の読み書き操作に対して10%の確率で失敗を引き起こします。この過程で、最大10バイトの長さを持つランダムな位置の1か所が0バイトに置き換えられます。

2. 設定ファイルの準備ができたら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f ./io-mistake.yaml
   ```

### フィールド説明

#### 共通フィールド

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| action | string | Indicates the specific type of faults. Only latency, fault, attrOverride, and mistake are supported. |  | Yes | latency |
| mode | string | Specifies the mode of the experiment. The mode options include `one` (selecting a Pod at random), `all` (selecting all eligible Pods), `fixed` (selecting a specified number of eligible Pods), `fixed-percent` (selecting a specified percentage of the eligible Pods), and `random-max-percent` (selecting the maximum percentage of the eligible Pods). | None | Yes | `one` |
| selector | struct | Specifies the target Pod. For details, refer to [Define the experiment scope](./define-chaos-experiment-scope.md). | None | Yes |  |
| value | string | Provides parameters for the `mode` configuration, depending on `mode`. For example, when `mode` is set to `fixed-percent`, `value` specifies the percentage of Pods. |  | No | 1 |
| volumePath | string | The mount point of volume in the target container. Must be the root directory of the mount. |  | Yes | /var/run/etcd |
| path | string | The valid range of fault injections, either a wildcard or a single file. | Valid for all files by default | No | /var/run/etcd/\*_/_ |
| methods | []string | Type of the file system call that requires injecting fault. For more information about supported types, refer to [Appendix A](#appendix-a: methods-type). | All Types | No | READ |
| percent | int | Probability of failure per operation, in %. | 100 | No | 100 |
| containerNames | []string | Specifies the name of the container into which the fault is injected. |  | No |  |
| duration | string | Specifies the duration of the experiment. |  | Yes | 30s |

#### action に関連するフィールド

以下は、action に対応するフィールドの具体的な情報です:

- レイテンシ

  | パラメータ | タイプ   | 説明               | デフォルト値 | 必須 | 例       |
  | ---------- | -------- | ------------------ | ------------ | ---- | -------- |
  | delay      | string   | 具体的な遅延時間   |              | はい  | 100 ms   |

- フォルト

  | パラメータ | タイプ | 説明                 | デフォルト値 | 必須 | 例 |
  | ---------- | ----- | -------------------- | ------------ | ---- | -- |
  | errno      | int   | 返されるエラー番号   |              | はい  | 22 |

  一般的なエラー番号については、[付録B](#appendix-b-common-error-numbers)を参照してください。

- attrOverride

  | パラメータ | タイプ             | 説明                          | デフォルト値 | 必須 | 例         |
  | ---------- | ------------------ | ----------------------------- | ------------ | ---- | ----------- |
  | attr       | AttrOverrideSpec   | 特定のプロパティ上書きルール  |              | はい  | 下記参照    |

  AttrOverrideSpecは以下のように定義されます:

  | パラメータ | タイプ | 説明 | デフォルト値 | 必須 | 例 |
  | --- | --- | --- | --- | --- | --- |
  | ino | int | ino番号 |  | いいえ |  |
  | size | int | ファイルサイズ |  | いいえ |  |
  | blocks | int | ファイルが使用するブロック数 |  | いいえ |  |
  | atime | TimeSpec | 最終アクセス時間 |  | いいえ |  |
  | mtime | TimeSpec | 最終更新時間 |  | いいえ |  |
  | ctime | TimeSpec | 最終ステータス変更時間 |  | いいえ |  |
  | kind | string | ファイルタイプ、[fuser::FileType](https://docs.rs/fuser/0.7.0/fuser/enum.FileType.html)参照 |  | いいえ |  |
  | perm | int | ファイルパーミッション（10進数） |  | いいえ | 72（8進数で110） |
  | nlink | int | ハードリンク数 |  | いいえ |  |
  | `uid` | int | 所有者のユーザーID |  | いいえ |  |
  | gid | int | 所有者のグループID |  | いいえ |  |
  | rdev | int | デバイスID |  | いいえ |  |

  TimeSpecは以下のように定義されます:

  | パラメータ | タイプ | 説明              | デフォルト値 | 必須 | 例 |
  | ---------- | ------ | ----------------- | ------------ | ---- | -- |
  | sec        | int    | 秒単位のタイムスタンプ |              | いいえ |    |
  | nsec       | int    | ナノ秒単位のタイムスタンプ |              | いいえ |    |

  パラメータの具体的な意味については、[man stat](https://man7.org/linux/man-pages/man2/lstat.2.html)を参照してください。

- mistake

  | パラメータ | タイプ        | 説明           | デフォルト値 | 必須 | 例 |
  | ---------- | ------------- | -------------- | ------------ | ---- | -- |
  | mistake    | MistakeSpec   | 特定のエラールール |              | はい  |    |

  MistakeSpecは以下のように定義されます:

  | パラメータ | タイプ | 説明 | デフォルト値 | 必須 | 例 |
  | --- | --- | --- | --- | --- | --- |
  | filling | string | 埋め込む誤ったデータ。zero（0埋め）またはrandom（ランダムバイト埋め）のみサポートされます。 |  | はい |  |
  | maxOccurrences | int | 各操作での最大エラー発生回数。 |  | はい | 1 |
  | maxLength | int | 各エラーの最大長（バイト単位）。 |  | はい | 1 |

:::warning mistakeはREADおよびWRITEファイルシステムコールでのみ使用することを推奨します。他のファイルシステムコールでmistakeを使用すると、ファイルシステムの破損やプログラムクラッシュなど、予期せぬ結果を招く可能性があります。:::

## ローカルデバッグ

特定のChaosの効果がわからない場合は、[toda](https://github.com/chaos-mesh/toda)を使用してローカルで機能をテストできます。Chaos MeshもIOChaosの実装にtodaを使用しています。

## 付録A: methodsタイプ

- lookup
- forget
- getattr
- setattr
- readlink
- mknod
- mkdir
- unlink
- rmdir
- symlink
- rename
- link
- open
- read
- write
- flush
- release
- fsync
- opendir
- readdir
- releasedir
- fsyncdir
- statfs
- setxattr
- getxattr
- listxattr
- removexatr
- access
- create
- getlk
- setlk
- bmap

詳細については、[fuser::Filesystem](https://docs.rs/fuser/0.7.0/fuser/trait.Filesystem.html)を参照してください。

## 付録B: 一般的なエラー番号

- 1: 操作が許可されていません
- 2: ファイルまたはディレクトリが存在しません
- 5: I/Oエラー
- 6: デバイスまたはアドレスが存在しません
- 12: メモリ不足
- 16: デバイスまたはリソースがビジー状態です
- 17: ファイルが既に存在します
- 20: ディレクトリではありません
- 22: 無効な引数
- 24: 開いているファイルが多すぎます
- 28: デバイスに空き領域がありません

詳細については、[Linuxソースコード](https://raw.githubusercontent.com/torvalds/linux/master/include/uapi/asm-generic/errno-base.h)を参照してください。