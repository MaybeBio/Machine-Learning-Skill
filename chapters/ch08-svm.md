# 第6章: 支持向量机 / Support Vector Machines

> **融合来源**: 西瓜书第6章(支持向量机) + 南瓜书第6章 + 李航第7章(支持向量机)

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 间隔与支持向量

SVM的核心思想：在特征空间中寻找一个**最大化分类间隔**的超平面。将超平面记为$\mathbf{w}^T\mathbf{x} + b = 0$，则样本点到超平面的距离为：

$$\gamma = \frac{|\mathbf{w}^T\mathbf{x} + b|}{\|\mathbf{w}\|}$$

若超平面正确分类，则有 $y_i(\mathbf{w}^T\mathbf{x}_i + b) \geq 1$（缩放后）。使等号成立的样本点称为**支持向量**(support vector)——它们是决定超平面的关键样本。

两个异类支持向量到超平面的距离之和为 **"间隔"(margin)**：$\gamma = \frac{2}{\|\mathbf{w}\|}$。最大化间隔等价于最小化$\|\mathbf{w}\|$。

### 1.2 硬间隔SVM (线性可分)

在训练样本线性可分的理想情况下，SVM的基本型：

$$\begin{aligned} \min_{\mathbf{w}, b} &\quad \frac{1}{2}\|\mathbf{w}\|^2 \\ \text{s.t.} &\quad y_i(\mathbf{w}^T\mathbf{x}_i + b) \geq 1, \quad i=1,2,\ldots,m \end{aligned}$$

这是**凸二次规划**(convex quadratic programming)问题，有全局最优解。

### 1.3 对偶问题与核技巧

通过拉格朗日乘子法得到对偶问题：

$$\max_{\boldsymbol{\alpha}} \sum_{i=1}^{m} \alpha_i - \frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{m} \alpha_i \alpha_j y_i y_j \mathbf{x}_i^T \mathbf{x}_j$$

$$\text{s.t.} \quad \sum_{i=1}^{m} \alpha_i y_i = 0, \quad \alpha_i \geq 0$$

对偶形式的优美之处在于：目标函数和决策函数都只涉样本间的内积$\mathbf{x}_i^T\mathbf{x}_j$。如果用核函数$\kappa(\mathbf{x}_i, \mathbf{x}_j)$替换内积，就自然地将SVM推广到非线性：

$$f(\mathbf{x}) = \text{sign}\left(\sum_{i=1}^{m} \alpha_i y_i \kappa(\mathbf{x}, \mathbf{x}_i) + b\right)$$

常见的核函数：
- 线性核：$\kappa(\mathbf{x}_i, \mathbf{x}_j) = \mathbf{x}_i^T \mathbf{x}_j$
- 多项式核：$\kappa(\mathbf{x}_i, \mathbf{x}_j) = (\mathbf{x}_i^T \mathbf{x}_j + c)^d$
- 高斯核(RBF)：$\kappa(\mathbf{x}_i, \mathbf{x}_j) = \exp(-\gamma \|\mathbf{x}_i - \mathbf{x}_j\|^2)$
- Laplace核：$\kappa(\mathbf{x}_i, \mathbf{x}_j) = \exp(-\gamma \|\mathbf{x}_i - \mathbf{x}_j\|_1)$
- Sigmoid核：$\kappa(\mathbf{x}_i, \mathbf{x}_j) = \tanh(\beta \mathbf{x}_i^T \mathbf{x}_j + \theta)$

核函数需满足**Mercer定理**：对任意平方可积函数$g$，有$\iint \kappa(\mathbf{x},\mathbf{z})g(\mathbf{x})g(\mathbf{z})d\mathbf{x}d\mathbf{z} \geq 0$（即核矩阵半正定）。

### 1.4 软间隔SVM

现实任务中数据通常非线性可分，允许少量样本不满足约束$y_i(\mathbf{w}^T\mathbf{x}_i+b) \geq 1$。引入**松弛变量**(slack variables) $\xi_i \geq 0$ 和惩罚参数 $C$：

