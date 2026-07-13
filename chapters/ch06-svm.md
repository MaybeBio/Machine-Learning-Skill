# Chapter 6: 支持向量机 / Support Vector Machines

## 核心概念 (Core Idea)

支持向量机（SVM）的核心思想是在特征空间中寻找一个最大化分类间隔的超平面。通过对偶理论将原始问题转化为凸二次规划，利用核技巧隐式地将样本映射到高维空间以处理非线性可分问题。软间隔引入松弛变量允许少量误分类，而支持向量回归（SVR）将间隔思想推广到回归任务。SVM的解具有稀疏性——最终模型仅由少量"支持向量"决定。

---

## 融合框架 (Integrated Frameworks)

- **硬间隔与对偶**：定义几何间隔，构建最大化间隔的凸优化问题，通过拉格朗日对偶转化 (西瓜书/南瓜书)
- **KKT条件与支持向量**：推导KKT条件，明确支持向量的定义和稀疏性 (南瓜书/Li Hang P1)
- **核技巧**：核函数的定义、Mercer定理、常见核函数及其组合性质 (西瓜书)
- **软间隔与替代损失**：引入松弛变量与hinge损失，建立结构风险与经验风险的统一框架 (西瓜书)
- **SMO算法**：序列最小优化——每次只优化两个拉格朗日乘子的高效求解算法 (西瓜书/Li Hang P1)
- **SVR与核方法**：将间隔思想推广至回归，通过表示定理统一核化方法 (西瓜书)

---

## 理论推导 (Theoretical Derivation)

### 1. 硬间隔SVM

给定训练集 $D = \{(\mathbf{x}_1, y_1), \ldots, (\mathbf{x}_m, y_m)\}, y_i \in \{-1, +1\}$，划分超平面为 $\mathbf{w}^\top\mathbf{x} + b = 0$。任意点 $\mathbf{x}$ 到超平面的距离为：

$$r = \frac{|\mathbf{w}^\top\mathbf{x} + b|}{\|\mathbf{w}\|}$$

若超平面能正确分类，则有约束 $y_i(\mathbf{w}^\top\mathbf{x}_i + b) \geq 1$。两个异类支持向量到超平面的距离之和（间隔）为 $\gamma = \frac{2}{\|\mathbf{w}\|}$。

最大化间隔等价于最小化 $\|\mathbf{w}\|^2$，SVM基本型为：

$$\min_{\mathbf{w}, b} \frac{1}{2}\|\mathbf{w}\|^2 \quad \text{s.t.} \; y_i(\mathbf{w}^\top\mathbf{x}_i + b) \geq 1, \; i=1,\ldots,m$$

### 2. 对偶问题与SMO

引入拉格朗日乘子 $\alpha_i \geq 0$，构造拉格朗日函数：

$$L(\mathbf{w}, b, \boldsymbol{\alpha}) = \frac{1}{2}\|\mathbf{w}\|^2 + \sum_{i=1}^{m} \alpha_i(1 - y_i(\mathbf{w}^\top\mathbf{x}_i + b))$$

令 $L$ 对 $\mathbf{w}$ 和 $b$ 偏导为零得 $\mathbf{w} = \sum_i \alpha_i y_i \mathbf{x}_i$ 和 $\sum_i \alpha_i y_i = 0$，代入得对偶问题：

$$\max_{\boldsymbol{\alpha}} \sum_{i=1}^{m} \alpha_i - \frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{m} \alpha_i \alpha_j y_i y_j \mathbf{x}_i^\top \mathbf{x}_j$$

$$\text{s.t.} \; \sum_{i=1}^{m} \alpha_i y_i = 0, \; \alpha_i \geq 0$$

KKT条件要求 $\alpha_i(y_i f(\mathbf{x}_i) - 1) = 0$，这意味着只有支持向量（$\alpha_i > 0$）对模型有贡献。

**SMO算法**：每次固定除 $\alpha_i, \alpha_j$ 外的所有参数，利用约束 $\alpha_i y_i + \alpha_j y_j = c$ 消去 $\alpha_j$，化为单变量二次规划问题，有闭式解。迭代选取违反KKT条件最大的两个变量更新，直至收敛。

### 3. 核函数

对于非线性可分问题，将样本映射到高维特征空间：$\mathbf{x} \to \phi(\mathbf{x})$。核函数定义为：

$$\kappa(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i)^\top\phi(\mathbf{x}_j)$$

**Mercer定理**：对称函数 $\kappa$ 是核函数当且仅当对任意数据，核矩阵 $\mathbf{K}$ 总是半正定的。

常用核函数：

