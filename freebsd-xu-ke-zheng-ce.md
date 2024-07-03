# FreeBSD 许可政策

<details open="" data-immersive-translate-walked="ad7d7e98-90aa-45d0-9398-88a0cdcc2588"><summary data-immersive-translate-walked="ad7d7e98-90aa-45d0-9398-88a0cdcc2588" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

制造商和销售商用来区分其产品的许多名称被声明为商标。 如果这些名称出现在本文件中，并且 FreeBSD 项目意识到了商标声明，这些名称旁边将跟随“™”或“®”符号。

</details>

## 1. 新文件的首选许可证

本节的其余部分旨在帮助您入门。一般而言，当有疑问时，请询问。接受建议要比修复源树容易得多。FreeBSD 项目同时使用明确许可证（在每个文件中重现许可证的原文）和分离许可证（文件中的标签指定许可证，如本文档所述）。

FreeBSD 项目使用此文本作为首选许可证：

```
/*-
 * Copyright (c) [year] [your name]
 *
 * SPDX-License-Identifier: BSD-2-Clause
 */
```

FreeBSD 项目不允许在新代码中使用“广告条款”。由于 FreeBSD 项目贡献者众多，许多商业供应商难以遵守此条款。如果您的代码中含有广告条款，请考虑更换为不包含广告条款的许可证。FreeBSD 的新贡献应使用 BSD-2-Clause 许可证。

FreeBSD 项目不鼓励使用全新的许可证和标准许可证的变种。新许可证需要 core@FreeBSD.org 的批准才能存入主仓库。过去，非标准许可证比标准许可证产生了更多的问题。非标准许可证的草拟不佳往往导致更多意想不到的后果，因此不太可能得到 core@FreeBSD.org 的批准。FreeBSD 项目正在标准化使用 SPDX 发布的 BSD-2-Clause 许可证。

此外，项目政策要求在某些非 BSD 许可证下授权的代码必须放置在仓库的特定部分。对于某些许可证，编译必须是有条件的或默认禁用的。例如，GENERIC 内核的静态部分中的代码必须在 BSD 或实质上相似的许可证下授权。GPL、APSL、CDDL 等许可证的软件不得编译到静态 GENERIC 内核中。但是，可以在预编译模块中使用这些许可证的代码。

开发人员被提醒，在开源中，正确理解"开放"与正确理解"源代码"一样重要。对知识产权的不当处理会产生严重后果。如有任何问题或疑虑，应立即告知 core@FreeBSD.org。

## 2. 软件许可政策

以下部分详细描述了项目的软件许可政策。我们大部分情况下希望开发人员阅读，理解并利用本节内容来为其贡献应用适当的许可。本文档的其余部分详细说明了政策的哲学背景以及政策内容。如果下文令人困惑，或者您需要帮助应用这些政策，请随时联系 core@FreeBSD.org。

### 2.1. 指导原则

FreeBSD 项目旨在生产一个完整的、基于 BSD 许可的操作系统，允许系统的使用者生成衍生产品而无需受到约束或进一步的许可义务。我们邀请并非常感激对两条款的 BSD 许可证下的变更和添加的贡献，并鼓励其他开源项目采用这一许可证。BSD 许可证的使用对于促进先进操作系统技术的采纳至关重要，并且在许多显著场合已经对新技术的广泛使用起到了关键作用。

然而，我们认识到存在强制性理由允许在 FreeBSD 源代码树中包含不同许可的软件。

我们要求根据某些非 BSD 许可证授权的软件在源代码树中进行严格隔离，以防止其污染仅限于 BSD 的组件。这种谨慎的管理有助于许可清晰，并促进仅限于 BSD 的衍生产品的生产。

除非特别例外，不得用更具限制性许可的软件替换现有的 BSD 许可的组件。我们鼓励 FreeBSD 和第三方开发人员寻求在 BSD 许可下重许可、双重许可或重新实现关键组件，以便更顺利地将其整合到 FreeBSD 操作系统中。

### 2.2. 政策

