---
title: Status Check in Workflow
---

Workflowでは、ステータスチェックを使用して、アプリケーションシステムや監視システムなどの外部システムに対して指定された操作を実行し、それらの状態を取得できます。システムが不健全であることが判明した場合、自動的にWorkflowを中止します。この概念はKubernetesの`Container Probes`と似ています。この記事では、YAMLファイルを使用してWorkflowでステータスチェックを実行する方法について説明します。

:::note

Chaos Meshは現在、Chaos Dashboard上で`StatusCheck`ノードを作成する機能をサポートしていないため、現時点ではYAMLを使用して`StatusCheck`ノードを作成する必要があります。

:::

## ステータスチェックのタイプ

Chaos Meshは、ステータスチェックを実行するための`HTTP`タイプのみをサポートしています。

### `HTTP`タイプの`StatusCheck`ノードを定義する

`StatusCheck`ノードは、特定のURLに`GET`または`POST` HTTPリクエストを送信し、カスタムヘッダーとボディを使用して、`criteria`フィールドの条件に基づいてリクエストの結果を判定します。

```yaml
- name: workflow-status-check
  templateType: StatusCheck
  deadline: 20s
  statusCheck:
    mode: Continuous
    type: HTTP
    intervalSeconds: 1
    timeoutSeconds: 1
    http:
      url: http://123.123.123.123
      method: GET
      criteria:
        statusCode: '200'
```

この設定では、`HTTP`タイプの`StatusCheck`ノードを確認できます。`deadline`フィールドは、このノードが最大20秒間実行可能であることを指定します。`mode`フィールドは、このノードが継続的にステータスチェックを実行することを指定します。`intervalSeconds`フィールドは、1秒間隔で繰り返すことを指定します。`timeoutSeconds`フィールドは、各実行のタイムアウトを指定します。

Workflowがこの`StatusCheck`ノードに到達すると、指定されたステータスチェックが毎秒実行されます。ステータスチェックは、`GET`メソッドを使用してURL `http://123.123.123.123`にHTTPリクエストを送信します。レスポンスが1秒以内に返され、ステータスコードが`200`の場合、この実行は成功とみなされます。それ以外の場合は失敗とみなされます。

## ステータスチェックの結果

ステータスチェックの各実行は、`Success`または`Failure`のいずれかの`execution result`を取得します。単一の`execution result`は、特定の条件の変動により、システムの実際の状況を反映しない可能性があるため、最終的な`status check result`は単一の`execution result`に基づいて決定されません。

`StatusCheck`ノードには`failureThreshold`と`successThreshold`の2つのフィールドがあります：

- 連続して失敗した`execution result`の数が`failureThreshold`を超えると、`status check result`は`Failure`とみなされます。
- 連続して成功した`execution result`の数が`successThreshold`を超えると、`status check result`は`Success`とみなされます。

```yaml
- name: workflow-status-check
  templateType: StatusCheck
  deadline: 20s
  statusCheck:
    mode: Continuous
    type: HTTP
    successThreshold: 1
    failureThreshold: 3
    http:
      url: http://123.123.123.123
      method: GET
      criteria:
        statusCode: '200'
```

この設定では、`StatusCheck`ノードは継続的にステータスチェックを実行します：

- `execution result`が1回以上連続して`Success`の場合、`status check result`は`Success`とみなされます。
- `execution result`が3回以上連続して`Failure`の場合、`status check result`は`Failure`とみなされます。

:::note

以下のセクションで「ステータスチェックが失敗する」とは、単一の`execution result`が`Failure`であることではなく、`status check result`が`Failure`であることを指します。

:::

### ステータスチェックが失敗した場合にWorkflowを中止する

:::note

`StatusCheck`ノードは、ステータスチェックが失敗した場合にのみWorkflowを自動的に中止する機能をサポートしています。Workflowを一時停止または再開することはできません。

:::

カオス実験を実行している際に、アプリケーションシステムが「不健全」になる可能性があります。この機能を使用すると、カオス実験を迅速に終了させることでアプリケーションシステムを復旧できます。ステータスチェックが失敗した場合にWorkflowを自動的に中止するには、`StatusCheck`ノードの`abortWithStatusCheck`フィールドを`true`に設定します。

```yaml
- name: workflow-status-check
  templateType: StatusCheck
  deadline: 20s
  abortWithStatusCheck: true
  statusCheck:
    mode: Continuous
    type: HTTP
    http:
      url: http://123.123.123.123
      method: GET
      criteria:
        statusCode: '200'
```

以下のいずれかの条件が満たされた場合、ステータスチェックは失敗とみなされます：

