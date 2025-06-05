Chaos Meshの稼働状況を確認するには、以下のコマンドを実行してください:

```sh
kubectl get pods -n chaos-mesh -l app.kubernetes.io/instance=chaos-mesh
```

期待される出力は以下の通りです:

```txt
NAME                                       READY   STATUS    RESTARTS   AGE
chaos-controller-manager-7b8c86cc9-44dzf   1/1     Running   0          17m
chaos-controller-manager-7b8c86cc9-mxw99   1/1     Running   0          17m
chaos-controller-manager-7b8c86cc9-xmc5v   1/1     Running   0          17m
chaos-daemon-sg2k2                         1/1     Running   0          17m
chaos-dashboard-b9dbc6b68-hln25            1/1     Running   0          17m
chaos-dns-server-546675d89d-qkjqq          1/1     Running   0          17m
```

実際の出力が期待される出力と類似している場合、Chaos Meshは正常にインストールされています。

:::note

実際の出力で`STATUS`が`Running`でない場合、以下のコマンドを実行してPodの詳細を確認し、エラー情報に基づいて問題をトラブルシューティングしてください。

```sh
# chaos-controllerを例として
kubectl describe po -n chaos-mesh chaos-controller-manager-7b8c86cc9-44dzf
```

:::

:::note

リーダー選出が無効になっている場合、`chaos-controller-manager`のレプリカ数は1つだけであるべきです。

```txt
NAME                                        READY   STATUS    RESTARTS   AGE
chaos-controller-manager-676d8567c7-ndr5j   1/1     Running   0          24m
chaos-daemon-6l55b                          1/1     Running   0          24m
chaos-dashboard-b9dbc6b68-hln25             1/1     Running   0          44m
chaos-dns-server-546675d89d-qkjqq           1/1     Running   0          44m
```

:::