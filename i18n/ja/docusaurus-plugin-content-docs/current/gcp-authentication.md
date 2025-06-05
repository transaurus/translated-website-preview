---
title: GCP OAuth Authentication
---

Chaos MeshをGoogle Cloud Platformにデプロイする場合、Google OAuthを通じてChaos Dashboardにログインできます。このドキュメントでは、この機能を有効化して設定する方法について説明します。

## OAuthクライアントの作成

[OAuth 2.0の設定](https://support.google.com/cloud/answer/6158849?hl=en)に従ってGCP OAuthクライアントを作成し、Client IDとClient Secretを取得します。

1. [Google Cloud Platformコンソール](https://console.cloud.google.com/)にアクセスします。
2. プロジェクト一覧からプロジェクトを選択するか、新規作成します。
3. APIs & servicesページが自動的に表示されない場合、コンソールの左側メニューから手動で「APIs & services」を選択します。
4. 左側の「Credentials」をクリックします。
5. 「Create Credentials」をクリックし、「OAuth client ID」を選択します。
6. アプリケーションタイプとして「Web Application」を選択し、追加情報とChaos DashboardのリダイレクトURL（`ROOT_URL/api/auth/gcp/callback`）を入力します。ここで、`ROOT_URL`はChaos DashboardのルートURL（例: `http://localhost:2333`）です。このURLは`helm`を通じて設定項目`dashboard.rootUrl`で設定できます。
7. 「Create」をクリックします。

クライアント作成後、以下の手順で使用するためClient IDとClient Secretを保存しておいてください。

## Chaos Meshの設定と起動

:::info

更新: `v2.7.0`以降、Client IDとClient Secretを保存するために**Secret**を提供できます。**この方法を使用することを推奨します**。

この変更は、Client IDとClient Secretを公開しないようにするためです。以前のバージョンでは、これらの値が直接valuesに指定されており、一般的に安全ではありませんでした。

詳細は https://github.com/chaos-mesh/chaos-mesh/issues/4206 を参照してください。

:::

この機能を有効化するには、helm chartsで以下の設定項目を設定する必要があります:

```yaml
dashboard:
  rootUrl: http://localhost:2333
  gcpSecurityMode:
    enabled: true
    # Old configuration items for compatibility.
    clientId: ''
    clientSecret: ''
    # References existing Kubernetes secret containing `GCP_CLIENT_ID` and `GCP_CLIENT_SECRET`.
    existingSecret: ''
```

Chaos Meshが既にインストールされている場合、`helm upgrade`を通じて設定項目を更新できます。インストールされていない場合、`helm install`でChaos Meshをインストールできます。

## Googleでのログイン

Chaos Dashboardを開き、認証ウィンドウの下にあるGoogleアイコンをクリックします。

![img](./img/google-auth.png)

Googleアカウントにログインし、OAuthクライアントに権限を付与すると、ページは自動的にログイン状態のChaos Dashboardにリダイレクトされます。この時点で、このクラスタ内のgoogleアカウントと同じ権限を持ちます。他の権限を追加する必要がある場合、RBAC（Role-based access control）を通じて権限を編集できます。例:

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: chaos-mesh-cluster-manager
rules:
  - apiGroups:
      - chaos-mesh.org
    resources: ['*']
    verbs: ['get', 'list', 'watch', 'create', 'delete', 'patch', 'update']
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-manager-binding
  namespace: chaos-mesh
subjects:
  - kind: User
    name: example@gmail.com
roleRef:
  kind: ClusterRole
  name: chaos-mesh-cluster-manager
  apiGroup: rbac.authorization.k8s.io
```

この設定により、ユーザー`example@gmail.com`はあらゆるカオス実験を閲覧または作成できるようになります。