# 在没有远程控制台的情况下远程安装 FreeBSD 操作系统

版权 © 2008-2021 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="c6948b39-df32-4331-8603-1f51da91600c"><summary data-immersive-translate-walked="c6948b39-df32-4331-8603-1f51da91600c" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

许多制造商和销售商用来区分其产品的名称被声称为商标。当这些名称出现在本文档中，并且 FreeBSD 项目意识到商标声明时，这些名称后面会附上“™”或“®”符号。

</details>

 摘要

本文档记录了在远程系统控制台不可用时远程安装 FreeBSD 操作系统的方法。本文的主要思想是与 Martin Matuska <<a href="mailto:mm@FreeBSD.org">mm@FreeBSD.org</a>> 合作的结果，并得到了 Paweł Jakub Dawidek <<a href="mailto:pjd@FreeBSD.org">pjd@FreeBSD.org</a>> 的宝贵意见。

---

## 1. 背景

世界上有许多服务器托管提供商，但其中很少有官方支持 FreeBSD。他们通常支持在他们提供的服务器上安装 Linux®发行版。

在某些情况下，如果您请求，这些公司会为您安装所选的 Linux®发行版。通过这个选项，我们将尝试安装 FreeBSD。在其他情况下，他们可能提供一个用于紧急情况的救援系统。我们也可以将其用于我们的目的。

本文介绍了启动具有 RAID-1 和 ZFS 功能的远程 FreeBSD 安装所需的基本安装和配置步骤。

## 2. 简介

本节将总结本文的目的，并更好地解释这里涵盖的内容。本文中包含的说明将使那些使用不支持 FreeBSD 的机房设施提供的服务的人受益。

1. 正如我们在背景部分中提到的，许多著名的服务器托管公司提供某种救援系统，可以从他们的局域网引导，并通过 SSH 访问。他们通常提供此支持以帮助他们的客户修复损坏的操作系统。正如本文将解释的那样，通过这些救援系统的帮助，安装 FreeBSD 是可能的。
2. 本文的下一部分将描述如何在本地机器上配置和构建最小化的 FreeBSD。该版本最终将在远程机器上从 ramdisk 运行，这将允许我们使用 sysinstall 工具从 FTP 镜像安装完整的 FreeBSD 操作系统。
3. 本文的其余部分将描述安装过程本身，以及 ZFS 文件系统的配置。

### 2.1. 要求

要成功继续，您必须：

* 拥有具有 SSH 访问权限的网络可访问操作系统
* 理解 FreeBSD 安装过程
* 熟悉 sysinstall(8) 实用程序
* 准备好 FreeBSD 安装 SO 镜像或 CD

## 3. 准备 - mfsBSD

在 FreeBSD 安装到目标系统之前，需要构建最小的 FreeBSD 操作系统镜像，该镜像将从硬盘启动。这样新系统就可以通过网络访问，并且可以在没有远程访问系统控制台的情况下完成安装的其余部分。

mfsBSD 工具集可用于构建一个微型的 FreeBSD 镜像。正如 mfsBSD 的名称所示（"mfs" 意味着 "内存文件系统"），生成的镜像完全从 ramdisk 运行。由于这个特性，不会受限于硬盘驱动器的操作，因此可以安装完整的 FreeBSD 操作系统。mfsBSD 的主页包含指向工具集最新发布的指针。

请注意，mfsBSD 的内部结构及其完整性超出了本文的范围。有兴趣的读者应查阅 mfsBSD 的原始文档以获取更多详细信息。

下载并解压最新的 mfsBSD 发行版，并将工作目录切换到 mfsBSD 脚本所在的目录：

```
# fetch http://mfsbsd.vx.sk/release/mfsbsd-2.1.tar.gz
# tar xvzf mfsbsd-2.1.tar.gz
# cd mfsbsd-2.1/
```

### 3.1. mfsBSD 的配置

在启动 mfsBSD 之前，必须设置一些重要的配置选项。最重要的是我们必须正确设置网络设置。配置网络选项的最合适方法取决于我们是否预先知道将要使用的网络接口类型，以及要为我们的硬件加载的网络接口驱动程序。我们将看到如何在这两种情况下配置 mfsBSD。

另一个重要的设置是 root 密码。可以通过编辑 conf/loader.conf 文件来完成。请参阅包含的注释。

#### 3.1.1. 使用 conf/interfaces.conf 方法

当安装的网络接口卡未知时，可以使用 mfsBSD 的自动检测功能。mfsBSD 的启动脚本可以根据接口的 MAC 地址检测正确的驱动程序，如果我们在 conf/interfaces.conf 中设置以下选项：

```
mac_interfaces="ext1"
ifconfig_ext1_mac="00:00:00:00:00:00"
ifconfig_ext1="inet 192.168.0.2/24"
```

不要忘记将 defaultrouter 信息添加到 conf/rc.conf：

```
defaultrouter="192.168.0.1"
```

#### 3.1.2. conf/rc.conf 方法

