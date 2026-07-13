# Chapter 19: EM算法及其推广 / EM Algorithm and Its Generalization

## 核心概念 (Core Idea)

EM 算法是一种迭代算法，用于含有**隐变量** (latent variable) 的概率模型参数的极大似然估计（或极大后验概率估计）。其核心思想是：当模型中存在不可观测的隐变量时，直接最大化观测数据的不完全似然很困难；EM 算法通过迭代地构造似然函数的下界（E 步求期望）并最大化该下界（M 步求极大），来逼近极大似然估计。该算法由 Dempster 等人于 1977 年系统总结提出。

## 理论基础 (Theoretical Foundation)

### 问题设定

设有概率模型，其中 $Y$ 表示观测变量的数据（不完全数据），$Z$ 表示隐变量的数据，$Y$ 和 $Z$ 连在一起称为完全数据。观测数据的似然函数为 $P(Y|\theta)$，对数似然函数为 $L(\theta) = \log P(Y|\theta)$；完全数据的对数似然函数为 $\log P(Y, Z|\theta)$。

目标是最大化观测数据的对数似然：
$$\hat{\theta} = \arg\max_\theta \log P(Y|\theta) = \arg\max_\theta \log \sum_Z P(Y, Z|\theta) \tag{1}$$

困难在于：式中有未观测数据 $Z$，并包含求和（或积分）的对数——即"和的 log"，没有解析解。

### Jensen 不等式

EM 算法的推导核心工具是 **Jensen 不等式**：对于凸函数 $f$，有 $f(\mathbb{E}[X]) \leq \mathbb{E}[f(X)]$。特别地，$\log$ 是凹函数，故：

$$\log \sum_j \lambda_j y_j \geq \sum_j \lambda_j \log y_j$$

其中 $\lambda_j \geq 0, \sum_j \lambda_j = 1$。

### EM 算法的核心：Q 函数

**定义**（Q 函数）：完全数据的对数似然函数 $\log P(Y, Z|\theta)$ 在给定观测数据 $Y$ 和当前参数 $\theta^{(i)}$ 下，关于未观测数据 $Z$ 的条件概率分布 $P(Z|Y, \theta^{(i)})$ 的期望称为 Q 函数：

$$Q(\theta, \theta^{(i)}) = E_Z[\log P(Y, Z|\theta) | Y, \theta^{(i)}] = \sum_Z \log P(Y, Z|\theta) P(Z|Y, \theta^{(i)}) \tag{2}$$

注意：$Q(\theta, \theta^{(i)})$ 的第一个变元表示要极大化的参数，第二个变元表示参数的当前估计值。

## 理论推导 (Theoretical Derivation)

### EM 算法的导出（下界最大化视角）

假设在第 $i$ 次迭代后 $\theta$ 的估计值是 $\theta^{(i)}$，希望新估计值 $\theta$ 能使 $L(\theta) > L(\theta^{(i)})$。考虑两者的差：

$$L(\theta) - L(\theta^{(i)}) = \log\left(\sum_Z P(Y|Z, \theta) P(Z|\theta)\right) - \log P(Y|\theta^{(i)})$$

引入条件分布 $P(Z|Y, \theta^{(i)})$ 作为 Jensen 不等式的权重：

$$\begin{aligned}
L(\theta) - L(\theta^{(i)}) &= \log\left(\sum_Z P(Z|Y, \theta^{(i)}) \frac{P(Y|Z, \theta) P(Z|\theta)}{P(Z|Y, \theta^{(i)})}\right) - \log P(Y|\theta^{(i)}) \\
&\geq \sum_Z P(Z|Y, \theta^{(i)}) \log\frac{P(Y|Z, \theta) P(Z|\theta)}{P(Z|Y, \theta^{(i)})} - \log P(Y|\theta^{(i)}) \\
&= \sum_Z P(Z|Y, \theta^{(i)}) \log\frac{P(Y|Z, \theta) P(Z|\theta)}{P(Z|Y, \theta^{(i)}) P(Y|\theta^{(i)})}
\end{aligned}$$

令下界函数为：

$$B(\theta, \theta^{(i)}) = L(\theta^{(i)}) + \sum_Z P(Z|Y, \theta^{(i)}) \log\frac{P(Y|Z, \theta) P(Z|\theta)}{P(Z|Y, \theta^{(i)}) P(Y|\theta^{(i)})} \tag{3}$$

则有 $L(\theta) \geq B(\theta, \theta^{(i)})$ 且 $L(\theta^{(i)}) = B(\theta^{(i)}, \theta^{(i)})$。因此，任何使 $B(\theta, \theta^{(i)})$ 增大的 $\theta$ 也会使 $L(\theta)$ 增大。

