# FreeBSD 上的打印服务器（CUPS）

- 原文：[CUPS on FreeBSD](https://docs.freebsd.org/en/articles/cups/)

## 摘要

本文介绍了如何在 FreeBSD 上配置 CUPS。


## 1. 了解通用 UNIX 打印系统（CUPS）

CUPS（Common UNIX Printing System）为基于 UNIX® 的操作系统提供可移植的打印层。它由 Easy Software Products 开发，旨在为所有 UNIX® 供应商和用户推广标准的打印解决方案。

CUPS 使用互联网打印协议（IPP）作为管理打印任务和队列的基础。它还支持 Line Printer Daemon（LPD）、Server Message Block（SMB）和 AppSocket（又名 JetDirect）协议，但功能有所减少。CUPS 增加了网络打印机浏览和基于 PostScript 打印机描述（PPD）的打印选项，支持在 UNIX® 环境中进行实际的打印操作。因此，CUPS 非常适合在 FreeBSD、Linux®、Mac OS® X 或 Windows® 的混合环境中共享和访问打印机。

CUPS 的官方网站是 [http://www.cups.org/](http://www.cups.org/)。

## 2. 安装 CUPS 打印服务器

要使用预编译的二进制文件安装 CUPS，可以在 root 终端中执行以下命令：

```sh
# pkg install cups
```

其他可选但推荐的包有 [print/gutenprint](https://cgit.freebsd.org/ports/tree/print/gutenprint/) 和 [print/hplip](https://cgit.freebsd.org/ports/tree/print/hplip/)，它们为多种打印机添加了驱动程序和实用工具。安装完成后，CUPS 的配置文件可以在目录 **/usr/local/etc/cups** 中找到。

## 3. 配置 CUPS 打印服务器

安装完成后，需要编辑一些文件来配置 CUPS 服务器。首先，创建或修改文件 **/etc/devfs.rules**，并添加以下内容，以设置所有潜在打印机设备的正确权限，并将打印机与 `cups` 用户组关联：

```ini
[system=10]
add path 'unlpt*' mode 0660 group cups
add path 'ulpt*' mode 0660 group cups
add path 'lpt*' mode 0660 group cups
add path 'usb/X.Y.Z' mode 0660 group cups
```

>**注意**
>
>*X*、*Y* 和 *Z* 应替换为 **/dev/usb** 目录中与打印机对应的目标 USB 设备。要查找正确的设备，可以查看 [dmesg(8)](https://man.freebsd.org/cgi/man.cgi?query=dmesg&sektion=8&format=html) 的输出，**ugenX.Y** 列出的是打印机设备，它是 **/dev/usb** 中 USB 设备的符号链接。

接下来，在 **/etc/rc.conf** 中添加以下两行：

```ini
cupsd_enable="YES"
devfs_system_ruleset="system"
```

这两项设置分别用于在启动时启动 CUPS 打印服务器和调用上述创建的本地 devfs 规则。

为了在某些 Microsoft® Windows® 客户端下启用 CUPS 打印，应该取消注释 **/usr/local/etc/cups/mime.types** 和 **/usr/local/etc/cups/mime.convs** 文件中的以下行：

```sh
application/octet-stream
```

完成这些更改后，必须重新启动 [devfs(8)](https://man.freebsd.org/cgi/man.cgi?query=devfs&sektion=8&format=html) 和 CUPS 系统，可以通过重启计算机或在 root 终端中执行以下两个命令：

```sh
# service devfs restart
# service cupsd restart
```

## 4. 配置 CUPS 打印服务器上的打印机

在安装并配置 CUPS 系统后，管理员可以开始配置附加到 CUPS 打印服务器的本地打印机。这一过程与在其他基于 UNIX® 的操作系统（如 Linux® 发行版）上配置 CUPS 打印机非常相似，甚至完全相同。

管理和维护 CUPS 服务器的主要方式是通过基于 Web 的界面，可以通过启动浏览器并在浏览器的 URL 栏中输入 [http://localhost:631](http://localhost:631/) 来访问。如果 CUPS 服务器位于网络上的另一台机器上，只需用服务器的本地 IP 地址替换 `localhost`。CUPS 的 Web 界面颇为直观，其中有打印机和打印任务管理、用户授权等部分。此外，在管理界面的右侧有几个复选框，方便访问常用设置，如是否共享连接到系统的已发布打印机、是否允许远程管理 CUPS 服务器，以及是否允许用户对打印机和打印任务授予额外的访问权限。

添加打印机通常很简单，只需在 CUPS Web 界面的管理屏幕上点击 `Add Printer`，或者点击 `New Printers Found` 按钮。当出现 `Device` 下拉框时，只需选择所需的本地连接打印机，然后继续操作。如果之前安装了 Ports 或包 [print/gutenprint-cups](https://cgit.freebsd.org/ports/tree/print/gutenprint-cups/) 或 [print/hplip](https://cgit.freebsd.org/ports/tree/print/hplip/)，那么后续界面中会提供额外的打印驱动程序，可能提供更多的稳定性或功能。

## 5. 配置 CUPS 客户端

在 CUPS 服务器配置完成、打印机添加并发布到网络之后，下一步是配置客户端，即将访问 CUPS 服务器的机器。如果有桌面计算机既充当服务器又充当客户端，那么很多信息可能不需要。

### 5.1. UNIX® 客户端

CUPS 也需要在 UNIX® 客户端上安装。在客户端上安装了 CUPS 后，通常桌面环境（如 GNOME 或 KDE）的打印管理器会自动发现网络上共享的 CUPS 打印机。或者，可以在客户端机器上通过 [http://localhost:631](http://localhost:631/) 访问本地 CUPS 界面，并在管理部分点击 `Add Printer`。当出现 `Device` 下拉框时，如果自动发现了网络 CUPS 打印机，只需选择它；或者选择 `ipp` 或 `http`，然后输入网络 CUPS 打印机的 IPP 或 HTTP URI，通常采用以下两种格式之一：

```sh
ipp://server-name-or-ip/printers/printername
```

```sh
http://server-name-or-ip:631/printers/printername
```

如果 CUPS 客户端难以发现网络上共享的其他 CUPS 打印机，有时可以添加或创建文件 **/usr/local/etc/cups/client.conf**，并在其中添加以下条目：

```sh
ServerName server-ip
```

在这种情况下，*server-ip* 应替换为网络上 CUPS 服务器的本地 IP 地址。

### 5.2. Windows® 客户端

在 XP 之前的 Windows® 版本中，原生不支持与基于 IPP 的打印机进行网络连接。然而，Windows® XP 及更高版本支持这一功能。因此，在这些版本的 Windows® 中添加 CUPS 打印机相当简单。一般来说，Windows® 管理员会运行 Windows® `Add Printer` 向导，选择 `Network Printer`，然后输入以下语法的 URI：

```sh
http://server-name-or-ip:631/printers/printername
```

如果使用的是较旧版本的 Windows®，没有原生的 IPP 打印支持，那么连接到 CUPS 打印机的常见方式是将 [net/samba416](https://cgit.freebsd.org/ports/tree/net/samba416/) 与 CUPS 一起使用，这超出了本章的讨论范围。

## 6. CUPS 故障排除

CUPS 的问题通常与权限有关。首先，仔细检查如上所述的 [devfs(8)](https://man.freebsd.org/cgi/man.cgi?query=devfs&sektion=8&format=html) 权限。接着，检查文件系统中创建的设备的实际权限。确认用户属于 `cups` 组也有帮助。如果 CUPS Web 界面中的权限复选框似乎无法正常工作，可以尝试手动备份位于 **/usr/local/etc/cups/cupsd.conf** 的主配置文件，并编辑其中的各种配置选项，尝试不同的配置组合。以下是一个可以测试的示例 **/usr/local/etc/cups/cupsd.conf** 配置文件。请注意，此示例 **cupsd.conf** 为了更容易配置牺牲了一些安全性；若管理员成功连接到 CUPS 服务器并配置好客户端，建议重新审视该配置文件并开始限制访问。


```ini
# 在 error_log 中记录一般信息 - 若进行故障排除，请将 "info" 改为
# "debug"...
LogLevel info

# 管理员用户组...
SystemGroup wheel

# 在端口 631 上监听连接。
Port 631
#Listen localhost:631
Listen /var/run/cups.sock

# 显示局域网共享的打印机。
Browsing On
BrowseOrder allow,deny
#BrowseAllow @LOCAL
BrowseAllow 192.168.1.* # 更改为本地 LAN 设置
BrowseAddress 192.168.1.* # 更改为本地 LAN 设置

# 需要身份验证时的默认身份验证类型...
DefaultAuthType Basic
DefaultEncryption Never # 注释掉此行以允许加密

# 允许 LAN 上任何计算机访问服务器
<Location />
  Order allow,deny
  #Allow localhost
  Allow 192.168.1.* # 更改为本地 LAN 设置
</Location>

# 允许 LAN 上任何计算机访问管理员页面
<Location /admin>
  # 要求加密
  Order allow,deny
  #Allow localhost
  Allow 192.168.1.* # 更改为本地 LAN 设置
</Location>

# 允许 LAN 上任何计算机访问配置文件
<Location /admin/conf>
  AuthType Basic
  Require user @SYSTEM
  Order allow,deny
  #Allow localhost
  Allow 192.168.1.* # 更改为本地 LAN 设置
</Location>

# 设置默认打印机/作业策略...
<Policy default>
  # 与作业相关的操作必须由所有者或管理员执行...
  <Limit Send-Document Send-URI Hold-Job Release-Job Restart-Job Purge-Jobs \
Set-Job-Attributes Create-Job-Subscription Renew-Subscription Cancel-Subscription \
Get-Notifications Reprocess-Job Cancel-Current-Job Suspend-Current-Job Resume-Job \
CUPS-Move-Job>
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>

  # 所有管理操作都需要管理员身份验证...
  <Limit Pause-Printer Resume-Printer Set-Printer-Attributes Enable-Printer \
Disable-Printer Pause-Printer-After-Current-Job Hold-New-Jobs Release-Held-New-Jobs \
Deactivate-Printer Activate-Printer Restart-Printer Shutdown-Printer Startup-Printer \
Promote-Job Schedule-Job-After CUPS-Add-Printer CUPS-Delete-Printer CUPS-Add-Class \
CUPS-Delete-Class CUPS-Accept-Jobs CUPS-Reject-Jobs CUPS-Set-Default>
    AuthType Basic
    Require user @SYSTEM
    Order deny,allow
  </Limit>

  # 只有所有者或管理员可以取消或认证作业...
  <Limit Cancel-Job CUPS-Authenticate-Job>
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>

  <Limit All>
    Order deny,allow
  </Limit>
</Policy>
```


