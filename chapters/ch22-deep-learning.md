# Chapter 22: 深度学习方法 / Deep Learning Methods

## 核心概念 (Core Idea)

深度学习以深层神经网络为模型，通过端到端的表示学习和随机梯度下降训练，自动从数据中提取层次化特征。从前馈网络到CNN、RNN、Transformer、预训练语言模型和GAN，深度学习方法逐步突破了传统机器学习对人工特征工程的依赖，实现了在图像、语言、生成等领域的巨大成功。

## 融合框架 (Integrated Frameworks)

（李航）第三篇系统介绍了深度学习的六大核心模型：前馈神经网络（Ch23）、卷积神经网络（Ch24）、循环神经网络（Ch25）、序列到序列模型（Ch26）、预训练语言模型（Ch27）、生成对抗网络（Ch28），并以深度学习方法总结（Ch29）概括了端到端学习、表示学习、计算图和自动微分的核心理念。

（西瓜书）第5章从单隐层前馈网络出发，介绍了BP算法和深度学习的基本概念，可作为本节前馈网络部分的前置阅读。

## 理论推导 (Theoretical Derivation)

### 22.1 前馈神经网络（简要回顾）

前馈神经网络（FNN）是深度学习的基石，已在前面的ch05中有详细讨论。此处补充李航给出的形式化描述。

**模型定义**：前馈神经网络是由参数化的非线性函数的复合构成的函数：

$$f(x; \theta) = f^{(L)} \circ f^{(L-1)} \circ \cdots \circ f^{(1)}(x)$$

其中每层 $f^{(l)}(h) = a^{(l)}(W^{(l)} h + b^{(l)})$，$a^{(l)}$ 是激活函数，$W^{(l)}, b^{(l)}$ 是参数。

**通用近似定理**：具有至少一个隐层、使用挤压型激活函数的前馈网络，可以任意精度逼近 $\mathbb{R}^n$ 的紧子集上的任意连续函数。但定理不保证学习算法能找到这些参数。

**反向传播（Backpropagation）**：利用链式法则高效计算所有参数的梯度。设损失为 $L$，则对于第 $l$ 层参数：

$$\frac{\partial L}{\partial W^{(l)}} = \frac{\partial L}{\partial h^{(l)}} \cdot \frac{\partial h^{(l)}}{\partial W^{(l)}}, \quad \frac{\partial L}{\partial h^{(l-1)}} = (W^{(l)})^{\mathrm{T}} \frac{\partial L}{\partial h^{(l)}} \odot a'(z^{(l-1)})$$

在计算图上，正向传播沿有向边依次计算结点函数值，反向传播逆着有向边依次计算梯度——两者都是张量的流动。

**正则化**：
- **早停法（Early Stopping）**：在验证误差不再下降时停止训练
- **暂退法（Dropout）**：训练时以概率 $p$ 随机丢弃神经元，预测时乘以 $p$ 进行权重缩放

### 22.2 卷积神经网络（CNN）

**动机**：全连接网络处理图像时参数量爆炸（如 $256 \times 256 \times 3$ 的图像输入需要每个神经元连接约20万个权重），且忽略了图像的局部相关性。

**卷积层**：使用可学习的卷积核（滤波器）在输入上滑动计算：

$$S(i, j) = (I * K)(i, j) = \sum_m \sum_n I(i+m, j+n) K(m, n)$$

核心特性：
- **稀疏连接（Sparse Connectivity）**：每个输出单元仅与输入的局部区域（感受野）连接
- **参数共享（Parameter Sharing）**：同一个卷积核在整张图的所有位置共享权重
- **平移等变性（Translation Equivariance）**：输入平移导致输出同样平移

**汇聚层（Pooling）**：对局部区域做统计汇总，如最大汇聚 $y = \max\{x_1, x_2, \ldots, x_k\}$。作用是降采样、增加平移不变性、扩大感受野。

