# FreeBSD 上的打印服务器（CUPS）

<details open="" data-immersive-translate-walked="e7dd97af-87af-4a4b-b80e-b22f7c4e3d03"><summary data-immersive-translate-walked="e7dd97af-87af-4a4b-b80e-b22f7c4e3d03" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

许多制造商和销售商用来区分其产品的名称被声称为商标。在本文档中出现这些名称时，如果 FreeBSD 项目知道商标声明，这些名称后面会附上 “™” 或 “®” 符号。

</details>

 摘要

有关在 FreeBSD 上配置 CUPS 的文章。

---

## 1. 介绍 Common Unix Printing System（CUPS）

CUPS，通用 UNIX 打印系统，为基于 UNIX®的操作系统提供便携式打印层。它由 Easy Software Products 开发，旨在为所有 UNIX®供应商和用户推广标准打印解决方案。

CUPS 使用 Internet 打印协议（IPP）作为管理打印作业和队列的基础。还支持行式打印守护程序（LPD）、服务器消息块（SMB）和 AppSocket（又称 JetDirect）协议，但功能有所减少。CUPS 添加了网络打印机浏览和基于 PostScript 打印机描述（PPD）的打印选项，以支持 UNIX®下的实际打印。因此，CUPS 非常适合在 FreeBSD、Linux®、Mac OS® X 或 Windows®等混合环境中共享和访问打印机。

CUPS 的主要网站是 http://www.cups.org/。

## 安装 CUPS 打印服务器

要使用预编译的二进制文件安装 CUPS，请在 root 终端中使用以下命令：

```
# pkg install cups
```

其他可选的但建议安装的软件包是 print/gutenprint 和 print/hplip，它们都为各种打印机添加驱动程序和实用程序。 安装完成后，可以在目录 /usr/local/etc/cups 中找到 CUPS 配置文件。

## 3. 配置 CUPS 打印服务器

安装后，必须编辑一些文件以配置 CUPS 服务器。首先，创建或修改文件/etc/devfs.rules，并添加以下信息以设置所有潜在打印机设备的适当权限，并将打印机与 cups 用户组关联：

```
[system=10]
add path 'unlpt*' mode 0660 group cups
add path 'ulpt*' mode 0660 group cups
add path 'lpt*' mode 0660 group cups
add path 'usb/X.Y.Z' mode 0660 group cups
```

>请注意，X、Y 和 Z 应替换为列在/dev/usb 目录中与打印机对应的目标 USB 设备。要找到正确的设备，请检查 dmesg(8) 的输出，其中 ugenX.Y 列出打印机设备，它是指向/dev/usb 中 USB 设备的符号链接。 

接下来，在 /etc/rc.conf 中添加以下两行：

```
cupsd_enable="YES"
devfs_system_ruleset="system"
```

这两个条目将在启动时启动 CUPS 打印服务器，并调用上面创建的本地 devfs 规则。

要在某些 Microsoft® Windows® 客户端下启用 CUPS 打印，请取消注释下面的行 /usr/local/etc/cups/mime.types 和 /usr/local/etc/cups/mime.convs 中。

```
application/octet-stream
```

一旦完成这些更改，必须重新启动 devfs(8)和 CUPS 系统，可以通过重新启动计算机或在 root 终端中发出以下两个命令来完成：

```
# /etc/rc.d/devfs restart
# /usr/local/etc/rc.d/cupsd restart
```

## 4. 在 CUPS 打印服务器上配置打印机

安装和配置 CUPS 系统后，管理员可以开始配置连接到 CUPS 打印服务器的本地打印机。这一部分的过程与在其他基于 UNIX®的操作系统（如 Linux®发行版）上配置 CUPS 打印机非常相似，甚至可以说是完全相同的。

通过基于 Web 的界面管理和管理 CUPS 服务器的主要手段是通过 Web 浏览器进入 http://localhost:631。如果 CUPS 服务器位于网络上的另一台机器上，请在浏览器的 URL 栏中输入服务器的本地 IP 地址代替 localhost 。CUPS Web 界面相当简单明了，有用于管理打印机和打印作业、授权用户等部分。此外，在管理屏幕的右侧有几个复选框，允许轻松访问常见更改的设置，例如是否共享连接到系统的已发布打印机、是否允许远程管理 CUPS 服务器，以及是否允许用户对打印机和打印作业获得额外访问和特权。

添加打印机通常只需在 CUPS Web 界面的管理屏幕上单击“添加打印机”，或者也可以单击管理屏幕上的“找到新打印机”按钮之一。在呈现“设备”下拉框时，只需选择所需的本地连接打印机，然后继续整个过程。如果已添加了打印/gutenprint-cups 或打印/hplip ports或上述的软件包，则在随后的屏幕上将提供额外的打印驱动程序，这些驱动程序可能提供更稳定或更多功能。

## 5. 配置 CUPS 客户端

一旦配置了 CUPS 服务器并将打印机添加并发布到网络，下一步就是配置客户端，即将访问 CUPS 服务器的机器。如果只有一台既充当服务器又充当客户端的桌面计算机，那么很多这些信息可能就不需要了。

### 5.1. UNIX® 客户端

