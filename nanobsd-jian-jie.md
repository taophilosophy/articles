# NanoBSD 简介

2006 年版权所有 © FreeBSD 文档项目

<details open="" data-immersive-translate-walked="b6947a84-7ced-4201-a3ad-883cfa3f929f"><summary data-immersive-translate-walked="b6947a84-7ced-4201-a3ad-883cfa3f929f" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

制造商和销售商使用的许多名称来区分其产品被声明为商标。在本文档中出现这些名称时，如果 FreeBSD 项目意识到了商标声明，这些名称将后跟“™”或“®”符号。

</details>

摘要

本文档提供有关 NanoBSD 工具的信息，这些工具可用于创建适用于嵌入式应用的 FreeBSD 系统镜像，适合用于 U 盘、存储卡或其他大容量存储介质。

---

## 1. NanoBSD 介绍

NanoBSD 是由 Poul-Henning Kamp <phk@FreeBSD.org>开发的工具，现在由 Warner Losh <imp@FreeBSD.org>维护。它为嵌入式应用程序创建一个 FreeBSD 系统映像，适用于 USB 键、存储卡或其他大容量存储介质。

它可用于构建专门的安装映像，旨在便于安装和维护通常称为“计算机设备”的系统。计算机设备将其硬件和软件捆绑在产品中，这意味着所有应用程序均预安装。该设备连接到现有网络后，几乎可以立即开始工作。

NanoBSD 的功能包括：

- Ports 和软件包与 FreeBSD 中的工作方式相同 - 每个应用程序都可以在 NanoBSD 镜像中安装和使用，方式与在 FreeBSD 中相同。
- 没有缺失的功能 - 如果在 FreeBSD 中可以做某事，那么在 NanoBSD 中也可以做同样的事情，除非在创建 NanoBSD 镜像时明确从中删除了特定功能或功能。
- 在运行时一切都是只读的 - 安全地拔掉电源插头是可以的。系统非正常关闭后无需运行 fsck(8)。
- 易于构建和定制 - 只需一个 shell 脚本和一个配置文件，即可构建满足任意一组需求的精简定制镜像。

## 2. NanoBSD Howto

### 2.1. NanoBSD 设计

一旦图像存在于介质上，就可以启动 NanoBSD。默认情况下，质量存储介质被分成三个部分：

- 两个图像分区： code#1 和 code#2 。
- 配置文件分区，在运行时可以挂载到/cfg 目录下。

这些分区通常是以只读方式挂载的。

/etc 和/var 目录是 md(4)（malloc）磁盘。

配置文件分区持久化存储在 /cfg 目录下。它包含 /etc 目录的文件，并在系统启动后短暂以只读方式挂载，因此在系统重新启动后，如果希望更改持久化，需要将修改过的文件从 /etc 复制回 /cfg 目录。

示例 1. 持久化更改 /etc/resolv.conf 文件

```
# vi /etc/resolv.conf
[...]
# mount /cfg
# cp /etc/resolv.conf /cfg
# umount /cfg
```

|     | 包含 /cfg 的分区应仅在启动时挂载，并在覆盖配置文件时挂载。<br /><br />持续挂载/cfg 并不是一个好主意，特别是当 NanoBSD 系统运行在可能受到大量写入分区影响的大容量存储介质上时（比如当文件系统同步器将数据刷新到系统磁盘时）。 |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 2.2. 构建 NanoBSD 映像

构建 NanoBSD 需要 FreeBSD 的源代码。要获取源代码：

```
# git clone https://git.FreeBSD.org/src.git /usr/src
```

有关更多详细信息，请按照这里的步骤进行。

使用一个简单的 nanobsd.sh shell 脚本构建 NanoBSD 镜像，该脚本可以在/usr/src/tools/tools/nanobsd 目录中找到。该脚本创建一个镜像，可以使用 dd(1)实用程序将其复制到存储介质上。

构建 NanoBSD 镜像所需的命令为：

```
# cd /usr/src/tools/tools/nanobsd
# sh nanobsd.sh
# cd /usr/obj/nanobsd.full
# dd if=_.disk.full of=/dev/da0 bs=64k
```

