# FreeBSD 对闰秒的支持

## 1. 介绍

闰秒是每年特定时间对 UTC 进行的一秒调整，以将原子时间尺度与地球自转的变化同步。本文介绍了 FreeBSD 与闰秒的交互方式。

截至目前撰写本文时，下一个闰秒将在 2015 年 6 月 30 日 23:59:60 UTC 发生。该闰秒将在北美、南美和亚太地区的工作日发生。

闰秒由 IERS 在公告 C 上宣布。

RFC 7164 中介绍了标准闰秒行为。还请参阅 time2posix(3)。

## FreeBSD 上的默认闰秒处理。

用 FreeBSD 默认使用的 POSIX 时间规则结合 NTP 处理闰秒是最简单的方法。当 ntpd(8) 在运行并且时间与正确处理闰秒的上游 NTP 服务器同步时，闰秒将导致系统时间自动重复当天的最后一秒。不需要进行其他调整。

如果上游 NTP 服务器没有正确处理闰秒，ntpd(8)将在错误的上游服务器注意到并自行调整之后将时间调整一秒。

如果没有使用 NTP，则在闰秒过去后需要手动调整系统时钟。

## 3. 注意事项

世界各地同时插入闰秒：协调世界时午夜。在日本这是上午,太平洋上午,美洲下午晚些时候,欧洲晚上。

我们相信并期待，如果 FreeBSD 提供正确且稳定的 NTP 服务，在这个闰秒期间将像在以前的闰秒期间一样正常工作。

但是，我们要警告的是，几乎没有应用程序曾经向内核询问过闰秒。我们的经验表明，按照设计，闰秒实质上是重播闰秒之前的那一秒，这对大多数应用程序员来说是个惊讶。

其他操作系统和其他计算机可能会或可能不会像 FreeBSD 一样处理闰秒，而没有正确和稳定的 NTP 服务的系统将完全不了解闰秒。

计算机因为闰秒而崩溃并非闻所未闻，经验表明，所有公共 NTP 服务器中有很大一部分可能会处理和宣布闰秒时出现错误。

请尽量确保由于闰秒不会发生可怕的事情。

## 4. 测试

可以测试是否会使用闰秒。由于 NTP 的性质，测试可能会在闰秒之前的 24 小时内有效。一些主要的参考时钟源仅在事件发生前一个小时宣布闰秒。查询 NTP 守护程序：

```sh
% ntpq -c 'rv 0 leap'
```

包含 leap_add_sec 的输出表明对闰秒的正确支持。在距离闰秒前 24 小时的时间段内，或者闰秒过去后，将显示 leap_none 。

## 5. 结论

在实践中，在 FreeBSD 上闰秒通常不是问题。我们希望这个概述可以帮助澄清对于闰秒事件可以期待什么，以及如何使闰秒事件更顺利进行。
