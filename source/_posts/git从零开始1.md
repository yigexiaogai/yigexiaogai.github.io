---
title: git从零开始1
date: 2023-01-28 15:17:39
tags: [git, 学习笔记]
categories: [git]
mathjax: true
cover: https://s2.loli.net/2023/01/27/476KjUiwNEzkVqb.png
---

# git指令与操作
顺着上一篇博客[上一篇博客链接](https://yigexiaogai.github.io/2023/01/14/git%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B0/) ，这一节我们来讲具体的git指令和操作了。

## 初始化本地库
初始化本地库的指令是
```git
git init
```

{%asset_img 1.png%}

使用这个命令的意思就是，用户将要使用git来管理项目，而不是其他方法（例如手动管理）。
此时文件夹中会出现一个名字为 **".git"** 的文件夹，但改文件夹默认为隐藏文件夹。（就是为了不要让普通人随意更改其中的文件，至于这之中的内容，我们要放到之后再说）

{%asset_img 2.png%}

* 可以看到，需要使用 **“ll -a”** 指令才能展示隐藏文件夹。

## 查看本地库状态
查看本地库状态的指令是
```git
git status
```

如图所示：

{%asset_img 3.png%}

在git的返回信息中， **“on branch master”** 表示此时项目在默认的分支“master”上， **“no commits yet”** 表示还没有提交任何东西，毕竟是空文件夹嘛，而且它也告诉你 **“nothing to commit”** 。

所以让我们来新建一个test用的文本文件来试一下：
1. 输入vim hello.txt，该命令新建一个hello的文本文件并进入输入界面
2. 按“i”进入输入模式
3. 输入“Hello world!”或者别的内容
4. 按“esc”退出
5. 将光标回退到第一行第一格，使用“yy”复制第一行，使用“p”黏贴内容，我粘贴了三行。
6. 输入“:wq”保存退出

* 这些用到的指令都是linux（包含vim）中的，因为git原先是基于linux内核开发的工具，所以。。。

{%asset_img 4.png%}

此时再输入“git status”，会发现反馈信息变了，git提示你有一个红色名字的文件“untracked（未追踪）”，这是因为你还没有告诉git这个你要对这个文件做什么，但是它推荐你使用 **“git add”** 命令将该文件加入到暂存区中，也就是说， **追踪它！** 。

{%asset_img 5.png%}

## 添加暂存区
添加暂存区的指令自然就是上面提到的
```git
git add
```

具体来说，有几种常见方法。
第一，添加所有未追踪文件进入暂存区
```git
git add .   //或者
git add -A
```

“.”表示当前文件夹，“-A”表示“ALL”所有文件，因为我们命令台打开的位置就是当前文件夹，所以这两个指令是一样的。
第二，添加指定文件
```git
git add hello.txt
```

执行之后，再看一下状态，你会发现git用绿色的字告诉你，有新文件被追踪了！

{%asset_img 6.png%}

* 注意这里有一个warning，意思是说，你的 **文章末尾的换行符会从LF格式替换成CRLF格式**, 因为你是万恶的windows系统啦~~ 在linux中人家和你不一样啦~ 但是你如果是高贵的mac，就会发现，没有这个警告，总之，不用管他~

同时git提示你，使用
```git
git rm --cached <file>
```
可以删除刚刚加入暂存区的文件，如图所示：

{%asset_img 7.png%}

## 提交本地库
将文件提交到暂存区之后，就要将文件提交到本地库了，这就形成一次完整的版本！其命令是
```git
git commit -m "XXX" <file>
```

* -m "XXX" 是提交日志的信息，如果你只输入git commit，界面会自动跳转到让你输入日志信息。输入“#”则是保留为空信息。

提交完成后，再次查看状态，git会告诉你“working tree clean”，工作树已经清空了，没有新的要递交的文件了。

{%asset_img 8.png%}

使用指令
```git
git reflog
```

可以查看提交库的日志信息，例如这里的反馈信息中的“c95011e”就是版本号。一般来说，这是自己做开发时使用的命令，如果是团队开发，则可以使用显示递交人的指令：
```git
git log
```

在这里，反馈信息就会提供完整的版本号和提交人信息了。团队开发很有用哦！

{%asset_img 9.png%}

## 修改文件
如果对项目中的文件进行了修改，被修改的文件实际可以上可以被理解为新的文件，实际上也就是版本更新了，要提交修改过的文件实际上也是相同的操作，即上面的命令流程，这里就不做重复解释了。

## 总结
这一节我们学习了对于本地库中文件的提交流程以及状态查看，至此，其实我们已经算得上是在用git管理项目了。下一节将会开始讲解项目版本的管理以及如何把项目推送到远端，例如github。敬请期待。
![](https://s2.loli.net/2023/01/28/rgB7yFv8cdwG9OT.jpg)
