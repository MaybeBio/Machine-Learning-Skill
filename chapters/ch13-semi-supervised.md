# Chapter 13: 半监督学习 / Semi-Supervised Learning

## 核心概念 (Core Idea)

半监督学习让学习器不依赖外界交互，自动利用大量未标记样本提升学习性能，核心思想是借助未标记样本揭示的数据分布信息（簇结构、流形结构）来辅助有标记样本构建更优的分类器——本质是将数据分布的先验与稀少标记相结合。(西瓜书/南瓜书)

## 融合框架 (Integrated Frameworks)

半监督学习的四大方法族及其来源框架：

| 方法族 | 核心假设 | 代表算法 | 来源 |
|--------|---------|---------|------|
| 生成式方法 | 数据由同一生成模型产生 | EM + GMM | 西瓜书13.2 |
| 半监督SVM | 低密度分隔 | TSVM/S3VM | 西瓜书13.3 |
| 图半监督学习 | 相似样本相似标记 | 标记传播 | 西瓜书13.4 |
| 基于分歧的方法 | 多学习器分歧互补 | 协同训练 | 西瓜书13.5 |

三种相关学习范式：(西瓜书)
- **主动学习 (Active Learning)**: 与外界交互，选择性查询标记，目标是减少查询次数
- **纯半监督学习 (Pure SSL)**: 开放世界，训练中的未标记样本不是待预测数据
- **直推学习 (Transductive Learning)**: 封闭世界，未标记样本恰是待预测数据

## 理论推导 (Theoretical Derivation)

### 1. 生成式方法：高斯混合模型 + EM

假设所有样本由 $N$ 个高斯混合成分生成，每个类别对应一个成分：(西瓜书)

$$
p(\mathbf{x}) = \sum_{i=1}^{N} \alpha_i \cdot p(\mathbf{x} \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i) \tag{13.1}
$$

分类决策为最大化后验概率：

$$
f(\mathbf{x}) = \arg\max_{j \in \mathcal{Y}} \sum_{i=1}^{N} p(y=j \mid \Theta=i, \mathbf{x}) \cdot p(\Theta=i \mid \mathbf{x}) \tag{13.2}
$$

其中 $p(\Theta=i \mid \mathbf{x}) = \frac{\alpha_i \cdot p(\mathbf{x} \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)}{\sum_{i=1}^{N} \alpha_i \cdot p(\mathbf{x} \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)}$ 是可以利用未标记数据改进估计的关键项。(南瓜书推导)

完整对数似然（有标记 + 未标记）：(西瓜书)

$$
LL(D_l \cup D_u) = \sum_{(\mathbf{x}_j, y_j) \in D_l} \ln\left(\sum_{i=1}^{N} \alpha_i \cdot p(\mathbf{x}_j \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i) \cdot p(y_j \mid \Theta=i, \mathbf{x}_j)\right) + \sum_{\mathbf{x}_j \in D_u} \ln\left(\sum_{i=1}^{N} \alpha_i \cdot p(\mathbf{x}_j \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)\right) \tag{13.4}
$$

EM迭代：(西瓜书/南瓜书)
- **E步**: $\gamma_{ji} = \frac{\alpha_i \cdot p(\mathbf{x}_j \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)}{\sum_{i=1}^{N} \alpha_i \cdot p(\mathbf{x}_j \mid \boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)}$ — 计算未标记样本对各成分的隶属度
- **M步**: 利用 $\gamma_{ji}$ 和有标记样本联合更新 $\boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i, \alpha_i$

南瓜书给出完整的求导过程：令 $\frac{\partial LL}{\partial \boldsymbol{\mu}_i} = 0$ 得均值更新式（13.6），令 $\frac{\partial LL}{\partial \boldsymbol{\Sigma}_i} = 0$ 得协方差更新式（13.7），利用拉格朗日乘子法得混合系数更新式（13.8）。

### 2. 半监督SVM (S3VM / TSVM)

TSVM目标：寻找划分超平面最大化间隔，同时穿过数据低密度区域：(西瓜书)

