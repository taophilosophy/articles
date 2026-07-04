# FreeBSD 中的 Linux® 仿真

- 原文：[Linux® emulation in FreeBSD](https://docs.freebsd.org/en/articles/linux-emulation/)

## 摘要

本篇硕士论文讨论了更新 Linux® 仿真层（即 *Linuxulator*）。任务是将该层更新，以匹配 Linux® 2.6 的功能。作为参考实现，选择了 Linux® 2.6.16 内核。该概念大致基于 NetBSD 实现。大部分工作在 2006 年夏季完成，作为谷歌编程之夏学生项目的一部分。重点是将 *NPTL*（新的 POSIX® 线程库）支持引入仿真层，包括 *TLS*（线程本地存储）、*futexes*（快速用户空间互斥锁）、*PID 混淆*等其他一些小问题。在这个过程中，识别并修复了许多小问题。我的工作已经被整合到 FreeBSD 的主源代码库，并将在即将发布的 7.0R 版本中发布。我们仿真开发团队，正在努力使 Linux® 2.6 仿真成为 FreeBSD 中的默认仿真层。

## 1. 引言

近年来，基于 UNIX® 的开源操作系统已广泛部署在服务器和客户端机器上。在这些操作系统中，我特别想提到两个：FreeBSD，它具有 BSD 传统、经过时间考验的代码库和许多有趣的特性；以及 Linux®，它具有广泛的用户基础、热情的开源开发者社区和大公司的支持。FreeBSD 通常用于服务器级机器，承担重型网络任务，在桌面类机器上的使用较少，而 Linux® 在服务器上也有相同的用途，但更多地被家庭用户使用。这导致了许多仅适用于 Linux® 的二进制程序，而 FreeBSD 缺乏对此的支持。

因此，在 FreeBSD 系统上运行 Linux® 二进制程序的需求自然产生了，而本篇论文正是探讨这一问题：在 FreeBSD 操作系统中仿真 Linux® 内核。

在 2006 年夏季，谷歌公司资助了一项项目，旨在扩展 FreeBSD 中的 Linux® 仿真层（即 Linuxulator），以包括 Linux® 2.6 的功能。本文作为该项目的一部分而编写。

## 2. 内部探讨……

在这一部分，我们将介绍每个相关操作系统，如何处理系统调用、陷阱帧等底层内容。我们还将概述它们如何理解常见的 UNIX® 原语，比如 PID 是什么，线程是什么等。在第三个小节中，我们讨论了 UNIX® 与 UNIX® 之间仿真的一般方法。

### 2.1. 什么是 UNIX®

UNIX® 是一款具有悠久历史的操作系统，几乎影响了当前使用的所有操作系统。自 1960 年代开始，它的开发至今仍在继续（尽管已分叉为不同的项目）。UNIX® 的发展很快分叉为两大主流：BSD 系列和 System III/V 系列。它们通过共同的 UNIX® 标准相互影响。在 BSD 系列中，值得一提的贡献有虚拟内存、TCP/IP 网络、FFS 等，而 System V 分支则贡献了 SysV 进程间通信原语、写时复制等。虽然 UNIX® 本身已不存在，但其思想已被世界上许多操作系统所采用，形成了所谓的类 UNIX® 操作系统。目前，最具影响力的操作系统包括 Linux®、Solaris 和可能在一定程度上包括 FreeBSD。还有一些公司内部的 UNIX® 派生操作系统（如 AIX、HP-UX 等），但这些系统越来越多地迁移到了上述操作系统。让我们总结一下 UNIX® 的典型特征。

### 2.2. 技术细节

每个运行的程序构成一个进程，代表计算的某个状态。运行中的进程分为内核空间和用户空间。某些操作只能在内核空间执行（如与硬件交互等），但进程应该大部分时间运行在用户空间。内核负责管理进程、硬件和底层细节，并为用户空间提供统一的 UNIX® API。下面列出了最重要的几个。

#### 2.2.1. 内核与用户空间进程之间的通信

常见的 UNIX® API 定义了系统调用（syscall）作为用户空间进程向内核发出命令的一种方式。最常见的实现方式是通过使用中断或专用指令（例如，ia32 的 `SYSENTER`/`SYSCALL` 指令）。系统调用是通过一个编号来定义的。例如，在 FreeBSD 中，系统调用编号 85 是 [swapon(2)](https://man.freebsd.org/cgi/man.cgi?query=swapon&sektion=2&format=html) 系统调用，编号 132 是 [mkfifo(2)](https://man.freebsd.org/cgi/man.cgi?query=mkfifo&sektion=2&format=html) 系统调用。某些系统调用需要参数，这些参数通过多种方式从用户空间传递到内核空间（具体实现依赖于操作系统）。系统调用是同步的。

另一种可能的通信方式是通过使用 *trap*（陷阱）。陷阱是由于某些事件发生后异步触发的（例如，除零错误、页面错误等）。陷阱可能对进程透明（如页面错误），或者可能导致某些反应，如发送一个 *signal*（信号）（例如，除零错误）。

#### 2.2.2. 进程间通信

除了其他 API（如 System V IPC、共享内存等）之外，最重要的 API 是信号。信号由进程或内核发送，并由进程接收。某些信号可以被忽略或由用户提供的例程处理，其他信号则会导致预定义的动作，这些动作无法改变或忽略。

#### 2.2.3. 进程管理

内核实例首先在系统中被处理（称为 init）。每个运行的进程都可以使用 [fork(2)](https://man.freebsd.org/cgi/man.cgi?query=fork&sektion=2&format=html) 系统调用创建它的副本。这个系统调用有一些稍微修改过的版本，但其基本语义是相同的。每个运行的进程可以通过调用 [exec(3)](https://man.freebsd.org/cgi/man.cgi?query=exec&sektion=3&format=html) 系统调用转换为另一个进程。这个系统调用也有一些修改过的版本，但所有版本的基本目的是一样的。进程通过调用 [exit(2)](https://man.freebsd.org/cgi/man.cgi?query=exit&sektion=2&format=html) 系统调用结束其生命周期。每个进程都有一个唯一的标识号，称为 PID，每个进程都有一个父进程（通过其 PID 来标识）。

#### 2.2.4. 线程管理

传统的 UNIX® 并未定义任何关于线程的 API 或实现，而 POSIX® 定义了其线程 API，但未定义其实现。传统上，线程有两种实现方式。将其作为独立进程处理（1:1 线程）或将整个线程组封装在一个进程中并在用户空间管理线程（1:N 线程）。比较这两种方法的主要特点：

1:1 线程

- 重型线程
- 用户无法修改调度（通过 POSIX® API 稍微可以缓解）
- 无需系统调用包装
- 可以利用多个 CPU

1:N 线程

- 轻量级线程
- 可以轻松修改调度
- 需要包装系统调用
- 无法利用多个 CPU

### 2.3. 什么是 FreeBSD？

FreeBSD 项目是目前可供日常使用的最古老的开源操作系统之一。它是正统 UNIX® 的直接后代，因此可以声称它是一个真正的 UNIX®，尽管由于许可证问题不能这样称呼。该项目始于 1990 年代初，当时一群 BSD 用户修补了 386BSD 操作系统。在这个补丁包的基础上，一个新的操作系统应运而生，名为 FreeBSD，因其宽松的许可证而得名。另一组人则创建了 NetBSD 操作系统，目标有所不同。我们将重点讨论 FreeBSD。

FreeBSD 是一个现代的基于 UNIX® 的操作系统，具备 UNIX® 的所有特性。包括抢占式多任务处理、多用户功能、TCP/IP 网络、内存保护、对称多处理支持、虚拟内存与合并的 VM 和缓冲区缓存等功能，它们都包含在内。一个有趣且非常有用的特性是能够仿真其他 UNIX® 类操作系统。截至 2006 年 12 月和 7-CURRENT 开发版本，以下仿真功能已经得到支持：

- FreeBSD/i386 在 FreeBSD/amd64 上的仿真
- FreeBSD/i386 在 FreeBSD/ia64 上的仿真
- Linux® 仿真：在 FreeBSD 上仿真 Linux® 操作系统
- NDIS 仿真：Windows 网络驱动接口的仿真
- NetBSD 仿真：仿真 NetBSD 操作系统
- PECoff 支持：支持 PECoff 格式的 FreeBSD 可执行文件
- SVR4 仿真：仿真 System V 第四版 UNIX®

目前正在积极开发的仿真功能有 Linux® 层和各种 FreeBSD-on-FreeBSD 层。其他的功能目前不被认为是能够正常工作或可用的。

#### 2.3.1. 技术细节

FreeBSD 是传统的 UNIX® 版本，在进程运行方面将其分为两个部分：内核空间和用户空间。进程进入内核有两种方式：系统调用和陷阱。返回的方式只有一种。在接下来的章节中，我们将介绍进出内核的三个入口。整段简介适用于 i386 架构，因为 Linuxulator 仅存在于此，但在其他架构上的概念类似。以下信息来源于 [1] 和源代码。

##### 2.3.1.1. 系统入口

FreeBSD 有个抽象概念，称为执行类加载器，它是 [execve(2)](https://man.freebsd.org/cgi/man.cgi?query=execve&sektion=2&format=html) 系统调用的一个切入点。它使用一个名为 `sysentvec` 的结构，描述可执行文件的 ABI。该结构包含错误号翻译表、信号翻译表、以及为系统调用提供服务的各种函数（例如堆栈修正、核心转储等）。FreeBSD 内核想要支持的每个 ABI 都必须定义这个结构，因为它会在系统调用处理代码和其他地方被使用。系统入口由陷阱处理程序处理，我们可以同时访问内核空间和用户空间。

##### 2.3.1.2. 系统调用

FreeBSD 中的系统调用通过执行中断 `0x80` 发起，其中寄存器 `%eax` 被设置为所需的系统调用号，参数则通过堆栈传递。

当一个进程发出中断 `0x80` 时，将调用 `int0x80` 系统调用陷阱处理程序（在 **sys/i386/i386/exception.s** 中定义），该处理程序为调用 C 函数 [syscall(2)](https://man.freebsd.org/cgi/man.cgi?query=syscall&sektion=2&format=html)（在 **sys/i386/i386/trap.c** 中定义）做准备，处理传入的陷阱帧。处理过程包括准备系统调用（取决于 `sysvec` 条目），确定系统调用是 32 位还是 64 位（这会改变参数的大小），然后将参数复制，包括系统调用。接下来，执行实际的系统调用函数，并处理返回代码（特殊情况包括 `ERESTART` 和 `EJUSTRETURN` 错误）。最后，调度一个 `userret()`，将进程切换回用户空间。实际的系统调用处理程序的参数以 `struct thread *td` 和 `struct syscall args *` 参数的形式传递，第二个参数是指向复制的参数结构的指针。

##### 2.3.1.3. 陷阱

在 FreeBSD 中，陷阱的处理与系统调用的处理类似。每当发生陷阱时，会调用一个汇编处理程序。这个处理程序的选择依据陷阱类型而不同，可以是 `alltraps`、`alltraps with regs pushed` 或 `calltrap`。该处理程序准备好参数后，调用 C 函数 `trap()`（定义在 **sys/i386/i386/trap.c** 中），然后处理发生的陷阱。处理完成后，可能会向进程发送信号和/或使用 `userret()` 返回到用户空间。

##### 2.3.1.4. 退出

从内核到用户空间的退出是通过汇编例程 `doreti` 完成的，无论是通过陷阱还是系统调用进入内核。该过程会从堆栈中恢复程序状态并返回到用户空间。

##### 2.3.1.5. UNIX® 原语

FreeBSD 操作系统遵循传统的 UNIX® 方案，其中每个进程都有一个唯一的标识号，称为 *PID*（进程 ID）。PID 数字的分配可以是线性分配或随机分配，范围从 `0` 到 `PID_MAX`。PID 数字的分配是通过线性搜索 PID 空间完成的。每个进程中的线程都会获得与进程相同的 PID，作为 [getpid(2)](https://man.freebsd.org/cgi/man.cgi?query=getpid&sektion=2&format=html) 调用的结果。

FreeBSD 中目前有两种实现线程的方法。第一种是 M:N 线程模型，接下来是 1:1 线程模型。默认使用的库是 M:N 线程模型（`libpthread`），并且可以在运行时切换到 1:1 线程模型（`libthr`）。计划是尽快将默认库切换为 1:1 库。尽管这两个库使用相同的内核原语，但它们通过不同的 API 访问。M:N 库使用 `kse_*` 系列系统调用，而 1:1 库使用 `thr_*` 系列系统调用。因此，内核和用户空间之间没有共享的线程 ID 概念。当然，这两个线程库都实现了 pthread 线程 ID API。每个内核线程（由 `struct thread` 描述）都有一个 td tid 标识符，但它不能直接从用户空间访问，只供内核使用。它也被用作 1:1 线程库中的 pthread 线程 ID，但其处理方式是库内部的，不能依赖于此。

如前所述，FreeBSD 中有两种线程实现。M:N 库将工作分配给内核空间和用户空间。线程是一个在内核中调度的实体，但它可以表示多个用户空间线程。M 个用户空间线程映射到 N 个内核线程，从而节省资源，同时保持利用多处理器并行性的能力。有关实现的更多信息可以从手册页或 [1] 获取。1:1 库则直接将用户空间线程映射到内核线程，从而大大简化了方案。这些设计都没有实现公平机制（虽然曾经实现过，但由于造成了严重的性能下降并使代码更难维护，最近已被移除）。

### 2.4. 什么是 Linux®？

Linux® 是一款类似 UNIX® 的内核，最初由 Linus Torvalds 开发，现在由全球大量程序员贡献。自从它的起步到今天，得到了 IBM 和谷歌等公司的广泛支持，Linux® 与其快速的开发速度、全面的硬件支持和仁慈独裁的组织模式相关联。

Linux® 的开发始于 1991 年，在芬兰赫尔辛基大学作为一个爱好者项目启动。从那时起，它就获得了现代 UNIX® 类操作系统的所有特性：多处理、支持多用户、虚拟内存、网络功能，基本上所有功能都具备了。还有一些先进的特性，如虚拟化等。

截至 2006 年，Linux® 已成为最广泛使用的开源操作系统，得到了独立软件厂商如 Oracle、RealNetworks、Adobe 等的支持。大多数为 Linux® 分发的商业软件只能以二进制形式获得，因此无法重新编译为其他操作系统。

大多数 Linux® 的开发都发生在 Git 版本控制系统中。Git 是一款分布式系统，因此 Linux® 代码没有中央来源，但一些分支被认为是突出和官方的。Linux® 实现的版本号方案由四个数字 A.B.C.D 组成。目前开发版本为 2.6.C.D，其中 C 代表主要版本，添加或更改新特性，而 D 是次要版本，仅用于修复 bug。

更多信息可以从 [3] 获取。

#### 2.4.1. 技术细节

Linux® 遵循传统的 UNIX® 方案，将进程的执行分为两个部分：内核空间和用户空间。内核可以通过两种方式进入：通过陷阱或通过系统调用。返回只有一种方式。以下描述适用于 i386™ 架构上的 Linux® 2.6。该信息来源于 [2]。

##### 2.4.1.1. 系统调用

Linux® 中的系统调用（在用户空间中）使用 `syscallX` 宏来执行，其中 X 是表示给定系统调用参数数量的数字。此宏会转换为一个代码，该代码将 `%eax` 寄存器加载为系统调用的编号，并执行中断 `0x80`。在系统调用返回后，负返回值会转换为正的 `errno` 值，并且在发生错误时将 `res` 设置为 `-1`。每当调用中断 `0x80` 时，进程就会进入内核，进入系统调用陷阱处理程序。该程序将所有寄存器保存到堆栈中，并调用选定的系统调用入口。需要注意的是，Linux® 的调用约定要求通过寄存器传递系统调用的参数，具体如下：

1. 参数 → `%ebx`
2. 参数 → `%ecx`
3. 参数 → `%edx`
4. 参数 → `%esi`
5. 参数 → `%edi`
6. 参数 → `%ebp`

有些系统调用例外，使用了不同的调用约定（最显著的例子是 `clone` 系统调用）。

##### 2.4.1.2. 陷阱

陷阱处理程序定义在 **arch/i386/kernel/traps.c** 中，处理大部分陷阱的代码位于 **arch/i386/kernel/entry.S**，其中进行陷阱的处理。

##### 2.4.1.3. 退出

从系统调用返回由系统调用 [exit(3)](https://man.freebsd.org/cgi/man.cgi?query=exit&sektion=3&format=html) 管理，系统调用会检查进程是否有未完成的工作，然后检查是否使用了用户提供的选择器。如果发生这种情况，会应用堆栈修复，最后从堆栈恢复寄存器，进程返回到用户空间。

##### 2.4.1.4. UNIX® 原语

在 2.6 版本中，Linux® 操作系统重新定义了一些传统的 UNIX® 原语，特别是 PID、TID 和线程。PID 并不被定义为每个进程唯一的标识符，因此对于某些进程（线程），[getppid(2)](https://man.freebsd.org/cgi/man.cgi?query=getppid&sektion=2&format=html) 返回相同的值。进程的唯一标识是通过 TID 提供的。这是因为 *NPTL*（新 POSIX® 线程库）将线程定义为普通进程（即 1:1 线程模型）。在 Linux® 2.6 中创建新进程是通过 `clone` 系统调用（`fork` 的变体）来完成的。这个 `clone` 系统调用定义了一组影响线程实现的克隆过程行为的标志。其语义有点模糊，因为没有单一的标志可以告诉系统调用创建一个线程。

实现的 `clone` 标志包括：

- `CLONE_VM` - 进程共享内存空间
- `CLONE_FS` - 共享 umask、当前工作目录和命名空间
- `CLONE_FILES` - 共享打开的文件
- `CLONE_SIGHAND` - 共享信号处理程序和被阻塞的信号
- `CLONE_PARENT` - 共享父进程
- `CLONE_THREAD` - 作为线程（后续解释）
- `CLONE_NEWNS` - 新命名空间
- `CLONE_SYSVSEM` - 共享 SysV undo 结构
- `CLONE_SETTLS` - 在提供的地址设置 TLS
- `CLONE_PARENT_SETTID` - 在父进程中设置 TID
- `CLONE_CHILD_CLEARTID` - 清除子进程中的 TID
- `CLONE_CHILD_SETTID` - 在子进程中设置 TID

`CLONE_PARENT` 将实际父进程设置为调用者的父进程。这对于线程非常有用，因为如果线程 A 创建了线程 B，那么我们希望线程 B 的父进程是整个线程组的父进程。`CLONE_THREAD` 完全执行与 `CLONE_PARENT` 相同的操作，`CLONE_VM` 和 `CLONE_SIGHAND`，将 PID 修改为与调用者的 PID 相同，设置退出信号为空，并进入线程组。`CLONE_SETTLS` 设置 TLS 处理的 GDT 条目。`CLONE_*_*TID` 系列标志设置/清除 TID 或 0 的用户提供地址。

正如你所见，`CLONE_THREAD` 完成了大部分工作，并且似乎并不完全适应当前的方案。其原始意图不明确（根据代码中的注释，甚至连作者也不清楚），但我认为最初可能有一个线程标志，后来被拆分为许多其他标志，但这种拆分从未完全完成。对于这一划分的意义也不清楚，因为 glibc 并没有使用它，所以只有通过手写的 `clone` 调用，程序员才能访问这些功能。

对于非线程程序，PID 和 TID 是相同的。对于线程程序，第一个线程的 PID 和 TID 是相同的，每个创建的线程共享相同的 PID，并获得一个唯一的 TID（因为传递了 `CLONE_THREAD`），同时所有进程共享父进程，形成该线程程序。

在 NPTL 中实现 [pthread_create(3)](https://man.freebsd.org/cgi/man.cgi?query=pthread_create&sektion=3&format=html) 的代码定义了 `clone` 标志，如下所示：

```c
int clone_flags = (CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGNAL
 | CLONE_SETTLS | CLONE_PARENT_SETTID
 | CLONE_CHILD_CLEARTID | CLONE_SYSVSEM
#if __ASSUME_NO_CLONE_DETACHED == 0
 | CLONE_DETACHED
#endif
 | 0);
```

`CLONE_SIGNAL` 定义为：

```c
#define CLONE_SIGNAL (CLONE_SIGHAND | CLONE_THREAD)
```

最后的 0 表示在任何线程退出时不会发送信号。

### 2.5. 什么是仿真

根据词典定义，仿真是程序或设备模仿另一个程序或设备的能力。这是通过对给定的刺激作出与被仿真对象相同的反应来实现的。在实际中，软件世界通常看到三种类型的仿真——用于仿真机器的程序（如 QEMU，各种游戏机仿真器等）、硬件设施的仿真软件（如 OpenGL 仿真器，浮点单元仿真等）以及操作系统仿真（无论是在操作系统的内核中还是作为用户空间程序）。

仿真通常在无法或完全不可能使用原始组件的地方使用。例如，有人可能希望使用为不同操作系统开发的程序。这时仿真就派上了用场。有时别无选择，只能使用仿真——例如当你试图使用的硬件设备不存在（还不存在或不再存在）时，那么唯一的选择就是仿真。通常在将操作系统移植到一个新平台（该平台尚不存在）时会发生这种情况。有时仿真只是更便宜。

从实现的角度来看，仿真的实现方法主要有两种。你可以仿真整个组件——接受原始对象的可能输入，维护内部状态并根据状态和/或输入发出正确的输出。这种仿真不需要特殊的条件，基本上可以在任何地方为任何设备/程序实现。缺点是，实现这种仿真相当困难、耗时且容易出错。在某些情况下，我们可以使用更简单的方法。假设你想在一台从右向左打印的打印机上仿真一台从左向右打印的打印机。显然，没有必要使用复杂的仿真层，只需简单地反转打印的文本即可。也有可能仿真环境与被仿真环境非常相似，因此只需要一层薄薄的转换即可提供完整的仿真！正如你所见，这种方法比前一种方法更不要求实现，因此也更不耗时且更不容易出错。但必要的条件是两个环境必须足够相似。第三种方法结合了前两种方法。大多数情况下，仿真对象提供的能力并不相同，因此在更强大的对象上仿真较弱的对象时，我们必须使用上述完全仿真所描述的方式来仿真缺失的功能。

本硕士论文涉及 UNIX® 在 UNIX® 上的仿真，这正是一个只需薄薄的转换层就能提供完整仿真的案例。UNIX® API 由一组系统调用组成，这些调用通常是自包含的，并不会影响全局的内核状态。

有一些系统调用会影响内部状态，但这可以通过提供一些结构来处理，这些结构维护额外的状态。

没有任何仿真是完美的，仿真通常会缺少一些部分，但这通常不会造成严重的缺陷。想象一个游戏机仿真器，仿真所有内容但没有声音输出。毫无疑问，游戏仍然是可以玩的，尽管不如原始游戏机那样舒适，但它是价格与舒适度之间的一个可以接受的折衷。

UNIX® API 也是如此。大多数程序可以在很有限的系统调用集工作下生存。这些系统调用通常是最古老的那些（[read(2)](https://man.freebsd.org/cgi/man.cgi?query=read&sektion=2&format=html)/[write(2)](https://man.freebsd.org/cgi/man.cgi?query=write&sektion=2&format=html)、[fork(2)](https://man.freebsd.org/cgi/man.cgi?query=fork&sektion=2&format=html) 系列、[signal(3)](https://man.freebsd.org/cgi/man.cgi?query=signal&sektion=3&format=html) 处理、[exit(3)](https://man.freebsd.org/cgi/man.cgi?query=exit&sektion=3&format=html)、[socket(2)](https://man.freebsd.org/cgi/man.cgi?query=socket&sektion=2&format=html) API)，因此它们很容易仿真，因为它们的语义在今天存在的所有 UNIX® 操作系统中是共享的。

## 3. 仿真

### 3.1. FreeBSD 中仿真的工作原理

如前所述，FreeBSD 支持运行来自其他多个 UNIX® 系统的二进制文件。之所以能实现这一点，是因为 FreeBSD 提供了一个叫做执行类加载器（execution class loader）的抽象机制。该机制与 [execve(2)](https://man.freebsd.org/cgi/man.cgi?query=execve&sektion=2&format=html) 系统调用结合使用，当 [execve(2)](https://man.freebsd.org/cgi/man.cgi?query=execve&sektion=2&format=html) 准备执行一个二进制文件时，它会检查该文件的类型。

在 FreeBSD 中，二进制文件大致分为两种类型。第一种是类似 shell 的文本脚本，它们的前两个字符是 `#!`；第二种是正常的（通常是 *ELF* 格式的）二进制文件，表示编译后的可执行对象。绝大多数（可以说所有）FreeBSD 中的二进制文件都是 ELF 类型。ELF 文件包含一个头部，其中指定了该 ELF 文件的操作系统 ABI（应用二进制接口）。通过读取这些信息，操作系统可以准确地确定给定文件的二进制类型。

每个操作系统 ABI 必须在 FreeBSD 内核中注册，包括 FreeBSD 本身的原生操作系统 ABI。因此，当 [execve(2)](https://man.freebsd.org/cgi/man.cgi?query=execve&sektion=2&format=html) 执行一个二进制文件时，它会遍历已注册的 ABI 列表，当找到匹配的 ABI 时，它会开始使用该 ABI 描述中包含的信息（如系统调用表、`errno` 转换表等）。因此，每当进程调用系统调用时，它都会使用自己的一套系统调用，而不是使用全局系统调用。这为支持执行各种二进制格式提供了一种非常优雅且简单的方式。

不同操作系统（以及其他一些子系统）仿真的特点促使开发者引入了事件处理机制。内核中的多个地方允许注册事件处理程序，这些处理程序会按需被调用。例如，当一个进程退出时，会调用一个处理程序，清理该子系统需要清理的内容。

这些简单的设施基本上提供了仿真基础架构所需的一切，实际上，它们是实现 Linux® 仿真层所必需的唯一内容。

### 3.2. FreeBSD 内核中的常见原语

仿真层需要操作系统的支持。接下来我将描述一些 FreeBSD 操作系统中支持的原语。

#### 3.2.1. 锁原语

贡献者：`Attilio Rao <attilio@FreeBSD.org>`

FreeBSD 的同步原语集基于提供大量不同原语的理念，以便在每种特定情况下可以使用最合适的原语。

从高层次来看，FreeBSD 内核中可以视为三种同步原语：

- 原子操作和内存屏障
- 锁
- 调度屏障

以下是这三类原语的描述。对于每种锁，你应该查阅相应的手册页（如果可能的话）以获取更详细的解释。

##### 3.2.1.1. 原子操作和内存屏障

原子操作通过一组函数实现，这些函数以原子的方式对内存操作数进行简单的算术运算，并且对外部事件（如中断、抢占等）保持原子性。原子操作仅能保证对小数据类型的原子性（在 `.long.` 架构的 C 数据类型量级范围内），因此它们应该在最终级代码中很少直接使用，除非是进行非常简单的操作（例如，在位图中设置标志）。事实上，单纯依赖原子操作编写错误的语义（通常被称为无锁操作）是相当常见的。FreeBSD 内核提供了一种方法，将原子操作与内存屏障结合使用。内存屏障能够保证原子操作按照指定的顺序执行，与其他内存访问之间保持一定的顺序关系。例如，如果我们需要确保原子操作在所有其他待处理写操作（指令重排缓冲区活动）完成之后才发生，就需要明确地在该原子操作之前使用内存屏障。因此，理解内存屏障在更高级别锁构建中的关键作用是很简单的（比如引用计数、互斥锁等）。关于原子操作的详细解释，请参考 [atomic(9)](https://man.freebsd.org/cgi/man.cgi?query=atomic&sektion=9&format=html)。然而，值得注意的是，原子操作（以及内存屏障）理想情况下应仅用于构建前端锁（如互斥锁）。

##### 3.2.1.2. 引用计数

引用计数是用于处理引用计数器的接口。它们通过原子操作实现，旨在仅在引用计数器是唯一需要保护的内容时使用，因此即使是旋转互斥锁（spin-mutex）也不再推荐使用。对于已经使用互斥锁保护的结构，使用引用计数接口通常是不正确的，因为我们可能应该在一些已经保护的路径中关闭引用计数器。当前没有关于引用计数的手册页面，请查看 **sys/refcount.h** 以获取现有 API 的概述。

##### 3.2.1.3. 锁

FreeBSD 内核有大量的锁类型。每种锁都由一些特定的属性定义，但最重要的属性可能是与争用持有者相关的事件（或换句话说，线程无法获取锁时的行为）。FreeBSD 的锁定方案为竞争者提供了三种不同的行为：

1. 自旋
2. 阻塞
3. 睡眠

>**注意**
>
>数字并非随意

##### 3.2.1.4. 自旋锁

自旋锁允许等待的线程自旋，直到它能够获取锁。需要处理的一个重要问题是，当线程在自旋锁上发生争用时，如果线程没有被重新调度，会发生什么情况。由于 FreeBSD 内核是抢占式的，这使得自旋锁面临死锁的风险，唯一的解决办法是禁用中断来获取锁。由于这些原因（例如缺乏优先级传播支持、CPU 之间负载均衡方案的不足等），自旋锁应该仅保护非常小的代码路径，或者如果没有明确要求，最好根本不使用它们（稍后将解释）。

##### 3.2.1.5. 阻塞

阻塞锁允许等待的线程被重新调度并阻塞，直到锁的拥有者释放锁并唤醒一个或多个竞争者。为了避免饥饿问题，阻塞锁会将优先级从等待者传递给锁的拥有者。阻塞锁必须通过转门（turnstile）接口实现，并且应该是内核中最常用的锁类型，除非满足某些特定条件。

##### 3.2.1.6. 睡眠

睡眠锁允许等待的线程被重新调度并进入睡眠，直到锁的持有者释放锁并唤醒一个或多个等待者。由于睡眠锁旨在保护较大的代码路径并处理异步事件，因此它们不会进行任何形式的优先级传播。它们必须通过 [sleepqueue(9)](https://man.freebsd.org/cgi/man.cgi?query=sleepqueue&sektion=9&format=html) 接口实现。

获取锁的顺序非常重要，不仅因为锁顺序反转可能导致死锁，还因为获取锁时应遵循特定的规则，这些规则与锁的性质有关。如果你查看上面的表格，可以得出一个实际规则：如果一个线程持有级别为 n 的锁（其中级别是与锁类型相对应的数字），则不允许获取更高级别的锁，因为这会破坏路径的语义。例如，如果一个线程持有一个阻塞锁（级别 2），它可以获取自旋锁（级别 1），但不能获取睡眠锁（级别 3），因为阻塞锁旨在保护比睡眠锁更小的路径（然而这些规则与原子操作或调度屏障无关）。

以下是锁及其相应行为的列表：

- 自旋互斥锁 - 自旋 - [mutex(9)](https://man.freebsd.org/cgi/man.cgi?query=mutex&sektion=9&format=html)
- 睡眠互斥锁 - 阻塞 - [mutex(9)](https://man.freebsd.org/cgi/man.cgi?query=mutex&sektion=9&format=html)
- 池互斥锁 - 阻塞 - [mtx(pool)](https://man.freebsd.org/cgi/man.cgi?query=mtx&sektion=pool&format=html)
- 睡眠家族 - 睡眠 - [sleep(9)](https://man.freebsd.org/cgi/man.cgi?query=sleep&sektion=9&format=html)，pause，tsleep，msleep，msleep，spin，msleep，rw，msleep，sx
- 条件变量 - 睡眠 - [condvar(9)](https://man.freebsd.org/cgi/man.cgi?query=condvar&sektion=9&format=html)
- 读写锁 - 阻塞 - [rwlock(9)](https://man.freebsd.org/cgi/man.cgi?query=rwlock&sektion=9&format=html)
- sx 锁 - 睡眠 - [sx(9)](https://man.freebsd.org/cgi/man.cgi?query=sx&sektion=9&format=html)
- lockmgr - 睡眠 - [lockmgr(9)](https://man.freebsd.org/cgi/man.cgi?query=lockmgr&sektion=9&format=html)
- 信号量 - 睡眠 - [sema(9)](https://man.freebsd.org/cgi/man.cgi?query=sema&sektion=9&format=html)

在这些锁中，只有互斥锁、sx 锁、读写锁和 lockmgr 锁是设计用来处理递归的，但目前只有互斥锁和 lockmgr 锁支持递归。

##### 3.2.1.7. 调度屏障

调度屏障用于驱动线程的调度。它们主要由三种不同的存根组成：

- 临界区（和抢占）
- sched_bind
- sched_pin

通常，这些应该仅在特定的上下文中使用，即使它们通常可以替代锁，也应该避免使用它们，因为它们不允许使用锁调试工具（如 [witness(4)](https://man.freebsd.org/cgi/man.cgi?query=witness&sektion=4&format=html)）来诊断简单的潜在问题。

##### 3.2.1.8. 临界区

FreeBSD 内核是为了处理中断线程而使其具有抢占性。实际上，为了避免高中断延迟，分时优先级线程可以被中断线程抢占（这样，它们无需等到正常调度路径的预设）。临界区定义了一段代码（由 [critical_enter(9)](https://man.freebsd.org/cgi/man.cgi?query=critical_enter&sektion=9&format=html) 和 [critical_exit(9)](https://man.freebsd.org/cgi/man.cgi?query=critical_exit&sektion=9&format=html) 这两个函数边界标定），在此期间保证不会发生抢占（直到受保护的代码完全执行完毕）。这通常可以有效地替代锁，但应小心使用，以免失去抢占带来的整体优势。

##### 3.2.1.9. sched_pin/sched_unpin

另一种处理抢占的方法是使用 `sched_pin()` 接口。如果一段代码被包含在 `sched_pin()` 和 `sched_unpin()` 这对函数中，保证该线程即使可以被抢占，也将始终在同一个 CPU 上执行。固定（pinning）在某些情况下非常有效，特别是当我们需要访问每个 CPU 特有的数据，并假设其他线程不会更改这些数据时。后者的条件将决定临界区对我们代码来说过于强的条件。

##### 3.2.1.10. sched_bind/sched_unbind

`sched_bind` 是一种 API，用于将线程绑定到特定的 CPU 上，直到 `sched_unbind` 函数调用将其解除绑定。这一功能在你无法信任当前 CPU 状态的情况下非常重要（例如，在引导的早期阶段），因为你希望避免线程迁移到非活动的 CPU 上。由于 `sched_bind` 和 `sched_unbind` 操作内部调度器结构，因此在使用时需要在 `sched_lock` 获取/释放之间进行封装。

#### 3.2.2. Proc 结构

各种仿真层有时需要一些额外的每个进程数据。可以为每个进程管理单独的结构（如列表、树等）来存储这些数据，但这往往效率低下且消耗内存。为了解决这个问题，FreeBSD 的 `proc` 结构包含了 `p_emuldata`，它是一个指向某些仿真层特定数据的 void 指针。这个 `proc` 条目受到 proc 锁的保护。

FreeBSD 的 `proc` 结构包含 `p_sysent` 条目，标识该进程正在运行的 ABI。事实上，它是指向上述 `sysentvec` 的一个指针。所以，通过将这个指针与给定 ABI 的 `sysentvec` 结构的存储地址进行比较，我们可以有效地判断该进程是否属于我们的仿真层。代码通常如下所示：

```c
if (__predict_true(p->p_sysent != &elf_Linux(R)_sysvec))
	  return;
```

如你所见，我们有效地使用 `__predict_true` 修饰符将最常见的情况（FreeBSD 进程）压缩为简单的返回操作，从而保持高性能。由于当前不支持 Linux®64 仿真或 i386 上的 A.OUT Linux® 进程，因此这段代码应当转换为宏，因为它目前不够灵活。

#### 3.2.3. VFS

FreeBSD 的 VFS 子系统非常复杂，但 Linux® 仿真层仅通过一个小子集通过明确定义的 API 使用它。它可以操作 vnodes 或文件句柄。Vnode 代表一个虚拟 vnode，即 VFS 中节点的表示。另一种表示是文件句柄，它从进程的角度表示一个已打开的文件。文件句柄可以代表套接字或普通文件。一个文件句柄可以指向其 vnode。多个文件句柄可以指向同一个 vnode。

##### 3.2.3.1. namei

[namei(9)](https://man.freebsd.org/cgi/man.cgi?query=namei&sektion=9&format=html) 例程是路径名查找和转换的中央入口点。它从起始点到终点逐点遍历路径，使用内部的查找功能。[namei(9)](https://man.freebsd.org/cgi/man.cgi?query=namei&sektion=9&format=html) 系统调用能够处理符号链接、绝对路径和相对路径。当一个路径通过 [namei(9)](https://man.freebsd.org/cgi/man.cgi?query=namei&sektion=9&format=html) 查找时，它被输入到名称缓存中。此行为可以被抑制。这个例程在整个内核中被广泛使用，其性能非常关键。

##### 3.2.3.2. vn_fullpath

[vn_fullpath(9)](https://man.freebsd.org/cgi/man.cgi?query=vn_fullpath&sektion=9&format=html) 函数尽最大努力遍历 VFS 名称缓存，并为给定（已锁定的）vnode 返回路径。这个过程不可靠，但对于大多数常见情况来说效果很好。不可靠的原因是它依赖于 VFS 缓存（它不会遍历介质结构），它不适用于硬链接等。这一例程在 Linuxulator 中的多个地方使用。


##### 3.2.3.3. Vnode 操作

- `fgetvp` - 给定线程和文件描述符号，它返回关联的 vnode
- [vn_lock(9)](https://man.freebsd.org/cgi/man.cgi?query=vn_lock&sektion=9&format=html) - 锁定 vnode
- `vn_unlock` - 解锁 vnode
- [VOP_READDIR(9)](https://man.freebsd.org/cgi/man.cgi?query=VOP_READDIR&sektion=9&format=html) - 读取由 vnode 引用的目录
- [VOP_GETATTR(9)](https://man.freebsd.org/cgi/man.cgi?query=VOP_GETATTR&sektion=9&format=html) - 获取由 vnode 引用的文件或目录的属性
- [VOP_LOOKUP(9)](https://man.freebsd.org/cgi/man.cgi?query=VOP_LOOKUP&sektion=9&format=html) - 查找给定目录的路径
- [VOP_OPEN(9)](https://man.freebsd.org/cgi/man.cgi?query=VOP_OPEN&sektion=9&format=html) - 打开由 vnode 引用的文件
- [VOP_CLOSE(9)](https://man.freebsd.org/cgi/man.cgi?query=VOP_CLOSE&sektion=9&format=html) - 关闭由 vnode 引用的文件
- [vput(9)](https://man.freebsd.org/cgi/man.cgi?query=vput&sektion=9&format=html) - 减少 vnode 的使用计数并解锁它
- [vrele(9)](https://man.freebsd.org/cgi/man.cgi?query=vrele&sektion=9&format=html) - 减少 vnode 的使用计数
- [vref(9)](https://man.freebsd.org/cgi/man.cgi?query=vref&sektion=9&format=html) - 增加 vnode 的使用计数

##### 3.2.3.4. 文件句柄操作

- `fget` - 给定线程和文件描述符号，它返回关联的文件句柄并引用它
- `fdrop` - 删除对文件句柄的引用
- `fhold` - 引用文件句柄

## 4. Linux® 仿真层——机器相关部分

本节讨论了 FreeBSD 操作系统中 Linux® 仿真层的实现。首先介绍了与用户态和内核交互的机器相关部分，涉及系统调用、信号、ptrace、陷阱、栈修正等内容。尽管本节以 i386 为例，但其编写方式是通用的，因此其他架构的实现应不会有很大不同。接下来是 Linuxulator 的机器无关部分，本节仅覆盖 i386 和 ELF 处理，A.OUT 已不再使用且未经测试。

### 4.1. 系统调用处理

系统调用处理大部分写在 **linux_sysvec.c** 中，该文件覆盖了 `sysentvec` 结构中列出的绝大多数例程。当在 FreeBSD 上运行的 Linux® 进程发出系统调用时，通用系统调用例程会调用 Linux® 的 prepsyscall 例程。

#### 4.1.1. Linux® prepsyscall

Linux® 通过寄存器传递系统调用的参数（这就是为什么它在 i386 上仅限于 6 个参数的原因），而 FreeBSD 则使用栈。Linux® 的 prepsyscall 例程必须将参数从寄存器复制到栈中。寄存器的顺序是：`%ebx`、`%ecx`、`%edx`、`%esi`、`%edi`、`%ebp`。需要注意的是，这对于 *大多数* 系统调用是成立的。有些（尤其是 `clone`）使用不同的顺序，但幸运的是，可以通过在 `linux_clone` 原型中插入一个虚拟参数来轻松修复。

#### 4.1.2. 系统调用写入

Linuxulator 中实现的每个系统调用必须在 **syscalls.master** 中定义其原型，并附带各种标志。该文件的格式如下：

```c
...
	AUE_FORK STD		{ int linux_fork(void); }
...
	AUE_CLOSE NOPROTO	{ int close(int fd); }
...
```

第一列表示系统调用编号。第二列用于审计支持。第三列表示系统调用类型。它可以是 `STD`、`OBSOL`、`NOPROTO` 或 `UNIMPL`。`STD` 是标准系统调用，具有完整的原型和实现。`OBSOL` 是过时的，仅定义原型。`NOPROTO` 表示系统调用在其他地方实现，因此不需要添加 ABI 前缀等。`UNIMPL` 表示该系统调用将被 `nosys` 系统调用替代（该系统调用仅打印一条关于该系统调用未实现的消息，并返回 `ENOSYS`）。

从 **syscalls.master** 文件中，脚本会生成三个文件：**linux_syscall.h**、**linux_proto.h** 和 **linux_sysent.c**。**linux_syscall.h** 包含系统调用名称及其数值的定义，例如：

```c
...
#define LINUX_SYS_linux_fork 2
...
#define LINUX_SYS_close 6
...
```

**linux_proto.h** 包含每个系统调用的参数结构定义，例如：

```c
struct linux_fork_args {
  register_t dummy;
};
```

最后，**linux_sysent.c** 包含描述系统入口表的结构，该表用于实际调度系统调用，例如：

```c
{ 0, (sy_call_t *)linux_fork, AUE_FORK, NULL, 0, 0 }, /* 2 = linux_fork */
{ AS(close_args), (sy_call_t *)close, AUE_CLOSE, NULL, 0, 0 }, /* 6 = close */
```

如你所见，`linux_fork` 是在 Linuxulator 中实现的，因此其定义为 `STD` 类型，并且没有参数，这通过虚拟参数结构来体现。另一方面，`close` 只是 FreeBSD [close(2)](https://man.freebsd.org/cgi/man.cgi?query=close&sektion=2&format=html) 的别名，因此没有与 Linux 参数结构相关联，并且在系统入口表中没有添加 `linux_` 前缀，因为它调用的是内核中的真实 [close(2)](https://man.freebsd.org/cgi/man.cgi?query=close&sektion=2&format=html)。

#### 4.1.3. 虚拟系统调用

Linux® 仿真层并不完整，因为某些系统调用没有正确实现，有些则根本没有实现。仿真层通过 `DUMMY` 宏标记未实现的系统调用。这些虚拟定义存在于 **linux_dummy.c** 中，形式为 `DUMMY(syscall);`，然后会被转换到各种系统调用辅助文件中，实际实现就是打印一条消息，表明该系统调用未实现。`UNIMPL` 原型没有使用，因为我们希望能够识别被调用的系统调用名称，以了解哪些系统调用更需要实现。

### 4.2. 信号处理

信号处理在 FreeBSD 内核中通常是针对所有二进制兼容性进行的，调用了与兼容性相关的层。Linux® 兼容性层定义了 `linux_sendsig` 例程来处理这一任务。

#### 4.2.1. Linux® sendsig

该例程首先检查信号是否已安装并带有 `SA_SIGINFO`，如果是，它会调用 `linux_rt_sendsig` 例程。除此之外，它会分配（或重用已存在的）信号处理上下文，然后为信号处理程序构建参数列表。它根据信号翻译表翻译信号编号，分配处理程序，翻译信号集。接着，它为 `sigreturn` 例程保存上下文（各种寄存器、翻译后的陷阱号和信号掩码）。最后，它将信号上下文复制到用户空间并准备实际运行的信号处理程序上下文。

#### 4.2.2. linux_rt_sendsig

这个例程与 `linux_sendsig` 类似，只是信号上下文的准备过程不同。它添加了 `siginfo`、`ucontext` 和一些 POSIX® 部分。值得考虑的是，是否可以将这两个函数合并，以减少代码重复，并可能提高执行效率。

#### 4.2.3. linux_sigreturn

这个系统调用用于从信号处理程序返回。它执行一些安全检查并恢复原始进程上下文，同时取消对进程信号掩码的屏蔽。

### 4.3. Ptrace

许多 UNIX® 派生系统实现了 [ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 系统调用，以允许各种跟踪和调试功能。该功能使得跟踪进程能够获取关于被跟踪进程的各种信息，如寄存器转储、进程地址空间中的任何内存等，并且能够像单步执行指令或在系统调用和陷阱之间进行跟踪一样跟踪进程。[ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 还允许你设置被跟踪进程中的各种信息（如寄存器等）。[ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 是一个 UNIX® 广泛使用的标准，在大多数 UNIX® 系统中都有实现。

在 FreeBSD 中，Linux® 仿真层实现了 [ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 功能，代码位于 **linux_ptrace.c** 文件中。该例程负责在 Linux® 和 FreeBSD 之间转换寄存器，并执行实际的 [ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 系统调用仿真。该系统调用是一个长的 switch 块，为每个 [ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 命令实现了 FreeBSD 对应的功能。[ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 命令在 Linux® 和 FreeBSD 之间大多是相同的，因此通常只需要做少量修改。例如，Linux® 中的 `PT_GETREGS` 直接操作数据，而 FreeBSD 使用指向数据的指针，因此在执行（本地）[ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 系统调用后，必须执行 `copyout` 以保持 Linux® 的语义。

Linuxulator 中的 [ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 实现存在一些已知的缺陷。使用 `strace`（它是 [ptrace(2)](https://man.freebsd.org/cgi/man.cgi?query=ptrace&sektion=2&format=html) 的消费方）时，曾经发生过 panic 错误。同时，`PT_SYSCALL` 也没有实现。

### 4.4. 陷阱

当在仿真层中运行的 Linux® 进程发生陷阱时，陷阱会被透明地处理，唯一的例外是陷阱的翻译。Linux® 和 FreeBSD 对于什么是陷阱有不同的看法，因此在这里进行处理。代码实际上非常简短：

```
static int
translate_traps(int signal, int trap_code)
{

  if (signal != SIGBUS)
    return signal;

  switch (trap_code) {

    case T_PROTFLT:
    case T_TSSFLT:
    case T_DOUBLEFLT:
    case T_PAGEFLT:
      return SIGSEGV;

    default:
      return signal;
  }
}
```

### 4.5. 栈修复

RTLD（运行时链接编辑器）期望在 `execve` 调用期间栈上存在所谓的 AUX 标签，因此必须进行修复以确保这一点。当然，每个 RTLD 系统都是不同的，因此仿真层必须提供自己的栈修复例程来执行此操作。Linuxulator 也不例外。`elf_linux_fixup` 简单地将 AUX 标签复制到栈上，并调整用户空间进程的栈，使其指向这些标签之后的位置。这样 RTLD 就能正常工作。

### 4.6. A.OUT 支持

在 i386 上，Linux® 仿真层还支持 Linux® A.OUT 二进制文件。前面描述的大多数内容（除了陷阱翻译和信号发送）都必须实现以支持 A.OUT。A.OUT 二进制文件的支持已经不再维护，特别是 2.6 版本的仿真并不支持它，但这并不会造成问题，因为在 Ports 中的 linux-base 可能根本不支持 A.OUT 二进制文件。此支持可能会在未来移除。加载 Linux® A.OUT 二进制文件所需的大部分内容都在 **imgact_linux.c** 文件中。

## 5. Linux® 仿真层——机器无关部分

本节讨论 Linuxulator 的机器无关部分。它涵盖了 Linux® 2.6 仿真所需的仿真基础设施、线程局部存储（TLS）实现（在 i386 上）以及 futex。然后简要讨论一些系统调用。

### 5.1. NPTL 的描述

Linux® 2.6 发展中的一个主要进展领域是线程支持。在 2.6 之前，Linux® 线程支持是通过 linuxthreads 库实现的。该库是 POSIX® 线程的部分实现。线程是通过为每个线程创建单独的进程来实现的，使用 `clone` 系统调用让它们共享地址空间（以及其他资源）。这种方法的主要弱点是每个线程都有不同的 PID，信号处理出现问题（从 pthreads 角度看），等等。并且性能不佳（使用 `SIGUSR` 信号进行线程同步，内核资源消耗等）。为了解决这些问题，开发了一个新的线程系统，命名为 NPTL。

NPTL 库专注于两件事，但第三件事也随之而来，因此通常认为它是 NPTL 的一部分。这两件事是将线程嵌入到进程结构中和 futex。第三件事是 TLS，虽然它不是 NPTL 直接要求的，但整个 NPTL 用户空间库都依赖于它。这些改进大大提升了性能并符合标准。NPTL 目前是 Linux® 系统中的标准线程库。

FreeBSD Linuxulator 的实现通过三个主要领域来接近 NPTL：TLS、futex 和 PID 混淆，这些是为了仿真 Linux® 线程。接下来的章节将描述这些领域。

### 5.2. Linux® 2.6 仿真基础设施

这些部分讨论了 Linux® 线程是如何管理的，以及我们如何在 FreeBSD 中仿真这一过程。

#### 5.2.1. 运行时确定 2.6 仿真版本

FreeBSD 中的 Linux® 仿真层支持在运行时设置仿真版本。这是通过 [sysctl(8)](https://man.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&format=html) 完成的，具体是 `compat.linux.osrelease`。设置该 [sysctl(8)](https://man.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&format=html) 会影响仿真层的运行时行为。当设置为 2.6.x 时，它会设置 `linux_use_linux26` 变量，而设置为其他值则保持未设置状态。这个变量（加上与之相关的每个 prison 变量）决定是否在代码中使用 2.6 基础设施（主要是 PID 混淆）。版本设置是系统范围的，影响所有 Linux® 进程。运行任何 Linux® 二进制文件时不应更改该 [sysctl(8)](https://man.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&format=html)，因为这可能会导致问题。

#### 5.2.2. Linux® 进程和线程标识符

Linux® 线程的语义有些令人困惑，并且使用与 FreeBSD 完全不同的术语。Linux® 中的一个进程由一个 `struct task` 组成，其中嵌套了两个标识符字段——PID 和 TGID。PID *不是* 进程 ID，而是线程 ID。TGID 用来标识一个线程组，也就是一个进程。对于单线程进程，PID 等于 TGID。

在 NPTL 中，线程只是一个普通的进程，只是它的 TGID 不等于 PID，并且其组领导者不等于它本身（当然，还共享虚拟内存等）。其他一切和普通进程相同。并没有像 FreeBSD 中那样将共享状态分离到某个外部结构中。这就导致了信息的重复和可能的数据不一致。Linux® 内核似乎在某些地方使用 task → group 信息，而在其他地方使用 task 信息，这种做法并不一致，容易出错。

每个 NPTL 线程都是通过调用 `clone` 系统调用并传入一组特定的标志来创建的（更多内容见下一小节）。NPTL 实现了严格的 1:1 线程模型。

在 FreeBSD 中，我们通过普通的 FreeBSD 进程来仿真 NPTL 线程，这些进程共享虚拟内存空间等，而 PID 操作则通过附加到进程的仿真特定结构来仿真。附加到进程的结构如下所示：

```c
struct linux_emuldata {
  pid_t pid;

  int *child_set_tid; /* 在 clone() 中：子进程的 TID 设置地址 */
  int *child_clear_tid;/* 在 clone() 中：子进程的 TID 清除地址 */

  struct linux_emuldata_shared *shared;

  int pdeath_signal; /* 父进程死亡信号 */

  LIST_ENTRY(linux_emuldata) threads; /* linux 线程的链表 */
};
```

PID 用于标识附加此结构的 FreeBSD 进程。`child_set_tid` 和 `child_clear_tid` 用于在进程退出和创建时进行 TID 地址的拷贝。`shared` 指针指向一个在线程之间共享的结构体。`pdeath_signal` 变量标识父进程死亡信号，而 `threads` 指针用于将此结构链接到线程列表中。`linux_emuldata_shared` 结构如下所示：

```c
struct linux_emuldata_shared {

  int refs;

  pid_t group_pid;

  LIST_HEAD(, linux_emuldata) threads; /* linux 线程链表的头部 */
};
```

`refs` 是一个引用计数器，用于确定何时可以释放结构体，以避免内存泄漏。`group_pid` 用于标识整个进程（即线程组）的 PID（=TGID）。`threads` 指针是该进程中所有线程的链表头。

可以通过 `em_find` 函数从进程中获取 `linux_emuldata` 结构。该函数的原型如下：

```c
struct linux_emuldata *em_find(struct proc *, int locked);
```

其中，`proc` 是我们想要从中获取仿真数据结构的进程，`locked` 参数决定是否需要加锁。接受的值为 `EMUL_DOLOCK` 和 `EMUL_DOUNLOCK`。关于加锁的内容稍后讨论。

#### 5.2.3. PID 混淆

由于 FreeBSD 和 Linux® 在进程 ID 和线程 ID 的概念上有所不同，我们必须以某种方式转换这种视图。我们通过 PID 混淆来实现这一点。这意味着我们在内核和用户空间之间伪造 PID（=TGID）和 TID（=PID）。大致规则是，在内核中（在 Linuxulator 中）PID = PID，而 TGID = shared → group_pid，在用户空间中，我们呈现 `PID = shared → group_pid` 和 `TID = proc → p_pid`。`linux_emuldata` 结构中的 PID 是一个 FreeBSD PID。

上述内容主要影响 `getpid`、`getppid` 和 `gettid` 系统调用。在这些调用中，我们分别使用 PID/TGID。在 `child_clear_tid` 和 `child_set_tid` 的 TID 拷贝中，我们拷贝的是 FreeBSD PID。

#### 5.2.4. Clone 系统调用

`clone` 系统调用是 Linux® 中创建线程的方式。该系统调用的原型如下所示：

```c
int linux_clone(l_int flags, void *stack, void *parent_tidptr, int dummy,
void * child_tidptr);
```

`flags` 参数告诉系统调用进程应该如何被克隆。如前所述，Linux® 可以创建共享各种资源的进程，例如两个进程可以共享文件描述符但不共享虚拟内存等。`flags` 参数的最后一个字节是新创建进程的退出信号。`stack` 参数如果非 `NULL`，则指示线程栈的位置；如果为 `NULL`，则表示我们应该进行写时复制调用进程的栈（即执行普通的 [fork(2)](https://man.freebsd.org/cgi/man.cgi?query=fork&sektion=2&format=html) 操作）。`parent_tidptr` 参数用于在进程足够初始化但尚未可运行时，将进程 PID（即线程 ID）拷贝到指定地址。`dummy` 参数是因为该系统调用在 i386 架构上的调用约定非常奇怪，直接使用寄存器而不让编译器处理，从而需要一个虚拟的系统调用。`child_tidptr` 参数用于在进程完成 fork 操作并退出时，拷贝 PID。

该系统调用通过设置传入的标志来执行相应的操作。例如，`CLONE_VM` 对应于 RFMEM（虚拟内存共享）等。唯一的细节是 `CLONE_FS` 和 `CLONE_FILES`，因为 FreeBSD 不允许单独设置这些标志，所以我们通过不设置 RFFDG（复制文件描述符表和其他文件系统信息）来伪造这种行为。由于这些标志总是一起设置，所以这不会引起任何问题。设置完标志后，进程通过内部 `fork1` 例程进行 fork，进程被设置为不可运行，即不会加入到运行队列中。在完成 fork 后，我们可能会将新创建的进程重新父化，以仿真 `CLONE_PARENT` 语义。接下来是创建仿真数据。Linux® 中的线程不会向其父进程发送信号，因此我们将退出信号设置为 0 以禁用此功能。然后设置 `child_set_tid` 和 `child_clear_tid`，以便在后续代码中启用这些功能。此时，我们将 PID 拷贝到 `parent_tidptr` 指定的地址。进程栈的设置通过简单地重写线程帧中的 `%esp` 寄存器（在 amd64 上是 `%rsp`）来完成。接下来是为新创建的进程设置 TLS。之后，可以仿真 [vfork(2)](https://man.freebsd.org/cgi/man.cgi?query=vfork&sektion=2&format=html) 的语义，最后将新创建的进程加入运行队列，并通过 `clone` 返回值将其 PID 拷贝到父进程中。

`clone` 系统调用实际上也用于仿真传统的 [fork(2)](https://man.freebsd.org/cgi/man.cgi?query=fork&sektion=2&format=html) 和 [vfork(2)](https://man.freebsd.org/cgi/man.cgi?query=vfork&sektion=2&format=html) 系统调用。在 2.6 内核版本中，更新后的 glibc 使用 `clone` 来实现 [fork(2)](https://man.freebsd.org/cgi/man.cgi?query=fork&sektion=2&format=html) 和 [vfork(2)](https://man.freebsd.org/cgi/man.cgi?query=vfork&sektion=2&format=html) 系统调用。

#### 5.2.5. 锁

锁的实现是按子系统来进行的，因为我们不期望会有大量的竞争。这里有两个锁：`emul_lock` 用于保护 `linux_emuldata` 的操作，`emul_shared_lock` 用于操作 `linux_emuldata_shared`。`emul_lock` 是一个不可睡眠的阻塞互斥锁，而 `emul_shared_lock` 是一个可睡眠的阻塞 `sx_lock`。由于按子系统锁定的方式，我们可以合并一些锁，因此 `em_find` 提供了不锁定的访问方式。

### 5.3. TLS

本节讨论 TLS，即线程局部存储。

#### 5.3.1. 线程简介

计算机科学中的线程是进程中的实体，它们可以独立于其他线程进行调度。进程中的线程共享进程范围的数据（如文件描述符等），但每个线程也有自己的栈来存储线程数据。有时需要线程特定的进程范围数据。想象一下，正在执行的线程的名称之类的东西。传统的 UNIX® 线程 API，pthread 提供了一种方法，可以通过 [pthread_key_create(3)](https://man.freebsd.org/cgi/man.cgi?query=pthread_key_create&sektion=3&format=html)、[pthread_setspecific(3)](https://man.freebsd.org/cgi/man.cgi?query=pthread_setspecific&sektion=3&format=html) 和 [pthread_getspecific(3)](https://man.freebsd.org/cgi/man.cgi?query=pthread_getspecific&sektion=3&format=html) 来创建线程局部数据的键，并使用 [pthread_getspecific(3)](https://man.freebsd.org/cgi/man.cgi?query=pthread_getspecific&sektion=3&format=html) 或 [pthread_getspecific(3)](https://man.freebsd.org/cgi/man.cgi?query=pthread_getspecific&sektion=3&format=html) 来操作这些数据。可以很容易地看出，这不是实现此功能的最方便方法。因此，许多 C/C++ 编译器的生产商引入了一种更好的方式。他们定义了一个新的修饰符关键字 `thread`，用于指定变量是线程特定的。还开发了访问此类变量的新方法（至少在 i386 上）。pthread 方法通常在用户空间实现为一个简单的查找表。这种解决方案的性能不太好。因此，新方法使用（在 i386 上）段寄存器来访问一个存储 TLS 区域的段，这样实际访问线程变量就像附加段寄存器到地址一样，从而通过它进行寻址。段寄存器通常是 `%gs` 和 `%fs`，它们充当段选择器。每个线程都有自己存储线程局部数据的区域，并且在每次上下文切换时都必须加载该段。此方法非常快速，几乎在整个 i386 UNIX® 世界中得到了广泛应用。FreeBSD 和 Linux® 都实现了这种方法，并且取得了非常好的效果。唯一的缺点是每次上下文切换时需要重新加载段，这可能会减慢上下文切换的速度。FreeBSD 通过只使用 1 个段描述符来避免这种开销，而 Linux® 使用了 3 个。值得注意的是，几乎没有使用多个描述符的情况（只有 Wine 似乎使用了 2 个），因此 Linux® 在上下文切换时付出了不必要的代价。

#### 5.3.2. i386 上的段

i386 架构实现了所谓的段。段是描述一块内存区域的结构。该内存区域的基址（底部）、结束地址（顶端）、类型、保护等信息。可以使用段选择器寄存器（`%cs`、`%ds`、`%ss`、`%es`、`%fs`、`%gs`）来访问该内存区域。例如，假设我们有一个基地址为 0x1234 且长度为 0x1000 的段，执行以下代码：

```c
mov %edx,%gs:0x10
```

这将把 `%edx` 寄存器的内容加载到内存位置 0x1244。一些段寄存器有特殊用途，例如 `%cs` 用于代码段，`%ss` 用于栈段，但 `%fs` 和 `%gs` 通常未被使用。段通常存储在全局 GDT 表中，或者存储在本地 LDT 表中。LDT 通过 GDT 中的一个条目来访问。LDT 可以存储更多类型的段，可以为每个进程单独设置 LDT。两个表最多可以定义 8191 个条目。

#### 5.3.3. 在 Linux® i386 上的实现

在 Linux® 中设置 TLS 主要有两种方式。可以在克隆进程时通过 `clone` 系统调用设置，或者可以调用 `set_thread_area`。当进程传递 `CLONE_SETTLS` 标志给 `clone` 时，内核期望 `%esi` 寄存器指向 Linux® 用户空间表示的段，该段被转换为机器表示的段，并加载到 GDT 插槽中。可以使用数字指定 GDT 插槽，也可以使用 -1，表示系统应该选择第一个空闲插槽。实际上，大多数程序只使用一个 TLS 项目，并且不关心条目的数量。我们在仿真中利用了这一点，实际上也依赖于这一点。

#### 5.3.4. Linux® TLS 的仿真

##### 5.3.4.1. i386

当前线程的 TLS 加载通过调用 `set_thread_area` 实现，而第二个进程的 TLS 加载则在 `clone` 中的单独块中完成。这两个函数非常相似，唯一的区别是 GDT 段的实际加载，这发生在为新创建的进程进行下一个上下文切换时，而 `set_thread_area` 必须直接加载该段。代码基本上是这样做的：它将 Linux® 形式的段描述符从用户空间复制出来，检查描述符的数量，但由于 FreeBSD 和 Linux® 之间的差异，我们对其进行伪造。我们只支持 6、3 和 -1 索引。6 是 Linux® 的真实数字，3 是 FreeBSD 的真实数字，-1 表示自动选择。然后我们将描述符号设置为常数 3，并将其复制到用户空间。我们依赖于用户空间进程使用来自描述符的数字，但大多数情况下这没有问题（我们从未见过不工作的情况），因为用户空间进程通常传入 1。接着我们将描述符从 Linux® 形式转换为机器无关的形式（即操作系统无关形式），并将其复制到 FreeBSD 定义的段描述符中。最后，我们加载它。我们将该描述符分配给线程的 PCB（进程控制块），并使用 `load_gs` 加载 `%gs` 段。加载必须在临界区内进行，以确保没有中断我们。`CLONE_SETTLS` 情况下，操作与此完全相同，只是加载操作使用 `load_gs` 并没有执行。此用途的段（段号 3）在 FreeBSD 进程和 Linux® 进程之间共享，因此 Linux® 仿真层不会比纯 FreeBSD 增加额外的开销。

##### 5.3.4.2. amd64

amd64 的实现与 i386 类似，但最初没有为此目的使用 32 位段描述符（因此甚至本地 32 位 TLS 用户也无法工作），所以我们不得不添加这样的段并在每次上下文切换时实现其加载（当设置了使用 32 位的标志时）。除此之外，TLS 加载完全相同，只是段号不同，描述符格式和加载方式稍有不同。

### 5.4. Futex

#### 5.4.1. 同步介绍

线程需要某种同步机制，而 POSIX® 提供了其中一些：用于互斥的互斥锁、具有偏置读写比率的读写锁以及用于信号状态变化的条件变量。有趣的是，POSIX® 线程 API 不支持信号量。同步例程的实现很大程度上依赖于我们拥有的线程支持类型。在纯 1:M（用户空间）模型中，实施可以完全在用户空间进行，因此非常快速（条件变量可能最终通过信号实现，即不是非常快速），并且很简单。在 1:1 模型中，情况也很清楚——线程必须使用内核设施进行同步（这非常慢，因为必须执行系统调用）。混合的 M:N 场景则结合了前两种方法，或者完全依赖于内核。线程同步是线程启用编程的一个关键部分，它的性能可能极大地影响结果程序。最近 FreeBSD 操作系统的基准测试表明，改进的 sx_lock 实现使 *ZFS*（一个重度使用 sx 的应用）性能提高了 40%，这是内核中的内容，但它清楚地表明了同步原语的性能是多么重要。

线程程序应尽量减少对锁的竞争。否则，线程只是等待锁，而没有做有用的工作。因此，编写得最好的线程程序通常表现出较少的锁竞争。

#### 5.4.2. Futex 介绍

Linux® 实现了 1:1 线程模型，即必须使用内核同步原语。如前所述，编写得很好的线程程序锁竞争较少。因此，一个典型的序列可能是执行两次原子增加/减少互斥锁引用计数，这非常快速，如下面的示例所示：

```c
pthread_mutex_lock(&mutex);
...
pthread_mutex_unlock(&mutex);
```

1:1 线程模型迫使我们为这些互斥锁调用执行两个系统调用，这非常慢。

Linux® 2.6 实现的解决方案被称为 futexes。Futexes 在用户空间中检查是否发生竞争，仅在发生竞争时才调用内核原语。因此，典型情况下不会有内核干预。这使得同步原语的实现既快速又灵活。

#### 5.4.3. Futex API

futex 系统调用的原型如下所示：

```c
int futex(void *uaddr, int op, int val, struct timespec *timeout, void *uaddr2, int val3);
```

在这个例子中，`uaddr` 是用户空间中互斥锁的地址，`op` 是我们要执行的操作，其他参数根据操作的不同有不同的含义。

Futexes 实现了以下操作：

- `FUTEX_WAIT`
- `FUTEX_WAKE`
- `FUTEX_FD`
- `FUTEX_REQUEUE`
- `FUTEX_CMP_REQUEUE`
- `FUTEX_WAKE_OP`

##### 5.4.3.1. FUTEX_WAIT

此操作验证在地址 `uaddr` 处的值是否为 `val`。如果不是，返回 `EWOULDBLOCK`，否则线程会被排队到 futex 并被挂起。如果 `timeout` 参数非零，它指定最大睡眠时间，否则睡眠将是无限期的。

##### 5.4.3.2. FUTEX_WAKE

此操作会在 `uaddr` 处取出一个 futex，并唤醒排队在此 futex 上的 `val` 个 futex。

##### 5.4.3.3. FUTEX_FD

此操作将文件描述符与给定的 futex 关联。

##### 5.4.3.4. FUTEX_REQUEUE

此操作会从 `uaddr` 处的 futex 上唤醒 `val` 个排队的线程，并将 `val2` 个线程重新排队到 `uaddr2` 处的 futex 上。

##### 5.4.3.5. FUTEX_CMP_REQUEUE

此操作与 `FUTEX_REQUEUE` 相同，但首先会检查 `val3` 是否等于 `val`。

##### 5.4.3.6. FUTEX_WAKE_OP

该操作对 `val3`（其中包含某些其他值）和 `uaddr` 执行原子操作。然后，它唤醒 `uaddr` 上的 `val` 个 futex 线程，如果原子操作返回一个正数，则它会唤醒 `uaddr2` 上的 `val2` 个 futex 线程。

`FUTEX_WAKE_OP` 中实现的操作如下：

- `FUTEX_OP_SET`
- `FUTEX_OP_ADD`
- `FUTEX_OP_OR`
- `FUTEX_OP_AND`
- `FUTEX_OP_XOR`

>**注意**
>
>futex 原型中没有 `val2` 参数。`val2` 通过 `struct timespec *timeout` 参数获取，适用于操作 `FUTEX_REQUEUE`、`FUTEX_CMP_REQUEUE` 和 `FUTEX_WAKE_OP`。

#### 5.4.4. Futex 在 FreeBSD 中的仿真

FreeBSD 中的 futex 仿真来源于 NetBSD，并由我们进一步扩展。它位于 `linux_futex.c` 和 **linux_futex.h** 文件中。`futex` 结构如下：

```c
struct futex {
  void *f_uaddr;
  int f_refcount;

  LIST_ENTRY(futex) f_list;

  TAILQ_HEAD(lf_waiting_paroc, waiting_proc) f_waiting_proc;
};
```

`waiting_proc` 结构如下：

```c
struct waiting_proc {

  struct thread *wp_t;

  struct futex *wp_new_futex;

  TAILQ_ENTRY(waiting_proc) wp_list;
};
```

##### 5.4.4.1. futex_get / futex_put

使用 `futex_get` 函数获取 futex，它在 futex 的线性列表中搜索并返回找到的 futex，或者创建一个新的 futex。释放 futex 时调用 `futex_put` 函数，该函数减少 futex 的引用计数，如果引用计数为零，则释放该 futex。

##### 5.4.4.2. futex_sleep

当 futex 将线程排队等待时，它会创建一个 `working_proc` 结构，并将该结构放入 futex 结构中的列表中，然后执行一个 [tsleep(9)](https://man.freebsd.org/cgi/man.cgi?query=tsleep&sektion=9&format=html) 来挂起线程。这个睡眠可以有超时限制。在 [tsleep(9)](https://man.freebsd.org/cgi/man.cgi?query=tsleep&sektion=9&format=html) 返回后（线程被唤醒或超时），`working_proc` 结构从列表中移除并销毁。所有这些操作都在 `futex_sleep` 函数中完成。如果线程是通过 `futex_wake` 被唤醒的，我们会设置 `wp_new_futex`，因此我们会在它上面睡眠。这样，实际的重新排队是在这个函数中完成的。

##### 5.4.4.3. futex_wake

在 `futex_wake` 函数中唤醒在 futex 上睡眠的线程。首先，我们在此函数中模仿了 Linux® 的奇怪行为，所有操作都会唤醒 N 个线程，唯一的例外是 REQUEUE 操作，它会唤醒 N+1 个线程。但通常这不会产生任何差异，因为我们唤醒了所有线程。接下来，在循环中唤醒 n 个线程后，我们会检查是否有新的 futex 需要重新排队。如果有，我们会将最多 n2 个线程重新排队到新的 futex 上。这与 `futex_sleep` 协作。

##### 5.4.4.4. futex_wake_op

`FUTEX_WAKE_OP` 操作比较复杂。首先，我们获取地址为 `uaddr` 和 `uaddr2` 的两个 futex，然后使用 `val3` 和 `uaddr2` 执行原子操作。然后，唤醒 `val` 个等待在第一个 futex 上的线程，如果原子操作条件成立，我们会唤醒 `val2`（即 `timeout`）个等待在第二个 futex 上的线程。

##### 5.4.4.5. futex 原子操作

原子操作接受两个参数：`encoded_op` 和 `uaddr`。编码操作将操作本身、比较值、操作参数和比较参数进行编码。该操作的伪代码如下所示：

```c
oldval = *uaddr2
*uaddr2 = oldval OP oparg
```

这个操作是原子执行的。首先，从 `uaddr` 复制数字，然后执行操作。代码处理了页错误，如果没有发生页错误，则 `oldval` 会与 `cmparg` 参数进行比较，使用指定的比较操作符。

##### 5.4.4.6. Futex 锁定

Futex 实现使用了两种锁列表来保护 `sx_lock` 和全局锁（无论是 Giant 还是其他 `sx_lock`）。每个操作从开始到结束都在锁定状态下进行。

### 5.5. 各种系统调用实现

在本节中，我将描述一些较小的系统调用，这些系统调用的实现比较特殊，或者从其他角度来看，它们的实现值得关注。

#### 5.5.1. *at 系列系统调用

在 Linux® 2.6.16 内核的开发过程中，添加了 *at 系列的系统调用。例如，`openat` 就是这样一个系统调用，它的工作方式与没有 `at` 的对应调用几乎相同，唯一的区别在于 `dirfd` 参数。该参数决定了给定文件的位置。当 `filename` 参数是绝对路径时，`dirfd` 会被忽略，但当文件路径是相对路径时，`dirfd` 就变得非常重要。`dirfd` 参数是一个目录，相对路径名将相对于该目录进行检查。`dirfd` 参数是某个目录的文件描述符，或者是 `AT_FDCWD`。例如，`openat` 系统调用可以像下面这样：

```
文件描述符 123 = /tmp/foo/, 当前工作目录 = /tmp/

openat(123, /tmp/bah, flags, mode)    /* 打开 /tmp/bah */
openat(123, bah, flags, mode)         /* 打开 /tmp/foo/bah */
openat(AT_FDCWD, bah, flags, mode)    /* 打开 /tmp/bah */
openat(stdio, bah, flags, mode)       /* 返回错误，因为 stdio 不是一个目录 */
```

这种机制的必要性在于避免在打开文件时发生竞争条件，尤其是在工作目录之外。假设一个进程包含两个线程，线程 A 和线程 B。线程 A 执行 `open(./tmp/foo/bah, flags, mode)` 系统调用，但在返回之前，线程 A 被抢占，接着线程 B 执行并不关心线程 A 的操作，可能会重命名或删除 **/tmp/foo/** 目录。这样就会发生竞争条件。为了解决这个问题，我们可以先打开 **/tmp/foo** 目录，并使用它作为 `dirfd`，然后再执行 `openat` 系统调用。这样还可以实现每个线程的工作目录。

Linux® 系列的 *at 系统调用包括：`linux_openat`、`linux_mkdirat`、`linux_mknodat`、`linux_fchownat`、`linux_futimesat`、`linux_fstatat64`、`linux_unlinkat`、`linux_renameat`、`linux_linkat`、`linux_symlinkat`、`linux_readlinkat`、`linux_fchmodat` 和 `linux_faccessat`。这些调用都通过修改后的 [namei(9)](https://man.freebsd.org/cgi/man.cgi?query=namei&sektion=9&format=html) 例程和简单的包装层实现。

##### 5.5.1.1. 实现

实现是通过修改 [namei(9)](https://man.freebsd.org/cgi/man.cgi?query=namei&sektion=9&format=html) 例程来完成的，给其 `nameidata` 结构添加了 `dirfd` 参数，这个参数指定路径名查找的起始点，而不是每次都使用当前工作目录。`dirfd` 的解析过程从文件描述符号到 vnode 的过程是在本地 *at 系统调用中完成的。当 `dirfd` 是 `AT_FDCWD` 时，`nameidata` 结构中的 `dvp` 项为 `NULL`，但当 `dirfd` 是其他数值时，我们会为这个文件描述符获取一个文件，检查它是否有效，如果有 vnode 附加到它，我们就获取这个 vnode，然后检查这个 vnode 是否是一个目录。在实际的 [namei(9)](https://man.freebsd.org/cgi/man.cgi?query=namei&sektion=9&format=html) 例程中，我们简单地将 `dp` 变量替换为 `dvp`，它决定了起始点。`namei(9)` 不是直接使用的，而是通过不同层级的函数调用。举个例子，`openat` 的调用过程如下：

```
openat() --> kern_openat() --> vn_open() -> namei()
```

因此，`kern_open` 和 `vn_open` 必须做相应的修改，以支持额外的 `dirfd` 参数。对于这些系统调用，并没有创建兼容层，因为它们的用户不多，且用户可以容易地进行转换。这种通用的实现方式使 FreeBSD 可以实现自己的 *at 系统调用。目前，这项工作仍在讨论中。

#### 5.5.2. Ioctl

`ioctl` 接口由于其通用性而变得相当脆弱。我们必须考虑到设备在 Linux® 和 FreeBSD 之间的差异，因此在执行 `ioctl` 仿真时必须小心。`ioctl` 的处理在 **linux_ioctl.c** 文件中实现，其中定义了 `linux_ioctl` 函数。该函数简单地遍历一组 `ioctl` 处理程序，查找能够实现给定命令的处理程序。`ioctl` 系统调用有三个参数：文件描述符、命令和一个参数。命令是一个 16 位数字，理论上高 8 位确定命令的类别，低 8 位则是该类别内的具体命令。仿真程序利用这个划分实现。我们为每个类别实现处理程序，比如 `sound_handler` 或 `disk_handler`。每个处理程序都有最大和最小命令的定义，用于确定使用哪个处理程序。尽管如此，这种方法也存在一些问题，因为 Linux® 并不总是按类别划分 `ioctl` 命令，有时不同类别的 `ioctl` 会被放在不应属于的类别中（例如，将 SCSI 通用 `ioctl` 放到 cdrom 类别中）。目前，FreeBSD 并没有实现很多 Linux® 的 `ioctl`（与 NetBSD 相比），但计划从 NetBSD 移植一些。趋势是，即便在 FreeBSD 原生驱动中，也开始使用 Linux® 的 `ioctl`，因为这使得应用程序的移植变得更加容易。

#### 5.5.3. 调试

每个系统调用都应该支持调试。为此，我们引入了一个小的调试基础设施。我们有一个 `ldebug` 功能，它会告知是否应对某个系统调用进行调试（可以通过 sysctl 设置）。为了打印调试信息，我们使用了 `LMSG` 和 `ARGS` 宏。这些宏用于调整打印的字符串，以实现统一的调试信息输出。

## 6. 结论

### 6.1. 结果

截至 2007 年 4 月，Linux® 仿真层已经能够相当好地仿真 Linux® 2.6.16 内核。剩余的问题主要集中在 futex、未完成的 *at 系列系统调用、信号传递问题、缺失的 `epoll` 和 `inotify`，以及可能存在的未发现的 BUG。尽管如此，我们已经能够运行几乎所有 FreeBSD Ports 中包含的 Linux® 程序（如 Fedora Core 4 版本的 2.6.16），并且有一些成功的初步报告表明 Fedora Core 6 版本的 2.6.16 也能运行。最近提交的 Fedora Core 6 `linux_base` 使得我们能够进一步测试仿真层，并为我们提供了一些线索，指出了在哪些地方我们应该投入更多精力来实现缺失的功能。

我们能够运行一些常用的应用程序，比如 [www/linux-firefox](https://cgit.freebsd.org/ports/tree/www/linux-firefox/)、[net-im/skype](https://cgit.freebsd.org/ports/tree/net-im/skype/) 以及一些游戏。有些程序在 2.6 仿真下表现不佳，但这个问题目前正在调查中，并且希望很快能修复。唯一一个已知无法工作的主要应用程序是 Linux® Java™ 开发工具包，因为它需要 `epoll` 功能，而这与 Linux® 2.6 内核并不直接相关。

我们希望在 FreeBSD 7.0 发布后不久默认启用 2.6.16 仿真，以便进行更广泛的测试。完成这一切后，我们就可以切换到 Fedora Core 6 `linux_base`，这也是我们的最终计划。

### 6.2. 后续工作

未来的工作应集中在修复与 futex 相关的剩余问题，完成其余 *at 系列系统调用的实现，修复信号传递问题，并可能实现 `epoll` 和 `inotify` 功能。

我们希望能够尽快无缝运行最重要的程序，这样我们就能默认切换到 2.6 仿真，并将 Fedora Core 6 设为默认的 `linux_base`，因为我们当前使用的 Fedora Core 4 已不再受支持。

另一个可能的目标是与 NetBSD 和 DragonflyBSD 共享我们的代码。NetBSD 对 2.6 仿真有一些支持，但还远未完成且没有经过充分测试。DragonflyBSD 已表达出将 2.6 改进移植过来的兴趣。

一般来说，随着 Linux® 的发展，我们希望能跟进其发展，实施新添加的系统调用，首先想到的是 `splice`。一些已实现的系统调用也存在一些优化空间，例如 `mremap` 等。还可以进行一些性能改进，精细化锁等。

### 6.3. 团队

我在这个项目中与以下人员合作（按字母顺序排列）：

- John Baldwin <jhb@FreeBSD.org>
- Konstantin Belousov <kib@FreeBSD.org>
- Emmanuel Dreyfus
- Scot Hetzel
- Jung-uk Kim <jkim@FreeBSD.org>
- Alexander Leidinger <netchild@FreeBSD.org>
- Suleiman Souhlal <ssouhlal@FreeBSD.org>
- Li Xiao
- David Xu <davidxu@FreeBSD.org>

我想感谢所有这些人对我的建议、代码审查和一般支持。

## 7. 文献

1. Marshall Kirk McKusick - George V. Neville-Neil. 《FreeBSD 操作系统的设计与实现》。Addison-Wesley, 2005。
2. [https://tldp.org](https://tldp.org)
3. [https://www.kernel.org](https://www.kernel.org)
