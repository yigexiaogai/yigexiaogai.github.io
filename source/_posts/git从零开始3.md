---
title: git从零开始3
date: 2023-02-16 19:47:38
tags: [git, 学习笔记]
categories: [git]
mathjax: true
cover: https://s2.loli.net/2023/01/27/476KjUiwNEzkVqb.png
---


# 团队协作和结合IDEA
上一节我们接触了 **版本穿梭、分支和远端推送** ，这说明我们已经能够在本地和远端管理项目了，接下来我们就需要在团队中合作了。

## 团队协作
团队协作其实是比较简单的一块内容，其实就是在github上邀请团队中的成员，当成员同意之后，就可以修改代码，并且推送到远端，覆盖原有项目了。
具体可以看这个视频， 这也是我在学习的教程：
[尚硅谷git教程](https://www.bilibili.com/video/BV1vy4y1s7k6?p=24&vd_source=3d73f51879206e4a4df14c2d1fb027e7)

## 结合IDEA
主流的idea都支持git，因为作者本人用VS code比较多，所以在这里以Code为实例。

code在侧边栏中有一个路线的符号:

{%asset_img 1.png%}

点击进入后，就会发现code提示了用户当前有的分支，当前文件的状态，当前可以执行的操作。
当有新文件时，code会在新文件后面加上“U”的字样，表明它未被追踪。当已经被追踪的文件内容有所改变时，code会在文件的后面加上“M”的字样，表明它已经被修改。点击该文文件，他就会告诉了你新加了哪些内容（或者删去了哪些内容）：

{%asset_img 2.png%}

code提供了一个“+”按钮，让用户可以暂存更改，对应的有一个“-”号，用来撤回更改。

接下来就是提交： code需要你在提交之前在文本框输入提交信息，然后进行提交。但是注意，这里的提交只是本地提交，实际上就是“git commit”，如果要更新到远端，则还要点击“同步按钮”，这个操作就等于“git pull”.

{%asset_img 3.png%}

## 小结
其实到这里，内容就基本上讲完了，还有一些特别细节的地方，或者不太常用的指令，我想这些就只有到真正工作中开发才能用到了。其实git内容不多，要使用并不难，但理解其底层逻辑才是最重要的。在这里贴上一下我学习，引用，参考的素材：
1. [一个很有用的学习git的游戏](https://learngitbranching.js.org/?locale=zh_CN)
2. [尚硅谷课程](https://www.bilibili.com/video/BV1vy4y1s7k6/?spm_id_from=333.337.top_right_bar_window_custom_collection.content.click)
3. [VS Code里使用Git和关联GitHub](https://www.bilibili.com/video/BV1r3411F7kn/?spm_id_from=333.337.search-card.all.click&vd_source=3d73f51879206e4a4df14c2d1fb027e7)

最后，其实我想说，我觉得还是命令行好，用idea总没那么舒爽~

![](https://s2.loli.net/2023/01/28/rgB7yFv8cdwG9OT.jpg)