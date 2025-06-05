---
title: Check Workflow Status
---

## Chaos Dashboardを使用したワークフローステータスの確認

1. Chaos Dashboardで全てのワークフローを一覧表示します。

   ![ダッシュボード上のワークフロー一覧](./img/list-workflow-on-dashboard.png)

2. 確認したいワークフローを選択し、その詳細を表示します。

   ![ダッシュボード上のワークフローステータス](./img/workflow-status-on-dashboard.png)

## `kubectl`コマンドを使用したワークフローステータスの確認

1. 以下のコマンドを実行して、指定したネームスペース内に作成された現在のワークフローを一覧表示します:

   ```shell
   kubectl -n <namespace> get workflow
   ```

2. 確認したいワークフローを選択し、以下のコマンドでワークフロー名を指定します。このコマンドを実行して、ワークフローの全てのノードを取得します:

   ```shell
   kubectl -n <namespace> get workflownode --selector="chaos-mesh.org/workflow=<workflow-name>"
   ```

   ワークフローのステップは、これらのワークフローノードの名前で表されます。

3. 以下のコマンドを実行して、指定したワークフローノードの詳細なステータスを取得します:

   ```shell
   kubectl -n <namespace> describe workflownode <workflow-node-name>
   ```

   出力には、現在のノードの実行が完了しているかどうか、その並列または直列ノードの実行ステータス、現在のノードに対応するChaos実験オブジェクトなどが含まれます。