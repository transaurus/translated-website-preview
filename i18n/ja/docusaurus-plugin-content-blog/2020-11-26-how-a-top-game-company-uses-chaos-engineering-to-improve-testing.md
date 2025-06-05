---
slug: /how-a-top-game-company-uses-chaos-engineering-to-improve-testing
title: 'How a Top Game Company Uses Chaos Engineering to Improve Testing'
authors: huizhang
image: /img/blog/fuxi-case-banner.jpg
tags: [Chaos Mesh, Chaos Engineering]
---

![How-a-Top-Game-Company-Uses-Chaos-Engineering-to-Improve-Testing](/img/blog/fuxi-case-banner.jpg)

NetEase Fuxi AI Labは中国初の専門的なゲームAI研究機関です。研究者はKubernetesベースのDanluプラットフォームを利用して、アルゴリズム開発、トレーニングとチューニング、オンライン公開を行っています。Kubernetesとの統合により、当社のプラットフォームは大幅に効率化されました。しかし、Kubernetesやマイクロサービス関連の問題により、プラットフォームの安定性を向上させるため、継続的なテストと改善を行っています。

<!--truncate-->

本記事では、当社が最も価値あるテストツールの1つである[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)について説明します。Chaos Meshはオープンソースのカオスエンジニアリングツールで、幅広い障害注入と優れた障害監視をDashboardを通じて提供します。

## Chaos Meshを選んだ理由

2018年にカオスエンジニアリングツールの探求を開始しました。私たちが求めたツールの要件は以下の通りです：

- クラウドネイティブ対応。Kubernetesは事実上のサービスオーケストレーションとスケジューリングの標準であり、アプリケーションランタイムは完全に標準化されています。K8s上で完全に動作するアプリケーションにとって、クラウドネイティブ対応は必須です。

- 十分な障害注入タイプ。ステートフルサービスにとって、ネットワーク障害シミュレーションは特に重要です。プラットフォームはPod、ネットワーク、I/Oなど異なるレベルでの障害をシミュレートできる必要があります。

- 優れた可観測性。障害が注入されたタイミングと回復可能なタイミングを把握することは、アプリケーションの異常を検知する上で極めて重要です。

- 活発なコミュニティサポート。十分にテストされ、継続的にメンテナンスされているオープンソースプロジェクトを利用したいと考えています。そのため、持続的かつ迅速なコミュニティサポートを重視しています。

- 既存アプリケーションへの侵入なし。ドメイン知識を必要としません。

- 評価と構築の基盤となる実際のユースケース。

2019年、Kubernetes向けカオスエンジニアリングプラットフォームであるChaos Meshがオープンソース化された時、私たちは探し求めていたツールを見つけました。当時はまだ初期段階でしたが、サポートする障害タイプの豊富さにすぐに魅了されました。これは他のカオスエンジニアリングツールに対する大きな優位性であり、ある程度までシステムで発見可能な問題の数を決定づけます。Chaos Meshがほぼ全ての面で私たちの期待に応えるものであることを即座に理解しました。

![Chaos Mesh architecture](/img/blog/chaos-mesh-architecture.png)

## Chaos Meshとの歩み

Chaos Meshはいくつかの重要なバグの発見に貢献しました。例えば、Danlu向けオープンソースメッセージキューソフトウェアである[rabbitMQ](https://www.rabbitmq.com/)のブレインスプリット問題を検出しました。[Wikipedia](https://en.wikipedia.org/wiki/Split-brain)によると、「ブレインスプリット状態は、スコープが重複する2つの別々のデータセットを維持することに起因するデータまたは可用性の不整合を示します」。rabbitMQクラスタでブレインスプリットエラーが発生すると、データ書き込みの競合やエラーが発生し、メッセージングサービスのデータ不整合などより深刻な問題を引き起こします。以下のアーキテクチャで示すように、ブレインスプリットが発生すると、コンシューマーが正常に機能せず、サーバー例外を報告し続けます。

![Architecture of a RabbitMQ cluster](/img/blog/architecture-of-a-rabbitmq-cluster.png)

Chaos Meshを使用することで、コンテナインスタンスクラウドに`pod-kill`障害を注入することで、この問題を安定的に再現することができました。

Chaos Meshはまた、起動失敗、クラッシュしたブローカークラスタの参加失敗、ハートビートタイムアウト、接続チャネルのシャットダウンなど、いくつかの他の問題も発見しました。時間の経過とともに、開発チームはこれらの問題を修正し、Danluプラットフォームの安定性を大幅に向上させました。

## 急速に成長するプロジェクト

Chaos Meshは常に更新され改善されています。私たちが最初に採用した時、安定版にすら達していませんでした。デバッグやログ収集ツールはなく、DashboardコンポーネントはTiDBにしか適用できませんでした。他のアプリケーションをテストする唯一の方法は、YAML設定ファイルを`kubectl apply`で実行することでした。

[Chaos Mesh 1.0](https://chaos-mesh.org/blog/chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-easier) では、これらの制限事項のほとんどが修正または改善されました。より細かく強力なカオスサポート、一般提供が開始されたChaos Dashboard、強化された可観測性、より正確なカオススコープ制御を提供しています。これらはすべて、オープンで協力的で活気あるコミュニティによって推進されています。

![Chaos Dashboardが一般提供開始](/img/blog/chaos-dashboard.gif)

## 今後の展望

Chaos Meshがどれほど成長し、どれほどの注目を集めているかは驚くべきものです。私たちも、Chaos Meshで達成した成果に満足しています。

しかし、カオスエンジニアリングは取り組むべき大きな領域です。将来的には、以下の機能が実現されることを期待しています：

- アトミックな障害注入
- カスタマイズされた障害タイプと実験対象を検証する標準化された方法を組み合わせた無人障害注入
- MySQL、Redis、Kafkaなどの一般的なコンポーネント向けの標準テストケース

これらの機能についてChaos Meshのメンテナーと議論したところ、これらはChaos Mesh 2.0のロードマップに含まれているとのことでした。

興味があれば、[Slack](https://slack.cncf.io/) (#project-chaos-mesh) または [GitHub](https://github.com/chaos-mesh/chaos-mesh) を通じてChaos Meshコミュニティに参加してください。