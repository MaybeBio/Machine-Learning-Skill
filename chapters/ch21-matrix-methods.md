# Chapter 21: 矩阵分解与图算法 / Matrix Factorization and Graph Algorithms

## 核心概念 (Core Idea)

矩阵分解将高维数据压缩为低维潜在结构，是连接线性代数与无监督学习的核心桥梁。奇异值分解（SVD）提供最优低秩近似，PageRank通过随机游走在有向图中发现结点重要性，二者共同构成了数据降维与图分析的理论基础。

## 融合框架 (Integrated Frameworks)

（李航）将无监督学习方法分为四类基本问题：聚类、降维、话题分析、图分析。矩阵分解（SVD、NMF）同时服务于降维和话题分析，PageRank服务于图分析中的链接分析。

（西瓜书）中以特征值分解和SVD作为降维的数学基础，但未单独成章讨论SVD的理论性质。（李航）则给出了严格的SVD定义、紧奇异值分解与截断奇异值分解的区分、几何解释以及最优近似定理的完整证明。

## 理论推导 (Theoretical Derivation)

### 15.1 奇异值分解 (SVD)

**定义（完全奇异值分解）**：对任意 $m \times n$ 实矩阵 $A$，存在 $m$ 阶正交矩阵 $U$ 和 $n$ 阶正交矩阵 $V$，使得

$$A = U \Sigma V^{\mathrm{T}}$$

其中 $\Sigma = \mathrm{diag}(\sigma_1, \sigma_2, \ldots, \sigma_p)$，$p = \min\{m, n\}$，且奇异值满足 $\sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_p \geq 0$。

**紧奇异值分解**：若 $A$ 的秩为 $r$，则 $A = U_r \Sigma_r V_r^{\mathrm{T}}$，其中 $\Sigma_r$ 是 $r \times r$ 对角阵，只保留正奇异值。这是与原始矩阵等秩的无损压缩。

**截断奇异值分解**：取前 $k < r$ 个最大奇异值及对应的奇异向量：

$$A \approx U_k \Sigma_k V_k^{\mathrm{T}}$$

这是低秩有损压缩，也是实际应用中最常用的形式。

**外积展开式**：

$$A = \sigma_1 u_1 v_1^{\mathrm{T}} + \sigma_2 u_2 v_2^{\mathrm{T}} + \cdots + \sigma_n u_n v_n^{\mathrm{T}}$$

每个外积项 $u_k v_k^{\mathrm{T}}$ 是秩为1的 $m \times n$ 矩阵，奇异值 $\sigma_k$ 充当该项的权重。

**几何解释**：SVD对应三个连续的线性变换：
1. $V^{\mathrm{T}}$：旋转/反射（基于行空间的标准正交基）
2. $\Sigma$：沿坐标轴的缩放（缩放因子即奇异值）
3. $U$：再次旋转/反射（基于列空间的标准正交基）

直观上，矩阵 $A$ 将单位球 $\{x: \|x\|_2 = 1\}$ 映射为超椭球，该椭球的半轴长度即奇异值 $\sigma_i$，方向即 $U$ 的列向量。

**重要性质**：
- $A^{\mathrm{T}}A = V(\Sigma^{\mathrm{T}}\Sigma)V^{\mathrm{T}}$，即 $V$ 的列是 $A^{\mathrm{T}}A$ 的特征向量，$\sigma_i^2$ 是其特征值
- $AA^{\mathrm{T}} = U(\Sigma\Sigma^{\mathrm{T}})U^{\mathrm{T}}$，即 $U$ 的列是 $AA^{\mathrm{T}}$ 的特征向量
- $\|A\|_F = \sqrt{\sum_i \sigma_i^2}$（弗罗贝尼乌斯范数）

**最优近似定理（Eckart-Young）**：在秩不超过 $k$ 的所有 $m \times n$ 矩阵中，截断SVD $A_k = U_k \Sigma_k V_k^{\mathrm{T}}$ 是 $A$ 在弗罗贝尼乌斯范数意义下的最优近似：

$$\|A - A_k\|_F = \sqrt{\sigma_{k+1}^2 + \cdots + \sigma_r^2} = \min_{\mathrm{rank}(X) \leq k} \|A - X\|_F$$

### 17.2 潜在语义分析 (LSA)

**动机**：单词向量空间模型（VSM）基于单词共现计算文本相似度，但无法处理同义词（synonymy）和多义词（polysemy）。例如，当"airplane"和"aircraft"表示同一概念时，VSM会将它们视为正交维度。

**核心思想**：将文本从高维稀疏的单词向量空间，通过线性变换映射到低维稠密的话题向量空间（latent semantic space）。

**模型形式**：给定单词-文本矩阵 $X \in \mathbb{R}^{m \times n}$，寻找单词-话题矩阵 $T$ 和话题-文本矩阵 $Y$，使得：

