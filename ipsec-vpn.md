# IPsec VPN

版权 © 2023 年 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="f8a0f701-4d13-4c1b-a2c9-324e5e37e354"><summary data-immersive-translate-walked="f8a0f701-4d13-4c1b-a2c9-324e5e37e354" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

制造商和销售商使用的许多名称，用于区分其产品，被视为商标所有权。如果这些名称出现在本文件中，并且 FreeBSD 项目意识到商标所有权声明，则这些名称将后跟“™”或“®”符号。

</details>

---

Internet 协议安全（IPsec）是一组协议，位于 Internet 协议（IP）层之上。它允许两个或更多主机通过对通信会话的每个 IP 数据包进行身份验证和加密来以安全方式进行通信。FreeBSD IPsec 网络堆栈基于 http://www.kame.net/实现，并支持 IPv4 和 IPv6 会话。

IPsec 由以下子协议组成：

* 封装安全有效载荷（ESP）：该协议通过使用对称加密算法（如 Blowfish 和 3DES）对内容进行加密，保护 IP 数据包免受第三方干扰。
* 认证头（AH）：该协议通过计算加密校验和并使用安全哈希函数对 IP 数据包头部字段进行哈希处理，保护 IP 数据包头部免受第三方干扰和欺骗。然后跟随一个包含哈希的附加头部，以允许对数据包中的信息进行认证。
* IP 有效载荷压缩协议（IPComp）：该协议通过压缩 IP 有效载荷以减少发送的数据量，试图提高通信性能。

这些协议可以根据环境的不同分别使用或分开使用。

IPsec 支持两种操作模式。第一种是传输模式，用于保护两个主机之间的通信。第二种是隧道模式，用于建立虚拟隧道，通常被称为虚拟私人网络（VPN）。有关 FreeBSD 中 IPsec 子系统的详细信息，请参阅 ipsec（4）。

本文演示了如何在家庭网络和企业网络之间建立 IPsecVPN 的过程。

在示例场景中：

* 两个站点通过运行 FreeBSD 的网关连接到互联网。
* 每个网络上的网关至少有一个外部 IP 地址。在此示例中，公司 LAN 的外部 IP 地址是 172.16.5.4 ，家庭 LAN 的外部 IP 地址是 192.168.1.12 。
* 两个网络的内部地址可以是公共的或私有的 IP 地址。但是，地址空间不能重叠。在这个例子中，公司 LAN 的内部 IP 地址是 10.246.38.1 ，家庭 LAN 的内部 IP 地址是 10.0.0.5 。

```
           corporate                          home
10.246.38.1/24 -- 172.16.5.4 <--> 192.168.1.12 -- 10.0.0.5/24
```

## 1. 在 FreeBSD 上配置 VPN

要开始，必须从Ports集合中安装 security/ipsec-tools。这个软件提供了许多支持配置的应用程序。

下一个要求是创建两个 gif(4) 伪设备，用于隧道数据包并允许两个网络正常通信。如 root 所示，在每个网关上运行以下命令：

```
corp-gw# ifconfig gif0 create
corp-gw# ifconfig gif0 10.246.38.1 10.0.0.5
corp-gw# ifconfig gif0 tunnel 172.16.5.4 192.168.1.12
```

```
home-gw# ifconfig gif0 create
home-gw# ifconfig gif0 10.0.0.5 10.246.38.1
home-gw# ifconfig gif0 tunnel 192.168.1.12 172.16.5.4
```

使用 ifconfig gif0 在每个网关上验证设置。这是来自家庭网关的输出：

```
gif0: flags=8051 mtu 1280
tunnel inet 172.16.5.4 --> 192.168.1.12
inet6 fe80::2e0:81ff:fe02:5881%gif0 prefixlen 64 scopeid 0x6
inet 10.246.38.1 --> 10.0.0.5 netmask 0xffffff00
```

这是来自公司网关的输出：

```
gif0: flags=8051 mtu 1280
tunnel inet 192.168.1.12 --> 172.16.5.4
inet 10.0.0.5 --> 10.246.38.1 netmask 0xffffff00
inet6 fe80::250:bfff:fe3a:c1f%gif0 prefixlen 64 scopeid 0x4
```

一旦完成，应该可以使用 ping(8)到达两个内部 IP 地址：

