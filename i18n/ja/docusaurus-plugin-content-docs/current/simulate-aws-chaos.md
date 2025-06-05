---
title: Simulate AWS Faults
---

このドキュメントでは、Chaos Meshを使用してAWSの障害をシミュレートする方法について説明します。

## AWSChaosの紹介

AWSChaosは、指定したAWSインスタンス上で障害シナリオをシミュレートするのに役立ちます。現在、AWSChaosは以下の障害タイプをサポートしています：

- **EC2 Stop**: 指定したEC2インスタンスを停止します。
- **EC2 Restart**: 指定したEC2インスタンスを再起動します。
- **Detach Volume**: 指定したEC2インスタンスからストレージボリュームをアンマウントします。

## `Secret`ファイル

AWSクラスタに簡単に接続するために、事前に認証情報を保存するKubernetesの`Secret`ファイルを作成できます。

`Secret`ファイルのサンプルは以下の通りです：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloud-key-secret
  namespace: chaos-mesh
type: Opaque
stringData:
  aws_access_key_id: your-aws-access-key-id
  aws_secret_access_key: your-aws-secret-access-key
  aws_session_token: your-aws-session-token
```

- **name**はKubernetesのSecretオブジェクトを指します。
- **namespace**はKubernetesのSecretオブジェクトのネームスペースです。
- **aws_access_key_id**はAWSクラスタへのアクセスキーIDを保存します。
- **aws_secret_access_key**はAWSクラスタへのシークレットアクセスキーを保存します。
- **aws_session_token**はAWSセッショントークンを保存します（[一時的なAWS認証情報](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)を使用する場合に必要です）。

## Chaos Dashboardを使用した実験の作成

:::note

Chaos Dashboardを使用して実験を作成する前に、以下の要件を満たしていることを確認してください：

1. Chaos Dashboardがインストールされていること。
2. Chaos Dashboardに`kubectl port-forward`でアクセスできること：

   ```bash
    kubectl port-forward -n chaos-mesh svc/chaos-dashboard 2333:2333
   ```

   その後、ブラウザで[`http://localhost:2333`](http://localhost:2333)からダッシュボードにアクセスできます。

:::

1. Chaos Dashboardを開き、ページ上の**NEW EXPERIMENT**をクリックして新しい実験を作成します：

   ![img](./img/create-new-exp.png)

2. **Choose a Target**エリアで、**AWS FAULT**を選択し、**STOP EC2**などの特定の動作を選択します。

3. 実験情報を入力し、実験のスコープと予定された実験期間を指定します。

4. 実験情報を送信します。

## YAMLファイルを使用した実験の作成

### `ec2-stop`設定の例

1. 以下のように、実験設定を`awschaos-ec2-stop.yaml`ファイルに記述します：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: AWSChaos
   metadata:
     name: ec2-stop-example
     namespace: chaos-mesh
   spec:
     action: ec2-stop
     secretName: 'cloud-key-secret'
     awsRegion: 'us-east-2'
     ec2Instance: 'your-ec2-instance-id'
     duration: '5m'
   ```

   この設定例に基づいて、Chaos Meshは指定したEC2インスタンスに`ec2-stop`障害を注入し、5分間EC2インスタンスを利用不可にします。

   EC2インスタンスの停止に関する詳細は、[AWSドキュメント - Stop and start your instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Stop_Start.html)を参照してください。

2. 設定ファイルの準備ができたら、`kubectl`を使用して実験を作成します：

   ```bash
   kubectl apply -f awschaos-ec2-stop.yaml
   ```

### `ec2-start`設定の例

1. 実験設定を `awchaos-ec2-restot.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: AWSChaos
   metadata:
     name: ec2-restart-example
     namespace: chaos-mesh
   spec:
     action: ec2-restart
     secretName: 'cloud-key-secret'
     awsRegion: 'us-east-2'
     ec2Instance: 'your-ec2-instance-id'
   ```

   この設定例に基づき、Chaos Mesh は指定された EC2 インスタンスに `ec2-restart` 障害を注入し、EC2 インスタンスが再起動されます。

   EC2 インスタンスの再起動に関する詳細は、[AWS ドキュメント - インスタンスの再起動](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-reboot.html)を参照してください。

2. 設定ファイルの準備が整ったら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f awschaos-ec2-restart.yaml
   ```

### `detach-volume` 設定例

1. 実験設定を `awschaos-detach-volume.yaml` ファイルに記述します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: AWSChaos
   metadata:
     name: ec2-detach-volume-example
     namespace: chaos-mesh
   spec:
     action: ec2-stop
     secretName: 'cloud-key-secret'
     awsRegion: 'us-east-2'
     ec2Instance: 'your-ec2-instance-id'
     volumeID: 'your-volume-id'
     deviceName: '/dev/sdf'
     duration: '5m'
   ```

   この設定例に基づき、Chaos Mesh は指定された EC2 インスタンスに `detail-volume` 障害を注入し、5分以内に指定されたストレージボリュームから EC2 インスタンスが切り離されます。

   Amazon EBS ボリュームの切り離しに関する詳細は、[AWS ドキュメント - Linux インスタンスからの Amazon EBS ボリュームの切り離し](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-detaching-volume.html)を参照してください。

2. 設定ファイルの準備が整ったら、`kubectl` を使用して実験を作成します:

   ```bash
   kubectl apply -f awschaos-detach-volume.yaml
   ```

### フィールド説明

以下の表は、YAML 設定ファイルのフィールドを示しています。

| Parameter | Type | Description | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| action | string | Indicates the specific type of faults. Only ec2-stop, ec2-restore, and detain-volume are supported. | ec2-stop | Yes | ec2-stop |
| mode | string | Specifies the mode of the experiment. The mode options include `one` (selecting a random Pod), `all` (selecting all eligible Pods), `fixed` (selecting a specified number of eligible Pods), `fixed-percent` (selecting a specified percentage of Pods from the eligible Pods), and `random-max-percent` (selecting the maximum percentage of Pods from the eligible Pods). | None | Yes | `one` |
| value | string | Provides parameters for the `mode` configuration, depending on `mode`.For example, when `mode` is set to `fixed-percent`, `value` specifies the percentage of Pods. | None | No | 1 |
| secretName | string | Specifies the name of the Kubernetes Secret that stores the AWS authentication information. | None | No | cloud-key-secret |
| awsRegion | string | Specifies the AWS region. | None | Yes | us-east-2 |
| ec2Instance | string | Specifies the ID of the EC2 instance. | None | Yes | your-ec2-instance-id |
| volumeID | string | This is a required field when the `action` is `detach-volume`. This field specifies the EBS volume ID. | None | No | your-volume-id |
| deviceName | string | This is a required field when the `action` is `detach-volume`. This field specifies the machine name. | None | No | /dev/sdf |
| duration | string | Specifies the duration of the experiment. | None | Yes | 30s |