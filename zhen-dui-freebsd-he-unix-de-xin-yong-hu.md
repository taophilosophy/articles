# 针对 FreeBSD 和 UNIX® 的新用户的简介

# 对于对 FreeBSD 和 UNIX®都是新手的人

<details open="" data-immersive-translate-walked="ca13c054-15b3-4ac4-955c-43510d8c7b2b"><summary data-immersive-translate-walked="ca13c054-15b3-4ac4-955c-43510d8c7b2b" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

IBM、AIX、OS/2、PowerPC、PS/2、S/390 和 ThinkPad 是国际商业机器公司在美国、其他国家或两者的商标。

Microsoft、IntelliMouse、MS-DOS、Outlook、Windows、Windows Media 和 Windows NT 是 Microsoft Corporation 在美国和/或其他国家的注册商标或商标。

Motif、OSF/1 和 UNIX 是 The Open Group 在美国和其他国家的注册商标，IT DialTone 和 The Open Group 是 The Open Group 的商标。

制造商和销售商用来区分其产品的许多名称都被视为商标。在本文档中出现这些名称时，如果 FreeBSD 项目意识到商标声明，则这些名称后面会跟着“™”或“®”符号。

</details>

 摘要

恭喜你安装了 FreeBSD！这份介绍是为 FreeBSD 和 UNIX® 新手准备的，因此从基础开始讲起。

---

## 1. 登录和退出

当您看到 login: 时登录为您在安装过程中创建的用户或 root 。（您的 FreeBSD 安装将已经为 root 创建了一个账户；谁可以去任何地方并做任何事情，包括删除重要文件，所以要小心！）以下符号中的%和#分别表示提示符（您可能不同），%表示普通用户，#表示 root 。

要注销（并获得新的 login: 提示符）请输入

```
# exit
```

根据需要频繁执行。是的，在命令后按回车，并记住 UNIX®区分大小写- exit ，而不是 EXIT 。

关闭机器请输入

```
# /sbin/shutdown -h now
```

或者重启请输入

```
# /sbin/shutdown -r now
```

 或

```
# /sbin/reboot
```

你也可以使用 Ctrl+Alt+Delete 重新启动。给它一点时间完成工作。这相当于 FreeBSD 最近版本中的 /sbin/reboot ，比按下重置按钮要好得多。你不想重新安装这个东西，对吗？

## 2. 添加具有 Root 权限的用户

如果你在安装系统时没有创建任何用户，因此以 root 身份登录，现在可能需要创建一个用户，使用以下命令:

```
# adduser
```

第一次使用 adduser 时，它可能会要求保存一些默认设置。如果它建议 sh 作为默认设置，您可能希望将默认 shell 更改为 csh(1)而不是 sh(1)。否则，只需按 Enter 键接受每个默认设置。这些默认设置将保存在可编辑的/etc/adduser.conf 文件中。

假设您创建一个用户名为 jack 、全名为 Jack Benimble 的用户。如果安全性（甚至周围可能会乱按键盘的孩子）是个问题，为 jack 设定一个密码。当它询问您是否要邀请 jack 加入其他组时，输入 wheel 。

```
Login group is "jack". Invite jack into other groups: wheel
```

这将使您能够以 jack 的身份登录，并使用 su(1)命令成为 root 。然后，您将不会再因以 root 的身份登录而被责备。

