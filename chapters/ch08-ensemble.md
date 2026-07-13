# Chapter 8: 集成学习 / Ensemble Learning

## 核心概念 (Core Idea)

集成学习通过构建并结合多个学习器来完成学习任务，其核心思想是"三个臭皮匠顶个诸葛亮"——多个弱学习器通过合理组合，常可超越单个强学习器的性能。关键在于个体学习器必须"好而不同"（既要有一定准确性，又要有差异性）。Boosting（串行、降低偏差）、Bagging/Random Forest（并行、降低方差）、Stacking（学习法结合）构成了三大主要范式。

---

## 融合框架 (Integrated Frameworks)

- **集成学习基础**：voting原理、误差-分歧分解、偏差-方差-协方差分析 (西瓜书)
- **AdaBoost**：加性模型 + 指数损失的前向分步算法推导，分类器权重与样本分布更新 (西瓜书/Li Hang P1)
- **Bagging与随机森林**：Bootstrap采样 + 投票/平均结合；随机森林的属性扰动机制；OOB估计 (西瓜书)
- **结合策略**：平均法、投票法（硬/软投票）、学习法(Stacking) (西瓜书)
- **多样性度量**：分歧项、列联表、不合度量、相关系数、Q统计量、$\kappa$统计量 (西瓜书)
- **理论分析**：Boosting的误差界（训练误差指数衰减）、Bagging的方差缩减 (Li Hang P1/西瓜书)
- **GBDT与XGBoost**：梯度提升——将Boosting推广到任意可微损失函数 (Li Hang P1)

---

## 理论推导 (Theoretical Derivation)

### 1. 集成的基本理论

**投票法的错误率**：假设 $T$ 个基分类器错误率均为 $\epsilon$ 且相互独立，则集成投票错误率：

$$P(H(\mathbf{x}) \neq f(\mathbf{x})) \leq \exp\left(-\frac{1}{2}T(1 - 2\epsilon)^2\right)$$

随着 $T$ 增大，错误率指数级下降——但独立假设在实践中不成立。

**误差-分歧分解** (Krogh & Vedelsby, 1995)：对回归任务使用加权平均法，有：

$$E = \overline{E} - \overline{A}$$

其中 $E$ 为集成泛化误差，$\overline{E}$ 为个体学习器泛化误差的加权均值，$\overline{A}$ 为个体学习器的加权分歧（多样性）。这明确揭示：个体准确性越高 + 多样性越大 = 集成越好。

### 2. Boosting与AdaBoost (西瓜书/Li Hang P1)

**加性模型**：$H(\mathbf{x}) = \sum_{t=1}^{T} \alpha_t h_t(\mathbf{x})$

**指数损失函数**：$\ell_{\exp}(H \mid \mathcal{D}) = \mathbb{E}_{\mathbf{x} \sim \mathcal{D}}[e^{-f(\mathbf{x})H(\mathbf{x})}]$

指数损失最小化等价于达到贝叶斯最优错误率，因为：

$$H(\mathbf{x}) = \frac{1}{2}\ln\frac{P(f(\mathbf{x})=1 \mid \mathbf{x})}{P(f(\mathbf{x})=-1 \mid \mathbf{x})} \Rightarrow \text{sign}(H(\mathbf{x})) = \arg\max_{y} P(f(\mathbf{x})=y \mid \mathbf{x})$$

**AdaBoost的前向分步推导**：

在第 $t$ 步，给定 $H_{t-1}$，求最优的 $\alpha_t$ 和 $h_t$。

- 分类器权重 $\alpha_t$ 的最优解（令指数损失对 $\alpha_t$ 的导数为0）：

$$\alpha_t = \frac{1}{2}\ln\left(\frac{1 - \epsilon_t}{\epsilon_t}\right)$$

其中 $\epsilon_t = P_{\mathbf{x}\sim\mathcal{D}_t}(h_t(\mathbf{x}) \neq f(\mathbf{x}))$ 为当前分布下的加权误差。

- 样本分布更新（推导得 $\mathcal{D}_{t+1}$ 与 $\mathcal{D}_t$ 的关系）：

$$\mathcal{D}_{t+1}(\mathbf{x}) = \frac{\mathcal{D}_t(\mathbf{x}) \exp(-\alpha_t f(\mathbf{x}) h_t(\mathbf{x}))}{Z_t}$$

等价于对误分类样本放大权重 $\exp(\alpha_t)$，对正确分类样本缩小权重 $\exp(-\alpha_t)$。

**AdaBoost训练误差界** (Li Hang P1)：

