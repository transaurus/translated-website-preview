---
slug: /deploy-chaos-mesh-on-kubesphere
title: 'Deploy Chaos Mesh on KubeSphere'
authors: cwen
image: /img/blog/chaos-mesh-kubesphere-banner.png
tags: [Chaos Mesh, Chaos Engineering, Community]
---

![KubeSphereにChaos Meshをデプロイ](/img/blog/chaos-mesh-kubesphere-banner.png)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、Kubernetes環境でカオスをオーケストレーションするクラウドネイティブなカオスエンジニアリングプラットフォームです。Chaos Meshを使用すると、Pod、ネットワーク、ファイルシステム、さらにはカーネルにさまざまな種類の障害を注入することで、Kubernetes上でのシステムの耐障害性と堅牢性をテストできます。

<!--truncate-->

![Chaos Meshアーキテクチャ](/img/blog/chaos-mesh-architecture-2.0.png)

## KubeSphereとは

[KubeSphere](https://kubesphere.io/)は、Kubernetesをカーネルとして使用するクラウドネイティブアプリケーション管理のための分散型オペレーティングシステムです。プラグアンドプレイアーキテクチャを提供し、サードパーティアプリケーションをそのエコシステムにシームレスに統合できます。

KubeSphere 3.2.0では、コミュニティで開発されたHelmチャートを[KubeSphere App Store](https://kubesphere.io/docs/pluggable-components/app-store/)に動的にロードする機能が追加されました。この新機能により、Chaos MeshがKubeSphereで利用可能になりました。このチュートリアルでは、KubeSphereにChaos Meshをデプロイしてカオス実験を行う方法を学びます。

## KubeSphereでApp Storeを有効にする

1. [KubeSphere App Store](https://kubesphere.io/docs/pluggable-components/app-store/)がインストールされ、有効になっていることを確認してください。

2. このチュートリアルでは、ワークスペース、プロジェクト、およびユーザーアカウント（project-regular）を作成する必要があります。アカウントはプラットフォームの通常ユーザーであり、オペレーターロールでプロジェクトオペレーターとして招待される必要があります。詳細については、[ワークスペース、プロジェクト、ユーザー、ロールの作成](https://kubesphere.io/docs/quick-start/create-workspace-and-project/)を参照してください。

## Chaos Meshを使ったカオス実験

### ステップ1: Chaos Meshをデプロイ

1. `project-regular`としてKubeSphereにログインし、**App Store**で**chaos-mesh**を検索し、検索結果をクリックしてアプリに入ります。

   ![Chaos Meshアプリ](/img/blog/chaos-mesh-app.png)

2. **アプリ情報**ページで、右上の**インストール**をクリックします。

   ![Chaos Meshをインストール](/img/blog/install-chaos-mesh.png)

3. **アプリ設定**ページで、アプリケーションの**名前**、**場所**（Namespaceとして）、および**アプリバージョン**を設定し、右上の**次へ**をクリックします。

   ![Chaos Mesh基本情報](/img/blog/chaos-mesh-basic-info.png)

4. `values.yaml`ファイルを必要に応じて設定するか、**インストール**をクリックしてデフォルト設定を使用します。

   ![Chaos Mesh設定](/img/blog/chaos-mesh-config.png)

5. デプロイが完了するのを待ちます。完了すると、Chaos MeshはKubeSphereで**実行中**と表示されます。

   ![Chaos Meshデプロイ完了](/img/blog/chaos-mesh-deployed.png)

### ステップ2: Chaos Dashboardにアクセス

1. **リソースステータス**ページで、`chaos-dashboard`の**NodePort**をコピーします。

   ![Chaos Mesh NodePort](/img/blog/chaos-mesh-nodeport.png)

2. ブラウザで`${NodeIP}:${NODEPORT}`を入力してChaos Dashboardにアクセスします。[ユーザー権限の管理](https://chaos-mesh.org/docs/manage-user-permissions/)を参照してTokenを生成し、Chaos Dashboardにログインします。

   ![Chaos Dashboardにログイン](/img/blog/login-to-dashboard.png)

### ステップ3: カオス実験を作成

カオス実験を作成する前に、実験対象を特定してデプロイする必要があります。たとえば、ネットワーク遅延下でアプリケーションがどのように動作するかをテストします。ここでは、テスト対象アプリケーションとしてデモアプリケーション`web-show`を使用し、テスト目標はシステムのネットワーク遅延を観察することです。以下のコマンドでデモアプリケーション`web-show`をデプロイできます: `web-show`。

```bash
curl -sSL https://mirrors.chaos-mesh.org/latest/web-show/deploy.sh | bash
```

> 注: Podのネットワーク遅延は、web-showアプリケーションのパッドからkube-systemのPodまで直接観察できます。

1. ウェブブラウザから`${NodeIP}:8081`にアクセスし、**Web Show**アプリケーションを開きます。

   ![Chaos Mesh web show app](/img/blog/web-show-app.png)

2. Chaos Dashboardにログインしてカオス実験を作成します。アプリケーションへのネットワーク遅延の影響を観察するため、**Target**を「Network Attack」に設定し、ネットワーク遅延シナリオをシミュレートします。

   ![Chaos Dashboard](/img/blog/chaos-dashboard-networkchaos.png)

   実験の**Scope**は`app: web-show`に設定します。

   ![Chaos Experiment scope](/img/blog/chaos-experiment-scope.png)

3. カオス実験を送信して開始します。

   ![Submit Chaos Experiment](/img/blog/start-chaos-experiment.png)

これで、**Web Show**にアクセスして実験結果を観察できるようになります:

![Chaos Experiment result](/img/blog/experiment-result.png)

## まとめ

KubeSphereはクラウドネイティブアプリケーションのデプロイとメンテナンスを容易にします。App Storeのおかげで、ユーザーは数回クリックするだけでKubeSphere上にChaos Meshを簡単にデプロイでき、独自のカオス実験をすぐに開始できます。

Chaos Meshについてさらに学ぶには、[Chaos Meshドキュメント](https://chaos-mesh.org/docs/)を参照するか、コミュニティSlack（[CNCF](https://slack.cncf.io/)/#project-chaos-mesh）に参加してください。