**经典架构**：
- **LeNet-5 (1998)**：卷积+汇聚交替+全连接分类，手写数字识别
- **AlexNet (2012)**：ReLU激活、Dropout正则化、GPU并行，ImageNet冠军
- **VGG (2014)**：使用 $3 \times 3$ 小卷积核堆叠代替大卷积核，深层统一架构
- **ResNet (2015)**：引入残差连接 $h_{l+1} = h_l + f_l(h_l)$，解决深层网络的退化问题，允许训练152层甚至更深的网络
- **Inception/GoogLeNet**：同一层使用多种尺度的卷积核并行处理

**感受野**：网络中某层特征图上的一个像素对应原始输入的区域大小。随着层数加深，感受野逐步增大，高层特征捕获更大范围的语义信息。

### 22.3 循环神经网络（RNN）

**动机**：序列数据（文本、语音、时间序列）中，当前输出依赖于历史上下文。前馈网络只能处理固定大小的输入，无法建模时序依赖。

**简单RNN**：

$$h_t = \tanh(W_{hh} h_{t-1} + W_{xh} x_t + b_h)$$
$$y_t = W_{hy} h_t + b_y$$

其中 $h_t$ 是隐状态，承载了从 $x_1$ 到 $x_t$ 的历史信息。

**时间反向传播（BPTT）**：展开RNN的计算图，在时间轴上应用反向传播。梯度需经过矩阵 $W_{hh}$ 的连乘：

$$\frac{\partial L}{\partial h_1} \propto \prod_{t=2}^{T} \frac{\partial h_t}{\partial h_{t-1}} \approx \prod_{t=2}^{T} W_{hh}^{\mathrm{T}} \mathrm{diag}(\tanh'(z_t))$$

当 $W_{hh}$ 的特征值 $\lambda < 1$ 时梯度消失，$\lambda > 1$ 时梯度爆炸。这使得简单RNN难以学习长距离依赖。

**LSTM（长短期记忆网络）**：引入门控机制和细胞状态 $c_t$ 解决梯度消失问题。

遗忘门：$f_t = \sigma(W_f [h_{t-1}, x_t] + b_f)$
输入门：$i_t = \sigma(W_i [h_{t-1}, x_t] + b_i)$
候选状态：$\tilde{c}_t = \tanh(W_c [h_{t-1}, x_t] + b_c)$
细胞状态更新：$c_t = f_t \odot c_{t-1} + i_t \odot \tilde{c}_t$
输出门：$o_t = \sigma(W_o [h_{t-1}, x_t] + b_o)$
隐状态：$h_t = o_t \odot \tanh(c_t)$

遗忘门和输入门的S型函数输出 $(0,1)$ 之间的值，控制信息流动。细胞状态的更新采用加法形式，使得梯度在时间上的反向传播不会经过连续的矩阵乘法，从而缓解梯度消失。

**GRU（门控循环单元）**：LSTM的精简版，合并遗忘门和输入门为更新门 $z_t$，合并细胞状态和隐状态。

重置门：$r_t = \sigma(W_r [h_{t-1}, x_t] + b_r)$
更新门：$z_t = \sigma(W_z [h_{t-1}, x_t] + b_z)$
候选状态：$\tilde{h}_t = \tanh(W_h [r_t \odot h_{t-1}, x_t] + b_h)$
隐状态更新：$h_t = (1 - z_t) \odot h_{t-1} + z_t \odot \tilde{h}_t$

**双向RNN**：同时使用正向和反向两个RNN处理序列，拼接两者的隐状态。适用于需要完整上下文的任务（如序列标注、机器翻译）。

**词向量（Word Embedding）**：将离散的词索引映射为稠密的实数向量 $e(w) \in \mathbb{R}^d$，语义相似的词向量在空间中相近。词向量层是RNN语言模型的第一层，将词映射为分布式表示。

### 22.4 序列到序列模型（Seq2Seq）

**编码器-解码器架构**：

编码器：将输入序列 $x = (x_1, \ldots, x_T)$ 编码为上下文向量 $c$：
$$h_t = \mathrm{RNN}(h_{t-1}, x_t), \quad c = h_T$$

解码器：基于上下文向量 $c$ 和已生成的部分序列生成输出：
$$s_t = \mathrm{RNN}(s_{t-1}, y_{t-1}, c), \quad P(y_t | y_{<t}, c) = \mathrm{softmax}(W s_t + b)$$

**注意力机制（Attention）**：解决编码器将所有信息压缩到单一向量 $c$ 的信息瓶颈问题。在解码的每一步 $t$，动态计算编码器所有隐状态 $h_i$ 的加权和作为上下文向量：

$$\alpha_{ti} = \frac{\exp(\mathrm{score}(s_{t-1}, h_i))}{\sum_j \exp(\mathrm{score}(s_{t-1}, h_j))}, \quad c_t = \sum_i \alpha_{ti} h_i$$

常见的打分函数：点积 $\mathrm{score}(s, h) = s^{\mathrm{T}} h$，加性 $\mathrm{score}(s, h) = v^{\mathrm{T}} \tanh(W[s; h])$。

**束搜索（Beam Search）**：在解码时维护 $B$ 个最优的已生成序列（束宽），每一步扩展所有可能的输出词并保留分数最高的 $B$ 个序列。

**Transformer模型**（李航第26.3节）：
Transformer完全基于自注意力机制，摒弃了循环结构。

核心组件：
- **多头自注意力**：$\mathrm{Attention}(Q, K, V) = \mathrm{softmax}\left(\frac{QK^{\mathrm{T}}}{\sqrt{d_k}}\right)V$
  
  多头的含义是将 $Q, K, V$ 分成 $h$ 组独立计算注意力，然后拼接：
  
  $$\mathrm{MultiHead}(Q, K, V) = \mathrm{Concat}(\mathrm{head}_1, \ldots, \mathrm{head}_h) W^O$$
  
  其中 $\mathrm{head}_i = \mathrm{Attention}(QW_i^Q, KW_i^K, VW_i^V)$

- **位置编码（Positional Encoding）**：由于自注意力不捕捉位置信息，使用正弦函数编码位置：
  
  $$PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{\text{model}}}), \quad PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d_{\text{model}}})$$

