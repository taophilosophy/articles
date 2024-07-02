# 构建你自己的 FreeBSD 更新服务器

# 构建您自己的 FreeBSD 更新服务器

版权 © 2009-2011 年，2013 年 Jason Helfman

<details open="" data-immersive-translate-walked="8eb17343-6bcf-4045-87b8-91174c0afece"><summary data-immersive-translate-walked="8eb17343-6bcf-4045-87b8-91174c0afece" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

AMD，AMD Athlon，AMD Opteron，AMD Phenom，AMD Sempron，AMD Turion，Athlon，Élan，Opteron 和 PCnet 是 Advanced Micro Devices, Inc. 的商标。

Intel，Celeron，Centrino，Core，EtherExpress，i386，i486，Itanium，Pentium 和 Xeon 是 Intel Corporation 或其子公司在美国及其他国家/地区的商标或注册商标。

许多制造商和销售商用来区分其产品的命名被视为商标。在本文档中出现这些命名时，FreeBSD 项目已知商标声明的情况下，这些命名后面会标注“™”或“®”符号。

</details>

|  | 本文中的指令适用于旧版本的 FreeBSD，在新版本的操作系统上可能无法正常工作。随着 pkgbase 的可用性，freebsd-update 实用程序计划将从 FreeBSD 中移除。届时，本文将更新以反映新的过程，或完全移除。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

 摘要

本文介绍了构建内部 FreeBSD Update Server。freebsd-update-server 由 FreeBSD 的名誉安全官 Colin Percival <<a href="mailto:cperciva@FreeBSD.org">cperciva@FreeBSD.org</a>> 编写。对于那些认为通过官方更新服务器更新系统方便的用户来说，构建自己的 FreeBSD Update Server 可能有助于通过支持手动调整的 FreeBSD 版本或提供本地镜像来扩展其功能，从而更快地更新多台机器。

---

## 1. 致谢

这篇文章随后在 BSD Magazine 上发表。

## 2. 介绍

经验丰富的用户或管理员经常负责多台机器或环境。他们了解维护这种基础架构所面临的困难需求和挑战。运行 FreeBSD Update Server 可以更轻松地将安全和软件补丁部署到选定的测试机器，然后再将其推向生产环境。这也意味着可以从本地网络而不是潜在较慢的互联网连接更新多个系统。本文概述了创建内部 FreeBSD Update Server 所涉及的步骤。

## 先决条件

要构建内部 FreeBSD Update Server，必须满足一些要求。

* 一个正在运行的 FreeBSD 系统。||||至少需要在大于或等于目标发行版本的 FreeBSD 版本上进行构建来进行更新。|
  | --| -----------------------------------------------------------------------|
* 一个至少有 4GB 可用空间的用户账户。这将允许创建 7.1 和 7.2 版本的更新，但确切的空间要求可能会因版本而异。
* 在远程机器上具有 ssh(1)账户，用于上传分布式更新。
* 像 Apache 这样的 Web 服务器，其所需的构建空间超过一半。例如，7.1 和 7.2 的测试构建总共消耗了 4 GB，用于分发这些更新的 Web 服务器空间需求为 2.6 GB。
* 掌握使用 Bourne Shell（sh(1)）进行基本的shell脚本编写知识。

## 4. 配置：安装与设置

安装 devel/git 和 security/ca_root_nss 软件包以下载 freebsd-update-server 软件，并执行：

```
% git clone https://github.com/freebsd/freebsd-update-build.git freebsd-update-server
```

更新 scripts/build.conf 配置文件。它在所有构建操作期间被引用。

这里是默认的 build.conf，需要根据您的环境进行修改。

```
# Main configuration file for FreeBSD Update builds.  The
# release-specific configuration data is lower down in
# the scripts tree.

# Location from which to fetch releases
export FTP=ftp://ftp2.freebsd.org/pub/FreeBSD/releases 

# Host platform
export HOSTPLATFORM=`uname -m`

# Host name to use inside jails
export BUILDHOSTNAME=${HOSTPLATFORM}-builder.daemonology.net 

# Location of SSH key
export SSHKEY=/root/.ssh/id_dsa 

# SSH account into which files are uploaded
MASTERACCT=builder@wadham.daemonology.net 

# Directory into which files are uploaded
MASTERDIR=update-master.freebsd.org 
```

考虑的参数如下：

