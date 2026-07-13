# Chapter 11: 特征选择与稀疏学习 / Feature Selection and Sparse Learning

## 核心概念 (Core Idea)

现实数据常包含冗余特征（redundant features）和无关特征（irrelevant features）。特征选择从给定的 $n$ 维特征集合中选择一个 $d$ 维（$d \ll n$）的子集，使得在这个子集上训练出的模型性能尽可能接近（甚至优于）使用全部特征。这对缓解维数灾难、降低过拟合风险、提高模型可解释性至关重要。

子集搜索是NP-hard的组合优化问题——从 $n$ 个特征中选 $d$ 个有 $\binom{n}{d}$ 种可能。因此必须用启发式搜索策略。子集评价则需衡量所选子集的信息量和判别力。

**稀疏学习（sparse learning）**将特征选择嵌入到模型训练过程中，通过引入稀疏正则化（如L1惩罚）迫使模型权重向量中的大部分分量为零，从而自动实现特征选择。稀疏学习的数学基础是LASSO和近端梯度下降（PGD）。

## 融合框架 (Integrated Frameworks)

特征选择方法的分类体系：

1. **过滤式（Filter）**：先用统计指标独立评价每个特征，过滤后训练模型。代表：Relief、Relief-F、互信息、卡方检验、相关系数。与后续学习器无关。
2. **包裹式（Wrapper）**：以学习器性能作为子集评价准则，直接为给定学习器选择最"量身定制"的特征子集。代表：LVW（Las Vegas Wrapper）。计算开销大但效果好。
3. **嵌入式（Embedded）**：在模型训练过程中自动完成特征选择。代表：L1/LASSO、弹性网（Elastic Net）、基于树模型的特征重要性。
4. **稀疏表示类**：字典学习（K-SVD）、压缩感知（compressed sensing）、矩阵补全——将稀疏性推广到一般表示问题。

## 理论推导 (Theoretical Derivation)

### Relief与Relief-F

Relief为每个特征赋予"相关统计量"，核心思想：好的特征应使同类最近邻距离近、异类最近邻距离远。对样本 $x_i$，找其同类最近邻 $x_{i,nh}$（near-hit）和异类最近邻 $x_{i,nm}$（near-miss），特征 $j$ 的权重更新：

$$\delta_j = \sum_i -\text{diff}(x_i^j, x_{i,nh}^j)^2 + \text{diff}(x_i^j, x_{i,nm}^j)^2$$

其中 $\text{diff}(x_a^j, x_b^j)$ 对连续特征为 $|x_a^j - x_b^j|$（归一化后），对离散特征为 $[x_a^j \neq x_b^j]$。

**Relief-F（多分类扩展）**：对多分类问题，找到每个异类的 $k$ 个最近邻，按类先验概率加权：

$$\delta_j = \sum_i \left(-\sum_{x \in \text{NH}} \text{diff}(x_i^j, x^j)^2 + \sum_{l \neq y_i} \frac{p_l}{1-p_{y_i}} \sum_{x \in \text{NM}_l} \text{diff}(x_i^j, x^j)^2\right)$$

### L1正则化与LASSO

LASSO（Least Absolute Shrinkage and Selection Operator）在线性回归的平方损失上施加L1惩罚：

$$\min_w \frac{1}{2m} \|Xw - y\|_2^2 + \lambda \|w\|_1$$

L1惩罚 $\|w\|_1 = \sum_j |w_j|$ 的非光滑特性导致解倾向于落在坐标轴上（$w_j = 0$），产生稀疏解。与之对比，L2（Ridge）惩罚 $\|w\|_2^2$ 产生的解所有分量都非零但数值缩小。

**几何解释**：L1的约束域是菱形（L1-ball），L2的是圆形。目标函数的等高线最先接触L1-ball的顶点（轴上的尖点）——这些点有零分量，产生稀疏性。L2-ball是光滑的，接触点一般在内部，分量都不为零。

