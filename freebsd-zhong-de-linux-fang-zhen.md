# FreeBSD 中的 Linux® 仿真

<details open="" data-immersive-translate-walked="78bc16f3-c72d-449d-8fba-079e4c5b06f1"><summary data-immersive-translate-walked="78bc16f3-c72d-449d-8fba-079e4c5b06f1" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

IBM、AIX、OS/2、PowerPC、PS/2、S/390 和 ThinkPad 是国际商业机器公司在美国、其他国家或两者都是商标。

Adobe，Acrobat，Acrobat Reader，Flash 和 PostScript 是 Adobe Systems Incorporated 在美国和/或其他国家的注册商标或商标。

Linux 是 Linus Torvalds 的注册商标。

Sun，Sun Microsystems，Java，Java 虚拟机，JDK，JRE，JSP，JVM，Netra，OpenJDK，Solaris，StarOffice，SunOS 和 VirtualBox 是 Sun Microsystems，Inc.在美国和其他国家的商标或注册商标。

NetBSD 是 NetBSD 基金会的注册商标。

RealNetworks、RealPlayer 和 RealAudio 是 RealNetworks, Inc 的注册商标。

Oracle 是 Oracle 公司的注册商标。

制造商和销售商用来区分其产品的许多名称被声明为商标。在本文件中出现这些名称时，FreeBSD 项目已经意识到了商标声明，这些名称后面已经加上了“™”或“®”符号。

</details>

摘要

这篇硕士论文涉及更新 Linux® 模拟层（即所谓的 Linuxulator）。该任务是更新该层以匹配 Linux® 2.6 的功能。作为参考实现，选择了 Linux® 2.6.16 内核。该概念基本上是基于 NetBSD 的实现。大部分工作是在 2006 年夏季作为谷歌编程之夏学生项目的一部分完成的。重点是将 NPTL（新 POSIX® 线程库）支持带入模拟层，包括 TLS（线程本地存储）、futexes（快速用户空间互斥锁）、PID 篡改和一些其他小事情。在这个过程中，识别并修复了许多小问题。我的工作已经集成到主 FreeBSD 源代码库中，并将在即将发布的 7.0R 版本中发布。我们，模拟开发团队，正在努力使 Linux® 2.6 模拟成为 FreeBSD 中的默认模拟层。

---

## 1. 引言

在过去几年里，开源的基于 UNIX® 的操作系统开始广泛部署在服务器和客户端机器上。在这些操作系统中，我想要指出两个：FreeBSD，因为它的 BSD 传承、经过时间验证的代码库和许多有趣的功能；以及 Linux®，因为它拥有广泛的用户群体、充满热情的开发者社区以及大公司的支持。FreeBSD 倾向于在提供重型网络任务的服务器级机器上使用，而在桌面级普通用户机器上的使用较少。而 Linux® 在服务器上有同样的用途，但在家庭用户中使用更为广泛。这导致一个情况：有许多仅支持 Linux® 的二进制程序在 FreeBSD 上缺乏支持。

自然而然地，FreeBSD 系统上运行 Linux® 二进制程序的需求应运而生，这正是本文所涉及的内容：在 FreeBSD 操作系统中对 Linux® 内核进行仿真。

在 2006 年夏天，Google 公司赞助了一个项目，重点是扩展 FreeBSD 中的 Linux® 仿真层（所谓的 Linuxulator），以包括 Linux® 2.6 设施。 这篇论文是作为这个项目的一部分编写的。

## 2. 一探究竟…

在这一部分中，我们将描述每个操作系统。 它们如何处理系统调用、陷阱帧等等，所有底层内容。 我们还描述它们理解常见 UNIX® 原语的方式，比如 PID 是什么，线程是什么等等。 在第三小节中，我们谈论 UNIX® 对 UNIX® 仿真如何在一般情况下完成。

UNIX® 具有悠久历史的操作系统，影响了几乎所有当前正在使用的其他操作系统。从 20 世纪 60 年代开始，它的发展一直持续至今（尽管在不同的项目中）。UNIX® 的发展很快分成了两个主要的方向：BSD 和 System III/V 系列。它们通过制定一个共同的 UNIX® 标准相互影响。在 BSD 中产生的贡献包括虚拟内存、TCP/IP 网络、FFS 等。System V 分支为 SysV 进程间通信原语、写时复制等做出了贡献。UNIX® 本身不再存在，但其思想已被许多其他操作系统在全球范围内使用，从而形成了所谓的类 UNIX® 操作系统。如今，最具影响力的是 Linux®、Solaris，可能（在某种程度上）还有 FreeBSD。存在一些公司内部的 UNIX® 衍生产品（如 AIX、HP-UX 等），但这些产品已经越来越多地迁移到前述的系统上。让我们总结一下 UNIX® 的典型特征。

### 技术细节

每个运行中的程序都构成一个进程，代表了计算的某个状态。运行中的进程分为内核空间和用户空间。一些操作只能从内核空间完成（如处理硬件等），但进程大部分时间应该在用户空间中执行。内核是处理进程管理、硬件和底层细节的地方。内核为用户空间提供了标准统一的 UNIX® API。下面介绍了最重要的几个。

#### 2.2.1. 内核与用户空间进程之间的通信

常见的 UNIX® API 将系统调用定义为用户空间进程向内核发出命令的一种方式。最常见的实现方法是使用中断或专用指令（比如 ia32 架构下的 SYSENTER / SYSCALL 指令）。系统调用通过一个数字来定义。例如，在 FreeBSD 中，系统调用号 85 是 swapon(2) 系统调用，系统调用号 132 是 mkfifo(2)。某些系统调用需要参数，这些参数以各种方式（依赖于具体实现）从用户空间传递到内核空间。系统调用是同步的。

另一种可能的通信方式是使用陷阱。陷阱在某些事件发生后异步发生（例如除以零、页错误等）。陷阱对进程可以是透明的（页错误），也可以导致发送信号等反应（除以零）。

#### 2.2.2. 进程之间的通信

还有其他的 API（System V IPC，共享内存等），但最重要的 API 是信号。信号由进程或内核发送，被进程接收。某些信号可以被忽略或由用户提供的例程处理，某些会导致不能更改或忽略的预定义动作。

#### 2.2.3. 进程管理

内核实例首先在系统中处理（所谓的 init）。每个运行中的进程都可以使用 fork(2)系统调用创建其相同的副本。一些稍作修改的此系统调用版本被引入，但基本语义相同。每个运行中的进程可以使用 exec(3)系统调用变形为其他进程。此系统调用的一些修改被引入，但都有相同的基本目的。进程通过调用 exit(2)系统调用来结束其生命周期。每个进程由称为 PID 的唯一编号标识。每个进程都有一个定义的父进程（由其 PID 标识）。

#### 2.2.4. 线程管理

传统的 UNIX® 并未为线程定义任何 API 或实现，而 POSIX® 定义了其线程 API，但实现未定义。传统上有两种实现线程的方式。将它们处理为独立进程（1:1 线程）或将整个线程组包装在一个进程中，并在用户空间管理线程（1:N 线程）。比较每种方法的主要特点：

1:1 线程


- 重量级线程

- 由用户无法更改调度（通过 POSIX® API 略有缓解）

- 不需要系统调用包装

- 可利用多个 CPU

1:N 线程


- 轻量级线程

- 调度可以被用户轻松改变

- 系统调用必须包装

- 不能利用超过一个 CPU

### 2.3. 什么是 FreeBSD?

FreeBSD 项目是目前可供日常使用的最古老的开源操作系统之一。它是真正 UNIX® 的直系后裔，因此可以说它是真正的 UNIX®，尽管许可问题不允许这样说。该项目始于 1990 年代初，当时一群 BSD 用户对 386BSD 操作系统进行了补丁。基于这个补丁包，一个名为 FreeBSD 的新操作系统出现了，以其自由许可证而闻名。另一组人创建了具有不同目标的 NetBSD 操作系统。我们将专注于 FreeBSD。

FreeBSD 是一款现代的基于 UNIX® 的操作系统，具有 UNIX® 的所有特性。抢占式多任务处理、多用户设施、TCP/IP 网络、内存保护、对称多处理支持、具有合并 VM 和缓存的虚拟内存，它们都在那里。有趣且极其有用的一个功能是能够模拟其他类 UNIX® 操作系统。截至 2006 年 12 月和 7-CURRENT 开发，支持以下模拟功能：

- FreeBSD/i386 在 FreeBSD/amd64 上的仿真
- 在 FreeBSD / ia64 上模拟 FreeBSD / i386
- 在 FreeBSD 上模拟 Linux® 操作系统
- Windows 网络驱动程序接口的 NDIS 模拟
- NetBSD 操作系统的 NetBSD 仿真
- 支持 PECoff FreeBSD 可执行文件的 PECoff
- System V 修订版 4 UNIX® 的 SVR4 仿真

积极开发的仿真是 Linux® 层和各种 FreeBSD-on-FreeBSD 层。其他的不应该在这些日子里正常工作或可用。

#### 2.3.1. 技术细节

FreeBSD 是 UNIX® 传统风味，它将进程的运行划分为内核空间和用户空间运行的两半。内核有两种类型的进程入口：系统调用和陷阱。只有一种返回方式。在后续章节中，我们将描述从内核进入/离开的三个门。整个描述适用于 i386 架构，因为 Linuxulator 仅在那里存在，但在其他架构上，概念类似。此信息来源于[1]和源代码。