当网络接口驱动程序已知时，使用 conf/rc.conf 更方便进行网络选项设置。该文件的语法与 FreeBSD 标准 rc.conf(5)文件中使用的语法相同。

例如，如果您知道将有一个 re(4)网络接口可用，您可以在 conf/rc.conf 中设置以下选项：

```
defaultrouter="192.168.0.1"
ifconfig_re0="inet 192.168.0.2/24"
```

### 3.2. 构建 mfsBSD 映像

构建 mfsBSD 镜像的过程非常简单直接。

第一步是挂载 FreeBSD 安装光盘，或者将安装 ISO 镜像挂载到 /cdrom。举例来说，在本文中我们假设您已经下载了 FreeBSD 10.1-RELEASE ISO。使用 mdconfig(8) 实用工具将这个 ISO 镜像挂载到 /cdrom 目录非常容易：

```
# mdconfig -a -t vnode -u 10 -f FreeBSD-10.1-RELEASE-amd64-disc1.iso
# mount_cd9660 /dev/md10 /cdrom
```

由于最近的 FreeBSD 发行版不包含常规的发行套件，因此需要从位于 ISO 镜像上的发行存档中提取 FreeBSD 发行文件：

```
# mkdir DIST
# tar -xvf /cdrom/usr/freebsd-dist/base.txz -C DIST
# tar -xvf /cdrom/usr/freebsd-dist/kernel.txz -C DIST
```

接下来，构建可启动的 mfsBSD 镜像：

```
# make BASE=DIST
```

|  | 上述 make 必须从 mfsBSD 目录树的顶层运行，例如~/mfsbsd-2.1/。 |
| -- | --------------------------------------------------------------- |

### 3.3. 启动 mfsBSD

现在 mfsBSD 镜像已准备就绪，必须上传到运行活动救援系统或预先安装的 Linux®发行版的远程系统。 这项任务最适合的工具是 scp：

```
# scp disk.img root@192.168.0.2:.
```

要正确引导 mfsBSD 镜像，必须将其放置在给定机器的第一个（可引导）设备上。 这可以通过以下示例实现，假设 sda 是第一个可引导的磁盘设备：

```
# dd if=/root/disk.img of=/dev/sda bs=1m
```

如果一切顺利，镜像现在应在第一个设备的 MBR 中，并且可以重新启动机器。 使用 ping(8)工具监视机器正常引导。 一旦它重新上线，应该可以作为配置的密码的用户 root 通过 ssh(1)访问它。

## 安装 FreeBSD 操作系统

mfsBSD 已成功引导，并且应该可以通过 ssh(1) 登录。本节将描述如何创建和标记切片，设置 gmirror 用于 RAID-1，并使用 sysinstall 安装 FreeBSD 操作系统的最小发行版。

### 准备硬盘

分配 FreeBSD 的磁盘空间是第一项任务，即创建分区和分片。显然，当前运行的系统已完全加载到系统内存中，因此在操作硬盘时不会出现问题。要完成此任务，可以使用 sysinstall 或者与 bsdlabel(8) 结合使用 fdisk(8)。

首先，将所有系统磁盘标记为空。对每个硬盘重复以下命令：

```
# dd if=/dev/zero of=/dev/ad0 count=2
```

接下来，使用您偏好的工具创建分区并标记它们。虽然使用 sysinstall 更容易，但一个更强大且可能更少错误的方法是使用标准的基于文本的 UNIX® 工具，如 fdisk(8) 和 bsdlabel(8)，本节也将介绍它们。前者在 FreeBSD 手册的安装 FreeBSD 章节中有详细说明。正如在介绍中提到的，本文将展示如何设置具有 RAID-1 和 ZFS 功能的系统。我们的设置将包括一个小的 gmirror(8) 镜像的 /（根）、/usr 和 /var 数据集，其余的磁盘空间将用于 zpool(8) 镜像的 ZFS 文件系统。请注意，ZFS 文件系统将在成功安装和引导 FreeBSD 操作系统后配置。

以下示例将描述如何创建切片和标签，在每个分区上初始化 gmirror(8)，以及如何在每个镜像分区中创建 UFS2 文件系统：

```
# fdisk -BI /dev/ad0 
# fdisk -BI /dev/ad1
# bsdlabel -wB /dev/ad0s1 
# bsdlabel -wB /dev/ad1s1
# bsdlabel -e /dev/ad0s1 
# bsdlabel /dev/ad0s1 > /tmp/bsdlabel.txt && bsdlabel -R /dev/ad1s1 /tmp/bsdlabel.txt 
# gmirror label root /dev/ad[01]s1a 
# gmirror label var /dev/ad[01]s1d
# gmirror label usr /dev/ad[01]s1e
# gmirror label -F swap /dev/ad[01]s1b 
# newfs /dev/mirror/root 
# newfs /dev/mirror/var
# newfs /dev/mirror/usr
```

