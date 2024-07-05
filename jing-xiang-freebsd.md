# 镜像 FreeBSD

<details open="" data-immersive-translate-walked="30b1f719-3771-4cc1-bdc6-73a87c82bab0"><summary data-immersive-translate-walked="30b1f719-3771-4cc1-bdc6-73a87c82bab0" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

许多制造商和销售商用来区分其产品的名称都被声称为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目知道该商标声明，这些名称后面会附有“™”或“®”符号。

</details>

 摘要

一篇正在进行中的文章，介绍如何镜像 FreeBSD，面向中心管理员。

---

|  | 我们暂时不接受新的社区镜像。 |
| -- | ------------------------------ |

## 联系信息

镜像系统协调员可以通过邮件 mirror-admin@FreeBSD.org 联系。还有一个 FreeBSD 镜像站邮件列表。

## FreeBSD 镜像的要求

### 2.1. 磁盘空间

磁盘空间是最重要的要求之一。根据要镜像的版本、架构和完整程度的不同，可能需要消耗大量的磁盘空间。还要注意，官方镜像可能需要完整镜像。网页应始终完全镜像。还要注意，这里提到的数字反映了当前状态（在 12.0-RELEASE/11.3-RELEASE）。进一步的开发和发布只会增加所需的空间。此外，务必留出一些（大约 10-20%）额外的空间以确保安全。以下是一些大约的数字：

* 完整的 FTP 分发：1.4 TB
* CTM 变化：10 GB
* 网页：1GB

FTP 发布的当前磁盘使用情况可以在 ftp://ftp.FreeBSD.org/pub/FreeBSD/dir.sizes 找到。

### 2.2. 网络连接/带宽

当然，您需要连接到互联网。所需的带宽取决于您使用镜像的目的。如果您只是想在您的站点/局域网上镜像 FreeBSD 的一些部分，需求可能远低于如果您想公开提供文件。如果您打算成为官方镜像站点，所需的带宽将更高。我们只能提供粗略的估计：

* 本地站点，无公共访问：基本上没有最低要求，但是 < 2 Mbps 的带宽可能会使同步过程过慢。
* 非官方公共网站：34 Mbps 可能是一个不错的起点。
* 官方网站：建议使用 >100 Mbps，并且您的主机应尽可能靠近您的边界路由器连接。

### 2.3. 系统要求，CPU，内存

这取决于预期的客户数量，这由服务器的策略确定。它还受您想要提供的服务类型的影响。普通的 FTP 或 HTTP 服务可能不需要大量资源。但如果提供 rsync，请注意它可能对 CPU 和内存需求产生巨大影响，因为它被认为是内存消耗大的程序。以下仅是示例，仅供参考。

对于一个访问量适中并提供 rsync 的网站，您可能需要考虑当前至少 800MHz - 1 GHz 的 CPU，至少 512MB 的 RAM。这可能是您希望为官方网站提供的最低配置。

对于一个经常使用的网站，您绝对需要更多的 RAM（考虑起步为 2GB 是个不错的选择），可能需要更多的 CPU，这也可能意味着您需要选择 SMP 系统。

You also want to consider a fast disk subsystem. Operations on the SVN repository require a fast disk subsystem (RAID is highly advised). A SCSI controller that has a cache of its own can also speed up things since most of these services incur a large number of small modifications to the disk.

### 2.4. Services to Offer

您 text: 您还还应应考考考虑虑虑快速的速的的磁磁磁盘子盘子系统子系统。。。对对对 SVN 仓库的库库的操作的操作需要作需要一个要一个快一个快速的快速的磁速的磁盘的磁盘子磁盘子系统子系统（建统（强使用 RAID 用 RAID）。议 RAID）。带有 ID）。带有自）。具。具有自具有自己有自己缓自己缓存 CSISI 控 CSISI 控 I 控制 控制器制器也器也可以也可以加以加快加快速快速度速度，度，因，因为务大多数大多数会多数会对中大部大部分部分会行会对大量小量小修改。 -小修改。

#### 2.4.1. FTP (required for FTP Fileset)

这是其中最基本的服务之一，每个提供公共 FTP 分发的镜像都需要它。FTP 访问必须是匿名的，不允许上传/下载比例（这本身就是荒谬的事情）。不需要上传功能（绝不能允许在 FreeBSD 文件空间上进行上传）。此外，FreeBSD 存档应在路径/pub/FreeBSD 下可用。

有许多可用的软件可以设置为允许匿名 FTP（按字母顺序排列）。

