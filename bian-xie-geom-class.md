# 编写一个 GEOM 类

<details open="" data-immersive-translate-walked="5c754e06-3850-404e-bd21-b334a3372e1f"><summary data-immersive-translate-walked="5c754e06-3850-404e-bd21-b334a3372e1f" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Intel，Celeron，Centrino，Core，EtherExpress，i386，i486，Itanium，Pentium 和 Xeon 是 Intel Corporation 或其在美国及其他国家的子公司的商标或注册商标。

制造商和销售商用来区分其产品的许多名称被视为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目意识到商标声明，这些名称后面将跟随“™”或“®”符号。

</details>

摘要

本文档记录了开发 GEOM 类和内核模块的一些起点，以及一般情况下内核模块的开发。假定读者熟悉 C 用户空间编程。

---

## 1. 介绍

### 1.1. 文档

内核编程的文档很少 - 这是很少有友好教程的领域之一，而短语“查看源代码！”确实是正确的。然而，在开始编写代码之前，应该学习一些零散的（有些严重过时的）内容。

- FreeBSD 开发者手册 - 作为文档项目的一部分，不包含任何与内核编程相关的特定内容，而是一些一般有用的信息。
- FreeBSD 架构手册 - 也来自文档项目，包含对几个低级设施和过程的描述。最重要的章节是第 13 章，编写 FreeBSD 设备驱动程序。
- FreeBSD Diary 网站的蓝图部分 - 包含有关内核设施的几篇有趣文章。
- 内核函数的重要文档位于第 9 节的 man 页面。
- geom(4)的 man 页面和 PHK 的 GEOM 幻灯片 - 用于介绍 GEOM 子系统的概况。
- g_bio(9)、g_event(9)、g_data(9)、g_geom(9)、g_provider(9)、g_consumer(9)、g_access(9)等与这些页面链接的其他页面，用于特定功能的文档。
- 样式（9）手册 - 有关必须遵循的编码风格约定的文档，以便提交到 FreeBSD 树中的任何代码。

## 2. 准备工作

进行内核开发的最佳方法是拥有（至少）两台独立的计算机。其中一台包含开发环境和源代码，另一台用于通过网络引导和从第一台网络挂载文件系统来测试新编写的代码。这样，如果新代码包含错误并使机器崩溃，也不会弄乱源代码（和其他“活”数据）。第二个系统甚至不需要正确的显示。相反，它可以通过串行电缆或 KVM 连接到第一台计算机。

但是，由于并非每个人都有两台或更多台计算机可用，因此有一些可以做的事情来为开发内核代码做好准备。这种设置也适用于在 VMWare 或 QEmu 虚拟机中开发（在专用开发机之后是第二好的选择）。

### 2.1. 修改系统以进行开发

对于任何内核编程，必须启用 INVARIANTS 内核。因此，请在内核配置文件中输入以下内容：

```
options INVARIANT_SUPPORT
options INVARIANTS
```

对于更多的调试，您还应该包括 WITNESS 支持，这将在锁定错误时通知您：

```
options WITNESS_SUPPORT
options WITNESS
```

对于调试崩溃转储，需要带有调试符号的内核：

```
  makeoptions    DEBUG=-g
```

通过通常安装内核的方式（ make installkernel ），调试内核不会自动安装。它被称为 kernel.debug，位于/usr/obj/usr/src/sys/KERNELNAME/。为方便起见，应该将其复制到/boot/kernel/。

另一个方便之处是启用内核调试器，这样您就可以在内核发生 panic 时对其进行检查。为此，请在内核配置文件中输入以下行：

```
options KDB
options DDB
options KDB_TRACE
```

为了使其工作，您可能需要设置一个 sysctl（如果默认情况下不打开）：

```
  debug.debugger_on_panic=1
```

内核 panic 会发生，因此在文件系统缓存方面需要小心。特别是，如果发生 panic 时尚未将最新文件版本提交到存储中，那么可能会丢失最新文件版本。禁用 softupdates 会造成很大的性能损失，但仍无法保证数据一致性。为此需使用带有"sync"选项的挂载文件系统。为了在性能和数据一致性之间取得平衡，可以缩短 softupdates 缓存延迟。有三个 sysctl 非常有用（最好设置在 /etc/sysctl.conf 中）。

```
kern.filedelay=5
kern.dirdelay=4
kern.metadelay=3
```

这些数字代表秒数。

