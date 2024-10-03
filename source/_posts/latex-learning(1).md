---
title: latex-learning(1)
tags:
  - latex
excerpt: note of latex
categories:
  - tools
date: 2024-10-03 15:01:15
---


中秋的时候就通过lshort学习了一些latex相关内容，但一直都没有自己试着整理一份资料，导致翻手册翻得很折磨，这里就整理一下，顺便也是学习更多关于latex的知识。

## 1. 基础知识

### 1.1 latex基础框架

```latex
\documentclass{...} % ... 为某文档类
% 导言区
\begin{document}
% 正文内容
\end{document}
% 此后内容会被忽略
```

1. `\documentclass{...}`：指定文档类，一般有article、report、book（这三个被称为基础文档类）等。
   
   对于文档类，也有一些可选参数（写在`[...]`中），一般比较常用的有：
    - `10/11/12pt`：设置字体大小为10/11/12pt
    - `a4paper`：设置纸张大小为A4
    - `onecolumn, twocolumn`：设置单双栏排版
    - `oneside, twoside`：设置单面/双面打印，区别在于双面打印会在左页右侧和右页左侧额外留白
    - `titlepage, notitlepage`: 指定标题命令 \maketitle 是否生成单独的标题页。article 默认为
notitlepage，report 和 book 默认为 titlepage
    - `landscape`：设置横向打印

2. `\begin{document}`和`\end{document}`：定义文档的开始和结束，在这之间的所有内容都会被latex编译。

    一个完整的正文区往往包括多级结构，以下是基础文档类的常用的划分文章层次的语句：
    - `\chapter{...}`：定义一个章节，一般对应一节
    - `\section{...}`：定义一个小节，一般对应一个小标题
    - `\subsection{...}`：定义一个小小节，一般对应一个小小标题
    - `\subsubsection{...}`：定义一个小小小节，一般对应一个小小小标题
    - `\paragraph{...}`：定义一个段落，一般对应一个正文段落
    - `\subparagraph{...}`：定义一个小段落，一般对应一个小标题段落

    这里也有一些变化，比如：
    - 带可选参数的变体：\section[⟨short title⟩]{⟨title⟩}
    - 带星号的变体：\section*{⟨title⟩} 此时标题不带编号，也不生成目录项和页眉页脚
    - 较低层次如 \paragraph 和 \subparagraph 即使不用带星号的变体，生成的标题默认也不
带编号

   但注意，article不包含\chapter，即不会生成目录项。

3. 注释：`%`表示注释，注释内容会被忽略。

下面提供一个更完整的结构：

```latex
\documentclass{book}
% 导言区，加载宏包和各项设置，包括参考文献、索引等
\usepackage{makeidx} % 调用 makeidx 宏包，用来处理索引
\makeindex % 开启索引的收集
\bibliographystyle{plain} % 指定参考文献样式为 plain
\begin{document}
\frontmatter % 前言部分
\maketitle % 标题页
\include{preface} % 前言章节 preface.tex
\tableofcontents
\mainmatter % 正文部分
\include{chapter1} % 第一章 chapter1.tex
\include{chapter2} % 第二章 chapter2.tex
...
\appendix % 附录
\include{appendixA} % 附录 A appendixA.tex
...
\backmatter % 后记部分
\include{epilogue} % 后记 epilogue.tex
\bibliography{books} % 利用 BibTeX 工具从数据库文件 books.bib 生成参考文献
\printindex % 利用 makeindex 工具生成索引
\end{document}
```

### 1.2 latex一些比较重要的知识

#### 1.2.1 文件名后缀一览：

- .sty 宏包文件。宏包的名称与文件名一致。
- .cls 文档类文件。文档类名称与文件名一致。
- .bib BIBTEX 参考文献数据库文件。
- .bst BIBTEX 用到的参考文献格式模板。详见 6.1.4 小节。

LATEX 在编译过程中除了生成 .dvi 或 .pdf 格式的文档外，还可能会生成相当多的辅助文
件和日志。一些功能如交叉引用、参考文献、目录、索引等，需要先通过编译生成辅助文件，然
后再次编译时读入辅助文件得到正确的结果，所以复杂的 LATEX 源代码可能要编译多次：
- .log 排版引擎生成的日志文件，供排查错误使用。
- .aux LATEX 生成的主辅助文件，记录交叉引用、目录、参考文献的引用等。
- .toc LATEX 生成的目录记录文件。
- .lof LATEX 生成的图片目录记录文件。
- .lot LATEX 生成的表格目录记录文件。
- .bbl BIBTEX 生成的参考文献记录文件。
- .blg BIBTEX 生成的日志文件。
- .idx LATEX 生成的供 makeindex 处理的索引记录文件。
- .ind makeindex 处理 .idx 生成的用于排版的格式化索引文件。
- .ilg makeindex 生成的日志文件。
- .out hyperref 宏包生成的 PDF 书签记录文件。

