## Partially Observable Markov decision processes
- $(\mathcal{S}, \mathcal{A}, \mathcal{O}, T, R, O, \gamma)$
  - $\mathcal{S}$:状态空间
  - $\mathcal{A}$:动作空间
  - $\mathcal{O}$:观测空间
  - $T$:状态转移概率
  - $R$:奖励方程
  - $O$:观测方程
  - $\gamma$:折扣因子
- 代理在状态$s\in S$时,有$T\left(s, a, s^{\prime}\right)=\operatorname{Pr}\left(s^{\prime} \mid s, a\right)$概率采取动作$a\in \mathcal{A}$,采取动作后进入状态$s^{\prime}$。此时,环境给予代理一个奖励值$R\left(s, a, s^{\prime}\right)$。
- 代理不能直接得到状态值,而是通过观测方程$O(o, s, a)=\operatorname{Pr}(o \mid s, a)$判断状态值。$b(s)$:判断状态为$s$的置信度
- $Q^{\pi}(b, a)$:在置信状态为b时采取a动作,然后遵循策略$\pi$的**累计**折扣奖励

## Reinforcement Learning
- Bellman equation:
  - $Q^{*}(o, a)=R(o, a)+\gamma \sum_{o^{\prime}} T\left(o, a, o^{\prime}\right) \max _{a} Q^{*}\left(o^{\prime}, a\right)$
- 采用神经网络拟合$Q^{*}(o, a)$,loss
  - $J(\theta)=\mathbb{E}_{o^{\prime}}\left[\left(r+\gamma \max _{a^{\prime}} Q\left(o^{\prime}, a^{\prime} ; \theta\right)-Q(o, a ; \theta)\right)^{2}\right]$

### 状态量
- $s_{t}^{i}=\left(x_{t}^{i}, v_{t}^{i}, a_{t}^{i}, c_{t}^{i}\right)$
  - c:在主车道的车是否进行减速避让
  - $\mathcal{s}_{t}^{e}$:自动驾驶车的状态,没有c
- 环境状态:$s_{t}=\left(s_{t}^{e}, s_{t}^{1}, \ldots, s_{t}^{n_{t}}\right)$
  
### 观测量
- 假设没有噪声
- 获取的状态量为相邻的4辆车
- **将匝道的车映射到主干道**

### 动作值
- $a_{t}=a_{t+1}+\Delta a$
- 动作量
  - $\Delta a=\left\{-1 \mathrm{m} / \mathrm{s}^{2},-0.5 \mathrm{m} / \mathrm{s}^{2}, 0 \mathrm{m} / \mathrm{s}^{2}, 0.5 \mathrm{m} / \mathrm{s}^{2}, 1 \mathrm{m} / \mathrm{s}^{2}\right\}$
  - $a_{t}=0 \mathrm{m} / \mathrm{s}^{2},-4 \mathrm{m} / \mathrm{s}^{2}$

### 奖励值
- 碰撞-1
- 到达目的点+1