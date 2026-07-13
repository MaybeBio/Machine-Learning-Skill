# Chapter 14: 概率图模型 / Probabilistic Graphical Models

## 核心概念 (Core Idea)

概率图模型用图结构编码随机变量间的条件依赖关系：节点表示变量，边表示概率关系。有向图（贝叶斯网）表达因果关系，无向图（马尔可夫随机场）表达相关关系。概率图模型的三部曲是：表示 (Representation)、推断 (Inference)、学习 (Learning)。(南瓜书)

## 融合框架 (Integrated Frameworks)

本章融合了概率图模型的核心体系：

| 主题 | 图类型 | 核心内容 | 来源 |
|------|--------|---------|------|
| PGM基础 | 有向/无向 | 贝叶斯网、MRF、因子分解 | 南瓜书14.0-14.2 |
| 隐马尔可夫模型 | 有向 | 前向后向、Viterbi、Baum-Welch | 南瓜书14.1, 李航P1-Ch10 |
| 条件随机场 | 无向 | 线性链CRF、特征函数 | 南瓜书14.3, 李航P1-Ch11 |
| 精确推断 | — | 变量消去、信念传播 | 南瓜书14.4 |
| 近似推断 | — | MCMC、变分推断 | 南瓜书14.5 |
| 话题模型 | 有向 | PLSA、LDA | 南瓜书14.6, 李航P2-Ch18/20 |

## 理论推导 (Theoretical Derivation)

### 1. PGM 表示论

**隐马尔可夫模型 (HMM)** — 有向图模型的典型代表。(南瓜书)

HMM的联合概率分解（基于条件独立性）：(南瓜书推导)

$$
\begin{aligned}
P(x_1, y_1, \ldots, x_n, y_n) &= P(x_1, \ldots, x_n \mid y_1, \ldots, y_n) \cdot P(y_1, \ldots, y_n) \\
&= \left(\prod_{i=1}^{n} P(x_i \mid y_i)\right) \cdot \left(P(y_1) \prod_{i=2}^{n} P(y_i \mid y_{i-1})\right) \\
&= P(y_1) P(x_1 \mid y_1) \prod_{i=2}^{n} P(y_i \mid y_{i-1}) P(x_i \mid y_i) \tag{14.1}
\end{aligned}
$$

HMM三组参数：$\lambda = [\mathbf{A}, \mathbf{B}, \boldsymbol{\pi}]$
- **状态转移概率** $\mathbf{A} = [a_{ij}]_{N \times N}$，其中 $a_{ij} = P(y_{t+1}=s_j \mid y_t=s_i)$
- **观测概率** $\mathbf{B} = [b_j(k)]_{N \times M}$，其中 $b_j(k) = P(x_t=v_k \mid y_t=s_j)$
- **初始状态概率** $\boldsymbol{\pi} = [\pi_i]$，其中 $\pi_i = P(y_1=s_i)$

**马尔可夫随机场 (MRF)** — 无向图模型。(南瓜书)

联合概率基于极大团分解（Hammersley-Clifford定理）：

$$
P(\mathbf{x}) = \frac{1}{Z} \prod_{Q \in \mathcal{C}} \psi_Q(\mathbf{x}_Q) \tag{14.3}
$$

其中 $\mathcal{C}$ 为极大团集合，$\psi_Q$ 为势函数，$Z = \sum_{\mathbf{x}} \prod_{Q \in \mathcal{C}} \psi_Q(\mathbf{x}_Q)$ 为配分函数。

势函数通常定义为指数形式：(南瓜书)

$$
\psi_Q(\mathbf{x}_Q) = e^{-H_Q(\mathbf{x}_Q)} \tag{14.8}
$$

其中 $H_Q(\mathbf{x}_Q) = \sum_{u,v \in Q, u \neq v} \alpha_{uv} x_u x_v + \sum_{v \in Q} \beta_v x_v$ 为能量函数。(14.9)

三种等价的马尔可夫性：
- **成对马尔可夫性**: 非邻接变量条件独立于其余变量
- **局部马尔可夫性**: 给定邻接变量（马尔可夫毯），变量条件独立于其他变量
- **全局马尔可夫性**: 给定分离集，两组变量条件独立

