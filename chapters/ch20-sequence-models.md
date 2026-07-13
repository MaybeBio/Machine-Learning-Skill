# Chapter 20: 序列模型：HMM与CRF / Sequence Models: HMM and CRF

## 核心概念 (Core Idea)

隐马尔可夫模型 (HMM) 和条件随机场 (CRF) 是处理序列标注问题的两类核心概率模型。HMM 是**生成模型**，对观测序列和状态序列的联合分布 $P(X, Y)$ 建模，依赖两个基本假设（齐次马尔可夫性和观测独立性）。CRF 是**判别模型**，直接对条件分布 $P(Y|X)$ 建模，通过特征函数灵活引入上下文依赖，克服了 HMM 的强独立性假设限制。两者共享相同的推断算法框架（前向-后向、Viterbi），但在建模能力和灵活性上有本质区别。

---

## 第一部分：隐马尔可夫模型 (HMM)

### 理论基础

#### HMM 的定义

隐马尔可夫模型是关于时序的概率模型，描述由一个隐藏的马尔可夫链随机生成不可观测的状态序列，再由各个状态生成一个观测从而产生观测序列的过程。

设 $Q = \{q_1, q_2, \ldots, q_N\}$ 为所有可能的状态集合（$N$ 为状态数），$V = \{v_1, v_2, \ldots, v_M\}$ 为所有可能的观测集合（$M$ 为观测数）。$I = (i_1, i_2, \ldots, i_T)$ 为长度为 $T$ 的状态序列，$O = (o_1, o_2, \ldots, o_T)$ 为对应的观测序列。

HMM 由三元组 $\lambda = (A, B, \pi)$ 决定：

- **状态转移概率矩阵** $A = [a_{ij}]_{N \times N}$：
  $$a_{ij} = P(i_{t+1} = q_j | i_t = q_i), \quad i,j = 1,2,\ldots,N \tag{H1}$$

- **观测概率矩阵** $B = [b_j(k)]_{N \times M}$：
  $$b_j(k) = P(o_t = v_k | i_t = q_j), \quad j = 1,\ldots,N; \; k = 1,\ldots,M \tag{H2}$$

- **初始状态概率向量** $\pi = (\pi_i)$：
  $$\pi_i = P(i_1 = q_i), \quad i = 1,2,\ldots,N \tag{H3}$$

#### HMM 的两个基本假设

**(1) 齐次马尔可夫性假设**：隐藏的马尔可夫链在任意时刻 $t$ 的状态只依赖于前一时刻的状态：
$$P(i_t | i_{t-1}, o_{t-1}, \ldots, i_1, o_1) = P(i_t | i_{t-1}) \tag{H4}$$

**(2) 观测独立性假设**：任意时刻的观测只依赖于该时刻的马尔可夫链的状态：
$$P(o_t | i_T, o_T, \ldots, i_{t+1}, o_{t+1}, i_t, i_{t-1}, o_{t-1}, \ldots, i_1, o_1) = P(o_t | i_t) \tag{H5}$$

#### HMM 的三个基本问题

**(1) 概率计算问题**：给定 $\lambda = (A, B, \pi)$ 和观测序列 $O$，计算 $P(O|\lambda)$。

**(2) 学习问题**：已知观测序列 $O$，估计 $\lambda = (A, B, \pi)$ 使 $P(O|\lambda)$ 最大。

**(3) 预测（解码）问题**：已知 $\lambda$ 和 $O$，求最可能的状态序列 $I^* = \arg\max_I P(I|O)$。

### 理论推导

#### 前向-后向算法（解决问题1）

直接计算 $P(O|\lambda)$ 需要对所有 $N^T$ 种状态序列求和——$O(TN^T)$ 复杂度，不可行。

**前向概率**：定义 $\alpha_t(i) = P(o_1, o_2, \ldots, o_t, i_t = q_i | \lambda)$，表示到时刻 $t$ 的部分观测序列为 $o_1, \ldots, o_t$ 且状态为 $q_i$ 的概率。

递推公式：$\alpha_{t+1}(i) = \left[\sum_{j=1}^{N} \alpha_t(j) a_{ji}\right] b_i(o_{t+1})$

终止：$P(O|\lambda) = \sum_{i=1}^{N} \alpha_T(i)$

