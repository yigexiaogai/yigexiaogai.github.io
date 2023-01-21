---
title: matlab基础 工程篇（三 线性方程式和线性系统）
date: 2022-07-09 14:13:03
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

# matlab中求解线性方程组
在线性代数的课程中，我们学习过求解一个线性方程组，可以将方程组写成一个矩阵，然后用初等行变换进行求解。例如:

{%asset_img 1.png%}

## 高斯消去法（Gaussian Elimination）和高斯约旦消去法（Gauss-Jordan Elimination）
上述图片的步骤就是高斯消元法。我们通过矩阵中 **1.行间的交换2.行间的加减3.行的倍数** 这三种变换，将矩阵变成一个上三角矩阵，然后将最后一行的解带入方程，一个一个回代求出所有的解。不过在计算机里，我们不能手动回代求解，所以我们可以让计算机继续简化这个矩阵，仍然用行变换的规则，将矩阵变成对角矩阵，这样解就都出来了，这就是高斯约旦消元法，实际上就是在高斯消元法的基础上继续往下做而已。

可以参看[这篇博文](https://blog.csdn.net/IAN27/article/details/108280355?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108280355-blog-12954213.pc_relevant_multi_platform_whitelistv1_exp2&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-108280355-blog-12954213.pc_relevant_multi_platform_whitelistv1_exp2&utm_relevant_index=1)，有图，很详细.

### rref()
在matlab中，可以使用rref方法求解线性方程组，且rref函数用的方法是高斯约旦消元法。

```matlab
A = [1 2 1; 3 5 6; 4 5 2;];
b = [4 7 9]'; % 注意这里有个转置符号
format rat % 结果以分数形式显示
r =rref([A b])
```

## LU Factorization
在我们求 $Ax=b$ 时，如果A非常复杂，那么直接求x比较困难，即使使用高斯消元法，可能要做好多次变换，这就造成计算量大，那么我们可不可以规避大的计算量，尽量用**矩阵乘法**来解决这个问题呢？LU分解法（LU Factorization）就可以做到。

LU分解法的主要思想如下：
1. 在求解 $Ax=b$ 时，我们先将 $A=LU$，L是一个下三角矩阵，U是一个上三角矩阵，于是 $Ax=b$ 转化成 $LUx=b \rightarrow L(Ux)=b$
2. 然后我们令 $Ux=y$
3. 所以我们把 问题转化为两个步骤：
   * 求解 $Ly=b$ 中的 $y$
   * 解的 $y$ 后，求解 $Ux=y$ 中的 $x$

第三点的两步都很简单，因为L和U都是三角矩阵，相当于给你做好了高斯消元法，所以很容易通过高斯消元法的回代得到解。**那么关键就在于如何得到L和U。**

我们可以一步一步的将矩阵A消成上三角矩阵U！
即：
$$ L_1L_2L_3 \dots L_m A =U$$

$$ L=L_1L_2L_3 \dots L_m $$

给出一个例子：
有矩阵A：
$$ A=\begin{bmatrix}
   1 & 1 & 1 \\
   2 & 3 & 5 \\
   4 & 6 & 8
\end{bmatrix}$$

先找一个下三角矩阵L1，使得A的第二三行第一个元素为0：
$$L_1*A=\begin{bmatrix}
   1 & 0 & 0 \\
   a & 1 & 0 \\
   b & c & 1
\end{bmatrix} *\
\begin{bmatrix}
   1 & 1 & 1 \\
   2 & 3 & 5 \\
   4 & 6 & 8
\end{bmatrix}=
\begin{bmatrix}
   1 & 1 & 1 \\
   0 & ? & ? \\
   0 & ? & ?
\end{bmatrix}
$$

我们可以得到一组解a=-2;b=-4;c=0.
$$L_1*A=\begin{bmatrix}
   1 & 0 & 0 \\
   -2 & 1 & 0 \\
   -4 & 0 & 1
\end{bmatrix} *\
\begin{bmatrix}
   1 & 1 & 1 \\
   2 & 3 & 5 \\
   4 & 6 & 8
\end{bmatrix}=
\begin{bmatrix}
   1 & 1 & 1 \\
   0 & 1 & 3 \\
   0 & 2 & 4
\end{bmatrix}
$$

接着我们要找一个L2，使得A的第三行第二个元素为0：
$$L_2*(L_1*A)=\begin{bmatrix}
   1 & 0 & 0 \\
   0 & 1 & 0 \\
   0 & d & 1
\end{bmatrix} *\
\begin{bmatrix}
   1 & 1 & 1 \\
   0 & 1 & 3 \\
   0 & 2 & 4
\end{bmatrix}=
\begin{bmatrix}
   1 & 1 & 1 \\
   0 & 1 & 3 \\
   0 & 0 & ?
\end{bmatrix}
$$

我们可以得到解d=-2.

$$L_2*(L_1*A)=\begin{bmatrix}
   1 & 0 & 0 \\
   0 & 1 & 0 \\
   0 & -2 & 1
\end{bmatrix} *\
\begin{bmatrix}
   1 & 1 & 1 \\
   0 & 1 & 3 \\
   0 & 2 & 4
\end{bmatrix}=
\begin{bmatrix}
   1 & 1 & 1 \\
   0 & 1 & 3 \\
   0 & 0 & -2
\end{bmatrix}
$$

所以我们找到了U和L：
$$U=\begin{bmatrix}
   1 & 1 & 1 \\
   0 & 1 & 3 \\
   0 & 0 & -2
\end{bmatrix};
L=L_1*L_2=
\begin{bmatrix}
   1 & 0 & 0 \\
   -2 & 1 & 0 \\
   0 & -2 & 1
\end{bmatrix}
$$

**看上去还是很麻烦对吧，但如果计算机来做，他就可以避免很多行变换的计算，反而增加了速度。**

## \运算符
虽然说这些算法有点小复杂，但matlab中，你可以完全不管这些原理，只要用“\”符号就可以算出x了。

```matlab
A = [1 2 1; 2 6 1; 1 1 4;];
b = [2 7 3]';
R =A\b
% 此时
R =
      -3       
       2       
       1    
```
* 使用rref方法也可以得到，不过其返回的是一个带有对角矩阵的矩阵，最后一列是解向量。\算符可以直接得到解向量。


## Cramer's method
克莱默法则（Cramer's method）实际就是用逆矩阵求解，而根据该法则，逆矩阵求起来非常麻烦，所以不实用，效率低下。

$$求 Ax=b \\
则 x=A^{-1}b \\
A^{-1}=\frac{1}{det(A)}adj(A)$$

这里det是行列式，adj是伴随矩阵，这个伴随矩阵，在矩阵三阶及以上的时候特别难算，具体可以百科一下伴随矩阵。不过matlab中你不用考虑这些，$A^{-1}$用inv(A)就可以了。

```matlab
A = [1 2 1; 2 6 1; 1 1 4;];
b = [2 7 3]';
R =inv(A)*b %结果和A\b一样
```

**然而，这是有问题的。** 我们看到在求逆矩阵的时候，公式里面有一个 $A^{-1}=\frac{1}{det(A)}$ ，这就糟了，有可能 $det(A)=0$ 哦，也就是说，不是所有矩阵都有逆矩阵。根据线代的知识，我们知道，奇异矩阵没有逆矩阵。所以综上，inv这个方法存在其局限性。

## cond()
cond方法是用来判断一个矩阵的健康程度的。我们先看一个矩阵：

$$A=\begin{bmatrix}
   1 & 2 & 3 \\
   2 & 4 & 6 \\
   0 & -2 & 1
\end{bmatrix}$$

矩阵A就是一个奇异矩阵，因为第二行是第一行的两倍，该矩阵行列式等于0.

再看一个矩阵：

$$A'=\begin{bmatrix}
   1 & 2 & 3 \\
   2 & 4.0001 & 6 \\
   0 & -2 & 1
\end{bmatrix}$$

该矩阵不是一个奇异矩阵，但是大差不差，这就说明该矩阵健康程度低。那么如何用数学语言说明呢？

假如说我们对A有一个等式 $Ax=b$ 成立，此时对A进行一个小小的改动，称为 $\delta A$，如果要让等式继续成立，那么x要有多少的改动。来分析一下：
$$Ax=b \\
x=A^{-1}b$$

如果A很接近一个奇异矩阵，那么小小的改动就会造成 $A^{-1}$ 变得很大，x就变得很大——x改动越大，说明A的健康程度越差。

我们再引入一个常数k来表征（放大）x的改动，如果A的改动需要乘以一个很大的数才能大于x的改动，那么就说明A 健康程度很差。这就是“条件数”cond()的原理。

$$\frac{||\delta x||}{||x||}<k(A)\frac{||\delta A||}{||A||}$$

```matlab
A = [1 2 3; 2 4 6; 9 8 7]; cond(A)
B = [1 2 3; 2 4.0001 6; 9 8 7]; cond(B)
```

# 线性系统（linear system）
线性系统是线性方程组对立的一面。如果收线性方程组是要求解Ax=b，那么线性系统就是要要讨论和分析y=A*b中A怎么影响y。

## 特征值和特征向量
$A*b$ 两个矩阵相乘有些复杂，为研究线性系统，我们要尝试简化这个式子，那么能不能找到一个常数 $\lambda$和一个向量v，使 $A*v=\lambda *v$，这就是特征值和特征向量了。

同时，b这个向量可以被分解 $b=v_1+v_2+v_3+\dots+v_n$，那么：

$$A*b=A*(v_1+v_2+\dots+v_n) \\
     =\lambda_1 *v_1+\dots+\lambda_n *v_n$$

于是，一个系统A对向量b的影响找到了，实际上就是表达成b在各个分量上的伸缩倍率。那么系统对一个向量组成的平面，实际上就有了固定的影响。

### eig()
使用eig方法可以找到矩阵A的特征值和特征向量：

```matlab
A=[2 -12; 1 -5];
[V D]=eig(A) # V是特征向量，D是特征值，在矩阵对角线上。
```

**此小节结束，你真厉害！**

![](https://s2.loli.net/2022/07/06/Kh4zpItmHrbkAPN.jpg)



