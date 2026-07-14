# 第20章: 计算学习理论 / Computational Learning Theory

> **融合来源**: 西瓜书第12章(计算学习理论) + 南瓜书第12章 + 清华大学PPT(统计学习方法概论中的泛化误差界) + 李航第1章(泛化误差上界)

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 计算学习理论的基本问题

计算学习理论(computational learning theory)研究机器学习的理论基础，目的是分析学习任务的困难本质，为学习算法提供理论保证。最核心的是**概率近似正确**(Probably Approximately Correct, PAC)学习理论。

泛化误差：$E(h; \mathcal{D}) = P_{\mathbf{x} \sim \mathcal{D}}(h(\mathbf{x}) \neq y)$
经验误差：$\widehat{E}(h; D) = \frac{1}{m}\sum_{i=1}^{m} \mathbb{I}(h(\mathbf{x}_i) \neq y_i)$

核心问题：经验误差与泛化误差之间的逼近程度如何？

### 1.2 关键不等式

西瓜书列出三个基础不等式：

- **Jensen不等式**：对凸函数 $f$，有 $f(\mathbb{E}(x)) \leq \mathbb{E}(f(x))$
- **Hoeffding不等式**：$m$ 个独立随机变量 $x_i \in [0,1]$，$P(|\frac{1}{m}\sum x_i - \frac{1}{m}\sum \mathbb{E}(x_i)| \geq \epsilon) \leq 2\exp(-2m\epsilon^2)$
- **McDiarmid不等式**：有界差异函数的集中不等式

### 1.3 PAC学习

**PAC辨识**：若存在学习算法 $\mathfrak{L}$，其输出假设 $h \in \mathcal{H}$ 满足 $P(E(h) \leq \epsilon) \geq 1 - \delta$，则称能PAC辨识概念类 $\mathcal{C}$。

**PAC可学习**：若存在学习算法 $\mathfrak{L}$ 和多项式函数 $\text{poly}(\cdot,\cdot,\cdot,\cdot)$，使当样本数 $m \geq \text{poly}(1/\epsilon, 1/\delta, \text{size}(\mathbf{x}), \text{size}(c))$ 时能PAC辨识 $\mathcal{C}$，则 $\mathcal{C}$ 是PAC可学习的。

**PAC学习算法**：若算法 $\mathfrak{L}$ 的运行时间也是多项式函数，则为高效PAC可学习。

### 1.4 有限假设空间

**可分情形**（$c \in \mathcal{H}$）：只需保留与训练集一致的假设。样本复杂度：
$$m \geq \frac{1}{\epsilon}\left(\ln|\mathcal{H}| + \ln\frac{1}{\delta}\right)$$

**不可分情形**（$c \notin \mathcal{H}$）：引出**不可知PAC学习**，以H中泛化误差最小的假设 $\arg\min_{h \in \mathcal{H}} E(h)$ 为近似目标。

泛化误差界（H为有限假设空间）：
$$P\left(|E(h) - \widehat{E}(h)| \leq \sqrt{\frac{\ln|\mathcal{H}| + \ln(2/\delta)}{2m}}\right) \geq 1 - \delta$$

### 1.5 VC维

VC维(Vapnik-Chervonenkis dimension)是无限假设空间的复杂度度量。

**定义**：假设空间 $\mathcal{H}$ 的VC维是能被 $\mathcal{H}$ 打散(shatter)的最大样本集的大小。"打散"意味着H中的假设能将这些样本按所有 $2^m$ 种方式分开。

**泛化误差界**（基于VC维）：
$$E(h) \leq \widehat{E}(h) + \sqrt{\frac{h(\log(2m/h) + 1) - \log(\delta/4)}{m}}$$
其中 $h$ 为VC维。VC维越大，模型容量越高，所需样本越多。

**实例**：
- 实数轴上的区间：VC维 = 2
- d维线性分类器（含偏置）：VC维 = d+1
- 正弦函数的VC维：无穷大

### 1.6 Rademacher复杂度与稳定性

**Rademacher复杂度**：比VC维更紧的泛化误差界工具，通过随机标签扰动度量假设空间的表达能力。

**稳定性**(stability)：算法输出受单个训练样本改变的影响程度。"稳定的学习算法泛化性能好"——算法越稳定，泛化误差界越紧。

---

## 二、公式推导 — 南瓜书补充

### 2.1 可分情形泛化误差界的完整推导

南瓜书补全了式(12.14)的推导链：

1. 泛化误差 $> \epsilon$ 的假设在训练集上表现完美的概率：$P < (1-\epsilon)^m$
2. 对 $|\mathcal{H}|$ 个假设用Union Bound：$P < |\mathcal{H}|(1-\epsilon)^m$
3. 利用不等式 $1-x \leq e^{-x}$：$P < |\mathcal{H}|e^{-m\epsilon}$
4. 令 $P \leq \delta$ 解出 $m$：$m \geq \frac{1}{\epsilon}(\ln|\mathcal{H}| + \ln\frac{1}{\delta})$

