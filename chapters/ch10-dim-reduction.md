# Chapter 10: 降维与度量学习 / Dimensionality Reduction and Metric Learning

## 核心概念 (Core Idea)

高维数据中，随着维数增加，样本的欧氏距离趋于均匀化（维数灾难），且计算和存储开销急剧增长。降维（dimensionality reduction）通过某种数学变换将原始高维空间映射到一个低维子空间，同时在最大程度上保留数据的内在结构信息。其数学表述为：给定样本矩阵 $X \in \mathbb{R}^{m \times n}$，寻找映射 $f: \mathbb{R}^n \to \mathbb{R}^d$（$d < n$），得到低维表示 $Z \in \mathbb{R}^{m \times d}$。

与降维紧密相连的是**度量学习（metric learning）**——与其降维到欧氏空间再计算距离，不如直接学习一个合适的距离度量函数。

## 融合框架 (Integrated Frameworks)

降维方法可按三条主线组织：

1. **特征值分解类**（MDS, PCA, KPCA）：基于样本协方差矩阵或Gram矩阵的谱分解，主方向对应最大方差
2. **流形学习类**（Isomap, LLE, Laplacian Eigenmaps, t-SNE）：假设数据位于嵌入高维空间的低维流形上，通过保持局部结构实现非线性降维
3. **度量学习类**（NCA, LMNN）：显式学习马氏距离的参数矩阵

李航书从SVD的严格数学定义出发，将PCA视为SVD的直接推论——这为理解降维提供了坚实的线性代数基础。

## 理论推导 (Theoretical Derivation)

### SVD 奇异值分解

（李航第15章）对任意矩阵 $A \in \mathbb{R}^{m \times n}$，存在正交矩阵 $U \in \mathbb{R}^{m \times m}$、$V \in \mathbb{R}^{n \times n}$ 和对角矩阵 $\Sigma \in \mathbb{R}^{m \times n}$ 使得：

$$A = U \Sigma V^T$$

其中 $\Sigma$ 对角线上是奇异值 $\sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_r > 0$，$r = \text{rank}(A)$。

**几何解释**：SVD将线性变换 $A$ 分解为三步——$V^T$（旋转）、$\Sigma$（缩放）、$U$（旋转）。奇异值 $\sigma_i$ 是变换后在各正交方向上的缩放因子。

**紧奇异值分解**：$A = U_r \Sigma_r V_r^T$，$U_r \in \mathbb{R}^{m \times r}$ 取前 $r$ 列。**截断奇异值分解**：$A_d = U_d \Sigma_d V_d^T$（$d < r$），这是在Frobenius范数意义下的最优 $d$ 秩近似——Eckart-Young定理。

### PCA 主成分分析

（李航第16章，西瓜书10.3节）PCA有两种等价推导：

**最大方差视角（总体PCA）**：寻找投影方向 $w$ 使投影后方差最大：

$$\max_w \frac{1}{m}\sum_{i=1}^{m} \left(w^T x_i - w^T \bar{x}\right)^2 = w^T S w$$

约束 $\|w\| = 1$。用拉格朗日乘子法：$S w = \lambda w$。即 $w$ 是协方差矩阵 $S = \frac{1}{m}X^T X$ 的特征向量，$\lambda$ 是对应特征值。选前 $d$ 个最大特征值对应的特征向量构成投影矩阵 $W = [w_1, w_2, \ldots, w_d]$，低维表示为 $Z = X W$。

**最小重构误差视角**：寻找投影矩阵 $W$ 使得重构误差最小：

$$\min_W \|X - X W W^T\|_F^2, \quad \text{s.t. } W^T W = I$$

两个优化问题等价，最优解都是取前 $d$ 个最大特征值对应的特征向量。

**SVD求解PCA**（算法16.1，李航）：对中心化后的 $X$ 做截断SVD：$X = U \Sigma V^T$，则 $W = V_d$（右奇异向量的前 $d$ 列），$Z = U_d \Sigma_d$（或直接用 $X V_d$）。SVD方法数值更稳定，不需要显式计算 $X^T X$。

**累计方差贡献率**（确定 $d$）：

$$\eta_d = \frac{\sum_{i=1}^{d} \lambda_i}{\sum_{i=1}^{n} \lambda_i}$$

通常选择使 $\eta_d \geq 85\%$（或90%、95%）的最小 $d$。

### KPCA 核主成分分析

当数据线性不可分时，通过核函数 $\kappa(x_i, x_j) = \phi(x_i)^T \phi(x_j)$ 隐式映射到高维特征空间再做PCA。不显式计算 $\phi(x_i)$，而是对核矩阵 $K_{ij} = \kappa(x_i, x_j)$ 做中心化 $\tilde{K} = K - \frac{1}{m}11^T K - \frac{1}{m}K 11^T + \frac{1}{m^2}11^T K 11^T$，然后对 $\tilde{K}$ 做特征值分解取前 $d$ 个特征向量。

### MDS 多维缩放

