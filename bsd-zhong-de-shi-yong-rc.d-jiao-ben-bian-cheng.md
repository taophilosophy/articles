# BSD 中的实用 rc.d 脚本编程

版权所有 © 2005-2006 年、2012 年 FreeBSD 项目

<details open="" data-immersive-translate-walked="909f3d6c-1300-4d98-8985-b4bcc35266e4"><summary data-immersive-translate-walked="909f3d6c-1300-4d98-8985-b4bcc35266e4" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

NetBSD 是 NetBSD 基金会的注册商标。

许多制造商和销售商用于区分其产品的名称都被视为商标。在本文档中出现这些名称时，FreeBSD 项目已意识到商标声明，因此这些名称后面加上了“™”或“®”符号。

</details>

摘要

初学者可能会发现，从正式文档关于 BSD rc.d 框架的事实与 rc.d 脚本的实际任务之间建立联系是困难的。在本文中，我们考虑了一些逐渐增加复杂性的典型案例，展示了适用于每种情况的 rc.d 特性，并讨论它们的运作方式。这样的研究应该为进一步研究 rc.d 的设计和有效应用提供参考点。

---

## 1. 引言

历史上的 BSD 有一个整体的启动脚本，/etc/rc。它在系统启动时由 init(8) 调用，并执行所有用户空间任务，包括检查和挂载文件系统、设置网络、启动守护进程等。在每个系统中，任务列表并不完全相同；管理员需要进行自定义。除了少数例外，/etc/rc 必须被修改，而真正的黑客喜欢这样做。

单片系统方法的真正问题在于，它无法控制从 /etc/rc 启动的各个组件。例如，/etc/rc 无法重新启动单个守护进程。系统管理员必须手动查找守护进程进程，将其杀死，等待直到它实际退出，然后浏览 /etc/rc 查找标志，最后输入完整的命令行来重新启动守护进程。如果要重新启动的服务由多个守护进程组成或需要执行其他操作，则此任务将变得更加困难且容易出错。简而言之，这个单一脚本未能达到脚本的目的：让系统管理员的生活变得更轻松。

后来，有人试图将 /etc/rc 的某些部分分割出来，以便单独启动最重要的子系统。臭名昭着的例子是 /etc/netstart 用于启动网络。它确实允许在单用户模式下访问网络，但它无法很好地融入自动启动过程，因为其代码的某些部分需要与与网络毫不相关的行动交错。这就是为什么 /etc/netstart 变异成 /etc/rc.network。后者不再是一个普通的脚本；它由大型的、纠缠不清的 sh(1) 函数组成，从 /etc/rc 在系统启动的不同阶段调用。然而，随着启动任务变得更加多样化和复杂，这种“准模块化”方法变得比单片式 /etc/rc 还要繁琐。

没有干净且设计良好的框架，启动脚本必须尽最大努力来满足快速发展的基于 BSD 的操作系统的需求。最终，人们意识到在打造一个细粒度且可扩展的 rc 系统时需要更多的步骤。因此诞生了 BSD rc.d。其公认的创始人是 Luke Mewburn 和 NetBSD 社区。后来它被引入到 FreeBSD 中。它的名称指的是系统脚本的位置，即个别服务的系统脚本在 /etc/rc.d 中。很快我们将了解更多有关 rc.d 系统的组件，并看到如何调用个别脚本。

BSD rc.d 背后的基本思想是良好的模块化和代码重用。良好的模块化意味着每个基本的“服务”，如系统守护进程或基本的启动任务，都拥有自己的 sh(1)脚本，能够启动服务、停止服务、重新加载服务、检查服务状态。通过命令行参数选择特定操作。/etc/rc 脚本仍然驱动系统启动，但现在只是逐个调用这些更小的脚本并传递一个 start 参数。通过以 stop 参数运行相同一组脚本同样可以轻松执行关机任务，这是由/etc/rc.shutdown 完成的。请注意，这如何紧随 Unix 的方式，即拥有一组小而专业的工具，每个都尽最大可能地完成其任务。代码重用意味着常用操作被实现为 sh(1)函数并收集在/etc/rc.subr 中。现在，一个典型的脚本可能只值得几行 sh(1)代码。最后，rc.d 框架的一个重要部分是 rcorder(8)，它帮助/etc/rc 按照它们之间的依赖关系有序地运行这些小脚本。这对/etc/rc.shutdown 也有帮助，因为关机序列的正确顺序与启动相反。

BSD rc.d 设计在 Luke Mewburn 的原始文章中有所描述，rc.d 组件在各自的手册页面中有非常详细的文档。然而，对于一个 rc.d 新手来说，如何将许多零碎部分组合在一起以创建一个针对特定任务的样式良好的脚本可能并不明显。因此，本文将尝试用不同的方法来描述 rc.d。它将展示在许多典型情况下应该使用哪些特性，以及为什么。请注意，这不是一个操作说明文档，因为我们的目的不是提供现成的配方，而是展示几个轻松进入 rc.d 领域的方法。这篇文章也不是相关手册页面的替代品。在阅读本文时，不要犹豫参考它们以获取更正式和完整的文档。

