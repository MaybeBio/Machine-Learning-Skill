# 第3章 线性模型

> **融合来源**: 西瓜书第3章 + 南瓜书第3章 + 李航第6章 + 李航第12章

---

## 一、核心概念

**线性模型**是机器学习中最基础且最重要的模型族。其基本形式为：

$$f(\mathbf{x}) = \mathbf{w}^\mathrm{T}\mathbf{x} + b = \sum_{i=1}^{d} w_i x_i + b$$

其中 $\mathbf{w} = (w_1; w_2; \ldots; w_d)$ 为权重向量，$b$ 为偏置项。线性模型的优势在于形式简单、可解释性强（权重大小直接反映特征重要性），且是许多复杂模型的基础构件。

本章涵盖四大核心模型：

| 模型 | 任务 | 核心思想 | 损失函数 |
|------|------|----------|----------|
| **线性回归** | 回归 | 最小二乘法拟合 | 平方损失 $\sum (y_i - \mathbf{w}^\mathrm{T}\mathbf{x}_i - b)^2$ |
| **对数几率回归（逻辑回归）** | 二分类 | Sigmoid + 极大似然 | 交叉熵/对数损失 $-\sum[y_i \ln \hat{y}_i + (1-y_i)\ln(1-\hat{y}_i)]$ |
| **线性判别分析 (LDA)** | 分类+降维 | Fisher 准则最大化类间与类内散度比 | 广义瑞利商 |
| **最大熵模型** | 多分类 | 最大熵原理 + 特征约束 | 负对数似然 + 正则化 |

---

## 二、融合框架

线性模型的学习可统一为以下框架：

$$\min_{\mathbf{w}, b} \frac{1}{m}\sum_{i=1}^{m} L(y_i, \mathbf{w}^\mathrm{T}\mathbf{x}_i + b) + \lambda \Omega(\mathbf{w})$$

其中 $L(\cdot, \cdot)$ 为损失函数，$\Omega(\cdot)$ 为正则化项：
- 线性回归：$L$ 为平方损失，$\Omega$ 常用 L2（Ridge）或 L1（Lasso）
- 逻辑回归：$L$ 为交叉熵损失（从极大似然推导而来）
- 最大熵模型：$L$ 为负对数似然，$\Omega$ 为 L2 正则化

**广义线性模型（GLM）统一视角**：线性回归对应正态分布假设（恒等连接），逻辑回归对应二项分布假设（Logit 连接）。两者都可以从指数族分布的极大似然估计推导而来。

---

## 三、理论推导

### 3.1 线性回归——最小二乘的矩阵推导

令 $\hat{\mathbf{w}} = (\mathbf{w}; b) \in \mathbb{R}^{(d+1) \times 1}$，增广样本 $\hat{\mathbf{x}}_i = (x_{i1}; \ldots; x_{id}; 1)$。构造数据矩阵 $\mathbf{X} \in \mathbb{R}^{m \times (d+1)}$ 和标记向量 $\mathbf{y} \in \mathbb{R}^m$。

目标函数：$E_{\hat{\mathbf{w}}} = (\mathbf{y} - \mathbf{X}\hat{\mathbf{w}})^\mathrm{T}(\mathbf{y} - \mathbf{X}\hat{\mathbf{w}})$

对 $\hat{\mathbf{w}}$ 求导：$\frac{\partial E_{\hat{\mathbf{w}}}}{\partial \hat{\mathbf{w}}} = 2\mathbf{X}^\mathrm{T}(\mathbf{X}\hat{\mathbf{w}} - \mathbf{y})$

令导数为零（假设 $\mathbf{X}^\mathrm{T}\mathbf{X}$ 可逆）：$\hat{\mathbf{w}}^* = (\mathbf{X}^\mathrm{T}\mathbf{X})^{-1}\mathbf{X}^\mathrm{T}\mathbf{y}$

当 $\mathbf{X}^\mathrm{T}\mathbf{X}$ 不可逆（特征数超过样本数或多重共线性），需引入正则化：
- **Ridge 回归（L2）**：$\hat{\mathbf{w}}^* = (\mathbf{X}^\mathrm{T}\mathbf{X} + \lambda\mathbf{I})^{-1}\mathbf{X}^\mathrm{T}\mathbf{y}$
- **Lasso 回归（L1）**：无闭式解，需用坐标下降法等优化

