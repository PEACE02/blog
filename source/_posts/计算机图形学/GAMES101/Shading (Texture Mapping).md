---
title: Shading (Texture Mapping)
date: 2023-10-18 00:00:00
tags:
updated:
categories:
- 计算机图形学
- GAMES101
keywords:
description: 纹理映射、重心坐标插值
cover: https://s2.loli.net/2023/10/18/xcXwVrkaTWKFOSi.jpg
---

# 纹理映射 Texture Mapping

## 定义

在下图中，我们可以把两个台灯当作两个点光源，只考虑漫反射项，可以看到图中不同的地方有不同的颜色，它们的区别在于：共用一个着色模型，但是漫反射系数 $k_d$ 不同。

也就是说我们希望定义一种方法：对于一个物体，它上面的任何一个点，可以有不同的属性。这就是纹理映射，它的根本作用是定义任何一个点的不同属性，不只限于漫反射系数。

![Different Colors at Different Places](https://s2.loli.net/2023/10/18/KS2RDJqG6B4VMEl.png)

> 每个三维物体的表面，都可以展开成一个二维平面，多个物体的表面可以展开成多个平面，再放到一个平面内。那么物体的表面，通过这种方式，可以和一张图有一个一一对应关系。

Every 3D surface point also has a place where it goes in the 2D image (**texture**).
![Surfaces are 2D](https://s2.loli.net/2023/10/18/FcueLUTo3XMKhNk.png)


## 纹理坐标系

也就是我们要把一张图，贴到物体表面上，怎么贴？找到每个三角形在纹理空间（纹理图像）上的坐标位置。
> 这个纹理坐标我们认为是已知的，实际开发中由美术同学或自动化设计等。

![Texture Applied to Surface](https://s2.loli.net/2023/10/18/i9HzL8vbPxA5uFa.png)

定义纹理坐标系：每个三角形顶点都分配有一个纹理坐标 (u, v)，规定 u, v ∈ [0, 1] 。
![Visualization of Texture Coordinates](https://s2.loli.net/2023/10/18/VbjaCRLzmhIontg.png)

纹理可以多次使用，比如以平铺的方式，那么纹理图的设计也要注意上下和左右的无缝衔接。[Wang tile](https://en.wikipedia.org/wiki/Wang_tile)
![example textures](https://s2.loli.net/2023/10/18/oA75LGWjKIU1par.png)


## 三角形插值：重心坐标

为什么要在三角形内部进行插值？希望根据三角形顶点的属性值，计算得到三角形内部的值以得到一个平滑的过渡。

要插值什么内容？我们在三角形顶点上可以定义各种不同的属性，比如纹理坐标、颜色、深度、法向量……

怎么插值？重心坐标。

对 $\bigtriangleup ABC$ **内**任一点 $(x, y)$ 都可以表示为三个顶点坐标的线性组合，定义 $(x,y)=\alpha A+\beta B+\gamma C$，$\alpha+\beta+\gamma=1$，$\alpha,\beta,\gamma\geq0$
![Barycentric Coordinates](https://s2.loli.net/2023/10/18/t6mFKW1krfOoiLx.png)

比如 A 点的重心坐标为 (1, 0, 0)
![the barycentric coordinate of A](https://s2.loli.net/2023/10/18/g3XI5MqWAQwFrNi.png)

重心坐标 $(\alpha,\beta,\gamma)$ 的几何意义：区域面积
![Geometric viewpoint — proportional areas](https://s2.loli.net/2023/10/18/NfHPZQlgGF5bBpW.png)

三角形重心的重心坐标是 $(\frac{1}{3},\frac{1}{3},\frac{1}{3})$，与三个顶点相连，将三角形分成了三个等面积的三角形。
![the barycentric coordinate of the centroid](https://s2.loli.net/2023/10/18/lgL8AmsM6tpPc7G.png)

对于任意一个点的重心坐标计算，可以算面积（三角形面积 $S=\frac{1}{2}absinC$，可以利用向量叉积的值），最终可以得到下面一个简化的重心坐标公式：
![Barycentric Coordinates Formulas](https://s2.loli.net/2023/10/18/rkz89WaeFLfq4IR.png)

重心坐标插值可以计算任何定义在顶点的属性在三角形内部的值，包括位置、纹理坐标、颜色、深度、材质属性等。

然而，重心坐标在投影变换下会发生变化。所以插值应该在三维空间中的三角形做，而不能在投影之后的三角形里做插值。比如深度，光栅化后计算像素中心的深度时，应该找到像素中心点对应的三角形位置在三维空间中的坐标（逆变换），在三维空间中计算深度插值再对应到二维结果中。
![Using Barycentric Coordinates](https://s2.loli.net/2023/10/18/f2pPWeiOFBYtjN3.png)


## 纹理的应用

### 简单纹理映射：漫反射颜色 Diffuse Color

对光栅化屏幕上的任何一个采样点 (x, y)，根据定义在三角形顶点上的纹理坐标，可以通过重心插值得到每一个采样点的纹理坐标 (u, v)，查询纹理图片中 (u, v) 的位置，也就得到了采样点 (x, y) 的纹理颜色 `texcolor = texture.sample(u, v)`，可以直接用纹理颜色 texcolor 来代替采样点的颜色（通常是说漫反射系数 kd，即 sample color = kd = texcolor）

![Simple Texture Mapping Diffuse Color](https://s2.loli.net/2023/10/27/Vv1DbjC4hfHY8yp.png)

但是实际上，并没有这么简单，或者说这么简单的处理会出现一些问题。


### 纹理的放大 Texture Magnification（Easy Case

如果纹理太小怎么办？（纹理分辨率不足

也就是分辨率较低的纹理图被映射到分辨率较高场景中，可以简单理解为 256x256 的图片被映射到 4k 的屏幕上。那么纹理会被拉大，会查到一些非整数的纹理坐标值。

我们将查询到的非整数的纹理坐标值四舍五入成整数，这样的话就相当于在一定的范围内我们查找到的纹理坐标是相同的，也就是一个相同的纹理上的像素。纹理上的像素 A **pixel** on a texture，我们称为 **texel**（纹理元素、纹素）。
> pixel 就是我们生成的画面上的一个像素，texel 就是纹理上的一个像素。

也就是说一定范围内的 pixel 都会被映射到同一个 texel，这是因为纹理太小了，对应结果为 **Nearest**。

![Texture Magnification - Easy Case](https://s2.loli.net/2023/10/27/CGemYjMNbcfzDSB.png)

但是显然结果不太能让人满意，我们希望得到中间或右边的效果（可以看起来模糊一点，但得到的结果至少连续一点，有一个平滑的过渡）。也就是说当纹理查询的结果得到是非整数的坐标，那我们应该如何得到它的值？

#### 双线性插值 Bilinear Interpolation

下图中，我们想要在红点处采样纹理值 f(x,y)，黑点表示纹理图中的采样位置（texel 中心点）。
![Bilinear Interpolation](https://s2.loli.net/2023/10/27/p9eCmIiwdWgu4Zx.png)

取 4 个最近的样本位置 $u_{00}$、$u_{01}$、$u_{10}$、$u_{11}$，并标记纹理值和（距左下角）小数偏移量 (s, t)，如图所示。（假设两个 texel 中心之间的距离为 1，则 0 < s, t < 1）
![Bilinear Interpolation](https://s2.loli.net/2023/10/27/bmvQTfCk17pK4DN.png)

一维线性插值 Linear Interpolation(1D)，假设 $x$ 为 $v_0$ 和 $v_1$ 之间、与 $v_0$ 的距离是 x (0~1) 的点，可以根据 $v_0$ 和 $v_1$ 的值和 $x$ 与 $v_0$、$v_1$ 的距离，定义 $x$ 的值为 $lerp(x, v_0, v_1) = v_0 + x(v_1 - v_0)$。

由此我们定义 $u_{00}$ 和 $u_{10}$ 之间一点 $u_0$，$u_{01}$ 和 $u_{11}$ 之间一点 $u_1$，利用（水平方向）一维线性插值得到 $u_0$、$u_1$ 两点处的值，再（竖直方向）一维线性插值得到 $u_0$ 和 $u_1$ 之间红点处的值，如图所示。
![Bilinear Interpolation](https://s2.loli.net/2023/10/27/uIcFja7f1CPxq5O.png)

因为我们分别在水平和竖直方向上（顺序无所谓）做线性插值，所以称为双线性插值，双线性插值通常以合理的成本给出很好的结果，如下图中间所示。
![Bilinear Interpolation](https://s2.loli.net/2023/10/27/jlT6LgMs1294v8y.png)

当然双线性插值跟一些更复杂的方法（比如 [Bicubic Interpolation](https://en.wikipedia.org/wiki/Bicubic_interpolation)）相比，效果会差一些，但是开销会更小。


### 纹理的放大 Texture Magnification（Hard Case

如果纹理太大怎么办？（会引起更严重的问题

对下图左侧纹理，如果做简单的纹理映射，会得到右侧走样的结果，远处有摩尔纹，近处有锯齿。
![Point Sampling Textures — Problem](https://s2.loli.net/2023/10/28/QGqwSgy93i1h5sn.png)

如下图，当一个屏幕像素 pixel 覆盖的纹理像素 texel 较多时，一个 pixel 内有非常高频的信息，当然会产生走样问题。
![Screen Pixel “Footprint” in Texture](https://s2.loli.net/2023/10/28/3zrh9xANmOkYjSc.png)

按照之前抗锯齿的思想（MSAA），如果我们做超采样，简单理解为将屏幕分辨率“变大”，以此对应较大的纹理，如下图 512x 超采样（一个 pixel 用 512 个采样点），这可以做到比较好的效果，但是代价太大。
![Will Supersampling Do Antialiasing](https://s2.loli.net/2023/10/28/b7istvcYJxjroHG.png)


这里我们用另一种思路：**采样会引起走样问题，那不采样怎么样？**

如何避免采样？超采样之所以代价较大，是要根据子像素采样点的值求平均来计算一个像素的值，如果能不采样，立刻得到这个平均值，也就是纹理图上任何一个区域，我们需要快速的知道它的平均值是多少，这就是 Mipmap 解决的问题。

不同位置的像素（远近），在纹理图上有不同大小的覆盖范围。
![Different Pixels -> Different-Sized Footprints](https://s2.loli.net/2023/10/28/lPVjRfMQbT9gJCK.png)


### Mipmap（快速、近似、方形）范围查询

“Mip” 源自拉丁语 “multum in parvo”，意思是狭小空间内的众多。

我们把原始纹理图称作 Level 0，然后生成更多层，每一层都是上一（i - 1）层分辨率的宽高各缩小一半，一直到最后只剩 1x1 的分辨率。
![Mipmap (L. Williams 83)](https://s2.loli.net/2023/10/28/YJV24Tku7m8ERoe.png)

假设 nxn 的图，会有 logn 层，下图是层次结构（图像金字塔）。
![Mip hierarchy](https://s2.loli.net/2023/10/28/TmID8AUgBnQC6ur.png)

Mipmap 的关键在于我们拿到纹理后就可以在渲染之前提前计算做一个预处理并存储。而它的额外存储开销（不包括 Level 0）是原存储开销（Level 0）的 $\frac{1}{3}$。


任何像素都可以映射到纹理上的一个区域，怎么得到这个区域？

下图中，左侧屏幕空间，以红色采样点为例，我们希望知道（左下角）红点在纹理中的覆盖面积，取它和它邻居的中心，如红色箭头所示。分别投影到纹理空间上。
![Computing Mipmap Level D](https://s2.loli.net/2023/10/28/BzR8Kk1OFu2eGsv.png)

我们认为屏幕空间中相邻采样点之间的距离为 1，求映射到纹理空间后采样点与两个相邻采样点之间的距离分别为 $L_1=\sqrt{\left(\frac{du}{dx}\right)^2+\left(\frac{dv}{dx}\right)^2}$ 和 $L_2=\sqrt{\left(\frac{du}{dy}\right)^2+\left(\frac{dv}{dy}\right)^2}$。其实就是两点之间的距离公式，各取一半，近似得到屏幕空间中一个 pixel 在纹理上对应的区域。
![Computing Mipmap Level D](https://s2.loli.net/2023/10/28/fxzMblGAHmXpgQE.png)

然后取 $L=max(L_1, L_2)$ 近似作为一个正方形的覆盖范围的边长，也就是用右侧纹理空间中一个正方形框，来近似作为屏幕空间中一个 pixel 在纹理中的覆盖范围。
![Computing Mipmap Level D](https://s2.loli.net/2023/10/28/TFQ7Lj2m5nbOe69.png)


现在我们得到了映射的区域，如何根据之前计算好的 mipmap 来查询这个区域的平均值？

如果 L = 1，也就是一个 pixel 对应一个 texel，那么只需要在原始的纹理图上找对应位置的纹理像素值；如果 L = 4，也就是在 level 0 是一个 4x4 的正方形区域，那么我们知道在 level 2 中一定是一个 1x1 的区域，也就是一个 texel。

也就是说，我们应该在第 $D=log_{2}L$ 层查询，会得到一个 texel 的映射结果，也就是原始纹理中一个方形区域的平均值。

如果我们对 D 的结果四舍五入到整数，会得到 Mipmap Level 的可视化结果为下图，一个颜色就是在一个 Level 的查询。
![D rounded to nearest integer level](https://s2.loli.net/2023/10/28/sPutvBK8Ha5W4cd.png)

与前面 Nearest 结果类似，仍然是不连续的现象，我们希望有一个平滑的过渡，线性插值是一个办法。

#### 三线性插值 Trilinear Interpolation

比如要查询第 1.8 层，即 D = 1.8，那么分别在第 1 层和第 2 层做双线性插值，然后根据两个双线性插值的结果，在层与层之间再做一次线性插值，作为第 1.8 层的查询结果，我们称为这个过程为三线性插值。
![Trilinear Interpolation](https://s2.loli.net/2023/10/28/VtCQNOJSrvlipbU.png)

通过三线性插值，我们可以在浮点数、连续层进行查询，得到的结果如下图，可以看到明显多了平滑的过渡。
![Visualization of Mipmap Level](https://s2.loli.net/2023/10/28/j1MxKEotbHPArQZ.png)


### 各向异性过滤 Anisotropic Filtering

以 512x 超采样的结果为标准，我们可以看到 mipmap 在远处的结果非常模糊。
![Mipmap Limitations](https://s2.loli.net/2023/10/28/ouem4PYCMAxLr3H.png)

这是因为在查询过程中做的近似处理较多，比如方形范围的近似，只能查询方形区域，三线性插值也是近似。

各向异性过滤可以部分解决 mipmap 的这个问题。
![Anisotropic Filtering](https://s2.loli.net/2023/10/28/XuW3tPxANOnwKfd.png)

在下图右侧中，mipmap 做的就是对角线上等比例的方形图片，但是对不同的长宽比，是 mipmap 没有的。通过对不同长宽比的预处理计算，我们就可以对原始纹理做一个矩形查询，因为一个 “压扁的” 层级中的一个点，对应在原始纹理中即为一个矩形。这样就不用限制在正方形区域上的查询。
![Anisotropic Filtering](https://s2.loli.net/2023/10/28/4QE53YVsfcyGNlp.png)

屏幕上的像素映射到纹理上不一定是规则的形状，各向异性过滤可以查找轴对齐的矩形区域，比 mipmap 只能做方形区域的查询要好（但是额外的开销是原始纹理的 3 倍，而 mipmap 只是原本的 1/3），当然对角线查询仍是一个问题。
![Irregular Pixel Footprint in Texture](https://s2.loli.net/2023/10/28/12cfr6zTBeDiUhG.png)

应对不规则形状的查询，还有其他方法。比如 [EWA 过滤](https://zhuanlan.zhihu.com/p/105167411)，使用多次查询的加权平均，mipmap 的层次结构思想仍然有用，可以应对不规则查询。
![EWA filtering](https://s2.loli.net/2023/10/28/3fqMoe5QxPmc1vb.png)