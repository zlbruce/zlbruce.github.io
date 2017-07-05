---
title: 一些 LaTeX 的笔记
author: zlbruce
layout: post
permalink: /2007/06/18/latex-note/
categories:
  - 笔记
tags:
  - LaTeX
  - VIM-LaTeX
---
其实从大二开始我的论文都是用 LaTeX 来写的，因为只是在期末的时候用用，所以每次用的时候都需要在网上找资料来解决某些问题。这次的毕业设计也不例外，所以决定记下来，以备后用，不过也不知道以后能不能用到她了。除了使用 LaTeX 之外，这次还学习了如何使用 metapost 来画图，以前一直以为很难，不过画的时候还是觉得蛮简单的，反正也就只用了一些简单的东西。

由于我一直是使用大二时折腾 LaTeX 的时候的模板，每次期末都会遇到问题，都会改进一点。而我记性又不大好，以前遇到的问题都忘了，所以下面基本上都是我这次遇到的问题。

### 参考文献的引用使用上标

使用如下语句添加一个新的命令 {% ihighlight tex %}\upcite{% endihighlight %} ：

{% highlight tex %}\newcommand{\upcite}[1]{\textsuperscript{\cite{#1}}}{% endhighlight %}

然后在文档中使用 {% ihighlight tex %}\upcite{% endihighlight %} 来引用参考文献即可

### 将“参考文献”四个字居中

我是使用的下面的命令

{% highlight tex %}\renewcommand{\refname}{\centerline{参考文献}}{% endhighlight %}

### 使有题目的一页也具有页眉页脚

使用 {% ihighlight tex %}\maketitle{% endihighlight %} 后会改变其页面样式，可以用 {% ihighlight tex %}\thispagestyle{XXX}{% endihighlight %} 修改回来，其中 XXX 是你的页面样式。比如：{% ihighlight tex %}\thispagestyle{fancy}{% endihighlight %}

### 目录的样式

其实我觉得 LaTeX 自动生成的目录的样式挺好看的，可是我的指导老师非要我在 section 和页码之间加上点&#8230;使用 titletoc 宏包，命令如下：

{% highlight tex %}\titlecontents{section}[0pt]{}{\hei\thecontentslabel\quad}{}
{\hspace{1em}\titlerule*[10pt]{.}\contentspage}{% endhighlight %}

### 在目录中添加目录项

可以使用 {% ihighlight tex %}\addcontentsline{% endihighlight %} 命令，比如：

{% highlight tex %}\addcontentsline{toc}{section}{\hei 参考文献}{% endhighlight %}

其中 toc 表示 table of contents，说明加入到什么目录，类似的还有 lof(list of figures)和 lot(list of tables)。section 表示加入到目录的 section 一层，当然可以改为 subsection 之类的。在这个命令之前还可能需要加入 {% ihighlight tex %}\clearpage{% endihighlight %}  或者 {% ihighlight tex %}\cleardoublepage{% endihighlight %} ，如果使用了 hyperref 宏包，则还需要加入 {% ihighlight tex %}\phantomsection{% endihighlight %} 。

### 关于数学公式

其实我的文档很少使用数学公式，需要在一行描述性的文字中写上数学公式，直接使用 {% ihighlight tex %}$\sum_{j = i}^nX_j${% endihighlight %}  出来的效果如下：  

![sum]({{ site.url }}/images/2007-06/latex-note-1.jpg)

如果想使用和 {% ihighlight tex %}\[\sum_{j = i}^nX_j\]{% endihighlight %}  一样的效果可以使用 {% ihighlight tex %}$\displaystyle\sum_{j = i}^nX_j${% endihighlight %} ，效果如图：  

![sum]({{ site.url }}/images/2007-06/latex-note-2.jpg)

### 关于 VIM-LaTeX

Vim 是我非常喜欢的编辑器，装上 [VIM-LaTeX][1] 插件后写 LaTeX 的文档很不错，下面是我的一些配置：

{% highlight vim %}let g:Tex_DefaultTargetFormat = 'pdf' "编译为 PDF
let g:Tex_FormatDependency_pdf = 'dvi,gbk2uni,oneLatex,pdf' "设置编译的顺序
let g:Tex_CompileRule_dvi = 'latex -src-specials --interaction=nonstopmode $*'
let g:Tex_CompileRule_oneLatex = 'latex -src-specials --interaction=nonstopmode $*' "使用 gbk2uni 处理后再运行一次 latex
let g:Tex_CompileRule_gbk2uni = 'gbk2uni $*.out' "使用 gbk2uni 处理书签
let g:Tex_CompileRule_pdf = 'dvipdfmx -o $*.pdf $*.dvi'
let g:Tex_ViewRule_pdf = '/the/path/to/your/PDFreader' " 设置 PDF Reader，修改为你自己的{% endhighlight %}

 [1]: http://vim-latex.sourceforge.net/ "VIM-LaTeX"