##### 2.3.1.1 系统项

FreeBSD 有一个称为执行类载入器的抽象概念，它是 execve(2) 系统调用的一个嵌入点。这使用了一个结构 sysentvec ，描述了可执行 ABI。它包含诸如 errno 转换表、信号转换表、各种用于满足系统调用需求的函数（堆栈修复、核心转储等）。FreeBSD 内核希望支持的每种 ABI 都必须定义这个结构，因为它稍后在系统调用处理代码中和其他一些地方使用。系统项由 trap 处理程序处理，在那里我们可以同时访问内核空间和用户空间。

##### 2.3.1.2 系统调用

在 FreeBSD 上，通过执行中断 0x80 ，并将寄存器 %eax 设置为所需的系统调用号，同时通过堆栈传递参数来发出系统调用。

当进程发出中断 0x80 时，会触发 int0x80 系统调用陷阱处理程序（定义在 sys/i386/i386/exception.s 中），该处理程序会准备参数（即将它们复制到堆栈上），以便调用 C 函数 syscall(2)（定义在 sys/i386/i386/trap.c 中），该函数处理传入的 trapframe。处理过程包括准备系统调用（取决于 sysvec 条目）、确定系统调用是 32 位还是 64 位（改变参数的大小），然后复制参数，包括系统调用。接下来，实际的系统调用函数会执行，并处理返回码（对于 ERESTART 和 EJUSTRETURN 错误有特殊处理）。最后会调度一个 userret() ，将进程切换回用户空间。实际系统调用处理程序的参数以 struct thread _td 、 struct syscall args _ 形式传递，其中第二个参数是指向复制参数结构的指针。

##### 2.3.1.3. trap

在 FreeBSD 中，对 trap 的处理类似于对 syscall 的处理。每当发生 trap 时，会调用一个汇编处理程序。根据 trap 的类型，在 alltraps、alltraps with regs pushed 或 calltrap 之间选择此处理程序。该处理程序为调用 C 函数 trap() （定义在 sys/i386/i386/trap.c 中）准备参数，然后该函数处理发生的 trap。处理完毕后，它可能会向进程发送信号和/或使用 userret() 退出到用户态。

##### 2.3.1.4. 退出

从内核到用户空间的退出使用汇编例程 doreti ，无论内核是通过 trap 还是通过 syscall 进入的。此例程从堆栈恢复程序状态并返回到用户空间。

##### 2.3.1.5. UNIX® 原语

FreeBSD 操作系统遵循传统的 UNIX® 方案，其中每个进程都有一个唯一的标识号，即所谓的 PID（进程 ID）。PID 号码是线性或随机分配的，范围从 0 到 PID_MAX 。PID 号码的分配是通过线性搜索 PID 空间来完成的。进程中的每个线程通过 getpid(2) 调用都会收到相同的 PID 号码作为结果。

FreeBSD 目前有两种实现线程的方法。第一种方法是 M:N 线程模型，接着是 1:1 线程模型。默认使用的库是 M:N 线程模型（ libpthread ），您可以在运行时切换到 1:1 线程模型（ libthr ）。计划很快将默认切换到 1:1 库。尽管这两个库使用相同的内核原语，但它们通过不同的 API 访问。M:N 库使用 kse** 系列的系统调用，而 1:1 库使用 thr** 系列的系统调用。由于这一点，在内核和用户空间之间没有线程 ID 的通用概念。当然，这两个线程库都实现了 pthread 线程 ID API。每个内核线程（如 struct thread 所述）都有一个 td tid 标识符，但这并不能直接从用户空间访问，而且仅仅为内核的需求服务。它也被用作 1:1 线程库的 pthread 线程 ID，但这个处理是内部库处理的，不能依赖它。

正如之前所述，在 FreeBSD 中有两种线程实现。M:N 库将工作划分为内核空间和用户空间。线程是在内核中调度的实体，但它可以代表多个用户空间线程。 M 个用户空间线程映射到 N 个内核线程，从而节省资源，同时保持利用多处理器并行性的能力。有关该实现的更多信息可以从 man 页面或[1]获取。 1:1 库直接将用户空间线程映射到内核线程，从而极大地简化了方案。这些设计都没有实现公平机制（尽管曾经实现过一种机制，但最近已经移除，因为它导致严重减速并使代码更难处理）。

### 2.4.Linux® 是什么

Linux® 是一个类 Unix 内核，最初是由 Linus Torvalds 开发的，现在正在全世界众多程序员的贡献。从其刚开始的阶段到今天，得到了 IBM 或谷歌等公司的大力支持，Linux® 被认为具有快速的发展步伐，完整的硬件支持和仁慈独裁者组织模式。

Linux® 的开发始于 1991 年芬兰赫尔辛基大学的一个业余项目。从那时起，Linux® 就具备了现代 UNIX® 类操作系统的所有功能：多进程、多用户支持、虚拟内存、网络，基本上应有尽有。此外，还有虚拟化等非常先进的功能。

截至 2006 年，Linux® 似乎是使用最广泛的开放源代码操作系统，并得到了甲骨文、RealNetworks、Adobe 等独立软件供应商的支持。为 Linux® 发布的大多数商业软件只能以二进制形式获得，因此不可能为其他操作系统重新编译。

大多数 Linux® 开发都是在 Git 版本控制系统中进行的。Git 是一个分布式系统，因此没有 Linux® 代码的中心源，但有些分支被认为是重要的官方分支。Linux® 采用的版本号方案由 A.B.C.D 四个数字组成。目前的开发版本为 2.6.C.D，其中 C 代表主要版本，用于添加或更改新功能，而 D 则是次要版本，仅用于修复错误。

可以从[3]获取更多信息。

#### 2.4.1. 技术细节

Linux® 遵循传统的 UNIX® 方案，将进程的运行划分为两个部分：内核和用户空间。内核可以通过陷阱或系统调用两种方式进入。返回仅以一种方式处理。进一步的描述适用于 Linux® 2.6 在 i386™ 架构上。此信息来自[2]。

##### 2.4.1.1. 系统调用

在 Linux® 中，系统调用是使用 syscallX 宏（在用户空间中）执行的，其中 X 代表给定系统调用的参数数量。这个宏将转换成一个代码，将 %eax 寄存器加载为系统调用的编号，并执行中断 0x80 。然后调用系统调用返回，将负的返回值转换为正的 errno 值，并在出现错误时将 res 设置为 -1 。当中断 0x80 被调用时，进程进入内核中的系统调用陷阱处理程序。这个程序将所有寄存器保存在堆栈上，并调用所选的系统调用入口。请注意，Linux® 调用约定期望系统调用的参数通过寄存器传递，如下所示：

1. 参数 → %ebx
2. 参数 → %ecx
3. 参数 → %edx
4. 参数 → %esi
5. 参数 → %edi
6. 参数 → %ebp

在某些情况下会有例外情况，Linux® 使用不同的调用约定（特别是 clone 系统调用）。

##### 2.4.1.2. 捕获

捕获处理程序在 arch/i386/kernel/traps.c 中引入，这些处理程序大多数存在于 arch/i386/kernel/entry.S 中，捕获的处理就在这里发生。

##### 2.4.1.3. 退出

从系统调用返回由系统调用 exit(3) 处理，它检查进程是否有未完成的工作，然后检查我们是否使用了用户提供的选择器。如果发生这种情况，将应用堆栈修复，最后从堆栈恢复寄存器，进程返回到用户空间。

##### 2.4.1.4. UNIX® 原语

在 2.6 版本中，Linux® 操作系统重新定义了一些传统的 UNIX® 原语，尤其是 PID、TID 和线程。PID 不是每个进程都唯一的定义，因此对于一些进程（线程），getppid(2) 返回相同的值。进程的唯一标识由 TID 提供。这是因为 NPTL（新 POSIX® 线程库）将线程定义为普通进程（所谓的 1:1 线程）。在 Linux® 2.6 中通过 clone 系统调用生成新进程（fork 变体被重新实现以使用它）。这个 clone 系统调用定义了一组标志，影响克隆进程的行为，关于线程实现的语义有些模糊，因为没有单一的标志告诉系统调用创建线程。

实现的克隆标志有：

- CLONE_VM - 进程共享它们的内存空间
- CLONE_FS - 共享 umask、当前工作目录和命名空间
- CLONE_FILES - 共享打开的文件
- CLONE_SIGHAND - 共享信号处理程序和被阻止的信号
- CLONE_PARENT - 共享父级
- CLONE_THREAD - 线程优化 (以下进一步解释)
- CLONE_NEWNS - 新命名空间
- CLONE_SYSVSEM - 共享 SysV 撤销结构
- CLONE_SETTLS - 在提供的地址设置 TLS
- CLONE_PARENT_SETTID - 在父级中设置 TID
- CLONE_CHILD_CLEARTID - 在子级中清除 TID
- CLONE_CHILD_SETTID - 在子级中设置 TID

CLONE*PARENT 将真实父进程设置为调用者的父进程。对于线程来说很有用，因为如果线程 A 创建线程 B，我们希望线程 B 被赋予整个线程组的父进程。 CLONE_THREAD 正好做了与 CLONE_PARENT 、 CLONE_VM 和 CLONE_SIGHAND 相同的事情，重写 PID 为调用者的 PID，设置退出信号为无，然后进入线程组。 CLONE_SETTLS 设置 GDT 条目以处理 TLS。一组 CLONE* ___ TID 标志设置/清除用户提供的地址为 TID 或 0。

