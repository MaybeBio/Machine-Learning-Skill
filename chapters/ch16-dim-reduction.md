# 第16章: 降维与度量学习 / Dimensionality Reduction & Metric Learning

> **融合来源**: 西瓜书第10章(降维与度量学习) + 南瓜书第10章 + 李航第15章(奇异值分解) + 李航第16章(主成分分析) + 清华大学PPT

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 k近邻学习与维数灾难

k近邻(kNN)学习是"懒惰学习"(lazy learning)的代表：训练阶段仅保存样本，收到测试样本后才处理。最近邻分类器(1NN)的泛化错误率不超过贝叶斯最优分类器错误率的两倍。

然而此结论依赖**密采样**(dense sample)假设。若属性维数为20，满足密采样需 $(10^3)^{20} = 10^{60}$ 个样本——现实中无法达到。**维数灾难**(curse of dimensionality)：高维情形下数据样本稀疏、距离计算困难。

### 1.2 低维嵌入与MDS

缓解维数灾难的重要途径是**降维**(dimension reduction)，通过数学变换将高维空间转变为低维子空间。可行原因是：与学习任务密切相关的往往只是高维空间中的一个低维嵌入(embedding)。

**多维缩放(MDS)**：要求原始空间中样本之间的距离在低维空间中得以保持。给定距离矩阵 $\mathbf{D} \in \mathbb{R}^{m \times m}$，目标是获得 $d'$ 维表示 $\mathbf{Z} \in \mathbb{R}^{d' \times m}$ 使 $\|\mathbf{z}_i - \mathbf{z}_j\| = dist_{ij}$。

令内积矩阵 $\mathbf{B} = \mathbf{Z}^T\mathbf{Z}$，由距离可求：
$$b_{ij} = -\frac{1}{2}(dist_{ij}^2 - dist_{i\cdot}^2 - dist_{\cdot j}^2 + dist_{\cdot\cdot}^2)$$

对B做特征值分解 $\mathbf{B} = \mathbf{V}\boldsymbol{\Lambda}\mathbf{V}^T$，取前 $d'$ 个最大特征值对应的特征向量，得 $\mathbf{Z} = \tilde{\boldsymbol{\Lambda}}^{1/2}\tilde{\mathbf{V}}^T \in \mathbb{R}^{d' \times m}$。

### 1.3 主成分分析(PCA)

PCA是最常用的降维方法，可从两个等价角度推导：

- **最近重构性**：样本点到超平面的距离足够近 → 最小化重构误差
- **最大可分性**：样本点在超平面上的投影尽可能分开 → 最大化投影方差

两者导出同一优化目标：$\min_{\mathbf{W}} -\text{tr}(\mathbf{W}^T\mathbf{X}\mathbf{X}^T\mathbf{W}), \; s.t. \mathbf{W}^T\mathbf{W} = \mathbf{I}$

对协方差矩阵 $\mathbf{X}\mathbf{X}^T$ 做特征值分解，取前 $d'$ 个最大特征值对应的特征向量构成投影矩阵 $\mathbf{W}$。实践中常通过SVD代替协方差矩阵的特征值分解。

### 1.4 核化线性降维与流形学习

**核PCA(KPCA)**：通过核技巧将PCA推广到非线性降维。先将样本映射到高维特征空间，再在高维空间中做PCA：$K(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i)^T\phi(\mathbf{x}_j)$。

**流形学习**(manifold learning)：假设数据分布在低维流形上，目标是恢复低维流形结构。

- **等度量映射(Isomap)**：用测地线距离替代欧氏距离，通过最短路径算法近似测地线，再对测地线距离矩阵用MDS
- **局部线性嵌入(LLE)**：保持局部线性关系。为每个样本用其近邻线性重构：$\min \|\mathbf{x}_i - \sum_j w_{ij}\mathbf{x}_j\|^2$，再在低维空间保持此重构权重

### 1.5 度量学习

降维的目的是找到合适的低维空间，本质上是寻找合适的距离度量。**度量学习**(metric learning)直接学习距离度量。最常用的是学习马氏距离(Mahalanobis distance)的参数矩阵 $\mathbf{M}$：

$$dist_{\mathbf{M}}^2(\mathbf{x}_i, \mathbf{x}_j) = (\mathbf{x}_i - \mathbf{x}_j)^T \mathbf{M} (\mathbf{x}_i - \mathbf{x}_j) = \|\mathbf{L}(\mathbf{x}_i - \mathbf{x}_j)\|^2$$

其中 $\mathbf{M} = \mathbf{L}^T\mathbf{L}$ 为半正定矩阵。NCA(Neighbourhood Component Analysis)通过最大化留一法准确率来学习 $\mathbf{M}$。

---

## 二、公式推导 — 南瓜书补充

### 2.1 PCA最大方差推导的细节

南瓜书补全了PCA的Lagrange乘子法推导。对优化问题 $\max_{\mathbf{W}} \text{tr}(\mathbf{W}^T\mathbf{X}\mathbf{X}^T\mathbf{W}), s.t. \mathbf{W}^T\mathbf{W} = \mathbf{I}$，构造Lagrange函数：

$$\mathcal{L} = \text{tr}(\mathbf{W}^T\mathbf{X}\mathbf{X}^T\mathbf{W}) - \text{tr}(\boldsymbol{\Lambda}(\mathbf{W}^T\mathbf{W} - \mathbf{I}))$$

对 $\mathbf{W}$ 求导并令为零得：$\mathbf{X}\mathbf{X}^T\mathbf{W} = \mathbf{W}\boldsymbol{\Lambda}$，即 $\mathbf{W}$ 的列为协方差矩阵的特征向量。

