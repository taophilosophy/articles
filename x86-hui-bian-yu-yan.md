# x86 汇编语言

- 原文：[x86 Assembly Language Programming](https://docs.freebsd.org/en/articles/x86-assembly/)

_本文由 G. Adam Stanislav（2001）撰写，由 mhorne（2026）调整。_

> **注意**
> 本文内容具有历史参考价值。

## 1. 概要

在 UNIX® 下进行汇编语言编程的资料极为匮乏。人们通常认为没有人会愿意使用汇编语言，因为各种 UNIX 系统运行在不同的微处理器上，所以为了可移植性，一切都应该用 C 语言编写。

事实上，C 语言的可移植性不过是个神话。即使是 C 程序，从一个 UNIX 移植到另一个 UNIX 时也需要修改，无论它们各自运行在什么处理器上。通常，这样的程序中充斥着依赖于编译目标的条件语句。

即便我们相信所有的 UNIX 软件都应该用 C 或其他高级语言编写，我们仍然需要汇编语言程序员：还有谁会编写 C 库中访问内核的那部分代码呢？

在本文中，我将尝试向你展示如何使用汇编语言编写 UNIX 程序，特别是在 FreeBSD 下。

本文不讲解汇编语言的基础知识。这方面的资源已经足够多了（关于汇编语言的完整在线课程，可参阅 Randall Hyde 的 [The Art of Assembly Language](http://webster.cs.ucr.edu/)；如果你更喜欢印刷书籍，可以看看 Jeff Duntemann 的《Assembly Language Step-by-Step》（ISBN: 0471375233）。不过，在阅读完本文后，任何汇编语言程序员都能快速高效地为 FreeBSD 编写程序。

版权所有 © 2000-2001 G. Adam Stanislav。保留所有权利。

## 2. 工具

### 汇编器

汇编语言编程最重要的工具是汇编器，即将汇编语言代码转换为机器语言的软件。FreeBSD 提供了两种截然不同的汇编器。

第一种是 GNU `as(1)`（`devel/binutils`），它使用传统的 UNIX 汇编语言语法。此外，也可以使用 `clang(1)`，它是 GNU 汇编器的兼容替代品，随系统附带。

另一种是 `nasm(1)`（`devel/nasm`）。这种汇编器使用 Intel 语法，其主要优势是可以为多种操作系统汇编代码。

本文使用 nasm 语法，因为大多数从其他操作系统转向 FreeBSD 的汇编语言程序员会发现它更容易理解。而且，坦率地说，这也是我所习惯的语法。

### 链接器

汇编器的输出与任何编译器的输出一样，需要经过链接才能形成可执行文件。

FreeBSD 自带标准的 `ld(1)` 链接器。它适用于任一汇编器汇编的代码。

## 3. 系统调用

### 默认调用约定

默认情况下，FreeBSD 内核使用 C 调用约定。此外，虽然内核通过 `int 80h` 访问，但约定假设程序会调用一个发出 `int 80h` 的函数，而不是直接发出 `int 80h`。

这种约定非常方便，且远优于 MS-DOS® 所使用的 Microsoft® 约定。为什么？因为 UNIX 约定允许用任何语言编写的任何程序访问内核。

汇编语言程序同样可以做到这一点。例如，我们可以打开一个文件：

```asm
kernel:
	int	80h	; 调用内核
	ret

open:
	push	dword mode
	push	dword flags
	push	dword path
	mov	eax, 5
	call	kernel
	add	esp, byte 12
	ret
```

这是一种非常简洁且可移植的编码方式。如果你需要将代码移植到一个使用不同中断或不同参数传递方式的 UNIX 系统，只需修改 kernel 过程即可。

但是汇编语言程序员喜欢节省时钟周期。上面的例子需要 `call/ret` 组合。我们可以通过额外 `push` 一个 dword 来消除它：

```asm
open:
	push	dword mode
	push	dword flags
	push	dword path
	mov	eax, 5
	push	eax		; 或任何其他 dword
	int	80h
	add	esp, byte 16
```

我们放入 `EAX` 中的 `5` 标识了内核函数，这里是 `open`。

### 备用调用约定

FreeBSD 是一个极其灵活的系统。它还提供了其他调用内核的方式。不过，要使其生效，系统必须安装 Linux® 模拟。

Linux 是一个类 UNIX 系统。然而，它的内核使用与 MS-DOS 相同的寄存器传参的系统调用约定。与 UNIX 约定一样，函数编号放在 `EAX` 中。但参数不是在栈上传递，而是在 `EBX, ECX, EDX, ESI, EDI, EBP` 中：

```asm
open:
	mov	eax, 5
	mov	ebx, path
	mov	ecx, flags
	mov	edx, mode
	int	80h
```

就汇编语言编程而言，这种约定相比 UNIX® 方式有一个很大的缺点：每次进行内核调用时，你必须 `push` 寄存器，然后再 `pop` 它们。这会使你的代码更臃肿、更慢。尽管如此，FreeBSD 还是给了你选择的权利。

如果你确实选择了 Linux 约定，必须让系统知道这一点。在程序汇编和链接完成后，你需要为可执行文件打上标记：

```sh
% brandelf -t Linux filename
```

### 应使用哪种约定？

如果你专门为 FreeBSD 编程，应该始终使用 UNIX 约定：它更快，你可以将全局变量存储在寄存器中，不必为可执行文件打标记，也不会在目标系统上强制安装 Linux 模拟包。

如果你想创建也能在 Linux 上运行的可移植代码，你可能仍然希望为 FreeBSD 用户提供尽可能高效的代码。我将在讲解完基础知识后向你展示如何实现这一点。

### 调用号

要告诉内核你正在调用哪个系统服务，请将其编号放入 `EAX`。当然，你需要知道编号是多少。

#### syscalls 文件

这些编号列在 `syscalls` 中。`locate syscalls` 会以几种不同的格式找到该文件，它们都是由 `syscalls.master` 自动生成的。

你可以在 **/usr/src/sys/kern/syscalls.master** 中找到默认 UNIX 调用约定所用的主文件。如果需要使用 Linux 模拟模式中实现的另一种约定，请阅读 **/usr/src/sys/i386/linux/syscalls.master**。

> **注意**
> FreeBSD 和 Linux 不仅使用不同的调用约定，有时对相同函数也使用不同的编号。

`syscalls.master` 描述了如何进行调用：

```asm
0	STD	NOHIDE	{ int nosys(void); } syscall nosys_args int
1	STD	NOHIDE	{ void exit(int rval); } exit rexit_args void
2	STD	POSIX	{ int fork(void); }
3	STD	POSIX	{ ssize_t read(int fd, void *buf, size_t nbyte); }
4	STD	POSIX	{ ssize_t write(int fd, const void *buf, size_t nbyte); }
5	STD	POSIX	{ int open(char *path, int flags, int mode); }
6	STD	POSIX	{ int close(int fd); }
etc...
```

最左列告诉我们放入 `EAX` 的编号。

最右列告诉我们需要 `push` 哪些参数。它们是 _从右到左_ `push` 的。

例如，要 `open` 一个文件，我们需要先 `push` `mode`，然后是 `flags`，最后是存储 `path` 的地址。

## 4. 返回值

如果系统调用不返回某种值，大多数时候它是无用的：打开文件的文件描述符、读到缓冲区的字节数、系统时间等。

此外，系统还需要在发生错误时通知我们：文件不存在、系统资源耗尽、我们传递了无效参数等。

### 手册页

在 UNIX 系统中查找各种系统调用信息的传统去处是手册页。FreeBSD 在第 2 节（有时在第 3 节）描述其系统调用。

例如，`open(2)` 中写道：

> 如果成功，`open()` 返回一个非负整数，称为文件描述符。失败时返回 `-1`，并设置 `errno` 以指示错误。

刚接触 UNIX 和 FreeBSD 的汇编语言程序员会立即提出一个令人困惑的问题：`errno` 在哪里，我如何获取它？

> **注意**
> 手册页中的信息适用于 C 程序。汇编语言程序员需要额外的信息。

### 返回值在哪里？

不幸的是，这要看情况……对于大多数系统调用，它在 `EAX` 中，但并非全部。第一次使用某个系统调用时的一个好经验法则是先在 `EAX` 中查找返回值。如果不在那里，你需要进一步研究。

> **注意**
> 我知道有一个系统调用在 `EDX` 中返回值：`SYS_fork`。我使用过的所有其他系统调用都用 `EAX`。但我还没有全部用过。

> **技巧**
> 如果你在这里或其他地方找不到答案，请研究 libc 的源代码，看看它是如何与内核交互的。

### errno 在哪里？

实际上，哪里都没有……

`errno` 是 C 语言的一部分，而非 UNIX 内核的一部分。当直接访问内核服务时，错误码返回在 `EAX` 中，与正常返回值通常所在的寄存器相同。

这完全合理。如果没有错误，就没有错误码。如果有错误，就没有返回值。一个寄存器可以容纳二者之一。

### 判断是否发生错误

使用标准 FreeBSD 调用约定时，成功时清除 `carry flag`，失败时设置它。

使用 Linux 模拟模式时，`EAX` 中的有符号值在成功时为非负，并包含返回值。出错时，该值为负，即 `-errno`。

## 5. 创建可移植代码

可移植性通常不是汇编语言的强项。然而，为不同平台编写汇编语言程序是可能的，特别是使用 nasm。我曾编写过汇编语言库，可以为 Windows® 和 FreeBSD 等截然不同的操作系统汇编。

当你希望代码运行在两个虽然不同但基于相似架构的平台上时，这更加可行。

例如，FreeBSD 是 UNIX，Linux 是类 UNIX。我只提到了它们之间的三个差异（从汇编语言程序员的角度）：调用约定、函数编号和返回值方式。

### 处理函数编号

在很多情况下，函数编号是相同的。然而，即使不同，这个问题也很容易解决：不要在代码中使用数字，而是使用常量，根据目标架构以不同方式声明：

```asm
%ifdef	LINUX
%define	SYS_execve	11
%else
%define	SYS_execve	59
%endif
```

### 处理调用约定

调用约定和返回值（`errno` 问题）都可以通过宏来解决：

```asm
%ifdef	LINUX

%macro	system	0
	call	kernel
%endmacro

align 4
kernel:
	push	ebx
	push	ecx
	push	edx
	push	esi
	push	edi
	push	ebp

	mov	ebx, [esp+32]
	mov	ecx, [esp+36]
	mov	edx, [esp+40]
	mov	esi, [esp+44]
	mov	ebp, [esp+48]
	int	80h

	pop	ebp
	pop	edi
	pop	esi
	pop	edx
	pop	ecx
	pop	ebx

	or	eax, eax
	js	.errno
	clc
	ret

.errno:
	neg	eax
	stc
	ret

%else

%macro	system	0
	int	80h
%endmacro

%endif
```

### 处理其他可移植性问题

上述解决方案可以处理在 FreeBSD 和 Linux 之间编写可移植代码的大多数情况。然而，对于某些内核服务，差异更深。

在这种情况下，你需要为这些特定的系统调用编写两个不同的处理程序，并使用条件汇编。幸运的是，你的大部分代码做的不是调用内核，所以通常只需要少量这样的条件代码段。

### 使用库

你可以通过编写系统调用库来完全避免主代码中的可移植性问题。为 FreeBSD 创建一个单独的库，为 Linux 创建另一个，为其他操作系统再创建更多库。

在你的库中，为每个系统调用编写单独的函数（如果你更喜欢传统的汇编语言术语，叫过程）。使用 C 调用约定传递参数。但仍然使用 `EAX` 传递调用号。在这种情况下，你的 FreeBSD 库可以非常简单，因为许多看似不同的函数可能只是指向相同代码的标签：

```asm
sys.open:
sys.close:
[etc...]
	int	80h
	ret
```

你的 Linux 库将需要更多不同的函数。但即便在这里，你也可以按相同参数数量的系统调用分组：

```asm
sys.exit:
sys.close:
[etc... 单参数函数]
	push	ebx
	mov	ebx, [esp+12]
	int	80h
	pop	ebx
	jmp	sys.return

...

sys.return:
	or	eax, eax
	js	sys.err
	clc
	ret

sys.err:
	neg	eax
	stc
	ret
```

库方法乍看可能不太方便，因为它需要你生成一个代码所依赖的单独文件。但它有很多优势：首先，你只需编写一次，就可以用于所有程序。你甚至可以让其他汇编语言程序员使用它，或者使用别人编写的库。但库最大的优势或许是，你的代码可以移植到其他系统——甚至可以由其他程序员移植——只需编写新的库而无需修改你的代码。

如果你不喜欢使用库的想法，至少可以将所有系统调用放在一个单独的汇编语言文件中，并将其与主程序链接。同样，移植者只需创建一个新的目标文件与主程序链接即可。

### 使用包含文件

如果你以源代码形式发布软件（或随软件一起发布源代码），你可以使用宏并将它们放在单独的文件中，然后将其包含到你的代码里。

你的软件移植者只需编写新的包含文件。不需要库或外部目标文件，而你的代码无需编辑即可移植。

> **注意**
> 这是我们在本文中将使用的方法。我们将包含文件命名为 `system.inc`，并在处理新的系统调用时向其添加内容。

我们可以从声明标准文件描述符开始编写 `system.inc`：

```asm
%define	stdin	0
%define	stdout	1
%define	stderr	2
```

接下来，我们为每个系统调用创建一个符号名称：

```asm
%define	SYS_nosys	0
%define	SYS_exit	1
%define	SYS_fork	2
%define	SYS_read	3
%define	SYS_write	4
; [etc...]
```

我们添加一个简短的非全局过程，名字很长，这样就不会在代码中意外重用该名称：

```asm
section	.text
align 4
access.the.bsd.kernel:
	int	80h
	ret
```

我们创建一个接受一个参数（系统调用编号）的宏：

```asm
%macro	system	1
	mov	eax, %1
	call	access.the.bsd.kernel
%endmacro
```

最后，我们为每个系统调用创建宏。这些宏不接受参数。

```asm
%macro	sys.exit	0
	system	SYS_exit
%endmacro

%macro	sys.fork	0
	system	SYS_fork
%endmacro

%macro	sys.read	0
	system	SYS_read
%endmacro

%macro	sys.write	0
	system	SYS_write
%endmacro

; [etc...]
```

动手吧，将它输入编辑器并保存为 `system.inc`。随着我们讨论更多系统调用，我们将向其中添加更多内容。

## 6. 我们的第一个程序

我们现在可以编写第一个程序了——必不可少的 Hello, World!

```asm
	%include	'system.inc'

	section	.data
	hello	db	'Hello, World!', 0Ah
	hbytes	equ	$-hello

	section	.text
	global	_start
_start:
	push	dword hbytes
	push	dword hello
	push	dword stdout
	sys.write

	push	dword 0
	sys.exit
```

下面是它的工作原理：第 1 行包含 `system.inc` 中的定义、宏和代码。

第 3-5 行是数据：第 3 行开始数据段。第 4 行包含字符串 "Hello, World!"，后跟一个换行符（`0Ah`）。第 5 行创建一个常量，包含第 4 行字符串的字节长度。

第 7-16 行包含代码。注意，FreeBSD 对其可执行文件使用 _elf_ 文件格式，要求每个程序从标记为 `_start` 的位置开始（更准确地说，链接器期望如此）。此标签必须是全局的。

第 10-13 行请求系统将 `hello` 字符串的 `hbytes` 个字节写入 `stdout`。

第 15-16 行请求系统以返回值 `0` 结束程序。`SYS_exit` 系统调用从不返回，所以代码到此结束。

> **注意**
> 如果你是从 MS-DOS 汇编语言背景转到 UNIX 的，你可能习惯于直接写入视频硬件。在 FreeBSD 或任何其他 UNIX 变体中，你永远不必担心这一点。就你而言，你是在写入一个名为 `stdout` 的文件。它可能是视频屏幕、telnet 终端、实际文件，甚至是另一个程序的输入。具体是哪一个，由系统来判断。

### 汇编代码

在编辑器中输入代码，并将其保存在名为 `hello.asm` 的文件中。你需要 `devel/nasm` 来汇编它。

现在你可以汇编、链接并运行代码：

```sh
% nasm -f elf hello.asm
% ld -s -o hello hello.o
% ./hello
Hello, World!
%
```

## 7. 编写 UNIX 过滤器

UNIX 应用程序的一种常见类型是过滤器——一种从 `stdin` 读取数据，以某种方式处理，然后将结果写入 `stdout` 的程序。

在本章中，我们将开发一个简单的过滤器，并学习如何从 `stdin` 读取和写入 `stdout`。此过滤器将其输入的每个字节转换为后跟一个空格的十六进制数字。

```asm
%include	'system.inc'

section	.data
hex	db	'0123456789ABCDEF'
buffer	db	0, 0, ' '

section	.text
global	_start
_start:
	; 从 stdin 读取一个字节
	push	dword 1
	push	dword buffer
	push	dword stdin
	sys.read
	add	esp, byte 12
	or	eax, eax
	je	.done

	; 转换为十六进制
	movzx	eax, byte [buffer]
	mov	edx, eax
	shr	dl, 4
	mov	dl, [hex+edx]
	mov	[buffer], dl
	and	al, 0Fh
	mov	al, [hex+eax]
	mov	[buffer+1], al

	; 打印输出
	push	dword 3
	push	dword buffer
	push	dword stdout
	sys.write
	add	esp, byte 12
	jmp	short _start

.done:
	push	dword 0
	sys.exit
```

在数据段中，我们创建了一个名为 `hex` 的数组。它按升序包含 16 个十六进制数字。数组后面是一个缓冲区，我们将同时用于输入和输出。缓冲区的前两个字节初始设置为 `0`。这是我们将写入两个十六进制数字的地方（第一个字节也是我们读取输入的地方）。第三个字节是空格。

代码段由四部分组成：读取字节、将其转换为十六进制数字、写入结果，最终退出程序。

要读取字节，我们请求系统从 `stdin` 读取一个字节，并将其存储在 `buffer` 的第一个字节中。系统在 `EAX` 中返回读取的字节数。当有数据时，它为 `1`；当没有更多输入数据可用时，它为 `0`。因此，我们检查 `EAX` 的值。如果为 `0`，我们跳到 `.done`，否则继续。

> **注意**
> 为简单起见，我们此时忽略了错误条件的可能性。

十六进制转换从 `buffer` 中读取字节到 `EAX`，或者实际上只是 `AL`，同时将 `EAX` 的其余位清零。我们还将字节复制到 `EDX`，因为我们需要分别转换高四位（半字节）和低四位。我们将结果存储在缓冲区的前两个字节中。

接下来，我们请求系统将缓冲区的三个字节（即两个十六进制数字和空格）写入 `stdout`。然后我们跳回程序开头，处理下一个字节。

一旦没有更多输入，我们请求系统退出程序，返回零，这是表示程序成功的传统值。

动手吧，将代码保存在名为 `hex.asm` 的文件中，然后输入以下命令（`^D` 表示按住 control 键的同时按 `D`）：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A ^D %
```

> **注意**
> 如果你正在从 MS-DOS 迁移到 UNIX，你可能想知道为什么每行以 `0A` 而非 `0D 0A` 结尾。这是因为 UNIX 不使用 cr/lf 约定，而是使用"换行"约定，即十六进制的 `0A`。

我们能改进它吗？首先，它有点令人困惑，因为一旦我们转换了一行文本，输入就不再从行首开始了。我们可以修改它，在每个 `0A` 之后打印一个换行而非空格：

```asm
%include	'system.inc'

section	.data
hex	db	'0123456789ABCDEF'
buffer	db	0, 0, ' '

section	.text
global	_start
_start:
	mov	cl, ' '

.loop:
	; 从 stdin 读取一个字节
	push	dword 1
	push	dword buffer
	push	dword stdin
	sys.read
	add	esp, byte 12
	or	eax, eax
	je	.done

	; 转换为十六进制
	movzx	eax, byte [buffer]
	mov	[buffer+2], cl
	cmp	al, 0Ah
	jne	.hex
	mov	[buffer+2], al

.hex:
	mov	edx, eax
	shr	dl, 4
	mov	dl, [hex+edx]
	mov	[buffer], dl
	and	al, 0Fh
	mov	al, [hex+eax]
	mov	[buffer+1], al

	; 打印输出
	push	dword 3
	push	dword buffer
	push	dword stdout
	sys.write
	add	esp, byte 12
	jmp	short .loop

.done:
	push	dword 0
	sys.exit
```

我们将空格存储在 `CL` 寄存器中。这样做是安全的，因为与 Microsoft Windows 不同，UNIX 系统调用不会修改它不用于返回值的任何寄存器。

这意味着我们只需设置 `CL` 一次。因此，我们添加了一个新标签 `.loop`，并为下一个字节跳转到它，而不是跳到 `_start`。我们还添加了 `.hex` 标签，这样 `buffer` 的第三个字节可以是空格或换行。

修改 `hex.asm` 以反映这些更改后，输入：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

看起来好多了。但这段代码效率很低！我们为每个字节做两次系统调用（一次读取，一次写入输出）。

## 8. 缓冲输入与输出

我们可以通过缓冲输入和输出来提高代码效率。我们创建一个输入缓冲区，一次读取一整串字节。然后从缓冲区中逐一取出它们。

我们还创建一个输出缓冲区。我们将输出存储在其中，直到它满。那时我们请求内核将缓冲区的内容写入 `stdout`。

当没有更多输入时程序结束。但我们仍然需要请求内核最后一次将输出缓冲区的内容写入 `stdout`，否则部分输出会进入输出缓冲区但永远不会被发出。不要忘记这一点，否则你会疑惑为什么部分输出不见了。

```asm
%include	'system.inc'

%define	BUFSIZE	2048

section	.data
hex	db	'0123456789ABCDEF'

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
global	_start
_start:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

.loop:
	; 从 stdin 读取一个字节
	call	getchar

	; 转换为十六进制
	mov	dl, al
	shr	al, 4
	mov	al, [hex+eax]
	call	putchar

	mov	al, dl
	and	al, 0Fh
	mov	al, [hex+eax]
	call	putchar

	mov	al, ' '
	cmp	dl, 0Ah
	jne	.put
	mov	al, dl

.put:
	call	putchar
	jmp	short .loop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword stdin
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; 刷新输出缓冲区
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	sub	edi, ecx	; 缓冲区起始位置
	push	ecx
	push	edi
	push	dword stdout
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在为空
	ret
```

我们现在在源代码中有了第三个段，名为 `.bss`。这个段不包含在我们的可执行文件中，因此无法初始化。我们使用 `resb` 而非 `db`。它只是为我们保留所需大小的未初始化内存。

我们利用了系统不修改寄存器这一事实：我们将寄存器用于原本必须存储在 `.data` 段中的全局变量。这也是 UNIX 通过栈向系统调用传递参数的约定优于 Microsoft 通过寄存器传递的约定的原因：我们可以将寄存器留作己用。

我们使用 `EDI` 和 `ESI` 作为指向下一个要读取或写入字节的指针。我们使用 `EBX` 和 `ECX` 来记录两个缓冲区中的字节数，这样我们就知道何时将输出转储到系统或从系统读取更多输入。

让我们看看现在它是如何工作的：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
Here I come!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

不是你期望的结果？程序直到我们按下 `^D` 才打印输出。这很容易修复，只需插入三行代码，在我们每次将换行符转换为 `0A` 时写入输出。我用 > 标记了这三行（不要在你的 `hex.asm` 中复制 >）。

```asm
%include	'system.inc'

%define	BUFSIZE	2048

section	.data
hex	db	'0123456789ABCDEF'

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
global	_start
_start:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

.loop:
	; 从 stdin 读取一个字节
	call	getchar

	; 转换为十六进制
	mov	dl, al
	shr	al, 4
	mov	al, [hex+eax]
	call	putchar

	mov	al, dl
	and	al, 0Fh
	mov	al, [hex+eax]
	call	putchar

	mov	al, ' '
	cmp	dl, 0Ah
	jne	.put
	mov	al, dl

.put:
	call	putchar
>	cmp	al, 0Ah
>	jne	.loop
>	call	write
	jmp	short .loop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword stdin
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; 刷新输出缓冲区
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	sub	edi, ecx	; 缓冲区起始位置
	push	ecx
	push	edi
	push	dword stdout
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在为空
	ret
```

现在，让我们看看它是如何工作的：

```sh
% nasm -f elf hex.asm
% ld -s -o hex hex.o
% ./hex
Hello, World!
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21 0A
Here I come!
48 65 72 65 20 49 20 63 6F 6D 65 21 0A
^D %
```

对于一个 644 字节的可执行文件来说，还不错吧！

> **注意**
> 这种缓冲输入/输出方法仍然存在一个隐患。我稍后在讨论[缓冲的阴暗面](#缓冲的阴暗面)时会谈到并修复它。

### 如何退回字符

> **警告**
> 这可能是一个稍微高级的主题，主要对熟悉编译器理论的程序员感兴趣。如果你愿意，可以[跳到下一节](#命令行参数)，以后再阅读这部分。

虽然我们的示例程序不需要它，但更复杂的过滤器通常需要向前看。换句话说，它们可能需要查看下一个字符是什么（甚至多个字符）。如果下一个字符是某个值，它就是当前正在处理的标记的一部分。否则就不是。

例如，你可能正在解析输入流中的文本字符串（例如，在实现语言编译器时）：如果一个字符后面跟着另一个字符或数字，它就是你正在处理的标记的一部分。如果后面跟着空白或其他值，它就不是当前标记的一部分。

这带来了一个有趣的问题：如何将下一个字符返回到输入流中，以便以后可以再次读取它？

一种可能的解决方案是将它存储在字符变量中，然后设置一个标志。我们可以修改 `getchar` 来检查该标志，如果已设置，则从该变量而非输入缓冲区中取字节，并重置标志。但当然，这会拖慢我们。

C 语言有一个 `ungetc()` 函数正是为此目的而设计。有没有快速的方法在我们的代码中实现它？我想请你向上滚动查看 `getchar` 过程，看看能否在阅读下一段之前找到一个又好又快的解决方案。然后回到这里看看我自己的解决方案。

将字符返回到流的关键在于我们如何获取字符：

首先通过测试 `EBX` 的值来检查缓冲区是否为空。如果为零，我们调用 `read` 过程。

如果我们确实有可用字符，我们使用 `lodsb`，然后递减 `EBX` 的值。`lodsb` 指令实际上等同于：

```asm
mov	al, [esi]
	inc	esi
```

我们取出的字节保留在缓冲区中，直到下次调用 `read`。我们不知道那是什么时候，但我们知道它不会在下次调用 `getchar` 之前发生。因此，要"返回"最后读取的字节到流中，我们要做的就是递减 `ESI` 的值并递增 `EBX` 的值：

```asm
ungetc:
	dec	esi
	inc	ebx
	ret
```

但要小心！如果我们每次最多向前看一个字符，这样做是完全安全的。如果我们检查多个即将到来的字符并连续多次调用 `ungetc`，大多数时候它会工作，但并非总是如此（而且很难调试）。为什么？

因为只要 `getchar` 不必调用 `read`，所有预读的字节仍在缓冲区中，我们的 `ungetc` 就能完美工作。但一旦 `getchar` 调用 `read`，缓冲区的内容就会改变。

我们总是可以依赖 `ungetc` 正确处理用 `getchar` 读取的最后一个字符，但不能处理在此之前读取的任何字符。

如果你的程序向前读取多个字节，你至少有两个选择：

如果可能，修改程序使其只向前读取一个字节。这是最简单的解决方案。

如果该选项不可行，首先确定你的程序一次需要返回到输入流的最大字符数。将该数字稍微增大一点以防万一，最好是 16 的倍数——这样对齐更好。然后修改代码的 `.bss` 段，在输入缓冲区之前创建一个小的"备用"缓冲区，如下所示：

```asm
section	.bss
	resb	16	; 或你算出的任何值
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE
```

你还需要修改 `ungetc`，以在 `AL` 中传递要退回的字节值：

```asm
ungetc:
	dec	esi
	inc	ebx
	mov	[esi], al
	ret
```

经过此修改，你可以安全地连续调用 `ungetc` 最多 17 次（第一次调用仍在缓冲区内，其余 16 次可能在缓冲区内或在"备用"区域中）。

## 9. 命令行参数

如果我们的 hex 程序能从命令行读取输入和输出文件的名称，即能处理命令行参数，它将更有用。但是……它们在哪里？

在 UNIX 系统启动程序之前，它会将一些数据 `push` 到栈上，然后跳转到程序的 `_start` 标签。是的，我说的是跳转，不是调用。这意味着可以通过读取 `[esp+offset]` 或简单地 `pop` 来访问数据。

栈顶的值包含命令行参数的数量。按传统它被称为 `argc`，即"argument count"（参数计数）。

接下来是命令行参数，共 `argc` 个。这些通常被称为 `argv`，即"argument value(s)"（参数值）。也就是说，我们得到 `argv[0]`、`argv[1]`、`...`、`argv[argc-1]`。这些不是实际的参数，而是指向参数的指针，即实际参数的内存地址。参数本身是以 NUL 结尾的字符串。

`argv` 列表后面跟着一个 NULL 指针，它就是一个 `0`。后面还有更多内容，但这对我们目前的目的来说已经足够了。

> **注意**
> 如果你来自 MS-DOS 编程环境，主要区别在于每个参数都在单独的字符串中。第二个区别是参数数量没有实际限制。

有了这些知识，我们几乎可以开始编写下一版 `hex.asm` 了。不过首先，我们需要在 `system.inc` 中添加几行：

首先，我们需要在系统调用编号列表中添加两个新条目：

```asm
%define	SYS_open	5
%define	SYS_close	6
```

然后我们在文件末尾添加两个新宏：

```asm
%macro	sys.open	0
	system	SYS_open
%endmacro

%macro	sys.close	0
	system	SYS_close
%endmacro
```

下面是我们修改后的源代码：

```asm
%include	'system.inc'

%define	BUFSIZE	2048

section	.data
fd.in	dd	stdin
fd.out	dd	stdout
hex	db	'0123456789ABCDEF'

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
align 4
err:
	push	dword 1		; 返回失败
	sys.exit

align 4
global	_start
_start:
	add	esp, byte 8	; 丢弃 argc 和 argv[0]

	pop	ecx
	jecxz	.init		; 没有更多参数

	; ECX 包含输入文件路径
	push	dword 0		; O_RDONLY
	push	ecx
	sys.open
	jc	err		; 打开失败

	add	esp, byte 8
	mov	[fd.in], eax

	pop	ecx
	jecxz	.init		; 没有更多参数

	; ECX 包含输出文件路径
	push	dword 420	; 文件模式（八进制 644）
	push	dword 0200h | 0400h | 01h
	; O_CREAT | O_TRUNC | O_WRONLY
	push	ecx
	sys.open
	jc	err

	add	esp, byte 12
	mov	[fd.out], eax

.init:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

.loop:
	; 从输入文件或 stdin 读取一个字节
	call	getchar

	; 转换为十六进制
	mov	dl, al
	shr	al, 4
	mov	al, [hex+eax]
	call	putchar

	mov	al, dl
	and	al, 0Fh
	mov	al, [hex+eax]
	call	putchar

	mov	al, ' '
	cmp	dl, 0Ah
	jne	.put
	mov	al, dl

.put:
	call	putchar
	cmp	al, dl
	jne	.loop
	call	write
	jmp	short .loop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword [fd.in]
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; 刷新输出缓冲区

	; 关闭文件
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	; 返回成功
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	sub	edi, ecx	; 缓冲区起始位置
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在为空
	ret
```

在我们的 `.data` 段中现在有两个新变量 `fd.in` 和 `fd.out`。我们在这里存储输入和输出文件描述符。

在 `.text` 段中，我们用 `[fd.in]` 和 `[fd.out]` 替换了对 `stdin` 和 `stdout` 的引用。

`.text` 段现在以一个简单的错误处理程序开始，它除了以返回值 `1` 退出程序外什么也不做。错误处理程序位于 `_start` 之前，这样我们就与错误发生的地方距离很近。

当然，程序执行仍然从 `_start` 开始。首先，我们从栈中移除 `argc` 和 `argv[0]`：它们对我们没有用处（至少在这个程序中如此）。

我们将 `argv[1]` 弹出到 `ECX`。这个寄存器特别适合用于指针，因为我们可以用 `jecxz` 处理 NULL 指针。如果 `argv[1]` 不为 NULL，我们尝试打开第一个参数中指定的文件。否则，我们像之前一样继续程序：从 `stdin` 读取，写入 `stdout`。如果打开输入文件失败（例如它不存在），我们跳到错误处理程序并退出。

如果一切顺利，我们现在检查第二个参数。如果存在，我们打开输出文件。否则，我们将输出发送到 `stdout`。如果打开输出文件失败（例如它存在而我们没有写权限），我们同样跳到错误处理程序。

其余代码与之前相同，只是我们在退出前关闭输入和输出文件，而且如前所述，我们使用 `[fd.in]` 和 `[fd.out]`。

我们的可执行文件现在长达 768 字节。

还能改进吗？当然！每个程序都可以改进。以下是一些我们可以做的事情的思路：

* 让错误处理程序向 `stderr` 打印消息。
* 为 `read` 和 `write` 函数添加错误处理程序。
* 打开输入文件时关闭 `stdin`，打开输出文件时关闭 `stdout`。
* 添加命令行开关，如 `-i` 和 `-o`，这样我们可以以任意顺序列出输入和输出文件，或者从 `stdin` 读取并写入文件。
* 如果命令行参数不正确，打印用法消息。

我将把这些增强功能留作读者的练习：你已经具备了实现它们所需的所有知识。

## 10. UNIX 环境

一个重要的 UNIX 概念是环境，它由 _环境变量_ 定义。有些由系统设置，有些由你设置，还有些由 shell 或任何加载其他程序的程序设置。

### 如何查找环境变量

我之前说过，当程序开始执行时，栈上包含 `argc`，其后是以 NULL 结尾的 `argv` 数组，再后面还有其他内容。那"其他内容"就是 _环境_，更准确地说，是一个以 NULL 结尾的、指向 _环境变量_ 的指针数组。这通常被称为 `env`。

`env` 的结构与 `argv` 相同，是一个内存地址列表，后跟一个 NULL（`0`）。在这种情况下，没有所谓的 `"envc"`——我们通过搜索最后的 NULL 来确定数组在哪里结束。

变量通常以 `name=value` 格式出现，但有时 `=value` 部分可能缺失。我们需要考虑到这种可能性。

### webvars

我本可以直接向你展示一些以 UNIX env 命令相同方式打印环境的代码。但我认为编写一个简单的汇编语言 CGI 工具会更有趣。

#### CGI：快速概览

我在我的网站上有一份[详细的 CGI 教程](https://web.archive.org/web/20230612082452/http://www.whizkidtech.redprince.net/cgi-bin/tutorial)，但这里是对 CGI 的快速概览：

* Web 服务器通过设置 _环境变量_ 与 CGI 程序通信。
* CGI 程序将其输出发送到 `stdout`。Web 服务器从那里读取它。
* 输出必须以 HTTP 头开始，后跟两个空行。
* 然后打印 HTML 代码或它产生的任何其他类型的数据。

> **注意**
> 虽然某些 _环境变量_ 使用标准名称，但其他的会因 Web 服务器而异。这使得 webvars 成为一个相当有用的诊断工具。

#### 代码

那么，我们的 webvars 程序必须先发送 HTTP 头，然后是一些 HTML 标记。接着它必须逐一读取 _环境变量_ 并将它们作为 HTML 页面的一部分发送出去。

代码如下。我将注释和说明直接放在代码中：

```asm
;;;;;;; webvars.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Copyright (c) 2000 G. Adam Stanislav
; All rights reserved.
;
; Redistribution and use in source and binary forms, with or without
; modification, are permitted provided that the following conditions
; are met:
; 1. Redistributions of source code must retain the above copyright
;    notice, this list of conditions and the following disclaimer.
; 2. Redistributions in binary form must reproduce the above copyright
;    notice, this list of conditions and the following disclaimer in the
;    documentation and/or other materials provided with the distribution.
;
; THIS SOFTWARE IS PROVIDED BY THE AUTHOR AND CONTRIBUTORS ``AS IS'' AND
; ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
; IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
; ARE DISCLAIMED.  IN NO EVENT SHALL THE AUTHOR OR CONTRIBUTORS BE LIABLE
; FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
; DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
; OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
; HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
; LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
; OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
; SUCH DAMAGE.
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Version 1.0
;
; Started:	 8-Dec-2000
; Updated:	 8-Dec-2000
;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
%include	'system.inc'

section	.data
http	db	'Content-type: text/html', 0Ah, 0Ah
	db	'<?xml version="1.0" encoding="utf-8"?>', 0Ah
	db	'<!DOCTYPE html PUBLIC "-//W3C/DTD XHTML Strict//EN" '
	db	'"DTD/xhtml1-strict.dtd">', 0Ah
	db	'<html xmlns="http://www.w3.org/1999/xhtml" '
	db	'xml.lang="en" lang="en">', 0Ah
	db	'<head>', 0Ah
	db	'<title>Web Environment</title>', 0Ah
	db	'<meta name="author" content="G. Adam Stanislav" />', 0Ah
	db	'</head>', 0Ah, 0Ah
	db	'<body bgcolor="#ffffff" text="#000000" link="#0000ff" '
	db	'vlink="#840084" alink="#0000ff">', 0Ah
	db	'<div class="webvars">', 0Ah
	db	'<h1>Web Environment</h1>', 0Ah
	db	'<p>The following <b>environment variables</b> are defined '
	db	'on this web server:</p>', 0Ah, 0Ah
	db	'<table align="center" width="80" border="0" cellpadding="10" '
	db	'cellspacing="0" class="webvars">', 0Ah
httplen	equ	$-http
left	db	'<tr>', 0Ah
	db	'<td class="name"><tt>'
leftlen	equ	$-left
middle	db	'</tt></td>', 0Ah
	db	'<td class="value"><tt><b>'
midlen	equ	$-middle
undef	db	'<i>(undefined)</i>'
undeflen	equ	$-undef
right	db	'</b></tt></td>', 0Ah
	db	'</tr>', 0Ah
rightlen	equ	$-right
wrap	db	'</table>', 0Ah
	db	'</div>', 0Ah
	db	'</body>', 0Ah
	db	'</html>', 0Ah, 0Ah
wraplen	equ	$-wrap

section	.text
global	_start
_start:
	; 首先，发送所有在我们开始显示环境之前
	; 所需的 http 和 xhtml 内容
	push	dword httplen
	push	dword http
	push	dword stdout
	sys.write

	; 现在找出栈上的环境指针有多远。
	; 我们在 "argc" 之前已经 push 了 12 字节
	mov	eax, [esp+12]

	; 我们需要从栈中移除以下内容：
	;
	;	为 sys.write push 的 12 字节
	;	argc 的 4 字节
	;	argv 的 EAX*4 字节
	;	argv 后面 NULL 的 4 字节
	;
	; 总计：
	;	20 + eax * 4
	;
	; 因为栈向下生长，我们需要向 ESP 加上
	; 那么多字节。
	lea	esp, [esp+20+eax*4]
	cld		; 这应该已经是这种情况了，但确认一下。

	; 循环遍历环境，打印输出
.loop:
	pop	edi
	or	edi, edi	; 完成了吗？
	je	near .wrap

	; 打印 HTML 的左半部分
	push	dword leftlen
	push	dword left
	push	dword stdout
	sys.write

	; 接下来可能很想在 env 字符串中搜索 '='。
	; 但可能没有 '='，所以我们先搜索
	; 终止符 NUL。
	mov	esi, edi	; 保存字符串起始位置
	sub	ecx, ecx
	not	ecx		; ECX = FFFFFFFF
	sub	eax, eax
repne	scasb
	not	ecx		; ECX = 字符串长度 + 1
	mov	ebx, ecx	; 保存在 EBX 中

	; 现在是查找 '=' 的时候了
	mov	edi, esi	; 字符串起始位置
	mov	al, '='
repne	scasb
	not	ecx
	add	ecx, ebx	; 名称长度

	push	ecx
	push	esi
	push	dword stdout
	sys.write

	; 打印 HTML 表格代码的中间部分
	push	dword midlen
	push	dword middle
	push	dword stdout
	sys.write

	; 查找值的长度
	not	ecx
	lea	ebx, [ebx+ecx-1]

	; 如果为 0 则打印 "undefined"
	or	ebx, ebx
	jne	.value

	mov	ebx, undeflen
	mov	edi, undef

.value:
	push	ebx
	push	edi
	push	dword stdout
	sys.write

	; 打印表格行的右半部分
	push	dword rightlen
	push	dword right
	push	dword stdout
	sys.write

	; 去掉我们 push 的 60 字节
	add	esp, byte 60

	; 获取下一个变量
	jmp	.loop

.wrap:
	; 打印 HTML 的剩余部分
	push	dword wraplen
	push	dword wrap
	push	dword stdout
	sys.write

	; 返回成功
	push	dword 0
	sys.exit
```

这段代码产生一个 1,396 字节的可执行文件。其中大部分是数据，即我们需要发送的 HTML 标记。

像往常一样汇编和链接它：

```sh
% nasm -f elf webvars.asm
% ld -s -o webvars webvars.o
```

要使用它，你需要将 `webvars` 上传到你的 Web 服务器。根据你的 Web 服务器的设置方式，你可能需要将其存储在特殊的 `cgi-bin` 目录中，或者用 `.cgi` 扩展名重命名它。

然后你需要使用浏览器查看其输出。要查看我的 Web 服务器上的输出，请访问 [http://www.int80h.org/webvars](https://web.archive.org/web/20100818031015/http://www.int80h.org/webvars/)。

## 11. 处理文件

我们已经做了一些基本的文件操作：我们知道如何打开和关闭文件，如何使用缓冲区读写文件。但 UNIX 在文件方面提供了更多功能。我们将在本节中考察其中一些，并以一个不错的文件转换工具结束。

确实，让我们从终点开始，即从文件转换工具开始。当我们从一开始就知道最终产品应该做什么时，编程总是会变得更容易。

我为 UNIX 编写的最早程序之一是 `tuc(1)`，一个文本到 UNIX 的文件转换器。它将文本文件从其他操作系统转换为 UNIX 文本文件。换句话说，它将不同类型的行尾更改为 UNIX 的换行约定。它将输出保存到不同的文件中。可选地，它可以将 UNIX 文本文件转换为 DOS 文本文件。

我大量使用 tuc，但始终只是从其他操作系统转换到 UNIX，从不反向。我一直希望它能直接覆盖文件，而不是让我把输出发送到另一个文件。大多数时候，我最终这样使用它：

```sh
% tuc myfile tempfile
% mv tempfile myfile
```

如果有一个 ftuc，即 _快速 tuc_，并这样使用它，那就好了：

```sh
% ftuc myfile
```

因此，在本文中，我们将用汇编语言编写 ftuc（原始的 tuc 是用 C 编写的），并在过程中研究各种面向文件的内核服务。

乍一看，这样的文件转换非常简单：你只需要去掉回车符，对吧？

如果你回答是，再想想：这种方法大多数时候有效（至少对 MS-DOS 文本文件而言），但偶尔会失败。

问题在于，并非所有非 UNIX 文本文件都以回车符/换行符序列结束行。有些使用不带换行符的回车符。其他一些将多个空行合并为一个回车符后跟多个换行符。等等。

因此，文本文件转换器必须能够处理任何可能的行尾：

* 回车符/换行符
* 回车符
* 换行符/回车符
* 换行符

它还应该处理使用上述某种组合的文件（例如，回车符后跟多个换行符）。

### 有限状态机

这个问题可以通过使用一种称为 _有限状态机_ 的技术轻松解决，该技术最初由数字电子电路设计者开发。_有限状态机_ 是一种数字电路，其输出不仅取决于输入，还取决于其先前的输入，即其状态。微处理器就是 _有限状态机_ 的一个例子：我们的汇编语言代码被汇编为机器语言，其中一些汇编语言代码产生单个字节的机器语言，而另一些则产生多个字节。当微处理器从内存中逐一取出字节时，其中一些只是改变其状态而非产生输出。当操作码的所有字节都被取出后，微处理器产生一些输出，或改变寄存器的值等。

因此，所有软件本质上都是微处理器的一系列状态指令。尽管如此，_有限状态机_ 的概念在软件设计中同样有用。

我们的文本文件转换器可以设计为一个具有三种可能状态的 _有限状态机_。我们可以称它们为状态 0-2，但如果我们给它们起符号名称，生活会更容易：

* ordinary
* cr
* lf

我们的程序将从 ordinary 状态开始。在此状态下，程序的操作取决于其输入，如下所示：

* 如果输入是回车符或换行符以外的任何内容，输入直接传递到输出。状态保持不变。
* 如果输入是回车符，状态变为 cr。然后输入被丢弃，即不产生输出。
* 如果输入是换行符，状态变为 lf。然后输入被丢弃。

每当我们处于 cr 状态时，是因为上一个输入是未处理的回车符。我们的软件在此状态下的操作同样取决于当前输入：

* 如果输入是回车符或换行符以外的任何内容，输出一个换行符，然后输出输入，然后将状态更改为 ordinary。
* 如果输入是回车符，我们已连续收到两个（或更多）回车符。我们丢弃输入，输出一个换行符，并保持状态不变。
* 如果输入是换行符，我们输出换行符并将状态更改为 ordinary。注意这与上面第一种情况不同——如果我们试图合并它们，我们会输出两个换行符而非一个。

最后，当我们收到前面没有回车符的换行符时，我们处于 lf 状态。当我们的文件已经是 UNIX 格式，或者当连续几行由一个回车符后跟多个换行符表示，或当行以换行符/回车符序列结束时，就会发生这种情况。以下是我们在此状态下需要处理输入的方式：

* 如果输入是回车符或换行符以外的任何内容，我们输出一个换行符，然后输出输入，然后将状态更改为 ordinary。这与 cr 状态下收到相同类型输入时的操作完全相同。
* 如果输入是回车符，我们丢弃输入，输出一个换行符，然后将状态更改为 ordinary。
* 如果输入是换行符，我们输出换行符，并保持状态不变。

#### 最终状态

上述 _有限状态机_ 适用于整个文件，但留下了最终行尾被忽略的可能性。每当文件以单个回车符或单个换行符结束时就会发生这种情况。我在编写 tuc 时没有考虑到这一点，后来才发现它偶尔会去掉最后的行尾。

这个问题很容易修复，只需在处理完整个文件后检查状态。如果状态不是 ordinary，我们只需输出最后一个换行符。

> **注意**
> 既然我们已将算法表达为 _有限状态机_，我们可以轻松设计一个专用的数字电子电路（"芯片"）来为我们进行转换。当然，这样做会比编写汇编语言程序昂贵得多。

#### 输出计数器

因为我们的文件转换程序可能将两个字符合并为一个，我们需要使用输出计数器。我们将其初始化为 `0`，每次向输出发送字符时递增它。在程序结束时，计数器将告诉我们需要将文件设置为多大。

### 在软件中实现 FSM

使用 _有限状态机_ 工作的最难部分是分析问题并将其表达为 _有限状态机_。一旦完成，软件几乎自动完成。

在高级语言（如 C）中，有几种主要方法。一种是使用 `switch` 语句选择应运行哪个函数。例如，

```c
switch (state) {
	default:
	case REGULAR:
		regular(inputchar);
		break;
	case CR:
		cr(inputchar);
		break;
	case LF:
		lf(inputchar);
		break;
	}
```

另一种方法是使用函数指针数组，如下所示：

```c
(output[state])(inputchar);
```

还有一种方法是让 `state` 成为函数指针，设置为指向适当的函数：

```c
(*state)(inputchar);
```

这是我们将要在程序中使用的方法，因为它在汇编语言中非常容易实现，而且非常快。我们只需将正确过程的地址保存在 `EBX` 中，然后发出：

```asm
call	ebx
```

这可能比在代码中硬编码地址更快，因为微处理器不必从内存中获取地址——它已经存储在其某个寄存器中了。我说 _可能_ 是因为鉴于现代微处理器的缓存机制，两种方式可能一样快。

### 内存映射文件

因为我们的程序处理单个文件，我们不能使用之前对我们有效的方法，即从输入文件读取并写入输出文件。

UNIX 允许我们将文件或文件的一部分映射到内存中。为此，我们首先需要以适当的读/写标志打开文件。然后我们使用 `mmap` 系统调用将其映射到内存中。`mmap` 的一个好处是它自动与虚拟内存协同工作：我们可以将比物理内存可用容量更多的文件映射到内存中，但仍可通过常规内存操作码（如 `mov`、`lods` 和 `stos`）访问它。我们对文件内存映像所做的任何更改都会由系统写入文件。我们甚至不必保持文件打开：只要它保持映射状态，我们就可以对其进行读写。

32 位 Intel 微处理器可以访问多达 4 GB 的内存——物理或虚拟的。FreeBSD 系统允许我们使用其中一半用于文件映射。

为简单起见，在本教程中我们只转换可以完整映射到内存中的文件。可能有不超过 2 GB 大小的文本文件并不多。如果我们的程序遇到这样的文件，它只会显示一条消息建议我们使用原始的 tuc。

如果你查看 `syscalls.master` 的副本，会发现有两个名为 `mmap` 的单独系统调用。这是因为 UNIX 的演进：有传统的 BSD `mmap`，系统调用号 71。它被 POSIX® `mmap`，系统调用号 197 所取代。FreeBSD 系统同时支持两者，因为旧程序是使用原始 BSD 版本编写的。但新软件使用 POSIX 版本，我们也将使用它。

`syscalls.master` 中列出的 POSIX 版本如下：

```asm
197	STD	BSD	{ caddr_t mmap(caddr_t addr, size_t len, int prot, \
			    int flags, int fd, long pad, off_t pos); }
```

这与 `mmap(2)` 中描述的略有不同。这是因为 `mmap(2)` 描述的是 C 版本。

区别在于 `long pad` 参数，它在 C 版本中不存在。然而，FreeBSD 系统调用在 `push` 64 位参数后添加了一个 32 位填充。在这种情况下，`off_t` 是一个 64 位值。

当我们完成对内存映射文件的操作后，我们用 `munmap` 系统调用取消映射：

> **技巧**
> 关于 `mmap` 的深入论述，可参阅 W. Richard Stevens 的《Unix Network Programming》第 2 卷第 12 章。

### 确定文件大小

因为我们需要告诉 `mmap` 要将文件的多少字节映射到内存中，而且我们希望映射整个文件，所以需要确定文件大小。

我们可以使用 `fstat` 系统调用获取系统能提供的关于打开文件的所有信息。其中包括文件大小。

同样，`syscalls.master` 列出了 `fstat` 的两个版本，一个传统版本（系统调用 62）和一个 POSIX 版本（系统调用 189）。自然，我们将使用 POSIX 版本：

```asm
189	STD	POSIX	{ int fstat(int fd, struct stat *sb); }
```

这是一个非常直接的调用：我们向它传递 `stat` 结构的地址和打开文件的描述符。它会填充 `stat` 结构的内容。

不过我得说，我曾尝试在 `.bss` 段中声明 `stat` 结构，`fstat` 不接受：它设置了表示错误的进位标志。在我将代码改为在栈上分配结构后，一切正常。

### 更改文件大小

因为我们的程序可能将回车符/换行符序列合并为单纯的换行符，我们的输出可能比输入小。然而，由于我们将输出放在读取输入的同一文件中，我们可能需要更改文件大小。

`ftruncate` 系统调用允许我们做到这一点。尽管其名称有些误导，但 `ftruncate` 系统调用既可用于截断文件（使其更小），也可用于扩展它。

是的，我们会在 `syscalls.master` 中找到 `ftruncate` 的两个版本，一个较旧（130），一个较新（201）。我们将使用较新的：

```asm
201	STD	BSD	{ int ftruncate(int fd, int pad, off_t length); }
```

请注意这个又包含了一个 `int pad`。

### ftuc

我们现在知道了编写 ftuc 所需的一切。我们首先在 `system.inc` 中添加一些新行。首先，我们在文件开头附近定义一些常量和结构：

```asm
;;;;;;; open flags
%define	O_RDONLY	0
%define	O_WRONLY	1
%define	O_RDWR	2

;;;;;;; mmap flags
%define	PROT_NONE	0
%define	PROT_READ	1
%define	PROT_WRITE	2
%define	PROT_EXEC	4
;;
%define	MAP_SHARED	0001h
%define	MAP_PRIVATE	0002h

;;;;;;; stat structure
struc	stat
st_dev		resd	1	; = 0
st_ino		resd	1	; = 4
st_mode		resw	1	; = 8, 大小为 16 位
st_nlink	resw	1	; = 10, 同上
st_uid		resd	1	; = 12
st_gid		resd	1	; = 16
st_rdev		resd	1	; = 20
st_atime	resd	1	; = 24
st_atimensec	resd	1	; = 28
st_mtime	resd	1	; = 32
st_mtimensec	resd	1	; = 36
st_ctime	resd	1	; = 40
st_ctimensec	resd	1	; = 44
st_size		resd	2	; = 48, 大小为 64 位
st_blocks	resd	2	; = 56, 同上
st_blksize	resd	1	; = 64
st_flags	resd	1	; = 68
st_gen		resd	1	; = 72
st_lspare	resd	1	; = 76
st_qspare	resd	4	; = 80
endstruc
```

我们定义新的系统调用：

```asm
%define	SYS_mmap	197
%define	SYS_munmap	73
%define	SYS_fstat	189
%define	SYS_ftruncate	201
```

我们添加使用它们的宏：

```asm
%macro	sys.mmap	0
	system	SYS_mmap
%endmacro

%macro	sys.munmap	0
	system	SYS_munmap
%endmacro

%macro	sys.ftruncate	0
	system	SYS_ftruncate
%endmacro

%macro	sys.fstat	0
	system	SYS_fstat
%endmacro
```

下面是我们的代码：

```asm
;;;;;;; Fast Text-to-Unix Conversion (ftuc.asm) ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;
;; Started:	21-Dec-2000
;; Updated:	22-Dec-2000
;;
;; Copyright 2000 G. Adam Stanislav.
;; All rights reserved.
;;
;;;;;;; v.1 ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
%include	'system.inc'

section	.data
	db	'Copyright 2000 G. Adam Stanislav.', 0Ah
	db	'All rights reserved.', 0Ah
usg	db	'Usage: ftuc filename', 0Ah
usglen	equ	$-usg
co	db	"ftuc: Can't open file.", 0Ah
colen	equ	$-co
fae	db	'ftuc: File access error.', 0Ah
faelen	equ	$-fae
ftl	db	'ftuc: File too long, use regular tuc instead.', 0Ah
ftllen	equ	$-ftl
mae	db	'ftuc: Memory allocation error.', 0Ah
maelen	equ	$-mae

section	.text

align 4
memerr:
	push	dword maelen
	push	dword mae
	jmp	short error

align 4
toolong:
	push	dword ftllen
	push	dword ftl
	jmp	short error

align 4
facerr:
	push	dword faelen
	push	dword fae
	jmp	short error

align 4
cantopen:
	push	dword colen
	push	dword co
	jmp	short error

align 4
usage:
	push	dword usglen
	push	dword usg

error:
	push	dword stderr
	sys.write

	push	dword 1
	sys.exit

align 4
global	_start
_start:
	pop	eax		; argc
	pop	eax		; 程序名
	pop	ecx		; 要转换的文件
	jecxz	usage

	pop	eax
	or	eax, eax	; 参数太多？
	jne	usage

	; 打开文件
	push	dword O_RDWR
	push	ecx
	sys.open
	jc	cantopen

	mov	ebp, eax	; 保存 fd

	sub	esp, byte stat_size
	mov	ebx, esp

	; 查找文件大小
	push	ebx
	push	ebp		; fd
	sys.fstat
	jc	facerr

	mov	edx, [ebx + st_size + 4]

	; 如果 EDX != 0 则文件太长 ...
	or	edx, edx
	jne	near toolong
	mov	ecx, [ebx + st_size]
	; ... 或如果超过 2 GB
	or	ecx, ecx
	js	near toolong

	; 如果文件大小为 0 字节则什么也不做
	jecxz	.quit

	; 将整个文件映射到内存
	push	edx
	push	edx		; 从偏移量 0 开始
	push	edx		; pad
	push	ebp		; fd
	push	dword MAP_SHARED
	push	dword PROT_READ | PROT_WRITE
	push	ecx		; 整个文件大小
	push	edx		; 让系统决定地址
	sys.mmap
	jc	near memerr

	mov	edi, eax
	mov	esi, eax
	push	ecx		; 用于 SYS_munmap
	push	edi

	; 使用 EBX 作为状态机
	mov	ebx, ordinary
	mov	ah, 0Ah
	cld

.loop:
	lodsb
	call	ebx
	loop	.loop

	cmp	ebx, ordinary
	je	.filesize

	; 输出最终的 lf
	mov	al, ah
	stosb
	inc	edx

.filesize:
	; 将文件截断为新大小
	push	dword 0		; 高 dword
	push	edx		; 低 dword
	push	eax		; pad
	push	ebp
	sys.ftruncate

	; 关闭它（ebp 仍然 pushed）
	sys.close

	add	esp, byte 16
	sys.munmap

.quit:
	push	dword 0
	sys.exit

align 4
ordinary:
	cmp	al, 0Dh
	je	.cr

	cmp	al, ah
	je	.lf

	stosb
	inc	edx
	ret

align 4
.cr:
	mov	ebx, cr
	ret

align 4
.lf:
	mov	ebx, lf
	ret

align 4
cr:
	cmp	al, 0Dh
	je	.cr

	cmp	al, ah
	je	.lf

	xchg	al, ah
	stosb
	inc	edx

	xchg	al, ah
	; 落入

.lf:
	stosb
	inc	edx
	mov	ebx, ordinary
	ret

align 4
.cr:
	mov	al, ah
	stosb
	inc	edx
	ret

align 4
lf:
	cmp	al, ah
	je	.lf

	cmp	al, 0Dh
	je	.cr

	xchg	al, ah
	stosb
	inc	edx

	xchg	al, ah
	stosb
	inc	edx
	mov	ebx, ordinary
	ret

align 4
.cr:
	mov	ebx, ordinary
	mov	al, ah
	; 落入

.lf:
	stosb
	inc	edx
	ret
```

> **警告**
> 不要对存储在 MS-DOS 或 Windows 格式化的磁盘上的文件使用此程序。在 FreeBSD 下对这些挂载的驱动器使用 `mmap` 时，FreeBSD 代码中似乎存在一个微妙的 bug：如果文件超过一定大小，`mmap` 只会用零填充内存，然后将它们复制到文件中覆盖其内容。

## 12. 专心致志

作为禅学的学生，我喜欢"一心专注"的理念：一次只做一件事，并做好它。

这确实也正是 UNIX 的工作方式。典型的 Windows 应用程序试图做所有可以想象到的事情（因此充满了 bug），而典型的 UNIX 程序只做一件事，并做好它。

然后，典型的 UNIX 用户通过编写 shell 脚本，将各种现有程序的输出通过管道传递给另一个程序的输入，从而组合出自己的应用程序。

在编写自己的 UNIX 软件时，通常的好做法是看看问题的哪些部分可以用现有程序处理，只为你没有现有解决方案的那部分问题编写自己的程序。

### CSV

我将用一个我最近遇到的真实案例来说明这个原则：

我需要从网站下载的数据库中提取每条记录的第 11 个字段。该数据库是 CSV 文件，即 _逗号分隔值_ 列表。这是一种在不同数据库软件用户之间共享数据的相当标准的格式。

文件的第一行包含以逗号分隔的各种字段列表。文件的其余部分逐行列出数据，值之间以逗号分隔。

我尝试了 awk，使用逗号作为分隔符。但由于某些行包含带引号的逗号，awk 从这些行中提取了错误的字段。

因此，我需要编写自己的软件来从 CSV 文件中提取第 11 个字段。然而，本着 UNIX 的精神，我只需要编写一个简单的过滤器来执行以下操作：

* 从文件中删除第一行；
* 将所有未加引号的逗号更改为不同的字符；
* 删除所有引号。

严格来说，我可以使用 sed 从文件中删除第一行，但在我自己的程序中这样做非常容易，所以我决定这样做并减小管道的大小。

无论如何，编写这样一个程序花了我大约 20 分钟。而编写一个从 CSV 文件中提取第 11 个字段的程序会花长得多的时间，而且我无法重用它来从其他数据库中提取其他字段。

这次我决定让它比典型的教程程序做更多的工作：

* 它解析命令行选项；
* 如果发现错误参数，它会显示正确的用法；
* 它产生有意义的错误消息。

以下是它的用法消息：

```sh
Usage: csv [-t<delim>] [-c<comma>] [-p] [-o <outfile>] [-i <infile>]
```

所有参数都是可选的，可以以任何顺序出现。

`-t` 参数声明用什么替换逗号。这里默认是 `tab`。例如，`-t;` 会将所有未加引号的逗号替换为分号。

我不需要 `-c` 选项，但它在将来可能派上用场。它让我声明要用其他内容替换非逗号的某个字符。例如，`-c@` 会替换所有 at 符号（如果你想将电子邮件地址列表拆分为用户名和域，这很有用）。

`-p` 选项保留第一行，即不删除它。默认情况下我们删除第一行，因为在 CSV 文件中它包含字段名而非数据。

`-i` 和 `-o` 选项让我指定输入和输出文件。默认是 `stdin` 和 `stdout`，所以这是一个标准的 UNIX 过滤器。

我确保同时接受 `-i filename` 和 `-ifilename`。我还确保只能指定一个输入文件和一个输出文件。

要获取每条记录的第 11 个字段，我现在可以这样做：

```sh
% csv '-t;' data.csv | awk '-F;' '{print $11}'
```

代码将选项（文件描述符除外）存储在 `EDX` 中：逗号在 `DH` 中，新分隔符在 `DL` 中，`-p` 选项的标志在 `EDX` 的最高位中，因此检查其符号可以让我们快速决定怎么做。

下面是代码：

```asm
;;;;;;; csv.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; 将逗号分隔的文件转换为以其他字符分隔的文件。
;
; Started:	31-May-2001
; Updated:	 1-Jun-2001
;
; Copyright (c) 2001 G. Adam Stanislav
; All rights reserved.
;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

%include	'system.inc'

%define	BUFSIZE	2048

section	.data
fd.in	dd	stdin
fd.out	dd	stdout
usg	db	'Usage: csv [-t<delim>] [-c<comma>] [-p] [-o <outfile>] [-i <infile>]', 0Ah
usglen	equ	$-usg
iemsg	db	"csv: Can't open input file", 0Ah
iemlen	equ	$-iemsg
oemsg	db	"csv: Can't create output file", 0Ah
oemlen	equ	$-oemsg

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE

section	.text
align 4
ierr:
	push	dword iemlen
	push	dword iemsg
	push	dword stderr
	sys.write
	push	dword 1		; 返回失败
	sys.exit

align 4
oerr:
	push	dword oemlen
	push	dword oemsg
	push	dword stderr
	sys.write
	push	dword 2
	sys.exit

align 4
usage:
	push	dword usglen
	push	dword usg
	push	dword stderr
	sys.write
	push	dword 3
	sys.exit

align 4
global	_start
_start:
	add	esp, byte 8	; 丢弃 argc 和 argv[0]
	mov	edx, (',' << 8) | 9

.arg:
	pop	ecx
	or	ecx, ecx
	je	near .init		; 没有更多参数

	; ECX 包含指向参数的指针
	cmp	byte [ecx], '-'
	jne	usage

	inc	ecx
	mov	ax, [ecx]

.o:
	cmp	al, 'o'
	jne	.i

	; 确保没有被要求两次输出文件
	cmp	dword [fd.out], stdout
	jne	usage

	; 查找输出文件路径——它在 [ECX+1]，
	; 即 -ofile --
	; 或者在下一个参数中，
	; 即 -o file

	inc	ecx
	or	ah, ah
	jne	.openoutput
	pop	ecx
	jecxz	usage

.openoutput:
	push	dword 420	; 文件模式（八进制 644）
	push	dword 0200h | 0400h | 01h
	; O_CREAT | O_TRUNC | O_WRONLY
	push	ecx
	sys.open
	jc	near oerr

	add	esp, byte 12
	mov	[fd.out], eax
	jmp	short .arg

.i:
	cmp	al, 'i'
	jne	.p

	; 确保没有被要求两次
	cmp	dword [fd.in], stdin
	jne	near usage

	; 查找输入文件路径
	inc	ecx
	or	ah, ah
	jne	.openinput
	pop	ecx
	or	ecx, ecx
	je near usage

.openinput:
	push	dword 0		; O_RDONLY
	push	ecx
	sys.open
	jc	near ierr		; 打开失败

	add	esp, byte 8
	mov	[fd.in], eax
	jmp	.arg

.p:
	cmp	al, 'p'
	jne	.t
	or	ah, ah
	jne	near usage
	or	edx, 1 << 31
	jmp	.arg

.t:
	cmp	al, 't'		; 重定义输出分隔符
	jne	.c
	or	ah, ah
	je	near usage
	mov	dl, ah
	jmp	.arg

.c:
	cmp	al, 'c'
	jne	near usage
	or	ah, ah
	je	near usage
	mov	dh, ah
	jmp	.arg

align 4
.init:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	mov	edi, obuffer

	; 查看是否要保留第一行
	or	edx, edx
	js	.loop

.firstline:
	; 去掉第一行
	call	getchar
	cmp	al, 0Ah
	jne	.firstline

.loop:
	; 从 stdin 读取一个字节
	call	getchar

	; 是逗号（或用户指定的任何字符）吗？
	cmp	al, dh
	jne	.quote

	; 用 tab（或用户想要的任何字符）替换逗号
	mov	al, dl

.put:
	call	putchar
	jmp	short .loop

.quote:
	cmp	al, '"'
	jne	.put

	; 打印所有内容直到遇到另一个引号或 EOL。如果是
	; 引号则跳过它。如果是 EOL 则打印它。
.qloop:
	call	getchar
	cmp	al, '"'
	je	.loop

	cmp	al, 0Ah
	je	.put

	call	putchar
	jmp	short .qloop

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	ret

read:
	jecxz	.read
	call	write

.read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword [fd.in]
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.done
	sub	eax, eax
	ret

align 4
.done:
	call	write		; 刷新输出缓冲区

	; 关闭文件
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	; 返回成功
	push	dword 0
	sys.exit

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	jecxz	.ret	; 无内容可写
	sub	edi, ecx	; 缓冲区起始位置
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在为空
.ret:
	ret
```

其中大部分取自上面的 `hex.asm`。但有一个重要区别：我不再在每次输出换行符时都调用 `write`。然而，该代码可以交互使用。

自从我最初开始写这篇文章以来，我找到了一个更好的交互问题解决方案。我想确保每行仅在需要时才单独打印出来。毕竟，在非交互使用时没有必要刷新每一行。

我现在使用的新解决方案是在每次发现输入缓冲区为空时调用 `write`。这样，在交互模式下运行时，程序从用户键盘读取一行，处理它，发现输入缓冲区为空。它刷新输出并读取下一行。

#### 缓冲的阴暗面

此更改防止了在非常特定情况下的神秘死锁。我称之为 _缓冲的阴暗面_，主要是因为它呈现了一个不太明显的危险。

像上面的 csv 这样的程序不太可能发生这种情况，所以让我们考虑另一个过滤器：在这种情况下，我们期望输入是表示颜色值的原始数据，例如像素的 _红_、_绿_、_蓝_ 强度。我们的输出将是输入的负片。

这样的过滤器编写起来非常简单。它的大部分看起来与我们迄今编写的所有其他过滤器一样，所以我只向你展示其内循环：

```asm
.loop:
	call	getchar
	not	al		; 创建负片
	call	putchar
	jmp	short .loop
```

因为此过滤器处理原始数据，所以不太可能交互使用。

但它可能被图像处理软件调用。而且，除非它在每次调用 `read` 之前调用 `write`，否则很可能会死锁。

可能发生的情况如下：

1. 图像编辑器将使用 C 函数 `popen()` 加载我们的过滤器。
2. 它将从位图或像素图中读取第一行像素。
3. 它将第一行像素写入通向我们过滤器 `fd.in` 的 _管道_。
4. 我们的过滤器将从其输入中读取每个像素，将其转为负片，并写入其输出缓冲区。
5. 我们的过滤器将调用 `getchar` 来获取下一个像素。
6. `getchar` 将发现输入缓冲区为空，因此它将调用 `read`。
7. `read` 将调用 `SYS_read` 系统调用。
8. _内核_ 将挂起我们的过滤器，直到图像编辑器向管道发送更多数据。
9. 图像编辑器将从连接到我们过滤器 `fd.out` 的另一个管道读取，以便在它发送第二行输入之前设置输出图像的第一行。
10. _内核_ 挂起图像编辑器，直到它从我们的过滤器接收到一些输出，以便将其传递给图像编辑器。

此时，我们的过滤器等待图像编辑器发送更多数据来处理，而图像编辑器等待我们的过滤器发送第一行处理的结果。但结果还在我们的输出缓冲区中。

过滤器和图像编辑器将永远相互等待（或至少直到它们被杀死）。我们的软件刚刚进入了竞态条件。

如果我们的过滤器在向 _内核_ 请求更多输入数据之前刷新其输出缓冲区，则不存在此问题。

## 13. 使用 FPU

奇怪的是，大多数汇编语言文献甚至没有提到 FPU（_浮点单元_）的存在，更不用说讨论如何编程了。

然而，汇编语言最闪耀的时刻莫过于通过做 _只能_ 在汇编语言中做的事情来创建高度优化的 FPU 代码。

### FPU 的组织结构

FPU 由 8 个 80 位浮点寄存器组成。它们以栈的方式组织——你可以向 TOS（_栈顶_）`push` 一个值，也可以 `pop` 它。

话虽如此，汇编语言操作码不是 `push` 和 `pop`，因为它们已被占用。

你可以使用 `fld`、`fild` 和 `fbld` 向 TOS `push` 一个值。还有其他几个操作码让你将许多常见 _常量_——如 _pi_——`push` 到 TOS。

类似地，你可以使用 `fst`、`fstp`、`fist`、`fistp` 和 `fbstp` 来 `pop` 一个值。实际上，只有以 _p_ 结尾的操作码才会真正 `pop` 该值，其余的只是将其 `store` 到其他地方而不从 TOS 移除。

我们可以在 TOS 和计算机内存之间以 32 位、64 位或 80 位 _实数_、16 位、32 位或 64 位 _整数_，或 80 位 _压缩十进制数_ 的形式传输数据。

80 位 _压缩十进制数_ 是 _二进制编码十进制数_ 的一个特例，在 ASCII 数据表示和 FPU 内部数据之间转换时非常方便。它允许我们使用 18 位有效数字。

无论我们在内存中如何表示数据，FPU 始终在其寄存器中以 80 位 _实数_ 格式存储它。

其内部精度至少为 19 位十进制数字，因此即使我们选择以完整的 18 位精度将结果显示为 ASCII，我们仍然显示的是正确的结果。

我们可以在 TOS 上执行数学运算：我们可以计算其 _正弦_，可以对其进行 _缩放_（即乘以或除以 2 的幂），可以计算其以 2 为底的 _对数_，以及许多其他操作。

我们还可以将它与任何 FPU 寄存器（包括自身）_相乘_ 或 _相除_，或 _加上_ 或 _减去_。

TOS 的 Intel 官方操作码是 `st`，而 _寄存器_ 的是 `st(0)`-`st(7)`。`st` 和 `st(0)` 指的是同一个寄存器。

出于某种原因，nasm 的原作者决定使用不同的操作码，即 `st0`-`st7`。换句话说，没有括号，且 TOS 始终是 `st0`，而非仅仅是 `st`。

#### 压缩十进制格式

_压缩十进制_ 格式使用 10 字节（80 位）内存来表示 18 位数字。其中表示的数字始终是 _整数_。

> **技巧**
> 你可以通过先将 TOS 乘以 10 的幂来获得小数位。

最高字节的最高位（字节 9）是 _符号位_：如果设置，数字为 _负_，否则为 _正_。该字节的其余位未使用/忽略。

其余 9 字节存储数字的 18 位数字：每字节 2 位数字。

_更高有效数字_ 存储在高 _半字节_（4 位）中，_更低有效数字_ 存储在低 _半字节_ 中。

话虽如此，你可能认为 `-1234567` 会这样存储在内存中（使用十六进制表示法）：

```asm
80 00 00 00 00 00 01 23 45 67
```

可惜不是！与 Intel 制造的所有东西一样，即使是 _压缩十进制_ 也是 _小端序_。

这意味着我们的 `-1234567` 这样存储：

```asm
67 45 23 01 00 00 00 00 00 80
```

记住这一点，否则你会绝望地抓头发！

> **注意**
> 值得一读的书——如果你能找到的话——是 Richard Startz 的 [8087/80287/80387 for the IBM PC & Compatibles](http://www.amazon.com/exec/obidos/ASIN/013246604X/whizkidtechnomag)。不过它确实将 _压缩十进制_ 的小端序存储视为理所当然。我绝不是在开玩笑——在我想起应该对这种类型的数据也尝试小端序之前，我下面展示的过滤器出了什么问题，那种绝望真是难以言表。

### 针孔摄影探幽

要编写有意义的软件，我们不仅要了解编程工具，还要了解我们为之创建软件的领域。

我们的下一个过滤器将在我们想构建 _针孔相机_ 时提供帮助，因此在继续之前，我们需要了解一些 _针孔摄影_ 的背景知识。

#### 相机

描述任何曾经制造过的相机的最简单方式是：一些被某些不透光材料包围的空余空间，在围护结构上有一个小孔。

围护结构通常是坚固的（例如盒子），但有时是柔性的（皮腔）。相机内部相当暗。然而，小孔让光线通过一个点进入（虽然在某些情况下可能有多个点）。这些光线在孔前形成图像，即相机外部景物的再现。

如果将一些感光材料（如胶片）放在相机内部，它就能捕捉图像。

孔中通常包含一个 _透镜_ 或透镜组，通常称为 _镜头_。

#### 针孔

但严格来说，透镜不是必需的：最初的相机不使用透镜而是使用 _针孔_。即使在今天，_针孔_ 仍在使用，既作为研究相机工作原理的工具，也用于实现一种特殊的图像效果。

_针孔_ 产生的图像整体上同样清晰。或者说同样模糊。针孔有一个理想尺寸：如果更大或更小，图像都会失去清晰度。

#### 焦距

此理想针孔直径是 _焦距_（即针孔到胶片的距离）平方根的函数。

```asm
D = PC * sqrt(FL)
```

其中，`D` 是针孔的理想直径，`FL` 是焦距，`PC` 是针孔常数。根据 Jay Bender 的说法，其值为 `0.04`，而 Kenneth Connors 确定为 `0.037`。其他人提出了其他值。此外，此值仅适用于日光：其他类型的光需要不同的常数，其值只能通过实验确定。

#### F 数

F 数是衡量多少光到达胶片的一个非常有用的指标。测光表可以确定，例如，要用 f/5.6 曝光特定感光度的胶片可能需要曝光持续 1/1000 秒。

无论是 35 毫米相机还是 6x9 厘米相机等都无关紧要。只要我们知道 F 数，就能确定正确的曝光。

F 数易于计算：

```asm
F = FL / D
```

换句话说，F 数等于焦距除以针孔直径。这也意味着更高的 F 数意味着更小的针孔或更大的焦距，或两者兼有。这又意味着，F 数越高，曝光时间越长。

此外，虽然针孔直径和焦距是一维测量，但胶片和针孔都是二维的。这意味着如果你在 F 数 `A` 下测量的曝光为 `t`，那么在 F 数 `B` 下的曝光为：

```asm
t * (B / A)²
```

#### 标准化 F 数

虽然许多现代相机可以平滑、渐进地改变针孔直径，从而改变 F 数，但并非总是如此。

为了提供不同的 F 数，相机通常包含一块钻有多个不同大小孔的金属板。

它们的尺寸根据上述公式选择，使得产生的 F 数是所有相机上使用的标准 F 数之一。例如，我拥有的一台非常旧的 Kodak Duaflex IV 相机有三个这样的孔，对应 F 数 8、11 和 16。

更新近制造的相机可能提供 F 数 2.8、4、5.6、8、11、16、22 和 32（以及其他）。这些数字并非任意选择：它们都是 2 的平方根的幂，尽管可能有所舍入。

#### F 档

典型的相机设计为设置任何标准化 F 数时都会改变拨盘的触感。它会在该位置自然 _停止_。因此，拨盘的这些位置称为 F 档。

由于每个档位的 F 数是 2 的平方根的幂，将拨盘移动 1 档会使正确曝光所需的光量加倍。移动 2 档会使所需曝光变为四倍。将拨盘移动 3 档会使曝光增加到 8 倍，等等。

### 设计针孔软件

我们现在准备好决定到底希望我们的针孔软件做什么。

#### 处理程序输入

由于其主要目的是帮助我们设计可用的针孔相机，我们将使用 _焦距_ 作为程序的输入。这是无需软件即可确定的：正确的焦距由胶片尺寸以及拍摄"常规"照片、广角照片还是长焦照片的需求决定。

我们迄今编写的大多数程序都以单个字符或字节作为输入：hex 程序将单个字节转换为十六进制数字，csv 程序要么让字符通过、删除它，要么将其更改为不同字符，等等。

一个程序——ftuc——使用状态机最多一次考虑两个输入字节。

但我们的针孔程序不能仅处理单个字符，它必须处理更大的语法单元。

例如，如果我们希望程序计算焦距为 `100 mm`、`150 mm` 和 `210 mm` 时的针孔直径（以及我们稍后讨论的其他值），我们可能希望输入如下内容：

```sh
 100, 150, 210
```

我们的程序需要一次考虑多于单个字节的输入。当它看到第一个 `1` 时，必须理解它正在看到一个十进制数的第一位数字。当它看到 `0` 和另一个 `0` 时，必须知道它正在看到同一数字的更多位。

当它遇到第一个逗号时，必须知道它不再接收第一个数字的位。它必须能够将第一个数字的位转换为 `100` 的值。将第二个数字的位转换为 `150` 的值。当然，还有将第三个数字的位转换为 `210` 的数值。

我们需要决定接受哪些分隔符：输入数字必须用逗号分隔吗？如果是，如何处理用其他东西分隔的两个数字？

就我个人而言，我喜欢保持简单。某个东西要么是数字，我就处理它。要么不是数字，我就丢弃它。我不喜欢计算机在我输入额外字符时抱怨——当这个字符 _明显_ 是额外字符时。哼！

此外，它让我打破计算的单调性，输入一个查询而非仅仅一个数字：

```sh
What is the best pinhole diameter for the
	    focal length of 150?
```

计算机没有理由吐出一堆抱怨：

```sh
Syntax error: What
Syntax error: is
Syntax error: the
Syntax error: best
```

等等，等等，等等。

其次，我喜欢用 `#` 字符表示注释的开始，注释延伸到行尾。这不需太多编码工作，还让我可以将软件的输入文件视为可执行脚本。

在我们的情况下，还需要决定输入应使用什么单位：我们选择 _毫米_，因为大多数摄影师以此衡量焦距。

最后，我们需要决定是否允许使用小数点（在这种情况下我们还必须考虑世界上大部分地区使用小数 _逗号_ 的事实）。

在我们的情况下，允许小数点/逗号会给人一种虚假的精度感：焦距 `50` 和 `51` 之间几乎没有任何明显差异，因此允许用户输入类似 `50.5` 的内容不是好主意。这是我的看法，但写这个程序的人是我。当然，你可以在你的程序中做出不同的选择。

#### 提供选项

构建针孔相机时我们最需要知道的是针孔直径。由于我们希望拍摄清晰的图像，我们将使用上述公式从焦距计算针孔直径。由于专家们对 `PC` 常数提供了几个不同的值，我们需要有选择。

在 UNIX 编程中，传统上有两种选择程序参数的方式，外加一个用户未做选择时的默认值。

为什么要有两种选择方式？

一种是允许（相对）_永久_ 的选择，每次运行软件时自动应用，而无需我们反复告诉它要做什么。

永久选择可以存储在配置文件中，通常位于用户的主目录中。该文件通常与应用程序同名，但以点开头。文件名中常添加 _"rc"_。因此，我们的文件可以是 **~/.pinhole** 或 **~/.pinholerc**。（**~/** 表示当前用户的主目录。）

配置文件主要由具有许多可配置参数的程序使用。只有一两个参数的程序通常使用不同的方法：它们期望在 _环境变量_ 中找到参数。在我们的情况下，我们可以查看名为 `PINHOLE` 的环境变量。

通常，程序使用上述方法之一。否则，如果配置文件说一件事，而环境变量说另一件事，程序可能会混乱（或者变得太复杂）。

因为我们只需要选择 _一个_ 这样的参数，我们将采用第二种方法，在环境中搜索名为 `PINHOLE` 的变量。

另一种方式允许我们做出 _临时_ 决定：_"虽然我通常希望你使用 0.039，但这次我要 0.03872。"_ 换句话说，它允许我们 _覆盖_ 永久选择。

这类选择通常通过命令行参数完成。

最后，程序 _始终_ 需要 _默认值_。用户可能不做任何选择。也许他不知道该选什么。也许他只是"随便看看"。理想情况下，默认值是大多数用户无论如何都会选择的值。这样他们就不需要选择了。或者更确切地说，他们可以无需额外努力地选择默认值。

给定此系统，程序可能会发现冲突的选项，并按以下方式处理：

1. 如果它发现 _临时_ 选择（例如命令行参数），应接受该选择。必须忽略任何永久选择和任何默认值。
2. _否则_，如果它发现永久选项（例如环境变量），应接受它，并忽略默认值。
3. _否则_，应使用默认值。

我们还需要决定 `PC` 选项应具有什么 _格式_。

乍一看，环境变量使用 `PINHOLE=0.04` 格式，命令行使用 `-p0.04` 似乎很明显。

允许这样做实际上是一种安全风险。`PC` 常数是一个非常小的数字。自然，我们将使用各种小的 `PC` 值测试我们的软件。但如果有人选择一个巨大的值运行程序会怎样？

它可能使程序崩溃，因为我们没有设计它来处理巨大数字。

或者，我们可以在程序上花更多时间使其能处理巨大数字。如果我们在为计算机盲受众编写商业软件，我们可能会这样做。

或者，我们可以说，_"太苛刻了！用户应该更懂事。"_ 

或者，我们只需使用户不可能输入巨大数字。这就是我们将采用的方法：我们将使用一个隐含的 `0.` 前缀。

换句话说，如果用户想要 `0.04`，我们期望他输入 `-p04`，或在他的环境中设置 `PINHOLE=04`。因此，如果他说 `-p9999999`，我们将其解释为 `0.9999999`——仍然荒谬但至少更安全。

其次，许多用户只会选择 Bender 常数或 Connors 常数。为了方便他们，我们将 `-b` 解释为等同于 `-p04`，将 `-c` 解释为等同于 `-p037`。

#### 输出

我们需要决定软件要向输出发送什么，以及以什么格式发送。

由于我们的输入允许不指定数量的焦距条目，使用传统的数据库风格输出是有意义的，即在每个单独的行上显示每个焦距的计算结果，同时用 `tab` 字符分隔一行上的所有值。

可选地，我们还应允许用户指定使用我们之前学习过的 CSV 格式。在这种情况下，我们将打印一行以逗号分隔的名称来描述每行的每个字段，然后像以前一样显示结果，但用 `逗号` 替换 `tab`。

我们需要一个用于 CSV 格式的命令行选项。我们不能使用 `-c` 因为它已经表示 _使用 Connors 常数_。出于某种奇怪的原因，许多网站将 CSV 文件称为 _"Excel 电子表格"_（尽管 CSV 格式早于 Excel）。因此，我们将使用 `-e` 开关来通知软件我们希望输出为 CSV 格式。

我们将以焦距开始输出的每一行。乍一听可能觉得重复，特别是在交互模式下：用户输入焦距，我们再重复它。

但用户可以在一行上输入多个焦距。输入也可以来自文件或另一个程序的输出。在这种情况下，用户根本看不到输入。

同样，输出可以转到我们以后想查看的文件，或转到打印机，或成为另一个程序的输入。

因此，以用户输入的焦距开始每一行完全合理。

等等，不！不是 _用户输入的那样_。如果用户输入如下内容怎么办：

```sh
 00000000150
```

显然，我们需要去掉那些前导零。

因此，我们可能会考虑按原样读取用户输入，在 FPU 内部将其转换为二进制，然后从那里打印出来。

但是……

如果用户输入如下内容怎么办：

```sh
 17459765723452353453534535353530530534563507309676764423
```

哈！压缩十进制 FPU 格式允许我们输入 18 位数字。但用户输入了超过 18 位。我们如何处理？

好吧，我们 _可以_ 修改代码以读取前 18 位数字，将其输入 FPU，然后读取更多，将 TOS 上已有的值乘以 10 的附加位数次幂，然后再 `add` 到它。

是的，我们可以这样做。但在 _这个_ 程序中这样做是荒谬的（在另一个程序中这可能正是该做的）：即使是地球的周长以毫米表示也只需 11 位数字。显然，我们无法建造那么大的相机（至少目前不行）。

所以，如果用户输入这么大的数字，他要么是无聊，要么是在测试我们，要么是试图入侵系统，要么是在玩游戏——做什么都行，就是不是在设计针孔相机。

我们该怎么做？

我们会算是给他一个下马威：

```sh
17459765723452353453534535353530530534563507309676764423	???	???	???	???	???
```

为此，我们只需忽略任何前导零。一旦找到非零数字，我们将计数器初始化为 `0` 并开始执行三个步骤：

1. 将数字发送到输出。
2. 将数字追加到我们稍后用于生成可发送给 FPU 的压缩十进制数的缓冲区。
3. 递增计数器。

在执行这三个步骤的同时，我们还需要注意两种条件之一：

* 如果计数器超过 18，我们停止追加到缓冲区。我们继续读取数字并将其发送到输出。
* 如果，或者更确切地说 _当_，下一个输入字符不是数字时，我们暂时完成输入。
  顺便说一下，我们可以直接丢弃非数字字符，除非它是 `#`，我们必须将其返回到输入流。它开始一个注释，所以我们必须在完成输出并开始查找更多输入后看到它。

这还有一种可能性未涵盖：如果用户只输入一个零（或多个零），我们将永远找不到要显示的非零数字。

我们可以在计数器保持为 `0` 时确定这种情况已发生。在这种情况下，我们需要向输出发送 `0`，并执行另一个"下马威"：

```sh
0	???	???	???	???	???
```

一旦我们显示了焦距并确定其有效（大于 `0` 但不超过 18 位），就可以计算针孔直径。

_pinhole_ 包含 _pin_（针）一词并非巧合。确实，许多针孔字面意思就是 _针孔_，用针尖小心地刺出的孔。

这是因为典型的针孔非常小。我们的公式得到的结果以毫米为单位。我们将它乘以 `1000`，以便以 _微米_ 输出结果。

此时我们还要面对另一个陷阱：_精度过高_。

是的，FPU 是为高精度数学而设计的。但我们处理的不是高精度数学。我们处理的是物理（具体说是光学）。

假设我们想将一辆卡车改装成针孔相机（我们不会是第一批这样做的人！）。假设它的货箱长 `12` 米，所以焦距为 `12000`。使用 Bender 常数，我们得到 `12000` 的平方根乘以 `0.04`，即 `4.381780460` 毫米，或 `4381.780460` 微米。

怎么说，结果都过于精确。我们的卡车并非 _恰好_ `12000` 毫米长。我们没有那么精确地测量其长度，所以说我们需要直径为 `4.381780460` 毫米的针孔是具有欺骗性的。`4.4` 毫米就足够了。

> **注意**
> 我在上面"只"使用了十位数字。想象一下使用全部 18 位的荒谬！

我们需要限制结果的有效数字位数。一种方法是使用表示微米的整数。因此，我们的卡车需要直径为 `4382` 微米的针孔。看着那个数字，我们仍然认为 `4400` 微米（即 `4.4` 毫米）足够接近。

此外，我们可以决定无论结果多大，我们只想显示四位有效数字（当然也可以是任何其他位数）。可惜 FPU 不提供舍入到特定位数的功能（毕竟它将数字视为二进制而非十进制）。

因此，我们必须设计一个算法来减少有效数字位数。

这是我的算法（我认为它很笨拙——如果你知道更好的，_请_ 告诉我）：

1. 将计数器初始化为 `0`。
2. 当数字大于或等于 `10000` 时，将其除以 `10` 并递增计数器。
3. 输出结果。
4. 当计数器大于 `0` 时，输出 `0` 并递减计数器。

> **注意**
> `10000` 仅在你想 _四位_ 有效数字时适用。对于任何其他有效数字位数，将 `10000` 替换为 `10` 的有效数字位数次幂。

因此，我们将输出以微米为单位的针孔直径，舍入到四位有效数字。

此时，我们知道 _焦距_ 和 _针孔直径_。这意味着我们有足够的信息也来计算 _F 数_。

我们将显示 F 数，舍入到四位有效数字。F 数很可能对我们没什么意义。为使其更有意义，我们可以找到最近的 _标准化 F 数_，即 2 的平方根的最近幂次。

我们通过将实际 F 数自乘来实现，这当然会得到其 _平方_。然后我们计算其以 2 为底的对数，这比计算以 2 的平方根为底的对数容易得多！我们将结果舍入到最近的整数。接下来，我们将 2 的该次幂。实际上，FPU 给我们提供了一个很好的捷径：我们可以使用 `fscale` 操作码来"缩放"1，这类似于将整数左 _移_。最后，我们计算其平方根，就得到了最近的标准化 F 数。

如果这听起来令人不知所措——或者可能太费力——当你看到代码时会清楚得多。它总共需要 9 条操作码：

```asm
fmul	st0, st0
	fld1
	fld	st1
	fyl2x
	frndint
	fld1
	fscale
	fsqrt
	fstp	st1
```

第一行 `fmul st0, st0`，对 TOS（栈顶，与 `st` 相同，nasm 称为 `st0`）的内容求平方。`fld1` 将 `1` push 到 TOS。

下一行 `fld st1`，将平方 push 回 TOS。此时平方同时在 `st` 和 `st(2)` 中（稍后就会清楚为什么我们在栈上保留第二个副本）。`st(1)` 包含 `1`。

接下来，`fyl2x` 计算 `st` 乘以 `st(1)` 的以 2 为底的对数。这就是我们之前在 `st(1)` 上放置 `1` 的原因。

此时，`st` 包含我们刚计算的对数，`st(1)` 包含我们为以后保存的实际 F 数的平方。

`frndint` 将 TOS 舍入到最近的整数。`fld1` push 一个 `1`。`fscale` 用 `st(1)` 中的值移位 TOS 上的 `1`，有效地将 2 提升为 `st(1)` 次幂。

最后，`fsqrt` 计算结果的平方根，即最近的标准化 F 数。

我们现在在 TOS 上有最近的标准化 F 数，在 `st(1)` 中有舍入到最近整数的以 2 为底的对数，在 `st(2)` 中有实际 F 数的平方。我们将 `st(2)` 中的值留作后用。

但我们不再需要 `st(1)` 的内容。最后一行 `fstp st1`，将 `st` 的内容放入 `st(1)`，然后 pop。结果是，原来的 `st(1)` 现在是 `st`，原来的 `st(2)` 现在是 `st(1)`，依此类推。新的 `st` 包含标准化 F 数。新的 `st(1)` 包含我们存储在那里的实际 F 数的平方。

此时，我们准备好输出标准化 F 数。因为它是标准化的，我们不会将其舍入到四位有效数字，而是以完整精度发送。

标准化 F 数只要足够小且能在我们的测光表上找到就有用。否则，我们需要一种不同的方法来确定正确曝光。

之前我们已经想出了在任意 F 数下计算正确曝光的公式，该曝光是从在不同 F 数下测得的曝光计算得出的。

我见过的每个测光表都能确定 f5.6 下的正确曝光。因此，我们将计算一个 _"f5.6 倍数"_，即我们需要将 f5.6 下测得的曝光乘以多少来确定针孔相机的正确曝光。

从上述公式我们知道，此系数可以通过将我们的 F 数（实际值，非标准化值）除以 `5.6` 并对结果求平方来计算。

从数学上讲，将我们 F 数的平方除以 `5.6` 的平方会得到相同的结果。

从计算角度来说，我们不想在只需求一个数的平方时求两个数的平方。所以，第一个解决方案乍看更好。

但是……

`5.6` 是一个 _常数_。我们不必让 FPU 浪费宝贵的时钟周期。我们可以直接告诉它将 F 数的平方除以 `5.6²` 等于的值。或者我们可以将 F 数除以 `5.6`，然后对结果求平方。这两种方式现在看起来一样。

但它们不一样！

研究了上述摄影原理后，我们记得 `5.6` 实际上是 2 的平方根的五次方。一个 _无理数_。该数的平方 _恰好_ 是 `32`。

`32` 不仅是一个整数，还是 2 的幂。我们不需要将 F 数的平方除以 `32`。我们只需使用 `fscale` 将其右移五位。在 FPU 术语中，这意味着我们将 `st(1)` 等于 `-5` 时进行 `fscale`。这比除法 _快得多_。

所以，现在清楚了我们为什么将 F 数的平方保存在 FPU 栈顶。f5.6 倍数的计算是整个程序中最简单的计算！我们将输出舍入到四位有效数字。

我们还能计算另一个有用的数字：我们的 F 数距离 f5.6 有多少档。如果我们的 F 数刚好超出测光表的范围但我们有一个可以设置各种速度的快门，并且此快门使用档位，这可能对我们有帮助。

假设我们的 F 数距离 f5.6 有 5 档，测光表说我们应该使用 1/1000 秒。然后我们可以先将快门速度设为 1/1000，然后将拨盘移动 5 档。

此计算也很容易。我们要做的就是计算刚才计算的 f5.6 倍数的以 2 为底的对数（不过我们需要舍入之前的值）。然后我们将结果舍入到最近的整数。我们不必担心这个数字超过四位有效数字：结果最可能只有一两位数字。

### FPU 优化

在汇编语言中，我们可以用高级语言（包括 C）无法做到的方式优化 FPU 代码。

每当 C 函数需要计算浮点值时，它会将所有必要的变量和常量加载到 FPU 寄存器中。然后它执行所需的任何计算以获得正确结果。优秀的 C 编译器可以很好地优化这部分代码。

它通过将结果留在 TOS 上来"返回"值。然而，在返回之前，它会进行清理。它在计算中使用的任何变量和常量现在都已从 FPU 中消失。

它无法做到我们刚才所做的：我们计算了 F 数的平方并将其保留在栈上供另一个函数稍后使用。

我们 _知道_ 稍后会需要该值。我们还知道栈上有足够的空间（只能容纳 8 个数字）来存储它。

C 编译器无法知道栈上的某个值在不久的将来会再次被需要。

当然，C 程序员可能知道这一点。但他唯一的办法是将该值存储在内存变量中。

这意味着，首先，该值将从 FPU 内部使用的 80 位精度更改为 C 的 _double_（64 位）甚至 _single_（32 位）。

这也意味着该值必须从 TOS 移到内存中，然后再移回来。唉，在所有 FPU 操作中，访问计算机内存的那些是最慢的。

因此，在用汇编语言编程 FPU 时，要寻找将中间结果保留在 FPU 栈上的方法。

我们可以将这个想法更进一步！在我们的程序中，我们使用一个 _常数_（我们命名为 `PC` 的那个）。

无论我们计算多少个针孔直径：1、10、20、1000，我们始终使用同一个常数。因此，我们可以通过始终将常数保留在栈上来优化程序。

在程序的早期，我们计算上述常数的值。我们需要为常数中的每一位数字将输入除以 `10`。

乘法比除法快得多。因此，在程序开始时，我们将 `10` 除入 `1` 得到 `0.1`，然后将其保留在栈上：不再为每个数字将输入除以 `10`，而是将其乘以 `0.1`。

顺便说一下，我们不直接输入 `0.1`，尽管我们可以。我们这样做有原因：虽然 `0.1` 只需一位小数即可表示，但我们不知道它需要多少 _二进制_ 位。因此，我们让 FPU 以其自身的高精度计算其二进制值。

我们还使用其他常数：我们将针孔直径乘以 `1000` 以将其从毫米转换为微米。我们在将数字舍入到四位有效数字时将数字与 `10000` 比较。因此，我们将 `1000` 和 `10000` 都保留在栈上。当然，在将数字舍入到四位时我们重用 `0.1`。

最后但同样重要的是，我们将 `-5` 保留在栈上。我们需要它来缩放 F 数的平方，而非将其除以 `32`。我们最后加载此常数并非巧合。这使得当栈上只有常数时它位于栈顶。因此，当 F 数的平方被缩放时，`-5` 位于 `st(1)`，正是 `fscale` 期望它所在的位置。

从零开始创建某些常数而非从内存加载是很常见的。这就是我们对 `-5` 所做的：

```asm
	fld1			; TOS =  1
	fadd	st0, st0	; TOS =  2
	fadd	st0, st0	; TOS =  4
	fld1			; TOS =  1
	faddp	st1, st0	; TOS =  5
	fchs			; TOS = -5
```

我们可以将所有这些优化概括为一条规则：_将重复使用的值保留在栈上！_

> **技巧**
> _PostScript®_ 是一种面向栈的编程语言。关于 PostScript® 的书籍比关于 FPU 汇编语言的书籍多得多：掌握 PostScript® 将帮助你掌握 FPU。

### pinhole——代码

```asm
;;;;;;; pinhole.asm ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; 查找针孔相机构造和使用的各种参数
;
; Started:	 9-Jun-2001
; Updated:	10-Jun-2001
;
; Copyright (c) 2001 G. Adam Stanislav
; All rights reserved.
;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

%include	'system.inc'

%define	BUFSIZE	2048

section	.data
align 4
ten	dd	10
thousand	dd	1000
tthou	dd	10000
fd.in	dd	stdin
fd.out	dd	stdout
envar	db	'PINHOLE='	; 恰好 8 字节，即 2 个 dword
pinhole	db	'04,', 		; Bender 的常数 (0.04)
connors	db	'037', 0Ah	; Connors 的常数
usg	db	'Usage: pinhole [-b] [-c] [-e] [-p <value>] [-o <outfile>] [-i <infile>]', 0Ah
usglen	equ	$-usg
iemsg	db	"pinhole: Can't open input file", 0Ah
iemlen	equ	$-iemsg
oemsg	db	"pinhole: Can't create output file", 0Ah
oemlen	equ	$-oemsg
pinmsg	db	"pinhole: The PINHOLE constant must not be 0", 0Ah
pinlen	equ	$-pinmsg
toobig	db	"pinhole: The PINHOLE constant may not exceed 18 decimal places", 0Ah
biglen	equ	$-toobig
huhmsg	db	9, '???'
separ	db	9, '???'
sep2	db	9, '???'
sep3	db	9, '???'
sep4	db	9, '???', 0Ah
huhlen	equ	$-huhmsg
header	db	'focal length in millimeters,pinhole diameter in microns,'
	db	'F-number,normalized F-number,F-5.6 multiplier,stops '
	db	'from F-5.6', 0Ah
headlen	equ	$-header

section .bss
ibuffer	resb	BUFSIZE
obuffer	resb	BUFSIZE
dbuffer	resb	20		; 十进制输入缓冲区
bbuffer	resb	10		; BCD 缓冲区

section	.text
align 4
huh:
	call	write
	push	dword huhlen
	push	dword huhmsg
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	ret

align 4
perr:
	push	dword pinlen
	push	dword pinmsg
	push	dword stderr
	sys.write
	push	dword 4		; 返回失败
	sys.exit

align 4
consttoobig:
	push	dword biglen
	push	dword toobig
	push	dword stderr
	sys.write
	push	dword 5		; 返回失败
	sys.exit

align 4
ierr:
	push	dword iemlen
	push	dword iemsg
	push	dword stderr
	sys.write
	push	dword 1		; 返回失败
	sys.exit

align 4
oerr:
	push	dword oemlen
	push	dword oemsg
	push	dword stderr
	sys.write
	push	dword 2
	sys.exit

align 4
usage:
	push	dword usglen
	push	dword usg
	push	dword stderr
	sys.write
	push	dword 3
	sys.exit

align 4
global	_start
_start:
	add	esp, byte 8	; 丢弃 argc 和 argv[0]
	sub	esi, esi

.arg:
	pop	ecx
	or	ecx, ecx
	je	near .getenv		; 没有更多参数

	; ECX 包含指向参数的指针
	cmp	byte [ecx], '-'
	jne	usage

	inc	ecx
	mov	ax, [ecx]
	inc	ecx

.o:
	cmp	al, 'o'
	jne	.i

	; 确保没有被要求两次输出文件
	cmp	dword [fd.out], stdout
	jne	usage

	; 查找输出文件路径——它在 [ECX+1]，
	; 即 -ofile --
	; 或者在下一个参数中，
	; 即 -o file

	or	ah, ah
	jne	.openoutput
	pop	ecx
	jecxz	usage

.openoutput:
	push	dword 420	; 文件模式（八进制 644）
	push	dword 0200h | 0400h | 01h
	; O_CREAT | O_TRUNC | O_WRONLY
	push	ecx
	sys.open
	jc	near oerr

	add	esp, byte 12
	mov	[fd.out], eax
	jmp	short .arg

.i:
	cmp	al, 'i'
	jne	.p

	; 确保没有被要求两次
	cmp	dword [fd.in], stdin
	jne	near usage

	; 查找输入文件路径
	or	ah, ah
	jne	.openinput
	pop	ecx
	or	ecx, ecx
	je near usage

.openinput:
	push	dword 0		; O_RDONLY
	push	ecx
	sys.open
	jc	near ierr		; 打开失败

	add	esp, byte 8
	mov	[fd.in], eax
	jmp	.arg

.p:
	cmp	al, 'p'
	jne	.c
	or	ah, ah
	jne	.pcheck

	pop	ecx
	or	ecx, ecx
	je	near usage

	mov	ah, [ecx]

.pcheck:
	cmp	ah, '0'
	jl	near usage
	cmp	ah, '9'
	ja	near usage
	mov	esi, ecx
	jmp	.arg

.c:
	cmp	al, 'c'
	jne	.b
	or	ah, ah
	jne	near usage
	mov	esi, connors
	jmp	.arg

.b:
	cmp	al, 'b'
	jne	.e
	or	ah, ah
	jne	near usage
	mov	esi, pinhole
	jmp	.arg

.e:
	cmp	al, 'e'
	jne	near usage
	or	ah, ah
	jne	near usage
	mov	al, ','
	mov	[huhmsg], al
	mov	[separ], al
	mov	[sep2], al
	mov	[sep3], al
	mov	[sep4], al
	jmp	.arg

align 4
.getenv:
	; 如果 ESI = 0，我们没有 -p 参数，
	; 需要检查环境中的 "PINHOLE="
	or	esi, esi
	jne	.init

	sub	ecx, ecx

.nextenv:
	pop	esi
	or	esi, esi
	je	.default	; 未找到 PINHOLE 环境变量

	; 检查此环境变量是否以 'PINHOLE=' 开头
	mov	edi, envar
	mov	cl, 2		; 'PINHOLE=' 为 2 个 dword 长
rep	cmpsd
	jne	.nextenv

	; 检查后面是否跟着数字
	mov	al, [esi]
	cmp	al, '0'
	jl	.default
	cmp	al, '9'
	jbe	.init
	; 落入

align 4
.default:
	; 我们到这里是因为没有 -p 参数，
	; 且未找到 PINHOLE 环境变量。
	mov	esi, pinhole
	; 落入

align 4
.init:
	sub	eax, eax
	sub	ebx, ebx
	sub	ecx, ecx
	sub	edx, edx
	mov	edi, dbuffer+1
	mov	byte [dbuffer], '0'

	; 将针孔常数转换为实数
.constloop:
	lodsb
	cmp	al, '9'
	ja	.setconst
	cmp	al, '0'
	je	.processconst
	jb	.setconst

	inc	dl

.processconst:
	inc	cl
	cmp	cl, 18
	ja	near consttoobig
	stosb
	jmp	short .constloop

align 4
.setconst:
	or	dl, dl
	je	near perr

	finit
	fild	dword [tthou]

	fld1
	fild	dword [ten]
	fdivp	st1, st0

	fild	dword [thousand]
	mov	edi, obuffer

	mov	ebp, ecx
	call	bcdload

.constdiv:
	fmul	st0, st2
	loop	.constdiv

	fld1
	fadd	st0, st0
	fadd	st0, st0
	fld1
	faddp	st1, st0
	fchs

	; 如果正在创建 CSV 文件，
	; 打印表头
	cmp	byte [separ], ','
	jne	.bigloop

	push	dword headlen
	push	dword header
	push	dword [fd.out]
	sys.write

.bigloop:
	call	getchar
	jc	near done

	; 如果收到 '#' 则跳到行尾
	cmp	al, '#'
	jne	.num
	call	skiptoeol
	jmp	short .bigloop

.num:
	; 查看是否得到了一个数字
	cmp	al, '0'
	jl	.bigloop
	cmp	al, '9'
	ja	.bigloop

	; 是的，我们有一个数字
	sub	ebp, ebp
	sub	edx, edx

.number:
	cmp	al, '0'
	je	.number0
	mov	dl, 1

.number0:
	or	dl, dl		; 跳过前导 0
	je	.nextnumber
	push	eax
	call	putchar
	pop	eax
	inc	ebp
	cmp	ebp, 19
	jae	.nextnumber
	mov	[dbuffer+ebp], al

.nextnumber:
	call	getchar
	jc	.work
	cmp	al, '#'
	je	.ungetc
	cmp	al, '0'
	jl	.work
	cmp	al, '9'
	ja	.work
	jmp	short .number

.ungetc:
	dec	esi
	inc	ebx

.work:
	; 现在，做所有的工作
	or	dl, dl
	je	near .work0

	cmp	ebp, 19
	jae	near .toobig

	call	bcdload

	; 计算针孔直径

	fld	st0	; 保存它
	fsqrt
	fmul	st0, st3
	fld	st0
	fmul	st5
	sub	ebp, ebp

	; 舍入到 4 位有效数字
.diameter:
	fcom	st0, st7
	fstsw	ax
	sahf
	jb	.printdiameter
	fmul	st0, st6
	inc	ebp
	jmp	short .diameter

.printdiameter:
	call	printnumber	; 针孔直径

	; 计算 F 数

	fdivp	st1, st0
	fld	st0

	sub	ebp, ebp

.fnumber:
	fcom	st0, st6
	fstsw	ax
	sahf
	jb	.printfnumber
	fmul	st0, st5
	inc	ebp
	jmp	short .fnumber

.printfnumber:
	call	printnumber	; F 数

	; 计算标准化 F 数
	fmul	st0, st0
	fld1
	fld	st1
	fyl2x
	frndint
	fld1
	fscale
	fsqrt
	fstp	st1

	sub	ebp, ebp
	call	printnumber

	; 计算从 F-5.6 的时间倍数

	fscale
	fld	st0

	; 舍入到 4 位有效数字
.fmul:
	fcom	st0, st6
	fstsw	ax
	sahf

	jb	.printfmul
	inc	ebp
	fmul	st0, st5
	jmp	short .fmul

.printfmul:
	call	printnumber	; F 倍数

	; 计算从 5.6 的 F 档数

	fld1
	fxch	st1
	fyl2x

	sub	ebp, ebp
	call	printnumber

	mov	al, 0Ah
	call	putchar
	jmp	.bigloop

.work0:
	mov	al, '0'
	call	putchar

align 4
.toobig:
	call	huh
	jmp	.bigloop

align 4
done:
	call	write		; 刷新输出缓冲区

	; 关闭文件
	push	dword [fd.in]
	sys.close

	push	dword [fd.out]
	sys.close

	finit

	; 返回成功
	push	dword 0
	sys.exit

align 4
skiptoeol:
	; 继续读取直到遇到 cr、lf 或 eof
	call	getchar
	jc	done
	cmp	al, 0Ah
	jne	.cr
	ret

.cr:
	cmp	al, 0Dh
	jne	skiptoeol
	ret

align 4
getchar:
	or	ebx, ebx
	jne	.fetch

	call	read

.fetch:
	lodsb
	dec	ebx
	clc
	ret

read:
	jecxz	.read
	call	write

.read:
	push	dword BUFSIZE
	mov	esi, ibuffer
	push	esi
	push	dword [fd.in]
	sys.read
	add	esp, byte 12
	mov	ebx, eax
	or	eax, eax
	je	.empty
	sub	eax, eax
	ret

align 4
.empty:
	add	esp, byte 4
	stc
	ret

align 4
putchar:
	stosb
	inc	ecx
	cmp	ecx, BUFSIZE
	je	write
	ret

align 4
write:
	jecxz	.ret	; 无内容可写
	sub	edi, ecx	; 缓冲区起始位置
	push	ecx
	push	edi
	push	dword [fd.out]
	sys.write
	add	esp, byte 12
	sub	eax, eax
	sub	ecx, ecx	; 缓冲区现在为空
.ret:
	ret

align 4
bcdload:
	; EBP 包含 dbuffer 中的字符数
	push	ecx
	push	esi
	push	edi

	lea	ecx, [ebp+1]
	lea	esi, [dbuffer+ebp-1]
	shr	ecx, 1

	std

	mov	edi, bbuffer
	sub	eax, eax
	mov	[edi], eax
	mov	[edi+4], eax
	mov	[edi+2], ax

.loop:
	lodsw
	sub	ax, 3030h
	shl	al, 4
	or	al, ah
	mov	[edi], al
	inc	edi
	loop	.loop

	fbld	[bbuffer]

	cld
	pop	edi
	pop	esi
	pop	ecx
	sub	eax, eax
	ret

align 4
printnumber:
	push	ebp
	mov	al, [separ]
	call	putchar

	; 打印 TOS 处的整数
	mov	ebp, bbuffer+9
	fbstp	[bbuffer]

	; 检查符号
	mov	al, [ebp]
	dec	ebp
	or	al, al
	jns	.leading

	; 我们得到了一个负数（应该永远不会发生）
	mov	al, '-'
	call	putchar

.leading:
	; 跳过前导零
	mov	al, [ebp]
	dec	ebp
	or	al, al
	jne	.first
	cmp	ebp, bbuffer
	jae	.leading

	; 我们在这里是因为结果为 0。
	; 打印 '0' 并返回
	mov	al, '0'
	jmp	putchar

.first:
	; 我们找到了第一个非零。
	; 但它仍然是压缩的
	test	al, 0F0h
	jz	.second
	push	eax
	shr	al, 4
	add	al, '0'
	call	putchar
	pop	eax
	and	al, 0Fh

.second:
	add	al, '0'
	call	putchar

.next:
	cmp	ebp, bbuffer
	jb	.done

	mov	al, [ebp]
	push	eax
	shr	al, 4
	add	al, '0'
	call	putchar
	pop	eax
	and	al, 0Fh
	add	al, '0'
	call	putchar

	dec	ebp
	jmp	short .next

.done:
	pop	ebp
	or	ebp, ebp
	je	.ret

.zeros:
	mov	al, '0'
	call	putchar
	dec	ebp
	jne	.zeros

.ret:
	ret
```

此代码遵循与我们之前见过的所有其他过滤器相同的格式，但有一个微妙的例外：

> 我们不再假设输入结束就意味着要做的事情结束了——这在 _面向字符_ 的过滤器中是我们理所当然认为的。
>
> 此过滤器不处理字符。它处理一种 _语言_（尽管非常简单，只包含数字）。
>
> 当我们没有更多输入时，它可能意味着两件事之一：
>
> * 我们完成了，可以退出。这与之前相同。
> * 我们读取的最后一个字符是数字。我们已将其存储在 ASCII 转浮点转换缓冲区的末尾。我们现在需要将该缓冲区的内容转换为数字并写入输出的最后一行。
>
> 为此，我们修改了 `getchar` 和 `read` 例程，在从输入中获取另一个字符时返回 `carry flag` _清除_ 状态，在没有更多输入时返回 `carry flag` _设置_ 状态。
>
> 当然，我们仍然在使用汇编语言的魔法来做到这一点！仔细看看 `getchar`。它 _始终_ 以 `carry flag` _清除_ 状态返回。
>
> 然而，我们的主代码依赖 `carry flag` 来告诉它何时退出——而且有效。
>
> 魔法在于 `read`。每当它从系统接收到更多输入时，它就直接返回到 `getchar`，后者从输入缓冲区中取出一个字符，_清除_ `carry flag` 并返回。
>
> 但当 `read` 没有从系统收到更多输入时，它 _不_ 返回到 `getchar`。相反，`add esp, byte 4` 操作码将 `4` 加到 `ESP`，_设置_ `carry flag`，然后返回。
>
> 那么它返回到哪里？每当程序使用 `call` 操作码时，微处理器会 `push` 返回地址，即将其存储在栈顶（不是 FPU 栈，而是系统栈，位于内存中）。当程序使用 `ret` 操作码时，微处理器从栈中 `pop` 返回值，并跳转到存储在那里的地址。
>
> 但由于我们向 `ESP`（栈指针寄存器）加了 `4`，我们实际上给了微处理器轻微的 _失忆_：它不再记得是 `getchar` `call` 了 `read`。
>
> 由于 `getchar` 在 `call` `read` 之前从未 `push` 过任何东西，栈顶现在包含的是 `call` 了 `getchar` 的任何人或任何东西的返回地址。就那个调用者而言，他 `call` 了 `getchar`，而 `getchar` 以 `carry flag` 设置 `ret`urn 了！

除此之外，`bcdload` 例程陷入了大小端之间的"小人国之争"。

它正在将数字的文本表示转换为该数字本身：文本以大端序存储，但 _压缩十进制_ 是小端序。

为解决此冲突，我们尽早使用 `std` 操作码。我们稍后用 `cld` 取消它：在 `std` 激活期间，不要 `call` 任何可能依赖于 _方向标志_ 默认设置的东西，这非常重要。

此代码中的其他一切都应该很清楚，前提是你已阅读了它之前的整篇文章。

这是"编程需要大量思考而只需少量编码"这一格言的经典例证。一旦我们思考了每一个微小的细节，代码几乎是自动完成的。

### 使用 pinhole

因为我们决定让程序 _忽略_ 除数字以外的任何输入（甚至注释中的数字），我们实际上可以执行 _文本查询_。我们 _不必_ 这样做，但我们可以。

依我之见，形成文本查询而非必须遵循非常严格的语法，使软件对用户更加友好。

假设我们想构建一个使用 4x5 英寸胶片的针孔相机。该胶片的标准焦距约为 150 毫米。我们希望 _微调_ 焦距，使针孔直径尽可能是一个整数。让我们还假设我们对相机相当熟悉，但对计算机有些畏惧。与其只需输入一堆数字，我们更想 _问_ 几个问题。

我们的会话可能是这样的：

```sh
% pinhole

Computer,

What size pinhole do I need for the focal length of 150?
150	490	306	362	2930	12
Hmmm... How about 160?
160	506	316	362	3125	12
Let's make it 155, please.
155	498	311	362	3027	12
Ah, let's try 157...
157	501	313	362	3066	12
156?
156	500	312	362	3047	12
That's it! Perfect! Thank you very much!
^D
```

我们发现，虽然焦距为 150 时针孔直径应为 490 微米（即 0.49 毫米），但如果我们选择几乎相同的焦距 156 毫米，我们可以使用恰好半毫米的针孔直径。

### 脚本编程

因为我们选择 `#` 字符来表示注释的开始，我们可以将针孔软件视为 _脚本语言_。

你可能见过以如下方式开头的 shell _脚本_：

```sh
#! /bin/sh
```

……或……

```sh
#!/bin/sh
```

……因为 `#!` 后的空格是可选的。

每当 UNIX 被要求运行以 `#!` 开头的可执行文件时，它假定该文件是脚本。它将该命令添加到脚本第一行的其余部分，并尝试执行它。

假设我们已将 pinhole 安装在 **/usr/local/bin/** 中，我们现在可以编写一个脚本来计算 120 胶片常用的各种焦距所适合的各种针孔直径。

脚本可能如下所示：

```sh
#! /usr/local/bin/pinhole -b -i
# 查找 120 胶片的最佳针孔直径

### 标准
80

### 广角
30, 40, 50, 60, 70

### 长焦
100, 120, 140
```

因为 120 是中画幅胶片，我们可以将此文件命名为 medium。

我们可以将其权限设置为可执行，并像运行程序一样运行它：

```sh
% chmod 755 medium
% ./medium
```

UNIX 会将最后一条命令解释为：

```sh
% /usr/local/bin/pinhole -b -i ./medium
```

它将运行该命令并显示：

```sh
80	358	224	256	1562	11
30	219	137	128	586	9
40	253	158	181	781	10
50	283	177	181	977	10
60	310	194	181	1172	10
70	335	209	181	1367	10
100	400	250	256	1953	11
120	438	274	256	2344	11
140	473	296	256	2734	11
```

现在，让我们输入：

```sh
% ./medium -c
```

UNIX 会将其视为：

```sh
% /usr/local/bin/pinhole -b -i ./medium -c
```

这给了它两个冲突的选项：`-b` 和 `-c`（使用 Bender 常数和使用 Connors 常数）。我们编程为后面的选项覆盖前面的——我们的程序将使用 Connors 常数计算一切：

```sh
80	331	242	256	1826	11
30	203	148	128	685	9
40	234	171	181	913	10
50	262	191	181	1141	10
60	287	209	181	1370	10
70	310	226	256	1598	11
100	370	270	256	2283	11
120	405	296	256	2739	11
140	438	320	362	3196	12
```

我们决定最终还是使用 Bender 常数。我们想将其值保存为逗号分隔文件：

```sh
% ./medium -b -e > bender
% cat bender
focal length in millimeters,pinhole diameter in microns,F-number,normalized F-number,F-5.6 multiplier,stops from F-5.6
80,358,224,256,1562,11
30,219,137,128,586,9
40,253,158,181,781,10
50,283,177,181,977,10
60,310,194,181,1172,10
70,335,209,181,1367,10
100,400,250,256,1953,11
120,438,274,256,2344,11
140,473,296,256,2734,11
%
```

## 14. 注意事项

在 MS-DOS 和 Windows 下"成长"的汇编语言程序员往往喜欢走捷径。读取键盘扫描码和直接写入视频内存是两个典型的例子，这些做法在 MS-DOS 下不仅不被反对，反而被认为是正确的做法。

原因？PC BIOS 和 MS-DOS 在执行这些操作时以缓慢著称。

你可能会想在 UNIX 环境中继续类似的做法。例如，我曾看到一个网站解释如何在某个流行的 UNIX 克隆上访问键盘扫描码。

在 UNIX 环境中，这通常是 _非常糟糕的主意_！让我解释原因。

### UNIX 是受保护的

首先，它可能根本不可能。UNIX 运行在保护模式下。只有内核和设备驱动程序才被允许直接访问硬件。也许某个特定的 UNIX 克隆会允许你读取键盘扫描码，但真正的 UNIX 操作系统很可能不会。即使某个版本允许你这样做，下一个版本也可能不允许，所以你精心制作的软件可能一夜之间就过时了。

### UNIX 是一种抽象

但不直接尝试访问硬件有一个更重要的原因（当然，除非你在编写设备驱动程序），即使在允许你这样做的类 UNIX 系统上也是如此：

_UNIX 是一种抽象！_

MS-DOS 和 UNIX 的设计哲学存在重大差异。MS-DOS 被设计为单用户系统。它运行在直接连接了键盘和视频屏幕的计算机上。用户输入几乎可以保证来自那个键盘。你的程序输出几乎总是出现在那个屏幕上。

在 UNIX 下，这 _永远_ 无法保证。UNIX 用户经常通过管道和重定向程序的输入和输出：

```sh
% program1 | program2 | program3 > file1
```

如果你编写了 program2，你的输入不来自键盘而来自 program1 的输出。同样，你的输出不会去屏幕而是成为 program3 的输入，而 program3 的输出又去了 `file1`。

还有更多！即使你确保输入来自终端且输出去终端，也无法保证终端是一台 PC：它的视频内存可能不在你期望的位置，它的键盘也可能不产生 PC 风格的扫描码。它可能是 Macintosh 或任何其他计算机。

现在你可能摇头：我的软件是用 PC 汇编语言写的，怎么能在 Macintosh 上运行？但我没有说你的软件会在 Macintosh 上运行，只是说它的终端可能是 Macintosh。

在 UNIX 下，终端不必直接连接到运行你软件的计算机上，它甚至可以在另一个大洲，甚至另一个星球。澳大利亚的 Macintosh 用户通过 telnet 连接到北美（或任何其他地方）的 UNIX 系统是完全可能的。然后软件在一台计算机上运行，而终端在另一台计算机上：如果你试图读取扫描码，你会得到错误的输入！

其他硬件也是如此：你正在读取的文件可能在你无法直接访问的磁盘上。你正在读取图像的相机可能在航天飞机上，通过卫星连接到你。

这就是为什么在 UNIX 下你绝不能对数据来自何处和去向何处做任何假设。始终让系统处理对硬件的物理访问。

> **注意**
> 这些是注意事项，而非绝对规则。例外是可能的。例如，如果文本编辑器已确定它在本地机器上运行，它可能想直接读取扫描码以获得更好的控制。我提到这些注意事项不是为了告诉你该做什么或不该做什么，只是让你了解如果你刚从 MS-DOS 转到 UNIX 时等待你的某些陷阱。当然，有创造力的人经常打破规则，只要他们知道自己在打破规则以及为什么打破，这没问题。

## 15. 致谢

没有 [FreeBSD 技术讨论邮件列表](https://lists.freebsd.org/subscription/freebsd-hackers) 中许多经验丰富的 FreeBSD 程序员的帮助，本教程永远不可能完成。他们中许多人耐心地回答了我的问题，并在我探索 UNIX 系统编程的内部机制（特别是 FreeBSD 的内部机制）的尝试中为我指明了正确的方向。

Thomas M. Sommers 为我打开了大门。他创建的 [How do I write "Hello, world" in FreeBSD assembler?](https://web.archive.org/web/20090914064615/http://www.codebreakers-journal.com/content/view/262/27) 网页是我第一次接触 FreeBSD 下汇编语言编程的示例。

Jake Burkholder 耐心回答了我所有的问题，并为我提供了汇编语言源代码示例，让这扇门一直敞开。

版权所有 © 2000-2001 G. Adam Stanislav。保留所有权利。