- **残差连接与层归一化**：每层输出 $\mathrm{LayerNorm}(x + \mathrm{Sublayer}(x))$

- **编码器-解码器注意力**：解码器对编码器输出做交叉注意力

### 22.5 预训练语言模型（Pre-trained LMs）

**核心范式**：大规模无标注语料上预训练（pre-training）+ 下游任务标注数据上微调（fine-tuning）。

**发展脉络**：
- **word2vec/GloVe**：静态词向量，每个词获得固定的向量表示
- **ELMo (2018)**：双向LSTM语言模型，上下文相关的词表示（contextualized）
- **GPT (2018)**：单向Transformer解码器自回归语言模型，$P(x) = \prod_t P(x_t | x_{<t})$
- **BERT (2019)**：双向Transformer编码器，掩码语言模型（MLM）预训练

**BERT的核心机制**：

预训练任务：
1. **掩码语言模型（Masked Language Model, MLM）**：随机遮盖15%的输入词，预测被遮盖的词。其中80%用[MASK]替换，10%替换为随机词，10%保持不变。
2. **下一句预测（Next Sentence Prediction, NSP）**：判断句子B是否是句子A的下一句（后续研究表明NSP并非必要）。

微调阶段：在BERT编码器之上添加简单的任务特定层（如分类层、序列标注层），用少量标注数据微调所有参数。

**GPT的生成范式**：自回归地逐个生成词，$P(y_t | y_{<t}, x)$。经过prompt engineering和in-context learning的发展，GPT系列模型可在不更新参数的情况下通过提示词完成多种下游任务。

### 22.6 生成对抗网络（GAN）

**核心思想**：生成器 $G(z; \theta)$ 和判别器 $D(x; \varphi)$ 进行二人零和博弈。

**Minimax目标函数**：

$$\min_G \max_D \; \mathbb{E}_{x \sim P_{\text{data}}}[\log D(x)] + \mathbb{E}_{z \sim P_{\text{seed}}}[\log(1 - D(G(z)))]$$

- 判别器 $D$ 试图最大化区分真实数据（输出接近1）和生成数据（输出接近0）
- 生成器 $G$ 试图最小化被识别为假数据的概率（使 $D(G(z))$ 接近1）

