---
title: Simulate Host Faults
---

このドキュメントでは、Chaosdを使用してホストシャットダウンの障害をシミュレートする方法について説明します。

## ホストシャットダウン実験のヘルプ情報を表示

障害実験を作成する前に、以下のコマンドを実行してホストシャットダウン実験のヘルプ情報を確認できます：

```bash
chaosd attack host shutdown -h
```

出力は以下のようになります：

```bash
shutdowns system, this action will trigger shutdown of the host machine

Usage:
  chaosd attack host shutdown [flags]

Flags:
  -h, --help   help for shutdown

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

## ホストシャットダウン実験の作成

ホストシャットダウン実験を作成するには、以下のコマンドを実行します：

```bash
chaosd attack host shutdown
```

出力例は以下のようになります：

```bash
chaosd attack host shutdown
Shutdown successfully, uid: 4bc9b74a-5fe2-4038-b4f3-09ae95b57694
```

このコマンドを実行すると、すべてのプロセスが終了した後にホストがシャットダウンします。