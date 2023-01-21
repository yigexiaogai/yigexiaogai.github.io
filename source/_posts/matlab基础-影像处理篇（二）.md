---
title: matlab基础 影像处理篇（二）
date: 2022-07-21 13:58:37
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

# matlab中的几种图像处理方法
本节我们将接触几种图像处理方法：
* 图像二值化（image thresholding）
* 背景估计（background estimation）
* 连通域标记（connected-component labeling）

## 问题背景
我们看下面这张拍摄了大米的图片：

{%asset_img rice.png%}

现在我们要数出里面有几颗米粒，并且测量其中米粒的大小。

作为人来说，我们可以一颗一颗数，手动测量米粒的长度，但是机器首先得识别出什么是米粒，再进行计数，这要如何做呢？实际上，识别米粒的过程就可以称作简单的计算机视觉的工作。

## 策略
在清楚了任务目标后，我们就需要制定一系列的策略来解决这个问题，我们可以分成以下步骤：
1. 将图像二值化。直接将米的部分变成白色，背景变成黑色，这样就容易分开主体与背景了。
2. 统计图片中，白色像素联通的部分，有几个部分，就有几颗大米。

## image thresholding
首先先要将图像二值化，所以我们需要知道图像的直方图，然后根据直方图来判断，以灰度值多少为界来将图像二值化。

```matlab
%%
img  = imread('test image/rice.png');
img = rgb2gray(img);% 转换成灰度图，不然imhist会报错

figure
subplot(1,2,1); imshow(img); title('rice');
subplot(1,2,2); imhist(img); title('hist of rice');
```

{%asset_img 1.png%}

```matlab
%%
level = graythresh(img); % 根据灰度分布来找到一个合适的阈值
bw = im2bw(img, level); % 根据阈值来区分主体与背景
subplot(1,2,1); imshow(img);
subplot(1,2,2); imshow(bw);
```

{%asset_img 2.png%}

经过上面的代码我们就初步将大米与背景分开了，这里有一个可以了解一下的问题，就是这个graythresh的工作原理，我暂时也没搞懂，以后弄懂会写一篇博客。总之在这里，graythresh可以智能地获得阈值。

但是看图我们会发现两个问题，1. 在图片下部的一些大米好像丢失了；2. 在图片中间还存在着一些白色的小点，这些噪声可能会影响到之后的识别。

问题1的原因是，图片本身的背景就是中间比较亮，下方比较暗，比较暗的大米可能被划分成背景了。问题2的原因是，图片中存在一些小亮点噪声。

我们依次解决这两个问题。

## background estimation
解决两个问题的办法就是让matlab识别出图片的背景，然后让图片减去背景就可以了。这里的原理比较复杂，总之就是使用imopen函数和strel方法就可以智能得到背景。

```matlab
%%
BG = imopen(img, strel('disk', 15));
figure
imshow(BG); title('background');
```

{%asset_img 3.png%}

### imsubtract()
使用imsubtract则可以使两张图片相减。

```matlab
%%
img2=imsubtract(img, BG);
subplot(1,3,1); imshow(img); title('rice');
subplot(1,3,2); imshow(BG); title('background');
subplot(1,3,3); imshow(img2); title('rice-background');
```

{%asset_img 4.png%}

重新进行threshold:

```matlab
level  = graythresh(img2);
bw2 = im2bw(img2, level);

subplot(2,2,1); imshow(img);
subplot(2,2,2); imshow(bw);
subplot(2,2,3); imshow(img2);
subplot(2,2,4); imshow(bw2);
```

{%asset_img 5.png%}

于是我们得到了一张比较完美的二值图像，接下来就是要计数了。

## connected-component labeling
连通域标记算法比较复杂，需要一些图例才能说明，在这里推荐大家仔细看一下课程内容，老师讲的很清楚:[影像处理（二）](https://www.bilibili.com/video/BV1GJ41137UH?p=9&vd_source=3d73f51879206e4a4df14c2d1fb027e7)。课程中老师介绍的方法是种子填充（seed-filling）算法，但实际上，常用的连通域标记算法还有两步法（two-pass）、一步法（两步法的改进）等等。**具体的算法将在另一篇博客中介绍：[连通域标记算法（connected-component labeling）](https://yigexiaogai.github.io/2022/07/23/%E8%BF%9E%E9%80%9A%E5%9F%9F%E6%A0%87%E8%AE%B0%E7%AE%97%E6%B3%95%EF%BC%88connected-component-labeling%EF%BC%89/)**

在matlab中，其为我们内置了连通域标记方法bwlabel，但是看名字显然，这个方法只对黑白二值图有效。

```matlab
%%
[labeled, numObjects]=bwlabel(bw2, 8);
% labeled是标记图，numObjects是图中被标记的个数
RGB_label = label2rgb(labeled); imshow(RGB_label);  % 使用rgb颜色图来标注连通域，使其可视化
```

{%asset_img 6.png%}

## 后续处理和优化
接下来我们找到这些大米中最大的那颗以及它们的平均值:

```matlab
%% 找到哪颗米最大，以及它们的平均大小
rice_size=zeros(1, numObjects);
for i = 1:size(labeled, 1)
    for j = 1:size(labeled, 2)
        if labeled(i ,j)~=0
            rice_size(labeled(i, j))=rice_size(labeled(i, j))+1;
        end
    end
end

max_rice=max(rice_size);
mean_rice = mean(rice_size);
```

其实本来的计数会存在一些问题，比如图片中仍然可能存在一些噪声，所以会存在一些特别小的“米粒”，我们需要把这些去除；而如果有两颗米粒连在一起，可能会造成一颗特别大的米粒，这时候我们就需要把这个米粒算成两颗。

```matlab
%% 绘制米大小的直方图，步长为50
hist_length=round((max_rice+200)/50);
rice_hist=zeros(1, hist_length);

for i = 1:size(rice_size, 2)
    pointer = floor(rice_size(i)/50)+1;
    rice_hist(pointer)=rice_hist(pointer)+1;
end

bar(rice_hist); title('米粒大小直方图');
set(gca, 'xtick', 1:1:hist_length);
```

{%asset_img 7.png%}

```matlab
%% 优化计数
new_count=numObjects;
for i =1:hist_length
    if i<=3
        new_count = new_count -rice_hist(i);
    elseif i>=27 % 阈值可以自己设定
        new_count = new_count + rice_hist(i);
    end
end
```

matlab也自带了归纳连通域属性的方法regionprops()

```matlab
graindata = regionprops(labeled, 'basic');
graindata(51) % 第51个连通域的属性
```

还可以使用bwselect()，它是一个交互的方法，点击可以显示需要的连通域，很神奇：

```matlab
%%
objI = bwselect(bw2); imshow(objI);
```

{%asset_img 8.png%}

**此小节结束，你真厉害！**

![](https://s2.loli.net/2022/07/06/Kh4zpItmHrbkAPN.jpg)