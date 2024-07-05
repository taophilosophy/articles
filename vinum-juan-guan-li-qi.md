# vinum 卷管理器

## 1. 概要

无论是什么类型的磁盘，都可能存在潜在问题。磁盘可能太小、太慢或太不可靠，无法满足系统的要求。虽然磁盘变得越来越大，但数据存储需求也在增加。通常需要一个比磁盘容量更大的文件系统。针对这些问题已经提出并实施了各种解决方案。

一种方法是通过使用多个，有时是冗余的磁盘。除了支持各种卡和控制器用于硬件独立磁盘阵列 RAID 系统，基本的 FreeBSD 系统包括 vinum 卷管理器，这是一个实现虚拟磁盘驱动器并解决这三个问题的块设备驱动程序。vinum 提供比传统磁盘存储更灵活、性能更好、可靠性更高的解决方案，并实现 RAID -0， RAID -1 和 RAID -5 模型，可以单独或组合使用。

本章节概述了传统磁盘存储可能出现的问题，并介绍了 vinum 卷管理器。

|  | 从 FreeBSD 5 开始，vinum 已被重写以适应 GEOM 架构，同时保留原始思想、术语和磁盘上的元数据。这个重写被称为 gvinum（代表 GEOM vinum）。虽然本章使用术语 vinum，但任何命令调用都应该使用 gvinum 。内核模块的名称已从原始的 vinum.ko 更改为 geom_vinum.ko，并且所有设备节点都位于/dev/gvinum 而不是/dev/vinum 下。从 FreeBSD 6 开始，原始的 vinum 实现在代码库中不再可用。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

## 2. 访问瓶颈

现代系统经常需要以高度并发的方式访问数据。例如，大型 FTP 或 HTTP 服务器可以维护成千上万个并发会话，并且与外部世界有多个 100 兆位/s 的连接，远远超出大多数磁盘的持续传输速率。

当前磁盘驱动器可以按顺序传输数据，速度高达 70 MB/s，但在许多独立进程访问驱动器且它们可能只达到这些值的一小部分的环境中，这个值就不那么重要了。在这种情况下，更有趣的是从磁盘子系统的角度看问题。重要的参数是传输对子系统的负载，或者传输占用参与传输的驱动器的时间。

在任何磁盘传输中，驱动器必须首先定位磁头，等待第一个扇区通过读头，然后执行传输。这些操作可以被视为原子操作，因为中断它们是没有意义的。

考虑一个典型的约 10 kB 的传输：当前的高性能硬盘可以在平均 3.5 毫秒内定位磁头。最快的驱动器以每分钟 15,000 转的速度旋转，因此平均旋转延迟（半个旋转）为 2 毫秒。以 70 MB/s 的速度，传输本身大约需要 150 微秒，与定位时间相比几乎可以忽略不计。在这种情况下，有效的传输速率下降到略高于 1 MB/s，并且明显地高度依赖于传输大小。

这个瓶颈的传统和显而易见的解决方案是“更多的主轴”：不是使用一个大硬盘，而是使用几个存储空间相同的较小硬盘。每个硬盘都能独立定位和传输，因此有效吞吐量增加了一个接近于使用的硬盘数量的因子。

实际的吞吐量提高小于涉及的硬盘数量。尽管每个驱动器可以并行传输，但没有办法确保请求均匀分布在各驱动器之间。不可避免地，一个驱动器的负载会比另一个驱动器的负载更高。

磁盘上负载的均匀性在很大程度上取决于数据在驱动器之间的共享方式。在下面的讨论中，将磁盘存储看作是大量可通过编号寻址的数据扇区是很方便的，有点像书中的页面。最明显的方法是将虚拟磁盘划分为连续扇区组，大小与各个物理磁盘相同，并以这种方式存储它们，有点像将一本大书撕成小节。这种方法称为串联，它的优点是磁盘不需要具有任何特定的大小关系。当对虚拟磁盘的访问均匀分布在其地址空间时，它的效果很好。当访问集中在较小区域时，改进不是那么明显。串联组织说明了以串联方式分配存储单元的顺序。

