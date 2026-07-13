# Chapter 9: 聚类 / Clustering

## 核心概念 (Core Idea)

聚类是无监督学习中最核心的任务——将无标记样本划分成若干个不相交的子集（簇，cluster），使得同一簇内样本尽可能相似、不同簇间样本尽可能不同。与分类不同，聚类没有标签作为监督信号，因此其核心挑战在于**如何定义"相似"以及如何自动发现数据的内在分组结构**。

聚类的数学定义：设样本集 $D = \{x_1, x_2, \ldots, x_m\}$，每个样本 $x_i = (x_{i1}, x_{i2}, \ldots, x_{in})$ 是 $n$ 维向量。聚类算法将 $D$ 划分为 $k$ 个不相交的簇 $\{C_1, C_2, \ldots, C_k\}$，满足 $C_i \cap C_{j \neq i} = \emptyset$ 且 $\bigcup_{l=1}^{k} C_l = D$。聚类的解空间巨大——将 $m$ 个样本划分到 $k$ 个簇的方案数为第二类 Stirling 数，NP-hard。

## 融合框架 (Integrated Frameworks)

三大经典方法体系：

1. **基于原型的聚类** (k-means, LVQ, GMM)：每个簇由一个或多个原型向量/分布表示
2. **基于密度的聚类** (DBSCAN, OPTICS, DENCLUE)：簇是样本密度足够高的区域，由低密度区域分隔
3. **层次聚类** (AGNES, DIANA)：自底向上或自顶向下逐步合并或分裂簇

李航书中给出了无监督学习的三要素框架：**模型**（聚类划分函数）、**策略**（损失函数/相似度准则）、**算法**（迭代优化）。这为理解不同聚类算法提供了统一的视角。

## 理论推导 (Theoretical Derivation)

### k-means 损失函数与收敛性

k-means的目标是最小化簇内平方误差：

$$\text{SSE} = \sum_{l=1}^{k} \sum_{x \in C_l} \|x - \mu_l\|_2^2$$

其中 $\mu_l = \frac{1}{|C_l|}\sum_{x \in C_l} x$ 是簇 $C_l$ 的均值向量。这是一个组合优化问题，k-means使用EM风格的交替优化：

1. **E步（簇分配）**：固定均值向量，将每个样本分配到最近的均值向量
2. **M步（均值更新）**：固定簇分配，重新计算每个簇的均值

引理：每步操作都不增加SSE值。由于样本划分方案有限，算法必定收敛（到局部最优）。

**全局最优的条件**：当数据分布为各向同性的球状高斯时，k-means的局部最优即是全局最优。

### 高斯混合模型与EM算法

GMM假设数据由 $k$ 个高斯分布的混合生成：

$$p(x) = \sum_{l=1}^{k} \alpha_l \cdot \mathcal{N}(x | \mu_l, \Sigma_l), \quad \sum_{l=1}^{k} \alpha_l = 1$$

引入隐变量 $z_i \in \{1, \ldots, k\}$ 表示样本 $x_i$ 来自哪个高斯分量。EM算法在E步计算后验概率（责任值）：

$$\gamma_{il} = P(z_i = l | x_i) = \frac{\alpha_l \cdot \mathcal{N}(x_i | \mu_l, \Sigma_l)}{\sum_{j=1}^{k} \alpha_j \cdot \mathcal{N}(x_i | \mu_j, \Sigma_j)}$$

M步最大化期望对数似然更新参数：

$$\mu_l = \frac{\sum_{i=1}^{m} \gamma_{il} x_i}{\sum_{i=1}^{m} \gamma_{il}}, \quad \Sigma_l = \frac{\sum_{i=1}^{m} \gamma_{il} (x_i - \mu_l)(x_i - \mu_l)^T}{\sum_{i=1}^{m} \gamma_{il}}, \quad \alpha_l = \frac{1}{m}\sum_{i=1}^{m} \gamma_{il}$$

对比k-means与GMM-EM：k-means是GMM-EM在 $\Sigma_l = \sigma^2 I$ 且 $\sigma \to 0$ 时的退化情况，此时后验概率退化为hard assignment（最近邻分配）。

### DBSCAN 密度定义

核心思想基于 $\epsilon$-邻域：$N_\epsilon(x) = \{y \in D: \text{dist}(x, y) \leq \epsilon\}$。

- **核心对象**：$|N_\epsilon(x)| \geq \text{MinPts}$
- **密度直达**：$y$ 在 $x$ 的 $\epsilon$-邻域内且 $x$ 是核心对象
- **密度可达**：存在核心对象序列 $p_1, p_2, \ldots, p_T$ 使得 $p_{t+1}$ 由 $p_t$ 密度直达
- **密度相连**：存在核心对象 $o$ 使 $x$ 和 $y$ 都由 $o$ 密度可达