* /usr/libexec/ftpd ：FreeBSD 自带的 ftpd 可以使用。请确保阅读 ftpd(8)。
* ftp/ncftpd：一款商业软件包，供教育用途免费。
* ftp/oftpd：一个以安全性为主要关注点设计的 ftpd。
* ftp/proftpd: 一个模块化且非常灵活的 ftpd。
* ftp/pure-ftpd: 另一个注重安全性的 ftpd。
* ftp/twoftpd: 如上所述。
* ftp/vsftpd: "非常安全"的 ftpd。

FreeBSD 的 ftpd ， proftpd ，也许 ncftpd 是最常用的 FTPd 之一。其他的在镜像站点中没有很大的用户群。需要考虑的一点是，您可能需要灵活地限制允许的同时连接数量，从而限制网络带宽和系统资源的消耗。

#### 2.4.2. Rsync（FTP 文件集的可选部分）

Rsync 经常用于访问 FreeBSD 的 FTP 区域的内容，这样其他镜像站点可以使用您的系统作为它们的源。该协议在许多方面与 FTP 不同。它更节省带宽，因为仅在文件更改时传输文件差异，而不是整个文件。对于每个实例，Rsync 确实需要大量内存。大小取决于同步模块的大小，即目录和文件的数量。Rsync 可以使用 rsh 和 ssh （现在默认）作为传输，或者使用其自己的协议进行独立访问（这是公共 rsync 服务器的首选方法）。可以应用身份验证、连接限制和其他限制。只有一个软件包可用：

* [net/rsync](https://cgit.freebsd.org/ports/tree/net/rsync/)

#### 2.4.3. HTTP（Web 页面所需，FTP 文件集可选）

如果您想提供 FreeBSD 网页，您需要安装一个 Web 服务器。您可以选择通过 HTTP 提供 FTP 文件集。Web 服务器软件的选择由镜像管理员决定。一些最流行的选择包括：

* www/apache24: Apache 仍然是互联网上部署最广泛的 Web 服务器之一。FreeBSD 项目广泛使用它。
* www/boa: Boa 是一个单任务的 HTTP 服务器。与传统的 Web 服务器不同，它不会为每个传入连接复制进程，也不会复制多个副本来处理多个连接。尽管如此，它应该为纯静态内容提供相当好的性能。
* www/cherokee: Cherokee 是一个非常快速、灵活且易于配置的 Web 服务器。它支持当今广泛使用的技术：FastCGI、SCGI、PHP、CGI、SSL/TLS 加密连接、虚拟主机、用户认证、实时编码和负载均衡。它还生成与 Apache 兼容的日志文件。
* www/lighttpd: lighttpd 是一个安全、快速、兼容且非常灵活的 Web 服务器，专为高性能环境进行了优化。与其他 Web 服务器相比，它的内存占用非常低，并且处理 CPU 负载。
* www/nginx: nginx 是一个高性能边缘 Web 服务器，具有低内存占用和构建现代高效 Web 基础设施的关键特性。功能包括 HTTP 服务器、HTTP 和邮件反向代理、缓存、负载均衡、压缩、请求限速、连接复用和重用、SSL 卸载和 HTTP 媒体流。
* www/thttpd: 如果您将要提供大量静态内容的服务，您可能会发现使用 thttpd 这样的应用比其他应用效率更高。它还针对 FreeBSD 进行了优化，性能出色。

## 3. 如何镜像 FreeBSD

好的，现在您知道了提供服务的要求以及如何提供服务，但不知道如何获取它。:-) 本节将解释如何实际镜像 FreeBSD 的各个部分，使用哪些工具，以及从哪里镜像。

### 3.1. 复制 FTP 站点

FTP 区域是需要镜像的数据量最大的地方。它包括用于网络安装的分发集，实际上是检出源树的快照分支，用于写入安装分发的 ISO 镜像，一个实时文件系统，以及 ports 树的快照。当然，这些都是针对各种 FreeBSD 版本和各种架构。

镜像 FTP 区域的最佳方式是使用 rsync。您可以安装 net/rsync 然后使用 rsync 与上游主机进行同步。rsync 已经在 Rsync（用于 FTP 文件集的可选项）中提到过。由于不需要 rsync 访问权限，您的首选上游站点可能不允许它。您可能需要四处寻找一下，找到一个允许 rsync 访问的站点。

