# Chapter 5: 神经网络基础 / Neural Networks Fundamentals

## 核心概念 (Core Idea)

神经网络是由具有适应性的简单单元（神经元）组成的广泛并行互连网络。其核心思想是通过多层非线性变换，从数据中自动学习层次化的特征表示，从而完成分类、回归等任务。训练多层网络的关键是误差逆传播（BP）算法，它利用链式法则将输出层的误差逐层反向传播以更新所有参数。

---

## 融合框架 (Integrated Frameworks)

- **感知机模型与学习**：从M-P神经元出发，构建感知机模型及参数更新规则 （西瓜书）/(南瓜书)
- **多层前馈网络**：定义隐层、输出层，给出任意深度的矩阵形式前馈网络 (李航P2)
- **BP算法推导**：基于梯度下降策略，利用链式法则逐层计算梯度 (西瓜书)/(南瓜书) — 李航P2提供计算图视角的实现
- **深度学习理念**：无监督逐层预训练 + BP微调（DBN），卷积+池化架构（CNN） (西瓜书)
- **正则化技术**：早停法、权重衰减、Dropout正则化 (李航P2)
- **优化算法**：SGD到Adam的演进，学习率调度 (李航P2)

---

## 理论推导 (Theoretical Derivation)

### 1. 感知机学习算法

感知机模型为 $y = \varepsilon(\mathbf{w}^\top\mathbf{x} - \theta)$，其中 $\varepsilon$ 为阶跃函数。将阈值 $\theta$ 视为固定输入 $-1$ 对应的权重 $w_{n+1}$，模型简化为 $y = \varepsilon(\mathbf{w}^\top\mathbf{x})$。

对误分类样本 $(x_i, y_i) \in M$，损失函数定义为：

$$L(\mathbf{w}) = \sum_{\mathbf{x}_i \in M} (\hat{y}_i - y_i) \mathbf{w}^\top \mathbf{x}_i$$

采用随机梯度下降（SGD），每次随机选取一个误分类点，权重更新为：

$$\mathbf{w} \leftarrow \mathbf{w} + \eta (y_i - \hat{y}_i) \mathbf{x}_i$$

即 $\Delta w_i = \eta (y - \hat{y}) x_i$。当数据线性可分时，算法收敛；否则将发生振荡。

### 2. 前馈神经网络的矩阵形式

一个 $s$ 层前馈神经网络定义为复合函数 (李航P2)：

$$\begin{cases}
h^{(1)} = a(W^{(1)\top}x + b^{(1)}) \\
h^{(2)} = a(W^{(2)\top}h^{(1)} + b^{(2)}) \\
\quad \vdots \\
y = h^{(s)} = g(W^{(s)\top}h^{(s-1)} + b^{(s)})
\end{cases}$$

其中 $W^{(t)}$ 为第 $t$ 层权重矩阵，$b^{(t)}$ 为偏置向量，$a(\cdot)$ 为隐层激活函数，$g(\cdot)$ 为输出层激活函数。

### 3. BP算法详细推导 (链式法则)

对训练样例 $(\mathbf{x}_k, \mathbf{y}_k)$，均方误差为：

$$E_k = \frac{1}{2}\sum_{j=1}^{l} (\hat{y}_j^k - y_j^k)^2$$

采用梯度下降策略，对任意参数 $v$ 有 $\Delta v = -\eta \frac{\partial E_k}{\partial v}$。

**隐层到输出层的连接权 $w_{hj}$ 的梯度** (西瓜书)：

$$\frac{\partial E_k}{\partial w_{hj}} = \frac{\partial E_k}{\partial \hat{y}_j^k} \cdot \frac{\partial \hat{y}_j^k}{\partial \beta_j} \cdot \frac{\partial \beta_j}{\partial w_{hj}}$$

其中 $\beta_j = \sum_{h=1}^{q} w_{hj} b_h$，利用 Sigmoid 函数性质 $f'(x) = f(x)(1-f(x))$：

$$g_j = -\frac{\partial E_k}{\partial \hat{y}_j^k} \cdot \frac{\partial \hat{y}_j^k}{\partial \beta_j} = \hat{y}_j^k(1-\hat{y}_j^k)(y_j^k - \hat{y}_j^k)$$

得更新公式：$\Delta w_{hj} = \eta g_j b_h$，$\Delta \theta_j = -\eta g_j$。

**输入层到隐层的连接权 $v_{ih}$ 的梯度** (南瓜书)：

类似链式展开得 $e_h = b_h(1-b_h)\sum_{j=1}^{l} w_{hj} g_j$，则：

$$\Delta v_{ih} = \eta e_h x_i, \quad \Delta \gamma_h = -\eta e_h$$

**反向传播的误差传播公式** (李航P2)：

$$\delta_j^{(t)} = \frac{\mathrm{d}a}{\mathrm{d}z_j^{(t)}} \sum_{k=1}^{l} w_{kj}^{(t+1)} \delta_k^{(t+1)}$$

