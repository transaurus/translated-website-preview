---
slug: /run_your_first_chaos_experiment
title: Run Your First Chaos Experiment in 10 Minutes
authors: cwen
image: /img/blog/run-first-chaos-experiment-in-ten-minutes.jpg
tags: [Chaos Mesh, Chaos Engineering, Kubernetes]
---

![10分で初めてのカオス実験を実行する](/img/blog/run-first-chaos-experiment-in-ten-minutes.jpg)

カオスエンジニアリングは、異常または破壊的な状況をシミュレートすることで、本番環境のソフトウェアシステムの堅牢性をテストする方法です。しかし多くの人にとって、カオスエンジニアリングを学ぶことから自社システムで実践するまでの移行は困難に感じられます。まるで十分な装備を持ったチームが事前に計画を立てる必要がある大きなアイデアのように思えるかもしれません。しかし、必ずしもそうではありません。カオス実験を始めるには、適切なプラットフォームがあれば十分なのです。

<!--truncate-->

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、**使いやすい**オープンソースのクラウドネイティブなカオスエンジニアリングプラットフォームで、Kubernetes環境でカオスをオーケストレーションします。この10分チュートリアルでは、カオスエンジニアリングを迅速に開始し、Chaos Meshで初めてのカオス実験を実行する方法を紹介します。