**后向概率**：定义 $\beta_t(i) = P(o_{t+1}, o_{t+2}, \ldots, o_T | i_t = q_i, \lambda)$，表示在时刻 $t$ 状态为 $q_i$ 的条件下，从 $t+1$ 到 $T$ 的部分观测序列的概率。

递推公式：$\beta_t(i) = \sum_{j=1}^{N} a_{ij} b_j(o_{t+1}) \beta_{t+1}(j)$

初值：$\beta_T(i) = 1$

终止：$P(O|\lambda) = \sum_{i=1}^{N} \pi_i b_i(o_1) \beta_1(i)$

复杂度：$O(N^2 T)$，远优于直接法的 $O(TN^T)$。

**统一的概率表示**：
$$P(O|\lambda) = \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_t(i) a_{ij} b_j(o_{t+1}) \beta_{t+1}(j), \quad t = 1, 2, \ldots, T-1 \tag{H6}$$

**关键中间量**：

- 时刻 $t$ 处于状态 $q_i$ 的概率：
  $$\gamma_t(i) = P(i_t = q_i | O, \lambda) = \frac{\alpha_t(i) \beta_t(i)}{\sum_{j=1}^{N} \alpha_t(j) \beta_t(j)} \tag{H7}$$

- 时刻 $t$ 处于 $q_i$ 且 $t+1$ 处于 $q_j$ 的概率：
  $$\xi_t(i, j) = \frac{\alpha_t(i) a_{ij} b_j(o_{t+1}) \beta_{t+1}(j)}{\sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_t(i) a_{ij} b_j(o_{t+1}) \beta_{t+1}(j)} \tag{H8}$$

#### Baum-Welch 算法（解决问题2 — EM 在 HMM 的实现）

当只有观测序列而未知状态序列时，HMM 的学习是一个含隐变量的概率模型参数估计问题，可用 EM 算法求解，其具体形式称为 **Baum-Welch 算法**。

完全数据的对数似然：$\log P(O, I|\lambda) = \log\left(\pi_{i_1} b_{i_1}(o_1) \prod_{t=2}^{T} a_{i_{t-1}i_t} b_{i_t}(o_t)\right)$

通过 E 步计算 $\gamma_t(i)$ 和 $\xi_t(i, j)$，M 步更新参数：

$$a_{ij}^{(n+1)} = \frac{\sum_{t=1}^{T-1} \xi_t(i, j)}{\sum_{t=1}^{T-1} \gamma_t(i)} \tag{H9}$$

$$b_j(k)^{(n+1)} = \frac{\sum_{t=1, o_t=v_k}^{T} \gamma_t(j)}{\sum_{t=1}^{T} \gamma_t(j)} \tag{H10}$$

$$\pi_i^{(n+1)} = \gamma_1(i) \tag{H11}$$

#### Viterbi 算法（解决问题3 — 动态规划最优路径）

定义 $\delta_t(i)$ 为时刻 $t$ 状态为 $i$ 的所有单个路径中概率最大值：

$$\delta_t(i) = \max_{i_1, i_2, \ldots, i_{t-1}} P(i_t = i, i_{t-1}, \ldots, i_1, o_t, \ldots, o_1 | \lambda)$$

递推公式：
$$\delta_{t+1}(i) = \max_{1 \leq j \leq N} [\delta_t(j) a_{ji}] \cdot b_i(o_{t+1}) \tag{H12}$$

定义 $\Psi_t(i)$ 记录最优路径的前驱状态：
$$\Psi_t(i) = \arg\max_{1 \leq j \leq N} [\delta_{t-1}(j) a_{ji}] \tag{H13}$$

算法步骤：
1. 初始化：$\delta_1(i) = \pi_i b_i(o_1), \Psi_1(i) = 0$
2. 递推：对 $t = 2, \ldots, T$，计算 $\delta_t(i)$ 和 $\Psi_t(i)$
3. 终止：$P^* = \max_i \delta_T(i), \; i_T^* = \arg\max_i \delta_T(i)$
4. 回溯：$i_t^* = \Psi_{t+1}(i_{t+1}^*), \; t = T-1, \ldots, 1$

**本质**：Viterbi 是动态规划——如果最优路径在时刻 $t$ 通过结点 $i_t^*$，那么从 $i_t^*$ 到终点 $i_T^*$ 的部分路径对所有可能的部分路径必然最优。

**Viterbi vs 前向算法的核心区别**：
- 前向算法：求和 $\sum_j$（边际化所有可能路径）
- Viterbi 算法：求最大 $\max_j$（仅保留最优路径）

