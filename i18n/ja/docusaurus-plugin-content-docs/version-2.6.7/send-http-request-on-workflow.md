---
title: Send HTTP Requests on Workflow
---

Chaos Mesh Workflowは、Kubernetesの`Job`と同様の機能を持つ`Task`ノードを提供し、あらゆるワークロードをサポートします。ユーザーエクスペリエンスを向上させるため、Chaos Dashboardは`Task`ベースのテンプレートを提供し、WebUIでHTTPリクエストを簡単に作成できます。

Chaos Mesh Workflowは、Kubernetesの`Job`と同様の機能を持つ`Task`ノードを提供し、あらゆるワークロードをサポートします。ユーザーエクスペリエンスを向上させるため、Chaos Dashboardは`Task`ベースのテンプレートを提供しています。このテンプレートを使用すると、WebUIで簡単にHTTPリクエストを作成できます。このドキュメントでは、Chaos Dashboardを通じてHTTPリクエストを作成する方法について説明します。

:::note

Chaos Meshには`HTTPRequest`タイプのワークフローノードは存在**しません**。この機能は`Task`ノードをベースとしており、この機能を使用することでHTTPリクエストをより簡単に送信できます。

:::

:::note

この機能は現在実験的機能であるため、本番環境での使用は推奨されません。機能の構成や動作は将来変更される可能性があります。

:::

## Chaos DashboardでHTTPリクエストを作成する

Slack Webhookを使用してメッセージを送信する例を以下に示します。

### ステップ1. `HTTPRequest`タイプのワークフローノードを作成

タスクタイプとして「HTTP Request」を選択します：

![create-http-request-workflow-node](img/create-http-request-workflow-node.png)

### ステップ2. HTTPリクエストを設定

以下の設定を行います：

- ノード名: `send-slack-message`
- リクエストURL: `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX`
- リクエストメソッド: `POST`
- リクエストボディ: `{"text": "Hello, world."}` 、その後「For JSON content」をチェックします。

![configure-http-request-workflow-node](img/configure-http-request-workflow-node.png)

### ステップ3. ワークフローノードを送信

「Submit」ボタンをクリックして、プレビューウィンドウでタスクを確認します：

![http-request-task-node-preview](img/http-request-task-node-preview.png)

## フィールド説明

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| Name | string | Name of the workflow node |  | Yes | `send-slack-message` |
| URL | string | URL of an HTTP request |  | Yes | `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX` |
| Method | string | Method of an HTTP request |  | Yes | `POST` |
| Body | string | Body of an HTTP request |  | No | `{"text": "Hello, world."}` |
| Follow 301/302 Location | boolean | The value of this parameter corresponds to the `-L` parameter of `curl`. | `false` | No | `false` |
| Json Content | boolean | The parameter appends `Content-Type: application/json` to the header of an HTTP request. | `false` | No | `false` |

生成されたタスクノードの`name`フィールドの値は、「name」の末尾に`http-request`という接尾辞が追加されます。