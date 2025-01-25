# 在台机上实现 UFS 日志功能

<details open="" data-immersive-translate-walked="94cf6408-7487-4d2f-ba87-b0aba28cf54c"><summary data-immersive-translate-walked="94cf6408-7487-4d2f-ba87-b0aba28cf54c" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

许多制造商和卖家用来区分其产品的名称被声称为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目知道这些商标声明，这些名称将附有 “™” 或 “®” 符号。

</details>

## 概述

日志型文件系统使用日志记录系统中发生的所有事务，并在系统崩溃或断电时保持其完整性。虽然仍然有可能丢失未保存的更改，但日志记录几乎完全消除了因非正常关闭而导致的文件系统损坏的可能性。它还将失败后的文件系统检查所需的时间缩短到最低。尽管 FreeBSD 使用的 UFS 文件系统本身不实现日志记录，但 FreeBSD 7.X 中的新日志类可以用来提供独立于文件系统的日志记录。本文介绍了如何在典型的桌面 PC 场景上实现 UFS 日志记录。

---

## 1. 介绍

尽管专业服务器通常受到良好的保护，免受意外关机的影响，但典型的台式机则受制于停电、意外重置以及其他用户相关事件，可能导致非正常关机。 软更新通常在这种情况下有效地保护文件系统，尽管大多数情况下需要进行漫长的后台检查。在罕见情况下，文件系统损坏达到需要用户干预的程度，数据可能会丢失。

GEOM 提供的新日志功能可以在这种情景中大大帮助，几乎消除了文件系统检查所需的时间，并确保文件系统迅速恢复到一致状态。

本文描述了在典型台式电脑场景（一个硬盘用于操作系统和数据）上实施 UFS 日志记录的步骤。 应在安装 FreeBSD 时遵循。 这些步骤足够简单，不需要与命令行进行过度复杂的交互。

阅读本文后，您将了解：

- 如何在新安装的 FreeBSD 中为日志记录保留空间。
- 如何加载和启用 geom_journal 模块（或在自定义内核中构建支持）。
- 如何将现有文件系统转换为使用日志记录，并在/etc/fstab 中使用哪些选项进行挂载。
- 如何在新的（空）分区实施日志记录。
- 如何排除与日志记录相关的常见问题。

阅读此文章之前，您应该能够：

- 理解基本的 UNIX® 和 FreeBSD 概念。
- 熟悉 FreeBSD 的安装过程和 sysinstall 实用程序。

|     | 这里描述的过程适用于准备一个新安装，其中磁盘上还没有存储实际用户数据。虽然可以修改和扩展此过程以用于已经在生产中的系统，但在这样做之前，您应备份所有重要数据。在低级别操作磁盘和分区可能导致致命错误和数据丢失。 |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## 2. 了解 FreeBSD 中的日志记录

FreeBSD 7.X 中由 GEOM 提供的日志记录不是文件系统特定的（不像例如 Linux® 中的 ext3 文件系统），而是在块级别运行。虽然这意味着它可以应用于不同的文件系统，但对于 FreeBSD 7.0-RELEASE，它只能用于 UFS2。

通过将 geom_journal.ko 模块加载到内核中（或将其构建到自定义内核中），并使用 gjournal 命令来配置文件系统来提供此功能。一般来说，您希望对大型文件系统（如/usr）进行日志记录。但是，您需要（请参阅以下部分）保留一些可用磁盘空间。

当文件系统被日志记录时，需要一些磁盘空间来保存日志本身。容纳实际数据的磁盘空间被称为数据提供者，而容纳日志的提供者被称为日志提供者。数据提供者和日志提供者在对现有（非空）分区进行日志记录时需要位于不同的分区。在对新分区进行日志记录时，您有选择将单个提供者用于数据和日志。在任何情况下， gjournal 命令将合并两个提供者以创建最终的日志记录文件系统。例如：

- 您希望对存储在/dev/ad0s1f 中（已包含数据）的/usr 文件系统进行日志记录。
- 您在/dev/ad0s1g 分区中保留了一些空闲磁盘空间。
- 使用 gjournal ，创建了一个新的/dev/ad0s1f.journal 设备，其中/dev/ad0s1f 是数据提供者，/dev/ad0s1g 是日志提供者。随后所有的文件操作都将使用这个新设备。

您需要为日志提供者预留的磁盘空间量取决于文件系统的使用负载，而不是数据提供者的大小。例如，在典型的办公桌面上，为/usr 文件系统预留 1 GB 的日志提供者就足够了，而处理大量磁盘 I/O 的机器（如视频编辑）可能需要更多。如果在提交之前日志空间耗尽，将会发生内核恐慌。