|  | 由于 rsync 客户端的数量对服务器的影响很大，大多数管理员会对他们的服务器施加限制。对于镜像站点，您应该向正在同步的站点维护者询问其策略，可能还可以为您的主机请求例外（因为您是镜像站点）。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

同步 FreeBSD 的命令行可能如下所示：

```
% rsync -vaHz --delete rsync://ftp4.de.FreeBSD.org/FreeBSD/ /pub/FreeBSD/
```

请参考 rsync 的文档，也可在 http://rsync.samba.org/找到，了解使用 rsync 的各种选项。如果您同步整个模块（而不是子目录），请注意模块目录（此处为"FreeBSD"）不会被创建，因此不能省略目标目录。您可能还希望设置一个脚本框架，通过 cron(8)调用这样的命令。

### 3.2. 镜像 WWW 页面

|  | 自 2021-01-25 起，文档迁移到 Hugo/Asciidoctor 后，使用 rsync 镜像网站已不再有效。 |
| -- | ----------------------------------------------------------------------------------- |

正在进行研究以利用官方基础设施实现网站镜像。

对于以前的网站镜像，实现今天的网站镜像的方法是在本地构建网站，并使用相应的地址进行托管。

```
% cd website && env HUGO_baseURL="https://www.XX.freebsd.org/" make
```

在 FreeBSD 文档项目新贡献者入门手册中查找有关构建工具的更多详细信息。

|  | 注意网站已分为 www.FreeBSD.org 和 docs.FreeBSD.org，并且它们之间存在链接；此外，在此时， HUGO_baseURL 变量不会涵盖所有链接，因此不鼓励镜像网站。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------- |

### 3.3. 包镜像

由于带宽、存储和管理要求非常高，FreeBSD 项目决定不允许公开镜像包。对于有大量计算机的站点，运行一个用于 pkg(8)进程的缓存 HTTP 代理可能是有利的。或者可以通过运行类似以下内容来获取特定软件包及其依赖项：

```
% pkg fetch -d -o /usr/local/mirror vim
```

一旦这些软件包被获取，必须通过运行以下命令生成存储库元数据：

```
% pkg repo /usr/local/mirror
```

一旦软件包已获取并生成了存储库的元数据，通过 HTTP 向客户机提供这些软件包。有关更多信息，请参阅 pkg(8)的 man 手册，特别是 pkg-repo(8)页面。

### 3.4. 我应该多久同步一次镜像？

每个镜像至少应每天更新一次。肯定需要一个带锁定功能的脚本来防止同时运行多个实例，并通过 cron(8)定期运行。由于几乎每位管理员都有自己的方式，因此无法提供具体的说明。可能的工作流程如下所示：

1. 将运行镜像应用程序的命令放入脚本中。建议使用普通脚本。
2. 添加一些输出重定向，以便将诊断消息记录到文件中。
3. 测试脚本是否有效。检查日志。
4. 使用 crontab(1) 将脚本添加到适当用户的 crontab(5) 中。这应该是一个不同的用户，而不是您的 FTP 守护程序运行的用户，以便如果 FTP 区域内的文件权限不是全局可读的话，这些文件就不能被匿名 FTP 访问。这用于“分阶段”发布 - 确保所有官方镜像站在发布日拥有所有必要的发布文件。

这里是一些推荐的计划：

* FTP 文件集：每天
* WWW 页面: 每日

## 4. 从哪里镜像

这是一个重要问题。因此，本节将花费一些精力来解释背景。我们将多次强调：在任何情况下都不应该从 ftp.FreeBSD.org 镜像。

### 4.1. 关于组织的几句话

镜像按国家组织。所有官方镜像都具有形式 ftpN.CC.FreeBSD.org 的 DNS 条目。CC（即，国家代码）是镜像所在国家的顶级域（TLD）。N 是一个数字，表示主机将是该国家第 N 个镜像。（ wwwN.CC.FreeBSD.org 等同样适用）有些镜像没有 CC 部分。这些是非常良好连接并允许大量并发用户的镜像站点。 ftp.FreeBSD.org 实际上是两台机器，一台目前位于丹麦，另一台位于美国。这不是一个主站点，也不应该用来进行镜像。大量在线文档引导“交互”用户到 ftp.FreeBSD.org ，所以自动镜像系统应该从不同的机器进行镜像。