* 导入任何非 BSD 许可证和 BSD-Like 许可证（如下定义）许可的新软件需要事先获得 FreeBSD 核心团队的批准。导入请求必须包括：

  * 新版本或补丁中包含的功能或 bug 修复列表，以及用户需要这些功能的证据。PR 或邮件列表讨论的参考是理想的证据形式。
  * 这个过程应该用于所有的软件导入，而不仅仅是那些需要核心团队审核的软件。新版本的存在本身并不能证明将软件导入源代码或ports是合理的。
  * 可能受影响的 FreeBSD 分支列表。如果范围扩展，需要向 FreeBSD 核心团队提交新的请求并获得批准。
* Apache 许可证 2.0 在某些情况下可接受使用。必须经过核心团队批准，才能导入新的 Apache 许可证许可的组件，或将现有组件的许可证更改为 Apache 许可证。

  * 以下组件已批准使用此许可证：

    * LLVM 工具链以及（带有 LLVM 例外）运行时组件。
* BSD+专利许可证在某些情况下可以使用。核心团队必须批准将新的 BSD+专利许可证许可的组件导入，或者将现有组件的许可证更改为 BSD+专利许可证。

  * 此许可证适用于以下组件：

    * 基于 UEFI 功能的 EDK2 衍生代码
* 在某些情况下，通用开发和分发许可证（CDDL）可接受。核心团队必须批准导入新的 CDDL 许可的组件或更改现有组件为 CDDL。

  * 此许可证已批准用于以下组件:

    * DTrace
    * ZFS 文件系统，包括内核支持和用户空间实用程序
* 历史上，“保留所有权利”一词包含在所有版权声明中。 所有的 BSD 版本都有它，以便遵守 1910 年在美洲签署的布宜诺斯艾利斯公约。 随着尼加拉瓜于 2000 年批准《伯尔尼公约》，布宜诺斯艾利斯公约 — 以及该短语 — 已经过时。 因此，FreeBSD 项目建议新代码省略该短语，并鼓励现有版权持有者删除它。 2018 年，该项目更新了其模板以删除它。
* 最初，FreeBSD 树中的许多项目都标有 BSD-2-Clause-FreeBSD。然而，SPDX 已经将该许可证作为一种变体而废弃；而废弃标记的 SPDX 文本与标准 FreeBSD 许可证有足够的不同，不应使用。对其当前用途的审查正在进行中。

#### 2.2.1. 可接受的许可证

对于本政策的目的，以下许可证被认为是可接受的类似 BSD 许可证。偏离或使用任何其他许可证必须经 FreeBSD 核心团队批准：

* BSD 许可证的 2 个条款版本

```
/*-
 * Copyright (c) [year] [your name]
 *
 * SPDX-License-Identifier: BSD-2-Clause
 */
```

* BSD 许可证的 3 个条款版本

```
/*-
 * Copyright (c) [year] [your name]
 *
 * SPDX-License-Identifier: BSD-3-Clause
 */
```

* ISC 许可证

```
/*-
 * Copyright (c) [year] [copyright holder]
 *
 * SPDX-License-Identifier: ISC
 */
```

* MIT 许可证

```
/*-
 * Copyright (c) [year] [copyright holders]
 *
 * SPDX-License-Identifier: MIT
 */
```

## 3. 软件集合许可证

FreeBSD 项目根据 BSD-2-Clause 许可证在 COPYRIGHT 中描述其软件编译。此许可证不取代各个文件的许可证，后者在下文描述。未明确许可的文件均受 BSD-2-Clause 许可证的约束。

## 4. 许可文件位置

为了尽可能遵守 REUSE 软件标准，所有许可文件将存储在存储库的 LICENSES/目录中。在这个顶层目录下有三个子目录。LICENSES/text/子目录中以分离形式包含了允许在 FreeBSD 软件集合中的所有许可的文本。这些文件使用 SPDX-License-Identifier 名称后跟.txt 进行存储。LICENSES/exceptions/子目录包含了在 FreeBSD 软件集合中以分离形式允许的所有异常的文本。这些文件使用异常标识符名称后跟.txt。LICENSES/other/目录中以分离形式包含了在 SPDX-License-Identifier 表达式中引用的许可文件，但在其他方面不允许作为分离许可。所有这样的文件必须至少在 FreeBSD 软件集合中出现一次，并且应在删除最后引用它们的文件时移除。没有已识别出的没有充分的 SPDX 匹配许可证的文件，但如果有，一个完整的列表将出现在这里。

