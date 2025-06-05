---
slug: /chaos_mesh_your_chaos_engineering_solution
title: Chaos Mesh - Your Chaos Engineering Solution for System Resiliency on Kubernetes
authors: cwen
image: /img/blog/chaos-engineering.png
tags: [Chaos Mesh, Chaos Engineering, Kubernetes]
---

![カオスエンジニアリング](/img/blog/chaos-engineering.png)

## Chaos Meshを選ぶ理由

分散コンピューティングの世界では、クラスタに予期せぬ障害がいつどこで発生するかわかりません。従来のユニットテストや統合テストでは、システムが本番環境に対応可能であることを保証できますが、クラスタのスケール拡大、複雑性の増加、PBレベルのデータ量増加に伴い、これらは氷山の一角に過ぎません。システムの脆弱性をよりよく特定し、レジリエンスを向上させるため、Netflixは[Chaos Monkey](https://netflix.github.io/chaosmonkey/)を開発し、インフラストラクチャや業務システムにさまざまな種類の障害を注入しました。これがカオスエンジニアリングの起源です。

<!--truncate-->

[PingCAP](https://chaos-mesh.org/)では、オープンソースの分散型NewSQLデータベース[TiDB](https://github.com/pingcap/tidb)を構築する際に同じ課題に直面しました。データベースユーザーにとって最も重要な資産であるデータそのものが危険にさらされるため、フォールトトレランスやレジリエンスは特に重要です。レジリエンスを確保するため、私たちは初期段階からテストフレームワーク内で[カオスエンジニアリングを実践](https://pingcap.com/blog/chaos-practice-in-tidb/)してきました。しかし、TiDBが成長するにつれ、テスト要件も増加しました。TiDBだけでなく、他の分散システムにも対応できる汎用的なカオステストプラットフォームが必要であることに気づきました。

そこで、Kubernetes環境でカオス実験をオーケストレーションするクラウドネイティブなカオスエンジニアリングプラットフォーム、Chaos Meshを発表します。これはオープンソースプロジェクトで、[https://github.com/chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)から利用可能です。

以下のセクションでは、Chaos Meshの概要、設計と実装方法、そして実際の環境での使用方法について説明します。

## Chaos Meshでできること

Chaos Meshは、Kubernetes上の複雑なシステム向けに包括的な障害注入方法を備えた多機能なカオスエンジニアリングプラットフォームです。Pod、ネットワーク、ファイルシステム、さらにはカーネルに至るまでの障害をカバーします。

以下は、Chaos Meshを使用してTiDBシステムのバグを特定した例です。この例では、分散ストレージエンジン([TiKV](https://docs.pingcap.com/tidb/stable/tidb-architecture#tikv-server))のPodダウンタイムをシミュレートし、1秒あたりのクエリ数(QPS)の変化を観察しました。通常、1つのTiKVノードがダウンしても、QPSは一時的なジッターを経験した後、障害前のレベルに戻ります。これが高可用性を保証する仕組みです。

![Chaos MeshがTiKVのダウンタイム回復異常を発見](/img/blog/chaos-mesh-discovers-downtime-recovery-exceptions-in-tikv.png)

ダッシュボードからわかるように:

- 最初の2回のダウンタイムでは、QPSは約1分で正常に戻ります。
- しかし、3回目のダウンタイム後、QPSの回復には約9分かかりました。このような長いダウンタイムは予期せぬもので、オンラインサービスに確実に影響を与えます。

調査の結果、テスト対象のTiDBクラスタバージョン(V3.0.1)には、TiKVダウンタイム処理時に微妙な問題があることが判明しました。これらの問題は後のバージョンで解決されました。

Chaos Meshはダウンタイムのシミュレーション以上の機能を備えています。以下の障害注入方法が含まれます:

- **pod-kill:** Kubernetes Podの強制終了をシミュレート
- **pod-failure:** Kubernetes Podの継続的な利用不可をシミュレート
- **network-delay:** ネットワーク遅延をシミュレート
- **network-loss:** ネットワークパケット損失をシミュレート
- **network-duplication:** ネットワークパケット重複をシミュレート
- **network-corrupt:** ネットワークパケット破損をシミュレート
- **network-partition:** ネットワーク分断をシミュレート
- **I/O delay:** ファイルシステムI/O遅延をシミュレート
- **I/O errno:** ファイルシステムI/Oエラーをシミュレート

## 設計原則

Chaos Meshは、使いやすさ、拡張性、Kubernetes向け設計をコンセプトに設計されています。

### 使いやすさ

Chaos Meshを簡単に利用するためには、以下の要件を満たす必要があります:

- 特別な依存関係を必要とせず、[Minikube](https://github.com/kubernetes/minikube)を含むKubernetesクラスターに直接デプロイ可能であること。
- テスト対象システム（SUT）のデプロイロジックを変更せずに、本番環境でカオス実験を実行できること。
- カオス実験における障害注入の動作を容易に調整でき、実験の状態と結果を簡単に確認できること。また、注入された障害を迅速にロールバックできること。
- 実装の詳細を隠蔽し、ユーザーがカオス実験の調整に集中できること。

### 拡張性

Chaos Meshは拡張可能であるべきで、新しい要件を簡単に「プラグイン」できるようにする必要があります。具体的には、以下の要件を満たす必要があります:

- 既存の実装を活用し、障害注入方法を簡単に拡張できること。
- 他のテストフレームワークと容易に統合できること。

### Kubernetes向け設計

コンテナの世界において、Kubernetesは絶対的なリーダーです。その採用率の成長は誰の予想もはるかに超えており、コンテナオーケストレーションの戦いに勝利しました。本質的に、Kubernetesはクラウドのためのオペレーティングシステムです。

TiDBはクラウドネイティブな分散データベースです。私たちの内部自動テストプラットフォームは最初からKubernetes上に構築されました。毎日、さまざまな実験のために数百のTiDBクラスターがKubernetes上で実行されており、本番環境で発生するあらゆる種類の障害や問題をシミュレートするための広範なカオステストも含まれています。これらのカオス実験をサポートするために、カオスとKubernetesの組み合わせは私たちの実装における自然な選択であり、原則となりました。

## CustomResourceDefinitionsの設計

Chaos Meshは[CustomResourceDefinitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)（CRD）を使用してカオスオブジェクトを定義します。Kubernetesの領域において、CRDはカスタムリソースを実装するための成熟したソリューションであり、豊富な実装事例とツールセットが利用可能です。CRDを使用することで、Chaos MeshはKubernetesエコシステムと自然に統合されます。

すべての種類の障害注入を統一されたCRDオブジェクトで定義する代わりに、異なる種類の障害注入に対して柔軟で個別のCRDオブジェクトを許可します。既存のCRDオブジェクトに準拠する障害注入方法を追加する場合、このオブジェクトを直接拡張します。まったく新しい方法の場合は、新しいCRDオブジェクトを作成します。この設計により、カオスオブジェクトの定義とロジック実装がトップレベルから抽出され、コード構造がより明確になります。このアプローチは結合度を低下させ、エラーの発生確率を減らします。さらに、Kubernetesの[controller-runtime](https://github.com/kubernetes-sigs/controller-runtime)はコントローラーを実装するための優れたラッパーであり、各CRDプロジェクトに対して同じコントローラーセットを繰り返し実装する必要がないため、多くの時間を節約できます。

Chaos MeshはPodChaos、NetworkChaos、IOChaosオブジェクトを実装しています。これらの名前は対応する障害注入タイプを明確に識別します。

たとえば、PodのクラッシュはKubernetes環境で非常に一般的な問題です。多くのネイティブリソースオブジェクトは、新しいPodを作成するなどの典型的なアクションでこのようなエラーを自動的に処理します。しかし、私たちのアプリケーションは本当にこのようなエラーに対処できるでしょうか？Podが起動しない場合はどうなるでしょうか？

`pod-kill`などの明確に定義されたアクションを使用することで、PodChaosはこの種の問題をより効果的に特定するのに役立ちます。PodChaosオブジェクトは以下のコードを使用します:

```yml
spec:
 action: pod-kill
 mode: one
 selector:
   namespaces:
     - tidb-cluster-demo
   labelSelectors:
     "app.kubernetes.io/component": "tikv"
  scheduler:
   cron: "@every 2m"
```

このコードは以下のことを行います:

- `action`属性は注入する特定のエラータイプを定義します。この場合、`pod-kill`はPodをランダムに強制終了します。
- `selector`属性はカオス実験の範囲を特定の範囲に制限します。この場合、範囲は`tidb-cluster-demo`ネームスペースのTiDBクラスターのTiKV Podです。
- `scheduler`属性は各カオス障害アクションの間隔を定義します。

NetworkChaosやIOChaosなどのCRDオブジェクトの詳細については、[Chaos-meshドキュメント](https://github.com/chaos-mesh/chaos-mesh)を参照してください。

## Chaos Meshの動作原理

CRDの設計が決まったところで、Chaos Meshがどのように動作するかの全体像を見てみましょう。以下の主要なコンポーネントが関与しています：

- **controller-manager**

  プラットフォームの「頭脳」として機能します。CRDオブジェクトのライフサイクルを管理し、カオス実験をスケジュールします。CRDオブジェクトインスタンスをスケジュールするためのオブジェクトコントローラーと、[admission-webhooks](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)コントローラーが動的にサイドカーコンテナをPodに注入します。

- **chaos-daemon**

  特権を持つDaemonSetとして実行され、ノード上のネットワークデバイスやCgroupを操作できます。

- **sidecar**

  admission-webhooksによってターゲットPodに動的に注入される特殊なタイプのコンテナとして実行されます。例えば、`chaosfs`サイドカーコンテナはfuse-daemonを実行してアプリケーションコンテナのI/O操作をハイジャックします。

![Chaos Meshのワークフロー](/img/blog/chaos-mesh-workflow.png)

これらのコンポーネントがカオス実験をどのように効率化するかは以下の通りです：

1. ユーザーはYAMLファイルまたはKubernetesクライアントを使用して、Kubernetes APIサーバーにカオスオブジェクトを作成または更新します。
2. Chaos MeshはAPIサーバーを使用してカオスオブジェクトを監視し、作成、更新、削除イベントを通じてカオス実験のライフサイクルを管理します。このプロセスでは、controller-manager、chaos-daemon、およびサイドカーコンテナが協力してエラーを注入します。
3. admission-webhooksがPod作成リクエストを受信すると、作成されるPodオブジェクトが動的に更新されます。例えば、サイドカーコンテナが注入されます。

## カオスの実行

これまでのセクションでは、Chaos Meshの設計と動作原理を紹介しました。ここからは実際にChaos Meshを使用する方法について説明します。カオステストの時間は、テスト対象のアプリケーションの複雑さやCRDで定義されたテストスケジュールルールによって異なることに注意してください。

### 環境の準備

Chaos MeshはKubernetes v1.12以降で動作します。Kubernetesのパッケージ管理ツールであるHelmを使用して、Chaos Meshをデプロイおよび管理します。Chaos Meshを実行する前に、KubernetesクラスターにHelmが正しくインストールされていることを確認してください。環境をセットアップするには、以下の手順に従います：

1. Kubernetesクラスターが利用可能であることを確認してください。すでにある場合は手順2に進みます。ない場合は、Chaos Meshが提供するスクリプトを使用してローカルにクラスターを起動します：

   ```bash
   // kindのインストール
   curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.6.1/kind-$(uname)-amd64
   chmod +x ./kind
   mv ./kind /some-dir-in-your-PATH/kind

   // スクリプトの取得
   git clone https://github.com/chaos-mesh/chaos-mesh
   cd chaos-mesh
   // クラスターの起動
   hack/kind-cluster-build.sh
   ```

   **注記：** ローカルでKubernetesクラスターを起動すると、ネットワーク関連の障害注入に影響を与える可能性があります。

2. Kubernetesクラスターが準備できたら、[Helm](https://helm.sh/)と[Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)を使用してChaos Meshをデプロイします：

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh.git
   cd chaos-mesh
   // CRDリソースの作成
   kubectl apply -f manifests/
   // chaos-meshのインストール
   helm install helm/chaos-mesh --name=chaos-mesh --namespace=chaos-mesh
   ```

   すべてのコンポーネントがインストールされるまで待ち、以下のコマンドでインストール状態を確認します：

   ```bash
   // chaos-meshの状態確認
   kubectl get pods --namespace chaos-mesh -l app.kubernetes.io/instance=chaos-mesh
   ```

   インストールが成功すると、すべてのPodが起動していることが確認できます。これで準備は完了です。

   YAML定義またはKubernetes APIを使用してChaos Meshを実行できます。

### YAMLファイルを使用したカオスの実行

YAMLファイル方式を使用して独自のカオス実験を定義できます。これは、アプリケーションをデプロイした後にカオス実験を迅速かつ便利に実行する方法を提供します。YAMLファイルを使用してカオスを実行するには、以下の手順に従います：

**注:** 説明のため、ここではテスト対象システムとしてTiDBを使用しています。お好みのターゲットシステムを使用し、YAMLファイルを適宜変更してください。

1. `chaos-demo-1`という名前のTiDBクラスターをデプロイします。[TiDB Operator](https://github.com/pingcap/tidb-operator)を使用してTiDBをデプロイできます。
2. `kill-tikv.yaml`という名前のYAMLファイルを作成し、以下の内容を追加します:

   ```yml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: PodChaos
   metadata:
     name: pod-kill-chaos-demo
     namespace: chaos-mesh
   spec:
     action: pod-kill
     mode: one
     selector:
       namespaces:
         - chaos-demo-1
       labelSelectors:
         'app.kubernetes.io/component': 'tikv'
     scheduler:
       cron: '@every 1m'
   ```

3. ファイルを保存します。
4. カオスを開始するには、`kubectl apply -f kill-tikv.yaml`を実行します。

以下のカオス実験は、`chaos-demo-1`クラスター内のTiKV Podが頻繁にkillされる状況をシミュレートします:

![カオス実験の実行](/img/blog/chaos-experiment-running.gif)

sysbenchプログラムを使用してTiDBクラスターのリアルタイムQPS変化を監視しています。クラスターにエラーが注入されると、QPSに激しい変動が見られ、これは特定のTiKV Podが削除され、Kubernetesが新しいTiKV Podを再作成したことを意味します。

より多くのYAMLファイルの例については、https://github.com/chaos-mesh/chaos-mesh/tree/master/examples を参照してください。

### Kubernetes APIを使用したカオスの実行

Chaos MeshはCRDを使用してカオスオブジェクトを定義しているため、Kubernetes APIを直接操作してCRDオブジェクトを操作できます。この方法により、カスタマイズされたテストシナリオと自動化されたカオス実験を使用して、独自のアプリケーションにChaos Meshを簡単に適用できます。

[test-infra](https://github.com/pingcap/tipocket/tree/35206e8483b66f9728b7b14823a10b3e4114e0e3/test-infra)プロジェクトでは、Kubernetes上の[etcd](https://github.com/pingcap/tipocket/blob/35206e8483b66f9728b7b14823a10b3e4114e0e3/test-infra/tests/etcd/nemesis_test.go)クラスターで発生する可能性のあるエラー（ノードの再起動、ネットワーク障害、ファイルシステム障害など）をシミュレートしています。

以下はKubernetes APIを使用したChaos Meshのサンプルスクリプトです:

```go
import (
  "context"

  "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
  "sigs.k8s.io/controller-runtime/pkg/client"
)

func main() {
  // ...
  delay := &chaosv1alpha1.NetworkChaos{
    Spec: chaosv1alpha1.NetworkChaosSpec{
      // ...
    },
  }
  k8sClient := client.New(conf, client.Options{ Scheme: scheme.Scheme })
  k8sClient.Create(context.TODO(), delay)
  k8sClient.Delete(context.TODO(), delay)
}
```

## 今後の展望

本記事では、オープンソースのクラウドネイティブなカオスエンジニアリングプラットフォームであるChaos Meshを紹介しました。まだ進行中の部分が多く、設計、ユースケース、開発に関する詳細はこれから明らかになる予定です。ご期待ください。

オープンソース化は始まりに過ぎません。これまでに紹介したインフラストラクチャレベルのカオス実験に加えて、より細かい粒度での幅広い障害タイプのサポートを進めています。例えば:

- eBPFなどのツールを活用したシステムコールレベルやカーネルレベルでのエラー注入
- [failpoint](https://github.com/pingcap/failpoint)の統合によるアプリケーションの関数レベルやステートメントレベルでの特定のエラー注入（従来の注入方法では不可能なシナリオをカバー）

今後は、Chaos Meshダッシュボードの改善を継続し、ユーザーがオンラインビジネスに障害注入がどのような影響を与えるかを簡単に確認できるようにします。さらに、使いやすい障害オーケストレーションインターフェースの導入を予定しています。Chaos Mesh VerifierやChaos Mesh Cloudなど、他にも魅力的な機能を計画中です。

これらの取り組みに興味を持たれた方は、ぜひ世界クラスのカオスエンジニアリングプラットフォーム構築にご参加ください。Kubernetes上で私たちのアプリケーションがカオスの中で踊る日を！

バグを見つけたり、何か不足していると思われる場合は、遠慮なく[issue](https://github.com/chaos-mesh/chaos-mesh/issues)を登録したり、PRを送ったり、[CNCF Slack](https://slack.cncf.io/)ワークスペースの#project-chaos-meshチャンネルでメッセージを送ってください。

GitHub: [https://github.com/chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)