# LaTeX

## 安装和入门

建议在Linux系统安装texlive工具包。从texlive官网上[下载](http://tug.org/texlive/acquire-netinstall.html)，根据里面的"quick install instructions"一步步做就可以了。安装过程需要联网，时间比较长（我当时用了40分钟左右），耐心等待一段时间。

下载安装好之后，下载[first-latex-doc](http://www.ctan.org/pkg/first-latex-doc)教程进行学习，教程提供了入门文档和示例代码，可以很快入门。

## 中文支持

pdflatex对中文的支持不太好，中文无法在PDF文档中显示。要让LaTeX支持中文，目前比较好的方法是使用xelatex+xeCJK。xelatex已经包含在texlive工具包里。

首先你要保证系统里有你喜欢的字体，参见Linux章节的“[显示](../chapter-linux/display.md)”部分。记住你需要使用的字体名（如宋体是"SimSun"），在tex文档里这么写：

```TeX
% 设置中文字体
\usepackage{xeCJK}
\setCJKmainfont{SimSun}
```

注意设置字体的时候使用`\setCJKmainfont`，这样只会设置中文为宋体，英文还是默认的字体。如果设置`\setmainfont{SimSun}`，英文也会变成宋体，那样就太丑了。

使用xeCJK的时候有什么疑问，直接在命令行运行`texdoc xeCJK`即可。

## 插入图片

插入图片使用graphics或graphicx宏包，插入的方法可以参考《LaTeX插图指南》。

## 代码高亮

插入代码可以使用listings宏包，但listings的效果太丑，尤其是字体竟然不是等宽字体。要实现比较好看的效果，可以使用minted宏包。运行`texdoc minted`查看文档。注意使用minted的时候需要加一个编译选项。

```Shell
xelatex -shell-escape a.tex
```

具体内容参看文档。

## TikZ

TikZ是LaTeX环境的绘图工具，关于TikZ的使用方法可以参考[TikZ](../chapter-draw/tikz.md)部分。