对于调试内核崩溃，需要内核核心转储。由于内核崩溃可能导致文件系统不可用，因此此崩溃转储首先写入原始分区。通常，这是交换分区。此分区的大小必须至少与机器中的物理 RAM 一样大。在下次启动时，转储将被复制到常规文件中。这发生在文件系统被检查和挂载之后，交换被启用之前。这由两个/etc/rc.conf 变量控制：

```
dumpdev="/dev/ad0s4b"
dumpdir="/usr/core
```

dumpdev 变量指定交换分区， dumpdir 告诉系统在重启时将核心转储重新定位到文件系统的哪个位置。

写入内核核心转储很慢，需要很长时间，因此，如果您有大量内存（>256M）和大量的 panic ，坐在一边等待完成可能会很沮丧（首先将其写入交换空间，然后将其重新定位到文件系统）。因此，通过/boot/loader.conf 可调节限制系统将使用的 RAM 量是很方便的：

```
  hw.physmem="256M"
```

如果 panic 频繁发生且文件系统较大（或者您根本不信任 softupdates+后台 fsck），建议通过/etc/rc.conf 变量关闭后台 fsck：

```
  background_fsck="NO"
```

这样，文件系统将始终在需要时进行检查。请注意，使用后台 fsck 时，可能会在检查磁盘时发生新的 panic 。同样，最安全的方法是不要通过使用另一台计算机作为 NFS 服务器来拥有许多本地文件系统。

### 2.2. 启动项目

为了创建一个新的 GEOM 类，必须在任意用户可访问的目录下创建一个空的子目录。您不必在/usr/src 下创建模块目录。

### 2.3. Makefile

为每个非平凡编程项目创建 Makefile 是一个很好的实践，当然也包括内核模块。

由于系统提供了一套广泛的辅助程序例程，创建 Makefile 变得很简单。简而言之，这是一个用于内核模块的最小 Makefile 的外观：

```
SRCS=g_journal.c
KMOD=geom_journal

.include <bsd.kmod.mk>
```

这个 Makefile（带有更改后的文件名）适用于任何内核模块，一个 GEOM 类可以仅驻留在一个内核模块中。如果需要多个文件，请在 SRCS 变量中列出，用空格与其他文件名分隔。

## 3. 在 FreeBSD 内核编程

### 3.1. 内存分配

请参阅 malloc(9)。基本内存分配与其用户空间等效略有不同。特别是， malloc ()和 free ()接受额外参数，如手册中所述。

在源文件的声明部分必须声明"malloc 类型"，就像这样：

```
  static MALLOC_DEFINE(M_GJOURNAL, "gjournal data", "GEOM_JOURNAL Data");
```

要使用此宏，必须包含 sys/param.h、sys/kernel.h 和 sys/malloc.h 头文件。

还有另一种分配内存的机制，即 UMA（通用内存分配器）。有关详情，请参阅 uma(9)，但它是一种特殊类型的分配器，主要用于快速分配由相同大小项目组成的列表（例如，结构动态数组）。

### 3.2. 列表和队列

查看 queue(3)。有很多情况需要维护一组事物的列表。幸运的是，这种数据结构由系统中包含的 C 宏（以几种方式）实现。最常用的列表类型是 TAILQ，因为它是最灵活的。它也是内存需求最大的一个（其元素是双向链接的），也是最慢的（尽管速度变化在几个 CPU 指令的数量级上，所以不应该认真对待）。

如果数据检索速度非常重要，请参阅 tree(3) 和 hashinit(9)。

### 3.3. BIOs

结构 bio 用于所有与 GEOM 相关的输入/输出操作。它基本上包含有关哪个设备（“提供者”）应满足请求、请求类型、偏移量、长度、指向缓冲区的指针以及一堆“用户特定”标志和字段的信息，这些标志和字段可以帮助实现各种黑客技术。

这里重要的是 bio 是异步处理的。这意味着，在代码的大多数部分中，没有类似于用户态 read(2) 和 write(2) 调用的操作，直到请求完成才返回。而是当请求完成（或出错）时，调用一个开发者提供的函数作为通知。

异步编程模型（也称为“事件驱动”）比在用户界面中使用的更广泛的命令式编程模型要稍微困难一些（至少需要一段时间才能适应）。在某些情况下，可以使用辅助函数 g_write_data （）和 g_read_data （）。但并非总是如此。特别是在持有互斥锁时，例如，不能在保持 GEOM 拓扑互斥锁或在执行 .start （）和 .stop （）函数期间保持的内部互斥锁时，它们不能被使用。

## 4. 关于 GEOM 编程