|  | 这是 ISO 镜像下载的位置（由 scripts/build.subr 的 fetchiso() 子程序下载）。配置的位置不限于 FTP URI。任何标准 fetch(1) 实用工具支持的 URI 方案都应该正常工作。可以通过将默认的 build.subr 脚本复制到 release 和特定架构区域的 scripts/RELEASE/ARCHITECTURE/build.subr，并应用本地更改来安装对 fetchiso() 代码的定制。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  | 构建主机的名称。此信息将在更新的系统上执行以下操作时显示：<br /><br />`% uname -v`                                                                                                                                                                                                                                                        |
|  | 用于向更新服务器上传文件的 SSH 密钥。可以通过输入 ssh-keygen -t dsa 创建密钥对。当未定义 SSHKEY 时，将使用标准密码身份验证作为备用身份验证方法。有关 SSH 的更详细信息和创建及使用 SSH 密钥的适当步骤，请参阅 ssh-keygen(1) 手册页。                                                                                   |
|  | 用于向更新服务器上传文件的帐户。                                                                                                                                                                                                                                                                                      |
|  | 更新服务器上用于上传文件的目录。                                                                                                                                                                                                                                                                                      |

FreeBSD-update-server 源代码中附带的默认 build.conf 适用于构建 FreeBSD 的 i386 版本。作为构建其他架构更新服务器的示例，以下步骤概述了配置 amd64 所需的配置更改：

1. 为 amd64 创建一个构建环境：

    ```
    % mkdir -p /usr/local/freebsd-update-server/scripts/7.2-RELEASE/amd64
    ```
2. 将构建配置文件.conf 安装到新创建的构建目录中。在 amd64 上，FreeBSD 7.2-RELEASE 的构建配置选项应该是类似的：

    ```
    # SHA256 hash of RELEASE disc1.iso image.
    export RELH=1ea1f6f652d7c5f5eab7ef9f8edbed50cb664b08ed761850f95f48e86cc71ef5 
    # Components of the world, source, and kernels
    export WORLDPARTS="base catpages dict doc games info manpages proflibs lib32"
    export SOURCEPARTS="base bin contrib crypto etc games gnu include krb5  \
                    lib libexec release rescue sbin secure share sys tools  \
                    ubin usbin cddl"
    export KERNELPARTS="generic"

    # EOL date
    export EOL=1275289200 
    ```

    |  | 所需版本的 sha256(1)哈希键在各自的发布公告中发布。                                                                                                             |
    | -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    |  | 要生成 build.conf 的“终身期限”数字，请参考 FreeBSD 安全网站上发布的“预计终身期限”。 EOL 的值可以通过网站上列出的日期使用 date(1)实用程序来推导，例如：<br /><br />`% date -j -f '%Y%m%d-%H%M%S' '20090401-000000' '+%s'` |

## 构建更新代码

第一步是运行 scripts/make.sh。这将构建一些二进制文件，创建目录，并生成用于批准构建的 RSA 签名密钥。在此步骤中，需要提供一个密码来最终创建签名密钥。

```
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

|  | 记下生成的密钥指纹。此值在 /etc/freebsd-update.conf 中用于二进制更新。 |
| -- | ------------------------------------------------------------------------ |

到了这一步，我们已经准备好进行构建。

```
# cd /usr/local/freebsd-update-server
# sh scripts/init.sh amd64 7.2-RELEASE
```

接下来是初始构建运行的示例。

```
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

然后再次执行世界的构建，使用世界补丁。更详细的解释可以在 scripts/build.subr 中找到。

|  | 在这第二个构建周期期间，网络时间协议守护程序 ntpd(8)已关闭。根据 FreeBSD 的安全主任 Emeritus ，"freebsd-update-server 的构建代码需要识别存储在文件中的时间戳，以便在比较构建以确定哪些文件需要更新时可以忽略它们。这种时间戳查找的工作方式是间隔 400 天进行两次构建，然后比较结果。" |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

```
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

```
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

如果一切正确，请批准构建。有关如何确定这一点的更多信息，请参阅名为 USAGE 的分发源文件。按照指示执行 scripts/approve.sh。这将签署发布，并将组件移动到适合上传的暂存区。

```
# cd /usr/local/freebsd-update-server
# sh scripts/mountkey.sh
```

```
# sh -e scripts/approve.sh amd64 7.2-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Signing build for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Copying files to patch source directories for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:06 PDT 2009 Copying files to upload staging area for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:07 PDT 2009 Updating databases for FreeBSD/amd64 7.2-RELEASE
Wed Aug 26 12:50:07 PDT 2009 Cleaning staging area for FreeBSD/amd64 7.2-RELEASE
```

在审批过程完成后，可以开始上传过程。

```
# cd /usr/local/freebsd-update-server
# sh scripts/upload.sh amd64 7.2-RELEASE
```

```
# cd /usr/local/freebsd-update-server/pub/7.2-RELEASE/amd64
# touch -t 200801010101.01 uploaded
```

上传的文件需要放在网页服务器的文档根目录中，才能进行更新分发。具体配置会根据所使用的网页服务器而有所不同。对于 Apache 网页服务器，请参考手册中的 Apache 服务器配置部分。