$$\min_{\mathbf{w},b,\boldsymbol{\xi}} \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_{i=1}^{m} \xi_i$$

$$\text{s.t.} \quad y_i(\mathbf{w}^T\mathbf{x}_i+b) \geq 1-\xi_i, \quad \xi_i \geq 0$$

其中$C$控制对误分类的惩罚力度——$C$大→更注重正确分类(可能过拟合)，$C$小→更注重间隔最大化(可能欠拟合)。这也可视为**结构风险最小化**(SRM)：$\frac{1}{2}\|\mathbf{w}\|^2$是正则化项，$\sum\xi_i$是经验风险。

西瓜书用Hinge损失重新解释软间隔SVM：$\min_{\mathbf{w},b} \sum_{i=1}^{m} \max(0, 1 - y_i(\mathbf{w}^T\mathbf{x}_i+b)) + \frac{1}{2C}\|\mathbf{w}\|^2$

### 1.5 支持向量回归 (SVR)

SVR将间隔思想推广到回归。容忍$f(\mathbf{x})$与$y$之间有最多$\epsilon$的偏差：

$$\min_{\mathbf{w},b} \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_{i=1}^{m} \ell_\epsilon(f(\mathbf{x}_i) - y_i)$$

其中$\ell_\epsilon(z) = \max(0, |z| - \epsilon)$为$\epsilon$-不敏感损失。

---

## 二、公式推导 — 南瓜书补充

### 2.1 对偶问题完整推导

南瓜书给出了从原始问题到对偶问题的完整数学推导链。

构造拉格朗日函数：$L(\mathbf{w},b,\boldsymbol{\alpha}) = \frac{1}{2}\|\mathbf{w}\|^2 + \sum_{i=1}^{m} \alpha_i (1 - y_i(\mathbf{w}^T\mathbf{x}_i+b))$

(1) 令$\frac{\partial L}{\partial \mathbf{w}} = 0$得$\mathbf{w} = \sum_{i=1}^{m} \alpha_i y_i \mathbf{x}_i$
(2) 令$\frac{\partial L}{\partial b} = 0$得$\sum_{i=1}^{m} \alpha_i y_i = 0$

代入$L$即得对偶问题。KKT条件：$\alpha_i \geq 0, y_i f(\mathbf{x}_i)-1 \geq 0, \alpha_i(y_i f(\mathbf{x}_i)-1)=0$。

**关键结论**：训练完成后，大部分样本的$\alpha_i=0$，仅支持向量的$\alpha_i > 0$——这就是SVM解的**稀疏性**。

### 2.2 SMO算法的详细推导

南瓜书对SMO做了详细展开。SMO每次选取两个变量$\alpha_i$, $\alpha_j$，固定其他变量，求解该子问题。

由约束$\alpha_i y_i + \alpha_j y_j = -\sum_{k \neq i,j} \alpha_k y_k = \zeta$，可消去一个变量化为二次函数求极值。对$\alpha_j$的最优解进行**剪辑**以保持在$[0, C]$内（软间隔情况）。

南瓜书给出了SMO的解析更新公式（包含剪辑）：

$$\alpha_j^{\text{new,unc}} = \alpha_j^{\text{old}} + \frac{y_j(E_i - E_j)}{\eta}$$

其中$E_k = f(\mathbf{x}_k)-y_k$为误差，$\eta = \|\mathbf{x}_i - \mathbf{x}_j\|^2$（线性核）。剪辑后得$\alpha_j^{\text{new}}$，再由$\zeta$关系得$\alpha_i^{\text{new}}$。更新$b$的公式也做了推导。

### 2.3 正定核的判定

南瓜书补充了Mercer定理的等价表述：一个对称函数$\kappa(\mathbf{x},\mathbf{z})$是正定核，当且仅当对任意$\{\mathbf{x}_1,\ldots,\mathbf{x}_m\}$，对应的核矩阵$\mathbf{K}_{ij} = \kappa(\mathbf{x}_i,\mathbf{x}_j)$是半正定的。

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 李航Ch7的统计学习三要素

