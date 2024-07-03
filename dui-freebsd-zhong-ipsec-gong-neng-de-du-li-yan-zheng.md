# FreeBSD 中 IPsec 功能的独立验证

<details open="" data-immersive-translate-walked="1f4aa8c9-c474-4ebb-9a79-6d7b5212ecb2"><summary data-immersive-translate-walked="1f4aa8c9-c474-4ebb-9a79-6d7b5212ecb2" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Motif，OSF/1 和 UNIX 是 The Open Group 在美国和其他国家的注册商标，而 IT DialTone 和 The Open Group 则是商标。

许多制造商和卖方用来区分其产品的名称被声称为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目意识到商标声明，则这些名称后面会跟上“™”或“®”符号。

</details>

 摘要

你安装了 IPsec，并且它似乎在工作。你怎么知道？我描述了一种实验验证 IPsec 是否工作的 方法。

---

## 1. 问题

首先，让我们假设您已经安装了 IPsec。您如何知道这是一个警告呢？当您将其错配置时，您的连接将无法工作，并且当您最终设置正确时它将工作。 netstat(1) 将列出它。但您能独立确认它吗？

## 2. 解决方案

第一，一些与加密相关的信息理论：

1. 加密数据是均匀分布的，即每个符号具有最大熵；
2. 原始，未压缩的数据通常是冗余的，即具有次最大熵。

假设您可以测量数据通过您的网络接口的熵。然后，您可以看到未加密数据和加密数据之间的差异。即使“加密模式”中的某些数据未加密，也是如此---因为如果数据包可路由，则最外层的 IP 标头必须是加密的。

### 2.1. 必须

Ueli Maurer 的“用于随机比特生成器的通用统计测试”(必须)可以快速测量样本的熵。它使用一种类似压缩的算法。 Maurer 的通用统计测试(块大小=8 位)用于一种变体，该变体测量文件的连续(~四分之一兆字节)块。

### 2.2. Tcpdump

我们还需要一种捕获原始网络数据的方法。一个名为 tcpdump(1)的程序可以让您这样做，如果您已经在 src/sys/i386/conf/KERNELNAME 中启用了伯克利包过滤器接口。

 命令：

```
 tcpdump -c 4000 -s 10000 -w dumpfile.bin
```

将捕获 4000 个原始数据包到 dumpfile.bin。在此示例中，每个数据包最多捕获 10,000 字节。

## 3. 实验

这是实验:

1. 打开一个窗口到一个 IPsec 主机，另一个窗口到一个不安全的主机。
2. 现在开始 Tcpdump。
3. 在“安全”窗口中运行 UNIX®命令 yes(1)，它将流式传输 y 字符。过一会儿，停止这个。切换到不安全的窗口，并重复。过一会儿，停止。
4. 现在在捕获的数据包上运行莫雷尔通用统计测试（块大小=8 位）。你应该看到类似以下的内容。需要注意的重要事项是，安全连接的期望值为 93%（6.7），实际值为（7.18），而“普通”连接的期望值为 29%（2.1）。

    ```
    % tcpdump -c 4000 -s 10000 -w ipsecdemo.bin
    % uliscan ipsecdemo.bin
    Uliscan 21 Dec 98
    L=8 256 258560
    Measuring file ipsecdemo.bin
    Init done
    Expected value for L=8 is 7.1836656
    6.9396 --------------------------------------------------------
    6.6177 -----------------------------------------------------
    6.4100 ---------------------------------------------------
    2.1101 -----------------
    2.0838 -----------------
    2.0983 -----------------
    ```

## 4. 警告

这个实验表明，IPsec 似乎在分布有效载荷数据时是均匀的，就像加密应该做的那样。然而，这里描述的实验不能检测到系统中可能存在的许多缺陷（其中没有我有任何证据的）。这些缺陷包括密钥生成或交换不良、数据或密钥对其他人可见、使用弱算法、内核篡改等。研究源代码，了解代码。

## 5. IPsec---定义

IPv4 的 Internet 协议安全扩展；IPv6 所需的。用于在 IP（主机到主机）级别协商加密和认证的协议。SSL 仅保护一个应用程序套接字；SSH 仅保护登录；PGP 仅保护特定文件或消息。IPsec 加密两个主机之间的所有内容。

## 6. 安装 IPsec

大多数现代版本的 FreeBSD 在其基础源中具有 IPsec 支持。因此，您需要在内核配置中包含 IPSEC 选项，并在重新编译和重新安装内核后，使用 setkey(8)命令配置 IPsec 连接。

FreeBSD 手册中提供了关于在 FreeBSD 上运行 IPsec 的全面指南。

## 7. src/sys/i386/conf/KERNELNAME

需要出现在内核配置文件中，以使用 tcpdump(1)捕获网络数据。在添加此内容后，请运行 config(8)，然后重新构建和重新安装。

```
device	bpf
```

## 8. Maurer 的通用统计测试（块大小=8 位）

您可以在此链接找到相同的代码。

```
/*
  ULISCAN.c   ---blocksize of 8

  1 Oct 98
  1 Dec 98
  21 Dec 98       uliscan.c derived from ueli8.c

  This version has // comments removed for Sun cc

  This implements Ueli M Maurer's "Universal Statistical Test for Random
  Bit Generators" using L=8

  Accepts a filename on the command line; writes its results, with other
  info, to stdout.

  Handles input file exhaustion gracefully.

  Ref: J. Cryptology v 5 no 2, 1992 pp 89-105
  also on the web somewhere, which is where I found it.

  -David Honig
  honig@sprynet.com

  Usage:
  ULISCAN filename
  outputs to stdout
*/

#define L 8
#define V (1<<L)
#define Q (10*V)
#define K (100   *Q)
#define MAXSAMP (Q + K)

#include <stdio.h>
#include <math.h>

int main(argc, argv)
int argc;
char **argv;
{
  FILE *fptr;
  int i,j;
  int b, c;
  int table[V];
  double sum = 0.0;
  int iproduct = 1;
  int run;

  extern double   log(/* double x */);

  printf("Uliscan 21 Dec 98 \nL=%d %d %d \n", L, V, MAXSAMP);

  if (argc < 2) {
    printf("Usage: Uliscan filename\n");
    exit(-1);
  } else {
    printf("Measuring file %s\n", argv[1]);
  }

  fptr = fopen(argv[1],"rb");

  if (fptr == NULL) {
    printf("Can't find %s\n", argv[1]);
    exit(-1);
  }

  for (i = 0; i < V; i++) {
    table[i] = 0;
  }

  for (i = 0; i < Q; i++) {
    b = fgetc(fptr);
    table[b] = i;
  }

  printf("Init done\n");

  printf("Expected value for L=8 is 7.1836656\n");

  run = 1;

  while (run) {
    sum = 0.0;
    iproduct = 1;

    if (run)
      for (i = Q; run && i < Q + K; i++) {
        j = i;
        b = fgetc(fptr);

        if (b < 0)
          run = 0;

        if (run) {
          if (table[b] > j)
            j += K;

          sum += log((double)(j-table[b]));

          table[b] = i;
        }
      }

    if (!run)
      printf("Premature end of file; read %d blocks.\n", i - Q);

    sum = (sum/((double)(i - Q))) /  log(2.0);
    printf("%4.4f ", sum);

    for (i = 0; i < (int)(sum*8.0 + 0.50); i++)
      printf("-");

    printf("\n");

    /* refill initial table */
    if (0) {
      for (i = 0; i < Q; i++) {
        b = fgetc(fptr);
        if (b < 0) {
          run = 0;
        } else {
          table[b] = i;
        }
      }
    }
  }
}
```
