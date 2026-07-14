# Chapter 25: 卷积神经网络 / Convolutional Neural Networks

> **导航**: 本章深入卷积神经网络的体系结构、数学原理和经典架构，从卷积运算的基本定义到ResNet的残差学习，系统覆盖CNN从基础到前沿的核心知识。与第22章中CNN的简要概述互补，本章提供李航《机器学习方法》中的完整理论体系。

---

## 一、入门理解

### 1.1 为什么需要卷积神经网络

假设我们要识别一张 $256 \times 256 \times 3$ 的彩色图像中的物体。如果使用全连接网络，第一个隐层的每个神经元需要 $256 \times 256 \times 3 \approx 200,000$ 个权重。这不仅导致参数爆炸，还完全忽略了图像的**空间结构**——相邻像素之间有强相关性，而远距离像素几乎无关。

CNN通过两个关键设计解决了这些问题：
- **局部连接（稀疏连接）**：每个神经元只连接输入的一个局部区域（感受野），而非全部像素
- **参数共享（权重共享）**：同一个卷积核在整个图像上共享权重，无论特征出现在图像的哪个位置

这就像人类视觉系统：我们识别一只猫时，不需要把图像的所有像素同时考虑，而是通过局部特征（边缘、纹理、形状）逐层抽象，最终形成"这是一只猫"的高层语义判断。

### 1.2 从卷积运算到特征提取

**一维卷积的直觉**：给定信号 $x(t)$ 和滤波器 $w(t)$，卷积 $(x * w)(t) = \int x(\tau) w(t - \tau) d\tau$ 度量了信号与翻转平移后的滤波器之间的重叠程度。在图像处理中，离散二维卷积：

$$S(i, j) = (I * K)(i, j) = \sum_m \sum_n I(i-m, j-n) K(m, n)$$

由于深度学习使用的是**互相关（cross-correlation）**而非严格意义上的卷积（未翻转滤波器）：

$$S(i, j) = (I \star K)(i, j) = \sum_m \sum_n I(i+m, j+n) K(m, n)$$

但习惯上仍称为"卷积"，因为：翻转与否不影响网络的学习能力（网络会自动学习合适的方向），且实现的都是局部加权求和。

### 1.3 CNN的层次化特征学习

CNN通过堆叠多个卷积层实现从局部到全局、从具体到抽象的层次化特征提取：
- **浅层（Layer 1-2）**：检测边缘、角点、颜色blob等低级特征
- **中层（Layer 3-5）**：组合低级特征形成纹理、形状等中级特征
- **深层（Layer 6+）**：形成对象部件、完整物体等高级语义特征

这种层次化结构与生物视觉皮层的层级处理方式高度一致（Hubel & Wiesel的发现），是CNN成功的关键原因之一。

---

## 二、公式推导

### 2.1 二维卷积的数学定义

**二维卷积（互相关）**：输入特征图 $X \in \mathbb{R}^{H \times W}$，卷积核 $K \in \mathbb{R}^{k_h \times k_w}$：

$$Y_{i,j} = \sum_{m=0}^{k_h-1} \sum_{n=0}^{k_w-1} X_{i+m, j+n} \cdot K_{m,n}$$

**三维卷积**（多通道输入）：输入 $X \in \mathbb{R}^{H \times W \times C_{in}}$，对每个输出通道 $c_{out}$ 有一个三维卷积核 $K_{c_{out}} \in \mathbb{R}^{k_h \times k_w \times C_{in}}$：

$$Y_{i,j,c_{out}} = \sum_{m=0}^{k_h-1} \sum_{n=0}^{k_w-1} \sum_{c=0}^{C_{in}-1} X_{i+m, j+n, c} \cdot K_{m,n,c,c_{out}} + b_{c_{out}}$$

### 2.2 带填充和步长的卷积输出尺寸

**补零（Zero Padding）**：在输入周围补 $p$ 圈零，以控制输出尺寸：
- 无填充（valid）：输出尺寸缩小，$H_{out} = H - k + 1$
- 相同填充（same）：输出尺寸不变，$p = \lfloor k/2 \rfloor$，$H_{out} = H$