**LASSO的偏差问题**：L1惩罚会缩小所有系数的绝对值（shrinkage），即使是非零系数也被缩小。弹性网（Elastic Net）结合L1和L2：

$$\min_w \frac{1}{2m} \|Xw - y\|_2^2 + \lambda\left(\alpha\|w\|_1 + \frac{1-\alpha}{2}\|w\|_2^2\right)$$

既得到稀疏性，又保持相关特征组的稳定性。

### 近端梯度下降（Proximal Gradient Descent, PGD）

解决形如 $\min_x f(x) + \lambda g(x)$ 的优化问题，其中 $f$ 光滑凸、$g$ 非光滑凸（如L1范数）。PGD的核心是近端算子（proximal operator）：

$$x^{(t+1)} = \text{prox}_{\eta \lambda g}\left(x^{(t)} - \eta \nabla f(x^{(t)})\right)$$

对于LASSO（$g(x) = \|x\|_1$），近端算子有闭式解——**软阈值函数（soft-thresholding）**：

$$\text{prox}_{\eta\lambda\|\cdot\|_1}(v)_j = \text{sign}(v_j) \cdot \max(0, |v_j| - \eta\lambda) = \begin{cases} v_j + \eta\lambda, & v_j < -\eta\lambda \\ 0, & |v_j| \leq \eta\lambda \\ v_j - \eta\lambda, & v_j > \eta\lambda \end{cases}$$

本质：将梯度下降更新后绝对值小于阈值的分量置零，绝对值大于阈值的分量向零收缩。

### 字典学习与K-SVD

给定样本 $X \in \mathbb{R}^{m \times n}$，字典学习寻找过完备字典 $B \in \mathbb{R}^{n \times p}$（$p > n$）和稀疏表示 $A \in \mathbb{R}^{p \times m}$，使得 $X \approx B A$ 且 $A$ 的每列稀疏：

$$\min_{B, A} \|X - BA\|_F^2, \quad \text{s.t. } \|a_i\|_0 \leq s, \forall i$$

$\ell_0$ 约束NP-hard，实际求解用交替优化：固定 $B$ 用OMP求 $A$，固定 $A$ 用K-SVD逐列更新 $B$。

**K-SVD算法核心**：更新第 $k$ 个字典原子 $b_k$ 时，仅考虑用到 $b_k$ 的那些样本子集 $I_k$，对其残差矩阵 $E_k = X_I - \sum_{j \neq k} b_j a_j^T$ 做SVD，取最大奇异值对应的左右奇异向量作为更新后的 $b_k$ 和 $a_k$。

### 压缩感知（Compressed Sensing）

突破奈奎斯特采样定理：若信号 $x \in \mathbb{R}^n$ 在某个基 $\Psi$ 下是 $s$-稀疏的（$\|s\|_0 \leq s$），则只需 $m = O(s \log \frac{n}{s}) \ll n$ 次线性测量 $y = \Phi x$（$\Phi \in \mathbb{R}^{m \times n}$），通过求解L1优化问题可精确恢复：

$$\min_x \|x\|_1, \quad \text{s.t. } y = \Phi \Psi x$$

**RIP条件（Restricted Isometry Property）**：测量矩阵 $\Phi$ 需满足对任意 $s$-稀疏向量 $x$，存在 $\delta_s \in (0, 1)$ 使得：

$$(1-\delta_s)\|x\|_2^2 \leq \|\Phi x\|_2^2 \leq (1+\delta_s)\|x\|_2^2$$

RIP保证了由L1优化恢复稀疏解的理论正确性。随机高斯/伯努利矩阵以高概率满足RIP。

### 矩阵补全与核范数

矩阵补全（如Netflix评分预测）：已知矩阵 $X$ 的部分元素，恢复全部元素。假设 $X$ 低秩，优化目标：

$$\min_X \text{rank}(X), \quad \text{s.t. } X_{ij} = M_{ij}, \forall (i,j) \in \Omega$$

秩最小化NP-hard，用**核范数**（nuclear norm）$\|X\|_* = \sum \sigma_i(X)$ 作为秩的凸松弛：

