# 编写 GEOM 类

- 原文：[Writing a GEOM Class](https://docs.freebsd.org/en/articles/geom-class/)

## 摘要

本文记录了开发 GEOM 类和内核模块的一些起点。假设读者已经熟悉 C 用户空间编程。

## 1. 引言

### 1.1. 文档

内核编程的文档较为匮乏，这是少数几个几乎没有友好教程的领域之一，而“使用源码！”这句话也确实适用。然而，还是有一些零散的资源（其中一些已经严重过时），在开始编码之前应该进行学习：

- [FreeBSD 开发者手册](https://docs.freebsd.org/en/books/developers-handbook/) - 这是文档项目的一部分，虽然不包含与内核编程相关的内容，但提供了一些通用的有用信息。
- [FreeBSD 架构手册](https://docs.freebsd.org/en/books/arch-handbook/) - 同样是文档项目的一部分，包含了多个低级功能和程序的描述。最重要的章节是第 13 章，[编写 FreeBSD 设备驱动程序](https://docs.freebsd.org/en/books/arch-handbook/#driverbasics)。
- [FreeBSD Diary](http://www.freebsddiary.org/) 网站上的蓝图部分 - 包含了多篇关于内核功能的有趣文章。
- 第 9 节的手册页 - 提供内核函数的重要文档。
- [geom(4)](https://man.freebsd.org/cgi/man.cgi?query=geom&sektion=4&format=html) 手册页和 [PHK 的 GEOM 幻灯片](http://phk.freebsd.dk/pubs/) - 提供 GEOM 子系统的一般介绍。
- 手册页 [g_bio(9)](https://man.freebsd.org/cgi/man.cgi?query=g_bio&sektion=9&format=html)、[g_event(9)](https://man.freebsd.org/cgi/man.cgi?query=g_event&sektion=9&format=html)、[g_data(9)](https://man.freebsd.org/cgi/man.cgi?query=g_data&sektion=9&format=html)、[g_geom(9)](https://man.freebsd.org/cgi/man.cgi?query=g_geom&sektion=9&format=html)、[g_provider(9)](https://man.freebsd.org/cgi/man.cgi?query=g_provider&sektion=9&format=html)、[g_consumer(9)](https://man.freebsd.org/cgi/man.cgi?query=g_consumer&sektion=9&format=html)、[g_access(9)](https://man.freebsd.org/cgi/man.cgi?query=g_access&sektion=9&format=html) 等，提供有关特定功能的文档。
- [style(9)](https://man.freebsd.org/cgi/man.cgi?query=style&sektion=9&format=html) 手册页 - 提供有关编码风格的文档，所有提交到 FreeBSD 树的代码必须遵循这些风格。

## 2. 准备工作

进行内核开发的最佳方式是拥有（至少）两台独立的计算机。其中一台包含开发环境和源代码，另一台用于通过网络启动并通过网络从第一台计算机挂载文件系统来测试新编写的代码。这样，如果新代码包含错误并导致机器崩溃，它将不会破坏源代码（和其他“实时”数据）。第二台计算机甚至不需要显示器。它可以通过串行电缆或 KVM 连接到第一台计算机。

然而，由于并非每个人都有两台或更多计算机，因此可以采取一些措施来准备一个“实时”系统进行内核代码开发。此设置也适用于在 [VMWare](http://www.vmware.com/) 或 [QEmu](http://www.qemu.org/) 虚拟机中进行开发（这是除专用开发机外的下一个最佳选择）。

### 2.1. 为开发修改系统

对于任何内核编程，启用 `INVARIANTS` 的内核是必须的。因此，在内核配置文件中添加以下内容：

```sh
options INVARIANT_SUPPORT
options INVARIANTS
```

为了进行更多调试，你还应当包括 `WITNESS` 支持，它会提醒你在锁定方面的错误：

```sh
options WITNESS_SUPPORT
options WITNESS
```

为了调试崩溃转储，需要一个带有调试符号的内核：

```sh
makeoptions    DEBUG=-g
```

使用常规的安装内核方式（`make installkernel`），调试内核不会被自动安装。它被称为 **kernel.debug**，并位于 **/usr/obj/usr/src/sys/KERNELNAME/**。为了方便，它应当被复制到 **/boot/kernel/**。

另一个便利功能是启用内核调试器，这样你可以在内核崩溃时进行检查。为此，请在内核配置文件中添加以下行：

```sh
options KDB
options DDB
options KDB_TRACE
```

为了使其生效，你可能需要设置一个 `sysctl`（如果它默认未开启）：

```sh
debug.debugger_on_panic=1
```

内核崩溃是不可避免的，因此需要注意文件系统缓存。特别是，启用软更新可能意味着，如果在提交到存储之前发生崩溃，最新的文件版本可能会丢失。禁用软更新会导致性能显著下降，并且仍然无法保证数据一致性。需要使用 `sync` 选项挂载文件系统。作为折衷，可以缩短软更新缓存的延迟。有三个 `sysctl` 对此非常有用（最好在 **/etc/sysctl.conf** 中设置）：

```sh
kern.filedelay=5
kern.dirdelay=4
kern.metadelay=3
```

这些数字表示秒数。

为了调试内核崩溃，内核核心转储是必需的。由于内核崩溃可能使文件系统无法使用，因此该崩溃转储首先写入一个原始分区。通常，这是交换分区。此分区必须至少与机器的物理内存大小相同。在下次启动时，转储将被复制到常规文件中。这发生在文件系统被检查和挂载之后，交换区启用之前。这个过程通过两个 **/etc/rc.conf** 变量控制：

```sh
dumpdev="/dev/ad0s4b"
dumpdir="/usr/core"
```

`dumpdev` 变量指定了交换分区，`dumpdir` 告诉系统在重启时将核心转储移动到文件系统的哪个位置。

写入内核核心转储是非常缓慢的，并且需要很长时间，所以如果你有大量内存（>256M）并且经常遇到崩溃，那么等待它完成可能会令人沮丧（两次 - 首先写入交换分区，然后将其移到文件系统）。因此，限制系统将使用的内存量会很方便，可以通过 **/boot/loader.conf** 调整：

```sh
hw.physmem="256M"
```

如果崩溃频繁发生且文件系统较大（或者你根本不信任软更新+后台 fsck），建议通过 **/etc/rc.conf** 变量禁用后台 fsck：

```sh
background_fsck="NO"
```

这样，文件系统将在需要时始终进行检查。请注意，启用后台 fsck 时，在检查磁盘时可能会发生新的崩溃。再次强调，最安全的做法是使用另一台计算机作为 NFS 服务器，而不是拥有多个本地文件系统。

### 2.2. 启动项目

为了创建一个新的 GEOM 类，必须在一个任意用户可访问的目录下创建一个空子目录。你不必在 **/usr/src** 下创建模块目录。

### 2.3. Makefile

为每个非平凡的编码项目（当然包括内核模块）创建 **Makefile** 是一种良好的实践。

由于系统提供了一套完善的辅助例程，创建 **Makefile** 非常简单。简而言之，下面是一个内核模块的最小 **Makefile**：

```sh
SRCS=g_journal.c
KMOD=geom_journal

.include <bsd.kmod.mk>
```

这个 **Makefile**（文件名可以更改）适用于任何内核模块，并且一个 GEOM 类可以仅驻留在一个内核模块中。如果需要多个文件，只需在 `SRCS` 变量中列出它们，文件名之间用空格分隔。

## 3. FreeBSD 内核编程

### 3.1. 内存分配

请参见 [malloc(9)](https://man.freebsd.org/cgi/man.cgi?query=malloc&sektion=9&format=html)。基本的内存分配与用户空间中的类似，只是略有不同。最显著的不同是，`malloc()` 和 `free()` 会接受额外的参数，详细信息可以参考手册页。

必须在源文件的声明部分声明 "malloc 类型"，例如：

```sh
static MALLOC_DEFINE(M_GJOURNAL, "gjournal data", "GEOM_JOURNAL Data");
```

要使用此宏，必须包含 **sys/param.h**、**sys/kernel.h** 和 **sys/malloc.h** 头文件。

还有另一种内存分配机制，称为 UMA（通用内存分配器）。有关详细信息，请参见 [uma(9)](https://man.freebsd.org/cgi/man.cgi?query=uma&sektion=9&format=html)，但它是一种特殊类型的分配器，主要用于快速分配由相同大小项组成的列表（例如，结构体的动态数组）。

### 3.2. 列表和队列

请参见 [queue(3)](https://man.freebsd.org/cgi/man.cgi?query=queue&sektion=3&format=html)。在很多情况下，需要维护一个事物列表。幸运的是，系统通过 C 宏提供了几种数据结构来实现这些列表。最常用的列表类型是 TAILQ，因为它最灵活。它也是内存需求最大的（它的元素是双向链接的），并且速度最慢（尽管速度差异仅在几个 CPU 指令的数量级，因此不应过于在意）。

如果数据检索速度非常重要，可以参考 [tree(3)](https://man.freebsd.org/cgi/man.cgi?query=tree&sektion=3&format=html) 和 [hashinit(9)](https://man.freebsd.org/cgi/man.cgi?query=hashinit&sektion=9&format=html)。

### 3.3. BIO

结构 `bio` 用于所有与 GEOM 相关的输入/输出操作。它基本上包含有关哪个设备（“提供者”）应满足请求、请求类型、偏移量、长度、缓冲区指针以及一些“用户特定”的标志和字段的信息，这些字段有助于实现各种技巧。

这里重要的一点是，`bio` 是异步处理的。意味着在大多数代码中，没有类似用户空间的 [read(2)](https://man.freebsd.org/cgi/man.cgi?query=read&sektion=2&format=html) 和 [write(2)](https://man.freebsd.org/cgi/man.cgi?query=write&sektion=2&format=html) 调用，这些调用会一直等待直到请求完成。相反，当请求完成（或者出错）时，会调用开发者提供的函数作为通知。

异步编程模型（也称为“事件驱动”）比用户空间中更常用的命令式编程模型要困难一些（至少需要一段时间才能适应）。在某些情况下，可以使用辅助例程 `g_write_data()` 和 `g_read_data()`，但 *并不总是*。特别是当持有互斥锁时，它们无法使用；例如，在 GEOM 拓扑互斥锁或者在 `.start()` 和 `.stop()` 函数执行期间持有的内部互斥锁下。

## 4. GEOM 编程

### 4.1. Ggate

如果不需要最高性能，一种更简单的数据转换方法是通过 ggate（GEOM 门）设施在用户空间实现。遗憾的是，两种方法之间没有简单的转换方式，甚至没有共享代码的方式。

### 4.2. GEOM 类

GEOM 类是对数据的变换。这些变换可以以树状结构组合在一起。GEOM 类的实例称为 *geoms*。

每个 GEOM 类都有几个“类方法”，这些方法在没有可用 geom 实例时（或者它们未绑定到单个实例时）会被调用：

- `.init` 在 GEOM 意识到一个 GEOM 类时调用（当内核模块被加载时）。
- `.fini` 在 GEOM 放弃该类时调用（当模块被卸载时）。
- `.taste` 会被调用一次，每次针对系统可用的每个提供者。如果适用，这个函数通常会创建并启动一个 geom 实例。
- `.destroy_geom` 在 geom 应该被解散时调用。
- `.ctlconf` 在用户请求重新配置现有 geom 时调用。

还定义了 GEOM 事件函数，这些函数会被复制到 geom 实例中。

`g_class` 结构中的 `.geom` 字段是一个从类实例化的 geoms 列表。

这些函数是从 `g_event` 内核线程中调用的。

### 4.3. Softc

"softc" 这个名称是 "driver private data" 的遗留术语，最有可能来源于过时的术语 "software control block"。在 GEOM 中，它是一个结构体（更准确地说，是指向一个结构体的指针），可以附加到 geom 实例上，用来存储 geom 实例私有的数据。大多数 GEOM 类具有以下成员：

- `struct g_provider *provider`：这个 geom 实例化的“提供者”
- `uint16_t n_disks`：这个 geom 消耗的消费者数量
- `struct g_consumer **disks`：`struct g_consumer*` 的数组。（因为 `struct g_consumer*` 是由 GEOM 为我们创建的，所以无法使用单一间接指针）

`softc` 结构体包含了 geom 实例的所有状态。每个 geom 实例都有自己的 softc。

### 4.4. 元数据

元数据的格式多少依赖于类，但必须以以下内容开始：

- 16 字节的缓冲区，用于存储以 null 结尾的签名（通常是类名）
- uint32 版本 ID

假设 geom 类能够处理低于其版本 ID 的元数据。

元数据位于提供者的最后一个扇区（因此必须适合它）。

（所有这些都是实现相关的，但现有的代码都以此方式工作，并且得到了库的支持。）

### 4.5. 标记/创建 GEOM

事件的顺序是：

- 用户调用 [geom(8)](https://man.freebsd.org/cgi/man.cgi?query=geom&sektion=8&format=html) 实用程序（或其硬链接的其他工具）
- 实用程序确定它应该处理的 GEOM 类，并搜索 **geom_CLASSNAME.so** 库（通常在 **/lib/geom** 中）。
- 它使用 [dlopen(3)](https://man.freebsd.org/cgi/man.cgi?query=dlopen&sektion=3&format=html) 加载库，提取命令行参数和辅助函数的定义。

在创建/标记一个新的 geom 时，发生的步骤如下：

- [geom(8)](https://man.freebsd.org/cgi/man.cgi?query=geom&sektion=8&format=html) 在命令行参数中查找命令（通常是 `label`），并调用辅助函数。
- 辅助函数检查参数并收集元数据，随后将其写入所有相关提供者。
- 这会“破坏”现有的 geoms（如果有的话），并初始化新一轮对提供者的“尝试”。目标 geom 类会识别元数据并启动 geom。

（上述事件序列是实现相关的，但现有的代码都按照这种方式工作，并且得到了库的支持。）

### 4.6. GEOM 命令结构

辅助 **geom_CLASSNAME.so** 库导出 `class_commands` 结构体，它是 `struct g_command` 元素数组。命令具有统一的格式，看起来像：

```sh
verb [-options] geomname [other]
```

常见的动词有：

- label - 将元数据写入设备，以便它们可以在尝试时被识别并在 geom 中启动
- destroy - 销毁元数据，以便 geoms 被销毁

常见的选项有：

- `-v`：详细模式
- `-f`：强制执行

许多操作，如标记和销毁元数据，可以在用户空间中执行。为此，`struct g_command` 提供了 `gc_func` 字段，可以设置为一个函数（位于同一个 **.so** 文件中），该函数将被调用来处理动词。如果 `gc_func` 为 NULL，命令将传递给内核模块，调用 geom 类的 `.ctlreq` 函数。

### 4.7. Geoms

Geoms 是 GEOM 类的实例。它们有内部数据（一个 softc 结构）以及一些函数，用于响应外部事件。

事件函数包括：

- `.access`：计算权限（读取/写入/独占）
- `.dumpconf`：返回关于 geom 的 XML 格式信息
- `.orphan`：当某个底层提供者被断开连接时调用
- `.spoiled`：当某个底层提供者被写入时调用
- `.start`：处理 I/O

这些函数是从 `g_down` 内核线程中调用的，并且在这个上下文中不能进行睡眠操作（参见其他地方对睡眠的定义），这相当大程度上限制了可以做的事情，但也强制要求处理速度要快。

其中，最重要的函数是 `.start()`，它在收到针对 geom 类实例管理的提供者的 BIO 请求时被调用，用于执行实际的有用工作。

### 4.8. GEOM 线程

GEOM 框架创建并运行了三个内核线程：

- `g_down`：处理来自高级实体（如用户空间请求）到物理设备的请求
- `g_up`：处理来自设备驱动程序的响应，这些响应是对高级实体发出的请求的回答
- `g_event`：处理所有其他情况：geom 实例的创建、访问计数、“损坏”事件等

当一个用户进程发出“读取文件中偏移量 Y 处的数据 X”请求时，发生的过程如下：

- 文件系统将请求转换为一个 `struct bio` 实例，并将其传递给 GEOM 子系统。它知道哪个 geom 实例应该处理该请求，因为文件系统直接托管在 geom 实例上。
- 请求最终会调用 `g_down` 线程中的 `.start()` 函数，并到达顶层 geom 实例。
- 这个顶层 geom 实例（例如分区切片器）确定该请求应该路由到较低级别的实例（例如磁盘驱动程序）。它会复制该 bio 请求（bio 请求 *总是* 需要在实例间复制，使用 `g_clone_bio()`！），修改数据偏移量和目标提供者字段，并使用 `g_io_request()` 执行复制。
- 磁盘驱动程序同样会收到一个 bio 请求，作为对 `g_down` 线程中 `.start()` 的调用。它与硬件通信，获取数据并调用 `g_io_deliver()` 以完成该 bio 请求。
- 现在，bio 完成的通知在 `g_up` 线程中“上浮”。首先，分区切片器会在 `g_up` 线程中调用 `.done()`，它使用 bio 中存储的信息来释放克隆的 `bio` 结构（使用 `g_destroy_bio()`），并在原始请求上调用 `g_io_deliver()`。
- 文件系统获取数据并将其传输到用户空间。

请参阅 [g_bio(9)](https://man.freebsd.org/cgi/man.cgi?query=g_bio&sektion=9&format=html) 手册页，了解数据如何在 `bio` 结构中来回传递（特别注意 `bio_parent` 和 `bio_children` 字段，以及它们如何处理）。

一个重要的特点是：**在 G_UP 和 G_DOWN 线程中不能进行睡眠操作**。这意味着不能在这些线程中执行以下任何操作（这个列表当然不完整，但只是提供信息）：

- 调用 `msleep()` 和 `tsleep()`，显然。
- 调用 `g_write_data()` 和 `g_read_data()`，因为这些操作在将数据传递给消费者并返回时会进行睡眠。
- 等待 I/O。
- 调用 [malloc(9)](https://man.freebsd.org/cgi/man.cgi?query=malloc&sektion=9&format=html) 和 `uma_zalloc()`，并设置 `M_WAITOK` 标志。
- sx 和其他可睡眠锁

这一限制的目的是防止 GEOM 代码在 I/O 请求路径中造成阻塞，因为睡眠操作通常没有时间限制，且无法保证会花费多少时间（还有一些其他更为技术性的原因）。这也意味着在这些线程中可以做的事情非常有限；例如，几乎任何复杂的操作都需要内存分配。幸运的是，有一种解决方法：创建额外的内核线程。

### 4.9. GEOM 代码中的内核线程

内核线程是通过 [kthread_create(9)](https://man.freebsd.org/cgi/man.cgi?query=kthread_create&sektion=9&format=html) 函数创建的，它们的行为类似于用户空间线程，只是它们不能通过返回调用者来表示终止，而必须调用 [kthread_exit(9)](https://man.freebsd.org/cgi/man.cgi?query=kthread_exit&sektion=9&format=html)。

在 GEOM 代码中，线程的常见用途是将请求的处理从 `g_down` 线程（即 `.start()` 函数）中卸载出去。这些线程像是“事件处理程序”：它们有一个与之关联的事件链表（该链表中的事件可以由不同线程中的各种函数发布，因此它必须通过互斥锁保护），它们逐一从链表中获取事件并在一个大的 `switch()` 语句中处理它们。

使用线程来处理 I/O 请求的主要好处是它可以在需要时进行睡眠。现在，这听起来不错，但需要仔细考虑。睡眠是非常方便的，但它会非常有效地摧毁 geom 转换的性能。对性能极为敏感的类可能应该将所有工作都放在 `.start()` 函数调用中，并特别小心地处理内存不足和类似的错误。

拥有一个这样的事件处理线程的另一个好处是，它可以将来自不同 geom 线程的所有请求和响应序列化到一个线程中。这也非常方便，但可能会很慢。在大多数情况下，`.done()` 请求的处理可以留给 `g_up` 线程。

FreeBSD 内核中的互斥锁（参见 [mutex(9)](https://man.freebsd.org/cgi/man.cgi?query=mutex&sektion=9&format=html)）与它们在用户空间中的常见形式有一个区别——代码在持有互斥锁时不能进行睡眠。如果代码需要频繁睡眠，[sx(9)](https://man.freebsd.org/cgi/man.cgi?query=sx&sektion=9&format=html) 锁可能更合适。另一方面，如果你几乎所有的工作都在一个线程中完成，那么你可能完全不需要互斥锁。
