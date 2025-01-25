# 面向 Linux® 用户的 FreeBSD 快速入门指南

版权 © 2008 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="d630d6fa-f0c5-4d71-9877-d3212b7534ff"><summary data-immersive-translate-walked="d630d6fa-f0c5-4d71-9877-d3212b7534ff" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

红帽（Red Hat）、RPM 是 Red Hat, Inc. 在美国和其他国家的商标或注册商标。

英特尔（Intel）、Celeron、Centrino、Core、EtherExpress、i386、i486、Itanium、Pentium 和 Xeon 是英特尔公司或其子公司在美国和其他国家的商标或注册商标。

Linux 是 Linus Torvalds 的注册商标。

UNIX 是 The Open Group 在美国和其他国家的注册商标。

许多制造商和销售商用来区分其产品的名称被声称为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目知道商标声明，则这些名称会跟随 “™” 或 “®” 符号。

</details>

摘要

本文档旨在快速使中高级 Linux® 用户熟悉 FreeBSD 的基础知识。

---

## 1. 介绍

本文档突出显示 FreeBSD 和 Linux® 之间的一些技术差异，以便中高级 Linux® 用户快速熟悉 FreeBSD 的基础知识。

此文档假定已经安装了 FreeBSD。参考 FreeBSD 手册中的安装 FreeBSD 章节以获取安装过程的帮助。

## 2. 默认 Shell

Linux 用户经常会惊讶地发现在 FreeBSD 中 Bash 不是默认的 shell。事实上，默认安装中不包括 Bash。相反，默认用户 shell 是与 Bourne shell-兼容的 sh(1)。在 FreeBSD 13 及更早版本中默认用户 shell 是 tcsh(1)，在 FreeBSD 14 及之后版本中是 sh(1)。sh(1)与 Bash 非常相似，但功能集较小。通常为 sh(1)编写的脚本将在 Bash 中运行，反之则并非总是成立。

然而，在 FreeBSD 软件包和定制集合中可安装 Bash 和其他 shell。

安装另一个 shell 后，使用 chsh(1) 来更改用户的默认 shell。建议保持 root 用户的默认 shell 不变，因为 shell 不包含在基础分发中，安装到 /usr/local/bin。如果出现问题，/usr/local/bin 所在的文件系统可能未挂载。在这种情况下， root 将无法访问其默认 shell，导致 root 无法登录并解决问题。

## 3. FreeBSD 软件包和 Ports：在 FreeBSD 中添加软件

FreeBSD 提供了两种安装应用程序的方法：二进制软件包和编译 ports。每种方法都有其自身的好处：

二进制软件包

- 与编译大型应用程序相比，安装速度更快。
- 不需要理解如何编译软件。
- 不需要安装编译器。

Ports

- 可以定制安装选项。
- 可以应用自定义补丁。

如果应用程序安装不需要任何定制，只需安装包即可。每当应用程序需要定制默认选项时，请编译 port。如果需要，可以使用 make package 从 ports 编译自定义包。

可用的 ports 和 软件包 的完整列表可以在这里找到。

### 3.1. 软件包

软件包是预编译的应用程序，相当于 Debian/Ubuntu 系统上的 .deb 文件和 Red Hat/Fedora 系统上的 .rpm 文件。软件包是使用 pkg 安装的。例如，以下命令安装 Apache 2.4:

```
# pkg install apache24
```

获取有关软件包的更多信息，请参阅 FreeBSD 手册的第 5.4 节：使用 pkgng 进行二进制包管理。

### 3.2. Ports

FreeBSD 定制集合是一组特定定制的 Makefile 和补丁框架，专门用于在 FreeBSD 上从源代码安装应用程序。在安装一个定制时，系统将获取源代码，应用任何所需的补丁，编译代码，然后安装应用程序及其所需的任何依赖项。

Ports，有时被称为 ports 树，可以使用 Git 安装到/usr/ports。在 FreeBSD 手册的 4.5.1 节中可以找到安装 Ports 的详细说明。

要编译 port，请切换到 port 的目录并开始构建过程。以下示例从 Ports 安装 Apache 2.4：