$$
\begin{aligned}
\min_{\mathbf{w}, b, \hat{\mathbf{y}}, \boldsymbol{\xi}} \quad & \frac{1}{2} \|\mathbf{w}\|_2^2 + C_l \sum_{i=1}^{l} \xi_i + C_u \sum_{i=l+1}^{m} \xi_i \\
\mathrm{s.t.} \quad & y_i(\mathbf{w}^T \mathbf{x}_i + b) \geq 1 - \xi_i, \quad i=1,\ldots,l, \\
& \hat{y}_i(\mathbf{w}^T \mathbf{x}_i + b) \geq 1 - \xi_i, \quad i=l+1,\ldots,m, \\
& \xi_i \geq 0, \quad i=1,\ldots,m.
\end{aligned} \tag{13.9}
$$

其中 $C_u \ll C_l$ 初始，使有标记样本权重更高，逐步增大 $C_u$ 至等于 $C_l$。

南瓜书详细图解了 $\xi_i$ 的三种情况：(a) $0 < \xi_i < 1$：在间隔带内但位于自己标记一侧；(b) $1 < \xi_i < 2$：在间隔带内但位于相反侧；(c) $\xi_i > 2$：在间隔带外且位于相反侧。交换条件 $\xi_i + \xi_j > 2$ 且 $\hat{y}_i\hat{y}_j < 0$ 保证交换后目标函数下降。

类别平衡修正：(西瓜书)

$$
C_u^+ = \frac{u_-}{u_+} C_u^- \tag{13.10}
$$

### 3. 图半监督学习：标记传播

构建亲和矩阵 (高斯核)：(西瓜书)

$$
(\mathbf{W})_{ij} = \begin{cases}
\exp\left(\frac{-\|\mathbf{x}_i - \mathbf{x}_j\|_2^2}{2\sigma^2}\right), & \text{if } i \neq j; \\
0, & \text{otherwise.}
\end{cases} \tag{13.11}
$$

定义能量函数：(西瓜书/南瓜书)

$$
E(f) = \frac{1}{2} \sum_{i=1}^{m} \sum_{j=1}^{m} (\mathbf{W})_{ij} \big(f(\mathbf{x}_i) - f(\mathbf{x}_j)\big)^2 = \mathbf{f}^T (\mathbf{D} - \mathbf{W}) \mathbf{f} \tag{13.12}
$$

南瓜书详细展开：利用 $\mathbf{W}$ 对称性，将二次型展开后用分块矩阵 $\mathbf{f} = [\mathbf{f}_l; \mathbf{f}_u]$ 表示，得到解析解：

$$
\mathbf{f}_u = (\mathbf{D}_{uu} - \mathbf{W}_{uu})^{-1} \mathbf{W}_{ul} \mathbf{f}_l \tag{13.15}
$$

等价于 $(\mathbf{I} - \mathbf{P}_{uu})^{-1} \mathbf{P}_{ul} \mathbf{f}_l$，其中 $\mathbf{P} = \mathbf{D}^{-1}\mathbf{W}$。

**多分类标记传播**：定义标记矩阵 $\mathbf{F}$，迭代 $\mathbf{F}(t+1) = \alpha \mathbf{S} \mathbf{F}(t) + (1-\alpha) \mathbf{Y}$，其中 $\mathbf{S} = \mathbf{D}^{-\frac{1}{2}} \mathbf{W} \mathbf{D}^{-\frac{1}{2}}$。收敛解：(西瓜书/南瓜书)

$$
\mathbf{F}^* = (1-\alpha)(\mathbf{I} - \alpha \mathbf{S})^{-1} \mathbf{Y} \tag{13.20}
$$

南瓜书推导了极限形式：$\mathbf{F}(t) = (\alpha\mathbf{S})^t \mathbf{Y} + (1-\alpha)(\sum_{i=0}^{t-1}(\alpha\mathbf{S})^i) \mathbf{Y}$，由于 $\alpha \in (0,1)$ 且 $\mathbf{S}$ 的特征值在 $[-1,1]$，有 $\lim_{t \to \infty} (\alpha\mathbf{S})^t = 0$，结合等比级数得解。