**条件随机场 (CRF)** — 给定观测 $\mathbf{x}$ 条件下输出 $\mathbf{y}$ 的MRF。(南瓜书)

CRF定义为：$P(y_v \mid \mathbf{x}, \mathbf{y}_{V \setminus \{v\}}) = P(y_v \mid \mathbf{x}, \mathbf{y}_{n(v)})$，即满足局部马尔可夫性。(14.10)

线性链CRF的条件概率：

$$
P(\mathbf{y} \mid \mathbf{x}) = \frac{1}{Z(\mathbf{x})} \exp\left(\sum_{t,k} \lambda_k t_k(y_{t-1}, y_t, \mathbf{x}, t) + \sum_{t,l} \mu_l s_l(y_t, \mathbf{x}, t)\right) \tag{14.11}
$$

其中 $t_k$ 为转移特征函数（刻画相邻标记关系），$s_l$ 为状态特征函数（刻画标记与观测关系），$\lambda_k, \mu_l$ 为对应权重。

### 2. 精确推断

**变量消去 (Variable Elimination)**：(南瓜书)

基于条件独立性的边际化：$P(x_5) = \sum_{x_1} \sum_{x_2} \sum_{x_3} \sum_{x_4} P(x_1) P(x_2 \mid x_1) P(x_3 \mid x_2) P(x_4 \mid x_3) P(x_5 \mid x_3)$

引入消息传递记号 $m_{ij}(x_j)$ 表示从节点 $i$ 到 $j$ 的消息：(南瓜书)

$$
m_{ji}(x_i) = \sum_{x_j} \psi(x_i, x_j) \prod_{k \in n(j) \setminus i} m_{kj}(x_j) \tag{14.19}
$$

**信念传播 (Belief Propagation)**：两阶段消息传递过程——从叶向根收集消息，再从根向叶分发消息。当图为树结构时，信念传播给出精确边缘概率。

### 3. 近似推断

**MCMC采样** — 已知分布 $p(\mathbf{x})$，构造马尔可夫链使其平稳分布为 $p(\mathbf{x})$，通过采样估计期望：(南瓜书)

$$
\mathbb{E}[f] \approx \frac{1}{N} \sum_{i=1}^{N} f(\mathbf{x}_i) \tag{14.22}
$$

**细致平稳条件 (Detailed Balance)**：(南瓜书推导)

$$
p(\mathbf{x}^{t-1}) T(\mathbf{x}^{t} \mid \mathbf{x}^{t-1}) = p(\mathbf{x}^{t}) T(\mathbf{x}^{t-1} \mid \mathbf{x}^{t}) \tag{14.26}
$$

证明：$\sum_i \pi_i T_{ij} = \sum_i \pi_j T_{ji} = \pi_j \sum_i T_{ji} = \pi_j$，即 $\boldsymbol{\pi} \mathbf{T} = \boldsymbol{\pi}$。

**Metropolis-Hastings 算法**：引入提议分布 $Q$ 和接受概率 $A$，(南瓜书)

$$
A(\mathbf{x}^* \mid \mathbf{x}^{t-1}) = \min\left(1, \frac{p(\mathbf{x}^*) Q(\mathbf{x}^{t-1} \mid \mathbf{x}^*)}{p(\mathbf{x}^{t-1}) Q(\mathbf{x}^* \mid \mathbf{x}^{t-1})}\right) \tag{14.28}
$$

南瓜书给出接受概率的详细构造推导：令 $A(\mathbf{x}^* \mid \mathbf{x}^{t-1}) = C \cdot p(\mathbf{x}^*) Q(\mathbf{x}^{t-1} \mid \mathbf{x}^*)$，为使概率最大且不超过1，取 $C = \min(\frac{1}{p(\mathbf{x}^*)Q(\mathbf{x}^{t-1}\mid\mathbf{x}^*)}, \frac{1}{p(\mathbf{x}^{t-1})Q(\mathbf{x}^*\mid\mathbf{x}^{t-1})})$ 即得 (14.28)。

