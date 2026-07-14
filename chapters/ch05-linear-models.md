# 第3章: 线性模型与逻辑回归 / Linear Models & Logistic Regression

> **融合来源**: 西瓜书第3章(线性模型) + 南瓜书第3章 + 李航第6章(逻辑斯谛回归与最大熵模型) + 清华大学PPT

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 线性模型的基本形式

线性模型试图学得一个通过属性的线性组合来进行预测的函数：

$$f(\mathbf{x}) = w_1 x_1 + w_2 x_2 + \cdots + w_d x_d + b = \mathbf{w}^T \mathbf{x} + b$$

其中 $\mathbf{w}$ 直观表达了各属性在预测中的重要性，因此线性模型具有**很好的可解释性**(comprehensibility)。

### 1.2 线性回归

线性回归的目标是学得 $f(\mathbf{x}_i) \approx y_i$。最常用的性能度量是均方误差(MSE)：

$$(\mathbf{w}^*, b^*) = \arg\min_{(\mathbf{w}, b)} \sum_{i=1}^{m} (f(\mathbf{x}_i) - y_i)^2 = \arg\min_{(\mathbf{w}, b)} \sum_{i=1}^{m} (\mathbf{w}^T\mathbf{x}_i + b - y_i)^2$$

基于均方误差最小化进行模型求解的方法称为**最小二乘法**(least square method)。

西瓜书给出了完整的矩阵推导：令 $\hat{\mathbf{w}} = (\mathbf{w}; b)$，将数据集表示为一个 $m \times (d+1)$ 的矩阵 $\mathbf{X}$。最小二乘的解析解为：

$$\hat{\mathbf{w}}^* = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{y}$$

当 $\mathbf{X}^T \mathbf{X}$ 不可逆时(属性数多于样本数，或属性间线性相关)，可引入正则化：$\hat{\mathbf{w}}^* = (\mathbf{X}^T \mathbf{X} + \lambda \mathbf{I})^{-1} \mathbf{X}^T \mathbf{y}$——这就是岭回归(ridge regression)。

西瓜书强调：**广义线性模型**(GLM)联系函数$g(\cdot)$将线性模型的预测值映射到不同的目标空间：$y = g^{-1}(\mathbf{w}^T\mathbf{x}+b)$。当$g(\cdot)=\ln(\cdot)$时为对数线性回归。

### 1.3 对数几率回归 (逻辑回归)

对于二分类任务，需要将实值输出 $z = \mathbf{w}^T\mathbf{x} + b$ 转换为0/1值。最理想的是**单位阶跃函数**(unit-step function)，但它不连续不可导。**对数几率函数**(logistic function)是替代品：

$$y = \frac{1}{1 + e^{-z}} = \frac{1}{1 + e^{-(\mathbf{w}^T\mathbf{x}+b)}}$$

变换后得：

$$\ln\frac{y}{1-y} = \mathbf{w}^T\mathbf{x} + b$$

将$y$视为样本为正例的可能性，则$1-y$为反例可能性，比值$\frac{y}{1-y}$称为**几率**(odds)，取对数后为**对数几率**(log odds / logit)。

逻辑回归的优越性：无需事先假设数据分布，直接对分类可能性建模；不仅预测类别，还得到近似概率；对数几率函数是任意阶可导的凸函数。

### 1.4 线性判别分析 (LDA)

LDA的核心思想：将样本投影到一条直线上，使得同类样本的投影点尽可能接近(最小化类内散度)，异类样本的投影点尽可能远离(最大化类间散度)。

类内散度矩阵：$\mathbf{S}_w = \sum_{i=0}^{1} \sum_{\mathbf{x}\in X_i} (\mathbf{x} - \boldsymbol{\mu}_i)(\mathbf{x} - \boldsymbol{\mu}_i)^T$

类间散度矩阵：$\mathbf{S}_b = (\boldsymbol{\mu}_0 - \boldsymbol{\mu}_1)(\boldsymbol{\mu}_0 - \boldsymbol{\mu}_1)^T$

最大化**广义瑞利商**(generalized Rayleigh quotient)：

$$J = \frac{\mathbf{w}^T \mathbf{S}_b \mathbf{w}}{\mathbf{w}^T \mathbf{S}_w \mathbf{w}}$$

解得 $\mathbf{w}^* = \mathbf{S}_w^{-1}(\boldsymbol{\mu}_0 - \boldsymbol{\mu}_1)$（等价于求解广义特征值问题 $\mathbf{S}_b\mathbf{w} = \lambda \mathbf{S}_w \mathbf{w}$）。LDA可从贝叶斯决策论角度解释：当两类数据同协方差、先验相等时，LDA等价于最优贝叶斯分类器。

---

## 二、公式推导 — 南瓜书补充

