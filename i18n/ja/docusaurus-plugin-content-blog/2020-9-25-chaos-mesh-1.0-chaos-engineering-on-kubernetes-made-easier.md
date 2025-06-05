---
slug: /chaos-mesh-1.0-chaos-engineering-on-kubernetes-made-easier
title: 'Chaos Mesh 1.0: Chaos Engineering on Kubernetes Made Easier'
authors: chaos-mesh
image: /img/blog/chaos-mesh-1.0.png
tags: [Chaos Mesh, Chaos Engineering, Announcement]
---

![Chaos-Mesh-1.0 - Chaos-Engineering-on-Kubernetes-Made-Easier](/img/blog/chaos-mesh-1.0.png)

本日、私たちはChaos Mesh 1.0の一般提供開始を発表できることを誇りに思います。これは2020年7月にCNCFの[サンドボックスプロジェクト](https://pingcap.com/blog/announcing-chaos-mesh-as-a-cncf-sandbox-project)として採択されて以来の大きな進展です。

<!--truncate-->

Chaos Mesh 1.0はこのプロジェクトの開発における主要なマイルストーンです。オープンソースコミュニティにおける10ヶ月にわたる努力の結果、Chaos Meshは機能性、拡張性、使いやすさの面で完成の域に達しました。以下に主な特徴をご紹介します。

## 強力なカオスサポート

[Chaos Mesh](https://chaos-mesh.org)は分散型データベース[TiDB](https://pingcap.com/products/tidb)のテストフレームワークから生まれたため、分散システムで発生しうる障害を考慮しています。Chaos MeshはPod、ネットワーク、システムI/O、カーネルなど、包括的で細かい粒度の障害タイプを提供します。カオス実験はYAMLで定義され、迅速かつ簡単に使用できます。

Chaos Mesh 1.0では以下の障害タイプをサポートしています：

- clock-skew: 時刻のずれをシミュレート
- container-kill: コンテナの強制終了をシミュレート
- cpu-burn: CPU負荷をシミュレート
- io-attribution-override: ファイル例外をシミュレート
- io-fault: ファイルシステムI/Oエラーをシミュレート
- io-latency: ファイルシステムI/O遅延をシミュレート
- kernel-injection: カーネル障害をシミュレート
- memory-burn: メモリ負荷をシミュレート
- network-corrupt: ネットワークパケット破損をシミュレート
- network-duplication: ネットワークパケット重複をシミュレート
- network-latency: ネットワーク遅延をシミュレート
- network-loss: ネットワーク損失をシミュレート
- network-partition: ネットワーク分断をシミュレート
- pod-failure: Kubernetes Podの継続的利用不可をシミュレート
- pod-kill: Kubernetes Podの強制終了をシミュレート

## 視覚的なカオスオーケストレーション

Chaos Dashboardコンポーネントは、Chaos Meshユーザーがカオス実験をオーケストレーションするためのワンストップWebインターフェースです。以前はTiDBのテスト専用でしたが、Chaos Mesh 1.0では誰でも利用可能になりました。Chaos Dashboardはカオス実験の複雑さを大幅に軽減します。数回のマウスクリックだけで、カオス実験の範囲を定義し、注入する障害の種類を指定し、スケジューリングルールを定義し、カオス実験の結果を観察できます - すべて同じWebインターフェース上で行えます。

![Chaos Dashboard](/img/blog/chaos-dashboard.gif)

## 可観測性を強化するGrafanaプラグイン

カオス実験の可観測性をさらに向上させるため、Chaos Mesh 1.0にはGrafanaプラグインが含まれており、アプリケーション監視パネルにリアルタイムのカオス実験情報を直接表示できます。現在、カオス実験情報は注釈として表示されます。これにより、アプリケーションの実行状態と現在のカオス実験情報を同時に観察できます。

![Grafana上のカオス状態とアプリケーション状態](/img/blog/chaos-status.png)

## 安全で制御可能なカオス

カオス実験を実施する際には、カオスの範囲または「爆発半径」を厳密に制御することが極めて重要です。Chaos Mesh 1.0は実験範囲を正確に制御するための豊富なセレクターを提供するだけでなく、重要なアプリケーションを保護するための保護されたNamespaceを設定することも可能にします。また、Namespace権限を使用してChaos Meshの範囲を特定のNamespaceに制限することもできます。これらの機能を組み合わせることで、Chaos Meshを使ったカオス実験は安全かつ制御可能になります。

## 今すぐお試しください

:::note

2022-10-24: https://www.oreilly.com/online-learning/leveraging-katacoda-technology.html および [#356](https://github.com/chaos-mesh/website/pull/356) により、インタラクティブチュートリアルは一時的に利用できません。

:::

`install.sh`スクリプトまたはHelmツールを使用して、Kubernetes環境にChaos Meshを迅速にデプロイできます。具体的なインストール手順については、[Chaos Mesh 入門ガイド](https://chaos-mesh-website-archived.netlify.app/docs/1.2.4/user_guides/installation)ドキュメントを参照してください。また、`Katakodaインタラクティブチュートリアル`を利用すれば、デプロイせずにChaos Meshをすぐに体験することも可能です。

1.0 GAにアップグレードしていない場合は、変更点とアップグレードガイドラインについて[1.0 リリースノート](https://github.com/chaos-mesh/chaos-mesh/releases/tag/v1.0.0)を参照してください。

## 謝辞

すべてのChaos Mesh[コントリビューター](https://github.com/chaos-mesh/chaos-mesh/graphs/contributors)の皆様に感謝いたします。

Chaos Meshに興味をお持ちの方は、Issueの投稿やコード・ドキュメント・記事のコントリビューションを通じてぜひご参加ください。皆様の参加とフィードバックをお待ちしております！