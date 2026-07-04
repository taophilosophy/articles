# 对 FreeBSD 中 IPsec 功能的独立验证

- 原文：[Independent Verification of IPsec Functionality in FreeBSD](https://docs.freebsd.org/en/articles/ipsec-must/)

## 摘要

你安装了 IPsec，并且它似乎在工作。你如何确认这一点呢？我将介绍一种实验方法来验证 IPsec 是否工作。


## 1. 问题

首先，假设你已经完成 [安装 IPsec](https://docs.freebsd.org/en/articles/ipsec-must/#ipsec-install)。那么，你如何确认它是否真的有效（参见 [警告](https://docs.freebsd.org/en/articles/ipsec-must/#caveat)）？当然，如果配置错误，你的连接无法工作，而最终正确配置后它会正常工作。[netstat(1)](https://man.freebsd.org/cgi/man.cgi?query=netstat&sektion=1&format=html) 会列出它。但你能否独立确认这一点？

## 2. 解决方案

首先，介绍一些与加密相关的信息论：

1. 加密数据是均匀分布的，即每个符号具有最大的熵；
2. 原始未压缩的数据通常是冗余的，即具有次最大熵。

假设你可以测量通过你的网络接口的进出数据的熵。那么你就可以看到未加密数据与加密数据之间的区别。即使在“加密模式”中某些数据没有被加密——因为如果数据包需要被路由，最外层的 IP 头部必须是未加密的——这也是成立的。

### 2.1. MUST

Ueli Maurer 的《随机比特生成器的通用统计测试》([MUST](https://web.archive.org/web/20011115002319/http://www.geocities.com/SiliconValley/Code/4704/universal.pdf)) 可以快速测量样本的熵。它使用类似压缩的算法。[Maurer’s Universal Statistical Test (对于块大小 8 位)](https://docs.freebsd.org/en/articles/ipsec-must/#code) 是一个变种，用于测量文件的连续（约四分之一兆字节）块。

### 2.2. Tcpdump

我们还需要一种捕获原始网络数据的方法。名为 [tcpdump(1)](https://man.freebsd.org/cgi/man.cgi?query=tcpdump&sektion=1&format=html) 的程序可以做到这一点，前提是你在 [src/sys/i386/conf/KERNELNAME](https://docs.freebsd.org/en/articles/ipsec-must/#kernel) 中启用了 *Berkeley 数据包过滤器* 接口。

命令：

```sh
tcpdump -c 4000 -s 10000 -w dumpfile.bin
```

将捕获 4000 个原始数据包并保存到 *dumpfile.bin* 文件中。在此示例中，每个数据包最多捕获 10,000 字节。

## 3. 实验

这是实验步骤：

1. 打开一个连接到 IPsec 主机的窗口和一个连接到不安全主机的窗口。
2. 现在启动 [Tcpdump](https://docs.freebsd.org/en/articles/ipsec-must/#tcpdump)。
3. 在“安全”窗口中，运行 UNIX® 命令 [yes(1)](https://man.freebsd.org/cgi/man.cgi?query=yes&sektion=1&format=html)，该命令将流式输出 `y` 字符。运行一段时间后停止。在不安全的窗口中，重复该操作。运行一段时间后停止。
4. 然后在捕获的数据包上运行 [Maurer’s Universal Statistical Test (对于块大小 8 位)](https://docs.freebsd.org/en/articles/ipsec-must/#code)。你应该会看到类似以下的输出。需要注意的是，安全连接的熵值为期望值（7.18）的 93%（6.7），而“正常”连接的熵值为期望值的 29%（2.1）。

   ```sh
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

该实验表明，IPsec *确实* 似乎将负载数据 *均匀* 分布，正如加密应该做的那样。然而，本文所述的实验 *不能* 检测出系统中的许多潜在缺陷（目前我没有任何证据表明存在这些缺陷）。这些缺陷包括糟糕的密钥生成或交换、数据或密钥被他人看到、使用弱算法、内核被篡改等。请研究源代码，了解代码的实现。

## 5. IPsec 定义

IPsec 是对 IPv4 的安全扩展；IPv6 中是必需的。它是一种在 IP（主机到主机）层面上协商加密和身份验证的协议。SSL 仅保护一个应用程序套接字；SSH 仅保护登录；PGP 仅保护指定的文件或消息。IPsec 加密主机之间的一切。

## 6. 安装 IPsec

大多数现代版本的 FreeBSD 在其基础源代码中就支持 IPsec。因此，你需要在内核配置中包括 `IPSEC` 选项，并在重新构建并重新安装内核后，使用 [setkey(8)](https://man.freebsd.org/cgi/man.cgi?query=setkey&sektion=8&format=html) 命令配置 IPsec 连接。

关于在 FreeBSD 上运行 IPsec 的完整指南，请参阅 [通过 IPsec 实现 VPN](https://docs.freebsd.org/en/articles/vpn-ipsec/)。

## 7. src/sys/i386/conf/KERNELNAME

要使用 [tcpdump(1)](https://man.freebsd.org/cgi/man.cgi?query=tcpdump&sektion=1&format=html) 捕获网络数据，必须在内核配置文件中包含此项。添加此项后，请确保运行 [config(8)](https://man.freebsd.org/cgi/man.cgi?query=config&sektion=8&format=html)，并重新构建和重新安装内核。

```sh
device	bpf
```

## 8. Maurer 的通用统计测试（MUST，对于块大小=8 位）

你可以在 [此链接](https://web.archive.org/web/20031204230654/http://www.geocities.com:80/SiliconValley/Code/4704/uliscanc.txt) 找到相同的代码。

```c
/*
  ULISCAN.c   --- 8 位块大小

  1998 年 10 月 1 日
  1998 年 12 月 1 日
  1998 年 12 月 21 日       uliscan.c 源自 ueli8.c

  该版本去除了 // 注释，便于 Sun cc 编译器使用

  该程序实现了 Ueli M Maurer 的 "随机比特生成器的通用统计测试"，使用 L=8

  接受命令行上的文件名；将结果和其他信息写入标准输出（stdout）。

  能够优雅地处理输入文件耗尽的情况。

  参考文献：J. Cryptology v 5 no 2, 1992 页 89-105
  也可以在某些网站上找到，那里是我找到它的地方。

  -David Honig
  honig@sprynet.com

  用法：
  ULISCAN 文件名
  输出到标准输出（stdout）
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

    /* 重新填充初始表格 */
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
