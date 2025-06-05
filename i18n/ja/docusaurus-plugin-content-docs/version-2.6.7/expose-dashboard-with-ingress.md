# Ingressを使用してChaos Dashboardを公開する

外部ユーザーに対してChaos Dashboardをアクセス可能にしつつ、現在の監視ダッシュボードのサブパスに配置したい場合があります。

以下は、Chaos Dashboardを`/chaos-mesh`パスで公開する方法の例です：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-chaos-dashboard-under-subpath
  namespace: chaos-mesh
  annotations:
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/configuration-snippet: |
      sub_filter '<head>' '<head> <base href="/chaos-mesh/">';
spec:
  rules:
    - http:
        paths:
          - path: /chaos-mesh/?(.*)
            pathType: Prefix
            backend:
              service:
                name: chaos-dashboard
                port:
                  number: 2333
```

この例はhttps://github.com/chaos-mesh/chaos-mesh/blob/master/examples/dashboard/ingress-subpath.yamlでも確認できます。