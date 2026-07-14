# 第15章: 聚类 / Clustering

> **融合来源**: 西瓜书第9章(聚类) + 南瓜书第9章 + 李航第14章(聚类方法) + 李航第22章(无监督学习方法总结) + 清华大学PPT

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 聚类任务

在"无监督学习"(unsupervised learning)中，训练样本的标记信息未知，目标是通过对无标记训练样本的学习揭示数据内在规律。聚类(clustering)是其中最核心的任务。

聚类试图将数据集中的样本划分为若干个互不相交的子集，每个子集称为一个**簇**(cluster)。形成的簇可能对应一些潜在概念（如"浅色瓜""深色瓜"），但这些概念对聚类算法事先未知，簇的语义需由使用者把握和命名。

形式化地说，假定样本集 $D = \{\mathbf{x}_1, \mathbf{x}_2, \ldots, \mathbf{x}_m\}$ 包含m个无标记样本，每个样本是n维特征向量，聚类将D划分为k个不相交的簇 $\{C_1, C_2, \ldots, C_k\}$，满足 $C_i \cap C_j = \emptyset (i \neq j)$ 且 $D = \bigcup_{l=1}^{k} C_l$。

聚类既能单独用于找寻数据内在分布结构，也可作为分类等后续任务的前驱过程。

### 1.2 性能度量

聚类性能度量亦称**有效性指标**(validity index)。西瓜书将其分为两类：

**外部指标**(external index)：将聚类结果与某个"参考模型"比较。定义样本对之间的关系：

- $a = |SS|$：在聚类和参考模型中都属同簇的样本对数
- $b = |SD|$：在聚类中同簇但参考模型中不同簇
- $c = |DS|$：在聚类中不同簇但参考模型中同簇
- $d = |DD|$：在两者中都不同簇

常用外部指标：
- **Jaccard系数(JC)**：$JC = \frac{a}{a+b+c}$
- **FM指数(FMI)**：$FMI = \sqrt{\frac{a}{a+b} \cdot \frac{a}{a+c}}$
- **Rand指数(RI)**：$RI = \frac{2(a+d)}{m(m-1)}$

**内部指标**(internal index)：直接考察聚类结果，不利用参考模型。定义簇内平均距离 $\text{avg}(C)$、簇内最远距离 $\text{diam}(C)$、簇间最近距离 $d_{\min}(C_i, C_j)$、簇中心距 $d_{cen}(C_i, C_j)$。

- **DB指数(DBI)**：$DBI = \frac{1}{k}\sum_{i=1}^{k}\max_{j \neq i}\left(\frac{\text{avg}(C_i) + \text{avg}(C_j)}{d_{cen}(\mu_i, \mu_j)}\right)$ — 值越小越好
- **Dunn指数(DI)**：$DI = \min_{1 \leq i \leq k}\left\{\min_{j \neq i}\left(\frac{d_{\min}(C_i, C_j)}{\max_{1 \leq l \leq k}\text{diam}(C_l)}\right)\right\}$ — 值越大越好

### 1.3 距离计算

距离度量需满足：非负性、同一性、对称性、直递性（三角不等式）。常用距离：

- **闵可夫斯基距离**(Minkowski)：$\text{dist}_{mk}(\mathbf{x}_i, \mathbf{x}_j) = \left(\sum_{u=1}^{n}|x_{iu} - x_{ju}|^p\right)^{1/p}$
  - $p=1$：曼哈顿距离
  - $p=2$：欧氏距离
  - $p \to \infty$：切比雪夫距离
- **VDM**(Value Difference Metric)：处理无序属性
- **MinkovDM**：结合闵可夫斯基距离与VDM处理混合属性
- **加权距离**：$\text{dist}_{wmk}(\mathbf{x}_i, \mathbf{x}_j) = \left(\sum_{u=1}^{n} w_u \cdot |x_{iu} - x_{ju}|^p\right)^{1/p}$

### 1.4 原型聚类

