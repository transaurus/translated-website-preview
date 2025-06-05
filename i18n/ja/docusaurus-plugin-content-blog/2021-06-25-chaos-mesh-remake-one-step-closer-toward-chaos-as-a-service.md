---
slug: /chaos-mesh-remake-one-step-closer-towards-chaos-as-a-service
title: 'Chaos Mesh Remake: One Step Closer toward Chaos as a Service'
authors:
  - xiangwang
  - changyu
image: /img/blog/chaos-engineering-tools-as-a-service.jpeg
tags: [Chaos Mesh, Chaos Engineering]
---

![カオスエンジニアリングツール](/img/blog/chaos-engineering-tools-as-a-service.jpeg)

[Chaos Mesh](https://chaos-mesh.org/)は、Kubernetes環境でカオスをオーケストレーションするクラウドネイティブなカオスエンジニアリングプラットフォームです。Chaos Meshを使用すると、Pod、ネットワーク、ファイルシステム、さらにはカーネルにさまざまな種類の障害を注入することで、Kubernetes上でシステムの回復力と堅牢性をテストできます。

<!--truncate-->

オープンソース化され、Cloud Native Computing Foundation（CNCF）のサンドボックスプロジェクトとして受け入れられて以来、Chaos Meshは世界中のコントリビューターを惹きつけ、ユーザーがシステムをテストするのを支援してきました。しかし、まだ改善の余地が多くあります：

- ユーザビリティの向上が必要です。一部の機能は使用が複雑です。たとえば、カオス実験を適用する際、実験が開始されたかどうかを手動で確認する必要があることがよくあります。
- 主にKubernetes環境向けです。Chaos Meshは複数のKubernetesクラスターを管理できないため、各KubernetesクラスターごとにChaos Meshをデプロイする必要があります。[chaosd](https://github.com/chaos-mesh/chaosd)は物理マシン上でカオス実験を実行することをサポートしていますが、機能はかなり限定されており、コマンドラインの使用はユーザーフレンドリーではありません。
- プラグインを許可していません。カスタマイズされたカオス実験を適用するには、ソースコードを変更する必要があります。さらに、Chaos MeshはGolangのみをサポートしています。

確かに、Chaos Meshは一流のカオスエンジニアリングプラットフォームですが、Chaos as a Service（CaaS）を提供するにはまだ長い道のりがあります。そのため、[TiDB Hackathon 2020](https://pingcap.com/community-activity/tidb-hackathon-2020/)では、**Chaos Meshのアーキテクチャを変更し、CaaSに一歩近づけました**。

この記事では、CaaSとは何か、Chaos Meshでそれをどのように実現したか、そして私たちの計画と学んだことについて話します。この経験が、あなた自身のカオスエンジニアリングシステムを構築するのに役立つことを願っています。

## Chaos as a Service（CaaS）とは何か？

Gremlinの共同創設者であるMatt Fornaciariが[述べているように](https://jaxenter.com/chaos-engineering-service-144113.html)、CaaSとは「直感的なUI、カスタマーサポート、すぐに使える統合、そして数分で実験を開始するために必要なすべてを提供する」ことを意味します。

私たちの視点では、CaaSは以下を提供するべきです：

- 管理のための統一されたコンソール。設定を編集し、カオス実験を作成できます。
- 実験ステータスを確認するための可視化されたメトリクス。
- 実験を一時停止またはアーカイブする操作。
- 簡単なインタラクション。オブジェクトをドラッグアンドドロップして実験をオーケストレーションできます。

[NetEase Fuxi AI Lab](https://pingcap.com/blog/how-a-top-game-company-uses-chaos-engineering-to-improve-testing)やFreeWheelなどの企業は、すでにChaos Meshを自社のニーズに合わせて適応させ、CaaSのモデルケースとしています。

## CaaSに向けたChaos Meshの開発

CaaSの理解に基づいて、Hackathon期間中にChaos Meshのアーキテクチャを洗練させ、さまざまなシステムのサポートの改善と観測性の向上を図りました。私たちのコードは[wuntun/chaos-mesh](https://github.com/wuntun/chaos-mesh/tree/caas)と[wuntun/chaosd](https://github.com/wuntun/chaosd/tree/caas)で確認できます。

### Chaos Dashboardのリファクタリング

現在のChaos Meshのアーキテクチャは、個々のKubernetesクラスターに適しています。ウェブUIであるChaos Dashboardは、指定されたKubernetes環境にバインドされています：

![Chaos Meshアーキテクチャ](/img/blog/chaos-mesh-remake-architecture.jpeg)

このリファクタリングでは、**Chaos Dashboardが複数のKubernetesクラスターを管理できるように、Chaos Dashboardをメインアーキテクチャから分離しました**。これにより、Kubernetesクラスターの外にChaos Dashboardをデプロイした場合、ウェブUIを通じてクラスターを追加できます。クラスター内にデプロイした場合、環境変数を通じて自動的にクラスター情報を取得します。

Chaos DashboardにChaos Mesh（技術的にはKubernetes設定）を登録するか、`chaos-controller-manager`に設定を通じてChaos Dashboardに報告させることができます。Chaos Dashboardと`chaos-controller-manager`はCustomResourceDefinitions（CRD）を介して連携します。`chaos-controller-manager`がChaos MeshのCRDイベントを検知すると、`chaos-daemon`を呼び出して関連するカオス実験を実行します。したがって、Chaos DashboardはCRDを操作することで実験を管理できます。

### chaosdのリファクタリング

chaosdは物理マシン上でカオス実験を実行するためのツールキットです。以前はコマンドラインツールとしてのみ機能し、機能も限られていました。

![chaosd、カオスエンジニアリング用コマンドラインツール](/img/blog/chaosd-chaos-engineering-command-line-tool.jpeg)

リファクタリングでは、**chaosdがRESTful APIをサポートし、CRD形式のJSONまたはYAMLファイルを解析してカオス実験を設定できるようにサービスを強化しました**。

現在、chaosdは設定を通じてChaos Dashboardに自己登録し、定期的にハートビートを送信できます。ハートビート信号により、Chaos Dashboardはchaosdノードの状態を管理できます。Web UIからもchaosdノードをChaos Dashboardに追加できます。

さらに、**chaosdは指定された時間にカオス実験をスケジュールし、実験ライフサイクルを管理できるようになりました。これにより、Kubernetesと物理マシンでのユーザーエクスペリエンスが統一されます**。

新しいChaos Dashboardとchaosdにより、最適化されたChaos Meshのアーキテクチャは以下の通りです：

![Chaos Meshの最適化されたアーキテクチャ](/img/blog/chaos-mesh-optimized-architecture.jpeg)

### 可観測性の向上

もう一つの改善点は可観測性、つまり実験が正常に実行されたかどうかを判断する方法です。

改善前は、実験メトリクスを手動で確認する必要がありました。例えば、Podに[StressChaos](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/chaos_experiments/stresschaos)を注入した場合、Pod内に入って`stress-ng`プロセスが存在するか確認し、`top`コマンドでCPUとメモリ使用率を確認する必要がありました。これらのメトリクスからStressChaos実験が正常に作成されたか判断していました。

このプロセスを合理化するため、`node_exporter`を`chaos-daemon`とchaosdに統合してノードメトリクスを収集するようにしました。また、Kubernetesクラスター内に`kube-state-metrics`をデプロイし、cadvisorと組み合わせてKubernetesメトリクスを収集します。収集されたメトリクスはPrometheusとGrafanaによって保存・可視化され、実験ステータスを簡単に確認できる方法を提供します。

#### さらなる改善が必要な点

全体的に、メトリクスの目的は以下を支援することです：

- カオスが注入されたことを確認する。
- サービスへのカオスの影響を観察し、定期的に分析する。
- 異常なカオスイベントに対応する。

これらの目標を達成するため、システムは実験データメトリクス、通常のメトリクス、および実験イベントを監視する必要があります。Chaos Meshにはまだ改善の余地があります：

- 実験データメトリクス：注入されたネットワーク遅延の正確な持続時間や、シミュレートされたワークロードの具体的な負荷など。
- 実験イベント：実験の作成、削除、実行に関するKubernetesイベント。

[Litmus](https://github.com/litmuschaos/chaos-exporter#example-metrics)のメトリクス例は良い参考になります。

## Chaos Meshへのその他の提案

Hackathonの時間制限により、すべての計画を完了できませんでした。以下は、Chaos Meshコミュニティが将来検討するための提案の一部です。

### オーケストレーション

カオスエンジニアリングの閉ループには、4つのステップが含まれます：カオスの探索、システムの欠陥の発見、根本原因の分析、改善のためのフィードバック送信。

![カオスエンジニアリングの閉ループ](/img/blog/closed-loop-of-chaos-engineering.jpeg)

しかし、**現在のオープンソースのカオスエンジニアリングツールのほとんどは探索にのみ焦点を当てており、実用的なフィードバックを提供していません。** 改善された可観測性コンポーネントを基に、カオス実験をリアルタイムで監視し、実験結果を比較・分析することが可能です。

これらの結果を活用することで、オーケストレーションという重要なコンポーネントを追加することで閉ループを実現できます。Chaos Meshコミュニティは既に[Workflow](https://github.com/chaos-mesh/rfcs/pull/10/files)機能を提案しており、これによりカオス実験のオーケストレーションやコールバックを容易に行ったり、Chaos Meshを他のシステムと簡単に統合したりできます。CI/CDフェーズやカナリアリリース後にカオス実験を実行することも可能です。

**可観測性とオーケストレーションを組み合わせることで、カオスエンジニアリングの閉じたフィードバックループが実現します。** 例えば、Podに対して100ミリ秒のネットワーク遅延テストを実施する場合、可観測性コンポーネントで遅延の変化を観察し、オーケストレーションに基づいてPromQLやその他のDSLを使用してPodサービスが依然として利用可能かどうかを確認できます。サービスが利用不能であれば、遅延が>=100ミリ秒の場合にサービスが利用不能になるという結論を導き出すことができます。

しかし、100ミリ秒はサービスの閾値ではありません。サービスが処理できる最大の遅延時間を知る必要があります。カオス実験の値をオーケストレーションすることで、サービスレベル目標を満たすために確保しなければならない閾値がわかります。また、さまざまなネットワーク条件下でのサービスパフォーマンスと、それが期待通りかどうかも明らかになります。

### データ形式

Chaos MeshはCRDを使用してカオスオブジェクトを定義します。CRDをJSONファイルに変換できれば、コンポーネント間の通信を実現できます。

データ形式に関して、chaosdはJSON形式のCRDデータを消費して登録するだけです。カオスツールがCRDデータを消費して自身を登録できれば、さまざまなシナリオでカオス実験を実行できます。

### プラグイン

Chaos Meshのプラグインサポートは限定的です。[新しいChaosを追加する](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/development_guides/develop_a_new_chaos/)には、Kubernetes APIにCRDを登録する必要があります。これには2つの問題があります:

- Golang（Chaos Meshが記述されている言語）を使用してプラグインを開発する必要があります。
- 拡張コードをChaos Meshプロジェクトにマージする必要があります。Chaos MeshにはBerkeley Packet Filter（BPF）のようなセキュリティメカニズムがないため、プラグインコードのマージは追加のリスクをもたらす可能性があります。

完全なプラグインサポートを実現するには、新しいプラグイン追加方法を探る必要があります。Chaos Meshは本質的にCRDに基づいてカオス実験を実行するため、カオス実験にはCRDの生成、監視、削除のみが必要です。この点に関して、試す価値のあるいくつかのアイデアがあります:

- CRDを管理するコントローラーまたはオペレーターを開発する。
- CRDイベントを一元的に処理し、HTTPコールバック経由でCRDを操作する。この方法ではHTTP APIのみを使用し、Golangは不要です。例として[Whitebox Controller](https://github.com/summerwind/whitebox-controller)を参照。
- WebAssembly（Wasm）を使用する。カオス実験ロジックを呼び出す必要がある場合、Wasmプログラムを呼び出すだけです。
- SQLを使用してカオス実験のステータスをクエリする。Chaos MeshはCRDに基づいているため、SQLを使用してKubernetesを操作できます。例として[Presto connector](https://github.com/xuxinkun/kubesql)や[osquery extension](https://github.com/aquasecurity/kube-query)があります。
- SDKベースの拡張を使用する。例：[Chaos Toolkit](https://docs.chaostoolkit.org/reference/api/experiment/)。

### 他のカオスツールとの統合

現実のシステムでは、単一のカオスエンジニアリングツールですべての可能なユースケースを網羅することは困難です。そのため、他のカオスツールと統合することで、カオスエンジニアリングのエコシステムをより強力にすることができます。

市場には数多くのカオスエンジニアリングツールが存在します。Litmusの[Kubernetes実装](https://github.com/litmuschaos/litmus-go/tree/2.14.1/chaoslib/powerfulseal)は[PowerfulSeal](https://github.com/powerfulseal/powerfulseal)を基にしており、その[コンテナ実装](https://github.com/litmuschaos/litmus-go/tree/2.14.1/chaoslib/pumba)は[Pumba](https://github.com/alexei-led/pumba)を基にしています。[Kraken](https://github.com/cloud-bulldozer/kraken)はKubernetesに焦点を当て、[AWSSSMChaosRunner](https://github.com/amzn/awsssmchaosrunner)はAWSに特化し、[Toxiproxy](https://github.com/shopify/toxiproxy)はTCPを対象としています。また、[Envoy](https://docs.google.com/presentation/d/1gMlmXqH6ufnb8eNO10WqVjqrPRGAO5-1S1zjcGo1Zr4/edit#slide=id.g58453c664c_2_75)やIstioを基にした統合プロジェクトも存在します。

様々なカオスツールを管理するためには、[Chaos Hub](https://hub.litmuschaos.io/)のような統一されたパターンが必要となるかもしれません。

## コミュニティからの声

ここでは、中国の主要なサイバーセキュリティ企業であり、Chaos Meshのユーザーでもある組織が、どのようにChaos Meshを自社のニーズに合わせて適応させているかを共有します。彼らの適応は、物理ノード、コンテナ、アプリケーションの3つの側面から行われています。

### 物理ノード

- 物理サーバー上でスクリプトを実行するサポート。CRDでスクリプトディレクトリを設定し、`chaos-daemon`を使用してスクリプトを実行できます。
- カスタマイズされたスクリプトを使用して、再起動、シャットダウン、カーネルパニックをシミュレート。
- カスタマイズされたスクリプトを使用してノードのNICをシャットダウン。
- sysbenchを使用して頻繁なコンテキストスイッチングを作成し、「ノイジーネイバー」効果をシミュレート。
- BPFの`seccomp`を使用してコンテナのシステムコールをインターセプト。これはPIDの受け渡しとフィルタリングによって実現されます。

### コンテナ

- Deploymentのレプリカ数をランダムに変更し、アプリケーションのトラフィックに異常がないかテスト。
- CRDオブジェクトに基づいて埋め込み：カオスCRDにIngressオブジェクトを埋め込み、インターフェースの速度制限をシミュレート。
- CRDオブジェクトに基づいて埋め込み：カオスCRDにCiliumネットワークポリシーオブジェクトを埋め込み、変動するネットワーク条件をシミュレート。

### アプリケーション

- カスタマイズされたジョブの実行をサポート。現在、Chaos Meshは`chaos-daemon`を使用してカオスを注入しますが、これはスケジューリングの公平性とアフィニティを保証しません。この問題に対処するため、`chaos-controller-manager`を使用して異なるCRDに対して直接ジョブを作成できます。
- カスタマイズされたジョブで[Newman](https://github.com/postmanlabs/newman)を実行し、HTTPパラメータをランダムに変更するサポート。これは、ユーザーが異常な動作を行ったときに発生するHTTPインターフェースに対するカオス実験を実装するためです。

## まとめ

従来の障害テストは、システム内の脆弱性が予想される特定のポイントを対象とします。これはしばしばアサーションです：特定の条件が特定の結果を生み出します。

**カオスエンジニアリングは、「未知の未知」を発見するのにさらに強力です。** より広範な領域で探索することで、カオスエンジニアリングはテスト対象のシステムに関する知識を深め、新たな情報を掘り起こします。

まとめると、これらはカオスエンジニアリングとChaos Meshに関する私たちの個人的な考えと実践の一部です。私たちのハッカソンプロジェクトはまだ本番環境に適していませんが、CaaSに光を当て、Chaos Meshの有望なロードマップを草案することを願っています。Chaos as a Serviceの構築に興味がある方は、[Slackに参加](https://slack.cncf.io/) (#project-chaos-mesh)してください！