**步长（Stride）** $s$：卷积核每次滑动的像素数：

$$H_{out} = \left\lfloor \frac{H + 2p - k_h}{s} \right\rfloor + 1$$
$$W_{out} = \left\lfloor \frac{W + 2p - k_w}{s} \right\rfloor + 1$$

### 2.3 汇聚层（Pooling）的数学表示

汇聚层对局部区域做统计汇总，降采样特征图的同时增强平移不变性。

**最大汇聚（Max Pooling）**：$y = \max\{x_1, x_2, \ldots, x_{m}\}$

对于输入区域 $\mathcal{R}$，输出：$y = \max_{i \in \mathcal{R}} x_i$。

最大汇聚的梯度仅传导向最大值位置：
$$\frac{\partial L}{\partial x_i} = \begin{cases} \frac{\partial L}{\partial y} & \text{如果 } x_i = \max(\mathcal{R}) \\ 0 & \text{否则} \end{cases}$$

**平均汇聚（Average Pooling）**：$y = \frac{1}{|\mathcal{R}|} \sum_{i \in \mathcal{R}} x_i$

平均汇聚的梯度均匀分配到区域内所有元素：
$$\frac{\partial L}{\partial x_i} = \frac{1}{|\mathcal{R}|} \frac{\partial L}{\partial y}$$

### 2.4 卷积层的反向传播

设卷积输出 $Y = X * K$，损失关于输出的梯度 $\frac{\partial L}{\partial Y}$ 已知。

**对卷积核的梯度**：

$$\frac{\partial L}{\partial K_{m,n}} = \sum_{i} \sum_{j} \frac{\partial L}{\partial Y_{i,j}} \cdot X_{i+m, j+n}$$

这是 $\frac{\partial L}{\partial Y}$ 与 $X$ 的卷积（互相关）。

**对输入的梯度**：将卷积核翻转180度，与 $\frac{\partial L}{\partial Y}$ 做完全卷积（full convolution）：

$$\frac{\partial L}{\partial X_{i,j}} = \sum_{m} \sum_{n} \frac{\partial L}{\partial Y_{i-m, j-n}} \cdot K_{m,n}$$

（仅在 $i-m, j-n$ 在有效范围内时求和，可视作补零后的卷积）

### 2.5 感受野计算

第 $l$ 层特征图上一个像素对应的输入图像上的区域大小即为感受野。递推公式：

$$r_l = r_{l-1} + (k_l - 1) \prod_{i=1}^{l-1} s_i$$

其中 $r_l$ 是第 $l$ 层的感受野大小，$k_l$ 是第 $l$ 层的卷积核大小，$s_i$ 是第 $i$ 层的步长。

---

## 三、理论深化 —— 李航《机器学习方法》第24章 + PPT

### 3.1 李航对CNN的体系化定义

李航在第24章（第二篇）给出了CNN的完整形式化描述：

**定义24.1（卷积层）**：卷积层是由多个卷积核组成的映射。设输入为 $X \in \mathbb{R}^{H_{in} \times W_{in} \times C_{in}}$，卷积层的输出 $Y \in \mathbb{R}^{H_{out} \times W_{out} \times C_{out}}$ 由 $C_{out}$ 个卷积核 $\{K_c \in \mathbb{R}^{k_h \times k_w \times C_{in}}\}_{c=1}^{C_{out}}$ 和偏置 $\{b_c\}$ 决定。每个卷积核以滑动窗口方式与输入做内积。

**定义24.2（池化层）**：池化层是一个下采样操作，对输入特征图的每个通道独立地在 $p_h \times p_w$ 的窗口内取最大值或平均值，步长通常等于窗口大小。

### 3.2 CNN的三大结构特性

李航系统阐述了CNN的三个核心结构特性：

