# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库概览

这是 FreeBSD 官方文章（FreeBSD Articles）的中文翻译项目。基于 GitBook 格式，发布在 <https://docs.freebsd.org.cn/articles/>。

源内容为 Markdown 文件，由 GitBook 平台自动构建和部署，无需本地构建步骤。

**翻译基准：** 英文原版归档于仓库内的 `en/` 目录（AsciiDoc `.adoc` 源），所有翻译均以仓库内 `en/<article-slug>/_index.adoc` 为唯一权威源。`vm-bhyve.md` 一篇来自外部 GitHub Wiki（[vm-bhyve wiki](https://github.com/churchers/vm-bhyve/wiki)），不在 `en/` 内。`pgpkeys` 与 `contributors` 两篇因无翻译意义而故意不译。

## 同步更新方法

流程：拉取 git 项目（注意，因为部分文件存在英文冒号，在 Windows 下无法拉取，需要下载 zip 压缩包）：https://github.com/freebsd/freebsd-doc

当前翻译 git 版本：93d98e91f6bfaacd7fae40dd398efe47d44f085b

路径：**documentation/content/en/**

将 **documentation/content/en/** 文件夹内的内容完全复制到 en 路径下，然后进行比较翻译（自行分析结构）。

最后更新本文中的相关信息（如翻译 git 版本）。

需要进一步核查时，才参考 <https://docs.freebsd.org/en/books/handbook/book/>。

## 内容架构

### 核心文件

- **`SUMMARY.md`** — 全书唯一数据源，定义完整目录结构和导航树。GitBook 用它来生成侧边栏。**第一行 `# Table of contents` 绝对不能变更**，否则 GitBook 同步失效。
- **`mu-lu.md`** — 由 `mulu.yml` CI 工作流从 `SUMMARY.md` 自动复制生成，**不要手动编辑**。
- **`README.md`** — 项目说明，记录项目维护状态与故意不译的两篇文章。
- **`CHANGELOG.md`**（如存在）— 编辑日志，记录翻译进度和同步上游的 commit。

### 目录结构

- 每篇文章是仓库根目录下单个 `.md` 文件，文件名采用拼音 slug（如 `vm-bhyve.md`、`jie-shi-bsd.md`、`zi-ti-yu-freebsd.md`）。
- 英文原版位于 `en/<article-slug>/_index.adoc`（AsciiDoc 语法），每个英文目录还含一个 `_index.po` 翻译模板文件。
- 文件名中不得包含空格、中文字符或英文冒号 `:`，必须兼容 Windows 操作系统对文件名的要求。
- `vm-bhyve.md` 一篇来自外部 GitHub Wiki，**无对应 `en/` 目录**，校对时直接对照 GitHub Wiki 原文链接。
- `pgpkeys`、`contributors` 两篇故意不译，`en/` 下保留这两个目录仅作存档，**不要为它们创建中文翻译**。
- `x86-assembly` 一篇此前缺失中文翻译，已于本次补译为 `x86-hui-bian-yu-yan.md`。
- `.gitbook/assets/` — GitBook 静态资源（logo 等）。

### 标题管理（关键约束）

`sync-headers.yml` 工作流会在每次 push 时自动将 `SUMMARY.md` 中的标题同步到对应 `.md` 文件的 H1。**这意味着直接编辑 `.md` 文件的 `# 标题` 会被 CI 覆盖。** 修改文章标题时，只改 `SUMMARY.md` 中对应的 `* [标题](路径)` 条目即可。

## CI/CD 工作流

所有工作流位于 `.github/workflows/`：

| 工作流 | 触发条件 | 说明 |
| ------ | -------- | ---- |
| `sync-headers.yml` | push + `mulu.yml` 完成后 | 从 SUMMARY.md 同步一级标题到各 .md 文件 |
| `mulu.yml` | SUMMARY.md 变更时 push | 复制 SUMMARY.md → mu-lu.md |
| `markdown-lint2.yml` | workflow_dispatch | markdownlint 检查全部 .md，规则见 `.github/.markdownlint.json` |
| `Markdown-lint2-pr.yml` | pull_request | 仅对 PR 中变更的 .md 文件运行 markdownlint |
| `md-padding.yml` | workflow_dispatch | 自动在 CJK 与英文/数字间添加空格 |
| `AutoCorrect.yml` | workflow_dispatch + repository_dispatch | 自动修正常见中文笔误与格式问题，并提交 PR |
| `links.yml` | 定时 + workflow_dispatch | lychee 死链检查，配置见 `.github/lychee.toml` |
| `file-name-check.yml` | --- | 检查 SUMMARY.md 中引用的文件是否存在 |
| `create-pdf.yml` | --- | 导出 PDF |

## 编写规范

### 格式

- **命令行前缀：** `#` 表示 root 权限，`$` 表示普通用户。不要使用 `sudo`。
- **提示块：** tip/important/note/warning/caution 使用 `>` 缩进引用，关键词 **加粗**。
- **代码块：** 无法判断的，再使用 ` \`\``sh ` 兜底，禁止使用 text 作为代码块标记，不窜改既有的 ` ```ini ` 标记。
- **表格：** 一律居中。
- **禁止 HTML：** 本项目不支持任何 HTML 语法。
- **文件命名：** 使用拼音 slug，文件名中不得包含空格、中文字符或英文冒号 `:`，必须兼容 Windows 操作系统对文件名的要求。
- **路径与 IP：** 全书正文中的路径（带 `/` 或 `\` 的，如 `/etc/rc.conf`、`/usr/local/etc/`）和 IP 地址一律使用 **加粗**，不使用反引号 `` ` `` 包裹；不要混合使用 `*` 和 `` ` ``（如 `*`path`*` 是错误的）。
- **命令、选项、参数：** 命令、选项、可调选项、可调参数等格式使用 `行间代码`（反引号）包裹。注意：不带选项和参数的裸命令，如 mv cp 等，一般不加反引号，即不进行包裹。
- **转义字符：** 除非是命令、选项和参数，否则含转义字符 `\` 的元素一律使用 **加粗** 包裹整个元素，不在正文中直接使用转义字符（如写 **PROTO_TYPE** 而非 `PROTO\_TYPE` 在正文里裸露）。逐个手动修改，禁止批量替换。
- **避免滥用"已"字：** 如"XX 已新增"应改为"新增 XX"；"已修复"应改为"修复完成"。禁止机械替换。
- **禁止篡改：** 不要篡改软件版本号、用户名、带圈数字（如 ①②③ 等），确保其位置、数量和英语原文一致。
- **避免章节交叉引用：** 正文不要出现具体的章节交叉引用（如"参见第 5 章"），改用语义化链接。
- **代码块注释翻译：** 代码块（` ``` ` 围栏）内的英文注释必须翻译为中文（如 shell 注释 `# This is a comment` → `# 这是一个注释`）。只翻译注释部分，不修改实际命令或代码。保持代码结构和格式不变。
- **fstab 不翻译：** fstab 文件表头（如 `# Device Mountpoint FStype Options Dump Pass#`）及其相关内容保持英文原样，不翻译。

### 术语

本项目不存在独立的术语对照表文件，以下术语规范直接遵守：

- `Ports` 保持英文不翻译，且保持首字母大写（注意区分真正的"端口"）禁止机械替换。
- "Jail" 保持英文（不翻译为"监狱"、"监牢"）禁止机械替换。
- "拷贝" → "复制"，"壳/外壳" → "shell"。禁止机械替换。
- 第二人称一律使用"你"而非"您"。

### CJK 空格

中英文、中文与数字之间必须加半角空格。文件和命令名用 `` ` `` 括起来。`md-padding.yml` 和 `AutoCorrect.yml` 工作流会自动检测修复。

### 图片

在正文中插入图片，使用 markdown 格式。

### 翻译流程

1. 参考仓库内 `en/<article-slug>/_index.adoc` 英文原版进行人工校对
2. 提交 PR 到 main 分支

### 补译缺失文章

当 `en/` 下出现 `SUMMARY.md` 中未收录的文章目录时，按本流程补译：

1. 读取 `en/<article-slug>/_index.adoc` 全文作为唯一权威源
2. 在仓库根目录新建 `<pinyin-slug>.md` 文件（拼音 slug，文件名不含空格/中文/英文冒号）
3. 文件首行写入 `# 标题`（与下一步 `SUMMARY.md` 条目文字一致，CI 后续会从 `SUMMARY.md` 同步覆盖）
4. 在 `SUMMARY.md` 文章列表末尾追加 `* [标题](<pinyin-slug>.md)` 条目
5. 翻译时遵循「编写规范」与「术语」小节（`Ports`/`Jail` 保留英文、第二人称用"你"）
6. **不要** 重命名任何已有 `.md` 文件或目录，**不要** 修改跨章节引用路径

### 翻译校对工作流程（Claude Code）

对已翻译文章进行质量审校时，参考以下步骤：

1. **获取原文**：直接读取仓库内 `en/<article-slug>/_index.adoc` 文件全文作为唯一权威源。`vm-bhyve.md` 一篇通过 `WebFetch` 抓取 GitHub Wiki 原文。**禁止** 以中文版已有翻译作为修改依据，避免错误沿袭。

2. **逐句对照**：将中文翻译与英文原文逐句比对，重点检查以下问题类别：

   | 问题类型 | 示例 |
   | -------- | ---- |
   | 事实性错误 | "large parts" 误译为"三个文件" |
   | 漏译 | 英文原版有但中文缺失的句子 |
   | 机翻腔/表达生硬 | "在阅读本章后，你将会收获" → "通过阅读本章，你将了解" |
   | 用词不当 | "独家优势"应为"主要优势"（原文 "particular strengths"） |
   | 版本过时 | 与英文原版 adoc 源中版本号、链接不一致 |
   | 语法/文字错误 | 重复字词、"非常地"应为"非常" |
   | 欧化汉语、倒装句、后置句、偷换主语、不必要的被动句 | 自行联网制定标准，避免语法错误和非地道汉语表述 |
   | 滥用"一个 XX" | 如 "an operating system" 译为"一个操作系统"应改为"操作系统" |

3. **提交修改**：
   - 基于 `main` 创建分支 `fix/article-<article-slug>-translation`
   - 仅提交修改的 `.md` 文件，不要包含无关文件
   - 提交信息格式：`fix: 修正 <文章标题> 翻译错误及同步英文原版更新`
   - 用 `gh pr create` 创建 PR，目标分支为 `main`
   - PR 描述中列出每处修改及原因

4. **注意事项**：
   - 不要修改 `# 标题`——会被 `sync-headers.yml` CI 覆盖
   - 保持术语一致性，参考上文「术语」小节
   - 英文原版可能比翻译版本更新，版本号差异应以英文原版 adoc 源为准
   - 添加新内容（如英文原版新增的段落）时，确保翻译风格与上下文一致
   - 禁止篡改带圈数字，如 ①②③ 等，确保其位置、数量和英语原文一致
   - 禁止篡改代码块（包括 shell 命令、配置文件内容、ini 段等），仅修改正文翻译
   - 禁止篡改用户名、邮箱、URL、IP 地址等技术参数
   - 禁止篡改软件版本号
   - 以仓库内 `en/<article-slug>/_index.adoc` 为唯一权威源；禁止以中文版已有翻译作为修改依据，避免错误沿袭

5. **逐句校对递归工作流**（核心约束）：
   - **逐行逐句子**遍历内容，检查逻辑一致性，联网复核后修改
   - **三轮 + 一轮复查**：完整执行三轮逐句校对，然后严格、完整地复查一轮；若仍有问题，继续执行三轮，递归直至无问题
   - **逐个修改**：修改时禁止机械修改、禁止批量；必须逐个手动修改，禁止使用软件工具批量替换或批量逐个复核
   - **修改前后复核**：每次修改前确认问题真实存在、修改后确认改动准确且未引入新错误
   - **不复查旧错**：不得复查以前已经完成三轮的错误；每轮查询内容必须是新的（若与之前的复查重复则跳过、不改，只看是否有新增内容）
   - **联网复查**：对于不存在的实际访问查询（如失效链接、版本号、API 端点），需联网复查三次后修改
   - **禁止绕过审查**：不得以任何方式搜索、批量、绕过逐句子审查流程

## Lint 配置

- **markdownlint**（`.github/.markdownlint.json`）：本地已安装 `markdownlint-cli2`，可在本地直接运行检查与修复
  - **本地运行命令**（在仓库根目录执行）：

    ```sh
    # 检查全部中文 .md（排除 en/、.github/、.gitbook/、node_modules/）
    markdownlint-cli2 "**/*.md" "!en/**" "!.github/**" "!.gitbook/**" "!node_modules/**" --config .github/.markdownlint.json

    # 自动修复可修复的问题（MD012 多余空行、MD047 末尾换行、MD034 裸 URL 等）
    markdownlint-cli2 "**/*.md" "!en/**" "!.github/**" "!.gitbook/**" "!node_modules/**" --config .github/.markdownlint.json --fix
    ```

  - **MD056 表格列数规则**：表头声明几列，所有数据行都必须有几列；末尾缺空单元格时需补 `| |`，而非省略管道符
- **textlint**（`.textlintrc`）：仅启用 `ja-space-between-half-and-full-width` 规则，用于 CJK/英文空格检查
- **lychee**（`.github/lychee.toml`）：6 线程、30 并发、30 秒超时、最多 3 次重试、Chrome UA、排除私有 IP 和 `ftp.freebsd.org`