**理论分析**（李航第28.1.3节）：
- 固定生成器 $G$ 时，最优判别器为：$D_G^*(x) = \frac{P_{\text{data}}(x)}{P_{\text{data}}(x) + P_{\text{gen}}(x)}$
- 固定最优判别器时，最小化问题等价于最小化JS散度：$L(G, D_G^*) = 2 \cdot \mathrm{JS}(P_{\text{data}} \| P_{\text{gen}}) - 2\log 2$
- 理论最优解（纳什均衡）：$P_{\text{gen}}^*(x) = P_{\text{data}}(x)$，$D^*(x) = \frac{1}{2}$

**训练不稳定性**：GAN的训练是minimax优化而非标准的min优化，容易出现模式崩塌（mode collapse）和训练振荡。

**DCGAN**：将CNN引入GAN，特点包括：
- 生成器使用转置卷积（上采样），判别器使用常规卷积（下采样）
- 不使用汇聚层和全连接隐层
- 生成器使用ReLU（除输出层用tanh），判别器使用LeakyReLU
- 使用批量归一化

**WGAN**：使用Wasserstein距离替代JS散度，缓解训练不稳定问题。

**转置卷积**：对应于矩阵表示中卷积矩阵的转置 $C^{\mathrm{T}}$。若卷积核为 $W$，则转置卷积核为 $\mathrm{rot180}(W)$。原始卷积下采样，转置卷积上采样。输入矩阵相邻元素之间插0实现微步卷积效果。

### 22.7 深度学习方法总结

**端到端学习（End-to-End Learning）**：与传统机器学习分阶段（特征工程 + 模型学习）不同，深度学习将特征提取和任务预测统一在一个可导的神经网络中，联合优化所有参数。

**表示学习（Representation Learning）**：神经网络的隐层自动学习数据的分布式表示（distributed representation）——每个概念由多个神经元的组合表示，每个神经元参与表示多个概念。这与符号AI的"局部表示"（一个概念一个符号）形成鲜明对比。

**计算图与自动微分**：深度学习框架（TensorFlow, PyTorch）将网络表示为有向无环计算图。正向传播计算输出，反向传播自动计算所有参数梯度——这就是autograd的核心。

**深度学习的三个优势**（李航第29.4节）：
1. **强大的函数近似能力**：通用近似定理保证浅网络就能以任意精度近似任意连续函数，但深层网络拥有更精简的表达能力
2. **更好的样本效率**：深窄网络比浅宽网络需要更少的参数和样本
3. **强泛化能力**：过参数化网络配合SGD训练，常在不加正则化时也不严重过拟合

**深度学习的不足**：
- **缺乏稳健性（Robustness）**：微小的对抗性扰动可导致预测错误
- **恰当性问题**：模型可能学到不合理的关联（如有轮胎就是汽车）
- **可解释性差**：在某些领域（如金融、医疗）不可接受

**优化算法对比**：

| 算法 | 核心思想 | 更新公式 |
|------|---------|---------|
| SGD | 沿负梯度方向 | $\theta_t = \theta_{t-1} - \eta g_t$ |
| 动量（Momentum） | 指数移动平均梯度 | $v_t = \beta_1 v_{t-1} + (1-\beta_1)g_t$; $\theta_t = \theta_{t-1} - \eta v_t$ |
| RMSProp | 自适应学习率 | $s_t = \beta_2 s_{t-1} + (1-\beta_2)g_t^2$; $\theta_t = \theta_{t-1} - \eta \frac{g_t}{\sqrt{s_t + \varepsilon}}$ |
| Adam | 动量+RMSProp | $v_t, s_t$ 结合; $\theta_t = \theta_{t-1} - \eta \frac{\hat{v}_t}{\sqrt{\hat{s}_t + \varepsilon}}$ |

Adam默认超参数：$\beta_1 = 0.9, \beta_2 = 0.999, \varepsilon = 10^{-8}, \eta = 10^{-3}$（李航第29.3节）。

**梯度消失/爆炸**：深层网络反向传播时梯度经过多层矩阵连乘，若连乘矩阵的特征值偏离1则梯度趋于0（消失）或无穷（爆炸）。解决策略：ReLU激活、LSTM门控、残差连接、批量归一化。

