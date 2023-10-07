# 三角形重心坐标
### **基础定义**

![](https://pic4.zhimg.com/80/v2-e5142833a1194a1f0ee81927babfc573_720w.webp)
(如果该点在三角形内部则三点坐标都为非负数)
### 面积角度求解
重心坐标还有一个另外一个角度的等价定义，如下图：

![](https://pic2.zhimg.com/80/v2-b0c0d74b3d9b57ff74d01fa0719b12a5_720w.webp)

将一点(x,y)与A,B,C三点直接连接，构成三个三角形面积分别为Aa,Ab,Ac即可直接定义出重心坐标如图中公式所示。
已知图中4点坐标，那么求出面积自然是很简单的，这里利用行列式的几何意义直接求解，即任意2个2维向量组合成矩阵的行列式的绝对值，为这两条向量所围成平行四边形的面积 (如果对行列式几何意义不清楚的话建议前往B站搜索3B1B的线代本质)。设任意一点为P(x,y)，则由：

**Ab=|AP,AC|, 其中AP,AC为列向量 符号||为行列式**
**Ac=|AB,AP|**
**Aa=|BC,BP|**

此处省略行列式计算步骤，不难得出

![](https://pic4.zhimg.com/80/v2-7430b9fe2ef0813064fd19fc2de51a37_720w.webp)

![](https://pic1.zhimg.com/80/v2-64f8b20578a421aa35dffe2fddd454e4_720w.webp)

注意，求出其中2个之后第三个可以根据性质直接求出。
### 几何角度求解
针对重心坐标系的另外一种等价视角便是，以A点为原点，AB,AC分别为新的坐标系的单位向量构建坐标系，如图所示：

![](https://pic2.zhimg.com/80/v2-34f542500eb2b8b4679dc351609398ad_720w.webp)

  

其中给定任意点P的坐标可以用 $\alpha,\beta$来表示，即$P(\alpha,\beta)$,不难得出，P点坐标在原坐标系中的位置满足：

![](https://pic2.zhimg.com/80/v2-2e4aa679fdeffc76accb673e9a0e63cd_720w.webp)

进一步化简得到：

![](https://pic4.zhimg.com/80/v2-069b7c32aed6c23f0e850f57f3364787_720w.webp)

到这里其实就可以发现，我们已经成功的把P用三角形的三个顶点的线性组合表示了，并且3个权重之和为1，完全符合定义，因此这是一种完全等价的角度！

从这种角度出发，我们完全可以将上式表现成一个线性方程组来求解出 $\alpha,\beta$，两个未知量，两个式子，如下：

![](https://pic3.zhimg.com/80/v2-05e96c2ccfc2476783c24fd25e57ed7a_720w.webp)

解出该方程组的解其实与1.2节中面积的角度式完全一样的，不在这里展开。
知道重心坐标![(\alpha ,\beta ,\gamma )](https://latex.csdn.net/eq?%28%5Calpha%20%2C%5Cbeta%20%2C%5Cgamma%20%29)之后，可以根据A,B,C三点的属性值![V_{A},V_{B},V_{C}](https://latex.csdn.net/eq?V_%7BA%7D%2CV_%7BB%7D%2CV_%7BC%7D)算出在三角形内的坐标点(x,y)的属性V,公式如下

![V=\alpha V_{A}+\beta V_{B}+\gamma V_{C}](https://latex.csdn.net/eq?V%3D%5Calpha%20V_%7BA%7D&plus;%5Cbeta%20V_%7BB%7D&plus;%5Cgamma%20V_%7BC%7D)

这里的![V_{A},V_{B},V_{C}](https://latex.csdn.net/eq?V_%7BA%7D%2CV_%7BB%7D%2CV_%7BC%7D)可以表示各种属性

**注：在三维空间的属性应该现在三维空间求重心坐标之后投影回二维坐标。而不是先投影再求二维坐标系中的重心坐标。（由于中心坐标会在投影过程中产生变化）**
