---
title: Add a New Chaos Experiment Type
---

import PickHelmVersion from '@site/src/components/PickHelmVersion'

このドキュメントでは、新しいカオス実験タイプを追加する方法について説明します。

以下は、ログに`Hello world!`と出力する新しいカオス実験タイプ`HelloWorldChaos`の例を通じて手順を説明します。手順は以下の通りです：

- [Step 1: HelloWorldChaosのスキーマを定義する](#step-1-define-the-schema-of-helloworldchaos)
- [Step 2: CRDを登録する](#step-2-register-the-crd)
- [Step 3: helloworldオブジェクトのイベントハンドラを登録する](#step-3-register-the-event-handler-for-helloworldchaos-objects)
- [Step 4: Dockerイメージをビルドする](#step-4-build-docker-images)
- [Step 5: HelloWorldChaosを実行する](#step-5-run-helloworldchaos)

## Step 1: HelloWorldChaosのスキーマを定義する

1. `api/v1alpha1` APIディレクトリに以下の内容で`helloworldchaos_types.go`を追加します：

   ```go
   package v1alpha1

   import (
           metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
   )

   // +kubebuilder:object:root=true
   // +chaos-mesh:experiment
   // +chaos-mesh:oneshot=true

   // HelloWorldChaosはhelloworldchaos APIのスキーマです
   type HelloWorldChaos struct {
           metav1.TypeMeta   `json:",inline"`
           metav1.ObjectMeta `json:"metadata,omitempty"`

           Spec   HelloWorldChaosSpec   `json:"spec"`
           Status HelloWorldChaosStatus `json:"status,omitempty"`
   }

   // HelloWorldChaosSpecはHelloWorldChaosの望ましい状態を定義します
   type HelloWorldChaosSpec struct {
           // ContainerSelectorは注入対象を指定します
           ContainerSelector `json:",inline"`

           // Durationはカオスの継続時間を表します
           // +optional
           Duration *string `json:"duration,omitempty"`

           // RemoteClusterはカオスがデプロイされるリモートクラスタを表します
           // +optional
           RemoteCluster string `json:"remoteCluster,omitempty"`
   }

   // HelloWorldChaosStatusはHelloWorldChaosの観測された状態を定義します
   type HelloWorldChaosStatus struct {
           ChaosStatus `json:",inline"`
   }

   // GetSelectorSpecsはセレクタのゲッターです
   func (obj *HelloWorldChaos) GetSelectorSpecs() map[string]interface{} {
           return map[string]interface{}{
                   ".": &obj.Spec.ContainerSelector,
           }
   }
   ```

   このファイルは`HelloWorldChaos`のスキーマタイプを定義しており、YAMLファイルで記述できます：

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: HelloWorldChaos
   metadata:
     name: <リソース名>
     namespace: <名前空間>
   spec:
     duration: <継続時間>
   #...
   ```

## Step 2: CRDを登録する

Kubernetes APIとやり取りするために、`HelloWorldChaos`のCRD（カスタムリソース定義）を登録する必要があります。

1. 前のステップで生成した`config/crd/bases/chaos-mesh.org_helloworldchaos.yaml`を`manifests/crd.yaml`に組み込むために、`config/crd/kustomization.yaml`に追加します：

   ```yaml
   resources:
     - bases/chaos-mesh.org_podchaos.yaml
     - bases/chaos-mesh.org_networkchaos.yaml
     - bases/chaos-mesh.org_iochaos.yaml
     - bases/chaos-mesh.org_helloworldchaos.yaml # これが新しい行です
   ```

2. Chaos Meshのルートディレクトリで`make generate`を実行し、Chaos Meshがコンパイルするための`HelloWorldChaos`のボイラープレートを生成します：

   ```bash
   make generate
   ```

   これにより、`manifests/crd.yaml`に`HelloWorldChaos`の定義が表示されます。

## Step 3: helloworldchaosオブジェクトのイベントハンドラを登録する

1. 以下の内容で新しいファイル `controllers/chaosimpl/helloworldchaos/types.go` を作成します:

   ```go
   package helloworldchaos

   import (
           "context"

           "github.com/go-logr/logr"
           "go.uber.org/fx"
           "sigs.k8s.io/controller-runtime/pkg/client"

           "github.com/chaos-mesh/chaos-mesh/api/v1alpha1"
           impltypes "github.com/chaos-mesh/chaos-mesh/controllers/chaosimpl/types"
           "github.com/chaos-mesh/chaos-mesh/controllers/chaosimpl/utils"
   )

   var _ impltypes.ChaosImpl = (*Impl)(nil)

   type Impl struct {
           client.Client
           Log logr.Logger

           decoder *utils.ContainerRecordDecoder
   }

   // これはHelloWorldChaosのApplyフェーズに対応します。HelloWorldChaosの実行がトリガーされます。
   func (impl *Impl) Apply(ctx context.Context, index int, records []*v1alpha1.Record, obj v1alpha1.InnerObject) (v1alpha1.Phase, error) {
           impl.Log.Info("Hello world!")
           return v1alpha1.Injected, nil
   }

   // これはHelloWorldChaosのRecoverフェーズに対応します。リコンシラーがトリガーされ、カオスアクションが回復されます。
   func (impl *Impl) Recover(ctx context.Context, index int, records []*v1alpha1.Record, obj v1alpha1.InnerObject) (v1alpha1.Phase, error) {
           impl.Log.Info("Goodbye world!")
           return v1alpha1.NotInjected, nil
   }

   // NewImplは新しいHelloWorldChaosの実装インスタンスを返します。
   func NewImpl(c client.Client, log logr.Logger, decoder *utils.ContainerRecordDecoder) *impltypes.ChaosImplPair {
           return &impltypes.ChaosImplPair{
                   Name:   "helloworldchaos",
                   Object: &v1alpha1.HelloWorldChaos{},
                   Impl: &Impl{
                           Client:  c,
                           Log:     log.WithName("helloworldchaos"),
                           decoder: decoder,
                   },
                   ObjectList: &v1alpha1.HelloWorldChaosList{},
        }
   }

   var Module = fx.Provide(
            fx.Annotated{
                    Group:  "impl",
                    Target: NewImpl,
            },
   )
   ```

2. Chaos Meshは依存性注入に[fx](https://github.com/uber-go/fx)ライブラリを使用しています。コントローラーマネージャーに`HelloWorldChaos`を登録するには、`controllers/chaosimpl/fx.go`に以下の行を追加します:

   ```go
   var AllImpl = fx.Options(
           gcpchaos.Module,
           stresschaos.Module,
           jvmchaos.Module,
           timechaos.Module,
           helloworldchaos.Module // 新しい行を追加します。helloworldchaosを先にインポートしていることを確認してください。
           //...
   )
   ```

   次に、`controllers/types/types.go`で、`ChaosObjects`に以下の内容を追加します:

   ```go
   var ChaosObjects = fx.Supply(
          //...
          fx.Annotated{
                  Group: "objs",
                  Target: Object{
                          Name:   "helloworldchaos",
                          Object: &v1alpha1.HelloWorldChaos{},
                  },
          },
   )
   ```

## ステップ4: Dockerイメージのビルド

1. 本番用イメージをビルドします:

   ```bash
   make image
   ```

2. minikubeを使用してKubernetesクラスターをデプロイしている場合、イメージをクラスターにロードする必要があります:

   ```bash
   minikube image load ghcr.io/chaos-mesh/chaos-dashboard:latest
   minikube image load ghcr.io/chaos-mesh/chaos-mesh:latest
   minikube image load ghcr.io/chaos-mesh/chaos-daemon:latest
   ```

## ステップ5: HelloWorldChaosの実行

このステップでは、最新の変更を加えたChaos Meshをデプロイし、HelloWorldChaosをテストする必要があります。

1. クラスタにCRDを登録します:

   ```bash
   kubectl create -f manifests/crd.yaml
   ```

   出力から`HelloWorldChaos`が作成されたことを確認できます:

   ```log
   customresourcedefinition.apiextensions.k8s.io/helloworldchaos.chaos-mesh.org created
   ```

   以下のコマンドで`HelloWorldChaos`のCRDを取得できます:

   ```bash
   kubectl get crd helloworldchaos.chaos-mesh.org
   ```

2. Chaos Meshをデプロイします:

   ```bash
   helm install chaos-mesh helm/chaos-mesh -n=chaos-mesh --set controllerManager.leaderElection.enabled=false,dashboard.securityMode=false
   ```

   デプロイが成功したことを確認するには、`chaos-mesh`ネームスペース内のすべてのPodを確認します:

   ```bash
   kubectl get pods --namespace chaos-mesh -l app.kubernetes.io/instance=chaos-mesh
   ```

3. テスト用のデプロイメントをデプロイします。minikubeドキュメントのエコーサーバー例を使用できます:

   ```bash
   kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
   ```

   Podが実行中になるまで待ちます:

   ```bash
   kubectl get pods
   ```

   出力例:

   ```log
   NAME                              READY   STATUS    RESTARTS   AGE
   hello-minikube-77b6f68484-dg4sw   1/1     Running   0          2m
   ```

4. 以下の内容で`hello.yaml`ファイルを作成します:

   ```yaml
   apiVersion: chaos-mesh.org/v1alpha1
   kind: HelloWorldChaos
   metadata:
     name: hello-world
     namespace: chaos-mesh
   spec:
     selector:
       labelSelectors:
         app: hello-minikube
     mode: one
     duration: 1h
   ```

5. 実行します:

   ```bash
   kubectl apply -f hello.yaml
   # helloworldchaos.chaos-mesh.org/hello-world created
   ```

   これで、`chaos-controller-manager`のログに`Hello world!`が表示されるか確認できます:

   ```bash
   kubectl logs -n chaos-mesh chaos-controller-manager-xxx
   ```

   出力例:

   ```txt
   2023-07-16T06:19:40.068Z INFO records records/controller.go:149 apply chaos {"id": "default/hello-minikube-77b6f68484-dg4sw/echo-server"}
   2023-07-16T06:19:40.068Z INFO helloworldchaos helloworldchaos/types.go:26 Hello world!
   ```

## 次のステップ

このプロセスで問題が発生した場合は、Chaos Meshリポジトリに[issue](https://github.com/chaos-mesh/chaos-mesh/issues)を作成してください。

次のセクションでは、`HelloWorldChaos`の動作をさらに拡張する方法について学びます。