**内部协变量偏移（Internal Covariate Shift）**：训练过程中网络各层输入分布不断变化，导致深层学习困难。批量归一化（Batch Normalization）和层归一化（Layer Normalization）通过对每层输入做归一化来缓解此问题。

## 核心概念 (Key Concepts)

- **卷积**：局部连接 + 权值共享 + 平移等变性的线性运算
- **感受野**：CNN中某层输出单元在原始输入上的对应区域大小
- **隐状态**：RNN中编码历史信息的向量，在时间步之间传递
- **门控机制**：用S型函数控制的逐元素乘法，调节信息流动
- **注意力**：对编码器输出的动态加权求和，使解码器能"看到"输入的任意位置
- **自注意力**：序列内部元素之间的注意力，$Q, K, V$ 来自同一个序列
- **预训练-微调范式**：大规模语料无监督预训练 + 下游任务有监督微调
- **对抗训练**：生成器与判别器的交替优化，最终达到纳什均衡
- **分布式表示**：信息分布在大批神经元的活动模式中，而非单个神经元

## 算法步骤 (Algorithm Steps)

### CNN图像分类
1. 输入图像归一化（mean/std 标准化）
2. 交替堆叠卷积层（ReLU）和汇聚层
3. 展平特征图，送入全连接分类层（softmax输出）
4. 反向传播训练，使用 SGD/Adam 优化交叉熵损失

### LSTM序列建模
1. 初始化 $h_0 = 0, c_0 = 0$
2. 对每个时间步 $t$：计算 $f_t, i_t, \tilde{c}_t, o_t$，更新 $c_t, h_t$
3. 基于 $h_t$ 做分类/预测
4. BPTT 训练，使用截断反向传播处理长序列

### Transformer训练流程
1. 输入序列嵌入 + 位置编码
2. 编码器：$L$ 层（多头自注意力 + FFN + 残差 + LN）
3. 解码器：$L$ 层（掩码自注意力 + 交叉注意力 + FFN + 残差 + LN）
4. 线性投影 + softmax 输出概率分布
5. 交叉熵损失优化

### GAN训练（算法28.1）
1. 从训练数据采样M个真实样本 $\{x^{(m)}\}$
2. 从噪声分布采样M个样本 $\{z^{(m)}\}$
3. 用梯度上升更新判别器参数 $\varphi$（重复S次）
4. 采样M个噪声样本，用梯度上升更新生成器参数 $\theta$（1次）
5. 重复直到收敛

## 直观理解 (Intuition)

**CNN如同视觉皮层**：Hubel和Wiesel发现猫的视觉皮层中存在"简单细胞"和"复杂细胞"，分别对局部边缘和位置不变的特征敏感。CNN的卷积层模拟简单细胞（局部特征检测），汇聚层模拟复杂细胞（位置不变性）。

**LSTM如同高速公路上的匝道**：细胞状态 $c_t$ 是一条贯穿时间的"信息高速公路"，遗忘门控制哪些信息从主干道移除，输入门控制哪些信息从匝道汇入。这种加法式的信息传递使得梯度可以在时间轴上无衰减地流动。

**Transformer的注意力如同数据库查询**：将 Query（解码器当前位置的表示）与所有 Key（编码器位置的表示）做匹配，得到每个 Key 对应的 Value 的重要性权重，最终输出是 Values 的加权和。这是一种"软寻址"——不是精确匹配一个位置，而是所有位置都参与，但权重不同。

**GAN如同警匪博弈**：生成器是"伪造者"，试图制造以假乱真的赝品；判别器是"鉴定师"，试图分辨真品与赝品。两者在对抗中共同进步，最终伪造者的技术达到鉴定师无法分辨的程度。

## 常见误区 (Anti-patterns)

