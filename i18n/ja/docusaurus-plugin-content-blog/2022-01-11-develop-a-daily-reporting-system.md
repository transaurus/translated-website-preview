---
slug: /develop-a-daily-reporting-system
title: 'How to Develop a Daily Reporting System to Track Chaos Testing Results'
authors: leili
image: /img/blog/chaos-mesh-digitalchina-banner.png
tags: [Chaos Mesh, Chaos Engineering, Use Cases]
---

![カオステスト結果を追跡する日次レポートシステムの開発方法](/img/blog/chaos-mesh-digitalchina-banner.png)

Chaos Meshは、Kubernetes環境でカオス実験をオーケストレーションするクラウドネイティブなカオスエンジニアリングプラットフォームです。ネットワーク障害、ファイルシステム障害、Pod障害などの問題をシミュレートすることで、システムの耐障害性をテストできます。各カオス実験の後、ログを確認することでテスト結果をレビューできますが、これは直接的でも効率的でもありません。そのため、ログを自動的に分析しレポートを生成する日次レポートシステムを開発することにしました。これにより、ログの確認と問題の特定が容易になります。

<!--truncate-->

この記事では、日次レポートシステムを構築する方法についての洞察と、その過程で遭遇した問題とその解決方法を紹介します。

## KubernetesにChaos Meshをデプロイ

Chaos MeshはKubernetes向けに設計されており、これが特定のアプリケーションに対してファイルシステム、Pod、またはネットワークに障害を注入できる重要な理由の一つです。

