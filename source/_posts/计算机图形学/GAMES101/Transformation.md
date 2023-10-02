---
title: Transformation
date: 2023-09-30 00:00:00
tags:
updated:
categories:
- 计算机图形学
- GAMES101
keywords:
description: 2D变换、齐次坐标、组合变换、3D变换、视图变换、投影变换
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

![3D Transformations](https://s2.loli.net/2023/10/01/6J95PpRVmG8vtUg.png)

![3D Transformations](https://s2.loli.net/2023/10/01/CmX6aDLiG5Po8hn.png)

- 绕 x 轴旋转，x 坐标不变，$\overset\rightharpoonup y\times\overset\rightharpoonup z=+\overset\rightharpoonup x$，在 y-z 平面旋转 θ，$R_x(\theta)$ 的 y, z 坐标矩阵与 2D 旋转矩阵 $R_\theta$ 相同；

- 绕 z 轴旋转，z 坐标不变，$\overset\rightharpoonup x\times\overset\rightharpoonup y=+\overset\rightharpoonup z$，在 x-y 平面旋转 θ，$R_z(\theta)$ 的 x, y 坐标矩阵与 2D 旋转矩阵 $R_\theta$ 相同；

- 绕 y 轴旋转，y 坐标不变，$\overset\rightharpoonup z\times\overset\rightharpoonup x=+\overset\rightharpoonup y$，在 z-x 平面旋转 θ，$R_y(\theta)$ 的 z, x 坐标矩阵与 2D 旋转矩阵 $R_\theta$ 相同；
但是因为我们矩阵坐标顺序为 x, y, z，所以实际是 $R_y(\theta)$ 的 x, z 坐标矩阵与 2D 旋转矩阵 $R_\theta^{-1}$ 相同。

> 这里关于 $R_y$ 是自己的理解，GAMES101没有细讲

为什么是 $R_\theta^{-1}$ ？

上面我们提到**逆变换 Inverse Transform**，不难看出，在 z-x 平面逆时针旋转 θ 的角度，相当于在 x-z 平面顺时针旋转 θ 的角度，也就是逆时针旋转 -θ 的角度，自然是一个逆变换。

![Inverse Transform](https://s2.loli.net/2023/10/01/LmXgJP8naeDvRlC.png)

$$R_\theta=\begin{bmatrix}\cos\theta&-\sin\theta\\\sin\theta&\cos\theta\end{bmatrix}$$

$$R_{-\theta}=\begin{bmatrix}\cos(-\theta)&-\sin(-\theta)\\\sin(-\theta)&\cos(-\theta)\end{bmatrix}=\begin{bmatrix}\cos\theta&\sin\theta\\-\sin\theta&\cos\theta\end{bmatrix}=R_\theta^T$$

$$R_{-\theta}=R_\theta^{-1}\;\;(by\;definition)$$

可以得到，旋转矩阵的逆等于旋转矩阵的转置，$R_\theta^{-1}=R_\theta^T$，这在后面会用到。

另外在数学上，如果一个矩阵的逆等于它的转置，我们称之为[正交矩阵](https://baike.baidu.com/item/%E6%AD%A3%E4%BA%A4%E7%9F%A9%E9%98%B5/407284)。


## 3D Rotations

从 $R_x$、$R_y$、$R_z$ 组成任何 3D 旋转：
$${\boldsymbol R}_{xyz}(\alpha,\beta,\gamma)={\boldsymbol R}_x(\alpha){\boldsymbol R}_y(\beta){\boldsymbol R}_z(\gamma)$$

也就是把任意旋转都写成绕 x，绕 y，绕 z 轴旋转的组合，旋转的角度分别是 α，β，γ 角，这三个角又被称作**欧拉角**。

三个轴可以对应飞机的运动：横滚 **roll**、俯仰 **pitch**、偏航 **yaw**，可以看出确实任意旋转都可以分解为绕三个轴旋转的组合。
![roll, pitch, yaw](https://s2.loli.net/2023/10/01/Qitn9I7msvzycj5.png)

## [罗德里格斯旋转公式 Rodrigues’ Rotation Formula](https://baike.baidu.com/item/%E7%BD%97%E5%BE%B7%E9%87%8C%E6%A0%BC%E6%97%8B%E8%BD%AC%E5%85%AC%E5%BC%8F/18878562)

绕 n 轴（用单位向量 $\widehat n$ 表示，起点为原点，方向是 n 轴的方向），旋转角度 α 的旋转矩阵：

![Rodrigues’ Rotation Formula](https://s2.loli.net/2023/10/01/MGk1XtsE2aRAy5Q.png)

如何绕任意轴旋转？变换分解：将轴起点平移到原点；旋转；再移回去。

但是 $R(\boldsymbol n,\;\alpha)+R(\boldsymbol n,\;\beta)\neq R(\boldsymbol n,\;\frac{\alpha+\beta}2)$，因为三角函数并不是线性的，也就是说旋转矩阵不适合作差值，四元数可以解决这个问题。

> GAMES101没有讲关于四元数，这里先不写了，以后再找一下欧拉角、旋转矩阵、四元数之间的转换。
> 留个问题，旋转矩阵转欧拉角作差值可以吗？


### 推导

参考百度百科的过程简单梳理一下：

设 $\overset\rightharpoonup v$ 是一个三维空间向量，$\widehat k$ 是旋转轴的单位向量，则 $\overset\rightharpoonup v$ 在右手螺旋定则意义下绕旋转轴 $\widehat k$ 旋转角度 θ 得到的向量可以由三个不共面的向量 $\overset\rightharpoonup v$ , $\widehat k$ 和 $\widehat k\times\overset\rightharpoonup v$ 表示：

$$\overset\rightharpoonup{v_{rot}}=\cos\theta\overset\rightharpoonup v+(1-\cos\theta)(\overset\rightharpoonup v\cdot\widehat k)\widehat k+\sin\theta\widehat k\times\overset\rightharpoonup v$$

#### 向量形式

思路：对于与旋转轴 $\widehat k$ 呈任意角度的向量 $\overset\rightharpoonup v$，可以通过正交分解，把被旋转向量转化为与旋转轴平行的分量 $\overset\rightharpoonup{v_\parallel}$ 和与旋转轴垂直的分量 $\overset\rightharpoonup{v_\perp}$，其中 $\overset\rightharpoonup{v_\parallel}$ 在旋转中是不变的，而 $\overset\rightharpoonup{v_\perp}$ 则恰好旋转了角度 θ，把 $\overset\rightharpoonup{v_\perp}$ 旋转后的向量 $\overset\rightharpoonup{v_{\perp rot}}$ 与 $\overset\rightharpoonup{v_\parallel}$ 相加，即可得到旋转以后的向量 $\overset\rightharpoonup{v_{rot}}=\overset\rightharpoonup{v_\parallel}+\overset\rightharpoonup{v_{\perp rot}}$ 。

![罗德里格旋转公式](https://s2.loli.net/2023/10/02/bMOLI37riYoEzjl.png)

**Step 1:**
对向量 $\overset\rightharpoonup v$ 正交分解：$\overset\rightharpoonup v=\overset\rightharpoonup{v_\parallel}+\overset\rightharpoonup{v_\perp}$
利用向量投影公式，可得 $\overset\rightharpoonup{v_\parallel}$，其中设 $\widehat k$ 与 $\overset\rightharpoonup v$ 夹角为 α，且 $\widehat k$ 为单位向量，$\left|\widehat k\right|=1$
$$\overset\rightharpoonup{v_\parallel}=proj(\overset\rightharpoonup v,\;\widehat k)=(\left|\overset\rightharpoonup v\right|\cos\alpha)\widehat k=(\frac{\overset\rightharpoonup v\cdot\widehat k}{\left|\widehat k\right|})\widehat k=(\overset\rightharpoonup v\cdot\widehat k)\widehat k$$
通过向量减法，可得 $\overset\rightharpoonup{v_\perp}=\overset\rightharpoonup v-\overset\rightharpoonup{v_\parallel}=\overset\rightharpoonup v-(\overset\rightharpoonup v\cdot\widehat k)\widehat k$

**Step 2:**
首先得到 $\overset\rightharpoonup{v_\perp}$ 对应的单位向量 $\widehat{v_\perp}=normalize(\overset\rightharpoonup{v_\perp})=\frac{\overset\rightharpoonup{v_\perp}}{\left|\overset\rightharpoonup{v_\perp}\right|}$

利用叉乘得到与 $\widehat k$ 和 $\widehat{v_\perp}$ 都垂直的单位向量 $\widehat w$：

$$\widehat w=\widehat k\times\widehat{v_\perp}=\frac{\widehat k\times\overset\rightharpoonup{v_\perp}}{\left|\overset\rightharpoonup{v_\perp}\right|}=\frac{\widehat k\times(\overset\rightharpoonup v-(\overset\rightharpoonup v\cdot\widehat k)\widehat k)}{\left|\overset\rightharpoonup{v_\perp}\right|}=\frac{\widehat k\times\overset\rightharpoonup v-\overset\rightharpoonup0}{\left|\overset\rightharpoonup{v_\perp}\right|}=\frac{\widehat k\times\overset\rightharpoonup v}{\left|\overset\rightharpoonup{v_\perp}\right|}$$

> 此时 $\widehat k$，$\widehat{v_\perp}$，$\widehat w$ 组成一个右手坐标系。
为什么令 $\widehat w=\widehat k\times\widehat{v_\perp}$ 而不是 $\widehat w=\widehat{v_\perp}\times\widehat k$ ?
> 
> 个人理解是方便计算：
> 因为逆时针旋转，一般假设旋转角度 $\theta\;(0\leq\theta\leq180^\circ)$，正交分解时计算方便，否则要处理负号；后面由 $\overset\rightharpoonup{v_{rot}}$ 推导旋转矩阵时也容易提取 $\overset\rightharpoonup v$
>
> 当然令 $\widehat w=\widehat{v_\perp}\times\widehat k$ 也完全 OK，只是某些部分会多一个负号，这只是我们推导的中间过程，最终结果是一样的，因为有 $\overset\rightharpoonup a\times\overset\rightharpoonup b=-\overset\rightharpoonup b\times\overset\rightharpoonup a$

**Step 3:**
然后可以对旋转 θ 角度之后的 $\overset\rightharpoonup{v_{\perp rot}}$ 作正交分解：

$$\overset\rightharpoonup{v_{\perp rot}}=(\left|\overset\rightharpoonup{v_{\perp rot}}\right|\cos\theta)\widehat{v_\perp}+(\left|\overset\rightharpoonup{v_{\perp rot}}\right|\sin\theta)\widehat w$$

显然，有 $\left|\overset\rightharpoonup{v_{\perp rot}}\right|=\left|\overset\rightharpoonup{v_\perp}\right|$，所以：

$$\overset\rightharpoonup{v_{\perp rot}}=\cos\theta\overset\rightharpoonup{v_\perp}+\sin\theta\widehat k\times\overset\rightharpoonup v=\cos\theta(\overset\rightharpoonup v-(\overset\rightharpoonup v\cdot\widehat k)\widehat k)+\sin\theta\widehat k\times\overset\rightharpoonup v$$

最终得到旋转后的向量表达式 $\overset\rightharpoonup{v_{rot}}$：

$$\overset\rightharpoonup{v_{rot}}=\overset\rightharpoonup{v_\parallel}+\overset\rightharpoonup{v_{\perp rot}}=(\overset\rightharpoonup v\cdot\widehat k)\widehat k+\cos\theta(\overset\rightharpoonup v-(\overset\rightharpoonup v\cdot\widehat k)\widehat k)+\sin\theta\widehat k\times\overset\rightharpoonup v$$

即为：

$$\overset\rightharpoonup{v_{rot}}=\cos\theta\overset\rightharpoonup v+(1-\cos\theta)(\overset\rightharpoonup v\cdot\widehat k)\widehat k+\sin\theta\widehat k\times\overset\rightharpoonup v$$

#### 矩阵形式

在计算机图形学中，罗德里格向量旋转公式通常被用来填写旋转矩阵 $R(k, \theta)$。

旋转以后的向量可以表示为：$\overset\rightharpoonup{v_{rot}}=R(k,\;\theta)\overset\rightharpoonup v$

只需从向量表达式中提取出一个右乘的 $\overset\rightharpoonup v$，即可得：

$$R(k,\;\theta)=\cos\theta I+(1-\cos\theta)\widehat k\widehat k^T+\sin\theta K$$

其中，$I$ 为 3 阶单位矩阵（也有用 $E$ 表示的），$K$ 为单位向量 $\widehat k$ 的矩阵表示：$\widehat k\times\overset\rightharpoonup v=K\overset\rightharpoonup v$

假设：$\widehat k=\begin{pmatrix}k_x&k_y&k_z\end{pmatrix}^T,\;\;\;\overset\rightharpoonup v=\begin{pmatrix}v_x&v_y&v_z\end{pmatrix}^T$，则：

$$R(k,\;\theta)=\cos\theta I+(1-\cos\theta)\begin{pmatrix}k_x\\k_y\\k_z\end{pmatrix}\begin{pmatrix}k_x&k_y&k_z\end{pmatrix}+\sin\theta\begin{pmatrix}0&-k_z&k_y\\k_z&0&-k_x\\-k_y&k_x&0\end{pmatrix}$$


# 观测变换 Viewing transformation

## 视图变换 View/Camera transformation

## 投影变换 Projection transformation

### 正交投影 Orthographic projection

### 透视投影 Perspective projection