正如您所见， CLONE_THREAD 完成了大部分工作，但似乎不太适合这个方案。最初的意图不清楚（即使对于代码的作者来说，根据代码中的注释），但我认为最初有一个线程标志，然后被分成许多其他标志，但这种分离从未完全完成。不清楚这种分区有何用处，因为 glibc 不使用它，所以只有手工编写的 clone 使用允许程序员访问这些功能。

针对非线程程序，PID 和 TID 相同。对于线程程序，第一个线程的 PID 和 TID 相同，并且每个创建的线程共享相同的 PID 并被分配一个唯一的 TID（因为被传递的 CLONE_THREAD 是相同的），父进程也被所有构成这个线程程序的进程所共享。

在 NPTL 中实现 pthread_create(3) 的代码定义了克隆标志如下：

```c
int clone_flags = (CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGNAL

 | CLONE_SETTLS | CLONE_PARENT_SETTID

| CLONE_CHILD_CLEARTID | CLONE_SYSVSEM
#if __ASSUME_NO_CLONE_DETACHED == 0

| CLONE_DETACHED
#endif

| 0);
```

CLONE_SIGNAL 被定义为

```c
#define CLONE_SIGNAL (CLONE_SIGHAND | CLONE_THREAD)
```

最后一个 0 表示当任何线程退出时不发送任何信号。

### 仿真是什么

根据词典定义，仿真是程序或设备模仿另一个程序或设备的能力。这是通过对给定刺激提供与被仿真对象相同的反应来实现的。在实践中，软件世界主要看到三种类型的仿真 - 用于模拟机器的程序（QEMU，各种游戏主机仿真器等），对硬件设施的软件仿真（OpenGL 仿真器，浮点单位仿真等）以及操作系统仿真（无论是在操作系统的内核中还是作为用户空间程序）。

仿真通常用于无法或完全不可能使用原始组件的地方。例如，有人可能希望使用为不同操作系统开发的程序。这时就可以使用仿真。有时候别无选择，只能使用仿真 - 例如，当您尝试使用的硬件设备不存在（尚未/不再存在）时，只能通过仿真来实现。这在将操作系统移植到新（不存在的）平台时经常发生。有时候只是为了节约成本而仿真。

从实现角度来看，实现仿真有两种主要方法。您可以仿真整个对象 - 接受原始对象可能的输入，维护内部状态，并根据状态和/或输入生成正确的输出。这种仿真不需要任何特殊条件，基本上可以在任何设备/程序上实现。缺点是实现这种仿真非常困难、耗时且容易出错。在某些情况下，我们可以使用更简单的方法。想象一下，您要在从右到左打印的打印机上仿真从左到右打印的打印机。显然，不需要复杂的仿真层，只需简单地反转打印文本即可。有时仿真环境与仿真对象非常相似，因此只需要一些翻译的薄层即可提供完全工作的仿真！正如您所看到的，这要求要实现的工作量要少得多，因此比前一种方法少耗时且少出错。但是必要的条件是这两个环境必须足够相似。第三种方法结合了前两种方法。大多数时候，对象并不提供相同的功能，因此在较强大的对象上对较弱的对象进行仿真时，我们必须使用上述全面仿真来仿真缺失的功能。

本硕士论文涉及 UNIX® 在 UNIX® 上的仿真，这正是只需一层薄薄的翻译就能提供完整仿真的情况。UNIX® API 由一组系统调用组成，这些调用通常是自包含的，不会影响一些全局内核状态。

有一些系统调用会影响内部状态，但可以通过提供一些维护额外状态的结构来解决这个问题。

没有完美的仿真，仿真往往会缺少一些部分，但这通常不会造成严重的缺陷。想象一下一个游戏主机模拟器，它模拟了一切，但没有音乐输出。毫无疑问，游戏是可以玩的，人们可以使用模拟器。这也许不像原始游戏主机那样舒适，但在价格和舒适性之间是一个可接受的妥协。

与 UNIX® API 一样。 大多数程序可以使用非常有限的一组系统调用而生存。 这些系统调用往往是最古老的（read（2）/write（2），fork（2）系列，signal（3）处理，exit（3），socket（2）API），因此很容易进行模拟，因为它们的语义在所有 UNIX® 之间共享，这些 UNIX® 存在于今天。

## 3. 模拟

### 3.1. FreeBSD 中的模拟工作方式

正如早前所述，FreeBSD 支持从其他几个 UNIX®es 运行二进制文件。这是因为 FreeBSD 具有一个称为执行类加载器的抽象。它嵌入到 execve(2)系统调用中，因此当 execve(2)即将执行一个二进制文件时，它会检查其类型。

在 FreeBSD 中基本上有两种类型的二进制文件。类似 Shell 的文本脚本，它们通过 #! 作为前两个字符进行标识，以及通常是 ELF 格式的普通二进制文件，它们是编译后的可执行对象的表示。绝大多数（可以说全部）的 FreeBSD 二进制文件属于 ELF 类型。ELF 文件包含一个标头，该标头指定了此 ELF 文件的操作系统 ABI。通过阅读此信息，操作系统可以准确确定给定文件的二进制类型。

每个操作系统 ABI 必须在 FreeBSD 内核中注册。这也适用于 FreeBSD 本机操作系统 ABI。因此，当 execve(2)执行一个二进制文件时，它会遍历已注册 API 列表，一旦找到正确的 API，它就开始使用操作系统 ABI 描述中包含的信息（其系统调用表、 errno 转换表等）。因此，每次进程调用系统调用时，它都会使用自己的一组系统调用，而不是一组全局的系统调用。这有效地提供了一种非常优雅且简单的方式来支持各种二进制格式的执行。

不同操作系统（以及一些其他子系统）仿真的本质促使开发者引入了处理器事件机制。内核中的各个地方都会调用事件处理器列表。每个子系统都可以注册一个事件处理器，并相应地调用它们。例如，当一个进程退出时，会调用一个处理器，可能清理子系统需要清理的内容。

这些简单的设施基本上提供了实现 Linux® 仿真层所需的一切。事实上，这些基本上是实现 Linux® 仿真层所需的唯一必要条件。

### 3.2. FreeBSD 内核中的常见原语

仿真层需要操作系统的一些支持。我将描述 FreeBSD 操作系统中支持的一些原语。

#### 3.2.1. 锁定原语

贡献者: Attilio Rao <<a href="mailto:attilio@FreeBSD.org">attilio@FreeBSD.org</a>>

FreeBSD 同步原语集基于一个理念，即以一种方式提供大量不同的原语，以便在每个特定的、适当的情况下可以使用更好的原语。

从一个高层次的角度来看，你可以在 FreeBSD 内核中考虑三种类型的同步原语：

- 原子操作和内存屏障
- 锁定
- 调度屏障

下面是 3 个家族的描述。对于每个锁定，您应该真正检查链接的手册页（在可能的情况下）以获取更详细的解释。

##### 3.2.1.1. 原子操作和内存屏障

原子操作是通过一组函数在内存操作数上执行简单算术操作的方式实现的，与外部事件（中断、抢占等）保持原子性。原子操作只能保证在小数据类型上（大致在 .long. 架构 C 数据类型的数量级上）是原子的，因此在最终层代码中应该很少直接使用，除非是非常简单的操作（例如在位图中设置标志）。事实上，仅仅依赖原子操作（通常称为无锁）很容易写出错误的语义。FreeBSD 内核提供了一种方法来执行原子操作，并与内存屏障结合使用。内存屏障将保证原子操作按照指定的顺序与其他内存访问发生。例如，如果我们需要一个原子操作在所有其他挂起的写操作（按照指令重排序缓冲区的活动）完成之后发生，我们需要显式地在这个原子操作中使用内存屏障。因此，理解为什么内存屏障在构建更高层级的锁（如互斥体等）中起关键作用是很简单的。关于原子操作的详细说明，请参阅 atomic(9)。然而，需要注意的是，原子操作（以及内存屏障）理想情况下仅应用于构建前端锁（如互斥体）。

##### 3.2.1.2. 引用计数

引用计数是处理引用计数器的接口。它们通过原子操作实现，并且仅用于引用计数器是唯一需要保护的情况，因此甚至像自旋互斥体这样的东西已经不推荐使用了。对于已经使用互斥体的结构使用引用计数接口通常是错误的，因为我们可能应该在一些已经受保护的路径中关闭引用计数。当前并没有一个讨论引用计数的手册页面，只需查看 sys/refcount.h 以获得现有 API 的概述。

##### 3.2.1.3. 锁

FreeBSD 内核具有多种类型的锁。每个锁由一些特殊属性定义，但可能最重要的是与竞争持有者相关联的事件（或者换句话说，无法获取锁的线程的行为）。FreeBSD 的锁定方案提供了三种不同的竞争者行为：

1. 旋转
2. 阻塞
3. 休眠

|     | 数字并不随意 |
| --- | ------------ |

##### 3.2.1.4. 旋转锁

自旋锁允许等待者旋转，直到无法获取锁。处理的一个重要问题是当一个线程竞争一个自旋锁时，如果它没有被取消调度。由于 FreeBSD 内核是抢占式的，这使得自旋锁面临死锁的风险，可以通过在获取锁时禁用中断来解决这个问题。由于这个原因以及其他原因（如缺乏优先级传播支持，在 CPU 之间负载平衡方案方面贫乏等），自旋锁旨在保护非常小的代码路径，或者在明确请求时根本不使用（稍后解释）。