```sh
# cd /usr/ports/www/apache24
# make install clean
```

使用 ports 安装软件的好处是能够定制安装选项。此示例指定了也应安装 mod_ldap 模块：

```sh
# cd /usr/ports/www/apache24
# make WITH_LDAP="YES" install clean
```

有关更多信息，请参阅 Ports 。

## 4. 系统启动

许多 Linux® 发行版使用 SysV init 系统，而 FreeBSD 使用传统的 BSD 风格 init(8)。在 BSD 风格的 init(8)中，不存在运行级别，也没有/etc/inittab 文件。相反，启动由 rc(8)脚本控制。在系统启动时，/etc/rc 会读取/etc/rc.conf 和/etc/defaults/rc.conf 以确定要启动的服务。然后，通过运行位于/etc/rc.d/和/usr/local/etc/rc.d/中的相应服务初始化脚本来启动指定的服务。这些脚本类似于 Linux® 系统中/etc/init.d/中的脚本。

/etc/rc.d/ 中找到的脚本是属于“基本”系统的应用程序，比如 cron(8)、sshd(8) 和 syslog(3)。/usr/local/etc/rc.d/ 中的脚本是用于用户安装的应用程序，比如 Apache 和 Squid。

由于 FreeBSD 是作为一个完整的操作系统开发的，用户安装的应用程序不被视为“基本”系统的一部分。通常使用 Packages 或 Ports 安装用户安装的应用程序。为了将它们与基本系统区分开来，用户安装的应用程序安装在 /usr/local/ 下。因此，用户安装的二进制文件位于 /usr/local/bin/，配置文件位于 /usr/local/etc/，等等。

通过在 /etc/rc.conf 中为服务添加条目来启用服务。系统默认设置位于 /etc/defaults/rc.conf，这些默认设置会被 /etc/rc.conf 中的设置覆盖。有关可用条目的更多信息，请参阅 rc.conf(5)。在安装额外应用程序时，请查看应用程序的安装消息，以确定如何启用任何相关服务。

在 /etc/rc.conf 中，以下条目启用了 sshd(8)，启用了 Apache 2.4，并指定 Apache 应该使用 SSL 启动。

```sh
# enable SSHD
sshd_enable="YES"
# enable Apache with SSL
apache24_enable="YES"
apache24_flags="-DSSL"
```

一旦在 /etc/rc.conf 中启用了服务，就可以在不重启系统的情况下启动它：

```sh
# service sshd start
# service apache24 start
```

如果某个服务未被启用，可以使用命令行来启动它： onestart

```sh
# service sshd onestart
```

## 5. 网络配置

而不是 Linux® 用于标识网络接口的通用 ethX 标识符，FreeBSD 使用驱动程序名称后跟一个数字。下面是从 ifconfig(8)显示的两个 Intel® Pro 1000 网络接口（em0 和 em1）的输出：

```sh
% ifconfig
em0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        options=b<RXCSUM,TXCSUM,VLAN_MTU>
        inet 10.10.10.100 netmask 0xffffff00 broadcast 10.10.10.255
        ether 00:50:56:a7:70:b2
        media: Ethernet autoselect (1000baseTX <full-duplex>)
        status: active
em1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        options=b<RXCSUM,TXCSUM,VLAN_MTU>
        inet 192.168.10.222 netmask 0xffffff00 broadcast 192.168.10.255
        ether 00:50:56:a7:03:2b
        media: Ethernet autoselect (1000baseTX <full-duplex>)
        status: active
```

可以使用 ifconfig(8)为接口分配 IP 地址。要在重新启动后保持持久性，IP 配置必须包含在/etc/rc.conf 中。以下/etc/rc.conf 条目指定主机名、IP 地址和默认网关：

```sh
hostname="server1.example.com"
ifconfig_em0="inet 10.10.10.100 netmask 255.255.255.0"
defaultrouter="10.10.10.1"
```

使用以下条目来为 DHCP 配置接口：

```sh
hostname="server1.example.com"
ifconfig_em0="DHCP"
```

## 6. 防火墙