---

## 第二部分：条件随机场 (CRF)

### 理论基础

#### 概率无向图模型（马尔可夫随机场）

**定义**：设有联合概率分布 $P(Y)$，由无向图 $G = (V, E)$ 表示。如果 $P(Y)$ 满足成对、局部或全局马尔可夫性（三者等价），就称其为概率无向图模型或马尔可夫随机场。

**因式分解定理 (Hammersley-Clifford)**：概率无向图模型的联合概率分布可以表示为最大团上的势函数的乘积形式：

$$P(Y) = \frac{1}{Z} \prod_C \Psi_C(Y_C) \tag{C1}$$

其中 $Z = \sum_Y \prod_C \Psi_C(Y_C)$ 是规范化因子，$\Psi_C(Y_C) = \exp(-E(Y_C))$ 是严格正的势函数。

#### 线性链条件随机场的定义

**定义**：设 $X = (X_1, \ldots, X_n), Y = (Y_1, \ldots, Y_n)$ 均为线性链随机变量序列。若在给定 $X$ 的条件下，$Y$ 的条件概率分布构成条件随机场，即满足：

$$P(Y_i | X, Y_1, \ldots, Y_{i-1}, Y_{i+1}, \ldots, Y_n) = P(Y_i | X, Y_{i-1}, Y_{i+1}) \tag{C2}$$

则称 $P(Y|X)$ 为线性链条件随机场。

**核心区别**：CRF 是**判别模型**，直接建模 $P(Y|X)$；不同于 HMM 建模 $P(X, Y)$。在标注问题中，$X$ 是输入观测序列，$Y$ 是输出标记序列。

#### 参数化形式（线性链 CRF 的核心公式）

$$P(y|x) = \frac{1}{Z(x)} \exp\left(\sum_{i,k} \lambda_k t_k(y_{i-1}, y_i, x, i) + \sum_{i,l} \mu_l s_l(y_i, x, i)\right) \tag{C3}$$

$$Z(x) = \sum_y \exp\left(\sum_{i,k} \lambda_k t_k(y_{i-1}, y_i, x, i) + \sum_{i,l} \mu_l s_l(y_i, x, i)\right) \tag{C4}$$

其中：
- $t_k$ 是**转移特征函数**（定义在边上，依赖 $y_{i-1}, y_i$）
- $s_l$ 是**状态特征函数**（定义在结点上，依赖 $y_i$）
- $\lambda_k, \mu_l$ 是对应的权值
- $Z(x)$ 是规范化因子（partition function）
- 特征函数通常取值为 1 或 0（满足条件时取 1）

CRF 本质上是对数线性模型 (log-linear model)。

#### 简化形式

将转移特征和状态特征统一为 $f_k$，对应的权值为 $w_k$：

$$P_w(y|x) = \frac{\exp(w \cdot F(y, x))}{Z_w(x)} \tag{C5}$$

其中 $F(y, x) = (f_1(y,x), \ldots, f_K(y,x))^T$ 为全局特征向量，$Z_w(x) = \sum_y \exp(w \cdot F(y, x))$。

#### 矩阵形式

定义每个位置 $i$ 的 $m \times m$ 矩阵（$m$ 为标记数）：

$$M_i(y_{i-1}, y_i|x) = \exp\left(\sum_{k=1}^{K} w_k f_k(y_{i-1}, y_i, x, i)\right) \tag{C6}$$

则：
$$P_w(y|x) = \frac{1}{Z_w(x)} \prod_{i=1}^{n+1} M_i(y_{i-1}, y_i|x) \tag{C7}$$
$$Z_w(x) = (M_1(x) M_2(x) \cdots M_{n+1}(x))_{\text{start, stop}} \tag{C8}$$

矩阵形式将路径概率的求和转化为矩阵连乘，极大简化了前向-后向计算。

### 理论推导

#### CRF 的前向-后向算法

定义前向向量 $\alpha_i(x)$ 和后向向量 $\beta_i(x)$：

$$\alpha_0(y|x) = \begin{cases} 1, & y = \text{start} \\ 0, & \text{否则} \end{cases}$$

$$\alpha_i^T(x) = \alpha_{i-1}^T(x) M_i(x) \tag{C9}$$