目标：在低维空间中保持样本间的距离不变。对距离矩阵 $D$（$D_{ij} = \|x_i - x_j\|$），构造内积矩阵 $B = -\frac{1}{2}J D^{(2)} J$，其中 $J = I - \frac{1}{m}11^T$ 是中心化矩阵。对 $B$ 做特征值分解 $B = V \Lambda V^T$，则 $Z = V_d \Lambda_d^{1/2}$。若距离来自欧氏空间，则 $m-1$ 维内零误差可完美恢复。

### Isomap 等距映射

将MDS中的欧氏距离替换为**测地线距离**（geodesic distance）。算法三步：①构建近邻图（$k$-NN或$\epsilon$-球）；②用Dijkstra或Floyd算法计算所有点对间的最短路径距离作为测地线距离估计；③在估计的测地线距离矩阵上运行MDS。参数选择：$k$ 太小会导致图不连通，太大则短路（short-circuit）破坏流形结构。

### LLE 局部线性嵌入

假设每个样本可由其近邻线性重构，在低维空间中保持同样的重构权重。两步：
1. 计算重构权重 $w_{ij}$：$\min_w \sum_i \|x_i - \sum_{j \in Q_i} w_{ij} x_j\|^2$，约束 $\sum_j w_{ij} = 1$
2. 固定权重求低维嵌入 $z_i$：$\min_z \sum_i \|z_i - \sum_{j \in Q_i} w_{ij} z_j\|^2$

第二步等价于稀疏矩阵 $M = (I-W)^T (I-W)$ 的特征值分解（取第2到第 $d+1$ 小的特征向量）。

### t-SNE

t-SNE 用KL散度度量高维概率分布 $p_{ij}$ 与低维概率分布 $q_{ij}$ 的差异：

$$C = \sum_{i \neq j} p_{ij} \log \frac{p_{ij}}{q_{ij}}$$

高维使用高斯核、低维使用t分布（$q_{ij} \propto (1 + \|z_i - z_j\|^2)^{-1}$）——t分布的长尾特性解决了拥挤问题（crowding problem）。梯度下降优化非凸目标，不同随机种子结果可能不同。

### 度量学习与NCA

度量学习旨在学习马氏距离 $d_M(x_i, x_j) = (x_i - x_j)^T M (x_i - x_j)$ 中的矩阵 $M$，其中 $M = L^T L$（半正定保证距离非负）。NCA（近邻成分分析）使用可微的留一法kNN分类准确率作为目标，通过softmax将距离转为概率：

$$p_{ij} = \frac{\exp(-\|Lx_i - Lx_j\|^2)}{\sum_{k \neq i} \exp(-\|Lx_i - Lx_k\|^2)}$$

优化目标为最大化正确分类概率 $\sum_i \sum_{j \in \Omega_i} p_{ij}$，梯度下降优化 $L$。

## 核心概念

| 概念 | 公式/含义 |
|------|------|
| 奇异值 | $A = U\Sigma V^T$ 中对角元素 $\sigma_i$ |
| 主成分 | 协方差矩阵的特征向量，数据方差最大的方向 |
| 流形 | 局部与欧氏空间同胚的拓扑空间 |
| 测地线距离 | 流形上两点间沿曲面"行走"的最短路径长度 |
| 重构误差 | $\|X - ZW^T\|_F^2$，衡量降维后信息损失 |
| 核技巧 | $\kappa(x_i, x_j) = \phi(x_i)^T \phi(x_j)$，隐式高维映射 |
| 马氏距离 | $(x_i - x_j)^T M (x_i - x_j)$，$M$ 半正定 |

## 算法步骤 (Algorithm Steps)

### PCA via SVD（算法16.1，李航）

1. 计算样本均值 $\bar{x} = \frac{1}{m}\sum_i x_i$，中心化：$X_c = X - 1\bar{x}^T$
2. 对 $X_c$ 做截断SVD：$X_c = U_d \Sigma_d V_d^T$，取前 $d$ 个奇异值
3. 投影矩阵：$W = V_d \in \mathbb{R}^{n \times d}$
4. 低维表示：$Z = X_c \cdot V_d = U_d \Sigma_d \in \mathbb{R}^{m \times d}$
5. 重构：$\hat{X} = Z W^T + 1\bar{x}^T$

### t-SNE算法概要

1. 计算高维联合概率 $p_{ij}$（高斯核，perplexity控制带宽）
2. 随机初始化低维嵌入 $Y$
3. 迭代直到收敛：
   - 计算低维联合概率 $q_{ij}$（t分布）
   - 计算KL散度梯度：$\frac{\partial C}{\partial y_i} = 4\sum_j (p_{ij} - q_{ij})(y_i - y_j)(1+\|y_i-y_j\|^2)^{-1}$
   - 动量梯度下降更新 $Y$

## 直观理解 (Intuition)

**PCA像是给数据拍"最佳角度"的快照**。高维数据就像一块3D水晶——从某个角度（第一主成分）看去，它最宽（方差最大）；从正交的另一个角度（第二主成分）看去，宽度次之。PCA就是找出这些最佳角度，然后只保留最重要的几个。