|     | 将当前目录更改为 NanoBSD 构建脚本的根目录。 |
| --- | ------------------------------------------- |
|     | 开始构建过程。                              |
|     | 将当前目录更改为存放构建镜像的地方。        |
|     | 将 NanoBSD 安装到存储介质上。               |

#### 构建 NanoBSD 镜像时的选项

在构建 NanoBSD 镜像时，可以在命令行上传递几个构建选项给 nanobsd.sh。这些选项可能对构建过程产生重大影响。

一些选项是为了冗长而存在的：

- -h ：打印帮助总结页面。
- -q ：使输出更安静。
- -v ：使输出更加详细

一些其他选项可用于限制构建过程。有时候没有必要从源代码重新构建所有内容，特别是如果镜像已经构建完成，只进行了少量更改。

- -k ：不要构建内核
- -w ：不建立世界
- -b ：也不要建立内核和世界
- -i ：根本不要建立磁盘映像。由于不会创建文件，因此将无法将其用 dd(1) 工具写入存储媒体。
- -f : 不要构建第一个分区的磁盘镜像（这对升级很有用）
- -n : 将 -DNO_CLEAN 添加到 buildworld ， buildkernel 。此外，所有在之前运行中已构建的文件将被保留。

可以使用配置文件来调整任意数量的元素。用 -c 加载配置文件。

最后的选项是:

- -K ：不安装内核。没有内核的磁盘映像将无法实现正常的引导顺序。

#### 2.2.2. 完整的镜像构建过程

完整的镜像构建过程经过了许多步骤。具体采取的步骤将取决于启动脚本时选择的选项。假设脚本在没有特定选项的情况下运行，这就是将会发生的情况。

1. run_early_customize : 在提供的配置文件中定义的命令。
2. clean_build : 只是通过删除之前构建的文件来清理构建环境。
3. make_conf_build ：从 CONF_WORLD 和 CONF_BUILD 变量组装 make.conf 文件。
4. build_world ：构建世界。
5. build_kernel ：构建内核文件。
6. clean_world ：清洁目标目录。
7. make_conf_install ：从 CONF_WORLD 和 CONF_INSTALL 变量组装 make.conf。
8. install_world ：安装在 buildworld 期间构建的所有文件。
9. install_etc ：根据 make distribution 命令在/etc 目录中安装必要的文件。
10. setup_nanobsd_etc ：NanoBSD 的第一个配置特定于此阶段。创建/etc/diskless 并将根文件系统定义为只读。
11. install_kernel ：安装内核和模块文件。
12. run_customize ：将调用用户定义的所有自定义规则。
13. setup_nanobsd ：设置特殊的配置目录布局。/usr/local/etc 被移动到 /etc/local，然后创建一个符号链接，从 /etc/local 指向 /usr/local/etc。
14. prune_usr ：从 /usr 中删除空目录。
15. run_late_customize : 最后的定制脚本可以在此时运行。
16. fixup_before_diskimage : 在 metalog 中列出所有已安装的文件
17. create_diskimage : 根据磁盘几何参数创建实际的磁盘镜像。
18. last_orders ：目前什么也不做。

### 2.3. 定制化 NanoBSD 镜像

这可能是 NanoBSD 最重要和最有趣的特性。在使用 NanoBSD 进行开发时，这也是你将花费大部分时间的地方。

执行以下命令将强制 nanobsd.sh 从当前目录中的 myconf.nano 读取其配置：

```
# sh nanobsd.sh -c myconf.nano
```

定制有两种方式：

- 配置选项
- 定制函数

#### 2.3.1. 配置选项

通过配置设置，可以配置传递给 NanoBSD 构建过程的 buildworld 和 installworld 阶段的选项，以及传递给 NanoBSD 主要构建过程的内部选项。通过这些选项，可以将系统裁减到适合仅 64MB 的空间。您可以使用配置选项进一步精简 FreeBSD，直到它只包含内核和用户空间中的两三个文件。

配置文件由覆盖默认值的配置选项组成。最重要的指令是：