##### 3.2.1.5. 阻塞

阻塞锁允许等待者在锁所有者不释放它并唤醒一个或多个竞争者之前被暂停和阻塞。为了避免饥饿问题，阻塞锁从等待者向所有者进行优先级传播。阻塞锁必须通过转门接口实现，并且打算成为内核中使用最频繁的锁类型，如果没有特殊条件。

##### 3.2.1.6. 休眠

睡眠锁允许等待者被取消调度并入睡，直到锁持有者放弃锁并唤醒一个或多个等待者。由于睡眠锁旨在保护大量代码路径并满足异步事件，它们不执行任何形式的优先级传播。它们必须通过 sleepqueue(9)接口实现。

获取锁的顺序非常重要，不仅因为由于锁顺序颠倒可能造成死锁的可能性，而且因为锁获取应遵循与锁性质相关的特定规则。如果查看上面的表格，实际规则是，如果一个线程持有级别为 n 的锁（其中级别是紧靠锁类型旁列出的数字），那么不允许该线程获取更高级别的锁，因为这会违反路径的指定语义。例如，如果一个线程持有一个块锁（级别 2），那么可以获取一个自旋锁（级别 1），但不能获取一个睡眠锁（级别 3），因为块锁旨在保护比睡眠锁更小的路径（这些规则与原子操作或调度屏障无关）。

这是一个带有各自行为的锁列表：

- 自旋互斥锁 - 自旋 - 互斥锁(9)
- 睡眠互斥锁 - 阻塞 - 互斥锁(9)
- 池互斥锁 - 阻塞 - mtx(池)
- 睡眠家庭 - 睡眠中 - 睡眠（9）暂停 tsleep msleep msleep spin msleep rw msleep sx
- condvar - 睡眠中 - condvar（9）
- rwlock - 阻塞 - rwlock（9）
- sxlock - 睡眠 - sx(9)
- lockmgr - 睡眠 - lockmgr(9)
- semaphores - 睡眠 - sema(9)

只有互斥锁、sxlocks、读写锁和 lockmgrs 这些锁才用于处理递归，但目前只有互斥锁和 lockmgrs 支持递归。

##### 3.2.1.7. 调度屏障

调度屏障旨在用于驱动线程调度。主要包括三个不同的存根：

- 临界区（和抢占）
- sched_bind
- sched_pin

通常情况下，这些仅应在特定上下文中使用，即使它们通常可以替代锁，也应避免使用，因为它们不允许使用锁定调试工具来诊断简单的潜在问题（如 witness(4) 所示）。

##### 3.2.1.8. 临界区

FreeBSD 内核已经被设计为可抢占，基本上是为了处理中断线程。实际上，为了避免高中断延迟，时间共享优先级线程可以被中断线程抢占（这样，它们不需要等待正常路径预览进行调度）。然而，抢占引入了新的竞争点，这也需要处理。通常，为了处理抢占，最简单的方法是完全禁用它。临界区定义了一段代码（由函数 critical_enter(9) 和 critical_exit(9) 之间的区域限定），在该区域内可以保证不会发生抢占（直到保护的代码完全执行完毕）。这通常可以有效地替代锁，但应谨慎使用，以免失去抢占带来的全部优势。

##### 3.2.1.9. sched_pin/sched_unpin

另一种处理抢占的方法是 sched_pin() 接口。如果一段代码封闭在 sched_pin() 和 sched_unpin() 函数对中，确保相应的线程，即使可以被抢占，也始终在同一个 CPU 上执行。在需要访问每 CPU 数据且假定其他线程不会更改这些数据的特定情况下，固定是非常有效的。后一个条件将把关键段作为我们代码的一个过于严格的条件。

##### 3.2.1.10. sched_bind/sched_unbind

sched_bind 是一个用于将线程绑定到特定 CPU 的 API，直到 sched_unbind 函数调用将其解除绑定。在您无法信任 CPU 当前状态（例如，在引导的早期阶段）时，此功能具有关键作用，因为您希望避免线程迁移到不活跃的 CPU 上。由于 sched_bind 和 sched_unbind 操作内部调度器结构，因此在使用时需要包含在 sched_lock 获取/释放中。

#### 3.2.2. 进程结构

各种仿真层有时需要一些额外的每进程数据。它可以管理包含每个进程这些数据的单独结构（列表、树等），但这往往既慢又耗费内存。为了解决这个问题，FreeBSD proc 结构包含 p_emuldata ，这是一个指向某些仿真层特定数据的 void 指针。这个 proc 条目受 proc 互斥锁保护。

FreeBSD 的 proc 结构包含一个 p_sysent 条目，用于标识此进程正在运行的 ABI。实际上，它是指向上述 sysentvec 的指针。因此，通过比较此指针与存储给定 ABI 的 sysentvec 结构的地址，我们可以有效地确定进程是否属于我们的仿真层。代码通常如下所示：

```c
if (__predict_true(p->p_sysent != &elf_Linux(R)_sysvec))
	  return;
```

正如您所看到的，我们有效地使用 __predict_true 修饰符将最常见的情况（FreeBSD 进程）折叠为简单的返回操作，从而保持高性能。这段代码应该转换为宏，因为当前它并不是很灵活，即我们不支持 Linux®64 仿真，也不支持 i386 上的 A.OUT Linux® 进程。

#### 3.2.3. VFS

FreeBSD VFS 子系统非常复杂，但 Linux® 模拟层仅使用通过明确定义的 API 的小子集。它可以在 vnode 或文件处理程序上操作。Vnode 表示虚拟 vnode，即 VFS 中节点的表示。另一个表示是文件处理程序，它表示进程视角中打开的文件。文件处理程序可以表示套接字或普通文件。文件处理程序包含指向其 vnode 的指针。多个文件处理程序可以指向同一个 vnode。

##### 3.2.3.1. namei

namei(9) 例程是路径名查找和转换的中心入口点。它逐点从起始点到终点遍历路径，使用内部于 VFS 的查找函数。namei(9) 系统调用可以处理符号链接、绝对路径和相对路径。当使用 namei(9) 查找路径时，它会输入名称缓存。此行为可以被抑制。此例程在整个内核中都被广泛使用，其性能非常关键。

##### 3.2.3.2. vn_fullpath

vn_fullpath(9) 函数尽力遍历 VFS 名称缓存，并返回给定（已锁定）vnode 的路径。这个过程不可靠，但对于大多数常见情况而言效果还不错。其不可靠性在于它依赖于 VFS 缓存（不遍历介质上的结构），不适用于硬链接等情况。该例程在 Linuxulator 中的多个位置使用。

##### 3.2.3.3. Vnode 操作

- fgetvp - 给定一个线程和文件描述符号，返回关联的 vnode
- vn_lock(9) - 锁定一个 vnode
- vn_unlock - 解锁一个 vnode
- VOP_READDIR(9) - 读取由 vnode 引用的目录
- VOP_GETATTR(9) - 获取由 vnode 引用的文件或目录的属性
- VOP_LOOKUP(9) - 查找到给定目录的路径
- VOP_OPEN(9) - 打开由 vnode 引用的文件
- VOP_CLOSE(9) - 关闭由 vnode 引用的文件
- vput(9) - 减少 vnode 的使用计数并解锁
- vrele(9) - 减少 vnode 的使用计数
- vref(9) - 增加 vnode 的使用计数

##### 3.2.3.4. 文件处理器操作

- fget - 给定一个线程和文件描述符号，返回相关联的文件处理程序并引用它
- fdrop - 释放文件处理程序的引用
- fhold - 引用文件处理程序

## 4. Linux® 模拟层 - MD 部分

本节涉及 FreeBSD 操作系统中 Linux® 模拟层的实现。首先描述了机器相关部分，讨论用户空间和内核之间的交互是如何以及在哪里实现的。它涉及到系统调用、信号、ptrace、陷阱、堆栈修复。这一部分讨论了 i386，但它总体上编写得很通用，因此其他体系结构不应该有太大区别。下一部分是 Linuxulator 的机器无关部分。本节仅涵盖 i386 和 ELF 处理。A.OUT 已经过时且未经测试。

### 4.1. 系统调用处理

系统调用处理主要是在 linux_sysvec.c 中编写的，它涵盖了 sysentvec 结构中指出的大多数例程。当运行在 FreeBSD 上的 Linux® 进程发出系统调用时，通用系统调用例程会调用 Linux® ABI 的 linux prepsyscall 例程。

#### 4.1.1. Linux® prepsyscall

Linux® 通过寄存器传递系统调用参数（这就是为什么在 i386 上仅限于 6 个参数），而 FreeBSD 使用堆栈。Linux® prepsyscall 例程必须将参数从寄存器复制到堆栈中。寄存器的顺序是： %ebx ， %ecx ， %edx ， %esi ， %edi ， %ebp 。问题是对于大多数系统调用来说只是这样。一些系统调用（尤其是 clone ）使用不同的顺序，但幸运的是，通过在 linux_clone 原型中插入一个虚拟参数很容易解决。

#### 4.1.2. 系统调用写入

Linuxulator 中实现的每个系统调用都必须在 syscalls.master 中具有带有各种标志的原型。文件的形式为：

