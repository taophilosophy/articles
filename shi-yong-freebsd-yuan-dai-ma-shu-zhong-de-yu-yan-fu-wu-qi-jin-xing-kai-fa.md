# 使用 FreeBSD 源代码树中的语言服务器进行开发

版权 © 2021 FreeBSD 基金会

<details open="" data-immersive-translate-walked="59cb05c3-6911-4a2b-b8ee-30938d9467d1"><summary data-immersive-translate-walked="59cb05c3-6911-4a2b-b8ee-30938d9467d1" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标.

</details>

## 1. 介绍

本指南介绍了如何使用语言服务器执行源代码索引对 FreeBSD 源代码树进行设置。该指南描述了针对 Vim/NeoVim 和 VSCode 的步骤。如果您使用不同的文本编辑器，您可以将本指南用作参考，并搜索您首选编辑器的等效命令。

## 2. 需求

为了遵循本指南，我们需要安装某些要求。我们需要一个 Language server， ccls 或 clangd ，以及一个可选的编译数据库。

Language server 的安装可以通过 pkg 或通过 ports 进行。如果我们选择 clangd ，我们需要安装 llvm 。

使用 pkg 来安装 ccls ：

```
# pkg install ccls
```

如果我们想要使用 clangd ，我们需要安装 llvm （示例命令使用 llvm15 ，但请选择您想要的版本）：

```
# pkg install llvm15
```

要通过 ports 安装，请从以下每个类别中选择喜爱的工具组合：

