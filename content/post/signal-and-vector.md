+++
title = '信号与向量'
date = 2025-02-02T23:06:46+08:00
draft = false
math = true

+++

## 信号的向量表示
把某个离散信号 $x[n]$ 看为 $n$ 维的向量 $v_x$
$$v_x (x[1], x[2], x[3], .., x[n])$$

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
**卷积和(convolution sum)**
对于离散信号
$$
x[n] = \sum_{k=-\infty}^{+\infty} x[k] \delta[n-k]
$$
我们把 $\delta[n-k]$ 的单位脉冲看为基向量  

某个线性系统对于$x[n]$的响应$y[n]$
可以由变换后的基向量$h[n-k]$表示
$$
y[n] = \sum_{k=-\infty}^{+\infty} x[k] h[n-k]
$$

$$
\delta[n-k] \rightarrow h_k[n]
$$
$$
h_k[n] \rightarrow h[n-k]
$$

**矩阵乘法(matrix multiplication)**

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

## TODO