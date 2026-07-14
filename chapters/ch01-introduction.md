# 第1章: 绪论与统计学习概论 / Introduction to Machine Learning & Statistical Learning

> **融合来源**: 西瓜书第1章(绪论) + 南瓜书第1章 + 李航第1章(机器学习及监督学习概论) + 李航第13章(无监督学习概论) + 清华大学PPT第1章(统计学习方法概论)

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 什么是机器学习？

机器学习致力于研究如何通过计算的手段，利用经验来改善系统自身的性能。在计算机系统中，"经验"通常以"数据"形式存在，因此机器学习的核心是**从数据中产生"模型"的算法**，即"学习算法"。

Mitchell (1997) 的定义：假设用 $P$ 来评估计算机程序在某任务类 $T$ 上的性能，若一个程序通过利用经验 $E$ 在 $T$ 中任务上获得了性能改善，则称该程序对 $E$ 进行了学习。

### 1.2 基本术语（以西瓜数据集为例）

| 编号 | 色泽 | 根蒂 | 敲声 | 好瓜 |
|------|------|------|------|------|
| 1 | 青绿 | 蜷缩 | 浊响 | 是 |
| 2 | 乌黑 | 蜷缩 | 浊响 | 是 |
| 3 | 青绿 | 硬挺 | 清脆 | 否 |
| 4 | 乌黑 | 稍蜷 | 沉闷 | 否 |

- **数据集**(data set)：记录的集合
- **示例/样本**(instance/sample)：每条记录，$\mathbf{x}_i \in \mathcal{X}$
- **属性/特征**(attribute/feature)：反映对象性质的事项（色泽、根蒂、敲声）
- **属性空间/样本空间**：属性张成的空间，$\mathcal{X} \subseteq \mathbb{R}^d$
- **特征向量**：$\mathbf{x}_i = (x_{i1}; x_{i2}; \ldots; x_{id})$
- **标记**(label)：示例的结果信息，$y_i \in \mathcal{Y}$
- **样例**(example)：拥有标记的示例，$(\mathbf{x}_i, y_i)$
- **训练集**(training set)：训练样本的集合

预测离散值 → **分类**(classification)；预测连续值 → **回归**(regression)；无标记数据分组 → **聚类**(clustering)。学得模型适用于新样本的能力称为**泛化**(generalization)能力。通常假设样本独立同分布于未知分布 $\mathcal{D}$。

### 1.3 假设空间、版本空间与归纳偏好

学习过程可看作在所有**假设**(hypothesis)组成的空间中搜索与训练集"匹配"的假设。例如色泽有3种取值+通配符"\*"，根蒂和敲声也类似，加上空假设 $\emptyset$，假设空间规模约为 $4 \times 3 \times 3 + 1 = 37$（或李航的 $4 \times 4 \times 4 + 1 = 65$，取决于具体取值数量）。

所有与训练集一致的假设构成**版本空间**(version space)。版本空间中有多个假设时，学习算法必须根据自身的**归纳偏好**(inductive bias)做出选择。