```
home-gw# ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5): 56 data bytes
64 bytes from 10.0.0.5: icmp_seq=0 ttl=64 time=42.786 ms
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=19.255 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=20.440 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=21.036 ms
--- 10.0.0.5 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max/stddev = 19.255/25.879/42.786/9.782 ms

corp-gw# ping 10.246.38.1
PING 10.246.38.1 (10.246.38.1): 56 data bytes
64 bytes from 10.246.38.1: icmp_seq=0 ttl=64 time=28.106 ms
64 bytes from 10.246.38.1: icmp_seq=1 ttl=64 time=42.917 ms
64 bytes from 10.246.38.1: icmp_seq=2 ttl=64 time=127.525 ms
64 bytes from 10.246.38.1: icmp_seq=3 ttl=64 time=119.896 ms
64 bytes from 10.246.38.1: icmp_seq=4 ttl=64 time=154.524 ms
--- 10.246.38.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max/stddev = 28.106/94.594/154.524/49.814 ms
```

如预期，双方都能够从各自的私人配置地址发送和接收 ICMP 数据包。接下来，必须告诉两个网关如何路由数据包，以便正确地从每个网关后面的网络发送流量。以下命令将实现这一目标：

```
corp-gw# route add 10.0.0.0 10.0.0.5 255.255.255.0
corp-gw# route add net 10.0.0.0: gateway 10.0.0.5
home-gw# route add 10.246.38.0 10.246.38.1 255.255.255.0
home-gw# route add host 10.246.38.0: gateway 10.246.38.1
```

内部机器应该可以从每个网关以及网关后面的机器到达。再次使用 ping(8)进行确认：

```
corp-gw# ping -c 3 10.0.0.8
PING 10.0.0.8 (10.0.0.8): 56 data bytes
64 bytes from 10.0.0.8: icmp_seq=0 ttl=63 time=92.391 ms
64 bytes from 10.0.0.8: icmp_seq=1 ttl=63 time=21.870 ms
64 bytes from 10.0.0.8: icmp_seq=2 ttl=63 time=198.022 ms
--- 10.0.0.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 21.870/101.846/198.022/74.001 ms

home-gw# ping -c 3 10.246.38.107
PING 10.246.38.1 (10.246.38.107): 56 data bytes
64 bytes from 10.246.38.107: icmp_seq=0 ttl=64 time=53.491 ms
64 bytes from 10.246.38.107: icmp_seq=1 ttl=64 time=23.395 ms
64 bytes from 10.246.38.107: icmp_seq=2 ttl=64 time=23.865 ms
--- 10.246.38.107 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 21.145/31.721/53.491/12.179 ms
```

在这一点上，通过一个 gif 隧道封装的网络之间的流量正在流动，但没有任何加密。 接下来，使用 IPSec 使用预共享密钥(PSK)加密流量。 除了 IP 地址外，在两个网关上，/usr/local/etc/racoon/racoon.conf 将是相同的，并且看起来类似于：

```
path    pre_shared_key  "/usr/local/etc/racoon/psk.txt"; #location of pre-shared key file
log     debug;	#log verbosity setting: set to 'notify' when testing and debugging is complete

padding	# options are not to be changed
{
        maximum_length  20;
        randomize       off;
        strict_check    off;
        exclusive_tail  off;
}

timer	# timing options. change as needed
{
        counter         5;
        interval        20 sec;
        persend         1;
#       natt_keepalive  15 sec;
        phase1          30 sec;
        phase2          15 sec;
}

listen	# address [port] that racoon will listen on
{
        isakmp          172.16.5.4 [500];
        isakmp_natt     172.16.5.4 [4500];
}

remote  192.168.1.12 [500]
{
        exchange_mode   main,aggressive;
        doi             ipsec_doi;
        situation       identity_only;
        my_identifier   address 172.16.5.4;
        peers_identifier        address 192.168.1.12;
        lifetime        time 8 hour;
        passive         off;
        proposal_check  obey;
#       nat_traversal   off;
        generate_policy off;

                        proposal {
                                encryption_algorithm    blowfish;
                                hash_algorithm          md5;
                                authentication_method   pre_shared_key;
                                lifetime time           30 sec;
                                dh_group                1;
                        }
}

sainfo  (address 10.246.38.0/24 any address 10.0.0.0/24 any)	# address $network/$netmask $type address $network/$netmask $type ( $type being any or esp)
{								# $network must be the two internal networks you are joining.
        pfs_group       1;
        lifetime        time    36000 sec;
        encryption_algorithm    blowfish,3des;
        authentication_algorithm        hmac_md5,hmac_sha1;
        compression_algorithm   deflate;
}
```

有关每个可用选项的描述，请参阅 racoon.conf 的手册页。

安全策略数据库(SPD)需要进行配置，以便 FreeBSD 和 racoon 能够在主机之间加密和解密网络流量。