理解本文需要先决条件。首先，您应当熟悉 sh(1)脚本语言以掌握 rc.d。此外，您应当了解系统如何执行用户态启动和关闭任务，这在 rc(8)中有描述。

本文侧重于 FreeBSD 分支的 rc.d。然而，这对 NetBSD 开发人员也许有用，因为 BSD rc.d 的这两个分支不仅在设计上相同，而且在脚本作者可见的方面上也保持相似。

## 2. 描述任务

在开始 $EDITOR 之前稍作考虑不会有坏处。为了为系统服务编写一个良好的 rc.d 脚本，我们应该首先能够回答以下几个问题：

- 服务是必需的还是可选的？
- 脚本是否只服务于单个程序，例如守护进程，还是执行更复杂的操作？
- 我们的服务将依赖哪些其他服务，反之亦然？

从接下来的例子中，我们将看到为什么了解这些问题的答案是如此重要。

## 3. 一个虚拟的脚本

以下脚本在每次系统启动时只是发出一条消息：

```sh
#!/bin/sh

. /etc/rc.subr

name="dummy"
start_cmd="${name}_start"
stop_cmd=":"

dummy_start()
{
	echo "Nothing started."
}

load_rc_config $name
run_rc_command "$1"
```

注意以下事项：

➊ 一个解释脚本应该以魔术的 "shebang" 行开始。这一行指定了脚本的解释器程序。由于有了 shebang 行，脚本可以像二进制程序一样被调用，只要它的执行位被设置了。（参见 chmod(1)。）例如，系统管理员可以在命令行手动运行我们的脚本：

```
# /etc/rc.d/dummy start
```

>要被 rc.d 框架正确管理，其脚本需要用 sh(1) 语言编写。如果您有一个服务或 port 使用了其他语言编写的二进制控制实用程序或启动例程，请将该元素安装在 /usr/sbin（用于系统）或 /usr/local/sbin（用于 ports）中，并从适当的 rc.d 目录中的 sh(1) 脚本调用它。 

>如果您想了解 rc.d 脚本为何必须用 sh(1) 语言编写的详细信息，请查看 /etc/rc 通过 run_rc_script 调用它们的方式，然后研究 /etc/rc.subr 中 run_rc_script 的实现。 