**Gibbs采样** — MH的特例（接受概率恒为1）：(南瓜书推导)

依次对每个变量 $x_i$ 按条件概率 $p(x_i \mid \mathbf{x}_{\bar{i}})$ 采样。代入MH接受概率得：

$$
\frac{p(\mathbf{x}^*) Q(\mathbf{x}^{t-1} \mid \mathbf{x}^*)}{p(\mathbf{x}^{t-1}) Q(\mathbf{x}^* \mid \mathbf{x}^{t-1})} = \frac{p(x_i^* \mid \mathbf{x}_{\bar{i}}^*) p(\mathbf{x}_{\bar{i}}^*) p(x_i^{t-1} \mid \mathbf{x}_{\bar{i}}^*)}{p(x_i^{t-1} \mid \mathbf{x}_{\bar{i}}^{t-1}) p(\mathbf{x}_{\bar{i}}^{t-1}) p(x_i^* \mid \mathbf{x}_{\bar{i}}^{t-1})} = 1
$$

**变分推断** — 用简单分布 $q(\mathbf{z})$ 近似复杂后验 $p(\mathbf{z} \mid \mathbf{x})$：(南瓜书推导)

$$
\ln p(\mathbf{x}) = \underbrace{\int q(\mathbf{z}) \ln \frac{p(\mathbf{x}, \mathbf{z})}{q(\mathbf{z})} d\mathbf{z}}_{\text{ELBO } \mathcal{L}(q)} + \underbrace{\left(-\int q(\mathbf{z}) \ln \frac{p(\mathbf{z} \mid \mathbf{x})}{q(\mathbf{z})} d\mathbf{z}\right)}_{\text{KL}(q \| p)} \tag{14.32-14.34}
$$

由于 $\ln p(\mathbf{x})$ 是常数，最小化KL等价于最大化ELBO。假定 $q(\mathbf{z}) = \prod_{i=1}^{M} q_i(\mathbf{z}_i)$（平均场假设），(14.35)。通过拉格朗日变分法得最优解：

$$
\ln q_j^*(\mathbf{z}_j) = \mathbb{E}_{i \neq j}[\ln p(\mathbf{x}, \mathbf{z})] + \text{const} \tag{14.39}
$$

归一化后得 $q_j^*(\mathbf{z}_j) = \frac{\exp(\mathbb{E}_{i \neq j}[\ln p(\mathbf{x}, \mathbf{z})])}{\int \exp(\mathbb{E}_{i \neq j}[\ln p(\mathbf{x}, \mathbf{z})]) d\mathbf{z}_j}$。(14.40)

### 4. 话题模型 — LDA

LDA的生成过程（三层贝叶斯模型）：(南瓜书)

$$
p(W, \mathbf{z}, \beta, \boldsymbol{\theta} \mid \alpha, \eta) = \prod_{t=1}^{T} p(\boldsymbol{\theta}_t \mid \alpha) \prod_{k=1}^{K} p(\beta_k \mid \eta) \left(\prod_{n=1}^{N} P(w_{t,n} \mid z_{t,n}, \beta_k) P(z_{t,n} \mid \boldsymbol{\theta}_t)\right) \tag{14.41}
$$

三个独立层次：
1. $T$ 篇文档的话题分布 $\boldsymbol{\theta}_t \sim \text{Dir}(\alpha)$ — 相互独立
2. $K$ 个话题的单词分布 $\beta_k \sim \text{Dir}(\eta)$ — 相互独立
3. 每篇文档中 $N$ 个单词的生成：先采样话题 $z_{t,n}$，再采样单词 $w_{t,n}$

对数似然：(南瓜书)

$$
LL(W \mid \alpha, \eta) = \sum_{t=1}^{T} \ln p(\mathbf{w}_t \mid \alpha, \eta) \tag{14.43}
$$

其中 $p(\mathbf{w}_t \mid \alpha, \eta) = \iiint p(\mathbf{w}_t, \mathbf{z}, \beta, \Theta \mid \alpha, \eta) d\mathbf{z} d\beta d\Theta$ 需要边际化隐变量，一般用Gibbs采样或变分EM近似求解。

