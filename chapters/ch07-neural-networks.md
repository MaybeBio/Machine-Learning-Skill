# 第5章: 神经网络与BP算法 / Neural Networks & Backpropagation

> **融合来源**: 西瓜书第5章(神经网络) + 南瓜书第5章 + 李航第23章(前馈神经网络) + 清华大学PPT

---

## 一、入门理解 — 周志华《机器学习》(西瓜书)

### 1.1 神经元模型与感知机

神经网络由具有适应性的**简单单元**(神经元)组成。M-P神经元模型：接收来自$d$个其他神经元的输入信号，通过带权重的连接传递，总输入与阈值比较后经**激活函数**处理产生输出：

$$y = f\left(\sum_{i=1}^{d} w_i x_i - \theta\right)$$

西瓜书介绍了最早的神经网络——**感知机**(Perceptron)，由两层神经元组成(输入层+输出层)，只能解决线性可分问题。感知机的学习规则为：$w_i \leftarrow w_i + \Delta w_i$，其中 $\Delta w_i = \eta(y - \hat{y})x_i$。

### 1.2 多层前馈网络与BP算法

多层前馈网络(multi-layer feedforward network)包含输入层、隐层、输出层，每层神经元与下一层全连接，不存在同层连接和跨层连接。

**误差逆传播**(Error BackPropagation, BP)是训练多层网络最经典的算法。核心思想：将输出误差反向传播到隐层，通过**链式法则**计算每个权重的梯度。

给定训练例$(\mathbf{x}_k, \mathbf{y}_k)$，输出为$\hat{\mathbf{y}}_k$，均方误差 $E_k = \frac{1}{2}\sum_{j=1}^{l} (\hat{y}_j^k - y_j^k)^2$。

BP基于梯度下降策略，以目标的负梯度方向调整参数：

$$\Delta w_{hj} = -\eta \frac{\partial E_k}{\partial w_{hj}}$$

西瓜书详细推导了输出层和隐层神经元的梯度项。输出层梯度项：$g_j = \hat{y}_j^k(1-\hat{y}_j^k)(y_j^k - \hat{y}_j^k)$（以sigmoid为例）。隐层梯度项：$e_h = b_h(1-b_h)\sum_{j=1}^{l} w_{hj} g_j$。

### 1.3 全局最小与局部极小

西瓜书讨论了神经网络优化的难点——目标函数非凸，存在多个**局部极小**(local minimum)。跳出局部极小的策略：多组初始值、模拟退火、随机梯度下降的噪声、遗传算法。

**随机梯度下降(SGD)**：每次随机取一个样本更新参数，而非计算整个训练集的梯度（标准/批量梯度下降）。SGD的随机性有助于跳出局部极小，但可能在最优解附近震荡。

---

## 二、公式推导 — 南瓜书补充

### 2.1 BP算法的完整推导链

南瓜书对西瓜书的BP推导做了更系统的展开。对于单隐层网络，南瓜书使用统一的矩阵记号和链式法则。

设第$l$层的净输入为 $\mathbf{z}^{(l)} = \mathbf{W}^{(l)} \mathbf{a}^{(l-1)} + \mathbf{b}^{(l)}$，激活后为 $\mathbf{a}^{(l)} = f(\mathbf{z}^{(l)})$。

输出层误差信号：$\boldsymbol{\delta}^{(L)} = \nabla_{\mathbf{a}^{(L)}} E \odot f'(\mathbf{z}^{(L)})$

隐层误差信号（反向传播）：$\boldsymbol{\delta}^{(l)} = ((\mathbf{W}^{(l+1)})^T \boldsymbol{\delta}^{(l+1)}) \odot f'(\mathbf{z}^{(l)})$

权重梯度：$\nabla_{\mathbf{W}^{(l)}} E = \boldsymbol{\delta}^{(l)} (\mathbf{a}^{(l-1)})^T$

偏置梯度：$\nabla_{\mathbf{b}^{(l)}} E = \boldsymbol{\delta}^{(l)}$

南瓜书特别推导了Sigmoid函数的导数性质：$\sigma'(z) = \sigma(z)(1-\sigma(z))$，Tanh: $\tanh'(z) = 1 - \tanh^2(z)$，ReLU: $\text{ReLU}'(z) = \mathbb{I}(z > 0)$。

### 2.2 激活函数对比

