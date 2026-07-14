# 第11章: EM算法及其推广 / EM Algorithm and Its Generalization

> **融合来源**: 李航第9章(EM算法及其推广) + 西瓜书第7章(贝叶斯分类器中GMM的EM) + 清华大学PPT

---

## 一、入门理解 (西瓜书仅简要提及，此处自行建立直观)

### 1.1 为什么需要EM算法？

当你有一袋硬币，想知道正面概率$\theta$，直接数就行——这是**完整的**MLE。但当数据有**隐变量**(latent variable)时——比如你只知道每次抛的是A、B、C三枚硬币之一，但不知道是哪枚——直接MLE变得困难。

**EM算法的直觉**：如果知道隐变量(哪枚硬币)，就能MLE；如果知道参数，就能推断隐变量。鸡生蛋、蛋生鸡——EM交替进行这两步，迭代逼近最优解。

### 1.2 简单例子：三硬币模型

三枚硬币A/B/C，正面概率$\pi, p, q$。先抛A决定用B还是C，再用B或C抛。观测到的只有最终结果(正面/反面)，不知道是哪枚硬币抛的(A的结果是隐变量)。

---

## 二、公式推导

### 2.1 EM算法的核心推导

目标是最大化不完全数据的对数似然：$\ell(\theta) = \log P(Y|\theta) = \log \sum_Z P(Y,Z|\theta)$

直接优化困难（对数内有求和）。**Jensen不等式**：对于凹函数$\log$，$\log \sum \lambda_i x_i \geq \sum \lambda_i \log x_i$（$\sum \lambda_i=1, \lambda_i \geq 0$）。

构造下界函数 $B(\theta, \theta^{(i)})$：

$$\begin{aligned}
\ell(\theta) &= \log \sum_Z P(Y,Z|\theta) = \log \sum_Z P(Z|Y,\theta^{(i)}) \frac{P(Y,Z|\theta)}{P(Z|Y,\theta^{(i)})} \\
&\geq \sum_Z P(Z|Y,\theta^{(i)}) \log \frac{P(Y,Z|\theta)}{P(Z|Y,\theta^{(i)})} \equiv B(\theta, \theta^{(i)})
\end{aligned}$$

最大化此下界 → **EM算法**：

- **E步**：计算 $Q(\theta, \theta^{(i)}) = \mathbb{E}_{Z|Y,\theta^{(i)}}[\log P(Y,Z|\theta)]$
- **M步**：$\theta^{(i+1)} = \arg\max_\theta Q(\theta, \theta^{(i)})$

### 2.2 收敛性定理

**定理9.1**：EM算法每次迭代后对数似然单调不减：$\ell(\theta^{(i+1)}) \geq \ell(\theta^{(i)})$

证明：$\ell(\theta^{(i+1)}) \geq B(\theta^{(i+1)}, \theta^{(i)}) \geq B(\theta^{(i)}, \theta^{(i)}) = \ell(\theta^{(i)})$，其中第一步为Jensen下界，第二步因M步最大化。

**定理9.2**：若$Q(\theta, \theta^{(i)})$连续，EM算法收敛到$\ell(\theta)$的稳定点（不一定是全局最优）。

### 2.3 三硬币模型的EM求解

- E步：$\mu_j^{(i+1)} = P(\text{抛B} | y_j, \theta^{(i)}) = \frac{\pi^{(i)} (p^{(i)})^{y_j} (1-p^{(i)})^{1-y_j}}{\pi^{(i)} (p^{(i)})^{y_j} (1-p^{(i)})^{1-y_j} + (1-\pi^{(i)}) (q^{(i)})^{y_j} (1-q^{(i)})^{1-y_j}}$
- M步：$\pi^{(i+1)} = \frac{1}{n}\sum_j \mu_j^{(i+1)}$, $p^{(i+1)} = \frac{\sum_j \mu_j^{(i+1)} y_j}{\sum_j \mu_j^{(i+1)}}$, $q^{(i+1)} = \frac{\sum_j (1-\mu_j^{(i+1)}) y_j}{\sum_j (1-\mu_j^{(i+1)})}$

### 2.4 GMM的EM推导

高斯混合模型：$P(\mathbf{x}|\theta) = \sum_{k=1}^{K} \alpha_k \mathcal{N}(\mathbf{x}|\boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k)$

E步（计算响应度/责任值）：$\gamma_{ik} = \frac{\alpha_k \mathcal{N}(\mathbf{x}_i|\boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k)}{\sum_{j=1}^{K} \alpha_j \mathcal{N}(\mathbf{x}_i|\boldsymbol{\mu}_j, \boldsymbol{\Sigma}_j)}$

M步（闭式更新）：$\boldsymbol{\mu}_k^{\text{new}} = \frac{\sum_i \gamma_{ik} \mathbf{x}_i}{\sum_i \gamma_{ik}}$, $\boldsymbol{\Sigma}_k^{\text{new}} = \frac{\sum_i \gamma_{ik} (\mathbf{x}_i-\boldsymbol{\mu}_k^{\text{new}})(\mathbf{x}_i-\boldsymbol{\mu}_k^{\text{new}})^T}{\sum_i \gamma_{ik}}$, $\alpha_k^{\text{new}} = \frac{\sum_i \gamma_{ik}}{m}$

---

## 三、理论深化 — 李航《机器学习方法》

### 3.1 EM算法的统计学习三要素

**模型**：含隐变量的概率模型 $P(Y,Z|\theta)$（如GMM、HMM、PLSA）

**策略**：不完全数据的极大似然估计 $\max_\theta \log P(Y|\theta)$

**算法**：EM（E步求期望→构造Q函数；M步最大化Q）。GEM(广义EM)当M步难解时，只需增大Q（不必最大化）。

### 3.2 EM的F函数解释

李航引入F函数：$F(\tilde{P}, \theta) = \mathbb{E}_{\tilde{P}}[\log P(Y,Z|\theta)] + H(\tilde{P})$

其中$\tilde{P}(Z)$是隐变量的任意分布，$H$是其熵。EM可以理解为F函数的坐标上升：E步固定$\theta$优化$\tilde{P}$，M步固定$\tilde{P}$优化$\theta$。

### 3.3 EM的应用全景

- **GMM的EM** → 软聚类
- **HMM的Baum-Welch** → 序列标注
- **k-means** → EM的硬分配退化版（$\gamma_{ik} \in \{0,1\}$，硬指派）
- **PLSA/LDA** → 话题建模
- **缺失数据填补** → 将缺失值视为隐变量

---

## 四、综合总结

### 关键结论

1. **EM = 隐变量的MLE通用框架**：Jensen不等式 → 下界 → 迭代优化
2. **E步算期望(软分配)，M步最大似然(参数更新)**
3. **单调收敛保证**：$\ell(\theta)$单调不减，但可能到局部最优
4. **初始值敏感**：不同初值→不同局部最优→多次随机初始化
5. **k-means是EM的硬分配特例**：$\gamma_{ik} \in \{0,1\}$ vs $(0,1)$
6. **GMM的EM有闭式解**，一般模型可能需要数值优化M步

### 章节连接

- **前置依赖**: 第9章(贝叶斯分类器—MLE/MAP)；概率论(Jensen不等式)
- **后续延伸**: 第12章(HMM)—Baum-Welch = HMM的EM；第17章(概率图模型)—含隐变量的图模型推断
