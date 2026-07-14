# 第10章: 集成学习与Boosting / Ensemble Learning & Boosting

> **融合来源**: 西瓜书第8章(集成学习) + 南瓜书第8章 + 李航第8章(Boosting/AdaBoost)

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 集成学习的核心思想

集成学习(Ensemble Learning)通过构建并结合多个学习器来完成任务。核心思想是"三个臭皮匠，顶个诸葛亮"——多个弱学习器合理组合，常可超越单个强学习器。

西瓜书给出集成的两个核心条件：个体学习器必须**"好而不同"**——既要有一定准确性(好)，又要有差异性(不同)。若所有学习器都一样，集成没有意义；若所有学习器都随机猜测，集成也不会更好。

**误差-分歧分解**：$E = \bar{E} - \bar{A}$，其中 $\bar{E}$ 是个体学习器的平均泛化误差，$\bar{A}$ 是个体学习器的平均分歧(差异性)。集成泛化误差 = 个体平均误差 - 平均分歧。差异性越大，集成效果越好。

### 1.2 三大集成范式

**Boosting（串行、降低偏差）**：个体学习器串行生成，后一个聚焦前一个的错分样本。代表：AdaBoost(指数损失)、GBDT/XGBoost(梯度提升)

**Bagging（并行、降低方差）**：个体学习器并行生成，通过对训练集Bootstrap采样产生差异性。代表：随机森林(Bagging + 随机属性选择)

**Stacking（学习法）**：用次级学习器(meta-learner)来组合初级学习器的输出，而不是简单投票/平均。

### 1.3 AdaBoost详解

AdaBoost(Adaptive Boosting)的核心机制：
1. 初始化样本权重均等 $w_i = 1/m$
2. 每轮训练一个基学习器，计算其加权错误率 $\epsilon_t$
3. 计算该学习器的权重 $\alpha_t = \frac{1}{2}\ln\frac{1-\epsilon_t}{\epsilon_t}$（错误率越低，权重越大）
4. 更新样本权重：错分样本权重增大 $w_i \leftarrow w_i \cdot e^{\alpha_t}$，正确样本权重减小，然后归一化
5. 最终：$H(\mathbf{x}) = \text{sign}(\sum_{t=1}^{T} \alpha_t h_t(\mathbf{x}))$

西瓜书的关键洞察：AdaBoost可以用**加性模型**(additive model)来解释——最终分类器是基学习器的加权线性组合。损失函数为**指数损失** $L = e^{-y f(\mathbf{x})}$，算法 = **前向分步算法**（每步固定之前的基学习器，只优化当前）。

### 1.4 Bagging与随机森林

**Bagging**(Bootstrap Aggregating)：从训练集有放回采样$m$个样本构成一个Bootstrap采样集（约63.2%原样本出现），训练基学习器，重复$T$次后投票/平均。由于是有放回采样，约36.8%的样本未被抽中，可作为**包外估计**(OOB)来评估泛化性能。

**随机森林**(Random Forest)：Bagging + 属性扰动。在决策树的每个结点分裂时，先从全部$d$个属性中随机选取$k$个($k \approx \log_2 d$ 或 $\sqrt{d}$)，再从这$k$个中选最优。进一步增加个体学习器的差异性。

### 1.5 结合策略

- **平均法**(回归)：简单平均 $\frac{1}{T}\sum h_i$ 或加权平均 $\sum w_i h_i$
- **投票法**(分类)：绝对多数(>50%)、相对多数(最高票)、加权投票
- **学习法**(Stacking)：用初级学习器的输出作为次级学习器的输入

---

## 二、公式推导 — 南瓜书补充

### 2.1 AdaBoost = 前向分步加法模型 + 指数损失的严格推导

南瓜书给出完整推导。加法模型：$f_m(\mathbf{x}) = \sum_{t=1}^{M} \beta_t b(\mathbf{x}; \gamma_t)$。前向分步：已知 $f_{m-1}$ 时，仅优化 $\beta_m, \gamma_m$。

以指数损失 $\ell(y, f(\mathbf{x})) = e^{-y f(\mathbf{x})}$ 训练：

$$(\beta_m, \gamma_m) = \arg\min_{\beta, \gamma} \sum_{i=1}^{m} e^{-y_i(f_{m-1}(\mathbf{x}_i) + \beta b(\mathbf{x}_i; \gamma))}$$

