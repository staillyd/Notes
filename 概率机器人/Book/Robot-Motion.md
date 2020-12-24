# Introduction
- 在实践过程中，用精确的模型不如首先假定一些不确定的存在。

# Preliminaries
## Kinematic Configuration
- 刚体移动机器人用6个变量描述位姿，3个代表坐标，3个（pitch,roll,yaw）代表方向。
<center>

![](imgs/Robot-Motion/Figure&#32;5.1.png)**Figure 5.1** _全局坐标系下机器人的位姿_
</center>
在平面环境中，位姿通常用3个变量进行描述，2个（x,y）代表平面位置，1个（yaw）代表头部朝向。

## Probabilistic Kinematics
概率运动模型$p(x_t|u_t,x_{t-1})$在机器人中起着状态变换模型的作用。这里$x_t$和$x_{t-1}$都是机器人位姿，$u_t$是运动控制。该模型描述了对$x_{t-1}$执行运动控制$u_t$后，机器人取得的运动学状态的后验分布。实际应用中，$u_t$有时由机器人的里程计提供，但是，由于概念原因，把$u_t$称为控制。
<center>

![](imgs/Robot-Motion/Figure&#32;5.2.png)**Figure 5.2** _运动模型:执行了实线所示的运动控制后的机器位姿的后验分布_
</center>

- 分布$p(x_t|u_t,x_{t-1})$以阴影区域显示：阴影越黑，可能性越大。图中，位姿的后验概率被映射到$x-y$空间，没有机器人的方向维。图a 中，机器人向前移动一段距离，在这个过程中它可能产生所标示的平移和旋转误差。图b给出了一个更复杂的运动控制下得到的分布，该运动控制导致了更大的不确定性范围。

- 之后介绍两种模型：第一种模型假定运动数据$u_t$指定机器人电动机的速度指令。第二种模型假设机器人具有测距信息。
- 实际上，里程计模型往往比速度模型更精确，一个简单的原因是，大多数商业机器人不能执行与通过计量机器人轮子旋转而获得的精度等级相同的速度控制。但是，里程计信息仅在执行完运动控制后才有能获得，因此它不能用于运动规划。规划算法(如防撞等)必须预测运动的影响。因此，里程计模型通常用于估计（定位），而速度模型用于概率运动规划。

# Velocity Motion Model
速度运动模型 (velocity motion model) 认为可以通过两个速度——一个旋转的和一个平移的速度，来控制机器人。许多商业机器人提供速度的控制接口，通过此接口程序员可以指定速度。

驱动系统通常是差分驱动、阿克曼驱动和同步驱动。这里的模型不包括那些没有非完整约束的驱动系统，如配备了万向轮的机器人或者有腿机器人。

用$v_t$表示$t$时刻的平移速度，向前运动为正；用$w_t$表示旋转速度，逆时针旋转(向左转)为正。
$$u_{t}=\left(\begin{array}{l}{v_{t}} \\ {\omega_{t}}\end{array}\right)$$

<center>

![](imgs/Robot-Motion/Figure&#32;5.5.png)**Figure 5.5**
</center>
- 假设机器人的控制为$$u_{t}=\left(\begin{array}{l}{v_{t}} \\ {\omega_{t}}\end{array}\right)$$其中，用$v_t$表示$t$时刻的平移速度，向前运动为正；用$w_t$表示旋转速度，逆时针旋转(向左转)为正。
- 机器人的位姿为$$\left(\begin{array}{l}{x} \\ {y} \\ \theta \end{array}\right)$$
- 在没有噪声的前提下，假设在一个短时间$\Delta t$内，机器人平移速度$v$和角速度$w$保持不变，那么机器人会做半径$r=|\frac{v}{w}|$的圆周运动。设圆心坐标为$x_c,y_c$，有：
$$\begin{aligned}
    x_c&=x-r*cos(\theta-\frac{\pi}{2})\\
    &=x-\frac{v}{w}*sin(\theta)
\end{aligned}$$  $$\begin{aligned}
    y_c&=y-r*sin(\theta-\frac{\pi}{2})\\
    &=y+\frac{v}{w}*cos(\theta)
\end{aligned}$$
<center>

![](imgs/Robot-Motion/Figure&#32;5.5_new.png)**Figure 5.5_new**
</center>
$$\begin{aligned}
    x'&=x_c+r*cos(w\Delta t+\theta-\frac{\pi}{2})\\
    &=x_c+\frac{v}{w}*sin(w\Delta t+\theta)
\end{aligned}$$  $$\begin{aligned}
    y'&=y_c+r*sin(w\Delta t+\theta-\frac{\pi}{2})\\
    &=y_c-\frac{v}{w}*cos(w\Delta t+\theta)
\end{aligned}$$得到无噪声下的速度运动模型：
$$\begin{aligned}
    \left(\begin{array}{l}{x'} \\ {y'} \\ \theta ' \end{array}\right)&=\left(\begin{array}{l}{x_c+\frac{v}{w}*sin(w\Delta t+\theta)} \\ {y_c-\frac{v}{w}*cos(w\Delta t+\theta)} \\ \theta + w\Delta t \end{array}\right)\\
    &=\left(\begin{array}{l}{x} \\ {y} \\ \theta \end{array}\right)+\left(\begin{array}{l}{-\frac{v}{w}sin\theta+\frac{v}{w}*sin(w\Delta t+\theta)} \\ {\frac{v}{w}cos\theta-\frac{v}{w}*cos(w\Delta t+\theta)} \\ w\Delta t \end{array}\right)
\end{aligned}
$$机器人真实速度和角速度不会和设定值或测量值相等，总是会有噪声影响。且实际运动中短时间内机器人运动不可能总保持圆周运动，机器人的方向也会有噪声干扰。考虑这些噪声干扰，最终得到速度运动模型。
$$\begin{aligned}
    \left(\begin{array}{l}{x'} \\ {y'} \\ \theta ' \end{array}\right)&=\left(\begin{array}{l}{x} \\ {y} \\ \theta \end{array}\right)+\left(\begin{array}{l}{-\frac{\hat{v}}{\hat{w}}sin\theta+\frac{\hat{v}}{\hat{w}}*sin(\hat{w}\Delta t+\theta)} \\ {\frac{\hat{v}}{\hat{w}}cos\theta-\frac{\hat{v}}{\hat{w}}*cos(\hat{w}\Delta t+\theta)} \\ \hat{w}\Delta t+\hat{\gamma}\Delta t \end{array}\right)\\
    \left(\begin{array}{l}{\hat{v}} \\ {\hat{w}}\end{array}\right)&=\left(\begin{array}{l}{v} \\ {w}\end{array}\right)+\left(\begin{array}{l}{\epsilon_{\alpha_1 v^2+\alpha_2 w^2}} \\ {\epsilon_{\alpha_3 v^2+\alpha_4 w^2}}\end{array}\right)\\
    \hat{\gamma}&=\epsilon_{\alpha_5 v^2+\alpha_6 w^2}
\end{aligned}
$$此时，输入$u_t$和$x_{t-1}$，通过速度运动模型可以采样得到$x_t$。

$Algorithm\; sample\_motion\_model\_velocity(u_t,x_{t-1}):\\
...$

# Odometry Motion Model
