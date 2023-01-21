---
title: matlab基础 工程篇（一 数值微积分）
date: 2022-07-05 16:32:34
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

# 前言
matlab是一款功能强大的计算工具。最近我重新学习了计算机视觉的相关知识，其中，使用matlab编写程式码是不可缺少的环节。然而在过程中我遗忘了许多的编写规则和内置函数，因此，为了重新掌握matlab语言，我试图寻找一些基础的课程或者教程。最终，我在B站上发现了一个讲解得极其优秀的课程，是由台湾大学教授郭彦甫主讲的《**matlab教学**》。

该课程共分14P，13节基础内容。涵盖初中级、高级、影像处理、工程相关四块内容。我在写这篇博客之前，已经将这些内容都学习完成，写文章的主要目的是回顾和总结学到的知识。而为了趁热打铁，**博客开头将会从我最近学完的内容写起，即“工程相关”的一块。**

在使用matlab之前，请确保拥有一些必要的**高等数学、微积分、线性代数、概率统计**知识。（不过别担心，稍微复杂的地方在课程中会有解释）

**这里附上B站视频链接：[MATLAB教程_台大郭彦甫（14课）原视频补档](https://www.bilibili.com/video/BV1GJ41137UH?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click&vd_source=3d73f51879206e4a4df14c2d1fb027e7)** 

最后我要再一次强烈安利一下这门课：**台湾口音很可爱、课程内容不困难、老师讲解很细致，总之就是非非非常的nice!**

下面让我们开始吧！

# matlab与数值微积分
matlab与微积分的联系主要在关于两块内容：多项式微分、积分（polynomial differentiation and integration）和数值微分、积分（Numerical differentiation and integration）

多项式方面主要与多项式求导、积分的公式相关，数值方面主要与求导数定义方法有关。

## matlab中的多项式微分
在微积分与高等数学中我们知道，多项式（polynomial）可表达为：
$$ f(x)=a_nx^n+a_{n-1}x^{n-1}+\dots+a_1x+a_0 $$

其导数形式为：
$$ f'(x)=na_nx^{n-1}+(n-1)a_{n-1}x^{n-2}+\dots+a_1 $$

对于一个多项式，在matlab中我们只需要考虑用系数表达就可以，例如以下这个多项式：
$$y=3x^3+x-4$$

我们可以提取系数写作一个向量复制给P：
```matlab
p = [3 0 1 -4]
```
* 显然原式中没有二次项，或者说二次项系数等于0.

### polyval()
在matlab中绘制一个多项式函数可以使用polyval的方法。

**例**，在定义域 $-2\leq x \leq 5$ 绘制函数 $y=5x^4+8x^2-2x+9$

我们可以在定义域上对函数进行采样，取许多点连成线即可。polyval方法实际上的工作就是，当x=a时，计算f(x)=f(a)的值。

```matlab
P=[5 0 8 -2 9]; x=-2:0.01:5;
f = polyval(P, x);
plot(x, f, 'r-', 'linewidth', 3');
xlabel('x'); ylabel('y=f(x)');
set(gca, 'fontsize', 14); % 设定窗口字体大小
```
{% asset_img 1.png %}

### polyder()
在matlab中绘制一个多项式的导函数可以使用polyder方法。
**例**，在定义域 $-2\leq x \leq 5$ 绘制函数 $y=5x^4+8x^2-2x+9$ 的导函数。

polyder的作用就是返回该多项式函数的导函数的系数向量。
$$[5,0,8,-2,9] \rightarrow [20,0,16,-2]$$

```matlab
P=[5 0 8 -2 9]; x=-2:0.01:5;
Q = polyder(P);
g = polyval(Q, x);

plot(x, g, 'b-', 'linewidth', 3');
xlabel('x'); ylabel('y=g(x)');
set(gca, 'fontsize', 14);
```

{% asset_img 2.png %}

**例题:**
Plot the polynomial 

$$f(x)=(20x^3-7x^2+5x+10)(4x^2+12x-3)$$

and its derivative for $-2 \leq x \leq 1 $

Hint: Try to use function **conv()**

```matlab
a = [20 -7 5 10]; b = [4 12 -3];
x = -2:0.01:1;
P =conv(a, b);
f = polyval(P, x);

figure
plot(x, f, 'b-.', 'linewidth', 2);
xlabel('x'); ylabel('y=f(x)');
set(gca, 'fontsize', 12);
```

{% asset_img 3.png %}

## matlab中的多项式积分
同样的多项式
$$ f(x)=a_nx^n+a_{n-1}x^{n-1}+\dots+a_1x+a_0 $$

其积分形式为：
$$ \int f(x)=\frac{1}{n+1} a_nx^{n+1}+\frac{1}{n}a_{n-1}x^{n}+\dots+\frac{1}{2}a_1x^2+a_0x+C(常数项) $$

### polyint()
> 因为积分之后表达式多出了一个常数项C，C可以为任意值，这使得表达式不唯一了。所以在绘制时需要给定常数项具体值，polyint方法的第二个参数就是给定的常数。

**例**，在定义域 $-2\leq x \leq 5$ 绘制函数 $y=5x^4+8x^2-2x+9$ 的积分函数，常数项 $C=3$

```matlab
P=[5 0 8 -2 9]; x=-2:0.01:5;
I = polyint(P, 3);
r = polyval(I, x);

plot(x, r, 'g-', 'linewidth', 3');
xlabel('x'); ylabel('y=r(x)');
set(gca, 'fontsize', 14);
```

## matlab中的数值微分
显然，对于多项式的微分和积分我们有现成的公式可以套用，可对于sin(x),cos(x),ln(x)这种，虽然许多我们也有公式，但是太不统一了，我们需要有一种统一的方法去描述，所以我们可以从导数定义法入手。

$$f'(x_0)=\lim_{\Delta x \rightarrow 0}\frac{f(x_0+\Delta x)-f(x_0)}{\Delta x}$$

这个表达式的右侧实际上是一个非常小的三角形（微分三角形）的tan值，可以近似于某一点的斜率。所以实际上，我们只要在原函数上采样，算一系列点的斜率，然后再绘制这一系列点，连起来即可，显然，这一系列点越近效果就越好。例如当差值为0.01，取x=0.01,0.02,0.03……

### diff()
我们可以使用diff方法批量处理差值。diff返回的就是一个向量中间元素的差值，例如：

```matlab
v = [1 2 5 2 1];
diff(v)

ans =[1     3    -3    -1]
```

* 注意方向，是后一个元素减去前一个元素。

应用到求斜率上呢？

```matlab
x = [1,2]; y=[3,4]; %已知两个点
slope = diff(y)./diff(x); % 求tan
```

再推广到函数导数上：
例如，要找y=sin(x)的当x=pi/2时的导数值，使用$\Delta x$ = 0.1：

```matlab
x0 = pi/2; dx=0.1;
x = [x0, x0+dx]; y = [sin(x0), sin(x0+dx)]
slope = diff(y)./diff(x);
``` 

再推广到整个区间上，例如当$-2\leq x \leq 5$时，sin(x)的导函数，为精确一点，另dx=0.01

```matlab
dx = 0.01; 
x = -2:dx:5; y=sin(x);
m = diff(y)./diff(x);
plot(x(1:end-1), m, 'linewidth', 2);
xlabel('x'); ylabel('y=sin’(x)');
set(gca, 'fontsize', 12);
```

* 注意，使用diff之后，差值构成的向量维度要比原数据构成的向量少1，所以要使用x(1:end-1)，截去最后一个元素。

{% asset_img 5.png %}

这里给出一个用不同精确度dx的例子，使dx分别等于0.1，0.01，0.001，0.0001,看看效果差多大。

```matlab
g = colormap(lines);
hold on
for i = 1:4
    x = -2:power(10,-i):5;
    y = sin(x);
    m = diff(y)./diff(x);
    plot(x(1:end-1), m, 'color', g(i,:));
end
hold off
h = legend('h=0.1', 'h=0.01', 'h=0.001', 'h=0.0001');
set(gca, 'xlim', [-2,5]);
xlabel('x'); ylabel('y=sin(x)');
```

{% asset_img 6.png %}

### 二次导数和三次导数
二次导数实际上就是对diff求得的差值向量再做一次差值，三次导数类似。例如：

```matlab
x = -2:dx:5; y=sin(x);
m = diff(y)./diff(x); % 一次导数
m2 = diff(m)./diff(x(1:end-1)) % 二次导数
m3 = diff(m2)./diff(x(1:end-2)) % 三次导数
```

## matlab中的数值积分
数值积分实际上可以看作求曲线与坐标X轴包围的面积。在之前我们已经在x轴上取出了一个一个的小范围，我们只需要用上这个结合函数划分一块一块小区域就行。不过区域也有不同的画法，主流的有三种方法（规则）。

{% asset_img 7.gif %}

### 中点法则（midpoint rule）
{% asset_img 8.jpeg %}

例如上图中最左侧的小长方形，其左侧为x0，右侧为x1，所以小矩形的面积为 $(x_1-x_0) \times f(\frac{x_0+x_1}{2})$  

**例题：** 计算$y=4x^3sin(x)$在$[0,2\pi]$上的积分。dx设置为0.02.

```matlab
dx = 0.02;
x = 0:dx:2*pi;
midpoint = (x(1:end-1)+x(2:end))./2; % 找到各个中点
y = 4*midpoint.^3.*sin(midpoint);
integral = sum(dx*y); % 使用sum进行求和
```

### 梯形法则（trapezoidal rule）
{% asset_img 9.png %}

如上图，实际上就是用一个梯形取代了矩形，我们就不需要求中点了，每一块梯形用梯形面积公式就可以。
matlab为我们提供了梯形法则的函数trapz()，当然我们也可以自己算。
**仍然用刚刚的例题：**

```matlab
dx = 0.02;
x = 0:dx:2*pi;
y = 4*x.^3.*sin(x);

% 用内置函数
integral1 = dx*trapz(y);

% 自己算
integral2 = sum((y(1:end-1)+y(2:end))*dx/2);

% 结果没啥区别。
```

### 辛普森法则（Simpson rule）
辛普森法则又被叫做三分之一辛普森法则（1/3 Simpson rule）

{% asset_img 10.png %}

实际上他是结合了上述的两种方法，一次性算两块小区域，同时考虑三个点的影响。如果说矩形的上端可以看作零次函数（曲线），梯形的上端可以看作一次函数（曲线），那么辛普森法则的小区域可以被看作一个二次函数（曲线）。用越高次的曲线去贴近原函数，当然就更加精确咯！

具体的表达式是这样的：
$$ \int_{x_0}^{x_2}f(x)dx \approx \frac{h}{3}(f(x_0)+4f(x_1)+f(x_2)) $$

**同样的例题：**

```matlab
dx = 0.02;
x = 0:dx:2*pi;
y = 4*x.^3.*sin(x);
integral = dx/3*sum(y(1:2:end-2)+4*y(2:2:end-1)+y(3:2:end))

% 可能会遇到小区域块为单数的情况，所以可以单独拿出第一块和最后一块，式子变化为
integral4 = dx/3*(y(1)+y(end)+2*sum(y(3:2:end-2))+4*sum(y(2:2:end-1)))
```

## 补充方法
实际上matlab中为我们准备了内置的积分函数integral()，不过使用这个函数需要我们了解matlab的函数句柄（function handle）符号“@”。

function handle实际上就是一个函数的指针，例如C++中的回调函数等等，它允许我们可以在一个函数中调用另一个函数。

语法为：   **变量名=@(输入参数列表)运算表达式**

这样产生的函数句柄变量不指向特定的函数, 而是一个函数表达式。

例如：

```matlab
y = @(x) x.^2*log(x);
integral(y, 2, 4); % 这里就调用了y，y是一个指针，指向x.^2*log(x)这个表达式
% integral函数后面两个参数是积分上下限
```

另外，二重积分和三重积分分别可以使用integral2()和integral3()

```matlab
f = @(x,y) x.*cos(y)+y.*sin(x);
integral(f, pi, 2*pi, 0, pi);
% 第二第三个参数是第一重积分的上下限，第四第五个参数是第二重积分的上下限。三重积分以此类推。

g = @(x,y,z) x.*cos(y)+z.*sin(x);
integral(g, 0, pi, 0, 1, -1, 1);
```

**此小节结束，你真厉害！**

![](https://s2.loli.net/2022/07/06/Kh4zpItmHrbkAPN.jpg)
