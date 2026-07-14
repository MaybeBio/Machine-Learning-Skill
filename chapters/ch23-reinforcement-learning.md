# Chapter 23: 强化学习 / Reinforcement Learning

> **导航**: 本章与第16章"强化学习"互补。第16章覆盖西瓜书/南瓜书的传统RL基础（MDP、Bellman方程、MC/TD、Sarsa/Q-Learning），本章在此基础上引入深度强化学习的系统化理论与现代方法体系。

---

## 一、入门理解 —— 周志华《机器学习》（西瓜书）

### 1.1 从种西瓜到强化学习

强化学习（Reinforcement Learning, RL）的核心思想可以用一个种西瓜的故事来理解：种瓜人面对一株瓜苗（环境的状态 $x$），可以做浇水、施肥、除虫等动作（动作 $a$），执行后瓜苗长到一个新状态并获得"长得好"或"枯萎"的反馈（奖赏 $r$）。种瓜人不知道土壤的真实情况，只能通过不断试错来摸索最优的种瓜策略——这就是"在交互中学习"的精髓。

与监督学习的本质区别在于：
- **无标签**：没有老师告诉你"在状态 $x$ 下应该做动作 $a$"，只有延迟的奖赏信号
- **序贯决策**：当前动作不仅影响即时奖赏，还影响后续所有状态和奖赏
- **探索-利用困境**：是选择已知的高奖赏动作（利用），还是尝试未探索的动作以可能发现更好的策略（探索）

### 1.2 K-摇臂赌博机与探索-利用困境

K-摇臂赌博机（K-armed Bandit）是理解探索-利用困境的最简模型：

- 有 $K$ 个摇臂，每次选择一个摇臂获得一个服从概率分布的奖赏
- 目标：在 $T$ 次尝试后最大化累积奖赏

**核心矛盾**：
- **仅探索**（纯探索）：将所有尝试机会平均分配到每个摇臂，可能错过了最优摇臂的尝试次数
- **仅利用**（纯利用）：按当前估计的最优摇臂一直选择，可能因初始估计不准而永远错过真正的最优

**$\epsilon$-贪心法**：
$$\text{以概率 } 1-\epsilon \text{ 选择当前估计最优的摇臂，以概率 } \epsilon \text{ 随机选择}$$

核心机制是：奖赏估计值以概率 $1-\epsilon$ 贪心选择（利用），以概率 $\epsilon$ 均匀随机选择（探索）。$\epsilon$ 取常数值如 $0.1$，或随时间衰减 $\epsilon_t = 1/\sqrt{t}$。

**Softmax法**：基于当前估计的奖赏均值，以Boltzmann分布分配选择概率：

$$P(a) = \frac{e^{Q(a)/\tau}}{\sum_{i=1}^{K} e^{Q(i)/\tau}}$$

其中 $\tau > 0$ 是温度参数。$\tau \to 0$ 趋向纯利用，$\tau \to \infty$ 趋向纯探索。

### 1.3 马尔可夫决策过程（MDP）

完整的强化学习任务对应**马尔可夫决策过程（MDP）**，由四元组定义：

$$E = \langle X, A, P, R \rangle$$

- **状态空间** $X$：所有可能的环境状态
- **动作空间** $A$：机器可执行的动作集合
- **状态转移函数** $P: X \times A \times X \mapsto \mathbb{R}$：$P_{x \to x'}^a$ 表示在状态 $x$ 执行动作 $a$ 后转移到 $x'$ 的概率，满足 $\sum_{x' \in X} P_{x \to x'}^a = 1$
- **奖赏函数** $R: X \times A \times X \mapsto \mathbb{R}$：$R_{x \to x'}^a$ 表示该转移所获的即时奖赏

**马尔可夫性质**：下一个状态 $x_{t+1}$ 仅依赖于当前状态 $x_t$ 和动作 $a_t$，与历史状态无关：

$$P(x_{t+1} \mid x_t, a_t, x_{t-1}, a_{t-1}, \ldots, x_0, a_0) = P(x_{t+1} \mid x_t, a_t)$$

### 1.4 策略与值函数

**策略（Policy）** $\pi: X \times A \mapsto [0, 1]$ 定义了在每个状态下选择各动作的概率分布。确定性策略可视为退化的随机策略。

**累积奖赏**有两种常用定义：
- **$T$步累积奖赏**：$V_T^\pi(x) = \mathbb{E}_\pi \left[ \frac{1}{T} \sum_{t=1}^{T} r_t \mid x_0 = x \right]$
- **$\gamma$折扣累积奖赏**：$V_\gamma^\pi(x) = \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x \right]$

**状态-动作值函数（Q函数）**：

$$Q_\gamma^\pi(x, a) = \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x, a_0 = a \right]$$