**凸性证明**：$E_{\hat{\mathbf{w}}}$ 的 Hessian 矩阵 $\nabla^2 E_{\hat{\mathbf{w}}} = 2\mathbf{X}^\mathrm{T}\mathbf{X}$ 是半正定的，故 $E_{\hat{\mathbf{w}}}$ 是凸函数。当 $\mathbf{X}^\mathrm{T}\mathbf{X}$ 正定时梯度为零的点即为全局最优解。

### 3.2 对数几率回归——从 Sigmoid 到交叉熵

对数几率函数（Sigmoid）：$\sigma(z) = \frac{1}{1 + e^{-z}}$，其反函数为对数几率（Logit）：$\ln\frac{y}{1-y}$。

用线性模型拟合对数几率：$\ln\frac{p(y=1 \mid \mathbf{x})}{p(y=0 \mid \mathbf{x})} = \mathbf{w}^\mathrm{T}\mathbf{x} + b$

解得：$p(y=1 \mid \mathbf{x}) = \frac{e^{\mathbf{w}^\mathrm{T}\mathbf{x} + b}}{1 + e^{\mathbf{w}^\mathrm{T}\mathbf{x} + b}}$

参数估计采用**极大似然估计（MLE）**。令 $\beta = (\mathbf{w}; b)$，对数似然：

$$\ell(\beta) = \sum_{i=1}^{m} \left(y_i \beta^\mathrm{T} \hat{\mathbf{x}}_i - \ln(1 + e^{\beta^\mathrm{T} \hat{\mathbf{x}}_i})\right)$$

最大化该似然等价于最小化负对数似然（即交叉熵损失）：

$$\min_\beta \sum_{i=1}^{m} \left(-y_i \beta^\mathrm{T} \hat{\mathbf{x}}_i + \ln(1 + e^{\beta^\mathrm{T} \hat{\mathbf{x}}_i})\right)$$

梯度（向量化形式）：$\frac{\partial \ell}{\partial \beta} = \mathbf{X}^\mathrm{T}(\hat{\mathbf{y}} - \mathbf{y})$

其中 $\hat{\mathbf{y}} = \sigma(\mathbf{X}\beta)$。此梯度形式与线性回归的梯度 $2\mathbf{X}^\mathrm{T}(\mathbf{X}\beta - \mathbf{y})$ 结构相似——都是预测值减去真实值乘以特征矩阵的转置。

Hessian 矩阵：$\nabla^2 \ell(\beta) = \sum_{i=1}^{m} \hat{\mathbf{x}}_i \hat{\mathbf{x}}_i^\mathrm{T} p_1(\hat{\mathbf{x}}_i; \beta)(1 - p_1(\hat{\mathbf{x}}_i; \beta))$

矩阵形式：$\nabla^2 \ell = \mathbf{X}^\mathrm{T} \mathbf{D} \mathbf{X}$，其中 $\mathbf{D} = \text{diag}(\hat{y}_1(1-\hat{y}_1), \ldots, \hat{y}_m(1-\hat{y}_m))$。由于 $\hat{y}_i \in (0,1)$，$\mathbf{D}$ 为正定对角阵，故 $\nabla^2 \ell$ 半正定——目标函数为凸函数。

### 3.3 线性判别分析 (LDA)——Fisher 准则

LDA 的优化目标：找到一个投影方向 $\mathbf{w}$，使得：
- **类间散度** $S_b = (\mu_0 - \mu_1)(\mu_0 - \mu_1)^\mathrm{T}$ 在投影方向上最大化
- **类内散度** $S_w = \Sigma_0 + \Sigma_1$ 在投影方向上最小化

广义瑞利商（广义 Rayleigh Quotient）：

$$J(\mathbf{w}) = \frac{\mathbf{w}^\mathrm{T}\mathbf{S}_b\mathbf{w}}{\mathbf{w}^\mathrm{T}\mathbf{S}_w\mathbf{w}}$$

最大化 $J(\mathbf{w})$ 等价于求解广义特征值问题：$\mathbf{S}_b\mathbf{w} = \lambda \mathbf{S}_w\mathbf{w}$

其闭式解为：$\mathbf{w}^* = \mathbf{S}_w^{-1}(\mu_0 - \mu_1)$

