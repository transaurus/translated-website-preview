---
title: Simulate Azure Faults
---

このドキュメントでは、Chaos Meshを使用してAzure障害をシミュレートする方法について説明します。

## AzureChaosの概要

AzureChaosは、指定されたAzureインスタンス上で障害シナリオをシミュレートするのに役立ちます。現在、AzureChaosは以下の障害タイプをサポートしています：

- **VM停止**: 指定されたVMインスタンスを停止します。
- **VM再起動**: 指定されたVMインスタンスを再起動します。
- **ディスクデタッチ**: 指定されたVMインスタンスからデータディスクを取り外します。

## `Secret`ファイル

Azureクラスターに簡単に接続するために、認証情報を事前に保存するKubernetes `Secret`ファイルを作成できます。

`Secret`ファイルのサンプルは以下の通りです：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloud-key-secret
  namespace: chaos-mesh
type: Opaque
stringData:
  client_id: your-client-id
  client_secret: your-client-secret
  tenant_id: your-tenant-id
```

- **name**はKubernetes Secretオブジェクトを意味します。
- **namespace**はKubernetes Secretオブジェクトのネームスペースを意味します。
- **client_id**はAzureアプリ登録のアプリケーション（クライアント）IDを保存します。
- **client_secret**はAzureアプリ登録のアプリケーション（クライアント）シークレット値を保存します。
- **tenant_id**はAzureアプリ登録のディレクトリ（テナント）IDを保存します。`client_id`と`client_secret`については、[機密クライアントアプリケーション](https://docs.microsoft.com/en-us/azure/healthcare-apis/azure-api-for-fhir/register-confidential-azure-ad-client-app)を参照してください。

:::note

Secretファイル内のアプリ登録が、VMインスタンスのアクセス制御（IAM）に共同作成者または所有者として追加されていることを確認してください。

:::

## Chaos Dashboardを使用した実験の作成

1. Chaos Dashboardを開き、ページ上の**NEW EXPERIMENT**をクリックして新しい実験を作成します：

   ![img](./img/create-new-exp.png)

2. **Choose a Target**エリアで、**Azure FAULT**を選択し、**VM STOP**などの特定の動作を選択します。

3. 実験情報を入力し、実験範囲と予定された実験期間を指定します。

4. 実験情報を送信します。

## YAMLファイルを使用した実験の作成

### `vm-stop`設定例

1. 以下のように、実験設定を`azurechaos-vm-stop.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: AzureChaos
   metadata:
     name: vm-stop-example
     namespace: chaos-mesh
   spec:
     action: vm-stop
     secretName: 'cloud-key-secret'
     subscriptionID: 'your-subscription-id'
     resourceGroupName: 'your-resource-group-name'
     vmName: 'your-vm-name'
     duration: '5m'
   ```

   この設定例に基づき、Chaos Meshは指定されたVMインスタンスに`vm-stop`障害を注入し、VMインスタンスは5分間利用不可になります。

   VMインスタンスの停止に関する詳細は、[Azureドキュメント - VMの開始または停止](https://docs.microsoft.com/en-us/azure/devtest-labs/use-command-line-start-stop-virtual-machines)を参照してください。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f azurechaos-vm-stop.yaml
   ```

### `vm-restart`設定例

1. 実験設定を `azurechaos-vm-restart.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: AzureChaos
   metadata:
     name: vm-restart-example
     namespace: chaos-mesh
   spec:
     action: vm-restart
     secretName: 'cloud-key-secret'
     subscriptionID: 'your-subscription-id'
     resourceGroupName: 'your-resource-group-name'
     vmName: 'your-vm-name'
   ```

   この設定例に基づき、Chaos Mesh は指定された VM インスタンスに `vm-restart` 障害を注入し、VM インスタンスが再起動されます。

   VM インスタンスの再起動に関する詳細は、[Azure ドキュメント - VM の再起動](https://docs.microsoft.com/en-us/azure/devtest-labs/devtest-lab-restart-vm)を参照してください。

2. 設定ファイルの準備ができたら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f azurechaos-vm-restart.yaml
   ```

### `detach-volume` 設定例

1. 実験設定を `azurechaos-disk-detach.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: AzureChaos
   metadata:
     name: disk-detach-example
     namespace: chaos-mesh
   spec:
     action: disk-detach
     secretName: 'cloud-key-secret'
     subscriptionID: 'your-subscription-id'
     resourceGroupName: 'your-resource-group-name'
     vmName: 'your-vm-name'
     diskName: 'your-disk-name'
     lun: 'your-disk-lun'
     duration: '5m'
   ```

   この設定例に基づき、Chaos Mesh は指定された VM インスタンスに `disk-detach` 障害を注入し、5分以内に指定されたデータディスクが VM インスタンスからデタッチされます。

   Azure データディスクのデタッチに関する詳細は、[Azure ドキュメント - データディスクのデタッチ](https://docs.microsoft.com/en-us/azure/devtest-labs/devtest-lab-attach-detach-data-disk#detach-a-data-disk)を参照してください。

2. 設定ファイルの準備ができたら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f azurechaos-disk-detach.yaml
   ```

### フィールド説明

以下の表は、YAML 設定ファイルのフィールドを示しています。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| action | string | Indicates the specific type of faults. Only `vm-stop`, `vm-restart`, and `disk-detach` are supported. | `vm-stop` | Yes | `vm-stop` |
| mode | string | Specifies the mode of the experiment. The mode options include `one` (selecting a random Pod), `all` (selecting all eligible Pods), `fixed` (selecting a specified number of eligible Pods), `fixed-percent` (selecting a specified percentage of Pods from the eligible Pods), and `random-max-percent` (selecting the maximum percentage of Pods from the eligible Pods). | N/A | Yes | `one` |
| value | string | Provides parameters for the `mode` configuration, depending on `mode`. For example, when `mode` is set to `fixed-percent`, `value` specifies the percentage of Pods. | N/A | No | `1` |
| secretName | string | Specifies the name of the Kubernetes Secret that stores the Azure authentication information. | N/A | No | `cloud-key-secret` |
| subscriptionID | string | Specifies the VM instacnce's subscription ID. | N/A | Yes | `your-subscription-id` |
| resourceGroupName | string | Specifies the Resource group of VM. | N/A | Yes | `your-resource-group-name` |
| vmName | string | VMName defines the name of Virtual Machine. | N/A | Yes | `your-vm-name` |
| diskName | string | This is a required field when the `action` is `disk-detach`, specifies the name of data disk. | N/A | No | `DATADISK_0` |
| lun | string | This is a required field when the `action` is `disk-detach`, specifies the LUN (Logic Unit Number) of data disk. | N/A | No | `0` |
| duration | string | Specifies the duration of the experiment. | N/A | Yes | `30s` |