这可以通过在企业网关上使用类似以下内容的shell脚本来实现。此文件将在系统初始化期间使用，并应保存为/usr/local/etc/racoon/setkey.conf。

```
flush;
spdflush;
# To the home network
spdadd 10.246.38.0/24 10.0.0.0/24 any -P out ipsec esp/tunnel/172.16.5.4-192.168.1.12/use;
spdadd 10.0.0.0/24 10.246.38.0/24 any -P in ipsec esp/tunnel/192.168.1.12-172.16.5.4/use;
```

放置后，可以使用以下命令在两个网关上启动 racoon：

```
# /usr/local/sbin/racoon -F -f /usr/local/etc/racoon/racoon.conf -l /var/log/racoon.log
```

输出应类似于以下内容：

```
corp-gw# /usr/local/sbin/racoon -F -f /usr/local/etc/racoon/racoon.conf
Foreground mode.
2006-01-30 01:35:47: INFO: begin Identity Protection mode.
2006-01-30 01:35:48: INFO: received Vendor ID: KAME/racoon
2006-01-30 01:35:55: INFO: received Vendor ID: KAME/racoon
2006-01-30 01:36:04: INFO: ISAKMP-SA established 172.16.5.4[500]-192.168.1.12[500] spi:623b9b3bd2492452:7deab82d54ff704a
2006-01-30 01:36:05: INFO: initiate new phase 2 negotiation: 172.16.5.4[0]192.168.1.12[0]
2006-01-30 01:36:09: INFO: IPsec-SA established: ESP/Tunnel 192.168.1.12[0]->172.16.5.4[0] spi=28496098(0x1b2d0e2)
2006-01-30 01:36:09: INFO: IPsec-SA established: ESP/Tunnel 172.16.5.4[0]->192.168.1.12[0] spi=47784998(0x2d92426)
2006-01-30 01:36:13: INFO: respond new phase 2 negotiation: 172.16.5.4[0]192.168.1.12[0]
2006-01-30 01:36:18: INFO: IPsec-SA established: ESP/Tunnel 192.168.1.12[0]->172.16.5.4[0] spi=124397467(0x76a279b)
2006-01-30 01:36:18: INFO: IPsec-SA established: ESP/Tunnel 172.16.5.4[0]->192.168.1.12[0] spi=175852902(0xa7b4d66)
```

为了确保隧道正常工作，请切换到另一个控制台，并使用 tcpdump(1)命令查看网络流量。使用以下命令。根据需要将 em0 替换为网络接口卡：

```
corp-gw# tcpdump -i em0 host 172.16.5.4 and dst 192.168.1.12
```

类似以下数据应该出现在控制台上。如果没有，那么就存在问题，需要调试返回的数据。

```
01:47:32.021683 IP corporatenetwork.com > 192.168.1.12.privatenetwork.com: ESP(spi=0x02acbf9f,seq=0xa)
01:47:33.022442 IP corporatenetwork.com > 192.168.1.12.privatenetwork.com: ESP(spi=0x02acbf9f,seq=0xb)
01:47:34.024218 IP corporatenetwork.com > 192.168.1.12.privatenetwork.com: ESP(spi=0x02acbf9f,seq=0xc)
```

此时，应该可以使用并且似乎是同一网络的一部分。很可能，这两个网络都受防火墙保护。要允许流量在它们之间流动，需要添加规则以通过数据包。对于 ipfw(8)防火墙，将以下行添加到防火墙配置文件中：

```
ipfw add 00201 allow log esp from any to any
ipfw add 00202 allow log ah from any to any
ipfw add 00203 allow log ipencap from any to any
ipfw add 00204 allow log udp from any 500 to any
```

|  | 规则号码可能需要根据当前主机配置进行更改。 |
| -- | -------------------------------------------- |

对于使用 pf(4)或 ipf(8)的用户，以下规则应该起作用：

```
pass in quick proto esp from any to any
pass in quick proto ah from any to any
pass in quick proto ipencap from any to any
pass in quick proto udp from any port = 500 to any port = 500
pass in quick on gif0 from any to any
pass out quick proto esp from any to any
pass out quick proto ah from any to any
pass out quick proto ipencap from any to any
pass out quick proto udp from any port = 500 to any port = 500
pass out quick on gif0 from any to any
```

最后，在系统初始化期间允许机器启动 VPN 支持，请将以下行添加到/etc/rc.conf：

```
ipsec_enable="YES"
ipsec_program="/usr/local/sbin/setkey"
ipsec_file="/usr/local/etc/racoon/setkey.conf" # allows setting up spd policies on boot
racoon_enable="yes"
```
