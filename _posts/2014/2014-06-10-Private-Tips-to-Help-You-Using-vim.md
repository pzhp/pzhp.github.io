---
layout: post
title: 迅速提升使用VIM幸福感的三招
categories:
- Common Tec
tags:
- vim
---



##引言

> 很多时候使用Vim的痛点在于有些操作总是不方便，不能流畅使用，本文是作者解决自己痛点时积累下来的，应该对其他人同样有用。
> 学习VIM的诀窍就是在用中学，所以您在看本文的时候最好打开VIM顺便实践一下。
> 由于本人需要同时使用众多的SERVER，不可能到处装插件，所以本文中所有的技巧都是VIM中自带的，不涉及插件。

--------


##第一招，文件浏览类


```
:pwd
:cd
:e .
:Sex
:bro ol
```

大多数VIM教程关注的都是文本编辑本身，所以如何方便的打开文件就被忽略了。
首先要明白VIM中有当前文件夹（CWD）的概念，默认是你打开VIM时所在的文件夹。你可以用与Linux一样的`:pwd`查看，
接下来你可以`:cd`到其他文件夹，再接下来就可以用`:e . (:edit .)`来浏览当前目录，vim会在一个新window中打开目录，你还可以进行查看父目录，排序等各种操作。这里顺便提一下用vim也可以打开各种压缩文件来查看目录结构。

上面的方式可以让你切换目录并打开文件。由于vim支持通过多窗口和多Tab同时编辑多个文件，假如你要不停的修改两个目录下的不同文件，那么用上面的方式不停的切换当前目录也是不方便的，是时候就轮到`:Sex `(或者:Sex!)了，它直接在新window中打开当前文件（而不是CWD）所在的目录，这样可你让你方便的浏览不同的目录。

同样是浏览目录，如果你用的是GUI版的vim，比如Windows版的GVIM，这时可以用`:bro e` （:browse explore）指令打开操作系统如Windows的文件浏览器。

最后一个快速打开文件的方法就是从编辑过的文件历史中选择，指令`:bro ol`（:browse old）可以打开编辑历史供你选择要打开的文件。

上面`:Sex`方式打开的文件是在一个Window里，如果你想在一个单独的Tab里编辑文件又该怎么办呢？`ctrol + w T`可以将当前的Window移动到一个新的Tab里。


##第二招，查看历史类

使用vim时需要键入不少的指令，有些还相当的复杂，比如查找替换时用的正则表达式，上面提到过的历史编辑文件等。复用那些之前的指令可以大幅提升效率，各类常用查看历史的指令罗列如下：

```
q/            查看搜索历史
q:            查看和执行历史指令，不可修改
:+up/down     查看指令历史，可修改       
:reg          查看剪贴板和宏历史
:marks        查看所有mark过的位置
```

##第三招，另类搜索



当一个patten的match太多的时候，用n在match中跳跃也比较费眼，这种情况我一般用`:vimgrep`，它可以把所有匹配显示在一个quickhelp窗口，然后你可以用`:cn` `:cp`等在结果中跳跃（也可用鼠标）。如果如果不需要跳转的话可以简单的键入`:g /patten`也会新开一个窗口并列出所有匹配。

```

:vimgrep patten %   %指当前文件名
:copen 
:cn
:cp
:g /pattern

```

##其它提升幸福感的方法

1.  使用`Ctrl+c`替代`ESC`。这是因为在Vi发明的时代ESC其实是在CapsLK的位置的，习惯`Ctrl+c`后会发现更幸福。
1.  安装中文帮助文档，提升阅读文档的幸福感。
1.  学习使用Shift键，比如`H` `J` `K` `L` `M` `I` `A` `O`等提升操作的流畅性。

---------

> 觉得本文还过得去的话，别忘了回到主页顺便看看作者的其它原创文章哟？