此外，还存在一种镜像层次结构，用层级来描述。主站点没有被提及，但可以被描述为 Tier-0。从这些站点镜像的镜像可以被视为 Tier-1，Tier-1 的镜像，是 Tier-2，等等。鼓励官方站点处于较低的层级，但层级越低，对 Requirements for FreeBSD Mirrors 中描述的要求越高。访问低层镜像可能会受到限制，对主站点的访问肯定会受到限制。层次结构并不由 DNS 反映，一般也没有在任何地方记录，除了主站点。然而，像 1-4 这样低数字的官方镜像通常是 Tier-1（这只是一个大致的提示，没有规则）。

### 4.2. Ok, but Where Should I get the Stuff Now?

绝不应从镜像 ftp.FreeBSD.org 下载。简短回答是：从对你来说最接近的互联网站点，或者提供最快访问速度的站点下载。

#### 4.2.1. 我只是想从某个地方镜像！

如果您没有特殊意图或需求，则 Ok，但现在我应该从哪里获取这些东西？这意味着：

1. 检查那些提供最快访问（跳数、往返时间）并提供您打算使用的服务（如 rsync）的选项。
2. 联系您选择站点的管理员，说明您的请求，并询问他们的条款和政策。
3. 按照上面描述的设置你的镜像。

#### 4.2.2. 我是官方镜像，哪个站点适合我？

一般来说，我只想从某个地方镜像的描述仍然适用。当然，你可能希望注意到你的上游应该是较低的层级。关于官方镜像还有一些其他的考虑，在官方镜像中有描述。

#### 4.2.3. 我想要访问主站点！

如果您有充分的理由和条件，您可能希望并获得访问其中一个主站点的权限。这些站点通常受限制，访问有特殊的政策。如果您已经是官方镜像站点，这无疑有助于您获得访问权限。在任何其他情况下，请确保您的国家确实需要另一个镜像站点。如果已有三个或更多，请首先询问“区域管理员”（hostmaster@CC.FreeBSD.org）或 FreeBSD 镜像站点邮件列表。

任何帮助您成为官方镜像站点的人，都应该帮助您获得适当的上游主机访问权限，无论是其中一个主站点还是适当的一级站点。如果没有，请发送电子邮件至 mirror-admin@FreeBSD.org 请求帮助。

FTP 文件集的主站点只有一个。

##### 4.2.3.1. [ftp-master.FreeBSD.org](http://ftp-master.FreeBSD.org)

这是 FTP 文件集的主站点。

ftp-master.FreeBSD.org 提供了 rsync 访问，除了 FTP。请参阅镜像 FTP 站点。

为了支持 FTP 内容，也鼓励镜像提供 rsync 访问，因为它们是一级镜像。

## 5. 官方镜像

官方镜像是指

* a) 具有 FreeBSD.org DNS 条目（通常是 CNAME）。
* b) 在 FreeBSD 文档（如手册）中被列为官方镜像。

到目前为止,区分官方镜像。官方镜像不一定是 Tier-1 镜像。然而,您可能找不到 Tier-1 镜像不是官方镜像。

### 5.1. 官方（一级）镜像的特殊要求

对于所有官方镜像进行要求并不是那么容易,因为这里的项目有一定的包容性。更容易说,官方一级镜像需要满足什么要求。所有其他官方镜像可以认为这是一个很重要的应该。

Tier-1 mirrors are required to:

* carry the complete fileset
* allow access to other mirror sites
* 提供 FTP 和 rsync 访问

此外，管理员应订阅 FreeBSD 镜像站点的邮件列表。有关详细信息和订阅方法，请参阅此链接。

|  | 对于中心管理员，特别是一级中心管理员，检查下一个 FreeBSD 发布版本的发布计划非常重要。这很重要，因为它将告诉您下一个版本预计何时发布，从而为随之而来的高峰流量做好准备提供充足时间。<br /><br />保持镜像尽可能最新对于中心管理员来说也很重要（对于一级镜像而言尤为关键）。如果 Mirror1 镜像有一段时间没有更新，低级别镜像将开始复制 Mirror1 的旧数据，从而开始恶性循环… 请保持镜像的更新！ |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

### 5.2. 如何成为官方呢？

请按照 https://www.FreeBSD.org/administration/#t-clusteradm 上记录的集群管理员联系。

## 6. 一些镜像站点的统计数据

这里是您喜爱镜像站点的统计页面链接（也就是唯一愿意提供统计数据的站点）。

### 6.1. FTP 站点统计

* ftp.is.FreeBSD.org - hostmaster@is.FreeBSD.org - (带宽) (FTP 进程) (HTTP 进程)
* ftp2.ru.FreeBSD.org - mirror@macomnet.ru - (带宽) (HTTP 和 FTP 用户)