FreeBSD 不使用 Linux® IPTABLES 作为其防火墙。相反，FreeBSD 提供三种内核级防火墙选择：

- [ 精简](https://docs.freebsd.org/en/books/handbook/#firewalls-pf)
- [IPFILTER](https://docs.freebsd.org/en/books/handbook/#firewalls-ipf)
- [IPFW](https://docs.freebsd.org/en/books/handbook/#firewalls-ipfw)

PF 由 OpenBSD 项目开发，并移植到 FreeBSD。PF 被创建作为 IPFILTER 的替代品，其语法与 IPFILTER 类似。PF 可以与 altq(4)配合提供 QoS 功能。

这个示例 PF 条目允许入站 SSH：

```sh
pass in on $ext_if inet proto tcp from any to ($ext_if) port 22
```

IPFILTER 是由 Darren Reed 开发的防火墙应用程序。它不专门针对 FreeBSD，已移植到多个操作系统，包括 NetBSD、OpenBSD、SunOS、HP/UX 和 Solaris。

允许入站 SSH 的 IPFILTER 语法是：

```sh
pass in on $ext_if proto tcp from any to any port = 22
```

IPFW 是由 FreeBSD 开发和维护的防火墙。它可以与 dummynet(4)配合使用，以提供流量整形功能并模拟不同类型的网络连接。

允许入站 SSH 的 IPFW 语法是：

```sh
ipfw add allow tcp from any to me 22 in via $ext_if
```

## 7. 更新 FreeBSD

更新 FreeBSD 系统有两种方法：从源代码更新或使用二进制更新。

从源代码更新是最复杂的更新方法，但提供了最大量的灵活性。该过程涉及将本地副本的 FreeBSD 源代码与 FreeBSD Git 存储库同步。一旦本地源代码是最新的，就可以编译新版本的内核和用户空间。

二进制更新类似于在 Linux® 系统上使用 yum 或 apt-get 进行更新。在 FreeBSD 中，可以使用 freebsd-update(8) 获取新的二进制更新并安装它们。这些更新可以使用 cron(8) 进行调度。

```sh
0 3 * * * root /usr/sbin/freebsd-update cron
```

要了解有关源码和二进制更新的更多信息，请参阅《FreeBSD 手册》中的更新章节。

## 8. procfs: 已消失但未被遗忘

在某些 Linux® 发行版中，可以查看 /proc/sys/net/ipv4/ip_forward 来确定是否已启用 IP 转发。在 FreeBSD 中，使用 sysctl(8) 来查看此类和其他系统设置。

例如，在 FreeBSD 系统上，可以使用以下命令来确定是否已启用 IP 转发：

```sh
% sysctl net.inet.ip.forwarding
net.inet.ip.forwarding: 0
```

使用 -a 列出所有系统设置：

```sh
% sysctl -a | more
```

如果一个应用程序需要 procfs，请将以下条目添加到/etc/fstab：

```sh
proc                /proc           procfs  rw,noauto       0       0
```

包括 noauto 将阻止/proc 在启动时自动挂载。

要在不重新启动的情况下挂载文件系统：

```sh
# mount /proc
```

## 常见命令

一些常见命令等价如下：

| Linux® 命令 (Red Hat/Debian)                      | FreeBSD 等效            | 目的                 |
| ------------------------------------------------- | ----------------------- | -------------------- |
| `yum install package` / `apt-get install package` | `pkg install package`   | 从远程仓库安装软件包 |
| `rpm -ivh package` / `dpkg -i package`            | `pkg add package`       | 安装本地软件包       |
| `rpm -qa` / `dpkg -l`                             | `pkg info`              | 列出已安装的软件包   |
| `lspci`                                           | `pciconf`               | 列出 PCI 设备        |
| `lsmod`                                           | `kldstat`               | 列出已加载的内核模块 |
| `modprobe`                                        | `kldload` / `kldunload` | 加载/卸载内核模块    |
| `strace`                                          | `truss`                 | 跟踪系统调用         |

## 10. 结论

本文档概述了 FreeBSD。更详细地了解这些主题，请参阅 FreeBSD 手册，以及本文档未涵盖的许多其他主题。