### 4.1. Ggate

如果不需要最大性能，通过实现用户区域的数据转换方式，可以更简单地通过 ggate（GEOM gate）设施来实现。不幸的是，目前没有简单的方法在这两种方法之间进行转换，甚至无法共享代码。

### 4.2. GEOM 类

GEOM 类是数据的转换。这些转换可以以类似树状的方式组合在一起。GEOM 类的实例被称为 geoms。

每个 GEOM 类都有几个“类方法”，当没有 geom 实例可用时（或它们根本没有绑定到单个实例）时会调用这些方法：

- 当 GEOM 意识到 GEOM 类时调用 .init （当内核模块加载时）。
- 当 GEOM 放弃该类时调用 .fini （当模块卸载时）。
- .taste 被称为接下来，对系统每个可用的提供程序调用一次。如果适用，此函数通常会创建并启动一个 geom 实例。
- .destroy_geom 在应该解散 geom 时被调用
- .ctlconf 在用户请求重新配置现有 geom 时被调用

还定义了将复制到 geom 实例的 GEOM 事件函数。

g_class 结构中的字段 .geom 是从类实例化的 geoms 的 LIST。

这些函数是从 g_event 内核线程调用的。

### 4.3. Softc

"softc" 这个名字是“驱动程序私有数据”的遗留术语。这个名字很可能来自古老的术语“软件控制块”。在 GEOM 中，它是一个结构体（更确切地说：指向结构体的指针），可以附加到一个 geom 实例上以保存该 geom 实例的私有数据。大多数 GEOM 类有以下成员：

- struct g_provider \*provider ：该 geom 实例化的“provider”
- uint16_t n_disks ：此几何体消耗的消费者数量
- struct g_consumer **disks ： struct g_consumer**​*的数组。（无法仅使用单个间接引用，因为 struct g_consumer* 是由 GEOM 代表我们创建的）。

softc 结构包含几何体实例的所有状态。每个几何体实例都有自己的 softc。

### 4.4. 元数据

元数据的格式在很大程度上取决于类，但必须始于：

- 16 字节缓冲区用于空终结签名（通常是类名）
- uint32 版本 ID

假定 geom 类知道如何处理低于其版本 ID 的元数据。

元数据位于提供程序的最后一个扇区中（因此必须适合其中）。

（所有这些是依赖实现的，但所有现有代码都是这样工作的，并且受到库的支持。）

### 4.5. 标记/创建 GEOM

事件的顺序是：

- 用户调用 geom（8）实用程序（或其硬链接朋友之一）
- 实用程序确定它应该处理的 geom 类，并搜索 geom_CLASSNAME.so 库（通常位于/lib/geom）。
- 它 dlopen（3）加载库，提取命令行参数和辅助函数的定义。

在创建/标记新几何体时，会发生以下情况：

- geom(8)在命令行参数中查找命令（通常为 label ），然后调用一个辅助函数。
- 辅助函数检查参数并收集元数据，然后将其写入所有相关提供者。
- 这会“破坏”现有的 geoms（如果有的话），并初始化提供者的新一轮“品尝”。预期的 geom 类将识别元数据并将 geom 带上。

（上述事件序列依赖于实现，但所有现有代码都按照这种方式工作，并由库支持。）

### 4.6. GEOM 命令结构

helper geom_CLASSNAME.so 库导出 class_commands 结构，这是一个包含 struct g_command 元素的数组。命令具有统一的格式，如下所示：

```
  verb [-options] geomname [other]
```

常见动词有：

- label - 将元数据写入设备，以便它们在 tasting 时被识别并在 geoms 中启动。
- 摧毁 - 摧毁元数据，使几何体被摧毁

常见选项是：

- -v ：详细输出
- -f ：强制

许多操作，如标记和销毁元数据，可以在用户空间中执行。为此， struct g_command 提供了字段 gc_func ，可以设置为一个函数（在同一个.so 中），该函数将被调用以处理一个动作。如果 gc_func 为 NULL，则该命令将传递给内核模块，传递给 geom 类的 .ctlreq 函数。

### 4.7. 几何体

Geoms 是 GEOM 类的实例。它们具有内部数据（softc 结构）和一些函数，用于响应外部事件。

事件函数包括：

- .access ：计算权限（读/写/独占）
- .dumpconf ：返回有关 geom 的 XML 格式信息
- .orphan ：当某些底层提供程序断开连接时调用
- .spoiled ：当某些底层提供程序被写入时调用
- .start ：处理 I/O