| 名称 | 表达式 | 参数 |
|------|--------|------|
| 线性核 | $\kappa(\mathbf{x}_i, \mathbf{x}_j) = \mathbf{x}_i^\top\mathbf{x}_j$ | -- |
| 多项式核 | $\kappa(\mathbf{x}_i, \mathbf{x}_j) = (\mathbf{x}_i^\top\mathbf{x}_j)^d$ | $d \geq 1$ |
| 高斯核(RBF) | $\kappa(\mathbf{x}_i, \mathbf{x}_j) = \exp(-\frac{\|\mathbf{x}_i - \mathbf{x}_j\|^2}{2\sigma^2})$ | $\sigma > 0$ |
| 拉普拉斯核 | $\kappa(\mathbf{x}_i, \mathbf{x}_j) = \exp(-\frac{\|\mathbf{x}_i - \mathbf{x}_j\|}{\sigma})$ | $\sigma > 0$ |
| Sigmoid核 | $\kappa(\mathbf{x}_i, \mathbf{x}_j) = \tanh(\beta\mathbf{x}_i^\top\mathbf{x}_j + \theta)$ | $\beta>0, \theta<0$ |

学习后的模型为支持向量展式：$f(\mathbf{x}) = \sum_{i \in SV} \alpha_i y_i \kappa(\mathbf{x}, \mathbf{x}_i) + b$

### 4. 软间隔与正则化

引入松弛变量 $\xi_i \geq 0$ 和惩罚参数 $C > 0$：

$$\min_{\mathbf{w}, b, \boldsymbol{\xi}} \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_{i=1}^{m} \xi_i$$

$$\text{s.t.} \; y_i(\mathbf{w}^\top\mathbf{x}_i + b) \geq 1 - \xi_i, \; \xi_i \geq 0$$

这与使用hinge损失 $\ell_{\text{hinge}}(z) = \max(0, 1 - z)$ 等价。对偶形式仅在约束上将 $\alpha_i \geq 0$ 改为 $0 \leq \alpha_i \leq C$。

KKT条件揭示三种情况：$\alpha_i = 0$（非支持向量），$0 < \alpha_i < C$（间隔边界上的支持向量），$\alpha_i = C$（间隔内部或误分类的支持向量）。

统一正则化框架：$\min_f \Omega(f) + C\sum_i \ell(f(\mathbf{x}_i), y_i)$，其中 $\Omega(f)$ 为结构风险（正则化项），第二项为经验风险。

### 5. 支持向量回归 (SVR)

SVR容忍 $f(\mathbf{x})$ 与 $y$ 之间有最多 $\epsilon$ 的偏差。引入 $\epsilon$-不敏感损失和两组松弛变量 $\xi_i, \hat{\xi}_i$：

$$\min_{\mathbf{w}, b} \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_{i=1}^{m}(\xi_i + \hat{\xi}_i)$$

$$\text{s.t.} \; f(\mathbf{x}_i) - y_i \leq \epsilon + \xi_i, \; y_i - f(\mathbf{x}_i) \leq \epsilon + \hat{\xi}_i$$

对偶问题中的支持向量变为 $(\hat{\alpha}_i - \alpha_i) \neq 0$ 的样本，落在 $\epsilon$-间隔带之外。SVR的解形如 $f(\mathbf{x}) = \sum_i (\hat{\alpha}_i - \alpha_i) \kappa(\mathbf{x}, \mathbf{x}_i) + b$。

### 6. 表示定理

对于优化问题 $\min_{h \in \mathbb{H}} \Omega(\|h\|_{\mathbb{H}}) + \ell(h(\mathbf{x}_1), \ldots, h(\mathbf{x}_m))$，其中 $\mathbb{H}$ 为核函数对应的RKHS，无论 $\ell$ 是否凸函数，最优解均可表示为 $h^*(\mathbf{x}) = \sum_{i=1}^{m} \alpha_i \kappa(\mathbf{x}, \mathbf{x}_i)$。这是核方法的理论基础。

---

## 核心概念 (Key Concepts)

- **支持向量**：距离分类超平面最近的训练样本，满足 $y_i(\mathbf{w}^\top\mathbf{x}_i + b) = 1$ 或 $\alpha_i > 0$
- **间隔 (Margin)**：两个异类支持向量到超平面的距离之和 $\gamma = 2/\|\mathbf{w}\|$。大间隔意味着更强的泛化能力
- **对偶问题**：将原始问题转化为以拉格朗日乘子为变量的优化问题，便于引入核函数和利用SMO求解
- **KKT条件**：最优解的必要条件，$\alpha_i(y_i f(\mathbf{x}_i) - 1 + \xi_i) = 0$ 区分支持向量与非支持向量
- **核技巧**：通过核函数隐式计算高维特征空间中的内积，避免显式映射和维度灾难
- **Hinge损失**：$\ell(z) = \max(0, 1-z)$，是0/1损失的凸上界，具有稀疏性——零区域不产生梯度
- **结构风险最小化**：$\min \Omega(f) + C \cdot R_{\text{emp}}(f)$，正则化项 $\Omega(f)$ 控制模型复杂度
- **参数 $C$**：正则化常数，$C$ 越大对误分类惩罚越重，$C \to \infty$ 退化为硬间隔
- **参数 $\sigma$ (RBF)**：高斯核带宽，$\sigma$ 越小模型越复杂，越容易过拟合

