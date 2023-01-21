---
title: matlab基础 工程篇（五 回归与内插）
date: 2022-07-11 14:46:01
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

# matlab中的回归与内插
## 简单线性回归（simple linear regression）
首先想象，在一个坐标系中存在着一些离散的点，现在我们需要用一条线去拟合这些点，这就是简单线性回归。使用一次函数拟合是最为简单的情况。

{%asset_img 1.png%}
在平面中，存在i组点 $(x_i,y_i)$，现在我们要用一条直线 $\hat{y_i}=\beta_0+\beta_1 x$ 拟合这些点，此时我们就需要用到最小二乘法。

## 最小二乘法（ordinary least squares）
最小二乘法简单理解为每个离散点到拟合直线的距离的平方和最小，不过这个距离不是点到直线的距离，而是y方向的差值，即 $dis_i=y_i-\hat y_i$，那么现在我们的目标时找到这条直线，如何做呢？

首先，根据最小二乘法的原理，我们可以写出距离平方和（sum of squared errors, SSE）:
$$SSE=\sum_i dis_i^2=\sum_i(y_i-\hat y_i)^2$$

$$\hat y_i =\beta_0+\beta_1 x_i$$

$$\sum_i dis_i^2=\sum_i(y_i-\beta_0-\beta_1 x_i)^2$$

现在我们要使SSE最小，则需要求偏导。

$$\frac{\partial \sum_i dis_i^2}{\partial \beta_0} = -2\sum_i(y_i-\beta_0-\beta_1 x_i)=0$$

$$\frac{\partial \sum_i dis_i^2}{\partial \beta_1} = -2\sum_i(y_i-\beta_0-\beta_1 x_i)x_i=0$$

整理式子得：
$$\sum_{i=1}^N y_i=\beta_0 N+\beta_1 \sum_{i=1}^N x_i$$

$$\sum_{i=1}^N y_ix_i=\beta_0 \sum_{i=1}^N x_i+\beta_1 \sum_{i=1}^N x_i^2$$

写成矩阵形式：
$$	
\begin{bmatrix}
   \sum y_i \\
   \sum y_ix_i
\end{bmatrix}=
\begin{bmatrix}
   N & \sum x_i \\
   \sum x_i & \sum x_i^2
\end{bmatrix}
\begin{bmatrix}
   \beta_0 \\
   \beta_1
\end{bmatrix}
$$

由此，我们可以解出 $\beta_0 和\beta_1$

那么如果不是一次函数，按照同样的方法也可以解出对应的需要的参数。

### polyfit()
在matlab中，我们只需要提供点序列x的向量和y的向量，再使用polyfit函数就可以很快的拟合出需要的线了。

```matlab
x=[-1.2 -0.5 0.3 0.9 1.8 2.6 3.0 3.5];
y = [-15.6 -8.5 2.2 4.4 6.6 8.2 8.9 10.0];
fit=polyfit(x, y, 1)

x1=-2:0.1:4
y1=polyval(fit, x1)
figure
hold on
plot(x,y,'o');
plot(x1,y1)
hold off

```

{%asset_img 2.png%}

### corrcoef()
corrcoef函数用来判定这些散点是否有线性相关（Linearly correlated），返回值为一个大于-1小于1的常数k，如果k接近-1，则说明这些点呈负相关，如果k接近1，则说明它们正相关，如果接近0，则说明它们线性无关。

```matlab
x=[-1.2 -0.5 0.3 0.9 1.8 2.6 3.0 3.5];
y = [-15.6 -8.5 2.2 4.4 6.6 8.2 8.9 10.0];
corrcoef(x,y)
```

* corrcoef(x,y)返回的是一个二阶矩阵，左上角表示x和x的相关，右下角表示y和y的相关，所以肯定都是1，右上角和左下角表示的是x和y的相关以及y和x的相关。

用更高次的函数拟合，有时候效果会更好，但也有可能过拟合。
```matlab
x=[-1.2 -0.5 0.3 0.9 1.8 2.6 3.0 3.5];
y = [-15.6 -8.5 2.2 4.4 6.6 8.2 8.9 10.0];
x1=x(1):0.1:x(end);
set(gcf, 'position', [100 100 1500 400]);
for i=1:3
    subplot(1, 3, i); fit = polyfit(x, y, i);
    y1=polyval(fit, x1);
    plot(x, y, 'ro', x1, y1);
    set(gca, 'fontsize', 14);
    ylim([-17 11]);     legend('location', 'southeast', 'Data points', 'Fitted curve');
end
```
{%asset_img 3.png%}

### regress()
上面提到的polyfit仅适合拟合一元函数，如果我们要拟合多元函数怎么办？这时候就要用到regress().

先看一个matlab中内置的实例：