更新客户端的 KeyPrint 和 ServerName 在 /etc/freebsd-update.conf，并按照手册中 FreeBSD 更新部分的说明进行更新。

|  | 为了使 FreeBSD Update Server 正常工作，需要构建当前版本和想要升级到的版本的更新内容。这对于确定不同版本之间文件差异是必要的。例如，当将 FreeBSD 系统从 7.1-RELEASE 升级到 7.2-RELEASE 时，需要构建并上传两个版本的更新内容到您的分发服务器。 |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

供参考，init.sh 的整个运行附在此处。

## 6. 构建补丁

每当发布安全咨询或安全通知时，可以构建补丁更新。

例如，将使用 7.1-RELEASE。

对于不同版本构建，做出了一些假设。

* 设置初始构建的正确目录结构。
* 为 7.1-RELEASE 执行初始构建。

在/usr/local/freebsd-update-server/patches/下为相应版本的发布创建补丁目录。

```
% mkdir -p /usr/local/freebsd-update-server/patches/7.1-RELEASE/
% cd /usr/local/freebsd-update-server/patches/7.1-RELEASE
```

例如，以 named(8)的补丁为例。阅读公告，并从 FreeBSD 安全公告中获取所需文件。关于如何解释公告的更多信息，请参阅 FreeBSD 手册。

在安全摘要中，此公告称为 SA-09:12.bind 。下载文件后，需要将文件重命名为适当的补丁级别。建议与官方 FreeBSD 补丁级别保持一致，但名称可以自由选择。对于此构建，请遵循当前已建立的 FreeBSD 惯例，称其为 p7 。重命名文件：

```
% cd /usr/local/freebsd-update-server/patches/7.1-RELEASE/; mv bind.patch 7-SA-09:12.bind
```

|  | 运行补丁级别构建时，假定已存在先前的补丁。当运行补丁构建时，它将运行补丁目录中包含的所有补丁。<br /><br />可以向任何构建添加定制补丁。使用零号或任何其他数字。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  | 由 FreeBSD 更新服务器管理员采取适当措施验证每个补丁的真实性。 |
| -- | --------------------------------------------------------------- |

此时，diff 已准备好进行构建。软件在运行 diff 构建之前首先检查是否在相应的版本发布前运行了 scripts/init.sh。

```
# cd /usr/local/freebsd-update-server
# sh scripts/diff.sh amd64 7.1-RELEASE 7
```

以下是差异生成运行的示例。

```
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

更新已打印，并请求批准。

```
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

对于批准生成，遵循之前指定的相同过程：

```
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

```
# cd /usr/local/freebsd-update-server
# sh scripts/upload.sh amd64 7.1-RELEASE
```

供参考，diff.sh 的完整运行已附上。

## 7. 提示

* 如果使用本地 make release 过程构建定制发行版，freebsd-update-server 代码将适用于您的发行版。举例来说，可以通过清除与文档子程序 findextradocs () 、 addextradocs () 相关的功能，并在 scripts/build.subr 中分别更改下载位置来构建没有 ports 或文档的发行版。最后一步，修改 build.conf 中相应发行版和架构下的 sha256(1) 哈希值，即可基于您的定制发行版进行构建。

  ```
  # Compare ${WORKDIR}/release and ${WORKDIR}/$1, identify which parts
  # of the world|doc subcomponent are missing from the latter, and
  # build a tarball out of them.
  findextradocs () {
  }
  # Add extra docs to ${WORKDIR}/$1
  addextradocs () {
  }
  ```
* 在 scripts/build.subr 脚本中向 buildworld 和 obj 目标添加 -j<span> </span><em>NUMBER</em> 标志可能会根据使用的硬件加快处理速度，但这不是必须的。不建议在其他目标中使用这些标志，因为这可能导致构建变得不可靠。

  ```
                # Build the world
  		   log "Building world"
  		   cd /usr/src &&
  		   make -j 2 ${COMPATFLAGS} buildworld 2>&1
  		# Distribute the world
  		   log "Distributing world"
  		   cd /usr/src/release &&
  		   make -j 2 obj &&
  		   make ${COMPATFLAGS} release.1 release.2 2>&1
  ```
* 为更新服务器创建适当的 DNS SRV 记录，并将其他服务器放在其后面，设置不同的权重。使用此功能将提供更新镜像，但除非您希望提供冗余服务，否则此技巧不是必须的。

  ```
  _http._tcp.update.myserver.com.		IN SRV   0 2 80   host1.myserver.com.
  					IN SRV   0 1 80   host2.myserver.com.
  					IN SRV   0 0 80   host3.myserver.com.
  ```
