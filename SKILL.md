---
name: ml-skill 
description: "融合周志华《机器学习》(西瓜书)、李航《机器学习方法》、南瓜书公式推导、清华大学课件的统计学习与机器学习理论skill。涵盖监督学习、无监督学习、深度学习、强化学习等全套理论体系。用于学习机器学习理论、查阅概念公式、推导算法原理、辅助论文写作或工程实现。"
---

<!-- argument-hint: [章节号/主题名, 如 "SVM", "ch06", "EM算法", "集成学习", "对比LR和SVM" ] -->

# 统计学习方法 - 机器学习理论融合 Skill

**融合来源**: 周志华《机器学习》(西瓜书) + 南瓜书(公式推导) + 李航《机器学习方法》(第二版) + 清华大学《统计学习方法》课件
**生成日期**: 2026-07-13 | **总章节**: 28 | **定位**: 理论解析 + 理论使用

## How to Use This Skill

- **无参数** — 加载核心框架与决策规则
- **章节查询** — "ch06" 或 "SVM那一章" 加载指定章节全部内容
- **概念查询** — "什么是偏差-方差分解？" 或 "L1和L2正则化的区别？" 读取相关章节
- **对比查询** — "对比LR和SVM" / "Bagging和Boosting的区别"
- **公式推导** — "BP反向传播推导" / "EM算法E步M步推导"
- **算法选择** — "二分类不平衡数据用什么算法？" 根据决策规则给出建议
- **浏览** — "有哪些章节？" 查看完整章节索引

当你问及某个不在 Core Frameworks 中的具体话题时，我会加载对应章节文件来回答。

---

## Core Frameworks & Mental Models

### 统计学习三要素（贯穿全书的核心框架）

一切统计学习方法都可分解为三个要素的统一体：

| 要素 | 内涵 | 示例 |
|------|------|------|
| **模型 (Model)** | 假设空间 $\mathcal{H} = \{f_\theta \mid \theta \in \Theta\}$ | 线性函数族、神经网络架构、决策树空间 |
| **策略 (Strategy)** | 按什么准则选择最优模型 | 经验风险最小化(ERM)、结构风险最小化(SRM)、极大似然估计 |
| **算法 (Algorithm)** | 求解最优模型的具体计算方法 | 梯度下降、SMO、EM、BP |

**应用法则**: 面对任何新算法，先拆解它的三要素——模型是什么形式？损失函数是什么？优化方法是什么？

### 偏差-方差分解（模型选择的根本依据）

泛化误差 = $\text{Bias}^2 + \text{Variance} + \text{Noise}$

- **高偏差(欠拟合)**: 模型太简单 → 增加模型复杂度、添加特征、减少正则化
- **高方差(过拟合)**: 模型太复杂 → 增加数据、加正则化(L1/L2/Dropout)、早停、降维
- **噪声**: 不可约误差，决定了贝叶斯最优误差下界

### No Free Lunch 定理

在所有可能的问题上，所有学习算法的期望性能相同。**脱离具体问题谈算法优劣毫无意义**——算法必须在特定问题上下文中评估。归纳偏置(Inductive Bias)是学习能够发生的必要条件。

### PAC学习与样本复杂度

Probably Approximately Correct: 以高概率 $(1-\delta)$ 学到近似正确 $(\epsilon)$ 的假设。样本复杂度 $m \geq \frac{1}{\epsilon}(\ln|\mathcal{H}| + \ln\frac{1}{\delta})$。VC维是二分类假设空间的复杂度度量——VC维越大，模型容量越高，所需样本越多。

### 正则化统一视角

$$\min_f \frac{1}{m}\sum_{i=1}^m L(y_i, f(\mathbf{x}_i)) + \lambda \cdot \Omega(f)$$

- **L1 (Lasso)**: $\Omega = \|\mathbf{w}\|_1$ → 稀疏解，特征选择
- **L2 (Ridge/权重衰减)**: $\Omega = \|\mathbf{w}\|_2^2$ → 收缩系数，防过拟合
- **Elastic Net**: $\Omega = \alpha\|\mathbf{w}\|_1 + (1-\alpha)\|\mathbf{w}\|_2^2$ → 兼得稀疏与稳定性
- **Dropout**: 训练时随机丢弃神经元 → 隐式Bagging
- **Early Stopping**: 验证集误差开始上升时停止训练