$$\frac{1}{m}\sum_{i=1}^{m} \mathbb{I}(H(\mathbf{x}_i) \neq y_i) \leq \prod_{t=1}^{T} Z_t = \prod_{t=1}^{T} 2\sqrt{\epsilon_t(1-\epsilon_t)} \leq \exp\left(-2\sum_{t=1}^{T} \gamma_t^2\right)$$

其中 $\gamma_t = 0.5 - \epsilon_t$。这说明只要有 $\epsilon_t < 0.5$（优于随机猜测），训练误差将随 $T$ 增大而指数下降。

### 3. 梯度提升 (Gradient Boosting)

将Boosting推广到任意可微损失函数 $L$。在第 $t$ 步：

- 计算负梯度（残差的近似）：$r_{ti} = -\left[\frac{\partial L(y_i, f(\mathbf{x}_i))}{\partial f(\mathbf{x}_i)}\right]_{f=H_{t-1}}$
- 用基学习器 $h_t$ 拟合残差 $\{(\mathbf{x}_i, r_{ti})\}_{i=1}^{m}$
- 线搜索步长：$\rho_t = \arg\min_\rho \sum_i L(y_i, H_{t-1}(\mathbf{x}_i) + \rho h_t(\mathbf{x}_i))$
- 更新：$H_t = H_{t-1} + \rho_t h_t$

GBDT是梯度提升 + 决策树（通常CART回归树）的组合。XGBoost在此基础上增加了二阶泰勒展开和正则化项，大幅提升了性能和效率。

### 4. Bagging与随机森林

**Bootstrap采样**：$m$ 个样本中每轮随机有放回抽取 $m$ 个样本。每个样本被抽中的概率为 $1 - (1 - 1/m)^m \to 1 - 1/e \approx 0.632$。

**OOB估计**：每个基学习器使用约63.2%样本训练，剩余36.8%作为包外验证集。包外误差可作为泛化误差的无偏估计：

$$\epsilon^{oob} = \frac{1}{|D|}\sum_{(\mathbf{x},y)\in D} \mathbb{I}(H^{oob}(\mathbf{x}) \neq y)$$

**随机森林的属性扰动**：对每个分裂结点，随机选择 $k$ 个属性（通常 $k = \log_2 d$），仅从该子集中选择最优划分属性。双重扰动（样本扰动 + 属性扰动）增大了个体差异，提升泛化性能。

### 5. 多样性度量

给定二分类问题下 $h_i$ 和 $h_j$ 的预测列联表（a=均为正, b=$h_i$正$h_j$负, c=$h_i$负$h_j$正, d=均为负）：

- **不合度量**：$\text{dis}_{ij} = \frac{b + c}{a + b + c + d}$，越大越多样
- **相关系数**：$\rho_{ij} = \frac{ad - bc}{\sqrt{(a+b)(a+c)(c+d)(b+d)}}$，越接近0越多样
- **Q统计量**：$Q_{ij} = \frac{ad - bc}{ad + bc}$，与相关系数符号一致
- **$\kappa$统计量**：$\kappa = \frac{p_1 - p_2}{1 - p_2}$，其中 $p_1$ 为两分类器一致率，$p_2$ 为随机一致率，$\kappa$ 越小多样性越高

---

## 核心概念 (Key Concepts)

- **好而不同**：集成的两个必要条件。准确性保证基学习器不差于随机猜测，多样性保证错误不相关
- **Boosting vs Bagging**：Boosting串行训练、降低偏差、从弱学习器构建强学习器；Bagging并行训练、降低方差、从强但不稳定的学习器（如未剪枝决策树）获益最大
- **加性模型**：基学习器的线性组合 $H = \sum \alpha_t h_t$，AdaBoost通过前向分步加性建模最小化指数损失
- **指数损失**：$\ell_{\exp}(z) = e^{-z}$，与0/1损失具有一致性，是AdaBoost的核心设计选择
- **样本重赋权 vs 重采样**：Boosting的两种实现方式。重采样的优势在于可提供"重启动"机会
- **包外估计(OOB)**：Bagging无需单独划分验证集即可评估泛化性能，且可用于特征重要性评估和剪枝
- **软投票 vs 硬投票**：软投票使用类概率（后验概率估计）投票，通常优于仅用标签的硬投票
- **Stacking**：用元学习器学习如何组合初级学习器的输出，本质上是对结合策略本身进行学习

---

## 算法步骤 (Algorithm Steps)

### AdaBoost (二分类，西瓜书/Li Hang P1)

