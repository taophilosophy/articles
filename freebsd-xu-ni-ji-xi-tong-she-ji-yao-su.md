# FreeBSD 虚拟内存系统设计要素


<details open="" data-immersive-translate-walked="8bc458a5-bf4b-4880-a77a-a1279ebbba0e"><summary data-immersive-translate-walked="8bc458a5-bf4b-4880-a77a-a1279ebbba0e" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Linux 是 Linus Torvalds 的注册商标。

微软公司, IntelliMouse, MS-DOS, Outlook, Windows, Windows Media 和 Windows NT 是微软公司在美国和/或其他国家/地区的已注册商标或商标。

Motif, OSF/1 和 UNIX 是 The Open Group 在美国和其他国家/地区的已注册商标，IT DialTone 和 The Open Group 是 The Open Group 的商标。

本文最初发表在 DaemonNews 的 2000 年 1 月号。本文的这个版本可能包括来自 Matt 和其他作者的更新，以反映 FreeBSD 的 VM 实现中的变化。

制造商和销售商用来区分其产品的许多名称被视为商标。如果 FreeBSD 项目知悉商标声明，则在文档中出现这些名称时，它们会后跟“™”或“®”符号。

</details>

 摘要

Matthew Dillon <[dillon@apollo.backplane.com](mailto:dillon@apollo.backplane.com)>

标题实际上只是一种花哨的说法，我将尝试描述整个 VM 组件，希望每个人都能理解。在过去的一年里，我专注于 FreeBSD 内的许多主要内核子系统，最有趣的是 VM 和 Swap 子系统，而 NFS 则是“必不可少的琐事”。我只重写了代码的一小部分。在 VM 领域，我唯一主要的重写是对交换子系统的重写。我的大部分工作是清理和维护，只进行了适度的代码重写，在 VM 子系统内没有进行重大的算法调整。大多数 VM 子系统的理论基础保持不变，近年来对现代化工作的许多贡献应归功于 John Dyson 和 David Greenman。我不像 Kirk 那样是历史学家，不会尝试用人名标记所有不同的功能，因为我肯定会搞错。

---

## 1.介绍

在继续实际设计之前，让我们花一点时间讨论维护和现代化任何长寿代码库的必要性。在编程世界中，算法往往比代码更重要，正是由于 BSD 具有学术根源，从一开始就对算法设计给予了很多关注。更多关注设计通常会导致一个干净灵活的代码库，可以在时间的推移中相对容易地修改、扩展或替换。虽然有些人认为 BSD 是一个“老式”操作系统，但我们这些在其上工作的人更倾向于将其视为一个“成熟”的代码库，其中的各种组件都已被现代代码修改、扩展或替换。它已经发展，而 FreeBSD 正处于前沿，无论其中一些代码有多老。这是一个重要的区别，不幸的是，很多人却忽略了这一点。程序员可能犯的最大错误就是不从历史中汲取教训，这正是许多其他现代操作系统所犯的错误。Windows NT®就是这个最好的例子，后果是可怕的。Linux 也在某种程度上犯了这个错误，足以让我们 BSD 人偶尔开些小玩笑。Linux 的问题简单地说就是缺乏经验和历史来比较各种想法，这个问题正在被 Linux 社区以与 BSD 社区相同的方式迅速解决——通过持续的代码开发。另一方面，Windows NT®的人反复犯着 UNIX®数十年前解决的同样错误，然后花费数年来修复。一遍又一遍。他们有严重的“非本地设计”和“我们总是对的，因为我们的营销部门这么说”的情况。我对那些不能从历史中学习的人几乎没有容忍。

FreeBSD 设计的许多显而易见的复杂性，特别是在 VM/Swap 子系统中，直接源自于必须解决在各种条件下发生的严重性能问题。这些问题并非由于糟糕的算法设计，而是起因于环境因素。在直接比较不同平台时，这些问题在系统资源开始受到压力时变得最为显著。当我描述 FreeBSD 的 VM/Swap 子系统时，读者应始终牢记两点：