$$\beta_{n+1}(y|x) = \begin{cases} 1, & y = \text{stop} \\ 0, & \text{否则} \end{cases}$$

$$\beta_i(x) = M_{i+1}(x) \beta_{i+1}(x) \tag{C10}$$

**概率计算公式**：

$$P(Y_i = y_i | x) = \frac{\alpha_i^T(y_i|x) \beta_i(y_i|x)}{Z(x)} \tag{C11}$$

$$P(Y_{i-1} = y_{i-1}, Y_i = y_i | x) = \frac{\alpha_{i-1}^T(y_{i-1}|x) M_i(y_{i-1}, y_i|x) \beta_i(y_i|x)}{Z(x)} \tag{C12}$$

其中 $Z(x) = \alpha_n^T(x) \cdot \mathbf{1}$。

#### CRF 的参数学习

**目标**：极大化训练数据的对数似然函数：

$$L(w) = \sum_{x,y} \tilde{P}(x, y) \log P_w(y|x) = \sum_{j=1}^{N} \sum_{k=1}^{K} w_k f_k(y_j, x_j) - \sum_{j=1}^{N} \log Z_w(x_j)$$

**优化方法**：

1. **改进的迭代尺度法 (IIS)**：
   对转移特征 $t_k$，$\delta_k$ 的更新方程：
   $$\delta_k = \frac{1}{S} \log \frac{E_{\tilde{P}}[t_k]}{E_P[t_k]}$$
   其中 $S$ 是足够大的常数使松弛特征非负。$E_{\tilde{P}}$ 是经验期望，$E_P$ 是模型期望（通过前向-后向计算）。

2. **拟牛顿法 (BFGS)**：
   梯度函数：
   $$g(w) = \sum_{x,y} \tilde{P}(x) P_w(y|x) f(x, y) - E_{\tilde{P}}(f)$$
   使用 BFGS 近似 Hessian 矩阵的逆，实现超线性收敛——比 IIS 更快，是 CRF 学习的主流方法。

#### CRF 的 Viterbi 预测算法

预测问题化为求最大非规范化概率的最优路径：

$$y^* = \arg\max_y (w \cdot F(y, x)) = \arg\max_y \sum_{i=1}^{n} w \cdot F_i(y_{i-1}, y_i, x) \tag{C13}$$

递推公式（注意：与 HMM 不同，这里是**加法**而非乘法，因为 CRF 工作在 log 空间）：

$$\delta_{i}(l) = \max_{1 \leq j \leq m} \{\delta_{i-1}(j) + w \cdot F_i(y_{i-1}=j, y_i=l, x)\} \tag{C14}$$

$$\Psi_i(l) = \arg\max_{1 \leq j \leq m} \{\delta_{i-1}(j) + w \cdot F_i(y_{i-1}=j, y_i=l, x)\} \tag{C15}$$

终止时 $\max_y (w \cdot F(y, x)) = \max_j \delta_n(j)$，然后回溯得到最优路径。

---

## 第三部分：HMM vs CRF 深度比较

### 生成模型 vs 判别模型

| 维度 | HMM | 线性链 CRF |
|------|-----|-----------|
| **模型类型** | 生成模型—建模 $P(X,Y)$ | 判别模型—建模 $P(Y|X)$ |
| **核心假设** | 齐次马尔可夫性 + 观测独立性 | 只假设马尔可夫性，无观测独立性限制 |
| **特征灵活性** | 仅能使用当前观测 $o_t$ | 可使用任意位置的全序列特征 $x$ |
| **参数形式** | $A$ (转移), $B$ (发射), $\pi$ (初始) | 特征函数 $t_k, s_l$ + 权值 $w_k$ |
| **学习目标** | $\max \prod_t P(o_t|i_t) P(i_t|i_{t-1})$ | $\max P(y|x)$ 直接建模条件概率 |
| **归一化** | 局部归一化（每步概率和为1） | 全局归一化 $Z(x)$（所有路径） |
| **标签偏差** | 无（MEMM 有此问题） | 无—全局归一化解决标签偏差 |

### MEMM 与标签偏差问题

最大熵马尔可夫模型 (MEMM) 是 HMM 到 CRF 之间的过渡模型：在每个位置 $i$，用最大熵模型建模 $P(y_i | y_{i-1}, x)$。但是 MEMM 每步做**局部归一化**，导致**标签偏差** (label bias problem)：模型偏向选择后继状态数较少的状态，因为从该状态出发的概率分布熵小。

