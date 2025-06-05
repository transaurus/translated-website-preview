---
title: Simulate Network Faults
---

このドキュメントでは、Chaosdを使用してネットワーク障害をシミュレートする方法を紹介します。iptables、ipsets、tcなどを使用してネットワークルーティングやトラフィックフロー制御を変更することでシミュレーションを完了できます。

:::note

LinuxカーネルにNET_SCH_NETEMモジュールがインストールされていることを確認してください。CentOSを使用している場合、kernel-modules-extraパッケージを通じてこのモジュールをインストールできます。他のほとんどのLinuxディストリビューションではデフォルトでインストールされています。

:::

## コマンドラインモードでネットワーク障害実験を作成

このセクションでは、コマンドラインモードを使用してネットワーク障害実験を作成する方法を説明します。

実験を作成する前に、以下のコマンドを実行してChaosdがサポートするネットワーク障害の種類を確認できます：

```bash
chaosd attack network --help
```

出力は以下のようになります：

```bash
Network attack related commands

Usage:
  chaosd attack network [command]

Available Commands:
  bandwidth   limit network bandwidth
  corrupt     corrupt network packet
  delay       delay network
  dns attack  DNS server or map specified host to specified IP
  duplicate   duplicate network packet
  loss        loss network packet
  partition   partition
  port        attack network port

Flags:
  -h, --help   help for network

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'

Use "chaosd attack network [command] --help" for more information about a command.
```

現在、Chaosdを使用して4つの実験シナリオをシミュレートできます：ネットワーク破損、ネットワーク遅延、ネットワーク重複、ネットワーク損失。

### コマンドラインモードでネットワーク破損をシミュレート

Chaosdを使用したネットワーク破損の設定を確認するために、以下のコマンドを実行できます。

#### ネットワーク破損のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network corrupt --help
```

出力は以下のようになります：

```bash
corrupt network packet

Usage:
  chaosd attack network corrupt [flags]

Flags:
  -c, --correlation string   correlation is percentage (10 is 10%) (default "0")
  -d, --device string        the network interface to impact
  -e, --egress-port string   only impact egress traffic to these destination ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp
  -h, --help                 help for corrupt
  -H, --hostname string      only impact traffic to these hostnames
  -i, --ip string            only impact egress traffic to these IP addresses
      --percent string       percentage of packets to corrupt (10 is 10%) (default "1")
  -p, --protocol string      only impact traffic using this IP protocol, supported: tcp, udp, icmp, all
  -s, --source-port string   only impact egress traffic from these source ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### ネットワーク破損に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| correlation | c | The correlation between the percentage of current corrupt occurrence and the previous occurrence. | int. It is a percentage ranging from 0 to 100 (10 is 10%) ("0" by default ). |
| device | d | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | e | The egress traffic that only impacts specific destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | H | The host name impacted by traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "chaos-mesh.org". |
| ip | i | The IP address impacted by egress traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "123.123.123.123". |
| protocol | p | The IP protocol impacted by traffic. | string. Supported protocols: tcp, udp, icmp, all (all network protocols). |
| source-port | s | The egress traffic which only impact specific source ports. It can only be configured when the protocol is TCP or UDP. | string. Use a `,` to delimit the specific port or to indicate the range of the ports, such as "80,8001:8010". |
| percent |  | Ratio of network packet corruption. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "1"). |

#### コマンドモードでネットワーク破損をシミュレートする例

ネットワーク破損をシミュレートするために以下のコマンドを実行します：

```bash
chaosd attack network corrupt -d eth0 -i 172.16.4.4 --percent 50
```

コマンドが正常に実行されると、出力は以下のようになります：

```bash
Attack network successfully, uid: 4eab1e62-8d60-45cb-ac85-3c17b8ac4825
```

### コマンドラインモードでネットワーク遅延をシミュレート

Chaosdを使用したネットワーク遅延の設定を確認するために、以下のコマンドを実行できます。

#### ネットワーク遅延のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network delay --help
```

出力は以下のようになります：

```bash
delay network

Usage:
  chaosd attack network delay [flags]

