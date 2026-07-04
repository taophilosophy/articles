# 镜像 FreeBSD

- 原文：[Mirroring FreeBSD](https://docs.freebsd.org/en/articles/hubs/)

## 摘要

这篇正在进行中的文章介绍了如何镜像 FreeBSD，旨在为集群管理员提供帮助。

>**注意**
>
> 我们现在不接受新的社区镜像。

## 1. 联系信息

镜像系统协调员可通过电子邮件与我们联系：[mirror-admin@FreeBSD.org](mailto:mirror-admin@FreeBSD.org)。也可以通过 [FreeBSD 镜像站点邮件列表](https://lists.freebsd.org/subscription/freebsd-hubs)联系。

## 2. FreeBSD 镜像要求

### 2.1. 磁盘空间

磁盘空间是最重要的要求之一。根据你希望镜像的发布版本、架构和完整性要求，可能会消耗大量的磁盘空间。还要记住，*官方*镜像可能要求必须是完整的。网页应该始终被完整地镜像。请注意，这里列出的数字反映了当前的状态（在 12.0-RELEASE/11.3-RELEASE 时）。进一步的开发和发布只会增加所需的空间量。同时，确保预留一些（大约 10-20%）额外空间，以防万一。以下是一些近似数字：

- 完整的 FTP 发行版：1.4 TB
- CTM 增量：10 GB
- 网页：1 GB

可以通过 [ftp://ftp.FreeBSD.org/pub/FreeBSD/dir.sizes](ftp://ftp.freebsd.org/pub/FreeBSD/dir.sizes) 查看当前 FTP 发行版的磁盘使用情况。

### 2.2. 网络连接/带宽

当然，你需要连接到互联网。所需的带宽取决于你计划如何使用镜像。如果你只是想为本地站点/内网镜像 FreeBSD 的某些部分，需求可能会比公开提供文件时要小得多。如果你打算成为官方镜像，所需的带宽将更高。我们只能在这里给出粗略的估算：

- 本地站点，无公开访问：基本上没有最低要求，但带宽小于 2 Mbps 可能会导致同步太慢。
- 非官方公共站点：大约 34 Mbps 是一个不错的起点。
- 官方站点：推荐带宽大于 100 Mbps，并且你的主机应尽可能靠近边界路由器连接。

### 2.3. 系统要求，CPU，RAM

这取决于预计的客户端数量，这由服务器的政策决定。它还受你想要提供的服务类型的影响。提供普通的 FTP 或 HTTP 服务可能不需要大量的资源。如果你提供 rsync，这会对 CPU 和内存的需求产生巨大影响，因为 rsync 被认为是个内存消耗大户。以下只是一些示例，给出一个大概的提示。

对于适度访问的站点，提供 rsync，你可能需要一颗约 800 MHz 到 1 GHz 的当前 CPU，以及至少 512 MB 的内存。这大概是你为 *官方* 站点所需要的最低配置。

对于频繁访问的站点，你肯定需要更多的内存（考虑 2 GB 作为一个不错的起点）和可能更多的 CPU，这也可能意味着你需要一个 SMP 系统。

你还需要考虑一个快速的磁盘子系统。SVN 仓库的操作需要一个快速的磁盘子系统（强烈建议使用 RAID）。具有自己缓存的 SCSI 控制器也能加速操作，因为这些服务大多会对磁盘进行大量的小修改。

### 2.4. 提供的服务

每个镜像站点必须提供一组核心服务。除了这些必需的服务外，服务器管理员还可以选择提供其他可选服务。本节将解释你可以提供哪些服务，以及如何实施它们。

#### 2.4.1. FTP（FTP 文件集必需）

这是最基本的服务之一，每个提供公共 FTP 发行版的镜像都必须提供此服务。FTP 访问必须是匿名的，并且不允许上传/下载比率（这本来就是个荒谬的要求）。不需要上传权限（并且 *绝不*允许对 FreeBSD 文件空间进行上传）。FreeBSD 存档应该位于路径 **/pub/FreeBSD** 下。

有很多软件可以用来设置匿名 FTP（按字母顺序排列）：

- `/usr/libexec/ftpd`：FreeBSD 自带的 ftpd 可以使用。请务必阅读 [ftpd(8)](https://man.freebsd.org/cgi/man.cgi?query=ftpd&sektion=8&format=html)。
- [ftp/ncftpd](https://cgit.freebsd.org/ports/tree/ftp/ncftpd/)：一款商业软件，供教育使用免费。
- [ftp/oftpd](https://cgit.freebsd.org/ports/tree/ftp/oftpd/)：一款注重安全的 ftpd。
- [ftp/proftpd](https://cgit.freebsd.org/ports/tree/ftp/proftpd/)：一款模块化且非常灵活的 ftpd。
- [ftp/pure-ftpd](https://cgit.freebsd.org/ports/tree/ftp/pure-ftpd/)：另一款注重安全的 ftpd。
- [ftp/twoftpd](https://cgit.freebsd.org/ports/tree/ftp/twoftpd/)：同上。
- [ftp/vsftpd](https://cgit.freebsd.org/ports/tree/ftp/vsftpd/)：一款“非常安全”的 ftpd。

FreeBSD 的 `ftpd`、`proftpd` 以及可能的 `ncftpd` 是最常用的 FTP 服务器。其他的在镜像站点中用户不多。需要考虑的一点是，你可能需要灵活限制允许的同时连接数，从而限制消耗的网络带宽和系统资源。

#### 2.4.2. Rsync（FTP 文件集的可选服务）

Rsync 通常用于访问 FreeBSD FTP 区域的内容，这样其他镜像站点可以使用你的系统作为源。该协议与 FTP 在许多方面不同。它更加节省带宽，因为仅在文件发生变化时传输文件的差异，而不是传输整个文件。Rsync 对每个实例要求大量的内存。内存大小取决于同步模块的大小（即目录和文件的数量）。Rsync 可以使用 `rsh` 和 `ssh`（现在是默认的）作为传输方式，或者使用其自己的协议进行独立访问（这是公共 rsync 服务器的首选方法）。可以应用认证、连接限制和其他限制。只有一个软件包可供使用：

- [net/rsync](https://cgit.freebsd.org/ports/tree/net/rsync/)

#### 2.4.3. HTTP（网页必需，FTP 文件集的可选服务）

如果你希望提供 FreeBSD 网页，你需要安装一台 Web 服务器。你也可以选择通过 HTTP 提供 FTP 文件集。Web 服务器软件的选择由镜像管理员决定。一些最流行的选择包括：

- [www/apache24](https://cgit.freebsd.org/ports/tree/www/apache24/)：Apache 仍然是互联网中最广泛部署的 Web 服务器之一。FreeBSD 项目广泛使用它。
- [www/boa](https://cgit.freebsd.org/ports/tree/www/boa/)：Boa 是一款单任务 HTTP 服务器。与传统 Web 服务器不同，它不会为每个传入连接创建新的进程，也不会为处理多个连接创建多个进程。尽管如此，它对于纯静态内容提供了相当好的性能。
- [www/cherokee](https://cgit.freebsd.org/ports/tree/www/cherokee/)：Cherokee 是一款非常快速、灵活且易于配置的 Web 服务器。它支持当前广泛使用的技术：FastCGI、SCGI、PHP、CGI、SSL/TLS 加密连接、虚拟主机、用户认证、动态编码和负载均衡。它还生成与 Apache 兼容的日志文件。
- [www/lighttpd](https://cgit.freebsd.org/ports/tree/www/lighttpd/)：lighttpd 是一款安全、快速、符合规范并且非常灵活的 Web 服务器，已针对高性能环境进行了优化。与其他 Web 服务器相比，它具有非常低的内存占用，并注重 CPU 负载。
- [www/nginx](https://cgit.freebsd.org/ports/tree/www/nginx/)：nginx 是一款高性能的边缘 Web 服务器，具有低内存占用和构建现代高效 Web 基础设施所需的关键特性。包括 HTTP 服务器、HTTP 和邮件反向代理、缓存、负载均衡、压缩、请求限速、连接复用和重用、SSL 卸载以及 HTTP 媒体流。
- [www/thttpd](https://cgit.freebsd.org/ports/tree/www/thttpd/)：如果你将提供大量静态内容，你可能会发现使用 thttpd 这样的应用程序比其他服务器更高效。它还针对在 FreeBSD 上的卓越性能进行了优化。

## 3. 如何镜像 FreeBSD

现在你已经了解了要求以及如何提供服务，但还不知道如何获取这些内容。:-) 本节将解释如何实际镜像 FreeBSD 的各个部分，使用什么工具，以及从哪里镜像。

### 3.1. 镜像 FTP 站点

FTP 区域是需要镜像的最大数据量。它包括用于网络安装的 *分发集*、实际检出的源树快照的 *分支*、用于将安装分发写入 CD-ROM 的 *ISO 镜像*、一个实时文件系统以及 Ports 树的快照。所有这些内容涵盖不同版本的 FreeBSD，以及不同架构。

镜像 FTP 区域的最佳方式是使用 rsync。你可以安装 Port [net/rsync](https://cgit.freebsd.org/ports/tree/net/rsync/) 并使用 rsync 与上游主机进行同步。rsync 在 [Rsync（FTP 文件集的可选服务）](https://docs.freebsd.org/en/articles/hubs/#mirror-serv-rsync) 中介绍过。由于 rsync 访问不是必须的，你的首选上游站点可能不允许它。你可能需要稍微搜索一下，找一个允许 rsync 访问的站点。

>**注意**
>
>由于 rsync 客户端数量对服务器机器有显著影响，大多数管理员会对其服务器施加限制。对于镜像站，你应该向你同步的站点管理员询问他们的政策，并可能为你的主机申请一个例外（因为你是镜像站）。

镜像 FreeBSD 的命令行可能如下所示：

```sh
% rsync -vaHz --delete rsync://ftp4.de.FreeBSD.org/FreeBSD/ /pub/FreeBSD/
```

查阅 rsync 的文档，文档也可以在 [http://rsync.samba.org/](http://rsync.samba.org/) 找到，了解可以与 rsync 一起使用的各种选项。如果你同步整个模块（不同于子目录），请注意模块目录（此处为“FreeBSD”）不会被创建，因此你不能省略目标目录。此外，你可能希望设置一个脚本框架，通过 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 来调用此命令。

### 3.2. 镜像 WWW 页面

>**警告**
>
> 由于自 2021 年 1 月 25 日文档迁移到 Hugo/Asciidoctor，使用 rsync 镜像网站不再有效。

目前正在研究如何使用 [官方基础设施](https://docs.freebsd.org/en/books/handbook/mirrors/) 实现网站镜像。

对于以前的网站镜像，今天实现网站镜像的一种方法是使用相应的托管地址在本地构建网站。

```sh
% cd website && env HUGO_baseURL="https://www.XX.freebsd.org/" make
```

有关构建工具的更多细节，请参见 [FreeBSD Documentation Project Primer for New Contributors](https://docs.freebsd.org/en/books/fdp-primer/overview/#overview-quick-start) 书籍。

>**注意**
>
>请注意，网站拆分为 [www.FreeBSD.org](http://www.FreeBSD.org) 和 docs.FreeBSD.org，并且它们之间有链接；此外，目前 `HUGO_baseURL` 变量无法涵盖所有链接，因此不建议镜像该网站。

### 3.3. 镜像包

由于带宽、存储和管理要求非常高，FreeBSD 项目决定不再允许公共镜像包。对于有大量机器的站点，运行一个用于 [pkg(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg&sektion=8&format=html) 进程的缓存 HTTP 代理可能是有利的。或者，可以通过运行以下命令来获取特定的包及其依赖项：

```sh
% pkg fetch -d -o /usr/local/mirror vim
```

这些包被获取后，必须通过运行以下命令来生成仓库元数据：

```sh
% pkg repo /usr/local/mirror
```

获取包并生成仓库的元数据后，通过 HTTP 将包提供给客户端机器。有关更多信息，请参阅 [pkg(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg&sektion=8&format=html) 的手册页，特别是 [pkg-repo(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg-repo&sektion=8&format=html) 页面。

### 3.4. 我应该多久镜像一次？

每个镜像站点至少应每天更新一次。为了防止多个运行实例同时发生，脚本中应有锁机制，以便通过 [cron(8)](https://man.freebsd.org/cgi/man.cgi?query=cron&sektion=8&format=html) 定期执行。由于几乎每个管理员的做法不同，不能提供具体的指令。一个可能的工作流程如下：

1. 将镜像应用程序的命令放入一个脚本中。建议使用纯粹的 `/bin/sh` 脚本。
2. 添加一些输出重定向，以便将诊断信息记录到文件中。
3. 测试脚本是否有效，检查日志。
4. 使用 [crontab(1)](https://man.freebsd.org/cgi/man.cgi?query=crontab&sektion=1&format=html) 将脚本添加到适当用户的 [crontab(5)](https://man.freebsd.org/cgi/man.cgi?query=crontab&sektion=5&format=html) 中。此用户应与运行 FTP 守护进程的用户不同，这样，如果 FTP 区域中的文件权限不是所有用户可读的，匿名 FTP 就无法访问这些文件。此步骤用于“暂存”版本，确保所有官方镜像站在发布日都有必要的发布文件。

以下是一些推荐的更新频率：

- FTP 文件集：每日
- WWW 页面：每日

## 4. 从哪里镜像

这是一个重要的问题。因此，本节将花一些时间解释背景。我们会多次提到：在任何情况下，都不应从 `ftp.FreeBSD.org` 镜像。

### 4.1. 关于组织的一些话

镜像按国家组织。所有官方镜像都有类似 `ftpN.CC.FreeBSD.org` 的 DNS 条目。*CC*（即国家代码）是该镜像所在国家的 *顶级域名*（TLD）。*N* 是一个数字，表示该主机在该国家的 *第 N* 个镜像。（同样适用于 `wwwN.CC.FreeBSD.org` 等）。也有没有 *CC* 部分的镜像站。这些镜像站连接非常好，能容纳大量并发用户。`ftp.FreeBSD.org` 实际上是两台机器，一台位于丹麦，另一台位于美国。它并不是主站，绝不能用来作为镜像源。许多在线文档将“交互式”用户引导到 `ftp.FreeBSD.org`，因此自动化的镜像系统应从其他机器获取镜像。

此外，还存在一个镜像层次结构，通常称为 *层级*。主站不常被提及，但可以描述为 *Tier-0*。从这些主站镜像的镜像站可以被视为 *Tier-1*，而从 *Tier-1* 镜像站镜像的镜像站则是 *Tier-2*，以此类推。鼓励官方站点的 *层级* 较低，但层级越低，对镜像站点的要求也越高，如 [FreeBSD 镜像要求](https://docs.freebsd.org/en/articles/hubs/#mirror-requirements) 所述。低层级的镜像站点访问可能会受到限制，且主站点的访问肯定是受限制的。层级结构不会通过 DNS 反映，通常也没有在任何地方文档化，除非是主站点。然而，具有较低数字（如 1-4）的官方镜像通常是 *Tier-1*（这只是一个大致的提示，并没有严格的规则）。

### 4.2. 好的，那我应该从哪里获取镜像？

在任何情况下，都不应从 `ftp.FreeBSD.org` 镜像。简短的答案是：从离你最近的站点，或者给你最快访问的站点获取。

#### 4.2.1. 我只想从某个地方镜像

如果你没有特别的意图或要求，可以参考 [《好的，那我应该从哪里获取镜像？》](https://docs.freebsd.org/en/articles/hubs/#mirror-where-where) 中的说明。这意味着：

1. 检查哪些站点提供最快的访问（跳数、往返时间），并提供你打算使用的服务（如 rsync）。
2. 联系你选择的站点的管理员，说明你的请求，并询问他们的条款和政策。
3. 按照上面描述的方式设置你的镜像。

#### 4.2.2. 我是一个官方镜像，适合我的正确站点是哪个？

通常 [《我只想从某个地方镜像！》](https://docs.freebsd.org/en/articles/hubs/#mirror-where-simple) 中的介绍仍然适用。当然，你可能希望考虑到你的上游镜像站点应处于较低的层级。有关 *官方* 镜像的一些其他考虑因素，可以参见 [《官方镜像》](https://docs.freebsd.org/en/articles/hubs/#mirror-official) 部分。

#### 4.2.3. 我想访问主站点

如果你有充分的理由和前提条件，你可能希望并能够访问其中一个主站点。对这些站点的访问通常是受限的，并且有专门的访问政策。如果你已经是一个 *官方* 镜像，这肯定有助于你获得访问权限。在其他情况下，请确保你的国家确实需要另一个镜像。如果该国有三个或更多镜像，请首先联系“区域管理员”([hostmaster@CC.FreeBSD.org](mailto:hostmaster@CC.FreeBSD.org)) 或 [FreeBSD 镜像站点邮件列表](https://lists.freebsd.org/subscription/freebsd-hubs)。

无论谁帮助你成为 *官方* 镜像，都应帮助你获得适当的上游主机访问权限，可能是某个主站点或合适的 *Tier-1* 站点。如果没有，你可以发送电子邮件到 [mirror-admin@FreeBSD.org](mailto:mirror-admin@FreeBSD.org) 请求帮助。

有一个用于 FTP 文件集的主站点。

##### 4.2.3.1. ftp-master.FreeBSD.org

这是 FTP 文件集的主站点。

`ftp-master.FreeBSD.org` 除了提供 FTP 外，还提供 rsync 访问。有关更多信息，请参考 [镜像 FTP 站点](https://docs.freebsd.org/en/articles/hubs/#mirror-ftp-rsync)。

还鼓励镜像站点允许访问 FTP 内容的 rsync，因为它们是 *Tier-1* 镜像站点。

## 5. 官方镜像

官方镜像是指：

- a) 具有 `FreeBSD.org` DNS 条目（通常是 CNAME）。
- b) 在 FreeBSD 文档（如手册）中列为官方镜像。

目前官方镜像的区分标准就是这些。官方镜像不一定是 *Tier-1* 镜像。但你很可能不会找到不是官方的 *Tier-1* 镜像。

### 5.1. 官方（Tier-1）镜像的特殊要求

对于所有官方镜像来说，要求并不容易一概而论，因为该项目对此有一定的宽容度。更容易说明的是 *官方 Tier-1 镜像* 的要求。其他官方镜像可以认为这些要求是一个很大的 *应该*。

Tier-1 镜像需要：

- 托管完整的文件集。
- 允许其他镜像站访问。
- 提供 FTP 和 rsync 访问。

此外，管理员应订阅 [FreeBSD 镜像站点邮件列表](https://lists.freebsd.org/subscription/freebsd-hubs)。请参见 [此链接](https://docs.freebsd.org/en/books/handbook/#eresources-mail) 了解如何订阅。

>**重要**
>
>对于集群管理员，尤其是 Tier-1 集群管理员来说，检查 [发布计划](https://www.freebsd.org/releng/) 是 *非常* 重要的。这很重要，因为它会告诉你下一个 FreeBSD 版本的发布日期，从而为你准备迎接随之而来的流量高峰提供时间。同样重要的是，集群管理员要尽量保持镜像站点尽可能最新（对于 Tier-1 镜像来说尤其如此）。如果 Mirror1 长时间未更新，低层级镜像站点将开始从 Mirror1 镜像旧数据，从而形成恶性循环……保持镜像站点更新！

### 5.2. 如何成为官方镜像？

请联系集群管理员，联系方式见 [https://www.FreeBSD.org/administration/#t-clusteradm](https://www.FreeBSD.org/administration/#t-clusteradm)。

## 6. 来自镜像站点的一些统计数据

以下是你喜欢的镜像站点的统计页面链接（即那些愿意提供统计数据的站点）。

### 6.1. FTP 站点统计数据

- ftp.is.FreeBSD.org - [hostmaster@is.FreeBSD.org](mailto:hostmaster@is.FreeBSD.org) - [(带宽)](http://www.rhnet.is/status/draupnir/draupnir.html) [(FTP 进程)](http://www.rhnet.is/status/ftp/ftp-notendur.html) [(HTTP 进程)](http://www.rhnet.is/status/ftp/http-notendur.html)
- ftp2.ru.FreeBSD.org - [mirror@macomnet.ru](mailto:mirror@macomnet.ru) - [(带宽)](http://mirror.macomnet.net/mrtg/mirror.macomnet.net_195.128.64.25.html) [(HTTP 和 FTP 用户)](http://mirror.macomnet.net/mrtg/mirror.macomnet.net_proc.html)