- NANO_NAME - 构建名称（用于构建工作目录名称）。
- NANO_SRC - 构建镜像所使用的源树路径。
- NANO_KERNEL - 用于构建内核的内核配置文件的名称。
- CONF_BUILD - 传递给构建过程中的 buildworld 阶段的选项。
- CONF_INSTALL - 传递给构建过程中的 installworld 阶段的选项。
- CONF_WORLD - 传递给构建的 buildworld 和 installworld 阶段的选项。
- FlashDevice - 定义要使用的媒体类型。详细信息请查看 FlashDevice.sub。

有许多其他配置选项，这取决于所需的 NanoBSD 类型。

##### 2.3.1.1. 通用定制

设计上，有三个阶段可以通过在提供的配置文件中设置变量来进行改变，这些变化会影响构建过程。

- run_early_customize ：在任何其他操作发生之前。
- run_customize ：在所有标准文件都摆放好之后
- run_late_customize ：在实际构建 NanoBSD 映像之前的整个过程的最后阶段

在任何这些步骤中要自定义 NanoBSD 映像，最好是向相应变量之一添加特定值。

该 NANO_EARLY_CUSTOMIZE 变量用于建筑过程的第一步。在这一点上，尚没有关于如何使用该变量的示例，但在未来可能会发生改变。

当内核、世界等配置文件已安装，并且 etc 文件已设置为 NanoBSD 安装时，使用该 NANO_CUSTOMIZE 变量。因此，在建筑过程中调整配置选项并添加包（如在 cust_nobeastie 示例中）是正确的步骤。

在创建磁盘映像之前使用该 NANO_LATE_CUSTOMIZE 变量，因此这是改变任何内容的最后时刻。请记住， setup_nanobsd 程序已执行，etc、conf 和 cfg 目录及子目录已被修改，因此现在不是更改它们的时候。相反，可以添加或删除特定文件。

##### 2.3.1.2. 引导选项

还有一些可以改变 NanoBSD 镜像引导方式的变量。两个选项传递给 boot0cfg(8) 来初始化磁盘镜像的引导扇区：

- `NANO_BOOT0CFG`
- `NANO_BOOTLOADER`

使用 NANO_BOOTLOADER 可以选择引导加载程序文件。最常见的选项是在是否带有串行 port 的设备上选择 boot0sio 或 boot0。最好避免使用不同的引导加载程序，但是这是可能的。为此，最好事先查阅 FreeBSD 手册中关于引导过程的章节。

使用 NANO_BOOT0CFG ，可以调整启动过程，比如选择 NanoBSD 镜像实际启动的分区。在更改此变量的默认值之前，最好先查看 boot0cfg(8) 页面。一个可能值得更改的选项是启动过程的超时。为此，可以将 NANO_BOOT0CFG 变量更改为 "-o packet -s 1 -m 3 -t 36" 。这样，启动过程将在大约 2 秒后开始；因为很少有人希望在实际启动前等待 10 秒。

值得一提的是： NANO_BOOT2CFG 变量仅在 cust_comconsole 例程中使用，如果设备有串行 port 并且所有控制台输入和输出都必须通过它进行，可以在 NANO_CUSTOMIZE 步骤调用该例程。请务必检查串行 port 的相关参数，因为设置错误的参数值会使其无效。

##### 2.3.1.3. 磁盘镜像创建

在引导过程的最后是磁盘镜像创建。通过这一步骤，NanoBSD 脚本提供一个文件，可以简单地复制到设备的磁盘上，这将使设备引导并启动。

有许多变量需要被正确设置，以便脚本生成可用的磁盘映像。