一个簇定义为从任一核心对象出发密度可达的所有样本的最大集合。DBSCAN自动确定簇数量，能发现任意形状的簇，且天然对噪声鲁棒。

### 距离度量

闵可夫斯基距离（Minkowski distance）是普适框架：

$$\text{dist}_{\text{mk}}(x_i, x_j) = \left(\sum_{u=1}^{n} |x_{iu} - x_{ju}|^p\right)^{1/p}$$

- $p=1$：曼哈顿距离（Manhattan）
- $p=2$：欧氏距离（Euclidean）
- $p \to \infty$：切比雪夫距离（Chebyshev）

处理离散属性时，VDM（Value Difference Metric）度量属性值在类别上的条件概率分布差异：

$$\text{VDM}_p(a, b) = \sum_{i=1}^{k} \left|\frac{m_{u,a,i}}{m_{u,a}} - \frac{m_{u,b,i}}{m_{u,b}}\right|^p$$

马氏距离（Mahalanobis distance）考虑特征间的相关性和尺度差异：

$$\text{dist}_{\text{mah}}(x_i, x_j) = \sqrt{(x_i - x_j)^T S^{-1} (x_i - x_j)}$$

其中 $S$ 为协方差矩阵。当 $S = I$ 时退化为欧氏距离。

## 核心概念 (Key Concepts)

| 概念 | 英文 | 含义 |
|------|------|------|
| 簇 | Cluster | 样本子集，内部相似、外部相异 |
| 原型 | Prototype | 代表簇的样本点或分布参数 |
| 类间距离 | Inter-cluster distance | 单连接/全连接/平均连接 |
| 轮廓系数 | Silhouette coefficient | $s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\}}$ |
| 不确定性 | Uncertainty | 无监督场景缺乏ground truth |
| 内在结构 | Intrinsic structure | 数据的低维流形或分布模式 |

**聚类有效性指标**：

内部指标（不利用外部标签）：
- Davies-Bouldin Index (DBI)：$\text{DBI} = \frac{1}{k}\sum_{i=1}^{k} \max_{j \neq i} \frac{\text{avg}(C_i) + \text{avg}(C_j)}{\text{dist}(\mu_i, \mu_j)}$，越小越好
- Dunn Index (DI)：$\text{DI} = \min_i \min_{j \neq i} \frac{\text{dist}(C_i, C_j)}{\max_l \text{diam}(C_l)}$，越大越好

外部指标（利用外部标签评估）：
- Jaccard系数：$\text{JC} = \frac{a}{a + b + c}$
- FMI：$\text{FMI} = \sqrt{\frac{a}{a+b} \cdot \frac{a}{a+c}}$
- Rand指数：$\text{RI} = \frac{2(a+d)}{m(m-1)}$

其中 $a$ 是同类且同簇的样本对数，$b$ 是同类不同簇，$c$ 是不同类同簇，$d$ 是不同类也不同簇。

## 算法步骤 (Algorithm Steps)

### k-means算法（Algorithm 14.2, 李航）

1. 随机选择 $k$ 个样本作为初始均值向量 $\mu_1, \ldots, \mu_k$
2. **重复**：
   - 对每个样本 $x_i$，计算其到各均值向量的距离，分配到最近的簇：$\lambda_i = \arg\min_l \|x_i - \mu_l\|_2^2$
   - 对每个簇 $C_l$，重新计算均值向量：$\mu_l = \frac{1}{|C_l|}\sum_{x \in C_l} x$
3. 直到均值向量不再变化（或SSE变化小于阈值）

时间复杂度：$O(m \cdot n \cdot k \cdot t)$，其中 $t$ 是迭代次数。

### 凝聚层次聚类（Algorithm 14.1, 李航）

1. 将每个样本视为一个簇，共 $m$ 个簇
2. 计算簇间距离矩阵
3. **重复**：
   - 合并距离最近的两个簇
   - 更新距离矩阵
4. 直到簇数为1（形成完整树状图树状图 dendrogram）

簇间距离度量决定合并策略：单连接（最近点距离）偏好细长簇，全连接（最远点距离）偏好紧凑球状簇，平均连接是折中。

### LVQ（学习向量量化）

LVQ假设有标记数据辅助。算法流程：
1. 初始化一组原型向量
2. 随机采样，找最近的原型向量
3. 若同类则拉近原型：$p' = p + \eta (x - p)$
4. 若异类则推远原型：$p' = p - \eta (x - p)$

LVQ学得的原型向量落在类别的Voronoi划分区域内，形成良好的分类边界。

## 直观理解 (Intuition)

**k-means像是领地划分游戏**：$k$ 个"首都"（均值向量）不断移动到自己领地的几何中心，居民（样本）总是效忠最近的首都。如果初始首都扎堆在大都市（密集区域），算法可能忽略边远村落。

