---
title: matlab基础 工程篇（四 统计）
date: 2022-07-11 11:17:44
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

# matlab中的统计
其实统计我们从小学到大学都有学习，已经是老朋友了，没有太多好说的，统计主要是需要我们掌握一些统计方法以及概率统计的知识。

主要的一些统计方法有：
* 平均数（mean）
* 中位数（median）
* 众数（mode）
* 四分位数（quartile）
* 范围（range）
* 方差（variance）
* 标准差（standard deviation）

## 方差和标准差
值得回顾的只有方差和标准差，因为其用到了概率统计的知识。matlab中的方差和标准差都是针对样本而言的，不是对于整体。

### var()和std()
var方法表示求样本方差，公式为：
$$ S^2=\frac{\Sigma(x_i-\bar x)^2}{n-1}$$

std方法表示求样本标准差，公式为：
$$ S=\sqrt\frac{\Sigma(x_i-\bar x)^2}{n-1}$$

## 绘制统计图
先看以下代码：
```matlab
x=1:14;
freqy=[1 0 1 0 4 0 1 0 3 1 0 0 1 1];
figure
subplot(1, 3, 1); bar(x, freqy); xlim([0 15]);
subplot(1, 3, 2); area(x, freqy); xlim([0 15]);
subplot(1, 3, 3); stem(x, freqy); xlim([0 15]);
```

{%asset_img 1.png%}

我们有若干种绘制统计图的方法，例如柱状图，区域图，点线图等。freqy代表的是x=n时y出现的次数。

## 箱线图（boxplot）
{%asset_img 2.png%}
箱子的中间一条线，是数据的中位数，代表了样本数据的平均水平。

箱子的上下限，分别是数据的上四分位数和下四分位数。这意味着箱子包含了50%的数据。因此，箱子的宽度在一定程度上反映了数据的波动程度。

在箱子的上方和下方，又各有一条线。有时候代表着最大最小值，有时候会有一些点“冒出去”。如果有点冒出去，理解成“异常值”就好。

箱线图大致描绘了一个样本的分布，之后会联系到一部分概率与统计的知识，由于该部分暂且不会用到，也比较少用，因此就不继续整理了。