---
title: Chaosd Introduction
---

## Chaosdの紹介

[Chaosd](https://github.com/chaos-mesh/chaosd)は、Chaos Meshが提供するカオスエンジニアリングテストツールです。別途ダウンロードおよびデプロイが必要です（[ダウンロードとデプロイ](#download-and-deploy)参照）。物理マシン環境に障害を注入し、また障害を回復するために使用されます。

Chaosdには以下のコア強みがあります：

- 使いやすさ：Chaosdで簡単なコマンドを実行するだけで、カオス実験の作成と管理が可能です。
- 多様な障害タイプ：Chaosdは、プロセス、ネットワーク、負荷、ディスク、ホストなど、さまざまなレベルで物理マシンに注入可能な多様な障害タイプを提供します。さらに多くの障害タイプが追加予定です。
- 複数の動作モード：Chaosdは、コマンドラインツールとしてもサービスとしても使用可能で、さまざまなシナリオのニーズに対応します。

### サポートされている障害タイプ

Chaosdを使用して以下の障害タイプをシミュレートできます：

- プロセス：プロセスに障害を注入します。プロセスの強制終了や停止などの操作がサポートされています。
- ネットワーク：物理マシンのネットワークに障害を注入します。ネットワーク遅延の増加、パケット損失、パケット破損などの操作がサポートされています。
- 負荷：物理マシンのCPUまたはメモリに負荷をかけます。
- ディスク：物理マシンのディスクに障害を注入します。読み書きのディスク負荷増加やディスクの容量逼迫などの操作がサポートされています。
- ホスト：物理マシン自体に障害を注入します。物理マシンのシャットダウンなどの操作がサポートされています。

各障害タイプの詳細な紹介と使用方法については、関連ドキュメントを参照してください。

### 動作環境

glibcのバージョンはv2.17以降である必要があります。

### ダウンロードとデプロイ

1. ダウンロードするChaosdのバージョンを環境変数として設定します。例：v1.0.0：

   ```bash
   export CHAOSD_VERSION=v1.0.0
   ```

   Chaosdの全リリースバージョンを確認するには、[リリースページ](https://github.com/chaos-mesh/chaosd/releases)を参照してください。

   最新バージョン（安定版ではない）をダウンロードするには、`latest`を使用します：

   ```bash
   export CHAOSD_VERSION=latest
   ```

2. Chaosdをダウンロードします：

   ```bash
   curl -fsSLO https://mirrors.chaos-mesh.org/chaosd-$CHAOSD_VERSION-linux-amd64.tar.gz
   ```

3. Chaosdファイルを解凍し、`/usr/local`ディレクトリに移動します：

   ```bash
   tar zxvf chaosd-$CHAOSD_VERSION-linux-amd64.tar.gz && sudo mv chaosd-$CHAOSD_VERSION-linux-amd64 /usr/local/
   ```

4. Chaosdディレクトリを`PATH`環境変数に追加します：

   ```bash
   export PATH=/usr/local/chaosd-$CHAOSD_VERSION-linux-amd64:$PATH
   ```

### 動作モード

Chaosdは以下のモードで使用できます：

- コマンドラインモード：Chaosdを直接コマンドラインツールとして実行し、障害の注入と回復を行います。
- サービスモード：Chaosdをバックグラウンドサービスとして実行し、HTTPリクエストを送信することで障害の注入と回復を行います。