您可以随时通过键入 Ctrl+C 来退出 adduser ，并在结束时您将有机会批准您的新用户，或者简单地输入 n 来拒绝。您可能希望创建第二个新用户，这样当您编辑`jack 的登录文件时，如果出现问题，您将有一个热备份。

完成此操作后，请使用 exit 返回到登录提示符，以 jack 登录。通常，尽可能以普通用户的身份进行尽可能多的工作是一个好主意，这样你就不会有 root 的权限和风险。

如果您已经创建了一个用户，并且希望用户能够 su 到 root ，您可以登录为 root ，并编辑文件/etc/group，在第一行（组 wheel ）添加 jack 。但首先，您需要练习 vi(1)文本编辑器，或者使用最新版本的 FreeBSD 上已安装的更简单的文本编辑器 ee(1)。

要删除用户，请使用 rmuser 。

## 3. 四处看看

作为普通用户登录后，四处看看并尝试一些命令，这些命令将访问 FreeBSD 中的帮助和信息来源。

这里是一些命令及其作用：

id 告诉你当前用户是谁！

pwd 显示当前工作目录在哪里。

ls 列出当前目录中的文件。

ls -F 列出当前目录中的文件，可执行文件后面加上 *，目录后面加上 / ，符号链接后面加上 @ 。

ls -l 以长格式列出文件-大小、日期、权限。

ls -a 列出其他所有文件夹中的隐藏“点”文件。如果您是 root ，则“点”文件将显示而无需 -a 开关。

cd 切换目录。 cd .. 后退一级；请注意 cd 后面的空格。 cd /usr/local 转到那里。 cd ~ 转到已登录用户的主目录，例如，/usr/home/jack。尝试 cd /cdrom ，然后 ls ，以查看 CDROM 是否已挂载并正常工作。

less<span> </span><em>filename</em> 允许您查看文件（名为文件名）而不会更改它。尝试 less /etc/fstab 。键入 q 退出。

cat<span> </span><em>filename</em> 在屏幕上显示文件名。如果文件名过长只能看到末尾部分，按下 ScrollLock 键，使用向上箭头向后移动；你也可以在手册页上使用 ScrollLock。再次按下 ScrollLock 键退出滚动。你可以尝试在你的主目录中的一些点文件上执行 cat ，如 cat .cshrc ， cat .login ， cat .profile 。

在 .cshrc 中可以看到一些 ls 命令的别名（它们非常方便）。你可以通过编辑 .cshrc 创建其他别名。将这些别名放在系统范围的 csh 配置文件 /etc/csh.cshrc 中，可以使它们对系统上的所有用户都可用。

## 4. 获取帮助和信息

这里有一些有用的帮助来源。文本通常指您键入的某些内容，例如命令或文件名。

apropos<span> </span><em>text</em> 包含字符串文本的所有内容 whatis database 。

man<span> </span><em>text</em> 文本的手册页面。这是 UNIX®系统的主要文档来源。 man ls 将告诉您如何使用 ls 的所有方法。按 Enter 键向前移动文本，按 Ctrl+B 返回上一页，按 Ctrl+F 前进，按 q 或 Ctrl+C 退出。

which<span> </span><em>text</em> 告诉您命令文本在用户路径中的位置。

locate<span> </span><em>text</em> 文本字符串被发现的所有路径。

whatis<span> </span><em>text</em> 告诉您命令文本的作用及其手册页。键入 whatis * 将告诉您当前目录中的所有二进制文件。

whereis<span> </span><em>text</em> 查找文件文本，提供完整路径。

你可能想尝试在一些常用实用命令上使用 whatis ，比如 cat ， more ， grep ， mv ， find ， tar ， chmod ， chown ， date ，和 script 。 more 让你像在 DOS 中那样每次读取一页，例如， ls -l | more 或者 more<span> </span><em>filename</em> 。* 作为通配符使用，例如， ls w* 将显示以 w 开头的文件。

有一些不太有效的吗？locate(1) 和 whatis(1) 都依赖于每周重建一次的数据库。如果您的计算机不会在周末开着（并且运行的是 FreeBSD），您可能会想立即运行日常、每周和每月维护的命令。运行它们时要作为 root 运行，并且现在，要给每个命令足够的时间让其完成，然后再开始下一个命令。

```
# periodic daily
output omitted
# periodic weekly
output omitted
# periodic monthly
output omitted
```

如果你厌倦了等待，按下 Alt+F2 键获取另一个虚拟控制台，然后重新登录。毕竟，这是一个多用户、多任务系统。不过，这些命令在运行时可能会在屏幕上闪烁消息；你可以在提示符处输入 clear 来清除屏幕。一旦它们运行完毕，你可能想查看/var/mail/root 和/var/log/messages。

运行这样的命令是系统管理的一部分-作为 UNIX®系统的单个用户，您就是自己的系统管理员。几乎您需要做的任何事情都是系统管理。这些责任甚至在那些大部头的 UNIX®书籍中也没有很好地涵盖，这些书籍似乎在拉下窗口管理器中的菜单上花费了很多篇幅。您可能需要获取两本主要的系统管理书籍之一，即 Evi Nemeth 等人的《UNIX 系统管理手册》（Prentice-Hall，1995 年，ISBN 0-13-15051-7）-第二版为红色封面；或Æleen Frisch 的《基本系统管理》（O’Reilly＆Associates，2002 年，ISBN 0-596-00343-9）。我使用了 Nemeth。

## 5. 编辑文本

要配置系统，您需要编辑文本文件。其中大部分将在 /etc 目录中；而且您需要从 su 到 root 以便能够修改它们。您可以使用简单的 ee ，但从长远来看，值得学习文本编辑器 vi 。如果您安装了系统源代码，/usr/src/contrib/nvi/docs/tutorial 中有一份优秀的 vi 教程。

在编辑文件之前，您应该先备份它。假设您想编辑 /etc/rc.conf。您可以使用 cd /etc 到达 /etc 目录并执行以下操作：

```
# cp rc.conf rc.conf.orig
```

这将复制 rc.conf 到 rc.conf.orig，稍后您可以将 rc.conf.orig 复制回 rc.conf 以恢复原始内容。但更好的方法是先移动（重命名），然后再复制回去：

```
# mv rc.conf rc.conf.orig
# cp rc.conf.orig rc.conf
```

因为 mv 保留了文件的原始日期和所有者。现在您可以编辑 rc.conf。如果您想要恢复原始内容，您可以 mv rc.conf rc.conf.myedit （假设您想保留已编辑的版本），然后

```
# mv rc.conf.orig rc.conf
```

把事情恢复到原来的样子。

要编辑文件，请输入

```
# vi filename
```

用箭头键浏览文本。按 Esc 键（退出键）进入 vi 命令模式。以下是一些命令：

x 删除光标所在位置的字母

dd 删除整行（即使在屏幕上换行）

i 在光标处插入文本

a 在光标后插入文本

一旦你输入 i 或 a ，你可以输入文本。 Esc 让你回到命令模式，在那里你可以输入

:w 写入您的更改并继续编辑

:wq 写入并退出

:q! 退出而不保存更改

/<em>text</em> 移动光标到文本; / 按 Enter（回车键）查找下一个文本实例。

G 到文件末尾

nG 转到文件中的第 n 行，其中 n 是一个数字

重新绘制屏幕的 Ctrl+L

Ctrl+b 和 Ctrl+f 会像 more 和 view 一样，回到上一个屏幕和前进一个屏幕。

在你的主目录中用 vi 练习，在那里创建一个新文件，用 vi<span> </span><em>filename</em> 添加和删除文本，保存文件，然后再次调用它。 vi 会带来一些惊喜，因为它真的很复杂，有时你会不经意地发出一个命令，做出一些你没有预料到的事情。（有些人实际上喜欢 vi - 它比 DOS 编辑器更强大 - 了解一下 :r 。）在遇到困难时，使用 Esc 一次或多次确保你处于命令模式，经常保存使用 :w ，并使用 :q! 退出并重新开始（从你最后一个 :w 处开始）当你需要的时候。

现在您可以使用 cd 进入/etc 目录， su 到 root ，使用 vi 编辑/etc/group 文件，并将用户添加到 wheel ，以便用户具有 root 权限。只需在文件的第一行末尾添加逗号和用户的登录名，按 Esc 键，并使用 :wq 写入文件到磁盘并退出。立即生效。（您没有在逗号后面加空格，对吧？）

## 6. 其他有用命令

df 显示文件空间和已挂载的系统。

ps aux 显示正在运行的进程。 ps ax 是一个更窄的形式。

  rm<span> </span><em>filename</em> 删除文件名。

rm -R<span> </span><em>dir</em> 删除目录 dir 及其所有子目录 - 小心！

ls -R 列出当前目录及所有子目录中的文件；我使用了一个变体， ls -AFR > where.txt ，来获取 / 和 /usr 中的所有文件列表，然后才找到了更好的查找文件的方法。

passwd 更改用户密码（或 root 的密码）

man hier UNIX® 文件系统的手册页

使用 find 定位 /usr 目录或其子目录中的文件名

```
% find /usr -name "filename"
```

您可以在 "<em>filename</em>" 中使用 * 作为通配符（应在引号中）。如果告诉 find 在 / 而不是 /usr 中搜索，它将在所有已挂载的文件系统上查找文件，包括 CDROM 和 DOS 分区上的文件。

一本很好的书，解释了 UNIX® 命令和实用程序，是 Abrahams & Larson, Unix for the Impatient（第 2 版，Addison-Wesley，1996）。互联网上也有很多 UNIX® 信息。

## 下一步

现在您应该有必要的工具来浏览和编辑文件，以便启动一切。在 FreeBSD 手册（可能已经在您的硬盘上）和 FreeBSD 网站中包含大量信息。CDROM 上以及网站上还有各种软件包和镜像。手册会告诉您如何使用它们（如果存在该软件包，请使用 pkg add<span> </span><em>packagename</em> 获取，其中 packagename 是软件包的文件名）。CDROM 中的 cdrom/packages/index、cdrom/packages/index.txt 和 cdrom/ports/index 中有软件包列表，并在/cdrom/ports/ / /pkg/DESCR 中有更详细的描述，其中*s 代表程序种类和程序名称的子目录。

如果您觉得手册对于从 CDROM 安装ports太复杂（带有 lndir 等内容），以下通常是有效的方法：

找到您想要的 port ，说 kermit 。光盘上会有一个目录。用以下命令将子目录复制到 /usr/local（这是您添加的软件应该对所有用户可用的好地方）：

```
# cp -R /cdrom/ports/comm/kermit /usr/local
```

这将导致一个 /usr/local/kermit 子目录，其中包含光盘上 kermit 子目录中的所有文件。

接下来，使用 mkdir 创建目录 /usr/ports/distfiles（如果尚不存在）。现在检查 /cdrom/ports/distfiles，寻找一个文件名表明它是您想要的 port 文件。将该文件复制到 /usr/ports/distfiles；在最新版本中，您可以跳过此步骤，因为 FreeBSD 会为您执行。对于 kermit ，没有 distfile。

然后 cd 到/usr/local/kermit 的子目录，找到 Makefile 文件。输入

```
# make all install
```

在这个过程中，port将通过 FTP 获取任何压缩文件，这些文件在 CDROM 或/usr/ports/distfiles 中没有找到。如果您的网络尚未运行，并且在/cdrom/ports/distfiles 中没有port的文件，您将不得不使用另一台机器获取 distfile，并将其复制到/usr/ports/distfiles。阅读 Makefile（使用 cat 或 more 或 view ）以找出去哪里（主分发站点）获取文件及其名称是什么。（使用二进制文件传输！）然后返回/usr/local/kermit，找到带有 Makefile 的目录，并输入 make all install 。

## 8.您的工作环境

您的 shell 是您工作环境中最重要的部分。shell 是解释您在命令行上输入的命令并与操作系统其余部分通信的组件。您还可以编写 shell 脚本来运行一系列命令而无需干预。

FreeBSD 自带两个 shells ： csh 和 sh 。 csh 适合命令行工作，但应使用 sh （或 bash ）来编写脚本。您可以通过输入 echo $SHELL 来查看您使用的 shell 。

csh shell 还行，但 tcsh 可以做 csh 做的一切并且更多。它允许您使用箭头键回溯命令并编辑它们。它支持文件名的 Tab 键补全（ csh 使用 Esc 键），并且可以让您切换到上次访问的目录。使用 tcsh 可以更轻松地修改提示符。它极大地简化了生活。

这里是安装新 shell 的三个步骤：

1. 安装 shell 作为 port 或包，就像安装其他 port 或包一样。
2. 使用 chsh 永久更改您的 shell 为 tcsh ，或在提示符处输入 tcsh 以更改您的 shell 而无需重新登录。

```
alias su su -m
```

當 tcsh 啟動時，它將讀取/etc/csh.cshrc 和/etc/csh.login 文件，就像 csh 一樣。它還將讀取您的家目錄中的.login 和.cshrc，除非您提供了.tcshrc。您只需將.cshrc 複製到.tcshrc 即可。

現在您已安裝 tcsh ，您可以調整您的提示符號。您可以在 tcsh 的手冊頁面中找到詳細信息，但以下是要放在您的.tcshrc 中的一行，它將告訴您已輸入了多少命令，現在幾點，以及您目前在哪個目錄中。如果您是普通用戶，它還會顯示$符號，如果您是 root ，則會顯示#號，但無論如何，tcsh 都會這樣做：

設置提示符號="%h %t %~ %#"

如果已经有现有的设置提示行，则应将此内容放置在同一位置；如果没有，则放置在 "if($?prompt) then" 下方。注释掉旧行；如果你喜欢旧行，随时可以切换回来。不要忘记空格和引号。你可以通过输入 source .tcshrc 让 .tcshrc 重新读取。

在提示符处输入 env 可以获取已设置的其他环境变量列表。结果将显示你的默认编辑器、分页器和终端类型，以及可能的其他信息。如果你从远程位置登录且由于终端不兼容而无法运行程序，这是一个有用的命令 setenv TERM vt100 。

## 9. 其他

作为 root ，你可以用 /sbin/umount /cdrom 卸载 CDROM ，将其从驱动器中取出，插入另一个，然后用 /sbin/mount_cd9660 /dev/cd0a /cdrom 挂载它，假设 cd0a 是你的 CDROM 驱动器的设备名称。最新版的 FreeBSD 允许你只用 /sbin/mount /cdrom 挂载 CDROM 。

使用 live 文件系统——FreeBSD 的第二张 CDROM 磁盘——在空间有限的情况下非常有用。live 文件系统上的内容因发行版而异。你可以尝试从 CDROM 上玩游戏。这涉及到使用 lndir （与 X Window 系统一起安装），告诉程序在哪里找到必要的文件，因为它们在 /cdrom 而不是 /usr 及其子目录中，这就是它们预期的地方。阅读 man lndir 。

## 10. 欢迎评论

如果您使用本指南，我会很乐意知道哪里不清楚以及您认为应该包括什么内容，以及其是否有帮助。感谢我的 SUNY-Stony Brook 的计算机科学教授 Eugene W. Stark 和 John Fieber 的帮助性评论。

Annelise Anderson, [andrsn@andrsn.stanford.edu](mailto:andrsn@andrsn.stanford.edu)