$$\min_X \|X\|_*, \quad \text{s.t. } X_{ij} = M_{ij}, \forall (i,j) \in \Omega$$

核范数是矩阵奇异值之和，类似L1范数是向量绝对值之和——二者分别是秩和sparsity的凸近似。

## 核心概念

| 概念 | 含义 |
|------|------|
| 冗余特征 | 可由其他特征线性/非线性推断的特征 |
| 无关特征 | 与目标变量独立/条件独立的特征 |
| 前向搜索 | 逐个添加特征，每次加最优者 |
| 后向搜索 | 从全集逐个删除特征 |
| L1惩罚 | 产生稀疏解的凸正则化 |
| 软阈值 | L1近端算子的闭式解 |
| 过完备字典 | 原子数大于特征维数的字典 |
| RIP | 压缩感知中测量矩阵需满足的条件 |

## 算法步骤 (Algorithm Steps)

### LVW（Las Vegas Wrapper）

1. 初始化最优特征子集 $F^* = \emptyset$，最优误差 $E^* = \infty$
2. **重复**直到满足停止条件（如预算耗尽）：
   - 随机采样特征子集 $F$
   - 在 $F$ 上交叉验证评估学习器，得误差 $E$
   - 若 $E < E^*$ 或（$E = E^*$ 且 $|F| < |F^*|$）：更新 $F^*, E^*$
3. 输出 $F^*$

LVW使用拉斯维加斯随机策略，每次随机生成子集，保证找到最优解但时间无上界。

### PGD求解LASSO

1. 初始化 $w^{(0)} = 0$，学习率 $\eta$
2. **重复**直到收敛：
   - 梯度步：$z^{(t)} = w^{(t)} - \eta \cdot \frac{1}{m}X^T(Xw^{(t)} - y)$
   - 近端步：$w_j^{(t+1)} = \text{sign}(z_j^{(t)}) \cdot \max(0, |z_j^{(t)}| - \eta\lambda)$
3. 输出 $w^*$

也可以使用加速近端梯度（FISTA）——引入Nesterov动量 $v$，收敛率从 $O(1/t)$ 提升至 $O(1/t^2)$。

## 直观理解 (Intuition)

**Relief像是在军棋游戏中试探对手的兵力**：一个个特征挨着试探——把这个特征遮住（看猜错最近邻居带来的影响）。如果遮住后最近邻的类别判断混乱了，这个特征重要；如果不影响，就不重要。

**LASSO像是橡皮筋包裹**：把每个特征权重想象成一根橡皮筋——模型训练时既要拟合（拉紧皮筋），又要防止权重太大（皮筋有张力）。L1惩罚像是一圈菱形的围栏——权重向量总是被推到围栏的四个尖角（坐标轴），产生大量零。

**K-SVD像是拼图**：每个字典原子是一块拼图，稀疏表示是每块拼图的分布系数。更新某块拼图时，只看那些用到它的样本，把残差拼成完整图形（SVD），就能找到更好的拼图形状。

**压缩感知的直觉**：传统采样定律说"想听到最高20000Hz的声音，至少要40000Hz采样"。但压缩感知说——如果你知道这段音频只由3个音符组成（稀疏），那么不需要采那么多点，用随机测量就够了。核心是预先知道"简洁性"。

## 常见误区 (Anti-patterns)

1. **混淆特征选择与降维**：特征选择选的是原始特征的子集（可解释性高），降维生成新特征（可解释性差）。在需要可解释性（如医疗诊断）时，优先特征选择。
2. **过滤式方法忽略特征交互**：两个特征单独看都与目标独立，但联合起来预测力强（XOR问题）。Relief只考虑单特征或简单交互。
3. **LASSO的"一山不容二虎"性质**：完美相关的两个特征，LASSO只保留其中一个。弹性网可缓解此问题。
4. **L1正则化参数的盲目调参**：$\lambda$ 控制稀疏度。太大：所有权重归零；太小：无稀疏效果。应用交叉验证选择，可用正则化路径高效探索。
5. **包裹式方法的过拟合陷阱**：用同一数据做特征选择和模型评估会得到过度乐观的错误估计。须用嵌套交叉验证（nested CV）。
6. **字典学习不是万能的降维工具**：过完备字典 $B \in \mathbb{R}^{n \times p}$（$p > n$）实际上增加了表示维数，稀疏性才是关键。

