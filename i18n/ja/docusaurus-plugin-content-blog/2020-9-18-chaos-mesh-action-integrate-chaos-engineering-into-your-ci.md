---
slug: /chaos-mesh-action-integrate-chaos-engineering-into-your-ci
title: 'chaos-mesh-action: Integrate Chaos Engineering into Your CI'
authors: xiangwang
image: /img/blog/chaos-mesh-action.png
tags: [Chaos Mesh, Chaos Engineering, GitHub Actions, CI]
---

![chaos-mesh-action - CIへのカオスエンジニアリングの統合](/img/blog/chaos-mesh-action.png)

[Chaos Mesh](https://chaos-mesh.org)は、Kubernetes環境でカオスをオーケストレーションするクラウドネイティブなカオステストプラットフォームです。豊富な障害注入タイプと使いやすいダッシュボードでコミュニティから高い評価を得ていますが、エンドツーエンドテストや継続的インテグレーション（CI）プロセスとの連携が困難でした。そのため、システム開発中に導入された問題がリリース前に発見できないケースがありました。

この記事では、Chaos MeshをCIプロセスに統合するためのGitHubアクションであるchaos-mesh-actionの使用方法を紹介します。

<!--truncate-->

chaos-mesh-actionは[GitHubマーケット](https://github.com/marketplace/actions/chaos-mesh)で利用可能で、ソースコードは[GitHub](https://github.com/chaos-mesh/chaos-mesh-action)に公開されています。

## chaos-mesh-actionの設計

[GitHub Actions](https://docs.github.com/ja/actions)はGitHubがネイティブにサポートするCI/CD機能で、GitHubリポジトリ内で自動化されたカスタマイズ可能なソフトウェア開発ワークフローを簡単に構築できます。

GitHub Actionsと組み合わせることで、Chaos Meshを日常的な開発やテストに容易に統合でき、GitHub上の各コードコミットがバグフリーで既存のコードを破壊しないことを保証できます。以下の図はCIワークフローに統合されたchaos-mesh-actionを示しています：

![CIワークフローに統合されたchaos-mesh-action](/img/blog/chaos-mesh-action-integrate-in-the-ci-workflow.png)

## GitHubワークフローでのchaos-mesh-actionの使用

[chaos-mesh-action](https://github.com/marketplace/actions/chaos-mesh)はGitHubワークフローで動作します。GitHubワークフローは、リポジトリ内で設定可能な自動化プロセスで、GitHubプロジェクトのビルド、テスト、パッケージ化、リリース、デプロイを行うことができます。CIにChaos Meshを統合するには、以下の手順に従います：

1. ワークフローを設計する
2. ワークフローを作成する
3. ワークフローを実行する

### ワークフローの設計

ワークフローを設計する前に、以下の点を考慮する必要があります：

- このワークフローでテストする機能は何か？
- 注入する障害のタイプは何か？
- システムの正しさをどのように検証するか？

例として、以下のステップを含む簡単なテストワークフローを設計してみましょう：

1. Kubernetesクラスタ内に2つのPodを作成する
2. 一方のPodからもう一方のPodにpingを実行する
3. Chaos Meshを使用してネットワーク遅延カオスを注入し、pingコマンドが影響を受けるかどうかをテストする

### ワークフローの作成

ワークフローを設計したら、次は作成します。

1. テスト対象のソフトウェアを含むGitHubリポジトリに移動します
2. ワークフローの作成を開始するには、**Actions**をクリックし、**New workflow**ボタンをクリックします：

![ワークフローの作成](/img/blog/creating-a-workflow.png)

ワークフローは本質的に、順次的かつ自動的に実行されるジョブの設定です。ジョブは単一のファイルで設定されますが、説明をわかりやすくするため、以下のようにスクリプトを異なるジョブグループに分割しています：

- ワークフローの名前とトリガールールを設定する。

  このジョブではワークフローに「Chaos」という名前を付けています。コードがmasterブランチにプッシュされたとき、またはmasterブランチへのプルリクエストが作成されたときに、このワークフローがトリガーされます。

  ```yaml
  name: Chaos

  on:
    push:
      branches:
        - master
    pull_request:
      branches:
        - master
  ```

- CI関連の環境をセットアップする。

  この設定では、オペレーティングシステム（Ubuntu）を指定し、[helm/kind-action](https://github.com/marketplace/actions/kind-cluster)を使用してKindクラスターを作成します。その後、クラスターに関する関連情報を出力し、最後にワークフローがアクセスするためのGitHubリポジトリをチェックアウトします。

  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - name: Creating kind cluster
          uses: helm/kind-action@v1.0.0-rc.1

        - name: Print cluster information
          run: |
            kubectl config view
            kubectl cluster-info
            kubectl get nodes
            kubectl get pods -n kube-system
            helm version
            kubectl version

        - uses: actions/checkout@v2
  ```

- アプリケーションをデプロイする。

  この例では、2つのKubernetes Podを作成するアプリケーションをデプロイします。

  ```yaml
  - name: Deploy an application
       run: |
         kubectl apply -f https://raw.githubusercontent.com/chaos-mesh/apps/master/ping/busybox-statefulset.yaml
  ```

- chaos-mesh-actionを使用してカオスを注入する。

  ```yaml
  - name: Run chaos mesh action
      uses: chaos-mesh/chaos-mesh-action@v0.5
      env:
        CHAOS_MESH_VERSION: v1.0.0
        CFG_BASE64: YXBpVmVyc2lvbjogY2hhb3MtbWVzaC5vcmcvdjFhbHBoYTEKa2luZDogTmV0d29ya0NoYW9zCm1ldGFkYXRhOgogIG5hbWU6IG5ldHdvcmstZGVsYXkKICBuYW1lc3BhY2U6IGJ1c3lib3gKc3BlYzoKICBhY3Rpb246IGRlbGF5ICMgdGhlIHNwZWNpZmljIGNoYW9zIGFjdGlvbiB0byBpbmplY3QKICBtb2RlOiBhbGwKICBzZWxlY3RvcjoKICAgIHBvZHM6CiAgICAgIGJ1c3lib3g6CiAgICAgICAgLSBidXN5Ym94LTAKICBkZWxheToKICAgIGxhdGVuY3k6ICIxMG1zIgogIGR1cmF0aW9uOiAiNXMiCiAgc2NoZWR1bGVyOgogICAgY3JvbjogIkBldmVyeSAxMHMiCiAgZGlyZWN0aW9uOiB0bwogIHRhcmdldDoKICAgIHNlbGVjdG9yOgogICAgICBwb2RzOgogICAgICAgIGJ1c3lib3g6CiAgICAgICAgICAtIGJ1c3lib3gtMQogICAgbW9kZTogYWxsCg==
  ```

  chaos-mesh-actionを使用すると、Chaos Meshのインストールとカオスの注入が自動的に完了します。必要なのは、使用するカオス設定を準備し、そのBase64表現を取得することだけです。ここでは、Podにネットワーク遅延カオスを注入したいため、以下のような元のカオス設定を使用します：

  ```yaml
  apiVersion: chaos-mesh.org/v1alpha1
  kind: NetworkChaos
  metadata:
    name: network-delay
    namespace: busybox
  spec:
    action: delay # 注入する具体的なカオスアクション
    mode: all
    selector:
      pods:
        busybox:
          - busybox-0
    delay:
      latency: '10ms'
    duration: '5s'
    scheduler:
      cron: '@every 10s'
    direction: to
    target:
      selector:
        pods:
          busybox:
            - busybox-1
      mode: all
  ```

  上記のカオス設定ファイルのBase64値は、以下のコマンドで取得できます：

  ```shell
  $ base64 chaos.yaml
  ```

- システムの正しさを検証する。

  このジョブでは、ワークフローが一方のPodからもう一方のPodにpingを実行し、ネットワーク遅延の変化を観察します。

  ```yaml
  - name: Verify
       run: |
         echo "do some verification"
         kubectl exec busybox-0 -it -n busybox -- ping -c 30 busybox-1.busybox.busybox.svc
  ```

### ワークフローの実行

ワークフローの設定が完了したので、masterブランチにプルリクエストを送信することでトリガーできます。ワークフローが完了すると、検証ジョブによって以下のような結果が出力されます。

```shell
do some verification
Unable to use a TTY - input is not a terminal or the right kind of file
PING busybox-1.busybox.busybox.svc (10.244.0.6): 56 data bytes
64 bytes from 10.244.0.6: seq=0 ttl=63 time=0.069 ms
64 bytes from 10.244.0.6: seq=1 ttl=63 time=10.136 ms
64 bytes from 10.244.0.6: seq=2 ttl=63 time=10.192 ms
64 bytes from 10.244.0.6: seq=3 ttl=63 time=10.129 ms
64 bytes from 10.244.0.6: seq=4 ttl=63 time=10.120 ms
64 bytes from 10.244.0.6: seq=5 ttl=63 time=0.070 ms
64 bytes from 10.244.0.6: seq=6 ttl=63 time=0.073 ms
64 bytes from 10.244.0.6: seq=7 ttl=63 time=0.111 ms
64 bytes from 10.244.0.6: seq=8 ttl=63 time=0.070 ms
64 bytes from 10.244.0.6: seq=9 ttl=63 time=0.077 ms
……
```

出力結果から、約5秒間隔で10ミリ秒の遅延が規則的に発生していることがわかります。これはchaos-mesh-actionに注入したカオス設定と一致しています。

## 現在の状況と次のステップ

現在、私たちはchaos-mesh-actionを[TiDB Operator](https://github.com/pingcap/tidb-operator)プロジェクトに適用しています。このワークフローでは、Podカオスを注入してオペレーターの指定インスタンスの再起動機能を検証しています。目的は、tidb-operatorがオペレーターのPodが注入された障害によってランダムに削除された場合でも正常に動作することを保証することです。詳細については[TiDB Operatorページ](https://github.com/pingcap/tidb-operator/actions?query=workflow%3Achaos)をご覧ください。

将来的には、chaos-mesh-actionをより多くのテストに適用し、TiDBと関連コンポーネントの安定性を確保する予定です。皆さんもchaos-mesh-actionを使用して独自のワークフローを作成することを歓迎します。

バグを発見したり、何か不足していると思われる場合は、遠慮なくissueを登録したり、プルリクエスト(PR)を開いたり、[CNCF](https://www.cncf.io/)のSlackワークスペースにある[#project-chaos-mesh](https://slack.cncf.io/)チャンネルで私たちに参加してください。