**多分类 LDA 推广**：优化目标推广为求 $\mathbf{W} \in \mathbb{R}^{d \times (N-1)}$，最大化 $\frac{\mathrm{tr}(\mathbf{W}^\mathrm{T}\mathbf{S}_b\mathbf{W})}{\mathrm{tr}(\mathbf{W}^\mathrm{T}\mathbf{S}_w\mathbf{W})}$。此时 $\mathbf{S}_b$ 的秩至多为 $N-1$，故至多得到 $N-1$ 个非零特征值对应的特征向量。

### 3.4 最大熵模型

**最大熵原理**：在满足已知约束的条件下，选择熵最大的概率分布——不引入任何主观偏见的"最不确定"分布。这确保了模型不做无根据的假设。

数学形式：给定特征函数 $f_i(\mathbf{x}, y)$ 和训练集上的经验期望 $\tilde{\mathbb{E}}(f_i)$，求解：

$$\begin{aligned} \max_{P} \quad & H(P) = -\sum_{\mathbf{x}, y} \tilde{P}(\mathbf{x}) P(y \mid \mathbf{x}) \log P(y \mid \mathbf{x}) \\ \text{s.t.} \quad & \mathbb{E}_P(f_i) = \tilde{\mathbb{E}}(f_i), \quad i = 1, \ldots, n \\ & \sum_{y} P(y \mid \mathbf{x}) = 1 \end{aligned}$$

通过拉格朗日乘子法解得指数族形式：$P_w(y \mid \mathbf{x}) = \frac{1}{Z_w(\mathbf{x})} \exp\left(\sum_{i=1}^{n} w_i f_i(\mathbf{x}, y)\right)$

其中 $Z_w(\mathbf{x}) = \sum_y \exp(\sum_{i=1}^{n} w_i f_i(\mathbf{x}, y))$ 为归一化因子。

**求解算法**：
- **IIS（改进迭代尺度法）**：每次迭代同时更新所有权重，保证似然单调不减
- **BFGS（拟牛顿法）**：利用近似 Hessian 逆矩阵加速收敛，适合特征较多场景

值得指出的是，最大熵模型对偶问题的目标函数等价于最大似然估计——最大熵模型的对偶形式与逻辑回归同构。

---

## 四、算法步骤

### 梯度下降法求解逻辑回归

1. 初始化 $\beta^0$（通常为零向量或小随机值）
2. 设学习率 $\alpha$，迭代次数 $t = 0$
3. 重复直到收敛：
   - 计算梯度：$\mathbf{g}^t = \mathbf{X}^\mathrm{T}(\hat{\mathbf{y}}^t - \mathbf{y})$，其中 $\hat{y}_i^t = \sigma(\mathbf{x}_i^\mathrm{T}\beta^t)$
   - 更新参数：$\beta^{t+1} = \beta^t - \alpha \mathbf{g}^t$
4. 检查收敛条件：$|L(\beta^{t+1}) - L(\beta^t)| < \epsilon$

### 牛顿法求解逻辑回归

每次迭代：$\beta^{t+1} = \beta^t - [\nabla^2 \ell(\beta^t)]^{-1} \nabla \ell(\beta^t)$

牛顿法二阶收敛，收敛速度快于梯度下降的一阶收敛，但每次需计算并求逆 Hessian 矩阵，计算复杂度 $O(d^3)$。

**拟牛顿法（BFGS）**：用迭代更新近似 Hessian 逆矩阵，避免显式矩阵求逆，在实践中最为常用。

---

## 五、直观理解

**线性回归 vs 逻辑回归**：尽管名称相似，但逻辑回归是分类模型。其名称来源于"回归"到对数几率——先做线性回归预测连
续值，再用 Sigmoid 函数将结果"挤压"到 $(0,1)$ 作为概率解释。

**LDA vs PCA**：LDA 是有监督降维（利用类别标签），PCA 是无监督降维（仅利用数据方差结构）。LDA 在分类任务上通常优于 PCA，但需要标签信息且假设各类协方差相同。

**多分类策略**：
- **OvO（一对一）**：两两类间训练一个分类器，共 $\binom{N}{2}$ 个
- **OvR（一对多）**：每类与其余各类训练一个分类器，共 $N$ 个
- **ECOC（纠错输出码）**：编码-解码框架，通过纠错码提高容错性和对噪音的鲁棒性。码长越长、码距越大，纠错能力越强。

**最大熵原理的哲学意义**：在没有更多信息的情况下，最合理的假设是"无所偏好"——这与信息论中熵最大化的不确定性最大化原则一致，也与统计物理中玻尔兹曼分布的推导逻辑相通。

