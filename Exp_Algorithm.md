<!-- mathjax config similar to math.stackexchange -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    TeX: { equationNumbers: { autoNumber: "AMS" } },
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
        inlineMath: [ ['$', '$'] ],
        displayMath: [ ['$$', '$$']],
        processEscapes: true,
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
    },
    messageStyle: "none",
    "HTML-CSS": { 
        preferredFont: "TeX",
        availableFonts: ["STIX","TeX"],
        linebreaks: { automatic: true } 
    },
    SVG: { linebreaks: { automatic: true } }
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"> </script>


[TOC]

# 图的层次聚类算法

## 一、 基础说明

### 1、层次聚类

- **本算法中，采用自上而下的方法**


- 自上而下(*Divisive*)：初始时，所有点在同一个簇内。在图中采取 **拆边** 的方式，对图进行进行聚类。

- 自下而上(*Agglomerative*)：初始时，每个点均为一个簇。在图中采取 **加边** 的方式，对图进行聚类。


###  2、图

- 图$G=(V, E)$，$V$ 为图中顶点集，$E$ 为图中边集。

- 点(*Vertex*)： 为生产所用的零件。

- 边(*Edge*)：零件与零件之前若有依赖、提供关系，则两个点之间有一条无向边，且边无权值。

- 间隔(*Betweenness*)：仅在聚类算法中所使用，衡量两个点的紧密程度。

- 权重(*Weight*)：仅在聚类算法中所使用，衡量从源点到每个点的路径数。

- 距离(*Distance*)：仅存聚类算法中所使用，辅助计算权重。

### 3、模块化程度

- $Q$：模块化程度的衡量值，$Q\leq1.0$

- $$
  Q = \sum_i^k {\left( e_{ii} - a_i^2 \right)} = Tr {\bf e} - 	    
  	\begin{Vmatrix} 
  		{\bf e^2} 
  	\end{Vmatrix}
  	\label{eq:1}
  $$

- ${\bf e}$：$k \times k$ 大小的矩阵，其中 $k$ 为模块的个数。

- ${\bf e}_{ij}$：模块 $i$ 中所有与模块 $j$ 相连的点对的间隔值总和。

- $a_i$：$a_i=\sum _{j} ^{k} {e _{ij}}$，与模块 $i$ 所连接的所有点对间隔总和。

- $Tr{\bf e}$：$Tr{\bf e} = \sum_i^k{e_{ii}}$ ，模块 $i$ 中所有彼此相连的点对的间隔总和。

- $\begin{Vmatrix} {\bf x} \end{Vmatrix}$ ：对矩阵内所有的元素进行求和。

## 二、 算法说明

### 1、算法总览

- Step.1 输入图 $G=(V, E)$ ，其中 $\begin{vmatrix} V \end{vmatrix} = n$， $\begin{vmatrix} E \end{vmatrix} = m$ 。

- Step.2 将 $G$ 中每一个顶点分别作为源点，使用最短路径的方法，求出每个边的间隔值，此时每个边有 $n \times m$ 个间隔值。

- Step.3 对每个边，将上步所求得的间隔值进行求和。在图 $G$ 中删除求和后间隔值最大的边，同时更新图 $G$。

- Step.4 使用公式 $\eqref{eq:1}$ 计算当前图 $G$ 的模块化程度值 $Q$。

- Step.5 重复 Step.2，Step.3，Step.4 直接满足结束条件或 $E = \emptyset$ 。

### 2、算法详述

- Step.1 输入图 $G=(V, E)$ ，其中 $\begin{vmatrix} V \end{vmatrix} = n$， $\begin{vmatrix} E \end{vmatrix} = m$ 。

- Step.2 选定一个点为源点，并建立一个“树形结构”，源点所在为第 $0$ 层，源点的孩子节点为第 $1$ 层，其余节点以此类推。计算各点权重方法(*Wight*)如下：
  - Step.2.1 初始化源点 $s$ 的权重 $w_s = 1$ ，距离 $d_s = 0$。

  - Step.2.2 对于每一个与 $s$ 相邻接的点 $i$，有：
    $$
    \begin{cases}
     	 d_i = d_s + 1 \\ w_i = w_s
    \end{cases}
    \label{eq:2}
    $$

  - Step.2.3 对于每一个与 Step.2.2 中点 $i$ 所相邻的点 $j$ ，有：
    - 如果点 $j$ 没有被赋予一个距离值， 则使用公式 $\eqref{eq:2}$ 进行赋值。

    - 如果点 $j$ 已经被赋予一个距离值，则有：

  $$
  \begin{cases}
    \text{Do Nothing} &  if  \; d_j < d_i + 1 \\
    w_j = w_i + w_j	& 	if \; d_j = d_i + 1
  \end{cases}
  \label{eq:3}
  $$

- Step.3 计算各边间隔(*Betweenness*)
  - Step.3.1 对于每一个叶节点 $p$ 所相邻的节点 $i$，两个点的间隔值为 $\frac{w_i}{w_p}$。
  - Step.3.2 自下向上计算各点对间隔值。对于节点 $i$ 的父节点 $j$，二者的间隔值为：
    $$
    \frac{w_j}{w_i} \times \left( 1 + \sum_i^c {v_i} \right) \\
    c: 节点 i 的孩子个数 \\
    v_i: 节点 i 与孩子节点之间的间隔值
    \label{eq:4}
    $$

- Step.4 重复执行Step.2，Step.3，保证所有的点都成为过一次源点。

- Step.5 对每个边，将上步所求得的间隔值进行求和。在图 $G$ 中删除求和后间隔值最大的边，同时更新图 $G$ 并使用公式 $\eqref{eq:1}$ 计算当前图 $G$ 的模块化程度值 $Q$。

- Step.6 重复 Step.4，Step.5 直接满足结束条件或 $E = \emptyset$ 。


