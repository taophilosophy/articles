# FreeBSD 许可政策

- 原文：[FreeBSD Licensing Policy](https://docs.freebsd.org/en/articles/license-guide/)

本政策适用于 FreeBSD 源代码仓库（src）。Ports 和文档仓库有其各自的许可证指南。

## 1. 新文件的首选许可证

本节的其余部分旨在帮助你入门。一般来说，当不确定时，请询问。获得建议比修复源代码仓库要容易得多。FreeBSD 项目使用显式许可证（其中许可证的原文被复制到每个文件中）和分离许可证（其中文件中的标签指定许可证，如本文所述）。仅使用 SPDX 标记（不复制完整许可证文本）是首选方式。

FreeBSD 项目使用以下文本作为首选许可证：

```c
/*
 * Copyright (c) [年份] [你的名字]
 *
 * SPDX-License-Identifier: BSD-2-Clause
 */
```

FreeBSD 项目不允许在新代码中使用“广告条款”（**译者注：即必须声明该软件的开发者或版权所有者**）。由于 FreeBSD 项目有大量贡献者，许多商业供应商遵守这一条款变得困难。如果你在树中有包含广告条款的代码，请考虑切换到没有该条款的许可证。对 FreeBSD 的新贡献应使用 BSD-2-Clause 许可证。

首选顺序是版权声明在前，后跟 SPDX-License-Identifier，但任意顺序都是可以接受的。当 SPDX-License-Identifier 和许可证全文同时存在时，全文优先，SPDX-License-Identifier 仅作为参考信息。当两者同时存在时，SPDX-License-Identifier 通常会位于版权和许可证文本之前。

FreeBSD 项目不鼓励使用全新的许可证和标准许可证的变种。新许可证需要得到 [core@FreeBSD.org](mailto:core@FreeBSD.org) 的批准才能存放在主仓库中。过去，非标准许可证比标准许可证产生了更多问题。非标准许可证的草拟不当通常会导致更多意外后果，因此这些许可证不太可能得到 [core@FreeBSD.org](mailto:core@FreeBSD.org) 的批准。FreeBSD 项目正在标准化使用由 SPDX 发布的 BSD-2-Clause 许可证。

此外，项目政策要求使用某些非 BSD 许可证授权的代码必须放置在仓库的特定部分。对于某些许可证，编译必须是有条件的，或者默认禁用。例如，GENERIC 内核中的静态部分的代码必须根据 BSD 或实质上相似的许可证授权。使用 GPL、APSL、CDDL 等许可证的软件不能编译进静态 GENERIC 内核中。然而，具有这些许可证的代码可以用于预编译的模块中。

开发者需要注意，在开源软件中，正确处理“开放”与正确处理“源代码”同样重要。不当处理知识产权会带来严重后果。任何问题或疑虑应立即报告给 [core@FreeBSD.org](mailto:core@FreeBSD.org)。

## 2. 软件许可证政策

以下部分详细概述了项目的软件许可证政策。在大多数情况下，我们希望开发者阅读、理解并利用本节以上的部分，针对自己的贡献应用适当的许可证。文档的其余部分详细阐述了政策的哲学背景以及政策的详细内容。一如既往，如果下文令人困惑，或者你需要帮助应用这些政策，请联系 [core@FreeBSD.org](mailto:core@FreeBSD.org)。

### 2.1. 指导原则

FreeBSD 项目旨在生产一款完整的、BSD 许可的操作系统，让系统的消费者在没有约束或进一步许可证义务的情况下，生产衍生产品。我们欢迎并非常感激在两条款 BSD 许可证下的变更和新增贡献，并鼓励其他开源项目采用这一许可证。使用 BSD 许可证对于推动先进操作系统技术的采用至关重要，且在许多值得注意的场合中，对新技术的广泛应用起到了关键作用。

然而，我们也承认，确实存在合理的理由让不同许可证授权的软件包含在 FreeBSD 源树中。

我们要求使用某些非 BSD 许可证授权的软件在源树中小心隔离，以便其不会污染仅限 BSD 的组件。这样的谨慎管理有助于许可证的清晰性，并促进生产仅限 BSD 的衍生产品。

除非有特别例外，否则不能用更为限制性的许可证软件替换现有的 BSD 许可组件。我们鼓励 FreeBSD 和第三方开发者寻求在 BSD 许可证下对关键组件进行重新许可、双重许可或重新实现。这样做将使它们更容易被更深入地纳入 FreeBSD 操作系统。

### 2.2. 政策

- 以下是被视为本政策中 BSD 许可证的 SPDX-License-Identifier：
  - BSD-1-Clause
  - BSD-2-Clause
  - BSD-2-Clause-first-lines
  - BSD-3-Clause
  - BSD-3-Clause-acpica
  - BSD-3-Clause-flex
  - BSD-3-Clause-LBNL
  - BSD-3-Clause-Open-MPI
  - BSD-3-Clause-Sun
  - BSD-4.3RENO
  - BSD-4.3TAHOE
  - BSD-Source-beginning-file
  - CMU-Mach
  - ISC
  - MIT
  - MIT-CMU
  - SSH-OpenSSH
  - Vixie-Cron
  - Zlib

- 导入使用 BSD 许可证和 BSD 类似许可证（如下所定义）以外任何许可证授权的新软件，必须事先获得 FreeBSD 核心团队的批准。导入请求必须包含：
  - 新版本或补丁所包含的功能或修复列表，并提供证据表明我们的用户需要这些功能。理想的证据形式是 PR 或邮件列表讨论的引用。
  - 这个过程应该适用于所有软件导入，而不仅仅是那些需要核心团队审查的软件。仅仅因为有新版本的存在，并不能证明就应导入该软件到源代码中。
  - 可能受到影响的 FreeBSD 分支列表。如果范围扩大，则需要重新提交请求并获得 FreeBSD 核心团队的批准。
- 在某些情况下，Apache 许可证 2.0（Apache-2.0）是可以接受的。核心团队必须批准新组件的 Apache 许可证导入，或者现有组件更改为 Apache 许可证。
  - 已批准以下组件使用该许可证：
    - LLVM 工具链和（带有 LLVM 异常）运行时组件。
- BSD+Patent 许可证（BSD-2-Clause-Patent）在某些情况下是可以接受的。核心团队必须批准新组件的 BSD+Patent 许可证导入，或者现有组件更改为 BSD+Patent 许可证。
  - 已批准以下组件使用该许可证：
    - 与 UEFI 功能相关的 EDK2 衍生代码
- 在某些情况下，通用开发和分发许可证（CDDL-1.1）是可以接受的。核心团队必须批准新组件的 CDDL 许可证导入，或者现有组件更改为 CDDL 许可证。
  - 已批准以下组件使用该许可证：
    - DTrace
    - ZFS 文件系统，包括内核支持和用户空间工具
- 最初，FreeBSD 树中的许多条目被标记为 BSD-2-Clause-FreeBSD。然而，SPDX 已将该许可证作为变种废弃；且大部分用此标签标记的文件本应是 BSD-2-Clause，因为 SPDX 的许可证文本是为别的内容准备的。明确禁止使用此标识符添加新文件。
- FreeBSD 项目不鼓励使用全新的许可证和标准许可证的变种。
- 历史上，“All Rights Reserved.”（保留所有权利）这个短语曾出现在所有版权声明中。所有 BSD 版本都包含了该短语，以遵守 [1910 年布宜诺斯艾利斯公约](https://en.wikipedia.org/wiki/Buenos_Aires_Convention)（美洲地区）。随着尼加拉瓜在 2000 年批准了 [伯尔尼公约](https://en.wikipedia.org/wiki/Berne_Convention)，布宜诺斯艾利斯公约和该短语变得过时。因此，FreeBSD 项目建议新代码中省略该短语，并鼓励现有的版权持有人移除此短语。FreeBSD 项目在 2018 年更新了模板，移除了这一短语。

#### 2.2.1. 可接受的许可证

以下许可证被认为是符合本政策的 BSD 类似许可证。任何偏差或使用其他许可证必须获得 FreeBSD 核心团队的批准：

- BSD 许可证的 2 条款版本

```
/*
 * Copyright (c) [年份] [你的名字]
 *
 * SPDX-License-Identifier: BSD-2-Clause
 */
```

- BSD 许可证的 3 条款版本

```
/*
 * Copyright (c) [年份] [你的名字]
 *
 * SPDX-License-Identifier: BSD-3-Clause
 */
```

- ISC 许可证

```
/*
 * Copyright (c) [年份] [版权持有者]
 *
 * SPDX-License-Identifier: ISC
 */
```

- MIT 许可证

```
/*
 * Copyright (c) [年份] [版权持有者]
 *
 * SPDX-License-Identifier: MIT
 */
```

## 3. 软件集合许可证

FreeBSD 项目按照 **COPYRIGHT** 中的描述，将其软件汇编以 BSD-2-Clause 许可证授权。此许可证不会取代各个文件的许可证，如下所述。没有明确许可证的文件将使用 BSD-2-Clause 许可证进行授权。

## 4. 许可证文件位置

为了尽可能符合 [REUSE 软件标准](https://reuse.software/)，所有许可证文件将存储在仓库的 **LICENSES/** 目录下。该顶级目录下有三个子目录。**LICENSES/text/** 子目录包含以分离形式存储的所有 FreeBSD 软件集合中允许的许可证文本。这些文件以 SPDX-License-Identifier 名称后跟 .txt 文件格式存储。**LICENSES/exceptions/** 子目录包含 FreeBSD 软件集合中允许的所有异常的文本，这些文件以异常标识符名称后跟 .txt 格式存储。**LICENSES/other/** 包含以分离形式存储的、在 SPDX-License-Identifier 表达式中被引用的许可证文件，但在其他情况下不允许作为分离许可证。这些文件必须至少出现在 FreeBSD 软件集合中，并应在引用它们的最后一个文件被移除时删除。没有合适 SPDX 匹配许可证的许可证必须放在 **LICENSES/other/** 中，文件名以 LicenseRef- 开头，后跟唯一的 idstring。目前还没有标识出这样的文件，但如果有，将会在此列出。

FreeBSD 项目目前没有使用 `REUSE Software` 标准中陈述的 `DEP5` 文件。FreeBSD 项目尚未根据该标准标记树中的所有文件，如本文后面所述。FreeBSD 项目尚未将这些文件包含在其仓库中，因为此政策仍在发展中。

本节正在编写中。

## 5. 个别文件许可证

FreeBSD 软件集合中的每个文件都有其自己的版权和许可证。它们的标记方式各不相同，本节将进行说明。

版权声明标识了谁声称拥有该文件的法律版权。项目尽力提供这些版权声明。由于版权可以依法转让，因此当前的版权持有者可能与文件中列出的不同。

许可证是贡献者与软件用户之间的法律文件，授权用户在遵守许可证中规定的条款和条件的前提下使用软件的版权部分。许可证在 FreeBSD 软件集合中可以通过两种方式表示。许可证可以在文件中明确表示。当文件中明确授予许可证时，可以根据该许可证使用、复制和修改该文件。许可证也可以通过间接方式表示，其中许可证的文本位于其他地方。为此，项目使用了软件包数据交换（SPDX）许可证标识符，具体如以下小节所述。SPDX 许可证标识符由 Linux 基金会下的 SPDX 工作组管理，并得到了业界各方、工具供应商和法律团队的共同认可。有关详细信息，请参见 [https://spdx.org/](https://spdx.org/) 以及以下小节，了解 FreeBSD 项目如何使用它们。

对于没有明确许可证的修复和增强贡献，贡献者同意按照修改文件所适用的条款对这些更改进行授权。与行业惯例一致，项目政策仅包含集合中文件的重大贡献者的版权声明。

FreeBSD 软件集合中的文件可以分为四种类型：

1. 仅具有明确版权声明和许可证的文件。
2. 同时具有明确版权声明和许可证，以及 SPDX-License-Identifier 标记的文件。
3. 仅具有版权声明和 SPDX-License-Identifier 标记，但没有明确许可证的文件。
4. 完全没有版权声明或许可证的文件。

### 5.1. 仅版权和许可证

FreeBSD 软件集合中的许多文件同时包含版权声明和明确的许可证。在这些情况下，文件中包含的许可证具有优先权。

### 5.2. 带有 SPDX-License-Identifier 表达式的版权和许可证

FreeBSD 软件集合中的某些文件包含版权声明、SPDX-License-Identifier 标记和明确的许可证。明确的许可证优先于 SPDX-License-Identifier 标记。SPDX-License-Identifier 标记是项目对许可证进行描述的尽力尝试，仅供自动化工具参考。有关如何解释该表达式，请参见 [SPDX-License-Identifier 表达式](https://docs.freebsd.org/en/articles/license-guide/#expressions)。

### 5.3. 仅有版权声明和 SPDX-License-Identifier 表达式

树中的一些文件包含分离的许可证。这些文件仅包含版权声明和 SPDX-License-Identifier 表达式，而没有明确的许可证。请参阅 [SPDX-License-Identifier 表达式](https://docs.freebsd.org/en/articles/license-guide/#expressions) 了解如何解释该表达式。注意：项目允许的用于分离许可证的表达式是用于信息目的或由标准定义的表达式的子集。

仅包含 SPDX-License-Identifier 的文件的许可证应理解为：

1. 以文件中的版权声明开始许可证。包括所有版权持有者。
2. 对于每个子表达式，从 **LICENSE/text/**`id`.txt 复制许可证文本。如果有例外情况，请从 **src/share/license/exceptions/**`id`.txt 追加它们。SPDX-License-Identifier 表达式应按 SPDX 标准中的描述进行理解。

其中，`id` 是 [SPDX 标识符](https://spdx.org/licenses/) 或 [许可证例外](https://spdx.org/licenses/exceptions-index.html) 中 `Identifier` 列的 SPDX 简短许可证标识符。如果在 **LICENSE/** 中没有该文件，则无法将该许可证或例外作为本节下的分离许可证。

在阅读与文件分离的许可证文本时，必须考虑以下几点，以确保分离的许可证能够成立：

1. 任何提到版权声明的地方，应当引用根据已授权文件构建的版权声明，而不是许可证文本文件本身中的任何版权声明。许多 SPDX 文件具有示例版权声明，应该理解为仅作为示例。
2. 当许可证文本中提到实体名称时，应理解为适用于已授权文件中的所有版权持有者。例如，BSD-4-Clause 许可证包含短语 "This product includes software developed by the organization"。此处的 'the organization' 应替换为版权持有者。
3. 当 SPDX 提供许可证的变体时，应理解为 **LICENSE/** 文件中的许可证表示所选许可证的确切版本。SPDX 标准旨在匹配许可证族，这些变体有助于匹配 SPDX 组织认为在法律上等同的相似许可证。

对于文本略有变化的许可证，SPDX 提供了匹配的指南。此处的指南不相关。希望使用 **LICENSE/** 中未明确列出的 SPDX 许可证变体的贡献者，不能使用分离选项，必须明确指定许可证。

### 5.4. 没有版权声明或任何许可证标记的文件

某些文件无法添加适当的注释。在这种情况下，可以在 **file.ext.license** 中找到许可证。例如，名为 **foo.jpg** 的文件可能会在 **foo.jpg.license** 中找到许可证，遵循 REUSE 软件规范。

由项目创建的缺少版权声明的文件，理解为属于 **COPYRIGHT** 中的总括版权和许可。该文件可能仅仅是事实的陈述，不受版权法保护，或者内容微不足道，不需要明确的许可证。

缺少标记且包含非微不足道版权材料的文件，或者其作者认为标记不当的文件，应引起 FreeBSD 核心团队的注意。FreeBSD 项目的坚定政策是遵守所有适当的许可证。

未来，所有此类文件将被明确标记，或遵循 REUSE 软件的 **.license** 约定。

### 5.5. SPDX-License-Identifier 表达式

`SPDX 许可证表达式` 在 FreeBSD 软件集合中有两个使用场景。首先，它的完整形式用于包含明确许可证声明以及总结性 SPDX-License-Identifier 表达式的文件。在这种情况下，可以使用这些表达式的全部功能。其次，在上述的限制形式中，它用于标示给定文件的实际许可证。在第二种情形下，项目只允许使用此表达式的一个子集。

`SPDX 许可证子表达式` 要么是来自 [SPDX 许可证列表](https://spdx.org/licenses/) 的 SPDX 简短许可证标识符，要么是当 [许可证例外](https://spdx.org/licenses/exceptions-index.html) 适用时，两个 SPDX 简短许可证标识符通过 "WITH" 组合而成。当多个许可证适用时，表达式由 "AND" 或 "OR" 关键字分隔的子表达式组成，并被括号 "(", ")" 包围。[表达式的完整规范](https://spdx.github.io/spdx-spec/appendix-IV-SPDX-license-expressions/) 详细说明了所有细节，当它与本节的简化处理冲突时，以此为准。

一些许可证标识符，如 **[L]GPL**，具有使用该版本或任何更高版本的选项。SPDX 定义了后缀 `-or-later`，表示该版本的许可证或更高版本。它还定义了 `-only`，表示仅该特定版本的许可证。有一种旧的约定，不使用后缀（这意味着新定义的 `-only` 后缀所表示的意思，但人们常常将其误解为 `-or-later`）。此外，添加一个 `+` 后缀意味着 `-or-later`。FreeBSD 中的新文件不应使用这两种约定。使用这些约定的旧文件应根据需要进行转换。

```c
// SPDX-License-Identifier: GPL-2.0-only
// SPDX-License-Identifier: LGPL-2.1-or-later
```

当需要许可证修饰符时，应使用 `WITH`。在 FreeBSD 项目中，一些来自 LLVM 的文件对 Apache 2.0 许可证有一个例外：

```c
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
```

[例外标签](https://spdx.org/licenses/exceptions-index.html) 由 SPDX 管理。许可证例外只能应用于特定的许可证，如例外中所指定。

如果文件有选择许可证并选择了其中一个许可证，则应使用 `OR`。例如，一些 dtsi 文件可以通过双重许可证获得：

```c
// SPDX-License-Identifier: GPL-2.0 OR BSD-3-Clause
```

如果文件有多个许可证，且所有许可证条款都适用于使用该文件，则应使用 `AND`。例如，如果代码合并自多个项目，每个项目都有自己的许可证：

```c
// SPDX-License-Identifier: BSD-2-Clause AND MIT
```