CRF 通过全局归一化 $Z(x)$ 解决标签偏差——所有路径一起竞争概率质量，不受局部约束。

### 关键公式对照

**概率计算**：
- HMM（前向）：$\alpha_{t+1}(i) = [\sum_j \alpha_t(j) a_{ji}] \cdot b_i(o_{t+1})$（乘法）
- CRF（前向）：$\alpha_i^T(x) = \alpha_{i-1}^T(x) M_i(x)$（矩阵乘法，元素为权重的指数）

**Viterbi 解码**：
- HMM：$\delta_{t+1}(i) = \max_j [\delta_t(j) a_{ji}] \cdot b_i(o_{t+1})$（概率相乘）
- CRF：$\delta_i(l) = \max_j \{\delta_{i-1}(j) + w \cdot F_i(j, l, x)\}$（log 空间相加）

---

## 核心概念 (Key Concepts)

- **马尔可夫链**：状态序列满足一阶马尔可夫性质 $P(i_t | i_{t-1}, \ldots, i_1) = P(i_t | i_{t-1})$
- **前向概率 $\alpha_t(i)$**：到时刻 $t$ 的部分观测 + 当前状态的联合概率
- **后向概率 $\beta_t(i)$**：当前状态条件下，未来部分观测的条件概率
- **$\gamma_t(i)$**：给定全观测序列，时刻 $t$ 处于状态 $i$ 的后验概率
- **$\xi_t(i, j)$**：给定全观测序列，时刻 $t$ 状态 $i$ 且 $t+1$ 状态 $j$ 的后验概率
- **势函数 (Potential Function)**：无向图中最大团上的非负函数，CRF 中为 $\exp(w \cdot F)$
- **特征函数 (Feature Function)**：$t_k$ (转移特征) 和 $s_l$ (状态特征)，通常为 {0,1} 指示函数
- **规范化因子 $Z(x)$**：所有可能标记序列的非规范化概率之和（分区函数）
- **全局归一化 vs 局部归一化**：CRF 对所有路径整体归一化，MEMM 每步局部归一化

## 常见误区 (Anti-patterns)

- **误区：HMM 和 CRF 可以用完全相同的特征**。HMM 只能使用当前时刻的观测 $o_t$ 作为特征（受观测独立性假设约束），CRF 可以使用整个输入序列 $x$ 的任何特征（包括前向和后向依赖），表达能力更强。
- **误区：HMM 的参数学习一定需要 Baum-Welch**。当有标注数据（状态序列已知）时，直接用频率估计参数即可——Baum-Welch 仅在无监督场景下使用。
- **误区：Viterbi 算法等价于在每个时刻贪心地选择最大 $\gamma_t(i)$ 的状态**。这个近似算法不能保证整体最优路径——可能产生概率为 0 的转移（$a_{ij}=0$），而 Viterbi 保证全局最优。
- **误区：CRF 的特征函数由算法自动学习**。特征函数需要人工设计（类似特征工程），算法只学习对应的权重。现在深度学习方法（如 BiLSTM-CRF）可自动学习特征。
- **误区：CRF 一定优于 HMM**。当数据量小、HMM 的假设大致成立时，HMM 作为生成模型（需建模更多因素）可能反而更好（生成模型在小样本下的有偏估计有时比判别模型的高方差估计更稳定）。

## 算法步骤 (Algorithm Steps)

### Viterbi 算法（通用框架，适用于 HMM 和 CRF）

1. **初始化**：计算 $t=1$ 时各状态的 $\delta_1(i)$，设置回溯指针 $\Psi_1(i) = 0$
2. **递推**：对 $t = 2, \ldots, T$：
   - HMM：$\delta_t(i) = \max_j [\delta_{t-1}(j) a_{ji}] \cdot b_i(o_t)$
   - CRF：$\delta_i(l) = \max_j \{\delta_{i-1}(j) + w \cdot F_i(j, l, x)\}$
   - 记录回溯指针 $\Psi_t(i) = \arg\max_j[...]$
3. **终止**：$P^* = \max_i \delta_T(i), i_T^* = \arg\max_i \delta_T(i)$
4. **回溯**：$i_t^* = \Psi_{t+1}(i_{t+1}^*)$ for $t = T-1, \ldots, 1$

### Baum-Welch 算法（HMM 的无监督学习）