### 模型选择决策链

```
数据 → [特征选择/降维] → [候选模型集] → [交叉验证评估] → [统计检验] → 选定模型
         ↑ L1/Relief        ↑ 按数据特点选  ↑ k-fold CV     ↑ McNemar/t-test
```

### 生成模型 vs 判别模型

| | 生成模型 | 判别模型 |
|---|---|---|
| **建模对象** | $P(X, Y)$ 联合分布 | $P(Y\|X)$ 条件分布或 $f(X)$ 决策函数 |
| **代表方法** | 朴素贝叶斯、HMM、LDA、GMM | 逻辑回归、SVM、CRF、感知机 |
| **优势** | 可生成新样本，可处理缺失数据 | 通常精度更高，对判别任务更直接 |
| **选择** | 数据量少或需要理解数据生成机制时 | 纯粹分类/预测任务时优先 |

### 集成学习三范式

- **Bagging (降低方差)**: 并行训练，Bootstrap采样，投票/平均 → 随机森林
- **Boosting (降低偏差)**: 串行训练，逐步聚焦前一步的错分样本 → AdaBoost, GBDT, XGBoost
- **Stacking (学习法)**: 用次级学习器组合初级学习器输出

**关键法则**: 个体学习器必须"好而不同"——准确性与多样性缺一不可。

### 最大间隔与核技巧

SVM核心思想: 寻找使分类间隔最大化的分离超平面。通过Lagrange对偶转化为凸二次规划。**核技巧**隐式地在高维空间做内积而无需显式计算映射 $\phi(\cdot)$: $K(\mathbf{x}_i, \mathbf{x}_j) = \langle \phi(\mathbf{x}_i), \phi(\mathbf{x}_j) \rangle$。核函数需满足Mercer条件。

### EM算法 = 含隐变量的MLE问题

当模型含不可观测的隐变量时，直接最大化 $\log P(X|\theta)$ 困难。EM交替进行:
- **E步**: 基于当前参数 $\theta^{(t)}$ 计算隐变量的后验期望 → 构造 $Q(\theta, \theta^{(t)})$
- **M步**: 最大化 $Q$ 得到 $\theta^{(t+1)}$

收敛到局部最优。GMM、HMM的Baum-Welch、K-means都是EM的特例。

### 深度学习的三个支柱

1. **层次化表示学习**: 浅层特征 → 中层模式 → 高层语义
2. **端到端训练**: 用可微的计算图连接输入到输出，自动求导 + SGD优化
3. **结构先验**: CNN的平移不变性(卷积+池化), RNN的时序依赖, Transformer的自注意力

---

## Chapter Index

### Part A: 绪论与基础 (2章)
| # | Title | 核心主题 |
|---|-------|----------|
| [ch01](chapters/ch01-introduction.md) | 绪论与统计学习概论 | 三要素、归纳偏置、NFL定理、监督/无监督分类、损失函数 |
| [ch02](chapters/ch02-model-evaluation.md) | 模型评估与选择 | 交叉验证、P/R/F1/ROC/AUC、偏差-方差分解、统计检验 |

### Part B: 传统监督学习 (12章)
| # | Title | 核心主题 |
|---|-------|----------|
| [ch03](chapters/ch03-perceptron.md) | 感知机 | 原始/对偶形式、SGD、Novikoff收敛证明 |
| [ch04](chapters/ch04-knn.md) | k近邻法 | 距离度量、k值选择、kd-tree搜索 |
| [ch05](chapters/ch05-linear-models.md) | 线性模型与逻辑回归 | 线性回归、Logistic回归、LDA、最大熵、GLM |
| [ch06](chapters/ch06-decision-trees.md) | 决策树 | ID3/C4.5/CART、信息增益、Gini指数、剪枝 |
| [ch07](chapters/ch07-neural-networks.md) | 神经网络与BP算法 | 感知机、BP推导、激活函数、优化算法 |
| [ch08](chapters/ch08-svm.md) | 支持向量机 | 对偶理论、KKT条件、核技巧、SMO、SVR |
| [ch09](chapters/ch09-bayes.md) | 贝叶斯分类器与朴素贝叶斯 | 贝叶斯定理、朴素贝叶斯、贝叶斯网、d-分离 |
| [ch10](chapters/ch10-ensemble.md) | 集成学习与Boosting | Bagging/RF、Boosting/AdaBoost/GBDT、Stacking |
| [ch11](chapters/ch11-em.md) | EM算法 | Jensen不等式、Q函数、GMM应用、收敛性 |
| [ch12](chapters/ch12-hmm.md) | 隐马尔可夫模型 | 前向-后向、Viterbi解码、Baum-Welch学习 |
| [ch13](chapters/ch13-crf.md) | 条件随机场 | 特征函数、全局归一化、与HMM对比、标注偏置 |
| [ch14](chapters/ch14-supervised-summary.md) | 监督学习方法总结 | 三要素统一表、模型分类、方法选择指南 |