- **误区：CNN只适用于图像**。一维卷积同样适用于文本和时序数据（如字符级CNN、WaveNet）。
- **误区：RNN一定比CNN更适合序列**。对某些序列任务，带因果卷积的CNN在训练速度和长程依赖上可能优于RNN。
- **误区：注意力机制替代了RNN**。注意力最初是作为RNN seq2seq的增强组件提出的，Transformer后来证明纯注意力也可以工作得很好。
- **误区：预训练模型越大越好**。大模型需要更多计算资源和推理时间，小模型（如DistilBERT）在特定场景下更实用。
- **误区：GAN的目标是最小化损失**。GAN是minimax问题而非min问题，损失值不能简单作为训练质量的指标。Inception Score和FID是更可靠的评估指标。
- **误区：深度学习不需要特征工程**。虽然深度学习自动学习特征，但数据预处理、归一化、增强仍然至关重要。

## 代码要点 (Code Essentials)

```python
# CNN (PyTorch)
class CNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(64, 128, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
        )
        self.fc = nn.Linear(128 * 8 * 8, 10)
    
    def forward(self, x):
        x = self.conv(x)
        return self.fc(x.view(x.size(0), -1))

# LSTM (PyTorch)
lstm = nn.LSTM(input_size=256, hidden_size=512, num_layers=2, batch_first=True)
output, (hn, cn) = lstm(x)

# Transformer (PyTorch)
transformer = nn.Transformer(d_model=512, nhead=8, num_encoder_layers=6,
                              num_decoder_layers=6, dim_feedforward=2048)

# GAN训练循环
for epoch in range(epochs):
    # 训练判别器
    real_out = D(real_data)
    fake_out = D(G(noise).detach())
    d_loss = -(torch.log(real_out) + torch.log(1 - fake_out)).mean()
    d_loss.backward()
    # 训练生成器
    g_loss = -torch.log(D(G(noise))).mean()
    g_loss.backward()
```

## 实例分析 (Worked Example)

**ResNet残差学习**：假设目标是学习映射 $H(x)$，ResNet改为学习残差 $F(x) = H(x) - x$，则 $H(x) = F(x) + x$。如果恒等映射是最优的（即深层不需要比浅层做更多），则 $F(x) = 0$ 比 $H(x) = x$ 更容易学习。这就是残差连接解决深层退化问题的直观解释——它让网络更容易学习"不做任何改变"的恒等映射。

**BERT的MLM示例**：输入"我[MASK]吃[MASK]果"，BERT需要预测[MASK]位置为"想"和"苹"。这要求模型同时利用左右双向上下文——"想"由后面的"吃"暗示（想吃），"苹"由前面的"吃"暗示（吃苹果）。单向语言模型（如GPT）无法利用右侧上下文。

**GAN的模式崩塌问题**：假设真实数据包含10种数字（0-9），但生成器学会只生成"1"来欺骗判别器。因为判别器独立判断每个样本，它发现"1"可以轻易通过后就放松了对其他数字的要求，而生成器"藏"在一种模式中最安全。WGAN通过使用Wasserstein距离提供了更平滑的梯度信号来缓解此问题。

## 关键结论 (Key Takeaways)

1. CNN的核心贡献是归纳偏置（inductive bias）：局部性、平移等变性——这些先验使得CNN能用远少于全连接网络的参数有效学习视觉特征
2. LSTM通过门控机制和细胞状态的加法更新，从根本上解决了简单RNN的梯度消失问题，使学习数百步的长距离依赖成为可能
3. 注意力机制实现了"软寻址"——模型可以在输入序列的任意位置动态提取信息，突破了固定长度上下文向量的瓶颈
4. Transformer的自注意力机制彻底抛弃了循环结构，使得计算高度可并行化，成为当前大语言模型的基础架构
5. 预训练-微调范式的核心洞察：语言结构的统计知识蕴含在大规模语料中，可从无标注数据中学习，然后适配到具体任务
6. GAN的minimax目标在理论上收敛到JS散度最小化，但实践中需要大量技巧（DCGAN架构、WGAN损失、谱归一化等）来稳定训练

## 章节连接 (Connects To)

- 前置: ch05-neural-networks.md（前馈网络基础与BP算法） | ch16-svm-kernel.md（核方法与表示学习的关系）
- 后续: （本系列后续章节可涵盖强化学习、图神经网络等高级主题）