### 2.2 SVD与PCA的等价性

南瓜书证明：对中心化后的数据矩阵 $\mathbf{X} \in \mathbb{R}^{d \times m}$，其SVD分解 $\mathbf{X} = \mathbf{U}\boldsymbol{\Sigma}\mathbf{V}^T$，右奇异向量 $\mathbf{V}$ 是 $\mathbf{X}^T\mathbf{X}$ 的特征向量（即样本协方差矩阵的特征向量），左奇异向量 $\mathbf{U}$ 对应主方向。因此PCA可直接通过SVD求解。

### 2.3 LLE重构权重的求解

南瓜书给出LLE最小化重构误差的推导。每个样本由k个近邻线性重构，约束 $\sum_j w_{ij} = 1$，代入Lagrange乘子法求解：

$$w_{ij} = \frac{\sum_k (C_{jk})^{-1}}{\sum_l \sum_m (C_{lm})^{-1}}$$

其中 $C_{jk} = (\mathbf{x}_i - \mathbf{x}_j)^T(\mathbf{x}_i - \mathbf{x}_k)$ 为局部协方差矩阵。

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 SVD的严格定义与性质（李航第15章）

**定义**：矩阵 $\mathbf{A} \in \mathbb{R}^{m \times n}$ 的奇异值分解为 $\mathbf{A} = \mathbf{U}\boldsymbol{\Sigma}\mathbf{V}^T$，其中 $\mathbf{U} \in \mathbb{R}^{m \times m}$ 和 $\mathbf{V} \in \mathbb{R}^{n \times n}$ 为正交矩阵，$\boldsymbol{\Sigma}$ 为对角矩阵（对角元素为奇异值 $\sigma_i \geq 0$，降序排列）。

**紧奇异值分解**：$\mathbf{A} = \mathbf{U}_r \boldsymbol{\Sigma}_r \mathbf{V}_r^T$，其中 $r = \text{rank}(\mathbf{A})$

**截断奇异值分解**：$\mathbf{A} \approx \mathbf{U}_k \boldsymbol{\Sigma}_k \mathbf{V}_k^T$，取前k个最大奇异值，是矩阵的最佳低秩近似（Eckart-Young定理）。

### 3.2 PCA的统计学习三要素（李航第16章）

| 要素 | 内容 |
|------|------|
| **模型** | 低维正交空间：$\mathbf{y} = \mathbf{W}^T\mathbf{x}$，其中 $\mathbf{W}^T\mathbf{W} = \mathbf{I}$ |
| **策略** | 方差最大原则：$\max_{\mathbf{W}} \text{tr}(\mathbf{W}^T\mathbf{X}\mathbf{X}^T\mathbf{W})$ |
| **算法** | SVD（对数据矩阵做奇异值分解，取前k个右奇异向量） |

### 3.3 PPT补充：降维方法的分类

PPT将降维方法分为：

- **线性降维**：PCA、MDS、LDA（监督降维）
- **非线性降维**：
  - 基于核的方法：KPCA、KDA
  - 基于流形学习：Isomap（保持测地线距离）、LLE（保持局部线性结构）、LE（拉普拉斯特征映射，保持局部邻近关系）
  - 基于神经网络：Autoencoder

### 3.4 PCA与SVD的主题模型应用

李航指出PCA/SVD也是话题分析的基础：LSA（潜在语义分析）直接基于SVD将词-文档矩阵分解为词-话题矩阵和话题-文档矩阵，实现降维与话题发现。这为后续的PLSA、LDA等概率话题模型奠定基础。

---

## 四、综合总结

### 算法对比

| 方法 | 线性/非线性 | 核心思想 | 适用场景 | 关键参数 |
|------|-----------|---------|---------|---------|
| MDS | 线性 | 保持距离 | 距离(不相似度)数据 | $d'$ |
| PCA | 线性 | 最大方差 | 高斯分布数据降维 | $d'$ |
| KPCA | 非线性 | 核技巧+PCA | 非线性结构数据 | 核函数, $d'$ |
| Isomap | 非线性 | 测地线距离+MDS | 流形数据(有全局结构) | 近邻数k, $d'$ |
| LLE | 非线性 | 保持局部重构 | 流形数据(保持局部结构) | 近邻数k, $d'$ |
| t-SNE | 非线性 | 保持邻域概率分布 | 可视化(2D/3D) | 困惑度perplexity |
| NCA | 度量学习 | 最大化留一法准确率 | 学习距离度量 | $d'$ |

### 关键结论

1. **维数灾难**是所有ML方法共同面临的严重障碍，降维是缓解的核心技术
2. **MDS→PCA→KPCA→流形学习**构成从线性到非线性的降维演进脉络
3. **PCA可视为SVD的特例应用**：PCA = 中心化数据矩阵的SVD
4. **流形学习假设数据分布在一个低维流形上**，Isomap和LLE分别从全局和局部角度恢复流形
5. **t-SNE是最流行的可视化降维方法**，但不适合作为其他学习任务的前处理（非参数化）
6. **度量学习直接学距离而非降维**，通过马氏距离的参数矩阵 $\mathbf{M}$ 实现
7. **降维与特征选择**是处理高维数据的两大主流技术，但降维产生新特征（原特征的组合），特征选择保留原始特征子集

### 章节连接

- **前置依赖**：ch01(绪论)—维度与泛化；ch05(线性模型)—线性变换基础；ch02(评估)—性能度量
- **后续延伸**：ch19(特征选择)—高维数据处理的另一途径；ch17(概率图模型)—LDA的概率降维视角；ch24(PageRank与LSA)—SVD的具体应用

