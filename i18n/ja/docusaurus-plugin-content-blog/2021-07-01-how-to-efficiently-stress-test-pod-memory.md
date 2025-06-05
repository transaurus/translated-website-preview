---
slug: /how-to-efficiently-stress-test-pod-memory
title: 'How to efficiently stress test Pod memory'
authors: yinghaowang
image: /img/blog/how-to-efficiently-stress-test-pod-memory-banner.jpg
tags: [Chaos Mesh, Chaos Engineering, StressChaos, Stress Testing]
---

![バナー](/img/blog/how-to-efficiently-stress-test-pod-memory-banner.jpg)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)にはStressChaosツールが含まれており、Podに対してCPUやメモリの負荷を注入することができます。このツールは、CPUやメモリに敏感なプログラムをテストまたはベンチマークする際、その圧力下での挙動を知りたい場合に非常に有用です。

しかし、StressChaosをテストおよび使用する中で、ユーザビリティやパフォーマンスに関するいくつかの問題を発見しました。例えば、なぜStressChaosは設定したよりもはるかに少ないメモリしか使用しないのか？これらの問題を修正するため、新たな一連のテストを開発しました。本記事では、これらの問題をトラブルシューティングし修正した過程を説明します。この情報により、StressChaosを最大限に活用できるようになるでしょう。

<!--truncate-->