1. 初始化样本权重分布 $\mathcal{D}_1(\mathbf{x}_i) = 1/m$
2. **For** $t = 1, 2, \ldots, T$:
   - (a) 基于分布 $\mathcal{D}_t$ 训练基学习器 $h_t = \mathfrak{L}(D, \mathcal{D}_t)$
   - (b) 计算 $h_t$ 的加权误差 $\epsilon_t = P_{\mathbf{x}\sim\mathcal{D}_t}(h_t(\mathbf{x}) \neq f(\mathbf{x}))$
   - (c) **If** $\epsilon_t > 0.5$ **then** break
   - (d) 确定 $h_t$ 权重 $\alpha_t = \frac{1}{2}\ln(\frac{1-\epsilon_t}{\epsilon_t})$
   - (e) 更新分布：$\mathcal{D}_{t+1}(\mathbf{x}_i) = \frac{\mathcal{D}_t(\mathbf{x}_i)}{Z_t} \exp(-\alpha_t y_i h_t(\mathbf{x}_i))$
     - 正确：权重 $\times \exp(-\alpha_t)$；错误：权重 $\times \exp(\alpha_t)$
3. 输出：$H(\mathbf{x}) = \text{sign}\left(\sum_{t=1}^{T} \alpha_t h_t(\mathbf{x})\right)$

### Bagging (西瓜书)

1. **For** $t = 1, 2, \ldots, T$:
   - (a) Bootstrap采样生成训练子集 $D_t$（$m$ 次有放回抽取）
   - (b) 在 $D_t$ 上训练基学习器 $h_t$
2. 输出：分类用多数投票 $H(\mathbf{x}) = \arg\max_y \sum_t \mathbb{I}(h_t(\mathbf{x}) = y)$；回归用简单平均 $H(\mathbf{x}) = \frac{1}{T}\sum_t h_t(\mathbf{x})$

### 随机森林 (西瓜书)

1. **For** $t = 1, 2, \ldots, T$:
   - (a) Bootstrap采样生成 $D_t$
   - (b) 在 $D_t$ 上训练决策树，每个结点选择划分时：
     - 从 $d$ 个属性中随机选取 $k$ 个（推荐 $k = \log_2 d$）
     - 仅从这 $k$ 个属性中选择最优划分
     - 结点不剪枝（充分生长）
2. 输出：投票法（分类）/ 平均法（回归）

### Stacking (西瓜书)

1. **For** $t = 1, 2, \ldots, T$: 在完整训练集上训练初级学习器 $h_t$
2. 生成次级训练集 $D' = \emptyset$
3. **For** $i = 1, \ldots, m$: $\mathbf{z}_i = (h_1(\mathbf{x}_i), \ldots, h_T(\mathbf{x}_i))^\top$，$D' = D' \cup \{(\mathbf{z}_i, y_i)\}$
4. 在 $D'$ 上训练次级学习器 $h'$
5. 输出：$H(\mathbf{x}) = h'(h_1(\mathbf{x}), \ldots, h_T(\mathbf{x}))$

注：实践中一般用交叉验证生成次级训练集以避免过拟合。

---

## 直观理解 (Intuition)

**Boosting如"师徒传承"**：新徒弟($h_{t+1}$)专注于师傅($H_t$)之前犯错的问题——师傅做不到的，徒弟来补救。每一代都在弥补前人的短板，最终的"师门"($H_T$)博采众长。

**Bagging如"民主投票"**：每个人($h_t$)从不同的角度（不同的Bootstrap子集）看问题，但都有自己的见解。最终按多数(投票)或平均(平均法)形成集体决策，比任何个人判断都稳定。

**随机森林如"带蒙眼的专家委员会"**：每个专家不仅看到的数据不同（样本扰动），连能参考的线索也不同（属性扰动）。因此他们不会互相模仿，反而各自形成独到的判断，委员们的综合意见往往比全知全能的单个专家还准。

**梯度提升如"沿着陡坡修台阶"**：你站在山脚（当前模型），用梯度找到最陡的上坡方向，放一块基学习器的"台阶"踩上去。每步只走一小段（学习率 $\eta$），这样走得稳——这就是 shrinkage 的思想。

**误差-分歧分解如"跷跷板"**：一边是个体准确性，另一边是分歧（多样性）。要压总误差这个跷跷板，你需要在两边同时施加向下压力——但个体准确性高了自然会减少多样性，这是个永恒的博弈。

---

## 常见误区 (Anti-patterns)