FreeBSD 项目目前不使用 DEP5 在 REUSE Software 标准中描述的文件。FreeBSD 项目尚未按照文档后面描述的标准标记树中的所有文件。由于该政策仍在不断发展中，FreeBSD 项目尚未在其存储库中包含这些文件。

## 5. 单个文件许可

FreeBSD 软件集合中的每个单独文件都有其自己的版权和许可。它们如何标记不同，并在本节中有描述。

版权声明会标识谁拥有文件的法律版权。这些由项目尽力提供。由于版权可能在法律上被转让，当前的版权持有者可能与文件中列出的不同。

许可是在贡献者和软件使用者之间的法律文件，授予使用受版权保护的软件部分的许可，受许可中规定的特定条款和条件约束。许可可以以两种方式之一在 FreeBSD 软件集合中表达。许可可以在文件中明确表示。当许可授予在文件中是明确的时，可以根据该许可使用、复制和修改该文件。许可还可以间接表达，许可的文本位于其他位置。项目使用 Software Package Data Exchange (SPDX)许可标识符用于此目的，如下面的小节所述。SPDX 许可标识符由 Linux 基金会的 SPDX 工作组管理，并且已经得到整个行业的合作伙伴、工具供应商和法律团队的认可。有关更多信息，请参阅 https://spdx.org/，以及 FreeBSD 项目如何使用它们的以下章节。

为软件集合贡献修复和增强功能的实体，如果没有明确许可，同意按适用于修改文件的条款许可这些更改。项目政策，与行业惯例相符，仅在集合中的重要贡献者的文件中包含版权声明。

FreeBSD 软件集合中有四种类型的文件：

1. 只有显式版权声明和许可证的文件。
2. 既有显式版权声明和许可证，又有 SPDX-License-Identifier 标签的文件。
3. 只有版权声明和 SPDX-License-Identifier 标签，但没有显式许可证的文件。
4. 完全没有任何版权或许可证的文件。

### 5.1. 只有版权和许可证

FreeBSD 软件集中的许多文件都在文件中包含了版权声明和明确的许可证。在这些情况下，文件中包含的许可证适用。

### 5.2. 版权和许可与 SPDX 许可标识符表达

FreeBSD 软件集合中的一些文件包含版权声明，SPDX-License-Identifier 标记和显式许可证。显式许可证优先于 SPDX-License-Identifier 标记。 SPDX-License-Identifier 标记是项目对许可证的最佳努力尝试，但仅适用于自动化工具的信息。请参阅如何解释表达式的 SPDX-License-Identifier 表达式。

### 5.3. 仅版权和 SPDX-License-Identifier 表达。

树中的一些文件包含独立的许可证。这些文件仅包含版权声明和 SPDX-License-Identifier 表达式，但没有明确的许可证。请参阅 SPDX-License-Identifier 表达式，了解如何解释该表达式。注意：项目允许的独立许可证表达式是标准信息性使用的表达式子集或由标准定义的表达式。

仅包含 SPDX-License-Identifier 的文件的许可证应被解释为

1. 从文件中的版权声明开始许可证。包括所有的版权持有者。
2. 对于每个子表达式，请从 LICENSE/text/ id .txt 中复制许可文本。当存在异常时，请从 src/share/license/exceptions/ id .txt 中追加它们。应根据 SPDX 标准理解 SPDX-License-Identifier 表达式。

其中 id 是来自 SPDX 标识符或许可异常 Identifier 列的 SPDX 短许可标识符。如果 LICENSE/中没有文件，则无法将该许可或异常指定为本节下的独立许可。

当阅读从文件分离的许可文本时，必须考虑多个因素，以使分离的许可合理。

