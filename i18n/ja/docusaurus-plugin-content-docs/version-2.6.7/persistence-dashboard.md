---
title: Persistence Chaos Dashboard
---

import PickHelmVersion from '@site/src/components/PickHelmVersion'

このドキュメントでは、Chaos Dashboardの永続化方法について説明します。

Chaos Dashboardは、永続化のためのデータベースバックエンドとして `SQLite`、`MySQL`、`PostgreSQL` をサポートしています。

## SQLite（デフォルト）

Chaos Dashboardはデフォルトのデータベースエンジンとして `SQLite` を使用しており、[PV（Persistent Volumes）](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) を有効にすることを推奨しています。PVを有効にするには、`dashboard.persistentVolume.enabled` を `true` に設定します。関連する設定は [`value.yaml`](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/values.yaml#L255-L282) で以下のように確認できます。

```yaml
dashboard:
  ...
  persistentVolume:
    # If you are using SQLite as your DB for Chaos Dashboard, it is recommended to enable persistence.
    # If enable, the chart will create a PersistenceVolumeClaim to store its state in. If you are
    # using a DB other than SQLite, set this to false to avoid allocating unused storage.
    # If set to false, Chaos Mesh will use an emptyDir instead, which is ephemeral.
    enabled: true

    # If you'd like to bring your own PVC for persisting chaos event, pass the name of the
    # created + ready PVC here. If set, this Chart will not create the default PVC.
    # Requires server.persistentVolume.enabled: true
    existingClaim: ""

    # Chaos Dashboard data Persistent Volume size.
    size: 8Gi

    # Chaos Dashboard data Persistent Volume Storage Class.
    # If defined, storageClassName: <storageClass>
    storageClassName: standard

    # Chaos Dashboard data Persistent Volume mount root path
    mountPath: /data

    # Subdirectory of Chaos Dashboard data Persistent Volume to mount
    # Useful if the volume's root directory is not empty
    subPath: ""
```

:::warning

PVなしでChaos Dashboardコンポーネントが再起動すると、Chaos Dashboardのデータは失われ、復元できません。

:::

## MySQL

Chaos Dashboardは、データベースエンジンとしてMySQL 5.6以降のバージョンをサポートしています。以下の例はMySQLデータベースの設定を示しています。接続文字列の設定の詳細については、[MySQL-Driver for Go](https://github.com/go-sql-driver/mysql#dsn-data-source-name) を参照してください。

<PickHelmVersion>
{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version latest --set dashboard.env.DATABASE_DRIVER=mysql --set dashboard.env.DATABASE_DATASOURCE=root:password@tcp(1.2.3.4:3306)/chaos-mesh?parseTime=true`}
</PickHelmVersion>

## PostgreSQL

Chaos Dashboardは、データベースエンジンとしてPostgreSQL 9.6以降のバージョンをサポートしています。以下の例はPostgreSQLデータベースの設定を示しています。接続文字列の設定の詳細については、[libpq connect](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING) を参照してください。

<PickHelmVersion>
{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version latest --set dashboard.env.DATABASE_DRIVER=postgres --set dashboard.env.DATABASE_DATASOURCE=postgres://root:password@1.2.3.4:5432/postgres?sslmode=disable`}
</PickHelmVersion>

## Chaos DashboardデータのTTL（Time To Live）設定

Chaos Dashboardは、データの有効期限を設定する機能をサポートしています。デフォルトでは `Event` 関連データは `168h` で、`Experiment` 関連データは `336h` で期限切れになります。変更が必要な場合は、`dashboard.env.TTL_EVENT` および `dashboard.env.TTL_EXPERIMENT` パラメータを以下のように設定できます。

<PickHelmVersion>
{`helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --version latest --set dashboard.env.TTL_EVENT=168h --set dashboard.env.TTL_EXPERIMENT=336h`}
</PickHelmVersion>