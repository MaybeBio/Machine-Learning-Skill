# 第3章: 感知机 / Perceptron

> **融合来源**: 李航第2章(感知机) + 清华大学PPT
> **说明**: 本章仅李航有覆盖，西瓜书/南瓜书无独立章节。入门理解部分用直观语言引入。

---

## 一、入门理解 (无西瓜书覆盖，自行直观引入)

### 1.1 什么是感知机？

感知机是二类分类的线性模型，由Rosenblatt于1957年提出。它要做的事情极其简单而优雅：**在特征空间中画一条线（或超平面），将正负两类样本分开**。

想象平面上散布着红点和蓝点。感知机试图找到一条直线 $\mathbf{w} \cdot \mathbf{x} + b = 0$，使得直线一侧全是红点、另一侧全是蓝点。如果能找到这样的直线，数据就是**线性可分**的；如果红蓝点像国际象棋棋盘那样交错排列，感知机就只能"望点兴叹"了——这就是著名的**XOR问题**。

感知机是神经网络的基本单元（单个M-P神经元），也是支持向量机的前身。理解感知机，就理解了"线性分类器"最纯粹的形式。

输入为实例的特征向量 $\mathbf{x} \in \mathbb{R}^n$，输出为类别 $y \in \{+1, -1\}$。模型为：

$$f(\mathbf{x}) = \text{sign}(\mathbf{w} \cdot \mathbf{x} + b)$$

其中 $\text{sign}(x) = \begin{cases} +1, & x \geq 0 \\ -1, & x < 0 \end{cases}$。

几何解释：$\mathbf{w}$ 是超平面的法向量，$b$ 是偏置。$\mathbf{w} \cdot \mathbf{x} + b = 0$ 定义了分离超平面，正类在法向量指向的一侧。

---

## 二、公式推导

### 2.1 损失函数：误分类点到超平面的距离

感知机的学习策略是极小化误分类点到超平面的总距离。

空间中任一点 $\mathbf{x}_0$ 到超平面 $\mathbf{w} \cdot \mathbf{x} + b = 0$ 的距离为：

$$d = \frac{|\mathbf{w} \cdot \mathbf{x}_0 + b|}{\|\mathbf{w}\|}$$

对于误分类点 $(\mathbf{x}_i, y_i)$，有 $-y_i(\mathbf{w} \cdot \mathbf{x}_i + b) > 0$（因为预测与真实符号相反）。因此：

$$|\mathbf{w} \cdot \mathbf{x}_i + b| = -y_i(\mathbf{w} \cdot \mathbf{x}_i + b)$$

所有误分类点的总距离（忽略 $\frac{1}{\|\mathbf{w}\|}$ 因子）即为损失函数：

$$L(\mathbf{w}, b) = -\sum_{\mathbf{x}_i \in M} y_i(\mathbf{w} \cdot \mathbf{x}_i + b)$$

其中 $M$ 为误分类点集合。此损失函数非负——正确分类时 $y_i(\mathbf{w} \cdot \mathbf{x}_i + b) > 0$，损失为0。

### 2.2 随机梯度下降 (SGD)

损失函数的梯度：

$$\nabla_{\mathbf{w}} L = -\sum_{\mathbf{x}_i \in M} y_i \mathbf{x}_i, \quad \nabla_{b} L = -\sum_{\mathbf{x}_i \in M} y_i$$

使用随机梯度下降（每次随机选一个误分类点进行更新）：

$$\mathbf{w} \leftarrow \mathbf{w} + \eta y_i \mathbf{x}_i$$
$$b \leftarrow b + \eta y_i$$

其中 $\eta \in (0, 1]$ 为学习率。直观理解：当点被误分类时，调整超平面使其向正确方向移动。

### 2.3 Novikoff收敛定理（完整证明）

**定理**：设训练集线性可分，存在超平面 $\hat{\mathbf{w}}_{opt} \cdot \hat{\mathbf{x}} = 0$（$\|\hat{\mathbf{w}}_{opt}\| = 1$）和 $\gamma > 0$ 满足 $y_i(\hat{\mathbf{w}}_{opt} \cdot \hat{\mathbf{x}}_i) \geq \gamma$。另设 $R = \max_{1 \leq i \leq m} \|\hat{\mathbf{x}}_i\|$。则感知机在训练集上的误分类次数 $k$ 满足：

