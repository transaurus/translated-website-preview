---
slug: /run-chaos-experiments-on-physical-machines
title: 'How to run chaos experiments on your physical machine'
authors: xiangwang
image: /img/blog/chaosd-banner.png
tags: [Chaos Mesh, Chaos Engineering, Chaosd]
---

![物理マシンでカオス実験を実行する方法](/img/blog/chaosd-banner.png)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、Kubernetes環境でカオスをオーケストレーションするクラウドネイティブなカオスエンジニアリングプラットフォームです。Chaos Meshを使用すると、さまざまな障害をシミュレートし、Web UIであるChaos Dashboardで直接カオス実験を管理できます。オープンソース化以来、多くの企業がシステムの回復力と堅牢性を確保するためにChaos Meshを採用しています。しかし、過去1年間、コミュニティから「Kubernetesにデプロイされていないサービスでカオス実験を実行する方法」という要望が頻繁に寄せられてきました。

<!--truncate-->

## chaosdとは

物理マシンでのカオステストの需要が高まる中、私たちはchaosdという強化されたツールキットを発表できることを嬉しく思います。この名前は聞き覚えがあるかもしれません。それは、Chaos Meshの主要コンポーネントである`chaos-daemon`から進化したためです。TiDB Hackathon 2020では、[chaosdをコマンドラインツール以上のものにするためにリファクタリングしました](https://pingcap.com/blog/chaos-mesh-remake-one-step-closer-toward-chaos-as-a-service#refactor-chaosd)。現在、[chaosd v1.0.1](https://github.com/chaos-mesh/chaosd/releases/tag/v1.0.1)を使用すると、物理マシンを対象とした特定のエラーをシミュレートし、何も起こらなかったかのようにカオス実験を元に戻すことができます。

## chaosdの利点

chaosdには以下の利点があります:

- **使いやすさ**: chaosdコマンドで簡単にカオス実験を作成・管理できます。
- **多様な障害タイプ**: プロセス障害、ネットワーク障害、Java Virtual Machine (JVM)アプリケーション障害、ストレスシナリオ、ディスク障害、ホスト障害など、物理マシンに注入するさまざまなレベルの障害をシミュレートできます。
- **複数の動作モード**: chaosdをコマンドラインツールとして、またはサービスとして使用できます。

さっそく試してみましょう。

## chaosdの使用方法

このセクションでは、chaosdを使用してネットワーク障害を注入する方法を説明します。glibcのバージョンはv2.17以降である必要があります。

### 1. chaosdのダウンロードと解凍

chaosdをダウンロードするには、次のコマンドを実行します:

```bash
curl -fsSL -o chaosd-v1.0.1-linux-amd64.tar.gz https://mirrors.chaos-mesh.org/chaosd-v1.0.1-linux-amd64.tar.gz
```

ファイルを解凍します。以下の2つのフォルダが含まれています:

- `chaosd`: chaosdのツールエントリが含まれています。
- `tools`: カオス実験を実行するために必要なツールが含まれており、[stress-ng](https://wiki.ubuntu.com/Kernel/Reference/stress-ng)（ストレスシナリオのシミュレーション）、[Byteman](https://github.com/chaos-mesh/byteman)（JVMアプリケーション障害のシミュレーション）、PortOccupyTool（ネットワーク障害のシミュレーション）などがあります。

### 2. カオス実験の作成

このカオス実験では、サーバーがchaos-mesh.orgにアクセスできなくなります。

次のコマンドを実行します:

```bash
sudo ./chaosd attack network loss --percent 100 --hostname chaos-mesh.org --device ens33
```

出力例:

```bash
Attack network successfully, uid: c55a84c5-c181-426b-ae31-99c8d4615dbe
```

このシミュレーションでは、ens33ネットワークインターフェースカードが[chaos-mesh.org](http://chaos-mesh.org)との間でネットワークパケットを送受信できなくなります。`sudo`コマンドを使用する必要がある理由は、カオス実験がネットワークルールを変更するためで、これにはroot権限が必要です。

また、カオス実験の`uid`を忘れずに保存してください。後で回復プロセスの一部として入力します。

### 3. 結果の検証

`ping`コマンドを使用して、サーバーがchaos-mesh.orgにアクセスできるかどうかを確認します:

```bash
ping chaos-mesh.org
PING chaos-mesh.org (185.199.109.153) 56(84) bytes of data.
```

コマンドを実行すると、おそらくサイトから応答が得られないでしょう。`CTRL`+`C`を押してpingプロセスを停止してください。`ping`コマンドの統計情報として`100%パケットロス`が表示されるはずです。

出力例:

```bash
2 packets transmitted, 0 received, 100% packet loss, time 1021ms
```

### 4. 実験の復旧

実験を復旧するには、次のコマンドを実行します:

```bash
sudo ./chaosd recover c55a84c5-c181-426b-ae31-99c8d4615dbe
```

出力例:

```bash
Recover c55a84c5-c181-426b-ae31-99c8d4615dbe successfully
```

このステップでもroot権限が必要なため、`sudo`コマンドを使用する必要があります。実験の復旧が完了したら、再度chaos-mesh.orgにpingを実行して接続を確認してください。

## 次のステップ

### ダッシュボードWebのサポート

ご覧の通り、chaosdは非常に使いやすいツールです。しかし、さらに使いやすくするため、chaosd用のダッシュボードWebが現在積極的に開発中です。

私たちは使いやすさをさらに向上させ、chaosdで実行したカオス実験とChaos Meshで実行した実験を一元的に管理する機能など、より多くの機能を実装する予定です。これにより、Kubernetesと物理マシンでのカオステストにおいて一貫性のある統一されたユーザー体験を提供します。以下のアーキテクチャは単純な例です:

![Chaos Meshの最適化されたアーキテクチャ](/img/blog/chaos-mesh-optimized-architecture.png)

詳細については、[Chaos Meshの最適化されたアーキテクチャ](https://pingcap.com/blog/chaos-mesh-remake-one-step-closer-toward-chaos-as-a-service#developing-chaos-mesh-towards-caas)をご覧ください。

### 追加の障害注入タイプ

現在、chaosdは6種類の障害注入を提供しています。私たちはChaos Meshで既にサポートされているHTTPChaosやIOChaosなど、さらに多くのタイプを開発する予定です。

chaosdの改善に興味がある方は、ぜひ[issueを選んで](https://github.com/chaos-mesh/chaosd/labels/help%20wanted)取り組んでください！

## 試してみましょう！

chaosdの使用に興味があり、さらに探求したい方は、[ドキュメント](https://chaos-mesh.org/docs/chaosd-overview)をチェックしてください。chaosdの実行中に問題が発生した場合や機能リクエストがある場合は、遠慮なく[issueを作成](https://github.com/chaos-mesh/chaosd/issues)してください。皆さんの声をお待ちしています！