**k均值(k-means)**：最小化平方误差 $E = \sum_{i=1}^{k} \sum_{\mathbf{x} \in C_i} \|\mathbf{x} - \boldsymbol{\mu}_i\|_2^2$，其中 $\boldsymbol{\mu}_i = \frac{1}{|C_i|}\sum_{\mathbf{x} \in C_i} \mathbf{x}$。采用贪心策略迭代优化：随机初始化均值向量→分配样本到最近均值向量→更新均值向量→重复直至收敛。NP难问题，算法收敛到局部最优。

**学习向量量化(LVQ)**：利用有标记数据辅助聚类，学得一组原型向量刻画聚类结构。

**高斯混合聚类(GMM)**：采用概率模型，每个簇对应一个高斯分布：$p(\mathbf{x}) = \sum_{i=1}^{k} \alpha_i \cdot p(\mathbf{x}|\boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)$。用EM算法进行参数估计。

### 1.5 密度聚类与层次聚类

**密度聚类(DBSCAN)**：基于密度的聚类，通过邻域参数$(\epsilon, MinPts)$定义核心对象、密度直达、密度可达、密度相连等概念。一个簇是满足连接性和最大性的密度相连样本集合。

**层次聚类(hierarchical clustering)**：在不同层次对数据集进行划分，形成树形聚类结构。AGNES(自底向上)算法：将每个样本视为一个初始簇→合并距离最近的两个簇→重复直到达到预设簇数。簇间距离度量：最小距离(单链接)、最大距离(全链接)、平均距离(均链接)。

---

## 二、公式推导 — 南瓜书补充

### 2.1 k-means收敛性

南瓜书指出k-means更新均值向量的公式源于对平方误差求导：

$$\frac{\partial E}{\partial \boldsymbol{\mu}_i} = \frac{\partial}{\partial \boldsymbol{\mu}_i}\sum_{i=1}^{k}\sum_{\mathbf{x} \in C_i}\|\mathbf{x} - \boldsymbol{\mu}_i\|^2 = -2\sum_{\mathbf{x} \in C_i}(\mathbf{x} - \boldsymbol{\mu}_i) = 0$$

得 $\boldsymbol{\mu}_i = \frac{1}{|C_i|}\sum_{\mathbf{x} \in C_i}\mathbf{x}$。k-means保证每次迭代平方误差单调不增，收敛到局部最优。

### 2.2 GMM的EM推导

GMM的对数似然：$LL(\Theta) = \sum_{j=1}^{m} \ln\left(\sum_{i=1}^{k} \alpha_i \cdot p(\mathbf{x}_j|\boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)\right)$

引入隐变量 $z_{ji} \in \{0,1\}$ 表示 $\mathbf{x}_j$ 是否由第i个高斯成分生成。

**E步**：计算后验概率 $\gamma_{ji} = P(z_{ji}=1|\mathbf{x}_j) = \frac{\alpha_i \cdot p(\mathbf{x}_j|\boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)}{\sum_{l=1}^{k} \alpha_l \cdot p(\mathbf{x}_j|\boldsymbol{\mu}_l, \boldsymbol{\Sigma}_l)}$

**M步**：更新参数
- $\boldsymbol{\mu}_i = \frac{\sum_{j=1}^{m} \gamma_{ji} \mathbf{x}_j}{\sum_{j=1}^{m} \gamma_{ji}}$
- $\boldsymbol{\Sigma}_i = \frac{\sum_{j=1}^{m} \gamma_{ji} (\mathbf{x}_j - \boldsymbol{\mu}_i)(\mathbf{x}_j - \boldsymbol{\mu}_i)^T}{\sum_{j=1}^{m} \gamma_{ji}}$
- $\alpha_i = \frac{1}{m}\sum_{j=1}^{m} \gamma_{ji}$

### 2.3 DBSCAN的簇定义推导

南瓜书严格定义了DBSCAN中的概念链：核心对象→密度直达→密度可达（传递闭包）→密度相连（对称的密度可达）。基于此定义，簇是满足**连接性**（簇内任意两点密度相连）和**最大性**（密度可达的点必在同一簇中）的最大密度相连样本集合。

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 聚类方法的统计学习三要素