**(1) 稀疏连接（Sparse Connectivity）**
- 每个输出单元只连接到输入的局部区域（大小为 $k_h \times k_w \times C_{in}$ 的窗口）
- 参数量从全连接的 $O(H_{in} W_{in} C_{in} \times H_{out} W_{out} C_{out})$ 降至 $O(k_h k_w C_{in} C_{out})$
- 计算量从 $O(H^2 W^2 C_{in} C_{out})$ 降至 $O(H_{out} W_{out} k_h k_w C_{in} C_{out})$

**(2) 参数共享（Parameter Sharing）**
- 同一卷积核在整个特征图的所有空间位置共享参数
- 直接导致了**平移等变性**：若输入平移 $\Delta$，输出同样平移 $\Delta$
- 平移等变性 = $f(T_\Delta x) = T_\Delta f(x)$，其中 $T_\Delta$ 是平移操作

**(3) 多尺度表示**
- 通过池化层降采样扩大感受野，形成金字塔式的多尺度表示
- 池化引入了局部的**平移不变性**（并非严格等变，而是对小位移鲁棒）

### 3.3 经典CNN架构的深入分析

李航详细分析了从LeNet到ResNet的经典架构演进：

**LeNet-5 (LeCun et al., 1998)**
- 结构：Conv(6@5x5) -> Pool(2x2) -> Conv(16@5x5) -> Pool(2x2) -> FC(120) -> FC(84) -> FC(10)
- 使用Sigmoid激活和平均池化
- 确立了卷积层+池化层交替、最后全连接分类的经典模式

**AlexNet (Krizhevsky et al., 2012)**
- 关键创新：
  - ReLU激活函数：$f(x) = \max(0, x)$，解决Sigmoid的梯度饱和问题
  - Dropout正则化：训练时以概率 $p$ 随机丢弃神经元，测试时所有神经元乘以 $p$
  - 数据增强：随机裁剪、水平翻转、PCA颜色扰动
  - GPU并行：双GPU训练，各负责一半的卷积核
- 结构：5个卷积层 + 3个全连接层，总参数量约6000万

**VGG (Simonyan & Zisserman, 2014)**
- 核心思想：用 $3 \times 3$ 小卷积核堆叠替代大卷积核
  - 两个 $3 \times 3$ 卷积核的感受野 = 一个 $5 \times 5$ 卷积核
  - 三个 $3 \times 3$ 卷积核的感受野 = 一个 $7 \times 7$ 卷积核
  - 优势：更少的参数，更多的非线性（每层有ReLU），更强的表达能力
- 结构极为统一：反复使用 $3 \times 3$ 卷积 + $2 \times 2$ 最大池化

**GoogLeNet / Inception (Szegedy et al., 2014)**
- **Inception模块**：同一层并行使用 $1 \times 1$、$3 \times 3$、$5 \times 5$ 卷积和池化，让网络自动选择合适尺度
- $1 \times 1$ 卷积用于降维（减少计算量）和增加非线性
- 用全局平均池化代替全连接层，大幅减少参数量
- 增加了辅助分类器以缓解梯度消失

**ResNet (He et al., 2015)**
- 核心创新：**残差学习（Residual Learning）**

$$h_{l+1} = h_l + \mathcal{F}(h_l, W_l)$$

其中 $\mathcal{F}$ 是残差函数（通常为Conv-BN-ReLU-Conv-BN的堆叠），$h_l$ 通过跳跃连接直接加到 $\mathcal{F}$ 的输出上。

- **为什么残差连接有效**：在极深网络中，恒等映射 $h_{l+1} = h_l$ 难以学习。残差连接使网络只需学习 $\mathcal{F}(h_l) = h_{l+1} - h_l \approx 0$ 即可实现近似的恒等映射。如果 $\mathcal{F}$ 的所有参数趋于0（因为权重衰减），网络就退化为恒等映射，因而添加残差块至少不会损害网络性能。

