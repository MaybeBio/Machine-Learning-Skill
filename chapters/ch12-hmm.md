# 第12章: 隐马尔可夫模型 / Hidden Markov Model

> **融合来源**: 李航第10章(隐马尔可夫模型) + 西瓜书第14章(概率图模型) + 清华大学PPT

---

## 一、入门理解

### 1.1 什么是HMM？

HMM描述由一个隐藏的马尔可夫链随机生成不可观测的状态序列，再由各个状态生成观测序列的过程。两个关键假设：

(1) **齐次马尔可夫假设**：$P(i_t|i_{t-1}, \ldots, i_1) = P(i_t|i_{t-1})$ — 当前状态仅与前一状态有关
(2) **观测独立假设**：$P(o_t|i_t, \ldots, i_1, o_{t-1}, \ldots, o_1) = P(o_t|i_t)$ — 观测仅由当前状态决定

经典例子：盒子与球——有4个盒子，每个盒子有不同比例的红/白球。随机选盒子、抽球、放回，你只能看到球的颜色(观测)，不能看到选了哪个盒子(状态/隐变量)。

### 1.2 三要素

HMM由三个参数决定：$\lambda = (A, B, \pi)$
- **初始状态概率向量** $\pi$: $\pi_i = P(i_1 = q_i)$
- **状态转移概率矩阵** $A$: $a_{ij} = P(i_{t+1}=q_j | i_t=q_i)$
- **观测概率矩阵** $B$: $b_j(k) = P(o_t = v_k | i_t = q_j)$

### 1.3 三个基本问题

1. **评估问题**（概率计算）：给定$\lambda$和观测$O$，计算$P(O|\lambda)$ → 前向-后向算法
2. **解码问题**（预测）：给定$\lambda$和$O$，求最优状态序列$I^*$ → Viterbi算法
3. **学习问题**（训练）：给定$O$，估计$\lambda$使$P(O|\lambda)$最大 → Baum-Welch算法(EM)

---

## 二、公式推导

### 2.1 前向-后向算法（评估问题）

暴力计算 $P(O|\lambda) = \sum_I P(O|I,\lambda)P(I|\lambda)$ 的复杂度为 $O(TN^T)$（$T$时刻，$N$状态），不可行。

**前向概率** $\alpha_t(i) = P(o_1, \ldots, o_t, i_t = q_i | \lambda)$

递推：$\alpha_{t+1}(i) = \left[\sum_{j=1}^{N} \alpha_t(j) a_{ji}\right] b_i(o_{t+1})$

初始化：$\alpha_1(i) = \pi_i b_i(o_1)$

终止：$P(O|\lambda) = \sum_{i=1}^{N} \alpha_T(i)$

**后向概率** $\beta_t(i) = P(o_{t+1}, \ldots, o_T | i_t = q_i, \lambda)$

递推：$\beta_t(i) = \sum_{j=1}^{N} a_{ij} b_j(o_{t+1}) \beta_{t+1}(j)$

前向-后向算法复杂度 $O(TN^2)$。

### 2.2 计算关键中间变量

- $\gamma_t(i) = P(i_t=q_i|O,\lambda) = \frac{\alpha_t(i)\beta_t(i)}{P(O|\lambda)}$ — 时刻$t$处于状态$i$的概率
- $\xi_t(i,j) = P(i_t=q_i, i_{t+1}=q_j|O,\lambda) = \frac{\alpha_t(i)a_{ij}b_j(o_{t+1})\beta_{t+1}(j)}{P(O|\lambda)}$ — 时刻$t$从$i$转移到$j$的概率

### 2.3 Viterbi算法（解码问题）

求使$P(I|O,\lambda)$最大的状态序列（等价于求最短路径）。

定义 $\delta_t(i) = \max_{i_1,\ldots,i_{t-1}} P(i_t=q_i, i_{t-1},\ldots,i_1, o_t,\ldots,o_1|\lambda)$

递推：$\delta_{t+1}(i) = \max_{1 \leq j \leq N} [\delta_t(j) a_{ji}] b_i(o_{t+1})$

记路径：$\psi_{t+1}(i) = \arg\max_j [\delta_t(j) a_{ji}]$

最优路径通过回溯$\psi$得到。Viterbi本质上是动态规划。

### 2.4 Baum-Welch算法（学习问题 = HMM的EM）

Baum-Welch是EM算法在HMM中的具体实现。

E步：用当前参数$\bar{\lambda}$计算$\gamma_t(i)$和$\xi_t(i,j)$

M步：更新参数
$$a_{ij} = \frac{\sum_{t=1}^{T-1} \xi_t(i,j)}{\sum_{t=1}^{T-1} \gamma_t(i)}, \quad b_j(k) = \frac{\sum_{t=1,o_t=v_k}^{T} \gamma_t(j)}{\sum_{t=1}^{T} \gamma_t(j)}, \quad \pi_i = \gamma_1(i)$$

---

## 三、理论深化 — 李航《机器学习方法》

### 3.1 统计学习三要素

**模型**：$P(O,I|\lambda) = \pi_{i_1} b_{i_1}(o_1) \prod_{t=2}^{T} a_{i_{t-1}i_t} b_{i_t}(o_t)$ — 生成模型，建模联合分布

**策略**：极大似然估计$\max_\lambda P(O|\lambda)$（不完全数据MLE，含隐状态序列$I$）

**算法**：Baum-Welch（EM特例）。前向-后向 → 计算期望 → 更新参数。

### 3.2 HMM实际应用

- **中文分词/词性标注**：状态=词性，观测=词语
- **语音识别**：状态=音素，观测=声学特征
- **基因序列分析**：状态=基因元件，观测=碱基序列
- **输入法**：状态=意图词，观测=拼音输入

PPT补充：HMM是**生成模型**→需要枚举所有可能的观测。相比CRF的判别式直接建模$P(I|O)$，HMM需要更强的观测独立性假设。

---

## 四、综合总结

### 关键结论

1. **HMM = 双重随机过程**：隐马尔可夫链(状态) + 观测生成(输出)
2. **三个算法**：前向-后向(计算概率)、Viterbi(最优路径)、Baum-Welch(参数学习)
3. **Viterbi = 动态规划**：$O(TN^2)$，回溯得到最优状态序列
4. **Baum-Welch = EM**：在HMM中E步计算$\gamma$和$\xi$，M步更新$A,B,\pi$
5. **HMM是生成模型**：对$P(O,I)$建模，隐含了强独立性假设
6. **观测独立性假设是HMM的主要限制** → CRF（第13章）通过特征函数克服

### 章节连接

- **前置依赖**: 第11章(EM算法)—Baum-Welch=EM的HMM特例；概率论(马尔可夫链)
- **后续延伸**: 第13章(CRF)—从生成式到判别式的进化；第17章(概率图模型)—HMM是有向时序图模型特例
