---
slug: /chaos-engineering-breaking-things-intentionally
title: Chaos Engineering - Breaking things Intentionally
authors: manishdangi
image: /img/blog/chaos-engineering2.png
tags: [Chaos Engineering, Chaos Mesh, Open Source]
---

![Chaos-Engineering-Breaking-things-Intentionally](/img/blog/chaos-engineering2.png)

「必要は発明の母」というように、Netflixは単なるオンラインメディアストリーミングプラットフォームではありません。Netflixは自らの必要性からChaos Engineeringを生み出しました。

<!--truncate-->

2008年、Netflixは[大規模なデータベース障害を経験しました](https://about.netflix.com/en/news/completing-the-netflix-cloud-migration)。3日間DVDを配送できなくなりました。この出来事が、Netflixのエンジニアにモノリシックなアーキテクチャから分散型のクラウドベースアーキテクチャへの移行を考えさせるきっかけとなりました。

Netflixの新しい分散型アーキテクチャは数百のマイクロサービスで構成されていました。分散型アーキテクチャへの移行は単一障害点の問題を解決しましたが、システムの信頼性と耐障害性をさらに高める必要がある多くの複雑さを生み出しました。この時点で、Netflixのエンジニアは顧客サービスに影響を与えることなくシステムの耐障害性をテストする革新的なアイデアを思いつきました。

彼らは[Chaos Monkey](https://github.com/Netflix/chaosmonkey)を作成しました。これはさまざまな場所でランダムな障害を引き起こすツールです。Chaos Monkeyの開発とともに、新しい分野が生まれました。それがChaos Engineeringです。

「Chaos Engineeringとは、システムが生産環境における乱れた状況に耐える能力に対する信頼を構築するために、システム上で実験を行う学問である」 - [Principle of Chaos](https://principlesofchaos.org/)

Chaos Engineeringは、経験的な探求の手法を適用してシステムの挙動を学ぶアプローチです。科学者が物理的・社会的現象を研究するために実験を行うのと同じように、Chaos Engineeringは実験を使って特定のシステム（システムの信頼性、安定性、予期せぬまたは不安定な状況での生存能力）について学びます。

大規模な分散システムでは、アプリケーション障害、インフラ障害、依存関係の障害、ネットワーク障害など、さまざまな要因で障害が発生する可能性があります。これらの障害は、統合テストや単体テストなどの従来の方法ではすべてをカバーできません。そのためChaos Engineeringが必要となります：

- システムの回復力を向上させるため
- システムの隠れた脅威や脆弱性を明らかにするため
- 本番環境で障害を引き起こす前にシステムの弱点を特定するため

多くの人は、自分たちはNetflixや他のテックジャイアントと比べて規模が小さく、そのような規模のデータベースやシステムを持っていないと考えています。

おそらくその通りでしょう。しかし時が経つにつれ、Chaos EngineeringはNetflixのようなデジタル企業に限定されないほど進化しました。システムの一貫したパフォーマンスと常時可用性を確保するために、さまざまな業界の企業がChaos実験を実施しています。

## Chaos Mesh

:::note

2022-10-24: https://www.oreilly.com/online-learning/leveraging-katacoda-technology.html および [#356](https://github.com/chaos-mesh/website/pull/356) により、インタラクティブチュートリアルは一時的に利用できません。

:::

[TiDB](https://pingcap.com/products/tidb)の回復力と信頼性をテストするために、[PingCAP](https://pingcap.com/)のエンジニアは[Chaos Mesh](https://chaos-mesh.org/)という素晴らしいChaosテストツールを開発しました。これはKubernetes環境でChaosをオーケストレーションするクラウドネイティブなChaos Engineeringプラットフォームです。Chaos Meshは分散システムの可能性のある障害（Pod、ネットワーク、システムI/O、カーネルなど）を考慮しています。

Chaos Meshは多くの障害注入方法を提供します：

- **clock-skew:** 時刻のずれをシミュレート
- **container-kill:** コンテナの強制終了をシミュレート
- **cpu-burn:** CPU負荷をシミュレート
- **io-attribution-override:** ファイル例外をシミュレート
- **io-fault:** ファイルシステムI/Oエラーをシミュレート
- **io-latency:** ファイルシステムI/O遅延をシミュレート
- **kernel-injection:** カーネル障害をシミュレート
- **memory-burn:** メモリ負荷をシミュレート
- **network-corrupt:** ネットワークパケット破損をシミュレート
- **network-duplication:** ネットワークパケット重複をシミュレート
- **network-latency:** ネットワーク遅延をシミュレート
- **network-loss:** ネットワーク損失をシミュレート
- **network-partition:** ネットワーク分断をシミュレート
- **pod-failure:** Kubernetes Podの継続的な利用不可をシミュレート
- **pod-kill:** Kubernetes Podの強制終了をシミュレート

Chaos Meshは主に、すべてのカオステストを迅速かつ簡単に行えるシンプルさに重点を置いており、利用者誰もが理解しやすい設計となっています。

最近リリースされた[バージョン1.0](https://chaos-mesh.org/blog/chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-easier/)では、Chaos Dashboardが一般提供され、カオス実験の複雑さが簡素化されました。数回のマウスクリックで、カオス実験の範囲を定義し、カオス注入のタイプを指定し、スケジューリングルールを定義し、実験結果を観察することができます。これらすべてがChaos Meshのダッシュボード上で可能です。

ブラウザ上でChaos Meshを試したい場合は、`Katakodaインタラクティブチュートリアル`をチェックしてください。Chaos Meshを実際にデプロイすることなく、手を動かして体験できます。設計原則やChaos Meshの動作原理を理解したい場合は、プロジェクトメンテナーの[Cwen Yin](https://www.linkedin.com/in/cwen-yin-81985318b/)による[このブログ記事](https://chaos-mesh.org/blog/chaos_mesh_your_chaos_engineering_solution)をお読みください。

## コミュニティに参加しよう

カオスエンジニアリングやChaos Meshに興味がある方は、ぜひChaos Meshコミュニティに参加してください。Chaos Meshコミュニティの一員として、このコミュニティは素晴らしい場所だと自信を持って言えます。プロジェクトのメンテナーは積極的にコミュニケーションを取り、プロジェクトやコミュニティの改善に向けて皆さんの意見や提案に耳を傾けています。

Chaos Meshに参加し、さらに学びたい方は、[CNCF Slackワークスペース](https://slack.cncf.io/)の#project-chaos-meshチャンネルをご利用ください。