南瓜书推导出：(1) $\gamma_m^* = \arg\min_\gamma \sum_i \bar{w}_{mi} \mathbb{I}(y_i \neq b(\mathbf{x}_i; \gamma))$，即加权错误率最小的基学习器；(2) $\beta_m^* = \frac{1}{2}\ln\frac{1-e_m}{e_m} = \alpha_t$，即AdaBoost的权值更新公式。**这严格证明了AdaBoost是前向分步算法的特例。**

### 2.2 AdaBoost训练误差上界

南瓜书推导了AdaBoost的训练误差以指数速率下降：

$$\frac{1}{m}\sum_{i=1}^{m} \mathbb{I}(H(\mathbf{x}_i) \neq y_i) \leq \prod_{t=1}^{T} Z_t = \prod_{t=1}^{T} 2\sqrt{\epsilon_t(1-\epsilon_t)}$$

其中 $\epsilon_t \leq 0.5 - \gamma$（存在一个gap $\gamma > 0$）时，上界 $\leq e^{-2T\gamma^2}$，随$T$指数下降。

### 2.3 梯度提升(GBDT)

南瓜书补充了GBDT：将提升视为在函数空间中的梯度下降。每步新基学习器拟合当前模型的**负梯度**（伪残差）：

$$r_{ti} = -\left[\frac{\partial L(y_i, f(\mathbf{x}_i))}{\partial f(\mathbf{x}_i)}\right]_{f=f_{t-1}}$$

平方损失下负梯度 = 残差 $y_i - f(\mathbf{x}_i)$；指数损失下负梯度与AdaBoost一致。

---

## 三、理论深化 — 李航《机器学习方法》

### 3.1 AdaBoost的统计学习三要素（李航Ch8）

**模型**：加法模型 $f(\mathbf{x}) = \sum_{t=1}^{T} \alpha_t G_t(\mathbf{x})$，其中$G_t$为基本分类器(通常为决策树桩)。

**策略**：极小化指数损失 $\sum_i \exp(-y_i f(\mathbf{x}_i))$。李航证明：指数损失函数最小化 → 分类误差最小化（且指数损失是0-1损失的单调连续上界）。

**算法**：前向分步算法。每次仅优化当前轮的 $\alpha_t$ 和 $G_t$。

### 3.2 提升树(Boosting Tree)与GBDT

李航将以决策树为基学习器的Boosting称为**提升树**。回归问题的提升树每步拟合残差。**梯度提升**(Gradient Boosting)将这一思想一般化：用损失函数的负梯度近似残差。

### 3.3 集成方法的偏差-方差解释（PPT补充）

- Boosting：主要**降低偏差**（每个新学习器纠正前一个的错误）→ 用强依赖的串行学习器
- Bagging：主要**降低方差**（平均消除个体随机性）→ 用独立的并行学习器
- 随机森林：Bagging的去相关性版本 → 进一步降低方差

---

## 四、综合总结

| 方法 | 个体学习器关系 | 主要降低 | 代表 |
|------|-------------|---------|------|
| Boosting | 串行、强依赖 | 偏差 | AdaBoost, GBDT, XGBoost |
| Bagging | 并行、独立 | 方差 | 随机森林 |
| Stacking | 并行+学习器组合 | 方差+偏差 | 各种Stacking变体 |

### 关键结论

1. **"好而不同"是集成的精髓**——准确性和多样性缺一不可
2. **AdaBoost = 加法模型 + 指数损失 + 前向分步**，数学上非常优雅
3. **GBDT = 函数空间的梯度下降**，Boosting的现代版本
4. **随机森林 = Bagging + 属性随机**，最实用的集成方法之一
5. **XGBoost/LightGBM** 是GBDT的工程优化版（二阶导数、正则化、并行、缺失值处理）
6. **Boosting易过拟合噪声**（会追着噪声样本跑），Bagging对噪声更鲁棒

### 章节连接

- **前置依赖**: 第6章(决策树)—最常用的基学习器；第2章(偏差-方差分解)
- **后续延伸**: 第28章(GAN)—"博弈"视角的集成；现代ML竞赛中GBDT仍是表格数据的SOTA
