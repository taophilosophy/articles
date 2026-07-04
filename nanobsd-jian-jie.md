# NanoBSD 简介

- 原文链接：[Introduction to NanoBSD](https://docs.freebsd.org/en/articles/nanobsd/)

## 摘要

本文提供了有关 NanoBSD 工具的信息，NanoBSD 用于为嵌入式应用程序创建 FreeBSD 系统映像，适用于 USB 密钥、存储卡或其他大容量存储介质。


## 1. NanoBSD 介绍

NanoBSD 是由 Poul-Henning Kamp \[[phk@FreeBSD.org](mailto:phk@FreeBSD.org)] 开发的工具，目前由 Warner Losh \[[imp@FreeBSD.org](mailto:imp@FreeBSD.org)] 维护。它创建用于嵌入式应用的 FreeBSD 系统映像，适用于 USB 密钥、存储卡或其他大容量存储介质。

它可以用来构建专门的安装映像，旨在便于安装和维护通常称为“计算机家电”的系统。计算机家电将硬件和软件捆绑在一起，这意味着所有应用程序都是预先安装好的。家电设备连接到现有网络后，可以（几乎）立即开始工作。

NanoBSD 的特点包括：

- Ports 和包的使用与 FreeBSD 相同 - 每个应用程序都可以像在 FreeBSD 中一样在 NanoBSD 映像中安装并使用。
- 无功能丢失 - 如果在 FreeBSD 中可以做某件事，那么在 NanoBSD 中也能做到，除非在创建 NanoBSD 映像时明确移除了特定的功能或特性。
- 运行时一切为只读 - 拔掉电源是安全的。系统非正常关机后，不需要运行 [fsck(8)](https://man.freebsd.org/cgi/man.cgi?query=fsck&sektion=8&format=html)。
- 易于构建和定制 - 仅需一个 shell 脚本和一个配置文件，就可以构建满足任意需求的精简定制映像。

## 2. NanoBSD 使用手册

### 2.1. NanoBSD 的设计

映像文件被放置在介质上后，就可以启动 NanoBSD。大容量存储介质默认分为三个部分：

- 两个映像分区：`code#1` 和 `code#2`。
- 配置文件分区，可以在运行时挂载到 **/cfg** 目录下。

这些分区通常是只读挂载的。

**/etc** 和 **/var** 目录是 [md(4)](https://man.freebsd.org/cgi/man.cgi?query=md&sektion=4&format=html)（malloc）磁盘。

配置文件分区在 **/cfg** 目录下持久化。它包含 **/etc** 目录的文件，在系统启动后会短暂地以只读方式挂载，因此，如果希望修改后的文件在系统重启后仍然存在，需要将修改后的文件从 **/etc** 复制回 **/cfg** 目录。

**示例 1. 对 /etc/resolv.conf 进行持久化更改**

```ini
# vi /etc/resolv.conf
[...]
# mount /cfg
# cp /etc/resolv.conf /cfg
# umount /cfg
```

>**注意**
>
>配置文件分区 **/cfg** 应该仅在启动时或覆盖配置文件时挂载。始终保持 **/cfg** 挂载并不是一个好主意，尤其是在 NanoBSD 系统运行在可能因大量写入而受影响的大容量存储介质上时（例如，当文件系统同步器将数据刷新到系统磁盘时）。

### 2.2. 构建 NanoBSD 映像

构建 NanoBSD 需要 FreeBSD 的源代码。获取源代码的命令为：

```sh
# git clone https://git.FreeBSD.org/src.git /usr/src
```

更多详细信息，请参阅 [这里](https://docs.freebsd.org/en/books/handbook/cutting-edge#updating-src-obtaining-src) 的步骤。

NanoBSD 映像是通过简单的 **nanobsd.sh** shell 脚本构建的，该脚本位于 **/usr/src/tools/tools/nanobsd** 目录中。该脚本可创建映像，可以使用 [dd(1)](https://man.freebsd.org/cgi/man.cgi?query=dd&sektion=1&format=html) 工具将其复制到存储介质中。

构建 NanoBSD 映像所需的命令为：

```sh
# cd /usr/src/tools/tools/nanobsd ①
# sh nanobsd.sh ②
# cd /usr/obj/nanobsd.full ③
# dd if=_.disk.full of=/dev/da0 bs=64k ④
```

- ① 将当前目录更改为 NanoBSD 构建脚本的基础目录。
- ② 启动构建过程。
- ③ 将当前目录更改为存放构建好映像的目录。
- ④ 将 NanoBSD 安装到存储介质上。

#### 2.2.1. 构建 NanoBSD 映像时的选项

在构建 NanoBSD 映像时，可以在命令行向 **nanobsd.sh** 传递多个构建选项。这些选项对构建过程有显著影响。

一些选项用于控制输出的详细程度：

- `-h`：打印帮助摘要页面。
- `-q`：使输出更静默。
- `-v`：使输出更加详细。

还有一些选项可以限制构建过程。有时不需要从源代码重建所有内容，特别是当映像已经构建并且只做了少量修改时。

- `-k`：不构建内核。
- `-w`：不构建 world。
- `-b`：既不构建内核也不构建 world。
- `-i`：完全不构建磁盘映像。由于文件不会被创建，因此无法使用 [dd(1)](https://man.freebsd.org/cgi/man.cgi?query=dd&sektion=1&format=html) 将其复制到存储介质。
- `-f`：不构建第一个分区的磁盘映像（对于升级目的很有用）。
- `-p`：不准备映像。跳过运行自定义和早期自定义脚本，以从 world、kernel 或包中进行增量映像优化。
- `-n`：向 `buildworld` 和 `buildkernel` 添加 `-DNO_CLEAN` 参数。还会保留先前构建中已经构建的所有文件。

可以使用配置文件调整任何所需的元素。使用 `-c` 加载配置文件。

最后的选项是：

- `-K`：不安装内核。没有内核的磁盘映像将无法完成正常的启动过程。
- `-W`：不安装 world。
- `-B`：不安装内核和 world。
- `-I`：从现有构建或安装构建磁盘映像。不构建或安装内核、world 和 etc 配置文件，仅创建磁盘映像。

#### 2.2.2. 完整的映像构建过程

完整的映像构建过程经过多个步骤。实际执行的步骤将取决于开始运行脚本时所选择的选项。如果脚本在没有任何特定选项的情况下运行，以下是将发生的步骤：

1. `run_early_customize`：在提供的配置文件中定义的命令。
2. `clean_build`：通过删除先前构建的文件来清理构建环境。
3. `make_conf_build`：从 `CONF_WORLD` 和 `CONF_BUILD` 变量组合生成 make.conf 文件。
4. `build_world`：构建 world。
5. `build_kernel`：构建内核文件。
6. `clean_world`：清理目标目录。
7. `make_conf_install`：从 `CONF_WORLD` 和 `CONF_INSTALL` 变量组合生成 make.conf 文件。
8. `install_world`：安装 `buildworld` 期间构建的所有文件。
9. `install_etc`：根据 `make distribution` 命令安装 **/etc** 目录中的必要文件。
10. `setup_nanobsd_etc`：此时开始进行与 NanoBSD 相关的首次配置。创建 **/etc/diskless** 并将根文件系统定义为只读。
11. `install_kernel`：安装内核和模块文件。
12. `run_customize`：调用用户定义的所有自定义例程。
13. `setup_nanobsd`：设置特殊的配置目录布局。将 **/usr/local/etc** 移动到 **/etc/local**，并从 **/etc/local** 创建符号链接到 **/usr/local/etc**。
14. `prune_usr`：删除 **/usr** 中的空目录。
15. `run_late_customize`：此时可以运行最后的自定义脚本。
16. `fixup_before_diskimage`：列出所有安装的文件并生成 metalog。
17. `create_diskimage`：根据提供的磁盘几何参数创建实际的磁盘映像。
18. `last_orders`：目前无操作。

### 2.3. 定制 NanoBSD 映像

这可能是 NanoBSD 最重要也是最有趣的功能。在使用 NanoBSD 开发时，你将花费大部分时间在这一部分。

以下命令会强制 **nanobsd.sh** 从当前目录中的 **myconf.nano** 配置文件读取配置：

```sh
# sh nanobsd.sh -c myconf.nano
```

定制有两种方式：

- 配置选项
- 自定义函数

#### 2.3.1. 配置选项

通过配置设置，可以配置传递给 NanoBSD 构建过程中的 `buildworld` 和 `installworld` 阶段的选项，以及传递给 NanoBSD 主构建过程的内部选项。通过这些选项，可以将系统缩减至仅适用于 64MB 的存储空间。你可以使用配置选项进一步精简 FreeBSD，直到只剩下内核和两个或三个用户空间文件。

配置文件由配置选项组成，这些选项会覆盖默认值。最重要的指令包括：

- `NANO_NAME` - 构建名称（用于构造工作目录名称）。
- `NANO_SRC` - 用于构建映像的源代码树路径。
- `NANO_KERNEL` - 用于构建内核的内核配置文件名称。
- `CONF_BUILD` - 传递给 `buildworld` 阶段的选项。
- `CONF_INSTALL` - 传递给 `installworld` 阶段的选项。
- `CONF_WORLD` - 传递给 `buildworld` 和 `installworld` 阶段的选项。
- `FlashDevice` - 定义使用的媒体类型。有关更多细节，请查看 **FlashDevice.sub**。

根据所需的 NanoBSD 类型，还有许多其他可能相关的配置选项。

##### 2.3.1.1. 一般定制

设计上有三个阶段可以进行更改，这些更改会影响构建过程，只需在提供的配置文件中设置相应的变量：

- `run_early_customize`：在其他任何操作之前。
- `run_customize`：所有标准文件布置之后。
- `run_late_customize`：在实际 NanoBSD 映像构建之前的最后阶段。

为了在任何这些步骤中定制 NanoBSD 映像，最好为相应的变量添加特定值。

`NANO_EARLY_CUSTOMIZE` 变量在构建过程的第一步使用。此时，尚无示例说明如何使用该变量，但未来可能会有所变化。

`NANO_CUSTOMIZE` 变量在内核、world 和 **etc** 配置文件安装完成、并且 **etc** 文件设置为 NanoBSD 安装后使用。因此，这是构建过程中的正确步骤，可以在此步骤调整配置选项并添加包，例如在 `cust_nobeastie` 示例中所示。

`NANO_LATE_CUSTOMIZE` 变量在磁盘映像创建之前使用，因此这是更改任何内容的最后时刻。请记住，`setup_nanobsd` 例程已执行，**etc**、**conf** 和 **cfg** 目录及其子目录已修改，因此此时不应更改它们。相反，可以在此时添加或删除特定的文件。

##### 2.3.1.2. 启动选项

还有一些变量可以改变 NanoBSD 映像的启动方式。两个选项会传递给 [boot0cfg(8)](https://man.freebsd.org/cgi/man.cgi?query=boot0cfg&sektion=8&format=html) 来初始化磁盘映像的启动扇区：

- `NANO_BOOT0CFG`
- `NANO_BOOTLOADER`

通过 `NANO_BOOTLOADER` 可以选择引导加载程序文件。最常见的选项是 **boot0sio** 和 **boot0**，具体选择取决于设备是否有串口。最好避免提供不同的引导加载程序，但这是可能的。如果要这样做，最好查看 [FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook/boot) 中关于启动过程的章节。

通过 `NANO_BOOT0CFG`，可以调整启动过程，比如选择 NanoBSD 映像实际从哪个分区启动。修改此变量的默认值之前，最好查看 [boot0cfg(8)](https://man.freebsd.org/cgi/man.cgi?query=boot0cfg&sektion=8&format=html) 页面。可能有用的更改是启动过程的超时设置。为此，可以将 `NANO_BOOT0CFG` 变量更改为 `"-o packet -s 1 -m 3 -t 36"`。这样启动过程大约在 2 秒后开始，因为等待 10 秒钟再启动是比较少见的需求。

值得注意的是，`NANO_BOOT2CFG` 变量仅在 `cust_comconsole` 例程中使用，当设备有串口且所有控制台输入输出都必须通过串口进行时，可以在 `NANO_CUSTOMIZE` 步骤中调用此例程。请确保检查串口的相关参数，设置错误的参数可能会导致其无法使用。

##### 2.3.1.3. 磁盘映像创建

在启动过程的最后，是磁盘映像的创建。通过此步骤，NanoBSD 脚本会生成可以直接复制到设备上的磁盘映像文件，这样设备就可以启动并开始工作。

有许多变量需要正确设置，才能使脚本生成可用的磁盘映像。

- `NANO_DRIVE` 变量必须在运行时设置为媒体的驱动器名称。通常，默认值 `ada0` 代表设备上的第一个 `IDE`/`ATA`/`SATA` 设备是正确的，但也可以使用不同类型的存储设备，如 USB 密钥，此时应使用 `da0`。
- `NANO_MEDIASIZE` 变量必须设置为将要使用的存储媒体的大小（以 512 字节扇区为单位）。如果设置错误，NanoBSD 映像可能无法启动，启动时会显示有关磁盘几何形状不正确的警告信息。
- **/etc**、**/var** 和 **/tmp** 目录在启动时作为 [md(4)](https://man.freebsd.org/cgi/man.cgi?query=md&sektion=4&format=html)（malloc）磁盘分配；因此，它们的大小可以根据设备需求进行调整。`NANO_RAM_ETCSIZE` 变量设置 **/etc** 的大小；而 `NANO_RAM_TMPVARSIZE` 变量设置 **/var** 和 **/tmp** 目录的大小，因为 **/tmp** 被符号链接到 **/var/tmp**。默认情况下，这两个 malloc 磁盘的大小分别为 20MB。它们可以随时更改，但通常 **/etc** 的大小不会增长太多，因此 20MB 是不错的起点，而 **/var** 和特别是 **/tmp** 可能会变得更大，因此在使用时需要小心。对于内存受限的系统，可以选择较小的文件系统大小。
- 由于 NanoBSD 主要设计用于构建嵌入式设备的系统映像，假设使用的存储媒体将相对较小。因此，所布置的文件系统配置为具有较小的块大小（4KB）和较小的碎片大小（512B）。可以通过 `NANO_NEWFS` 变量修改文件系统的配置选项，但语法必须遵循 [newfs(8)](https://man.freebsd.org/cgi/man.cgi?query=newfs&sektion=8&format=html) 命令格式。此外，默认情况下，文件系统启用了 Soft Updates。可以查看 [FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook/) 了解更多相关信息。
- 通过使用 `NANO_CODESIZE`、`NANO_CONFSIZE` 和 `NANO_DATASIZE` 变量，可以设置不同的分区大小，单位为 512 字节扇区。`NANO_CODESIZE` 定义了前两个映像分区的大小：`code#1` 和 `code#2`。它们必须足够大，以容纳 `buildworld` 和 `buildkernel` 过程产生的所有文件。`NANO_CONFSIZE` 定义了配置文件分区的大小，因此不需要很大，但不要设置得太小，以至于无法容纳所有配置文件。最后，`NANO_DATASIZE` 定义了可选分区的大小，可以在设备上使用。最后一个分区可以用来，例如，存储在磁盘上动态生成的文件。

#### 2.3.2. 自定义函数

可以使用配置文件中的 shell 函数来精细调整 NanoBSD。以下示例演示了自定义函数的基本模型：

```sh
cust_foo () (
	echo "bar=baz" > \
		${NANO_WORLDDIR}/etc/foo
)
customize_cmd cust_foo
```

更有用的自定义函数示例如下，它将 **/etc** 目录的默认大小从 5MB 更改为 30MB：

```sh
cust_etc_size () (
	cd ${NANO_WORLDDIR}/conf
	echo 30000 > default/etc/md_size
)
customize_cmd cust_etc_size
```

有一些预定义的默认自定义函数可以直接使用：

- `cust_comconsole` - 禁用 [getty(8)](https://man.freebsd.org/cgi/man.cgi?query=getty&sektion=8&format=html) 在 VGA 设备（`/dev/ttyv*` 设备节点）上的使用，并启用 COM1 串口作为系统控制台。
- `cust_allow_ssh_root` - 允许 `root` 用户通过 [sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html) 登录。
- `cust_install_files` - 从 **nanobsd/Files** 目录安装文件，该目录包含一些用于系统管理的有用脚本。
- `cust_pkgng` - 从 **nanobsd/Pkg** 目录安装包（还需要 `pkg-*` 包来引导安装）。

#### 2.3.3. 添加包

可以向 NanoBSD 映像添加包，以提供特定的功能。为此，可以：

- 将 `cust_pkgng` 添加到 `NANO_CUSTOMIZE` 变量中，或
- 在自定义配置文件中添加 `'customize_cmd cust_pkgng'` 命令。

这两种方法都能达到相同的结果：启动 `cust_pkgng` 例程。该例程将遍历 `NANO_PACKAGE_DIR` 目录，查找所有包或仅查找 `NANO_PACKAGE_LIST` 变量中的包列表。

在标准的 FreeBSD 环境中通过 `pkg` 安装应用程序时，安装过程通常会将配置文件放入 **/usr/local/etc** 目录，将启动脚本放入 **/usr/local/etc/rc.d** 目录。因此，在安装所需的包之后，需要对它们进行配置，以便它们可以直接开箱即用。为此，必须将必要的配置文件安装到正确的目录中。这可以通过编写专门的例程来实现，或者可以使用通用的 `cust_install_files` 例程，从 **/usr/src/tools/tools/nanobsd/Files** 目录正确放置文件。通常，还需要在 **/etc/rc.conf** 文件中为每个包添加一条语句，有时需要多条语句。

#### 2.3.4. 配置文件示例

构建自定义 NanoBSD 映像的完整配置文件示例如下：

```sh
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

所有构建和安装编译选项都可以在 [src.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=src.conf&sektion=5&format=html) 手册页中找到，但并不是所有选项都可以或应该在构建 NanoBSD 映像时使用。构建和安装选项应根据要构建的映像的需求进行定义。

例如，`ftp` 客户端和服务器可能不需要。将 `WITHOUT_FTP=TRUE` 添加到 `CONF_BUILD` 部分的配置文件中，可以避免构建这些组件。此外，如果 NanoBSD 器件不用于构建程序，则可以在 `CONF_INSTALL` 部分中添加 `WITHOUT_BINUTILS=TRUE`，但不要在 `CONF_BUILD` 部分中添加，因为它们将在构建 NanoBSD 映像时使用。

通过编译选项不构建某些特定程序，能缩短整体构建时间并减小磁盘映像的所需大小，而不安装这些特定程序则不会减少整体构建时间。

### 2.4. 更新 NanoBSD

NanoBSD 的更新过程相对简单：

1. 按照通常的方式构建新的 NanoBSD 映像。
2. 将新映像上传到运行中的 NanoBSD 设备的未使用的分区。
   这一步与最初安装 NanoBSD 的主要区别在于，现在使用的是 `_.disk.image` 映像（它包含的是单一系统分区的映像），而不是 `_.disk.full`（它包含整个磁盘的映像）。
3. 重启，并从新安装的分区启动系统。
4. 如果一切顺利，升级完成。
5. 如果出现问题，重新启动回到以前的分区（它包含旧的、正常工作的映像），以尽快恢复系统功能。修复新版本中的任何问题，然后重复该过程。

要将新映像安装到正在运行的 NanoBSD 系统中，可以使用 **/root** 目录中的 **updatep1** 或 **updatep2** 脚本，具体取决于当前系统运行的分区。

根据托管新 NanoBSD 映像的主机上可用的服务和首选的传输类型，可以选择以下三种方式之一：

#### 2.4.1. 使用 [ftp(1)](https://man.freebsd.org/cgi/man.cgi?query=ftp&sektion=1&format=html)

如果传输速度是首要考虑因素，请使用以下示例：

```sh
# ftp myhost
get _.disk.image "| sh updatep1"
```

#### 2.4.2. 使用 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html)

如果首选安全传输，请考虑使用以下示例：

```sh
# ssh myhost cat _.disk.image.gz | zcat | sh updatep1
```

#### 2.4.3. 使用 [nc(1)](https://man.freebsd.org/cgi/man.cgi?query=nc&sektion=1&format=html)

如果远程主机没有运行 [ftpd(8)](https://man.freebsd.org/cgi/man.cgi?query=ftpd&sektion=8&format=html) 或 [sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html) 服务，可以尝试以下示例：

1. 首先，在提供映像的主机上打开 TCP 监听端口，并使其将映像发送到客户端：

   ```sh
   myhost# nc -l 2222 < _.disk.image
   ```

   >**注意**
   >
   >确保所选端口未被防火墙阻止，以便能接收来自 NanoBSD 主机的传入连接。

2. 连接到提供新映像的主机，并执行 **updatep1** 脚本：

   ```sh
   # nc myhost 2222 | sh updatep1
   ```
