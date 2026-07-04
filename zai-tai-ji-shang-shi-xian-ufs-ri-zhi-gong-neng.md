# 在台机上实现 UFS 日志功能

- 原文：[Implementing UFS Journaling on a Desktop PC](https://docs.freebsd.org/en/articles/gjournal-desktop/)

## 摘要

日志文件系统使用日志来记录文件系统更新，并在系统崩溃或断电时保持一致性。尽管文件中未保存的更改仍可能丢失，但日志记录大大降低了因非正常关机导致文件系统损坏的风险，并显著缩短了恢复时间。虽然 FreeBSD 使用的 UFS 文件系统本身并未将日志记录实现为磁盘上的固有特性，但 FreeBSD 通过文件系统级机制（带日志记录的软更新）以及 GEOM 框架（`gjournal`）提供日志记录支持。本文介绍了可用的 UFS 日志记录机制，并说明了它们在现代 FreeBSD 系统上的正确用法。*本文内容已针对 FreeBSD 13 至 15 版本进行了审阅和更新*。


## 1. 介绍

虽然专业服务器通常能很好地防范意外关机，但典型的桌面 PC 往往容易受到断电、意外重启和其他用户相关事件的影响，这些情况可能导致非正常关机并使文件系统处于不一致状态。传统上，这需要运行 `fsck`，对于大型文件系统可能需要花费大量时间。在极少数情况下，文件系统损坏会严重到需要用户干预，并且可能导致数据丢失。

FreeBSD 为 UFS 文件系统提供了两种不同的日志记录机制：

- 带日志记录的软更新
- GEOM 日志记录（`gjournal`）

这些机制在系统的不同层次上运行，并具有不同的性能和语义特性。

>**重要**
>
>对于大多数现代 FreeBSD 系统，包括桌面和通用服务器，*带日志记录的软更新* 是推荐的解决方案。
>
>GEOM 日志记录是一种遗留的专用机制，仅当需要其特定语义时才应使用。

本文将解释这两种机制、它们的区别以及现代的正确用法。

阅读本章后，你将了解：

- FreeBSD 中可用于 UFS 文件系统的日志记录机制以及它们的区别。
- 何时使用文件系统级的软更新日志记录，以及何时适合使用 GEOM 日志记录。
- 如何在现有 UFS 文件系统上启用或禁用软更新日志记录。
- 如何在新分区或现有分区上配置 GEOM 日志记录，包括所需的内核支持和日志大小注意事项。
- 挂载日志文件系统需要进行哪些配置更改，以及日志记录如何影响系统行为。
- 如何诊断和解决与 UFS 日志记录相关的常见问题。

在阅读本文之前，你应该能够：

- 了解基本的 UNIX(R) 和 FreeBSD 概念。
- 熟悉使用 bsdinstall 的 FreeBSD 安装过程。
- 具备磁盘分区和文件系统的基础知识，包括 `gpart(8)`、`newfs(8)` 和 `mount(8)` 等工具。

>**警告**
>
>本文所述的部分操作涉及修改文件系统或磁盘配置，可能需要卸载文件系统或更改磁盘上的元数据。在生产系统上进行此类更改之前，请确保所有重要数据都有可靠的 *备份*。
>
>启用文件系统级的软更新日志记录通常是安全的，不需要重新分区磁盘。但是，配置 GEOM 日志记录涉及低级磁盘操作，只有完全理解其影响的经验丰富的管理员才应尝试。

## 2. 在 FreeBSD 中理解日志记录

### 2.1. 带日志记录的软更新（推荐）

软更新是 FreeBSD 中默认的 UFS 一致性机制。当与日志记录结合使用时，它可在文件系统内部提供 *元数据日志记录*。

带日志记录的软更新的优点包括：

- 崩溃恢复速度极快（通常只需数秒）
- 正常的 `sync(2)` 和 `fsync(2)` 语义
- 无需额外的分区或 GEOM 层
- 配置和维护简单

摘自 `tunefs(8)`：

>启用日志记录后，可将 `fsck_ffs(8)` 在崩溃后清理文件系统所需的时间从数分钟到数小时缩短至数秒。

此机制适用于几乎所有基于 UFS 的系统，并且是默认推荐方案。

### 2.2. GEOM 日志记录（`gjournal`）

GEOM 日志记录在块级别运行，位于文件系统之下。它记录所有块写入，*包括元数据和文件数据*。

GEOM 日志记录的主要特点：

- 作为 GEOM 类实现（`geom_journal`）
- 记录所有块 I/O，而不仅仅是元数据
- 需要使用 `-J` 标志获得 UFS 的明确配合
- 禁用软更新
- 改变 `sync(2)` 和 `fsync(2)` 的语义

>**警告**
>
>在 GEOM 日志文件系统上，`sync(2)` 和 `fsync(2)` 不保证数据已提交到稳定存储。要确保持久性，必须使用 `gjournal sync`。

由于这些差异，不建议将 GEOM 日志记录用于通用桌面或服务器。

## 3. 选择合适的日志记录方法

下表汇总了推荐用法：

| 使用场景 | 推荐机制 |
| :---: | :---: |
| 桌面或笔记本电脑系统 | 带日志记录的软更新 |
| 通用服务器 | 带日志记录的软更新 |
| 旧版 UFS 系统 | 带日志记录的软更新 |
| 与文件系统无关的日志记录需求 | GEOM 日志记录 |
| 专用块级日志记录需求 | GEOM 日志记录 |

## 4. 使用软更新与日志记录

带日志记录的软更新为 UFS 提供文件系统级日志记录。此机制在保留软更新的分配和排序优化的同时，维护文件系统一致性。与 GEOM 日志记录不同，它不需要单独的日志设备或分区。

### 4.1. 概述

带日志记录的软更新将文件系统元数据更改记录到存储在文件系统内部的日志中，该日志以文件系统根目录下一个名为 **.sujournal** 的隐藏文件形式实现。在发生崩溃或断电时，待处理的元数据操作会从日志中重放，使文件系统能够快速挂载而无需完整的一致性检查。

这种日志记录形式仅适用于文件系统元数据。文件数据完整性仍由应用程序负责。

### 4.2. 在现有文件系统上启用日志记录

更改日志记录设置之前，文件系统必须卸载或以只读方式挂载。

```sh
# umount /usr
# tunefs -n enable -j enable /dev/ada0p2
# mount /usr
```

无需额外配置。可以使用标准挂载选项。

### 4.3. 日志存储与大小

启用日志记录时会自动选择默认日志大小。在大多数情况下，默认大小足够，无需调整。

如有必要，可使用 `tunefs(8)` 的 `-S` 选项显式指定日志大小（以字节为单位）：

```sh
# tunefs -S 64000000 /dev/ada0p2
```

日志大小调整很少需要，仅应针对元数据更新率异常高的文件系统考虑。

### 4.4. 检查软更新与日志记录

使用软更新的 UFS 文件系统的日志记录状态可通过 `tunefs(8)` 或 `dumpfs(8)` 验证。

要显示当前文件系统设置，请运行：

```sh
# tunefs -p /dev/ada0p2 | grep -i journal
tunefs: soft updates journaling: (-j)   enabled
tunefs: gjournal: (-J)                  disabled
```

在输出中查找以下指标：

- 软更新启用
- 软更新日志记录启用

或者，可以使用 `dumpfs(8)` 检查文件系统超级块：

```sh
# dumpfs /dev/ada0p2 | grep -i journal
flags   soft-updates+journal
```

这些命令是只读的，不会修改文件系统状态。

### 4.5. 文件系统检查与维护

虽然日志记录显著缩短了崩溃后的恢复时间，但它并不能消除定期完整文件系统检查的需要。

- 日志记录保证一致性，而非正确性
- 媒体错误不会被日志记录修复
- 仍应安排定期的 `fsck`
- 可在活动文件系统上使用后台 `fsck`

## 5. 使用 GEOM 日志记录（高级）

>**警告**
>
>本节适用于理解块级日志记录以及变更后的 sync 语义所带来影响的高级用户。

### 5.1. 概述

此功能通过将 **geom_journal.ko** 模块加载到内核中（或将其构建到自定义内核中）并使用 `gjournal` 命令来配置文件系统。通常，你可能希望对大型文件系统（如 **/usr**）进行日志记录。但你需要（见下一节）预留一些空闲磁盘空间。

GEOM 日志记录由 `geom_journal` 内核模块实现，并使用 `gjournal(8)` 工具进行配置。它在文件系统层之下提供块级日志记录，并需要 UFS 的明确配合。

使用 GEOM 日志记录时，需要一些磁盘空间来存储日志本身。包含文件系统数据的提供者称为 *数据提供者*，而存储日志的提供者称为 *日志提供者*。

对现有（非空）文件系统启用日志记录时，数据提供者和日志提供者必须分开。对新创建的空文件系统启用日志记录时，可以使用单个提供者同时存储数据和日志信息。在这两种情况下，`gjournal(8)` 都会将数据提供者和日志提供者组合起来创建一个新的日志提供者，然后由文件系统挂载。例如：

- **/usr** 文件系统位于 **/dev/ada0p2** 上并已包含数据。
- 已在单独的分区 **/dev/ada0p4** 中分配了空闲磁盘空间用于存放日志。
- 配置 GEOM 日志记录后，将创建新的提供者 **/dev/ada0p2.journal**。此日志提供者将 **/dev/ada0p2** 作为数据提供者、**/dev/ada0p4** 作为日志提供者组合起来，用于所有后续的文件系统操作。

日志所需的磁盘空间主要取决于文件系统的写入工作负载，而非数据提供者的大小。具有持续或突发写入活动的系统需要更大的日志，以避免过多的日志切换或写入节流。

摘自 `gjournal(8)` 手册页：

- 默认日志大小为 1 GB。
- 推荐的最小日志大小为已安装的物理内存容量的两倍。
- 日志大小应根据预期的写入负载选择，而非文件系统大小。

日志过小可能在重度写入负载下导致性能下降或强制日志切换。

有关日志记录的更多信息，请阅读 `gjournal(8)` 的手册页。

### 5.2. FreeBSD 安装过程中的步骤

本节描述了为配置使用 GEOM 日志记录的系统而进行的可选安装时准备。其目标是为稍后与 **/usr** 和 **/var** 文件系统关联的日志提供者预留磁盘空间。

#### 5.2.1. 为日志记录预留空间

在典型的单磁盘系统上，操作系统、安装的软件和用户数据都驻留在同一设备上。FreeBSD 安装程序执行的默认自动分区会将大部分可用空间分配给 **/usr**，并为 **/var** 和其他挂载点分配较小的分区。

默认情况下，安装程序会将所有可用磁盘空间分配给文件系统，不会留下未使用的空间。当计划对现有（非空）文件系统使用 GEOM 日志记录时，必须预留额外的磁盘空间来存放日志提供者。每个要进行日志记录的文件系统都需要自己的日志提供者。

由于 **/usr** 通常占据磁盘的最大部分，因此通常最实际的做法是稍微缩小 **/usr** 分区，以便为日志提供者腾出空间。

在我们的示例中，计划对 **/usr** 和 **/var** 都使用 GEOM 日志记录。

#### 5.2.2. 分区策略

在安装过程中，使用安装程序提供的手动分区模式。为 **/** 创建任何受支持的文件系统类型，并为 **/usr** 和 **/var** 创建标准 UFS 分区，但缩小 **/usr** 的大小，以便在磁盘末尾留下足够的未分配空间。

从剩余的未分配空间中，创建两个额外分区，专门用作日志提供者：

- 一个分区用于 **/usr** 的日志
- 一个分区用于 **/var** 的日志

这些分区不得分配挂载点，也不得用于交换空间。在安装后使用 `gjournal(8)` 将它们与对应的数据提供者显式关联之前，它们将保持未使用状态。

典型布局可能如下所示：

**表 1. 为 GEOM 日志记录预留的分区**

| 提供者 | 预期用途 | 备注 |
| :---: | :---: | :---: |
| /dev/ada0p2 | /var | UFS 文件系统（数据提供者） |
| /dev/ada0p3 | /usr | UFS 文件系统（数据提供者） |
| /dev/ada0p4 | /var 的日志 | 未格式化、未使用的分区 |
| /dev/ada0p5 | /usr 的日志 | 未格式化、未使用的分区 |

确切的设备名称会因磁盘布局和分区方案而异。强烈建议在安装过程中记录提供者名称，因为在配置日志时需要用到它们。

#### 5.2.3. 日志大小注意事项

日志大小取决于预期的写入工作负载，而非文件系统大小。对于通用系统，1-2 GB 的日志大小通常足够。具有持续或突发写入活动的系统可能需要更大的日志。

在安装过程中应保守地分配日志空间，因为稍后调整分区大小可能需要额外的停机时间。

#### 5.2.4. 完成安装

预留所需分区后，按常规方式完成 FreeBSD 安装。建议将第三方软件的安装和额外的系统配置推迟到 GEOM 日志记录完全配置完成之后。

在此阶段，系统将使用标准 UFS 文件系统启动。预留的日志分区将保持未使用状态，直到显式启用日志记录。

#### 5.2.5. 安装后首次启动

首次成功启动后，无需立即进行配置更改。在继续之前，应验证系统能否正常启动和运行。

确认基本系统功能正常后，下一步包括：

- 加载或启用 GEOM 日志记录支持
- 将日志提供者与对应的数据提供者关联
- 更新 **/etc/fstab** 以挂载日志设备

这些步骤将在下一节中描述。

### 5.3. 设置日志记录

#### 5.3.1. 在现有文件系统上启用 GEOM 日志记录

本节描述如何使用安装过程中准备的分区在 **/usr** 和 **/var** 文件系统上启用 GEOM 日志记录。

预留所需的日志提供者后，即可配置 GEOM 日志记录。由于要进行日志记录的文件系统不得挂载，因此系统必须切换到单用户模式。

以 `root` 身份登录并输入：

```sh
# shutdown now
```

按回车键进入 root shell。卸载要进行日志记录的文件系统。在本示例中为 **/usr** 和 **/var**：

```sh
# umount /usr /var
```

加载 GEOM 日志记录内核模块：

```sh
# gjournal load
```

接下来，将每个数据提供者与其对应的日志提供者关联。使用你的记录确定每个日志使用哪个分区。

在本示例中：

- **/usr** 位于 **/dev/ada0p3**，其日志提供者为 **/dev/ada0p5**
- **/var** 位于 **/dev/ada0p2**，其日志提供者为 **/dev/ada0p4**

使用 `gjournal(8)` 创建日志提供者：

```sh
# gjournal label ada0p3 ada0p5
GEOM_JOURNAL: Journal 2948326772: ada0p5 contains journal.
GEOM_JOURNAL: Journal 2948326772: ada0p3 contains data.
GEOM_JOURNAL: Journal ada0p3 clean.

# gjournal label ada0p2 ada0p4
GEOM_JOURNAL: Journal 3193218002: ada0p4 contains journal.
GEOM_JOURNAL: Journal 3193218002: ada0p2 contains data.
GEOM_JOURNAL: Journal ada0p2 clean.
```

>**注意**
>
>如果 `gjournal` 报告某个提供者的最后一个扇区已被使用，命令将会失败。在全新创建或未使用的分区上，使用 `-f` 标志强制初始化以重试操作是安全的：
>
>```sh
># gjournal label -f /dev/ada0p2 /dev/ada0p4
>```

标记完成后，将创建两个新的日志提供者：

- **/dev/ada0p3.journal** 用于 **/usr**
- **/dev/ada0p2.journal** 用于 **/var**

挂载文件系统时，这些设备将替换原始的数据提供者。

但在挂载之前，我们必须在其上设置日志标志，并清除软更新标志：

```sh
# tunefs -J enable -n disable -j disable /dev/ada0p3.journal
tunefs: soft updates journaling cleared but soft updates still set
tunefs: remove .sujournal to reclaim space
tunefs: gjournal set
tunefs: soft updates cleared

# tunefs -J enable -n disable -j disable /dev/ada0p2.journal
tunefs: soft updates journaling cleared but soft updates still set
tunefs: remove .sujournal to reclaim space
tunefs: gjournal set
tunefs: soft updates cleared
```

现在，将新设备手动挂载到各自的位置以验证操作是否正确（注意现在可以使用 `async` 挂载选项），并删除 **.sujournal** 以回收空间：

```sh
# mount -o async /dev/ada0p2.journal /var
# rm /var/.sujournal
# mount -o async /dev/ada0p3.journal /usr
# rm /usr/.sujournal
```

编辑 **/etc/fstab** 并更新 **/usr** 和 **/var** 的条目：

```sh
/dev/ada0p3.journal    /usr    ufs     rw,async      2       2
/dev/ada0p2.journal    /var    ufs     rw,async      2       2
```

>**警告**
>
>请确保上述条目正确无误，否则在重启后你将无法正常启动！

最后，编辑 **/boot/loader.conf** 并添加以下行，以便每次启动时加载 `gjournal(8)` 模块：

```sh
geom_journal_load="YES"
```

恭喜！你的系统现在已设置好日志记录。你可以输入 `exit` 返回多用户模式，或重启以测试你的配置（推荐）。

```sh
# shutdown -r now
```

启动期间你将看到类似如下的消息：

```sh
# dmesg | grep GEOM_JOURNAL
GEOM_JOURNAL: Journal 2948326772: ada0p4 contains journal.
GEOM_JOURNAL: Journal 3193218002: ada0p5 contains journal.
GEOM_JOURNAL: Journal 2948326772: ada0p2 contains data.
GEOM_JOURNAL: Journal ada0p2 clean.
GEOM_JOURNAL: Journal 3193218002: ada0p3 contains data.
GEOM_JOURNAL: Journal ada0p3 clean.
```

非正常关机后，消息会略有不同，例如：

```sh
GEOM_JOURNAL: Journal ada0p2 consistent.
```

这通常意味着 `gjournal(8)` 使用日志提供者中的信息将文件系统恢复到一致状态。

#### 5.3.2. 在新创建的分区上启用 GEOM 日志记录

在已包含数据的文件系统上启用 GEOM 日志记录时，需要执行前面所述的过程。对新创建的空分区启用日志记录时，过程更简单，因为数据提供者和日志提供者可以位于同一分区内。

此方法通常用于尚未格式化为文件系统的新磁盘或新分配的分区。

虽然上述过程对于已包含数据的分区进行日志记录是必要的，但对空分区进行日志记录则相对简单，因为数据提供者和日志提供者可以存储在同一分区中。例如，假设安装了一个新磁盘，并创建了新分区 **/dev/ada1p1**，创建日志的过程就简单多了，你只需要执行：

```sh
# gjournal label /dev/ada1p1
```

默认情况下，日志大小将为 1 GB。你可以使用 `-s` 选项调整它。该值可以以字节为单位给出，或者附加 `K`、`M` 或 `G` 来分别表示千字节、兆字节或吉字节。请注意，`gjournal` 不允许你创建过小的日志。

例如，要创建 2 GB 的日志，你可以使用以下命令：

```sh
# gjournal label -s 2G /dev/ada1p1
```

然后，你可以在新分区上初始化 UFS 文件系统，使用 `newfs(8)` 在 `.journal` 设备上启用日志记录，并使用 `tunefs(8)` 禁用软更新：

```sh
# newfs -J /dev/ada1p1.journal
# tunefs -n disable -j disable /dev/ada1p1.journal
```

文件系统创建完成后，可以正常挂载。例如，将其挂载到 **/data**：

```sh
# mount /dev/ada1p1.journal /data
```

为确保文件系统在启动时自动挂载，请在 **/etc/fstab** 中添加条目：

```sh
/dev/ada1p1.journal    /data   ufs     rw,async      2       2
```

##### 5.3.2.1. 使用注意事项

当使用单个提供者同时存储数据和日志时：

- 标记之前分区必须为空
- 日志空间从分区内分配
- 文件系统大小将减少日志的大小

### 5.4. 将日志记录功能集成到自定义内核中

如果你不希望加载 `geom_journal` 模块，可以将其功能直接构建到内核中。编辑你的自定义内核配置文件，并确保包含以下两行：

```sh
options UFS_GJOURNAL # 此项已包含在 GENERIC 中

options GEOM_JOURNAL # 需要你添加此项
```

根据 [FreeBSD 手册中的相关说明](https://docs.freebsd.org/en/books/handbook/#kernelconfig) 重新构建并重新安装你的内核。

如果你之前使用过此模块，请不要忘记从 **/boot/loader.conf** 中删除相关的 "load" 条目。

### 5.5. 检查 GEOM 日志记录

GEOM 日志记录状态可通过检查活动的 GEOM 提供者来验证。

要列出所有活动的 GEOM 日志设备，请运行：

```sh
# gjournal list
```

或者查看日志状态摘要：

```sh
# gjournal status
```

日志文件系统通过以 `.journal` 结尾的设备名称来标识。这些提供者的存在表明 GEOM 日志记录处于活动状态。

可以使用以下命令获取有关 GEOM 设备堆叠的附加信息：

```sh
# geom -t
```

或者，可以使用 `dumpfs(8)` 检查文件系统超级块：

```sh
# dumpfs /dev/ada0p2.journal | grep -i journal
flags   gjournal
```

## 6. 日志记录故障排除

以下部分涵盖了有关日志记录问题的常见问题解答。

### 6.1. 在磁盘活动高峰期我遇到了内核恐慌。这与 GEOM 日志记录有关吗？

在现代 FreeBSD 系统上，仅因 GEOM 日志过小而导致的内核恐慌很少见。

在持续或突发重度写入负载下，日志过小通常会导致写入节流或在日志刷新到数据提供者时出现临时 I/O 停顿。内核恐慌仅在特殊情况下才会发生，通常与其他问题（如 I/O 错误或资源耗尽）结合出现，此时继续运行可能会危及文件系统完整性。

如果在重度写入活动期间观察到内核恐慌，建议增大受影响文件系统的日志提供者大小。

### 6.2. 我在配置过程中犯了错误，现在无法正常启动。这个问题能解决吗？

你可能忘记（或拼写错误）了 **/boot/loader.conf** 中的条目，或者 **/etc/fstab** 文件中有错误。这些问题通常很容易修复。在系统启动期间，当启动过程停止并提示进入单用户模式时（例如在挂载失败或 `fsck` 错误后），按回车键进入默认的单用户 shell。然后找到问题的根源：

```sh
# cat /boot/loader.conf
```

确保以下行存在且拼写正确：

```sh
geom_journal_load="YES"
```

如果 `geom_journal_load` 条目缺失或拼写错误，则不会创建日志设备。手动加载模块，挂载所有分区，然后继续多用户启动：

```sh
# gjournal load

GEOM_JOURNAL: Journal 2948326772: ada0p4 contains journal.
GEOM_JOURNAL: Journal 3193218002: ada0p5 contains journal.
GEOM_JOURNAL: Journal 2948326772: ada0p2 contains data.
GEOM_JOURNAL: Journal ada0p2 clean.
GEOM_JOURNAL: Journal 3193218002: ada0p3 contains data.
GEOM_JOURNAL: Journal ada0p3 clean.

# mount -a
# exit
(boot continues)
```

如果该条目正确，请检查 **/etc/fstab**。你可能会发现拼写错误或缺失的条目。在这种情况下，手动挂载所有剩余分区并继续多用户启动。

### 6.3. 我可以删除日志记录并恢复到带软更新的标准文件系统吗？

当然可以。使用以下步骤可以撤销之前的更改。你为日志提供者创建的分区可以在之后用于其他目的（如果需要的话）。

以 `root` 身份登录并切换到单用户模式：

```sh
# shutdown now
```

卸载已启用日志记录的分区：

```sh
# umount /usr /var
```

同步日志：

```sh
# gjournal sync
```

停止日志记录提供者：

```sh
# gjournal stop ada0p2.journal
# gjournal stop ada0p3.journal
```

下一步应在卸载 gjournal 内核模块后进行。如果以下命令失败：

```sh
# gjournal unload
```

请确保 **/boot/loader.conf** 中存在以下行且拼写正确：

```sh
geom_journal_load="NO"
```

在不加载 gjournal 内核模块的情况下重启桌面：

```sh
# shutdown -r now
```

再次启动到单用户模式。清除所有使用过的设备上的日志元数据：

```sh
# gjournal clear ada0p2
# gjournal clear ada0p3
# gjournal clear ada0p4
# gjournal clear ada0p5
```

清除文件系统的日志记录标志，并恢复软更新标志：

```sh
# tunefs -J disable -n enable -j enable ada0p2
tunefs: soft updates journaling set
tunefs: gjournal cleared
tunefs: soft updates set

# tunefs -J disable -n enable -j enable ada0p3
tunefs: soft updates journaling set
tunefs: gjournal cleared
tunefs: soft updates set
```

手动重新挂载旧设备：

```sh
# mount -o rw /dev/ada0p2 /var
# mount -o rw /dev/ada0p3 /usr
```

编辑 **/etc/fstab** 并恢复到原始状态：

```sh
/dev/ada0p2     /usr            ufs     rw      2       2
/dev/ada0p3     /var            ufs     rw      2       2
```

最后，编辑 **/boot/loader.conf**，删除加载 `geom_journal` 模块的条目（如果其他分区仍需使用 gjournal，则重新启用它），并重启。

### 6.4. 如何在不使用 GEOM 日志记录的情况下临时启动？

要在不使用 GEOM 日志记录的情况下启动系统，请阻止日志模块在启动期间加载。

从 **/boot/loader.conf** 中注释掉或删除以下行并重启：

```sh
geom_journal_load="YES"
```

在不使用 GEOM 日志记录启动后，将不会创建日志设备（`*.journal`）。必须使用原始（非日志）提供者挂载文件系统，或根据恢复或维护的需要手动挂载。

此方法不会修改磁盘上的日志元数据，可安全用于故障排除。

### 6.5. 创建 GEOM 日志后我可以调整其大小吗？

不可以。GEOM 日志在创建后无法原地调整大小。

日志大小在标记日志时固定。要更改日志大小，必须删除 GEOM 日志记录并使用所需大小的新日志提供者重新配置。对于包含数据的文件系统，这需要卸载文件系统并使用大小合适的提供者重新创建日志，如本文前面所述。

保守地规划日志大小以适应峰值写入活动，因为调整大小需要重新初始化日志配置。

### 6.6. 非正常关机或断电后会发生什么？

非正常关机或断电后，GEOM 日志记录和带日志记录的软更新会在下次启动期间重放待处理的日志记录，将文件系统恢复到一致状态。

如果日志重放成功完成，文件系统将正常挂载，无需完整的文件系统检查。日志重放通常只需要几秒钟，取决于最近的写入活动量，而非文件系统大小。

在极少数情况下可能仍需完整的文件系统检查，例如检测到底层存储错误或日志本身不一致时。

### 6.7. GEOM 日志记录如何与 fsck 交互？

GEOM 日志记录和带日志记录的软更新以不同方式与 `fsck_ffs(8)` 交互，反映了它们在 FreeBSD 存储栈中不同的集成级别。

对于 GEOM 日志记录，崩溃恢复由 GEOM 框架本身执行。在系统启动期间，`geom_journal` 类会检测日志元数据，并在 GEOM 对提供者进行品尝（tasting）时重放任何待处理的日志记录，此过程发生在日志设备对系统更高层可见之前。如果日志重放成功完成，文件系统将恢复到一致状态，并且 UFS 超级块会更新以指示不需要文件系统检查。当 `fsck_ffs(8)` 稍后在启动期间被调用时，它会从超级块读取此状态并跳过文件系统检查。

对于带日志记录的软更新，恢复由 `fsck_ffs(8)` 本身启动。在启动时的文件系统检查阶段，`fsck_ffs(8)` 检测到文件系统支持带日志记录的软更新，并调用日志重放逻辑。如果日志重放报告文件系统一致，`fsck_ffs(8)` 会提前终止，不执行完整检查。如果日志重放无法成功完成，则执行传统的文件系统检查。

因此，系统启动期间的恢复路径取决于所使用的日志记录机制，如下面的启动序列所示：

```sh
固件 / 启动加载器
        |
        v
内核初始化
        |
        v
GEOM 框架（提供者品尝）
        |
        +--> GEOM 日志记录（gjournal）
        |       - 此处执行日志重放
        |       - 恢复文件系统一致性
        |       - 更新超级块
        |
        v
文件系统检查阶段（fsck_ffs）
        |
        +--> 带日志记录的软更新
        |       - 由 fsck 启动日志重放
        |       - 一致则 fsck 提前退出
        |
        v
文件系统挂载
        |
        v
正常系统运行
```

当日志重放无法完成、检测到底层存储错误或进行定期手动或计划文件系统检查时，仍会使用 `fsck_ffs(8)`。GEOM 日志记录缩短了崩溃后的恢复时间，但并不能完全取代 `fsck_ffs(8)`。

### 6.8. 为什么即使在单用户模式下 `gjournal clear` 也会报告 "Operation not permitted"？

在 GEOM 中，相同的底层存储可以同时通过多个提供者名称暴露。例如，一个分区可能同时以基于磁盘的名称（如 `vtbd0p7`）和 GPT 标签（如 **/dev/gpt/labelname**）可见。这些是引用同一物理存储的不同 GEOM 提供者。

当在一个提供者名称上执行 `gjournal stop` 或 `gjournal clear` 时，GEOM 框架可能会在品尝过程中立即通过另一个提供者名称再次检测到日志元数据。结果，日志会自动重新附加，尝试清除其元数据的操作会失败并报 "Operation not permitted" 错误。

技术上可以通过禁用 GEOM 品尝或阻止创建替代提供者名称来抑制此行为，但强烈建议不要这样做。此类更改可能会对依赖正常提供者发现的其他 GEOM 类和系统组件产生负面影响。

因此，推荐且安全的做法是在删除日志元数据之前完全卸载 `geom_journal` 模块，如本文前面所述。模块卸载后，不会发生品尝，日志也无法重新附加，从而允许 `gjournal clear` 成功完成。

### 6.9. GEOM 日志记录可以用在根文件系统上吗？

可以。GEOM 日志记录可以用在根文件系统上，但在系统配置期间需要额外小心。

GEOM 日志模块必须在启动过程的早期加载，以便在挂载根文件系统之前日志根设备可用。配置错误可能导致系统无法正常启动，因此此设置通常仅推荐给完全理解启动序列和恢复过程的经验丰富的用户。

由于这些原因，GEOM 日志记录更常应用于非根文件系统，如 **/usr** 和 **/var**。

### 6.10. 使用 GEOM 日志记录会对性能产生什么影响？

GEOM 日志记录引入了额外的写入 I/O，因为所有写操作都会先记录到日志中，然后才提交到数据提供者。与非日志文件系统相比，这会导致写入延迟增加和整体写入放大更高。

读取性能通常不受影响。性能影响在写入密集型负载下最为明显，并取决于日志大小、放置位置和底层存储的速度。使用足够大的日志和用于日志提供者的快速存储可将性能损失降至最低。

### 6.11. GEOM 日志记录可以与其他 GEOM 类一起使用吗？

可以。GEOM 日志记录可以与其他 GEOM 类堆叠使用。

一起使用时，GEOM 日志记录通常在其他 GEOM 类（如 `gmirror(8)` 或 `graid3(8)`）创建的提供者之上运行。堆叠顺序很重要，应遵循相关 GEOM 类的文档化分层要求。

GEOM 日志记录在堆叠的提供者之间维护文件系统一致性，但它不能取代其他 GEOM 类提供的冗余或错误检测功能。

### 6.12. GEOM 日志记录和软更新可以同时启用吗？

不可以。GEOM 日志记录和软更新不得在同一文件系统上同时启用。

GEOM 日志记录需要对写入顺序和一致性保证的独占控制。如果软更新与 GEOM 日志记录一起启用，文件系统可能会出现不可预测的行为，并且无法保证数据完整性。

使用 GEOM 日志记录时，必须如本文前面所述在受影响的文件系统上禁用软更新。

### 6.13. 软更新与带日志记录的软更新有什么区别？

软更新和带日志记录的软更新都旨在维护文件系统一致性，但它们使用不同的机制，并在系统崩溃后表现出不同的行为。

*软更新* 通过在内存中仔细排序元数据写入来确保一致性，以便只将安全状态写入磁盘。如果发生崩溃，任何尚未写入的元数据更新都会丢失，使文件系统处于一致但可能较旧的状态。但是，资源统计可能暂时不准确，需要完整运行 `fsck_ffs(8)` 来回收未使用的块和 inode。

*带日志记录的软更新* 在 *软更新* 的基础上构建，添加了存储在文件系统内的元数据日志。元数据更改首先记录到日志中，并在非正常关机后的下次启动期间自动重放。这允许文件系统快速恢复到一致状态，通常无需完整运行 `fsck_ffs(8)`。

简而言之，*软更新* 在没有日志的情况下提供一致性，而 *带日志记录的软更新* 以额外的磁盘空间和轻微的性能开销为代价，增加了崩溃后快速且自动的恢复。

## 7. 进一步阅读

日志记录是 FreeBSD 中相对较新的功能，因此它的文档还不十分完善。然而，以下一些额外的参考资料可能会对你有所帮助：

- [FreeBSD 手册中的日志记录章节](https://docs.freebsd.org/en/books/handbook/#geom-gjournal)。
- [FreeBSD-CURRENT 邮件列表中的一篇帖子](https://lists.freebsd.org/pipermail/freebsd-current/2006-June/064043.html)，由 `gjournal(8)` 的开发者 Paweł Jakub Dawidek 提供。
- [FreeBSD questions 邮件列表中的一篇帖子](https://lists.freebsd.org/pipermail/freebsd-questions/2008-April/173501.html)，由 Ivan Voras 提供。
- YouTube 上的 [Journaled Soft-Updates, Dr. Kirk McKusick, BSDCan 2010](https://www.youtube.com/watch?v=_NuhRkiInvA)。
- YouTube 上的 [GEOM - in Infrastructure We Trust, Pawel Jakub Dawidek, AsiaBSDCon 2008](https://www.youtube.com/watch?v=xMpmOezBJZo)。