```c
...
	AUE_FORK STD		{ int linux_fork(void); }
...
	AUE_CLOSE NOPROTO	{ int close(int fd); }
...
```

第一列代表系统调用编号。第二列用于审计支持。第三列代表系统调用类型。 它可以是 STD ， OBSOL ， NOPROTO 和 UNIMPL 。 STD 是带有完整原型和实现的标准系统调用。 OBSOL 已过时，仅定义原型。 NOPROTO 意味着该系统调用在其他地方实现，因此不需要添加 ABI 前缀，等等。 UNIMPL 意味着该系统调用将被替换为 nosys 系统调用（一个仅输出有关未实现系统调用的消息并返回 ENOSYS 的系统调用）。

从 syscalls.master 脚本生成三个文件：linux_syscall.h、linux_proto.h 和 linux_sysent.c。linux_syscall.h 包含系统调用名称及其数值定义，例如：

```c
...
#define LINUX_SYS_linux_fork 2
...
#define LINUX_SYS_close 6
...
```

linux_proto.h 包含每个系统调用参数的结构定义，例如：

```c
struct linux_fork_args {
  register_t dummy;
};
```

最后，linux_sysent.c 包含描述系统调用入口表的结构，用于实际调度系统调用，例如：

```c
{ 0, (sy_call_t *)linux_fork, AUE_FORK, NULL, 0, 0 }, /* 2 = linux_fork */
{ AS(close_args), (sy_call_t *)close, AUE_CLOSE, NULL, 0, 0 }, /* 6 = close */
```

正如您所见， linux_fork 在 Linuxulator 自身中实现，因此其定义是 STD 类型，并且没有参数，这由虚拟参数结构体表现出来。另一方面， close 只是真实 FreeBSD close(2) 的别名，因此它没有关联的 Linux 参数结构，并且在系统入口表中没有以 linux 作为前缀，因为它调用内核中的真实 close(2)。

#### 4.1.3. 虚拟系统调用

Linux® 模拟层并不完整，因为某些系统调用没有正确实现，而有些根本没有实现。模拟层使用 DUMMY 宏标记未实现的系统调用。这些虚拟定义以 DUMMY(syscall); 的形式存储在 linux_dummy.c 中，然后被翻译成各种系统调用辅助文件，并且其实现包括打印一条消息，说明该系统调用未被实现。不使用 UNIMPL 原型是因为我们希望能够识别被调用的系统调用名称，以知道哪些系统调用更重要需要实现。

### 4.2. 信号处理

信号处理通常在 FreeBSD 内核中完成，用于所有二进制兼容性，调用与兼容性相关的层。Linux® 兼容性层为此目的定义 linux_sendsig 例程。

#### 4.2.1. Linux® sendsig

这个例程首先检查信号是否已经被安装，并在这种情况下调用 SA_SIGINFO 例程代替。此外，它会分配（或重用已经存在的）信号处理程序上下文，然后为信号处理程序构建参数列表。它会根据信号转换表翻译信号编号，分配一个处理程序，翻译 sigset。然后它会保存 sigreturn 例程的上下文（各种寄存器，转换后的陷阱编号和信号掩码）。最后，它会复制信号上下文到用户空间并准备实际信号处理程序运行所需的上下文。

#### 4.2.2. linux_rt_sendsig

这个例程类似于 linux_sendsig ，只是信号上下文的准备方式不同。它添加 siginfo ， ucontext 和一些 POSIX® 部分。也许值得考虑这两个函数是否可以合并，以减少代码重复，并可能实现更快的执行。

#### 4.2.3. linux_sigreturn

此系统调用用于从信号处理程序返回。它执行一些安全检查并恢复原始进程上下文。它还在进程信号掩码中取消信号屏蔽。

### 4.3. Ptrace

许多 UNIX® 派生版本实现 ptrace(2)系统调用，以允许各种跟踪和调试功能。此功能使跟踪进程能够获取有关被跟踪进程的各种信息，如寄存器转储、进程地址空间中的任何内存等，还能够跟踪进程，比如在指令之间进行步进操作或在系统入口之间进行跟踪（系统调用和陷阱）。ptrace(2)还允许您在被跟踪的进程中设置各种信息（寄存器等）。ptrace(2)是一个 UNIX® 范围的标准，在世界上大多数 UNIX® 中实现。

FreeBSD 中的 Linux® 仿真实现了 linux_ptrace.c 中的 ptrace(2)工具。 在 Linux® 和 FreeBSD 之间转换寄存器的例程以及实际的 ptrace(2)系统调用仿真系统调用。该系统调用是一个长开关块，用于为每个 ptrace(2)命令在 FreeBSD 中实现其 Linux® 对应项。 ptrace(2)命令在 Linux® 和 FreeBSD 之间大多是相同的，所以通常只需要进行一些小的修改。 例如，在 Linux® 中， PT_GETREGS 操作直接数据，而 FreeBSD 使用数据的指针，因此在执行(native) ptrace(2)系统调用后，必须执行复制操作以保留 Linux® 语义。

Linuxulator 中的 ptrace(2)实现存在一些已知弱点。在 Linuxulator 环境中使用 strace （这是一个 ptrace(2)使用者）时发生过紧急情况。还没有实现 PT_SYSCALL 。

### 4.4. 陷阱

每当在仿真层运行的 Linux® 进程陷入陷阱时，只有陷阱转换例外情况下才会透明地处理它。 Linux® 和 FreeBSD 对于陷阱的定义存在差异，因此需要在此处理。 代码实际上非常简短：

```c
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

### 4.5. 堆栈修复

RTLD 运行时链接编辑器期望在 execve 期间堆栈上有所谓的 AUX 标签，因此必须进行修正以确保这一点。当然，每个 RTLD 系统都是不同的，因此仿真层必须提供自己的堆栈修正例程来完成此操作。Linuxulator 也是如此。 elf_linux_fixup 只是将 AUX 标签复制到堆栈上，并调整用户空间进程的堆栈以指向这些标签之后的位置。因此 RTLD 以一种智能的方式工作。

### 4.6. A.OUT 支持

i386 上的 Linux® 仿真层也支持 Linux® A.OUT 二进制文件。前面部分描述的几乎所有内容都必须为 A.OUT 支持实现（除了陷阱翻译和信号发送）。A.OUT 二进制文件的支持不再维护，尤其是 2.6 仿真不支持它，但这并不会造成任何问题，因为 ports 中的 linux-base 可能根本不支持 A.OUT 二进制文件。这个支持可能会在未来被移除。加载 Linux® A.OUT 二进制文件所需的大部分内容都在 imgact_linux.c 文件中。

## 5. Linux® 仿真层 -MI 部分

本节讨论 Linuxulator 的机器无关部分。它涵盖了 Linux® 2.6 仿真所需的仿真基础设施，线程本地存储（TLS）实现（在 i386 上）和 futexes。然后我们简要讨论一些系统调用。

### 5.1. NPTL 描述

Linux® 2.6 开发中的主要进展之一是线程。在 2.6 之前，Linux® 的线程支持是在 linuxthreads 库中实现的。该库是 POSIX® 线程的部分实现。线程是使用单独的进程实现的，每个线程使用 clone 系统调用来共享地址空间（及其他内容）。这种方法的主要弱点是每个线程都有不同的 PID，信号处理在 pthreads 视角下是有问题的，等等。此外，性能也不是很好（使用 SIGUSR 信号进行线程同步，内核资源消耗等）。为了克服这些问题，开发了一个新的线程系统，命名为 NPTL。

NPTL 库专注于两件事，但第三件事也出现了，所以通常被视为 NPTL 的一部分。这两件事是将线程嵌入到进程结构中和使用 futex。额外的第三件事是 TLS，虽然 NPTL 并不直接需要它，但整个 NPTL 用户库依赖于它。这些改进显著提高了性能和标准符合性。NPTL 现在是 Linux® 系统中的标准线程库。

FreeBSD Linuxulator 实现在三个主要领域接近 NPTL。TLS、futexes 和 PID 操纵是模拟 Linux® 线程的关键点。后续章节将详细描述每个领域。

### 5.2. Linux® 2.6 模拟基础设施

这些部分涉及 Linux® 线程的管理方式以及我们在 FreeBSD 中如何模拟它们。

#### 5.2.1. 运行时确定 2.6 模拟

FreeBSD 中的 Linux® 模拟层支持模拟版本的运行时设置。 这是通过 sysctl(8) 完成的，即 compat.linux.osrelease 。 设置此 sysctl(8) 会影响模拟层的运行时行为。 当设置为 2.6.x 时，它设置 linux_use_linux26 的值，而设置为其他值会保持未设置状态。 此变量（以及非常相似的每个监狱变量）确定代码中是否使用 2.6 基础设施（主要是 PID 改名）。 版本设置是针对整个系统的，这会影响所有 Linux® 进程。 运行任何 Linux® 二进制文件时不应更改 sysctl(8)，因为这可能会损害事物。

#### 5.2.2. Linux® 进程和线程标识符

Linux® 线程的语义有点令人困惑，并且使用与 FreeBSD 完全不同的命名法。 Linux® 中的进程由一个 struct task 嵌入的两个标识符字段组成 - PID 和 TGID。 PID 不是进程 ID，而是线程 ID。 TGID 标识一个线程组，换句话说是一个进程。 对于单线程进程，PID 等于 TGID。

在 NPTL 中，线程只是一个普通进程，其 TGID 不等于 PID，并且具有不等于自身的组领导者（当然还有共享的虚拟内存等）。除此之外，一切都与普通进程相同。没有像 FreeBSD 中那样将共享状态分离到某些外部结构中。这会导致信息的重复和可能的数据不一致性。Linux® 内核似乎在某些地方使用任务 → 组信息，在其他地方使用任务信息，这实在不太一致，并且看起来容易出错。

每个 NPTL 线程是通过调用 clone 系统调用并使用特定的一组标志（在下一小节中详细说明）创建的。NPTL 实现了严格的 1:1 线程模型。

在 FreeBSD 中，我们通过普通的 FreeBSD 进程来模拟 NPTL 线程，这些进程共享虚拟内存等，并且 PID 的操作只是在附加到进程的模拟特定结构中模仿而已。附加到进程的结构看起来像：

```c
struct linux_emuldata {
  pid_t pid;

