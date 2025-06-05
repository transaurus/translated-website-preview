---
slug: /building_automated_testing_framework
title: Building an Automated Testing Framework Based on Chaos Mesh and Argo
authors: chaos-mesh
image: /img/blog/automated_testing_framework.png
tags: [Chaos Mesh, Chaos Engineering, Test Automation]
---

![TiPocket - 自動テストフレームワーク](/img/blog/automated_testing_framework.png)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、Kubernetes向けのオープンソースのカオスエンジニアリングプラットフォームです。異常なシステム状態をシミュレートする豊富な機能を提供していますが、それでもカオスエンジニアリングのパズルの一部しか解決していません。障害注入に加えて、完全なカオスエンジニアリングアプリケーションは、定義された安定状態に関する仮説の構築、本番環境での実験の実行、テストケースによるシステムの検証、そしてテストの自動化で構成されます。

この記事では、分散データベースTiDBのための完全なカオスエンジニアリングテストループを構築するために、自動テストフレームワーク[TiPocket](https://github.com/pingcap/tipocket)をどのように使用しているかを説明します。

<!--truncate-->

## TiPocketが必要な理由

[TiDB](https://github.com/pingcap/tidb)のような分散システムを本番環境に導入する前に、日常使用に十分な堅牢性を確保する必要があります。このため、数年前からテストフレームワークにカオスエンジニアリングを導入しました。テストフレームワークでは、以下のことを行います：

1. 通常のメトリクスを観察し、テスト仮説を構築します。
2. TiDBに一連の障害を注入します。
3. 障害シナリオでTiDBを検証するための様々なテストケースを実行します。
4. テスト結果を監視・収集し、分析と診断を行います。

これは堅実なプロセスに聞こえ、私たちは何年もこれを使用してきました。しかし、TiDBが進化するにつれて、テストの規模は倍増します。Kubernetesテストクラスターで実行される数十のテストケースに対して、複数の障害シナリオがあります。Chaos Meshが障害注入を支援しているとしても、残りの作業は依然として要求が高く、テストをスケーラブルかつ効率的にするためのパイプラインの自動化という課題もあります。

これが、KubernetesとChaos Meshに基づいた完全に自動化されたテストフレームワークであるTiPocketを構築した理由です。現在、主にTiDBクラスターのテストに使用しています。しかし、TiPocketのKubernetesフレンドリーな設計と拡張可能なインターフェースにより、Kubernetesの作成および削除ロジックを使用して他のアプリケーションを簡単にサポートできます。

## 動作原理

上記の要件に基づいて、以下の自動ワークフローが必要です：

- [カオスの注入](#injecting-chaos---chaos-mesh)
- [カオスの影響の検証](#verifying-chaos-impacts-test-cases)
- [カオスパイプラインの自動化](#automating-the-chaos-pipeline---argo)
- [結果の可視化](#visualizing-the-results-loki)

### カオスの注入 - Chaos Mesh

障害注入はカオステストの中核です。分散データベースでは、障害はいつでも、どこでも発生する可能性があります—ノードクラッシュ、ネットワーク分断、ファイルシステム障害、カーネルパニックなどです。ここでChaos Meshが役立ちます。

現在、TiPocketは以下のタイプの障害注入をサポートしています：

- **ネットワーク**: ネットワーク分断、ランダムなパケット損失、リンクの順序不同、重複、または遅延をシミュレートします。
- **時間のずれ**: テスト対象コンテナのクロックスキューをシミュレートします。
- **強制終了**: 指定されたpodを強制終了します。クラスター内でランダムに、またはコンポーネント（TiDB、TiKV、またはPlacement Driver（PD））内で行います。
- **I/O**: TiDBのストレージエンジンであるTiKVにI/O遅延を注入し、I/O関連の問題を特定します。

障害注入が処理されたら、検証について考える必要があります。TiDBがこれらの障害を乗り越えられることをどのように確認しますか？

## カオスの影響の検証: テストケース

TiDBがカオスに耐える方法を検証するために、TiPocketに数十のテストケースを実装し、様々な検査ツールと組み合わせました。障害発生時にTiPocketがどのようにTiDBを検証するかの概要を理解するために、以下のテストケースを考えてみてください。これらのケースは、SQL実行、トランザクションの一貫性、およびトランザクション分離に焦点を当てています。

### ファジーテスト: SQLsmith

[SQLsmith](https://github.com/pingcap/tipocket/tree/master/pkg/go-sqlsmith)は、ランダムなSQLクエリを生成するツールです。TiPocketはTiDBクラスターとMySQLインスタンスを作成します。SQLsmithによって生成されたランダムなSQLはTiDBとMySQLで実行され、TiDBクラスターにさまざまな障害が注入されてテストされます。最終的に実行結果が比較されます。もし不一致が検出された場合、システムに潜在的な問題がある可能性があります。

### トランザクション一貫性テスト: BankとPorcupine

[Bank](https://github.com/pingcap/tipocket/tree/master/cmd/bank)は、銀行システムの送金プロセスをシミュレートする古典的なテストケースです。スナップショット分離の下では、すべての送金はシステム障害が発生した場合でも、すべての口座の合計金額が常に一貫していることを保証しなければなりません。合計金額に不整合がある場合、システムに潜在的な問題がある可能性があります。

[Porcupine](https://github.com/anishathalye/porcupine)は、分散システムの正確性をテストするために構築されたGo言語の線形化可能性チェッカーです。これは、実行可能なGoコードとしての順次仕様と並行履歴を受け取り、その履歴が順次仕様に対して線形化可能かどうかを判断します。TiPocketでは、[Porcupine](https://github.com/pingcap/tipocket/tree/master/pkg/check/porcupine)チェッカーを複数のテストケースで使用し、TiDBが線形化可能性の制約を満たしているかどうかを確認します。

### トランザクション分離レベルテスト: Elle

[Elle](https://github.com/jepsen-io/elle)は、データベースのトランザクション分離レベルを検証する検査ツールです。TiPocketは[go-elle](https://github.com/pingcap/tipocket/tree/master/pkg/elle)（Elle検査ツールのGo実装）を統合し、TiDBの分離レベルを検証します。

これらは、TiPocketがTiDBの正確性と安定性を検証するために使用するテストケースの一部に過ぎません。より多くのテストケースと検証方法については、[ソースコード](https://github.com/pingcap/tipocket)を参照してください。

## カオステストパイプラインの自動化 - Argo

Chaos Meshで障害を注入し、テスト用のTiDBクラスターとTiDBを検証する方法が用意できた今、どのようにしてカオステストパイプラインを自動化できるでしょうか？2つの選択肢が考えられます。TiPocket内でスケジューリング機能を実装するか、既存のオープンソースツールに仕事を任せるかです。TiPocketをワークフローのテスト部分に専念させるため、私たちはオープンソースツールのアプローチを選択しました。これに加えて、すべてをK8s上で設計するという方針から、私たちは直接[Argo](https://github.com/argoproj/argo)を選びました。

ArgoはKubernetes向けに設計されたワークフローエンジンです。長い間オープンソース製品として存在し、広範な注目と応用を受けています。

Argoはワークフローのためにいくつかのカスタムリソース定義（CRD）を抽象化しています。最も重要なものには、Workflow Template、Workflow、およびCron Workflowが含まれます。以下に、ArgoがTiPocketにどのように適合するかを示します：

- **Workflow Template**は、各テストタスクのために事前に定義されたテンプレートです。テスト実行時にパラメータを渡すことができます。
- **Workflow**は、異なる順序で複数のワークフローテンプレートをスケジュールし、実行されるタスクを形成します。Argoでは、パイプラインに条件、ループ、有向非巡回グラフ（DAG）を追加することもできます。
- **Cron Workflow**は、cronジョブのようにワークフローをスケジュールできます。長時間にわたってテストタスクを実行したいシナリオに最適です。

私たちが事前定義したbankテストのサンプルワークフローを以下に示します：

```yml
spec:
  entrypoint: call-tipocket-bank
  arguments:
    parameters:
      - name: ns
        value: tipocket-bank
            - name: nemesis
        value: random_kill,kill_pd_leader_5min,partition_one,subcritical_skews,big_skews,shuffle-leader-scheduler,shuffle-region-scheduler,random-merge-scheduler
  templates:
    - name: call-tipocket-bank
      steps:
        - - name: call-wait-cluster
            templateRef:
              name: wait-cluster
              template: wait-cluster
        - - name: call-tipocket-bank
            templateRef:
              name: tipocket-bank
              template: tipocket-bank
```

この例では、ワークフローテンプレートとnemesisパラメータを使用して、注入する特定の障害を定義しています。このテンプレートを再利用して、さまざまなテストケースに適した複数のワークフローを定義できます。これにより、フローにさらにカスタマイズされた障害注入を追加できます。

[TiPocketの](https://github.com/pingcap/tipocket/tree/master/argo/workflow)サンプルワークフローとテンプレートに加えて、この設計では独自の障害注入フローを追加することもできます。コード化可能なワークフローを使用して複雑なロジックを処理することで、Argoは開発者に優しく、私たちのシナリオに理想的な選択肢となります。

これで、カオス実験は自動的に実行されています。しかし、結果が期待通りでない場合、どのように問題を特定すればよいでしょうか？TiDBはさまざまな監視情報を保存しており、TiPocketで可観測性を実現するためにはログ収集が不可欠です。

## 結果の可視化: Loki

クラウドネイティブシステムにおいて、可観測性は非常に重要です。一般的に、**メトリクス**、**ロギング**、**トレーシング**を通じて可観測性を実現できます。TiPocketの主なテストケースはTiDBクラスターを評価するため、メトリクスとログが問題特定のデフォルトソースとなります。

Kubernetes上では、Prometheusがメトリクスのデファクトスタンダードです。しかし、ログ収集に関しては共通の方法がありません。[Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch)、[Fluent Bit](https://fluentbit.io/)、[Kibana](https://www.elastic.co/kibana)などのソリューションは優れていますが、システムリソースの競合や高いメンテナンスコストを引き起こす可能性があります。私たちは[Grafana](https://grafana.com/)の[Loki](https://github.com/grafana/loki)を使用することにしました。これはPrometheusのようなログ集約システムです。

PrometheusはTiDBの監視情報を処理します。PrometheusとLokiは類似したラベルシステムを持っているため、Prometheusの監視指標と対応するPodログを簡単に組み合わせることができ、同様のクエリ言語を使用できます。GrafanaはLokiダッシュボードもサポートしており、Grafanaを使用して監視指標とログを同時に表示できます。GrafanaはTiDBの組み込み監視コンポーネントであり、Lokiはこれを再利用できます。

## すべてを統合 - TiPocket

これですべての準備が整いました。以下はTiPocketの簡略化された図です:

![TiPocket Architecture](/img/blog/tipocket-architecture.png)

ご覧の通り、Argoワークフローがすべてのカオス実験とテストケースを管理します。一般的に、完全なテストサイクルには以下のステップが含まれます:

1. ArgoがCron Workflowを作成し、テスト対象のクラスター、注入する障害、テストケース、タスクの期間を定義します。必要に応じて、Cron Workflowはリアルタイムでケースログを確認することも可能です。

![Argo Workflow](/img/blog/argo-workflow.png)

1. 指定された時間に、ワークフロー内で別個のTiPocketスレッドが起動され、Cron Workflowがトリガーされます。TiPocketはTiDB-Operatorにテスト対象クラスターの定義を送信します。TiDB-OperatorはターゲットTiDBクラスターを作成します。同時に、Lokiが関連ログを収集します。
2. Chaos Meshがクラスターに障害を注入します。
3. 前述のテストケースを使用して、ユーザーがシステムの健全性を検証します。テストケースの失敗はArgoのワークフロー失敗を引き起こし、Alertmanagerが指定されたSlackチャンネルに結果を送信します。テストケースが正常に完了すると、クラスターがクリアされ、Argoは次のテストまで待機状態になります。

![Alert in Slack](/img/blog/alert_message.png)

これがTiPocketの完全なワークフローです。

## 参加してください

[Chaos Mesh](https://github.com/pingcap/chaos-mesh)と[TiPocket](https://github.com/pingcap/tipocket)はともに活発に開発が進められています。私たちはChaos Meshを[CNCF](https://github.com/cncf/toc/pull/367)に寄贈し、より多くのコミュニティメンバーが参加して完全なカオスエンジニアリングエコシステムを構築することを期待しています。興味があれば、[ウェブサイト](https://chaos-mesh.org/)をチェックするか、[CNCF Slack](https://slack.cncf.io/)の#project-chaos-meshに参加してください。