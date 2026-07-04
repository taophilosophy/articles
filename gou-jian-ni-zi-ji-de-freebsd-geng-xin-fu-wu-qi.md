# 构建你自己的 FreeBSD 更新服务器

- 原文链接：[Build Your Own FreeBSD Update Server](https://docs.freebsd.org/en/articles/freebsd-update-server/)


>**警告**
>
>本文中的说明涉及较旧版本的 FreeBSD，可能无法在较新的操作系统版本中正常工作。随着 pkgbase 的出现，freebsd-update 工具预计将在未来从 FreeBSD 中移除。当这种情况发生时，本文将更新以反映新的流程，或者完全删除。

## 摘要

本文介绍了构建内部 FreeBSD 更新服务器的过程。[freebsd-update-server](https://github.com/freebsd/freebsd-update-build/) 由 FreeBSD 荣誉安全官 Colin Percival 编写。对于那些认为从官方更新服务器更新系统较为方便的用户，构建自己的 FreeBSD 更新服务器可以通过支持手动调整的 FreeBSD 版本，或通过提供一个本地镜像来帮助加速多个机器的更新，从而扩展其功能。


## 1. 致谢

本文随后在 [BSD Magazine](https://people.freebsd.org/~jgh/files/fus/BSD_03_2010_EN.pdf) 上印刷发布。

## 2. 介绍

有经验的用户或管理员通常负责多个机器或环境。他们理解维持这种基础设施的困难要求和挑战。运行 FreeBSD 更新服务器可以更容易地将安全性和软件补丁部署到选定的测试机器上，再推广到生产环境中。它还意味着多个系统可以通过本地网络而不是可能较慢的互联网连接来更新。本文概述了创建内部 FreeBSD 更新服务器的步骤。

## 3. 前提条件

要构建内部 FreeBSD 更新服务器，需满足一些要求。

- 正在运行的 FreeBSD 系统。
  >**注意**
  >
  >至少，更新需要在大于或等于目标发布版本的 FreeBSD 版本上构建。
  
- 至少 4 GB 可用空间的用户账户。这能创建 7.1 和 7.2 的更新，但确切的空间要求可能会根据版本的不同而有所变化。

- 在远程机器上的 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 账户，用于上传分发更新。

- 一台 Web 服务器，比如 [Apache](https://docs.freebsd.org/en/books/handbook/#network-apache)，其空间要求超过构建所需空间的一半。例如，7.1 和 7.2 的测试构建总共消耗 4 GB，分发这些更新所需的 Web 服务器空间为 2.6 GB。

- 基本的 POSIX shell 脚本知识，[sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html)。

## 4. 配置：安装与设置

通过安装 [devel/git](https://cgit.freebsd.org/ports/tree/devel/git/) 和 [security/ca_root_nss](https://cgit.freebsd.org/ports/tree/security/ca_root_nss/)，下载 [freebsd-update-server](https://github.com/freebsd/freebsd-update-build/) 软件，并执行：

```sh
% git clone https://github.com/freebsd/freebsd-update-build.git freebsd-update-server
```

适当地更新 **scripts/build.conf** 文件。该文件在所有构建操作中都会被引用。

以下是默认的 **build.conf**，应根据你的环境进行修改。

```ini
# FreeBSD 更新构建的主配置文件。特定版本的配置数据位于
# 脚本树的下方。

# 用于获取发布版本的位置
export FTP=ftp://ftp2.freebsd.org/pub/FreeBSD/releases ①

# 主机平台
export HOSTPLATFORM=`uname -m`

# 用于 jail 内的主机名
export BUILDHOSTNAME=${HOSTPLATFORM}-builder.daemonology.net ②

# SSH 密钥的位置
export SSHKEY=/root/.ssh/id_dsa ③

# 上传文件的 SSH 账户
MASTERACCT=builder@wadham.daemonology.net ④

# 上传文件的目录
MASTERDIR=update-master.freebsd.org ⑤
```

考虑的参数包括：

- ① 这是从中下载 ISO 镜像的位置（由 **scripts/build.subr** 中的 `fetchiso()` 子程序处理）。配置的地址不限于 FTP URI，任何由标准的 [fetch(1)](https://man.freebsd.org/cgi/man.cgi?query=fetch&sektion=1&format=html) 工具支持的 URI 方案都应当能正常工作。可以通过将默认的 **build.subr** 脚本复制到发布和架构特定的目录（即 **scripts/RELEASE/ARCHITECTURE/build.subr**）并进行本地更改来安装 `fetchiso()` 代码的自定义。
- ② 构建主机的名称。在已更新的系统上执行 `% uname -v` 时会显示此信息。
- ③ 用于上传文件到更新服务器的 SSH 密钥。可以通过输入 `ssh-keygen -t dsa` 来创建密钥对。此参数是可选的；当未定义 `SSHKEY` 时，将使用标准的密码身份验证作为备用身份验证方法。有关 SSH 和创建与使用密钥的详细信息，参见 [ssh-keygen(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-keygen&sektion=1&format=html) 手册。
- ④ 用于上传文件到更新服务器的账户。
- ⑤ 更新服务器中上传文件的目录。

随 freebsd-update-server 源码附带的默认 **build.conf** 文件适用于构建 FreeBSD 的 i386 版本。作为构建其他架构更新服务器的示例，以下步骤概述了为 amd64 配置所需的更改：

1. 为 amd64 创建构建环境：

   ```
   % mkdir -p /usr/local/freebsd-update-server/scripts/7.2-RELEASE/amd64
   ```

2. 在新创建的构建目录中安装 **build.conf** 文件。FreeBSD 7.2-RELEASE 版本在 amd64 上的构建配置选项应类似于：

   ```
   # RELEASE disc1.iso 镜像的 SHA256 哈希值。
   export RELH=1ea1f6f652d7c5f5eab7ef9f8edbed50cb664b08ed761850f95f48e86cc71ef5 ①
   # 系统、源代码和内核的组件
   export WORLDPARTS="base catpages dict doc games info manpages proflibs lib32"
   export SOURCEPARTS="base bin contrib crypto etc games gnu include krb5  \
                   lib libexec release rescue sbin secure share sys tools  \
                   ubin usbin cddl"
   export KERNELPARTS="generic"

   # 生命周期结束日期
   export EOL=1275289200 ②
   ```

- ① 所需发布版的 [sha256(1)](https://man.freebsd.org/cgi/man.cgi?query=sha256&sektion=1&format=html) 哈希值会在相应的[发布公告](https://www.freebsd.org/releases/)中公布。
- ② 要生成 **build.conf** 中的“生命周期结束”值，请参考 [FreeBSD 安全网站](https://www.freebsd.org/security/security/) 上发布的“预估 EOL”日期。`EOL` 的值可以通过使用 [date(1)](https://man.freebsd.org/cgi/man.cgi?query=date&sektion=1&format=html) 工具从网站上列出的日期生成，例如：

```sh
% date -j -f '%Y%m%d-%H%M%S' '20090401-000000' '+%s'
```


## 5. 构建更新代码

第一步是运行 **scripts/make.sh**。这将构建一些二进制文件，创建目录，并生成用于批准构建的 RSA 签名密钥。在此步骤中，最终创建签名密钥时需要提供一个密码短语。

```sh
# sh scripts/make.sh
cc -O2 -fno-strict-aliasing -pipe   findstamps.c  -o findstamps
findstamps.c: In function 'usage':
findstamps.c:45: warning: incompatible implicit declaration of built-in function 'exit'
cc -O2 -fno-strict-aliasing -pipe   unstamp.c  -o unstamp
install findstamps ../bin
install unstamp ../bin
rm -f findstamps unstamp
Generating RSA private key, 4096 bit long modulus
................................................................................++
...................++
e is 65537 (0x10001)

Public key fingerprint:
27ef53e48dc869eea6c3136091cc6ab8589f967559824779e855d58a2294de9e

Encrypting signing key for root
enter aes-256-cbc encryption password:
Verifying - enter aes-256-cbc encryption password:
```

>**注意**
>
>请记下生成的密钥指纹。该值在进行二进制更新时需要在 **/etc/freebsd-update.conf** 中使用。


此时，我们已准备好进行构建阶段。

```sh
# cd /usr/local/freebsd-update-server
# sh scripts/init.sh amd64 7.2-RELEASE
```

接下来是 *初始* 构建运行的示例。

```sh
# sh scripts/init.sh amd64 7.2-RELEASE
Mon Aug 24 16:04:36 PDT 2009 Starting fetch for FreeBSD/amd64 7.2-RELEASE
/usr/local/freebsd-update-server/work/7.2-RELE100 of  588 MB  359 kBps 00m00s
Mon Aug 24 16:32:38 PDT 2009 Verifying disc1 hash for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 16:32:44 PDT 2009 Extracting components for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 16:34:05 PDT 2009 Constructing world+src image for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 16:35:57 PDT 2009 Extracting world+src for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 23:36:24 UTC 2009 Building world for FreeBSD/amd64 7.2-RELEASE
Tue Aug 25 00:31:29 UTC 2009 Distributing world for FreeBSD/amd64 7.2-RELEASE
Tue Aug 25 00:32:36 UTC 2009 Building and distributing kernels for FreeBSD/amd64 7.2-RELEASE
Tue Aug 25 00:44:44 UTC 2009 Constructing world components for FreeBSD/amd64 7.2-RELEASE
Tue Aug 25 00:44:56 UTC 2009 Distributing source for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 17:46:18 PDT 2009 Moving components into staging area for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 17:46:33 PDT 2009 Identifying extra documentation for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 17:47:13 PDT 2009 Extracting extra docs for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 17:47:18 PDT 2009 Indexing release for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 17:50:44 PDT 2009 Indexing world0 for FreeBSD/amd64 7.2-RELEASE

Files built but not released:
Files released but not built:
Files which differ by more than contents:
Files which differ between release and build:
kernel|generic|/GENERIC/hptrr.ko
kernel|generic|/GENERIC/kernel
src|sys|/sys/conf/newvers.sh
world|base|/boot/loader
world|base|/boot/pxeboot
world|base|/etc/mail/freebsd.cf
world|base|/etc/mail/freebsd.submit.cf
world|base|/etc/mail/sendmail.cf
world|base|/etc/mail/submit.cf
world|base|/lib/libcrypto.so.5
world|base|/usr/bin/ntpq
world|base|/usr/lib/libalias.a
world|base|/usr/lib/libalias_cuseeme.a
world|base|/usr/lib/libalias_dummy.a
world|base|/usr/lib/libalias_ftp.a
...
```

然后，世界（world）的构建将再次执行，并应用世界补丁。更详细的说明可以在 **scripts/build.subr** 中找到。


>**警告**
>
>在第二次构建周期中，网络时间协议守护进程 [ntpd(8)](https://man.freebsd.org/cgi/man.cgi?query=ntpd&sektion=8&format=html) 将关闭。根据 FreeBSD 荣誉安全官 Colin Percival 的说法，“freebsd-update-server 构建代码需要识别存储在文件中的时间戳，以便在比较构建时忽略这些时间戳，从而确定哪些文件需要更新。这一时间戳查找过程通过进行两次相隔 400 天的构建，并比较其结果来实现。”

```sh
Mon Aug 24 17:54:07 PDT 2009 Extracting world+src for FreeBSD/amd64 7.2-RELEASE
Wed Sep 29 00:54:34 UTC 2010 Building world for FreeBSD/amd64 7.2-RELEASE
Wed Sep 29 01:49:42 UTC 2010 Distributing world for FreeBSD/amd64 7.2-RELEASE
Wed Sep 29 01:50:50 UTC 2010 Building and distributing kernels for FreeBSD/amd64 7.2-RELEASE
Wed Sep 29 02:02:56 UTC 2010 Constructing world components for FreeBSD/amd64 7.2-RELEASE
Wed Sep 29 02:03:08 UTC 2010 Distributing source for FreeBSD/amd64 7.2-RELEASE
Tue Sep 28 19:04:31 PDT 2010 Moving components into staging area for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 19:04:46 PDT 2009 Extracting extra docs for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 19:04:51 PDT 2009 Indexing world1 for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 19:08:04 PDT 2009 Locating build stamps for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 19:10:19 PDT 2009 Cleaning staging area for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 19:10:19 PDT 2009 Preparing to copy files into staging area for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 19:10:20 PDT 2009 Copying data files into staging area for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 12:16:57 PDT 2009 Copying metadata files into staging area for FreeBSD/amd64 7.2-RELEASE
Mon Aug 24 12:16:59 PDT 2009 Constructing metadata index and tag for FreeBSD/amd64 7.2-RELEASE

Files found which include build stamps:
kernel|generic|/GENERIC/hptrr.ko
kernel|generic|/GENERIC/kernel
world|base|/boot/loader
world|base|/boot/pxeboot
world|base|/etc/mail/freebsd.cf
world|base|/etc/mail/freebsd.submit.cf
world|base|/etc/mail/sendmail.cf
world|base|/etc/mail/submit.cf
world|base|/lib/libcrypto.so.5
world|base|/usr/bin/ntpq
world|base|/usr/include/osreldate.h
world|base|/usr/lib/libalias.a
world|base|/usr/lib/libalias_cuseeme.a
world|base|/usr/lib/libalias_dummy.a
world|base|/usr/lib/libalias_ftp.a
...
```

最后，构建完成。

```sh
Values of build stamps, excluding library archive headers:
v1.2 (Aug 25 2009 00:40:36)
v1.2 (Aug 25 2009 00:38:22)
@()FreeBSD 7.2-RELEASE 0: Tue Aug 25 00:38:29 UTC 2009
FreeBSD 7.2-RELEASE 0: Tue Aug 25 00:38:29 UTC 2009
    root@server.myhost.com:/usr/obj/usr/src/sys/GENERIC
7.2-RELEASE
Mon Aug 24 23:55:25 UTC 2009
Mon Aug 24 23:55:25 UTC 2009
 built by root@server.myhost.com on Tue Aug 25 00:16:15 UTC 2009
 built by root@server.myhost.com on Tue Aug 25 00:16:15 UTC 2009
 built by root@server.myhost.com on Tue Aug 25 00:16:15 UTC 2009
 built by root@server.myhost.com on Tue Aug 25 00:16:15 UTC 2009
Mon Aug 24 23:46:47 UTC 2009
ntpq 4.2.4p5-a Mon Aug 24 23:55:53 UTC 2009 (1)
 * Copyright (c) 1992-2009 The FreeBSD Project.
Mon Aug 24 23:46:47 UTC 2009
Mon Aug 24 23:55:40 UTC 2009
Aug 25 2009
ntpd 4.2.4p5-a Mon Aug 24 23:55:52 UTC 2009 (1)
ntpdate 4.2.4p5-a Mon Aug 24 23:55:53 UTC 2009 (1)
ntpdc 4.2.4p5-a Mon Aug 24 23:55:53 UTC 2009 (1)
Tue Aug 25 00:21:21 UTC 2009
Tue Aug 25 00:21:21 UTC 2009
Tue Aug 25 00:21:21 UTC 2009
Mon Aug 24 23:46:47 UTC 2009

FreeBSD/amd64 7.2-RELEASE initialization build complete.  Please
review the list of build stamps printed above to confirm that
they look sensible, then run
 sh -e approve.sh amd64 7.2-RELEASE
to sign the release.
```

如果一切正确，则批准构建。有关如何确认这一点的更多信息，可以在分发的源文件 **USAGE** 中找到。按照指示执行 **scripts/approve.sh**，这将签署该发布，并将组件移动到适合上传的暂存区。

```sh
# cd /usr/local/freebsd-update-server
# sh scripts/mountkey.sh
```

```sh
# sh -e scripts/approve.sh amd64 7.2-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Signing build for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Copying files to patch source directories for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Copying files to upload staging area for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:07 PDT 2009 Updating databases for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:07 PDT 2009 Cleaning staging area for FreeBSD/amd64 7.2-RELEASE
```

批准过程完成后，可以开始上传流程。

```sh
# cd /usr/local/freebsd-update-server
# sh scripts/upload.sh amd64 7.2-RELEASE
```

>**注意**
>
>如果需要重新上传更新代码，可以通过切换到目标发布的公共分发目录，并更新已上传文件的属性来完成此操作。
>
>```sh
># cd /usr/local/freebsd-update-server/pub/7.2-RELEASE/amd64
># touch -t 200801010101.01 uploaded
>```

上传的文件需要位于 Web 服务器的文档根目录中，以便更新能够分发。具体的配置将根据使用的 Web 服务器有所不同。对于 Apache Web 服务器，请参考手册中的 [Apache 服务器配置](https://docs.freebsd.org/en/books/handbook/#network-apache) 部分。

在 **/etc/freebsd-update.conf** 中更新客户端的 `KeyPrint` 和 `ServerName` 配置项，并按照手册中 [FreeBSD 更新](https://docs.freebsd.org/en/books/handbook/#updating-upgrading-freebsdupdate) 部分的说明进行更新。

>**重要**
>
>为了使 FreeBSD 更新服务器正常工作，需要为 *当前* 发布版本和 *目标升级版本* 构建更新。这对于确定不同版本之间的文件差异是必要的。例如，当将 FreeBSD 系统从 7.1-RELEASE 升级到 7.2-RELEASE 时，需要为这两个版本构建并上传更新到分发服务器。

作为参考，附上了整个 [init.sh](https://docs.freebsd.org/en/source/articles/freebsd-update-server/init.txt) 运行过程。

## 6. 构建补丁

每次发布 [安全公告](https://www.freebsd.org/security/advisories/) 或 [安全通知](https://www.freebsd.org/security/notices/) 时，都可以构建补丁更新。

在此示例中，将使用 7.1-RELEASE。

以下是针对不同发布版本构建补丁的几个假设：

- 设置正确的目录结构以进行初始构建。
- 为 7.1-RELEASE 执行初始构建。

在 **/usr/local/freebsd-update-server/patches/** 下创建相应发布版本的补丁目录。

```
% mkdir -p /usr/local/freebsd-update-server/patches/7.1-RELEASE/
% cd /usr/local/freebsd-update-server/patches/7.1-RELEASE
```

作为示例，获取 [named(8)](https://man.freebsd.org/cgi/man.cgi?query=named&sektion=8&format=html) 的补丁。阅读公告，并从 [FreeBSD 安全公告](https://www.freebsd.org/security/advisories/) 获取所需文件。有关如何解读公告的更多信息，可以参考 [FreeBSD 手册](https://docs.freebsd.org/en/books/handbook/#security-advisories)。

在 [安全简报](https://security.freebsd.org/advisories/FreeBSD-SA-09:12.bind.asc) 中，此公告被称为 `SA-09:12.bind`。下载文件后，需要将文件重命名为适当的补丁级别。建议与官方 FreeBSD 补丁级别保持一致，但文件名可以自由选择。对于此构建，遵循 FreeBSD 当前的做法，命名为 `p7`。重命名该文件：

```sh
% cd /usr/local/freebsd-update-server/patches/7.1-RELEASE/; mv bind.patch 7-SA-09:12.bind
```

>**注意**
>
>在运行补丁级别构建时，假定之前的补丁已到位。当运行补丁构建时，它会运行补丁目录中包含的所有补丁。
>
>可以向任何构建添加自定义补丁。使用数字零，或其他任何数字。

>**警告**
>
>FreeBSD 更新服务器的管理员有责任采取适当措施验证每个补丁的真实性。

此时，*diff* 已准备好进行构建。软件首先检查在运行差异构建之前，是否已在相应的发布版本上运行过 **scripts/init.sh**。

```
# cd /usr/local/freebsd-update-server
# sh scripts/diff.sh amd64 7.1-RELEASE 7
```

接下来是 *差异* 构建运行的示例。

```sh
# sh -e scripts/diff.sh amd64 7.1-RELEASE 7
Wed Aug 26 10:09:59 PDT 2009 Extracting world+src for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 17:10:25 UTC 2009 Building world for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 18:05:11 UTC 2009 Distributing world for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 18:06:16 UTC 2009 Building and distributing kernels for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 18:17:50 UTC 2009 Constructing world components for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 18:18:02 UTC 2009 Distributing source for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 11:19:23 PDT 2009 Moving components into staging area for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 11:19:37 PDT 2009 Extracting extra docs for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 11:19:42 PDT 2009 Indexing world0 for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 11:23:02 PDT 2009 Extracting world+src for FreeBSD/amd64 7.1-RELEASE-p7
Thu Sep 30 18:23:29 UTC 2010 Building world for FreeBSD/amd64 7.1-RELEASE-p7
Thu Sep 30 19:18:15 UTC 2010 Distributing world for FreeBSD/amd64 7.1-RELEASE-p7
Thu Sep 30 19:19:18 UTC 2010 Building and distributing kernels for FreeBSD/amd64 7.1-RELEASE-p7
Thu Sep 30 19:30:52 UTC 2010 Constructing world components for FreeBSD/amd64 7.1-RELEASE-p7
Thu Sep 30 19:31:03 UTC 2010 Distributing source for FreeBSD/amd64 7.1-RELEASE-p7
Thu Sep 30 12:32:25 PDT 2010 Moving components into staging area for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:32:39 PDT 2009 Extracting extra docs for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:32:43 PDT 2009 Indexing world1 for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:35:54 PDT 2009 Locating build stamps for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:36:58 PDT 2009 Reverting changes due to build stamps for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:37:14 PDT 2009 Cleaning staging area for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:37:14 PDT 2009 Preparing to copy files into staging area for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:37:15 PDT 2009 Copying data files into staging area for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:43:23 PDT 2009 Copying metadata files into staging area for FreeBSD/amd64 7.1-RELEASE-p7
Wed Aug 26 12:43:25 PDT 2009 Constructing metadata index and tag for FreeBSD/amd64 7.1-RELEASE-p7
...
Files found which include build stamps:
kernel|generic|/GENERIC/hptrr.ko
kernel|generic|/GENERIC/kernel
world|base|/boot/loader
world|base|/boot/pxeboot
world|base|/etc/mail/freebsd.cf
world|base|/etc/mail/freebsd.submit.cf
world|base|/etc/mail/sendmail.cf
world|base|/etc/mail/submit.cf
world|base|/lib/libcrypto.so.5
world|base|/usr/bin/ntpq
world|base|/usr/include/osreldate.h
world|base|/usr/lib/libalias.a
world|base|/usr/lib/libalias_cuseeme.a
world|base|/usr/lib/libalias_dummy.a
world|base|/usr/lib/libalias_ftp.a
...
Values of build stamps, excluding library archive headers:
v1.2 (Aug 26 2009 18:13:46)
v1.2 (Aug 26 2009 18:11:44)
@()FreeBSD 7.1-RELEASE-p7 0: Wed Aug 26 18:11:50 UTC 2009
FreeBSD 7.1-RELEASE-p7 0: Wed Aug 26 18:11:50 UTC 2009
    root@server.myhost.com:/usr/obj/usr/src/sys/GENERIC
7.1-RELEASE-p7
Wed Aug 26 17:29:15 UTC 2009
Wed Aug 26 17:29:15 UTC 2009
 built by root@server.myhost.com on Wed Aug 26 17:49:58 UTC 2009
 built by root@server.myhost.com on Wed Aug 26 17:49:58 UTC 2009
 built by root@server.myhost.com on Wed Aug 26 17:49:58 UTC 2009
 built by root@server.myhost.com on Wed Aug 26 17:49:58 UTC 2009
Wed Aug 26 17:20:39 UTC 2009
ntpq 4.2.4p5-a Wed Aug 26 17:29:42 UTC 2009 (1)
 * Copyright (c) 1992-2009 The FreeBSD Project.
Wed Aug 26 17:20:39 UTC 2009
Wed Aug 26 17:29:30 UTC 2009
Aug 26 2009
ntpd 4.2.4p5-a Wed Aug 26 17:29:41 UTC 2009 (1)
ntpdate 4.2.4p5-a Wed Aug 26 17:29:42 UTC 2009 (1)
ntpdc 4.2.4p5-a Wed Aug 26 17:29:42 UTC 2009 (1)
Wed Aug 26 17:55:02 UTC 2009
Wed Aug 26 17:55:02 UTC 2009
Wed Aug 26 17:55:02 UTC 2009
Wed Aug 26 17:20:39 UTC 2009
...
```

更新将打印出来，并请求批准。

```sh
New updates:
kernel|generic|/GENERIC/kernel.symbols|f|0|0|0555|0|7c8dc176763f96ced0a57fc04e7c1b8d793f27e006dd13e0b499e1474ac47e10|
kernel|generic|/GENERIC/kernel|f|0|0|0555|0|33197e8cf15bbbac263d17f39c153c9d489348c2c534f7ca1120a1183dec67b1|
kernel|generic|/|d|0|0|0755|0||
src|base|/|d|0|0|0755|0||
src|bin|/|d|0|0|0755|0||
src|cddl|/|d|0|0|0755|0||
src|contrib|/contrib/bind9/bin/named/update.c|f|0|10000|0644|0|4d434abf0983df9bc47435670d307fa882ef4b348ed8ca90928d250f42ea0757|
src|contrib|/contrib/bind9/lib/dns/openssldsa_link.c|f|0|10000|0644|0|c6805c39f3da2a06dd3f163f26c314a4692d4cd9a2d929c0acc88d736324f550|
src|contrib|/contrib/bind9/lib/dns/opensslrsa_link.c|f|0|10000|0644|0|fa0f7417ee9da42cc8d0fd96ad24e7a34125e05b5ae075bd6e3238f1c022a712|
...
FreeBSD/amd64 7.1-RELEASE update build complete.  Please review
the list of build stamps printed above and the list of updated
files to confirm that they look sensible, then run
 sh -e approve.sh amd64 7.1-RELEASE
to sign the build.
```

按照之前提到的相同流程来批准构建：

```sh
# sh -e scripts/approve.sh amd64 7.1-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Signing build for FreeBSD/amd64 7.1-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Copying files to patch source directories for FreeBSD/amd64 7.1-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Copying files to upload staging area for FreeBSD/amd64 7.1-RELEASE
Wed Aug 26 12:50:07 PDT 2009 Updating databases for FreeBSD/amd64 7.1-RELEASE
Wed Aug 26 12:50:07 PDT 2009 Cleaning staging area for FreeBSD/amd64 7.1-RELEASE

The FreeBSD/amd64 7.1-RELEASE update build has been signed and is
ready to be uploaded.  Remember to run
 sh -e umountkey.sh
to unmount the decrypted key once you have finished signing all
the new builds.
```

批准构建后，上传软件：

```sh
# cd /usr/local/freebsd-update-server
# sh scripts/upload.sh amd64 7.1-RELEASE
```

作为参考，已附上整个 [diff.sh](https://docs.freebsd.org/en/source/articles/freebsd-update-server/diff.txt) 运行过程。

## 7. 提示

- 如果使用本地 `make release` [过程](https://docs.freebsd.org/en/articles/releng/#release-build) 构建了自定义发布版本，freebsd-update-server 代码将能从你的发布版本中运行。例如，可以通过清除与文档子程序 `findextradocs ()`、`addextradocs ()` 相关的功能，并分别在 **scripts/build.subr** 中更改 `fetchiso ()` 的下载位置，来构建一个不包含 Ports 和文档的版本。最后一步，在相应的发布和架构下更改 **build.conf** 中的 [sha256(1)](https://man.freebsd.org/cgi/man.cgi?query=sha256&sektion=1&format=html) 哈希值，之后即可开始从自定义发布版本进行构建。

  ```sh
  # 比较 ${WORKDIR}/release 和 ${WORKDIR}/$1，找出缺少的世界或文档子组件，并
  # 将其打包。
  findextradocs () {
  }
  # 将额外的文档添加到 ${WORKDIR}/$1
  addextradocs () {
  }
  ```
- 在 **scripts/build.subr** 脚本中的 `buildworld` 和 `obj` 目标添加 `-j NUMBER` 标志，可以加速处理，具体取决于使用的硬件，但这不是必须的。不建议在其他目标中使用这些标志，因为它可能导致构建变得不可靠。

  ```sh
  # 构建世界
  		   log "Building world"
  		   cd /usr/src &&
  		   make -j 2 ${COMPATFLAGS} buildworld 2>&1
  		# 分发世界
  		   log "Distributing world"
  		   cd /usr/src/release &&
  		   make -j 2 obj &&
  		   make ${COMPATFLAGS} release.1 release.2 2>&1
  ```
- 为更新服务器创建适当的 [DNS](https://docs.freebsd.org/en/books/handbook/#network-dns) SRV 记录，并将其他服务器放置在其后，设置不同的权重。使用此功能可以提供更新镜像，但除非你希望提供冗余服务，否则此提示并非必需。

  ```sh
  _http._tcp.update.myserver.com.		IN SRV   0 2 80   host1.myserver.com.
  					IN SRV   0 1 80   host2.myserver.com.
  					IN SRV   0 0 80   host3.myserver.com.
  ```