正则化框架：(西瓜书)

$$
\min_{\mathbf{F}} \frac{1}{2} \left(\sum_{i,j=1}^{l+u} (\mathbf{W})_{ij} \left\|\frac{1}{\sqrt{d_i}} \mathbf{F}_i - \frac{1}{\sqrt{d_j}} \mathbf{F}_j\right\|^2 \right) + \mu \sum_{i=1}^{l} \|\mathbf{F}_i - \mathbf{Y}_i\|^2 \tag{13.21}
$$

第一项为光滑性正则项（拉普拉斯正则化），第二项为有标记样本的保真项。

## 核心概念 (Key Concepts)

- **聚类假设 (Cluster Assumption)**: 同一簇的样本属于同一类，决策边界应穿过数据低密度区域
- **流形假设 (Manifold Assumption)**: 数据分布在低维流形上，邻近样本拥有相似输出
- **平滑性假设 (Smoothness Assumption)**: 高密度区域中相近的点应有相似标记
- **低密度分隔 (Low-Density Separation)**: S3VM的核心，决策边界应避开高密度区域
- **伪标记 (Pseudo-Label)**: 半监督学习器为未标记样本赋予的临时标记
- **亲和矩阵 (Affinity Matrix)** $\mathbf{W}$: 图半监督学习中表示样本间相似度的矩阵
- **拉普拉斯矩阵 (Laplacian Matrix)** $\mathbf{L} = \mathbf{D} - \mathbf{W}$: 图光滑性的数学表达
- **视图 (View)**: 多视图数据的不同属性集，协同训练的基础
- **必连/勿连约束 (Must-Link / Cannot-Link)**: 半监督聚类的监督信息形式

## 算法步骤 (Algorithm Steps)

### TSVM算法 (图13.4)
1. 用 $D_l$ 训练初始 SVM
2. 用该 SVM 预测 $D_u$ 伪标记 $\hat{\mathbf{y}}$
3. 设 $C_u \ll C_l$
4. 在 $C_u < C_l$ 时循环：
   - 求解式(13.9)得 $(\mathbf{w}, b), \boldsymbol{\xi}$
   - 寻找异类且 $\xi_i + \xi_j > 2$ 的标记对，交换其标记并重新求解
   - $C_u = \min\{2C_u, C_l\}$

### 协同训练算法 (图13.6)
1. 为两个视图分别构建有标记训练集 $D_l^1, D_l^2$
2. 每轮迭代：
   - 在各视图上训练分类器 $h_1, h_2$
   - 各分类器在缓冲池 $D_s$ 中挑选置信度最高的正/反例
   - 将伪标记样本提供给对方视图作为新训练数据
   - 从 $D_u$ 补充缓冲池
3. 直至分类器不再变化或达到预设轮数

### 约束k均值算法 (图13.7)
标准k均值框架下，在分配样本到簇时逐簇检查是否违背 $\mathcal{M}$ 和 $\mathcal{C}$ 约束。

## 直观理解 (Intuition)

**种瓜的比喻**（西瓜书）：瓜农只告诉你三四个好瓜和五六个不好瓜，但地里还有几百个瓜。仅用几个样本训练不靠谱，但如果观察到未标记的瓜天然聚成几簇，而好瓜恰好集中在某簇中，就能有很大把握判断其他瓜的好坏。这就是聚类假设的直观体现——未标记数据的"簇结构"自带分类信息。

**图的染色扩散**：把样本看成图中的点，有标记样本已染色，未标记样本通过边的连接从邻居"吸收"颜色——最终所有点都被染色，形成了标记传播。

**协同训练的"互相教学"**：好比两个学生分别从图像和声音判断电影类型，各自把最有把握的判断告诉对方作为"参考答案"，反复互教，两人水平共同提高。

## 常见误区 (Anti-patterns)

- **误区：未标记数据越多越好**。如果模型假设错误（如假设高斯混合但实际并非如此），利用未标记数据反而会降低性能。这就是"安全半监督学习"的核心关切。(西瓜书/南瓜书)