$$X \approx TY$$

其中 $T \in \mathbb{R}^{m \times k}$，$Y \in \mathbb{R}^{k \times n}$，$k \ll \min(m, n)$。

**SVD实现**：$X \approx U_k \Sigma_k V_k^{\mathrm{T}}$，取：
- 话题向量空间：$T = U_k$（每个列向量 $u_l$ 表示一个话题在单词上的分布）
- 文本的话题表示：$Y = \Sigma_k V_k^{\mathrm{T}}$（每列表示一个文本在话题上的坐标）

**查询匹配**：对于查询向量 $q$（视作伪文本），其在话题空间的表示为 $q_{\text{lsa}} = \Sigma_k^{-1} U_k^{\mathrm{T}} q$，然后计算 $q_{\text{lsa}}$ 与 $Y$ 各列的余弦相似度进行检索。

**非负矩阵分解（NMF）**：作为LSA的替代方法。寻找非负矩阵 $W \geq 0$ 和 $H \geq 0$，使得 $X \approx WH$。NMF的优点是结果具有直观的"部分组合整体"解释——话题向量和文本向量都非负，对应"伪概率分布"。NMF的优化使用乘法更新规则：

$$H_{lj} \leftarrow H_{lj} \frac{(W^{\mathrm{T}}X)_{lj}}{(W^{\mathrm{T}}WH)_{lj}}, \quad W_{il} \leftarrow W_{il} \frac{(XH^{\mathrm{T}})_{il}}{(WHH^{\mathrm{T}})_{il}}$$

### 21.3 PageRank 算法

**基本定义**：给定有向图 $G = (V, E)$，PageRank值 $R(u)$ 定义为随机游走者的平稳分布：

$$R(u) = d \sum_{v \in B_u} \frac{R(v)}{L(v)} + \frac{1-d}{N}$$

其中 $B_u$ 是指向 $u$ 的结点集合，$L(v)$ 是结点 $v$ 的出度，$d \in (0, 1)$ 是阻尼因子（通常 $d = 0.85$），$N$ 是结点总数。

**随机冲浪者模型**：冲浪者以概率 $d$ 沿当前页面的超链接随机跳转，以概率 $1-d$ 随机跳到任意页面。PageRank值即冲浪者长期停留在某页面的概率。

**幂法计算**：PageRank向量 $R$ 是转移矩阵 $M$ 的主特征向量（对应特征值1的平稳分布）。幂法迭代：

$$R^{(t+1)} = d M^{\mathrm{T}} R^{(t)} + \frac{1-d}{N} \mathbf{1}$$

当 $\|R^{(t+1)} - R^{(t)}\|_1 < \epsilon$ 时停止。

**Dead Ends 与 Spider Traps**：
- **Dead End**：没有出链的结点。处理方式：将dead end结点的出链视为指向所有结点的均匀链接。
- **Spider Trap**：一组结点互相链接但不指向组外。阻尼因子 $(1-d)$ 确保了随机跳转可以逃脱陷阱。

**个性化PageRank**：将随机跳转均匀分布替换为偏向特定结点集的分布，使PageRank值反映相对于特定查询/用户的结点重要性。

## 核心概念 (Key Concepts)

- **奇异值 (Singular Value)**：$\sigma_i = \sqrt{\lambda_i(A^{\mathrm{T}}A)}$，度量矩阵沿第 $i$ 个正交方向的"拉伸强度"
- **紧SVD**：保留所有正奇异值的分解，等秩无损
- **截断SVD**：仅保留前 $k$ 个最大奇异值，低秩有损
- **弗罗贝尼乌斯范数**：$\|A\|_F^2 = \sum_{i,j} a_{ij}^2 = \sum_i \sigma_i^2$，平方损失的自然度量
- **话题向量空间**：由 $U_k$ 的列张成的低维空间，每个话题是单词的线性组合
- **稳态分布**：马尔可夫链的极限分布 $\pi = \pi P$，对应PageRank向量
- **阻尼因子**：控制随机跳转概率的 $(1-d)$ 项，确保马尔可夫链的遍历性

## 算法步骤 (Algorithm Steps)

### SVD截断分解
1. 构建数据矩阵 $X$（如单词-文本矩阵）
2. 计算 $X^{\mathrm{T}}X$ 的特征值和特征向量
3. 取前 $k$ 个最大特征值 $\lambda_1 \geq \cdots \geq \lambda_k$，奇异值 $\sigma_i = \sqrt{\lambda_i}$
4. 右奇异向量 $v_i$ = 对应 $\lambda_i$ 的单位特征向量
5. 左奇异向量 $u_i = \frac{1}{\sigma_i} X v_i$
6. 构造 $U_k, \Sigma_k, V_k$，得 $X \approx U_k \Sigma_k V_k^{\mathrm{T}}$

