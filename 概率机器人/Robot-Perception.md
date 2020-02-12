# Introduction
测量模型：$p(z_t|x_t,m)$，其中，$m$代表环境地图。
- 测量模型的特性取决于传感器：
  - 成像传感器通过投影几何学建立模型
  - 声呐传感器通过描述声波和声波在环境表面的反射建立模型

本章主要阐述测距传感器，但基本原理和公式却不局限于这类传感器，可以适用任何类型的传感器，如相机或者基于条形码的地标探测器。

<center>

![](imgs/Robot-Perception/Figure&#32;6.1.png)**Figure 6.1** 典型声纳传感器测量
</center>

- 声纳传感器
  - 大多数测量与测量锥内最近物体的距离相符,但是有些测量没有检测到任何物体，产生过大的测量距离。发生原因：

    - 物体表面材料
    - 物体表面法线与传感器main cone方向之间的夹角
    - 物体表面的范围
    - 传感器main cone的宽
    - 传感器的灵敏度

  - 读数位数少（？？？），可能是由不同传感器之间的串扰（声音迟缓！）或者由机器人附近没有建模的物体（人）引起的。

- 激光测距传感器和声纳测距类似，都是主动发送一个信号，记录回波。

概率机器人考虑传感器模型的随机误差：将测量过程建模为一个条件概率密度$p(z_t|x_t)$，而不是一个确定的函数$z_t=f(x_t)$。

概率机器人技术在实际应用中可以使用极其粗糙的模型，但必须考虑可能影响传感器测量的不同类型的不确定性。

- 测距传感器测量时通常产生一系列测量值，假设在$t$时刻有$K$个测量值，$z_t={z_t^1,z_t^2,...,z_t^K}$，其中，$z_t^k$表示某一个独立的测量。
$$p(z_t|x_t,m)=\prod_{k=1}^Kp(z_t^k|x_t,m)\tag{1}$$
- 这里和马尔可夫假设一样，认为噪声在所有时刻上都是相互独立。
- 实际噪声不相互独立，原因见Recursive State Estimation的马尔科夫假设里。现在先不用考虑违反独立性的假设，后面章节将详细讨论这个问题。

## Maps
> A map of the environment is a list of objects in the environment and their locations.

地图：环境中的一系列物体及其位置，$m={m_1,m_2,...,m_N}$。式中，$N$为环境中物体的总数，每个$m_n(1\leq n\leq N)$指定一个属性。
1. 基于特征的地图，$n$是一个特征的索引，$m_n$包括特征的属性及特征的笛卡尔位置。（建图中常用）
2. 基于位置的地图，索引$n$与特定的位置对应。在平面地图中，用$m_{x,y}$而不是$m_n$来表示指定坐标$(x,y)$的属性。（导航中常用，如栅格地图）

## Beam Models of Range Finders
测距传感器的近似物理模型。$p(z_t|x_t,m)$根据$x_t,m$推测$z_t$在测量域上的可能性。
### The Basic Measurement Algorithm
- 四类测量误差：
  > 1. Correct range with local measurement noise. 较小的测量误差。
  > 2. Unexpected objects. 意外物体引起的误差。
  > 3. Failures. 未检测到物体引起的误差。
  > 4. Random measurements. 随机意外噪声。
<center>

![](imgs/Robot-Perception/Figure&#32;6.3.png)**Figure 6.3** 四类误差
</center>

$z_t^k$：传感器的测量值
$z_t^{k*}$：由$x_t,m$通过射线投射计算得到的实际真实值
1. Correct range with local measurement noise. 
用一个高斯分布进行建模。测量值$z_t^k$在**测距范围内**服从均值$z_t^{k*}$（由$x_t,m$通过射线投射计算），方差$\sigma_{hit}^2$（下一小节说明如何计算）的正态分布。
$$p_{hit}(z_t^k|x_t,m)=\left\{ \begin{aligned}
  &\eta N(z_t^k;z_t^{k*},\sigma_{hit}^2)\qquad &if\;0\leq z_t^k\leq z_{max} \\
  &0 &otherwise
\end{aligned}\right.\\
\eta = \left( \int_0^{z_{max}}N(z_t^k;z_t^{k*},\sigma_{hit}^2)dz_t^k \right)^{-1}$$ 

2. Unexpected objects.
没有建在map里的意外物体（如人）会使传感器产生比$z_t^k$更小的测量值。
用一个指数分布进行建模。测量值$z_t^k$在**真实距离内**服从模型参数为$\lambda_{short}$（下一小节说明如何计算）的指数分布。
$$p_{short}(z_t^k|x_t,m)=\left\{ \begin{aligned}
  &\eta \lambda_{short}e^{-\lambda_{short}z_t^k}\qquad &if\;0\leq z_t^k\leq z_t^{k*} \\
  &0 &otherwise
\end{aligned}\right.\\
\begin{aligned}
\eta &= \left( \int_0^{z_t^{k*}}\lambda_{short}e^{-\lambda_{short}z_t^k}dz_t^k \right)^{-1} \\
&=\frac{1}{-e^{-\lambda_{short}z_t^{k*}}+e^{-\lambda_{short}0}}\\
&= \frac{1}{1-e^{-\lambda_{short}z_t^{k*}}}
\end{aligned}
$$ 

3. Failures.
有时候环境中的障碍会完全没测量到，如：声纳传感器遇到镜面反射，激光传感器检测到黑色吸光物。检测失败时，传感器通常返回它的最大测量值。
$$p_{max}(z_t^k|x_t,m)=I(z=z_{max})=\left\{ \begin{aligned}
  &1 \qquad &if\; z=z_{max}\\
  &0 \qquad &otherwise
\end{aligned} \right.
$$

4. Random measurements.
测距仪偶尔会产生完全无法解释的测量，例如当超声波被多面墙反射 或 受到不同传感器的串扰。
为了使之简单化，用一个在**传感器测量范围内**的均匀分布建立模型。
$$p_{rand}(z_t^k|x_t,m)=\left\{ \begin{aligned}
  &\frac{1}{z_{max}} \qquad &if\; 0\leq z_t^k\leq z_{max}\\
  &0 \qquad &otherwise
\end{aligned}  \right .$$

用4类不同的分布通过4个权重进行加权平均混合，得到类似Figure 6.4的$p(z_t^k|x_t,m)$分布。该分布的作用可见Recursive State Estimation的Bayes Filters部分。
$$p(z_t^k|x_t,m)=\left( \begin{aligned}
  &z_{hit}\\ &z_{short}\\ &z_{max}\\ &z_{rand}
\end{aligned}\right)^T \left( \begin{aligned}
  &p_{hit}(z_t^k|x_t,m)\\ &p_{short}(z_t^k|x_t,m)\\ &p_{max}(z_t^k|x_t,m)\\ &p_{rand}(z_t^k|x_t,m)
\end{aligned} \right) \\
z_{hit}+z_{short}+z_{max}+z_{rand}=1
$$

<center>

![](imgs/Robot-Perception/Figure&#32;6.4.png)

**Figure 6.4** $\quad p(z_t^k|x_t,m)示例$
</center>

公式1对应算法：

$Algorithm\quad beam\_range\_finder\_model(z_t,x_t,m):\\
\qquad q=1\\
\qquad for\; k=1\; to\; K\; do\\
\qquad \qquad compute\; z_t^{k*}\; for\; the\; measurement\; z_t^k\; using\; ray\; casting\\
\begin{aligned}
\qquad \qquad p=z_{hit}.p_{hit}(z_t^k|x_t,m)+z_{short}.p_{short}(z_t^k|x_t,m)+\\
z_{max}.p_{max}(z_t^k|x_t,m)+z_{rand}.p_{rand}(z_t^k|x_t,m)
\end{aligned}\\
\qquad \qquad q=q.p\\
\qquad return\; q
$

### Adjusting the Intrinsic Model Parameters
目前需要确定的固有参数有：$z_{hit},z_{short},z_{max},z_{rand},\sigma_{hit},\lambda_{short}$