- **误区：图半监督学习可以直接用于新样本预测**。图方法只能对构图时存在的节点进行传播，新样本需要重新构图或额外训练分类器。(西瓜书)

- **误区：协同训练必须有两个自然视图**。后续理论证明只需学习器间有显著分歧，不同算法、不同采样、不同参数设置都可产生有效分歧。(西瓜书)

- **误区：TSVM的伪标记修正一定收敛到全局最优**。TSVM只是局部搜索，目标函数非凸，可能陷入局部最优。(西瓜书)

## 代码要点 (Code Essentials)

```python
# 图半监督学习核心：标记传播矩阵计算
import numpy as np
from scipy.sparse.linalg import inv

def label_propagation(W, Y_labeled, alpha=0.99):
    """W: 亲和矩阵 (m×m), Y_labeled: 初始标记矩阵"""
    D = np.diag(np.sum(W, axis=1))
    D_inv_sqrt = np.diag(1.0 / np.sqrt(np.diag(D)))
    S = D_inv_sqrt @ W @ D_inv_sqrt
    m = W.shape[0]
    I = np.eye(m)
    F = (1 - alpha) * inv(I - alpha * S) @ Y_labeled
    return F
```

关键实现注意事项：
1. 亲和矩阵 $\mathbf{W}$ 的计算可用高斯核，$\sigma$ 的选择对性能影响大
2. TSVM实现中 $C_u$ 的逐步增大策略影响收敛质量
3. 协同训练中分类置信度的设计依赖于基学习器类型

## 实例分析 (Worked Example)

**西瓜数据集上的约束k均值**（西瓜书图13.8）：西瓜数据集4.0，设 $k=3$，添加必连约束 $\mathcal{M}$（如样本4与25、12与20同簇）和勿连约束 $\mathcal{C}$（如样本2与21、13与23异簇）。随机初始化均值向量 $\mathbf{x}_6, \mathbf{x}_{12}, \mathbf{x}_{27}$，约束k均值在5轮迭代后收敛，得到三个簇划分，必连和勿连约束均被满足。

**生成式方法的初始化**（南瓜书补充）：有标记样本可用于初始化GMM参数：
- $\alpha_i = l_i / |D_l|$
- $\boldsymbol{\mu}_i = \frac{1}{l_i} \sum_{(\mathbf{x}_j, y_j) \in D_l \land y_j=i} \mathbf{x}_j$
- $\boldsymbol{\Sigma}_i = \frac{1}{l_i} \sum_{(\mathbf{x}_j, y_j) \in D_l \land y_j=i} (\mathbf{x}_j - \boldsymbol{\mu}_i)(\mathbf{x}_j - \boldsymbol{\mu}_i)^\top$

## 关键结论 (Key Takeaways)

1. **半监督学习的有效性依赖于假设的正确性**：聚类假设和流形假设是数学基础，若数据不满足这些假设，半监督学习可能适得其反
2. **生成式方法简单但敏感**：模型假设必须准确，否则未标记数据降性能；但有标记数据极少时往往表现最好
3. **S3VM本质是SVM的非凸扩展**：寻找低密度分隔超平面，TSVM通过局部搜索迭代近似求解
4. **图方法的数学形式优美但扩展性差**：$O(m^2)$ 矩阵运算限制大规模应用，且天然是直推式的
5. **协同训练的关键是学习器分歧，而非多视图**：只需有显著差异的学习器，即可通过互教提升性能
6. **主动学习 != 半监督学习**：前者需要外部交互（查询），后者完全自动利用未标记数据

## 章节连接 (Connects To)

- **前置**: Ch6 SVM (S3VM的基础), Ch7 贝叶斯分类器 (生成式方法), Ch9 聚类 (高斯混合模型、k均值), Ch10 降维 (流形假设与流形学习)
- **后续**: Ch14 概率图模型 (生成式方法的图模型扩展), Ch16 强化学习 (学习范式对比)
- **互补**: 半监督学习与迁移学习、弱监督学习同属"利用有限/间接监督信息"的研究方向
