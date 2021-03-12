## 假设情况

- 状态量$\textbf{x}_t=[x_t, y_t, \phi_t, v_t]$
  - $x,y$:2D坐标
  - $\phi$:方向
  - $v$:速度
- 机器人具有speed sensor,gyro sensor.$\textbf{u}_t=[v_t,w_t]$
- 机器人具有GNSS sensor,可以每时每刻观测到位置$\textbf{z}_t=[x_t,y_t]$

## Motion Model

> 与[RobotMotion](../Book/Robot-Motion.md)相比,不考虑圆周运动,简化模型,而且貌似多加了个$v$状态,在状态空间方程上看是个冗余,但存在$v$与不存在$v$的雅可比矩阵不同

- $\begin{aligned}
  x_{t+1}&=x_t+vcos(\phi_t)dt\\
  y_{t+1}&=y_t+vsin(\phi_t)dt\\
  \phi_{t+1}&=\phi_t+wdt
  \end{aligned}$
- 将其转化为状态空间方程
- $\textbf{x}_{t+1} = F\textbf{x}_t+B\textbf{u}_t$
- $\begin{aligned}
  \begin{bmatrix}
  x_{t+1}\\
  y_{t+1}\\
  \phi_{t+1}\\
  v_{t+1}
  \end{bmatrix}=
  \begin{bmatrix}
  1 & 0 & 0 & 0\\
  0 & 1 & 0 & 0\\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 0 \\
  \end{bmatrix}\begin{bmatrix}
  x_{t}\\
  y_{t}\\
  \phi_{t}\\
  v_{t}
  \end{bmatrix}+
  \begin{bmatrix}
  cos(\phi)dt & 0\\
  sin(\phi)dt & 0\\
  0 & dt\\
  1 & 0\\
  \end{bmatrix}\begin{bmatrix}
  v_t\\
  w_t
  \end{bmatrix}
  \end{aligned}$
- $J_F=\begin{bmatrix}
  \frac{dx}{dx}& \frac{dx}{dy} & \frac{dx}{d\phi} &  \frac{dx}{dv}\\
  \frac{dy}{dx}& \frac{dy}{dy} & \frac{dy}{d\phi} &  \frac{dy}{dv}\\
  \frac{d\phi}{dx}& \frac{d\phi}{dy} & \frac{d\phi}{d\phi} &  \frac{d\phi}{dv}\\
  \frac{dv}{dx}& \frac{dv}{dy} & \frac{dv}{d\phi} &  \frac{dv}{dv}\\
  \end{bmatrix}=\begin{bmatrix}
  1& 0 & -v sin(\phi)dt &  cos(\phi)dt\\
  0 & 1 & v cos(\phi)dt & sin(\phi) dt\\
  0 & 0 & 1 & 0\\
  0 & 0 & 0 & 1\\
  \end{bmatrix}$

## Observation Model

- 机器人从gps中获取xy的位置信息
- $\textbf{z}_{t} = H\textbf{x}_t$
- $\begin{aligned}
  \begin{bmatrix}
    x_{t+1}\\
    y_{t+1}
  \end{bmatrix}=
  \begin{bmatrix}
    1 & 0 & 0& 0\\
    0 & 1 & 0& 0\\
  \end{bmatrix}
  \begin{bmatrix}
    x_{t}\\
    y_{t}
  \end{bmatrix}
\end{aligned}$
- $J_H=\begin{bmatrix}
  \frac{dx}{dx}& \frac{dx}{dy} & \frac{dx}{d\phi} &  \frac{dx}{dv}\\
  \frac{dy}{dx}& \frac{dy}{dy} & \frac{dy}{d\phi} &  \frac{dy}{dv}\\
\end{bmatrix}=
\begin{bmatrix}
  1& 0 & 0 & 0\\
  0 & 1 & 0 & 0\\
\end{bmatrix}
$

## EKF
=== Predict ===

$x_{Pred} = Fx_t+Bu_t$

$P_{Pred} = J_FP_t J_F^T + Q$

=== Update ===

$z_{Pred} = Hx_{Pred}$ 

$y = z - z_{Pred}$

$S = J_H P_{Pred}.J_H^T + R$

$K = P_{Pred}.J_H^T S^{-1}$

$x_{t+1} = x_{Pred} + Ky$

$P_{t+1} = ( I - K J_H) P_{Pred}$

## [Code](codes/extended_kalman_filter.py)
- 蓝线是真实轨迹,
- 黑线是航位推测法(dead reckoning)得到的轨迹,用初始x真值及u输入代入运动模型迭代得到的轨迹
- 绿色点代表观测值 (ex. GPS)
- 红色线是用 EKF推测得到的轨迹,红色椭圆是EKF估计的方差.