## 核心概念 (Key Concepts)

- **贝叶斯网 (Bayesian Network)**: 有向无环图，边表示因果关系
- **马尔可夫随机场 (MRF)**: 无向图，边表示相关关系，由极大团势函数定义联合分布
- **马尔可夫毯 (Markov Blanket)**: 某变量的父节点、子节点及子节点的其他父节点，给定后该变量条件独立
- **条件随机场 (CRF)**: 给定观测条件下的MRF，是判别式模型，避免了对观测变量建模
- **配分函数 (Partition Function)** $Z$: 无向图中确保概率归一化的常数，计算困难是推断的瓶颈
- **消息传递 (Message Passing)**: 图上局部信息交换的推断机制
- **ELBO (Evidence Lower Bound)**: 对数证据的下界，变分推断的优化目标
- **细致平稳条件**: MCMC收敛到目标分布的充要条件
- **话题 (Topic)**: LDA中单词分布的多项式参数，表示一个语义概念
- **狄利克雷分布**: 多项分布的共轭先验，用于LDA中生成话题和单词分布

## 算法步骤 (Algorithm Steps)

### HMM三大基本问题及算法
1. **概率计算** (Forward-Backward): 计算 $P(\mathbf{x} \mid \lambda)$
   - 前向: $\alpha_t(i) = P(x_1,\ldots,x_t, y_t=s_i \mid \lambda)$，递推 $\alpha_{t+1}(j) = b_j(x_{t+1})\sum_{i=1}^{N} \alpha_t(i) a_{ij}$
2. **解码** (Viterbi): 寻找最可能状态序列 $\mathbf{y}^*$
3. **学习** (Baum-Welch / EM): 估计 $\lambda = [A, B, \pi]$

### MH算法步骤
1. 初始化 $\mathbf{x}^0$
2. 对 $t = 1, 2, \ldots$:
   - 从提议分布 $Q(\mathbf{x}^* \mid \mathbf{x}^{t-1})$ 采样候选 $\mathbf{x}^*$
   - 计算接受概率 $A = \min(1, \frac{p(\mathbf{x}^*) Q(\mathbf{x}^{t-1} \mid \mathbf{x}^*)}{p(\mathbf{x}^{t-1}) Q(\mathbf{x}^* \mid \mathbf{x}^{t-1})})$
   - 以概率 $A$ 接受 $\mathbf{x}^t = \mathbf{x}^*$，否则 $\mathbf{x}^t = \mathbf{x}^{t-1}$
3. 丢弃burn-in期样本后，用剩余样本估计统计量

## 直观理解 (Intuition)

**PGM是概率论的"电路图"**：就像电路图用连线表示元件间的电气连接，概率图用边表示变量间的概率依赖——有向边像"因果关系线"（哪个变量生成哪个），无向边像"相关关系线"（两个变量相互影响）。

**HMM是"猜天气"游戏**：你在室内只能看到地面是否湿（观测），但想知道外面是晴天还是下雨（隐状态）。地面湿了可能是下雨，也可能是有人洒水——你需要综合"晴天转下雨的概率"和"下雨时地面湿的概率"来推断。

**MCMC是"聪明的随机游走"**：想在崎岖的概率地形上采样，不是盲目乱走，而是在高概率区域多逗留（高接受率），低概率区域少去（低接受率），最终走过的路径分布恰好等于目标分布。

**变分推断是"用简单函数逼近复杂函数"**：真实后验像一条不规则曲线，变分推断找一个简单的（如可分解的高斯）去近似它，通过不断调整简单分布使KL散度最小化。

## 常见误区 (Anti-patterns)

- **误区：有向图和无向图只是画法不同**。有向图天然有因果语义（生成式模型），无向图只能表达对称的相关关系（判别式模型常使用）。例如HMM是有向生成模型，CRF是无向判别模型。

- **误区：MCMC采样的样本都是有效的**。MCMC样本是相关的（自相关），需要足够长的burn-in期丢弃非平稳样本，且有效样本量可能远小于采样数。