Flags:
  -c, --correlation string   correlation is percentage (10 is 10%) (default "0")
  -d, --device string        the network interface to impact
  -e, --egress-port string   only impact egress traffic to these destination ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp
  -h, --help                 help for delay
  -H, --hostname string      only impact traffic to these hostnames
  -i, --ip string            only impact egress traffic to these IP addresses
  -j, --jitter string        jitter time, time units: ns, us (or µs), ms, s, m, h.
  -l, --latency string       delay egress time, time units: ns, us (or µs), ms, s, m, h.
  -p, --protocol string      only impact traffic using this IP protocol, supported: tcp, udp, icmp, all
  -s, --source-port string   only impact egress traffic from these source ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### ネットワーク遅延に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| correlation | c | The correlation between the current latency and the previous one. | string. It is a percentage ranging from 0 to 100 (10 is 10%) ("0" by default). |
| device | d | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | e | The egress traffic which only impact specific destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | H | The host name impacted by traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "chaos-mesh.org". |
| ip | i | The IP address impacted by egress traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "123.123.123.123". |
| jitter | j | Range of the length of network delay time. | string. The time units can be: ns, us (µs), ms, s, m, h, such as "1ms". |
| latency | l | Length of network delay time. | string. The time units can be: ns, us (μs), ms, s, m, h, such as "1ms". |
| protocol | p | The IP protocol impacted by traffic. | string. It supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |
| source-port | s | The egress traffic that only impacts specified source ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |

#### コマンドラインモードでネットワーク遅延をシミュレートする例

ネットワーク遅延をシミュレートするために以下のコマンドを実行します：

```bash
chaosd attack network delay -d eth0 -i 172.16.4.4 -l 10ms
```

コマンドが正常に実行されると、出力は以下のようになります：

```bash
Attack network successfully, uid: 4b23a0b5-e193-4b27-90a7-3e04235f32ab
```

### コマンドラインモードでネットワーク重複をシミュレート

Chaosdを使用したネットワーク重複の設定を確認するために、以下のコマンドを実行できます。

#### ネットワーク重複のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network duplicate --help
```

出力は以下のようになります：

```bash
duplicate network packet

Usage:
  chaosd attack network duplicate [flags]

Flags:
  -c, --correlation string   correlation is percentage (10 is 10%) (default "0")
  -d, --device string        the network interface to impact
  -e, --egress-port string   only impact egress traffic to these destination ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp
  -h, --help                 help for duplicate
  -H, --hostname string      only impact traffic to these hostnames
  -i, --ip string            only impact egress traffic to these IP addresses
      --percent string       percentage of packets to duplicate (10 is 10%) (default "1")
  -p, --protocol string      only impact traffic using this IP protocol, supported: tcp, udp, icmp, all
  -s, --source-port string   only impact egress traffic from these source ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### ネットワーク重複に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| correlation | c | The correlation between the percentage of current duplication occurrence and the previous one. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "0"). |
| device | d | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | e | The egress traffic that only impacts specified destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | H | The host name impacted by traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "chaos-mesh.org". |
| ip | i | The IP address impacted by egress traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "123.123.123.123". |
| percent | N/A | Ratio of network packet duplicate. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "1"). |
| protocol | p | The IP protocol impacted by traffic. | string. It supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |
| source-port | s | The egress traffic which only impact specific source ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |

#### コマンドラインモードでネットワーク重複をシミュレートする例

ネットワーク重複をシミュレートするために以下のコマンドを実行します：

```bash
chaosd attack network duplicate -d eth0 -i 172.16.4.4 --percent 50
```

コマンドが正常に実行されると、出力は以下のようになります：

```bash
Attack network successfully, uid: 7bcb74ee-9101-4ae4-82f0-e44c8a7f113c
```

### コマンドラインモードでネットワーク損失をシミュレート

Chaosdを使用したネットワーク損失の設定を確認するために、以下のコマンドを実行できます：

#### ネットワーク損失のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network loss --help
```

出力は以下の通りです：

```bash
loss network packet

Usage:
  chaosd attack network loss [flags]

