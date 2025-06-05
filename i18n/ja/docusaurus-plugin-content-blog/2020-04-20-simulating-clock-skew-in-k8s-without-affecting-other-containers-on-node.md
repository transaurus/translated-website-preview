---
slug: /simulating-clock-skew-in-k8s-without-affecting-other-containers-on-node
title: Simulating Clock Skew in K8s Without Affecting Other Containers on the Node
authors: cwen
image: /img/blog/clock-sync-chaos-engineering-k8s.jpg
tags: [Chaos Mesh, Chaos Engineering, Kubernetes, Distributed System]
---

![分散システムにおけるクロック同期](/img/blog/clock-sync-chaos-engineering-k8s.jpg)

[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)は、Kubernetes（K8s）向けの使いやすいオープンソースのクラウドネイティブなカオスエンジニアリングプラットフォームです。新機能のTimeChaosは、[クロックスキュー](https://en.wikipedia.org/wiki/Clock_skew#On_a_network)現象をシミュレートします。通常、コンテナ内のクロックを変更する際には、[影響範囲を最小限](https://learning.oreilly.com/library/view/chaos-engineering/9781491988459/ch07.html)に抑え、ノード上の他のコンテナに影響を与えたくありません。しかし、実際にこれを実装するのは想像以上に難しい場合があります。Chaos Meshはこの問題をどのように解決しているのでしょうか？

<!--truncate-->

この記事では、クロックスキューのさまざまなアプローチを試行錯誤した経緯と、Chaos MeshのTimeChaosがコンテナ内で時間を自由に操作できる仕組みについて説明します。

## ノード上の他のコンテナに影響を与えずにクロックスキューをシミュレート

クロックスキューとは、ネットワーク内のノード間のクロックの時間差を指します。分散システムにおいて信頼性の問題を引き起こす可能性があり、複雑な分散システムの設計者や開発者にとって懸念事項です。例えば、分散SQLデータベースでは、一貫性のあるグローバルスナップショットを実現し、トランザクションのACID特性を保証するために、ノード間で同期されたローカルクロックを維持することが重要です。

現在、[クロック同期のための確立された解決策](https://pingcap.com/blog/Time-in-Distributed-Systems/)がありますが、適切なテストなしでは、実装が堅牢であることを確認することはできません。

では、分散システムでグローバルスナップショットの一貫性をどのようにテストすればよいでしょうか？答えは明らかです：異常なクロック条件下で分散システムが一貫性のあるグローバルスナップショットを維持できるかどうかをテストするために、クロックスキューをシミュレートできます。一部のテストツールはコンテナ内でクロックスキューをシミュレートできますが、物理ノードに影響を与えます。

[TimeChaos](https://github.com/chaos-mesh/chaos-mesh/wiki/Time-Chaos)は、**ノード全体に影響を与えずにコンテナ内でクロックスキューをシミュレートし、アプリケーションへの影響をテストする**ツールです。これにより、クロックスキューが引き起こす潜在的な影響を正確に特定し、適切な対策を講じることができます。

## 試行したクロックスキューシミュレーションのさまざまなアプローチ

既存の選択肢を振り返ると、Kubernetes上で動作するChaos Meshには適用できないことが明らかです。クロックスキューをシミュレートする一般的な2つの方法（ノードクロックを直接変更する方法とJepsenフレームワークを使用する方法）は、ノード上のすべてのプロセスの時間を変更します。これらは私たちにとって許容できる解決策ではありません。Kubernetesコンテナで、ノード全体に影響を与えるクロックスキューエラーを注入すると、同じノード上の他のコンテナが干渉されます。このような粗雑なアプローチは許容できません。

では、この問題にどう対処すればよいでしょうか？まず思い浮かぶのは、カーネル内で[Berkeley Packet Filter](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter)（BPF）を使用して解決策を見つけることです。

### `LD_PRELOAD`

`LD_PRELOAD`は、プログラム実行前にどの動的リンクライブラリをロードするかを定義できるLinux環境変数です。

この変数には2つの利点があります：

- ソースコードを意識せずに独自の関数を呼び出せる。
- 特定の目的を達成するために他のプログラムにコードを注入できる。

RustやCなど、glibcの時間関数を呼び出すアプリケーションを使用する言語では、`LD_PRELOAD`を使用するだけでクロックスキューをシミュレートできます。しかし、Golangの場合はより複雑です。Golangなどの言語は、システムコールを高速化するメカニズムである仮想Dynamic Shared Object（[vDSO](http://man7.org/linux/man-pages/man7/vdso.7.html)）を直接解析するため、単純に`LD_PRELOAD`を使用してglicインターフェースをインターセプトすることはできません。したがって、`LD_PRELOAD`は私たちの解決策ではありません。

### BPFを使用して`clock_gettime`システムコールの戻り値を変更

また、BPFを使用してタスクの[プロセス識別番号](http://www.linfo.org/pid.html)（PID）をフィルタリングする方法も試しました。これにより、特定のプロセスに対してクロックスキューをシミュレートし、`clock_gettime`システムコールの戻り値を変更することが可能になります。

これは良いアイデアのように思えましたが、問題にも直面しました。ほとんどの場合、vDSOは`clock_gettime`を高速化しますが、`clock_gettime`はシステムコールを行いません。この選択肢も機能しませんでした。残念です。

幸いなことに、システムカーネルのバージョンが4.18以降で、[HPET](https://www.kernel.org/doc/html/latest/timers/hpet.html)クロックを使用している場合、`clock_gettime()`はvDSOではなく通常のシステムコールによって時間を取得することがわかりました。このアプローチを使用して[クロックスキューのバージョン](https://github.com/chaos-mesh/bpfki)を実装し、RustとCでは問題なく動作しました。しかし、Golangの場合、プログラムは正しく時間を取得できますが、クロックスキューの注入中に`sleep`を実行すると、sleep操作がブロックされる可能性が非常に高くなります。注入がキャンセルされた後でも、システムは回復できません。そのため、このアプローチも断念せざるを得ませんでした。

## TimeChaos、私たちの最終的な解決策

前のセクションから、プログラムは通常`clock_gettime`を呼び出してシステム時間を取得することがわかります。私たちのケースでは、`clock_gettime`はvDSOを使用して呼び出しプロセスを高速化するため、`LD_PRELOAD`を使用して`clock_gettime`システムコールをハックすることはできません。

原因がわかったところで、解決策は何でしょうか？ vDSOから始めます。vDSOに保存されている`clock_gettime`の戻り値のアドレスを、私たちが定義したアドレスにリダイレクトできれば、問題を解決できます。

言うは易く行うは難し。この目標を達成するためには、以下の問題に取り組む必要があります：

- vDSOが使用するユーザーモードのアドレスを知る
- vDSOのカーネルモードのアドレスを知る（カーネルモードの任意のアドレスでvDSOの`clock_gettime`関数を変更したい場合）
- vDSOデータを変更する方法を知る

まず、vDSOの中を覗く必要があります。`/proc/pid/maps`でvDSOのメモリアドレスを確認できます。

```
$ cat /proc/pid/maps
...
7ffe53143000-7ffe53145000 r-xp 00000000 00:00 0                     [vdso]
```

最後の行がvDSOの情報です。このメモリ空間の権限は`r-xp`：読み取り可能で実行可能ですが、書き込み不可です。つまり、ユーザーモードではこのメモリを変更できません。[ptrace](http://man7.org/linux/man-pages/man2/ptrace.2.html)を使用してこの制限を回避できます。

次に、`gdb dump memory`を使用してvDSOをエクスポートし、`objdump`を使用して中身を確認します。以下が得られた結果です：

```
(gdb) dump memory vdso.so 0x00007ffe53143000 0x00007ffe53145000
$ objdump -T vdso.so
vdso.so:    file format elf64-x86-64
DYNAMIC SYMBOL TABLE:
ffffffffff700600  w  DF .text   0000000000000545  LINUX_2.6  clock_gettime
```

vDSO全体が`.so`ファイルのように見えることがわかります。したがって、実行可能およびリンク可能形式（ELF）ファイルを使用してフォーマットできます。この情報をもとに、TimeChaosを実装するための基本的なワークフローが形になり始めます：

![TimeChaosのワークフロー](/img/blog/timechaos-workflow.jpg)

上の図は、Chaos Meshにおけるクロックスキューの実装である**TimeChaos**のプロセスです。

1. ptraceを使用して指定されたPIDプロセスにアタッチし、現在のプロセスを停止します。
2. ptraceを使用して、呼び出しプロセスの仮想アドレス空間に新しいマッピングを作成し、[`process_vm_writev`](https://linux.die.net/man/2/process_vm_writev)を使用して定義した`fake_clock_gettime`関数をメモリ空間に書き込みます。
3. `process_vm_writev`を使用して、指定されたパラメータを`fake_clock_gettime`に書き込みます。これらのパラメータは、注入したい時間（例えば2時間遅れや2日進みなど）です。
4. ptraceを使用してvDSOの`clock_gettime`関数を変更し、`fake_clock_gettime`関数にリダイレクトします。
5. ptraceを使用してPIDプロセスをデタッチします。

詳細に興味がある方は、[Chaos Mesh GitHubリポジトリ](https://github.com/chaos-mesh/chaos-mesh/blob/release-1.0/pkg/time/time_linux.go)をご覧ください。

## 分散型SQLデータベースでの時刻ずれシミュレーション

統計は雄弁です。ここでは、オープンソースの[NewSQL](https://en.wikipedia.org/wiki/NewSQL)分散型SQLデータベースであり、[Hybrid Transactional/Analytical Processing](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing)（HTAP）ワークロードをサポートする[TiDB](https://docs.pingcap.com/tidb/stable/overview/)に対してTimeChaosを試し、カオステストが実際に機能するかどうかを確認します。

TiDBは、グローバルに一貫したバージョン番号を取得し、トランザクションのバージョン番号が単調増加することを保証するために、集中型サービスTimestamp Oracle（TSO）を使用しています。TSOサービスはPlacement Driver（PD）コンポーネントによって管理されています。したがって、ランダムに選択したPDノードに対して定期的にTimeChaosを注入し、各注入で10ミリ秒の時刻を遡らせます。TiDBがこの課題に対処できるかどうかを見てみましょう。

テストをより効果的に行うために、銀行システムでの金融取引をシミュレートする[bank](https://github.com/cwen0/bank)をワークロードとして使用します。これはデータベーストランザクションの正確性を検証するためによく使用されます。

これが私たちのテスト設定です：

```
apiVersion: chaos-mesh.org/v1alpha1
kind: TimeChaos
metadata:
  name: time-skew-example
  namespace: tidb-demo
spec:
  mode: one
  selector:
    labelSelectors:
      "app.kubernetes.io/component": "pd"
  timeOffset:
    sec: -600
  clockIds:
    - CLOCK_REALTIME
  duration: "10s"
  scheduler:
    cron: "@every 1m"
```

このテスト中、Chaos Meshは選択したPD Podに対して1ミリ秒ごとに10秒間TimeChaosを注入します。この期間中、PDが取得する時刻は実際の時刻から600秒のオフセットを持ちます。詳細については、[Chaos Mesh Wiki](https://github.com/chaos-mesh/chaos-mesh/wiki/Time-Chaos)をご覧ください。

`kubectl apply`コマンドを使用してTimeChaos実験を作成しましょう：

```
kubectl apply -f pd-time.yaml
```

次のコマンドでPDログを取得できます：

```
kubectl logs -n tidb-demo tidb-app-pd-0 | grep "system time jump backward"
```

ログは以下の通りです：

```
[2020/03/24 09:06:23.164 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585041383060109693]
[2020/03/24 09:16:32.260 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585041992160476622]
[2020/03/24 09:20:32.059 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042231960027622]
[2020/03/24 09:23:32.059 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042411960079655]
[2020/03/24 09:25:32.059 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042531963640321]
[2020/03/24 09:28:32.060 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585042711960148191]
[2020/03/24 09:33:32.063 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043011960517655]
[2020/03/24 09:34:32.060 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043071959942937]
[2020/03/24 09:35:32.059 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043131978582964]
[2020/03/24 09:36:32.059 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043191960687755]
[2020/03/24 09:38:32.060 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043311959970737]
[2020/03/24 09:41:32.060 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043491959970502]
[2020/03/24 09:45:32.061 +00:00] [ERROR] [systime_mon.go:32] ["system time jump backward"] [last=1585043731961304629]
...
```

上記のログから、PDが定期的にシステム時刻が巻き戻されたことを検出していることがわかります。これは次のことを意味します：

- TimeChaosが時刻ずれを正常にシミュレートしている。
- PDが時刻ずれの状況に対処できる。

これは励みになります。しかし、TimeChaosはPD以外のサービスに影響を与えていないでしょうか？Chaos Dashboardで確認できます：

![Chaos Dashboard](/img/blog/chaos-dashboard.jpg)

モニターでは、TimeChaosが1ミリ秒ごとに注入され、全体の期間が10秒間続いたことが明確です。さらに、TiDBはその注入の影響を受けていません。bankプログラムは正常に動作し、パフォーマンスにも影響はありませんでした。

## Chaos Meshを試す

クラウドネイティブなカオスエンジニアリングプラットフォームとして、Chaos MeshはKubernetes上の複雑なシステムに対する包括的な[障害注入方法](https://pingcap.com/blog/chaos-mesh-your-chaos-engineering-solution-for-system-resiliency-on-kubernetes/)を特徴としており、Pod、ネットワーク、ファイルシステム、さらにはカーネルに至るまでの障害をカバーしています。

カオスエンジニアリングを実際に体験してみたいですか？[Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh)へようこそ。この[10分チュートリアル](https://pingcap.com/blog/run-first-chaos-experiment-in-ten-minutes/)で、カオスエンジニアリングを迅速に始め、Chaos Meshを使用して最初のカオス実験を実行できます。