值函数 $V^\pi(x)$ 和 $Q^\pi(x, a)$ 的关系：

$$V^\pi(x) = \sum_{a \in A} \pi(x, a) Q^\pi(x, a)$$

### 1.5 Bellman方程

**Bellman期望方程**将值函数递归分解为即时奖赏和后续状态的值函数：

$$V^\pi(x) = \sum_{a \in A} \pi(x, a) \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma V^\pi(x') \right]$$

$$Q^\pi(x, a) = \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma \sum_{a' \in A} \pi(x', a') Q^\pi(x', a') \right]$$

**最优Bellman方程**：

$$V^*(x) = \max_{a \in A} \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma V^*(x') \right]$$

$$Q^*(x, a) = \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma \max_{a' \in A} Q^*(x', a') \right]$$

最优策略 $\pi^*$ 是在最优Q函数上取argmax得到：$\pi^*(x) = \arg\max_{a \in A} Q^*(x, a)$。

---

## 二、公式推导 —— 南瓜书补充

### 2.1 Bellman期望方程的推导

**状态值函数展开推导**：

从 $V^\pi(x)$ 的定义出发：

$$
\begin{aligned}
V^\pi(x) &= \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x \right] \\
&= \mathbb{E}_\pi \left[ r_1 + \gamma \sum_{t=0}^{\infty} \gamma^t r_{t+2} \mid x_0 = x \right] \\
&= \sum_{a \in A} \pi(x, a) \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+2} \mid x_1 = x' \right] \right] \\
&= \sum_{a \in A} \pi(x, a) \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma V^\pi(x') \right]
\end{aligned}
$$

其中第一步利用了期望的线性性质将 $t=0$ 项单独提出，第二步对动作 $a$ 和下一状态 $x'$ 做全概率展开。

**Q函数Bellman方程的类似推导**：

$$
\begin{aligned}
Q^\pi(x, a) &= \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x, a_0 = a \right] \\
&= \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+2} \mid x_1 = x' \right] \right] \\
&= \sum_{x' \in X} P_{x \to x'}^a \left[ R_{x \to x'}^a + \gamma \sum_{a' \in A} \pi(x', a') Q^\pi(x', a') \right]
\end{aligned}
$$

### 2.2 状态值函数与状态-动作值函数的关系证明

证明 $V^\pi(x) = \sum_{a \in A} \pi(x, a) Q^\pi(x, a)$：

$$
\begin{aligned}
V^\pi(x) &= \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x \right] \\
&= \mathbb{E}_{a \sim \pi(\cdot \mid x)} \left[ \mathbb{E}_\pi \left[ \sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x, a_0 = a \right] \right] \\
&= \sum_{a \in A} \pi(x, a) \, Q^\pi(x, a)
\end{aligned}
$$

### 2.3 TD学习的更新公式推导