1. 初始化模型参数 $\lambda^{(0)} = (A^{(0)}, B^{(0)}, \pi^{(0)})$
2. 递推 $n = 1, 2, \ldots$：
   - E 步：利用前向-后向算法计算 $\gamma_t(i)$ 和 $\xi_t(i, j)$
   - M 步：使用式 (H9)-(H11) 更新参数
3. 直到对数似然收敛

## 代码要点 (Code Essentials)

- **前向算法数值稳定性**：概率连乘导致下溢，使用 log 空间（log-sum-exp 技巧）。对于 HMM：$\log\alpha_{t+1}(i) = \text{logsumexp}_j(\log\alpha_t(j) + \log a_{ji}) + \log b_i(o_{t+1})$
- **Viterbi 的 log 空间**：HMM 中将所有概率取 log 后将乘法变加法，CRF 中天然是加法形式
- **CRF 特征模板**：定义特征函数时使用模板（如 `U00:%x[-1,0]` 表示前一个词、`B` 表示当前词和前一标记的组合），通过模板自动展开为具体特征
- **CRF 实现库**：`sklearn-crfsuite`（Python）、`CRF++`（C++）、`pytorch-crf`（与深度学习结合）
- **HMM 实现**：`hmmlearn` 库覆盖 GaussianHMM、MultinomialHMM；或手动实现三个核心算法
- **CRF 的正则化**：目标函数通常加 $L_2$ 正则项 $-\frac{1}{2\sigma^2}\|w\|^2$（等价于参数的高斯先验），防止过拟合

## 直观理解 (Intuition)

- **HMM 的"因果关系"**：HMM 假设状态序列是"因"，观测序列是"果"。先按马尔可夫链生成隐状态，再按发射分布生成观测。这种生成式思维很自然，但限制了特征使用。

- **CRF 的"倒置思维"**：CRF 不过问观测是如何生成的，只关心"给定观测时标记应该是什么"。这种判别式思维允许使用任意复杂的观测特征。

- **前向-后向的"消息传递"**：前向和后向概率本质上是在线性链上传递消息——前向传递过去到现在的信息，后向传递未来到现在信息，汇合后得到全局信息。

- **Viterbi 的"带备忘的搜索"**：Viterbi 不回退递归穷举所有路径，而是在每个时间步、每个状态只保留最优部分路径——利用最优子结构，将 $O(N^T)$ 降至 $O(N^2 T)$。

- **标签偏差的直观解释**：想象一个状态只有一条出路——在 MEMM 中无论观测是什么，该状态"必须"转去唯一后继（概率质量只能给这一条边）。CRF 的全局归一化避免了这个问题——如果某路径整体不合理，不论局部转移概率如何，全局得分都会很低。

## 关键结论 (Key Takeaways)

1. HMM 是生成模型（建模 $P(X,Y)$），CRF 是判别模型（建模 $P(Y|X)$），两者共享前向-后向和 Viterbi 算法框架
2. HMM 的两个强假设（马尔可夫性 + 观测独立性）限制了特征使用，CRF 通过特征函数灵活引入任意依赖，表达能力更强
3. 前向-后向算法通过动态规划将概率计算复杂度从 $O(TN^T)$ 降至 $O(N^2T)$
4. Viterbi 算法利用最优子结构求全局最优状态序列，不同于贪心的逐时刻最大 $\gamma$ 近似算法
5. CRF 的全局归一化 $Z(x)$ 解决了 MEMM 的标签偏差问题，是 CRF 性能优于 MEMM 的核心原因
6. Baum-Welch 算法是 EM 算法在 HMM 的具体实现——E 步用前向-后向计算期望，M 步极大化重估参数

## 章节连接 (Connects To)

- **前置**: Chapter 19 (EM 算法——Baum-Welch 是其特例), Chapter 6 (逻辑斯谛回归与最大熵——对数线性模型的基础), 概率图模型基础（有向图/无向图、条件独立性）
- **后续**: 深度学习序列模型 (RNN/LSTM/Transformer——端到端替代 HMM/CRF 的特征工程), Chapter 11 (条件随机场的结构化预测推广), 概率上下文无关文法 (PCFG—HMM 向树状结构的推广)
- **互补阅读**: 南瓜书对应推导（前向-后向概率的递推公式、Baum-Welch 的拉格朗日乘子法推导、CRF 矩阵形式与规范化的关系）