这些函数是从 g_down 内核线程调用的，因此在这种情况下不能睡眠（请参阅其他地方的睡眠定义），这在很大程度上限制了可以做的事情，但迫使处理变得更快。

在这些函数中，用于执行实际有用工作的最重要函数是 .start ()函数，当为 geom 类的实例管理的提供程序到达 BIO 请求时调用该函数。

### 4.8. GEOM 线程

GEOM 框架创建并运行三个内核线程：

- g_down ：处理来自高级实体（如用户空间请求）到物理设备的请求
- g_up ：处理来自设备驱动程序对高级实体发出的请求的响应
- g_event ：处理所有其他情况：创建 geom 实例，访问计数，“损坏”事件等

当用户进程发出“读取文件中偏移量为 Y 的数据 X”请求时，会发生以下情况：

- 文件系统将请求转换为结构体 bio 实例，并将其传递给 GEOM 子系统。它知道应该由哪个 geom 实例处理它，因为文件系统直接托管在 geom 实例上。
- 请求最终成为在 g_down 线程上调用 .start () 函数，并达到顶层 geom 实例。
- 顶层 geom 实例（例如分区切片器）确定请求应该路由到较低级别实例（例如磁盘驱动器）。它复制了 bio 请求（bio 请求在实例之间始终需要复制，使用 g_clone_bio ()！），修改了数据偏移和目标提供程序字段，并使用 g_io_request () 执行复制。
- 磁盘驱动程序还将生物请求作为对 .start () 的调用传递给 g_down 线程。它与硬件通信，获取数据，并在生物上调用 g_io_deliver (）。
- 现在，生物完成的通知在 g_up 线程中“冒泡”上升。首先，分区切片器在 g_up 线程中调用 .done ()，它使用存储在生物中的信息来释放克隆的 bio 结构（带有 g_destroy_bio ()），并在原始请求上调用 g_io_deliver ()。
- 文件系统获取数据并将其传输到用户空间。

查看 g_bio(9) 手册以获取有关如何在 bio 结构中来回传递数据的信息（特别注意 bio_parent 和 bio_children 字段以及它们如何处理）。

一个重要的特点是：G_UP 和 G_DOWN 线程中不能休眠。这意味着这些线程中不能做以下任何事情（当然列表并不完整，仅供参考）：

- 显然不允许调用 msleep () 和 tsleep ()。
- 调用 g_write_data () 和 g_read_data ()，因为在将数据传递给消费者并返回之间会休眠。
- 等待 I/O。
- 调用 malloc(9) 和 uma_zalloc ()，设置 M_WAITOK 标志。
- sx 和其他可睡眠锁

此限制是为了阻止 GEOM 代码堵塞 I/O 请求路径，因为睡眠通常没有时间限制，无法保证需要多长时间（还有一些其他更技术性的原因）。这也意味着在这些线程中几乎无法做很多事情；例如，几乎任何复杂的事情都需要内存分配。幸运的是，有一个解决办法：创建额外的内核线程。

### 4.9. 在 GEOM 代码中使用内核线程

内核线程是使用 kthread_create(9)函数创建的，它们在行为上有点类似于用户态线程，只是它们不能返回给调用者以表示终止，而必须调用 kthread_exit(9)。

在 GEOM 代码中，线程的通常用途是将请求的处理从 g_down 线程（ .start （）函数）卸载。这些线程看起来像是“事件处理程序”：它们有一个与之关联的事件链表（可以由各种函数在各种线程中发布事件，因此必须受互斥体保护），逐个从列表中获取事件并在一个大的 switch （）语句中处理它们。

使用线程处理 I/O 请求的主要好处是它可以在需要时睡眠。现在，这听起来很好，但应该仔细考虑。睡眠很方便，但可能会有效地破坏 geom 变换的性能。极度注重性能的类可能应该在 .start （）函数调用中做所有工作，非常小心地处理内存不足和类似错误。

像这样拥有事件处理程序线程的另一个好处是将来自不同 geom 线程的所有请求和响应串行化到一个线程中。这也非常方便，但可能会很慢。在大多数情况下，处理 .done （）请求可以交给 g_up 线程处理。

FreeBSD 内核中的互斥锁（参见 mutex(9)）与其更常见的用户空间表亲有一个区别 - 在持有互斥锁时代码不能休眠）。如果代码需要频繁休眠，sx(9)锁可能更合适。另一方面，如果您几乎所有操作都在单个线程中完成，您可能根本不需要互斥锁。