选择 $\theta^{(i+1)}$ 使 $B(\theta, \theta^{(i)})$ 极大化，省去常数项后：

$$\begin{aligned}
\theta^{(i+1)} &= \arg\max_\theta \left(L(\theta^{(i)}) + \sum_Z P(Z|Y, \theta^{(i)}) \log\frac{P(Y|Z, \theta) P(Z|\theta)}{P(Z|Y, \theta^{(i)}) P(Y|\theta^{(i)})}\right) \\
&= \arg\max_\theta \left(\sum_Z P(Z|Y, \theta^{(i)}) \log P(Y, Z|\theta)\right) \\
&= \arg\max_\theta Q(\theta, \theta^{(i)})
\end{aligned}$$

这就从下界最大化的视角导出了 EM 算法的两步迭代。

### 收敛性理论

**定理 1**（似然函数单调递增）：设 $P(Y|\theta)$ 为观测数据的似然函数，$\theta^{(i)}$ 为 EM 算法得到的参数估计序列，则 $P(Y|\theta^{(i)})$ 是单调递增的：

$$P(Y|\theta^{(i+1)}) \geq P(Y|\theta^{(i)}) \tag{4}$$

**证明概要**：

由 $\log P(Y|\theta) = \log P(Y, Z|\theta) - \log P(Z|Y, \theta)$，定义：

$$Q(\theta, \theta^{(i)}) = \sum_Z \log P(Y, Z|\theta) P(Z|Y, \theta^{(i)})$$
$$H(\theta, \theta^{(i)}) = \sum_Z \log P(Z|Y, \theta) P(Z|Y, \theta^{(i)})$$

则 $\log P(Y|\theta) = Q(\theta, \theta^{(i)}) - H(\theta, \theta^{(i)})$。

考虑 $\log P(Y|\theta^{(i+1)}) - \log P(Y|\theta^{(i)})$：
- 第一项 $Q(\theta^{(i+1)}, \theta^{(i)}) - Q(\theta^{(i)}, \theta^{(i)}) \geq 0$（因为 M 步极大化 Q）
- 第二项 $H(\theta^{(i+1)}, \theta^{(i)}) - H(\theta^{(i)}, \theta^{(i)}) \leq 0$（由 Jensen 不等式）

二者相减得到非负，证毕。

**定理 2**（收敛性）：
1. 如果 $P(Y|\theta)$ 有上界，则 $L(\theta^{(i)})$ 收敛到某一值 $L^*$
2. 在一定条件下，参数估计序列 $\theta^{(i)}$ 的收敛值 $\theta^*$ 是 $L(\theta)$ 的稳定点

**重要说明**：定理只能保证收敛到稳定点，**不能保证收敛到全局最大值**或**局部极大值**。因此在实际中初值的选择非常重要，常用的方法是选取多个不同的初值进行迭代，从中选择最好的结果。

## 核心概念 (Key Concepts)

- **隐变量 (Latent Variable)**：模型中不可观测的随机变量，数据生成过程中的潜在因素
- **完全数据 (Complete Data)**：$(Y, Z)$，即观测数据和隐数据的并集
- **不完全数据 (Incomplete Data)**：仅有观测数据 $Y$
- **Q 函数 (Q-function)**：$Q(\theta, \theta^{(i)}) = E_Z[\log P(Y, Z|\theta) | Y, \theta^{(i)}]$，是 EM 算法的核心目标函数
- **E 步 (Expectation Step)**：计算 Q 函数——在当前参数 $\theta^{(i)}$ 下，求完全数据对数似然的期望
- **M 步 (Maximization Step)**：极大化 Q 函数——求 $\theta^{(i+1)} = \arg\max_\theta Q(\theta, \theta^{(i)})$
- **下界函数 (Lower Bound)**：$B(\theta, \theta^{(i)})$ 是对数似然的下界，在 $\theta = \theta^{(i)}$ 处与似然函数相等
- **GEM (Generalized EM)**：不要求 M 步精确最大化 Q，只要增大 Q 值即可

## 算法步骤 (Algorithm Steps)

### 算法1：EM 算法

**输入**：观测变量数据 $Y$，隐变量数据 $Z$，联合分布 $P(Y, Z|\theta)$，条件分布 $P(Z|Y, \theta)$
**输出**：模型参数 $\theta$

