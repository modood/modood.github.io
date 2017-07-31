『Vim 编辑器』之快捷键映射
==========================

Table of contents
-----------------

*   [模式](#模式)
*   [映射](#映射)
*   [键盘符号](#键盘符号)
*   [示例](#示例)

模式
----

*   **常规模式(Normal)**

    常规模式主要用来浏览和修改文本内容的。无论当前处于哪个模式，狂按 ESC 即可回到常规模式。

*   **插入模式(Insert)**

    插入模式则用来向文本中添加内容的。使用 i, I, o, O, a, A 等进入。

*   **可视模式(Visual)**

    可视模式相当于高亮选取文本后的普通模式。使用 v, V, Ctrl-v 可分别进入可视模式，行可视模式，块可视模式。

*   **选择模式(Select)**

    和可视模式不同的是，在这个模式下，选择完了高亮区域后，敲任何按键就直接输入并替换选择的文本了。使用 gh 进入。

*   **运算符模式(Operator Pending)**

    用来接受命令的状态，这个状态在我们调用操作符（Operator）时被激活。
    常用到的运算符有 d，y 和 c。例如执行命令 dw 删除单词，这一模式就在 d 及 w 键之间短暂时间间隔内存在。

*   **命令模式(Command)**

    命令模式则多用于操作文件，使用 Q, : 进入。

映射
----

各种映射的作用模式：

| Command | Normal  | Visual  | Operator Pending | Insert Only | Command Line |
|:--------|:-------:|:-------:|:----------------:|:-----------:|:------------:|
| :map    | *       | *       | *                |             |              |
| :nmap   | *       |         |                  |             |              |
| :vmap   |         | *       |                  |             |              |
| :omap   |         |         | *                |             |              |
| :map!   |         |         |                  | *           | *            |
| :imap   |         |         |                  | *           |              |
| :cmap   |         |         |                  |             | *            |

**提示** nore 表示非递归。例如：:noremap 和 :map 命令相对，作用模式和命令格式都相同。
递归的映射就是如果键 a 被映射成了 b，c 又被映射成了 a，如果映射是递归的，那么 c 就被映射成了 b。

键盘符号
--------

详细： `:h key-notation`

```
<CR>            carriage return
<Return>        same as <CR>
<Enter>         same as <CR>

<Esc>           escape
<Tab>           tab
<Space>         space
<BS>            backspace

<Home>          home
<End>           end
<PageUp>        page-up
<PageDown>      page-down
<Up>            cursor-up
<Down>          cursor-down
<Left>          cursor-left
<Right>         cursor-right

<F1> - <F12>    function keys 1 to 12

<S-*>           shift-key
<C-*>           control-key
<M-*>           alt-key or meta-key
<A-*>           same as <M-*>
<D-*>           command-key (Macintosh only)
```

示例
----

自动补全括号引号：

```
inoremap ( ()<ESC>i
inoremap [ []<ESC>i
inoremap { {}<ESC>i
inoremap < <><ESC>i
inoremap " ""<ESC>i
inoremap ' ''<ESC>i
```

利用 Tab 键和 Shift-Tab 键来缩进文本：

```
nmap <tab> V>
nmap <s-tab> V<
vmap <tab> >gv
vmap <s-tab> <gv
```