Flags:
  -c, --correlation string   correlation is percentage (10 is 10%) (default "0")
  -d, --device string        the network interface to impact
  -e, --egress-port string   only impact egress traffic to these destination ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp
  -h, --help                 help for loss
  -H, --hostname string      only impact traffic to these hostnames
  -i, --ip string            only impact egress traffic to these IP addresses
      --percent string       percentage of packets to drop (10 is 10%) (default "1")
  -p, --protocol string      only impact traffic using this IP protocol, supported: tcp, udp, icmp, all
  -s, --source-port string   only impact egress traffic from these source ports, use a ',' to separate or to indicate the range, such as 80, 8001:8010. It can only be used in conjunction with -p tcp or -p udp

Global Flags:
      --log-level string   the log level of chaosd, the value can be 'debug', 'info', 'warn' and 'error'
```

#### ネットワーク損失に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| correlation | c | The correlation between the percentage of the current network loss and the previous one. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "0"). |
| device | d | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | e | The egress traffic that only impacts specified destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | H | The host name impacted by traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "chaos-mesh.org". |
| ip | i | The IP address impacted by egress traffic. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "123.123.123.123". |
| percent | N/A | Ratio of network packet loss. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "1"). |
| protocol | p | Only impact traffic using this IP protocol. | string. It supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |
| source-port | s | The egress traffic which only impact specific source ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |

#### コマンドラインモードでネットワーク損失をシミュレートする例

以下のコマンドを実行してネットワーク損失をシミュレートします：

```bash
chaosd attack network loss -d eth0 -i 172.16.4.4 --percent 50
```

コマンドが正常に実行されると、出力は以下の通りです：

```bash
Attack network successfully, uid: 1e818adf-3942-4de4-949b-c8499f120265
```

### コマンドラインモードでネットワークパーティションをシミュレート

以下のコマンドを実行して、Chaosdを使用したネットワークパーティションの設定を確認できます。

#### ネットワークパーティションのコマンド

コマンドは以下の通りです：

```bash
chaosd attack network partition --help
```

出力は以下の通りです：

```bash
partition

Usage:
  chaosd attack network partition [flags]

Flags:
      --accept-tcp-flags string   only the packet which match the tcp flag can be accepted, others will be dropped. only set when the protocol is tcp.
  -d, --device string             the network interface to impact
      --direction string          specifies the partition direction, values can be 'to', 'from' or 'both'. 'from' means packets coming from the 'IPAddress' or 'Hostname' and going to your server, 'to' means packets originating from your server and going to the 'IPAddress' or 'Hostname'. (default "both")
  -h, --help                      help for partition
  -H, --hostname string           only impact traffic to these hostnames
  -i, --ip string                 only impact egress traffic to these IP addresses
  -p, --protocol string           only impact traffic using this IP protocol, supported: tcp, udp, icmp, all

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### ネットワークパーティションに関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| accept-tcp-flags | N/A | Only the packet which matches the tcp flag can be accepted, others will be dropped. Only set when the protocol is tcp. | string, such as "SYN,ACK SYN,ACK" |
| device | d | the network interface to impact | string, such as "eth0", required |
| direction | d | Specifies the partition direction, values can be 'to', 'from' or 'both'. 'from' means packets coming from the 'ip' or 'hostname' and going to your server, 'to' means packets originating from your server and going to the 'ip' or 'hostname'. | string, values can be "to", "from" or "both" (default "both") |
| hostname | H | Only impact traffic to these hostnames. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "chaos-mesh.org". |
| ip | i | Only impact egress traffic to these IP addresses. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "192.168.123.123". |
| protocol | p | Only impact traffic using this IP protocol | string. It supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |

#### コマンドラインモードでネットワークパーティションをシミュレートする例

以下のコマンドを実行してネットワークパーティションをシミュレートします：

```bash
chaosd attack network partition -i 172.16.4.4 -d eth0 --direction from
```

### コマンドラインモードでDNS障害をシミュレート

以下のコマンドを実行して、Chaosdを使用したDNS障害の設定を確認できます。

#### DNS障害のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network dns --help
```

出力は以下の通りです：

```bash
attack DNS server or map specified host to specified IP

Usage:
  chaosd attack network dns [flags]

Flags:
  -d, --dns-domain-name string   map this host to specified IP
  -i, --dns-ip string            map specified host to this IP address
      --dns-server string        update the DNS server in /etc/resolv.conf with this value (default "123.123.123.123")
  -h, --help                     help for dns

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### DNS障害に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| dns-domain-name | d | Map this host to specified IP(dns-ip) | string, such as "chaos-mesh.org". |
| dns-ip | i | Map specified host(dns-domain-name) to this IP address | string, such as "123.123.123.123" |
| dns-server | N/A | Update the DNS server in /etc/resolv.conf with this value | string, default is "123.123.123.123" |

