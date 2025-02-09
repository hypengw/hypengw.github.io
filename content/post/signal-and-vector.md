+++
title = '信号与向量'
date = 2025-02-02T23:06:46+08:00
draft = false
math = true
tags = ['数学']

+++

最近在看信号处理相关的书，看着看着，发现怎么看怎么像线性代数，于是记录下我能想到的对比。  
不得不说，这种可以把知识连成网的体验真不错，学起来事半功倍。  
> 博主不是相关方向的研究生，如果有书籍欢迎推荐给博主。  
这篇文章偏向一些有限维的直观理解，不打算引入更为抽象的无限维(主要是博主理解不能)。

## 信号的向量表示

把某个离散信号 $x[n],\ n \in [1,m]$ 看为 $m$ 维的向量 $\vec{v_x}$  
$$
\vec{v_x} \rightarrow (x[1], x[2], x[3], .., x[m])
$$

### 单位脉冲和基向量
**单位脉冲(unit impulse)**  
$$
\delta[n] = 
\begin{cases} 
0, & n \ne 0 \\ 
1, & n = 0
\end{cases}
$$
**基(basis)**  
A basis of $V$ is a list of vectors in $V$ that is linearly independent and
spans $V$ . For example,
$$
(1, 0, . . . , 0), (0, 1, 0, . . . , 0), . . . , (0, . . . , 0, 1)
$$

## 线性系统与线性变换
**线性系统(linear system)**  
令 $y_1(t)$ 是一个连续时间系统对输入 $x_1(t)$ 的响应，而 $y_2(t)$ 是对应于输入 $x_2(t)$ 的输出，那么一个线性系统有:  

- **可加性(additivity)**  
$$
y_1(t) + y_2(t)\ 是对\ x_1(t) + x_2(t)\ 的响应
$$
- **齐次性(homogeneity)**  
$$
ay_1(t)\ 是对\ ax_1(t)\ 的响应，此处 a 为任意复常数
$$

**线性变换(linear map)**  
A linear map from $V$ to $W$ is a function $T : V → W$ with the following
properties:  

- **可加性(additivity)**  
$$
T (u + v) = T u + T v,\ for\ all\ u, v ∈ V 
$$
- **齐次性(homogeneity)**  
$$
T (av) = a(T v),\ for\ all\ a ∈ F\ and\ all\ v ∈ V
$$

## 卷积和与矩阵乘法
### 卷积和(convolution sum)   
对于信号$x[n]$，可以把其表示为由一个序列和一串位移单位脉冲的线性组合
$$
x[n] = \sum_{k=-\infty}^{+\infty} x[k] \delta[n-k]
$$
$h_k[n]$ 对应位移单位脉冲 $\delta[n-k]$ 的响应，$y[n]$ 对应 $x[n]$ 的响应，由于线性系统的性质，线性组合的叠加有
$$
y[n] = \sum_{k=-\infty}^{+\infty} x[k]h_k[n]
$$
如果系统同时是**时不变 (time invariant)** 的，则有
$$
y[n] = \sum_{k=-\infty}^{+\infty} x[k] h[n-k]
$$

$$
h_k[n] \rightarrow h[n-k]
$$

### 矩阵乘法(matrix multiplication)

$$
\begin{bmatrix}
a_{11} & a_{12} & \cdots & a_{1n} \\
a_{21} & a_{22} & \cdots & a_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
a_{m1} & a_{m2} & \cdots & a_{mn}
\end{bmatrix}
\cdot
\begin{bmatrix}
x_1 \\
x_2 \\
\vdots \\
x_m
\end{bmatrix}=
\begin{bmatrix}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1m}x_m \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2m}x_m \\
\vdots \\
a_{n1}x_1 + a_{n2}x_2 + \cdots + a_{nm}x_m
\end{bmatrix}
$$

### 卷积和的矩阵表示

我们假设离散信号的范围是 $n \in [1, m]$ ，且为时不变  
$$
\vec{y[n]} =
\begin{bmatrix}
x[1]h_1[1] + x[2]h_2[1] + \cdots + x[m]h_m[1] \\
x[1]h_1[2] + x[2]h_2[2] + \cdots + x[m]h_m[2] \\
\vdots \\
x[1]h_1[m] + x[2]h_2[m] + \cdots + x[m]h_m[m] \\
\end{bmatrix} \\ \\ = 
\begin{bmatrix}
h_1[1] & h_2[1] & \cdots & h_m[1] \\
h_1[2] & h_2[2] & \cdots & h_m[2] \\
\vdots & \vdots & \ddots & \vdots \\
h_1[m] & h_2[m] & \cdots & h_m[m] \\
\end{bmatrix}
\cdot
\begin{bmatrix}
x[1] \\
x[2] \\
\vdots \\
x[m] \\
\end{bmatrix} \\ \\ =
\begin{bmatrix}
h[0] & 0 & 0 & \cdots & 0 \\
h[1] & h[0] & 0 & \cdots & 0 \\
h[2] & h[1] & h[0] & \cdots & 0 \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
h[m-1] & h[m-2] & h[m-3] & \cdots & h[0] \\
\end{bmatrix}
\cdot
\vec{x[n]}
$$
这里的最后一步假设了 $h[-1] = 0$，这要求系统是**因果的(causal)**，$h[n-k] = 0\ when\ k > n$。

同理，对 $\vec{x[n]}$ 的单位脉冲的线性组合的公式可以表示为
$$
\vec{x[n]} =
\begin{bmatrix}
1 & 0 & \cdots & 0 \\
0 & 1 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & 1 \\
\end{bmatrix}
\cdot
\begin{bmatrix}
x[1] \\
x[2] \\
\vdots \\
x[m] \\
\end{bmatrix}
$$
左边是位移单位脉冲 $\delta[n-k]$ 组成的单位矩阵。

## 特征函数与特征向量

### 特征函数(eigenfunction)

一个信号，若系统对该信号的输出响应仅一个常数乘以输入，则称该信号为系统的特征函数。  
同时幅度因子称为系统的特征值。

### 特征向量(eigenvectors)

A scalar $λ ∈ F$ is called an eigenvalue of $T ∈ L(V)$ if there exists a nonzero vector $u ∈ V$ such that $Tu = λu$.  
Suppose $T ∈ L(V)$ and $λ ∈ F$ is an eigenvalue of $T$ . A vector $u ∈ V$ is called an eigenvector of $T$ (corresponding to $λ$) if $Tu = λu$.

### 复指数信号为特征函数的向量表示

信号分析中最重要的基础，一个线性时不变系统对复指数信号的响应也同样是一个复指数信号。  
即复指数是线性时不变系统的特征函数。

> 这个结论只在无限维的情况下成立，向量化的解释更偏向泛函，属于博主的知识盲区

## TODO