还需要在 UNIX® 客户端上安装 CUPS。CUPS 一旦安装在客户端后，网络上共享的 CUPS 打印机通常会被 GNOME 或 KDE 等各种桌面环境的打印机管理器自动发现。或者，可以在客户端机器上访问本地 CUPS 界面，网址是 http://localhost:631，并在管理部分点击“添加打印机”。当出现“设备”下拉框时，只需选择网络 CUPS 打印机（如果它被自动发现的话），或者选择 ipp 或 http 并输入网络 CUPS 打印机的 IPP 或 HTTP URI，通常采用以下两种语法之一：

```
ipp://server-name-or-ip/printers/printername
```

```
http://server-name-or-ip:631/printers/printername
```

如果 CUPS 客户端在查找网络上共享的其他 CUPS 打印机时遇到困难，有时添加或创建一个文件/usr/local/etc/cups/client.conf 并添加以下单个条目会有所帮助：

```
ServerName server-ip
```

在这种情况下，server-ip 将被替换为网络上 CUPS 服务器的本地 IP 地址。

### 5.2. Windows® 客户端

Windows® XP 之前的版本不能直接与基于 IPP 的打印机进行网络连接。然而，Windows® XP 及更高版本具有此功能。因此，在这些版本的 Windows® 中添加 CUPS 打印机非常容易。通常，Windows® 管理员会运行 Windows® Add Printer 向导，选择 Network Printer ，然后按以下语法输入 URI：

```
http://server-name-or-ip:631/printers/printername
```

如果使用没有原生 IPP 打印支持的较旧版本的 Windows®，那么连接到 CUPS 打印机的一般方法是同时使用 net/samba416 和 CUPS，这是本章节范围之外的主题。

## 6. CUPS 故障排除

CUPS 的困难通常在于权限问题。首先，请仔细检查上述提到的 devfs(8) 权限。接下来，请检查文件系统中创建的设备的实际权限。还有帮助的是确保你的用户是 cups 组的成员。如果在 CUPS Web 界面的管理部分中的权限检查框似乎无法正常工作，另一个修复方法可能是手动备份位于 /usr/local/etc/cups/cupsd.conf 的主 CUPS 配置文件，并编辑各种配置选项并尝试不同的配置选项组合。以下列出了一个样本 /usr/local/etc/cups/cupsd.conf 供测试。请注意，这个样本 cupsd.conf 为了更简单的配置而牺牲了安全性；一旦管理员成功连接到 CUPS 服务器并配置客户端，建议重新审视此配置文件并开始限制访问。

```
# Log general information in error_log - change "info" to "debug" for
# troubleshooting...
LogLevel info

# Administrator user group...
SystemGroup wheel

# Listen for connections on Port 631.
Port 631
#Listen localhost:631
Listen /var/run/cups.sock

# Show shared printers on the local network.
Browsing On
BrowseOrder allow,deny
#BrowseAllow @LOCAL
BrowseAllow 192.168.1.* # change to local LAN settings
BrowseAddress 192.168.1.* # change to local LAN settings

# Default authentication type, when authentication is required...
DefaultAuthType Basic
DefaultEncryption Never # comment this line to allow encryption

# Allow access to the server from any machine on the LAN
<Location />
  Order allow,deny
  #Allow localhost
  Allow 192.168.1.* # change to local LAN settings
</Location>

# Allow access to the admin pages from any machine on the LAN
<Location /admin>
  #Encryption Required
  Order allow,deny
  #Allow localhost
  Allow 192.168.1.* # change to local LAN settings
</Location>

# Allow access to configuration files from any machine on the LAN
<Location /admin/conf>
  AuthType Basic
  Require user @SYSTEM
  Order allow,deny
  #Allow localhost
  Allow 192.168.1.* # change to local LAN settings
</Location>

# Set the default printer/job policies...
<Policy default>
  # Job-related operations must be done by the owner or an administrator...
  <Limit Send-Document Send-URI Hold-Job Release-Job Restart-Job Purge-Jobs 
Set-Job-Attributes Create-Job-Subscription Renew-Subscription Cancel-Subscription 
Get-Notifications Reprocess-Job Cancel-Current-Job Suspend-Current-Job Resume-Job 
CUPS-Move-Job>
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>

  # All administration operations require an administrator to authenticate...
  <Limit Pause-Printer Resume-Printer Set-Printer-Attributes Enable-Printer 
Disable-Printer Pause-Printer-After-Current-Job Hold-New-Jobs Release-Held-New-Jobs 
Deactivate-Printer Activate-Printer Restart-Printer Shutdown-Printer Startup-Printer 
Promote-Job Schedule-Job-After CUPS-Add-Printer CUPS-Delete-Printer CUPS-Add-Class 
CUPS-Delete-Class CUPS-Accept-Jobs CUPS-Reject-Jobs CUPS-Set-Default>
    AuthType Basic
    Require user @SYSTEM
    Order deny,allow
  </Limit>

  # Only the owner or an administrator can cancel or authenticate a job...
  <Limit Cancel-Job CUPS-Authenticate-Job>
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>

  <Limit All>
    Order deny,allow
  </Limit>
</Policy>
```