#### コマンドラインモードでDNS障害をシミュレートする例

以下のコマンドを実行して、指定したホストを指定したIPにマッピングすることでDNS障害をシミュレートします：

```bash
chaosd attack network dns --dns-ip 123.123.123.123 --dns-domain-name chaos-mesh.org
```

以下のコマンドを実行して、誤ったDNSサーバーを使用することでDNS障害をシミュレートします：

```bash
chaosd attack network dns --dns-server 123.123.123.123
```

### コマンドラインモードでネットワーク帯域幅をシミュレート

以下のコマンドを実行して、Chaosdを使用したネットワーク帯域幅の設定を確認できます。

#### ネットワーク帯域幅のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network bandwidth --help
```

出力は以下の通りです：

```bash
limit network bandwidth

Usage:
  chaosd attack network bandwidth [flags]

Flags:
  -b, --buffer uint32     the maximum amount of bytes that tokens can be available for instantaneously
  -d, --device string     the network interface to impact
  -h, --help              help for bandwidth
  -H, --hostname string   only impact traffic to these hostnames
  -i, --ip string         only impact egress traffic to these IP addresses
  -l, --limit uint32      the number of bytes that can be queued waiting for tokens to become available
  -m, --minburst uint32   specifies the size of the peakrate bucket
      --peakrate uint     the maximum depletion rate of the bucket
  -r, --rate string       the speed knob, allows bps, kbps, mbps, gbps, tbps unit. bps means bytes per second

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### ネットワーク帯域幅に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description | Value |
| :-- | :-- | :-- | :-- |
| buffer | b | The maximum amount of bytes that tokens can be available for instantaneously | int, such as `10000`, required |
| device | d | The network interface to impact | string, such as "eth0", required |
| hostname | H | Only impact traffic to these hostnames. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "chaos-mesh.org". |
| ip | i | Only impact egress traffic to these IP addresses. `hostname` and `ip` cannot be empty at the same time. When `hostname` and `ip` are set at the same time, the configuration item affects both the specified `hostname` and `ip`. | string, such as "123.123.123.123". |
| limit | l | The number of bytes that can be queued waiting for tokens to become available | int, such as `10000`, required |
| minburst | `m` | Specifies the size of the peakrate bucket | int, such as `10000` |
| peakrate | N/A | The maximum depletion rate of the bucket | int, such as `10000` |
| rate | r | The speed knob, allows bps, kbps, mbps, gbps, tbps unit. The bps unit means bytes per second. | string, such as "1mbps", required |

#### コマンドラインモードでネットワーク帯域幅をシミュレートする例

以下のコマンドを実行してネットワーク帯域幅をシミュレートします：

```bash
chaosd attack network bandwidth --buffer 10000 --device eth0 --limit 10000 --rate 10mbps
```

### コマンドラインモードでポート占有をシミュレート

以下のコマンドを実行して、ポート占有の設定を確認できます。

#### ポート占有のコマンド

コマンドは以下の通りです：

```bash
chaosd attack network port --help
```

出力は以下の通りです：

```bash
attack network port

Usage:
  chaosd attack network port [flags]

Flags:
  -h, --help          help for port
  -p, --port string   this specified port is to occupied

Global Flags:
      --log-level string   the log level of chaosd. The value can be 'debug', 'info', 'warn' and 'error'
      --uid string         the experiment ID
```

#### ポート占有に関連する設定項目

関連する設定項目は以下のように説明されます：

| Configuration item | Abbreviation | Description                       | Value                       |
| :----------------- | :----------- | :-------------------------------- | :-------------------------- |
| port               | p            | The specified port to be occupied | int, such as 8080, required |

#### コマンドラインモードでポート占有をシミュレートする例

ネットワーク帯域幅をシミュレートするには、次のコマンドを実行します：

```bash
chaosd attack network port --port 8080
```

## サービスモードを使用したネットワーク障害実験の作成

サービスモードを使用して実験を作成するには、以下の手順に従ってください：