**奥卡姆剃刀**(Occam's Razor)：若有多个假设与观察一致，选最简单的那个。但"简单"的定义并非平凡。

### 1.4 没有免费的午餐定理 (NFL)

**No Free Lunch Theorem** (Wolpert, 1996; Wolpert and Macready, 1995)：在所有可能的问题上（目标函数 $f$ 均匀分布），所有学习算法的期望性能相同。

$$\sum_f E_{ote}(\mathfrak{L}_a | X, f) = \sum_f E_{ote}(\mathfrak{L}_b | X, f)$$

**核心寓意**：脱离具体问题，空泛地谈论"什么学习算法更好"毫无意义。算法的优劣必须针对具体学习问题评估——归纳偏好与问题是否匹配，往往起决定性作用。

---

## 二、公式推导 — 南瓜书补充

### 2.1 NFL定理的完整推导

南瓜书对西瓜书式(1.1)的推导做了详细展开。

设样本空间 $\mathcal{X}$ 和假设空间 $\mathcal{H}$ 均离散，$P(h|X, \mathfrak{L}_a)$ 为算法 $\mathfrak{L}_a$ 基于训练集 $X$ 产生假设 $h$ 的概率。二分类问题中 $f: \mathcal{X} \mapsto \{0,1\}$。

算法 $\mathfrak{L}_a$ 的训练集外误差：

$$E_{ote}(\mathfrak{L}_a | X, f) = \sum_h \sum_{\mathbf{x} \in \mathcal{X}-X} P(\mathbf{x}) \cdot \mathbb{I}(h(\mathbf{x}) \neq f(\mathbf{x})) \cdot P(h | X, \mathfrak{L}_a)$$

对所有 $f$ 按均匀分布求和：

$$\begin{aligned}
\sum_f E_{ote}(\mathfrak{L}_a | X, f) &= \sum_f \sum_h \sum_{\mathbf{x} \in \mathcal{X}-X} P(\mathbf{x}) \cdot \mathbb{I}(h(\mathbf{x}) \neq f(\mathbf{x})) \cdot P(h | X, \mathfrak{L}_a) \\
&= \sum_{\mathbf{x} \in \mathcal{X}-X} P(\mathbf{x}) \sum_h P(h | X, \mathfrak{L}_a) \cdot \frac{1}{2} \cdot 2^{|\mathcal{X}|} \\
&= 2^{|\mathcal{X}|-1} \sum_{\mathbf{x} \in \mathcal{X}-X} P(\mathbf{x})
\end{aligned}$$

关键步骤：$\sum_f \mathbb{I}(h(\mathbf{x}) \neq f(\mathbf{x})) = \frac{1}{2} \cdot 2^{|\mathcal{X}|}$（$f$ 均匀分布时有一半在 $\mathbf{x}$ 处的预测与 $h(\mathbf{x})$ 不一致）。结果与 $\mathfrak{L}_a$ 完全无关。

### 2.2 监督学习的形式化定义

南瓜书给出监督学习的形式化框架：

给定训练集 $T = \{(\mathbf{x}_1, y_1), \ldots, (\mathbf{x}_m, y_m)\}$，其中 $(\mathbf{x}_i, y_i) \sim P(\mathbf{x}, y)$ i.i.d.

**期望风险**：$R_{exp}(f) = \mathbb{E}_{(\mathbf{x}, y) \sim P}[L(y, f(\mathbf{x}))]$

**经验风险**：$R_{emp}(f) = \frac{1}{m} \sum_{i=1}^{m} L(y_i, f(\mathbf{x}_i))$

由于 $P(\mathbf{x}, y)$ 未知，实际中用经验风险近似期望风险。但过拟合风险要求我们使用**结构风险最小化(SRM)**：$\min_f R_{emp}(f) + \lambda \cdot \Omega(f)$

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 统计学习三要素（全书核心框架）

李航以**统计学习三要素**为统一框架，贯穿监督学习和无监督学习全书：

| 要素 | 内涵 | 监督学习实例 | 无监督学习实例 |
|------|------|-------------|---------------|
| **模型** (Model) | 假设空间 | $P_\theta(y\mid\mathbf{x})$ 或 $y = f_\theta(\mathbf{x})$ | $P_\theta(\mathbf{x})$ 或聚类划分 |
| **策略** (Strategy) | 选最优模型的标准 | ERM / SRM / MLE / MAP | 极大似然 / 重构误差 / 对比损失 |
| **算法** (Algorithm) | 求解方法 | SGD, SMO, 前向分步 | EM, SVD, Gibbs采样, 梯度下降 |

**模型分类体系**（李航更细致的划分）：

- **生成模型** vs **判别模型**
  - 生成模型：学习 $P(X, Y)$ → 求 $P(Y|X)$。代表：朴素贝叶斯、HMM、GMM、LDA
  - 判别模型：直接学习 $P(Y|X)$ 或 $f(X)$。代表：感知机、LR、SVM、CRF
- **概率模型** vs **非概率模型**
  - 概率模型：$P(y|\mathbf{x})$ 形式（可采样、可处理缺失数据）
  - 非概率模型：$y = f(\mathbf{x})$ 形式（通常更简单直接）
- **线性模型** vs **非线性模型**
- **参数化模型** vs **非参数化模型**（参数数量是否固定）

**策略的统一视角**：

| 策略 | 形式 | 概率解释 |
|------|------|---------|
| ERM | $\min_f \frac{1}{m}\sum_i L(y_i, f(x_i))$ | — |
| SRM | $\min_f \frac{1}{m}\sum_i L(y_i, f(x_i)) + \lambda J(f)$ | MAP（正则化项 = 对数先验） |
| MLE | $\max_\theta \sum_i \log P(y_i \mid x_i; \theta)$ | 频率学派核心方法 |
| MAP | $\max_\theta \sum_i \log P(y_i \mid x_i; \theta) + \log P(\theta)$ | 贝叶斯学派核心方法 |

**常用损失函数**：

| 损失函数 | 数学形式 | 典型应用 |
|----------|---------|---------|
| 0-1损失 | $L = \mathbb{I}(y \neq f(x))$ | 分类基本度量 |
| 平方损失 | $L = (y - f(x))^2$ | 回归 |
| 绝对损失 | $L = |y - f(x)|$ | 回归(鲁棒) |
| 对数损失/交叉熵 | $L = -\log P(y|x)$ | 分类(逻辑回归) |
| Hinge损失 | $L = \max(0, 1 - yf(x))$ | SVM |
| 指数损失 | $L = e^{-yf(x)}$ | AdaBoost |
| Huber损失 | 分段二次+线性 | 回归(鲁棒+可微) |

### 3.2 泛化误差上界

清华大学PPT给出了统计学习理论的核心结论——**泛化误差上界**：

对于二分类问题，假设空间 $\mathcal{H}$ 有限，则对任意 $f \in \mathcal{H}$，以至少 $1-\delta$ 的概率：

$$R(f) \leq \hat{R}(f) + \sqrt{\frac{\log|\mathcal{H}| + \log(1/\delta)}{2m}}$$

更一般地（考虑VC维 $h$，见第20章计算学习理论），泛化误差界为：

$$R(f) \leq \hat{R}(f) + \sqrt{\frac{h(\log(2m/h) + 1) - \log(\delta/4)}{m}}$$

这表明：(1) 训练样本 $m$ 越大，泛化误差界越小；(2) 假设空间容量 $|\mathcal{H}|$（或VC维 $h$）越大，泛化误差界越大。

### 3.3 模型复杂度与偏差-方差权衡

PPT详细阐述了偏差-方差分解。对于回归任务，期望泛化误差可分解为：

$$\mathbb{E}_D[(f(\mathbf{x}; D) - y_D)^2] = \underbrace{(\mathbb{E}_D[f(\mathbf{x}; D)] - \bar{y})^2}_{\text{Bias}^2} + \underbrace{\mathbb{E}_D[(f(\mathbf{x}; D) - \mathbb{E}_D[f(\mathbf{x}; D)])^2]}_{\text{Variance}} + \underbrace{\sigma^2}_{\text{Noise}}$$

- 简单模型 → 高偏差、低方差 → **欠拟合**
- 复杂模型 → 低偏差、高方差 → **过拟合**

最优模型复杂度在偏差和方差的交汇处。

---

## 四、综合总结

### 算法与概念对比

| 概念 | 西瓜书视角 | 李航视角 |
|------|-----------|---------|
| 学习定义 | 利用经验改善性能 | 基于数据构建概率模型 |
| 模型分类 | 从任务类型分（分类/回归/聚类） | 从模型性质分（生成/判别、概率/非概率） |
| 核心框架 | 泛化+NFL+归纳偏好 | 三要素（模型/策略/算法） |
| 评估 | 偏差-方差分解 | 泛化误差上界 |
| 正则化 | 奥卡姆剃刀原则 | SRM的数学实现 |

### 关键结论

1. **机器学习的本质**：数据 → 模型 → 预测/分析，核心是泛化
2. **统计学习三要素**（模型/策略/算法）是全书统一分析框架
3. **NFL定理**：没有万能算法，必须针对具体问题选择
4. **归纳偏好**是学习成为可能的必要条件
5. **过拟合vs欠拟合**的权衡贯穿所有学习方法
6. **生成模型 vs 判别模型**：前者灵活（可生成样本），后者直接（通常更准）
7. **各种损失函数**对应不同的概率假设（平方损失↔高斯噪声，交叉熵↔伯努利/类别分布，Hinge↔最大间隔）

### 章节连接

- **前置依赖**：概率论与数理统计、线性代数基础
- **后续延伸**：第2章(模型评估与选择)—具体化泛化评估；第5章(线性模型)—第一个监督学习方法；第17章(感知机)—最简单的分类模型；第20章(计算学习理论)—理论深化
