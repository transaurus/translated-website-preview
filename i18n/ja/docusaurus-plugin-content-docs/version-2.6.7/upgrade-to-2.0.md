---
title: Upgrade to Chaos Mesh 2.0
---

このドキュメントでは、Chaos Meshを1.xから2.0へアップグレードするための詳細な手順を説明します。Chaos Mesh 2.0では新機能が追加され、多くの問題が修正されました。Chaos Mesh 2.0ではコードの一部が再構築されているため、アップグレードには追加の作業が必要です。

## アップグレードツール

Chaos Mesh 2.0ではCRDが変更されているため、以前のバージョンのChaos Mesh用に作成した実験用YAMLファイルは、Chaos Mesh 2.0では実行できません。これらのYAMLファイルを引き続き使用する場合は、YAMLファイルをエクスポートして更新し、Chaos Meshをアップグレードした後に再インポートする必要があります。

アップグレードプロセスを簡素化するため、Chaos Mesh 2.0では以下のアップグレードツールを提供しています：

- `migrate.sh`: YAMLファイルの自動エクスポートとアップグレード、CRDのアップグレード、アップグレード済みYAMLファイルのインポートを行います。
- `schedule-migration`: 古いバージョンのYAMLファイルを最新のYAMLファイルに更新します。

アップグレードツールを入手するには、Chaos Meshプロジェクトをローカルリポジトリにクローンし、`make schedule-migration.tar.gz`コマンドを実行することを推奨します。または、[https://mirrors.chaos-mesh.org/v2.0.0/schedule-migration.tar.gz](https://mirrors.chaos-mesh.org/v2.0.0/schedule-migration.tar.gz)からプロジェクトをダウンロードすることもできます。`tar.gz`パッケージをダウンロード後、以下のコマンドを実行すると上記2つのアップグレードツールが入手できます：

```bash
tar xvf ./schedule-migration.tar.gz
```

このパッケージに含まれる`schedule-migration`ツールはLinux x86_64プラットフォーム専用です。他のOSやアーキテクチャを使用している場合は、自分でコードをコンパイルする必要があります。

## ステップ1: 実験のエクスポートとアップグレード

アップグレードツール`migrate.sh`を使用して、実験を自動的にエクスポートおよびアップグレードできます。実行前に、クラスタにアクセスする十分な権限があることを確認してください。

`migrate.sh`が現在のディレクトリにある場合、`schedule-migration`ツールも同じディレクトリに配置します。その後、以下のコマンドを実行して実験をエクスポートおよびアップグレードします：

```bash
bash migrate.sh -e
```

すると、現在のディレクトリに多数のYAMLファイルが生成されます。これらはエクスポートされた実験ファイルで、自動的にアップグレードされています。

また、`schedule-migration`ツールを使用して、指定した古いバージョンのYAMLファイルをアップグレードすることもできます：

```bash
./schedule-migration <path-to-old-yaml> <path-to-new-yaml>
```

指定したYAMLファイルのパスに、アップグレード済みのYAMLファイルが生成されます。古いリソースを削除した後、新しいYAMLファイルを再適用して更新プロセスを完了します。

## ステップ2: CRDのアップグレード

Helmを使用してChaos Meshをアップグレードする前に、アップグレードの成功率を高めるため、以下のコマンドを実行して手動でCRDをアップグレードします：

```bash
bash migrate.sh -c
```

CRDが削除され、再追加されることが確認できます。

## ステップ3: Chaos Meshのアップグレード

Chaos Mesh 2.0には1.xからの多くの変更が含まれているため、まず1.xをアンインストールしてから2.0をインストールすることを推奨します。具体的なインストール手順については、[Helmを使用したインストール（本番環境推奨）](production-installation-using-helm.md)を参照してください。

## ステップ4: 実験のインポート

Chaos Mesh 2.0では実験定義にいくつかの変更が加えられています。引き続き使用する前に、以前のバージョンの実験ファイルをアップグレードする必要があります。

エクスポートした実験ファイルと同じディレクトリで、以下のコマンドを実行して実験をインポートします：

```bash
bash migrate.sh -i
```

## 問題の報告

アップグレードプロセスで問題が発生した場合は、コマンドの出力を[Slack](https://cloud-native.slack.com/archives/C0193VAV272)に投稿するか、[GitHub](https://github.com/pingcap/chaos-mesh/issues)でイシューを作成してください。フィードバックをお待ちしております。Chaos Meshチームは喜んで問題解決をサポートいたします。