|  | 创建覆盖整个磁盘的切片并初始化给定磁盘第 0 扇区中的引导代码。对系统中的所有硬盘重复此命令。                                                                                            |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  | 为每个磁盘写入标准标签，包括自举代码。                                                                                                                                                 |
|  | 现在，手动编辑所给磁盘的标签。参考 bsdlabel(8) 手册页面，了解如何创建分区。创建分区 a 用于 /（根）文件系统， b 用于交换空间， d 用于 /var， e 用于 /usr，最后创建 f 以备将来用于 ZFS。 |
|  | 导入最近为第二块硬盘创建的标签，以便两块硬盘标签相同。                                                                                                                                 |
|  | 在每个分区上初始化 gmirror(8)。                                                                                                                                                        |
|  | 注意， -F 用于交换分区。这指示 gmirror(8) 在断电/系统故障后假定设备处于一致状态。                                                                                                      |
|  | 在每个镜像分区上创建一个 UFS2 文件系统。                                                                                                                                               |

### 4.2. 系统安装

这是最重要的部分。本节将描述如何在我们在前一节准备好的硬驱动器上安装 FreeBSD 的最小发行版。为实现这一目标，所有文件系统都需要被挂载，以便 sysinstall 可以将 FreeBSD 的内容写入硬盘：

```
# mount /dev/mirror/root /mnt
# mkdir /mnt/var /mnt/usr
# mount /dev/mirror/var /mnt/var
# mount /dev/mirror/usr /mnt/usr
```

当你完成时，启动 sysinstall(8)。从主菜单中选择自定义安装。选择“选项”并按 Enter 键。借助箭头键，将光标移动到 Install Root 项目上，按空格并将其更改为/mnt。按 Enter 键提交更改，并通过按 q 键退出“选项”菜单。

|  | 请注意，这一步非常重要，如果跳过， sysinstall 将无法安装 FreeBSD。 |
| -- | -------------------------------------------------------------------- |

转到“发行版”菜单，使用箭头键移动光标到 Minimal ，按空格键进行检查。本文使用最小发行版来节省网络流量，因为系统本身将通过 ftp 进行安装。通过选择 Exit 来退出此菜单。

|  | 分区和标签菜单将被跳过，因为现在这些是无用的。 |
| -- | ------------------------------------------------ |

在“媒体”菜单中，选择 FTP 。选择最近的镜像，并让 sysinstall 假设网络已经配置好。您将被返回到自定义菜单。

最后，通过选择最后一个选项 Commit 来执行系统安装。在安装完成后，退出 sysinstall 。

### 4.3. 安装后步骤

FreeBSD 操作系统现在应该已安装；但是，流程还没有结束。有必要执行一些安装后步骤，以便未来引导 FreeBSD 并能够登录到系统中。

您现在必须 chroot(8)进入新安装的系统以完成安装。使用以下命令：

```
# chroot /mnt
```

要完成我们的目标，请执行以下步骤：

* 将 GENERIC 内核复制到/boot/kernel 目录中：

  ```
  # cp -Rp /boot/GENERIC/* /boot/kernel
  ```
* 创建/etc/rc.conf、/etc/resolv.conf 和/etc/fstab 文件。不要忘记正确设置网络信息，并在/etc/rc.conf 中启用 sshd。/etc/fstab 的内容将类似于以下内容：

  ```
  # Device                Mountpoint      FStype  Options         Dump    Pass#
  /dev/mirror/swap        none            swap    sw              0       0
  /dev/mirror/root        /               ufs     rw              1       1
  /dev/mirror/usr         /usr            ufs     rw              2       2
  /dev/mirror/var         /var            ufs     rw              2       2
  /dev/cd0                /cdrom          cd9660  ro,noauto       0       0
  ```
* 创建/boot/loader.conf 并添加以下内容：

  ```
  geom_mirror_load="YES"
  zfs_load="YES"
  ```
* 执行以下命令，这将使 ZFS 在下一次启动时可用：

  ```
  # sysrc zfs_enable="YES"
  ```
* 使用 adduser(8) 工具将额外的用户添加到系统中。不要忘记将用户添加到 wheel 组，这样您可以在重启后获得 root 访问权限。
* 仔细检查所有的设置。

系统现在应该已准备好进行下一次启动。使用 reboot(8) 命令来重启您的系统。

## 5. ZFS

如果您的系统经受住了重启的考验，现在应该可以登录了。欢迎来到全新的 FreeBSD 安装，远程执行而无需使用远程控制台！

唯一剩下的步骤就是配置 zpool(8)并创建一些 zfs(8)文件系统。创建和管理 ZFS 非常简单。首先，创建一个镜像池：

```
# zpool create tank mirror /dev/ad[01]s1f
```

接下来，创建一些文件系统：

```
# zfs create tank/ports
# zfs create tank/src
# zfs set compression=gzip tank/ports
# zfs set compression=on tank/src
# zfs set mountpoint=/usr/ports tank/ports
# zfs set mountpoint=/usr/src tank/src
```

这就是全部。如果您对在 FreeBSD 上的 ZFS 的更多细节感兴趣，请参考 FreeBSD Wiki 的 ZFS 部分。