続行する前に、クラスタにChaos Meshをインストールする必要があります。詳細な手順は[公式サイト](https://chaos-mesh.org/docs/quick-start)で確認できます。

## ターゲットへの負荷注入

ここでは、StressChaosをターゲットに注入する方法を実演します。この例では、[helm charts](https://helm.sh/)で管理されている[`hello-kubernetes`](https://github.com/paulbouwer/hello-kubernetes)を使用します。最初のステップは、[`hello-kubernetes`](https://github.com/paulbouwer/hello-kubernetes)リポジトリをクローンし、リソース制限を設定するためにチャートを修正することです。

```bash
git clone https://github.com/paulbouwer/hello-kubernetes.git
code deploy/helm/hello-kubernetes/values.yaml # or whichever editor you prefer
```

resources行を見つけ、以下のように変更します：

```yaml
resources:
  requests:
    memory: '200Mi'
  limits:
    memory: '500Mi'
```

ただし、何も注入する前に、ターゲットが消費しているメモリ量を確認しましょう。Podに入り、シェルを起動します。以下のコマンドを入力します（例のPod名を自身のものに置き換えてください）：

```bash
kubectl exec -it -n hello-kubernetes hello-kubernetes-hello-world-b55bfcf68-8mln6 -- /bin/sh
```

メモリ使用量のサマリーを表示します。次のコマンドを入力します：

```sh
/usr/src/app $ free -m
/usr/src/app $ top
```

以下の出力からわかるように、Podは4,269 MBのメモリを消費しています。

```sh
/usr/src/app $ free -m
              used
Mem:          4269
Swap:            0

/usr/src/app $ top
Mem: 12742432K used
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 node     S     285m   2%   0   0% npm start
   18     1 node     S     284m   2%   3   0% node server.js
   29     0 node     S     1636   0%   2   0% /bin/sh
   36    29 node     R     1568   0%   3   0% top
```

これは正しくないようです。メモリ使用量を500 MiBに制限しているのに、Podは数GBのメモリを使用しているように見えます。プロセスのメモリ使用量を合計しても500 MiBにはなりません。ただし、topとfreeの結果は少なくとも近い値です。

このPodにStressChaosを実行し、何が起こるか見てみましょう。使用するyamlは以下の通りです：

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: mem-stress
  namespace: chaos-mesh
spec:
  mode: all
  selector:
    namespaces:
      - hello-kubernetes
  stressors:
    memory:
      workers: 4
      size: 50MiB
      options: ['']
  duration: '1h'
```

このyamlをファイルに保存します。私は`memory.yaml`と名付けました。カオスを適用するには、次のコマンドを実行します：

```bash
~ kubectl apply -f memory.yaml
stresschaos.chaos-mesh.org/mem-stress created
```

では、再度メモリ使用量を確認しましょう。

```sh
              used
Mem:          4332
Swap:            0

Mem: 12805568K used
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
   54    50 root     R    53252   0%   1  24% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   57    52 root     R    53252   0%   0  22% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   55    53 root     R    53252   0%   2  21% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   56    51 root     R    53252   0%   3  21% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   18     1 node     S     289m   2%   2   0% node server.js
    1     0 node     S     285m   2%   0   0% npm start
   51    49 root     S    41048   0%   0   0% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   50    49 root     S    41048   0%   2   0% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   52    49 root     S    41048   0%   0   0% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   53    49 root     S    41048   0%   3   0% {stress-ng-vm} stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   49     0 root     S    41044   0%   0   0% stress-ng --vm 4 --vm-keep --vm-bytes 50000000
   29     0 node     S     1636   0%   3   0% /bin/sh
   48    29 node     R     1568   0%   1   0% top
```

stress-ngのインスタンスがPodに注入されていることがわかります。Podのメモリ使用量は60 MiB増加しており、これは予期していませんでした。[ドキュメント](https://manpages.ubuntu.com/manpages/focal/en/man1/stress-ng.1.html)によれば、増加量は200 MiB（4 * 50 MiB）であるべきです。

メモリ負荷を50 MiBから3,000 MiBに増やしてみましょう。これでPodのメモリ制限を超えるはずです。カオスを削除し、サイズを変更して再適用します。

すると、突然！シェルが終了コード137で終了します。しばらくしてコンテナに再接続すると、メモリ使用量は正常に戻っています。stress-ngのインスタンスは見つかりません！何が起こったのでしょうか？

## StressChaosはなぜ消えたのか？

Kubernetesは[cgroup](https://man7.org/linux/man-pages/man7/cgroups.7.html)というメカニズムを通じてコンテナのメモリ使用量を制限します。Podの500 MiB制限を確認するには、コンテナに入り次のコマンドを入力します：

```bash
/usr/src/app $ cat /sys/fs/cgroup/memory/memory.limit_in_bytes
524288000
```

出力はバイト単位で表示され、`500 * 1024 * 1024`に変換されます。

リクエストはPodの配置スケジューリングにのみ使用されます。Pod自体にはメモリ制限やリクエストはありませんが、すべてのコンテナの合計と見なすことができます。

最初から間違っていました。freeとtopは「cgroup化」されていません。これらはデータ取得に`/proc/meminfo`（procfs）を利用しています。残念ながら`/proc/meminfo`は古く、cgroup以前の時代のものです。これではコンテナではなく**ホスト**のメモリ情報が表示されます。最初からやり直して、今回は正しいメモリ使用量を確認しましょう。

cgroup化されたメモリ使用量を取得するには、次のコマンドを実行します：

```sh
/usr/src/app $ cat /sys/fs/cgroup/memory/memory.usage_in_bytes
39821312
```

50 MiBのStressChaosを適用すると、以下の結果が得られます：

```sh
/usr/src/app $ cat /sys/fs/cgroup/memory/memory.usage_in_bytes
93577216
```

これはStressChaosなしの場合と比べて約51 MiB多いメモリ使用量です。

次に、なぜシェルが終了したのでしょうか？終了コード137は「コンテナがSIGKILLを受信した失敗」を示しています。これによりPodの状態とイベントを確認する必要があります。

```bash
~ kubectl describe pods -n hello-kubernetes
......
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
......
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
......
  Warning  Unhealthy  10m (x4 over 16m)    kubelet            Readiness probe failed: Get "http://10.244.1.19:8080/": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
  Normal   Killing    10m (x2 over 16m)    kubelet            Container hello-kubernetes failed liveness probe, will be restarted
......
```

イベントからシェルクラッシュの原因がわかります。`hello-kubernetes`にはliveness probeがあり、コンテナのメモリが制限に達するとアプリケーションが失敗し始め、Kubernetesはそれを終了して再起動することを決定します。Podが再起動するとStressChaosは停止します。この場合、カオスは正常に機能していると言えます。それはPodの脆弱性を発見しました。修正してから再度カオスを適用できます。今ではすべてが完璧に見えますが、一つだけ疑問が残ります。なぜ4つの50 MiB vmワーカーで合計51 MiBになるのでしょうか？答えはstress-ngのソースコード[こちら](https://github.com/ColinIanKing/stress-ng/blob/819f7966666dafea5264cf1a2a0939fd344fcf08/stress-vm.c#L2074)を見るまでわかりません：

```c
vm_bytes /= args->num_instances;
```

おっと！ドキュメントが間違っていました。複数のvmワーカーは指定された総サイズを占有し、ワーカーごとにその量のメモリを`mmap`するわけではありません。これですべての疑問が解けました。次のセクションでは、メモリ負荷に関連する他の状況について議論します。

## liveness probeがない場合はどうなるか？

probeを削除してもう一度試してみましょう。`deploy/helm/hello-kubernetes/templates/deployment.yaml`から以下の行を見つけて削除します。

```yaml
livenessProbe:
  httpGet:
    path: /
    port: http
readinessProbe:
  httpGet:
    path: /
    port: http
```

その後、デプロイメントをアップグレードします。

このシナリオで興味深いのは、メモリ使用量が連続的に上昇した後、急激に下降することを繰り返すことです。何が起こっているのでしょうか？カーネルログを確認しましょう。最後の2行に注目してください。

```sh
/usr/src/app $ dmesg
......
[189937.362908] [ pid ]   uid  tgid total_vm      rss nr_ptes swapents oom_score_adj name
[189937.363092] [441060]  1000 441060    63955     3791      80     3030           988 node
[189937.363110] [441688]     0 441688   193367     2136     372   181097          1000 stress-ng-vm
......
[189937.363148] Memory cgroup out of memory: Kill process 443160 (stress-ng-vm) score 1272 or sacrifice child
[189937.363186] Killed process 443160 (stress-ng-vm), UID 0, total-vm:773468kB, anon-rss:152704kB, file-rss:164kB, shmem-rss:0kB
```

出力から明らかなように、`stress-ng-vm`プロセスはメモリ不足（OOM）エラーによりkillされています。

プロセスが必要なメモリを取得できない場合、事態は複雑になります。プロセスは非常に高い確率で失敗します。プロセスがクラッシュするのを待つよりも、いくつかのプロセスをkillしてメモリを確保する方が良いでしょう。OOM killerは一定の順序でプロセスを停止し、最も少ない問題で最大のメモリを回収しようとします。このプロセスの詳細については、OOM killerの[この解説](https://lwn.net/Articles/391222/)を参照してください。

上記の出力を見ると、アプリケーションプロセスである`node`の`oom_score_adj`が988であることがわかります。これは非常に危険な状態です。なぜなら、このスコアが最も高いプロセスが最初にkillされるためです。しかし、特定のプロセスがOOM killerによってkillされないようにする簡単な方法があります。Podを作成すると、Quality of Service (QoS)クラスが割り当てられます。詳細については、[Configure Quality of Service for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)を参照してください。

一般的に、リソースリクエストを正確に指定して作成されたPodは`Guaranteed` Podとして分類されます。OOM killerは、他のkill対象（非`Guaranteed` Podやstress-ngワーカーなど）が存在する場合、`Guaranteed` Pod内のコンテナをkillしません。一方、リソースリクエストが指定されていないPodは`BestEffort`としてマークされ、OOM killerによって最初にkillされます。

以上が今回の解説のすべてです。コンテナのメモリを評価する際に`free`や`top`を使用すべきではないというのが私たちのアドバイスです。Podにリソース制限を割り当てる際には注意し、適切なQoSを選択してください。今後、より詳細なStressChaosドキュメントを作成する予定です。

## Kubernetesのメモリ管理の詳細

Kubernetesは、メモリを過剰に使用している（ただし制限値を超えていない）Podをevictしようとします。KubernetesはPodのメモリ使用量を`/sys/fs/cgroup/memory/memory.usage_in_bytes`から取得し、`memory.stat`内の`total_inactive_file`の値を差し引きます。

Kubernetesは**swapをサポートしていない**ことに注意してください。たとえswapが有効なノードであっても、Kubernetesは`swappiness=0`でコンテナを作成するため、実質的にswapは無効化されます。これは主にパフォーマンス上の理由によるものです。

`memory.usage_in_bytes`は`resident set`と`cache`の合計であり、`total_inactive_file`はメモリが不足した際にOSが回収可能なキャッシュメモリです。`memory.usage_in_bytes - total_inactive_file`は`working_set`と呼ばれます。この`working_set`の値は`kubectl top pod <your pod> --containers`で取得できます。Kubernetesはこの値を使用してPodをevictするかどうかを決定します。

Kubernetesは定期的にメモリ使用量を検査します。コンテナのメモリ使用量が急激に増加したり、コンテナをevictできない場合、OOM killerが呼び出されます。Kubernetesは自身のプロセスを保護する方法を持っているため、常にコンテナが選択されます。コンテナがkillされると、再起動ポリシーに応じて再起動される場合とされない場合があります。killされた場合、`kubectl describe pod <your pod>`を実行すると、再起動されていてその理由が`OOMKilled`であることがわかります。

もう一つ注目すべきはカーネルメモリです。Kubernetesのカーネルメモリサポートは`v1.9`以降デフォルトで有効化されています。これはcgroupメモリサブシステムの機能でもあります。コンテナのカーネルメモリ使用量を制限できますが、残念ながら`v4.2`までのカーネルバージョンではcgroupリークを引き起こします。この問題を解決するには、カーネルを`v4.3`にアップグレードするか、この機能を無効化する必要があります。

## StressChaosの実装方法

StressChaosは、コンテナがメモリ不足状態になった際の挙動をテストする簡単な方法です。StressChaosは強力なツールである`stress-ng`を利用してメモリを割り当て、割り当てたメモリへの書き込みを継続します。コンテナにはメモリ制限があり、これらの制限はcgroupに紐づいているため、特定のcgroupで`stress-ng`を実行する方法を見つける必要がありました。幸い、この部分は簡単でした。十分な権限があれば、`/sys/fs/cgroup/`内のファイルに書き込むことで任意のプロセスを任意のcgroupに割り当てることができます。

Chaos Meshに興味があり、改善に協力したい方は、ぜひ私たちの[Slackチャンネル](https://slack.cncf.io/) (#project-chaos-mesh)に参加してください！または、[GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh)にプルリクエストやイシューを投稿してください。