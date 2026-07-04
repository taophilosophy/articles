# vinum 卷管理器

- 原文：[The vinum Volume Manager](https://docs.freebsd.org/en/articles/vinum/)

## 1. 概述

无论硬盘类型如何，总是存在潜在的问题。硬盘可能太小、太慢，或者可靠性不足，无法满足系统的要求。尽管硬盘的容量越来越大，但数据存储需求也在不断增加。常常需要一个比硬盘容量更大的文件系统。为了解决这些问题，人们提出了各种解决方案并加以实现。

其中一种方法是使用多个有时是冗余的硬盘。除了支持硬件独立磁盘冗余阵列（RAID）系统的各种卡和控制器外，FreeBSD 基本系统还包括 **vinum** 卷管理器，它是块设备驱动程序，实现了虚拟磁盘驱动器，并解决了这三个问题。**vinum** 提供比传统磁盘存储更高的灵活性、性能和可靠性，并实现了 `RAID`-0、`RAID`-1 和 `RAID`-5 模型，既可以单独使用，也可以组合使用。

本章概述了传统磁盘存储的潜在问题，并介绍了 **vinum** 卷管理器。

>**警告**
>
> 已弃用 **vinum**，并且在 FreeBSD 15.0 及以后的版本中不再存在。建议用户迁移到 [gconcat(8)](https://man.freebsd.org/cgi/man.cgi?query=gconcat&sektion=8&format=html)、[gmirror(8)](https://man.freebsd.org/cgi/man.cgi?query=gmirror&sektion=8&format=html)、[gstripe(8)](https://man.freebsd.org/cgi/man.cgi?query=gstripe&sektion=8&format=html)、[graid(8)](https://man.freebsd.org/cgi/man.cgi?query=graid&sektion=8&format=html) 或 [zfs(8)](https://man.freebsd.org/cgi/man.cgi?query=zfs&sektion=8&format=html)。

>**注意**
>
> 从 FreeBSD 5 开始，**vinum** 经过重写以适应 [GEOM 架构](https://docs.freebsd.org/en/books/handbook/#geom)，同时保留了原始的思想、术语和磁盘上的元数据。这个重写版本称为 *gvinum*（即 *GEOM vinum*）。虽然本章使用 **vinum** 这个术语，但所有命令的调用应使用 `gvinum`。内核模块的名称已从原来的 **vinum.ko** 更改为 **geom_vinum.ko**，所有设备节点位于 **/dev/gvinum** 下，而不是 **/dev/vinum**。从 FreeBSD 6 开始，原始的 **vinum** 实现不再出现在代码库中。

## 2. 访问瓶颈

现代系统经常需要高并发地访问数据。例如，大型 FTP 或 HTTP 服务器可以维护数千个并发会话，并通过多个 100 Mbit/s 的连接与外部世界通信，这远超大多数硬盘的持续传输速率。

当前的硬盘可以以最高 70 MB/s 的速度顺序传输数据，但在许多独立进程访问同一硬盘的环境下，这个速度并不重要，而且它们可能只能达到这些数值的一小部分。在这种情况下，更有意义的是从磁盘子系统的角度来看问题。关键的参数是传输给子系统带来的负载，或者传输占用参与传输的硬盘的时间。

在任何磁盘传输中，硬盘必须先定位磁头，等待第一个扇区通过读写磁头，然后执行传输。这些动作可以视为原子操作，因为中途打断它们是没有意义的。

假设一次典型的传输大约是 10 kB：当前一代高性能硬盘平均定位磁头的时间是 3.5 毫秒。最快的硬盘转速为每分钟 15,000 转，因此平均旋转延迟（半圈）是 2 毫秒。在 70 MB/s 的传输速率下，传输本身大约需要 150 微秒，这几乎可以忽略不计，相比之下，定位时间占据了绝大部分时间。在这种情况下，实际的传输速率降到了每秒 1 MB 以上，并且显然高度依赖于传输的大小。

传统且明显的解决方法是“更多的盘片”：与其使用一块大硬盘，不如使用多个较小的硬盘，总存储空间相同。每个硬盘都能独立进行定位和传输，因此有效吞吐量将接近所用硬盘的数量。

然而，实际的吞吐量提高幅度小于所用硬盘的数量。尽管每个硬盘都能够并行传输，但无法确保请求能均匀分布在各个硬盘上。不可避免地，某些硬盘的负载将高于其他硬盘。

硬盘负载的均匀性与数据在硬盘上的分布方式密切相关。以下讨论中，为便于理解，可以将磁盘存储看作大量可按编号访问的数据扇区，就像一本书的页面一样。最明显的方法是将虚拟磁盘划分为多个连续的扇区组，每组大小与单个硬盘的大小相同，并按此方式存储，类似于将一本大书撕成若干小部分。该方法称为 *连接（concatenation）*，其优点是硬盘不需要有任何特定的大小关系。当对虚拟磁盘的访问均匀分布在其地址空间时，这种方法效果良好。如果访问集中在较小的区域，效果则较差。[连接组织](https://docs.freebsd.org/en/articles/vinum/#vinum-concat) 说明了在连接组织中，存储单元分配的顺序。

![vinum concat](https://docs.freebsd.org/images/articles/vinum/vinum-concat.png)

**图 1. 连接组织**

另一种映射方法是将地址空间划分为更小的、大小相等的单元，并将它们顺序存储在不同的设备上。例如，前 256 个扇区存储在第一块硬盘上，接下来的 256 个扇区存储在第二块硬盘上，以此类推。填满最后一块硬盘后，该过程重复进行，直到所有硬盘都填满。这个映射方法称为 *条带化（striping）* 或 RAID-0。

`RAID` 提供了各种形式的容错机制，但 RAID-0 有些误导，因为它没有提供冗余。条带化需要稍多的工作来定位数据，并且当传输跨多个硬盘时，它可能会导致额外的 I/O 负载，但它也可以提供更均匀的硬盘负载。[条带化组织](https://docs.freebsd.org/en/articles/vinum/#vinum-striped) 说明了在条带化组织中，存储单元分配的顺序。

![vinum striped](https://docs.freebsd.org/images/articles/vinum/vinum-striped.png)

**图 2. 条带化组织**


## 3. 数据完整性

硬盘的最终问题是它们不可靠。尽管近年来硬盘的可靠性大大提高，但硬盘仍然是服务器中最容易发生故障的核心组件。当发生故障时，后果可能是灾难性的，更换故障的硬盘并恢复数据可能导致服务器停机。

解决这个问题的一种方法是 *镜像*（*mirroring*），即 `RAID-1`，它在不同的物理硬件上保存两份数据副本。对卷的任何写入操作都会同时写入两个硬盘；读取操作可以从任何一个硬盘上获取，因此如果一个硬盘发生故障，数据仍然可以从另一个硬盘上获得。

镜像有两个问题：

- 它需要比非冗余解决方案多一倍的磁盘存储空间。
- 写入操作必须同时写入两个硬盘，因此它们占用了比非镜像卷多一倍的带宽。读取操作不受性能影响，甚至可能更快。

另一种解决方案是 *奇偶校验*（*parity*），它在 `RAID` 2、3、4 和 5 等级中得到了实现。其中，`RAID-5` 是最有趣的。在 **vinum** 中，它是条带化组织的一种变体，其中每个条带中的一个块用于存储其他块的奇偶校验块。**vinum** 中实现的 `RAID-5` 结构类似于条带化结构，只不过它通过在每个条带中包含一个奇偶校验块来实现 `RAID-5`。根据 `RAID-5` 的要求，这个奇偶校验块的位置从一个条带到下一个条带不断变化。数据块中的数字表示相对块号。

![vinum raid5 org](https://docs.freebsd.org/images/articles/vinum/vinum-raid5-org.png)

**图 3. `RAID`-5 组织结构**

与镜像相比，`RAID-5` 的优势在于它显著减少了存储空间的需求。读取访问与条带化组织类似，但写入访问明显更慢，约为读取性能的 25%。如果一个硬盘发生故障，阵列仍然可以在降级模式下运行，剩余可访问硬盘中的读取操作仍然正常进行，而从故障硬盘读取的数据则会从所有剩余硬盘的相应块重新计算得出。

## 4. **vinum** 对象

为了解决这些问题，**vinum** 实现了四级层次结构的对象体系：

- 最显著的对象是虚拟磁盘，称为 *卷*（volume）。卷本质上与 UNIX® 磁盘驱动器具有相同的属性，尽管有一些小的区别。例如，卷没有大小限制。
- 卷由 *plex* 组成，每个 plex 表示卷的整个地址空间。这个层次结构提供了冗余。可以把 plex 看作是镜像阵列中的单个硬盘，每个硬盘都包含相同的数据。
- 由于 **vinum** 存在于 UNIX® 磁盘存储框架中，因此可以使用 UNIX® 分区作为构建多磁盘 plex 的基础块。事实上，这样做太不灵活，因为 UNIX® 磁盘只能有有限数量的分区。因此，**vinum** 将单个 UNIX® 分区（称为 *驱动器*）划分为连续区域，这些区域称为 *子磁盘*（subdisk），并将它们作为构建 plex 的基础块。
- 子磁盘位于 **vinum** *驱动器* 上，当前是 UNIX® 分区。**vinum** 驱动器可以包含任意数量的子磁盘。除了驱动器开头的小区域（用于存储配置和状态信息）外，整个驱动器都可以用于数据存储。

以下部分描述了这些对象如何提供 **vinum** 所需的功能。

### 4.1. 卷大小考虑

Plex 可以包含多个子磁盘，这些子磁盘分布在 **vinum** 配置中的所有驱动器上。因此，单个驱动器的大小不会限制 plex 或卷的大小。

### 4.2. 冗余数据存储

**vinum** 通过将多个 plex 附加到一个卷来实现镜像。每个 plex 表示卷中的数据。卷可以包含一到八个 plex。

虽然 plex 代表卷的完整数据，但某些表示部分可能会物理缺失，可能是有意的（通过不为 plex 的某些部分定义子磁盘）或偶然的（由于驱动器故障）。只要至少一个 plex 能提供卷完整地址范围的数据，卷就能正常工作。

### 4.3. 选择哪种 Plex 组织方式？

**vinum** 在 plex 层次上实现了连接和条带化：

- *连接 plex* 依次使用每个子磁盘的地址空间。连接的 plex 是最灵活的，因为它们可以包含任意数量的子磁盘，且子磁盘的长度可以不同。通过添加额外的子磁盘，可以扩展 plex。与条带化 plex 相比，它们所需的 CPU 时间较少，尽管 CPU 开销的差异不可测量。另一方面，连接的 plex 更容易出现热点，其中某块硬盘非常活跃，而其他硬盘处于空闲状态。
- *条带化 plex* 将数据条带化存储在每个子磁盘上。所有子磁盘必须具有相同的大小，且必须至少有两个子磁盘，以将其与连接 plex 区分开来。条带化 plex 的最大优势是它们减少了热点问题。通过选择最佳大小的条带（大约 256 kB），可以平衡各个组件驱动器的负载。通过添加新的子磁盘扩展 plex 是非常复杂的，**vinum** 没有实现这种扩展。

[vinum Plex 组织方式](https://docs.freebsd.org/en/articles/vinum/#vinum-comparison) 总结了每种 plex 组织方式的优缺点。

**表 1. **vinum** Plex 组织方式**

| Plex 类型 | 最小子磁盘数 | 可添加子磁盘数 | 必须相等的大小 | 应用场景                |
| ------- | ------ | ------- | ------- | ------------------- |
| 连接式     | 1      | 是       | 否       | 大型数据存储，具有最大布局灵活性和适度性能 |
| 条带化     | 2      | 否       | 是       | 高性能，适用于高度并发访问       |

## 5. 一些示例

**vinum** 维护 *配置数据库*，描述了某个特定系统中已知的对象。用户最初通过一个或多个配置文件使用 [gvinum(8)](https://man.freebsd.org/cgi/man.cgi?query=gvinum&sektion=8&format=html) 创建配置数据库。**vinum** 在每个受其控制的硬盘 *设备* 上存储其配置数据库的副本。该数据库在每次状态变化时更新，以便在重启时准确恢复每个 **vinum** 对象的状态。

### 5.1. 配置文件

配置文件描述了各个 **vinum** 对象。简单卷的定义可能如下所示：

```sh
drive a device /dev/da3h
    volume myvol
      plex org concat
        sd length 512m drive a
```

这个文件描述了四个 **vinum** 对象：

- *drive* 行描述了磁盘分区（*驱动器*）及其相对于底层硬件的位置。它被赋予符号名称 *a*。这种将符号名称与设备名称分开的方法允许磁盘在不同位置之间移动而不会产生混淆。
- *volume* 行描述了卷。唯一需要的属性是名称，这里是 *myvol*。
- *plex* 行定义了 plex。唯一必需的参数是组织方式，这里是 *concat*。不需要名称，因为系统会通过添加后缀 *.px* 自动生成名称，其中 *x* 是该卷中 plex 的编号。因此，这个 plex 将被命名为 *myvol.p0*。
- *sd* 行描述了子磁盘。最小规格是存储子磁盘的驱动器名称和子磁盘的长度。无需名称，因为系统会通过添加后缀 *.sx* 自动分配名称，其中 *x* 是该 plex 中子磁盘的编号。因此，**vinum** 将此子磁盘命名为 *myvol.p0.s0*。

在处理完该文件后，[gvinum(8)](https://man.freebsd.org/cgi/man.cgi?query=gvinum&sektion=8&format=html) 会产生以下输出：

```sh
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

此输出显示了 [gvinum(8)](https://man.freebsd.org/cgi/man.cgi?query=gvinum&sektion=8&format=html) 的简要列出格式。图形化表示见于 [简单的 **vinum** 卷](https://docs.freebsd.org/en/articles/vinum/#vinum-simple-vol)。

![vinum simple vol](https://docs.freebsd.org/images/articles/vinum/vinum-simple-vol.png)

**图 4. 简单的 vinum 卷**

该图以及随后出现的图形表示了卷，其中包含 plex，而 plex 又包含子磁盘。在这个例子中，卷包含一个 plex，而该 plex 包含一个子磁盘。

这个特定的卷与常规磁盘分区没有特别的优势。它包含单个 plex，因此没有冗余。该 plex 包含单个子磁盘，因此与常规磁盘分区在存储分配上没有区别。接下来的部分将展示更有趣的配置方法。

### 5.2. 增加弹性：镜像

可以通过镜像来增加卷的弹性。在布置镜像卷时，重要的是要确保每个 plex 的子磁盘位于不同的驱动器上，这样才能避免驱动器故障同时影响两个 plex。以下配置展示了镜像卷：

```sh
drive b device /dev/da4h
	volume mirror
      plex org concat
        sd length 512m drive a
	  plex org concat
	    sd length 512m drive b
```

在这个例子中，无需再次定义驱动器 *a*，因为 **vinum** 会在其配置数据库中跟踪所有对象。处理完这个定义后，配置变成了：

```sh
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

[镜像的 **vinum** 卷](https://docs.freebsd.org/en/articles/vinum/#vinum-mirrored-vol) 以图形化方式展示了结构。

![vinum mirrored vol](https://docs.freebsd.org/images/articles/vinum/vinum-mirrored-vol.png)

**图 5. 镜像的 **vinum** 卷**

在这个例子中，每个 plex 包含完整的 512 MB 地址空间。如同之前的例子，每个 plex 只包含一个子磁盘。

### 5.3. 优化性能

前面例子中的镜像卷比未镜像的卷更能抵抗故障，但其性能较差，因为每次对卷的写入都需要同时写入到两个驱动器，占用了更大比例的磁盘带宽。性能考量需要另一种方法：不使用镜像，而是将数据条带化分布到尽可能多的磁盘驱动器上。以下配置展示了条带化卷，plex 被条带化到四个磁盘驱动器上：

```sh
drive c device /dev/da5h
	drive d device /dev/da6h
	volume stripe
	plex org striped 512k
	  sd length 128m drive a
	  sd length 128m drive b
	  sd length 128m drive c
	  sd length 128m drive d
```

如之前一样，无需定义已为 **vinum** 所知的驱动器。处理完这个定义后，配置变成了：

```sh
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

**图 6. 条带化的 vinum 卷**

此卷在 [条带化的 **vinum** 卷](https://docs.freebsd.org/en/articles/vinum/#vinum-striped-vol) 中进行了表示。条带的深浅表示在 plex 地址空间中的位置，最浅的条带在前，最深的条带在后。

### 5.4. 韧性与性能

借助足够的硬件，可以构建出相较于标准 UNIX® 分区具有更高韧性和性能的卷。典型的配置文件可能如下所示：

```sh
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

第二个 plex 的子磁盘与第一个 plex 的子磁盘错开了两个驱动器。这有助于确保写入操作不会写入到相同的子磁盘，即使传输跨越了两个驱动器。

[镜像条带化的 **vinum** 卷](https://docs.freebsd.org/en/articles/vinum/#vinum-raid10-vol) 表示该卷的结构。

![vinum raid10 vol](https://docs.freebsd.org/images/articles/vinum/vinum-raid10-vol.png)

**图 7. 镜像条带化的 vinum 卷**

## 6. 对象命名

**vinum** 为 plex 和子磁盘分配默认名称，尽管可以覆盖这些名称。不建议覆盖默认名称，因为这样做不会带来显著的优势，且可能导致混淆。

名称可以包含任何非空白字符，但建议仅限字母、数字和下划线字符。卷、plex 和子磁盘的名称最长可达 64 个字符，驱动器的名称最长可达 32 个字符。

**vinum** 对象在 **/dev/gvinum** 层次结构中分配设备节点。上面的配置将导致 **vinum** 创建以下设备节点：

- 每个卷的设备条目。这些是 **vinum** 使用的主要设备。上述配置会包括设备 **/dev/gvinum/myvol**、**/dev/gvinum/mirror**、**/dev/gvinum/striped**、**/dev/gvinum/raid5** 和 **/dev/gvinum/raid10**。
- 所有卷都会在 **/dev/gvinum/** 下获得直接条目。
- 目录 **/dev/gvinum/plex** 和 **/dev/gvinum/sd**，分别包含每个 plex 和每个子磁盘的设备节点。

例如，考虑以下配置文件：

```sh
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

处理此文件后，[gvinum(8)](https://man.freebsd.org/cgi/man.cgi?query=gvinum&sektion=8&format=html) 会在 **/dev/gvinum** 中创建以下结构：

```sh
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

尽管建议不要为 plex 和子磁盘分配特定名称，但 **vinum** 驱动器必须命名。这使得在将驱动器移动到不同位置时，仍然可以自动识别它。驱动器名称最长可达 32 个字符。

### 6.1. 创建文件系统

卷在系统看来与磁盘相同，唯一的例外是，**vinum** 不对卷进行分区，因此卷中没有分区表。这要求对一些磁盘工具进行修改，特别是 [newfs(8)](https://man.freebsd.org/cgi/man.cgi?query=newfs&sektion=8&format=html)，以防它尝试将 **vinum** 卷名的最后一个字母解释为分区标识符。例如，磁盘驱动器的名称可能是 **/dev/ad0a** 或 **/dev/da2h**。这些名称分别表示第一个（0）IDE 磁盘（**ad**）上的第一个分区（**a**）和第三个（2）SCSI 磁盘（**da**）上的第八个分区（**h**）。相比之下，**vinum** 卷可能被称为 **/dev/gvinum/concat**，它与分区名称没有任何关系。

要在此卷上创建文件系统，请使用 [newfs(8)](https://man.freebsd.org/cgi/man.cgi?query=newfs&sektion=8&format=html)：

```sh
# newfs /dev/gvinum/concat
```

## 7. 配置 **vinum**

**GENERIC** 内核不包含 **vinum**。可以构建包含 **vinum** 的自定义内核，但不推荐这样做。启动 **vinum** 的标准方法是将其作为内核模块加载。因为 [gvinum(8)](https://man.freebsd.org/cgi/man.cgi?query=gvinum&sektion=8&format=html) 启动时会检查模块是否已加载，如果没有，它会自动加载模块，所以不需要使用 [kldload(8)](https://man.freebsd.org/cgi/man.cgi?query=kldload&sektion=8&format=html)。

### 7.1. 启动

**vinum** 将配置信息存储在磁盘切片中，形式与配置文件基本相同。当从配置数据库读取时，**vinum** 会识别一些在配置文件中不允许使用的关键字。例如，磁盘配置可能包含以下文本：

```sh
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

显而易见的区别是存在明确的位置和命名信息（允许但不推荐使用），以及状态信息。**vinum** 不存储关于驱动器的任何信息，它通过扫描配置的磁盘驱动器来查找具有 **vinum** 标签的分区。这使得 **vinum** 能够正确识别驱动器，即使它们被分配了不同的 UNIX® 驱动器 ID。

#### 7.1.1. 自动启动

*Gvinum* 总是能够在内核模块加载后通过 [loader.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=loader.conf&sektion=5&format=html) 实现自动启动。要在启动时加载 *Gvinum* 模块，请将 `geom_vinum_load="YES"` 添加到 **/boot/loader.conf**。

当 **vinum** 使用 `gvinum start` 启动时，**vinum** 会从某个 **vinum** 驱动器读取配置数据库。在正常情况下，每个驱动器包含配置数据库的副本，因此读取哪个驱动器并不重要。然而，在崩溃之后，**vinum** 必须确定哪个驱动器是最新更新的，并从该驱动器读取配置。然后，它会根据需要从逐渐较旧的驱动器更新配置。

## 8. 使用 **vinum** 作为根文件系统

对于使用 **vinum** 实现完全镜像文件系统的机器，镜像根文件系统也是理想的。设置这种配置比镜像任意文件系统要复杂一些，因为：

- 根文件系统必须在引导过程中很早就可用，因此 **vinum** 基础设施必须在此时已可用。
- 包含根文件系统的卷还包含系统引导程序和内核。这些必须使用主机系统的本地工具（如 BIOS）读取，而这些工具通常无法了解 **vinum** 的细节。

在以下章节中，术语“根卷”通常用来描述包含根文件系统的 **vinum** 卷。

### 8.1. 让 **vinum** 在启动时足够早地加载以支持根文件系统

为了确保 **vinum** 在系统启动时能被加载，必须让 [loader(8)](https://man.freebsd.org/cgi/man.cgi?query=loader&sektion=8&format=html) 能够在启动内核之前加载 **vinum** 内核模块。可以通过在 **/boot/loader.conf** 文件中加入以下行来实现：

```sh
geom_vinum_load="YES"
```

### 8.2. 使基于 **vinum** 的根卷对引导程序可访问

当前的 FreeBSD 引导程序只有 7.5 KB 的代码，并且不能理解 **vinum** 的内部结构。这意味着它无法解析 **vinum** 配置数据，也无法识别启动卷的元素。因此，需要一些解决方法来为引导程序提供标准 `a` 分区的假象，该分区包含根文件系统。

为了使这种情况成为可能，根卷必须满足以下要求：

- 根卷不能是条带化（stripe）或 `RAID`-5 类型。
- 根卷的每个 plex 中不能包含超过一个连接子磁盘（concatenated subdisk）。

需要注意的是，使用多个 plex（每个包含根文件系统副本）是可行且理想的。引导程序过程将只使用一个副本来查找引导程序和所有启动文件，直到内核挂载根文件系统为止。这些 plex 中的每个子磁盘需要自己的 `a` 分区假象，以便相应的设备可以引导系统。严格来说，这些伪造的 `a` 分区不必位于每个设备中的相同偏移量，但为了避免混淆，最好按这种方式创建 **vinum** 卷，使结果镜像设备是对称的。

为了为每个包含根卷部分的设备设置这些 `a` 分区，需要执行以下步骤：

1. 使用以下命令检查该设备的根卷子磁盘的位置、偏移量以及大小：

   ```sh
   # gvinum l -rv root
   ```

   **vinum** 的偏移量和大小是以字节为单位的。必须将这些值除以 512，以获得要用于 `bsdlabel` 的块号。

2. 对参与根卷的每个设备运行以下命令：

   ```sh
   # bsdlabel -e devname
   ```

   *devname* 必须是磁盘的名称，例如没有切片表的磁盘的名称 **da0**，或切片名称，例如 **ad0s1**。

   如果设备上已存在 `a` 分区（来自预 **vinum** 根文件系统），则应将其重命名为其他名称，以便它仍然可访问（以防万一），但默认情况下不再用于引导系统。当前挂载的根文件系统不能被重命名，因此必须在从 "Fixit" 媒体引导时执行此操作，或者在镜像中使用两步过程，首先操作当前未引导的磁盘。

   必须将该设备上的 **vinum** 分区的偏移量（如果有）与根卷子磁盘的偏移量相加。所得的值将成为新 `a` 分区的 `offset` 值。这个分区的 `size` 值可以直接从上面的计算中获得。`fstype` 应该是 `4.2BSD`。`fsize`、`bsize` 和 `cpg` 的值应选择与实际文件系统相匹配，尽管在这个上下文中，它们并不是特别重要。

   这样，就会建立新的 `a` 分区，它将与该设备上的 **vinum** 分区重叠。只有当 **vinum** 分区已正确标记为 `vinum` 文件系统类型时，`bsdlabel` 才会允许这种重叠。

3. 现在，所有包含根卷副本的设备上都有伪造的 `a` 分区。强烈建议使用以下命令验证结果：

   ```sh
   # fsck -n /dev/devnamea
   ```

请记住，所有包含控制信息的文件都必须相对于根文件系统，这个根文件系统在 **vinum** 卷中，在设置新的 **vinum** 根卷时，可能与当前活动的根文件系统不匹配。因此，必须特别小心处理 **/etc/fstab** 和 **/boot/loader.conf** 文件。

在下次重启时，引导程序应该能够从新的基于 **vinum** 的根文件系统获取相应的控制信息，并相应地进行操作。内核初始化过程结束时，在所有设备都已声明之后，显示的成功消息应该类似于：

```sh
Mounting root from ufs:/dev/gvinum/root
```

### 8.3. 基于 **vinum** 的根文件系统示例

在设置好 **vinum** 根卷之后，运行 `gvinum l -rv root` 的输出可能如下所示：

```sh
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

需要注意的值是 `135680`，即相对于分区 **/dev/da0h** 的偏移量。这在 `bsdlabel` 中等同于 265 个 512 字节的磁盘块。类似地，根卷的大小是 245760 个 512 字节的块。包含第二个副本的 **/dev/da1h** 设备有对称的设置。

这些设备的 `bsdlabel` 可能如下所示：

```sh
...
8 partitions:
#        size   offset    fstype   [fsize bsize bps/cpg]
  a:   245760      281    4.2BSD     2048 16384     0  # (Cyl.    0*- 15*)
  c: 71771688        0    unused        0     0        # (Cyl.    0 - 4467*)
  h: 71771672       16     vinum                       # (Cyl.    0*- 4467*)
```

可以观察到，伪造的 `a` 分区的 `size` 参数与上面计算出的值相匹配，而 `offset` 参数是 **vinum** 分区 `h` 内的偏移量与该分区在设备或切片中的偏移量之和。这是避免 [什么也没启动，引导程序崩溃](https://docs.freebsd.org/en/articles/vinum/#vinum-root-panic) 中描述的问题的典型设置。整个 `a` 分区完全位于包含所有 **vinum** 数据的 `h` 分区内。

在上面的示例中，整个设备专门用于 **vinum**，并且没有剩余的预 **vinum** 根分区。

### 8.4. 故障排除

以下是一些已知的常见问题和解决方法。

#### 8.4.1. 系统引导加载，但系统无法启动

如果由于某种原因系统无法继续启动，可以通过在 10 秒警告时按下空格键来中断引导程序。可以通过输入 `show` 来检查引导程序变量 `vinum.autostart`，并使用 `set` 或 `unset` 进行修改。

如果 **vinum** 内核模块尚未列在自动加载的模块列表中，可以输入 `load geom_vinum` 来加载它。

加载完成后，可以输入 `boot -as` 继续启动过程，`-as` 参数请求内核询问要挂载的根文件系统（`-a`），并使启动过程在单用户模式下停止（`-s`），此时根文件系统以只读方式挂载。这样，即使只有一个 plex 的多 plex 卷被挂载，也不会导致 plex 之间的数据不一致。

在提示输入根文件系统时，可以输入任何包含有效根文件系统的设备。如果 **/etc/fstab** 配置正确，默认值应该类似于 `ufs:/dev/gvinum/root`。典型的替代选择可能是 `ufs:da0d`，它可能是包含预 **vinum** 根文件系统的假设分区。如果此处输入的是某个别名 `a` 分区，必须确保它实际引用的是 **vinum** 根设备的子磁盘，因为在镜像设置中，这只会挂载一个镜像根设备的一部分。如果此文件系统以后要以读写方式挂载，则需要移除其他 plex，因为这些 plex 会携带不一致的数据。

#### 8.4.2. 只加载了主引导程序

如果 **/boot/loader** 无法加载，但主引导程序仍然加载（在启动过程开始后屏幕的左列可见一个破折号），可以尝试通过按空格键中断主引导程序。这样可以使引导程序停在 [第二阶段](https://docs.freebsd.org/en/books/handbook/#boot-boot1) 进行操作。在此阶段，可以尝试从备用分区启动，例如包含先前根文件系统的分区。

#### 8.4.3. 什么也没启动，引导程序崩溃

如果引导程序在 **vinum** 安装后被破坏，就会发生这种情况。不幸的是，**vinum** 会在其分区的开始处意外地留下 4 KB 的空闲空间，然后才开始写入 **vinum** 头信息。然而，第一阶段和第二阶段的引导程序以及 `bsdlabel` 需要 8 KB。因此，如果 **vinum** 分区从 0 偏移量开始，且该分区位于本应可启动的切片或磁盘中，**vinum** 设置将破坏引导程序。

同样，如果通过从 "Fixit" 媒体启动并使用 `bsdlabel -B` 重新安装引导程序来恢复上述情况，重新安装的引导程序会破坏 **vinum** 头，导致 **vinum** 无法找到其磁盘。尽管实际的 **vinum** 配置数据和 **vinum** 卷中的数据不会被破坏，并且可以通过重新输入相同的 **vinum** 配置数据来恢复所有数据，但这种情况很难修复。需要将整个 **vinum** 分区移动至少 4 KB，以避免 **vinum** 头和系统引导程序的冲突。