### Part C: 无监督学习 (4章)
| # | Title | 核心主题 |
|---|-------|----------|
| [ch15](chapters/ch15-clustering.md) | 聚类 | k-means、GMM/EM、DBSCAN、层次聚类、性能度量 |
| [ch16](chapters/ch16-dim-reduction.md) | 降维与度量学习 | SVD、PCA、KPCA、流形学习(Isomap/LLE/t-SNE)、度量学习 |
| [ch17](chapters/ch17-probabilistic-graphical.md) | 概率图模型与话题模型 | 贝叶斯网、MRF、HMM、LSA/PLSA/LDA、MCMC、变分推断 |
| [ch18](chapters/ch18-unsupervised-summary.md) | 无监督学习方法总结 | 聚类/降维/话题分析/图分析的统一视角、三要素表 |

### Part D: 特种专题 (4章)
| # | Title | 核心主题 |
|---|-------|----------|
| [ch19](chapters/ch19-feature-selection.md) | 特征选择与稀疏学习 | Relief、LVW、LASSO/L1正则化、PGD、字典学习 |
| [ch20](chapters/ch20-computational-learning.md) | 计算学习理论 | PAC学习、VC维、Rademacher复杂度、泛化误差界 |
| [ch21](chapters/ch21-semi-supervised.md) | 半监督学习 | 生成式方法、S3VM/TSVM、图方法、协同训练 |
| [ch22](chapters/ch22-rule-learning.md) | 规则学习 | 序贯覆盖、RIPPER、FOIL、ILP/一阶规则 |

### Part E: 深度学习与前沿 (6章)
| # | Title | 核心主题 |
|---|-------|----------|
| [ch23](chapters/ch23-reinforcement-learning.md) | 强化学习 | MDP、Bellman方程、TD学习(SARSA/Q-Learning)、策略梯度 |
| [ch24](chapters/ch24-pagerank-lsa.md) | PageRank与LSA | 幂法、随机游走、SVD与话题分析、NMF |
| [ch25](chapters/ch25-cnn.md) | 卷积神经网络 | 卷积/池化、经典CNN架构、迁移学习 |
| [ch26](chapters/ch26-rnn-seq2seq.md) | RNN与Seq2Seq | LSTM/GRU、编码器-解码器、Attention机制 |
| [ch27](chapters/ch27-pretrained-lm.md) | 预训练语言模型 | Transformer、BERT、GPT、预训练+微调范式 |
| [ch28](chapters/ch28-gan-summary.md) | GAN与深度学习总结 | GAN原理与变体、深度学习三支柱、未来展望 |

---

## Topic Index

