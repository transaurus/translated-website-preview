---
slug: /chaos-mesh-q&a
title: 'Chaos Mesh Q&A'
authors: chaos-mesh
image: /img/blog/chaos-mesh-q&a.jpeg
tags: [Chaos Mesh, Chaos Engineering]
---

![Chaos Mesh Q&A](/img/blog/chaos-mesh-q&a.jpeg)

KubeCon EU 2021では、[Chaos Mesh](https://chaos-mesh.org/)チームが2回の「オフィスアワーセッション」を開催し、新規参加者、コミュニティメンバー、プロジェクトメンテナーが互いに交流し、プロジェクトについて学ぶ機会を提供しました。

<!--truncate-->

200名以上の方々にご参加いただき、誠にありがとうございました！セッション中に多くの素晴らしい質問をいただいたため、Q&Aをまとめてご紹介します。

## 質問への回答

**Q: Chaos MeshはIstioなどのサービスメッシュと互換性がありますか？**

**A:** はい、サービスメッシュ環境でChaos Meshを使用できます。[以前のコミュニティミーティング](https://www.youtube.com/watch?v=paIgJYOhdGw)では、グアテマラのサンカルロス大学のSergio MéndezとJossie Castrilloが、LinkerdとChaos Meshを使用してプロジェクト「[COVID-19 Realtime Vaccinated People Visualizer](https://github.com/sergioarmgpl/operating-systems-usac-course/blob/master/lang/en/projects/project1v3/project1.md)」のカオス実験を実施した事例を共有しました。

![プロジェクトアーキテクチャ](/img/blog/chaos-mesh-linkerd-architecture.png)

**Q: Chaos Meshはオンプレミスで使用できますか、それともAmazon Web Services（AWS）やGoogle Cloud Platform（GCP）が必要ですか？**

**A:** どちらでも可能です！Chaos MeshはKubernetesクラスターにデプロイできるため、自分で管理している場合でもAWSやGCPでホストされている場合でも関係ありません。ただし、Kubernetes環境で使用する場合は、インストール時に[関連パラメータを設定](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/user_guides/installation)する必要があります。

**Q: 「カオスアクション」はどのように機能しますか？**

**A:** Chaos MeshはKubernetesのCustomResourceDefinitions（CRD）を使用してカオス実験を管理します。異なる故障注入動作は異なる方法で実装されますが、全体的な考え方は同じです：Chaos Meshはアプリケーションの実行リンクを使用してアプリケーションにカオスを注入します。例えば、ネットワークインタラクションの全体的なリンクにカオスを注入する場合、ネットワークインタラクションカードが通過します。Linuxはトラフィック制御を使用して特定のネットワークインタラクションカードへの干渉を増加させるため、ネットワーク故障注入に直接トラフィック制御を使用できます。

**Q: Chaos Meshにプローブサポートを追加して、定常状態の検出と実験検証を行いますか？**

**A:** 現在、このサポートを追加する予定はありません。定常状態の検出と実験検証は、アプリケーションが本番環境に適しているかどうかを判断するために必要です。Chaos Mesh自体は関連する監視作業を行いませんが、既存の監視システムやアプリケーションのステータスインターフェースにアクセスするためのインターフェースを提供し、アプリケーションの定常状態を監視・検出します。

**Q: Chaos Meshのポッドにはどのような特権昇格が必要ですか？**

**A:** デフォルトでは、Chaos MeshのChaos Daemonコンポーネントは`privileged`モードで実行されます。Kubernetesクラスターのバージョンがv3.11以上の場合は、`capabilities`を設定することで`privileged`モードを置き換えることができます。

**Q: ビルドパイプライン内にChaos Meshを実装して、特定のテスト結果をログに記録できますか？**

**A:** はい、簡単に実装できます。Chaos MeshはArgo、Jenkins、GitHub Action、Spannerなどのパイプラインシステムと統合できます。Chaos MeshはKubernetes CRDを使用してカオス実験を管理します。カオスを注入するには、パイプラインで目的のカオスCRDオブジェクトを作成するだけです。実験の実行ステータスは、そのステータス構造とイベントを通じて取得できます。

**Q: 2.0リリースでは何が期待できますか？HTTPChaosに関するアップデートを共有していただけますか？**

**A:** Chaos Mesh 2.0ではネイティブのワークフローサポートを提供し、ユーザーはChaos Mesh内でカオス実験を配置できるようになります。さらに、Chaos Mesh 2.0では既存のカオスコントローラーを再構築しており、ユーザーが新しい故障注入タイプをより簡単に追加できるようになります。HTTPChaosについては、HTTPアプリケーション層へのネットワーク障害シミュレーションを追加中です！

## Chaos Meshコミュニティに参加しましょう

Chaos Meshに興味があり、改善に協力したい方は、[Slackチャンネル](https://slack.cncf.io/)に参加するか、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやイシューを投稿してください。