  int *child_set_tid; /* in clone(): Child.s TID to set on clone */
  int *child_clear_tid;/* in clone(): Child.s TID to clear on exit */

  struct linux_emuldata_shared *shared;

  int pdeath_signal; /* parent death signal */

  LIST_ENTRY(linux_emuldata) threads; /* list of linux threads */
};
```

该 PID 用于标识附加到此结构的 FreeBSD 进程。 child_se_tid 和 child_clear_tid 在进程退出并重新创建时用于 TID 地址复制出。 shared 指针指向线程共享的结构。 pdeath_signal 变量标识父进程的死亡信号， threads 指针用于将此结构链接到线程列表。 linux_emuldata_shared 结构如下：

```c
struct linux_emuldata_shared {

  int refs;

  pid_t group_pid;

  LIST_HEAD(, linux_emuldata) threads; /* head of list of linux threads */
};
```

refs 是引用计数器，用于确定何时可以释放该结构以避免内存泄漏。 group_pid 用于标识整个进程（= 线程组）的 PID（= TGID）。 threads 指针是进程线程列表的头部。

可以使用 em_find 从进程中获取 linux_emuldata 结构。该函数的原型是：

```
struct linux_emuldata *em_find(struct proc *, int locked);
```

这里， proc 是我们希望从中获取 emuldata 结构的进程，locked 参数确定我们是否要锁定。接受的值为 EMUL_DOLOCK 和 EMUL_DOUNLOCK 。关于锁定的更多信息稍后会提到。

#### 5.2.3. PID 篡改

由于 FreeBSD 和 Linux® 在进程 ID 和线程 ID 的概念上存在差异，我们必须以某种方式来翻译这种观点。我们通过 PID 篡改来实现。这意味着我们在内核和用户空间之间伪造 PID（=TGID）和 TID（=PID）。经验法则是在内核（在 Linux 模拟器中）中，PID = PID，TGID = shared → group pid，在用户空间中我们呈现 PID = shared → group_pid 和 TID = proc → p_pid 。 linux_emuldata structure 的 PID 成员是 FreeBSD PID。

上述主要影响 getpid、getppid、gettid 系统调用。我们分别使用 PID/TGID。在 child_clear_tid 和 child_set_tid 的 TID 复制中，我们复制出 FreeBSD PID。

#### 5.2.4. Clone 系统调用

clone 系统调用是在 Linux® 中创建线程的方式。系统调用原型看起来像这样：

```c
int linux_clone(l_int flags, void *stack, void *parent_tidptr, int dummy,
void * child_tidptr);
```

参数 flags 告诉系统调用如何克隆进程。如前所述，Linux® 可以独立地创建共享各种资源的进程，例如两个进程可以共享文件描述符但不共享虚拟内存等。 flags 参数的最后一个字节是新创建进程的退出信号。如果 stack 参数非 NULL ，则指定线程堆栈的位置；如果是 NULL ，则应复制调用进程的堆栈（即执行正常 fork(2)例程的操作）。 parent_tidptr 参数用作在进程 PID（即线程 ID）足够实例化但尚未可运行时复制出的地址。 dummy 参数是由于 i386 架构上此系统调用的非常奇怪的调用约定而存在。它直接使用寄存器，不允许编译器完成操作，因此需要一个虚拟系统调用。 child_tidptr 参数用作在进程 fork 完成并且进程退出时复制出 PID 的地址。

系统调用本身通过设置相应的标志来进行，这取决于传入的标志。例如， CLONE_VM 对应于 RFMEM（共享虚拟内存），等等。唯一的小问题在于 CLONE_FS 和 CLONE_FILES ，因为 FreeBSD 不允许单独设置它们，所以我们通过不设置 RFFDG（复制文件描述符表和其他文件系统信息）来模拟它们的定义。这并不会导致任何问题，因为这些标志总是一起设置的。设置完标志后，使用内部 fork1 函数分叉进程，进程被设计为不会放到运行队列上，即不可运行。分叉完成后，我们可能会重新将新创建的进程重新归属，以模拟 CLONE_PARENT 的语义。接下来是创建仿真数据的部分。Linux® 中的线程不向其父进程发送信号，因此我们将退出信号设置为 0 以禁用此功能。之后执行 child_set_tid 和 child_clear_tid 的设置，以便在后续代码中启用这些功能。在这一点上，我们将进程的 PID 复制到 parent_tidptr 指定的地址。通过简单地重写线程帧 %esp 寄存器（在 amd64 上是 %rsp ），来完成进程堆栈的设置。接下来是为新创建的进程设置 TLS。在这之后可能会模拟 vfork(2) 的语义，最后将新创建的进程放到运行队列上，并通过 clone 返回值将其 PID 复制给父进程。

clone 系统调用能够且事实上用于模拟经典的 fork(2) 和 vfork(2) 系统调用。较新的 glibc 在 2.6 内核的情况下使用 clone 来实现 fork(2) 和 vfork(2) 系统调用。

#### 5.2.5. 锁定

实现加锁是针对每个子系统的，因为我们预计这些地方不会有太多的争用。有两个锁： emul_lock 用于保护 linux_emuldata 的操作， emul_shared_lock 用于操作 linux_emuldata_shared 。 emul_lock 是一个非可睡眠的阻塞互斥锁，而 emul_shared_lock 是一个可睡眠的阻塞 sx_lock 。由于每个子系统的锁定，我们可以合并一些锁，这就是为什么 em 查找提供了无锁访问。

### 5.3. 传输层安全协议

本节介绍传输层安全协议，也被称为线程局部存储。

#### 5.3.1. 线程概述

计算机科学中的线程是进程中可以独立调度的实体。进程中的线程共享进程范围的数据（文件描述符等），但也有自己的堆栈用于自己的数据。有时需要进程范围的数据特定于给定线程。想象一下线程在执行中的名称之类的东西。传统的 UNIX® 线程 API，pthread 提供了一种方法通过 pthread_key_create(3)，pthread_setspecific(3) 和 pthread_getspecific(3) 来实现，在这种方法中，线程可以创建一个用于线程本地数据的密钥，并使用 pthread_getspecific(3) 或 pthread_getspecific(3) 来操作这些数据。您很容易看出，这并不是完成这项任务的最舒适的方法。因此，各种 C/C++ 编译器的生产商引入了一个更好的方法。他们定义了一个新的修饰关键字 thread，指定一个变量是线程特定的。还开发了一种访问这些变量的新方法（至少在 i386 上是这样）。pthread 方法往往在用户空间中实现为一个简单的查找表。这种解决方案的性能并不是很好。因此，新方法在 i386 上使用段寄存器来定位存储 TLS 区域的段，因此访问线程变量实际上只是将段寄存器附加到地址上，从而通过它进行寻址。段寄存器通常是 %gs 和 %fs ，充当段选择器。每个线程都有自己的区域，线程本地数据存储在其中，必须在每次上下文切换时加载段。该方法非常快速，并且几乎完全在整个 i386 UNIX® 世界中使用。FreeBSD 和 Linux® 都实现了这种方法，并获得了非常好的结果。唯一的缺点是需要在每次上下文切换时重新加载段，这可能会减慢上下文切换的速度。FreeBSD 通过仅使用 1 个段描述符来尽量避免这种开销，而 Linux® 使用 3 个。 几乎没有什么东西使用超过 1 个描述符（只有 Wine 似乎使用了 2 个），因此 Linux® 为上下文切换支付了这个不必要的代价。

#### 5.3.2. i386 上的段

i386 架构实现了所谓的段。段是对内存区域的描述。内存区域的基地址（底部）、结束地址（顶部）、类型、保护等等。通过段选择器寄存器（ %cs , %ds , %ss , %es , %fs , %gs ）可以访问描述的内存。例如，假设我们有一个基地址为 0x1234 且长度为的段和这段代码：

```c
mov %edx,%gs:0x10
```

将 %edx 寄存器的内容加载到内存位置 0x1244。一些段寄存器有特殊用途，例如 %cs 用于代码段， %ss 用于堆栈段，但一般 %fs 和 %gs 未被使用。段可以存储在全局 GDT 表或本地 LDT 表中。通过 GDT 中的一个条目访问 LDT。LDT 可以存储更多类型的段。LDT 可以是每个进程独立的。这两个表最多定义 8191 个条目。

#### 5.3.3. 在 Linux® i386 上的实现

在 Linux® 中有两种主要的设置 TLS 的方法。它可以在克隆进程时使用 clone 系统调用设置，或者可以调用 set_thread_area 。当进程将 CLONE_SETTLS 标志传递给 clone 时，内核期望由 %esi 寄存器指向的内存是 Linux® 用户空间段的表示，它将被转换为机器段的表示并加载到 GDT 插槽中。GDT 插槽可以用一个数字指定，或者可以使用 -1，意味着系统本身应选择第一个空闲插槽。实际上，绝大多数程序只使用一个 TLS 条目，并且不关心条目的编号。我们在仿真中利用了这一点，并且实际上依赖于它。

#### 5.3.4. Linux® TLS 仿真

##### 5.3.4.1. i386

当前线程加载 TLS 时，通过调用 set_thread_area 来完成；而在第二个进程加载 TLS 时，则在 clone 中的单独块中完成。这两个函数非常相似。唯一的区别在于 GDT 段的实际加载，对于新创建的进程，在下一次上下文切换时完成，而 set_thread_area 必须直接加载它。代码基本上做了这个操作。它复制来自用户空间的 Linux® 形式段描述符。代码检查描述符的编号，但由于 FreeBSD 和 Linux® 之间存在差异，我们对其进行了模拟。我们仅支持索引 6、3 和-1。其中，6 是真正的 Linux® 编号，3 是真正的 FreeBSD 编号，-1 表示自动选择。然后我们将描述符编号设为常数 3，并将其复制到用户空间。我们依赖于用户空间进程使用描述符编号，但这在大多数情况下都有效（从未见过这种方法不起作用的情况）。接着，我们将描述符从 Linux® 形式转换为机器相关形式（即操作系统无关形式），并将其复制到 FreeBSD 定义的段描述符中。最后，我们可以加载它。我们将描述符分配给线程的 PCB（进程控制块），并使用 load_gs 加载 %gs 段。这种加载必须在临界区中完成，以确保没有中断。 CLONE_SETTLS 情况完全相同，只是不执行 load_gs 的加载。用于此目的的段（段编号 3）在 FreeBSD 进程和 Linux® 进程之间共享，因此 Linux® 仿真层在普通 FreeBSD 上不会增加任何开销。

##### 5.3.4.2. amd64

amd64 实现类似于 i386，但最初没有用于此目的的 32 位段描述符（因此即使本机 32 位 TLS 用户也无法工作），所以我们必须添加这样的段并在每次上下文切换时加载它（当一个标志信号使用 32 位时）。除此之外，TLS 加载完全相同，只是段号不同，描述符格式和加载略有不同。

### 5.4. Futexes

#### 5.4.1. 引入同步

线程需要某种形式的同步，而 POSIX® 提供了一些：互斥锁用于互斥访问，读写锁用于具有读写偏向比例的互斥访问，条件变量用于信号状态变化。有趣的是 POSIX® 线程 API 缺乏对信号量的支持。这些同步例程的实现严重依赖于我们拥有的线程支持类型。在纯 1:M（用户空间）模型中，实现可以完全在用户空间中完成，因此非常快速（条件变量可能最终是使用信号实现的，即不太快）且简单。在 1:1 模型中，情况也很明确 - 线程必须使用内核设施进行同步（这非常慢，因为必须执行系统调用）。混合的 M:N 方案将第一种和第二种方法结合在一起，或者完全依赖内核。线程同步是支持线程的编程的重要组成部分，其性能可能会对最终程序产生很大影响。在 FreeBSD 操作系统上的最新基准测试显示，改进的 sx_lock 实现使 ZFS（一个重度使用 sx 的用户）加速了 40%，这是内核中的东西，但清楚显示了同步原语性能的重要性。

编写线程化程序时应尽量减少对锁的争用。否则，线程将只是在锁上等待，而不是执行有用的工作。因此，最精心编写的线程化程序显示出较少的锁争用。

#### 5.4.2. 互斥体介绍

Linux® 实现 1:1 线程，即必须使用内核同步原语。正如前面所述，编写良好的多线程程序几乎没有锁争用。因此，一个典型的序列可以执行为两个原子增加/减少互斥体引用计数，这非常快，如下面的例子所示：

```c
pthread_mutex_lock(&mutex);
...
pthread_mutex_unlock(&mutex);
```

1:1 线程强制我们对这些互斥调用执行两个系统调用，这非常慢。

Linux® 2.6 实现的解决方案称为 futexes。Futexes 在用户空间实现争用检查，只有在发生争用时才调用内核原语。因此，典型情况发生时不需要任何内核干预。这提供了合理快速且灵活的同步原语实现。

#### 5.4.3. Futex API

futex 系统调用如下所示：

```c
int futex(void *uaddr, int op, int val, struct timespec *timeout, void *uaddr2, int val3);
```

在这个例子中， uaddr 是用户空间中的互斥体地址， op 是我们即将执行的操作，其他参数具有每个操作的含义。

Futexes 实现以下操作：

- `FUTEX_WAIT`
- `FUTEX_WAKE`
- `FUTEX_FD`
- `FUTEX_REQUEUE`
- `FUTEX_CMP_REQUEUE`
- `FUTEX_WAKE_OP`

##### 5.4.3.1. FUTEX_WAIT

此操作验证在地址 uaddr 上写入值 val 。如果没有，则返回 EWOULDBLOCK ，否则将线程排队在 futex 上并挂起。如果参数 timeout 非零，则指定睡眠的最大时间，否则睡眠是无限的。

##### 5.4.3.2. FUTEX_WAKE

此操作获取 uaddr 处的 futex 并唤醒在此 futex 上排队的 val 个第一个 futex。

##### 5.4.3.3. FUTEX_FD

这个操作将文件描述符与给定的 futex 关联起来。

##### 5.4.3.4. FUTEX_REQUEUE

这个操作在 futex 上排队了 val 个线程，在 uaddr 处唤醒它们，接着获取 val2 个下一个线程并重新排队到 uaddr2 处的 futex。

##### 5.4.3.5. FUTEX_CMP_REQUEUE

这个操作与 FUTEX_REQUEUE 相同，但首先检查 val3 是否等于 val 。

##### 5.4.3.6. FUTEX_WAKE_OP

此操作对 val3 （其中包含一些其他值）执行原子操作，并唤醒位于 uaddr 处的 futex 上的 val 个线程；如果原子操作返回正数，则唤醒位于 uaddr2 处的 futex 上的 val2 个线程。

在 FUTEX_WAKE_OP 中实现的操作有：

- `FUTEX_OP_SET`
- `FUTEX_OP_ADD`
- `FUTEX_OP_OR`
- `FUTEX_OP_AND`
- `FUTEX_OP_XOR`

|     | futex 原型中没有 val2 参数。 val2 参数是从 struct timespec *timeout 参数中获取，用于操作 FUTEX_REQUEUE ， FUTEX_CMP_REQUEUE 和 FUTEX_WAKE_OP 。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------ |

#### 5.4.4. FreeBSD 中的 Futex 仿真

FreeBSD 中的 futex 仿真源自 NetBSD，并由我们进一步扩展。它位于 linux_futex.c 和 linux_futex.h 文件中。 futex 结构看起来像：

```c
struct futex {
  void *f_uaddr;
  int f_refcount;