1. Chaosdをサービスモードで実行します：

   ```bash
   chaosd server --port 31767
   ```

2. Chaosdサービスの`/api/attack/process`パスに`POST` HTTPリクエストを送信します。

   ```bash
   curl -X POST 172.16.112.130:31767/api/attack/process -H "Content-Type:application/json" -d '{fault-configuration}'
   ```

   上記のコマンドでは、障害タイプに応じて`fault-configuration`を設定する必要があります。対応するパラメータについては、以下のセクションの各障害タイプのパラメータと例を参照してください。

:::note

実験を実行する際は、実験のUIDを記録しておいてください。UIDに対応する実験を終了するには、Chaosdサービスの`/api/attack/{uid}`パスに`DELETE` HTTPリクエストを送信する必要があります。

:::

### サービスモードを使用したネットワーク破損のシミュレーション

#### ネットワーク破損をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "corrupt" |
| correlation | The correlation between the current latency and the previous one. | string. It is a percentage ranging from 0 to 100 (10 is 10%) ("0" by default). |
| device | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | The egress traffic which only impact specific destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | The host name impacted by traffic. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "chaos-mesh.org". |
| ip-address | The IP address impacted by egress traffic. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "123.123.123.123". |
| ip-protocol | The IP protocol impacted by traffic. | string. Supported protocols: tcp, udp, icmp, all (all network protocols). |
| source-port | The egress traffic which only impact specific source ports. It can only be configured when the protocol is TCP or UDP. | string. Use a `,` to delimit the specific port or to indicate the range of the ports, such as "80,8001:8010". |
| percent | Ratio of network packet corruption. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "1"). |

#### サービスモードを使用したネットワーク破損のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"corrupt","device":"eth0","ip-address":"172.16.4.4","percent":"50"}'
```

### サービスモードを使用したネットワーク遅延のシミュレーション

#### ネットワーク遅延をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "delay" |
| correlation | The correlation between the current latency and the previous one. | string. It is a percentage ranging from 0 to 100 (10 is 10%) ("0" by default). |
| device | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | The egress traffic which only impact specific destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | The host name impacted by traffic. `hostname` and `ip-address` cannot be empty at the same time. When `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "chaos-mesh.org". |
| ip-address | The IP address impacted by egress traffic. `hostname` and `ip-address` cannot be empty at the same time. When `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "123.123.123.123". |
| jitter | Range of the length of network delay time. | string. The time units can be: ns, us (µs), ms, s, m, h, such as "1ms". |
| latency | Length of network delay time. | string. The time units can be: ns, us (μs), ms, s, m, h, such as "1ms". |
| ip-protocol | The IP protocol impacted by traffic. | string. It supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |
| source-port | The egress traffic that only impacts specified source ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |

#### サービスモードを使用したネットワーク遅延のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"delay","device":"eth0","ip-address":"172.16.4.4","latency":"10ms"}'
```

### サービスモードを使用したネットワーク重複のシミュレーション

#### ネットワーク重複をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "duplicate" |
| correlation | The correlation between the percentage of current duplication occurrence and the previous one. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "0"). |
| device | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | The egress traffic that only impacts specified destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | The host name impacted by traffic. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "chaos-mesh.org". |
| ip-address | The IP address impacted by egress traffic. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "123.123.123.123". |
| percent | Ratio of network packet duplicate. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "1"). |
| ip-protocol | The IP protocol impacted by traffic. | string. It supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |
| source-port | The egress traffic which only impact specific source ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |

#### サービスモードを使用したネットワーク重複のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"duplicate","ip-address":"172.16.4.4","device":"eth0","percent":"50"}'
```

### サービスモードを使用したネットワーク損失のシミュレーション

#### ネットワーク損失をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "loss" |
| correlation | The correlation between the percentage of the current network loss and the previous one. | string, it is a percentage which range is 0 to 100 (10 is 10%) (default "0"). |
| device | Name of the impacted network interface card. | string, such as "eth0", required. |
| egress-port | The egress traffic that only impacts specified destination ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |
| hostname | The host name impacted by traffic. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "chaos-mesh.org". |
| ip-address | The IP address impacted by egress traffic. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "123.123.123.123". |
| percent | Ratio of network packet loss. | string. It is a percentage which range is 0 to 100 (10 is 10%) (default "1"). |
| ip-protocol | Only impact traffic using this IP protocol. | string, it supports the following protocol types: "tcp", "udp", "icmp", "all" (all network protocols). |
| source-port | The egress traffic which only impact specific source ports. It can only be configured when the protocol is TCP or UDP. | string. You need to use a `,` to separate the specific port or to indicate the range of the port, such as "80,8001:8010". |

#### サービスモードを使用したネットワーク損失のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"loss","ip-address":"172.16.4.4","device":"eth0","percent":"50"}'
```