1. 选择参数的初值 $\theta^{(0)}$，开始迭代
2. **E 步**：记 $\theta^{(i)}$ 为第 $i$ 次迭代参数 $\theta$ 的估计值，在第 $i+1$ 次迭代的 E 步，计算 Q 函数：
   $$Q(\theta, \theta^{(i)}) = E_Z[\log P(Y, Z|\theta) | Y, \theta^{(i)}] = \sum_Z \log P(Y, Z|\theta) P(Z|Y, \theta^{(i)})$$
3. **M 步**：求使 $Q(\theta, \theta^{(i)})$ 极大化的 $\theta$，确定第 $i+1$ 次迭代的参数估计值：
   $$\theta^{(i+1)} = \arg\max_\theta Q(\theta, \theta^{(i)})$$
4. 重复步骤 2 和 3，直到收敛（例如 $\|\theta^{(i+1)} - \theta^{(i)}\| < \varepsilon_1$ 或 $\|Q(\theta^{(i+1)}, \theta^{(i)}) - Q(\theta^{(i)}, \theta^{(i)})\| < \varepsilon_2$）

### 算法2：GEM 算法2（弱化的 M 步）

当 $Q(\theta, \theta^{(i)})$ 的极大化困难时，不求精确极大，只需找 $\theta^{(i+1)}$ 使得：

$$Q(\theta^{(i+1)}, \theta^{(i)}) > Q(\theta^{(i)}, \theta^{(i)})$$

即可。

### F 函数的极大-极大算法解释

**定义**（F 函数）：假设隐变量数据 $Z$ 的概率分布为 $\tilde{P}(Z)$，定义：

$$F(\tilde{P}, \theta) = E_{\tilde{P}}[\log P(Y, Z|\theta)] + H(\tilde{P}) \tag{5}$$

其中 $H(\tilde{P}) = -E_{\tilde{P}} \log \tilde{P}(Z)$ 是 $\tilde{P}(Z)$ 的熵。

EM 算法的每次迭代等价于 F 函数的两次极大化：
1. 固定 $\theta^{(i)}$，求 $\tilde{P}^{(i+1)}$ 使 $F(\tilde{P}, \theta^{(i)})$ 极大化 $\to$ $\tilde{P}^{(i+1)}(Z) = P(Z|Y, \theta^{(i)})$
2. 固定 $\tilde{P}^{(i+1)}$，求 $\theta^{(i+1)}$ 使 $F(\tilde{P}^{(i+1)}, \theta)$ 极大化 $\to$ 等价于 M 步

## 直观理解 (Intuition)

- **"鸡生蛋、蛋生鸡"问题**：如果知道隐变量 $Z$ 的值，参数估计就很简单；如果知道参数 $\theta$，推断 $Z$ 也很简单。EM 通过交替地进行这两步来迭代求解。

- **下界攀升**：在每一次迭代中，EM 构造一个在当前参数点与似然函数相切的下界函数 $B(\theta, \theta^{(i)})$，然后最大化这个下界来得到新的参数。因为下界在切点处与似然函数相等，优化下界必然使似然函数增大。

- **"软指派"**：E 步不是硬性地将每个数据点指派给一个隐状态，而是计算一个"软"的概率分布（responsibility），即"这个数据点以多大概率来自某个隐状态"。这种软指派平滑了优化目标。

- **整体 vs 局部**：EM 算法保证似然函数单调递增，但爬山的方向取决于初值——从不同初值出发可能到达不同的局部极大值。

## 常见误区 (Anti-patterns)

- **误区：EM 保证找到全局最优解**。EM 只保证收敛到似然函数的稳定点，通常是局部极大，不能保证全局最优。实践中需多随机初始化和比选。
- **误区：EM 算法中 E 步和 M 步完全分离**。E 步需要当前参数 $\theta^{(i)}$，M 步需要 E 步算出的 Q 函数。两步交替、紧密耦合，不是独立的。
- **误区：M 步必须求出闭合解**。当 M 步无闭合解时，可使用 GEM（仅需增大 Q 而非最大化），或在 M 步内部使用数值优化（如梯度上升）。
- **误区：EM 算法收敛很快**。EM 在接近最优解时收敛可能非常慢（线性收敛），实践中常先用 EM 获得较好初始解再切换到 Newton 类方法。
- **误区：EM 只能用于极大似然**。EM 同样适用于极大后验概率估计（MAP），只需在 M 步目标函数中加入先验的对数项。

## 代码要点 (Code Essentials)

