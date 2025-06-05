---
slug: /better-observability-for-chaos-engineering
title: 'Chaos Mesh + SkyWalking: Better Observability for Chaos Engineering'
authors: ningxuanwang
image: /img/blog/chaos-mesh-skywalking-banner.png
tags: [Chaos Mesh, Chaos Engineering, Tutorials]
---

![Chaos Mesh + SkyWalking: カオスエンジニアリングのためのより優れた可観測性](/img/blog/chaos-mesh-skywalking-banner.png)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、オープンソースのクラウドネイティブな[カオスエンジニアリング](https://en.wikipedia.org/wiki/Chaos_engineering)プラットフォームです。Chaos Meshを使用すると、現実に発生する可能性のある障害や異常を簡単に注入・シミュレートでき、システム内の潜在的な問題を特定できます。Chaos MeshはChaos Dashboardも提供しており、カオス実験の状態を監視できます。しかし、このダッシュボードでは、実験中の障害がアプリケーションのサービスパフォーマンスにどのような影響を与えるかを観察することはできません。これにより、システムのさらなるテストや潜在的な問題の発見が妨げられています。

<!--truncate-->

[Apache SkyWalking](https://github.com/apache/skywalking)は、オープンソースのアプリケーションパフォーマンスモニタリング（APM）ツールで、特にクラウドネイティブでコンテナベースの分散システムを監視、追跡、診断するために設計されています。発生したイベントを収集し、ダッシュボードに表示することで、システム内で発生したイベントの種類や数、およびさまざまなイベントがサービスパフォーマンスにどのような影響を与えるかを直接観察できます。

SkyWalkingとChaos Meshをカオス実験中に併用することで、さまざまな障害がサービスパフォーマンスに与える影響を観察できます。

このチュートリアルでは、SkyWalkingとChaos Meshの設定方法を紹介します。また、2つのシステムを活用してイベントを監視し、カオス実験がアプリケーションのサービスパフォーマンスにリアルタイムでどのような影響を与えるかを観察する方法も学びます。

## 準備

SkyWalkingとChaos Meshの使用を開始する前に、以下の手順を実行する必要があります：

- [SkyWalking設定ガイド](https://github.com/apache/skywalking-kubernetes#install)に従ってSkyWalkingクラスタをセットアップします。
- [Helmを使用して](https://chaos-mesh.org/docs/production-installation-using-helm/)Chaos Meshをデプロイします。
- [JMeter](https://jmeter.apache.org/index.html)または他のJavaテストツールをインストールします（サービス負荷を増加させるため）。
- デモを実行したい場合は、[このガイド](https://github.com/chaos-mesh/chaos-mesh-on-skywalking)に従ってSkyWalkingとChaos Meshを設定します。

これで準備は整いました。本題に進みましょう。

## ステップ1: SkyWalkingクラスタにアクセスする

SkyWalkingクラスタをインストールした後、そのユーザーインターフェース（UI）にアクセスできます。ただし、この時点ではサービスが実行されていないため、監視を開始する前にサービスを追加し、エージェントを設定する必要があります。

このチュートリアルでは、軽量なマイクロサービスフレームワークであるSpring Bootを例に、簡略化されたデモ環境を構築します。

1. [このドキュメント](https://github.com/chaos-mesh/chaos-mesh-on-skywalking/blob/master/demo-deployment.yaml)を参照して、Spring BootでSkyWalkingデモを作成します。
2. コマンド`kubectl apply -f demo-deployment.yaml -n skywalking`を実行してデモをデプロイします。

デプロイが完了すると、SkyWalking UIでリアルタイムの監視結果を観察できます。

**注:** Spring BootとSkyWalkingは同じデフォルトポート番号（8080）を使用しています。ポートフォワーディングを設定する際には注意が必要です。そうしないと、ポート競合が発生する可能性があります。例えば、`kubectl port-forward svc/spring-boot-skywalking-demo 8079:8080 -n skywalking`のようなコマンドを使用してSpring Bootのポートを8079に設定することで、競合を回避できます。

## ステップ2: SkyWalking Kubernetes Event Exporterをデプロイする

[SkyWalking Kubernetes Event Exporter](https://github.com/apache/skywalking-kubernetes-event-exporter)は、Kubernetesイベントを監視、フィルタリングし、SkyWalkingバックエンドに送信することができます。SkyWalkingはこれらのイベントをシステムメトリクスに関連付け、メトリクスがいつ、どのようにイベントによって影響を受けたかについての概要を表示します。

SkyWalking Kubernetes Event Explorerを1行のコマンドでデプロイしたい場合は、[このドキュメント](https://github.com/chaos-mesh/chaos-mesh-on-skywalking/blob/master/exporter-deployment.yaml)を参照してYAML形式の設定ファイルを作成し、フィルターとエクスポーターのパラメーターをカスタマイズしてください。その後、`kubectl apply`コマンドを使用してSkyWalking Kubernetes Event Explorerをデプロイできます。

## ステップ3: JMeterを使用してサービス負荷を増加させる

サービスパフォーマンスの変化をよりよく観察するためには、Spring Boot上のサービス負荷を増加させる必要があります。このチュートリアルでは、広く採用されているJavaテストツールであるJMeterを使用してサービス負荷を増加させます。

JMeterを使用して`localhost:8079`に対して負荷テストを実行し、5つのスレッドを追加して継続的にサービス負荷を増加させます。

![JMeter Dashboard 1](/img/blog/jmeter-1.png)

![JMeter Dashboard 2](/img/blog/jmeter-2.png)

SkyWalkingダッシュボードを開きます。アクセス率が100%であり、サービス負荷が約5,300コール/分（CPM）に達していることがわかります。

![SkyWalking Dashboard](/img/blog/skywalking-dashboard.png)

## ステップ4: Chaos Meshで障害を注入し結果を観察する

上記の3つのステップを完了したら、Chaos Dashboardを使用してストレスシナリオをシミュレートし、カオス実験中のサービスパフォーマンスの変化を観察できます。

![StressChaos on Chaos Dashboard](/img/blog/chaos-dashboard-stresschaos.png)

以下のセクションでは、3つのカオス条件下でのサービスパフォーマンスの変化について説明します：

- CPU負荷: 10%; メモリ負荷: 128 MB

  最初のカオス実験では、低いCPU使用率をシミュレートします。実験の開始と終了をダッシュボードに表示するには、ダッシュボードの右側にある切り替えボタンをクリックします。実験がシステムに適用されたか、またはシステムから回復したかを確認するには、短い緑色の線にカーソルを合わせます。

  2つの短い緑色の線の間の期間中、サービス負荷は4,929 CPMに減少しますが、カオス実験が終了すると正常に戻ります。

  ![Test 1](/img/blog/cpuload-1.png)

- CPU負荷: 50%; メモリ負荷: 128 MB

  アプリケーションのCPU負荷が50%に増加すると、サービス負荷は4,307 CPMに減少します。

  ![Test 2](/img/blog/cpuload-2.png)

- CPU負荷: 100%; メモリ負荷: 128 MB

  CPU使用率が100%の場合、サービス負荷はカオス実験が行われていない場合のわずか40%に減少します。

  ![Test 3](/img/blog/cpuload-3.png)

  Linuxシステムのプロセススケジューリングでは、プロセスがCPUを常に占有することを許可しないため、デプロイされたSpring Bootデモは、CPU負荷が最大の場合でもアクセスリクエストの40%を処理できます。

## まとめ

SkyWalkingとChaos Meshを組み合わせることで、カオス実験がアプリケーションのサービスパフォーマンスにいつ、どの程度影響を与えるかを明確に観察できます。このツールの組み合わせにより、さまざまな極端な条件下でのサービスパフォーマンスを観察できるため、サービスの信頼性を高めることができます。

Chaos Meshは2021年に多くの成長を遂げました。これはPingCAPのエンジニアとコミュニティ貢献者のたゆまぬ努力のおかげです。多様なユーザーへのサポートを継続的に向上させ、カオスエンジニアリングにおけるユーザー体験をさらに理解するために、[このアンケート](https://www.surveymonkey.com/r/X77BCNM)にご参加いただき、貴重なフィードバックをいただければ幸いです。

Chaos Meshについてさらに知りたい場合は、[GitHubのChaos Meshコミュニティ](https://github.com/chaos-mesh)に参加するか、[Slackディスカッション](https://slack.cncf.io/) (#project-chaos-mesh)にご参加ください。Chaos Meshの使用中にバグや不足している機能を見つけた場合は、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストまたはイシューを提出してください。