$$k \leq \left(\frac{R}{\gamma}\right)^2$$

**证明**：令 $\hat{\mathbf{w}} = (\mathbf{w}^T, b)^T$，$\hat{\mathbf{x}}_i = (\mathbf{x}_i^T, 1)^T$。

(1) 从与最优超平面的内积角度：$\hat{\mathbf{w}}_k \cdot \hat{\mathbf{w}}_{opt} \geq k\eta\gamma$
(2) 从自身范数角度：$\|\hat{\mathbf{w}}_k\|^2 \leq k\eta^2 R^2$

结合柯西不等式 $\hat{\mathbf{w}}_k \cdot \hat{\mathbf{w}}_{opt} \leq \|\hat{\mathbf{w}}_k\| \|\hat{\mathbf{w}}_{opt}\| = \|\hat{\mathbf{w}}_k\|$，得 $k\eta\gamma \leq \eta R\sqrt{k}$，即 $k \leq (R/\gamma)^2$。证毕。

**推论**：线性可分时，感知机算法必收敛（有限步内找到分离超平面）。收敛步数与间隔 $\gamma$ 的平方成反比——间隔越大，收敛越快。

### 2.4 对偶形式

原始形式更新 $\mathbf{w}$ 和 $b$，对偶形式将 $\mathbf{w}$ 表示为样本的线性组合：

$$\mathbf{w} = \sum_{i=1}^{m} \alpha_i y_i \mathbf{x}_i, \quad b = \sum_{i=1}^{m} \alpha_i y_i$$

其中 $\alpha_i = n_i \eta$，$n_i$ 是样本 $i$ 被误分类（用于更新）的次数。

对偶形式的决策函数：$f(\mathbf{x}) = \text{sign}\left(\sum_{j=1}^{m} \alpha_j y_j (\mathbf{x}_j \cdot \mathbf{x}) + b\right)$

对偶形式中训练样本仅以内积形式出现，可方便地用核函数替换（→核感知机，但与SVM不同，感知机没有间隔最大化的保证）。

---

## 三、理论深化 — 李航《机器学习方法》

### 3.1 统计学习三要素

**模型**：$f(\mathbf{x}) = \text{sign}(\mathbf{w} \cdot \mathbf{x} + b)$，假设空间是定义在特征空间中的所有线性分类模型（函数集合 $\{f | f(\mathbf{x}) = \text{sign}(\mathbf{w} \cdot \mathbf{x} + b)\}$）。

**策略**：极小化误分类点到超平面的总距离（经验风险最小化）。损失函数非负且连续可导(w.r.t $\mathbf{w}, b$)。

**算法**：随机梯度下降。每次用一个误分类样本更新参数，算法实现简单，收敛有理论保证（线性可分条件下）。

### 3.2 感知机与SVM、神经网络的联系

PPT补充：感知机是神经网络的基础单元，将Sigmoid替换sign函数即为逻辑回归的"神经元"版本。感知机的损失函数（误分类距离之和）可以看作SVM的Hinge Loss在无间隔要求时的特例。实际上，若给感知机加上"间隔最大化"的目标，就得到了线性SVM。

### 3.3 局限性

感知机要求数据线性可分，对XOR问题无能为力（Minsky和Papert在1969年《Perceptrons》一书中指出）。这是导致神经网络研究在1970年代陷入低潮的主要原因，直到1986年BP算法（多层网络→可解XOR）重新发明。

---

## 四、综合总结

### 关键结论

1. **感知机 = 线性分类的最小可行模型**：先找误分类点，沿梯度方向修正
2. **Novikoff定理**：线性可分必收敛，收敛步数上界 $(R/\gamma)^2$
3. **对偶形式的Gram矩阵**使核技巧成为可能，但感知机没有最大间隔保证
4. **解不唯一**：不同初值/样本顺序导致不同分离超平面
5. **线性不可分时**算法不收敛（震荡），需要多层网络或核方法
6. **感知机是SVM和神经网络的历史起点**

### 章节连接

- **前置依赖**: 第1章(统计学习三要素)
- **后续延伸**: 第7章(神经网络)—多层感知机+BP解XOR；第8章(SVM)—加上间隔最大化