对于平方损失 + 恒等输出：$\delta^{(s)} = h^{(s)} - y$

对于交叉熵 + Softmax 输出：$\delta_k^{(s)} = h_k^{(s)} - y_k$

这说明无论回归还是分类，输出层误差 $\delta^{(s)}$ 恰为预测值与真实值之差——这也是"误差逆传播"名称的由来。

### 4. 梯度消失与爆炸

从反向传播公式 $\delta^{(t)} = a'(z^{(t)}) \odot (W^{(t+1)\top} \delta^{(t+1)})$ 可以看出：

- **Sigmoid/Tanh 饱和区**：当 $|z|$ 较大时，$a'(z) \to 0$，导致深层梯度趋近于零（梯度消失）
- **ReLU左饱和**：$z < 0$ 时导数为 0，可能产生"死亡ReLU"；$z > 0$ 时导数为 1，缓解梯度消失
- **梯度爆炸**：当权重初始值过大时，$W\delta$ 的连乘导致指数增长

---

## 核心概念 (Key Concepts)

- **M-P神经元模型**：$y = f(\sum_i w_i x_i - \theta)$，接收多个输入信号，通过加权求和并与阈值比较，经激活函数产生输出
- **激活函数 (Activation Function)**：
  - **Sigmoid**: $\sigma(z) = \frac{1}{1+e^{-z}}$，输出 (0,1)，导数为 $\sigma(z)(1-\sigma(z))$，饱和函数
  - **Tanh**: $\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}$，输出 (-1,1)，导数为 $1-\tanh^2(z)$，饱和函数
  - **ReLU**: $\text{relu}(z) = \max(0, z)$，输出 $[0, +\infty)$，左饱和非饱和，计算高效
- **多层前馈网络 (Multi-layer Feedforward)**：层级结构，每层神经元与下一层全互连，无同层/跨层连接
- **误差逆传播 (BP)**：利用链式法则计算损失函数对每层参数的梯度，从输出层向输入层反向传播误差
- **全局最小 vs 局部极小**：梯度下降可能陷入局部极小；应对策略：多组随机初始化、模拟退火、SGD
- **通用近似定理**：含足够多隐层神经元的单隐层前馈网络可以任意精度逼近任意连续函数

---

## 算法步骤 (Algorithm Steps)

### BP算法 (标准版 — 西瓜书)

1. 在 (0,1) 范围内随机初始化所有连接权和阈值
2. **Repeat**:
   - **For each** $(\mathbf{x}_k, \mathbf{y}_k) \in D$:
     - (i) 正向传播：根据当前参数计算 $\hat{\mathbf{y}}_k$
     - (ii) 计算输出层梯度项 $g_j = \hat{y}_j^k(1-\hat{y}_j^k)(y_j^k - \hat{y}_j^k)$
     - (iii) 计算隐层梯度项 $e_h = b_h(1-b_h)\sum_j w_{hj}g_j$
     - (iv) 更新 $w_{hj}, v_{ih}, \theta_j, \gamma_h$
3. **Until** 达到停止条件
4. 输出：连接权与阈值确定的多层前馈神经网络

### 随机梯度下降 BP (李航P2)

1. 随机打乱样本，分成 mini-batches
2. 随机初始化参数向量 $\theta$
3. **Do while** (不收敛):
   - **For each** mini-batch:
     - 正向传播得各层输出
     - 反向传播得各层误差 $\delta^{(s)}, \ldots, \delta^{(1)}$
     - 计算梯度 $\nabla_{W^{(t)}}L = \delta^{(t)} \cdot h^{(t-1)\top}$，$\nabla_{b^{(t)}}L = \delta^{(t)}$
     - 更新 $W^{(t)} \leftarrow W^{(t)} - \eta \nabla_{W^{(t)}}L$，$b^{(t)} \leftarrow b^{(t)} - \eta \nabla_{b^{(t)}}L$
     - 将误差反向传播: $\delta^{(t-1)} = \frac{\partial a}{\partial z^{(t-1)}} \odot (W^{(t)\top} \cdot \delta^{(t)})$

---

## 直观理解 (Intuition)

**感知机如"单刀直入"的法官**：只看一次证据就做判决。线性可分时必然收敛，但异或（XOR）这样的问题需要"迂回"判断，单层感知机无能为力。

**多层网络如"层层审批"**：原始输入经过逐层加工，低层提取简单特征（边缘），中层组合成局部模式（部件），高层形成全局语义（对象）。这正是"特征学习"的本质——模型自己学会"看什么"而不是依赖人工设计特征。

**BP算法如"追责机制"**：如果最终结果错了，从输出层开始逐层"追责"——每个神经元对错误的"贡献"是多少，谁贡献大就调整谁的参数更多。链式法则就是这套追责机制的数学表述。

