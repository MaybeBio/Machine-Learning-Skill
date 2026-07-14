# Chapter 26: 循环神经网络与Seq2Seq / RNN and Sequence-to-Sequence Models

> **导航**: 本章系统阐述序列建模的演进路径——从简单RNN的隐状态递归到LSTM/GRU的门控机制，再到Seq2Seq的编码器-解码器框架和注意力机制，最后到达Transformer的自注意力架构。与第22章中RNN的简要概述互补。

---

## 一、入门理解

### 1.1 为什么需要RNN：序列的"记忆"问题

考虑一句话："我昨天去了那家餐馆，吃了那里的招牌菜，味道非常____。" 要预测最后一个词（"好"或"棒"），模型必须"记住"前面所有的上下文信息。传统的全连接网络只能处理固定长度的输入，无法自然地处理变长序列和历史依赖。

RNN的核心思想是**隐状态（Hidden State）**——一个随时间传递的"记忆向量"：

$$h_t = f(h_{t-1}, x_t)$$

每个时刻，网络根据当前输入 $x_t$ 和前一时刻的隐状态 $h_{t-1}$ 来计算新的隐状态 $h_t$。$h_t$ 承载了从 $x_1$ 到 $x_t$ 的所有历史信息的压缩表示。

这就像阅读时我们在脑海中不断更新对文章的理解——每读一个新词，不是从头开始理解，而是在之前理解的基础上纳入新信息。

### 1.2 从RNN到Seq2Seq：序列转换的通用框架

Seq2Seq（序列到序列）模型扩展了RNN的能力，使其可以处理**不定长输入到不定长输出**的映射——如机器翻译（中文 -> 英文）、文本摘要（长文 -> 短文）、语音识别（音频 -> 文字）。