Chaos Meshの詳細については、[以前の記事](https://pingcap.com/blog/chaos-mesh-your-chaos-engineering-solution-for-system-resiliency-on-kubernetes/)またはGitHubの[chaos-meshプロジェクト](https://github.com/chaos-mesh/chaos-mesh)を参照してください。

## 実験のプレビュー

カオス実験は、科学の授業で行う実験に似ています。制御された環境で乱れた状況をシミュレートすることは全く問題ありません。ここでは、[web-show](https://github.com/chaos-mesh/web-show)という小さなWebアプリケーションでネットワークカオスをシミュレートします。カオスの効果を可視化するため、web-showは10秒ごとに自身のPodから`kube-system`ネームスペース内のkube-controller Podまでのレイテンシを記録します。

以下のクリップは、Chaos Meshのインストール、web-showのデプロイ、そして数コマンドでカオス実験を作成するプロセスを示しています：

![カオス実験の全プロセス](/img/blog/whole-process-of-chaos-experiment.gif)

さあ、あなたの番です！実際に手を動かしてみましょう。

## 始めましょう！

この簡単な実験では、Kubernetes開発用のDocker内Kubernetes（[Kind](https://kind.sigs.k8s.io/)）を使用します。[Minikube](https://minikube.sigs.k8s.io/)や既存のKubernetesクラスターを使用して進めることも自由です。

### 環境の準備

先に進む前に、ローカルコンピュータに[Git](https://git-scm.com/)と[Docker](https://www.docker.com/)がインストールされており、Dockerが実行中であることを確認してください。macOSの場合、Dockerに少なくとも6つのCPUコアを割り当てることを推奨します。詳細は、[Mac向けDocker設定](https://docs.docker.com/docker-for-mac/#advanced)を参照してください。

1. Chaos Meshを取得：

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh.git
   cd chaos-mesh/
   ```

2. `install.sh`スクリプトでChaos Meshをインストール：

   ```bash
   ./install.sh --local kind
   ```

   `install.sh`は自動化されたシェルスクリプトで、環境をチェックし、Kindをインストールし、ローカルにKubernetesクラスターを起動し、Chaos Meshをデプロイします。`install.sh`の詳細な説明を見るには、`--help`オプションを追加してください。

   > **注記:**
   >
   > ローカルコンピュータが`docker.io`または`gcr.io`からイメージをプルできない場合、ローカルのgcr.ioミラーを使用し、代わりに`./install.sh --local kind --docker-mirror`を実行してください。

3. システム環境変数を設定：

   ```bash
   source ~/.bash_profile
   ```

> **注記:**
>
> - ネットワーク環境によっては、これらの手順に数分かかる場合があります。
> - 以下のようなエラーメッセージが表示された場合:
>
>   ```bash
>   ERROR: failed to create cluster: failed to generate kubeadm config content: failed to get kubernetes version from node: failed to get file: command "docker exec --privileged kind-control-plane cat /kind/version" failed with error: exit status 1
>   ```
>
>   ローカルコンピュータ上のDockerに割り当てるリソースを増やし、次のコマンドを実行してください:
>
>   ```bash
>   ./install.sh --local kind --force-local-kube
>   ```

プロセスが完了すると、Chaos Meshが正常にインストールされたことを示すメッセージが表示されます。

### アプリケーションのデプロイ

次のステップは、テスト用のアプリケーションをデプロイすることです。ここでは、ネットワークカオスの効果を直接観察できるweb-showを選択します。テスト用に独自のアプリケーションをデプロイすることも可能です。

1. `deploy.sh`スクリプトを使用してweb-showをデプロイ:

   ```bash
   # Chaos Meshディレクトリにいることを確認
   cd examples/web-show &&
   ./deploy.sh
   ```

   > **注記:**
   >
   > ローカルコンピュータが`docker.io`からイメージをプルできない場合、`local gcr.io`ミラーを使用して`./deploy.sh --docker-mirror`を実行してください。

2. web-showアプリケーションにアクセス。ウェブブラウザから`http://localhost:8081`に移動します。

### カオス実験の作成

準備が整ったので、カオス実験を実行しましょう！

Chaos Meshは[CustomResourceDefinitions](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/)(CRD)を使用してカオス実験を定義します。CRDオブジェクトは実験シナリオごとに個別に設計されており、CRDオブジェクトの定義を大幅に簡素化しています。現在、Chaos Meshで実装されているCRDオブジェクトにはPodChaos、NetworkChaos、IOChaos、TimeChaos、KernelChaosがあります。今後、より多くの障害注入タイプをサポートする予定です。

この実験では、カオス実験に[NetworkChaos](https://github.com/chaos-mesh/chaos-mesh/blob/master/examples/web-show/network-delay.yaml)を使用します。YAMLで記述されたNetworkChaos設定ファイルは以下の通りです:

```
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay-example
spec:
  action: delay
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      "app": "web-show"
  delay:
    latency: "10ms"
    correlation: "100"
    jitter: "0ms"
  duration: "30s"
  scheduler:
    cron: "@every 60s"
```

NetworkChaosアクションの詳細な説明については、[Chaos Mesh wiki](https://github.com/chaos-mesh/chaos-mesh/wiki/Network-Chaos)を参照してください。ここでは、設定を以下のように言い換えます:

- ターゲット: `web-show`
- ミッション: `60s`ごとに`10ms`のネットワーク遅延を注入
- 攻撃期間: 毎回`30s`

NetworkChaosを開始するには、以下の手順を実行します:

1. `network-delay.yaml`を実行:

   ```bash
   # chaos-mesh/examples/web-showディレクトリにいることを確認
   kubectl apply -f network-delay.yaml
   ```

2. web-showアプリケーションにアクセス。ウェブブラウザから`http://localhost:8081`に移動します。

   折れ線グラフから、60秒ごとに10msのネットワーク遅延が発生していることがわかります。

![Chaos Meshを使用してweb-showに遅延を挿入](/img/blog/using-chaos-mesh-to-insert-delays-in-web-show.png)

おめでとうございます！あなたは少しばかりのカオスを引き起こしました。興味を持ち、Chaos Meshでさらに多くのカオス実験を試したい場合は、[examples/web-show](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples/web-show)をチェックしてください。

### カオス実験の削除

テストが終了したら、カオス実験を終了させます。

1. `network-delay.yaml`を削除:

   ```bash
   # chaos-mesh/examples/web-showディレクトリにいることを確認
   kubectl delete -f network-delay.yaml
   ```

2. web-showアプリケーションにアクセス。ウェブブラウザから`http://localhost:8081`に移動します。

折れ線グラフから、ネットワークの遅延レベルが正常に戻ったことが確認できます。

![ネットワーク遅延レベルが正常に戻った様子](/img/blog/network-latency-level-is-back-to-normal.png)

### Kubernetesクラスタの削除

カオス実験が終了したら、以下のコマンドを実行してKubernetesクラスタを削除してください：

```bash
kind delete cluster --name=kind
```

> **注記:**
>
> `kind: command not found` エラーが発生した場合は、まず `source ~/.bash_profile` コマンドを実行してからKubernetesクラスタを削除してください。

## 素晴らしい！次は何をしましょうか？

Chaos Engineeringへの初めての旅が成功しましたね。いかがでしたか？Chaos Engineeringは簡単でしょう？しかし、Chaos Meshがそれほど使いやすくないと感じたかもしれません。コマンドライン操作は不便だし、YAMLファイルを手動で書くのは少し面倒だし、実験結果を確認するのもやや煩雑かもしれませんか？心配しないでください、Chaos Dashboardが準備中です！Web上でカオス実験を実行するのは確かにエキサイティングです！クラウドプラットフォームのテスト標準を構築したり、Chaos Meshをさらに改善するお手伝いをしたい方は、ぜひご連絡ください！

バグを見つけたり、何か不足していると思われる場合は、遠慮なくissueを登録したり、プルリクエスト（PR）を開いたり、[CNCF slackワークスペース](https://slack.cncf.io/)の#project-chaos-meshチャンネルに参加してください。

GitHub: [https://github.com/chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)