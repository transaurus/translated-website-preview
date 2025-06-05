---
slug: /celebrating-one-year-of-chaos-mesh-looking-back-and-ahead
title: 'Celebrating One Year of Chaos Mesh: Looking Back and Ahead'
authors: chaos-mesh
image: /img/blog/celebrating-one-year-of-chaos-mesh-looking-back-and-ahead.jpg
tags: [Chaos Mesh, Chaos Engineering]
---

![Chaos Meshの1周年を祝う：振り返りと今後の展望](/img/blog/celebrating-one-year-of-chaos-mesh-looking-back-and-ahead.jpg)

Chaos MeshがGitHubでオープンソース化されてから1年が経過しました。Chaos Meshは当初、単なる障害注入ツールとしてスタートしましたが、現在ではカオスエンジニアリングのエコシステム構築を目指して進化しています。同時に、Chaos Meshコミュニティもゼロから構築され、[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)がCNCFのSandboxプロジェクトとして参加することを支援しました。

<!--truncate-->

本記事では、Chaos Meshがこの1年間でどのように成長し、変化してきたかを共有し、今後の目標と計画についても議論します。

## プロジェクト：明確な目標を持って成長する

この1年間、Chaos Meshはコミュニティの共同作業により驚異的なスピードで成長しました。最初のバージョンから最近リリースされた[v1.1.0](https://github.com/chaos-mesh/chaos-mesh/releases/tag/v1.1.0)まで、Chaos Meshは機能性、使いやすさ、セキュリティの面で大幅に改善されています。

### 機能性

オープンソース化当初、Chaos MeshはPodChaos、NetworkChaos、IOChaosの3つの障害タイプのみをサポートしていました。わずか1年で、Chaos Meshはネットワーク、システムクロック、JVMアプリケーション、ファイルシステム、オペレーティングシステムなどへの多様な障害注入を実行できるようになりました。

![カオステスト](/img/blog/chaos-tests.png)

継続的な最適化により、Chaos Meshは柔軟なスケジューリングメカニズムを提供し、ユーザーが独自のカオス実験をより効果的に設計できるようになりました。これはカオスオーケストレーションの基礎を築きました。

同時に、多くのユーザーが[主要なクラウドプラットフォームでChaos Meshをテスト](https://github.com/chaos-mesh/chaos-mesh/issues/1182)し始めていることを嬉しく思います。これにはAmazon Web Services（AWS）、Google Kubernetes Engine（GKE）、Alibaba Cloud、Tencent Cloudなどが含まれます。私たちは継続的に互換性テストと適応を行い、[特定のクラウドプラットフォーム向けの障害注入](https://github.com/chaos-mesh/chaos-mesh/pull/1330)をサポートしています。

Kubernetesネイティブコンポーネントとノードレベルの障害をよりよくサポートするため、[Chaosd](https://github.com/chaos-mesh/chaosd)を開発しました。これは物理ノードレベルの障害注入を提供します。この機能は今後数ヶ月以内にリリースするために、広範なテストと改良を行っています。

### 使いやすさ

使いやすさは、Chaos Mesh開発の最初から指針の一つでした。Chaos Meshは単一のコマンドラインでデプロイできます。V1.0リリースでは、待望のChaos Dashboardが導入され、ユーザーがカオス実験をオーケストレーションするためのワンストップWebインターフェースが提供されました。カオス実験の範囲を定義し、障害注入のタイプを指定し、スケジューリングルールを定義し、カオス実験の結果を観察するすべてを、同じWebインターフェースで数回のクリックだけで行えます。

![Chaos Dashboard](/img/blog/chaos-dashboard1.png)

V1.0以前、多くのユーザーがIOChaos障害を注入する際にさまざまな設定問題に直面していました。徹底的な調査と議論の末、元のSideCar実装を放棄し、代わりにchaos-daemonを使用してターゲットPodに動的に侵入する方法を採用しました。これによりロジックが大幅に簡素化され、Chaos Meshで動的なI/O障害注入が可能になりました。ユーザーは追加の設定を気にすることなく、実験に集中できるようになりました。

### セキュリティ

Chaos Meshのセキュリティを向上させました。実験の範囲を制御するための包括的なセレクターセットを提供し、重要なアプリケーションを保護するための特定のネームスペースの設定をサポートしています。さらに、ネームスペース権限のサポートにより、ユーザーはカオス実験の「爆発半径」を特定のネームスペースに制限できます。

加えて、Chaos MeshはKubernetesのネイティブな権限メカニズムを直接再利用し、Chaos Dashboardでの検証をサポートしています。これにより、他のユーザーのエラーによるカオス実験の失敗や制御不能を防ぐことができます。

## クラウドネイティブエコシステム：統合と協業

2020年7月、Chaos Meshは[CNCFサンドボックスプロジェクトとして承認されました](https://chaos-mesh.org/blog/chaos-mesh-join-cncf-sandbox-project)。これはChaos Meshがクラウドネイティブコミュニティから一定の評価を得たことを示しています。同時に、Chaos Meshには明確な使命が与えられました：クラウドネイティブ領域におけるカオスエンジニアリングの応用を推進し、他のクラウドネイティブプロジェクトと協力して共に成長することです。

### Grafana

カオス実験の可観測性をさらに向上させるため、Chaos Mesh専用の[Grafanaプラグイン](https://github.com/chaos-mesh/chaos-mesh-datasource)を開発しました。これにより、ユーザーはアプリケーション監視パネル上でリアルタイムのカオス実験情報を直接表示できるようになります。これで、アプリケーションの稼働状況と現在のカオス実験情報を同時に観察できます。

### GitHub Action

開発段階でもカオス実験を実行できるようにするため、[chaos-mesh-action](https://github.com/chaos-mesh/chaos-mesh-action)プロジェクトを開発しました。これにより、Chaos MeshをGitHub Actionsのワークフロー内で実行できるようになり、日常的なシステム開発とテストに簡単に統合できます。

### TiPocket

[TiPocket](https://github.com/pingcap/tipocket)は、Chaos MeshとKubernetes向けワークフローエンジンであるArgoを統合した自動テストプラットフォームです。TiPocketは、分散型データベースであるTiDBのための完全自動化されたカオスエンジニアリングテストループとして設計されています。カオス実験を実施する際には、アプリケーションのデプロイ、ワークロードの実行、例外の注入、ビジネスチェックなど多くのステップがあります。これらのステップを完全に自動化するため、ArgoがTiPocketに統合されました。Chaos Meshは豊富な障害注入機能を提供し、Argoは柔軟なオーケストレーションとスケジューリングを提供します。

![TiPocket](/img/blog/tipocket.png)

## コミュニティ：ゼロからの構築

Chaos Meshはコミュニティ主導のプロジェクトであり、活発で友好的でオープンなコミュニティなしでは進展できません。オープンソース化以来、Chaos Meshはカオスエンジニアリング分野で最も注目を集めるオープンソースプロジェクトの一つとなりました。1年足らずでGitHubで3,000以上のスターを獲得し、70人以上のコントリビューターが参加しています。採用企業にはTencent Cloud、XPeng Motors、Dailymotion、NetEase Fuxi Lab、JuiceFS、APISIX、Meituanなどが含まれます。振り返ると、Chaos Meshコミュニティはゼロから構築され、透明性が高く、オープンで友好的で自律的なオープンソースコミュニティの基盤が築かれました。

### CNCFファミリーの一員に

Chaos MeshのDNAには最初からクラウドネイティブが組み込まれています。CNCFへの参加は自然な選択であり、Chaos Meshがベンダー中立でオープンで透明性の高いオープンソースコミュニティになるための重要な一歩でした。クラウドネイティブエコシステム内での統合に加え、CNCFへの参加によりChaos Meshは以下を得ました：

- より多くのコミュニティとプロジェクトの露出。他のプロジェクトとの協業や、Kubernetes MeetupやKubeConなどの様々なクラウドネイティブコミュニティ活動を通じて、コミュニティと交流する素晴らしい機会を得ました。コミュニティが生み出す高品質なコンテンツがChaos Meshの普及に果たした積極的で遠大な役割にも驚かされています。

- より完全でオープンなコミュニティフレームワーク。CNCFはオープンソースコミュニティ運営のための成熟したフレームワークを提供しています。CNCFの指導のもと、私たちは行動規範、コントリビューションガイド、ロードマップを含む基本的なコミュニティフレームワークを確立しました。また、CNCFのSlack内に独自のチャンネル#project-chaos-meshも作成しました。

### 友好的で支援的なコミュニティ

:::note

2022-10-24: https://www.oreilly.com/online-learning/leveraging-katacoda-technology.html と [#356](https://github.com/chaos-mesh/website/pull/356) により、インタラクティブチュートリアルは一時的に利用できません。

:::

オープンソースコミュニティの質は、採用者やコントリビューターが長期的に参加し続けるかどうかを決定します。この点に関して、私たちは以下の取り組みを進めてきました：

- ドキュメントの充実と構造の最適化を継続的に進めています。これまでに、[ユーザーガイド](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/user_guides/installation)や[開発者ガイド](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/development_guides/development_overview)、[クイックスタートガイド](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/get_started/get_started_on_kind)、[ユースケース](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/use_cases/multi_data_centers)、[コントリビューションガイド](https://github.com/chaos-mesh/chaos-mesh/blob/master/CONTRIBUTING.md)など、さまざまな対象者向けの完全なドキュメントセットを開発しました。これらはリリースごとに更新されています。

- コミュニティと協力して、ブログ記事、チュートリアル、ユースケース、およびカオスエンジニアリングの実践を公開しています。これまでに、Chaos Mesh関連の記事を26本制作しました。その中には、O'ReillyのKatakodaサイトに公開された`インタラクティブチュートリアル`も含まれています。これらの資料はドキュメントを補完するのに役立っています。

- コミュニティミーティング、ウェビナー、ミートアップで生成された動画やチュートリアルを再利用し、拡散しています。コミュニティからのフィードバックや質問を重視し、対応しています。

## 今後の展望

Googleの最近のグローバルな障害は、システムの信頼性の重要性を改めて思い起こさせ、カオスエンジニアリングの重要性を浮き彫りにしました。CNCF TOC議長のLiz Riceは、[2021年に注目すべき5つの技術](https://twitter.com/CloudNativeFdn/status/1329863326428499971)を共有し、そのトップにカオスエンジニアリングを挙げています。私たちは、カオスエンジニアリングが近い将来に新たな段階に入ると大胆に予測しています。Chaos Mesh 2.0は現在積極的に開発中であり、より柔軟なカオスシナリオの定義と管理をサポートする組み込みのワークフローエンジン、アプリケーション状態チェックメカニズム、より詳細な実験レポートなど、コミュニティの要望が含まれています。プロジェクトの[ロードマップ](https://github.com/chaos-mesh/chaos-mesh/blob/master/ROADMAP.md)を通じて進捗を追跡してください。

## 最後に

Chaos Meshはこの1年で大きく成長しましたが、まだ若く、目標に向けて船出したばかりです。その間、皆さんに参加を呼びかけ、Chaos Engineeringのシステムエコシステムを一緒に構築する手助けをしてほしいと考えています！

Chaos Meshに興味があり、改善に協力したい方は、[Slackチャンネル](https://slack.cncf.io/)に参加するか、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやイシューを投稿してください。