  LIST_ENTRY(futex) f_list;

  TAILQ_HEAD(lf_waiting_paroc, waiting_proc) f_waiting_proc;
};
```

结构 waiting_proc 是：

```c
struct waiting_proc {

  struct thread *wp_t;

  struct futex *wp_new_futex;

  TAILQ_ENTRY(waiting_proc) wp_list;
};
```

##### 5.4.4.1. futex_get / futex_put

使用 futex_get 函数获取 futex，该函数搜索 futexes 的线性列表并返回找到的 futex 或创建新的 futex。释放 futex 时调用 futex_put 函数，该函数递减 futex 的引用计数，如果引用计数达到零，则释放。

##### 5.4.4.2. futex_sleep

当一个 futex 队列将一个线程置于睡眠状态时，它会创建一个 working_proc 结构并将该结构放入 futex 结构内部的列表中，然后仅执行 tsleep(9) 来挂起线程。睡眠可以超时。在 tsleep(9) 返回后（线程被唤醒或超时）， working_proc 结构将从列表中移除并销毁。所有这些操作都在 futex_sleep 函数中完成。如果从 futex_wake 被唤醒，我们会设置 wp_new_futex ，因此会在其上睡眠。这种方式实现了实际的重新排队操作。

##### 5.4.4.3. futex_wake

在 futex_wake 函数中，唤醒休眠在 futex 上的线程。首先，在这个函数中，我们模仿了奇怪的 Linux® 行为，在所有操作中唤醒 N 个线程，唯一的例外是 REQUEUE 操作在 N+1 个线程上执行。但通常这并没有任何区别，因为我们唤醒了所有线程。接下来，在循环中，我们唤醒 n 个线程，然后检查是否有新的 futex 需要重新排队。如果是这样，我们会在新的 futex 上重新排队最多 n2 个线程。这与 futex_sleep 协作。

##### 5.4.4.4. futex_wake_op

FUTEX_WAKE_OP 操作非常复杂。首先，我们获取地址为 uaddr 和 uaddr2 的两个 futex，然后使用 val3 和 uaddr2 执行原子操作。然后唤醒第一个 futex 上的 val 个等待者，如果原子操作条件成立，我们会唤醒第二个 futex 上的 val2 （即 timeout ）个等待者。

##### 5.4.4.5. futex 原子操作

原子操作接受两个参数 encoded_op 和 uaddr 。编码的操作将操作本身编码，比较值，操作参数和比较参数。操作的伪代码如下：

```c
oldval = *uaddr2
*uaddr2 = oldval OP oparg
```

这是原子地完成的。首先执行 uaddr 处的数字复制，然后执行操作。代码处理页面故障，如果没有页面故障发生，则使用 cmp 比较器将 oldval 与 cmparg 参数进行比较。

##### 5.4.4.6. Futex locking

Futex implementation uses two lock lists protecting `sx_lock` and global locks (either Giant or another `sx_lock`). Every operation is performed locked from the start to the very end.

### 5.5. Various syscalls implementation

在这一部分，我将描述一些较小的系统调用，这些系统调用值得一提，因为它们的实现并不明显，或者从其他角度看这些系统调用很有趣。

#### 5.5.1. *at 系列的系统调用

在 Linux® 2.6.16 内核的开发过程中，*at 系列的系统调用被添加进来了。这些系统调用（例如 openat ）与它们的非 at 版本完全相同，只是在 dirfd 参数上稍有不同。这个参数改变了要在哪个文件上执行系统调用。当 filename 参数是绝对路径时， dirfd 会被忽略，但当文件路径是相对路径时，它就会起作用。 dirfd 参数是一个相对于哪个目录来检查相对路径名的目录。 dirfd 参数是一些目录或 AT_FDCWD 的文件描述符。所以例如 openat 系统调用可以是这样的：

```c
file descriptor 123 = /tmp/foo/, current working directory = /tmp/