- **误区："集成学习一定能提高性能"**：不是。如果基学习器完全相同，集成无效果（如三个40%精度的相似弱分类器集成反而可能更差）。"好而不同"是必要条件
- **误区："Boosting不会过拟合"**：虽然Boosting在训练误差达到零后测试误差仍可能继续下降（margin理论），但这不是无边界的。当基学习器过于复杂（如深度决策树）且轮数过多时，Boosting仍会过拟合
- **误区："Bagging与Boosting效果差不多，随便选"**：两者机制完全不同——Boosting降低偏差适合弱学习器；Bagging降低方差适合不稳定的强学习器。选择错误可能导致效果不提升
- **误区："随机森林中 $k$ 越小越好"**：$k$ 越小属性扰动越大、多样性增强，但个体学习器精度下降。$k = \log_2 d$ 是经大量实验验证的经验值，不可随意取极小
- **误区："Stacking的次级学习器越复杂越好"**：次级学习器过复杂容易过拟合，反而降低泛化性能。实践中用简单模型（如线性回归、LR）作元学习器通常足够
- **误区："集成学习基学习器越多越好"**：边际效益递减——100个基学习器之后的提升往往微乎其微，但计算开销线性增长。实践中通常几十到几百个即可

---

## 代码要点 (Code Essentials)

- **基学习器选择**：Boosting通常用浅层决策树（如决策树桩/深度为1-3的树）；Bagging/RF用完全生长的决策树（不剪枝，高方差低偏差）
- **学习率 (shrinkage)**：GBDT和XGBoost中，$\nu$ (learning rate，如0.1) 与迭代次数 $T$ 构成trade-off。较小的 $\nu$ 需更大的 $T$，但泛化性能通常更好
- **早停 (Early Stopping)**：对GBDT/XGBoost在验证集上监控性能，连续N轮不提升则停止训练
- **子采样 (subsample)**：梯度提升中每轮随机采样部分样本（如0.8），增加随机性、防止过拟合——这也是随机梯度提升(SGB)的核心
- **特征重要性**：RF和GBDT可输出特征重要性（基于分裂次数或基尼指数/信息增益减少量），用于特征选择
- **类别不平衡处理**：对AdaBoost可调整样本初始权重；对Bagging可在Bootstrap采样时进行分层采样
- **并行化**：Bagging/RF天然适合并行（各基学习器独立训练）；XGBoost和LightGBM也支持特征/数据的并行化加速

---

## 实例分析 (Worked Example)

**AdaBoost在西瓜数据集3.0$\alpha$上的决策边界** (西瓜书):

以决策树桩（一层决策树）为基学习器，AdaBoost在西瓜数据集上的运行过程：

- 初始所有样本权重相等（$1/17$）
- 第1轮：学到的树桩可能以单一属性（如"含糖率"）分类，加权错误率 $\epsilon_1$，计算得 $\alpha_1$
- 第2轮：误分类样本权重放大，迫使新基学习器关注上一轮错分的样本
- 第3轮：集成决策边界开始变得非线性
- 第11轮：决策边界高度复杂，拟合了样本的细节分布

对比Bagging（同基学习器）：Bagging的决策边界提升相对平稳但有限，核心是降低方差；AdaBoost能显著改变边界的复杂度以降低偏差。图6.4展示了规模为3、5、11时AdaBoost边界的变化——边界从简单到复杂，体现了Boosting逐渐"纠正错误"的过程。

**Bagging vs Random Forest的方差降低**：假设每个基学习器方差为 $\sigma^2$，两两相关系数为 $\rho$，则 $T$ 个学习器经简单平均后的方差为：

$$\text{Var}(H) = \rho\sigma^2 + \frac{(1-\rho)\sigma^2}{T}$$

Bagging通过样本扰动降低 $\rho$；RF通过属性扰动进一步降低 $\rho$（即个体间的相关性更弱），从而获得更低的集成方差。

---

## 关键结论 (Key Takeaways)

1. 集成学习的核心是"好而不同"——准确性保证不差于随机，多样性保证互补。误差-分歧分解给出了数学表述
2. AdaBoost = 加性模型 + 指数损失的前向分步算法。这一推导框架揭示AdaBoost不是启发式而是具有坚实理论基础
3. Boosting降低偏差（适合弱学习器），Bagging降低方差（适合高方差学习器如未剪枝决策树），随机森林通过双重扰动进一步增强多样性
4. 梯度提升将Boosting推广到任意损失函数，是结构化数据上最强的集成方法之一（GBDT/XGBoost/LightGBM/CatBoost）
5. 集成规模(个体数T)并非越大越好——边际效益递减，且AdaBoost可能在噪声大时过拟合
6. Bagging的OOB估计提供了一个"免费的"验证集，无需像交叉验证额外划分数据

---

## 章节连接 (Connects To)

- **前置**: 决策树 (Ch4) — 集成学习中最常用的基学习器；偏差-方差分解 (Ch2) — 理解Bagging/Boosting的差异
- **后续**: 西瓜书Ch9聚类 — 聚类集成；深度学习中的集成 — Dropout可视为对神经网络神经元的隐式Bagging；XGBoost — Boosting在工业界的巅峰实现