| 激活函数 | 公式 | 导数 | 优缺点 |
|---------|------|------|--------|
| Sigmoid | $\sigma(z)=\frac{1}{1+e^{-z}}$ | $\sigma(1-\sigma)$ | 输出(0,1)，梯度消失 |
| Tanh | $\tanh(z)=\frac{e^z-e^{-z}}{e^z+e^{-z}}$ | $1-\tanh^2$ | 输出(-1,1)，仍有梯度消失 |
| ReLU | $\max(0, z)$ | $\mathbb{I}(z>0)$ | 缓解梯度消失，但神经元可能"死亡" |
| Leaky ReLU | $\max(\alpha z, z)$ | $\mathbb{I}(z>0)+\alpha\mathbb{I}(z<0)$ | 缓解死亡ReLU |

南瓜书详细推导了ReLU造成的梯度消失改善：ReLU在正半轴导数为1，Sigmoid的最大导数仅0.25，多层叠加后呈指数衰减。

---

## 三、理论深化 — 李航《机器学习方法》+ 清华大学PPT

### 3.1 李航Ch23的统计学习框架

李航从前馈神经网络的定义、计算图、自动求导、优化算法等角度做了系统的理论阐述。

**三要素**：模型(网络架构+激活函数)、策略(损失函数+正则化，如交叉熵+权重衰减)、算法(小批量随机梯度下降minibatch SGD及其变体)。

### 3.2 优化算法详解（李航+PPT）

**小批量SGD**：$\mathbf{W} \leftarrow \mathbf{W} - \frac{\eta}{B} \sum_{i=1}^{B} \nabla_{\mathbf{W}} L_i$

| 优化器 | 核心机制 | 公式要点 |
|--------|---------|---------|
| Momentum | 累积历史梯度 | $v_{t+1} = \beta v_t + (1-\beta)\nabla_t$ |
| Nesterov | 前瞻梯度 | $v_{t+1} = \beta v_t + \nabla(\theta_t - \eta\beta v_t)$ |
| RMSProp | 自适应学习率/指数移动平均 | $s_{t+1} = \beta s_t + (1-\beta)\nabla_t^2$ |
| Adam | Momentum + RMSProp + 偏差修正 | $\hat{v}_t = v_t/(1-\beta_1^t)$, $\hat{s}_t = s_t/(1-\beta_2^t)$ |

### 3.3 梯度消失/爆炸与解决方案

李航分析：深层网络中的反向传播涉及多层导数的连续乘积。设每层Jacobian的谱半径$\rho < 1$→梯度消失；$\rho > 1$→梯度爆炸。

解决方案：ReLU激活函数、批归一化(Batch Normalization)、残差连接(ResNet的skip connection)、合适的权重初始化(Xavier/He初始化)。

**批归一化(BN)**：

$$\hat{x}^{(k)} = \frac{x^{(k)} - \mu_B^{(k)}}{\sqrt{\sigma_B^{2(k)} + \epsilon}}, \quad y^{(k)} = \gamma^{(k)} \hat{x}^{(k)} + \beta^{(k)}$$

BN减少内部协变量偏移(Internal Covariate Shift)，加速训练，有一定正则化效果。

### 3.4 深度学习的基础思想（PPT补充）

PPT补充了深度学习的核心思想：(1) **表示学习**(representation learning)—自动学习层次化特征；(2) **端到端学习**(end-to-end learning)—从原始输入到最终输出一体化训练；(3) **计算图**(computation graph)与自动微分—现代框架(PyTorch/TensorFlow)的核心。

---

## 四、综合总结

### 算法对比

| 方法 | 模型 | 核心机制 | 主要问题 |
|------|------|---------|---------|
| 感知机 | 单层 | 线性分类 | XOR不可解 |
| 标准BP | 多层 | 链式法则+SGD | 局部极小，梯度消失 |
| 现代DNN | 深层 | ReLU+BN+Adam+Skip | 调参困难，可解释性差 |

### 关键结论

1. **BP = 链式法则 + 梯度下降**，是深度学习的技术基石
2. **激活函数决定梯度流**：ReLU系列是深层网络训练的默认选择
3. **SGD的随机性是优势**：帮助跳出局部极小，也是隐式的正则化
4. **Adam是最常用的默认优化器**：结合动量和自适应学习率，大部分场景下效果好
5. **梯度消失是深度网络的本质挑战**：ResNet的Skip Connection通过加法路径保持梯度流
6. **批归一化是现代网络的标配**：加速收敛、降低对初始化的敏感度
7. **表示学习是深度学习区别于传统ML的核心**：不需要人工特征工程

### 章节连接

- **前置依赖**：第17章(感知机)—神经网络的原子单元；第1章(ERM/SRM)—损失函数基础
- **后续延伸**：第25章(CNN)、第26章(RNN+Seq2Seq)、第27章(预训练语言模型)