#### 1.2.2 include,input以及includeonly命令


1. `\includeonly{...}`：置于导言区，意为后面正文部分只能插入此处指定的文件，其余文件不插入。
   
   使用此命令时，所有的\input命令都不会被执行，只有\include命令才会被执行。

   使用此命令可以用于指定哪些章节需要编译，这在处理大型文档时可以节省编译时间。。

2. \include{...}与\input{...}：这两个命令都用来插入其他文件的内容。

   \input 和 \include 有以下几个主要区别：

   - 内容处理方式
  
   \input: 直接将指定文件的内容插入到当前位置。它不会引入新的分页符或停止编译。

   \include: 引入的文件内容会在一个新页开始处插入，并且每次调用 \include 时都会停止编译，直到处理完当前文件。
   - 编译行为
  
   \input: 在编译时，\input 的行为类似于复制粘贴，不会影响编译的暂停或页面处理。

   \include: 在编译时，\include 会在每次调用时停止编译，生成一个临时的 .aux 文件，然后再继续编译。这种行为使得 \include 在处理大文档时分章节编译更为方便，尤其是配合 \includeonly 命令时。
   - 适用场景
  
   \input: 适用于插入局部内容，如代码片段、表格或其他不需要单独分页的小文件。

   \include: 适用于插入章节级别的文件，每个章节通常会有自己的 .tex 文件，并且可能会包含大量的内容。

3. 使用include的注意事项：

   - 避免循环引用：如果文件A包含文件B，而文件B又包含文件A，则会出现循环引用，导致编译失败。

   - 避免嵌套调用：如果文件A包含文件B，而文件B又包含文件C，则文件C不能再包含文件A，否则会出现编译失败。
   - 不需要完整的文档结构：使用 \include 引入的章节文件（例如 chapter1.tex）可以直接包含 LaTeX 段落、章节和小节等内容，而不需要写 \documentclass{}、\begin{document} 和 \end{document}。这些命令只出现在主文档中。
   - 路径书写时，注意转义符号的存在，以防出现不想出现的路径。

{% folding green::只编译不生成pdf的包 %}
```latex
using package{syntonly}
\syntaxonly % 用于导言区
```
这在编译大型文档时可以节省编译时间，只生成语法检查的.log文件，不生成.pdf文件。
{% endfolding %}

#### 1.2.3 中文支持

ctex 宏包和文档类能够识别操作系统和 TEX 发行版中安装的中文字体，因此基本无需额外配置即可排版中文文档。

注意源代码须保存为 UTF-8 编码，并使用 xelatex 或 lualatex 命令编译。虽然 ctex 宏
包和文档类保留了对 GBK 编码以及 latex 和 pdflatex 编译命令的兼容，但并不推荐这
样做。

### 1.3 常用结构

#### 列表（可以嵌套使用，最多四层）

- 有序列表

   ```latex
   \begin{enumerate}
       \item ...
       ...
   \end{enumrate}
   ```

   其中`\item[...]`可以加上可选参数，将默认符号替换成方括号中自定义的符号

   有序列表的符号由命令 \labelenumi 到 \labelenumiv 定义，重新定义这些命令需要用到计数器相关命令，这在后面会提到。

- 无序列表

   ```latex
   \begin{itemize}
       \item
       ...
   \end{itemize}
   ```

   `\item`同样可以带上参数改变默认符号

   各级无序列表的符号由命令 \labelitemi 到 \labelitemiv 定义，可以简单地重新定义它们

   ```latex
   % i ii iii iv
   \renewcommand{\lableitemi}{\ddag} % 改变第一级为\ddag
   \renewcommand{\lableitemii}{\dag} % 改变第二级为\dag
   ```

- 关键词列表

      ```latex
      \begin{description}
      \item[<item title>]
      ...
      \end{description}
      ```

      其中`item title`最终以粗体显示，必填

- 定制列表

      默认的列表间距比较宽，LATEX 本身也未提供方便的定制功能，可用 enumitem 宏包定制各
      种列表间距。enumitem 宏包还提供了对列表标签、引用等的定制。有兴趣的读者可参考其帮助
      文档。

#### 对齐环境

如字面意思，可以在如下环境中实现不同对其效果的对齐

```latex
\begin{center} … \end{center}
\begin{flushleft} … \end{flushleft}
\begin{flushright} … \end{flushright}
```

也可以直接通过如下命令直接改变后续文字的对齐方式

```latex
\centering \raggedright \raggedleft
```

三个命令和对应的环境经常被误用，有直接用所谓 \flushleft 命令或者 raggedright 环
境的，都是不甚严格的用法（即使它们可能有效）。有一点可以将两者区分开来：center 等环境
会在上下文产生一个额外间距，而 \centering 等命令不产生，只是改变对齐方式。比如在浮动
体环境 table 或 figure 内实现居中对齐，用 \centering 命令即可，没必要再用 center 环境。

