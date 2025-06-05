---
title: Integrate Chaos Mesh to GitHub Actions
---

このドキュメントでは、Chaos Meshを統合してchaos-mesh-actionを使用し、継続的インテグレーション（CI）をカスタマイズする方法について説明します。これにより、製品リリース前にシステム開発で発生した問題を特定するのに役立ちます。

chaos-mesh-actionは、[GitHub Marketplace](https://github.com/marketplace/actions/chaos-mesh)でリリースされているGitHub Actionです。そのソースコードは[GitHub](https://github.com/chaos-mesh/chaos-mesh-action)でも公開されています。

## chaos-mesh-actionの設計

[GitHub Action](https://docs.github.com/en/actions)は、GitHubがネイティブでサポートする継続的インテグレーション（CI）および継続的デプロイメント（CD）機能です。GitHub Actionを使用すると、リポジトリ内でソフトウェア開発ワークフローを自動化およびカスタマイズできます。

GitHub Actionを装備することで、Chaos Meshを日常の開発とテストに簡単に統合できます。これにより、GitHubに提出されたすべてのコードが（少なくともテストを通過する程度に）バグフリーであることを確認できます。以下の画像は、CIワークフローに統合されたchaos-mesh-actionを示しています：

![chaos-mesh-action-integrate-in-the-ci-workflow](./img/chaos-mesh-action-integrate-in-the-ci-workflow.png)

## GitHubワークフローでのchaos-mesh-actionの使用

chaos-mesh-actionはGitHubワークフローで動作します。GitHubワークフローは設定可能な自動化プロセスです。リポジトリにGitHubワークフローを設定して、GitHubプロジェクトのビルド、テスト、パッケージ化、公開、またはデプロイを行うことができます。CIにChaos Meshを統合するには、以下の手順に従います：

- ステップ1: ワークフローの設計
- ステップ2: ワークフローの作成
- ステップ3: ワークフローの実行

### ステップ1: ワークフローの設計

ワークフローを設計する前に、以下の質問を考慮してください：

- このワークフローでテストしたい機能は何ですか？
- 注入する障害のタイプは何ですか？
- システムの正確性をどのように検証しますか？

例えば、テスト用の簡単なワークフローを設計できます。以下のステップを含めることができます：

1. Kubernetesクラスター内に2つのPodを作成します。
2. 1つのPodから別のPodにpingリクエストを送信します。
3. Chaos Meshを使用してネットワーク遅延障害を注入し、pingコマンドが影響を受けるかどうかをテストします。

### ステップ2: ワークフローの作成

ワークフローを設計した後、以下の手順でワークフローを作成します。

1. テスト対象のソフトウェアのGitHubリポジトリに入ります。
2. `Actions`をクリックし、`New workflow`をクリックしてワークフローを作成します。

![creating-a-workflow](./img/creating-a-workflow.png)

本質的に、ワークフローは順次実行される自動化ジョブの設定です。以下のジョブは単一のファイルで設定されています。明確な説明のために、スクリプトは以下のように異なる作業グループに分割されています：

- ワークフローの名前とトリガールールを設定します。

  ワークフローに「Chaos」という名前を付けます。コードをコミットするか、マスターブランチにプルリクエストを作成すると、このワークフローがトリガーされます。

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

- CI関連の環境をセットアップします。

  この設定では、オペレーティングシステム（Ubuntu）を指定し、helm/kind-actionを使用してKindクラスターを作成します。その後、クラスター情報を表示します。最後に、ワークフローがアクセスするGitHubリポジトリをチェックアウトします。

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

- アプリケーションをデプロイします。

  以下の例では、2つのKubernetes Podを作成するアプリケーションをデプロイします。

  ```yaml
  - name: Deploy an application
       run: |
         kubectl apply -f https://raw.githubusercontent.com/chaos-mesh/apps/master/ping/busybox-statefulset.yaml
  ```

- Chaos Meshを使用して障害を注入します。

  ```yaml
  - name: Run chaos mesh action
      uses: chaos-mesh/chaos-mesh-action@v0.5
      env:
        CHAOS_MESH_VERSION: v1.0.0
        CFG_BASE64: YXBpVmVyc2lvbjogY2hhb3MtbWVzaC5vcmcvdjFhbHBoYTEKa2luZDogTmV0d29ya0NoYW9zCm1ldGFkYXRhOgogIG5hbWU6IG5ldHdvcmstZGVsYXkKICBuYW1lc3BhY2U6IGJ1c3lib3gKc3BlYzoKICBhY3Rpb246IGRlbGF5ICMgdGhlIHNwZWNpZmljIGNoYW9zIGFjdGlvbiB0byBpbmplY3QKICBtb2RlOiBhbGwKICBzZWxlY3RvcjoKICAgIHBvZHM6CiAgICAgIGJ1c3lib3g6CiAgICAgICAgLSBidXN5Ym94LTAKICBkZWxheToKICAgIGxhdGVuY3k6ICIxMG1zIgogIGR1cmF0aW9uOiAiNXMiCiAgc2NoZWR1bGVyOgogICAgY3JvbjogIkBldmVyeSAxMHMiCiAgZGlyZWN0aW9uOiB0bwogIHRhcmdldDoKICAgIHNlbGVjdG9yOgogICAgICBwb2RzOgogICAgICAgIGJ1c3lib3g6CiAgICAgICAgICAtIGJ1c3lib3gtMQogICAgbW9kZTogYWxsCg==
  ```

  chaos-mesh-actionを使用すると、Chaos Meshが自動的にインストールされ、障害が注入されます。Chaos実験の設定を準備し、base64でエンコードされた値を取得するだけで済みます。Podにネットワーク遅延を注入したい場合は、以下の設定例を使用できます:

  ```yaml
  apiVersion: chaos-mesh.org/v1alpha1
  kind: NetworkChaos
  metadata:
    name: network-delay
    namespace: busybox
  spec:
    action: delay # 注入する特定のChaosアクション
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

  上記のChaos実験設定ファイルのbase64エンコードされた値を取得するには、以下のコマンドを使用します:

  ```bash
  base64 chaos.yaml
  ```

- システムの正しさを検証します。

  このジョブでは、ワークフローは1つのPodから別のPodにpingリクエストを送信し、ネットワーク遅延を観察します。

  ```yaml
  - name: Verify
       run: |
         echo "do some verification"
         kubectl exec busybox-0 -it -n busybox -- ping -c 30 busybox-1.busybox.busybox.svc
  ```

### ステップ3: ワークフローの実行

ワークフローが作成されると、マスターブランチへのプルリクエストを作成することでトリガーできます。ワークフローの実行が完了すると、ジョブ検証の出力は以下のようなものになります:

```log
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

出力には10ミリ秒の遅延が一連で表示され、各遅延は5秒間（5回）継続します。これはchaos-mesh-actionを使用して注入されたカオス実験の設定と一致しています。

## 次のステップ

現在、chaos-mesh-actionは[TiDB Operator](https://github.com/pingcap/tidb-operator)で適用されています。ワークフローにPod障害を注入することで、Operatorインスタンスの再起動を検証できます。これは、TiDB OperatorのPodが注入された障害によってランダムに削除された場合に、TiDB Operatorが正しく動作することを保証するためです。詳細は、[TiDB Operatorワークフローページ](https://github.com/pingcap/tidb-operator/actions?query=workflow%3Achaos)を参照してください。

将来的には、chaos-mesh-actionはより多くのTiDBテストで適用され、TiDBとそのコンポーネントの安定性を確保します。あなたもchaos-mesh-actionを使用して独自のワークフローを作成することを歓迎します。

chaos-mesh-actionで問題を見つけた場合、または情報が不足していると感じた場合は、[GitHub issue](https://github.com/pingcap/chaos-mesh/issues)を作成するか、Chaos Meshリポジトリに[プルリクエスト (PR)](https://github.com/chaos-mesh/chaos-mesh/pulls)を送ってください。[CNCF](https://www.cncf.io/)ワークスペースの[#project-chaos-mesh](https://slack.cncf.io/) Slackチャンネルに参加することもできます。