### サービスモードを使用したネットワーク分断のシミュレーション

#### ネットワーク分断をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "partition" |
| accept-tcp-flags | Only the packet which match the tcp flag can be accepted, others will be dropped. Only set when the protocol is tcp. | string, such as "SYN,ACK SYN,ACK" |
| device | The network interface to impact | string, such as "eth0", required |
| direction | Specifies the partition direction, values can be 'to', 'from' or 'both'. 'from' means packets coming from the 'ip-address' or 'hostname' and going to your server, 'to' means packets originating from your server and going to the 'ip-address' or 'hostname'. | string, values can be "to", "from" or "both" (default "both") |
| hostname | Only impact traffic to these hostnames. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "chaos-mesh.org". |
| ip-address | Only impact egress traffic to these IP addresses. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "192.168.123.123". |
| ip-protocol | Only impact traffic using this IP protocol | string. It supports the following protocol types: tcp, udp, icmp, all (all network protocols). |

#### サービスモードを使用したネットワーク分断のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"partition","ip-address":"172.16.4.4","device":"eth0","direction":"from"}'
```

### サービスモードを使用したDNS障害のシミュレーション

#### DNS障害をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "dns" |
| dns-domain-name | Map this host to specified IP(dns-ip) | string, such as "chaos-mesh.org". |
| dns-ip | Map specified host(dns-domain-name) to this IP address | string, such as "123.123.123.123" |
| dns-server | Update the DNS server in /etc/resolv.conf with this value | string, such as "123.123.123.123" (default "123.123.123.123") |

#### サービスモードを使用したDNS障害のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"dns","dns-domain-name":"chaos-mesh.org","dns-ip":"123.123.123.123"}'
```

### サービスモードを使用したネットワーク帯域幅のシミュレーション

#### ネットワーク帯域幅をシミュレートするためのパラメータ

| Parameter | Description | Value |
| --- | --- | --- |
| action | Action of the experiment. | set to "bandwidth" |
| buffer | The maximum amount of bytes that tokens can be available for instantaneously | int, such as `10000`, required |
| device | The network interface to impact | string, such as "eth0", required |
| hostname | Only impact traffic to these hostnames. `hostname` and `ip-address` cannot be empty at the same time. when `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "chaos-mesh.org". |
| ip-address | Only impact egress traffic to these IP addresses. `hostname` and `ip-address` cannot be empty at the same time. When `hostname` and `ip-address` are set at the same time, the configuration item affects both the specified `hostname` and `ip-address`. | string, such as "123.123.123.123". |
| limit | The number of bytes that can be queued waiting for tokens to become available | int, such as `10000`, required |
| minburst | Specifies the size of the peakrate bucket | int, such as `10000` |
| peakrate | The maximum depletion rate of the bucket | int, such as `10000` |
| rate | The speed knob, allows bps, kbps, mbps, gbps, tbps unit. The bps unit means bytes per second. | string, such as "1mbps", required |

#### サービスモードを使用したネットワーク帯域幅のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"bandwidth","buffer":10000,"limit":10000,"rate":"10mbps","device":"eth0"}'
```

### サービスモードを使用したポート占有のシミュレーション

#### ポート占有をシミュレートするためのパラメータ

| Parameter | Description                        | Value                       |
| --------- | ---------------------------------- | --------------------------- |
| action    | Action of the experiment.          | set to "occupied"           |
| port      | The specified port to be occupied. | int, such as 8080, required |

#### サービスモードを使用したポート占有のシミュレーション例

```bash
curl -X POST 172.16.112.130:31767/api/attack/network -H "Content-Type:application/json" -d '{"action":"occupied","port":8080}'
```