# FreeBSD 和固态硬盘

版权所有 © 2001 - 2021 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="295e5e81-1012-437e-8a29-9dee61d2ab35"><summary data-immersive-translate-walked="295e5e81-1012-437e-8a29-9dee61d2ab35" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

制造商和销售商用于区分其产品的许多名称被声明为商标。在本文件中出现这些名称时，如果 FreeBSD 项目意识到商标声明，则这些名称后面会跟着“™”或“®”符号。

</details>

 摘要

本文介绍了在 FreeBSD 中使用固态硬盘设备创建嵌入式系统的方法。

嵌入式系统由于缺少整体运动部件（硬盘驱动器），具有增强的稳定性优势。然而，需要注意系统中通常可用的低磁盘空间和存储介质的耐久性。

要涵盖的具体主题包括适用于 FreeBSD 磁盘使用的固态媒体类型和属性，以及在这种环境下感兴趣的内核选项，自动化初始化此类系统的 rc.initdiskless 机制，以及只读文件系统的需求，并从头开始构建文件系统。文章将以一些适用于小型和只读 FreeBSD 环境的一般策略作结论。

---

## 1. 固态硬盘设备

本文的范围将仅限于由闪存制成的固态硬盘设备。 闪存是一种固态存储器（没有移动部件），是非易失性的（即使断开所有电源源也能保持数据）。 闪存可以承受巨大的物理冲击，并且速度相对较快（本文中涵盖的闪存解决方案对于写操作比 EIDE 硬盘稍慢，但对读操作则快得多）。 闪存的一个非常重要的方面，其后果将在本文后面讨论，是每个扇区具有有限的重写能力。 在将扇区的写入、擦除和重新写入多次之前，闪存扇区将变得永久不可用。 尽管许多闪存产品自动映射坏块，并且有些甚至均匀分配写操作到整个单元，但事实仍然存在，即设备上可以写入的数量是有限的。 竞争单位在其规格中具有每个扇区 1,000,000 到 10,000,000 次写入。 由于环境温度的影响，这个数字会有所变化。

具体来说，我们将讨论 ATA 兼容的紧凑型闪存单元，这些单元作为数字相机的存储介质非常流行。 特别有趣的是它们可以直接连接到 IDE 总线，并且兼容 ATA 命令集。 因此，通过一个非常简单和低成本的适配器，这些设备可以直接连接到计算机的 IDE 总线上。 一旦以这种方式实现，像 FreeBSD 这样的操作系统会将该设备视为正常的硬盘（尽管容量较小）。

其他固态硬盘解决方案确实存在，但它们的价格昂贵、不为人知以及相对不易使用的特点使它们超出了本文的范围。

## 2. 内核选项

一些内核选项对于创建嵌入式 FreeBSD 系统的人特别重要。

所有使用闪存作为系统磁盘的嵌入式 FreeBSD 系统都会对内存磁盘和内存文件系统感兴趣。由于闪存内存的写入次数有限，磁盘和磁盘上的文件系统很可能是以只读方式挂载的。在这种环境中，诸如/tmp 和/var 等文件系统被挂载为内存文件系统，以允许系统创建日志、更新计数器和临时文件。内存文件系统是成功实现固态 FreeBSD 实施的重要组成部分。

您应确保在内核配置文件中存在以下行：

```
options         MD_ROOT         # md device usable as a potential root device
```

## 3. rc 子系统和只读文件系统

嵌入式 FreeBSD 系统的启动后初始化由 /etc/rc.initdiskless 控制。

/etc/rc.d/var 挂载/var 作为内存文件系统，使用 mkdir(1)命令在/var 中创建可配置目录列表，并更改其中一些目录的模式。在执行/etc/rc.d/var 时，另一个 rc.conf 变量起作用 - varsize 。基于 rc.conf 中该变量的值，/etc/rc.d/var 创建了一个/var 分区：

```
varsize=8192
```

请记住，默认情况下此值以扇区为单位。

/var 是读写文件系统的事实是一个重要区别，因为根分区（以及闪存设备上可能有的其他分区）应该是以只读方式挂载的。请记住，在固态磁盘设备中，我们详细介绍了闪存存储的限制 - 特别是有限的写入能力。不能过分强调不要在闪存设备上以读写方式挂载文件系统，以及不使用交换文件的重要性。在繁忙的系统上，交换文件可能会在不到一年的时间内使闪存设备失效。大量日志记录或临时文件的创建和删除也会造成相同的问题。因此，除了从/etc/fstab 中删除 swap 条目外，还应将每个文件系统的 Options 字段更改为 ro ，如下所示:

```
# Device                Mountpoint      FStype  Options         Dump    Pass#
/dev/ad0s1a             /               ufs     ro              1       1
```

一些在常规系统中的应用程序将立即因此更改而开始出现故障。例如，由于/etc/rc.d/var 创建的/var 中缺少 cron 表，cron 将无法正常运行，并且由于只读文件系统和/var 中缺少的项目，syslog 和 dhcp 也将遇到问题。这些只是暂时的问题，通过《面向小型和只读环境的系统策略》中针对其他常见软件包执行的解决方案进行了解决。

一个重要的事情要记住的是，使用/etc/fstab 挂载为只读的文件系统可以随时通过执行命令来转为读写：

```
# /sbin/mount -uw partition
```

并且可以通过执行命令将其切换回只读状态：

```
# /sbin/mount -ur partition
```

## 4. 从头开始构建文件系统

由于 ATA 兼容的紧凑闪存卡被 FreeBSD 识别为普通的 IDE 硬盘，理论上你可以使用 kern 和 mfsroot 软盘或从光盘安装 FreeBSD。

然而，即使使用常规安装程序安装较小的 FreeBSD 系统也会产生大于 200 兆字节的大小。大多数人将使用更小的闪存设备（128 兆字节被认为相当大 - 32 或甚至 16 兆字节很常见），因此使用普通机制进行安装是不可能的-即使是最小的传统安装也没有足够的磁盘空间。

通过常规手段将 FreeBSD 安装到普通硬盘，是克服这一空间限制的最简单方法。安装完成后，精简操作系统以适应闪存介质的大小，然后对整个文件系统进行打包。以下步骤将指导您准备闪存介质以存放打包后的文件系统。请注意，由于不执行常规安装，需要手动执行分区、标签化、文件系统创建等操作。除了 kern 和 mfsroot 软盘外，还需要使用 fixit 软盘。

1. 使用 kern 和 mfsroot 软盘启动后，在安装菜单中选择 custom 。在定制安装菜单中，选择 partition 。在分区菜单中，使用 d 删除所有现有分区。删除所有现有分区后，使用 c 创建一个分区，并接受分区大小的默认值。当要求输入分区类型时，请确保值设置为 165 。现在按下 w 将此分区表写入磁盘（此选项在屏幕上不可见）。如果使用 ATA 兼容的紧凑型闪存卡，应选择 FreeBSD 引导管理器。现在按 q 退出分区菜单。您将再次显示引导管理器菜单 - 重复您之前的选择。
2. 退出定制安装菜单，在主安装菜单中选择 fixit 选项。进入 fixit 环境后，输入以下命令创建闪存设备上的文件系统：

    ```
    # disklabel -e /dev/ad0c
    ```

    在这一点上，您将在磁盘标签命令的支持下进入 vi 编辑器。接下来，您需要在文件末尾添加 a: 行。这个 a: 行应该看起来像：

    ```
    a:      123456  0       4.2BSD  0       0
    ```

    其中 123456 是与现有 c: 条目相同的数字，以便大小相同。基本上，您正在将现有 c: 行复制为 a: 行，确保 fstype 为 4.2BSD 。保存文件并退出。

    ```
    # disklabel -B -r /dev/ad0c
    # newfs /dev/ad0a
    ```
3. 将您的文件系统放在闪存媒体上，安装新准备的闪存媒体：

    ```
    # mount /dev/ad0a /flash
    ```

    将这台机器连接到网络，这样我们就可以传输我们的 tar 文件，并将其解压到我们的闪存媒体文件系统上。一个操作的示例：

    ```
    # ifconfig xl0 192.168.0.10 netmask 255.255.255.0
    # route add default 192.168.0.1
    ```

    现在机器已经连上网络了，传输你的 tar 文件。此时你可能会面临一个困境 - 如果你的闪存空间只有 128 兆字节，而你的 tar 文件大于 64 兆字节，那么你不能同时拥有 tar 文件和解压后的文件，否则你会空间不足。解决这个问题的一个方法是，在使用 FTP 时，可以在文件传输过程中解压文件。如果你以这种方式进行文件传输，你就永远不会在磁盘上同时拥有 tar 文件和其内容：

    ```
    ftp> get tarfile.tar "| tar xvf -"
    ```

    如果你的 tar 文件是经过 gzip 压缩的，你也可以这样做：

    ```
    ftp> get tarfile.tar "| zcat | tar xvf -"
    ```

    将您的 tarred 文件系统内容放在闪存文件系统上后，您可以卸载闪存并重新启动：

    ```
    # cd /
    # umount /flash
    # exit
    ```

    假设在正常硬盘上构建文件系统时配置正确（在读取-只读文件系统时，必要的选项已编译到内核中），您现在应该成功引导您的 FreeBSD 嵌入式系统。