---

## 算法步骤 (Algorithm Steps)

### SMO算法 (Li Hang P1)

1. 初始化所有 $\alpha_i = 0$ 及偏置 $b = 0$
2. 选取违反KKT条件的两个变量 $\alpha_i, \alpha_j$
   - 外层循环：选取违反KKT条件最严重的样本点（优先遍历支持向量点）
   - 内层循环：选取使 $|E_i - E_j|$ 最大的变量（$E_i$ 为预测误差）
3. 在约束 $\alpha_i y_i + \alpha_j y_j = c$ 和 $0 \leq \alpha_i, \alpha_j \leq C$ 下求解子问题
   - 计算剪辑后的最优解 $a_j^{\text{new}}$，进而得 $a_i^{\text{new}}$
4. 更新偏置 $b$（若 $0 < \alpha_i < C$，则用对应的样本求精确 $b$）
5. 检查是否全部 $\alpha_i$ 满足KKT条件：若满足则停止，否则返回步骤2
6. 输出：$\mathbf{w} = \sum_i \alpha_i y_i \mathbf{x}_i$，决策函数 $f(\mathbf{x}) = \sum_i \alpha_i y_i \kappa(\mathbf{x}, \mathbf{x}_i) + b$

### SVR训练

1. 选择核函数和参数 $\epsilon, C$
2. 构建并求解对偶问题
3. 从 $0 < \alpha_i < C$ 的样本计算偏置 $b$
4. 输出回归函数 $f(\mathbf{x}) = \sum_i (\hat{\alpha}_i - \alpha_i)\kappa(\mathbf{x}, \mathbf{x}_i) + b$

---

## 直观理解 (Intuition)

**最大间隔如"留足安全距离"**：在一条窄路上开车，最安全的策略是走正中间——两边的护栏（支持向量）决定了你的路径，中间的宽裕空间就是间隔。SVM不是简单地找到一条能分开两类点的线，而是找到"最宽的那条安全带"。

**核技巧如"向上帝借一个维度"**：平面上无法用一条直线分开的XOR问题，放在三维空间中就像捏起一副扑克牌的两半——高维给了你新的自由度。核技巧的神奇之处在于，你不需要真的去高维世界走一趟，而是通过核函数在低维就完成了高维的内积计算。

**SMO如"两人一组做优化"**：想象你要调整所有人的工作量使得一个约束成立。如果一次只调整一个人，约束会被破坏（因为 $\sum \alpha_i y_i = 0$）。SMO的智慧是一次调整两个人——一个多了，另一个就少些，维持总平衡。

**SVR如"铺一条管道"**：普通的回归是要让线穿过所有数据点，而SVR更像是铺一条宽度为 $2\epsilon$ 的管道——数据点落在管道里就万事大吉，落在外面才计入损失。这条管道自然地避开了噪声点的干扰。

**$C$ 和 $\sigma$ 如"松紧调节"**：$C$ 决定了你对犯错的容忍度（软硬程度），$\sigma$ 决定了单一样本的影响范围（影响力半径）。$C$ 大 $\sigma$ 小等于"一条道走到黑"，极易过拟合。

---

## 常见误区 (Anti-patterns)

- **误区："SVM只适用于二分类"**：基础SVM确实是为二分类设计的，但可通过"一对多"(one-vs-rest)或"一对一"(one-vs-one)策略扩展至多分类，且实际效果往往不错
- **误区："高斯核总是最好的"**：高斯核虽是最常用的非线性核，但对文本数据等稀疏高维数据，线性核往往更优且速度更快。经验做法是先从线性核开始，若欠拟合再尝试高斯核
- **误区："SVM的训练时间正比于样本数"**：对线性核，通过割平面法(Pegasos)或坐标下降(LIBLINEAR)可实现线性时间复杂度；对非线性核，复杂度理论上不低于$O(m^2)$，但可用Nyström近似或随机傅里叶特征加速
- **误区："调大 $C$ 一定提高训练精度"**：$C$ 增大确实能减少训练误差，但会削弱间隔（允许更复杂的模型），在数据有噪声时反而导致测试性能下降。$C$ 的调节需要在训练误差与模型复杂度之间折中
- **误区："SVM的概率输出就是支持度"**：标准SVM的输出 $f(\mathbf{x})$ 不是概率。如需概率，必须通过Platt缩放（Sigmoid映射）或等分回归进行校准