<!-- 按拼音排序 -->
- **AdaBoost** → ch10
- **Attention/注意力** → ch26
- **AUC/ROC** → ch02
- **Bagging** → ch10
- **Bayes Theorem/贝叶斯定理** → ch09
- **Bellman Equation** → ch23
- **BERT/GPT** → ch27
- **Bias-Variance/偏差-方差** → ch02
- **Boosting** → ch10
- **CNN/卷积神经网络** → ch25
- **CRF/条件随机场** → ch13
- **DBSCAN** → ch15
- **Decision Tree/决策树** → ch06
- **Dropout** → ch07
- **EM Algorithm** → ch11
- **Ensemble/集成** → ch10
- **Feature Selection/特征选择** → ch19
- **GAN/生成对抗网络** → ch28
- **Gaussian Mixture Model** → ch11, ch15
- **Gradient Boosting/XGBoost** → ch10
- **HMM/隐马尔可夫模型** → ch12
- **k-means** → ch15
- **k-NN** → ch04
- **Kernel Trick/核技巧** → ch08
- **KKT Conditions** → ch08
- **L1/L2 Regularization** → ch05, ch19
- **Lasso/Ridge** → ch05, ch19
- **LDA/线性判别分析** → ch05
- **Logistic Regression** → ch05
- **LSTM/GRU** → ch26
- **Maximum Entropy/最大熵** → ch05
- **MCMC** → ch17
- **MDP/马尔可夫决策过程** → ch23
- **Naive Bayes/朴素贝叶斯** → ch09
- **NFL Theorem** → ch01
- **PAC Learning** → ch20
- **PageRank** → ch24
- **PCA/主成分分析** → ch16
- **Perceptron/感知机** → ch03
- **PGM/概率图模型** → ch17
- **Q-Learning** → ch23
- **Random Forest/随机森林** → ch10
- **Reinforcement Learning** → ch23
- **RNN/循环神经网络** → ch26
- **Rule Learning/规则学习** → ch22
- **Seq2Seq** → ch26
- **Semi-supervised/半监督** → ch21
- **SMO Algorithm** → ch08
- **SRM/结构风险最小化** → ch01
- **SVD/奇异值分解** → ch16, ch24
- **SVM/支持向量机** → ch08
- **t-SNE** → ch16
- **Transformer** → ch27
- **VC Dimension/VC维** → ch20
- **Viterbi/维特比** → ch12

---

## Supporting Files

- [glossary.md](glossary.md) — 95+ 核心术语的精确定义与章节索引
- [patterns.md](patterns.md) — 20+ 算法模式/设计模式，含使用场景与公式
- [cheatsheet.md](cheatsheet.md) — 决策规则表、算法选择指南、超参数默认值、常见误区速查

---

## 章节依赖关系与学习路径

### 推荐学习次序

```
Phase 1 基础:    ch01(绪论) → ch02(评估) → ch20(学习理论) → ch03(感知机) → ch04(kNN)
Phase 2 经典:    ch05(线性) → ch06(决策树) → ch09(贝叶斯) → ch07(NN基础) → ch08(SVM)
Phase 3 集成:    ch10(集成) → ch19(特征选择) → ch21(半监督)
Phase 4 序列:    ch11(EM) → ch12(HMM) → ch13(CRF) → ch14(监督总结)
Phase 5 无监督:  ch15(聚类) → ch16(降维) → ch17(概率图) → ch18(无监督总结)
Phase 6 前沿:    ch22(规则) → ch23(强化) → ch24(PageRank/LSA) → ch25(CNN) → ch26(RNN) → ch27(预训练) → ch28(GAN总结)
```

### 关键前置依赖
- ch11(EM) 依赖 ch09(贝叶斯) 的基本概率知识
- ch27(预训练LM) 依赖 ch26(RNN/Seq2Seq) 和 ch07(NN基础)
- ch12(HMM) 和 ch13(CRF) 依赖 ch17(概率图) 的PGM概念
- ch16(降维) 依赖 ch24(PageRank/LSA) 的 SVD 部分
- ch28(GAN总结) 依赖 ch25(CNN) 和 ch07(NN基础)

---

## Scope & Limits

本 Skill 覆盖机器学习与统计学习的**完整理论体系**，融合4套中文教材的精华内容，共计28章。涵盖从最基础的概念（监督/无监督、过拟合）到前沿深度学习（Transformer、BERT、GAN）的全部理论知识点。

**适用范围**:
- 机器学习理论与算法的学习与复习
- 公式推导的精确查阅（含南瓜书和全部推导步骤）
- 算法选择与模型设计的理论指导
- 面试准备与知识体系梳理

**不适用范围**:
- 特定框架（PyTorch/TensorFlow）的API使用
- 最新前沿论文的方法细节（本Skill基于经典教材）
- 工程实践中的特定数据集调参经验

**来源覆盖**:
- 西瓜书(周志华) Ch1-16: 所有章节内容（入门理解与直观例子）
- 南瓜书: Ch1-16 全部公式推导
- 李航《机器学习方法》: Ch1-29 全部理论（方法与第二版全部内容）
- 清华大学课件: 核心章节实例与补充
