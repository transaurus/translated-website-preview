---
title: Simulate Time Faults
summary: This document describes how to use Chaosd to simulate a time offset scenario.
---

このドキュメントでは、Chaosdを使用して時間オフセットのシナリオをシミュレートする方法について説明します。コマンドラインモードまたはサービスモードのいずれかで実験を作成できます。

## コマンドラインモードを使用した実験の作成

このセクションでは、コマンドを使用して時間障害実験を作成する方法について説明します。

実験を作成する前に、次のコマンドを実行して時間障害のオプションを確認できます：

```
chaosd attack clock -h
```

結果は以下の通りです：

```bash
$ chaosd attack clock -h

clock skew

Usage:
  chaosd attack clock attack [flags]

Flags:
  -c, --clock-ids-slice string   The identifier of the particular clock on which to act.More clock description in linux kernel can be found in man page of clock_getres, clock_gettime, clock_settime.Muti clock ids should be split with "," (default "CLOCK_REALTIME")
  -h, --help                     help for clock
  -p, --pid int                  Pid of target program.
  -t, --time-offset string       Specifies the length of time offset.

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID

```

### クイック例

テストプログラムを準備します：

```bash
cat > time.c << EOF
#include <stdio.h>
#include <time.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    printf("PID : %ld\n", (long)getpid());
    struct  timespec ts;
    for(;;) {
        clock_gettime(CLOCK_REALTIME, &ts);
        printf("Time : %lld.%.9ld\n", (long long)ts.tv_sec, ts.tv_nsec);
        sleep(10);
    }
}
EOF

gcc -o get_time ./time.c
```

次に、get_timeを実行して攻撃を試みます。以下は例です：

```bash
chaosd attack clock -p $PID -t 11s
```

### 時間障害のシミュレーション設定

| Parameter | Type | Note | Default value | Required | Example |
| --- | --- | --- | --- | --- | --- |
| timeOffset | string | Specifies the length of time offset. | None | Yes | `-5m` |
| clockIds | []string | Specifies the ID of clock that will be offset. See the [clock_gettime documentation](https://man7.org/linux/man-pages/man2/clock_gettime.2.html) for details. | `["CLOCK_REALTIME"]` | No | `["CLOCK_REALTIME", "CLOCK_MONOTONIC"]` |
| pid | string | The identifier of the process. | None | Yes | `1` |

### サービスモードを使用した実験の作成

（更新中）