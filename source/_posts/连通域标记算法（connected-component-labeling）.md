---
title: 连通域标记算法（connected-component labeling）
date: 2022-07-23 13:17:42
tags: [matlab, 教程向]
categories: matlab基础
cover: https://s2.loli.net/2022/07/06/LmOeGJ7352pfnAi.png
katex: true
---

在之前的[matlab基础 影像处理篇（二）](https://yigexiaogai.github.io/2022/07/21/matlab%E5%9F%BA%E7%A1%80-%E5%BD%B1%E5%83%8F%E5%A4%84%E7%90%86%E7%AF%87%EF%BC%88%E4%BA%8C%EF%BC%89/)小节中，我们介绍了matlab中的连通域标记方法bwlabel，今天我们要根据算法的原理，自己写代码实现连通域标记。

本博客主要参考了 **[OpenCV_连通区域分析](https://blog.csdn.net/icvpr/article/details/10259577)** 这一篇CSDN上的博客，该博客通过opencv实现，我将算法改写成matlab的语法，且该博客中的第二种方法的代码有比较严重的问题，接下来我也会说明。

# 摘要
本文主要介绍在CVPR和图像处理领域中较为常用的一种图像区域（Blob）提取的方法——连通性分析法（连通区域标记法）。文中介绍了两种常见的连通性分析的算法：1）两步法（Two-pass）；2）种子填充法（Seed-Filling）

# 连通区域分析的算法
从连通区域的定义可以知道，一个连通区域是由具有相同像素值的相邻像素组成像素集合，因此，我们就可以通过这两个条件在图像中寻找连通区域，对于找到的每个连通区域，我们赋予其一个唯一的标识（Label），以区别其他连通区域。

连通区域分析有基本的算法，也有其改进算法，本文介绍其中的两种常见算法：
1）两步法（Two-pass）；2）种子填充法（Seed-Filling）

>note
>>a、这里的扫描指的是按行或按列访问以便图像的所有像素，本文算法采用的是按行扫描方式；
>>b、图像记为B，为二值图像：前景像素（pixel value = 1），背景像素（pixel value = 0）
>>c、label专门存储在label_map中，label从1开始计数；
>>d、像素相邻关系：4-领域、8-领域，本文算法采用4-邻域；

<table><tr>
<td><img src=1.png title='4邻域' width=50%></td>
<td><img src=2.png title='8邻域' width=50%></td>
</tr></table>


# 两步法（Two-pass）
两遍扫描法，正如其名，指的就是通过扫描两遍图像，就可以将图像中存在的所有连通区域找出并标记。思路：第一遍扫描时赋予每个像素位置一个label，扫描过程中同一个连通区域内的像素集合中可能会被赋予一个或多个不同label，因此需要将这些属于同一个连通区域但具有不同值的label合并，也就是记录它们之间的相等关系；第二遍扫描就是将具有相等关系的equal_labels所标记的像素归为一个连通区域并赋予一个相同的label（通常这个label是equal_labels中的最小值）。

## 步骤
> 1.第一次扫描
> 访问当前像素B(x,y)，如果B(x,y) == 1：
>> a. 如果B(x,y)的领域中像素值都为0，则赋予B(x,y)一个新的label：
label += 1， B(x,y) = label；

>> b.如果B(x,y)的领域中有像素值 > 1的像素Neighbors：
>>> (1) 将Neighbors中的最小值赋予给B(x,y):
B(x,y) = min{Neighbors}

>>> (2)记录Neighbors中各个值（label）之间的相等关系，即这些值（label）同属同一个连通区域；
labelSet[i] = { label_m, .., label_n }，labelSet[i]中的所有label都属于同一个连通区域（注：这里可以有多种实现方式，只要能够记录这些具有相等关系的label之间的关系即可）

> 2.第二次扫描
> 访问当前像素B(x,y)，如果B(x,y) > 1：
>> a、找到与label = B(x,y)同属相等关系的一个最小label值，赋予给B(x,y)；

>完成扫描后，图像中具有相同label值的像素就组成了同一个连通区域。

算法过程如动图所示：

{%asset_img 3.gif%}

## 算法
```matlab
function [map_label_second, count] = two_pass( image ) % 作为函数，传入的是二值化图片image
% 参考：https://blog.csdn.net/gaohanggaolegao/article/details/104911272

%% 初始化准备
[row, col] = size(image);

map_label_first = zeros(size(image)); % first pass
map_label_second = zeros(size(image)); % second pass

label_ID = 1;
label_set = [];

%% First pass for temporary labels.
for i = 1:row
    for j = 1:col
        tmp_neighbors = []; % 初始化邻域集
        
        if(image(i, j)==1) % foreground
            % 将上和左的label_id加进neighbor中
            if(i-1>0 && image(i-1, j)==1 && map_label_first(i-1, j)~=0) % up
                tmp_neighbors(1, length(tmp_neighbors)+1) = map_label_first(i-1,j);
            end
            
            if(j-1>0 && 1==image(i,j-1) && map_label_first(i,j-1)~=0) % left
                tmp_neighbors(1, length(tmp_neighbors)+1) = map_label_first(i,j-1);
            end
            
            label_neighbors = unique(tmp_neighbors); % 去重
            
            if (isempty(label_neighbors)==true) % 加入一个新的id
                map_label_first(i,j) = label_ID;
                label_set(1, label_ID) = label_ID; % 加入到label_set中
                label_ID = label_ID + 1;
            else % Update label_set
                map_label_first(i,j) = min(label_neighbors); % 取领域中的最小值
                
                if length(label_neighbors)~=1
                    label_set(1, max(label_neighbors)) = min(label_neighbors); % 把小值作为大值的指针，构成连接关系
                end
                
            end   
        end
        
    end
end

%% Second pass for merging labels.
for i = 1:1:row
    for j = 1:1:col
        if(map_label_first(i,j)~=0) % 像素点有连通域
            map_label_second(i,j) = label_set(1, map_label_first(i,j)); % 给定第一个指针后的初值
            while(label_set(1,map_label_second(i, j))~=map_label_second(i, j)) % 如果没有递归结束就继续进行递归
                map_label_second(i,j) = label_set(1, map_label_second(i,j));
            end
        end
    end
end

%% 统计个数
count=0;
start = 0;
for i = 1:row
    for j = 1:col
        if map_label_second(i,j)>start
            count = count+1;
            start = map_label_second(i,j);
        end
    end
end

end
```

## 注意点
**有两点需要注意：**

第一点是当领域集的长度为1的时候，说明这个像素点上方和左方都是相同的像素，这个图形很有可能是一个往下凸的形状，这个时候应该规避更新label_set,防止本来构建的链表被错误更新：
```matlab
if length(label_neighbors)~=1
    label_set(1, max(label_neighbors)) = min(label_neighbors); % 把小值作为大值的指针，构成连接关系
end
```

第二点是链表可能不止一层的指向，在没有指到底的时候，必须让它递归，指到最后
```matlab
if(map_label_first(i,j)~=0) % 像素点有连通域
    map_label_second(i,j) = label_set(1, map_label_first(i,j)); % 给定第一个指针后的初值
    while(label_set(1,map_label_second(i, j))~=map_label_second(i, j)) % 如果没有递归结束就继续进行递归
        map_label_second(i,j) = label_set(1, map_label_second(i,j));
    end
end
```

# 种子填充法（Seed-Filling）
种子填充方法来源于计算机图形学，常用于对某个图形进行填充。思路：选取一个前景像素点作为种子，然后根据连通区域的两个基本条件（像素值相同、位置相邻）将与种子相邻的前景像素合并到同一个像素集合中，最后得到的该像素集合则为一个连通区域。

## 步骤
> 1.扫描图像，直到当前像素点B(x,y) == 1：
>> a、将B(x,y)作为种子（像素位置），并赋予其一个label，然后将该种子相邻的所有前景像素都压入栈中；
b、弹出栈顶像素，赋予其相同的label，然后再将与该栈顶像素相邻的所有前景像素都压入栈中；
c、重复b步骤，直到栈为空；
此时，便找到了图像B中的一个连通区域，该区域内的像素值被标记为label；

> 2.重复第（1）步，直到扫描结束；
> 扫描结束后，就可以得到图像B中所有的连通区域；

算法过程如动图所示：

{%asset_img 4.gif%}

## 算法
```matlab
function [map_label, label_ID] = seed_filling( image )

%% 初始化准备
[row, col] = size(image);
label_ID = 0; 
map_label = zeros(size(image)); 

%% 
for i = 1:row
    for j = 1:col
        if image(i, j)==1
            neighborPixels ={}; % 初始化为空cell, 其中放的是坐标
            neighborPixels{end+1}=[i ,j];
            label_ID = label_ID+1; % 开始一个新的label
            
            while(~isempty(neighborPixels))
                % 获取label的值并赋给相应坐标
                curPixel=neighborPixels{end}; % 邻域最后那个坐标
                curX = curPixel(1);
                curY = curPixel(2);
                
                map_label(curX, curY)=label_ID;
                
                % 消去已经赋值的坐标
                neighborPixels(end)=[]; % 最后的坐标置空
                image(curX, curY)=0; % 把已经解决的像素置为0，不然会死循环
                
                % push四邻域
                if (curX-1>0 && image(curX-1,curY)==1) % up
                    neighborPixels{end+1}=[curX-1, curY];
                end
                if (curX+1<=row && image(curX+1,curY)==1) % down
                    neighborPixels{end+1}=[curX+1, curY];
                end
                if (curY-1>0 && image(curX,curY-1)==1) % left
                    neighborPixels{end+1}=[curX, curY-1];
                end
                if (curY+1<=col && image(curX,curY+1)==1) % right
                    neighborPixels{end+1}=[curX, curY+1];
                end
                
            end
        end
    end
end

end
```

## 注意点
**有一点需要注意:**

一定要把已经赋值结束的坐标的像素置为0，不然由于循环到下一个领域时，又会回到上一个位置，造成死循环。
```matlab
% 消去已经赋值的坐标
neighborPixels(end)=[]; % 最后的坐标置空
image(curX, curY)=0; % 把已经解决的像素置为0，不然会死循环
```