**时序差分（TD）学习的核心思想**：用后继状态的值函数估计值来更新当前状态的值函数估计，即用 $r + \gamma V(s')$ 作为 $V(s)$ 的目标值：

$$V(s) \leftarrow V(s) + \alpha \left[ r + \gamma V(s') - V(s) \right]$$

其中 $r + \gamma V(s')$ 称为**TD目标**，$r + \gamma V(s') - V(s)$ 称为**TD误差**。

**Sarsa的Q函数更新**：

$$Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma Q(s', a') - Q(s, a) \right]$$

其中 $(s, a, r, s', a')$ 构成一个完整的经验五元组，$a'$ 由当前策略（如 $\epsilon$-贪心）在 $s'$ 处采样得到。Sarsa是**同策略（on-policy）**算法。

**Q-Learning的Q函数更新**：

$$Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma \max_{a'} Q(s', a') - Q(s, a) \right]$$

Q-Learning使用 $\max_{a'} Q(s', a')$ 作为目标，不依赖于行为策略在 $s'$ 处实际选择的动作，因此是**异策略（off-policy）**算法。目标策略为贪心策略，行为策略可以是 $\epsilon$-贪心。

### 2.4 价值函数近似的梯度推导

当状态空间很大或连续时，用参数化函数 $V(x; \theta)$ 或 $Q(x, a; \theta)$ 来近似值函数。以线性近似为例：

$$V(x; \theta) = \theta^{\mathrm{T}} \phi(x)$$

其中 $\phi(x)$ 是状态 $x$ 的特征向量。

**梯度下降更新**（最小化TD误差的平方）：

$$\theta \leftarrow \theta + \alpha \left[ r + \gamma V(x'; \theta) - V(x; \theta) \right] \nabla_\theta V(x; \theta)$$

对线性近似，$\nabla_\theta V(x; \theta) = \phi(x)$，更新简化为：

$$\theta \leftarrow \theta + \alpha \left[ r + \gamma \theta^{\mathrm{T}} \phi(x') - \theta^{\mathrm{T}} \phi(x) \right] \phi(x)$$

### 2.5 策略梯度定理的推导

设策略 $\pi_\theta(a \mid s)$ 由参数 $\theta$ 参数化，优化目标为起始状态的期望回报 $J(\theta) = V^{\pi_\theta}(s_0)$。

**策略梯度定理**：

$$\nabla_\theta J(\theta) \propto \sum_{s} \mu^{\pi_\theta}(s) \sum_{a} \nabla_\theta \pi_\theta(a \mid s) \, Q^{\pi_\theta}(s, a)$$

其中 $\mu^{\pi_\theta}(s)$ 是在策略 $\pi_\theta$ 下状态 $s$ 的访问分布。

利用恒等式 $\nabla_\theta \pi_\theta(a \mid s) = \pi_\theta(a \mid s) \, \nabla_\theta \ln \pi_\theta(a \mid s)$，可写为期望形式：

$$\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta} \left[ \nabla_\theta \ln \pi_\theta(a \mid s) \, Q^{\pi_\theta}(s, a) \right]$$

REINFORCE算法使用实际回报 $G_t$ 代替 $Q^{\pi_\theta}(s, a)$ 作为无偏估计：

$$\nabla_\theta J(\theta) \approx \nabla_\theta \ln \pi_\theta(a_t \mid s_t) \, G_t$$

### 2.6 重要性采样公式

在异策略学习中，行为策略 $\pi_b$ 生成的轨迹需用**重要性采样比率**修正才能评估目标策略 $\pi$：

$$\rho_t^{T} = \prod_{k=t}^{T} \frac{\pi(a_k \mid s_k)}{\pi_b(a_k \mid s_k)}$$

每次访问的加权重要性采样估计为：

$$V^\pi(s) \approx \frac{\sum_{t \in \mathcal{T}(s)} \rho_t^{T(t)} G_t}{\sum_{t \in \mathcal{T}(s)} \rho_t^{T(t)}}$$

重要性采样虽然无偏（普通重要性采样）或有偏但方差小（加权重要性采样），但当轨迹较长时方差会指数级增长。

---

## 三、理论深化 —— PPT + 深度强化学习

### 3.1 强化学习方法分类体系

清华PPT给出了系统的RL方法分类：

```
强化学习
├── 有模型学习（Model-based）
│   ├── 策略迭代（Policy Iteration）
│   │   ├── 策略评估：Bellman期望方程迭代求V^π
│   │   └── 策略改进：贪心更新 π'(s) = argmax Q^π(s,a)
│   └── 值迭代（Value Iteration）
│       └── Bellman最优方程迭代，直接收敛到V^*
└── 免模型学习（Model-free）
    ├── 基于值函数
    │   ├── 蒙特卡洛方法（MC）
    │   └── 时序差分方法（TD）
    │       ├── Sarsa（on-policy TD control）
    │       ├── Q-Learning（off-policy TD control）
    │       └── DQN及其变体
    └── 基于策略搜索
        ├── 策略梯度方法（REINFORCE, PPO, TRPO）
        └── Actor-Critic方法（A2C, A3C, SAC, TD3）
```

清华PPT中关于MDP的五元组形式化：
$$\langle S, A, P, r, \gamma \rangle$$

其中 $S$ 为有限状态集，$A$ 为有限动作集，$P$ 为状态转移概率 $P(s' \mid s, a)$，$r$ 为立即奖赏，$\gamma \in [0, 1]$ 为折扣因子。

### 3.2 有模型学习：动态规划方法

**策略迭代（Policy Iteration）**交替执行：

**(1) 策略评估**：对给定策略 $\pi$，用Bellman方程迭代求 $V^\pi$：

$$V_{k+1}(s) = \sum_{a \in A} \pi(a \mid s) \sum_{s' \in S} P(s' \mid s, a) \left[ r(s, a, s') + \gamma V_k(s') \right]$$

**(2) 策略改进**：基于 $V^\pi$ 求 $Q^\pi$，然后贪心构造新策略：

$$\pi'(s) = \arg\max_{a \in A} Q^\pi(s, a) = \arg\max_{a \in A} \sum_{s' \in S} P(s' \mid s, a) \left[ r(s, a, s') + \gamma V^\pi(s') \right]$$

可以证明 $\pi'$ 一致优于或等于 $\pi$（策略改进定理）。

**值迭代（Value Iteration）**合并评估和改进为一步：

$$V_{k+1}(s) = \max_{a \in A} \sum_{s' \in S} P(s' \mid s, a) \left[ r(s, a, s') + \gamma V_k(s') \right]$$

收敛后得到 $V^*$，再用一步argmax得到 $\pi^*$。

两种方法的对比：
| 特性 | 策略迭代 | 值迭代 |
|------|---------|--------|
| 每轮计算量 | 大（完整策略评估） | 小（一步更新） |
| 收敛轮数 | 少 | 多 |
| 中间结果 | 每次都是完整策略 | 值函数逐步改进 |

### 3.3 免模型学习：MC与TD

**蒙特卡洛（MC）方法**：采样完整轨迹，用实际累积回报更新值函数估计：

$$V(s) \leftarrow V(s) + \alpha \left[ G_t - V(s) \right]$$

其中 $G_t = \sum_{k=0}^{\infty} \gamma^k r_{t+k+1}$ 是从时间 $t$ 起的实际折扣回报。MC方法无偏但方差大，且需要轨迹结束才能更新（episodic任务）。

**时序差分（TD）方法**：在每一步后立即更新，使用当前估计引导（bootstrapping）：

$$V(s) \leftarrow V(s) + \alpha \left[ r + \gamma V(s') - V(s) \right]$$

TD方法有偏（由于bootstrapping）但方差小，且适用于连续任务。

### 3.4 深度Q网络（DQN）

DQN是用深层神经网络 $Q(s, a; \theta)$ 逼近动作值函数的里程碑式方法。

**两大核心创新**：

**(1) 经验回放（Experience Replay）**：将 $(s, a, r, s')$ 元组存入回放缓冲池 $\mathcal{D}$，训练时从中随机采样小批量数据，打破了样本间的时间相关性，提高了数据利用率：

$$\mathcal{L}(\theta) = \mathbb{E}_{(s, a, r, s') \sim \mathcal{D}} \left[ \left( r + \gamma \max_{a'} Q(s', a'; \theta^-) - Q(s, a; \theta) \right)^2 \right]$$

**(2) 目标网络（Target Network）**：用参数 $\theta^-$ 独立的目标网络计算TD目标，每 $C$ 步将 $\theta$ 复制到 $\theta^-$，防止目标值随训练网络一同变化导致的振荡和不稳定。

**完整DQN算法**：在每步与环境交互后，将经验存入 $\mathcal{D}$，从中采样小批量用梯度下降更新 $\theta$，周期性同步 $\theta^- \leftarrow \theta$。

### 3.5 策略梯度方法（REINFORCE）

与基于值函数的方法（隐式通过Q函数导出策略）不同，策略梯度方法直接将策略参数化为 $\pi_\theta(a \mid s)$ 并沿梯度方向优化。

**REINFORCE算法**对每个轨迹的每步执行更新：

$$\theta \leftarrow \theta + \alpha \gamma^t G_t \nabla_\theta \ln \pi_\theta(a_t \mid s_t)$$

其中 $G_t = \sum_{k=t}^{T} \gamma^{k-t} r_{k+1}$ 是从 $t$ 时刻起的实际折扣回报。

**引入基线（Baseline）降低方差**：将 $G_t$ 替换为 $G_t - b(s_t)$，常用状态值函数 $V(s_t)$ 作为基线。这不改变期望梯度方向但显著降低方差：

$$\nabla_\theta J(\theta) = \mathbb{E} \left[ \nabla_\theta \ln \pi_\theta(a \mid s) \left( Q^{\pi_\theta}(s, a) - V^{\pi_\theta}(s) \right) \right]$$

其中 $A^{\pi_\theta}(s, a) = Q^{\pi_\theta}(s, a) - V^{\pi_\theta}(s)$ 称为**优势函数（Advantage Function）**。

### 3.6 Actor-Critic方法

Actor-Critic架构将策略（Actor）与值函数（Critic）结合：

- **Actor**：参数化策略 $\pi_\theta(a \mid s)$，按策略梯度方向更新
- **Critic**：参数化值函数 $V_w(s)$ 或 $Q_w(s, a)$，用TD学习评估当前策略

Critic提供的TD误差 $\delta_t = r_{t+1} + \gamma V_w(s_{t+1}) - V_w(s_t)$ 作为优势函数的估计来更新Actor：

$$\theta \leftarrow \theta + \alpha_\theta \, \delta_t \, \nabla_\theta \ln \pi_\theta(a_t \mid s_t)$$
$$w \leftarrow w + \alpha_w \, \delta_t \, \nabla_w V_w(s_t)$$

A3C（Asynchronous Advantage Actor-Critic）并行运行多个Actor-Critic线程，异步更新共享参数，消除了经验回放的需求。A2C是其同步版本。PPO使用裁剪的目标函数保证策略更新不会太大，TRPO使用KL散度约束。

---

## 四、综合总结

### 4.1 核心算法对比

| 算法 | 类型 | 更新源 | 收敛性 | 适用场景 |
|------|------|--------|--------|---------|
| 策略迭代 | Model-based | 动态规划 | 每次迭代策略必改进 | 已知MDP模型 |
| 值迭代 | Model-based | 动态规划 | 收缩到最优值函数 | 已知MDP模型 |
| MC | Model-free | 完整轨迹 | 渐近收敛 | Episodic任务 |
| TD(0) | Model-free | 单步bootstrapping | 随机近似收敛 | Continuous任务 |
| Sarsa | On-policy TD | $(s,a,r,s',a')$ | 渐近收敛（GLIE条件） | On-policy控制 |
| Q-Learning | Off-policy TD | $\max_{a'} Q(s',a')$ | 表格型收敛到最优 | Off-policy控制 |
| DQN | Off-policy + DNN | 经验回放 | 经验性 | 高维状态空间 |
| REINFORCE | Policy Gradient | 轨迹回报 | 局部最优 | 连续动作空间 |
| Actor-Critic | Policy + Value | TD误差 | 局部最优 | 连续控制 |

### 4.2 关键结论

1. **探索-利用困境**是RL的根本挑战，$\epsilon$-贪心和Softmax是最基本的解决方案
2. **Bellman方程**将序贯决策分解为递归的子问题，是所有值函数方法的理论基础
3. **TD学习**是RL中最核心的学习机制，融合了MC的采样和DP的自举
4. **Q-Learning**的异策略特性使其可以从任意行为策略产生的数据中学习最优策略，这是DQN的理论基础
5. **DQN的三大创新**（经验回放、目标网络、深度网络逼近）解决了RL与深度学习结合的稳定性问题
6. **策略梯度方法**直接优化策略参数，避免了值函数方法中argmax在高维动作空间的计算困难
7. **Actor-Critic**融合了策略搜索和值函数逼近的优点，是现代深度RL的主流框架

### 4.3 章节连接

- **前向连接**：第16章覆盖RL的传统方法基础（MDP、Bellman方程、MC/TD、Sarsa/Q-Learning的表格型实现），本章扩展至深度RL体系
- **后向连接**：DQN中的深度网络设计与第22章（深度学习）和第25章（CNN，用于处理图像状态输入）紧密相关
- **横向连接**：RL与第13章（半监督学习）共享"利用未标注信息"的思想；策略梯度中的优化与一般神经网络的梯度下降有共同的理论基础