- NANO_DRIVE 变量必须在运行时设置为媒体的驱动器名称。通常，期望第一个 IDE / ATA / SATA 设备的默认值为 ada0 ，在设备上是正确的，但也可以使用不同类型的存储 - 如 USB 键，这种情况下，它可能是 da0。
- NANO_MEDIASIZE 变量必须设置为将要使用的存储介质的大小（以 512 字节扇区为单位）。如果设置错误，可能导致 NanoBSD 映像根本无法启动，并在启动时会出现有关磁盘几何不正确的警告消息。
- /etc、/var 和 /tmp 目录在启动时被分配为 md(4) （malloc）磁盘；因此它们的大小可以根据应用需求进行调整。 NANO_RAM_ETCSIZE 变量设置了 /etc 的大小； NANO_RAM_TMPVARSIZE 变量设置了 /var 和 /tmp 目录的大小，因为 /tmp 被符号链接到 /var/tmp。默认情况下，两个 malloc 磁盘的大小均设置为 20MB。它们可以随时更改，但通常 /etc 的大小不会增长太多，因此 20MB 是一个很好的起点，而 /var 和尤其是 /tmp 可能会在不注意的情况下增长得更大。对于内存受限的系统，可以选择较小的文件系统大小。
- 由于 NanoBSD 主要设计用于构建用于设备的系统映像，因此假定所使用的存储介质相对较小。出于这个原因，布局的文件系统被配置为具有小的块大小（4Kb）和小的片段大小（512b）。文件系统的配置选项可以通过 NANO_NEWFS 变量进行修改，但语法必须遵守 newfs(8) 命令格式。此外，默认情况下，文件系统已启用软更新。可以查阅 FreeBSD 手册获取更多信息。
- 不同的分区大小可以通过 NANO_CODESIZE ， NANO_CONFSIZE 和 NANO_DATASIZE 作为 512 字节扇区的倍数来设置。 NANO_CODESIZE 定义了前两个图像分区的大小： code#1 和 code#2 。它们必须足够大，能够容纳由 buildworld 和 buildkernel 过程产生的所有文件。 NANO_CONFSIZE 定义了配置文件分区的大小，因此不需要太大；但不要使它太小，以至于不能容纳所有配置文件。最后， NANO_DATASIZE 定义了一个可选分区的大小，可用于设备。例如，最后一个分区可以用来保存在磁盘上动态创建的文件。

#### 2.3.2. 自定义函数

可以使用 shell 函数来微调 NanoBSD 在配置文件中。以下示例说明了自定义函数的基本模型。

```
cust_foo () (
	echo "bar=baz" > \
		${NANO_WORLDDIR}/etc/foo
)
customize_cmd cust_foo
```

更有用的自定义功能示例如下，它将/etc 目录的默认大小从 5MB 更改为 30MB：

```
cust_etc_size () (
	cd ${NANO_WORLDDIR}/conf
	echo 30000 > default/etc/md_size
)
customize_cmd cust_etc_size
```

有几个默认预定义的自定义功能可供使用：

- cust_comconsole - 禁用 VGA 设备上的 getty(8)（/dev/ttyv\*设备节点），并启用 COM1 串口 port 作为系统控制台。
- cust_allow_ssh_root - 允许 root 通过 sshd(8) 登录。
- cust_install_files - 从 nanobsd/Files 目录安装文件，该目录包含一些用于系统管理的实用脚本。
- cust_pkgng - 从 nanobsd/Pkg 目录安装软件包（还需要 pkg-\* 软件包进行引导）。

#### 2.3.3. 添加软件包

可以向 NanoBSD 映像添加软件包，以在设备上提供特定功能。要这样做，可以：

- 将 cust_pkgng 添加到 NANO_CUSTOMIZE 变量中，或
- 在定制配置文件中添加一个 'customize_cmd cust_pkgng' 命令。

这两种方法都可以达到相同的效果：启动 cust_pkgng 例行程序。该例行程序将遍历 NANO_PACKAGE_DIR 目录，以查找所有软件包或仅查找 NANO_PACKAGE_LIST 变量中的软件包列表。

在标准的 FreeBSD 环境上通过 pkg 安装应用程序时，通常安装过程会将配置文件放在 usr/local/etc 目录下，并将启动脚本放在/usr/local/etc/rc.d 目录下。因此，在安装所需软件包后，需要对其进行配置，以使其能够立即启动。为此，必须将必要的配置文件安装到正确的目录中。可以通过编写专用例行程序或使用通用 cust_install_files 例行程序从/usr/src/tools/tools/nanobsd/Files 目录正确布置文件。通常，还需要为每个软件包在/etc/rc.conf 中添加一个或多个语句。

#### 2.3.4. 配置文件示例

构建定制 NanoBSD 镜像的完整配置文件示例如下：