- **梯度流动**：反传时梯度通过跳跃连接直接流向浅层，避免了连乘导致的梯度消失：
  $$\frac{\partial L}{\partial h_l} = \frac{\partial L}{\partial h_{l+1}} \cdot \left(1 + \frac{\partial \mathcal{F}}{\partial h_l}\right)$$

- ResNet使得训练152层甚至1001层的网络成为可能，在ImageNet上实现了3.57%的Top-5错误率（超越人类水平）。

### 3.4 批量归一化（Batch Normalization）

BN是ResNet和大多数现代CNN的标准组件。对每个mini-batch $\mathcal{B} = \{x_1, \ldots, x_m\}$ 和每个通道：

$$\mu_{\mathcal{B}} = \frac{1}{m} \sum_{i=1}^{m} x_i, \quad \sigma_{\mathcal{B}}^2 = \frac{1}{m} \sum_{i=1}^{m} (x_i - \mu_{\mathcal{B}})^2$$

$$\hat{x}_i = \frac{x_i - \mu_{\mathcal{B}}}{\sqrt{\sigma_{\mathcal{B}}^2 + \epsilon}}, \quad y_i = \gamma \hat{x}_i + \beta$$

其中 $\gamma$（缩放）和 $\beta$（偏移）是可学习参数，$\epsilon$ 是防止除零的小常数。

BN的作用：
- 减小内部协变量偏移（Internal Covariate Shift）
- 允许使用更大的学习率，加速收敛
- 起到正则化效果，减轻对Dropout的需求
- 降低对初始化的敏感性

### 3.5 数据增强与迁移学习

**数据增强**：从有限训练数据中生成更多训练样本的方式。

常见方法：
- 随机裁剪、水平翻转、旋转、缩放
- 色彩抖动（Color Jittering）：随机调整亮度、对比度、饱和度
- Cutout：随机遮挡图像的矩形区域
- Mixup：线性插值两个样本及其标签

**迁移学习（Transfer Learning）**：将在大型数据集（如ImageNet）上预训练的CNN作为特征提取器或微调起点：

- **特征提取器模式**：冻结预训练卷积层，只训练新添加的分类器
- **微调模式**：使用预训练权重初始化，以较小的学习率在整个目标数据集上继续训练

这是CNN在数据量有限的场景中最实用的策略。

---

## 四、综合总结

### 4.1 经典CNN架构对比

| 架构 | 年份 | 深度 | 核心创新 | Top-5错误率 |
|------|------|------|---------|------------|
| LeNet-5 | 1998 | 7层 | CNN范式确立 | - |
| AlexNet | 2012 | 8层 | ReLU, Dropout, GPU | 15.3% |
| VGG-16 | 2014 | 16层 | $3 \times 3$ 小卷积核堆叠 | 7.3% |
| GoogLeNet | 2014 | 22层 | Inception模块, $1 \times 1$ 卷积 | 6.7% |
| ResNet-152 | 2015 | 152层 | 残差连接, 批归一化 | 3.57% |

### 4.2 关键结论

1. **CNN的三大结构特性**——稀疏连接、参数共享、多尺度表示——使其成为处理网格状数据（图像、视频）的自然选择
2. **$1 \times 1$ 卷积**是CNN中极为重要的工具，用于跨通道信息融合和维度变换
3. **残差连接**解决了深层网络的退化问题，使"越深越好"成为现实
4. **批归一化**是现代CNN训练的标准组件，稳定了训练过程
5. **迁移学习**使CNN在数据有限的领域也能取得优异性能，是实用部署的关键策略
6. **感受野的逐层扩大**使CNN自然地形成层次化特征金字塔，浅层检测局部细节，深层捕获全局语义

### 4.3 章节连接

- **前向连接**：CNN的前馈计算和反向传播与第5章（神经网络）和第22章（深度学习）中的BP算法一脉相承
- **后向连接**：CNN的卷积操作为第26章（RNN和Seq2Seq）中的一维时序卷积和第27章（预训练语言模型）中的Transformer自注意力提供了对比参照
- **横向连接**：CNN的参数共享思想与第3章（线性模型）中的正则化有共同的理论基础