openat(123, /tmp/bah, flags, mode)	/* opens /tmp/bah */
openat(123, bah, flags, mode)		/* opens /tmp/foo/bah */
openat(AT_FDWCWD, bah, flags, mode)	/* opens /tmp/bah */
openat(stdio, bah, flags, mode)	/* returns error because stdio is not a directory */
```

此基础设施是为了避免在工作目录之外打开文件时发生竞争。 想象一个进程由两个线程组成，线程 A 和线程 B。 线程 A 发出 open(./tmp/foo/bah., flags, mode) ，然后在返回之前被抢占，线程 B 运行。 线程 B 不关心线程 A 的需求，并重命名或删除/tmp/foo/。 我们就会发生竞争。 为了避免这种情况，我们可以打开/tmp/foo 并将其用作 dirfd 的 openat 系统调用。 这还可以使用户实现每个线程的工作目录。

Linux® *at 系统调用系列包括： linux_openat ， linux_mkdirat ， linux_mknodat ， linux_fchownat ， linux_futimesat ， linux_fstatat64 ， linux_unlinkat ， linux_renameat ， linux_linkat ， linux_symlinkat ， linux_readlinkat ， linux_fchmodat 和 linux_faccessat 。 所有这些都是使用修改后的 namei(9)例程和简单封装层实现的。

##### 5.5.1.1. 实现

实现是通过修改上述描述的 namei（9）例程来完成的，以便在其结构中添加附加参数 dirfd ，该参数指定路径名查找的起始点，而不是每次都使用当前工作目录。从文件描述符号到 vnode 的解析是在本机*at 系统调用中完成的。当 dirfd 是 AT_FDCWD 时， nameidata 结构中的 dvp 条目是 NULL ，但是当 dirfd 是一个不同的数字时，我们获取这个文件描述符的文件，检查这个文件是否有效，如果有 vnode 连接到它，然后我们获得一个 vnode。然后我们检查这个 vnode 是否是一个目录。在实际的 namei（9）例程中，我们只需将 dvp vnode 替换为 namei（9）函数中的 dp 变量，该函数确定起始点。namei（9）并不直接使用，而是通过各个级别的不同函数的跟踪。例如 openat 如下所示：

```c
openat() --> kern_openat() --> vn_open() -> namei()
```

因此，必须修改 kern_open 和 vn_open 以包含额外的 dirfd 参数。对于这些情况，没有为其创建兼容层，因为这些情况的用户并不多，而且用户可以很容易地转换。这种通用实现使 FreeBSD 能够实现自己的*at 系统调用。这正在讨论中。

#### 5.5.2. Ioctl

ioctl 接口由于其通用性而非常脆弱。我们必须记住，在 Linux® 和 FreeBSD 之间的设备有所不同，因此必须谨慎地进行 ioctl 仿真工作。 ioctl 处理实现在 linux_ioctl.c 中，其中定义了 linux_ioctl 函数。该函数简单地遍历 ioctl 处理程序集以查找实现给定命令的处理程序。 ioctl 系统调用有三个参数，文件描述符、命令和一个参数。 命令是一个 16 位数字，理论上分为确定 ioctl 命令类别的高 8 位和给定中实际命令的低 8 位。仿真利用了这种划分。我们为每个实现处理程序，例如 sound_handler 或 disk_handler 。每个处理程序都有最大命令和最小命令定义，用于确定使用哪个处理程序。这种方法存在一些小问题，因为 Linux® 未一致使用划分，因此有时不属于其应属于的中（SCSI 通用 ioctl 在 cdrom 中等）。目前 FreeBSD 没有实现许多 Linux®ioctls（例如与 NetBSD 相比），但计划从 NetBSD 中 port 这些 ioctls。趋势是在本地 FreeBSD 驱动程序中甚至使用 Linux®ioctls，因为这些应用程序易于移植。

#### 5.5.3. 调试

每个系统调用都应该是可调试的。为此，我们引入了一个小的基础设施。我们有 ldebug 工具，它告诉我们是否应调试给定的系统调用（可以通过 sysctl 设置）。用于打印的是 LMSG 和 ARGS 宏。这些用于修改可打印的字符串以获得统一的调试消息。

## 6. 结论

### 6.1. 结果

截至 2007 年 4 月，FreeBSD 的 Linux® 仿真层能够很好地仿真 Linux® 2.6.16 内核。剩余的问题涉及 futexes，未完成的*at 系统调用系列，信号传递问题，缺少 epoll 和 inotify ，可能还有一些尚未发现的 bug。尽管如此，我们能够基本运行 FreeBSD Ports 收藏中包含的所有 Linux® 程序，使用的是 Fedora Core 4 和 2.6.16 内核，并且有一些初步的成功报告表明在 Fedora Core 6 和 2.6.16 上也能运行。最近提交了 Fedora Core 6 的 linux_base，进一步测试了仿真层，并提供了一些线索，告诉我们在实现缺失功能方面应该投入更多的努力。

我们能够运行像 www/linux-firefox、net-im/skype 这样的最常用的应用程序，以及 Ports 系列中的一些游戏。一些程序在 2.6 模拟下表现出不良行为，但当前正在调查中，希望很快能解决。已知无法运行的唯一大型应用程序是 Linux® Java™ 开发工具包，这是因为需要 epoll 设施，这与 Linux® 内核 2.6 并不直接相关。

我们希望在 FreeBSD 7.0 发布后的某个时候启用 2.6.16 模拟，默认情况下至少公开 2.6 模拟部分以进行更广泛的测试。一旦完成这一步，我们就可以切换到 Fedora Core 6 linux_base，这是最终计划。

### 6.2. 未来工作

未来的工作应集中于修复与 futex 相关的剩余问题，实现 *at 系列系统调用，修复信号传递，并可能实现 epoll 和 inotify 设施。

我们希望能够尽快无缝运行最重要的程序，以便可以默认切换到 2.6 模拟，并将 Fedora Core 6 设为默认的 linux_base，因为我们目前使用的 Fedora Core 4 不再受到支持。

另一个可能的目标是与 NetBSD 和 DragonflyBSD 共享我们的代码。NetBSD 对 2.6 模拟有一些支持，但还远未完成，且测试不充分。DragonflyBSD 对移植 2.6 改进表示了一些兴趣。

一般来说，随着 Linux® 的发展，我们希望能跟上他们的发展，实现新增的系统调用。首先想到的是 Splice。一些已实现的系统调用也是亚优化的，例如 mremap 等。还可以进行一些性能改进，例如更细粒度的锁定等。

### 6.3. 团队

我参与了这个项目，与以下人员合作（按字母顺序排列）:

- `John Baldwin <<a href="mailto:jhb@FreeBSD.org">jhb@FreeBSD.org</a>>`
- `Konstantin Belousov <<a href="mailto:kib@FreeBSD.org">kib@FreeBSD.org</a>>`
- 埃曼纽尔·德里福斯
- 斯科特·赫泽尔
- `Jung-uk Kim <<a href="mailto:jkim@FreeBSD.org">jkim@FreeBSD.org</a>>`
- `Alexander Leidinger <<a href="mailto:netchild@FreeBSD.org">netchild@FreeBSD.org</a>>`
- `Suleiman Souhlal <<a href="mailto:ssouhlal@FreeBSD.org">ssouhlal@FreeBSD.org</a>>`
- 李晓
- `David Xu <<a href="mailto:davidxu@FreeBSD.org">davidxu@FreeBSD.org</a>>`

我想感谢所有那些给予我的建议、代码审查和一般支持的人。

## 7. 文献

1. Marshall Kirk McKusick - George V. Nevile-Neil. FreeBSD 操作系统的设计与实现。Addison-Wesley，2005。
2. [https://tldp.org](https://tldp.org/)
3. [https://www.kernel.org](https://www.kernel.org/)
