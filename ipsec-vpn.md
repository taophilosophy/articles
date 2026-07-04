# IPsec VPN

- 原文：[VPN over IPsec](https://docs.freebsd.org/en/articles/vpn-ipsec/)

Internet Protocol Security (IPsec) 是一组协议，位于互联网协议（IP）层之上。它通过对每个通信会话的 IP 包进行认证和加密，使两个或多个主机能够以安全的方式进行通信。FreeBSD 的 IPsec 网络栈基于 [http://www.kame.net/](http://www.kame.net/) 实现，支持 IPv4 和 IPv6 会话。

IPsec 包含以下子协议：

- *Encapsulated Security Payload (ESP)*：该协议通过使用对称加密算法（如 Blowfish 和 3DES）加密内容，保护 IP 包数据不受第三方干扰。
- *Authentication Header (AH)*：该协议通过计算加密校验和并使用安全哈希函数对 IP 包头字段进行哈希，保护 IP 包头免受第三方干扰和伪造。接着，附加一个包含哈希值的头部，允许对包中的信息进行认证。
- *IP Payload Compression Protocol (IPComp)*：该协议通过压缩 IP 负载以减少传输的数据量，尝试提高通信性能。

这些协议可以一起使用，也可以单独使用，具体取决于环境。

IPsec 支持两种操作模式。第一种模式是 *Transport Mode*，保护两个主机之间的通信。第二种模式是 *Tunnel Mode*，用于建立虚拟隧道，通常称为虚拟专用网络（VPN）。有关 FreeBSD 中 IPsec 子系统的详细信息，请参考 [ipsec(4)](https://man.freebsd.org/cgi/man.cgi?query=ipsec&sektion=4&format=html)。

本文演示了如何在家庭网络和公司网络之间设置 IPsec VPN。

在示例场景中：

- 两个站点都通过运行 FreeBSD 的网关连接到互联网。
- 每个网络的网关至少有一个外部 IP 地址。在此示例中，公司 LAN 的外部 IP 地址是 **172.16.5.4**，家庭 LAN 的外部 IP 地址是 **192.168.1.12**。
- 两个网络的内部地址可以是公共或私有 IP 地址。然而，地址空间不得重叠。在此示例中，公司 LAN 的内部 IP 地址是 **10.246.38.1**，家庭 LAN 的内部 IP 地址是 **10.0.0.5**。

```sh
           公司                          家
10.246.38.1/24 -- 172.16.5.4 <--> 192.168.1.12 -- 10.0.0.5/24
```

## 1. 在 FreeBSD 上配置 VPN

首先，必须从 Ports Collection 安装 [security/ipsec-tools](https://cgit.freebsd.org/ports/tree/security/ipsec-tools/)。该软件提供了多个支持配置的应用程序。

接下来需要创建两个 [gif(4)](https://man.freebsd.org/cgi/man.cgi?query=gif&sektion=4&format=html) 虚拟设备，用于隧道数据包并允许两个网络正确通信。作为 `root` 用户，在每个网关上运行以下命令：

```sh
corp-gw# ifconfig gif0 create
corp-gw# ifconfig gif0 10.246.38.1 10.0.0.5
corp-gw# ifconfig gif0 tunnel 172.16.5.4 192.168.1.12
```

```sh
home-gw# ifconfig gif0 create
home-gw# ifconfig gif0 10.0.0.5 10.246.38.1
home-gw# ifconfig gif0 tunnel 192.168.1.12 172.16.5.4
```

使用 `ifconfig gif0` 验证每个网关上的设置。以下是家庭网关的输出：

```sh
gif0: flags=8051 mtu 1280
tunnel inet 172.16.5.4 --> 192.168.1.12
inet6 fe80::2e0:81ff:fe02:5881%gif0 prefixlen 64 scopeid 0x6
inet 10.246.38.1 --> 10.0.0.5 netmask 0xffffff00
```

以下是公司网关的输出：

```sh
gif0: flags=8051 mtu 1280
tunnel inet 192.168.1.12 --> 172.16.5.4
inet 10.0.0.5 --> 10.246.38.1 netmask 0xffffff00
inet6 fe80::250:bfff:fe3a:c1f%gif0 prefixlen 64 scopeid 0x4
```

完成后，使用 [ping(8)](https://man.freebsd.org/cgi/man.cgi?query=ping&sektion=8&format=html) 检查两个内部 IP 地址是否可以互通：


```sh
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

正如预期，双方能够从私有配置的地址发送和接收 ICMP 包。接下来，必须告诉两个网关如何路由数据包，以便正确地从每个网关后面的网络发送流量。以下命令将实现此目标：

```sh
corp-gw# route add 10.0.0.0 10.0.0.5 255.255.255.0
corp-gw# route add net 10.0.0.0: gateway 10.0.0.5
home-gw# route add 10.246.38.0 10.246.38.1 255.255.255.0
home-gw# route add host 10.246.38.0: gateway 10.246.38.1
```

内部机器应该能够从每个网关以及网关后面的机器访问。同样，使用 [ping(8)](https://man.freebsd.org/cgi/man.cgi?query=ping&sektion=8&format=html) 来确认：

```sh
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

此时，流量已经在通过 gif 隧道封装的两个网络之间流动，但没有任何加密。接下来，使用 IPSec 通过预共享密钥（PSK）加密流量。除了 IP 地址之外，**/usr/local/etc/racoon/racoon.conf** 在两个网关上将是相同的，并且内容类似于：

```ini
path    pre_shared_key  "/usr/local/etc/racoon/psk.txt"; # 预共享密钥文件的位置
log     debug;	# 日志详细度设置：在测试和调试完成后设置为 'notify'

padding	# 选项不可更改
{
        maximum_length  20;
        randomize       off;
        strict_check    off;
        exclusive_tail  off;
}

timer	# 定时选项，根据需要更改
{
        counter         5;
        interval        20 sec;
        persend         1;
#       natt_keepalive  15 sec;
        phase1          30 sec;
        phase2          15 sec;
}

listen	# racoon 将监听的地址 [端口]
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

sainfo  (address 10.246.38.0/24 any address 10.0.0.0/24 any)	# 地址 $network/$netmask $type 地址 $network/$netmask $type ( $type 可以是 any 或 esp)
{								# $network 必须是你要连接的两个内部网络。
        pfs_group       1;
        lifetime        time    36000 sec;
        encryption_algorithm    blowfish,3des;
        authentication_algorithm        hmac_md5,hmac_sha1;
        compression_algorithm   deflate;
}
```


有关每个可用选项的描述，请参考 **racoon.conf** 的手册页。

需要配置安全策略数据库（SPD），以便 FreeBSD 和 racoon 能够加密和解密主机之间的网络流量。

在公司网关上，可以通过类似以下的 shell 脚本来实现这一点。该文件将在系统初始化期间使用，并应保存为 **/usr/local/etc/racoon/setkey.conf**。

```ini
flush;
spdflush;
# 到家用网络
spdadd 10.246.38.0/24 10.0.0.0/24 any -P out ipsec esp/tunnel/172.16.5.4-192.168.1.12/use;
spdadd 10.0.0.0/24 10.246.38.0/24 any -P in ipsec esp/tunnel/192.168.1.12-172.16.5.4/use;
```

文件配置好后，可以通过以下命令在两个网关上启动 racoon：

```sh
# /usr/local/sbin/racoon -F -f /usr/local/etc/racoon/racoon.conf -l /var/log/racoon.log
```

输出应类似于以下内容：

```sh
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

为了确保隧道正常工作，请切换到另一个控制台并使用 [tcpdump(1)](https://man.freebsd.org/cgi/man.cgi?query=tcpdump&sektion=1&format=html) 查看网络流量，使用以下命令。根据需要将 `em0` 替换为网络接口卡：

```sh
corp-gw# tcpdump -i em0 host 172.16.5.4 and dst 192.168.1.12
```

控制台应显示类似以下的数据。如果没有，说明存在问题，需要调试返回的数据。

```sh
01:47:32.021683 IP corporatenetwork.com > 192.168.1.12.privatenetwork.com: ESP(spi=0x02acbf9f,seq=0xa)
01:47:33.022442 IP corporatenetwork.com > 192.168.1.12.privatenetwork.com: ESP(spi=0x02acbf9f,seq=0xb)
01:47:34.024218 IP corporatenetwork.com > 192.168.1.12.privatenetwork.com: ESP(spi=0x02acbf9f,seq=0xc)
```

此时，两个网络应该都可用，并且看起来像是同一个网络。很可能这两个网络都受防火墙保护。为了允许流量在它们之间流动，需要在防火墙中添加规则以通过数据包。对于 [ipfw(8)](https://man.freebsd.org/cgi/man.cgi?query=ipfw&sektion=8&format=html) 防火墙，可以在防火墙配置文件中添加以下行：

```sh
ipfw add 00201 allow log esp from any to any
ipfw add 00202 allow log ah from any to any
ipfw add 00203 allow log ipencap from any to any
ipfw add 00204 allow log udp from any 500 to any
```

>**注意**
>
>规则号可能需要根据当前主机配置进行修改。


对于 [pf(4)](https://man.freebsd.org/cgi/man.cgi?query=pf&sektion=4&format=html) 或 [ipf(8)](https://man.freebsd.org/cgi/man.cgi?query=ipf&sektion=8&format=html) 的用户，以下规则应该可以起作用：

```sh
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

最后，为了在系统初始化期间启用 VPN 支持，可以将以下行添加到 **/etc/rc.conf**：

```sh
ipsec_enable="YES"
ipsec_program="/usr/local/sbin/setkey"
ipsec_file="/usr/local/etc/racoon/setkey.conf" # 允许在启动时设置 spd 策略
racoon_enable="yes"
```
