# 使用 FreeBSD 源代码树中的语言服务器进行开发

- 原文：[Use Language Servers for Development in the FreeBSD Src Tree](https://docs.freebsd.org/en/articles/freebsd-src-lsp/)

## 1. 介绍

本指南介绍如何设置 FreeBSD src 树，并使用语言服务器执行源代码索引。该指南涵盖了 Vim/NeoVim 和 VSCode 的设置步骤。如果你使用其他文本编辑器，可以将本指南作为参考，并搜索适用于你首选编辑器的相应命令。

## 2. 系统要求

为了按照本指南操作，我们需要安装一些必备组件。我们需要语言服务器，`ccls` 或 `clangd`，以及可选的编译数据库。

语言服务器的安装可以通过 `pkg` 或者 Ports 来完成。如果我们选择 `clangd`，则需要安装 `llvm`。

使用 `pkg` 安装 `ccls`：

```sh
# pkg install ccls
```

如果我们想使用 `clangd`，需要安装 `llvm`（示例命令使用 `llvm15`，但可以根据需要选择版本）：

```sh
# pkg install llvm15
```

通过 Ports 安装，可以根据下列类别选择你最喜欢的工具组合：

- 语言服务器实现

  - [devel/ccls](https://cgit.freebsd.org/ports/tree/devel/ccls/)
  - [devel/llvm12](https://cgit.freebsd.org/ports/tree/devel/llvm12/) （其他版本也可以，但较新的版本更好。如果使用其他版本，请将 `clangd12` 替换为 `clangdN`）
- 编辑器

  - [editors/vim](https://cgit.freebsd.org/ports/tree/editors/vim/)
  - [editors/neovim](https://cgit.freebsd.org/ports/tree/editors/neovim/)
  - [editors/vscode](https://cgit.freebsd.org/ports/tree/editors/vscode/)
- 编译数据库生成器

  - [devel/python](https://cgit.freebsd.org/ports/tree/devel/python/) （用于 llvm 的 scan-build-py 实现）
  - [devel/py-pip](https://cgit.freebsd.org/ports/tree/devel/py-pip/) （用于 rizsotto 的 scan-build 实现）
  - [devel/bear](https://cgit.freebsd.org/ports/tree/devel/bear/)

## 3. 编辑器设置

### 3.1. Vim/Neovim

#### 3.1.1. LSP 客户端插件

本示例中两个编辑器均使用内置插件管理器，LSP 客户端插件为 [prabirshrestha/vim-lsp](https://github.com/prabirshrestha/vim-lsp)。

为 Neovim 设置 LSP 客户端插件：

```sh
# mkdir -p ~/.config/nvim/pack/lsp/start
# git clone https://github.com/prabirshrestha/vim-lsp ~/.config/nvim/pack/lsp/start/vim-lsp
```

为 Vim 设置：

```sh
# mkdir -p ~/.vim/pack/lsp/start
# git clone https://github.com/prabirshrestha/vim-lsp ~/.vim/pack/lsp/start/vim-lsp
```

要在编辑器中启用 LSP 客户端插件，在 Neovim 中将以下代码片段添加到 **~/.config/nvim/init.vim**，在 Vim 中添加到 **~/.vim/vimrc**：

对于 ccls

```c
au User lsp_setup call lsp#register_server({
    \ 'name': 'ccls',
    \ 'cmd': {server_info->['ccls']},
    \ 'allowlist': ['c', 'cpp', 'objc'],
    \ 'initialization_options': {
    \     'cache': {
    \         'hierarchicalPath': v:true
    \     }
    \ }})
```

对于 clangd

```c
au User lsp_setup call lsp#register_server({
    \ 'name': 'clangd',
    \ 'cmd': {server_info->['clangd15', '--background-index', '--header-insertion=never']},
    \ 'allowlist': ['c', 'cpp', 'objc'],
    \ 'initialization_options': {},
    \ })
```

根据你安装的 `clangd` 版本，可能需要更新 `server-info` 以指向正确的二进制文件。

请参考 [https://github.com/prabirshrestha/vim-lsp/blob/master/README.md#registering-servers](https://github.com/prabirshrestha/vim-lsp/blob/master/README.md#registering-servers) 了解如何设置快捷键和代码补全。`clangd` 的官网是 [https://clangd.llvm.org](https://clangd.llvm.org/)，而 ccls 的仓库链接是 [https://github.com/MaskRay/ccls/](https://github.com/MaskRay/ccls/)。

以下是快捷键和代码补全的参考设置。将以下代码片段添加到 **~/.config/nvim/init.vim**，或者 Vim 用户可将其添加到 **~/.vim/vimrc**：

```ini
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

LSP 客户端插件是启动语言服务器守护进程所必需的。按 `Ctrl+Shift+X` 显示扩展在线搜索面板。当使用 clangd 时，输入 `llvm-vs-code-extensions.vscode-clangd`，或者使用 ccls 时，输入 `ccls-project.ccls`。

接着，按 `Ctrl+Shift+P` 显示编辑器命令面板，输入 `Preferences: Open Settings (JSON)`，然后按 `Enter` 打开 **settings.json**。根据所使用的语言服务器实现，在 **settings.json** 中添加以下 JSON 键值对之一：

对于 clangd

```ini
[
    /* 现有配置开始 */
    ...
    /* 现有配置结束 */
    "clangd.arguments": [
        "--background-index",
        "--header-insertion=never"
    ],
    "clangd.path": "clangd12"
]
```

对于 ccls

```ini
[
    /* 现有配置开始 */
    ...
    /* 现有配置结束 */
    "ccls.cache.hierarchicalPath": true
]
```

## 4. 编译数据库

编译数据库包含一个编译命令对象数组。每个对象指定了一种编译源文件的方式。编译数据库文件通常为 **compile_commands.json**。该数据库由语言服务器实现用于索引。

有关编译数据库文件格式的详细信息，请参考 [https://clang.llvm.org/docs/JSONCompilationDatabase.html#format](https://clang.llvm.org/docs/JSONCompilationDatabase.html#format)。

### 4.1. 生成器

#### 4.1.1. 使用 scan-build-py

##### 4.1.1.1. 安装

`scan-build-py` 中的 `intercept-build` 工具用于生成编译数据库。

首先安装 [devel/python](https://cgit.freebsd.org/ports/tree/devel/python/) 以获取 Python 解释器。然后，从 LLVM 获取 `intercept-build`：

```sh
# git clone https://github.com/llvm/llvm-project /path/to/llvm-project
```

其中 **/path/to/llvm-project/** 是你希望存放仓库的路径。为了方便起见，可以在 shell 配置文件中为其创建别名：

```sh
alias intercept-build='/path/to/llvm-project/clang/tools/scan-build-py/bin/intercept-build'
```

[rizsotto/scan-build](https://github.com/rizsotto/scan-build) 可以替代 LLVM 的 scan-build-py。rizsotto/scan-build 已被合并入 LLVM 代码树，成为 LLVM 的 scan-build-py。可以通过 `pip install --user scan-build` 来安装这个实现。`intercept-build` 脚本默认位于 **~/.local/bin** 中。

##### 4.1.1.2. 使用方法

在 FreeBSD src 树的顶级目录中，使用 `intercept-build` 生成编译数据库：

```sh
# intercept-build --append make buildworld buildkernel -j`sysctl -n hw.ncpu`
```

`--append` 标志告诉 `intercept-build` 读取现有的编译数据库（如果存在），并将结果追加到数据库中。具有重复命令键的条目将被合并。默认情况下，生成的编译数据库保存在当前工作目录下，文件名为 **compile_commands.json**。

#### 4.1.2. 使用 devel/bear

##### 4.1.2.1. 使用方法

在 FreeBSD src 树的顶级目录中，使用 `bear` 生成编译数据库：

```sh
# bear --append -- make buildworld buildkernel -j`sysctl -n hw.ncpu`
```

`--append` 标志告诉 `bear` 读取现有的编译数据库（如果存在），并将结果追加到数据库中。具有重复命令键的条目将被合并。默认情况下，生成的编译数据库保存在当前工作目录下，文件名为 **compile_commands.json**。

## 5. 最终步骤

生成编译数据库后，打开 FreeBSD src 树中的任何源文件，LSP 服务器守护进程将会在后台启动。第一次打开 src 树中的源文件时，LSP 服务器需要进行初始的后台索引，这可能会花费较长时间，因为它需要编译编译数据库中列出的所有条目。然而，语言服务器守护进程不会索引没有出现在编译数据库中的源文件，因此没有被 `make` 编译的源文件不会显示完整的结果。
