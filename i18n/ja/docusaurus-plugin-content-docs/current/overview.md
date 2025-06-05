---
slug: /
title: Chaos Mesh Overview
---

このドキュメントでは、Chaos Meshのコンセプト、ユースケース、コア強み、およびアーキテクチャについて説明します。

## Chaos Meshの概要

Chaos Meshは、オープンソースのクラウドネイティブなカオスエンジニアリングプラットフォームです。さまざまな種類の障害シミュレーションを提供し、障害シナリオをオーケストレーションする強力な能力を備えています。

Chaos Meshを使用すると、開発、テスト、本番環境で実際に発生する可能性のあるさまざまな異常を簡単にシミュレートし、システム内の潜在的な問題を発見できます。カオスエンジニアリングプロジェクトの参入障壁を下げるため、Chaos Meshは視覚的な操作を提供します。Web UI上で簡単にカオスシナリオを設計し、カオス実験の状態を監視できます。

## コア強み

業界をリードするカオステストプラットフォームとして、Chaos Meshには以下のコア強みがあります：

- 安定したコア機能：Chaos Meshは[TiDB](https://github.com/pingcap/tidb)のコアテストプラットフォームから派生し、初期リリース時からTiDBの既存のテスト経験を多く継承しています。
- 完全に検証済み：Chaos MeshはTencentやMeituanなど多くの企業や組織で使用されており、Apache APISIXやRabbitMQなど有名な分散システムのテストシステムでも採用されています。
- 使いやすいシステム：Chaos Meshは自動化を最大限に活用し、グラフィカル操作とKubernetesベースの使用法を提供します。
- クラウドネイティブ：Chaos MeshはKubernetes環境をサポートし、強力な自動化能力を備えています。
- 多様な障害シミュレーションシナリオ：Chaos Meshは分散テストシステムにおける基本的な障害シミュレーションのほとんどのシナリオをカバーしています。
- 柔軟な実験オーケストレーション能力：プラットフォーム上で独自のカオス実験シナリオを設計可能で、複数の混合実験やアプリケーション状態チェックを含みます。
- 高いセキュリティ：Chaos Meshは多層のセキュリティ制御を設計し、高い安全性を提供します。
- 活発なコミュニティ：Chaos MeshはCNCFがホストするインキュベーションプロジェクトで、世界中で[コントリビューター](https://github.com/chaos-mesh/chaos-mesh/graphs/contributors)と[採用企業](https://github.com/chaos-mesh/chaos-mesh/blob/master/ADOPTERS.md)が増加しています。
- 容易な拡張性：Chaos Meshに新しい障害テストタイプや機能を簡単に追加できます。

## アーキテクチャ概要

Chaos MeshはKubernetes CRD（Custom Resource Definition）上に構築されています。さまざまなカオス実験を管理するため、Chaos Meshは異なる障害タイプに基づいて複数のCRDタイプを定義し、異なるCRDオブジェクトに対して個別のコントローラーを実装しています。Chaos Meshは主に3つのコンポーネントで構成されます：

- **Chaos Dashboard**：Chaos Meshの可視化コンポーネント。Chaos DashboardはユーザーフレンドリーなWebインターフェースを提供し、ユーザーがカオス実験を操作・観察できます。同時に、RBAC権限管理メカニズムも提供します。
- **Chaos Controller Manager**：Chaos Meshのコア論理コンポーネント。Chaos Controller Managerは主にカオス実験のスケジューリングと管理を担当します。このコンポーネントには、Workflow Controller、Scheduler Controller、各種障害タイプのコントローラーなどが含まれます。
- **Chaos Daemon**：主要な実行コンポーネント。Chaos Daemonは[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)モードで実行され、デフォルトでPrivileged権限を持ちます（無効化可能）。このコンポーネントは主に、ターゲットPodのNamespaceに侵入して特定のネットワークデバイス、ファイルシステム、カーネルに干渉します。

![アーキテクチャ](img/architecture.png)

上記の画像に示すように、Chaos Meshの全体アーキテクチャは上から下まで3つの部分に分けられます：

- ユーザー入力と観察：ユーザー操作（User）から始まり、ユーザー入力はKubernetes API Serverに到達します。ユーザーはChaos Controller Managerと直接やり取りせず、すべてのユーザー操作は最終的にChaosリソースの変更（NetworkChaosリソースの変更など）として反映されます。
- リソース変更の監視、Workflowのスケジューリング、カオス実験の実行：Chaos Controller ManagerはKubernetes API Serverからのイベントのみを受け取ります。これらのイベントは、新しいWorkflowオブジェクトやChaosオブジェクトの作成など、特定のChaosリソースの変更を記述します。
- 特定のノード障害の注入：Chaos Daemonコンポーネントは主にChaos Controller Managerコンポーネントからのコマンドを受け取り、ターゲットPodのNamespaceに侵入して特定の障害注入を実行します。例えば、TCネットワークルールの設定、stress-ngプロセスの起動によるCPUやメモリリソースの奪取などです。