➋ 在 /etc/rc.subr 中，定义了许多 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 函数供 rc.d 脚本使用。这些函数在 [rc.subr(8) ](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 中有详细的文档。虽然理论上可以编写一个不使用 [rc.subr(8) ](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 的 rc.d 脚本，但它的函数非常方便，能够大大简化工作。因此，大家在编写 rc.d 脚本时通常都会使用 [rc.subr(8) ](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html)，我们也不例外。

一个 rc.d 脚本必须在调用 [rc.subr(8) ](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 的函数之前“source”/etc/rc.subr（使用 “.” 引入），以便 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 有机会加载这些函数。推荐的做法是首先 source /etc/rc.subr。

> 一些与网络相关的有用函数由另一个包含文件 /etc/network.subr 提供。

➌ 必需的变量 name 指定了我们脚本的名称。它被 rc.subr(8) 需要。也就是说，每个 rc.d 脚本在调用 rc.subr(8) 函数之前必须设置 name 。

现在是选择我们的脚本独特名称的正确时机。在开发脚本时，我们会在许多地方使用它。首先，让我们也给脚本文件取相同的名称。

>当前的 rc.d 脚本风格是用双引号括起分配给变量的值。请记住，这只是一个样式问题，不一定总是适用。当单词中没有 sh(1)元字符时，您可以安全地省略简单单词周围的引号，而在某些情况下，您将需要单引号来防止 sh(1)对值的任何解释。程序员应该能够区分语言语法和样式约定，并明智地使用它们。 

➍ rc.subr(8) 的主要思想是，一个 rc.d 脚本提供处理程序，或方法，供 rc.subr(8) 调用。特别是， start ， stop ，和其他参数到一个 rc.d 脚本以这种方式处理。一个方法是一个 sh(1)表达式存储在一个名为 argument_cmd 的变量中，其中参数对应于可以在脚本命令行上指定的内容。我们将看到 rc.subr(8) 如何为标准参数提供默认方法。

>为了使 rc.d 中的代码更统一，通常在适当的地方使用 ${name} 。因此，许多行代码可以从一个脚本直接复制到另一个脚本中。

➎ 我们应该记住 rc.subr(8) 为标准参数提供默认方法。因此，如果我们希望一个标准方法什么都不做，我们必须用一个空操作的 sh(1)表达式来覆盖它。

➏ 复杂方法的主体可以实现为函数。让函数名有意义是个好主意。

>强烈建议在我们脚本中定义的所有函数名称前添加前缀 ${name} ，这样它们永远不会与 rc.subr(8) 中的函数或另一个常见的包含文件中的函数冲突。 

➐ 对 rc.subr(8) 的此调用加载 rc.conf(5) 变量。我们的脚本尚未使用它们，但建议仍加载 rc.conf(5) ，因为可能有 rc.conf(5) 变量控制 rc.subr(8) 本身。

➑ 通常这是一个 rc.d 脚本中的最后一个命令。它调用 rc.subr(8) 机制，使用我们的脚本提供的变量和方法执行请求的操作。

## 4. 可配置的虚拟脚本

现在让我们为我们的虚拟脚本添加一些控制。正如您所知，rc.d 脚本是通过 rc.conf(5) 控制的。幸运的是，rc.subr(8) 会将所有复杂性隐藏起来。以下脚本通过 rc.conf(5) 通过 rc.subr(8) 查看是否首先启用它来查找一个要在启动时显示的消息。实际上，这两个任务是独立的。一方面，一个 rc.d 脚本可以支持启用和禁用其服务。另一方面，一个强制性的 rc.d 脚本可以有配置变量。尽管如此，我们将在同一个脚本中完成这两件事：

```sh
#!/bin/sh

. /etc/rc.subr

name=dummy
rcvar=dummy_enable

start_cmd="${name}_start"
stop_cmd=":"

load_rc_config $name
: ${dummy_enable:=no}
: ${dummy_msg="Nothing started."}

dummy_start()
{
	echo "$dummy_msg"
}

run_rc_command "$1"
```

此示例中有什么变化？

➊ 变量 rcvar 指定了开/关按钮变量的名称。

➋ 现在 load_rc_config 在脚本中被更早调用，在访问任何 rc.conf(5)  变量之前。

>在检查 rc.d 脚本时，请记住 sh(1)推迟对函数中表达式的评估直到调用后。因此，在就要调用 run_rc_command 之前晚至最后一刻调用是没有错误的，并且仍然可以访问从导出到 run_rc_command 的方法函数中的 rc.conf(5) 变量。这是因为方法函数将由 run_rc_command 调用，后者在 load_rc_config 之后被调用。 

如果设置了 rcvar 本身，但指示的旋钮变量未设置，则 run_rc_command 会发出警告。如果您的 rc.d 脚本用于基本系统，则应将旋钮的默认设置添加到/etc/defaults/rc.conf，并在 rc.conf(5) 中进行文档化。否则，应由您的脚本为旋钮提供默认设置。后一种情况的规范方法示例中显示。

```
# /etc/rc.d/dummy onestart
```

现在，在启动时显示的消息不再在脚本中硬编码。它由名为 dummy_msg 的 rc.conf(5) 变量指定。这是 rc.conf(5) 变量控制 rc.d 脚本的一个简单示例。

>我们脚本独占使用的所有 rc.conf(5)  变量的名称必须具有相同的前缀: ${name}_ 。例如: dummy_mode ， dummy_state_file ，等等。


>虽然在内部可以使用较短的名称，例如只有 msg ，但在我们脚本引入的所有全局名称中添加唯一前缀 ${name}_ 将使我们免受与 rc.subr(8)  命名空间可能发生冲突的困扰。<br /><br />作为一个原则，基本系统的 rc.d 脚本不需要为其 rc.conf(5)  变量提供默认值，因为应该在 /etc/defaults/rc.conf 中设置默认值。另一方面，对于 ports 的 rc.d 脚本应按示例中显示提供默认值。


➎ 在这里，我们使用 `dummy_msg` 来实际控制我们的脚本，即发出一个变量消息。使用 shell 函数有些过于复杂，因为它只运行一个命令；一个同样有效的替代方法是：

```sh
start_cmd="echo "$dummy_msg""
```

## 5. 启动和关闭一个简单的守护进程

我们之前提到过，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 可以提供默认方法。显然，这些默认方法不能太过通用，它们适用于启动和关闭一个简单守护进程程序的常见情况。现在假设我们需要为一个名为 `mumbled` 的守护进程编写一个 rc.d 脚本。下面是脚本示例：

```
#!/bin/sh

. /etc/rc.subr

name=mumbled
rcvar=mumbled_enable

command="/usr/sbin/${name}"

load_rc_config $name
run_rc_command "$1"
```

简单而令人愉悦，不是吗？让我们检查一下我们的小脚本。要注意的唯一新事物如下：

➊ command 变量对 rc.subr(8)  是有意义的。如果设置了它，rc.subr(8)  将根据为传统守护程序提供默认方法的情形进行操作。特别是将为这些参数提供默认方法： start ， stop ， restart ， poll 和 status 。

守护程序将通过运行 $command 以及由 $mumbled_flags 指定的命令行标志来启动。因此，默认 start 方法的所有输入数据都可以在我们的脚本设置的变量中使用。与 start 不同，其他方法可能需要关于启动的进程的其他信息。例如， stop 必须知道要终止的进程的 PID。在这种情况下，rc.subr(8)  将扫描所有进程列表，寻找其名称等于 procname 的进程。后者是 rc.subr(8)  的另一个有意义的变量，其值默认为 command 。换句话说，当我们设置 command 时， procname 实际上被设置为相同的值。这使我们的脚本能够终止守护程序并检查它是否正在运行。

> 一些程序实际上是可执行脚本。系统通过启动其解释器并将脚本的名称作为命令行参数传递给它来运行这样的脚本。这反映在进程列表中，可能会使 rc.subr(8) 混淆。如果 $command 是脚本，您还应设置 command_interpreter 以便 rc.subr(8) 知道进程的实际名称。<br /><br />对于每个 rc.d 脚本，都有一个可选的 rc.conf(5) 变量，优先于 command 。其名称构造如下： ${name}_program ，其中 name 是我们之前讨论的必需变量。例如，在这种情况下，它将是 mumbled_program 。正是 rc.subr(8) 安排 ${name}_program 覆盖 command 。
>
> 当然，sh(1)允许您即使 command 未设置也可以从 rc.conf(5) 或脚本本身设置 ${name}_program 。在这种情况下， ${name}_program 的特殊属性丢失了，它变成了您的脚本可以用于自己目的的普通变量。然而，强烈不建议仅使用 ${name}_program ，因为与 command 一起使用已成为 rc.d 脚本的习语。 

对于有关默认方法的更详细信息，请参阅 rc.subr(8) 。

## 6. 启动和关闭高级守护程序

让我们给上一个脚本增加些实质性内容，使其更复杂和功能丰富。默认方法可以为我们做得很好，但我们可能需要调整它们的某些方面。现在我们将学习如何根据我们的需要调优默认方法。

```sh
#!/bin/sh

. /etc/rc.subr

name=mumbled
rcvar=mumbled_enable

command="/usr/sbin/${name}"
command_args="mock arguments > /dev/null 2>&1"

pidfile="/var/run/${name}.pid"

required_files="/etc/${name}.conf /usr/share/misc/${name}.rules"

sig_reload="USR1"

start_precmd="${name}_prestart"
stop_postcmd="echo Bye-bye"

extra_commands="reload plugh xyzzy"

plugh_cmd="mumbled_plugh"
xyzzy_cmd="echo 'Nothing happens.'"

mumbled_prestart()
{
	if checkyesno mumbled_smart; then
		rc_flags="-o smart ${rc_flags}"
	fi
	case "$mumbled_mode" in
	foo)
		rc_flags="-frotz ${rc_flags}"
		;;
	bar)
		rc_flags="-baz ${rc_flags}"
		;;
	*)
		warn "Invalid value for mumbled_mode"
		return 1
		;;
	esac
	run_rc_command xyzzy
	return 0
}

mumbled_plugh()
{
	echo 'A hollow voice says "plugh".'
}

load_rc_config $name
run_rc_command "$1"
```

➊ 可以在 command_args 中传递到 $command 的附加参数。它们将被添加到 $mumbled_flags 之后的命令行中。由于最终命令行传递给 eval 进行实际执行，因此可以在 command_args 中指定输入和输出重定向。

>不要在 command_args 中包含带有破折号的选项，如 -X 或 --foo 。 command_args 的内容将出现在最终命令行的末尾，因此它们可能会跟在 ${name}_flags 中存在的参数后面；但是大多数命令在普通参数之后不会识别带有破折号的选项。将附加选项传递给 $command 的更好方法是将它们添加到 ${name}_flags 的开头。另一种方法是稍后展示的修改 rc_flags 。 

➋ 一位有礼貌的守护进程应该创建一个 pidfile，以便其进程可以更容易、更可靠地找到。如果设置了变量 pidfile ，它将告诉 rc.subr(8) 在哪里可以找到 pidfile 以供其默认方法使用。

>事实上，rc.subr(8)  还会使用 pidfile 来查看守护进程是否已经在运行，然后再启动它。可以通过使用 faststart 参数来跳过此检查。 

➌ 如果守护进程在某些文件存在之前无法运行，只需将它们列在 required_files 中，rc.subr(8)  将在启动守护进程之前检查这些文件是否存在。还有 required_dirs 和 required_vars 分别用于目录和环境变量。它们都在 rc.subr(8)  中有详细描述。

>可以通过使用 forcestart 作为脚本的参数强制跳过 rc.subr(8)  的默认检查方法。

➍ 我们可以定制发送给守护进程的信号，以防它们与常见信号不同。特别地， sig_reload 指定了使守护进程重新加载配置的信号；默认情况下是 SIGHUP。另一个信号用于停止守护进程；默认是 SIGTERM，但可以通过适当设置 sig_stop 进行更改。

>信号名称应以不带 SIG 前缀的形式指定给 rc.subr(8) ，如示例中所示。FreeBSD 版本的 kill(1) 能识别 SIG 前缀，但其他操作系统类型的版本可能无法识别。

➎➏ 在默认方法之前或之后执行额外任务很简单。对于我们脚本支持的每个命令参数，我们可以定义 argument_precmd 和 argument_postcmd 。这些 sh(1) 命令在各自的方法之前和之后被调用，正如它们的名称所示。

>通过使用自定义 argument_cmd 覆盖默认方法，仍然不会阻止我们在需要时使用 argument_precmd 或 argument_postcmd 。特别是前者非常适合检查在执行命令之前应满足的自定义复杂条件。与 argument_cmd 一起使用 argument_precmd 让我们在逻辑上将检查与动作分开。<br /><br />不要忘记，您可以将任何有效的 sh(1)表达式塞入您定义的方法、预处理和后处理命令中。在大多数情况下，仅调用执行实际工作的函数是一个很好的风格，但永远不要让风格限制您对幕后发生的理解。 

如果我们想要实现自定义参数，也可以将其视为我们脚本的命令，则需要在 extra_commands 中列出它们，并提供处理它们的方法。

> `reload` 命令是特殊的。一方面，它在 [rc.subr(8) ](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 中有一个预设的方法。另一方面，`reload` 并不是默认提供的。原因是并非所有守护进程使用相同的重新加载机制，有些守护进程根本没有需要重新加载的内容。因此，我们需要明确请求提供内置功能。我们可以通过 `extra_commands` 来实现。<br /><br />我们从默认方法中得到什么？通常，守护进程会在接收到信号时重新加载其配置——通常是 SIGHUP。因此，[rc.subr(8) ](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 尝试通过向守护进程发送信号来重新加载它。信号预设为 SIGHUP，但如果需要，可以通过 `sig_reload` 进行自定义。

➑⓮ 我们的脚本支持两个非标准命令，`plugh` 和 `xyzzy`。我们在 `extra_commands` 中看到了它们的列出，现在是时候为它们提供方法了。`xyzzy` 的方法直接内联实现，而 `plugh` 的方法则作为 `mumbled_plugh` 函数实现。

非标准命令在启动或关闭过程中不会被调用。通常它们是为系统管理员方便而设计的。如果在 devd.conf(5)中指定，在其他子系统中也可以使用，例如 devd(8)。

完整的可用命令列表可以在没有参数调用脚本时由 rc.subr(8) 打印的用法行中找到。例如，这是正在研究的脚本的用法行：

```sh
# /etc/rc.d/mumbled
Usage: /etc/rc.d/mumbled [fast|force|one](start|stop|restart|rcvar|reload|plugh|xyzzy|status|poll)
```

脚本可以在需要时调用自己的标准或非标准命令。这可能看起来类似于调用函数，但我们知道命令和 shell 函数并不总是相同的东西。例如， xyzzy 在这里不作为函数实现。另外，可以有一个预命令和后命令，应该按顺序调用。因此，脚本运行其自己命令的正确方式是通过 rc.subr(8) ，如示例所示。

➒ rc.subr(8)  提供了一个方便的名为 checkyesno 的函数。它以一个变量名作为参数，如果变量设置为 YES 、 TRUE 、 ON 或 1 （不区分大小写），则返回零退出码；否则返回非零退出码。在后一种情况下，函数测试变量是否设置为 NO 、 FALSE 、 OFF 或 0 （不区分大小写）；如果变量包含其他内容（即垃圾），则打印警告消息。

请记住，对于 sh(1)，零退出码表示真，非零退出码表示假。

```sh
if checkyesno mumbled_enable; then
        foo
fi
```

相反，如下所示调用 checkyesno 将不起作用——至少不符合预期。

```sh
if checkyesno "${mumbled_enable}"; then
        foo
fi
```

➓ 我们可以通过修改 $command 中的 rc_flags 来影响传递到 $start_precmd 的标志。

⓫ 在某些情况下，我们可能需要发出一个重要信息，该信息也应发送到 syslog 。这可以通过以下 rc.subr(8)  函数轻松完成： debug ， info ， warn 和 err 。然后，后一个函数使用指定的代码退出脚本。

⓬ 默认情况下，并不只是忽略方法和其前命令的退出代码。如果 argument_precmd 返回非零退出代码，则主方法将不会执行。反过来，除非主方法返回零退出码，否则将不会调用 argument_postcmd 。

> 然而，rc.subr(8) 可以通过在参数前加 force 来从命令行忽略这些退出码并调用所有命令。 

## 7. 将脚本连接到 rc.d 框架

编写脚本后，需要将其集成到 rc.d 中。关键步骤是将脚本安装在/etc/rc.d（用于基础系统）或/usr/local/etc/rc.d（用于 ports）中。bsd.prog.mk 和 bsd.port.mk 均提供了方便的钩子，通常无需担心正确的所有权和模式。系统脚本应通过 src/libexec/rc/rc.d 中的 Makefile 安装。Port 脚本可以使用 Porter’s Handbook 中描述的 USE_RC_SUBR 进行安装。

然而，我们应该事先考虑我们的脚本在系统启动顺序中的位置。由我们脚本处理的服务可能依赖于其他服务。例如，网络守护程序在没有网络接口和路由运行的情况下无法正常工作。即使一个服务似乎不需要任何东西，它也几乎不能在基本文件系统已经被检查和挂载之前启动。

我们已经提到了 rcorder(8)。现在是时候仔细看看它了。简而言之，rcorder(8)接受一组文件，检查它们的内容，并从该组文件中打印一个依赖顺序的列表到 stdout 。重点是将依赖信息保存在文件内部，使得每个文件只能代表它自己。一个文件可以指定以下信息：

- 它提供的“条件”名称（对我们来说是服务）。
- 定制的 "条件" 名称所需;
- 此文件应在其前运行的 "条件" 名称;
- 可用于从整套文件中选择子集的附加关键词（通过选项，rcorder(8) 可以指示包含或排除具有特定关键词的文件。)

rcorder(8) 只能处理语法接近于 sh(1)的文本文件并不足为奇。也就是说，rcorder(8)理解的特殊行看起来像 sh(1)的注释。这些特殊行的语法相当严格，以简化它们的处理。有关详细信息，请参阅 rcorder(8)。

除了使用 rcorder(8)的特殊行外，脚本还可以通过强制启动来强制其依赖于另一个服务。当另一个服务是可选的，并且由于系统管理员错误地在 rc.conf(5) 中禁用它而不会自行启动时，可能需要这样做。

有了这些基本知识，让我们考虑一下增强了依赖关系内容的简单守护程序脚本：

```sh
#!/bin/sh

# PROVIDE: mumbled oldmumble
# REQUIRE: DAEMON cleanvar frotz
# BEFORE:  LOGIN
# KEYWORD: nojail shutdown

. /etc/rc.subr

name=mumbled
rcvar=mumbled_enable

command="/usr/sbin/${name}"
start_precmd="${name}_prestart"

mumbled_prestart()
{
	if ! checkyesno frotz_enable && 
	    ! /etc/rc.d/frotz forcestatus 1>/dev/null 2>&1; then
		force_depend frotz || return 1
	fi
	return 0
}

load_rc_config $name
run_rc_command "$1"
```

如前所述，以下是详细分析：

➊ 该行声明了我们脚本提供的“条件”名称。现在，其他脚本可以通过这些名称记录对我们脚本的依赖。

> 通常，一个脚本只指定一个条件。然而，什么也不妨碍我们在此列出多个条件，例如出于兼容性原因。
>
> 无论如何，主要，或者唯一的 PROVIDE: 条件的名称应与 ${name} 相同。 

➋➌ 因此，我们的脚本指示它依赖其他脚本提供的“条件”。根据这些行，我们的脚本要求 rcorder(8)在提供 DAEMON 和 cleanvar 脚本之后，但在提供 LOGIN 脚本之前放置它。

>BEFORE: 行不应被滥用来解决其他脚本中不完整的依赖项列表。使用 BEFORE: 的适当情况是当其他脚本不关心我们的脚本，但我们的脚本在其他脚本之前运行时可以更好地完成其任务。典型的现实生活示例是网络接口与防火墙：虽然接口在完成其工作时不依赖防火墙，但在出现任何网络流量之前，系统安全将受益于防火墙的准备就绪。
>
>除了对应于单个服务的条件外，还存在元条件及其“占位符”脚本，用于确保某些操作组在其他操作之前执行。这些由大写名称表示。它们的列表和目的可在 rc(8) 中找到。
>
>请记住，在 REQUIRE: 行中放置服务名称并不能保证服务在我们的脚本启动时实际运行。所需服务可能启动失败或在 rc.conf(5)  中被禁用。显然，rcorder(8) 无法跟踪此类细节，rc(8) 也不会这样做。因此，我们的脚本启动的应用程序应能够处理任何所需服务不可用的情况。在某些情况下，我们可以按下面讨论的方式帮助它。 

➍ 正如我们从上文记得的那样，rcorder(8) 关键字可用于选择或留出某些脚本。即任何 rcorder(8) 消费者均可通过 -k 和 -s 选项指定哪些关键字位于“保留列表”和“跳过列表”上。从所有待依赖排序的文件中，rcorder(8) 将仅选择具有保留列表中关键字（除非为空）且不具有跳过列表中关键字的文件。

在 FreeBSD 中，/etc/rc 和 /etc/rc.shutdown 使用 rcorder(8)。 这两个脚本定义了 FreeBSD rc.d 关键字的标准列表及其含义如下：

nojail 服务不适用于 jail(8)环境。 如果在 jail 中，自动启动和关闭过程将忽略该脚本。

nostart 服务需要手动启动或根本不启动。 自动启动过程将忽略该脚本。 与关闭关键字一起，可用于编写仅在系统关闭时执行某些操作的脚本。

此关键字需要在系统关机前明确列出，以便在系统关机前停止服务。

>当系统即将关闭时，/etc/rc.shutdown 会运行。它假定大多数 rc.d 脚本在那个时候不需要做任何事情。因此，/etc/rc.shutdown 会有选择地使用带有 shutdown 关键字的 rc.d 脚本，从而有效地忽略其余的脚本。为了实现更快速的关机，/etc/rc.shutdown 会将 faststop 命令传递给运行的脚本，以便它们跳过预检查，例如 pidfile 检查。由于依赖服务应在其先决条件之前停止，/etc/rc.shutdown 会按照反向依赖顺序运行脚本。如果编写真正的 rc.d 脚本，您应考虑它在系统关机时是否相关。例如，如果您的脚本仅在响应启动命令时才执行工作，则不需要包含此关键字。但是，如果您的脚本管理一个服务，则在系统继续执行描述在 halt(8) 中的最终阶段之前停止它可能是一个好主意。特别是，如果服务在关机时需要较长时间或特殊操作来干净地关闭，则应明确停止服务。此类服务的典型示例是数据库引擎。

首先， force_depend 应当谨慎使用。如果您的 rc.d 脚本相互依赖，通常最好调整配置变量的层次结构。

如果您仍然无法没有 force_depend ，那么示例提供了如何在条件下调用它的习语。在本例中，我们的 mumbled 守护程序要求预先启动另一个 frotz 。但是， frotz 也是可选的；rcorder(8) 对这些细节一无所知。幸运的是，我们的脚本可以访问所有 rc.conf(5)  变量。如果 frotz_enable 为真，我们希望一切顺利，并依赖于 rc.d 已启动 frotz 。否则，我们会强制检查 frotz 的状态。最后，如果发现 frotz 未运行，我们会强制执行对其依赖。由于只有在检测到错误配置时才应调用它， force_depend 将发出警告消息。

## 8. 为 rc.d 脚本提供更大的灵活性

在启动或关闭期间调用时，rc.d 脚本应该对其负责的整个子系统进行操作。例如，/etc/rc.d/netif 应启动或停止 rc.conf(5)  描述的所有网络接口。可以通过单个命令参数（如 start 或 stop ）唯一指示任一任务。在启动和关闭之间，rc.d 脚本帮助管理员控制正在运行的系统，这就是需要更多灵活性和精度的时候。例如，管理员可能希望将新网络接口的设置添加到 rc.conf(5) ，然后启动它，而不干扰现有接口的操作。下次管理员可能需要关闭单个网络接口。在命令行的精神下，相应的 rc.d 脚本需要额外的参数，即接口名称。

幸运的是，rc.subr(8) 允许将任意数量的参数传递给脚本的方法（在系统限制范围内）。由于这个特性，脚本本身的更改可以很小。

rc.subr(8) 如何获得额外的命令行参数呢？直接获取吗？绝不是。首先，sh（1）函数无法访问其调用者的位置参数，但 rc.subr(8) 只是这些函数的集合。其次，rc.d 的良好习惯规定，由主脚本决定传递哪些参数给它的方法。

因此，rc.subr(8) 采取的方法如下： run_rc_command 将除第一个参数外的所有参数逐字传递给相应的方法。第一个被省略的参数是方法本身的名称： start ， stop 等。它将被 run_rc_command 移出，因此原始命令行中的 $2 将被呈现为 $1 给方法，依此类推。

为了说明这个机会，让我们修改原始的 dummy 脚本，使其消息取决于提供的附加参数。开始吧：

```sh
#!/bin/sh

. /etc/rc.subr

name="dummy"
start_cmd="${name}_start"
stop_cmd=":"
kiss_cmd="${name}_kiss"
extra_commands="kiss"

dummy_start()
{
        if [ $# -gt 0 ]; then
                echo "Greeting message: $*"
        else
                echo "Nothing started."
        fi
}

dummy_kiss()
{
        echo -n "A ghost gives you a kiss"
        if [ $# -gt 0 ]; then
                echo -n " and whispers: $*"
        fi
        case "$*" in
        *[.!?])
                echo
                ;;
        *)
                echo .
                ;;
        esac
}

load_rc_config $name
run_rc_command "$@"
```

我们可以在脚本中注意到哪些基本变化？

➊ 你在 start 后输入的所有参数都可以作为位置参数传递给相应的方法。我们可以根据任务、技能和爱好以任何方式使用它们。在当前示例中，我们只是将它们作为一个字符串传递给 echo(1) 的下一行 - 注意双引号内的 $* 。现在脚本的调用方式如下：

```sh
# /etc/rc.d/dummy start
Nothing started.

# /etc/rc.d/dummy start Hello world!
Greeting message: Hello world!
```

➊ 我们的脚本提供的任何方法都适用，不仅适用于标准方法。我们添加了一个定制方法名为 kiss ，它可以利用不少于 start 的额外参数。例如：

```sh
# /etc/rc.d/dummy kiss
A ghost gives you a kiss.

# /etc/rc.d/dummy kiss Once I was Etaoin Shrdlu...
A ghost gives you a kiss and whispers: Once I was Etaoin Shrdlu...
```

➋ 如果我们只想将所有额外的参数传递给任何方法，我们只需在脚本的最后一行用 "$1" 替换 "$@" ，在那里我们调用 run_rc_command 。

>sh(1) 程序员应了解 $* 和 $@ 之间微妙的区别，作为指定所有位置参数的方法。有关其深入讨论，请参考一本关于 sh(1) 脚本编写的好手册。在完全理解之前，请不要使用这些表达式，因为错误使用会导致脚本出现错误和不安全的情况。 

>目前 run_rc_command 可能存在一个 bug，该 bug 会导致它无法保留参数之间的原始边界。也就是说，带有嵌入空格的参数可能无法正确处理。该 bug 源于 $* 的误用。 

## 9. 使脚本准备好作为服务 Jails

启动长时间运行服务的脚本适合于服务 jails，并应配备适当的服务 jail 配置。

一些脚本示例，这些脚本不适合在服务中运行 jail：

- 任何仅在启动命令中更改程序或内核运行时设置的脚本，
- 或尝试挂载某些内容，
- 或查找并删除文件

脚本不适合在服务 jail 中运行，需要阻止在服务 jails 内使用。

一个长时间运行的服务脚本，在启动前或停止后需要执行上述操作之一，可以将其拆分为具有依赖关系的两个脚本，或者使用脚本的预命令和后命令部分执行此操作。

默认情况下，只有脚本的开始和停止部分在服务 jail 中运行，其余部分在外部运行。因此，在脚本的开始/停止部分使用的任何设置不能从例如预命令设置。

要使脚本准备好与服务 Jails 一起使用，只需要插入一行更多的配置。

```sh
#!/bin/sh

. /etc/rc.subr

name="dummy"
start_cmd="${name}_start"
stop_cmd=":"

: ${dummy_svcj_options:=""}

dummy_start()
{
        echo "Nothing started."
}

load_rc_config $name
run_rc_command "$1"
```

➊ 如果脚本在 jail 中运行是有意义的，它必须具有可重写的服务 jails 配置。如果它不需要网络访问或访问 jails 中受限制的任何其他资源，则像显示的空配置就足够了。

严格来说，不需要空配置，但它明确描述了脚本已经准备好作为服务 jails，并且不需要额外的 jail 权限。因此，强烈建议在这种情况下添加这样一个空配置。最常用的选项是"net_basic"，它允许使用主机的 IPv4 和 IPv6 地址。所有可能的选项在 rc.conf(5) 中都有解释。

如果启动/停止的设置取决于来自 rc-framework 的变量（例如在 rc.conf(5) 中设置），则需要由 load_rc_config and run_rc_command 而不是在 precommand 中处理。

如果由于某种原因脚本无法在服务 jail 中运行，例如因为无法运行它或者在 jail 中不运行它没有意义，使用以下方法：

```sh
#!/bin/sh

. /etc/rc.subr

name="dummy"
start_cmd="${name}_start"
stop_cmd=":"

dummy_start()
{
        echo "Nothing started."
}

load_rc_config $name
dummy_svcj="NO"		# does not make sense to run in a svcj
run_rc_command "$1"
```

➊ 禁用需要在 load_rc_config 调用之后进行，否则 rc.conf(5)  设置可能会覆盖它。

## 10. 进一步阅读

Luke Mewburn 的原始文章提供了 rc.d 的概述以及其设计决策的详细解释。它提供了对整个 rc.d 框架以及其在现代 BSD 操作系统中的位置的见解。

rc(8)、rc.subr(8)  和 rcorder(8) 的手册页面详细记录了 rc.d 组件。在编写自己的脚本时，如果不研究并参考这些手册页面，就无法充分利用 rc.d 的功能。

实际工作中的主要示例源自活动系统中的 /etc/rc.d。其内容易于阅读，因为大多数技术细节都深藏在 rc.subr(8) 中。但请记住，/etc/rc.d 脚本并非由天使编写，因此可能存在 bug 和次优的设计决策。现在，您可以改进它们了！
