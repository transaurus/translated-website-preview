---
title: Simulate Linux Kernel Faults
---

このドキュメントでは、KernelChaosを使用してLinuxカーネルの障害をシミュレートする方法について説明します。この機能は[BPF](https://lore.kernel.org/lkml/20171213180356.hsuhzoa7s4ngro2r@destiny/T/)を使用して、指定されたカーネルパスにI/Oベースまたはメモリベースの障害を注入します。

KernelChaosの注入ターゲットを1つまたは複数のPodに設定できますが、ホスト上の他のPodのパフォーマンスにも影響が及ぶ可能性があります。これは、すべてのPodが同じカーネルを共有しているためです。

:::warning

Linuxカーネル障害のシミュレーションはデフォルトで無効化されています。本番環境ではこの機能を使用しないでください。

:::

## 前提条件

- Linuxカーネルバージョン >= 4.18
- Linuxカーネル設定[CONFIG_BPF_KPROBE_OVERRIDE](https://cateee.net/lkddb/web-lkddb/BPF_KPROBE_OVERRIDE.html)が有効になっていること
- [values.yaml](https://github.com/chaos-mesh/chaos-mesh/blob/master/helm/chaos-mesh/values.yaml)の`bpfki.create`設定値が`true`であること

## 設定ファイル

シンプルなKernelChaos設定ファイルの例は以下の通りです：

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: KernelChaos
metadata:
  name: kernel-chaos-example
  namespace: chaos-mesh
spec:
  mode: one
  selector:
    namespaces:
      - chaos-mount
  failKernRequest:
    callchain:
      - funcname: '__x64_sys_mount'
    failtype: 0
```

より多くの設定例については、[examples](https://github.com/chaos-mesh/chaos-mesh/tree/master/examples)を参照してください。必要に応じてこれらの設定例を変更できます。

設定の説明：

- **mode** は実験の実行方法を指定します。オプションは以下の通りです:

  - `one`: 条件に合致するPodをランダムに1つ選択します。
  - `all`: 条件に合致する全てのPodを選択します。
  - `fixed`: 指定した数の条件に合致するPodを選択します。
  - `fixed-percent`: 条件に合致するPodの指定した割合を選択します。
  - `random-max-percent`: 条件に合致するPodの最大割合を選択します。

- **selector** は障害を注入する対象のPodを指定します。
- **FailedkernRequest** は障害モード（kmaloやbioなど）を指定します。また、特定のコールチェーンパスとオプションのフィルタリング条件も指定します。設定項目は以下の通りです:

  - **Failtype** は障害タイプを指定します。値のオプションは以下の通りです:

    - '0': slab割り当てエラーshould_failslabを注入します。
    - '1': メモリページ割り当てエラーshould_fail_alloc_pageを注入します。
    - '2': bioエラーshould_fail_bioを注入します。

    これら3つの障害タイプの詳細については、[fault-injection](https://www.kernel.org/doc/html/latest/fault-injection/fault-injection.html) と [inject_example](http://github.com/iovisor/bcc/blob/master/tools/inject_example.txt) を参照してください。

  - **Callchain** は特定のコールチェーンを指定します。例:

    ```c
    ext4_mount
    -> mount_subtree
    -> ...
        -> should_failslab
    ```

    関数のパラメータをフィルタリングルールとして使用し、より細かい粒度で障害を注入することもできます。詳細は [call chain and predicate examples](https://github.com/chaos-mesh/bpfki/tree/develop/examples) を参照してください。コールチェーンが指定されていない場合、`callchain` フィールドを空のままにします。これは、slab割り当てが呼び出される任意のパス（例えばkmalo）に障害が注入されることを意味します。

    コールチェーンタイプはフレーム配列で、以下の3つの部分で構成されます:

    - **funcname**: カーネルソースコードまたは `/proc/kallsyms` から見つけることができます。例: `ext4_mount`。
    - **parameters**: フィルタリングに使用されます。特別な名前 `bananas` を持つ `d_alloc_parallel(struct dentry *parent, const struct qstr *name)` パスでslabエラーを注入したい場合、`parameters` を `struct dentry *parent, const struct qstr *name` に設定する必要があります。それ以外の場合はこの設定を省略します。
    - **predicate**: フレーム配列のパラメータにアクセスするために使用されます。**parameters** の例として、`STRNCMP(name->name, "bananas", 8)` を設定して障害注入のパスを制御するか、空のままにして `d_allo_parallel` を実行する全てのコールパスにslab障害を注入することができます。

  - **headers** は必要なカーネルヘッダーファイルを指定します。例: "linux/mmzone.h" と "linux/blkdev.h"。
  - **probability** は障害の発生確率を指定します。1%の確率にしたい場合は '1' に設定します。
  - **times** は障害がトリガーされる最大回数を指定します。

## kubectlを使用した実験の作成

`kubectl` を使用して実験を作成します:

```bash
kubectl apply -f KernelChaos
```

KernelChaos機能は [inject.py](https://github.com/iovisor/bcc/blob/master/tools/inject.py) と似ています。詳細は [input_example.txt](https://github.com/iovisor/bcc/blob/master/tools/inject_example.txt) を参照してください。

簡単な例は以下の通りです:

```c
#include <sys/mount.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>

int main(void) {
  int ret;
  while (1) {
    ret = mount("/dev/sdc", "/mnt", "ext4",
          MS_MGC_VAL | MS_RDONLY | MS_NOSUID, "");
    if (ret < 0)
      fprintf(stderr, "%s\n", strerror(errno));
    sleep(1);
    ret = umount("/mnt");
    if (ret < 0)
      fprintf(stderr, "%s\n", strerror(errno));
  }
}
```

障害注入中の出力は以下の通りです:

```
> Cannot allocate memory
> Invalid argument
> Cannot allocate memory
> Invalid argument
> Cannot allocate memory
> Invalid argument
> Cannot allocate memory
> Invalid argument
> Cannot allocate memory
> Invalid argument
```

## 使用制限

container_idを使用して障害注入の範囲を制限できますが、一部のパスはシステムレベルの動作を引き起こします。例:

`failtype` が `1` の場合、物理ページの割り当て失敗を意味します。このイベントが短時間に頻繁にトリガーされると（例: `while (1) {memset(malloc(1M), '1', 1M)}`）、oom-killerシステムコールがトリガーされてメモリが回収されます。