**DBSCAN像是人口密度图**：在热力图上圈出热力区域——每个热力区是一个簇，冷区是噪声。它不预设簇的形状（可以是环、弯月），只关心密度连续性。

**层次聚类像是族谱**：一层层往上追溯，个体→家庭→家族→氏族，从树状图中可选取任意层次的切分。

**GMM是软聚类**：每个样本不是"非此即彼"而是"亦此亦彼"，用概率分布（阴影）描述归属的不确定性，像统计学家的聚类。

## 常见误区 (Anti-patterns)

1. **k-means + 非球状数据**：若数据呈弯月形或环形，k-means的Voronoi划分天生失败。这是对距离度量和目标函数的误配。
2. **k-means无特征标准化**：若某特征范围为[0, 1]，另一特征为[0, 1000]，则距离完全由后者主导。使用z-score标准化至关重要。
3. **忽略DBSCAN参数选择**：$\epsilon$ 和MinPts的选择极其关键。通用经验：MinPts $\geq$ 维数+1；$\epsilon$ 可通过k-距离图（k-distance graph）的拐点选择。
4. **GMM的协方差矩阵退化**：样本数少、维数高时，使用全协方差矩阵容易秩亏无法求逆。优先使用对角协方差或球形协方差。
5. **对k的"一选就定"心态**：应使用肘部法则（elbow method）结合轮廓系数（silhouette coefficient）综合判断，k不是先验已知的参数。
6. **混淆"欧氏距离不适用"与"没标准化"**：有时标准化后欧氏距离就能工作，不一定需要马氏距离。

## 代码要点 (Code Essentials)

```python
# k-means核心实现
def kmeans(X, k, max_iters=100):
    centroids = X[np.random.choice(len(X), k, replace=False)]
    for _ in range(max_iters):
        # E步：分配每个点到最近的centroid
        distances = np.linalg.norm(X[:, None] - centroids, axis=2)
        labels = np.argmin(distances, axis=1)
        # M步：更新centroids
        new_centroids = np.array([X[labels == j].mean(axis=0) for j in range(k)])
        if np.allclose(centroids, new_centroids):
            break
        centroids = new_centroids
    return labels, centroids
```

关键实践要点：
- 使用 `sklearn.cluster.KMeans` 时，`n_init` 参数做多次随机初始化，选SSE最小的结果
- `KMeans++` 初始化（`init='k-means++'`）理论上有 $O(\log k)$ 近似比的保证
- DBSCAN用 `sklearn.cluster.DBSCAN`，注意 `eps` 的选择
- 层次聚类用 `sklearn.cluster.AgglomerativeClustering`，`linkage` 可选 'ward'/'complete'/'average'
- GMM用 `sklearn.mixture.GaussianMixture`，`covariance_type` 可选 'full'/'tied'/'diag'/'spherical'
- 肘部法则：绘制 $k$ vs SSE图，找拐点
- 轮廓系数：`sklearn.metrics.silhouette_score`

## 实例分析 (Worked Example)

考虑二维数据集（来自西瓜书表9.1）：30个样本，2个特征（密度、含糖率），使用k-means聚类（$k=3$）。

初始化均值向量：随机选择3个样本点。第一轮后样本被分配调整，均值向量移动。经过3-4轮迭代后迭代终止，样本被分为3组：低密度低糖、中等水平、高密度高糖。轮廓系数约0.65，指示聚类质量合理。对比不同的 $k$ 值（2到5），$k=3$ 在肘部法则拐点处且轮廓系数最大。

同样的数据用DBSCAN（$\epsilon=0.15$, MinPts=5）：发现3个簇，另识别2个噪声点——这些点在特征空间中远离核心密度区域，DBSCAN自动将它们排除在簇外。

## 关键结论 (Key Takeaways)

1. 聚类是无监督学习的核心任务，关键挑战在于相似度度量和簇数量确定
2. k-means简单高效但只适用于球状簇；KMeans++初始化能改善结果质量
3. GMM-EM提供软聚类和概率解释，k-means是其退化特例
4. DBSCAN发现任意形状簇且对噪声鲁棒，但参数选择需谨慎
5. 层次聚类提供多粒度结构视图，无需预设 $k$
6. 距离度量的选择是聚类成败的前提——标准化、可视化和领域知识不可替代
7. 无单一最优聚类算法——需根据数据结构、任务目标和指标验证选择合适的算法

## 章节连接 (Connects To)

- **Ch10 降维**：聚类前常需降维以缓解维数灾难，高维空间中距离趋于均匀化
- **Ch11 特征选择**：无关特征会稀释相似度度量的有效性，过滤无用特征可提升聚类质量
- **Ch14 半监督学习**：半监督聚类利用少量标签信息引导聚类过程
- **Ch18 概率图模型**：GMM可视为含有离散隐变量的PGM，EM的数学框架是通用的
- **Ch19 EM算法**：GMM-EM是EM理论的最典型应用实例