以前のドキュメントでは、Chaos Meshはマシン上に仮想Kubernetesクラスタを迅速にデプロイする2つの方法を提供していました：[kind](https://github.com/kubernetes-sigs/kind)と[minikube](https://minikube.sigs.k8s.io/docs/start/)。一般的に、KubernetesクラスタのデプロイとChaos Meshのインストールには1行のコマンドしかかかりません。しかし、いくつかの問題があります：

- ローカルでKubernetesクラスタを起動すると、ネットワーク関連の障害タイプに影響を与えます。
- 中国本土のユーザーは、Dockerイメージのプルが極端に遅くなったり、タイムアウトしたりする可能性があります。

提供されたスクリプトを使用してkindでKubernetesクラスタをデプロイすると、すべてのKubernetesノードが仮想マシン（VM）になります。これにより、オフラインでイメージをプルするのが難しくなります。この問題に対処するために、複数の物理マシン上にKubernetesクラスタをデプロイし、各物理マシンをワーカーノードとして動作させることができます。イメージのプルプロセスを迅速化するために、`docker load`コマンドを使用して必要なイメージを事前にロードできます。上記の2つの問題以外は、ドキュメントに従って[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)と[Helm](https://helm.sh/)をインストールできます。

注：最新のインストールおよびデプロイ手順については、[Chaos Meshクイックスタート](https://chaos-mesh.org/docs/quick-start/)を参照してください。

## TiDBをデプロイ

次のステップは、Kubernetes上にTiDBをデプロイすることです。TiDB Operatorを使用してプロセスを効率化しました。詳細については、[KubernetesでTiDB Operatorを始める](https://docs.pingcap.com/tidb-in-kubernetes/stable/get-started)を参照してください。

このプロセスで特に強調したい点が2つあります：

- まず、TiDB Operatorのさまざまなコンポーネントを実装するためにCustom Resource Definitions（CRDs）をインストールします。そうしないと、TiDB Operatorをインストールしようとしたときにエラーが発生します。
- [Longhorn](https://longhorn.io/)（Kubernetes向けの分散ブロックストレージシステム）を使用して、Kubernetesクラスタ用のローカル永続ボリューム（PV）を作成します。これにより、事前にPVを作成する必要がなくなります：Podがプルされるたびに、PVが自動的に作成されマウントされます。

遭遇した最大の問題は、サービスのデプロイ時にイメージのプルが極端に遅くなることでした。Kubernetesクラスタのノードが仮想マシンの場合、必要なイメージを事前にプルし、各マシンのDockerにロードします：

```bash
## Pull required images on a machine with a good network connection
docker pull pingcap/tikv:latest
docker pull pingcap/tidb:latest
docker pull pingcap/pd:latest

## Export images and save them to each machine in the Kubernetes cluster
docker save -o tikv.tar pingcap/tikv:latest
docker save -o tidb.tar pingcap/tidb:latest
docker save -o pd.tar pingcap/pd:latest

## Load images to each machine
docker load &lt; tikv.tar
docker load &lt; tidb.tar
docker load &lt; pd.tar
```

上記のコマンドを使用すると、ローカルのDockerレジストリにあるTiDBイメージを使用して最新のTiDBクラスタをデプロイでき、リモートリポジトリからイメージをプルする手間を省けます。この考え方は、前述のChaos Meshのインストールにも適用されます。どのイメージをプルする必要があるかわからない場合は、Helmを使用してChaos Meshをインストールし、インストールプロセスをトリガーした後、`kubectl describe`コマンドを使用して確認します：

```bash
## Check pods that are deployed in a specific namespace.
kubectl describe pods -n tidb-test
```

イメージのプルプロセスは通常、完了までに最も時間がかかります。Podがノードにスケジュールされている場合は、後で確認してください。

## カオス実験を実行

カオス実験を実行するには、まずYAMLファイルで定義し、`kubectl apply`を使用して開始する必要があります。この例では、PodChaosを使用してPodのクラッシュをシミュレートするカオス実験を作成しました。詳細な手順については、[カオス実験の実行](https://chaos-mesh.org/docs/run-a-chaos-experiment/)を参照してください。

## デイリーレポートの生成

### ログの収集

通常、TiDBクラスターでカオス実験を実行すると、多くのエラーが返されます。これらのエラーログを収集するには、`kubectl logs`コマンドを実行します：

```bash
kubectl logs &lt;podname> -n tidb-test --since=24h >> tidb.log
```

過去24時間に`tidb-test`ネームスペース内の特定のPodで生成されたすべてのログが`tidb.log`ファイルに保存されます。

### エラーと警告のフィルタリング

このステップでは、ログからエラーメッセージと警告メッセージをフィルタリングする必要があります。2つのオプションがあります：

- awkなどのテキスト処理ツールを使用する。これにはLinux/Unixコマンドの精通が必要です。
- スクリプトを書く。Linux/Unixコマンドに慣れていない場合、こちらがより良い選択肢です。

### プロットの作成

プロット作成には、Linuxコマンドラインのグラフ作成ユーティリティである[gnuplot](http://www.gnuplot.info/)を使用しました。以下の例では、負荷測定結果をインポートし、特定のPodが利用不能になったときに1秒あたりのクエリ数（QPS）がどのように影響を受けたかを示す折れ線グラフを作成しました。カオス実験が定期的に実行されたため、QPSの数値にはパターンが見られました：急激に低下し、その後すぐに正常に戻ります。

![QPS折れ線グラフ](/img/blog/qps-line-graph.png)

### PDF形式でのレポート生成

現在、Chaos Meshのレポート生成や結果分析のための利用可能なAPIはありません。私は、異なるブラウザで読み取り可能なPDF形式でレポートを生成することにしました。この場合、[gopdf](https://github.com/signintech/gopdf)というサポートライブラリを使用しました。これはPDFファイルを作成できるだけでなく、画像の挿入や表の描画も可能で、私のニーズに合致しました。

デイリーレポートを生成するために、[crond](https://www.linux.org/docs/man8/cron.html)というコマンドラインユーティリティを使用して、毎朝早くにコマンドを実行するように設定しました。これにより、仕事を始める時には既にデイリーレポートが準備されています。

## デイリーレポート用のWebアプリケーション構築

しかし、レポートをもっと読みやすく、アクセスしやすくしたいと考えました。Webアプリケーションでレポートを確認できる方が便利ではありませんか？最初は、バックエンドAPIを追加してレポートが生成された時間を追跡しようと考えました。適用可能ですが、私が求めているのはどのレポートがさらにトラブルシューティングを必要とするかを知ることだけなので、作業が多すぎるかもしれません。正確な情報はファイル名に表示されています。例えば、`report-2021-07-09-bad.pdf`です。これにより、レポートシステムの作業負荷と複雑さが大幅に軽減されます。

それでも、バックエンドインターフェースを改善し、レポート内容を充実させる必要があります。しかし、今のところ、毎日利用可能なレポートシステムで十分です。

私のケースでは、[Vue.js](https://github.com/vuejs/vue)を使用してWebアプリケーションを構築し、UIライブラリ[antd](https://www.antdv.com/docs/vue/introduce/)を使用しました。その後、自動生成されたレポートを静的リソースフォルダ`static`に保存することでページ内容を更新しました。これにより、Webアプリケーションが静的レポートを読み取り、フロントエンドページにレンダリングできるようになります。詳細については、[vue-cli 3でantdを使用する](https://www.antdv.com/docs/vue/use-with-vue-cli/)を参照してください。

以下は、私がデイリーレポート用に開発したWebアプリケーションの例です。赤いカードは、カオス実験を実行した後に例外がスローされたため、テストレポートを確認する必要があることを示しています。

![デイリーレポート用のWebアプリケーション](/img/blog/web-app-for-daily-reporting.png)

赤いカードをクリックすると、以下のようにレポートが開きます。PDFの閲覧には[pdf.js](https://github.com/mozilla/pdf.js)を使用しました。

![Daily report in PDF](/img/blog/daily-report-pdf.png)

## まとめ

Chaos Meshを使用すると、クラウドネイティブアプリケーションが遭遇する可能性のある障害をシミュレートできます。この記事では、PodChaos実験を作成し、Podが利用不能になった際にTiDBクラスターのQPSがどのように影響を受けるかを観察しました。ログを分析することで、システムの堅牢性と高可用性を向上させることができます。トラブルシューティングやデバッグのための日次レポート生成用のWebアプリケーションも構築しました。独自の要件に合わせてレポートをカスタマイズすることも可能です。

私たちのチームは、[TiDBをPostgreSQLと互換性を持たせるプロジェクト](https://github.com/DigitalChinaOpenSource/TiDB-for-PostgreSQL)にも取り組んでいます。興味があり、貢献したい方は、ぜひイシューを選んで参加してください。

**初出: _[The New Stack](https://thenewstack.io/develop-a-daily-reporting-system-for-chaos-mesh-to-improve-system-resilience/)_**