- 初始化对 EM 至关重要：GMM 常用 k-means 结果初始化；HMM 用随机初始化
- 收敛判断：可检查参数变化量或对数似然的变化量。对数似然更可靠（单调性保证）
- 数值稳定性：计算概率乘积时可能下溢，E 步和 M 步中都应在 log 空间操作（log-sum-exp 技巧）
- GMM 的 M 步：均值 $\mu_k = \frac{\sum_j \hat{\gamma}_{jk} y_j}{\sum_j \hat{\gamma}_{jk}}$，方差 $\sigma_k^2 = \frac{\sum_j \hat{\gamma}_{jk} (y_j - \mu_k)^2}{\sum_j \hat{\gamma}_{jk}}$，混合系数 $\alpha_k = \frac{\sum_j \hat{\gamma}_{jk}}{N}$
- 协方差矩阵退化：当某个高斯分量的数据点太少时，协方差矩阵可能奇异，可加正则化项 ($\epsilon I$) 或限制协方差形式

## 实例分析 (Worked Example)

### 三硬币模型

假设有 3 枚硬币 A, B, C，正面概率分别为 $\pi, p, q$。试验过程：先掷 A，若正面选 B 掷，若反面选 C 掷；观测掷出硬币的结果，但不能观测掷硬币的过程。

观测数据：$1,1,0,1,0,0,1,0,1,1$（$n=10$ 次独立重复）。

**模型**：
$$P(y|\theta) = \sum_z P(y, z|\theta) = \pi p^y (1-p)^{1-y} + (1-\pi) q^y (1-q)^{1-y} \tag{6}$$

其中 $y$ 是观测变量，$z$ 是隐变量（掷 A 的结果），$\theta = (\pi, p, q)$。

**EM 算法**（取初值 $\pi^{(0)} = p^{(0)} = q^{(0)} = 0.5$）：

E 步 — 计算观测 $y_j$ 来自硬币 B 的概率（响应度）：
$$\mu_j^{(i+1)} = \frac{\pi^{(i)} (p^{(i)})^{y_j} (1-p^{(i)})^{1-y_j}}{\pi^{(i)} (p^{(i)})^{y_j} (1-p^{(i)})^{1-y_j} + (1-\pi^{(i)}) (q^{(i)})^{y_j} (1-q^{(i)})^{1-y_j}}$$

M 步 — 更新参数：
$$\pi^{(i+1)} = \frac{1}{n}\sum_{j=1}^{n} \mu_j^{(i+1)}$$
$$p^{(i+1)} = \frac{\sum_{j=1}^{n} \mu_j^{(i+1)} y_j}{\sum_{j=1}^{n} \mu_j^{(i+1)}}$$
$$q^{(i+1)} = \frac{\sum_{j=1}^{n} (1-\mu_j^{(i+1)}) y_j}{\sum_{j=1}^{n} (1-\mu_j^{(i+1)})}$$

**迭代结果**（初值 0.5, 0.5, 0.5）：$\hat{\pi} = 0.5, \hat{p} = 0.6, \hat{q} = 0.6$。

**初值敏感性**：若取初值 $\pi^{(0)} = 0.4, p^{(0)} = 0.6, q^{(0)} = 0.7$，则得 $\hat{\pi} \approx 0.406, \hat{p} \approx 0.537, \hat{q} \approx 0.643$。说明 EM 算法与初值选择有关，不同初值可能得到不同估计。

## 关键结论 (Key Takeaways)

1. EM 算法解决含隐变量的概率模型极大似然估计，通过 E 步（计算 Q 函数）和 M 步（最大化 Q 函数）迭代实现
2. EM 的数学基础是 Jensen 不等式和下界最大化——每次迭代构造并最大化似然函数的下界
3. EM 保证对数似然函数单调递增（每次迭代后不降低），但不保证收敛到全局最优——初值选择至关重要
4. EM 算法的核心在于定义 Q 函数——完全数据对数似然在当前参数下关于隐变量后验的期望
5. GMM 和 HMM (Baum-Welch) 是 EM 算法的两个最重要的应用，体现了 EM 在聚类和序列标注中的核心地位

## 章节连接 (Connects To)

- **前置**: 极大似然估计（MLE）、Jensen 不等式、Chapter 7 (SVM——同为优化驱动的学习方法)
- **后续**: Chapter 20 (HMM & CRF——Baum-Welch 算法是 EM 在 HMM 的具体实现), Chapter 6 (逻辑斯谛回归与最大熵——对数线性模型的 MLE), Chapter 14 (聚类——GMM 是一种基于概率模型的软聚类方法)
- **互补阅读**: 南瓜书对应推导（Jensen 不等式不等号方向、Q 函数求导细节）