1. 性能设计中最重要的方面是所谓的“优化关键路径”。通常情况下，性能优化会在代码中增加一些臃肿，以使关键路径表现更好。
2. 一个坚实的通用设计在长期内优于严重优化的设计。尽管在它们首次实施时，通用设计可能比严重优化的设计变得更慢，但通用设计倾向于更容易适应变化的条件，而严重优化的设计最终必须被丢弃。

任何能够在多年内生存并且易于维护的代码库，从一开始就必须进行合理设计，即使这可能会牺牲一些性能。二十年前，人们仍在争论用汇编语言编程比高级语言编程效率更高，因为它生成的代码快十倍。今天，这种论点的错误显而易见——算法设计和代码泛化的类比也同样如此。

## 2. VM 对象

描述 FreeBSD 虚拟内存系统的最佳方法是从用户级进程的角度来看待它。每个用户进程看到一个单独的、私有的、连续的虚拟内存地址空间，其中包含多种类型的内存对象。这些对象具有各种特性。程序代码和程序数据实际上是一个内存映射文件（运行的二进制文件），但程序代码是只读的，而程序数据是写时复制的。程序的 BSS 区只是按需分配并填充零值的内存，称为需求零页填充。任意文件也可以映射到地址空间中，这就是共享库机制的工作原理。这些映射可能需要修改以保持对使其的进程私有。fork 系统调用在现有复杂性的基础上增加了虚拟内存管理问题的全新维度。

一个程序二进制数据页（即基本的写时复制页）展示了其复杂性。程序二进制包含一个预初始化的数据段，最初直接从程序文件映射。当程序加载到进程的虚拟内存空间时，这个区域最初是内存映射的，并由程序二进制本身支持，允许虚拟内存系统释放/重用页面，并稍后从二进制文件中重新加载。然而，一旦进程修改了这些数据，虚拟内存系统必须为该进程制作页面的私有副本。由于私有副本已经被修改，虚拟内存系统可能不再释放它，因为后续无法恢复它。

你会立即注意到，最初的简单文件映射变得更加复杂。数据可以逐页修改，而文件映射则涵盖多个页面。当进程分叉时，复杂性进一步增加。进程分叉时，结果是两个具有自己私有地址空间的进程，包括原始进程在调用 fork() 之前所做的任何修改。在 fork() 时完全复制数据将是愚蠢的，因为至少两个进程中的一个可能只需从那时开始读取该页面，允许原始页面继续使用。一个私有页面再次被设为写时复制，因为每个进程（父进程和子进程）都希望他们自己的分叉后修改对自己私有，并且不影响其他进程。

FreeBSD 通过分层的 VM 对象模型管理所有这些。原始的二进制程序文件最终成为最低的 VM 对象层。在其上方推送一个写时复制层，用于保存那些必须从原始文件复制的页面。如果程序修改了属于原始文件的数据页面，VM 系统会出现故障，并在较高层中复制页面。当进程进行分叉时，会推送额外的 VM 对象层。通过一个相当基本的示例，这可能会更容易理解一些。对于任何 *BSD 系统来说， fork() 是一个常见的操作，因此此示例将考虑一个启动并进行分叉的程序。当进程启动时，VM 系统会创建一个对象层，让我们称其为 A：