### 2.1 线性回归的矩阵推导

将$\mathbf{w}$和$b$吸收入向量形式$\hat{\mathbf{w}} = (\mathbf{w}; b)$，$\mathbf{X}$的最后一列全为1。

$$E_{\hat{\mathbf{w}}} = (\mathbf{y} - \mathbf{X}\hat{\mathbf{w}})^T(\mathbf{y} - \mathbf{X}\hat{\mathbf{w}})$$

对$\hat{\mathbf{w}}$求导并令为零：

$$\frac{\partial E_{\hat{\mathbf{w}}}}{\partial \hat{\mathbf{w}}} = 2\mathbf{X}^T(\mathbf{X}\hat{\mathbf{w}} - \mathbf{y}) = 0$$

$$\Rightarrow \hat{\mathbf{w}}^* = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{y}$$

**南瓜书补充**：当样本数 < 属性数时，$\mathbf{X}^T\mathbf{X}$不可逆，需引入正则化项 $\frac{\lambda}{2}\|\hat{\mathbf{w}}\|_2^2$，解析解变为 $\hat{\mathbf{w}}^* = (\mathbf{X}^T\mathbf{X} + \lambda\mathbf{I})^{-1}\mathbf{X}^T\mathbf{y}$ (岭回归)。

### 2.2 逻辑回归的梯度推导

对数似然函数：

$$\ell(\boldsymbol{\beta}) = \sum_{i=1}^{m} \left[y_i \log p_1(\hat{\mathbf{x}}_i) + (1-y_i) \log p_0(\hat{\mathbf{x}}_i)\right]$$

其中 $p_1(\hat{\mathbf{x}}_i) = \frac{e^{\boldsymbol{\beta}^T \hat{\mathbf{x}}_i}}{1+e^{\boldsymbol{\beta}^T \hat{\mathbf{x}}_i}}$, $p_0(\hat{\mathbf{x}}_i) = \frac{1}{1+e^{\boldsymbol{\beta}^T \hat{\mathbf{x}}_i}}$，$\boldsymbol{\beta} = (\mathbf{w}; b)$。

南瓜书给出了梯度推导的完整过程：

$$\frac{\partial \ell}{\partial \boldsymbol{\beta}} = -\sum_{i=1}^{m} \hat{\mathbf{x}}_i (y_i - p_1(\hat{\mathbf{x}}_i))$$

Hessian矩阵：

$$\frac{\partial^2 \ell}{\partial \boldsymbol{\beta} \partial \boldsymbol{\beta}^T} = \sum_{i=1}^{m} p_1(\hat{\mathbf{x}}_i)(1-p_1(\hat{\mathbf{x}}_i))\hat{\mathbf{x}}_i \hat{\mathbf{x}}_i^T$$

这是一个半正定矩阵，$\ell$是凸函数，梯度下降保证收敛到全局最优。

### 2.3 LDA的广义特征值问题推导

南瓜书补充LDA中$\mathbf{S}_w^{-1}\mathbf{S}_b$的推导：

$$\mathbf{S}_b \mathbf{w} = (\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)^T \mathbf{w}$$

令 $\lambda_w = (\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)^T \mathbf{w}$（一个标量），则 $\mathbf{S}_b \mathbf{w} = \lambda_w (\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)$

代入广义特征值问题 $\mathbf{S}_b \mathbf{w} = \lambda \mathbf{S}_w \mathbf{w}$，得 $\mathbf{w} = \frac{\lambda_w}{\lambda} \mathbf{S}_w^{-1} (\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)$，方向由 $\mathbf{S}_w^{-1}(\boldsymbol{\mu}_0-\boldsymbol{\mu}_1)$ 决定。

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 逻辑斯谛回归的统计学习三要素（李航Ch6）

**模型**：二项逻辑斯谛回归的条件概率分布

$$P(Y=1|\mathbf{x}) = \frac{\exp(\mathbf{w} \cdot \mathbf{x} + b)}{1 + \exp(\mathbf{w} \cdot \mathbf{x} + b)}$$
$$P(Y=0|\mathbf{x}) = \frac{1}{1 + \exp(\mathbf{w} \cdot \mathbf{x} + b)}$$

逻辑斯谛回归属于**对数线性模型**(log-linear model)。一个事件的几率(odds)为 $\frac{p}{1-p}$，对数几率为 $\log\frac{p}{1-p} = \mathbf{w} \cdot \mathbf{x} + b$——这正是线性模型的形式。

**策略**：极大似然估计。对数似然函数 $\ell(\mathbf{w}) = \sum_{i=1}^m [y_i(\mathbf{w} \cdot \mathbf{x}_i) - \log(1 + \exp(\mathbf{w} \cdot \mathbf{x}_i))]$