## 5. 适用于小型和只读环境的系统策略

在  rc  子系统和只读文件系统中，有人指出 /var 文件系统由 /etc/rc.d/var 构建，并且只读根文件系统的存在会导致 FreeBSD 使用的许多常见软件包出现问题。本文将提供成功运行 cron、syslog、ports 安装和 Apache web 服务器的建议。

### 5.1. Cron

启动时，/var 会使用 /etc/mtree/BSD.var.dist 列表通过 /etc/rc.d/var 填充，因此会创建 cron、cron/tabs、at 和其他一些标准目录。

但是，这并不能解决在重启过程中跨重启保留 cron 表的问题。当系统重新启动时，位于内存中的/var 文件系统将消失，您可能在其中拥有的任何 cron 表也将消失。因此，一个解决方案是为需要这些 cron 表的用户创建 cron 表，将您的/文件系统挂载为读/写并将这些 cron 表复制到一个安全的位置，例如/etc/tabs，然后在系统初始化过程中在创建该目录后向/etc/rc.initdiskless 的末尾添加一行，将这些 cron 表复制到/var/cron/tabs。您还可能需要添加一行，在/etc/rc.initdiskless 中改变您创建的目录和复制文件的模式和权限的行。

### 5.2. 系统日志

syslog.conf 指定了存在于/var/log 中的某些日志文件的位置。这些文件并不是由/etc/rc.d/var 在系统初始化时创建的。因此，在/etc/rc.d/var 的某个位置，在创建/var 中的目录的部分之后，您需要添加类似这样的内容：

```
# touch /var/log/security /var/log/maillog /var/log/cron /var/log/messages
# chmod 0644 /var/log/*
```

### 5.3. Ports 安装

在讨论成功使用 ports 树所需的更改之前，有必要提醒一下您的闪存介质上文件系统的只读属性。由于它们是只读的，您需要使用挂载语法将它们临时挂载为读写方式，该语法显示在《 rc 子系统与只读文件系统》中。在完成任何维护工作后，您应该始终重新将这些文件系统重新挂载为只读状态-对闪存介质的不必要写入可能会大大缩短其寿命。

为了能够进入 ports 目录并成功运行 make install ，我们必须在非内存文件系统上创建一个包目录，用于跟踪我们的包在重启时的状态。由于对于安装软件包而言需要将文件系统挂载为读写方式，因此可以合理地假设闪存介质上的某个区域也可以用于将软件包信息写入其中。

首先，创建一个软件包数据库目录。通常这个目录在 /var/db/pkg 下，但我们不能将它放在那里，因为系统每次启动时都会消失。

```
# mkdir /etc/pkg
```

现在，在 /etc/rc.d/var 中添加一行，将 /etc/pkg 目录链接到 /var/db/pkg。例如：

```
# ln -s /etc/pkg /var/db/pkg
```

现在，每当您将文件系统挂载为读写模式并安装软件包时， make install 将会生效，并且软件包信息将成功写入 /etc/pkg（因为此时文件系统已经挂载为读写），这将始终对操作系统可用作 /var/db/pkg。

### 5.4. Apache Web Server

>仅当 Apache 被设置为在 /var 之外写入其 pid 或日志信息时，才需要执行本节中的步骤。默认情况下，Apache 将其 pid 文件保存在 /var/run/httpd.pid 中，并将日志文件保存在 /var/log 中。 

现在假设 Apache 将其日志文件保存在 /var 之外的目录 apache_log_dir 中。当此目录位于只读文件系统上时，Apache 将无法保存任何日志文件，并且可能会出现工作问题。如果是这样，则需要在 /etc/rc.d/var 中添加一个新目录到要在 /var 中创建的目录列表中，并将 apache_log_dir 链接到 /var/log/apache。还需要为此新目录设置权限和所有权。

首先，在 /etc/rc.d/var 中添加目录 log/apache 到要创建的目录列表中。

其次，在目录创建部分后添加以下命令到 /etc/rc.d/var 中：

```
# chmod 0774 /var/log/apache
# chown nobody:nobody /var/log/apache
```

最后，移除现有的 apache_log_dir 目录，并用链接替换它：

```
# rm -rf apache_log_dir
# ln -s /var/log/apache apache_log_dir
```
