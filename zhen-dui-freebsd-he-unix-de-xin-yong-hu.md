# 面向 FreeBSD 和 UNIX® 的新用户的简介

- 原文：[For People New to Both FreeBSD and UNIX®](https://docs.freebsd.org/en/articles/new-users/)

## 摘要

恭喜你安装了 FreeBSD！本简介适合那些对 FreeBSD *以及* UNIX® 都不熟悉的用户，因此它从基础开始。


## 1. 登录与退出

当你看到 `login:` 提示符时，作为在安装过程中创建的用户或 `root` 登录。（你的 FreeBSD 安装中已经存在 `root` 账户，`root` 可以访问任何地方并执行任何操作，包括删除重要文件，所以请小心！）以下的 % 和 # 表示提示符（你的提示符可能不同），% 表示普通用户，# 表示 `root`。

要退出（并获得新的 `login:` 提示符），输入：

```sh
# exit
```

根据需要多次输入。是的，输入命令后按 <kbd>enter</kbd>，并记住 UNIX® 是区分大小写的——`exit`，而不是 `EXIT`。

要关闭计算机，输入：

```sh
# /sbin/shutdown -h now
```

要重启计算机，输入：

```sh
# /sbin/shutdown -r now
```

或者

```sh
# /sbin/reboot
```

你也可以通过 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Delete</kbd> 重启计算机。稍等片刻，它就会完成工作。这等同于最近版本的 FreeBSD 中的 `/sbin/reboot`，并且比按重置按钮要好得多。你可不希望重新安装 FreeBSD，对吧？

## 2. 添加具有 Root 权限的用户

如果在安装系统时未创建任何用户，而你目前是以 `root` 身份登录，那么现在应该创建用户，可以使用以下命令：

```sh
# adduser
```

第一次使用 `adduser` 时，它可能会要求保存一些默认值。如果它建议默认使用 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 作为默认的 shell，你可能想要改为 [csh(1)](https://man.freebsd.org/cgi/man.cgi?query=csh&sektion=1&format=html)。否则只需按 Enter 键接受每个默认值。这些默认值会保存在 **/etc/adduser.conf** 文件中，这是可编辑的文件。

假设你创建了名为 `jack` 的用户，完整名称为 *Jack Benimble*。如果安全性是问题（例如周围有可能乱敲键盘的孩子），可以为 `jack` 设置密码。当它询问是否邀请 `jack` 加入其他组时，输入 `wheel`：

```sh
Login group is "jack". Invite jack into other groups: wheel
```

这将使你能够以 `jack` 身份登录并使用 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 命令切换到 `root`。这样，你就不必再因为直接以 `root` 身份登录而被责骂了。

你可以随时按 <kbd>Ctrl</kbd>+<kbd>C</kbd> 退出 `adduser`，结束时会有机会确认新用户，或者直接输入 <kbd>n</kbd> 来取消。你可能还想创建第二个用户，这样在编辑 `jack` 的登录文件时，如果出现问题，至少有备用账户可以使用。

完成后，使用 `exit` 返回登录提示符，并以 `jack` 用户身份登录。通常，尽可能多地以普通用户身份进行工作是个好主意，普通用户没有 `root` 所带来的权力和风险。

如果你已经创建了用户，并希望该用户能够通过 `su` 切换为 `root`，可以以 `root` 用户身份登录并编辑 **/etc/group** 文件，将 `jack` 添加到第一行（即 `wheel` 组）。但首先，你需要练习 [vi(1)](https://man.freebsd.org/cgi/man.cgi?query=vi&sektion=1&format=html) 编辑器——或者使用 FreeBSD 新版本中安装的更简单的文本编辑器 [ee(1)](https://man.freebsd.org/cgi/man.cgi?query=ee&sektion=1&format=html)。

要删除用户，可以使用 `rmuser` 命令。

## 3. 四处看看

作为普通用户登录后，四处看看并尝试一些命令，这些命令将帮助你获取 FreeBSD 系统中的帮助和信息。

以下是一些命令及其作用：

`id`

告诉你当前是谁！

`pwd`

显示你所在的目录，即当前工作目录。

`ls`

列出当前目录中的文件。

`ls -F`

列出当前目录中的文件，对于可执行文件在后面加上 `*`，对于目录加上 `/`，对于符号链接加上 `@`。

`ls -l`

以长格式列出文件——显示文件大小、日期和权限。

`ls -a`

列出包括隐藏的“点”文件在内的所有文件。如果你是 `root` 用户，隐藏的“点”文件在没有 `-a` 参数时就会显示。

`cd`

更改目录。`cd ..` 返回上一层目录；注意 `cd` 后面的空格。`cd /usr/local` 会进入该目录。`cd ~` 会进入当前登录用户的主目录，例如 **/usr/home/jack**。尝试 `cd /cdrom`，然后使用 `ls` 命令，查看你的 CDROM 是否已挂载并正常工作。

`less filename`

让你查看文件（名为 *filename*），而不更改文件内容。尝试 `less /etc/fstab`。输入 `q` 退出。

`cat filename`

显示 *filename* 文件的内容。如果文件过长，你只能看到文件的最后部分，可以按 <kbd>ScrollLock</kbd> 并使用 <kbd>上箭头</kbd> 向上滚动；你也可以在查看手册页时使用 <kbd>ScrollLock</kbd>。再次按 <kbd>ScrollLock</kbd> 退出滚动。你可能想尝试在家目录中的某些点文件上使用 `cat` 命令——例如 `cat .cshrc`、`cat .login`、`cat .profile`。

你会注意到 `.cshrc` 中有些 `ls` 命令的别名（这些别名非常方便）。你可以通过编辑 `.cshrc` 来创建其他别名。如果你想让这些别名对系统中的所有用户可用，可以将它们添加到系统范围的 `csh` 配置文件 **/etc/csh.cshrc** 中。

## 4. 获取帮助和信息

以下是一些有用的帮助来源。*Text* 表示你输入的内容，通常是命令或文件名。

`apropos text`

在 `whatis` 数据库中查找包含 *text* 字符串的所有内容。

`man text`

查看 *text* 的手册页。UNIX® 系统文档的主要来源。比如，`man ls` 会告诉你如何使用 `ls` 命令。按 <kbd>Enter</kbd> 键浏览文本，按 <kbd>Ctrl</kbd>+<kbd>B</kbd> 返回上一页，按 <kbd>Ctrl</kbd>+<kbd>F</kbd> 前进一页，按 <kbd>q</kbd> 或 <kbd>Ctrl</kbd>+<kbd>C</kbd> 退出。

`which text`

告诉你命令 *text* 在用户路径中的位置。

`locate text`

查找所有包含字符串 *text* 的路径。

`whatis text`

告诉你命令 *text* 的功能及其手册页。输入 `whatis *` 可以查看当前目录下所有二进制文件的信息。

`whereis text`

找到文件 *text*，并给出它的完整路径。

你可能想尝试使用 `whatis` 查找一些常见有用的命令，如 `cat`、`more`、`grep`、`mv`、`find`、`tar`、`chmod`、`chown`、`date` 和 `script`。`more` 命令让你逐页阅读文件，就像在 DOS 中一样，例如 `ls -l | more` 或 `more filename`。`*` 作为通配符使用，例如 `ls w*` 会显示以 `w` 开头的文件。

这些命令中有一些工作不太正常吗？`locate(1)` 和 `whatis(1)` 都依赖于每周重建的数据库。如果你的机器在周末不会长时间开启（并运行 FreeBSD），你可能需要定期运行每日、每周和每月维护命令。以 `root` 身份运行这些命令，并且每次运行时给每个命令足够的时间完成，再开始下一个。

```sh
# periodic daily
输出省略
# periodic weekly
输出省略
# periodic monthly
输出省略
```

如果你等得不耐烦，可以按 <kbd>Alt</kbd>+<kbd>F2</kbd> 获取另一个 *虚拟控制台* 并重新登录。毕竟，这是多用户、多任务的系统。尽管如此，这些命令运行时可能会在屏幕上显示消息；你可以在提示符下输入 `clear` 清除屏幕。它们运行完成后，你可能想查看 **/var/mail/root** 和 **/var/log/messages**。

运行这些命令是系统管理的一部分——作为 UNIX® 系统的唯一用户，你就是自己的系统管理员。几乎所有需要 `root` 权限的操作都属于系统管理工作。即使是那些厚厚的 UNIX® 书籍，也往往没有很好地涵盖这些内容，因为它们似乎更多地关注于窗口管理器中的菜单操作。你可能想参考两本系统管理的经典书籍，分别是 Evi Nemeth 等人编写的《UNIX 系统管理手册》（Prentice-Hall，1995年，ISBN 0-13-15051-7，第二版，红色封面）或 Æleen Frisch 的《Essential System Administration》（O’Reilly & Associates，2002年，ISBN 0-596-00343-9）。我用的是 Nemeth 的书。

## 5. 编辑文本

为了配置你的系统，你需要编辑文本文件。大多数文件都会在 **/etc** 目录下，你需要通过 `su` 切换到 `root` 用户才能修改它们。你可以使用简单的 `ee` 编辑器，但从长远来看，学习文本编辑器 `vi` 是值得的。如果你安装了系统源代码，可以在 **/usr/src/contrib/nvi/docs/tutorial** 找到 `vi` 的优秀教程。

在编辑文件之前，你可能应该备份它。假设你要编辑 **/etc/rc.conf** 文件，你可以先通过 `cd /etc` 进入 **/etc** 目录，然后执行：

```sh
# cp rc.conf rc.conf.orig
```

这样会将 `rc.conf` 复制为 `rc.conf.orig`，以后你可以将 `rc.conf.orig` 复制回 `rc.conf` 来恢复原始文件。但更好的方法是先移动（重命名）文件，然后再复制回来：

```sh
# mv rc.conf rc.conf.orig
# cp rc.conf.orig rc.conf
```

因为 `mv` 命令会保留文件的原始日期和所有者。现在你可以编辑 `rc.conf` 文件。如果想恢复原文件，可以将修改后的文件命名为 `rc.conf.myedit`（假设你想保留修改版本），然后执行：

```sh
# mv rc.conf rc.conf.myedit
# mv rc.conf.orig rc.conf
```

这样就能将文件恢复到原来的状态。

要编辑文件，输入：

```sh
# vi filename
```

通过方向键移动光标。按下 <kbd>Esc</kbd> 键（即 escape 键）进入 `vi` 的命令模式。以下是一些常用命令：

`x`

删除光标所在的字符

`dd`

删除整行（即使它在屏幕上换行）

`i`

在光标位置插入文本

`a`

在光标后插入文本

你按下 `i` 或 `a` 后，就可以开始输入文本。按下 <kbd>Esc</kbd> 键返回命令模式，然后你可以输入：

`:w`

保存修改并继续编辑

`:wq`

保存并退出

`:q!`

不保存修改退出

`/*text*`

将光标移动到指定的 *text* 处；按 `/` <kbd>Enter</kbd>（回车键）可以找到下一个出现的 *text*。

`G`

跳转到文件末尾

`nG`

跳转到文件中的第 *n* 行，`n` 是数字

<kbd>Ctrl</kbd>+<kbd>L</kbd>

重新绘制屏幕

<kbd>Ctrl</kbd>+<kbd>b</kbd> 和 <kbd>Ctrl</kbd>+<kbd>f</kbd>

像 `more` 和 `view` 一样翻页，前进和后退。

你可以在你的家目录里练习使用 `vi`，通过 `vi filename` 创建新文件，添加和删除文本，保存文件，并再次打开它。`vi` 有一些让人吃惊的地方，因为它其实相当复杂，有时你会无意中执行命令，导致一些你没有预料到的结果。（有些人实际上喜欢 `vi`——它比 DOS 的编辑器更强大——你可以了解一下 `:r` 命令。）确保经常按 <kbd>Esc</kbd> 键回到命令模式，遇到问题时从命令模式开始，频繁保存文件（使用 `:w`），当需要重新开始时使用 `:q!` 放弃修改（从你上次的 `:w` 开始）。

现在你可以 `cd` 到 **/etc** 目录，`su` 切换到 `root`，使用 `vi` 编辑 **/etc/group** 文件，并将用户添加到 `wheel` 组中，从而赋予该用户 `root` 权限。只需在文件的第一行末尾添加逗号和用户的登录名，按 <kbd>Esc</kbd> 键，然后使用 `:wq` 保存文件并退出。立刻生效。（你没有在逗号后加空格吧？）

## 6. 其他有用的命令

`df`

显示文件空间和挂载的系统。

`ps aux`

显示正在运行的进程。`ps ax` 是更简洁的形式。

`rm filename`

删除 *filename* 文件。

`rm -R dir`

删除目录 *dir* 及其所有子目录——小心使用！

`ls -R`

列出当前目录及所有子目录中的文件；我使用过变体 `ls -AFR > where.txt`，它可以列出 **/** 和（分别）**/usr** 中的所有文件，直到我找到更好的方法来查找文件。

`passwd`

更改用户的密码（或 `root` 的密码）。

`man hier`

查看 UNIX® 文件系统的手册页。

使用 `find` 在 **/usr** 或其任何子目录中查找文件名：

```sh
% find /usr -name "filename"
```

你可以在 `"filename"` 中使用 `*` 作为通配符（应该加引号）。如果你让 `find` 从 **/** 目录开始搜索，它会在所有挂载的文件系统中查找文件，包括 CDROM 和 DOS 分区。

一本很好的书，讲解了 UNIX® 命令和工具，是 Abrahams & Larson 的《Unix for the Impatient》（第二版，Addison-Wesley，1996）。互联网上也有大量的 UNIX® 信息。

## 7. 下一步

现在你应该拥有了必要的工具来浏览和编辑文件，能够让系统顺利运行。FreeBSD 手册中有大量的信息（它可能已经存储在你的硬盘上），还有 [FreeBSD 的官方网站](https://www.FreeBSD.org/)。CDROM 和网站上也有各种各样的包和 Ports。手册会告诉你如何使用它们（如果存在包，可以使用 `pkg add packagename` 安装，其中 *packagename* 是包的文件名）。CDROM 中有包和 Ports 的列表，简短的描述位于 **cdrom/packages/index**、**cdrom/packages/index.txt** 和 **cdrom/ports/index** 中，完整的描述位于 **/cdrom/ports/*/*/pkg/DESCR**，其中 `*` 代表程序种类和程序名称的子目录。

如果你发现手册中关于从 CDROM 安装 Ports 的内容过于复杂（涉及 `lndir` 等），通常可以按以下方法进行：

找到你需要的 Port，比如 `kermit`。它在 CDROM 上有个对应的目录。将该子目录复制到 **/usr/local**（这是添加的软件存放位置，所有用户都可以访问）：

```sh
# cp -R /cdrom/ports/comm/kermit /usr/local
```

这样就会在 **/usr/local/kermit** 子目录下生成与 CDROM 上 `kermit` 子目录相同的子目录。

接下来，使用 `mkdir` 创建目录 **/usr/ports/distfiles**（如果它尚不存在）。然后在 **/cdrom/ports/distfiles** 中查找文件名表明它是你所需 Port 的文件。将该文件复制到 **/usr/ports/distfiles** 目录；在最新版本中，你可以跳过这一步，因为 FreeBSD 会自动为你完成。如果是 `kermit`，则没有 distfile 文件。

然后 `cd` 到 **/usr/local/kermit** 子目录下，进入包含 `Makefile` 的目录。输入：

```sh
# make all install
```

在此过程中，Port 将通过 FTP 下载它所需的、在 CDROM 或 **/usr/ports/distfiles** 中未找到的压缩文件。如果你还没有启动网络，并且 **/cdrom/ports/distfiles** 中没有该 Port 的文件，你需要使用另一台机器下载该文件并将其复制到 **/usr/ports/distfiles**。你可以通过 `cat`、`more` 或 `view` 阅读 `Makefile`，了解该文件的主分发站点和文件名。（使用二进制文件传输！）然后返回到 **/usr/local/kermit**，找到包含 `Makefile` 的目录，再次执行 `make all install`。

## 8. 你的工作环境

你的 shell 是工作环境中最重要的部分。shell 解释你在命令行输入的命令，从而与操作系统的其他部分进行通信。你还可以编写 shell 脚本——一系列无需干预即可运行的命令。

FreeBSD 默认安装了两个 shell：`csh` 和 `sh`。`csh` 适合命令行操作，但脚本应该使用 `sh`（或 `bash`）编写。你可以通过输入 `echo $SHELL` 来查看你正在使用的 shell。

`csh` shell 还可以，但 `tcsh` 做了 `csh` 所做的所有事情，并且做得更多。它能让你用箭头键回顾命令并编辑它们。它还支持通过 tab 键完成文件名（而 `csh` 使用 <kbd>Esc</kbd> 键），并且可以使用 `cd -` 切换到你最后访问的目录。你还可以更轻松地修改提示符。`tcsh` 让生活更轻松。

安装新 shell 的三个步骤：

1. 将该 shell 作为 Port 或包安装，就像安装其他任何 Port 或包一样。
2. 使用 `chsh` 命令永久更改你的 shell 为 `tcsh`，或者在提示符下输入 `tcsh` 以便在不重新登录的情况下临时更改 shell。

>**注意**
>
>更改 `root` 的 shell 为 `sh` 或 `csh` 以外的其他 shell 在早期版本的 FreeBSD 和许多其他 UNIX® 版本中可能是危险的；当系统将你置于单用户模式时，可能无法使用有效的 shell。解决方案是使用 `su -m` 成为 `root`，这样你就可以作为 `root` 使用 `tcsh`，因为 shell 是环境的一部分。你可以通过将以下内容添加到你的 `.tcshrc` 文件中作为别名来使其永久生效：
>
>```sh
>alias su su -m
>```

当 `tcsh` 启动时，它将读取 **/etc/csh.cshrc** 和 **/etc/csh.login** 文件，就像 `csh` 一样。它还会读取你家目录中的 `.login` 和 `.cshrc` 文件，除非你提供了 `.tcshrc` 文件。你可以通过简单地将 `.cshrc` 复制为 `.tcshrc` 来实现这一点。

现在你已经安装了 `tcsh`，你可以调整你的提示符。你可以在 `tcsh` 的手册页中找到详细信息，但这里有一行代码可以放入你的 `.tcshrc` 中，它将告诉你输入了多少条命令、当前时间和你所在的目录。如果你是普通用户，它还会显示 `>`，如果你是 `root` 用户，它会显示 `#`，但无论如何 `tcsh` 都会这么做：

```sh
set prompt = "%h %t %~ %# "
```

如果已有 `set prompt` 行，这行代码应该放在同一位置，或者如果没有，就放在 `if($?prompt) then` 语句下。注释掉旧的行；如果你更喜欢它，你随时可以切换回来。不要忘记空格和引号。你可以通过输入 `source .tcshrc` 来重新读取 `.tcshrc`。

你可以通过在提示符下输入 `env` 来列出已设置的其他环境变量。结果将显示你的默认编辑器、分页程序和终端类型等，可能还有很多其他信息。如果你从远程位置登录，并且由于终端不支持，无法运行某个程序，可以使用 `setenv TERM vt100` 命令来设置终端类型。

## 9. 其他

作为 `root` 用户，你可以使用 `/sbin/umount /cdrom` 来卸载 CDROM，取出光盘，插入另一张，然后使用 `/sbin/mount_cd9660 /dev/cd0a /cdrom` 来挂载它，假设 `cd0a` 是你的 CDROM 驱动器的设备名称。最新版本的 FreeBSD 允许你仅使用 `/sbin/mount /cdrom` 来挂载 CDROM。

如果你的空间有限，使用 FreeBSD 的 CDROM 上的实时文件系统（第二张光盘）是很有用的。实时文件系统的内容因版本而异。你可以尝试从 CDROM 上玩游戏。这涉及使用 `lndir`，它与 X Window 系统一起安装，用来告诉程序在哪里找到必要的文件，因为它们在 **/cdrom** 中，而不是在预期的 **/usr** 和其子目录中。阅读 `man lndir`。

## 10. 欢迎提出意见

如果你使用了本指南，我很想知道哪些地方不清楚，以及你认为应该包括哪些被遗漏的内容，或者它是否有帮助。感谢 Eugene W. Stark（纽约州立大学石溪分校计算机科学教授）和 John Fieber 提供的有益意见。

Annelise Anderson, [andrsn@andrsn.stanford.edu](mailto:andrsn@andrsn.stanford.edu)
