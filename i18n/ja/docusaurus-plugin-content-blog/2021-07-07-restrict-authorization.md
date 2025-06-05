---
slug: /securing-tenant-namespaces-using-restrict-authorization-feature
title: 'Securing tenant namespaces using restrict authorization feature in Chaos Mesh'
authors: anuragpaliwal
image: /img/blog/chaos-engineering-tools-as-a-service.jpeg
tags: [Chaos Mesh, Chaos Engineering]
---

![カオスエンジニアリングツール](/img/blog/chaos-mesh-restrict-authorization.jpeg)

[マルチテナント](https://cloud.google.com/kubernetes-engine/docs/concepts/multitenancy-overview)クラスターは、複数のユーザーやワークロード（「テナント」と呼ばれる）によって共有されます。マルチテナントクラスターの運用者は、テナント間を分離し、侵害されたまたは悪意のあるテナントがクラスターや他のテナントに与える損害を最小限に抑える必要があります。

<!--truncate-->

## クラスターのマルチテナンシー

マルチテナントアーキテクチャを計画する際には、Kubernetesにおけるリソース分離の層（クラスター、ネームスペース、ノード、Pod、コンテナ）を考慮する必要があります。

Kubernetesはテナント間の完全に安全な分離を保証することはできませんが、特定のユースケースに十分な機能を提供しています。各テナントとそのKubernetesリソースを独自の[ネームスペース](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)に分離できます。Kubernetesは、同じ物理クラスターをバックエンドとする複数の仮想クラスターをサポートしています。これらの仮想クラスターはネームスペースと呼ばれます。[ネームスペース](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)は、多くのユーザーが複数のチームやプロジェクトに分散している環境での使用を想定しています。

## Chaos Meshを導入したクラスター

Kubernetesクラスターを設計し、複数のテナントサービスを実行するようにしました。Kubernetesのセキュリティベストプラクティスに従っています：各テナントサービスは独自のネームスペースで実行され、これらのテナントサービスのユーザーには、それぞれのネームスペースのみに対する適切なアクセス権が与えられています。

<!--truncate-->

クラスターにChaos Mesh（[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、Kubernetes環境でカオスをオーケストレーションするクラウドネイティブなカオスエンジニアリングプラットフォームです）を有効にし、テナントサービスがさまざまなカオスアクティビティを実行してアプリケーション/システムの回復力を確認できるようにしました。また、テナントサービスのユーザーにChaos Meshリソースを管理するための特定の権限を[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)を使用して付与しています。

<!--truncate-->

あるテナントユーザーが、自身のネームスペース（例：chaos-mesh）でPod kill操作を実行したいとします。これを実現するために、ユーザーは以下のChaos Mesh YAMLファイルを作成しました：

```yml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-kill
  namespace: chaos-mesh
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - tidb-cluster-demo
    labelSelectors:
      'app.kubernetes.io/component': 'tikv'
  scheduler:
    cron: '@every 1m'
```

ユーザーはchaos-meshネームスペースに対する必要な権限を持っていますが、tidb-cluster-demoネームスペースに対する権限は持っていません。ユーザーが上記のYAMLファイルをkubectlで適用すると、chaos-meshネームスペースにpod-kill Chaos Meshリソースが作成されます。セレクターセクションで指定されているように、ユーザーは別のネームスペース（tidb-cluster-demo）を指定しており、このカオス操作で選択されるPodは、ユーザーがアクセス権を持つchaos-meshではなく、tidb-cluster-demoネームスペースからのものになります。これは、ユーザーが権限を持たない他のネームスペースに影響を与えることができることを意味します。**問題発生!!!**

<!--truncate-->

Chaos Mesh 1.1.3のリリース以降、このセキュリティ問題は制限付き認可機能によって修正されました。現在、ユーザーが上記のYAMLファイルを適用すると、システムは以下のようなエラーを表示します：

```yml
Error when creating "pod/pod-kill.yaml": admission webhook "vauth.kb.io" denied the request: ... is forbidden on namespace
tidb-cluster-demo
```

**問題解決!**

ユーザーがtidb-cluster-demoネームスペースに対しても必要な権限を持っている場合、このようなエラーは発生しないことに注意してください。

## さらに学ぶには

ユーザーがネームスペースをまたがってカオスを作成することを一切許可しないように強制したい場合は、私の以前のブログ「[OPAを使用してChaos Meshを使用しながらテナントサービスを保護する](https://anuragpaliwal-93749.medium.com/securing-tenant-services-while-using-chaos-mesh-using-opa-3ae80c7f4b85)」を参照してください。

## 最後に

Chaos Meshに興味があり、さらに詳しく知りたい場合は、[Slackチャンネル](https://slack.cncf.io/)に参加するか、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやイシューを投稿してください。