**Isomap像是把弯曲的地图展平**。想象一张地球表面地图印在纸上——如果用欧氏距离计算北京到纽约，你会穿过纸的内部（在三维空间中的最短路径）。但实际的旅行路径是沿地球表面走的大圆弧。Isomap通过建图+最短路径来估计这个"沿表面"的真实距离。

**LLE像是保留衣服的缝纫关系**。一件衣服摊开是2D的布，穿在身上是3D的——但每个布料点和其邻居之间的关系（缝合方式）不变。LLE通过保持局部线性重构权重，实现流形展开。

**t-SNE像是社交网络的可视化压缩**：高维中"朋友"（近邻）在低维中也必须靠在一起，而"陌生人"则可以分散到任何位置。

## 常见误区 (Anti-patterns)

1. **PCA前不中心化/标准化**：PCA对变量尺度敏感。必须中心化，通常也应标准化（除以标准差）。
2. **盲目用PCA作分类预处理**：PCA只保留方差最大方向，此方向未必包含判别信息。LDA等有监督降维方法更适合分类任务。
3. **Isomap中的"短路"问题**：$k$ 近邻太大时，非流形上的近邻点被错误连接，测地线估计失效。
4. **LLE的"孔洞"和"翻转"**：近邻数过小流形分裂，过大则全局结构不展开。
5. **t-SNE用于距离/密度分析**：t-SNE只保持局部邻域结构，全局距离和簇大小无意义（t分布改变了尺度）。t-SNE仅适用于**可视化**。
6. **主成分数 $d$ 的随意选择**：应使用累计方差贡献率、交叉验证重构误差或下游任务性能来指导选择。
7. **混淆SVD的三种形式**：全SVD无信息损失，紧SVD去掉零奇异值，截断SVD是有损的低秩近似。

## 代码要点 (Code Essentials)

```python
# PCA via SVD
from sklearn.decomposition import PCA
pca = PCA(n_components=0.95)  # 保留95%方差
X_pca = pca.fit_transform(X_scaled)
# 主成分: pca.components_, 方差比: pca.explained_variance_ratio_

# KPCA
from sklearn.decomposition import KernelPCA
kpca = KernelPCA(n_components=2, kernel='rbf', gamma=0.1)
X_kpca = kpca.fit_transform(X)

# MDS
from sklearn.manifold import MDS
mds = MDS(n_components=2, dissimilarity='precomputed')
Z = mds.fit_transform(dist_matrix)

# Isomap
from sklearn.manifold import Isomap
isomap = Isomap(n_neighbors=10, n_components=2)
Z_iso = isomap.fit_transform(X)

# t-SNE
from sklearn.manifold import TSNE
tsne = TSNE(n_components=2, perplexity=30, random_state=42)
Z_tsne = tsne.fit_transform(X)
```

- PCA的 `explained_variance_ratio_` 绘制累计图以确定主成分数
- t-SNE的 `perplexity` 是关键参数，通常在5-50之间，影响全局/局部平衡
- Isomap的 `n_neighbors` 不宜太大以避免短路
- 大数据的PCA可用 `IncrementalPCA` 或 `TruncatedSVD`

## 实例分析 (Worked Example)

对西瓜书表9.1的30个西瓜样本（密度、含糖率），增加含糖率平方和交互项等扩展到6维后降维到2维：

- **PCA**：第一主成分占73.2%方差（主要是密度+含糖率线性组合），第二主成分占19.5%，累计92.7%
- **KPCA（RBF核）**：在两维投影中更清晰地分离了低密度低糖、中等、高密度高糖三组
- **Isomap**（k=5）：两维嵌入保持了数据沿唯一曲率方向展开的结构
- 各方法降维后运行k-means，KPCA+降维的组合聚类纯度最高

## 关键结论 (Key Takeaways)

1. 维数灾难是高维数据中距离度量失效的根源，降维是缓解手段之一
2. PCA是最经典的线性降维方法，可用最大方差/最小重构误差两种视角理解
3. SVD是PCA的数值实现基础，截断SVD给出Frobenius范数意义下的最优低秩近似
4. 流形学习方法（Isomap、LLE、t-SNE）能处理非线性结构但参数敏感
5. t-SNE仅适合可视化，不能用于保距分析
6. 度量学习将降维延伸为"学距离"，NCA是代表性方法
7. 降维方法的选择取决于目标：可视化用t-SNE/UMAP、预处理用PCA、流形分析用Isomap/LLE

## 章节连接 (Connects To)

- **Ch5 支持向量机**：核方法（RBF核）将数据映射到高维再生核希尔伯特空间，与KPCA原理相通
- **Ch9 聚类**：高维空间中距离失效，聚类前降维可显著改善效果
- **Ch11 特征选择**：特征选择是降维的特例（坐标轴子集选择），降维是连续的泛化
- **Ch15 矩阵分解**：SVD和NMF等矩阵分解方法是降维的核心数学工具
- **Ch17 表示学习**：深度学习中的自编码器可看作PCA的非线性推广
