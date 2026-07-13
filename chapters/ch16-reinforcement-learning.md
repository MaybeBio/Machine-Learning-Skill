# Chapter 16: 强化学习 / Reinforcement Learning

## 核心概念 (Core Idea)

强化学习让机器通过与环境交互、以试错方式学习最优策略——在状态 $x$ 下选择动作 $a$，观察转移后的状态和获得的奖赏，目标是最大化长期累积奖赏。其核心是"延迟奖赏"下的序贯决策，在探索与利用之间取得平衡。(西瓜书/南瓜书)

## 融合框架 (Integrated Frameworks)

强化学习的方法体系：

| 分类 | 代表算法 | 核心思想 | 来源 |
|------|---------|---------|------|
| 探索-利用 | $\epsilon$-贪心, Softmax | K-摇臂赌博机 | 西瓜书16.2 |
| 有模型学习 | 策略迭代, 值迭代 | 动态规划（需知 $P, R$） | 西瓜书16.3 |
| 免模型-MC | 同策略/异策略MC | 采样轨迹取平均 | 西瓜书16.4.1 |
| 免模型-TD | Sarsa, Q-Learning | 单步更新 + bootstrap | 西瓜书16.4.2 |
| 值函数近似 | 线性近似, DQN | 参数化 $V$ 或 $Q$ | 西瓜书16.5 |
| 策略梯度 | REINFORCE, Actor-Critic | 直接优化策略参数 | (延伸) |

## 理论推导 (Theoretical Derivation)

### 1. MDP 形式化

强化学习任务对应马尔可夫决策过程 (MDP)：$E = \langle X, A, P, R \rangle$（西瓜书）或五元组 $\langle S, A, P, r, \gamma \rangle$（清华PPT）。