#### 引用环境

```latex
\begin{quote} %或用quotation
Knowledge is power.
\end{quote}
```
quote 用于引用较短的文字，首行不缩进；quotation 用于
引用若干段文字，首行缩进。引用环境较一般文字有额外的左右缩进。

#### 代码环境

使用`listing`或`minted`宏包（不推荐使用lshort上介绍的方法）

- `listing`无需额外安装，一个基本的示例如下

    ```latex
        \documentclass{article}
        \usepackage{listings}
        \usepackage{xcolor} % 需要用于颜色高亮
        \begin{document}
        \begin{lstlisting}[language=Python,keywordstyle=\color{blue}]
        import numpy as np
        # Your code here
        \end{lstlisting}
        \end{document}
    ```

    对于`listing`可以通过`\lstset`命令设置全局代码央视，例如定义关键词颜色、颜色背景以及字体等

- `minted`

    这个宏包非常强大，它支持超过150种编程语言的高亮，并且可以自定义样式。使用 minted 需要你的 LaTeX 编译器支持外部程序 Pygments，因此在使用时可能需要在你的编译环境中安装 Pygments。

    ```shell
    pip install Pygments
    ```

    一个基本的示例是：

    ```latex
    \documentclass{ctexart}

    \usepackage{minted}

    \setminted{
        frame=lines
        tabsize=4
        encoding=utf8
    }

    \begin{document}


    \begin{minted}{C}
    #include <stdio.h>

    int main(void) {
        printf("hello world!");
        return 0;
    }
    \end{minted}


    \end{document}
    ```

#### 表格

建议直接通过[在线绘制表格](https://www.tablesgenerator.com/#)绘制表格导出代码

或通过其他软件绘制表格输出为图片，以图片形式导入

#### 图片

latex本身不支持图片，需要通过`graphicx`宏包实现。

这里比较推荐图片通过浮动体实现（latex预定义了figure、table两个浮动提环境）

```latex
\begin{figure}[htbp!] 
%h:当前位置 t:顶部 b:底部 p:单独成页 !:决定位置时忽略限制
\centering %对齐方式
\includegraphics[width=0.6\textwidth]{figures/image.png}
\caption{题注}
\lable{}
\end{figure}
```

排版位置的选取与参数里符号的顺序无关，LATEX 总是以 h-t-b-p 的优先级顺序决定浮动体位置。
也就是说 [!htp] 和 [ph!t] 没有区别。

在 \includegraphics 命令的可选参数 ⟨options⟩ 中可以使用 ⟨key⟩=⟨value⟩ 的形式，常用
的参数如下：

| 参数              | 含义                    |
| --------------- | --------------------- |
| width=⟨width⟩   | 将图片缩放到宽度为 ⟨width⟩     |
| height=⟨height⟩ | 将图片缩放到高度为 ⟨height⟩    |
| scale=⟨scale⟩   | 将图片相对于原尺寸缩放 ⟨scale⟩ 倍 |
| angle=⟨angle⟩   | 将图片逆时针旋转 ⟨angle⟩ 度    |

graphicx 宏包也支持 draft/final 选项。当 graphicx 宏包或文档类指定 draft 选项时，图
片将不会被实际插入，取而代之的是一个包含文件名的与原图片等大的方框。

table 和 figure 两种浮动体分别有各自的生成目录的命令：

```latex
\listoftables
\listoffigures
```

他们类似`\tableofcontents`生成单独的章节

##### 并排与子图片

我们时常有在一个浮动体里面放置多张图的用法。最简单的用法就是直接并排放置，也可以
通过分段或者换行命令 \\ 排版多行多列的图片。以下为示意代码

```latex
\begin{figure}[htbp]
\centering
\includegraphics[width=...]{...}
\qquad
\includegraphics[width=...]{...} \\[...pt]
\includegraphics[width=...]{...}
\caption{...}
\end{figure}
```

如果给不同图片加上不同标题，需要使用盒子中的（minipage），如

```latex
\begin{figure}[htbp]
    \centering
    \begin{minipage}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{minipage}
    \qquad
    \begin{minipage}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{minipage}
\end{figure}

而如果给有统一标题的图片定义小标题，需要用到`subcaption`宏包的功能，比如

```latex
\begin{figure}[htbp]
    \centering
    \begin{subfigure}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{subfigure}
    \qquad
    \begin{subfigure}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{subfigure}
\end{figure}
```


未完待续，后续会填坑数学公式、引用文献、个性格式设置等等

## 参考

* lshort-zh-cn：一份 (不太) 简短的 LATEX 2e 介绍
* https://maoyachen.com/posts/2019/12/22latex-minted-usage/
* https://smallsquare.github.io/LaTeX-insert-images/
* https://www.philfan.cn/Tools/latex#iguanatex-latex-in-ppt