- **误区：精确推断总是可行的**。仅在树结构（或低树宽）图上可行；一般图的推断是#P-hard，必须使用近似方法。

- **误区：LDA的话题数可以自动确定**。LDA中话题数 $K$ 需要人工指定；非参数贝叶斯方法（如HDP）可以自动推断 $K$。

## 代码要点 (Code Essentials)

```python
# HMM前向算法 (数值稳定版本 - 对数域)
def forward(pi, A, B, observations):
    N = len(pi)  # 状态数
    T = len(observations)
    log_alpha = np.zeros((T, N))
    # 初始化
    log_alpha[0] = np.log(pi) + np.log(B[:, observations[0]])
    # 递推
    for t in range(1, T):
        for j in range(N):
            log_alpha[t, j] = np.log(B[j, observations[t]]) + \
                np.logaddexp.reduce(log_alpha[t-1] + np.log(A[:, j]))
    return log_alpha

# Metropolis-Hastings核心
def metropolis_hastings(p_target, q_proposal, q_sample, x_init, n_iter):
    samples = [x_init]
    for t in range(n_iter):
        x_star = q_sample(samples[-1])
        alpha = min(1, p_target(x_star) * q_proposal(samples[-1], x_star) /
                        (p_target(samples[-1]) * q_proposal(x_star, samples[-1])))
        if np.random.random() < alpha:
            samples.append(x_star)
        else:
            samples.append(samples[-1])
    return np.array(samples)
```

关键实现注意：
1. HMM的前向后向算法必须使用对数域计算，否则长序列会导致数值下溢
2. MCMC的提议分布选择至关重要：太窄则混合慢，太宽则接受率低（建议目标接受率23-50%）
3. LDA实现通常使用折叠Gibbs采样，只需采样话题分配 $z$

## 实例分析 (Worked Example)

**变量消去实例**（南瓜书，图14.7）：计算 $P(x_5)$：
- $m_{12}(x_2) = \sum_{x_1} P(x_1) P(x_2 \mid x_1) = P(x_2)$
- $m_{23}(x_3) = \sum_{x_2} P(x_3 \mid x_2) m_{12}(x_2) = P(x_3)$
- $m_{43}(x_3) = \sum_{x_4} P(x_4 \mid x_3) = 1$
- $m_{35}(x_5) = \sum_{x_3} P(x_5 \mid x_3) m_{23}(x_3) m_{43}(x_3) = P(x_5)$

**LDA话题示例**：给定一组网页，设 $K=3$，学得话题1高频词为{深度学习，神经网络，CNN}，话题2为{市场，投资，股票}，话题3为{世界杯，进球，球队}。每篇文章的话题混合比例解释了其内容组成。

## 关键结论 (Key Takeaways)

1. **概率图模型 = 概率论 + 图论**：节点编码随机变量，边编码条件独立性，图结构决定了联合分布的因子分解形式
2. **有向 vs 无向 = 生成 vs 判别**：HMM是生成式（建模 $P(X,Y)$），CRF是判别式（建模 $P(Y|X)$）
3. **HMM三问题三算法**：概率计算→前向后向，解码→Viterbi，学习→Baum-Welch (EM)
4. **MCMC的精髓是构造正确的转移核**：通过细致平稳条件确保马尔可夫链的平稳分布为目标分布
5. **变分推断用优化替代采样**：将推断转化为ELBO最大化，通常比MCMC快但引入近似误差
6. **LDA是三层贝叶斯模型**：文档-话题-单词，Dirichlet-Multinomial共轭使得Gibbs采样高效

## 章节连接 (Connects To)

- **前置**: Ch7 贝叶斯分类器 (朴素贝叶斯、EM算法), Ch13 半监督学习 (生成式方法的EM推广)
- **后续**: 概率图模型的深入学习包括结构化预测（CRF的应用）、深度生成模型（VAE基于变分推断）、概率编程语言
- **互补**: 与深度学习的关系——图神经网络 (GNN) 可以看作PGM与深度学习的结合点