```matlab
load carsmall;
y = MPG;
x1=Weight; x2=Horsepower;
X = [ones(length(x1), 1), x1, x2];
b=regress(y, X);
x1fit=min(x1):100:max(x1);
x2fit=min(x2):10:max(x2);
[X1FIT, X2FIT]=meshgrid(x1fit, x2fit);
YFIT=b(1)+b(2)*X1FIT+b(3)*X2FIT;
scatter3(x1,x2,y, 'filled'); hold on
mesh(X1FIT,X2FIT,YFIT); hold off
xlabel('weight'); ylabel('horsepower'); zlabel('MPG');
rotate3d on;
```

{%asset_img 4.png%}

我们首先要按照 $y=a+bx+cx^2$ 的形式定义一个多项式，然后regress返回的是一个方程的系数，在这里就是一个平面方程，然后绘制出来的结果就如上图。

## 内插（interpolation）
内插的概念比较简单，主要看matlab中怎么应用吧。
### interp1()

```matlab
x=linspace(0, 2*pi, 40); x_m=x;
x_m([11:13, 28:30])=NaN; % 令部分点为空
y_m=sin(x_m);
plot(x_m, y_m, 'ro', 'markerfacecolor', 'r');
xlim([-0.5 2*pi+0.5]); ylim([-1.2 1.2]); box on;
set(gca, 'fontsize', 16);
set(gca, 'xtick', 0:pi/2:2*pi);
set(gca, 'xticklabel', {'0', '\pi/2', '\pi', '3\pi/2', '2\pi'});

m_i=~isnan(x_m); % 找到非空的点
y_i =interp1(x_m(m_i), y_m(m_i), x); % 对所有非空的点进行内插，且做内插的范围是原本的x的范围
hold on
plot(x, y_i, '-b', 'linewidth', 2);
hold off
```

{%asset_img 5.png%}

### spline()
如果要让内插变得光滑，那么可以使用spline方法。只需要把interp1改成spline.

```matlab
m_i=~isnan(x_m);
y_i=interp1(x_m(m_i), y_m(m_i), x);
y_s=spline(x_m(m_i), y_m(m_i), x);
hold on
plot(x, y_i, '-b', x, y_s, '--m', 'linewidth', 2);
hold off
h=legend('original', 'linear', 'spline');
```

{%asset_img 6.png%}

### pchip()
pchip是一种分段三次差值法，其原理来自于[**埃尔米特插值法**](https://blog.csdn.net/HG0724/article/details/121624725?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165759582116782391831331%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165759582116782391831331&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-2-121624725-null-null.142^v32^pc_rank_34,185^v2^control&utm_term=%E5%9F%83%E5%B0%94%E7%B1%B3%E7%89%B9%E6%8F%92%E5%80%BC&spm=1018.2226.3001.4187)。

既然我们能做到不光滑和光滑的差值了，还要别的方法干嘛，先来看一个实例：

```matlab
x = -3:3; y = [-1 -1 -1 0 1 1 1]; t=-3:0.01:3;
y_s = spline(x, y, t);
y_p = pchip(x, y, t);
hold on
plot(x, y, 'ro', 'markerfacecolor', 'r');
plot(t, y_s, 'm-', 'linewidth', 2);
plot(t, y_p, 'b--', 'linewidth', 2);
hold off
legend('location', 'southeast', 'origin', 'spline', 'pichip');
```

{%asset_img 7.png%}

不难发现，在这一个例子中，使用spline方法会让两边差值的形状产生波动，这就是**龙格效应（rouge phenomenon）**——有些函数在差值的时候，差值次数越高，边缘区域就越不准确，波动越大。至于龙格效应出现的原理，我也还没弄明白，在这里插个眼，以后明白了会补充说明。

那么使用pchip方法就可以有效避免这个问题，因为**埃尔米特插值法的一大优势就是为了解决龙格效应**，至于怎么就解决了，我也没弄明白，啊哈哈哈哈（尴尬又不失礼貌的笑容）。

### interp2()
当然，对于二维平面，matlab同样可以内插。

下面看一个实例，用法与以为相差不多。

```matlab
xx = -2:0.5:3; yy =xx;
[X Y] = meshgrid(xx, yy);
Z = X.*exp(-X.^2-Y.^2);
surf(X, Y, Z);

hold on
plot3(X, Y, Z+0.01, 'ro', 'markerfacecolor', 'r');

xx_i = -2:0.1:3; yy_i = xx_i;
[X_i, Y_i] = meshgrid(xx_i, yy_i);
Z_i = interp2(xx, yy, Z, X_i, Y_i);
surf(X_i, Y_i, Z_i);
% plot3(X_i, Y_i, Z_i+0.01, 'bx');
hold off
rotate3d on
```

{%asset_img 8.png%}
如果要用spline光滑的内插，那么就要在interp2最后的参数写上'cubic'：

```matlab
Z_i = interp2(xx, yy, Z, X_i, Y_i, 'cubic');
```

{%asset_img 9.png%}

* 为表现其光滑程度，不绘制差值点。

**此篇章结束，你真厉害！**

![](https://s2.loli.net/2022/07/06/Kh4zpItmHrbkAPN.jpg)