### LSA查询匹配
1. 对单词-文本矩阵 $X$ 做截断SVD得 $U_k, \Sigma_k, V_k$
2. 查询向量 $q$ 映射到话题空间：$q_{\text{lsa}} = \Sigma_k^{-1} U_k^{\mathrm{T}} q$
3. 文本在话题空间：$d_j = (\Sigma_k V_k^{\mathrm{T}})_j$（第 $j$ 列）
4. 计算余弦相似度：$\text{sim}(q, d_j) = \frac{q_{\text{lsa}} \cdot d_j}{\|q_{\text{lsa}}\| \|d_j\|}$

### PageRank幂法迭代
1. 初始化 $R^{(0)} = [1/N, \ldots, 1/N]$
2. 迭代 $R^{(t+1)} = d M^{\mathrm{T}} R^{(t)} + \frac{1-d}{N} \mathbf{1}$
3. 检查收敛 $\|R^{(t+1)} - R^{(t)}\|_1 < \epsilon$
4. 输出 $R$ 作为各结点PageRank值

## 直观理解 (Intuition)

**SVD的几何图像**：想象一个单位球体。任意矩阵 $A$ 作用于这个球体上，会将其变形为一个超椭球。SVD告诉我们这个椭球的形状——半轴的方向由 $U$ 的列给出，半轴的长度由奇异值 $\sigma_i$ 给出。截断SVD相当于"扔掉"最短的几个半轴，在损失最小信息的前提下将椭球压缩到更低维度。

**LSA与人类认知**：当人读到"苹果发布了新手机"和"iPhone销量创纪录"时，能理解两者相关，尽管它们没有共同单词。LSA通过发现"apple"和"iPhone"在同一话题维度上的共现模式来模拟这种认知——它学到的不是单词的匹配，而是话题的匹配。

**PageRank与学术引用**：一篇论文被高引用论文引用比被低引用论文引用更有分量——这是PageRank的核心理念。阻尼因子相当于"读者有时随机选一篇论文阅读"，防止引用环路垄断所有影响力。

## 常见误区 (Anti-patterns)

- **误区：SVD要求矩阵是方阵**。SVD适用于任意形状的 $m \times n$ 矩阵，这是它相比特征值分解的核心优势。
- **误区：话题个数 $k$ 越大越好**。$k$ 过大会引入噪声，过小会丢失语义区分。通常通过交叉验证或困惑度（perplexity）选取。
- **误区：PageRank值高的页面一定是好页面**。PageRank只度量链接结构中的"声望"，不度量内容质量。低质量的"链接农场"也可以获得高PageRank。
- **误区：NMF和SVD等价**。NMF强制非负约束，产生基于部分（parts-based）的表示，更适合解释为"话题是单词的加权组合"；SVD的正交约束使得话题之间线性无关。

## 代码要点 (Code Essentials)

```python
# 截断SVD
from sklearn.decomposition import TruncatedSVD
svd = TruncatedSVD(n_components=k)
X_reduced = svd.fit_transform(X)  # 文本在话题空间

# NMF
from sklearn.decomposition import NMF
nmf = NMF(n_components=k)
W = nmf.fit_transform(X)  # 单词-话题矩阵
H = nmf.components_        # 话题-文本矩阵

# PageRank (networkx)
import networkx as nx
pr = nx.pagerank(G, alpha=0.85)
```

## 实例分析 (Worked Example)

**LSA示例**（李航第17章）：9个文本、11个单词的集合。取 $k = 3$ 个话题，对单词-文本矩阵做截断SVD。第一列左奇异向量 $u_1$ 所有分量均为正，可解释为"通用话题"；$u_2$ 和 $u_3$ 有正有负，区分不同语义维度（如区分"投资金融"和"房产指南"）。查询"investing"将被映射到话题空间，与相关文本（即使不含该词）在该空间中相近。

## 关键结论 (Key Takeaways)

1. SVD是任意矩阵的最优低秩近似工具，截断SVD = 弗罗贝尼乌斯范数意义下的最优压缩
2. 紧SVD是无损的等秩分解，截断SVD是有损的降秩分解——区分两者至关重要
3. LSA通过SVD将稀疏的单词空间映射到稠密的话题空间，解决了同义词/多义词问题
4. NMF强制非负，结果具有"局部叠加构成整体"的直观解释
5. PageRank = 阻尼随机游走的平稳分布，幂法迭代是其标准计算方式
6. 无监督学习的本质是对数据矩阵进行"压缩"以发现潜在结构——聚类发掘纵向结构，降维发掘横向结构，话题分析同时发掘二者

## 章节连接 (Connects To)

- 前置: ch13-pca.md（PCA是SVD的直接应用） | ch09-clustering.md（聚类也是无监督学习方法）
- 后续: ch22-deep-learning.md（深度学习中的表示学习与LSA的话题学习理念相通）