![A picture](https://docs.freebsd.org/images/articles/vm-design/fig1.png)

A 代表文件页面，可以根据需要从文件的物理介质中调入和调出。从磁盘中调入对于程序来说是合理的，但我们确实不想再调出并覆盖可执行文件。因此，VM 系统创建了第二层，B，它将由交换空间物理支持：

![fig2](https://docs.freebsd.org/images/articles/vm-design/fig2.png)

在此之后第一次写入页面时，在 B 中创建一个新页面，并从 A 初始化其内容。B 中的所有页面都可以调入或调出到交换设备。当程序进行分叉时，VM 系统会为父进程创建两个新的对象层-C1，为子进程创建一个新的对象层-C2，它们都位于 B 的顶部：

![fig3](https://docs.freebsd.org/images/articles/vm-design/fig3.png)

在这种情况下，假设 B 中的一个页面被原始父进程修改。进程将执行写时复制错误并在 C1 中复制页面，保持 B 中的原始页面不变。现在，假设子进程修改了 B 中相同的页面。进程将执行写时复制错误并在 C2 中复制页面。由于 C1 和 C2 都有副本，B 中的原始页面现在完全隐藏，如果 B 不表示一个"真实"文件，则可能会被销毁；但是，这种优化并不容易实现，因为它非常细粒度。FreeBSD 不进行这种优化。现在，假设（通常情况下）子进程执行 exec() 。它的当前地址空间通常会被表示为一个新文件的新地址空间所取代。在这种情况下，C2 层被销毁：

![fig4](https://docs.freebsd.org/images/articles/vm-design/fig4.png)

在这种情况下，B 的子代数量减少到一个，并且现在所有对 B 的访问都通过 C1 进行。这意味着 B 和 C1 可以合并在一起。在合并过程中，从 B 中删除在 C1 中也存在的任何页面。因此，即使在前一步中无法进行优化，我们可以在任何一个进程退出或 exec() 时恢复死页面。

这种模型会产生一些潜在问题。首先是，当你发生错误时，你可能会得到一堆层次深的层 VM 对象，这可能会耗费扫描时间和内存。深层堆叠可能会在进程分叉后再次分叉时发生（无论是父进程还是子进程）。第二个问题是，你可能会在 VM 对象的堆栈深处得到一些死的、无法访问的页面。在我们的最后一个例子中，如果父进程和子进程都修改同一页，他们都会得到自己的页面副本，而 B 中的原始页面将不再被任何人访问。B 中的页面可以被释放。

FreeBSD 通过称为“全部覆盖案例”的特殊优化解决了深层分层问题。如果 C1 或 C2 进行足够的 COW 错误以完全覆盖 B 中的所有页面，则会发生这种情况。假设 C1 达到了这一点。C1 现在可以完全绕过 B，所以我们不再有 C1→B→A 和 C2→B→A，而是有 C1→A 和 C2→B→A。但看看还发生了什么——现在 B 只有一个引用（C2），所以我们可以将 B 和 C2 合并在一起。最终结果是 B 完全被删除，我们有 C1→A 和 C2→A。通常情况下，B 会包含大量页面，C1 和 C2 都无法完全覆盖它。但是，如果我们再次 fork 并创建一组 D 层，则更有可能最终会有一个 D 层能够完全覆盖 C1 或 C2 所代表的较小数据集。这种优化在图中的任何一点都适用，其最终结果是即使在大量 fork 的机器上，VM 对象堆栈的深度通常也不会超过 4。这对父进程和子进程都是如此，无论是父进程在 fork 还是子进程在级联 fork。

在 C1 或 C2 未能完全覆盖 B 的情况下，仍然存在死页问题。由于我们的其他优化，这种情况并不构成太大问题，我们只是允许这些页面死掉。如果系统内存不足，它会将它们交换出去，消耗一些交换空间，但仅此而已。

VM 对象模型的优势在于 fork() 非常快，因为不需要进行真正的数据复制。缺点是你可以构建一个相对复杂的 VM 对象层，这会稍微减慢页面错误处理速度，并且你需要花费内存来管理 VM 对象结构。FreeBSD 所做的优化证明足以减少问题，使其可以忽略不计，从而没有真正的缺点。

## 3. SWAP 层

私有数据页最初是写时复制或零填充页。当进行更改并因此进行复制时，原始后备对象（通常是文件）在虚拟内存系统需要重新使用该页以供其他目的时不能再用于保存该页的副本。这就是 SWAP 的作用所在。SWAP 被分配用于为其他情况下没有备份存储的内存创建后备存储。FreeBSD 只有在实际需要时才为 VM 对象分配交换管理结构。然而，交换管理结构在历史上存在问题：

* 在 FreeBSD 3.X 版本中，交换管理结构预先分配一个涵盖整个需要交换后备存储对象的数组，即使该对象只有几页被交换后备。当映射大对象或具有大运行大小（RSS）的进程进行复制时，这会导致内核内存碎片问题。
* 此外，为了跟踪交换空间，内核内存中保存着一个“空洞列表”，这往往也会严重碎片化。由于“空洞列表”是一个线性列表，因此交换分配和释放的性能是一个非最佳的 O(n)-每页操作。
* 在交换释放过程中需要进行内核内存分配，这会造成低内存死锁问题。
* 由于交错算法造成的空洞问题进一步恶化了这一问题。
* 而且，交换块映射很容易变得碎片化，导致非连续分配。
* 当发生交换时，必须动态分配内核内存以用于额外的交换管理结构。

从该列表中可以看出，有很多改进的空间。对于 FreeBSD 4.X，我完全重写了交换子系统：

* 通过哈希表分配交换管理结构，而不是线性数组，使它们具有固定的分配大小和更精细的粒度。
* 而不是使用线性链接列表来跟踪交换空间保留，现在使用一种位图，位于以 free-space hinting 为特征的基数树结构中。这实际上使得交换分配和释放成为 O(1) 操作。
* 整个基数树位图也是预先分配的，以避免在关键低内存交换操作期间分配内核内存。毕竟，系统在内存不足时往往会进行交换，因此我们应该避免在这种时候分配内核内存，以避免潜在的死锁。
* 为了减少碎片化，基数树能够一次分配大块连续的内存，跳过较小的碎片化块。

我并未采取最终步骤，即设置一个“分配提示指针”，当进行分配时，它会在一部分交换空间中移动，以进一步保证连续分配，或至少保证引用的局部性，但我确保可以添加这样的功能。

## 4. 何时释放页面

由于 VM 系统使用所有可用内存进行磁盘缓存，因此真正空闲的页面通常很少。VM 系统依赖于能够正确选择未使用的页面以便为新分配重用。选择要释放的最佳页面可能是任何 VM 系统可以执行的最重要功能，因为如果选择不当，VM 系统可能被迫不必要地从磁盘检索页面，严重降低系统性能。

在关键路径中，我们愿意承受多少额外开销以避免释放错误的页面？每次错误选择都会给我们带来数十万个 CPU 周期的损失，并显著延迟受影响的进程，因此我们愿意忍受相当大的开销，以确保选择正确的页面。这就是为什么在内存资源紧张时，FreeBSD 往往优于其他系统。

空闲页面确定算法建立在对内存页面使用历史的基础上。为了获取这个历史记录，系统利用大多数硬件页表具有的页面使用位功能。

在任何情况下，页面使用位被清除，稍后虚拟内存系统再次访问页面时发现页面使用位已设置。这表明页面仍然在被活动使用。如果位仍然清除，则表示页面未被活动使用。通过定期测试这个位，可以为物理页面开发使用历史（以计数形式）。当虚拟内存系统稍后需要释放一些页面时，检查这个历史成为确定重新使用最佳候选页面的基石。

对于那些没有这个特性的平台，系统实际上会模拟一个页面使用位。它会取消映射或保护页面，如果页面再次被访问就会强制产生页面错误。当发生页面错误时，系统简单地标记页面已被使用并取消保护页面以便可以使用。虽然仅仅为了确定页面是否正在使用而产生这样的页面错误似乎是一个昂贵的事情，但这比将页面重新用于其他目的然后发现某个进程需要它并且不得不去磁盘读取要便宜得多。

FreeBSD 利用多个页面队列进一步细化要重用的页面的选择，以及确定必须将脏页面刷新到其后备存储器的时间。由于在 FreeBSD 下，页表是动态实体，因此从使用它的任何进程的地址空间中取消映射页面几乎没有成本。当基于页面使用计数器选择了一个页面候选时，这正是所做的。系统必须区分可以在任何时候理论上释放的干净页面，以及必须在可重用之前将其写入后备存储器的脏页面。当找到一个页面候选时，如果它是脏的，则将其移动到非活动队列，如果是干净的，则移动到缓存队列。基于脏-干净页面比率的单独算法确定了何时必须刷新非活动队列中的脏页面到磁盘。完成此操作后，刷新的页面从非活动队列移动到缓存队列。此时，缓存队列中的页面仍然可以通过 VM 故障以相对较低的成本重新激活。但是，缓存队列中的页面被视为“立即可释放”，并且在系统需要分配新内存时将以 LRU（最近最少使用）方式重复使用。

需要注意的是，FreeBSD VM 系统试图分离干净页面和脏页面，以避免不必要地刷新脏页面（这会消耗 I/O 带宽），也不会在内存子系统未受到压力时毫无理由地在各种页面队列之间移动页面。这就是为什么在执行 systat -vm 命令时，你会看到一些系统具有非常低的缓存队列计数和高的活动队列计数。随着 VM 系统变得更加紧张，它会更加努力地保持各种页面队列在被确定为最有效的水平上。

多年来有一种都市传说认为 Linux 在避免交换方面比 FreeBSD 做得更好，但事实上并非如此。实际情况是，FreeBSD 主动将未使用的页面换出以腾出更多磁盘缓存空间，而 Linux 则将未使用的页面保留在内核中，从而减少了可用于缓存和进程页面的内存。我不知道现在是否仍然如此。

## 5. 预先错误和零填充优化

如果底层页面已经在内核中并且可以简单地映射到进程中，那么发生 VM 错误并不昂贵，但如果你经常发生大量这样的错误，它可能会变得昂贵。一个很好的例子是反复运行 ls(1) 或 ps(1) 这样的程序。如果程序二进制文件映射到内存中但没有映射到页表中，那么每次运行程序时，程序将访问的所有页面都必须被错误地调入。这是没有必要的，因为这些页面已经在 VM 缓存中，所以 FreeBSD 将尝试使用那些已经在 VM 缓存中的页面预先填充进程的页表。FreeBSD 目前还没有做的一件事是在 exec 期间预写某些页面。例如，如果你在运行 vmstat 1 时运行 ls(1) 程序，你会注意到即使你反复运行它，它总是会发生一定数量的页面错误。这些是零填充错误，而不是程序代码错误（这些错误已经预先发生）。在 exec 或 fork 时预复制页面是一个需要更多研究的领域。

大部分发生的页面错误是零填充错误。通常可以通过观察 vmstat -s 输出来看到这一点。这些错误发生在进程访问其 BSS 区域时。BSS 区域预期最初为零，但 VM 系统在进程实际访问之前不会分配任何内存。发生错误时，VM 系统不仅必须分配一个新页面，还必须将其清零。为了优化清零操作，VM 系统能够预清零页面并标记为这样，并在发生零填充错误时请求预清零页面。预清零发生在 CPU 空闲时，但系统预清零页面的数量有限，以避免影响内存缓存。这是一个优化关键路径的 VM 系统复杂性增加的绝佳示例。

## 6. 页面表优化

页表优化构成了 FreeBSD VM 设计中最有争议的部分，并且随着对 mmap() 的严重使用而显示出一些压力。我认为这实际上是大多数 BSD 的特性，尽管我不确定它是何时首次引入的。有两个主要的优化。第一个是硬件页表不包含持久状态，而是可以随时丢弃，只需很少量的管理开销。第二个是系统中的每个活动页表条目都有一个控制 pv_entry 结构，它与 vm_page 结构绑定。FreeBSD 可以简单地遍历已知存在的映射，而 Linux 必须检查可能包含特定映射的所有页表，以查看是否存在，这在某些情况下可能会造成 O(n^2) 的开销。正是因为这个原因，在内存受到压力时，FreeBSD 倾向于在哪些页面可以重新使用或交换时做出更好的选择，在负载下具有更好的性能。然而，FreeBSD 需要对内核进行调优，以适应可能在新闻系统中发生的大共享地址空间情况，因为它可能会用尽 pv_entry 结构。

Linux 和 FreeBSD 在这个领域都需要改进。FreeBSD 正在努力最大化潜在的稀疏活动映射模型的优势（例如，并非所有进程都需要映射共享库的所有页面），而 Linux 正在努力简化其算法。在这里，FreeBSD 通常具有性能优势，但会浪费一点额外的内存，但在大量进程大量共享一个大文件的情况下，FreeBSD 会崩溃。另一方面，在许多进程稀疏映射同一个共享库并且在尝试确定页面是否可以重新使用时，Linux 会崩溃。

## 7. 页着色

我们将以页面着色优化结束。页面着色是一种性能优化，旨在确保在虚拟内存中对连续页面的访问能够最佳利用处理器缓存。在古代（即 10 多年前），处理器缓存往往映射虚拟内存而不是物理内存。这导致了大量问题，包括在某些情况下每次上下文切换时必须清除缓存，以及缓存中的数据别名问题。现代处理器缓存精确映射物理内存来解决这些问题。这意味着在进程地址空间中相邻的两个页面在缓存中可能并不相邻。实际上，如果不小心，虚拟内存中相邻的页面可能最终使用处理器缓存中的同一页面——导致缓存数据被过早丢弃并降低 CPU 性能。即使在多路关联缓存中也是如此（尽管效果有所减弱）。

FreeBSD 的内存分配代码实现了页面着色优化，这意味着内存分配代码将尝试找到从缓存视角看连续的空闲页面。例如，如果物理内存的第 16 页被分配给进程虚拟内存的第 0 页，而缓存可以容纳 4 页，页面着色代码不会将物理内存的第 20 页分配给进程虚拟内存的第 1 页。相反，它会分配物理内存的第 21 页。页面着色代码会尽量避免分配第 20 页，因为这会与第 16 页映射到相同的缓存内存，从而导致非最佳缓存。这段代码为虚拟内存分配子系统增加了相当大的复杂性，但结果是值得的。页面着色使虚拟内存在缓存性能方面与物理内存一样具有确定性。

## 8. 结论

现代操作系统中的虚拟内存必须有效地解决许多不同问题，并适用于许多不同的使用模式。BSD 历史上采用的模块化和算法方法使我们能够研究和理解当前的实现，同时相对清晰地替换代码的大部分部分。在过去几年中，FreeBSD VM 系统已经进行了许多改进，工作仍在进行中。

## 9. Allen Briggs 的奖励问答环节

### 9.1. 在你列出的 FreeBSD 3.X 交换安排的问题中，你提到的交织算法是什么？

FreeBSD 使用固定的交换交错，默认为 4。这意味着即使只有一个、两个或三个交换区，FreeBSD 也会保留四个交换区的空间。由于交换是交错的，表示“四个交换区”的线性地址空间将在实际上没有四个交换区的情况下变得分散。例如，如果你有两个交换区 A 和 B，FreeBSD 对该交换区的地址空间表示将以 16 页的块进行交错：

```
A B C D A B C D A B C D A B C D
```

FreeBSD 3.X 使用“自由区域顺序列表”方法来记录自由交换区。其思想是可以用单个列表节点（kern/subr_rlist.c）表示较大的自由线性空间块。但由于碎片化，顺序列表最终会变得非常分散。在上面的例子中，完全未使用的交换将显示为 A 和 B 为“自由”，C 和 D 为“全部分配”。每个 A-B 序列都需要一个列表节点来记录，因为 C 和 D 是空洞，所以列表节点不能与下一个 A-B 序列合并。

我们为什么要交错使用交换空间，而不是将交换区附加到末尾并做某些更复杂的操作呢？在地址空间中分配线性空间段，并且自动将结果交错跨多个磁盘是要比尝试在其他地方实现这种复杂性容易得多。

碎片化会引发其他问题。在 3.X 以下是线性列表，并且由于固有碎片化非常严重，因此分配和释放交换空间最终变成了 O(N)算法而不是 O(1)算法。再加上其他因素（大量交换），就会导致 O(N^2)和 O(N^3)级别的开销，这是很糟糕的。在 3.X 系统中，可能还需要在交换操作期间分配 KVM 以创建一个新的列表节点，这可能会导致在低内存情况下系统试图将页面分页出现死锁。

在 4.X 以下，我们不使用顺序列表。相反，我们使用基数树和交换块的位图，而不是范围列表节点。我们付出了预先分配整个交换区域所需位图的代价，但由于使用了位图（每个块一个位），导致浪费的内存更少。使用基数树而不是顺序列表，使我们几乎无论树变得多么碎片化，性能都接近 O(1)。

### 9.2. 在 systat -vm 中，干净页面和脏（非活动）页面的分离与看到低缓存队列计数和高活动队列计数的情况有关吗？systat 统计数据是否将活动和脏页面合并到活动队列计数中？

是的，那很令人困惑。关系是“目标”与“现实”。我们的目标是分开页面，但现实是，如果我们没有内存紧缩，我们不一定需要。

这意味着当系统不受压力时，FreeBSD 不会非常努力地将脏页面（非活动队列）与干净页面（缓存队列）分开，也不会尝试停用页面（活动队列→非活动队列），即使它们没有被使用。

### 在 ls(1) / vmstat 1 的例子中，一些页面错误会不会是数据页面错误（从可执行文件到私人页面的写时复制）？即，我会期望页面错误是一些零填充和一些程序数据。或者你是否暗示 FreeBSD 确实为程序数据执行预写时复制？

一个 COW 故障可以是零填充或程序数据。无论哪种方式，机制都是相同的，因为支持程序数据的缓存几乎肯定已经存在。我确实把这两种情况合并在一起。FreeBSD 不会预先 COW 程序数据或零填充，但它会预先映射存在于其缓存中的页面。

### 9.4. 在页表优化部分中，你能否更详细地介绍一下 pv_entry 和 vm_page（或者 vm_page 应该是 vm_pmap，就像在第 4.4 节中一样，参见 McKusick、Bostic、Karel、Quarterman 的第 180-181 页）？具体来说，什么样的操作/反应会需要扫描映射？

vm_page 代表（对象，索引#）元组。 pv_entry 代表硬件页表条目（pte）。如果有五个共享同一物理页面的进程，其中三个进程的页表实际上映射了该页面，那么该页面将由一个 vm_page 结构和三个 pv_entry 结构表示。

pv_entry 结构仅代表 MMU 映射的页面（一个 pv_entry 代表一个 pte）。这意味着，当我们需要移除对 vm_page 的所有硬件引用（为了重新使用页面、将其分页、清除、标记脏等），我们只需扫描与该 vm_page 关联的 pv_entry 链表，以移除或修改它们的页表 pte。

在 Linux 下，没有这样的链表。要移除一个 vm_page 的所有硬件页表映射，Linux 必须索引到可能映射页面的每个 VM 对象。例如，如果有 50 个进程都映射了同一个共享库，并且想要移除该库中的页面 X，即使只有其中的 10 个进程实际映射了该页面，你也需要为这 50 个进程的每一个索引到页表中。因此，Linux 在设计的简单性与性能之间进行了权衡。许多在 FreeBSD 下是 O(1)或（小 N）的 VM 算法在 Linux 下可能变成了 O(N)、O(N^2)或更差。由于对象中表示特定页面的 pte 通常在所有映射它们的页表中都位于相同的偏移量处，减少对相同 pte 偏移量页表的访问次数通常可以避免刷新该偏移量的 L1 缓存行，从而提升性能。

FreeBSD 引入了复杂性（ pv_entry 方案）以提高性能（限制仅需修改的 pte 的页表访问）。

但 FreeBSD 存在一个扩展问题，而 Linux 没有，就是存在有限数量的 pv_entry 结构，当数据共享非常频繁的时候就会出问题。在这种情况下，即使有大量空闲内存，你可能会耗尽 pv_entry 结构。这可以很容易地通过在内核配置中增加 pv_entry 结构的数量来解决，但我们真的需要找到更好的解决办法。

关于页表与 pv_entry 方案的内存开销：Linux 使用“永久”页表，不是一次性使用的，但不需要为每个可能映射的 pte 使用 pv_entry 。FreeBSD 使用“一次性”页表，但为每个实际映射的 pte 添加一个 pv_entry 结构。我认为内存利用率最终大致相同，使得 FreeBSD 在能够随意丢弃页表的同时具有算法优势，而开销非常低。

### 最后，在页着色部分，更详细地描述你在这里的意思可能会有所帮助。我没有完全明白。

你知道 L1 硬件内存缓存是如何工作的吗？我来解释一下：考虑一台有 16MB 主内存但只有 128K L1 缓存的机器。一般来说，这种缓存的工作方式是，主内存中的每个 128K 块都使用相同的 128K 缓存。如果你访问主内存中的偏移量 0，然后访问偏移量 128K，你可能会丢弃你从偏移量 0 读取的缓存数据！

现在，我大大地简化了事情。我刚刚描述的是所谓的“直接映射”硬件内存缓存。大多数现代缓存是所谓的 2 路组关联或 4 路组关联缓存。组关联允许你访问最多 N 个不同的内存区域，这些区域与同一缓存内存重叠，而不会破坏先前缓存的数据。但是只有 N。

因此，如果我有一个 4 路组关联缓存，我可以访问偏移量 0、偏移量 128K、256K 和偏移量 384K，仍然可以再次访问偏移量 0，且从 L1 缓存中获取数据。然而，如果我然后访问偏移量 512K，之前缓存的四个数据对象中的一个将被缓存抛弃。

大多数处理器的大部分内存访问都能从 L1 缓存中获取是非常重要的，因为 L1 缓存的操作频率与处理器频率相同。一旦发生 L1 缓存未命中并不得不访问 L2 缓存或主内存，处理器将停顿，有可能在等待来自主内存的读取操作完成时，一动不动地等待数百条指令的时间。与现代处理器核心的速度相比，主内存（你放入计算机中的动态 RAM）速度较慢。

现在谈谈页面着色：所有现代内存缓存都是物理缓存。它们缓存物理内存地址，而不是虚拟内存地址。这使得在进行进程上下文切换时可以不用干涉缓存，这点非常重要。

但在 UNIX®世界中，你处理的是虚拟地址空间，而不是物理地址空间。你编写的任何程序都将看到分配给它的虚拟地址空间。支撑该虚拟地址空间的实际物理页面未必是物理上连续的！事实上，在一个进程的地址空间中，可能会有两个相邻的页面最终位于物理内存中的偏移 0 和偏移 128K。

一个程序通常假定两个相邻页面会被最优地缓存。也就是说，你可以访问这两个页面中的数据对象，而不会使它们相互干扰的缓存条目失效。但前提是虚拟地址空间下的物理页面是连续的（就缓存而言）。

这就是页面着色的作用。页面着色不是将随机的物理页面分配给虚拟地址，这可能导致非最优的缓存性能。页面着色将相对连续的物理页面分配给虚拟地址。因此，程序可以在这样一个假设下编写：对于其虚拟地址空间，底层硬件缓存的特性与在物理地址空间中直接运行程序时相同。

注意，我说的是“相对”连续而不是简单地“连续”。对于一个 128K 的直接映射缓存来说，物理地址 0 和物理地址 128K 是相同的。因此，你虚拟地址空间中的两个相邻页面可能在物理内存中是偏移 128K 和 132K，但也可能是偏移 128K 和 4K，仍保持相同的缓存性能特性。因此，页面着色不必将物理内存的真正连续页面分配给虚拟内存中的连续页面，它只需要确保从缓存性能和操作角度来看，分配的页面是连续的。
