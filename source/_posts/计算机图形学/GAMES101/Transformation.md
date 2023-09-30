---
title: Transformation
date: 2023-09-30 00:00:00
tags:
updated:
categories:
- 计算机图形学
- GAMES101
keywords:
description: 
cover: https://s2.loli.net/2023/09/28/VSBJan7xdz3pUik.jpg
---

# 2D transformations

## 缩放 Scale

- Transform: $x'=s_xx\;;\;y'=s_yy$
- Matrix: $\begin{bmatrix}x'\\y'\end{bmatrix}=\begin{bmatrix}s_x&0\\0&s_y\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix}$

### Scale (Uniform)
![Scale (Uniform)](https://s2.loli.net/2023/09/30/TKsNMBl4hunb9JP.png)

### Scale (Non-Uniform)
![Scale (Non-Uniform)](https://s2.loli.net/2023/09/30/JCYsUbV2j6NmSKM.png)

### Reflection Matrix
![Reflection Matrix](https://s2.loli.net/2023/09/30/uCv1ZPciJQre2RD.png)

### Shear Matrix
![Shear Matrix](https://s2.loli.net/2023/09/30/dgOmYUlKbIhLDC6.png)


## 旋转 Rotate 

- **默认绕 (0, 0) ，逆时针（Counterclockwise）旋转**

![Rotation Matrix](https://s2.loli.net/2023/09/30/plmaI8FCPxZAqic.png)

$R_\theta$ 矩阵推导：

设 $x'=R_\theta x$，即 $\begin{bmatrix}x'\\y'\end{bmatrix}=\begin{bmatrix}a&b\\c&d\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix}$

由特殊点：
( cosθ, sinθ) ← (1, 0) 得 a =  cosθ, c = sinθ ；
(-sinθ, cosθ) ← (0, 1) 得 b = -sinθ, d = cosθ 。


## Linear Transforms = Matrices (of the same dimension)

$$x'\;=\;ax\;+\;by\;\;\;\;\;y'\;=\;cx\;+\;dy$$

$$\begin{bmatrix}x'\\y'\end{bmatrix}=\begin{bmatrix}a&b\\c&d\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix}$$

$$x'=Mx$$


## 平移 Translation
![Translation](https://s2.loli.net/2023/09/30/kBLqyTsZKXD1JfM.png)

$$x'\;=\;x\;+\;t_x\;\;\;\;\;y'\;=\;y\;+\;t_y$$

$$\begin{bmatrix}x'\\y'\end{bmatrix}\;=\;\begin{bmatrix}a&b\\c&d\end{bmatrix}\;\begin{bmatrix}x\\y\end{bmatrix}\;+\;\begin{bmatrix}t_x\\t_y\end{bmatrix}$$

平移不能以矩阵形式表示，不是线性变换。但是我们不希望平移成为一种特殊情况。

是否有一个统一的方式来表示所有的变换？（代价是什么？）


# 齐次坐标 Homogeneous coordinates

**添加第三个坐标：_w_**
- 2D point &nbsp;= $(x, y, 1)^T$
- 2D vector = $(x, y, 0)^T$

**平移的矩阵表示：** $\begin{bmatrix}x'\\y'\\w'\end{bmatrix}\;=\;\begin{bmatrix}1&0&t_x\\0&1&t_y\\0&0&1\end{bmatrix}\cdot\begin{bmatrix}x\\y\\1\end{bmatrix}\;=\;\begin{bmatrix}x\;+\:t_x\\y\;+\:t_y\\1\end{bmatrix}$


如果平移一个向量会怎样？
向量只关注长度、方向，经过平移之后保持不变。

w = 1，表示一个点 point，w = 0 表示一个向量 vector：
- vector + vector = vector
- vector - vector = vector + (-vector)
- point + vector = point
- point - vector = point + (-vector)
- point + point = ???
- point - point = vector

当 point + point 时，结果的 w = 2，我们规定在齐次坐标中：
$$(x, y, w)^T\;is\;the\;2D\;point\;(\frac xw, \frac yw, 1)^T,\;w \neq 0$$
所以齐次坐标中，两点相加得到的是两点的中点 point + point = (mid) point。


## 仿射变换 Affine Transformations

- **先线性变换，再平移**（Affine map = linear map + translation
$$\begin{bmatrix}x'\\y'\end{bmatrix}=\begin{bmatrix}a&b\\c&d\end{bmatrix}\begin{bmatrix}x\\y\end{bmatrix}+\begin{bmatrix}t_x\\t_y\end{bmatrix}$$

- 使用齐次坐标：
$$\begin{bmatrix}x'\\y'\\1\end{bmatrix}\;=\;\begin{bmatrix}a&b&t_x\\c&d&t_y\\0&0&1\end{bmatrix}\cdot\begin{bmatrix}x\\y\\1\end{bmatrix}$$

![2D Transformations using Homogeneous coordinates](https://s2.loli.net/2023/09/30/Be19U3XEPI7g2md.png)

## 逆变换 Inverse Transform
![Inverse Transform](https://s2.loli.net/2023/09/30/zVvAyY5Ew9Xa4hZ.png)

$$x = Ix = M^{-1}Mx = M^{-1}x'$$


# 组合变换 Composing Transforms

![Transform Ordering Matters!](https://s2.loli.net/2023/09/30/YcrutqKwI5WTLPe.png)

- 变换的顺序很重要，因为矩阵乘法没有交换律：
$$R_{45}\cdot T_{(1,0)}\neq T_{(1,0)}\cdot R_{45}$$
- 注意变换矩阵是从右到左应用的：
$$T_{(1,\;0)}\cdot R_{45}\begin{bmatrix}x\\y\\1\end{bmatrix}=\begin{bmatrix}1&0&1\\0&1&0\\0&0&1\end{bmatrix}\begin{bmatrix}\cos45^\circ&-\sin45^\circ&0\\\sin45^\circ&\cos45^\circ&0\\0&0&1\end{bmatrix}\begin{bmatrix}x\\y\\1\end{bmatrix}$$


对仿射变换序列：$A_1, A_2, A_3, ..., A_n$，由矩阵乘法组成：
![Composing Transforms](https://s2.loli.net/2023/09/30/bmSQXWvGqzf3hid.png)

矩阵乘法虽然没有交换律，但是有结合律，**预乘 n 个矩阵以获得表示组合变换的单个矩阵**，这对性能来说非常重要。

$$\begin{bmatrix}x'\\y'\\1\end{bmatrix}=\begin{bmatrix}1&0&t_x\\0&1&t_y\\0&0&1\end{bmatrix}\begin{bmatrix}a&b&0\\c&d&0\\0&0&1\end{bmatrix}\begin{bmatrix}x\\y\\1\end{bmatrix}=\begin{bmatrix}a&b&t_x\\c&d&t_y\\0&0&1\end{bmatrix}\begin{bmatrix}x\\y\\1\end{bmatrix}$$

## 分解复杂的变换 Decomposing Complex Transforms

我们之前说的旋转，是绕原点 (0, 0)，如何绕给定的点 c 旋转？
1. 将点 c 平移至原点；
2. 旋转；
3. 将点 c 平移回去。

![rotate around a given point c](https://s2.loli.net/2023/09/30/hUAoEmMPvzTHtxa.png)


# 3D Transforms

类比 2D 变换，使用齐次坐标：
- 3D point &nbsp;= $(x, y, z, 1)^T$
- 3D vector = $(x, y, z, 0)^T$

$$In\;general,\;(x, y, z, w)^T\;is\;the\;3D\;point\;(\frac xw, \frac yw, \frac zw, 1)^T,\;w \neq 0$$

使用 4×4 矩阵进行仿射变换（先线性变换，再平移）：
$$\begin{bmatrix}x'\\y'\\z'\\1\end{bmatrix}=\begin{bmatrix}1&0&0&t_x\\0&1&0&t_y\\0&0&1&t_z\\0&0&0&1\end{bmatrix}\cdot\begin{bmatrix}a&b&c&0\\d&e&f&0\\g&h&i&0\\0&0&0&1\end{bmatrix}\cdot\begin{bmatrix}x\\y\\z\\1\end{bmatrix}=\begin{bmatrix}a&b&c&t_x\\d&e&f&t_y\\g&h&i&t_z\\0&0&0&1\end{bmatrix}\cdot\begin{bmatrix}x\\y\\z\\1\end{bmatrix}$$