|     | 此处建议的日志大小，在典型的桌面使用中几乎不太可能引起问题（例如网页浏览、文字处理和媒体文件的播放）。如果您的工作负载包括密集的磁盘活动，请使用以下规则以获得最大可靠性：您的 RAM 大小应适合于日志提供者空间的 30%。例如，如果您的系统有 1GB RAM，请创建大约 3.3GB 的日志提供者。（将 RAM 大小乘以 3.3 即可获得日志的大小）。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

关于日志记录的更多信息，请阅读 gjournal(8)的手册页面。

## 3. FreeBSD 安装过程中的步骤

### 3.1. 为日志保留空间

典型的台式机通常只有一个硬盘，用于存储操作系统和用户数据。可以说，sysinstall 选择的默认分区方案基本上是合适的：台式机不需要一个大的 /var 分区，而 /usr 则被分配了大部分磁盘空间，因为用户数据和许多软件包都安装在它的子目录中。

默认分区方案（通过在 FreeBSD 分区编辑器中按 A 键获得的那个，称为 Disklabel）不会留下任何未分配空间。每个将被记录日志的分区，都需要另一个用于日志的分区。由于 /usr 分区是最大的，略微缩小这个分区，以腾出所需的用于日志记录的空间是有道理的。

在我们的示例中，使用了一个 80 GB 的磁盘。下面的截图显示了安装期间 Disklabel 创建的默认分区：