李航从三要素框架分析聚类：

| 方法 | 模型 | 策略 | 算法 |
|------|------|------|------|
| 层次聚类 | 聚类树 | 类内样本距离最小 | 启发式算法（自底向上合并或自顶向下分裂） |
| k均值聚类 | k个中心 | 样本与类中心距离最小（平方误差） | 迭代算法 |
| 高斯混合模型 | 高斯混合分布 | 似然函数最大 | EM算法 |

### 3.2 硬聚类与软聚类

李航区分了两种聚类方式：
- **硬聚类**(hard clustering)：每个样本确定性地属于一个簇（k-means、层次聚类）
- **软聚类**(soft clustering)：每个样本以一定概率属于各簇（GMM）

### 3.3 PPT补充：聚类算法选择指南

| 场景 | 推荐算法 |
|------|---------|
| 数据大致呈球形分布，已知k | k-means |
| 数据形状不规则，有噪声 | DBSCAN |
| 需要层次化结果 | 层次聚类（AGNES/DIANA） |
| 需要概率输出 | GMM |
| 大规模数据 | Mini-batch k-means |
| 混合属性 | k-prototypes |

### 3.4 k-means初始化的影响

PPT强调初始中心点选择对k-means结果有显著影响。改进方案：
- **k-means++**：选择初始中心点时使各中心点尽可能远离
- 多次随机初始化，选择平方误差最小的结果

### 3.5 李航对无监督学习聚类方法的总结（第22章）

李航第22章将聚类置于无监督学习全景中：聚类发掘数据的纵向结构，将相似样本聚到同类。层次聚类基于启发式，k均值基于迭代，GMM基于EM。聚类还与其他无监督学习任务密切相关——话题分析(LSA/PLSA/LDA)兼有聚类和降维的特点。

---

## 四、综合总结

### 算法对比

| 算法 | 原理 | 优点 | 缺点 | 时间复杂度 |
|------|------|------|------|-----------|
| k-means | 最小化平方误差 | 简单高效，可扩展 | 需指定k，对初始值敏感，仅适用球形簇 | $O(mkdt)$ |
| GMM | 概率模型+EM | 软分配，可处理不同形状 | 需指定k，对初始化敏感 | $O(mkdt)$ |
| DBSCAN | 密度连通性 | 可发现任意形状簇，抗噪声 | 参数敏感，密度不均时效果差 | $O(m^2)$或$O(m\log m)$ |
| 层次聚类 | 树形合并/分裂 | 不需指定k，层次结构 | 计算复杂度高，不可逆 | $O(m^2\log m)$ |
| LVQ | 原型向量+有标记 | 利用标记信息 | 需有标记数据 | $O(mqdt)$ |

### 关键结论

1. **聚类是无监督学习的核心任务**：在无标记数据中发现内在结构和分布规律
2. **没有万能聚类算法**：不同算法基于不同假设（球形/任意形状、硬/软聚类）
3. **性能度量是聚类的基础**：外部指标(JC/FMI/RI)和内部指标(DBI/DI)各有适用场景
4. **距离度量至关重要**：闵可夫斯基距离处理有序属性，VDM处理无序属性，MinkovDM处理混合属性
5. **k-means是默认选择**：简单高效，为NP难问题提供贪心近似解
6. **EM算法是GMM的核心**：通过迭代的E步（计算后验）和M步（最大化期望）实现参数估计
7. **密度聚类适应性强**：DBSCAN不需要指定簇数，可发现任意形状簇

### 章节连接

- **前置依赖**：ch01(绪论)—无监督学习概念；ch11(EM算法)—GMM参数估计的基础；ch02(评估)—性能度量方法
- **后续延伸**：ch17(概率图模型)—LDA等话题模型兼有聚类思想；ch18(无监督学习总结)—与其他无监督学习方法的统一视角

