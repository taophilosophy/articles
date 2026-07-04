# BSD 中的实用 rc.d 脚本编程

- 原文：[Practical rc.d scripting in BSD](https://docs.freebsd.org/en/articles/rc-scripting/)

## 摘要

初学者可能会觉得很难将来自正式文档中关于 BSD **rc.d** 框架的事实与实际的 **rc.d** 脚本任务联系起来。本文将考虑几个典型的、逐渐复杂的案例，展示适用于每个案例的 **rc.d** 特性，并讨论它们如何工作。这样的探讨应该为进一步研究 **rc.d** 的设计和高效应用提供参考点。

## 1. 引言

历史上的 BSD 系统有一个单一的启动脚本 **/etc/rc**。它在系统启动时由 [init(8)](https://man.freebsd.org/cgi/man.cgi?query=init&sektion=8&format=html) 调用，并执行所有多用户操作所需的用户空间任务：检查和挂载文件系统、设置网络、启动守护进程等。具体的任务列表在不同的系统中有所不同；管理员需要根据需要进行定制。除少数例外，**/etc/rc** 必须进行修改，真正的黑客对此感到满意。

单一脚本方法的真正问题在于，它无法控制从 **/etc/rc** 启动的各个组件。例如，**/etc/rc** 无法重新启动单个守护进程。系统管理员必须手动找到守护进程，终止它，等待它实际退出，然后浏览 **/etc/rc** 查找标志，最后输入完整的命令行来重新启动守护进程。如果要重新启动的服务包含多个守护进程或需要其他操作，这项任务会变得更加困难且容易出错。简而言之，单一脚本未能实现脚本的基本目的：使系统管理员的工作更轻松。

后来，为了单独启动最重要的子系统，曾试图将一些 **/etc/rc** 的部分内容拆分出来。著名的例子是 **/etc/netstart** 用于启动网络。它确实允许从单用户模式访问网络，但由于其代码的一些部分需要与本质上与网络无关的操作交织在一起，因此未能很好地融入自动启动过程。这就是为什么 **/etc/netstart** 最终变成了 **/etc/rc.network** 的原因。后者不再是一个普通的脚本；它由大型、复杂的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 函数组成，这些函数在系统启动的不同阶段由 **/etc/rc** 调用。然而，随着启动任务的多样化和复杂化，这种“准模块化”方法变得比单一的 **/etc/rc** 更加繁琐。

在没有干净且设计良好的框架的情况下，启动脚本不得不竭尽全力满足快速发展的基于 BSD 的操作系统的需求。最终，越来越明显的是，要实现一个细粒度且可扩展的 **rc** 系统，还需要更多的步骤。于是，BSD **rc.d** 应运而生。它的公认创始人是 Luke Mewburn 和 NetBSD 社区。后来它被引入到 FreeBSD。它的名称来源于系统脚本的位置，这些脚本用于单独的服务，位于 **/etc/rc.d**。很快我们将了解 **rc.d** 系统的更多组件，并查看如何调用单个脚本。

BSD **rc.d** 背后的基本思想是 *细粒度模块化* 和 *代码重用*。*细粒度模块化* 意味着每个基本的“服务”如系统守护进程或基本启动任务都有自己的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 脚本，能够启动服务、停止它、重新加载它、检查其状态。特定的操作通过脚本的命令行参数来选择。**/etc/rc** 脚本仍然负责系统启动，但它现在只是依次调用这些小脚本，并传递 `start` 参数。通过用 `stop` 参数运行同一组脚本，执行关闭任务变得也很容易，这由 **/etc/rc.shutdown** 完成。请注意，这与 Unix 系统的方式非常接近，Unix 系统有一组小型的专用工具，每个工具尽可能地完成其任务。*代码重用* 意味着常见操作通过 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 函数实现，并集中在 **/etc/rc.subr** 中。现在，典型的脚本可能只是几行 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 代码。最后，**rc.d** 框架的一个重要部分是 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html)，它帮助 **/etc/rc** 按照它们之间的依赖关系有序地运行这些小脚本。它也可以帮助 **/etc/rc.shutdown**，因为关闭序列的正确顺序与启动顺序相反。

BSD **rc.d** 的设计在 [Luke Mewburn 的原始文章](https://docs.freebsd.org/en/articles/rc-scripting/#lukem) 中有所描述，**rc.d** 组件在 [相关手册页](https://docs.freebsd.org/en/articles/rc-scripting/#manpages) 中得到了详细的文档说明。然而，对于一个 **rc.d** 新手来说，如何将众多的片段和组件结合起来，创建一个风格良好的特定任务脚本，可能并不显而易见。因此，本文将采用一种不同的方法来描述 **rc.d**。它将展示在一些典型情况下应该使用哪些特性，以及为什么。请注意，这不是一份操作指南，因为我们的目标不是提供现成的配方，而是展示一些进入 **rc.d** 领域的简单入口。这篇文章也不是相关手册页的替代品。在阅读本文时，遇到需要正式和完整文档的部分，应该随时参考手册页。

理解本文有一些前提条件。首先，你应该熟悉 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 脚本语言，以掌握 **rc.d**。此外，你应该了解系统如何执行用户空间的启动和关闭任务，这在 [rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html) 中有所描述。

本文侧重于 FreeBSD 分支的 **rc.d**。尽管如此，它对于 NetBSD 开发人员也可能有用，因为两个 BSD **rc.d** 分支不仅共享相同的设计，而且在脚本作者可见的方面也保持相似。


## 2. 概述任务

在开始编辑 `$EDITOR` 之前，稍作思考不会有坏处。为了编写一个调和良好的 **rc.d** 脚本来管理系统服务，我们应该首先能够回答以下问题：

- 这个服务是必须的还是可选的？
- 脚本是为单个程序（例如守护进程）服务，还是执行更复杂的操作？
- 我们的服务依赖于哪些其他服务，反之亦然？

从以下的示例中，我们将看到为什么了解这些问题的答案如此重要。

## 3. 虚拟脚本

下面的脚本在系统每次启动时都会发出一条消息：

```sh
#!/bin/sh ①

. /etc/rc.subr ②

name="dummy" ③
start_cmd="${name}_start" ④
stop_cmd=":" ⑤

dummy_start() ⑥
{
	echo "Nothing started."
}

load_rc_config $name ⑦
run_rc_command "$1" ⑧
```

需要注意的事项如下：

① 解释型脚本应该以魔法的“shebang”行开始。该行指定脚本的解释器程序。由于 shebang 行，脚本可以像二进制程序一样被调用，只要设置了执行位。例如，系统管理员可以从命令行手动运行我们的脚本：

```sh
# /etc/rc.d/dummy start
```

>**注意**
>
> 为了被 **rc.d** 框架正确管理，脚本需要使用 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 语言编写。如果你有一个服务或 Port 使用二进制控制工具或用其他语言编写的启动例程，可以将该元素安装到 **/usr/sbin**（用于系统）或 **/usr/local/sbin**（用于 Port），并从相应的 **rc.d** 目录中的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 脚本中调用它。

>**技巧**
>
> 如果你想了解为什么 **rc.d** 脚本必须用 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 语言编写的详细原因，请查看 **/etc/rc** 如何通过 `run_rc_script` 调用它们，然后研究 **/etc/rc.subr** 中 `run_rc_script` 的实现。

② 在 **/etc/rc.subr** 中定义了多个供 **rc.d** 脚本使用的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 函数。这些函数在 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 中有文档说明。虽然理论上可以编写不使用 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 的 **rc.d** 脚本，但它的函数非常方便，使工作变得容易得多。所以，毫不奇怪，大家都在 **rc.d** 脚本中使用 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html)，我们也不例外。

**rc.d** 脚本必须在调用 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 函数之前“source”**/etc/rc.subr**（通过“.”包含它），以便 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 可以了解这些函数。推荐的写法是首先包含 **/etc/rc.subr**。

>**注意**
>
> 有关网络的某些有用函数由另一个包含文件 **/etc/network.subr** 提供。

③ 必须变量 `name` 指定了我们脚本的名称。它是 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 所要求的。也就是说，每个 **rc.d** 脚本 *必须* 在调用 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 函数之前设置 `name`。

现在是时候为我们的脚本选择一个唯一的名称了。我们将会在编写脚本的多个地方使用它。`name` 变量的内容需要与脚本名称匹配，FreeBSD 的某些部分（例如 [service jails](https://docs.freebsd.org/en/articles/rc-scripting/#rcng-service-jails) 和 rc 框架的 cpuset 功能）依赖于此。因此，文件名也不能包含可能在脚本中引起问题的字符（例如，不要使用连字符“-”等）。

>**注意**
>
> 当前的 **rc.d** 脚本风格是将分配给变量的值用双引号括起来。请记住，这仅仅是一个风格问题，可能并非总是适用。你可以安全地省略不包含 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 元字符的简单词语周围的引号，而在某些情况下你需要使用单引号来防止 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 对值的解释。程序员应能根据语言语法和风格约定来区分并合理使用两者。

④ [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 的主要思想是，**rc.d** 脚本提供处理程序或方法，以供 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 调用。特别是，`start`、`stop` 和 **rc.d** 脚本的其他参数就是通过这种方式处理的。方法是一个存储在名为 `argument_cmd` 的变量中的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 表达式，其中 *argument* 对应于脚本命令行中可以指定的内容。稍后我们将看到 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 为标准参数提供了默认的方法。

>**注意**
>
> 为了使 **rc.d** 代码更统一，通常在适当的地方使用 `${name}`。这样，许多行可以直接从一个脚本复制到另一个脚本。

⑤ 我们应该记住， [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 提供了标准参数的默认方法。因此，如果我们希望某个标准方法什么也不做，我们必须使用一个无操作的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 表达式来重载该方法。

⑥ 复杂方法的主体可以作为函数来实现。为函数命名时最好有意义。

>**重要**
>
> 强烈建议为脚本中定义的所有函数添加 `${name}` 前缀，这样它们就不会与 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 或其他公共包含文件中的函数冲突。

⑦ 这行调用 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 加载 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量。我们的脚本目前尚未使用这些变量，但仍然建议加载 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html)，因为可能有控制 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 本身的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量。

⑧ 通常这是 **rc.d** 脚本中的最后一条命令。它调用 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 的机制，根据我们提供的变量和方法执行请求的操作。

## 4. 可配置的虚拟脚本

现在让我们为我们的虚拟脚本添加一些控制选项。如你所知，**rc.d** 脚本通过 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 来进行控制。幸运的是，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 隐藏了所有复杂性。下面的脚本使用 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 通过 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 来检查它是否启用，并在启动时显示一条消息。这两个任务实际上是独立的。一方面，**rc.d** 脚本可以只支持启用和禁用它的服务。另一方面，强制性的 **rc.d** 脚本可以有配置变量。尽管如此，我们会在同一个脚本中做这两件事：

```sh
#!/bin/sh

. /etc/rc.subr

name=dummy
rcvar=dummy_enable ①

start_cmd="${name}_start"
stop_cmd=":"

load_rc_config $name ②
: ${dummy_enable:=no} ③
: ${dummy_msg="Nothing started."} ④

dummy_start()
{
	echo "$dummy_msg" ⑤
}

run_rc_command "$1"
```

在这个例子中有什么变化呢？

① 变量 `rcvar` 指定了 ON/OFF 开关变量的名称。

② 现在 `load_rc_config` 在脚本中较早地被调用，在访问任何 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量之前。

>**注意**
>
> 在检查 **rc.d** 脚本时，请记住 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 会推迟评估函数中的表达式，直到函数被调用。因此，在脚本中很晚才调用 `load_rc_config` 并且仍然能够访问 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量，是没有问题的，因为方法函数是在 `load_rc_config` 调用之后由 `run_rc_command` 调用的。

③ 如果 `rcvar` 本身已设置，但所指示的开关变量未设置，`run_rc_command` 将发出警告。如果你的 **rc.d** 脚本是系统自带的，你应当将开关变量的默认设置添加到 **/etc/defaults/rc.conf** 中，并在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中进行文档说明。否则，应该由你的脚本提供开关的默认设置。后者的标准方法如本例所示。

>**注意**
>
> 你可以通过在脚本的参数前加上 `one` 或 `force`，使 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 认为开关已设置为 `ON`，不管其当前设置如何，例如：`onestart` 或 `forcestop`。不过，请记住，`force` 会有其他危险影响，稍后会讨论，而 `one` 只是覆盖了开关。假设 `dummy_enable` 为 `OFF`，那么下面的命令将强制执行 `start` 方法，尽管该设置为 `OFF`：
>
>```sh
># /etc/rc.d/dummy onestart
>```

④ 现在，启动时显示的消息不再硬编码在脚本中。它由一个名为 `dummy_msg` 的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量指定。这是一个简单的例子，展示了如何通过 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量来控制 **rc.d** 脚本。

>**重要**
>
> 所有仅由我们的脚本使用的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量的名称 *必须* 具有相同的前缀：`${name}_`。例如：`dummy_mode`、`dummy_state_file` 等等。

>**注意**
>
> 尽管可以在内部使用更短的名称，例如仅使用 `msg`，但将唯一的前缀 `${name}_` 添加到脚本引入的所有全局名称，将帮助避免与 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 命名空间发生冲突。作为规则，基础系统中的 **rc.d** 脚本无需为它们的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量提供默认值，因为默认值应该在 **/etc/defaults/rc.conf** 中设置。另一方面，**rc.d** 脚本对于 Port 则应提供像示例中那样的默认值。

⑤ 现在我们使用 `dummy_msg` 来控制脚本，即显示一个变量消息。这里使用 shell 函数是多余的，因为它只运行一条命令；一个同样有效的替代方法是：

```sh
start_cmd="echo \"$dummy_msg\""
```

## 5. 简单守护进程的启动与关闭

我们之前提到过，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 可以提供默认方法。显然，这些默认方法不能过于通用，它们适用于启动和关闭简单守护进程。现在假设我们需要为一个名为 `mumbled` 的守护进程编写 **rc.d** 脚本。下面是该脚本：

```sh
#!/bin/sh

. /etc/rc.subr

name=mumbled
rcvar=mumbled_enable

command="/usr/sbin/${name}" ①

load_rc_config $name
run_rc_command "$1"
```

这个脚本看起来简单而又简洁，不是吗？让我们分析一下这个小脚本。唯一需要注意的新事物是：

① `command` 变量对于 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 是有意义的。如果它被设置，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 将根据提供的默认方法来启动和关闭守护进程。特别地，默认方法会为以下命令提供支持：`start`、`stop`、`restart`、`poll` 和 `status`。

守护进程将通过运行 `$command` 并使用 `$mumbled_flags` 指定的命令行标志来启动。因此，默认 `start` 方法的所有输入数据都可以通过我们脚本中设置的变量获得。与 `start` 方法不同，其他方法可能需要有关已启动进程的附加信息。例如，`stop` 方法必须知道要终止的进程的 PID。在这种情况下，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 会扫描所有进程列表，寻找名称为 `procname` 的进程。后者是另一个对 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 有意义的变量，默认情况下其值与 `command` 相同。换句话说，当我们设置 `command` 时，`procname` 会自动设置为相同的值。这使得我们的脚本能够终止守护进程，并首先检查它是否正在运行。

>**注意**
>
>有些程序实际上是可执行脚本。系统通过启动其解释器并将脚本的名称作为命令行参数传递给它来运行此类脚本。这会反映在进程列表中，可能会让 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 感到困惑。如果 `$command` 是一个脚本，应该额外设置 `command_interpreter` 变量，以便 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 知道实际的进程名称。
>
>对于每个 **rc.d** 脚本，都有一个可选的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量，它优先于 `command`。该变量的名称按照以下规则构造：`${name}_program`，其中 `name` 是我们之前讨论的必需变量。例如，在此案例中，它将是 `mumbled_program`。是 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 负责让 `${name}_program` 重写 `command`。
>
>当然，即使未设置 `command`，[sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 也允许你从 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 或脚本本身设置 `${name}_program`。在这种情况下，`${name}_program` 的特殊属性就会丧失，它变成了一个普通的变量，供脚本自用。然而，不建议单独使用 `${name}_program`，因为它与 `command` 一起使用已经成为 **rc.d** 脚本的惯例。

有关默认方法的更多详细信息，请参阅 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html)。

## 6. 启动和关闭高级守护进程

让我们为之前的脚本增加一些内容，使其变得更加复杂和功能丰富。默认方法对我们来说已经足够好，但有时我们可能需要调整它们的一些方面。现在我们将学习如何根据我们的需求来调整默认方法。

```sh
#!/bin/sh

. /etc/rc.subr

name=mumbled
rcvar=mumbled_enable

command="/usr/sbin/${name}"
command_args="mock arguments > /dev/null 2>&1" ①

pidfile="/var/run/${name}.pid" ②

required_files="/etc/${name}.conf /usr/share/misc/${name}.rules" ③

sig_reload="USR1" ④

start_precmd="${name}_prestart" ⑤
stop_postcmd="echo Bye-bye" ⑥

extra_commands="reload plugh xyzzy" ⑦

plugh_cmd="mumbled_plugh" ⑧
xyzzy_cmd="echo 'Nothing happens.'"

mumbled_prestart()
{
	if checkyesno mumbled_smart; then ⑨
		rc_flags="-o smart ${rc_flags}" ⑩
	fi
	case "$mumbled_mode" in
	foo)
		rc_flags="-frotz ${rc_flags}"
		;;
	bar)
		rc_flags="-baz ${rc_flags}"
		;;
	*)
		warn "Invalid value for mumbled_mode" ⑪
		return 1 ⑫
		;;
	esac
	run_rc_command xyzzy ⑬
	return 0
}

mumbled_plugh() ⑭
{
	echo 'A hollow voice says "plugh".'
}

load_rc_config $name
run_rc_command "$1"
```

① 除了 `$command` 外，还可以通过 `command_args` 传递额外的参数。这些参数将添加到 `$mumbled_flags` 后面。由于最终的命令行会通过 `eval` 执行，因此可以在 `command_args` 中指定输入输出重定向。

>**注意**
>
> *永远不要* 在 `command_args` 中包含以破折号开头的选项，如 `-X` 或 `--foo`。`command_args` 的内容会出现在最终命令行的末尾，因此它们可能会跟在 `${name}_flags` 中的参数后面，而大多数命令在普通参数后不会识别这些带破折号的选项。传递额外的选项给 `$command` 的更好方法是将它们放在 `${name}_flags` 的前面，或者修改 `rc_flags` [如后文所示](https://docs.freebsd.org/en/articles/rc-scripting/#rc-flags)。

② 良好设计的守护进程应该创建一个 *pidfile*，这样可以更容易且更可靠地找到它的进程。设置 `pidfile` 变量后，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 会在默认方法中使用该 pidfile。

>**注意**
>
> 实际上，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 还会使用 pidfile 检查守护进程是否已在运行。若要跳过此检查，可以使用 `faststart` 参数。

③ 如果守护进程必须在启动前确保某些文件存在，可以将这些文件列在 `required_files` 中，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 会在启动守护进程之前检查这些文件是否存在。此外，还有 `required_dirs` 和 `required_vars` 用于检查目录和环境变量，具体内容请查阅 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html)。

>**注意**
>
>可以通过在命令行中使用 `forcestart` 参数来强制跳过这些前置检查。

④ 如果守护进程需要的信号与默认的不同，我们可以自定义信号。例如，`sig_reload` 指定了用于重新加载守护进程配置的信号，默认是 SIGHUP。另一个信号是用于停止守护进程的，默认是 SIGTERM，但也可以通过设置 `sig_stop` 来改变它。

>**注意**
>
>信号名称应当不带 `SIG` 前缀，正如示例中所示。FreeBSD 版本的 [kill(1)](https://man.freebsd.org/cgi/man.cgi?query=kill&sektion=1&format=html) 可以识别 `SIG` 前缀，但其他操作系统版本可能不识别。

⑤⑥ 在默认方法之前或之后执行额外的任务很简单。对于脚本支持的每个命令参数，我们可以定义 `argument_precmd` 和 `argument_postcmd`，这些 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 命令将在相应方法的前后执行，名称也非常直观。

>**注意**
>
>即使我们重写了默认方法并定义了自定义的 `argument_cmd`，也可以使用 `argument_precmd` 或 `argument_postcmd`。特别地，前者非常适合用来检查自定义的复杂条件，只有在条件满足时才执行命令本身。结合使用 `argument_precmd` 和 `argument_cmd` 可以将检查与动作逻辑上分开。
>
>在方法、前后命令中，你可以放入任何有效的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 表达式。虽然调用一个实现实际工作的函数是大多数情况下的好风格，但不要让风格限制你对其背后原理的理解。

⑦ 如果我们想实现自定义的命令（也可以称为 *命令*），只需将它们列在 `extra_commands` 中，并提供相应的处理方法。

>**注意**
>
> `reload` 命令是特殊的。一方面，它在 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 中有预设的方法。另一方面，`reload` 默认是不会提供的。原因是并不是所有守护进程都使用相同的重新加载机制，有些甚至根本不需要重新加载。因此我们需要显式要求提供内建的功能，可以通过 `extra_commands` 来实现。
>
>默认的 `reload` 方法做了什么呢？通常情况下，守护进程会在接收到某个信号后重新加载其配置——通常是 SIGHUP。因此，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 会尝试通过向守护进程发送信号来重新加载它。默认信号是 SIGHUP，但可以通过 `sig_reload` 来进行定制。

⑧⑭ 我们的脚本支持两个非标准命令，`plugh` 和 `xyzzy`。它们在 `extra_commands` 中列出，现在是时候为它们提供方法了。`xyzzy` 的方法就是内联定义的，而 `plugh` 则由 `mumbled_plugh` 函数实现。

非标准命令在启动或关闭时不会被调用。它们通常是为了系统管理员的便利而设，也可以从其他子系统调用，例如在 [devd(8)](https://man.freebsd.org/cgi/man.cgi?query=devd&sektion=8&format=html) 中通过 [devd.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=devd.conf&sektion=5&format=html) 指定。

完整的可用命令列表可以在脚本没有参数时通过 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 打印的使用行中找到。例如，这是我们正在研究的脚本中的使用行：

```sh
# /etc/rc.d/mumbled
Usage: /etc/rc.d/mumbled [fast|force|one](start|stop|restart|rcvar|reload|plugh|xyzzy|status|poll)
```

⑬ 脚本可以根据需要调用它自己的标准或非标准命令。这看起来像是调用函数，但我们知道命令和 shell 函数并不总是相同的。例如，`xyzzy` 在这里不是作为函数实现的。此外，还可以有前命令和后命令，它们应该按顺序执行。因此，脚本运行自己的命令的正确方法是通过 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html)，如示例所示。

⑨ 有个非常方便的函数 `checkyesno` 是 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 提供的。它以变量名作为参数，只有当变量的值为 `YES`、`TRUE`、`ON` 或 `1`（不区分大小写）时，返回零退出码；否则返回非零退出码。如果变量包含其他值，即无效的值，它会打印一条警告信息。

记住，在 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 中，零退出码意味着真，非零退出码意味着假。

>**重要**
>
>`checkyesno` 函数接受的是 *变量名*。不要将变量的展开 *值* 传给它；那样不会按预期工作。
>
>下面是 `checkyesno` 的正确用法：
>
>```sh
>if checkyesno mumbled_enable; then
>        foo
>fi
>```
>
>相反，如下所示调用 `checkyesno` 将不起作用 —— 至少不会按预期工作：
>
>```sh
>if checkyesno "${mumbled_enable}"; then
>        foo
>fi
>```


⑩ 我们可以通过在 `$start_precmd` 中修改 `rc_flags` 来影响传递给 `$command` 的参数。

⑪ 在某些情况下，我们可能需要发出一条重要信息，并同时将其写入 `syslog`。这可以通过以下 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 提供的函数轻松实现：`debug`、`info`、`warn` 和 `err`。其中，`err` 函数在输出信息后会以指定的状态码退出脚本。

⑫ 方法及其 pre-command 返回的退出码默认不会被忽略。如果 `argument_precmd` 返回一个非零的退出码，则主方法将不会被执行。反过来，除非主方法返回零退出码，`argument_postcmd` 也不会被调用。

>**注意**
>
> 不过，可以通过在命令行参数前加上 `force`，如 `forcestart`，来指示 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 忽略这些退出码并无论如何执行所有命令。

## 7. 将脚本接入 rc.d 框架

写好脚本之后，就需要将它整合进 **rc.d**。关键的一步是将脚本安装到 **/etc/rc.d**（用于基础系统）或 **/usr/local/etc/rc.d**（用于 Port）。**bsd.prog.mk** 和 **bsd.port.mk** 都提供了方便的安装钩子，通常无需担心文件的所有权和权限。系统脚本应通过 **src/libexec/rc/rc.d** 下的 **Makefile** 进行安装；Port 的脚本则可以使用 `USE_RC_SUBR` 来安装，具体做法见 [Porter’s Handbook](https://docs.freebsd.org/en/books/porters-handbook/#rc-scripts)。

不过，在此之前，我们应该先考虑脚本在系统启动序列中的位置。脚本所管理的服务很可能依赖于其他服务。例如，网络守护进程在网络接口和路由尚未启动之前是无法工作的。即使某个服务看似不依赖其他内容，它也不可能在基本文件系统被检查和挂载之前启动。

我们之前提到过 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html)。现在是时候仔细了解一下它了。简而言之，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 会读取一组文件，分析其内容，然后按依赖顺序将这些文件输出到 `stdout`。关键点在于将依赖信息保存在文件*内部*，让每个文件只描述自身。一个文件可以指定以下信息：

- 它所 *提供* 的“条件”（即我们所说的服务）；
- 它所 *需要* 的“条件”；
- 它应该在其前运行的“条件”；
- 额外的 *关键字*，这些关键字可用于从整个文件集中选择子集（可以通过参数告知 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 包括或排除含特定关键字的文件）。

不出意外，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 只能处理语法类似 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 的文本文件。也就是说，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 所能识别的特殊行看起来像 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 的注释。这些特殊行的语法相当严格，以便于程序处理。详见 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html)。

除了使用 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 的特殊行之外，一个脚本还可以通过强制启动另一个服务来表达其依赖关系。这在所依赖服务是可选的情况下尤其有用——如果系统管理员在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中禁用了该服务，它本不会自动启动。

了解了以上这些基础知识之后，我们来看一个添加了依赖信息的简单守护进程脚本示例：

```sh
#!/bin/sh

# PROVIDE: mumbled oldmumble ①
# REQUIRE: DAEMON cleanvar frotz ②
# BEFORE:  LOGIN ③
# KEYWORD: nojail shutdown ④

. /etc/rc.subr

name=mumbled
rcvar=mumbled_enable

command="/usr/sbin/${name}"
start_precmd="${name}_prestart"

mumbled_prestart()
{
	if ! checkyesno frotz_enable && \
	    ! /etc/rc.d/frotz forcestatus 1>/dev/null 2>&1; then
		force_depend frotz || return 1 ⑤
	fi
	return 0
}

load_rc_config $name
run_rc_command "$1"
```

如前所述，接下来是详细分析：

① 这一行声明了我们的脚本所 *提供* 的“条件”名称。现在其他脚本就可以通过这些名称来记录对我们脚本的依赖关系。

>**注意**
>
>通常，一个脚本只会声明一个所提供的条件。不过，并没有什么限制我们在此列出多个条件，例如为了兼容性原因。
>
>无论如何，主要的（或唯一的）`PROVIDE:` 条件名称应当与 `${name}` 相同。

②③ 所以我们的脚本表明它依赖于其他脚本所提供的哪些“条件”。根据这些行，我们的脚本要求 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 将它排列在提供 **DAEMON** 和 **cleanvar** 的脚本之后，但在提供 **LOGIN** 的脚本之前。

>**注意**
>
>`BEFORE:` 行不应被滥用于绕过其他脚本中不完整的依赖列表。使用 `BEFORE:` 的合理情形是：另一个脚本并不关心我们的脚本是否存在，但如果我们的脚本能在它之前运行，则可以更好地完成任务。一个典型的现实示例是网络接口与防火墙：虽然接口在工作时并不依赖防火墙，但若防火墙在网络流量出现前就已就绪，系统安全性将受益。
>
>除了每个服务对应一个条件外，还有一些元条件及其“占位符”脚本，用于确保某些操作组在其他操作组之前执行。这些元条件以 **全大写字母** 表示。它们的列表和用途可在 [rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html) 中找到。
>
>请记住，在 `REQUIRE:` 行中列出某个服务名称，并不能保证该服务在我们的脚本启动时一定已运行。被要求的服务可能启动失败，或者只是被系统管理员在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中禁用了。显然，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 无法跟踪此类细节，[rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html) 也不会这样做。因此，我们脚本启动的应用程序应能应对任何所需服务不可用的情况。在某些情形下，我们可以像 [下文](https://docs.freebsd.org/en/articles/rc-scripting/#forcedep) 所述那样提供帮助。

④ 正如我们在上文中所了解到的，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 的关键字可用于选择或排除某些脚本。也就是说，任何 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 的使用者都可以通过 `-k` 和 `-s` 选项，分别指定哪些关键字属于“保留列表”或“跳过列表”。在所有待进行依赖排序的文件中，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 将只挑选那些具备保留列表中的关键字（如果该列表非空），且不具备跳过列表中关键字的文件。

在 FreeBSD 中，[rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 被 **/etc/rc** 和 **/etc/rc.shutdown** 所使用。这两个脚本定义了 FreeBSD **rc.d** 关键字的标准列表及其含义如下：

**nojail**
该服务不适用于 [jail(8)](https://man.freebsd.org/cgi/man.cgi?query=jail&sektion=8&format=html) 环境。如果在 Jail 中，自动启动和关闭流程将忽略此脚本。

**nostart**
该服务应由手动启动，或根本不启动。自动启动流程将忽略此脚本。若与 **shutdown** 关键字同时使用，可用于编写仅在系统关机时执行操作的脚本。

**shutdown**
若服务需要在系统关机前被停止，必须*显式*列出此关键字。


>**注意**
>
>当系统即将关机时，**/etc/rc.shutdown** 会运行。它假设大多数 **rc.d** 脚本在那时没有任何操作。因此，**/etc/rc.shutdown** 会选择性地调用具有 **shutdown** 关键字的 **rc.d** 脚本，忽略其他脚本。为了更快的关机，**/etc/rc.shutdown** 会向其运行的脚本传递 **faststop** 命令，促使它们跳过初步检查，例如 pidfile 检查。由于依赖的服务应当在其前提条件之前停止，**/etc/rc.shutdown** 按反向依赖顺序运行脚本。如果编写一个真实的 **rc.d** 脚本，你应当考虑它在系统关机时是否相关。例如，如果你的脚本仅在接收到 **start** 命令时才执行工作，那么你不需要包括此关键字。然而，如果你的脚本管理一个服务，最好在系统进入最终关机阶段（如 [halt(8)](https://man.freebsd.org/cgi/man.cgi?query=halt&sektion=8&format=html) 中描述的）之前停止该服务。特别是，当某个服务需要较长时间或特殊操作才能干净地关闭时，应该显式地停止它。数据库引擎就是此类服务的典型例子。

⑤ 首先，`force_depend` 应当谨慎使用。通常情况下，如果你的 **rc.d** 脚本之间有依赖关系，最好是重新审视它们的配置变量层次结构。

如果你仍然无法避免使用 `force_depend`，示例展示了如何有条件地调用它。在这个示例中，我们的 `mumbled` 守护进程要求另一个守护进程 `frotz` 先启动。然而，`frotz` 也是可选的；而 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 并不知道这些细节。幸运的是，我们的脚本可以访问所有的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 变量。如果 `frotz_enable` 为真，我们会尽力而为，依赖 **rc.d** 启动 `frotz`。否则，我们强制检查 `frotz` 的状态。最后，如果发现 `frotz` 没有运行，我们会强制执行对 `frotz` 的依赖。`force_depend` 会发出警告信息，因为它应该只在发现配置错误时才被调用。

## 8. 给 rc.d 脚本提供更多灵活性

在启动或关机时调用时，**rc.d** 脚本应当操作其负责的整个子系统。例如，**/etc/rc.d/netif** 应该启动或停止由 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 描述的所有网络接口。任何一项任务都可以通过单一的命令参数，如 `start` 或 `stop` 来唯一指示。在启动和关机之间，**rc.d** 脚本帮助管理员控制运行中的系统，这时就需要更多的灵活性和精确性。例如，管理员可能希望将一个新的网络接口的设置添加到 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中，然后启动它，而不干扰现有接口的操作。下次，管理员可能需要关闭单个网络接口。为了符合命令行精神，相应的 **rc.d** 脚本需要一个额外的参数，即接口名称。

幸运的是，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 允许将任何数量的参数传递给脚本的方法（在系统限制内）。因此，脚本本身的修改可以最小化。

那么，如何让 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 获得额外的命令行参数呢？它是否应该直接获取这些参数？当然不行。首先，**sh(1)** 函数无法访问调用者的定位参数，而 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 只是这些函数的集合。其次，**rc.d** 的良好习惯是由主脚本决定哪些参数传递给其方法。

因此，[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 采用的做法如下：`run_rc_command` 将所有参数（除了第一个）原样传递给相应的方法。第一个被省略的参数是方法本身的名称：`start`、`stop` 等。它会被 `run_rc_command` 移除，因此原始命令行中的 `$2` 将作为 `$1` 传递给方法，依此类推。

为了说明这个机会，让我们修改这个原始的虚拟脚本，使得其消息依赖于传递的额外参数。我们来看一下：

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
        if [ $# -gt 0 ]; then ①
                echo "Greeting message: $*"
        else
                echo "Nothing started."
        fi
}

dummy_kiss()
{
        echo -n "A ghost gives you a kiss"
        if [ $# -gt 0 ]; then ②
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
run_rc_command "$@" ③
```

我们可以注意到脚本中的哪些关键变化？

① 你在 `start` 后输入的所有参数都可以成为相应方法的位置参数。我们可以根据任务、技能和需要以任何方式使用它们。在当前的示例中，我们只是将所有参数作为一个字符串传递给 [echo(1)](https://man.freebsd.org/cgi/man.cgi?query=echo&sektion=1&format=html)，请注意双引号内的 `$*`。以下是如何调用该脚本的示例：

```
# /etc/rc.d/dummy start
Nothing started.

# /etc/rc.d/dummy start Hello world!
Greeting message: Hello world!
```

② 同样的方式适用于脚本提供的任何方法，而不仅仅是标准方法。我们添加了一个名为 `kiss` 的自定义方法，它同样可以利用额外的参数，就像 `start` 方法那样。例如：

```
# /etc/rc.d/dummy kiss
A ghost gives you a kiss.

# /etc/rc.d/dummy kiss Once I was Etaoin Shrdlu...
A ghost gives you a kiss and whispers: Once I was Etaoin Shrdlu...
```

③ 如果我们只想将所有额外的参数传递给某个方法，我们可以简单地在脚本的最后一行将 `"$@"` 替换为 `"$1"`，在该行调用 `run_rc_command`。

>**重要**
>
> [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 程序员应当理解 `$*` 和 `$@` 之间的微妙区别，因为它们表示所有位置参数。有关详细讨论，请参考一本好的 [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&format=html) 脚本手册。在完全理解之前*不要*使用这些表达式，因为它们的误用会导致脚本有漏洞并不安全。

>**注意**
>
> 当前 `run_rc_command` 可能存在一个 bug，导致它无法保持参数之间的原始边界。也就是说，包含空格的参数可能无法正确处理。该 bug 源于 `$*` 的误用。

## 9. 使脚本准备好用于服务 Jail

启动长期运行服务的脚本适合用于服务 Jail，并应配有适当的服务 Jail 配置。

以下是一些不适合在服务 Jail 中运行的脚本示例：

- 仅在 `start` 命令中更改程序或内核的运行时设置的任何脚本，
- 尝试挂载某些内容的脚本，
- 查找并删除文件的脚本。

不适合在服务 Jail 中运行的脚本需要防止在服务 Jail 中使用。

如果一个脚本具有长期运行的服务，在启动或停止之前需要执行上述操作中的某些操作，则可以将其拆分为两个有依赖关系的脚本，或使用脚本的 `precommand` 和 `postcommand` 部分来执行该操作。

默认情况下，只有脚本的 `start` 和 `stop` 部分会在服务 Jail 中运行，其余部分则在 Jail 外部运行。因此，任何在 `start`/`stop` 部分中使用的设置不能通过例如 `precommand` 来设置。

为了使脚本准备好与 [Service Jails](https://docs.freebsd.org/en/books/handbook/jails/#service-jails) 一起使用，只需要插入一行配置：

```sh
#!/bin/sh

. /etc/rc.subr

name="dummy"
start_cmd="${name}_start"
stop_cmd=":"

: ${dummy_svcj_options:=""} ①

dummy_start()
{
        echo "Nothing started."
}

load_rc_config $name
run_rc_command "$1"
```

① 如果脚本在 Jail 中运行是有意义的，它必须具有可覆盖的服务 Jail 配置。如果它不需要网络访问或访问 Jail 中受限的任何其他资源，则像显示的那样使用空配置即可。

严格来说，空配置并不是必需的，但它明确说明了该脚本已准备好用于服务 Jail，并且不需要额外的 Jail 权限。因此，在这种情况下，强烈建议添加这样的空配置。最常用的选项是 `net_basic`，它启用对主机 IPv4 和 IPv6 地址的使用。所有可能的选项在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中都有解释。

如果 `start`/`stop` 设置依赖于来自 rc 框架的变量（例如，在 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中设置的变量），则需要通过 `load_rc_config` 和 `run_rc_command` 来处理，而不是在 `precommand` 中处理。

如果出于某种原因，脚本无法在服务 Jail 中运行，例如因为无法运行或在 Jail 中运行没有意义，则使用以下方法：

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
dummy_svcj="NO"		# 在 svcj 中运行没有意义 ①
run_rc_command "$1"
```

① 禁用操作需要在 `load_rc_config` 调用之后进行，否则 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 中的设置可能会覆盖它。

## 10. 高级 rc 脚本：实例化

有时需要运行多个实例的服务。通常，你希望能够独立启动/停止这些实例，并且希望每个实例有一个单独的配置文件。每个实例应在启动时启动，能够在更新后存活，并受益于更新。

以下是支持此功能的 rc 脚本示例：

```sh
#!/bin/sh

#
# PROVIDE: dummy
# REQUIRE: NETWORKING SERVERS
# KEYWORD: shutdown
#
# 将以下行添加到 /etc/rc.conf.local 或 /etc/rc.conf
# 以启用此服务：
#
# dummy_enable (bool):	设置为 YES 以在启动时启用 dummy。
#			默认值：NO
# dummy_user (string):	运行时使用的用户账户。
#			默认值：www
#

. /etc/rc.subr

case $0 in ①
/etc/rc*)
	# 在启动（关机）时，$0 是 /etc/rc (/etc/rc.shutdown)，
	# 所以从 $_file 获取脚本的名称
	name=$_file
	;;
*)
	name=$0
	;;
esac

name=${name##*/} ②
rcvar="${name}_enable" ③
desc="该服务的简短描述"
command="/usr/local/sbin/dummy"

load_rc_config "$name"

eval "${rcvar}=\${${rcvar}:-'NO'}" ④
eval "${name}_svcj_options=\${${name}_svcj_options:-'net_basic'}" ⑤
eval "_dummy_user=\${${name}_user:-'www'}" ⑥

_dummy_configname=/usr/local/etc/${name}.cfg ⑦
pidfile=/var/run/dummy/${name}.pid
required_files ${_dummy_configname}
command_args="-u ${_dummy_user} -c ${_dummy_configfile} -p ${pidfile}"

run_rc_command "$1"
```


① 和 ② 确保将 `name` 变量设置为脚本文件名的 [basename(1)](https://man.freebsd.org/cgi/man.cgi?query=basename&sektion=1&format=html)。如果文件名是 **/usr/local/etc/rc.d/dummy**，则 `name` 设置为 **dummy**。这样，改变 rc 脚本的文件名将自动改变 `name` 变量的内容。

③ 指定用于 **rc.conf** 中启用该服务的变量名，基于此脚本的文件名。在此示例中，它解析为 `dummy_enable`。

④ 确保 `_enable` 变量的默认值为 NO。

⑤ 是为服务特定的框架变量提供一些默认值的示例，在此示例中是服务 Jail 选项。

⑥ 和 ⑦ 设置脚本内部的变量（注意在 `_dummy_user` 前的下划线，它使其与可以在 **rc.conf** 中设置的 `dummy_user` 区分开）。

⑤ 部分用于脚本内部未使用但在 rc 框架中使用的变量。所有在脚本中作为参数使用的变量都分配给一个通用变量，如⑦所示，以便更容易引用它们（不需要在每个使用位置重新评估它们）。

现在，如果启动脚本有不同的名称，此脚本将表现不同。这允许创建它的符号链接：

```sh
# ln -s dummy /usr/local/etc/rc.d/dummy_foo
# sysrc dummy_foo_enable=YES
# service dummy_foo start
```

上述命令创建了一个名为 `dummy_foo` 的 dummy 服务实例。它不会使用配置文件 **/usr/local/etc/dummy.cfg**，而是使用配置文件 **/usr/local/etc/dummy_foo.cfg**（⑦），并且它使用 **/var/run/dummy/dummy_foo.pid** 作为 PID 文件，而不是 **/var/run/dummy/dummy.pid**。

`dummy` 和 `dummy_foo` 服务可以独立管理，而启动脚本会在包更新时自动更新（由于符号链接）。这不会更新 `REQUIRE` 行，因此没有简单的方法依赖特定的实例。为了在启动顺序中依赖特定实例，必须进行复制，而不是使用符号链接。这将防止在安装更新时自动获取启动脚本的更改。


## 11. 深入阅读

[Luke Mewburn 的原始文章](http://www.mewburn.net/luke/papers/rc.d.pdf) 提供了 **rc.d** 的概述，并详细阐述了其设计决策的理由。它对整个 **rc.d** 框架以及它在现代 BSD 操作系统中的位置提供了深刻的见解。

手册页 [rc(8)](https://man.freebsd.org/cgi/man.cgi?query=rc&sektion=8&format=html)、[rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 和 [rcorder(8)](https://man.freebsd.org/cgi/man.cgi?query=rcorder&sektion=8&format=html) 详细记录了 **rc.d** 组件。要充分利用 **rc.d** 的强大功能，必须阅读这些手册页，并在编写自己的脚本时参考它们。

**/etc/rc.d** 中的内容是工作中的真实示例，来自正在运行的系统，是最主要的学习资源。其内容简单易读，因为大多数棘手问题都隐藏在 [rc.subr(8)](https://man.freebsd.org/cgi/man.cgi?query=rc.subr&sektion=8&format=html) 中。然而请记住，**/etc/rc.d** 脚本并非由天使编写，因此它们可能存在缺陷和不理想的设计决策。现在，你可以改进它们！