- 语言服务器实现

  - [ 开发/ccls](https://cgit.freebsd.org/ports/tree/devel/ccls/)
  - 开发/llvm12（其他版本也可以，但更新的更好。如果使用其他版本，请使用 clangdN 替换 clangd12 。）

- 编辑

  - [ 编辑器/vim](https://cgit.freebsd.org/ports/tree/editors/vim/)
  - [ 编辑器/neovim](https://cgit.freebsd.org/ports/tree/editors/neovim/)
  - [ 编辑器/vscode](https://cgit.freebsd.org/ports/tree/editors/vscode/)

- 编译数据库生成器

  - devel/python（用于 llvm 的 scan-build-py 实现）
  - devel/py-pip（用于 rizsotto 的 scan-build 实现）
  - [ 开发/bear](https://cgit.freebsd.org/ports/tree/devel/bear/)

## 3. 编辑器设置

### 3.1. Vim/Neovim

#### 3.1.1. LSP 客户端插件

此示例中的两个编辑器都使用内置插件管理器。使用的 LSP 客户端插件是 prabirshrestha/vim-lsp。

要为 Neovim 设置 LSP 客户端插件：

```
# mkdir -p ~/.config/nvim/pack/lsp/start
# git clone https://github.com/prabirshrestha/vim-lsp ~/.config/nvim/pack/lsp/start/vim-lsp
```

对于 Vim：

```
# mkdir -p ~/.vim/pack/lsp/start
# git clone https://github.com/prabirshrestha/vim-lsp ~/.vim/pack/lsp/start/vim-lsp
```

要在编辑器中启用 LSP 客户端插件，请在使用 Neovim 时将以下代码片段添加到 ~/.config/nvim/init.vim 中，或者在使用 Vim 时添加到 ~/.vim/vimrc 中：

对于 ccls

```
au User lsp_setup call lsp#register_server({
     'name': 'ccls',
     'cmd': {server_info->['ccls']},
     'allowlist': ['c', 'cpp', 'objc'],
     'initialization_options': {
         'cache': {
             'hierarchicalPath': v:true
         }
     }})
```

对于 clangd

```
au User lsp_setup call lsp#register_server({
     'name': 'clangd',
     'cmd': {server_info->['clangd15', '--background-index', '--header-insertion=never']},
     'allowlist': ['c', 'cpp', 'objc'],
     'initialization_options': {},
     })
```

根据您安装的版本 clangd ，您可能需要更新 server-info 以指向正确的二进制文件。

请参阅 https://github.com/prabirshrestha/vim-lsp/blob/master/README.md#registering-servers 了解如何设置快捷键和代码完成。clangd 的官方网站是 https://clangd.llvm.org，ccls 的存储库链接是 https://github.com/MaskRay/ccls/。

下面是键绑定和代码补全的参考设置。将以下代码片段放入 ~/.config/nvim/init.vim，或者对于 Vim 用户放入 ~/.vim/vimrc 以使用它：

```
function! s:on_lsp_buffer_enabled() abort
    setlocal omnifunc=lsp#complete
    setlocal completeopt-=preview
    setlocal keywordprg=:LspHover

    nmap <buffer> <C-]> <plug>(lsp-definition)
    nmap <buffer> <C-W>] <plug>(lsp-peek-definition)
    nmap <buffer> <C-W><C-]> <plug>(lsp-peek-definition)
    nmap <buffer> gr <plug>(lsp-references)
    nmap <buffer> <C-n> <plug>(lsp-next-reference)
    nmap <buffer> <C-p> <plug>(lsp-previous-reference)
    nmap <buffer> gI <plug>(lsp-implementation)
    nmap <buffer> go <plug>(lsp-document-symbol)
    nmap <buffer> gS <plug>(lsp-workspace-symbol)
    nmap <buffer> ga <plug>(lsp-code-action)
    nmap <buffer> gR <plug>(lsp-rename)
    nmap <buffer> gm <plug>(lsp-signature-help)
endfunction

augroup lsp_install
    au!
    autocmd User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
```

### 3.2. VSCode

#### 3.2.1. LSP 客户端插件

LSP 客户端插件需要启动语言服务器守护程序。按 Ctrl+Shift+X 以显示扩展在线搜索面板。运行 clangd 时输入 llvm-vs-code-extensions.vscode-clangd ，或者运行 ccls 时输入 ccls-project.ccls 。

然后，按 Ctrl+Shift+P 以显示编辑器命令面板。在面板中输入 Preferences: Open Settings (JSON) ，然后按 Enter 打开 settings.json。根据语言服务器的实现，将以下 JSON 键/值对之一放入 settings.json：

对于 clangd

```c
[
    /* Begin of your existing configurations */
    ...
    /* End of your existing configurations */
    "clangd.arguments": [
        "--background-index",
        "--header-insertion=never"
    ],
    "clangd.path": "clangd12"
]
```

为 ccls

```c
[
    /* Begin of your existing configurations */
    ...
    /* End of your existing configurations */
    "ccls.cache.hierarchicalPath": true
]
```

## 4. 编译数据库

编译数据库包含一系列编译命令对象。每个对象指定了编译源文件的方式。编译数据库文件通常是 compile_commands.json。该数据库被语言服务器实现用于索引目的。

请参考 https://clang.llvm.org/docs/JSONCompilationDatabase.html#format 以获取编译数据库文件格式的详细信息。

### 4.1. 生成器

#### 4.1.1. 使用 scan-build-py

##### 4.1.1.1. 安装

从 scan-build-py 工具生成编译数据库。

首先安装 devel/python 以获取 Python 解释器。要从 LLVM 获取 intercept-build ：

```
# git clone https://github.com/llvm/llvm-project /path/to/llvm-project
```

您的所需存储库的路径在 /path/to/llvm-project/ 处。为了方便起见，在 shell 配置文件中建立一个别名：

```
alias intercept-build='/path/to/llvm-project/clang/tools/scan-build-py/bin/intercept-build'
```

rizsotto/scan-build 可以用作 LLVM 的 scan-build-py 的替代品。LLVM 的 scan-build-py 已合并到 LLVM 树中。该实现可通过 pip install --user scan-build 安装。默认情况下， intercept-build 脚本在 ~/.local/bin 中。

##### 4.1.1.2. 用途

在 FreeBSD src 树的顶层目录中，使用 intercept-build 生成编译数据库：

```
# intercept-build --append make buildworld buildkernel -j`sysctl -n hw.ncpu`
```

--append 标志告诉 intercept-build 读取现有的编译数据库（如果存在）并将结果附加到数据库中。具有重复命令键的条目将被合并。默认情况下，生成的编译数据库将保存在当前工作目录中，其文件名为 compile_commands.json。

#### 使用 devel/bear 4.1.2.

##### 4.1.2.1. 用法

在 FreeBSD 源代码树的顶层目录中，使用 bear 生成编译数据库：

```
# bear --append -- make buildworld buildkernel -j`sysctl -n hw.ncpu`
```

--append 标志告诉 bear 如果存在编译数据库，则读取并将结果附加到数据库中。具有重复命令键的条目将被合并。默认情况下，生成的编译数据库将保存在当前工作目录中，命名为 compile_commands.json。

## 5. 最终

生成编译数据库后，打开 FreeBSD 源代码树中的任何源文件，LSP 服务器守护程序也将在后台启动。首次打开源代码树中的源文件在 LSP 服务器能够给出完整结果之前需要更长的时间，这是因为 LSP 服务器通过编译数据库中列出的所有条目进行初始后台索引。然而，语言服务器守护程序不会索引未出现在编译数据库中的源文件，因此在 make 期间未编译的源文件上不会显示完整结果。
