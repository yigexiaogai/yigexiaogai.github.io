---
title: matlab基础 工程篇（二 方程式求根）
date: 2022-07-06 16:32:33
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

# 使用matlab对方程式求根
matlab中解决方程式的根的问题主要可以通过两个方法：
* 设未知数法（symbolic approach）
* 数值求根工具（numeric root solvers）]

两个方法针对不同的情况，不存在孰优孰劣。下面我们开始逐一介绍。

## symbolic approach
刚知道这点的时候我真的感到相见恨晚，matlab居然可以直接设未知数，然后求解方程，甚至可以计算微积分！我要是早点知道的话……那高数作业岂不是起飞！好了，说回正题，**matlab中需要使用sym或者syms定义未知数，此时定义的未知数是一种类型为sym的特殊变量**。如下：

```matlab
x = sym('x'); % 字符与赋值对象最好一致，不然容易乱，除非你记得住

syms r t p q; % syms可以一次性设多个未知数，因此更加常用

A = sym('a', 3); % 不过如果要一次性设一个矩阵中所有的未知数，还是得用sym()
% 这里是设了3*3的未知数给矩阵A。
```

然后我们就可以对未知数进行各种运算：

```matlab
syms x;
y = x+x+x; % 此时创立的y也是sym类型变量，且其值为3*x
z = x^2-2*x+1;
```

### solve()
solve方法是用来解出f(x)=0的根的一个函数，用法如下：

```matlab
syms x;
y = x*sin(x)-x;
solve(y,x); 
% 第一个参数可以理解成一个句柄（虽然这里是一个未知数变量）
% 第二个参数就是要解出的未知数
% 解决的问题是 y=0 的解
```

* solve虽然返回的可能是具体的值，但是其仍然是一个未知数型变量。

对于一元二次方程同样可以解决。

```matlab
syms x y;
eq1=sin(x)+2*cos(y);
eq2=2*x-y;
r=solve(eq1, eq2, x, y);
r.x % 可以看到具体的值
r.y
```

对于通式也可以解，即用字母表示x的根：

```matlab
syms a b x;
eq = a*x^2+b;
solve(eq); % 第二个参数表示要求的未知数，该方法会默认去找ascii最大的未知数作为第二个参数
solve(eq, x); % 与上相同
solve(eq, a); % 用x,b表示a
```

一些奇奇怪怪的未知数用法也可以用，比如矩阵：

```matlab
syms a b c d;
A=[a b; c d];
inv(A)
```

比如求导数：

```matlab
syms x;
y = 5*x^4;
diff(y) % 这里的diff对一个连续函数使用可以进行近似求导。
```

比如求积分：

```matlab
syms x;
y = 5*x^4;
z = int(y); % int方法求积分
z=z-subs(z,x,0) % 减掉matlab为我们默认给的常数项
```

* 因为求积分会多出一个常数项C，matlab会自动给C赋值，如果我们要剔除常数项，可以让x=0带入z中，就可以得到C的具体值了，然后z本身减去它即可

## numeric root solvers
对于一些复杂的[超越方程](https://baike.baidu.com/item/%E8%B6%85%E8%B6%8A%E6%96%B9%E7%A8%8B)，solve方法无法无法解出精确的值，这个时候我们就需要用到数值求根工具。
### fsolve()
使用fsolve可以利用近似法求得解的近似值。使用该方法需要用到function handle。例如：

```matlab
y =@(x) 1.2*x+1+x*sin(x);
fsolve(y,0.1) % fsolve会从第二个参数作为起始点开始寻找根
```

**对于二元的例题：**
找到以下方程组的解：
$$
f(x,y)= \begin{cases}
   2x-y-e^{-x}  \\
   -x+2y-e^{-y} 
\end{cases}
$$
使用初始(x,y)值为(-5,-5).

```matlab
function F=root2d(m)

F(1)=2*m(1)-m(2)-exp(-m(1));
F(2)=-m(1)+2*m(2)-exp(-m(2));

% 上面代码保存为.m文件

fun=@root2d;
m0=[-5,-5];
m = fsolve(fun, m0);
x = m(1); y=m(2);
```

### fzero()
fzero方法与fsolve方法很类似，但是如果函数和x相切，只有一个交点的时候，fzero方法无法求出该点，这与fzero和fsolve本身函数的算法有关。例如以下代码无法达到目的：

```matlab
f=@(x) x.^2;
fzero(f, 0.1) % 此时ans=NaN
```

我们可以设定方法在寻找根时候的精确度：

```matlab
f=@(x) x.^2;
options = optimset('MaxIter', 1e3, 'TolFun', 1e-10); % 第二个参数是迭代次数
% 第四个参数是最小精度 
fsolve(f, 0.1, options)
fzero(f, 0.1, options) % 还是解不出，因为原函数和x轴相切的关系
```

### roots()
roots方法也是求f(x)=0时的根，但是要注意，此时f(x)只能是多项式，其他超越函数不可以。

$$ f(x)= 1.2x^3+3.26x^2+8.9876x+0.6877 $$

```matlab
roots([1.2 3.26 8.9876 0.6877])
```

## 寻点策略
通常找寻零点的策略分为两种：
* 二分法（bisection method）
* 牛顿-拉弗森算法（Newton-Raphson method）

### bisection method
二分法非常的简单。

二分法是一种求解方程 $f(x)=0$ 的解的一种方法。
假设函数 $f(x)$ 在区间 $[a,b]$ 上连续，并且 $f(a) \times f(b) <0$ ,此时就可以用二分法求解。

求解伪代码：

1.a1 = a; b1 = b;
2.计算中点 $p_1=\frac{a_1+b_1}{2}$
3.如果 $f(p_1)=0$ ,那么方程的解为 $x=p_1$ ，终止
4.如果 $f(p_1) \not = 0 $
如果 $f(p_1)\times f(a_1)>0 $ ，$a_1=p_1; b_1=b_1$
如果 $f(p_1)\times f(b_1)>0 $ ，$a_1=a_1; b_1=p_1$
重复上述步骤2到4，直到满足误差，停止迭代。

> 有一点需要注意，如果函数在 $[a,b]$ 中间出现2n+1(n>=1)个点时，我们使用二分法不能把所有的根都找到，我们首先需要判断函数在某个区间上的单调性，再确定寻根的范围。

### Newton-Raphson method
牛顿-拉弗森算法不难理解，但原理实际上还是有点复杂的，具体可以参看CSDN的[这篇博客](https://blog.csdn.net/zhangphil/article/details/78913243?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165733277116781435420046%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165733277116781435420046&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-3-78913243-null-null.142^v32^pc_rank_34,185^v2^control&utm_term=Newton-Raphson%20method&spm=1018.2226.3001.4187)，其讲解的相对清楚。

简而言之就是利用函数的切线确定其与x轴的交点，再根据交点确定新的切线，不断迭代，切线与x轴的交点将会越来越接近零点。但和二分法一样，其有以下两个缺点：
* 迭代可能无法收敛。
* 如果初始点定的太远，可能最终会找到别的零点。

**此小节结束，你真厉害！**

![](https://s2.loli.net/2022/07/06/Kh4zpItmHrbkAPN.jpg)
