---
slug: /chaos-mesh-celebrates-100th-contributor
title: 'Chaos Mesh Celebrates 100th Contributor'
authors: chaos-mesh
image: /img/blog/chaos-mesh-celebrates-100-contributors.png
tags: [Chaos Mesh, Chaos Engineering, Community]
---

![Chaos Meshが100人目のコントリビューターを迎える](/img/blog/chaos-mesh-celebrates-100-contributors.png)

[Chaos Meshプロジェクト](https://github.com/chaos-mesh/chaos-mesh)は2つの大きなマイルストーンを達成しました。コミュニティでは最近、[100人目のコントリビューター](https://github.com/chaos-mesh/chaos-mesh/graphs/contributors)がchaos-meshリポジトリに加わり、[Twitter](https://twitter.com/chaos_mesh)のフォロワー数も1,000人を突破しました！

<!--truncate-->

Chaos Meshは、Kubernetes環境でカオス実験をオーケストレーションするカオスエンジニアリングプラットフォームです。2019年12月31日にGitHubでオープンソース化されて以来、止まることなく進化を続けています。2020年7月には、CNCFの[Sandboxプロジェクト](https://chaos-mesh.org/blog/chaos-mesh-join-cncf-sandbox-project)として参加し、その数ヶ月後の9月にはChaos Mesh 1.0が[正式リリース](https://chaos-mesh.org/blog/chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-easier)されました。2021年7月には、いくつかのベータ版を経て、[Chaos Mesh 2.0が一般利用可能](https://github.com/chaos-mesh/chaos-mesh/releases/tag/v2.0.0)として発表されました！

これまでに、Chaos Meshは35のリリースを公開し、100人以上のコントリビューターから1,500以上のコミットを受け取り、3.8k以上のスターと420以上のフォークを獲得しています。これらの成果は、素晴らしいコミュニティなしには実現できませんでした。

![Chaos Meshのコントリビューターたち](/img/blog/chaos-mesh-all-contributors.jpeg)

以下は、私たちが特に注目したいコントリビューションの一部です：

- [@YangKeao](https://github.com/YangKeao)は、CRDを使用してKubernetes APIを構築するためのSDKである`kubebuilder`をChaos Meshに導入し、コントローラーの実装手順を簡素化しました。
- [@g1eny0ung](https://github.com/g1eny0ung)は、カオス実験を操作・観察するためのWeb UIであるChaos Dashboardを導入しました。
- [@Yiyiyimu](https://github.com/Yiyiyimu)は、カオス開発とデバッグを簡素化するツール`chaosctl`をコントリビュートしました。
- [@Gallardot](https://github.com/Gallardot)は、JVMChaosの実装を支援し、Chaos MeshがJVMアプリケーションの障害をシミュレートできるようにしました。
- [@STRRL](https://github.com/STRRL)は、Chaos Mesh Workflowの作業を開始しました。これは、本番レベルのエラーをシミュレートするために、異なるカオス実験を直列または並列で実行できる組み込みのワークフローエンジンです。

カオスエンジニアリングとオープンソースを同等に楽しむ方々にとって、この場所が皆さんにとっての居場所となるよう、コントリビューションの旅を豊かにすることを私たちの使命としています。以下は、これまでの進捗です：

- 2021年初頭にChaos Meshの[ガバナンス](https://github.com/chaos-mesh/chaos-mesh/blob/master/GOVERNANCE.md)を公開し、各コミュニティメンバーの役割と責任、意思決定プロセスを明確にし、これまでに9人のコミッターを昇格させました。
- これまでにLFXメンタープログラムを通じて4人のメンティーを指導しました。メンティーたちはブログを執筆し、LFX体験を共有するトークを開催しました。
- 3回のKubeConに参加し、バグバッシュコンテストに参加したり、Office Hoursを開催して古い顔ぶれと会話したり、新しいメンバーをコミュニティに迎えたりしました。KubeCon EU 2021の後には、多くの質問を受けたため、[Q&A](https://chaos-mesh.org/blog/chaos-mesh-q&a)も投稿しました！
- 現在、Chaos MeshをCNCFの[インキュベーティングステージ](https://github.com/cncf/toc/pull/683)に昇格させることを提案中で、次の成熟段階に進むことでプロジェクトに新たな機会とさらなる注目が集まることを期待しています。

これは祝福に値する成果ですが、まだやるべきことがたくさんあることを私たちは認識しています：

- また、コミュニティと協力してChaos Meshの[ドキュメント](https://chaos-mesh.org/docs/)を改善しています：各リリースごとに英語版を更新し、増え続ける中国語ユーザーやコントリビューター向けに中国語版を追加しています。
- クラウドネイティブエコシステムへの貢献を継続したいと考えています：例えば、カオスエンジニアリング関連のコンテンツを開発・拡充したり、他のコミュニティと協力してミートアップやプロジェクトを行ったりすることです。

私たちのもう一つの目標は、より多様で活発なコミュニティを構築し続けることです。Chaos Meshコミュニティに参加し、コントリビューターになることに障壁はありません。コントリビューションはコーディングに限定されず、ドキュメントの執筆、機能のアイデア提供、issueの投稿、ブログの執筆、コミュニティの質問への回答、ケーススタディの共有など、すべてが貢献の一部です。

## まとめ

心から感謝申し上げます！私たちはこの良い仕事を続け、この小さからぬコミュニティをさらに発展させ、CNCFとカオスエンジニアリングのエコシステムに貢献し続けたいと考えています。

Chaos Meshを初めて知った方で、さらに詳しく知りたい場合は、[CNCF Slackワークスペース](https://slack.cncf.io/)の#project-chaos-meshチャンネルを探すか、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやissueを投稿するか、次回の[月例コミュニティミーティング](https://community.cncf.io/chaos-mesh-community/)に参加登録してください！