- **状态空间** $X$: 环境的描述
- **动作空间** $A$: 机器可执行的动作
- **状态转移函数** $P: X \times A \times X \mapsto \mathbb{R}$: $P_{x \to x'}^a$ 表示在状态 $x$ 执行动作 $a$ 后转移到 $x'$ 的概率
- **奖赏函数** $R: X \times A \times X \mapsto \mathbb{R}$: $R_{x \to x'}^a$ 表示该转移获得的奖赏
- **折扣因子** $\gamma \in [0,1]$: 平衡近期与远期收益 (清华PPT)

**策略 (Policy) $\pi$**: 可以是确定性 $\pi: X \mapsto A$ 或随机性 $\pi: X \times A \mapsto \mathbb{R}$。

### 2. 值函数与Bellman方程

**状态值函数** $V^\pi(x)$ 和 **状态-动作值函数** $Q^\pi(x, a)$：(西瓜书)

$$
\begin{aligned}
V_T^\pi(x) &= \mathbb{E}_\pi\left[\frac{1}{T}\sum_{t=1}^{T} r_t \mid x_0 = x\right] \quad \text{(T步累积)} \\
V_\gamma^\pi(x) &= \mathbb{E}_\pi\left[\sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x\right] \quad \text{($\gamma$折扣累积)} \tag{16.5}
\end{aligned}
$$

$$
Q_\gamma^\pi(x, a) = \mathbb{E}_\pi\left[\sum_{t=0}^{\infty} \gamma^t r_{t+1} \mid x_0 = x, a_0 = a\right] \tag{16.6}
$$

**Bellman 递归方程**（西瓜书+南瓜书推导）：

对 $\gamma$ 折扣累积奖赏：

$$
V_\gamma^\pi(x) = \sum_{a \in A} \pi(x, a) \sum_{x' \in X} P_{x \to x'}^a \left(R_{x \to x'}^a + \gamma V_\gamma^\pi(x')\right) \tag{16.8}
$$

南瓜书详细推导：从期望定义出发，将无穷和拆分为即时奖赏 $r_1$ 加后续 $\sum_{t=1}^{\infty} \gamma^{t-1} r_{t+1}$，利用全概率展开（动作和状态转移）得递归式。

**最优Bellman方程**（策略空间无约束时）：(西瓜书)

$$
V^*(x) = \max_{a \in A} \sum_{x' \in X} P_{x \to x'}^a \left(R_{x \to x'}^a + \gamma V^*(x')\right) \tag{16.13}
$$

代入 $V^*(x) = \max_{a} Q^{\pi^*}(x, a)$ (16.14) 得：

$$
Q^*(x, a) = \sum_{x' \in X} P_{x \to x'}^a \left(R_{x \to x'}^a + \gamma \max_{a' \in A} Q^*(x', a')\right) \tag{16.15}
$$

### 3. K-摇臂赌博机：探索-利用窘境

**增量式均值更新**（西瓜书/南瓜书推导）：

$$
Q_n(k) = \frac{1}{n}\left((n-1) \times Q_{n-1}(k) + v_n\right) = Q_{n-1}(k) + \frac{1}{n}(v_n - Q_{n-1}(k)) \tag{16.2-16.3}
$$

南瓜书验证：$Q_n(k) = \frac{1}{n}\sum_{i=1}^{n} v_i = \frac{1}{n}\left(\sum_{i=1}^{n-1} v_i + v_n\right) = \frac{1}{n}\left((n-1)Q_{n-1}(k) + v_n\right)$。

两种折中策略：
- **$\epsilon$-贪心**: 以概率 $\epsilon$ 随机探索，以 $1-\epsilon$ 选择当前最优
- **Softmax (Boltzmann)**: $P(k) = \frac{e^{Q(k)/\tau}}{\sum_{i=1}^{K} e^{Q(i)/\tau}}$ (16.4) — $\tau$ 越小越贪心（利用），$\tau$ 越大越均匀（探索）

南瓜书说明：$P(k) \propto e^{Q(k)/\tau} \propto \frac{Q(k)}{\tau} \propto \frac{1}{\tau}$ — 当 $\tau$ 很大时所有动作几乎等概率。

### 4. 有模型学习：策略迭代与值迭代

**策略评估**：用 Bellman 方程迭代计算 $V^\pi$（图16.7）。

**策略改进**：$\pi'(x) = \arg\max_{a \in A} Q^\pi(x, a)$ (16.17)。

南瓜书推导策略改进的单调性 (16.16)：

$$
V^\pi(x) \leq Q^\pi(x, \pi'(x)) = \sum_{x'} P_{x \to x'}^{\pi'(x)} (R + \gamma V^\pi(x')) \leq \cdots = V^{\pi'}(x)
$$

即值函数对策略的每一点改进都是单调不减的。

**策略迭代 (Policy Iteration)**：评估→改进→评估→...→收敛（图16.8）

**值迭代 (Value Iteration)**：直接迭代 $V(x) = \max_{a} \sum_{x'} P_{x \to x'}^a (R + \gamma V(x'))$ (16.18)，收敛后提取策略（图16.9）

### 5. 免模型学习

**蒙特卡罗 (MC) 强化学习**：采样完整轨迹，用平均奖赏估计 $Q$ 值。

同策略 (On-policy)：评估和改进同一 $\epsilon$-贪心策略 $\pi^\epsilon$ (图16.10)。

异策略 (Off-policy)：用 $\pi'$（如 $\epsilon$-贪心）采样，通过重要性采样评估 $\pi$。

重要性权重：$\frac{P^\pi}{P^{\pi'}} = \prod_{i=0}^{T-1} \frac{\pi(x_i, a_i)}{\pi'(x_i, a_i)}$ (16.28)。南瓜书指出：若 $\pi$ 确定性而 $\pi'$ 为 $\epsilon$-贪心，则 $\pi(x_i, a_i)=1$，$\pi'(x_i, a_i)$ 为 $\frac{\epsilon}{|A|}$ 或 $1-\epsilon+\frac{\epsilon}{|A|}$。

**时序差分 (TD) 学习** — 结合MC和DP思想：(西瓜书)

Sarsa (同策略) 更新：(16.31)

$$
Q_{t+1}^\pi(x, a) = Q_t^\pi(x, a) + \alpha \left(R_{x \to x'}^a + \gamma Q_t^\pi(x', a') - Q_t^\pi(x, a)\right)
$$

南瓜书推导：将增量式更新中的 $r_{t+1}$ 替换为TD目标 $R_{x \to x'}^a + \gamma Q_t^\pi(x', a')$，这正是"bootstrap"思想的体现。

**Q-Learning (异策略)**：更新时用 $\max_{a'} Q(x', a')$ 而非实际采样 $a'$：

$$
Q(x, a) \leftarrow Q(x, a) + \alpha \left(r + \gamma \max_{a'} Q(x', a') - Q(x, a)\right)
$$

评估的是最优策略，执行的是 $\epsilon$-贪心策略。

### 6. 值函数近似

用参数化函数 $V_{\boldsymbol{\theta}}(\mathbf{x}) = \boldsymbol{\theta}^T \mathbf{x}$ 近似值函数，最小化均方误差：(西瓜书/南瓜书)

$$
E_{\boldsymbol{\theta}} = \mathbb{E}_{\mathbf{x} \sim \pi}\left[(V^\pi(\mathbf{x}) - V_{\boldsymbol{\theta}}(\mathbf{x}))^2\right] \tag{16.33}
$$

南瓜书推导梯度：

$$
-\frac{\partial E_{\boldsymbol{\theta}}}{\partial \boldsymbol{\theta}} = \mathbb{E}_{\mathbf{x} \sim \pi}\left[2(V^\pi(\mathbf{x}) - V_{\boldsymbol{\theta}}(\mathbf{x})) \frac{\partial V_{\boldsymbol{\theta}}(\mathbf{x})}{\partial \boldsymbol{\theta}}\right] = \mathbb{E}_{\mathbf{x} \sim \pi}\left[2(V^\pi(\mathbf{x}) - V_{\boldsymbol{\theta}}(\mathbf{x})) \mathbf{x}\right] \tag{16.34}
$$

## 核心概念 (Key Concepts)

- **马尔可夫决策过程 (MDP)**: 强化学习的形式化框架，核心是马尔可夫性——下一状态仅依赖当前状态和动作
- **策略 $\pi$**: 状态到动作的映射，是强化学习的最终产出
- **值函数 $V^\pi(x)$**: 从状态 $x$ 出发按策略 $\pi$ 执行的期望累积奖赏
- **Q函数 $Q^\pi(x, a)$**: 从状态 $x$ 出发先执行动作 $a$ 再按 $\pi$ 执行的期望累积奖赏
- **Bellman方程**: 值函数的递归关系，是动态规划求解MDP的数学基础
- **探索-利用窘境 (Exploration-Exploitation Dilemma)**: 强化学习的核心挑战——尝试新动作（探索）与选择已知最优（利用）的矛盾
- **Bootstrap**: 用估计值更新估计值（如TD学习的 $Q(x', a')$），是TD与MC的核心区别
- **同策略 (On-policy) vs 异策略 (Off-policy)**: 前者评估和改进同一策略（如Sarsa），后者用不同策略采样（如Q-Learning）
- **重要性采样 (Importance Sampling)**: 用分布 $q$ 的样本估计分布 $p$ 的期望：$\mathbb{E}_p[f] = \int q(x) \frac{p(x)}{q(x)} f(x) dx$

## 算法步骤 (Algorithm Steps)

### Q-Learning (图16.13)
1. 初始化 $Q(x, a) = 0, \forall x, a$
2. 设置起始状态 $x$
3. 对每一步：
   - 用 $\epsilon$-贪心从 $Q$ 选择动作 $a$
   - 执行 $a$，观察奖赏 $r$ 和新状态 $x'$
   - $Q(x, a) \leftarrow Q(x, a) + \alpha(r + \gamma \max_{a'} Q(x', a') - Q(x, a))$
   - $x \leftarrow x'$

### 策略迭代 (图16.8)
1. 初始化 $V(x) = 0$，随机策略 $\pi$
2. 循环：
   - **策略评估**: 迭代更新 $V(x) = \sum_a \pi(x,a) \sum_{x'} P_{x \to x'}^a (\frac{1}{t}R_{x \to x'}^a + \frac{t-1}{t}V(x'))$ 直至收敛
   - **策略改进**: $\pi'(x) = \arg\max_a Q(x,a)$ (由式16.10从 $V$ 计算 $Q$)
   - 若 $\pi' = \pi$ 则停止，否则 $\pi \leftarrow \pi'$

## 直观理解 (Intuition)

**种西瓜的比喻**（西瓜书）：种瓜从选种到收获需多步，每一步（浇水、施肥、除草）无法立刻知道对最终"好瓜"的贡献——可能瓜苗暂时茁壮但最终枯死。你只能多次种植，在反复试错中总结策略。这就是"延迟奖赏"的核心：当前动作的好坏需要长期验证。

**K-摇臂赌博机的直观**：赌场有 $K$ 台老虎机，每台的吐币概率未知。你想最大化收益——既要尝试新机器（探索），又要在已知好机器上多玩（利用）。全部探索则浪费机会，全部利用则可能错过真正最好的机器。

**Bellman方程的直觉**：当前状态的价值 = 立即获得的奖赏 + 折价后的未来价值。这就像下棋时判断局面：不仅要看当前能吃几个子，更要看这步棋导致未来10步的局面好坏。

**TD学习的"边走边学"**：MC要等一局结束才能总结（像复盘整局棋），TD每走一步就更新估计（像边下边反思"刚才那步走得如何"）。TD相当于在MC的"批处理"和DP的"一步前瞻"之间取折中。

## 常见误区 (Anti-patterns)

- **误区：强化学习就是有监督学习加上延迟标签**。形式上相似，但强化学习中动作会影响后续状态分布（非i.i.d.），且没有"正确答案"——只有好坏之别。探索时的错误动作也是学习的必要代价。

- **误区：Q-Learning学到的就是最优策略**。在表格情形下、所有状态-动作对都充分探索时Q-Learning收敛到最优。但函数近似的引入可能破坏收敛性——这是Deep RL面临的重大理论挑战。

- **误区：$\epsilon$ 固定不变就够了**。固定 $\epsilon$ 在后期仍以固定概率随机探索，浪费了已知信息。更好的做法是 $\epsilon$ 随训练递减（如 $\epsilon = 1/\sqrt{t}$），即GLIE (Greedy in the Limit with Infinite Exploration) 条件。

- **误区：策略迭代和值迭代总是等价的**。策略迭代要等到精确策略评估完成再改进，计算开销大；值迭代边评估边改进，通常更快达到近似最优。

## 代码要点 (Code Essentials)

```python
import numpy as np

# Q-Learning 核心实现 (表格型)
def q_learning(env, episodes=1000, alpha=0.1, gamma=0.99, epsilon=0.1):
    Q = np.zeros((env.n_states, env.n_actions))  # 初始化Q表

    for ep in range(episodes):
        state = env.reset()
        done = False
        while not done:
            # epsilon-贪心选择动作
            if np.random.random() < epsilon:
                action = np.random.randint(env.n_actions)
            else:
                action = np.argmax(Q[state])

            next_state, reward, done = env.step(action)

            # TD更新 (异策略)
            td_target = reward + gamma * np.max(Q[next_state])
            td_error = td_target - Q[state, action]
            Q[state, action] += alpha * td_error

            state = next_state
    return Q

# Softmax动作选择
def softmax_action(Q, state, tau=1.0):
    q_values = Q[state]
    exp_q = np.exp(q_values / tau)  # tau小→利用, tau大→探索
    probs = exp_q / np.sum(exp_q)
    return np.random.choice(len(q_values), p=probs)
```

关键实现注意：
1. 学习率 $\alpha$ 需随时间递减以保证收敛（满足Robbins-Monro条件）
2. Q-Learning中 $\max$ 操作会导致Q值系统性高估——Double Q-Learning通过解耦选择和评估来缓解
3. 经验回放 (Experience Replay) 是DQN成功的关键：打破样本时序相关性，提高数据利用率

## 实例分析 (Worked Example)

**给西瓜浇水的MDP**（西瓜书图16.2）：

状态空间：{健康, 缺水, 溢水, 凋亡}，动作空间：{浇水, 不浇水}。部分转移：
- 健康状态 + 浇水 → 溢水($p=0.5$, $r=-1$)或健康($p=0.5$, $r=1$)
- 健康状态 + 不浇水 → 缺水($p=0.5$, $r=-1$)或健康($p=0.5$, $r=1$)
- 凋亡状态奖赏 $-100$，无法恢复

最优策略：健康时浇水，溢水时不浇水，缺水时浇水。

**2-摇臂赌博机实验**（西瓜书图16.6）：摇臂1以0.4概率返1（$E=0.4$），摇臂2以0.2概率返1（$E=0.2$）。$\epsilon$-贪心和Softmax在不同温度参数下的累积奖赏曲线显示：Softmax($\tau=0.01$)退化为仅利用，而适度探索的参数能发现真正最优臂。

## 关键结论 (Key Takeaways)

1. **MDP是强化学习的数学心脏**：$E = \langle X, A, P, R \rangle$（或五元组），值函数和Bellman方程是整个理论体系的基石
2. **有模型 vs 免模型是首要二分**：有模型可用动态规划（策略/值迭代），免模型需要采样估计（MC/TD）
3. **TD是MC和DP的优雅结合**：既像MC通过采样克服模型未知，又像DP通过bootstrap实现单步高效更新
4. **Q-Learning是异策略TD的里程碑**：学习最优策略同时可用探索策略收集数据，是DQN的前身
5. **探索-利用是所有RL算法的共同挑战**：$\epsilon$-贪心和Softmax是经典解法，更深层的方案包括UCB、Thompson采样等
6. **值函数近似连接表格RL和深度RL**：线性近似到神经网络的跃迁（DQN）开启了深度强化学习时代

## 章节连接 (Connects To)

- **前置**: Ch2 模型评估 (偏差-方差权衡与探索-利用的类比), Ch6 SVM (最大间隔思想与RL中max操作的联系), Ch8 集成学习 (Bootstrap与MC方法的关系)
- **后续**: 深度强化学习 (DQN → Policy Gradient → Actor-Critic → PPO); 基于模型的RL (World Models); 多智能体RL; 逆强化学习
- **互补**: 与监督学习的关键差异——RL无标注样本、数据非i.i.d.、动作影响未来状态分布；与进化计算的交叉——遗传算法也可用于策略搜索但无MDP结构