### 2.2 VC维泛化误差界的推导

南瓜书给出基于VC维的泛化误差界推导。关键引理是**生长函数**(growth function) $\Pi_{\mathcal{H}}(m)$（H在m个样本上最多能给出的不同标记结果数）与VC维的关系。

**Sauer引理**：若VC维为 $h$，则 $\Pi_{\mathcal{H}}(m) \leq \sum_{i=0}^{h} \binom{m}{i} \leq \left(\frac{em}{h}\right)^h$

代入有限假设空间的泛化误差界，将 $|\mathcal{H}|$ 替换为生长函数的上界，即得VC维泛化误差界。

### 2.3 Rademacher复杂度推导

南瓜书推导了基于Rademacher复杂度的泛化误差界。经验Rademacher复杂度：
$$\widehat{\mathfrak{R}}_D(\mathcal{H}) = \mathbb{E}_{\boldsymbol{\sigma}}\left[\sup_{h \in \mathcal{H}} \frac{1}{m}\sum_{i=1}^{m} \sigma_i h(\mathbf{x}_i)\right]$$
其中 $\sigma_i$ 为均匀分布的 ±1 Rademacher随机变量。Rademacher复杂度越大的假设空间越容易过拟合。

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 泛化误差上界（清华大学PPT + 李航第1章）

PPT给出了统计学习理论的核心结论——**泛化误差上界**：

对于二分类问题，假设空间 $\mathcal{H}$ 有限，则对任意 $f \in \mathcal{H}$，以至少 $1-\delta$ 的概率：

$$R(f) \leq \hat{R}(f) + \sqrt{\frac{\log|\mathcal{H}| + \log(1/\delta)}{2m}}$$

泛化误差界由两部分组成：
1. **经验风险** $\hat{R}(f)$：训练集上的表现
2. **置信范围**：随 $m$ 增大而减小，随 $|\mathcal{H}|$ 增大而增大

### 3.2 结构风险最小化(SRM)的理论依据

PPT阐述了SRM的理论依据——泛化误差界自然地导向SRM：

$$\min_f \hat{R}(f) + \sqrt{\frac{\log|\mathcal{H}|}{2m}}$$

即模型选择应同时考虑经验风险和模型复杂度（假设空间容量）。这正是第1章所述"结构风险最小化"的统计学基础。

### 3.3 统计学习理论的核心洞察

- **一致性**(consistency)：当 $m \to \infty$ 时，经验风险最小化解的泛化误差趋于贝叶斯最优误差
- **收敛速率**：误差的收敛速度为 $O(1/\sqrt{m})$（有限H）或 $O(\sqrt{h/m})$（VC维为h）
- **模型选择**：给定样本量m，存在最优的模型复杂度（VC维），过于简单或复杂都会导致泛化性能下降
- **没有免费的午餐定理的理论解释**：在所有可能问题上期望性能相同 = 归纳偏置的必要性

---

## 四、综合总结

### 理论工具对比

| 复杂度度量 | 适用范围 | 紧度 | 可计算性 |
|-----------|---------|------|---------|
| $|\mathcal{H}|$ | 有限假设空间 | 松（Union Bound） | 容易 |
| VC维 | 二分类无限假设空间 | 中等 | 可计算（部分模型） |
| Rademacher复杂度 | 任意假设空间 | 较紧 | 可数据依赖估计 |
| 算法稳定性 | 特定算法 | 较紧 | 可分析 |

### 关键结论

1. **PAC学习是计算学习理论的基础框架**：以"概率高"和"近似正确"刻画可学习性
2. **样本复杂度随假设空间容量增加**：$m \propto \ln|\mathcal{H}|$（有限H）或 $m \propto \text{VC维}$（无限H）
3. **泛化误差界 = 经验风险 + 置信范围**：置信范围随m增大而减小，随H增大而增大
4. **VC维是二分类模型的核心复杂度度量**：d维线性分类器的VC维 ≈ d+1
5. **不可知学习是PAC的推广**：当 $c \notin \mathcal{H}$ 时，目标是逼近H中最好的假设
6. **泛化误差界为SRM提供了理论正当性**：模型选择 = 经验风险 + 复杂度惩罚
7. **计算学习理论揭示了学习的基本限制**：NFL定理、无免费午餐、偏差-方差权衡

### 章节连接

- **前置依赖**：ch01(绪论)—NFL定理、ERM/SRM、偏差-方差概念；ch02(评估)—泛化误差与经验误差的区别
- **后续延伸**：ch06(SVM)—最大间隔分类器的理论保证与VC维；ch08(集成)—Boosting的泛化误差界分析