```
NANO_NAME=custom
NANO_SRC=/usr/src
NANO_KERNEL=MYKERNEL
NANO_IMAGES=2

CONF_BUILD='
WITHOUT_KLDLOAD=YES
WITHOUT_NETGRAPH=YES
WITHOUT_PAM=YES
'

CONF_INSTALL='
WITHOUT_ACPI=YES
WITHOUT_BLUETOOTH=YES
WITHOUT_FORTRAN=YES
WITHOUT_HTML=YES
WITHOUT_LPR=YES
WITHOUT_MAN=YES
WITHOUT_SENDMAIL=YES
WITHOUT_SHAREDOCS=YES
WITHOUT_EXAMPLES=YES
WITHOUT_INSTALLLIB=YES
WITHOUT_CALENDAR=YES
WITHOUT_MISC=YES
WITHOUT_SHARE=YES
'

CONF_WORLD='
WITHOUT_BIND=YES
WITHOUT_MODULES=YES
WITHOUT_KERBEROS=YES
WITHOUT_GAMES=YES
WITHOUT_RESCUE=YES
WITHOUT_LOCALES=YES
WITHOUT_SYSCONS=YES
WITHOUT_INFO=YES
'

FlashDevice SanDisk 1G

cust_nobeastie() (
	touch ${NANO_WORLDDIR}/boot/loader.conf
	echo "beastie_disable=\"YES\"" >> ${NANO_WORLDDIR}/boot/loader.conf
)

customize_cmd cust_comconsole
customize_cmd cust_install_files
customize_cmd cust_allow_ssh_root
customize_cmd cust_nobeastie
```

所有构建和安装编译选项都可以在 src.conf(5)手册页中找到，但并非所有选项在构建 NanoBSD 镜像时都能或应该使用。构建和安装选项应根据所构建镜像的需求来定义。

例如，ftp 客户端和服务器可能不需要。在 CONF_BUILD 部分的配置文件中添加 WITHOUT_FTP=TRUE 将避免构建它们。另外，如果 NanoBSD 设备不用于构建程序，则可以将 WITHOUT_BINUTILS=TRUE 添加到 CONF_INSTALL 部分；但不能在 CONF_BUILD 部分中添加，因为它们将用于构建 NanoBSD 映像。

通过编译选项不构建特定的程序可以缩短总体构建时间并降低磁盘映像所需的大小，而不安装相同的特定程序集不会降低总体构建时间。

### 2.4. 更新 NanoBSD

NanoBSD 的更新过程相对简单：

1. 像往常一样构建一个新的 NanoBSD 映像。
2. 将新映像上传到运行中的 NanoBSD 设备的未使用分区中。此步骤与初始 NanoBSD 安装的最重要区别在于，现在安装的是*.disk.image 映像（其中包含单个系统分区的映像），而不是使用包含整个磁盘映像的*.disk.full。
3. 重新启动，并从新安装的分区启动系统。
4. 如果一切顺利，升级完成。
5. 如果出现任何问题，重新启动到先前的分区（其中包含旧的、正常运行的镜像），以尽快恢复系统功能。解决新构建的任何问题，并重复该过程。

要将新镜像安装到正在运行的 NanoBSD 系统上，可以使用位于/root 目录中的 updatep1 或 updatep2 脚本之一，具体取决于当前系统正在运行的分区。

根据托管新 NanoBSD 镜像的主机上可用的服务以及首选的传输类型，可以考虑以下三种方式之一：

#### 2.4.1. 使用 ftp(1)

如果传输速度排在第一位，请使用此示例：

```
# ftp myhost
get _.disk.image "| sh updatep1"
```

#### 2.4.2. 使用 ssh(1)

如果更倾向于安全传输，请考虑使用此示例：

```
# ssh myhost cat _.disk.image.gz | zcat | sh updatep1
```

#### 使用 nc(1)

如果远程主机既没有运行 ftpd(8) 也没有运行 sshd(8) 服务，请尝试此示例：

1. 首先，在提供图像的主机上打开一个 TCP 监听器，并让它将图像发送到客户端：

   ```
   myhost# nc -l 2222 < _.disk.image
   ```

   |     | 确保所使用的 port 没有被防火墙阻拦以接收来自 NanoBSD 主机的传入连接。 |
   | --- | --------------------------------------------------------------------- |

2. 连接到提供新镜像的主机并执行 updatep1 脚本：

   ```
   # nc myhost 2222 | sh updatep1
   ```
