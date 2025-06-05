---
slug: /chaos-mesh-qa-at-kubecon-eu-2022
title: 'Chaos Mesh Q&A at KUBECON EU 2022'
authors: chaos-mesh
image: /img/blog/chaos-mesh-q&a.jpeg
tags: [Chaos Mesh, Chaos Engineering, KubeCon, CloudNativeCon]
---

![Chaos Mesh Q&A](/img/blog/chaos-mesh-q&a.jpeg)

KubeCon EU 2022において、[Chaos Mesh](https://chaos-mesh.org/)チームは「Make Cloud Native Chaos Engineering Easier - Deep Dive into Chaos Mesh」と「office hours session」の2つのアクティビティを主催しました。私たちは皆さんと共に過ごした時間に心から感謝し、非常に楽しむことができました。お互いに知識を共有し、理解を深め、多くのことを掘り下げて議論しました。

<!--truncate-->

プレゼンテーションでは、Chaos Meshの概要を簡単に説明した後、Chaos Meshの実装方法と実際の活用事例について詳しく解説し、チームが行っているカオスエンジニアリングに関する最新の探求とChaos Meshの開発計画を共有しました。

Office Hourでは、Chaos Meshプロジェクトとその最新の進捗状況を紹介し、参加者からのオンライン質問に回答しました。

私たちをサポートしてくださった皆様に心から感謝申し上げます！Office Hourでは素晴らしい質問を多数いただき、フォローアップQ&Aを実施することにしました。

## 質問への回答

**Q: Chaos MeshはWindows/Linuxハイブリッドクラスターで動作しますか？**

**A:** 現在、Chaos MeshはLinux環境でのみ動作します。ただし、Windowsへの機能移植に取り組んでくださっているコントリビューターの方がいます：[github.com/chaos-mesh/chaos-mesh/issues/2956](https://github.com/chaos-mesh/chaos-mesh/issues/2956)

**Q: IstioやLinkerdも障害注入をサポートしていますが、Chaos Meshとの違いは何ですか？Chaos MeshはIOChaosやTimeChaosなどより豊富なカオス注入を提供していますが、LinkerdやIstioの注入はネットワーク層に焦点を当てていると認識しています。**

**A:** その通りです！サービスメッシュフレームワークはRPC/ネットワーク層での障害発生の可能性があります。Chaos Meshでは、stresschaos、pod kill、DNSChaos、IOChaosなど、より多様なタイプのカオスを注入できます（先ほど言及したものに加えて）。さらに、JVM、GCP、Azureなどへの対応も提供しています...

**Q: Chaos Meshの一部として、カオス実験を開始する前に事前初期化スクリプトを実行できますか？**

**A:** はい可能です！Chaos Meshの統合Workflowエンジンを使用して、カスタムスクリプトと様々なカオス実験をまとめて管理できます。詳細は[ワークフローのtaskフィールド](https://chaos-mesh.org/docs/next/create-chaos-mesh-workflow/#task-field-description)のドキュメントをご覧ください。

**Q: これはGremlinのカオスエンジニアリングツールと似ていますか？**

**A:** はい、これはKubernetesに特化したオープンソースプロジェクトです。Kubernetesプラグインとして利用可能で、詳細はhttps://chaos-mesh.org でご確認いただけます。

**Q: ネットワークカオスにおいて、ネットワーク遅延はどのように注入されますか？iptablesを使用しないCilium CNI環境でも、この遅延注入は機能しますか？**

**A:** Chaos Meshにはchaos-daemonコンポーネントがあります。ネットワークカオスを発生させるとき、chaos-daemonはターゲットPodのネットワーク名前空間に入り、ネットワークデバイスにTCルールとiptablesルールを設定します。

iptablesを使用しないCilium CNI環境においても、Chaos Meshは正常に動作します。

## Chaos Meshコミュニティに参加しよう

Chaos Meshに興味があり、改善に協力したい方は、ぜひ[Slackチャンネル](https://slack.cncf.io/)(#project-chaos-mesh)に参加するか、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやイシューを投稿してください。