---

## 代码要点 (Code Essentials)

- **数据标准化**：SVM对特征尺度敏感，训练前必须将特征标准化（零均值、单位方差）或归一化到 $[0,1]$
- **核函数选择**：优先尝试线性核(LIBLINEAR效率高)；若线性不可分则选RBF核。通过交叉验证选 $C$ 和 $\gamma$（RBF参数 $\gamma = 1/(2\sigma^2)$）
- **参数搜索**：$C$ 和 $\gamma$ 应在对数尺度网格上搜索，如 $C \in \{10^{-3}, 10^{-2}, \ldots, 10^3\}$，$\gamma \in \{10^{-3}, 10^{-2}, \ldots, 10^3\}$
- **多分类策略**：sklearn的SVC默认使用"一对一"(ovo)，共训练 $N(N-1)/2$ 个二分类器，存储开销较大但准确率通常较高
- **类别不平衡**：设置 `class_weight='balanced'` 对不同类别样本施加不同的 $C$ 惩罚权重
- **核缓存**：对于大规模非线性SVM，开启核矩阵缓存(`cache_size`)可减少重复计算
- **SVR参数**：$\epsilon$ 控制管道宽度，通常需要根据问题调整；默认值如 $\epsilon = 0.1$（数据已标准化时）

---

## 实例分析 (Worked Example)

**二维空间中3个样本的硬间隔SVM求解** (Li Hang P1):

给定训练集 $\mathbf{x}_1 = (3,3)^\top, y_1 = +1$；$\mathbf{x}_2 = (4,3)^\top, y_2 = +1$；$\mathbf{x}_3 = (1,1)^\top, y_3 = -1$。

对偶问题：

$$\min_{\boldsymbol{\alpha}} \frac{1}{2}\sum_i\sum_j \alpha_i\alpha_j y_i y_j (\mathbf{x}_i^\top\mathbf{x}_j) - \sum_i \alpha_i$$

代入内积计算（如 $\mathbf{x}_1^\top\mathbf{x}_1 = 18$, $\mathbf{x}_1^\top\mathbf{x}_2 = 21$, $\mathbf{x}_1^\top\mathbf{x}_3 = 6$ 等），约束 $\alpha_1 + \alpha_2 - \alpha_3 = 0$（注意 $y_3=-1$）。

解得 $\alpha_1 = \alpha_2 = 1/4, \alpha_3 = 1/2$，即 $\mathbf{x}_1, \mathbf{x}_2, \mathbf{x}_3$ 均为支持向量。

$$\mathbf{w} = \frac{1}{4} \cdot 1 \cdot (3,3)^\top + \frac{1}{4} \cdot 1 \cdot (4,3)^\top + \frac{1}{2} \cdot (-1) \cdot (1,1)^\top = (1, 1)^\top$$

由支持向量 $\mathbf{x}_1$ 求 $b$：$b = y_1 - \mathbf{w}^\top\mathbf{x}_1 = 1 - (3+3) = -5$。

最终分类超平面：$x^{(1)} + x^{(2)} - 5 = 0$，间隔 $\gamma = 2/\sqrt{2} = \sqrt{2}$。

---

## 关键结论 (Key Takeaways)

1. SVM = 最大化间隔 + 对偶求解 + 核技巧。三者的结合使其成为最优雅的分类算法之一
2. KKT条件的推论 $\alpha_i(y_i f(\mathbf{x}_i) - 1 + \xi_i) = 0$ 带来了解的稀疏性——只有少数支持向量决定模型
3. 核函数选择是SVM实践中最关键的决策。高斯核是最常用的默认选择，但并非万能
4. 从结构风险最小化的角度看，SVM通过 $\ell_2$ 正则化（最大化间隔）来控制模型复杂度，这是其泛化能力优异的核心原因
5. SMO算法是SVM能在大规模数据上实际应用的关键——它将大规模二次规划转化为一系列微型解析问题
6. 表示定理揭示了核方法的本质：大量学习问题的最优解都可用训练样本的核函数线性表示

---

## 章节连接 (Connects To)

- **前置**: 线性模型 (Ch3) — 理解线性分类器；感知机/神经网络 (Ch5) — 感知机与线性SVM的关系
- **后续**: 贝叶斯分类器 (Ch7) — 生成式模型vs判别式模型的对比；核方法进阶 — KLDA、核PCA；深度学习 — NN与SVM的关系（RBF网络等价于高斯核SVM）