![disklabel1](https://docs.freebsd.org/images/articles/gjournal-desktop/disklabel1.png)

如果这正是您所需要的，调整以进行日志记录非常容易。只需使用箭头键将高亮显示移动到/usr 分区，然后按 D 键将其删除。

现在，将高亮显示移动到屏幕顶部的磁盘名称，然后按 C 键为/usr 创建一个新分区。该新分区应该比 1 GB（如果您只打算对/usr 进行日志记录）或 2 GB（如果您打算对/usr 和/var 都进行日志记录）小。从弹出窗口中选择创建文件系统，并将/usr 键入为挂载点。

|     | 如果你想日志记录 /var 分区吗？通常来说，在相当大的分区上进行日志记录是有意义的。你可以决定不对 /var 进行日志记录，尽管在典型的桌面上这样做不会造成任何损害。如果文件系统的使用量不大（对于桌面来说是很可能的），你可能希望为其日志分配更少的磁盘空间。<br /><br />在我们的示例中，我们对 /usr 和 /var 都进行了日志记录。当然，你可以根据自己的需要调整程序。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

为了尽可能简单，我们将使用 sysinstall 来创建日志记录所需的分区。然而，在安装过程中，sysinstall 坚持要求为你创建的每个分区指定挂载点。在这一点上，你并没有为将保存日志的分区分配任何挂载点，实际上你甚至不需要它们。这些不是我们将要在某个地方挂载的分区。

为了避免 sysinstall 的这些问题，我们将把日志分区创建为交换空间。交换空间从未挂载过，sysinstall 在创建任意数量的交换分区时没有问题。第一次重启后，需要编辑 /etc/fstab 并删除多余的交换空间条目。

要创建交换空间，请再次使用箭头键将高亮移动到 Disklabel 屏幕顶部，使磁盘名称本身高亮显示。然后按 N，输入所需大小（1024M），并从弹出菜单中选择“交换空间”。对每个要创建的日志重复此操作。在我们的示例中，我们创建了两个分区来提供 /usr 和 /var 的日志。最终结果如下截图所示：

![disklabel2](https://docs.freebsd.org/images/articles/gjournal-desktop/disklabel2.png)

创建分区后，建议您写下分区名称和挂载点，以便在配置阶段轻松参考这些信息。这将有助于减少可能损坏安装的错误。下表显示了我们对示例配置的记录：

表 1. 分区和日志

| 分区   | 挂载点 | 日志   |
| ------ | ------ | ------ |
| ad0s1d | /var   | ad0s1h |
| ad0s1f | /usr   | ad0s1g |

按照正常方式继续安装。然而，我们建议您推迟安装第三方软件（软件包），直到完全设置好日志记录。

### 首次启动

您的系统将正常启动，但您需要编辑 /etc/fstab 并删除为日志创建的额外交换分区。通常，您实际使用的交换分区是带有“b”后缀的那个（即在我们的示例中为 ad0s1b）。删除所有其他交换空间条目并重新启动，以便 FreeBSD 停止使用它们。

当系统再次启动时，我们将准备好配置日志记录。

## 4. 设置日志记录

### 4.1. 执行 gjournal

准备了所有必需的分区后，配置日志记录就变得相当容易了。我们需要切换到单用户模式，因此登录为 root ，然后输入:

```sh
# shutdown now
```

按 Enter 以获取默认值 shell。我们需要卸载将被日志记录的分区，在我们的示例中为/usr 和/var:

```
# umount /usr /var
```

加载所需的日志记录模块:

```sh
# gjournal load
```

现在，使用您的笔记确定每个日志记录将用于哪个分区。在我们的示例中，/usr 是 ad0s1f，其日志将是 ad0s1g，而/var 是 ad0s1d，并将被日志记录到 ad0s1h。需要以下命令:

```sh
# gjournal label ad0s1f ad0s1g
GEOM_JOURNAL: Journal 2948326772: ad0s1f contains data.
GEOM_JOURNAL: Journal 2948326772: ad0s1g contains journal.

# gjournal label ad0s1d ad0s1h
GEOM_JOURNAL: Journal 3193218002: ad0s1d contains data.
GEOM_JOURNAL: Journal 3193218002: ad0s1h contains journal.
```

```sh
# gjournal label -f ad0s1d ad0s1h
```

由于这是一次全新安装，实际覆盖任何内容的可能性非常小。

此时，创建了两个新设备，即 ad0s1d.journal 和 ad0s1f.journal。这些设备代表我们需要挂载的 /var 和 /usr 分区。然而，在挂载之前，我们必须为它们设置 journal 标志并清除 Soft Updates 标志：

```sh
# tunefs -J enable -n disable ad0s1d.journal
tunefs: gjournal set
tunefs: soft updates cleared

# tunefs -J enable -n disable ad0s1f.journal
tunefs: gjournal set
tunefs: soft updates cleared
```

现在，将新设备手动挂载到各自的位置（请注意，我们现在可以使用 `async` 挂载选项）：
```sh
# mount -o async /dev/ad0s1d.journal /var
# mount -o async /dev/ad0s1f.journal /usr
```

编辑 /etc/fstab 并更新 /usr 和 /var 的条目：

```sh
/dev/ad0s1f.journal     /usr            ufs     rw,async      2       2
/dev/ad0s1d.journal     /var            ufs     rw,async      2       2
```

|     | 确保以上条目正确，否则重启后会有启动问题！ |
| --- | ------------------------------------------ |

最后，编辑 /boot/loader.conf 并添加以下行，以便每次启动时加载 gjournal(8) 模块：

```sh
geom_journal_load="YES"
```

恭喜！您的系统现在已设置为日志记录。您可以输入 exit 返回到多用户模式，或者重新启动以测试您的配置（推荐）。在启动过程中，您将看到如下消息：

```sh
ad0: 76293MB XEC XE800JD-00HBC0 08.02D08 at ata0-master SATA150
GEOM_JOURNAL: Journal 2948326772: ad0s1g contains journal.
GEOM_JOURNAL: Journal 3193218002: ad0s1h contains journal.
GEOM_JOURNAL: Journal 3193218002: ad0s1d contains data.
GEOM_JOURNAL: Journal ad0s1d clean.
GEOM_JOURNAL: Journal 2948326772: ad0s1f contains data.
GEOM_JOURNAL: Journal ad0s1f clean.
```

在不干净的关闭之后，消息会略有不同，即：

```sh
GEOM_JOURNAL: Journal ad0s1d consistent.
```

这通常意味着 gjournal(8)使用了日志提供程序中的信息来将文件系统恢复到一致状态。

### 4.2. Journaling Newly Created Partitions

While the above procedure is necessary for journaling partitions that already contain data, journaling an empty partition is somewhat easier, since both the data and the journal provider can be stored in the same partition. For example, assume a new disk was installed, and a new partition /dev/ad1s1d was created. Creating the journal would be as simple as:

```sh
# gjournal label ad1s1d
```

日志大小默认为 1 GB。您可以使用 -s 选项进行调整。 值可以用字节给出，也可以附加 K ， M 或 G ，分别表示千字节，兆字节或千兆字节。 请注意， gjournal 不会允许您创建不适合的小日志大小。

例如，要创建一个 2 GB 的日志，您可以使用以下命令：

```sh
# gjournal label -s 2G ad1s1d
```

然后，您可以在新分区上创建一个文件系统，并使用 -J 选项启用日志记录：

```sh
# newfs -J /dev/ad1s1d.journal
```

### 4.3. 将日志记录构建到您的自定义内核中

如果您不希望将 geom_journal 作为一个模块加载，您可以将其函数直接构建到您的内核中。编辑自定义内核配置文件，并确保它包含以下两行：

```sh
options UFS_GJOURNAL # Note: This is already in GENERIC

options GEOM_JOURNAL # You will have to add this one
```

按照 FreeBSD 手册中的相关说明重新构建和重新安装您的内核。

如果之前使用过，请不要忘记从 /boot/loader.conf 中删除相应的 "load" 条目。

## 5. 故障排除日记

以下部分涵盖了有关日记相关问题的常见问题解答。

### 5.1. 我在高磁盘活动期间遇到内核崩溃。这与日记有什么关系？

日志可能会在提交（刷新）到磁盘之前填满。请记住，日志的大小取决于使用负载，而不是数据提供者的大小。如果您的磁盘活动很高，您需要一个更大的分区用于日志。请参见 FreeBSD 部分中关于理解日志的注记。

### 5.2.在配置过程中犯了一些错误，现在无法正常启动。有办法修复吗？

您可能忘记了（或拼错了）/boot/loader.conf 文件中的条目，或者您的/etc/fstab 文件中存在错误。这些问题通常很容易解决。按 Enter 键进入默认的单用户 shell。然后找到问题的根源：

```sh
# cat /boot/loader.conf
```

如果 geom_journal_load 条目丢失或拼错，日志设备将永远不会被创建。手动加载模块，挂载所有分区，然后继续进行多用户引导:

```sh
# gjournal load

GEOM_JOURNAL: Journal 2948326772: ad0s1g contains journal.
GEOM_JOURNAL: Journal 3193218002: ad0s1h contains journal.
GEOM_JOURNAL: Journal 3193218002: ad0s1d contains data.
GEOM_JOURNAL: Journal ad0s1d clean.
GEOM_JOURNAL: Journal 2948326772: ad0s1f contains data.
GEOM_JOURNAL: Journal ad0s1f clean.

# mount -a
# exit
(boot continues)
```

另一方面，如果此条目正确，请查看 /etc/fstab。您可能会发现拼写错误或丢失的条目。在这种情况下，请手动挂载所有其余分区，然后继续进行多用户引导。

### 5.3.我可以删除日志记录并返回到具有软更新的标准文件系统吗？

当然。使用以下过程，它会撤销更改。您为日志提供者创建的分区随后可用于其他目的，如果您愿意的话。

以 root 登录并切换到单用户模式：

```
# shutdown now
```

卸载日志分区：

```sh
# umount /usr /var
```

同步日志：

```sh
# gjournal sync
```

停止日志提供商：

```sh
# gjournal stop ad0s1d.journal
# gjournal stop ad0s1f.journal
```

清除所有设备使用的日志元数据：

```sh
# gjournal clear ad0s1d
# gjournal clear ad0s1f
# gjournal clear ad0s1g
# gjournal clear ad0s1h
```

清除文件系统日志标志，并恢复软更新标志：

```sh
# tunefs -J disable -n enable ad0s1d
tunefs: gjournal cleared
tunefs: soft updates set

# tunefs -J disable -n enable ad0s1f
tunefs: gjournal cleared
tunefs: soft updates set
```

手动重新挂载旧设备：

```sh
# mount -o rw /dev/ad0s1d /var
# mount -o rw /dev/ad0s1f /usr
```

编辑 /etc/fstab 并将其恢复到原始状态：

```sh
/dev/ad0s1f     /usr            ufs     rw      2       2
/dev/ad0s1d     /var            ufs     rw      2       2
```

最后，编辑 /boot/loader.conf，移除加载 geom_journal 模块的条目并重启。

## 6. 进一步阅读

日志记录是 FreeBSD 的一个相当新的功能，因此，它尚未得到很好的文档记录。不过，你可能会发现以下附加参考资料很有用：

- 自由 BSD 手册现在有一个关于日记的新章节。
- 自由 BSD-CURRENT 邮件列表中，gjournal(8)的开发者发布了这篇帖子。
- 在自由 BSD 常见问题邮件列表中， Ivan Voras <<a href="mailto:ivoras@FreeBSD.org">ivoras@FreeBSD.org</a>> 发布了这篇帖子。
- gjournal(8)和 geom(8)的手册页面。