核心架构分为两部分：
- **编码器（Encoder）**：将输入序列 $\{x_1, \ldots, x_T\}$ 压缩为一个固定长度的**上下文向量** $c$
- **解码器（Decoder）**：从上下文向量 $c$ 出发，逐步生成输出序列 $\{y_1, \ldots, y_{T'}\}$

这就像：先把一句话读完理解其含义（编码），然后用自己的话重新表达出来（解码）。

### 1.3 注意力机制：选择性"关注"的艺术

最初的Seq2Seq将所有输入信息压缩到单一固定长度的向量 $c$，对于长序列会形成信息瓶颈。注意力机制（Attention）允许解码器在每个时刻有选择地"关注"输入序列的不同部分：

$$\text{context}_t = \sum_{i=1}^{T} \alpha_{ti} h_i^{enc}$$

其中注意力权重 $\alpha_{ti} \geq 0, \sum_i \alpha_{ti} = 1$ 表示在解码时刻 $t$ 对编码位置 $i$ 的关注程度。这类似于人类在翻译长句时会反复回看原文的对应部分。

---

## 二、公式推导

### 2.1 简单RNN（S-RNN）的前向传播

**基本递归公式**：

$$h_t = \tanh(W_{hh} h_{t-1} + W_{xh} x_t + b_h)$$
$$y_t = W_{hy} h_t + b_y$$

其中 $W_{hh} \in \mathbb{R}^{d_h \times d_h}$ 是隐状态到隐状态的权重矩阵，$W_{xh} \in \mathbb{R}^{d_h \times d_x}$ 是输入到隐状态的权重矩阵，$W_{hy} \in \mathbb{R}^{d_y \times d_h}$ 是隐状态到输出的权重矩阵。

**展开形式**：将RNN按时间展开后，可视为一个深层（深度 = 时间长度）的前馈网络，其中每层的权重矩阵 $W_{hh}, W_{xh}$ 是共享的。

### 2.2 时间反向传播（BPTT）

BPTT实质上是将展开的计算图视为普通的前馈网络，应用标准反向传播。

**损失关于隐状态的梯度**：设每步的损失为 $L_t$，总损失 $L = \sum_t L_t$：

$$\frac{\partial L}{\partial h_t} = \frac{\partial L_t}{\partial h_t} + \frac{\partial L}{\partial h_{t+1}} \cdot \frac{\partial h_{t+1}}{\partial h_t}$$

逆向递推展开：

$$\frac{\partial L}{\partial h_t} = \frac{\partial L_t}{\partial h_t} + \sum_{k=t+1}^{T} \frac{\partial L_k}{\partial h_k} \cdot \prod_{j=t+1}^{k} \frac{\partial h_j}{\partial h_{j-1}}$$

每步的雅可比矩阵：

$$\frac{\partial h_t}{\partial h_{t-1}} = \text{diag}\left(\tanh'(z_t)\right) \cdot W_{hh}^{\mathrm{T}}$$

其中 $z_t = W_{hh} h_{t-1} + W_{xh} x_t + b_h$。

**梯度消失和爆炸分析**：

$$\frac{\partial L}{\partial h_1} \propto \prod_{t=2}^{T} \frac{\partial h_t}{\partial h_{t-1}} \approx \prod_{t=2}^{T} W_{hh}^{\mathrm{T}} \text{diag}(\tanh'(\cdot))$$

设 $W_{hh}$ 的特征值为 $\lambda_1, \ldots, \lambda_{d_h}$。若 $\max |\lambda_i| < 1$，连乘导致指数衰减（梯度消失）；若 $\max |\lambda_i| > 1$，连乘导致指数增长（梯度爆炸）。由于 $\tanh'(\cdot) \leq 1$，这个上界对忽略激活函数的分析仍然成立。

梯度爆炸可通过**梯度裁剪**解决：$\|g\| > \theta \implies g \leftarrow \theta \cdot g / \|g\|$。梯度消失需要引入门控机制。

### 2.3 LSTM的门控机制推导

LSTM通过三个门和一个细胞状态 $c_t$ 解决了梯度消失问题。

**核心组件**：

$$\begin{aligned}
f_t &= \sigma(W_f [h_{t-1}, x_t] + b_f) \quad &\text{(遗忘门)} \\
i_t &= \sigma(W_i [h_{t-1}, x_t] + b_i) \quad &\text{(输入门)} \\
o_t &= \sigma(W_o [h_{t-1}, x_t] + b_o) \quad &\text{(输出门)} \\
\tilde{c}_t &= \tanh(W_c [h_{t-1}, x_t] + b_c) \quad &\text{(候选细胞状态)}
\end{aligned}$$

**细胞状态更新**：

$$c_t = f_t \odot c_{t-1} + i_t \odot \tilde{c}_t$$

**隐状态输出**：

$$h_t = o_t \odot \tanh(c_t)$$

**梯度分析**：BPTT中细胞状态 $c_t$ 的梯度流动不经过连乘 $W_{hh}$：

$$\frac{\partial c_t}{\partial c_{t-1}} = \text{diag}(f_t) + \text{diag}(c_{t-1}) \cdot \frac{\partial f_t}{\partial c_{t-1}} + \text{diag}(\tilde{c}_t) \cdot \frac{\partial i_t}{\partial c_{t-1}} + \text{diag}(i_t) \cdot \frac{\partial \tilde{c}_t}{\partial c_{t-1}}$$

由于 $f_t$ 是sigmoid输出 $\in (0, 1)$，通过 $f_t \odot c_{t-1}$ 的前向传播，梯度可以经由 $f_t$ 稳定地跨越多个时间步（当 $f_t \approx 1$ 时完全保留）。这就是LSTM缓解梯度消失的核心机制。

### 2.4 GRU的简化门控机制

GRU将LSTM的三个门简化为两个：

$$\begin{aligned}
r_t &= \sigma(W_r [h_{t-1}, x_t] + b_r) \quad &\text{(重置门)} \\
z_t &= \sigma(W_z [h_{t-1}, x_t] + b_z) \quad &\text{(更新门)} \\
\tilde{h}_t &= \tanh(W_h [r_t \odot h_{t-1}, x_t] + b_h) \quad &\text{(候选隐状态)} \\
h_t &= (1 - z_t) \odot h_{t-1} + z_t \odot \tilde{h}_t \quad &\text{(隐状态更新)}
\end{aligned}$$

GRU没有单独的细胞状态，当更新门 $z_t = 0$ 时完全保留旧状态，$z_t = 1$ 时完全使用新状态。参数更少，训练更快，效果通常与LSTM相当。

### 2.5 注意力机制的数学形式

**Bahdanau注意力（加法注意力）**：

$$e_{ti} = v_a^{\mathrm{T}} \tanh(W_a s_{t-1} + U_a h_i)$$
$$\alpha_{ti} = \frac{\exp(e_{ti})}{\sum_{j=1}^{T} \exp(e_{tj})}$$

其中 $s_{t-1}$ 是解码器隐状态，$h_i$ 是编码器隐状态，$v_a, W_a, U_a$ 是参数。上下文向量：$c_t = \sum_{i} \alpha_{ti} h_i$。

**Luong注意力（乘法注意力）**：

$$e_{ti} = s_t^{\mathrm{T}} W_a h_i \quad \text{(general形式)}$$

或 $e_{ti} = s_t^{\mathrm{T}} h_i$（dot-product形式）。

两种形式的区别：加法注意力通过小型前馈网络计算对齐得分，乘法注意力直接使用矩阵乘法，后者计算更高效。

### 2.6 Transformer自注意力机制推导

**缩放点积注意力（Scaled Dot-Product Attention）**：

输入 $Q, K, V$ 分别表示查询（Query）、键（Key）、值（Value）：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^{\mathrm{T}}}{\sqrt{d_k}}\right) V$$

其中 $d_k$ 是键向量的维度，$\sqrt{d_k}$ 缩放因子防止点积过大导致softmax进入饱和区域（梯度接近零）。

**多头注意力（Multi-Head Attention）**：并行运行 $h$ 个独立的注意力"头"，每个头在不同的投影子空间中学习不同的注意力模式：

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$
$$\text{head}_i = \text{Attention}(Q W_i^Q, K W_i^K, V W_i^V)$$

其中 $W_i^Q \in \mathbb{R}^{d_{model} \times d_k}, W_i^K \in \mathbb{R}^{d_{model} \times d_k}, W_i^V \in \mathbb{R}^{d_{model} \times d_v}, W^O \in \mathbb{R}^{h d_v \times d_{model}}$ 是可学习的投影矩阵。

**Transformer编码器层**由自注意力+前馈网络组成，每部分配有残差连接和层归一化：

$$\begin{aligned}
Z &= \text{LayerNorm}(X + \text{MultiHead}(X, X, X)) \\
\text{Output} &= \text{LayerNorm}(Z + \text{FFN}(Z))
\end{aligned}$$

其中 $\text{FFN}(x) = W_2 \max(0, W_1 x + b_1) + b_2$。

**位置编码（Positional Encoding）**：由于自注意力没有序列顺序概念，需加入位置信息：

$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$
$$PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

---

## 三、理论深化 —— 李航《机器学习方法》第25-26章

### 3.1 李航对RNN的体系化定义（第25章）

李航在第25章中系统定义了RNN的数学框架。

**定义25.1（循环神经网络）**：RNN是处理序列数据的神经网络，其隐状态 $h_t$ 的更新同时依赖于当前输入 $x_t$ 和前一时刻的隐状态 $h_{t-1}$，形成递归计算结构。

**深度RNN**：多层堆叠的RNN，第 $l$ 层的隐状态 $h_t^{(l)}$ 同时接收同层上一时刻的 $h_{t-1}^{(l)}$ 和下层当前时刻的 $h_t^{(l-1)}$：

$$h_t^{(l)} = \tanh\left(W_{hh}^{(l)} h_{t-1}^{(l)} + W_{xh}^{(l)} h_t^{(l-1)} + b^{(l)}\right)$$

**双向RNN（Bi-directional RNN）**：由前向RNN和后向RNN组成，每个时刻的隐状态是两个方向的拼接：

$$h_t = [\overrightarrow{h_t}; \overleftarrow{h_t}]$$

双向RNN能同时利用过去和未来的上下文信息，适合序列标注等非自回归任务。

### 3.2 词向量（Word Vectors）

李航将词向量作为RNN语言建模的基础进行了阐述。

**表示向量与词嵌入**：
- **独热编码（One-hot）**：$v \in \{0, 1\}^{|V|}$，维度等于词表大小，极度稀疏
- **稠密向量（Word Embedding）**：$\mathbf{e} \in \mathbb{R}^d$（$d \ll |V|$），通过嵌入矩阵 $E \in \mathbb{R}^{d \times |V|}$ 映射 $\mathbf{e} = E \cdot \text{onehot}(w)$

**CBOW（Continuous Bag of Words）**：用上下文词的平均向量预测中心词。目标函数：

$$\mathcal{L} = -\frac{1}{T} \sum_{t=1}^{T} \log P(w_t \mid w_{t-c}, \ldots, w_{t-1}, w_{t+1}, \ldots, w_{t+c})$$

**Skip-gram**：用中心词预测上下文词：

$$\mathcal{L} = -\frac{1}{T} \sum_{t=1}^{T} \sum_{-c \leq j \leq c, j \neq 0} \log P(w_{t+j} \mid w_t)$$

概率 $P(w_O \mid w_I)$ 由softmax定义：

$$P(w_O \mid w_I) = \frac{\exp(\mathbf{v}_{w_O}^{\mathrm{T}} \mathbf{v}_{w_I})}{\sum_{w \in V} \exp(\mathbf{v}_{w}^{\mathrm{T}} \mathbf{v}_{w_I})}$$

由于分母需遍历整个词表，实际使用**负采样（Negative Sampling）**或**层次Softmax（Hierarchical Softmax）**加速训练。

### 3.3 Seq2Seq的完整形式化（第26章）

**定义26.1（序列到序列模型）**：Seq2Seq模型是一个条件概率模型 $P(Y \mid X)$，其中 $X = (x_1, \ldots, x_T)$ 是源序列，$Y = (y_1, \ldots, y_{T'})$ 是目标序列。概率通过链式法则分解：

$$P(Y \mid X) = \prod_{t=1}^{T'} P(y_t \mid y_1, \ldots, y_{t-1}, X)$$

**编码器**：将 $X$ 编码为上下文表示序列 $H^{enc} = (h_1^{enc}, \ldots, h_T^{enc})$。

**解码器**：自回归地生成 $Y$，每步概率：

$$P(y_t \mid y_{<t}, X) = \text{softmax}(W_{out} \cdot s_t + b_{out})$$

$$s_t = \text{RNN}_{dec}(s_{t-1}, y_{t-1}, c_t)$$

其中 $c_t$ 是上下文向量。

**教师强制（Teacher Forcing）**：训练时解码器的输入使用真实的目标序列 $y_{t-1}$（而非模型自身生成的 $\hat{y}_{t-1}$），加速收敛但可能导致训练-推理不一致（exposure bias）。

**束搜索（Beam Search）**：推理时保留 $k$ 个得分最高的候选序列（而非仅贪心选择），在每步扩展并剪枝：

$$\text{score}(y_1, \ldots, y_t) = \sum_{i=1}^{t} \log P(y_i \mid y_{<i}, X)$$

### 3.4 Transformer详解

李航详细阐述了Transformer的架构设计。

**整体结构**：N个编码器层 + N个解码器层。每个编码器层包含多头自注意力 + FFN，每个解码器层包含（掩码）多头自注意力 + 交叉注意力（编码器-解码器注意力） + FFN。

**自注意力（Self-Attention）**：$Q, K, V$ 都来自同一个序列（编码器中来自输入，解码器中来自目标序列的历史），使序列中每个位置直接关注所有其他位置。

**层归一化（Layer Normalization）**：对每个样本的所有特征维度做归一化，适合变长序列处理：

$$\mu = \frac{1}{d} \sum_{i=1}^{d} x_i, \quad \sigma^2 = \frac{1}{d} \sum_{i=1}^{d} (x_i - \mu)^2, \quad \hat{x}_i = \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}}, \quad y_i = \gamma \hat{x}_i + \beta$$

与批归一化不同，层归一化在训练和测试时的行为一致，不依赖于batch大小。

**Transformer的优势**：与RNN相比，Transformer通过自注意力实现了完全并行的序列处理，计算复杂性为 $O(n^2 \cdot d)$，但可以充分利用GPU并行优势。

---

## 四、综合总结

### 4.1 序列建模方法的演进对比

| 方法 | 核心机制 | 序列长度限制 | 并行性 | 长期依赖 | 典型应用 |
|------|---------|------------|--------|---------|---------|
| S-RNN | 隐状态递归 | 短序列 | 串行 | 差（梯度消失） | 基础序列建模 |
| LSTM | 门控+细胞状态 | 中等序列 | 串行 | 较好 | 语言模型, NER |
| GRU | 简化门控 | 中等序列 | 串行 | 较好 | 轻量序列任务 |
| Seq2Seq+Attention | 编码-解码+对齐 | 中长序列 | 编码并行/解码串行 | 好 | 翻译, 摘要 |
| Transformer | 自注意力 | 长序列 | 完全并行 | 好 | 翻译, 预训练 |

### 4.2 关键结论

1. **梯度消失**是RNN的根本难题，LSTM通过细胞状态和门控提供了结构性的解决方案
2. **门控机制的本质**是让网络学会"记住什么、遗忘什么"——这是LSTM/GRU超越简单RNN的关键
3. **注意力机制**将序列对齐从固定长度的上下文向量中解放出来，使长序列建模成为可能
4. **Transformer**用完全基于注意力的架构取代RNN，实现了更大规模的并行训练，是GPT/BERT等预训练语言模型的基础
5. **自注意力**使每个位置可直接关注序列中的所有位置，捕获任意距离的依赖关系，代价是 $O(n^2)$ 的计算复杂度
6. **Seq2Seq**是NLP中最为通用的框架之一，从翻译延伸到几乎所有序列转换任务

### 4.3 章节连接

- **前向连接**：RNN的反向传播（BPTT）与第5章（神经网络）的BP算法和第22章（深度学习）的计算图自动微分一脉相承
- **后向连接**：Transformer架构直接奠定了第27章（预训练语言模型）中GPT（解码器）和BERT（编码器）的基础
- **横向连接**：LSTM的门控思想与第20章（序列模型）中HMM/CRF的序列标注形成互补（神经网络 vs. 概率图模型两种范式）