**模型**：决策函数$f(\mathbf{x}) = \text{sign}(\sum_i \alpha_i y_i K(\mathbf{x}_i,\mathbf{x}) + b^*)$。李航强调：SVM的解仅有部分$\alpha_i > 0$（支持向量），具有**稀疏性**。

**策略**：软间隔最大化 → 合页损失(Hinge Loss)最小化。李航给出合页损失函数形式：$\sum_i \max(0, 1-y_i(\mathbf{w}\cdot\mathbf{x}_i+b)) + \lambda\|\mathbf{w}\|^2$，并指出合页损失是0-1损失的上界（凸代理损失）。

**算法**：SMO(Sequential Minimal Optimization)是求解SVM对偶问题的高效算法。

### 3.2 李航的SMO算法步骤

1. 初始化$\boldsymbol{\alpha}=0$
2. 选取违背KKT条件最严重的$\alpha_i$作为第一个变量
3. 选择能使$\alpha_j$变化最大的$\alpha_j$（最大化$|E_i - E_j|$）作为第二个变量
4. 解析求解两变量子问题，剪辑$\alpha_j$，更新$\alpha_i$
5. 更新阈值$b$和误差缓存$E$
6. 检查KKT条件，不满足则返回步骤2

### 3.3 核技巧的理论基础（李航+PPT）

**表示定理**(Representer Theorem)：对于任意单调递增函数$\Omega$和非负损失函数，正则化经验风险最小化问题的最优解可表示为核函数的线性组合：$f^*(\mathbf{x}) = \sum_{i=1}^{m} \alpha_i K(\mathbf{x}_i, \mathbf{x})$。

这为SVM之外的所有**核方法**(kernel methods)——如核PCA、核LDA、核k-means——提供了理论基础。

PPT补充：核函数的选择是SVM最重要的超参数。RBF核最常用（通用近似器），需要调$\gamma$（宽度）。一个小技巧是：若特征数$\gg$样本数，用线性核即可（无需非线性映射）。

---

## 四、综合总结

### 算法对比

| 方法 | 核心思路 | 损失函数 | 核函数 | 输出 |
|------|---------|---------|--------|------|
| 硬间隔SVM | 最大间隔 | 约束优化 | 可选 | 类别 |
| 软间隔SVM | 最大软间隔 | Hinge loss + $\frac{1}{2}\|\mathbf{w}\|^2$ | 可选 | 类别 |
| SVR | $\epsilon$管道+松弛 | $\epsilon$-不敏感 + $\frac{1}{2}\|\mathbf{w}\|^2$ | 可选 | 实值 |
| 核方法 | 表示定理 | 任意凸损失+正则化 | 必须 | 任意 |

### 关键结论

1. **SVM = 最大间隔 + 核技巧 + 凸优化（对偶+SMO）**
2. **解的稀疏性**：只有支持向量($\alpha_i>0$)决定最终模型
3. **核技巧是关键创新**：将线性SVM无缝推广到非线性，且计算量不随特征空间的维度增长
4. **$C$是最重要的超参数**：大$C$→低偏差高方差；小$C$→高偏差低方差
5. **RBF核是默认首选**：只有一个超参数$\gamma$，通用近似任意连续函数
6. **SMO算法高效**：每次仅优化两个变量，有解析解
7. **核方法不限于SVM**：通过表示定理可应用于PCA、LDA、k-means等
8. **合页损失 = 0-1损失的凸代理**：软间隔SVM的统计学习解释

### 章节连接

- **前置依赖**：第5章(线性模型)—最大间隔是线性分类的极限追求；优化理论(凸优化、KKT条件)
- **后续延伸**：第8章(贝叶斯分类器)—对比生成式vs判别式；第17章(感知机)—最简单的线性分类器，SVM是其最优版本