---

## 六、常见误区

| 误区 | 正确理解 |
|------|----------|
| "逻辑回归只能做二分类" | 可通过 OvR、多分类交叉熵（Softmax 回归）扩展到多分类 |
| "线性回归有闭式解就是最优方法" | 当 $\mathbf{X}^\mathrm{T}\mathbf{X}$ 接近奇异时，闭式解数值不稳定；梯度下降或正则化更可靠 |
| "LDA 假设数据服从正态分布" | 虽然 LDA 源于贝叶斯决策的等协方差正态假设，但实际使用时即使不严格满足该假设也常有效 |
| "最大熵模型比逻辑回归好" | 两者本质上等价（最大熵的对偶形式 = 逻辑回归）；选择取决于特征工程是否使用特征函数的形式 |
| "梯度下降法一定能找到全局最优" | 仅对凸函数保证全局最优；非凸问题（如神经网络）只能保证局部最优。线性模型的目标函数是凸的，因此梯度下降找到的是全局最优 |

---

## 七、代码要点

```python
import numpy as np
from sklearn.linear_model import (LinearRegression, Ridge, Lasso,
    LogisticRegression)
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis as LDA

# 线性回归：闭式解 vs 梯度下降
# sklearn 默认用 SVD 解决 X^T X 不可逆问题
lr = LinearRegression().fit(X_train, y_train)
ridge = Ridge(alpha=1.0).fit(X_train, y_train)  # L2 正则化
lasso = Lasso(alpha=0.1).fit(X_train, y_train)  # L1 正则化（稀疏解）

# 逻辑回归：默认使用 L2 正则化（liblinear solver）
# 重要超参数：C = 1/lambda（C 越大正则化越弱）
clf = LogisticRegression(C=1.0, penalty='l2', solver='lbfgs')
clf.fit(X_train, y_train)
print(f"权重系数: {clf.coef_}, 截距: {clf.intercept_}")

# LDA：监督降维 + 分类
lda = LDA(n_components=2)  # n_components <= n_classes - 1
X_lda = lda.fit_transform(X_train, y_train)
```

---

## 八、实例分析

**信用评分场景**：给定用户的年收入、负债率、历史违约记录等特征，预测其是否会违约。

逻辑回归天然适合此场景：
- 输出既可用作分类（阈值 0.5 判是否违约），也可输出风险概率（如 0.83 的风险分）
- 权重系数可直接解释：若 $w_{\text{负债率}} = 2.3$，则负债率每增加一个单位，违约对数几率增加 2.3
- 金融监管要求模型可解释，逻辑回归的线性结构完全满足此需求

LDA 在此场景下也可用，但其分类面与逻辑回归的差异在于：逻辑回归不要求数据正态分布，建模更灵活；LDA 假设各类服从等协方差正态分布，当假设满足时通常更有效。

---

## 九、关键结论

1. **线性回归 = 平方损失 + ERM**，闭式解 $( \mathbf{X}^\mathrm{T}\mathbf{X})^{-1}\mathbf{X}^\mathrm{T}\mathbf{y}$ 是凸优化最优解
2. **逻辑回归 = Sigmoid + 交叉熵损失 + MLE**，梯度 $\mathbf{X}^\mathrm{T}(\hat{\mathbf{y}} - \mathbf{y})$ 简洁优美，无闭式解但凸函数保证收敛
3. **最大熵原理**提供了从约束条件推导概率模型的统一框架，其与极大似然估计的对偶关系是统计学习理论的核心结论之一
4. **LDA 的闭式解** $\mathbf{S}_w^{-1}(\mu_0 - \mu_1)$ 体现了"类间分离最大化、类内聚合最小化"的直观思想
5. 线性模型的核心优势在于**可解释性**和**数学简洁性**——权重直接反映特征贡献，所有理论都可以严格证明

---

## 十、章节连接

- **ch01 绪论**：线性模型是统计学习三要素框架的经典落地实例
- **ch02 模型评估**：MSE 评估线性回归，AUC/F1 评估逻辑回归
- **ch04 决策树**：线性模型 vs 树模型的经典对比——全局线性划分 vs 局部递归划分
- **ch06 SVM**：线性模型 + 最大间隔 + 核技巧 = 支持向量机，从线性到非线性的重要跃迁
- **ch07 贝叶斯分类器**：LDA 是贝叶斯最优分类器在高斯假设下的特例