- ステータスチェックが失敗した場合。
- `StatusCheck`ノードのタイムアウトが発生し、`status check result`が成功していない場合。例えば、`successThreshold`が1、`failureThreshold`が3で、タイムアウト時に2回連続で失敗し、0回成功した場合。この場合、「ステータスチェック失敗」の条件を満たしていなくても、不成功とみなされます。

## ステータスチェックモード

### 継続的ステータスチェック

`mode`フィールドが`Continuous`の場合、この`StatusCheck`ノードはノードがタイムアウトするか、ステータスチェックが失敗するまで継続的にステータスチェックを実行します。

```yaml
- name: workflow-status-check
  templateType: StatusCheck
  deadline: 20s
  statusCheck:
    mode: Continuous
    type: HTTP
    intervalSeconds: 1
    successThreshold: 1
    failureThreshold: 3
    http:
      url: http://123.123.123.123
      method: GET
      criteria:
        statusCode: '200'
```

この設定では、`StatusCheck`ノードは毎秒ステータスチェックを実行し、以下のいずれかの条件が満たされると終了します：

- ステータスチェックが失敗した場合（3回以上連続で`execution result`が失敗）
- 20秒後にノードのタイムアウトが発生した場合

### ワンタイムステータスチェック

`mode`フィールドが`Synchronous`の場合、この`StatusCheck`ノードは`status check result`が確定するか、ノードがタイムアウトすると即座に終了します。

```yaml
- name: workflow-status-check
  templateType: StatusCheck
  deadline: 20s
  statusCheck:
    mode: Synchronous
    type: HTTP
    intervalSeconds: 1
    successThreshold: 1
    failureThreshold: 3
    http:
      url: http://123.123.123.123
      method: GET
      criteria:
        statusCode: '200'
```

この設定では、`StatusCheck`ノードは毎秒ステータスチェックを実行し、以下のいずれかの条件が満たされると終了します：

- ステータスチェックが成功した場合（1回以上連続で`execution result`が成功）
- ステータスチェックが失敗した場合（3回以上連続で`execution result`が失敗）
- 20秒後にノードのタイムアウトが発生した場合

## StatusCheck vs HTTPリクエストタスク

共通点：

- `StatusCheck`ノードと`HTTP Request Task`ノード（HTTPリクエストを実行する`Task`ノード）は、どちらもWorkflowのノードタイプです。
- `StatusCheck`ノードと`HTTP Request Task`ノードは、HTTPリクエストを通じて外部システムの情報を取得できます。

相違点：

- `HTTP Request Task`ノードはHTTPリクエストを1回しか送信できず、継続的に送信することはできません。
- `HTTP Request Task`ノードは、リクエストが失敗した場合にワークフローの状態に影響を与えることができません（例：ワークフローを中止するなど）。

## フィールド説明

WorkflowとTemplateの詳細については、[Chaos Mesh Workflowの作成](create-chaos-mesh-workflow.md#field-description)を参照してください。

### StatusCheckフィールド説明

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| mode | `string` | The execution mode of the status check. Support value: `Synchronous`/`Continuous`. | None | Yes | `Synchronous` |
| type | `string` | The type of the status check. Support value: `HTTP`. | `HTTP` | Yes | `HTTP` |
| duration | `string` | The duration of the whole status check if the number of failed execution does not exceed the `failureThreshold`. It is available in both `Synchronous` and `Continuous` modes. | None | No | `100s` |
| timeoutSeconds | `int` | The timeout seconds when the status check fails. | `1` | No | `1` |
| intervalSeconds | `int` | Defines how often (in seconds) to perform an execution of status check. | `1` | No | `1` |
| failureThreshold | `int` | The minimum consecutive failure for the status check to be considered failed. | `3` | No | `3` |
| successThreshold | `int` | The minimum consecutive successes for the status check to be considered successful. | `1` | No | `1` |
| recordsHistoryLimit | `int` | The number of records to retain. | `100` | No | `100` |
| http | `HTTPStatusCheck` | Configure the detail of the HTTP request to execute. | None | No |  |

### HTTPStatusCheckフィールド説明

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| url | `string` | The URL of the HTTP request. | None | Yes | `http://123.123.123.123` |
| method | `string` | The method of the HTTP request. Support value: `GET`/`POST`. | `GET` | No | `GET` |
| headers | `map[string][]string` | The headers of the HTTP request. | None | No |  |
| body | `string` | The body of the HTTP request. | None | No | `{"a":"b"}` |
| criteria | `HTTPCriteria` | Defines how to determine the result of the HTTP StatusCheck. | None | Yes |  |

### HTTPCriteriaフィールド説明

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| statusCode | `string` | The expected http status code for the request. A statusCode string could be a single code (e.g. `200`), or an inclusive range (e.g. `200-400`, both `200` and `400` are included). | None | Yes | `200` |