**梯度下降如"盲人下山"**：蒙着眼睛往最陡的下坡方向走。局部极小就是个山坳——每次走都往上走，以为到了山谷底，但其实真正的谷底（全局最小）还在远处。SGD 的随机性则像每步都稍微"滑一下"，有机会滑出小山坳。

---

## 常见误区 (Anti-patterns)

- **误区: "层数越多越好"**：不总是。增加了梯度消失/爆炸风险，训练难度增大。实际中需通过残差连接 (ResNet)、BatchNorm 等技术配合
- **误区: "BP算法能保证找到全局最优"**：不能。神经网络的优化目标是非凸的，存在大量等价局部最优点。实际中多数局部最优的泛化性能已足够好
- **误区: "Sigmoid 激活函数在所有层统一使用"**：Sigmoid 的输出非零中心 (0.5)，导致后层神经元输入恒为正，参数更新"之"字形下降，收敛慢。现代实践偏好 ReLU 及其变体
- **误区: "学习率越大收敛越快"**：过大导致参数在最优解附近振荡甚至发散；过小导致收敛极慢。需要学习率衰减策略或自适应方法如 Adam
- **误区: "正则化 $\lambda$ 越大越好"**：过大的正则化会使模型退化到过于简单，欠拟合严重

---

## 代码要点 (Code Essentials)

- **权重初始化**：不能初始化为全零（对称性问题）。常用 Xavier/Glorot 初始化：$W \sim U(-\frac{\sqrt{6}}{\sqrt{n_{in}+n_{out}}}, \frac{\sqrt{6}}{\sqrt{n_{in}+n_{out}}})$ 或 He 初始化（ReLU 激活时）
- **梯度检查**：实现时用有限差分法验证解析梯度：$\frac{\partial L}{\partial \theta} \approx \frac{L(\theta+\epsilon)-L(\theta-\epsilon)}{2\epsilon}$，相对误差应在 $10^{-7}$ 量级
- **特征归一化**：输入特征应归一化到 [0,1] 或标准化为 N(0,1)，加速收敛
- **Mini-batch 训练**：batch size 通常取 32/64/128/256，兼顾梯度估计准确性和计算效率
- **损失函数选择**：回归用 MSE，二分类用 Binary Cross-Entropy，多分类用 Categorical Cross-Entropy + Softmax
- **防止过拟合**：早停 (Early Stopping) 基于验证集监控；Dropout 在训练时随机丢弃神经元输出；L2 正则化 (权重衰减)

---

## 实例分析 (Worked Example)

**XOR 问题用二层神经网络求解** (西瓜书 + 李航P2):

构建一个二层前馈网络：
- 隐层有 2 个神经元，激活函数为阶跃函数（或 ReLU）
- 第一层权重: $W^{(1)} = \begin{bmatrix} 1 & 1 \\ 1 & 1 \end{bmatrix}$，偏置: $b^{(1)} = \begin{bmatrix} 0 \\ -1 \end{bmatrix}$
- 第二层权重: $W^{(2)} = \begin{bmatrix} 1 \\ -2 \end{bmatrix}$，偏置: $b^{(2)} = [0]$

输入矩阵 $X$ (批量四个样本)：

$$X = \begin{bmatrix} 0 & 0 & 1 & 1 \\ 0 & 1 & 0 & 1 \end{bmatrix}$$

第一层输出（ReLU 激活）：

$$H^{(1)} = \text{relu}\left(\begin{bmatrix} 0 & 1 & 1 & 2 \\ -1 & 0 & 0 & 1 \end{bmatrix}\right) = \begin{bmatrix} 0 & 1 & 1 & 2 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

第二层输出：$H^{(2)} = \begin{bmatrix} 1 & -2 \end{bmatrix} H^{(1)} = \begin{bmatrix} 0 & 1 & 1 & 0 \end{bmatrix}$

输出结果为 $[0, 1, 1, 0]$，精确实现 XOR 功能。单层感知机无法做到，但两层网络以非常简单的方式实现了。

---

## 关键结论 (Key Takeaways)

1. 感知机仅适用于线性可分问题，多层网络通过引入隐层和非线性激活函数，可以处理非线性可分问题
2. BP算法 = 正向传播（计算输出）+ 反向传播（链式法则计算梯度），本质是梯度下降在多层的具体实现
3. 激活函数的导数性质决定了梯度传播效率：ReLU 缓解梯度消失，Sigmoid/Tanh 易饱和
4. 神经网络的优化是非凸问题，存在海量等价局部最优点；实践中 SGD 的随机性反而有助于跳出不良局部极小
5. 深度网络比浅层网络有更好的表示效率——同等表示能力下深度网络参数更少（指数级优势）
6. 正则化（Dropout、权重衰减、早停）是防止过拟合的三大标准手段

---

## 章节连接 (Connects To)

- **前置**: 线性模型 (Ch3)、决策树 (Ch4) — 理解基础分类/回归
- **后续**: 支持向量机 (Ch6) — SVM 可视为一种特殊的神经网络；深度学习 (高级课程) — CNN/RNN/Transformer
