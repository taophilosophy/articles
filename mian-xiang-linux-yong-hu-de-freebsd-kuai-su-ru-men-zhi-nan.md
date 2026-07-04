# 面向 Linux® 用户的 FreeBSD 快速入门指南

- 原文：[FreeBSD Quickstart Guide for Linux® Users](https://docs.freebsd.org/en/articles/linux-users/)

## 摘要

本文档旨在帮助中级到高级的 Linux® 用户快速了解 FreeBSD 的基础知识。

## 1. 介绍

本文档突出了 FreeBSD 和 Linux® 之间的一些技术差异，以帮助中级到高级的 Linux® 用户快速熟悉 FreeBSD 的基础知识。

本文档假设 FreeBSD 已经安装完毕。有关安装过程的帮助，请参考 FreeBSD 手册中的 [安装 FreeBSD](https://docs.freebsd.org/en/books/handbook/#bsdinstall) 章节。

## 2. 默认 Shell

Linux® 用户通常会感到惊讶，因为 FreeBSD 中的默认 shell 不是 Bash。事实上，Bash 并未包含在默认系统中。相反，FreeBSD 使用兼容 POSIX 的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 作为默认用户 shell。对于 FreeBSD 13 及之前版本，root 用户的默认 shell 是 [tcsh(1)](https://man.freebsd.org/cgi/man.cgi?query=tcsh&sektion=1&format=html)，而对于 FreeBSD 14 及之后版本，则是 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html)。虽然 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 与 Bash 十分相似，但其功能集要小得多。通常，为 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 编写的 shell 脚本可以在 Bash 中运行，但反之并不总是成立。

不过，可以通过 FreeBSD 的 [Ports 和包](https://docs.freebsd.org/en/books/handbook/#ports) 来安装 Bash 和其他 shell。

安装了其他 shell 后，可以使用 [chsh(1)](https://man.freebsd.org/cgi/man.cgi?query=chsh&sektion=1&format=html) 来更改用户的默认 shell。建议不要更改 `root` 用户的默认 shell，因为不包含在基础发行版中的 shell 是安装到 **/usr/local/bin** 目录下的。如果出现问题，可能会导致包含 **/usr/local/bin** 的文件系统未挂载，在这种情况下，`root` 用户将无法访问默认 shell，无法登录并修复问题。

## 3. Ports 和包：在 FreeBSD 中添加软件

FreeBSD 提供了两种安装应用程序的方法：二进制包和编译 Ports。每种方法都有其优点：

**二进制包**

- 相比编译大型应用程序，安装速度更快。
- 不需要了解如何编译软件。
- 不需要安装编译器。

**Ports**

- 可以自定义安装选项。
- 可以应用自定义补丁。

如果应用程序的安装无需任何定制，安装二进制包即可。如果应用程序需要定制默认选项，则应编译 Port。必要时，可以通过 `make package` 从 Ports 编译自定义包。

可以在 [这里](https://ports.freebsd.org/) 找到所有可用的 Ports 和包的完整列表。

### 3.1. 包

包是预编译的应用程序，相当于基于 Debian/Ubuntu 系统的 **.deb** 文件和基于 Red Hat/Fedora 系统的 **.rpm** 文件。包通过 `pkg` 安装。例如，以下命令安装 Apache 2.4：

```sh
# pkg install apache24
```

有关包的更多信息，请参考 FreeBSD 手册第 4.4 节：[使用 pkgng 进行二进制包管理](https://docs.freebsd.org/en/books/handbook/#pkgng-intro)。

### 3.2. Ports

FreeBSD Ports 是一套专门为从源代码安装应用程序而定制的 **Makefile** 和补丁框架。在安装 Port 时，系统会获取源代码、应用所需的补丁、编译代码，并安装应用程序及其所需的依赖项。

Ports 有时也称为 Ports 树，可以使用 [Git](https://docs.freebsd.org/en/books/handbook/mirrors/#git) 安装到 **/usr/ports**。有关安装 Ports 的详细说明，请参见 FreeBSD 手册第 4.5.1 节。

要编译 Port，请切换到该 Port 的目录并启动构建过程。以下示例展示了如何从 Ports 安装 Apache 2.4：

```sh
# cd /usr/ports/www/apache24
# make install clean
```

使用 Ports 安装软件的一个好处是能够自定义安装选项。例如，以下命令指定还要安装 `mod_ldap` 模块：

```sh
# cd /usr/ports/www/apache24
# make WITH_LDAP="YES" install clean
```

有关更多信息，请参考 [使用 Ports](https://docs.freebsd.org/en/books/handbook/#ports-using) 章节。

## 4. 系统启动

许多 Linux® 发行版使用 SysV init 系统，而 FreeBSD 使用传统的 BSD 风格的 [init(8)](https://man.freebsd.org/cgi/man.cgi?query=init&sektion=8&format=html)。在 BSD 风格的 [init(8)](https://man.freebsd.org/cgi/man.cgi?query=init&sektion=8&format=html) 下，没有运行级别，并且 **/etc/inittab** 文件不存在。相反，启动过程由 [rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html) 脚本控制。系统启动时，**/etc/rc** 会读取 **/etc/rc.conf** 和 **/etc/defaults/rc.conf** 来确定要启动哪些服务。指定的服务通过运行位于 **/etc/rc.d/** 和 **/usr/local/etc/rc.d/** 中的相应服务初始化脚本来启动。这些脚本类似于 Linux® 系统中 **/etc/init.d/** 中的脚本。

**/etc/rc.d/** 中的脚本用于那些属于“基础”系统的应用程序，如 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html)、[sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html) 和 [syslog(3)](https://man.freebsd.org/cgi/man.cgi?query=syslog&sektion=3&format=html)。**/usr/local/etc/rc.d/** 中的脚本用于用户安装的应用程序，如 Apache 和 Squid。

由于 FreeBSD 被开发为一个完整的操作系统，用户安装的应用程序不被认为是“基础”系统的一部分。用户安装的应用程序通常通过 [Ports 和包](https://docs.freebsd.org/en/books/handbook/#ports-using) 安装。为了将它们与基础系统分开，用户安装的应用程序会被安装到 **/usr/local/** 下。因此，用户安装的二进制文件位于 **/usr/local/bin/**，配置文件位于 **/usr/local/etc/**，以此类推。

通过在 **/etc/rc.conf** 中添加相应的条目来启用服务。系统默认设置位于 **/etc/defaults/rc.conf** 中，这些默认设置可以通过 **/etc/rc.conf** 中的设置进行覆盖。有关可用条目的更多信息，请参见 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html)。安装附加应用程序时，请查看应用程序的安装信息，以确定如何启用相关服务。

以下是 **/etc/rc.conf** 中的条目，启用 [sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html)，启用 Apache 2.4，并指定 Apache 应该以 SSL 启动。

```ini
# 启用 SSHD
sshd_enable="YES"
# 启用带 SSL 的 Apache
apache24_enable="YES"
apache24_flags="-DSSL"
```

只要在 **/etc/rc.conf** 中启用了服务，就可以在不重启系统的情况下启动它：

```sh
# service sshd start
# service apache24 start
```

如果某个服务尚未启用，则可以通过命令行使用 `onestart` 启动它：

```sh
# service sshd onestart
```

## 5. 网络配置

与 Linux® 使用通用的 *ethX* 标识符来识别网络接口不同，FreeBSD 使用驱动程序名称后跟数字的方式来标识网络接口。以下是 [ifconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=ifconfig&sektion=8&format=html) 输出的两个 Intel® Pro 1000 网络接口（**em0** 和 **em1**）的示例：

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

可以使用 [ifconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=ifconfig&sektion=8&format=html) 为接口分配 IP 地址。为了在重启后保持配置不变，必须将 IP 配置包含在 **/etc/rc.conf** 中。以下是 **/etc/rc.conf** 中的条目，指定了主机名、IP 地址和默认网关：

```sh
hostname="server1.example.com"
ifconfig_em0="inet 10.10.10.100 netmask 255.255.255.0"
defaultrouter="10.10.10.1"
```

以下是配置 DHCP 接口的条目：

```sh
hostname="server1.example.com"
ifconfig_em0="DHCP"
```

## 6. 防火墙

FreeBSD 并不使用 Linux® 的 IPTABLES 防火墙，而是提供了三种内核级防火墙的选择：

- [PF](https://docs.freebsd.org/en/books/handbook/#firewalls-pf)
- [IPFILTER](https://docs.freebsd.org/en/books/handbook/#firewalls-ipf)
- [IPFW](https://docs.freebsd.org/en/books/handbook/#firewalls-ipfw)

PF 由 OpenBSD 项目开发，并移植到 FreeBSD。PF 作为 IPFILTER 的替代品创建，语法与 IPFILTER 类似。PF 可以与 [altq(4)](https://man.freebsd.org/cgi/man.cgi?query=altq&sektion=4&format=html) 配合使用，提供 QoS 特性。

以下是允许入站 SSH 的 PF 示例条目：

```sh
pass in on $ext_if inet proto tcp from any to ($ext_if) port 22
```

IPFILTER 是 Darren Reed 开发的防火墙应用程序。它并非专门为 FreeBSD 开发，已经移植到多个操作系统，包括 NetBSD、OpenBSD、SunOS、HP/UX 和 Solaris。

允许入站 SSH 的 IPFILTER 语法如下：

```sh
pass in on $ext_if proto tcp from any to any port = 22
```

IPFW 是由 FreeBSD 开发和维护的防火墙。它可以与 [dummynet(4)](https://man.freebsd.org/cgi/man.cgi?query=dummynet&sektion=4&format=html) 配合使用，提供流量整形功能，并模拟不同类型的网络连接。

允许入站 SSH 的 IPFW 语法如下：

```sh
ipfw add allow tcp from any to me 22 in via $ext_if
```

## 7. 更新 FreeBSD

更新 FreeBSD 系统有两种方法：从源代码更新或二进制更新。

从源代码更新是最复杂的更新方法，但却是最灵活的。该过程涉及将本地的 FreeBSD 源代码副本与 FreeBSD Git 仓库同步。在本地源代码更新到最新版本后，就可以编译新的内核和用户空间。

二进制更新类似于使用 `yum` 或 `apt-get` 更新 Linux® 系统。在 FreeBSD 中，可以使用 [freebsd-update(8)](https://man.freebsd.org/cgi/man.cgi?query=freebsd-update&sektion=8&format=html) 获取新的二进制更新并安装它们。这些更新可以使用 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 定时。

> **注意**
>
> 在使用 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 定时更新时，在 [crontab(1)](https://man.freebsd.org/cgi/man.cgi?query=crontab&sektion=1&format=html) 中使用 `freebsd-update cron`，减少大量机器同时拉取更新的可能性：
>
> ```sh
> 0 3 * * * root /usr/sbin/freebsd-update cron
> ```

有关源代码和二进制更新的更多信息，请参阅 FreeBSD 手册中的 [更新章节](https://docs.freebsd.org/en/books/handbook/#updating-upgrading-freebsdupdate)。

## 8. procfs: 已消失但未被遗忘

在一些 Linux® 发行版中，可以通过查看 **/proc/sys/net/ipv4/ip_forward** 来确定是否启用了 IP 转发。而在 FreeBSD 中，则使用 [sysctl(8)](https://man.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&format=html) 来查看此项以及其他系统设置。

例如，使用以下命令来确定 FreeBSD 系统是否启用了 IP 转发：

```sh
% sysctl net.inet.ip.forwarding
net.inet.ip.forwarding: 0
```

使用 `-a` 列出所有系统设置：

```sh
% sysctl -a | more
```

如果某个应用程序需要 procfs，可以将以下条目添加到 **/etc/fstab** 中：

```sh
proc                /proc           procfs  rw,noauto       0       0
```

包含 `noauto` 可以防止 **/proc** 在启动时自动挂载。

如果不重启系统，可以使用以下命令挂载文件系统：

```sh
# mount /proc
```

## 9. 常见命令

以下是一些常见命令的等效命令：

| Linux® 命令 (Red Hat/Debian)                        | FreeBSD 等效命令            | 目的         |
| ------------------------------------------------- | ----------------------- | ---------- |
| `yum install package` / `apt-get install package` | `pkg install package`   | 从远程仓库安装软件包 |
| `rpm -ivh package` / `dpkg -i package`            | `pkg add package`       | 安装本地软件包    |
| `rpm -qa` / `dpkg -l`                             | `pkg info`              | 列出已安装的软件包  |
| `lspci`                                           | `pciconf`               | 列出 PCI 设备  |
| `lsmod`                                           | `kldstat`               | 列出已加载的内核模块 |
| `modprobe`                                        | `kldload` / `kldunload` | 加载/卸载内核模块  |
| `strace`                                          | `truss`                 | 跟踪系统调用     |

## 10. 结论

本文提供了 FreeBSD 的概述。有关这些主题的更深入内容以及本文未涵盖的许多主题，请参考 [FreeBSD 手册](https://docs.freebsd.org/en/books/handbook/)。