1. 任何提及版权声明的内容都应指的是从许可文件中构建的版权声明，而不是许可文件本身中的任何版权声明。许多 SPDX 文件中有示例版权声明，这些声明被理解为仅为示例。
2. 当许可证文本中提到实体名称时，应被解释为适用于许可文件版权声明中列出的所有版权持有者的名单。例如，BSD-4-clause 许可证包含短语“该产品包含该组织开发的软件”。短语“该组织”应被版权持有者所取代。
3. 当 SPDX 提供许可证的变体时，应理解 LICENSE/ 文件中的许可证代表所选择的许可证的确切版本。SPDX 标准旨在匹配许可证系列，这些变体有助于匹配 SPDX 组织认为在法律上相同的类似许可证。

对于文本中有轻微变化的许可证，SPDX 有匹配它们的指南。这些指南在这里不相关。希望在 LICENSE/ 中未包含逐字版本的 SPDX 许可证变体下授权的贡献者不能使用分离选项，必须明确指定许可证。

### 5.4. 没有版权或任何许可证标记的文件

有些文件无法添加合适的注释。在这种情况下，可以在 file.ext.license 中找到许可证。例如，一个名为 foo.jpg 的文件可能会在 foo.jpg.license 中有许可证，遵循 REUSE 软件约定。

项目创建的没有版权声明的文件被理解为受版权和授权在版权中。要么文件只是事实的背诵，不受版权法保护，要么内容太琐碎，不值得专门许可的开销。

缺乏标记并且具有超过微不足道数量的受版权材料的文件，或其作者认为它们标记不当，应引起 FreeBSD 核心团队的注意。FreeBSD 项目强调遵守所有适当的许可。

将来，所有这类文件将被明确标记，或遵循 REUSE 软件 .license 约定。

### 5.5. SPDX-License-Identifier 表达式

一个'SPDX 许可证表达式'在 FreeBSD 软件集合中有两种上下文中使用。首先，它的完整形式用于文件，文件内含有明确的许可声明，以及总结性的 SPDX-License-Identifier 表达式。在这种情况下，可以使用这些表达式的全部功能。其次，在上述受限形式中，它用于表示给定文件的实际许可证。在第二种情况下，项目只允许此表达式的子集。

一个 SPDX License sub-expression 是来自 SPDX 许可证列表的 SPDX 简短形式许可证标识符，或者是两个 SPDX 简短形式许可证标识符的组合，由"WITH"分隔，当许可证有例外适用时。当多个许可证适用时，一个表达式由关键字"AND"，"OR"分隔子表达式并被"(", ")"括起来。表达式的完整规范详细说明了所有细节，并在与本节简化处理产生冲突时优先。

一些许可标识，如 [L]GPL，具有仅使用该版本或任何后续版本的选项。SPDX 定义后缀 -or-later 表示该许可证版本或更新版本。它定义 -only 表示文件的仅该特定版本。存在一种旧惯例，即没有后缀（这意味着新的“-only”后缀意味着的内容，但人们将其误解为 -or-later ）。此外，添加 + 后缀旨在表示 -or-later 。FreeBSD 中的新文件不应使用这两种惯例。使用此惯例的旧文件应根据需要进行转换。

```
      // SPDX-License-Identifier: GPL-2.0-only
      // SPDX-License-Identifier: LGPL-2.1-or-later
```

当需要许可证修改器时应使用 WITH 。在 FreeBSD 项目中，来自 LLVM 的一些文件对 Apache 2.0 许可证有一个例外：

```
      // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
```

SPDX 管理异常标签。许可证异常仅适用于特定许可证，如例外中所指定的那样。

如果文件有多个许可选择，并且选择了一个许可，则应使用 OR 。例如，某些 dtsi 文件提供双重许可：

```
      // SPDX-License-Identifier: GPL-2.0 OR BSD-3-Clause
```

如果文件有多个许可证，所有许可证条款均适用于使用该文件，则应使用 AND 。例如，如果代码已被多个项目合并，每个项目都有自己的许可证：

```
      // SPDX-License-Identifier: BSD-2-Clause AND MIT
```
