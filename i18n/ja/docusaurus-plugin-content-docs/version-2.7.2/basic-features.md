---
title: Basic Features
---

このドキュメントでは、Chaos Meshの基本機能について説明します。これには、[障害注入](#fault-injection)、[Chaosワークフロー](#chaos-workflows)、[可視化操作](#visualized-operations)、および[セキュリティ保証](#security-guarantees)が含まれます。

## 障害注入

障害注入はChaos実験の核心です。Chaos Meshは、分散システムで発生する可能性のあるあらゆる障害を網羅し、基本リソース障害、プラットフォーム障害、アプリケーション層障害という3つの包括的で詳細な障害タイプを提供します。

- 基本リソース障害:
  - [PodChaos](simulate-pod-chaos-on-kubernetes.md): Podの障害をシミュレートします。例えば、Podノードの再起動、Podの永続的な利用不可、特定のPod内のコンテナ障害など。
  - [NetworkChaos](simulate-network-chaos-on-kubernetes.md): ネットワーク障害をシミュレートします。例えば、ネットワーク遅延、パケット損失、パケット順序の乱れ、ネットワーク分断など。
  - [DNSChaos](simulate-dns-chaos-on-kubernetes.md): DNS障害をシミュレートします。例えば、DNSドメイン名の解決失敗や誤ったIPアドレスの返却など。
  - [HTTPChaos](simulate-http-chaos-on-kubernetes.md): HTTP通信障害をシミュレートします。例えば、HTTP通信の遅延など。
  - [StressChaos](simulate-heavy-stress-on-kubernetes.md): CPU競合やメモリ競合をシミュレートします。
  - [IOChaos](simulate-io-chaos-on-kubernetes.md): アプリケーションファイルのI/O障害をシミュレートします。例えば、I/O遅延、読み書き失敗など。
  - [TimeChaos](simulate-time-chaos-on-kubernetes.md): 時間ジャンプの異常をシミュレートします。
  - [KernelChaos](simulate-kernel-chaos-on-kubernetes.md): カーネル障害をシミュレートします。例えば、アプリケーションのメモリ割り当て異常など。
- プラットフォーム障害:
  - [AWSChaos](simulate-aws-chaos.md): AWSプラットフォームの障害をシミュレートします。例えば、AWSノードの再起動など。
  - [GCPChaos](simulate-gcp-chaos.md): GCPプラットフォームの障害をシミュレートします。例えば、GCPノードの再起動など。
- アプリケーション障害:
  - [JVMChaos](simulate-jvm-application-chaos.md): JVMアプリケーションの障害をシミュレートします。例えば、関数呼び出しの遅延など。

## Chaosワークフロー

Chaosワークフローには、一連のChaos実験とアプリケーション状態チェックが含まれており、プラットフォーム上でChaosエンジニアリングプロジェクトの全プロセスを完了できます。

Chaosワークフローを使用すると、一連のChaos実験を実行し、爆発半径（攻撃範囲を含む）を拡大し、障害タイプを増やすことができます。Chaosワークフローを実行した後、Chaos Meshを使用してアプリケーションの現在の状態を簡単に確認し、フォローアップ実験を実行するかどうかを判断できます。同時に、Chaosワークフローの維持コストを削減するために、既存の実験ワークフローを更新・蓄積し、他のワークフローに適用できます。

現在、Chaosワークフローは以下の機能を提供します:

- シリアルChaos実験のオーケストレーション
- パラレルChaos実験のオーケストレーション
- 実験状態と結果のチェックをサポート
- Chaos実験の一時停止をサポート
- YAMLファイルを使用したChaosワークフローの定義と管理をサポート
- Web UIを使用したChaosワークフローの定義と管理をサポート

特定のワークフローの設定については、[Chaos Meshワークフローの作成](create-chaos-mesh-workflow.md)を参照してください。

## 可視化操作

Chaos Meshは、可視化操作のためにChaos Dashboardコンポーネントを提供しており、Chaos実験を大幅に簡素化します。可視化インターフェースを通じて直接Chaos実験を管理および監視できます。例えば、インターフェース上で数回クリックするだけで、Chaos実験の範囲を定義し、Chaos注入のタイプを指定し、スケジューリングルールを定義し、Chaos実験の結果を取得できます。

![Chaosワークフロー](img/dashboard-overview.png)

## セキュリティ保証

Chaos Meshは、Kubernetesのネイティブな[RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)機能を使用して権限を管理します。

実際の権限要件に基づいて複数のロールを自由に作成し、それらのロールをユーザー名サービスアカウントにバインドできます。その後、サービスアカウントに対応するトークンを生成します。このトークンを使用してDashboardにログインすると、サービスアカウントに与えられた権限の範囲内でのみChaos実験を実行できます。

さらに、namespaceアノテーションを設定することでChaos実験を許可するnamespaceを指定でき、Chaos実験の制御をさらに強化できます。