## 代码要点 (Code Essentials)

```python
# LASSO with L1 penalty
from sklearn.linear_model import Lasso, LassoCV
lasso_cv = LassoCV(alphas=np.logspace(-4, 1, 50), cv=5)
lasso_cv.fit(X_train, y_train)
# 最优alpha: lasso_cv.alpha_, 稀疏系数: np.sum(lasso_cv.coef_ != 0)

# Elastic Net
from sklearn.linear_model import ElasticNet
en = ElasticNet(alpha=0.1, l1_ratio=0.7)

# 基于树的特征选择
from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier()
rf.fit(X_train, y_train)
importances = rf.feature_importances_
# 或用 from sklearn.feature_selection import SelectFromModel

# 互信息过滤
from sklearn.feature_selection import mutual_info_classif
mi = mutual_info_classif(X_train, y_train)

# 前向序列选择
from sklearn.feature_selection import SequentialFeatureSelector
sfs = SequentialFeatureSelector(estimator, n_features_to_select=5, direction='forward')
```

- L1正则化参数的路径：`sklearn.linear_model.lasso_path` 可视化系数随 $\lambda$ 变化的轨迹
- `SelectKBest` + `f_classif`/`mutual_info_classif` 实现过滤式选择
- `RFE`（递归特征消除）是从全集逐步剔除的后向搜索方法
- 稀疏模型的系数直接指示被选中的特征（非零系数对应的特征）
- 稳定性选择（stability selection）：多次在不同子采样上做L1选择，统计每个特征被选中的频率

## 实例分析 (Worked Example)

在UCI的sonar数据集（60个特征，208个样本，二分类）上进行特征选择对比：

- **Relief-F**：筛选出15个特征，它们在物理上对应于高频声纳频率
- **LVW + kNN**：找到仅8个特征的子集，5折交叉验证错误率从全特征的12.5%降低到7.2%
- **LASSO ($\lambda$ via 5-fold CV)**：10个特征系数非零，测试错误率8.7%
- **弹性网 ($\alpha=0.5$)**：12个非零系数，比纯LASSO稍多（保留了相关特征对），性能相仿
- **RF特征重要性**：Top-10特征与LASSO选出的有6个重叠，验证了选择的稳定性

结论：LVW在测试错误率上最优但耗时最长（数百倍于过滤式），LASSO达到良好平衡。

## 关键结论 (Key Takeaways)

1. 特征选择是缓解维数灾难的关键，三个典型范式：过滤式（快）、包裹式（准）、嵌入式（平衡）
2. Relif/Relief-F基于近邻一致性的统计量，简单高效但忽略高阶交互
3. L1正则化是实现嵌入式特征选择的核心数学工具，其稀疏性源于L1-ball的菱形几何
4. 近端梯度下降（PGD）是求解带L1等非光滑惩罚的优化问题的通用算法
5. 字典学习和压缩感知将稀疏性思想推广到一般表示和信号恢复问题
6. 矩阵补全中的核范数是秩的凸松弛，正如L1范数是$\ell_0$的凸松弛
7. 特征选择必须与模型评估在独立的hold-out数据上进行，嵌套交叉验证是金标准

## 章节连接 (Connects To)

- **Ch3 线性回归**：LASSO是在线性回归损失上施加L1惩罚
- **Ch10 降维**：特征选择是坐标轴子集的"硬"降维，PCA是坐标变换的"软"降维
- **Ch12 计算学习理论**：VC维和Rademacher复杂度给出的泛化界解释了特征选择为何能降低过拟合
- **Ch15 优化方法**：近端梯度下降、ISTA/FISTA、ADMM等是求解稀疏优化的主力算法
