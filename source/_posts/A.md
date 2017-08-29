---
title: A*
date: 2017-08-28 23:02:45
tags:
---

写了一篇关于jump point search，想着不写一篇A*，似乎不对。

首先关于A*需要了解的几个概念：
	
 - manhattan   `曼哈顿距离，两个点在水平方向和垂直方向的距离，并且把这两个距离相加，结果就是曼哈顿值。`
 - start   `起始点`
 - end    `结束点`
 - G        `'起始点start'到'当前点'产生的移动消耗` 
 - H        `它只是计算出'当前点'到'结束点end'的曼哈顿距离值，当然，你也可以用其他算距离的方法，例如euclidean(欧几里得距离)，octile距离，chebyshev(切比雪夫距离)。` 
 - F    `F = G + H`
 - openlist   `待检索的列表,此列表需要根据元素的F值自动排序`
 - closelist   `不需要检索的列表`
 - openedlist   `已检索的列表`
 
## demo
 说完了这些，我们还是往常一样，来个demo。
 
![A*](http://img.blog.csdn.net/20170828212451395?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzgxODYxODE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

图中的黄线上的绿色点为起点，红色点为终点。
搜索过程：

 - 1、   `把起始点s压入openlist,并将起始点放进openedlist。`

 - 2、   `从openlist中弹出一个点，以这个点为当前节点。并把这个点放进closelist。`

 - 3、   `获得当前点附近可以走的所有点。算出这些点各自的F值，并把这些点放进一个openlist中,同时也需要放进openedlist。` 
 
 - 4、   `取出openlist中f指最小的点作为当前点，并且放进closelist,此处是图中的1点，接着再获取1点周围的附近可以走的，未在关闭列表的点，判断此邻点是否是结束点，如果是，那么Ok, 我们可以根据这些节点的父节点倒推出路径了，就结束啦；如果不是，那么需要判断此邻点是否已经在openedlist，如果是，那么就需要判断此邻点当前路径的G值是否小于已经保存的G值，是的话那么就需要更新在openlist中此邻点的G值，并且把此邻点的父节点设置为1点(就是我们当前所在的点)，如果不在openedlist，那么就需要把其加入openedlist，并加入openlist。` 
 
 - 5、   `经过第4步的更新，我们就可以继续重复2-4。跟着找到2,3,4直到找到结束点为止。`

好了，  A* 搜索的过程其实就这些，接下来，各位同学去找个实际的代码demo看下，加深下理解吧！go !