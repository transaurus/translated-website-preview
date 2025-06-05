---
title: Chaosctl
---

Chaosctlは、Chaos Meshのデバッグを支援するツールです。Chaosctlを使用することで、新しいカオスタイプの開発とデバッグのプロセスを簡素化し、他の開発者がIssueを提起する際の参考情報を提供できます。

## Chaosctlの取得

Linuxユーザーの場合、Chaosctlの実行ファイルを直接ダウンロードできます。

```bash
curl -sSL https://mirrors.chaos-mesh.org/latest/chaosctl -O
```

WindowsまたはmacOSユーザーの場合、ソースコードからコンパイルできます。コンパイルにはGo v1.15以上を推奨します。以下の手順を実行してください：

1. Chaos Meshリポジトリをローカルマシンにクローンします。

   ```bash
   git clone https://github.com/chaos-mesh/chaos-mesh.git
   ```

2. Chaos Meshディレクトリに移動します。

3. 以下のコマンドを実行します：

   ```bash
   make chaosctl
   ```

   コンパイルされた実行ファイルは`bin/chaosctl`にあります。

## 機能

現在、ChaosctlはChaos実験のログとデバッグ情報の出力をサポートしています。

### ログの出力

すべてのChaos Meshコンポーネントのログを出力するには、`chaosctl logs`コマンドを使用します。この機能のヘルプ情報と例を確認するには、`chaosctl logs -h`コマンドを使用してください。コマンドの例は以下の通りです：

```bash
chaosctl logs -t 100 # Print the last 100 lines of logs from all components
```

### デバッグ情報の出力

デバッグ情報を出力するには、`chaosctl debug`コマンドを使用します。この機能のヘルプ情報と例を確認するには、`chaosctl debug -h`コマンドを使用してください。デバッグ時には、Chaosctlが対応する`chaos-daemon`に接続されていることを確認する必要があります。Chaos Meshのデプロイ時にTLSを無効にした場合（デフォルトで有効）、`-i`オプションを追加してChaosctlにTLSが使用されていないことを伝えてください。コマンドの例は以下の通りです：

```bash
./chaosctl debug -i networkchaos web-show-network-delay
```

現在、ChaosctlはIOChaos、NetworkChaos、およびStressChaosのデバッグのみをサポートしています。

### Chaosd用TLS証明書の生成

ChaosdとChaos Mesh間でリクエストが開始される際、ChaosdとChaos-controller-managerサービス間の通信セキュリティを確保するために、Chaos MeshはmTLS（Mutual Transport Layer Security）モードの有効化を推奨しています。

mTLSモードを有効にするには、ChaosdとChaos MeshでTLS証明書パラメータを設定する必要があります。そのため、ChaosdとChaos MeshがTLS証明書を生成していることを確認した上で、TLS証明書をパラメータとして指定してChaosdとChaos Meshを起動してください。

- Chaosd: TLS証明書パラメータを設定する**前または後**にChaosdを起動できます。クラスタのセキュリティのため、TLS証明書パラメータを先に設定してからChaosdを起動することを推奨します。詳細は[Chaosdサーバーのデプロイ](simulate-physical-machine-chaos.md#deploy-chaosd-server)を参照してください。
- Chaos Mesh: Helmを使用してChaos Meshをデプロイした場合、TLS証明書パラメータはデフォルトで設定されています。

ChaosdがTLS証明書を生成していない場合、Chaosctlを使用してコマンドラインから簡単に証明書を生成できます。以下の使用例では、Chaosctlが異なるスキームを通じてコマンドを実行します。

**ケース1**: Chaosctlを実行するノードがKubernetesクラスタにアクセス可能で、SSHツールを使用して物理マシンに接続できる場合。

以下のコマンドを実行して操作を完了します：

- コマンド: `chaosctl pm init`コマンドを使用します：

  ```bash
  ./chaosctl pm init pm-name --ip=123.123.123.123 -l arch=amd64,anotherkey=value
  ```

- 操作: このコマンドは以下の操作を実行します。
  - Chaosdに必要な証明書を簡単に生成し、対応する物理マシンに保存します。
  - Kubernetesクラスタに対応する`PhysicalMachine`リソースを作成します。

この機能の詳細と例については、`chaosctl pm init -h`を参照してください。

**ケース2**: Chaosctlを実行するノードがKubernetesクラスタにアクセス可能だが、SSHツールを使用して物理マシンに接続できない場合。

以下のコマンドを実行して操作を完了します:

1. コマンドを実行する前に、KubernetesクラスターからCA証明書を手動で取得する必要があります。例:

   ```bash
   kubectl get secret chaos-mesh-chaosd-client-certs -n chaos-mesh -o "jsonpath={.data['ca\.crt']}" | base64 -d > ca.crt

   kubectl get secret chaos-mesh-chaosd-client-certs -n chaos-mesh -o "jsonpath={.data['ca\.key']}" | base64 -d> ca.key
   ```

2. `ca.crt`と`ca.key`ファイルを**対応する物理マシン**にコピーします。例えば、ファイルを`/etc/chaosd/pki`ディレクトリにコピーします。
3. **物理マシン**上で`chaosctl pm generate`コマンドを使用してTLS証明書を生成します（デフォルトでは`/etc/chaosd/pki`に保存されます）。例:

   ```bash
   ./chaosctl pm generate --cacert=/etc/chaosd/pki/ca.crt --cakey=/etc/chaosd/pki/ca.key
   ```

   この機能の詳細と例については、`chaosctl pm generate -h`を参照してください。

4. Kubernetesクラスターにアクセスできるマシン上で、`chaosctl pm create`コマンドを使用してKubernetesクラスター内に`PhysicalMachine`リソースを作成します。例:

   ```bash
   ./chaosctl pm create pm-name --ip=123.123.123.123 -l arch=amd64
   ```

   この機能の詳細と例については、`chaosctl pm create -h`を参照してください。

## 質問とフィードバック

Chaosctlのコードは現在、Chaos Meshプロジェクトでホストされています。詳細は[chaos-mesh/pkg/chaosctl](https://github.com/chaos-mesh/chaos-mesh/tree/master/pkg/chaosctl)を参照してください。

操作中に問題が発生した場合、またはこのツールの改善に興味がある場合は、[CNCF Slack](https://cloud-native.slack.com/archives/C0193VAV272)を通じてChaos Meshチームに連絡するか、[GitHub issue](https://github.com/chaos-mesh/chaos-mesh/issues)を作成してください。

問題を説明する際には、関連するログとChaos情報を添付すると役立ちます。開発者向けの参考資料として、質問に`chaosctl logs`の結果を添付することを推奨します。また、iochaos、networkchaos、stresschaosに関連する質問の場合、`chaosctl debug`の関連情報も問題の診断に役立ちます。