![vinum concat](https://docs.freebsd.org/images/articles/vinum/vinum-concat.png)

图 1. 串联组织

另一种映射是将地址空间划分为较小的、大小相等的组件，并将它们顺序存储在不同的设备上。例如，前 256 个扇区可以存储在第一块磁盘上，接下来的 256 个扇区存储在下一块磁盘上，依此类推。填满最后一个磁盘后，该过程重复，直到磁盘满。这种映射称为条带化或 RAID-0。

RAID 提供各种形式的容错，尽管 RAID-0 有点误导，因为它不提供冗余。条带化需要更多的工作来定位数据，并且可能会导致额外的 I/O 负载，当传输分布在多个磁盘上时，但它也可以在磁盘上提供更均匀的负载。条带化组织说明了条带组织中存储单元分配的顺序。

![vinum striped](https://docs.freebsd.org/images/articles/vinum/vinum-striped.png)

图 2. 条带化组织

## 3. 数据完整性

磁盘的最后一个问题是它们不可靠。尽管可靠性在过去几年里有了巨大提高，但磁盘驱动器仍然是服务器最有可能出现故障的核心组件。一旦出现故障，后果可能是灾难性的，更换故障的磁盘驱动器并恢复数据可能导致服务器停机。

解决这个问题的一种方法是镜像，或者 RAID-1 ，它在不同的物理硬件上保留数据的两个副本。对卷的任何写操作都会写入两个磁盘；读取可以从任一磁盘中满足，因此如果一个驱动器出现故障，则数据仍然可以在另一个驱动器上找到。

镜像有两个问题：

* 它所需的磁盘存储量是非冗余解决方案的两倍。
* 写操作必须同时针对两个驱动器执行，因此占用了非镜像卷两倍的带宽。读取不会受到性能惩罚，甚至可能更快。

另一种解决方案是奇偶校验，实现在 RAID 级别的 2、3、4 和 5。其中， RAID-5 最有趣。在 vinum 的实现中，它是条带组织的一种变体，其中每个条带的一个块专门用于奇偶校验的另一块。通过 vinum 的实现，一个 RAID-5 plex 类似于条带 plex，不同之处在于它通过在每个条带中包含一个奇偶校验块，实现了 RAID-5 。根据 RAID-5 的要求，这个奇偶校验块的位置会从一个条带变到下一个。数据块中的数字表示相对块号码。

![vinum raid5 org](https://docs.freebsd.org/images/articles/vinum/vinum-raid5-org.png)

图 3. RAID -5 组织

与镜像相比， RAID-5 的优势在于需要的存储空间明显较少。读取访问类似于条带组织，但写入访问速度明显较慢，大约为读取性能的 25%。如果一个驱动器故障，阵列可以继续在降级模式下运行，其中从剩余可访问驱动器之一读取继续正常进行，但从故障驱动器读取则从所有剩余驱动器的相应块重新计算。

## 4. vinum 对象

要解决这些问题，vinum 实现了一个四级对象层次结构：

* 最显著的对象是虚拟磁盘，称为卷。 卷基本上具有与 UNIX®磁盘驱动器相同的属性，尽管有一些细微差异。 首先，它们没有大小限制。
* 卷由 plexes 组成，每个 plex 代表一个卷的总地址空间。 这个层次结构中的层提供了冗余性。 将 plexes 视为镜像阵列中的单独磁盘，每个磁盘包含相同的数据。
* 由于 vinum 存在于 UNIX®磁盘存储框架中，因此可以将 UNIX®分区作为多磁盘 plexes 的构建模块。实际上，这种做法过于僵化，因为 UNIX®磁盘只能具有有限数量的分区。相反，vinum 将单个 UNIX®分区——驱动器细分为称为子磁盘的连续区域，这些区域用作 plexes 的构建模块。
* 子磁盘驻留在 vinum 驱动器上，目前为 UNIX®分区。vinum 驱动器可以包含任意数量的子磁盘。除了驱动器开始处的一个小区域用于存储配置和状态信息外，整个驱动器都可用于数据存储。

以下各节描述了这些对象提供 vinum 所需功能的方式。

### 4.1. 卷大小考虑

Plexes 可以包含分布在 vinum 配置中所有驱动器上的多个子磁盘。因此，单个驱动器的大小不限制 plex 或卷的大小。

### 4.2. 冗余数据存储

vinum 通过将多个 plexes 附加到一个 volume 来实现镜像。每个 plex 是一个 volume 中数据的表示。一个 volume 可以包含一个到八个 plexes。

尽管一个 plex 表示一个 volume 的完整数据，但在物理上部分表示可能会缺失，这可能是设计使然（未定义部分 plex 的 subdisk）或事故导致（驱动器故障的结果）。只要至少有一个 plex 可以提供完整地址范围的数据，volume 就可以完全正常工作。

### 4.3. 哪种 Plex 结构？

vinum 在 plex 级别同时实现串联和条带化：

* 串联 plex 依次使用每个子磁盘的地址空间。串联 plex 最灵活，因为它们可以包含任意数量的子磁盘，并且子磁盘的长度可以不同。可以通过添加额外的子磁盘来扩展 plex。它们所需的 CPU 时间比条带化 plex 少，尽管 CPU 开销的差异无法测量。另一方面，它们最容易受到热点的影响，其中一个磁盘非常活跃，而其他磁盘处于空闲状态。
* 条带化 plex 将数据条带化到每个子磁盘上。所有子磁盘的大小必须相同，且至少有两个子磁盘才能与串联 plex 区分开来。条带化 plex 的最大优势在于减少热点。通过选择最佳大小的条带，约为 256 kB，可以平衡组件驱动器上的负载。通过添加新的子磁盘来扩展 plex 非常复杂，vinum 不实现这一功能。

vinum Plex Organizations 总结了每种 plex 组织的优缺点。

表 1。vinum Plex Organizations|Plex 类型|最小子磁盘|可添加子磁盘|必须相等大小|应用| | -----------| ------------------| ------------------| --------------------| -------------| |串联|1|是|否|大数据存储，具有最大的放置灵活性和中等性能| |条带化|2|否|是|与高度并发访问相结合的高性能|

## 5. 一些例子

vinum 维护一个配置数据库，描述了个体系统中已知对象。 最初，用户使用 gvinum(8)从一个或多个配置文件创建配置数据库。 vinum 在控制下的每个磁盘设备上存储其配置数据库的副本。 此数据库在每个状态更改时更新，以便重新启动准确恢复每个 vinum 对象的状态。

### 5.1. 配置文件

配置文件描述了单个 vinum 对象。 一个简单卷的定义可能是：

```
    drive a device /dev/da3h
    volume myvol
      plex org concat
        sd length 512m drive a
```

该文件描述了四个 vinum 对象：

* 驱动线描述了磁盘分区（驱动器）及其相对底层硬件的位置。它被赋予符号名称 a。符号名称与设备名称的分离允许磁盘在不同位置之间移动而不会引起混淆。
* 卷线描述了一个卷。唯一必需的属性是名称，在本例中为 myvol。
* plex 行定义一个 plex。唯一必需的参数是组织，这里是 concat。不需要名称，因为系统会自动通过在卷名称后添加后缀.px 生成名称，其中 x 是卷中 plex 的编号。因此，这个 plex 将被称为 myvol.p0。
* sd 行描述一个子磁盘。最低规格是存储它的驱动器名称和子磁盘的长度。不需要名称，因为系统会自动分配从 plex 名称派生的名称，通过添加后缀.sx，其中 x 是 plex 中子磁盘的编号。因此，vinum 给这个子磁盘命名为 myvol.p0.s0。

处理此文件后，gvinum(8)会产生以下输出：

```
#  gvinum -> create config1
Configuration summary
Drives:         1 (4 configured)
Volumes:        1 (4 configured)
Plexes:         1 (8 configured)
Subdisks:       1 (16 configured)

  D a                     State: up       Device /dev/da3h      Avail: 2061/2573 MB (80%)

  V myvol                 State: up       Plexes:       1 Size:      512 MB

  P myvol.p0            C State: up       Subdisks:     1 Size:      512 MB

  S myvol.p0.s0           State: up       PO:        0  B Size:      512 MB
```

这个输出显示了 gvinum(8)的简要列表格式。它在一个简单的 vinum 卷中以图形方式表示。

![vinum simple vol](https://docs.freebsd.org/images/articles/vinum/vinum-simple-vol.png)

图 4. 一个简单的 vinum 卷

这个图和接下来的图代表一个包含 plexes 的卷，这些 plexes 又包含 subdisks。在这个例子中，这个卷包含一个 plex，而这个 plex 又包含一个 subdisk。

这个特定的卷与传统磁盘分区没有特定优势。它包含一个 plex，因此不是冗余的。该 plex 包含一个子磁盘，因此与传统磁盘分区在存储分配上没有区别。以下各节说明了各种更有趣的配置方法。

### 5.2. 增加韧性：镜像

通过镜像可以增加卷的韧性。在布置镜像卷时，重要的是确保每个 plex 的子磁盘位于不同的驱动器上，以便驱动器故障不会导致两个 plex 都失效。以下配置镜像卷：

```
	drive b device /dev/da4h
	volume mirror
      plex org concat
        sd length 512m drive a
	  plex org concat
	    sd length 512m drive b
```

在这个例子中，不需要再次指定驱动器 a 的定义，因为 vinum 会跟踪其配置数据库中的所有对象。处理完这个定义后，配置看起来像这样：

```
	Drives:         2 (4 configured)
	Volumes:        2 (4 configured)
	Plexes:         3 (8 configured)
	Subdisks:       3 (16 configured)

	D a                     State: up       Device /dev/da3h       Avail: 1549/2573 MB (60%)
	D b                     State: up       Device /dev/da4h       Avail: 2061/2573 MB (80%)

    V myvol                 State: up       Plexes:       1 Size:        512 MB
    V mirror                State: up       Plexes:       2 Size:        512 MB

    P myvol.p0            C State: up       Subdisks:     1 Size:        512 MB
    P mirror.p0           C State: up       Subdisks:     1 Size:        512 MB
    P mirror.p1           C State: initializing     Subdisks:     1 Size:        512 MB

    S myvol.p0.s0           State: up       PO:        0  B Size:        512 MB
	S mirror.p0.s0          State: up       PO:        0  B Size:        512 MB
	S mirror.p1.s0          State: empty    PO:        0  B Size:        512 MB
```

镜像的 vinum 卷以图形方式显示结构。

![vinum mirrored vol](https://docs.freebsd.org/images/articles/vinum/vinum-mirrored-vol.png)

图 5. 镜像的 vinum 卷

在这个例子中，每个 plex 包含完整的 512 MB 地址空间。与上一个例子一样，每个 plex 只包含一个子磁盘。

### 5.3. 优化性能

在前面的示例中，镜像卷比未镜像卷更耐故障，但性能较差，因为对卷的每次写入都需要同时写入两个驱动器，占用了总磁盘带宽的更大比例。性能考虑需要采用不同的方法：数据不是镜像，而是跨尽可能多的磁盘驱动器进行条带化。以下配置显示了一个 plex 跨越四个磁盘驱动器的卷：

```
        drive c device /dev/da5h
	drive d device /dev/da6h
	volume stripe
	plex org striped 512k
	  sd length 128m drive a
	  sd length 128m drive b
	  sd length 128m drive c
	  sd length 128m drive d
```

与以前一样，不需要定义已知于 vinum 的驱动器。处理完此定义后，配置如下：

```
	Drives:         4 (4 configured)
	Volumes:        3 (4 configured)
	Plexes:         4 (8 configured)
	Subdisks:       7 (16 configured)

    D a                     State: up       Device /dev/da3h        Avail: 1421/2573 MB (55%)
    D b                     State: up       Device /dev/da4h        Avail: 1933/2573 MB (75%)
    D c                     State: up       Device /dev/da5h        Avail: 2445/2573 MB (95%)
    D d                     State: up       Device /dev/da6h        Avail: 2445/2573 MB (95%)

    V myvol                 State: up       Plexes:       1 Size:        512 MB
    V mirror                State: up       Plexes:       2 Size:        512 MB
    V striped               State: up       Plexes:       1 Size:        512 MB

    P myvol.p0            C State: up       Subdisks:     1 Size:        512 MB
    P mirror.p0           C State: up       Subdisks:     1 Size:        512 MB
    P mirror.p1           C State: initializing     Subdisks:     1 Size:        512 MB
    P striped.p1            State: up       Subdisks:     1 Size:        512 MB

    S myvol.p0.s0           State: up       PO:        0  B Size:        512 MB
    S mirror.p0.s0          State: up       PO:        0  B Size:        512 MB
    S mirror.p1.s0          State: empty    PO:        0  B Size:        512 MB
    S striped.p0.s0         State: up       PO:        0  B Size:        128 MB
    S striped.p0.s1         State: up       PO:      512 kB Size:        128 MB
    S striped.p0.s2         State: up       PO:     1024 kB Size:        128 MB
    S striped.p0.s3         State: up       PO:     1536 kB Size:        128 MB
```

![vinum striped vol](https://docs.freebsd.org/images/articles/vinum/vinum-striped-vol.png)

图 6. 条带化的 vinum 卷

此卷在条带化的 vinum 卷中表示。条纹的深浅表示在 plex 地址空间中的位置，最浅的条纹在前，最深的在后。

### 5.4. 韧性和性能

有足够的硬件，可以构建与标准 UNIX® 分区相比具有更高韧性和性能的卷。典型的配置文件可能是：

```
	volume raid10
      plex org striped 512k
        sd length 102480k drive a
        sd length 102480k drive b
        sd length 102480k drive c
        sd length 102480k drive d
        sd length 102480k drive e
      plex org striped 512k
        sd length 102480k drive c
        sd length 102480k drive d
        sd length 102480k drive e
        sd length 102480k drive a
        sd length 102480k drive b
```

第二个数据复制的子磁盘与第一个数据复制的子磁盘相隔两个驱动器。这有助于确保即使传输跨越两个驱动器，写入也不会进入相同的子磁盘。

一个镜像，条纹的 vinum 卷代表该卷的结构。

![vinum raid10 vol](https://docs.freebsd.org/images/articles/vinum/vinum-raid10-vol.png)

图 7。一个镜像，条纹的 vinum 卷

## 6。对象命名

vinum 分配默认名称给 plexes 和 subdisks，尽管它们可以被覆盖。不建议覆盖默认名称，因为这不会带来显著优势，而且会引起混淆。

名称可以包含任何非空白字符，但建议将它们限制在字母、数字和下划线字符。卷、plexes 和 subdisks 的名称可以长达 64 个字符，而驱动器的名称可以长达 32 个字符。

vinum 对象在层次结构/dev/gvinum 中分配设备节点。上面显示的配置将导致 vinum 创建以下设备节点：

* 每个卷的设备条目。这些是 vinum 主要使用的设备。上面的配置将包括设备 /dev/gvinum/myvol、/dev/gvinum/mirror、/dev/gvinum/striped、/dev/gvinum/raid5 和 /dev/gvinum/raid10。
* 所有卷都直接在 /dev/gvinum/ 下得到条目。
* 目录 /dev/gvinum/plex 和 /dev/gvinum/sd，它们包含每个 plex 和每个子磁盘的设备节点。

例如，考虑以下配置文件：

```
	drive drive1 device /dev/sd1h
	drive drive2 device /dev/sd2h
	drive drive3 device /dev/sd3h
	drive drive4 device /dev/sd4h
    volume s64 setupstate
      plex org striped 64k
        sd length 100m drive drive1
        sd length 100m drive drive2
        sd length 100m drive drive3
        sd length 100m drive drive4
```

处理此文件后，gvinum(8) 在 /dev/gvinum 中创建以下结构：

```
	drwxr-xr-x  2 root  wheel       512 Apr 13
16:46 plex
	crwxr-xr--  1 root  wheel   91,   2 Apr 13 16:46 s64
	drwxr-xr-x  2 root  wheel       512 Apr 13 16:46 sd

    /dev/vinum/plex:
    total 0
    crwxr-xr--  1 root  wheel   25, 0x10000002 Apr 13 16:46 s64.p0

    /dev/vinum/sd:
    total 0
    crwxr-xr--  1 root  wheel   91, 0x20000002 Apr 13 16:46 s64.p0.s0
    crwxr-xr--  1 root  wheel   91, 0x20100002 Apr 13 16:46 s64.p0.s1
    crwxr-xr--  1 root  wheel   91, 0x20200002 Apr 13 16:46 s64.p0.s2
    crwxr-xr--  1 root  wheel   91, 0x20300002 Apr 13 16:46 s64.p0.s3
```

尽管建议不为 plexes 和 subdisks 分配特定名称，vinum 驱动器必须具有名称。这样可以将驱动器移动到不同位置并仍然自动识别它。驱动器的名称最长可达 32 个字符。

### 6.1. 创建文件系统

卷对系统来说与磁盘相同，但有一个例外。与 UNIX®驱动器不同，vinum 不会对卷进行分区，因此不包含分区表。这要求修改一些磁盘工具，特别是 newfs(8)，以便它不会尝试将 vinum 卷名称的最后一个字母解释为分区标识符。例如，磁盘驱动器的名称可能类似于/dev/ad0a 或/dev/da2h。这些名称分别代表第一个 IDE 磁盘(ad)上的第一个分区(a)和第三个 SCSI 磁盘(da)上的第八个分区(h)。相比之下，vinum 卷的名称可能是/dev/gvinum/concat，它与分区名称没有任何关系。

要在此卷上创建文件系统，请使用 newfs(8)：

```
# newfs /dev/gvinum/concat
```

## 7. 配置 vinum

GENERIC 内核不包含 vinum。可以构建包含 vinum 的自定义内核，但不建议这样做。启动 vinum 的标准方式是作为一个内核模块。不需要 kldload(8)，因为当 gvinum(8)启动时，它会检查模块是否已被加载，如果没有加载，则会自动加载。

### 7.1. 启动

在磁盘切片上存储配置信息，形式与配置文件中基本相同。从配置数据库读取时，vinum 识别一些配置文件中不允许的关键字。例如，磁盘配置可能包含以下文本：

```
volume myvol state up
volume bigraid state down
plex name myvol.p0 state up org concat vol myvol
plex name myvol.p1 state up org concat vol myvol
plex name myvol.p2 state init org striped 512b vol myvol
plex name bigraid.p0 state initializing org raid5 512b vol bigraid
sd name myvol.p0.s0 drive a plex myvol.p0 state up len 1048576b driveoffset 265b plexoffset 0b
sd name myvol.p0.s1 drive b plex myvol.p0 state up len 1048576b driveoffset 265b plexoffset 1048576b
sd name myvol.p1.s0 drive c plex myvol.p1 state up len 1048576b driveoffset 265b plexoffset 0b
sd name myvol.p1.s1 drive d plex myvol.p1 state up len 1048576b driveoffset 265b plexoffset 1048576b
sd name myvol.p2.s0 drive a plex myvol.p2 state init len 524288b driveoffset 1048841b plexoffset 0b
sd name myvol.p2.s1 drive b plex myvol.p2 state init len 524288b driveoffset 1048841b plexoffset 524288b
sd name myvol.p2.s2 drive c plex myvol.p2 state init len 524288b driveoffset 1048841b plexoffset 1048576b
sd name myvol.p2.s3 drive d plex myvol.p2 state init len 524288b driveoffset 1048841b plexoffset 1572864b
sd name bigraid.p0.s0 drive a plex bigraid.p0 state initializing len 4194304b driveoff set 1573129b plexoffset 0b
sd name bigraid.p0.s1 drive b plex bigraid.p0 state initializing len 4194304b driveoff set 1573129b plexoffset 4194304b
sd name bigraid.p0.s2 drive c plex bigraid.p0 state initializing len 4194304b driveoff set 1573129b plexoffset 8388608b
sd name bigraid.p0.s3 drive d plex bigraid.p0 state initializing len 4194304b driveoff set 1573129b plexoffset 12582912b
sd name bigraid.p0.s4 drive e plex bigraid.p0 state initializing len 4194304b driveoff set 1573129b plexoffset 16777216b
```

显而易见的区别在于明确的位置信息和命名的存在，这两者都是允许但不鼓励的，并且状态信息。vinum 不会在配置信息中存储有关驱动器的信息。它通过扫描配置的磁盘驱动器，查找具有 vinum 标签的分区来查找驱动器。这使 vinum 能够正确识别驱动器，即使它们已被分配不同的 UNIX®驱动器 ID。

#### 7.1.1. 自动启动

Gvinum 一旦加载内核模块，始终具有自动启动功能，通过 loader.conf(5)。要在启动时加载 Gvinum 模块，请将 geom_vinum_load="YES" 添加到/boot/loader.conf。

当使用 gvinum start 启动 vinum 时，vinum 会从 vinum 驱动器之一读取配置数据库。在正常情况下，每个驱动器包含配置数据库的相同副本，因此读取哪个驱动器并不重要。但是，在崩溃后，vinum 必须确定哪个驱动器最近更新，并从该驱动器读取配置。然后，如果需要，它会从逐渐较旧的驱动器更新配置。

## 8. 使用 vinum 作为根文件系统

对于具有完全镜像文件系统的机器，使用 vinum，最好也镜像根文件系统。设置这样的配置比镜像任意文件系统更不平凡，因为：

* 根文件系统在引导过程中必须非常早就可用，因此 vinum 基础设施必须在此时已经可用。
* 包含根文件系统的卷还包含系统引导程序和内核。这些必须使用主机系统的本机实用程序进行读取，例如 BIOS，通常无法了解 vinum 的详细信息。

在接下来的章节中，通常使用术语“根卷”来描述包含根文件系统的 vinum 卷。

### 8.1. 为根文件系统足够早地启动 vinum

vinum 必须在系统引导的早期可用，因为 loader(8)必须能够在启动内核之前加载 vinum 内核模块。这可以通过将此行放入/boot/loader.conf 文件中来实现：

```
geom_vinum_load="YES"
```

### 8.2. 使基于 vinum 的根卷可以自举

当前的 FreeBSD 自举代码只有 7.5 KB，并不了解内部的 vinum 结构。这意味着它无法解析 vinum 配置数据或识别引导卷的元素。因此，需要一些变通方法来为自举代码提供包含根文件系统的标准 a 分区的假象。

为了实现这一点，根卷必须满足以下要求：

* 根卷不得是条带或 RAID -5。
* 根卷不得包含超过一个每个 plex 的串联子磁盘。

请注意，使用多个包含根文件系统副本的 plex 是可取且可能的。引导过程只会使用一个副本来查找引导程序和所有引导文件，直到内核挂载根文件系统。在这些 plex 中的每个单独的子磁盘都需要其自己的 a 分区幻像，以使对应的设备可引导。并不严格要求这些虚假的 a 分区都位于其设备内的相同偏移量上，与包含根卷 plex 的其他设备相比。然而，最好以这种方式创建 vinum 卷，以便生成的镜像设备是对称的，以避免混淆。

为每个包含根卷一部分的设备设置这些 a 分区，需要执行以下操作：

1. 需要检查作为根卷一部分的此设备子磁盘的位置、距离设备开头的偏移量和大小，使用以下命令：

    ```
    # gvinum l -rv root
    ```

    vinum 的偏移量和大小以字节为单位。必须将它们除以 512 以获得 bsdlabel 要使用的块号。
2. 对于参与根卷的每个设备，请运行此命令：

    ```
    # bsdlabel -e devname
    ```

    devname 必须是磁盘的名称，如对于没有分区表的磁盘为 da0，或者分区的名称，如 ad0s1。

    如果设备上已经有来自预-vinum 根文件系统的 a 分区，则应将其重命名为其他名称，以便保持可访问性（以防万一），但不再默认用于引导系统。当前挂载的根文件系统无法重命名，因此必须在从“Fixit”媒体引导时执行此操作，或者在镜像中，在当前未引导的磁盘上首先执行操作。

    如果有的话，这个设备上 vinum 分区的偏移量必须加上该设备上相应根卷次级磁盘的偏移量。得到的值将成为新的 offset 分区的值。这个分区的 size 值可以直接从上面的计算中获取。 fstype 应该是 4.2BSD 。 fsize 、 bsize 和 cpg 值应该选择以匹配实际文件系统，尽管它们在这种情况下并不是很重要。

    这样，一个新的 a 分区将被建立，它与该设备上的 vinum 分区重叠。只有在使用 vinum fstype 正确标记 vinum 分区时， bsdlabel 才允许出现这种重叠。
3. 现在，每个具有根卷一个副本的设备上都存在一个伪装的 a 分区。强烈建议使用类似以下命令来验证结果：

    ```
    # fsck -n /dev/devnamea
    ```

应记住，所有包含控制信息的文件必须相对于 vinum 卷中的根文件系统，当设置新的 vinum 根卷时，可能与当前活动的根文件系统不匹配。因此，特别需要注意的是，/etc/fstab 和/boot/loader.conf 需要被处理。

在下一次重新启动时，引导程序应该从新的基于 vinum 的根文件系统中找出适当的控制信息，并相应地采取行动。在内核初始化过程结束后，在所有设备都已经宣布之后，显示此设置成功的重要通知的消息是：

```
Mounting root from ufs:/dev/gvinum/root
```

### 8.3. 基于 vinum 的根设置示例

设置了 vinum 根卷后， gvinum l -rv root 的输出可能如下所示：

```
...
Subdisk root.p0.s0:
		Size:        125829120 bytes (120 MB)
		State: up
		Plex root.p0 at offset 0 (0  B)
		Drive disk0 (/dev/da0h) at offset 135680 (132 kB)

Subdisk root.p1.s0:
		Size:        125829120 bytes (120 MB)
		State: up
		Plex root.p1 at offset 0 (0  B)
		Drive disk1 (/dev/da1h) at offset 135680 (132 kB)
```

要注意的值为 135680 ，相对于分区/dev/da0h 的偏移量，这在`bsdlabel`的术语中相当于 265 个 512 字节的磁盘块。同样，此根卷的大小为 245760 个 512 字节块。包含此根卷第二副本的/dev/da1h 具有对称设置。

这些设备的 bsdlabel 可能如下所示：

```
...
8 partitions:
#        size   offset    fstype   [fsize bsize bps/cpg]
  a:   245760      281    4.2BSD     2048 16384     0  # (Cyl.    0*- 15*)
  c: 71771688        0    unused        0     0        # (Cyl.    0 - 4467*)
  h: 71771672       16     vinum                       # (Cyl.    0*- 4467*)
```

观察到伪造的 a 分区的 size 参数与上述数值相匹配，而 offset 参数是 vinum 分区 h 内的偏移量之和，以及该分区在设备或切片内的偏移量。这是一个典型的设置，必须避免《无法引导，引导程序发生紧急情况》中描述的问题。整个分区完全位于包含此设备所有 vinum 数据的 h 分区内。

在上述例子中，整个设备都专用于 vinum，并且没有剩余的预-vinum 根分区。

### 8.4. 故障排除

以下列表包含一些已知的陷阱和解决方案。

#### 8.4.1. 系统引导加载，但系统无法启动

如果由于任何原因系统无法继续启动，引导程序可通过在 10 秒警告时按下空格键来中断。可通过输入 show 来检查加载程序变量 vinum.autostart ，并使用 set 或 unset 进行操作。

如果 vinum 内核模块尚未列在要自动加载的模块列表中，请键入 load geom_vinum 。

准备就绪后，通过键入 boot -as 可继续引导过程，该过程 -as 请求内核请求根文件系统挂载（ -a ），并使引导过程停在单用户模式（ -s ），在单用户模式下，根文件系统以只读方式挂载。这样，即使仅挂载了复合卷的一个 plex，也不会在 plexes 之间出现数据不一致的风险。

在提示输入根文件系统挂载的提示符下，可输入包含有效根文件系统的任何设备。如果/etc/fstab 设置正确，应该默认是类似于 ufs:/dev/gvinum/root 的内容。一个典型的备选选择可能是类似于 ufs:da0d 的内容，这可能是包含预-vinum 根文件系统的假想分区。如果在这里输入了一个别名 a 分区之一，请务必确保它确实引用 vinum 根设备的子磁盘，因为在镜像设置中，这将仅挂载镜像根设备的一部分。如果此文件系统稍后要挂载为读写模式，则必须删除 vinum 根卷的其他 plex（es），因为否则这些 plexes 将携带不一致的数据。

#### 8.4.2. 仅主引导加载

如果 /boot/loader 无法加载，但主引导仍然加载（在启动过程开始后屏幕左侧的单破折号可见），可以尝试通过按下空格键来中断主引导。这将使引导停在第二阶段。在这里可以尝试从备用分区引导，比如包含先前已移开的根文件系统的分区 a 。

#### 8.4.3. 无法引导，引导程序发生故障

如果引导程序已被 vinum 安装破坏，将会发生这种情况。不幸的是，vinum 在开始写入其 vinum 头部信息之前，意外地只在其分区的开头留下了 4 KB 的空闲空间。然而，阶段一和阶段二的引导程序加上 bsdlabel 需要 8 KB。因此，如果一个 vinum 分区从一个旨在可引导的切片或磁盘的偏移量 0 开始，vinum 设置将破坏引导程序。

同样，如果上述情况已通过从“Fixit”媒体引导并使用 bsdlabel -B 重新安装引导程序来恢复，引导程序将破坏 vinum 头部，vinum 将不再找到其磁盘。虽然实际的 vinum 配置数据或 vinum 卷中的数据不会被破坏，并且通过再次输入完全相同的 vinum 配置数据可以恢复所有数据，但这种情况很难修复。必须将整个 vinum 分区至少移动 4 KB，以使 vinum 头部和系统引导程序不再发生冲突。