**算法**：梯度下降法、改进的迭代尺度法(IIS)、拟牛顿法(BFGS/L-BFGS)。

李航指出：LR的损失函数 = **交叉熵损失**(cross-entropy loss)，等价于KL散度的最小化。

### 3.2 最大熵模型（李航Ch6）

**最大熵原理**：在所有满足约束条件的概率模型中，选择熵最大的模型。熵最大 = 最"均匀" = 不做无根据的假设。

特征函数$f(x,y)$描述了$x$与$y$的某一事实。约束条件：$\mathbb{E}_P[f] = \mathbb{E}_{\tilde{P}}[f]$——特征关于模型的期望与关于经验分布的期望一致。

最大熵模型的解具有参数化形式：

$$P_w(y|x) = \frac{\exp(\sum_i w_i f_i(x,y))}{Z_w(x)}$$

其中 $Z_w(x) = \sum_y \exp(\sum_i w_i f_i(x,y))$ 为归一化因子。

**最大熵模型 = 逻辑斯谛回归的推广**：当$y \in \{0,1\}$且特征函数为 $f(x,y) = x \cdot \mathbb{I}(y=1)$ 时，最大熵模型退化为逻辑回归。

**参数学习**：IIS(Improved Iterative Scaling)和拟牛顿法是两种主要算法。IIS每次更新 $\delta_i = \frac{1}{M}\log\frac{\mathbb{E}_{\tilde{P}}[f_i]}{\mathbb{E}_P[f_i]}$，收敛速度较慢；拟牛顿法(BFGS)收敛更快。

### 3.3 多分类扩展

- **多项逻辑斯谛回归**(multinomial logistic regression / softmax regression)：$P(Y=k|\mathbf{x}) = \frac{\exp(\mathbf{w}_k \cdot \mathbf{x})}{\sum_{j=1}^K \exp(\mathbf{w}_j \cdot \mathbf{x})}$
- **OvR**(One-vs-Rest)：每类训练一个二分类器
- **OvO**(One-vs-One)：每对类训练一个二分类器，投票
- **MvM**(Many-vs-Many)：ECOC纠错输出码

PPT补充：Softmax回归的对数似然梯度为 $\nabla_{\mathbf{w}_k} \ell = \sum_i \mathbf{x}_i(\mathbb{I}(y_i=k) - P(Y=k|\mathbf{x}_i))$，与二分类LR的梯度形式一致。

---

## 四、综合总结

### 算法对比

| 方法 | 模型 | 输出 | 损失函数 | 求解 |
|------|------|------|---------|------|
| 线性回归 | $\mathbf{w}^T\mathbf{x}+b$ | 实值 | 平方损失 | 解析解/梯度下降 |
| 岭回归 | $\mathbf{w}^T\mathbf{x}+b$ | 实值 | 平方损失 + $\lambda\|\mathbf{w}\|_2^2$ | 解析解 |
| Lasso | $\mathbf{w}^T\mathbf{x}+b$ | 实值 | 平方损失 + $\lambda\|\mathbf{w}\|_1$ | 近端梯度下降 |
| 逻辑回归 | $\sigma(\mathbf{w}^T\mathbf{x})$ | 概率 | 交叉熵/对数似然 | 梯度下降/BFGS |
| Softmax | softmax | 概率向量 | 交叉熵 | 梯度下降 |
| 最大熵模型 | $\frac{\exp(\mathbf{w}^T\mathbf{f})}{Z}$ | 概率 | 对数似然 | IIS/BFGS |
| LDA | 投影+阈值 | 类别 | 广义瑞利商 | 广义特征值 |

### 关键结论

1. **线性模型简单但强大**：可解释性好，高维数据下表现优秀，是复杂模型的基础构件
2. **逻辑回归 = 广义线性模型 + 伯努利分布 + Sigmoid联系函数**
3. **LR的输出是校准良好的概率**，在需要概率估计的场景（如风控评分卡）中至关重要
4. **最大熵 = 无偏性原则**：只满足已知的约束，对其他不做假设
5. **LDA vs LR**：LDA假设数据服从高斯分布且各类协方差相同（生成式），LR不做分布假设（判别式）——LR的假设更弱，通常更鲁棒
6. **正则化的必要性**：L1/L2/L1+L2(弹性网)应对共线性和高维问题
7. **IIS vs 拟牛顿法**：IIS收敛慢但每次迭代简单，BFGS收敛快但需要Hessian近似

### 章节连接

- **前置依赖**：最优化方法（梯度下降、牛顿法）；概率论（MLE、贝叶斯）
- **后续延伸**：第8章(支持向量机)—线性模型+最大间隔；第17章(感知机)—最简单的线性分类器；第19章(特征选择)—Lasso与稀疏性
