# 机器学习与统计学习术语表 / Machine Learning & Statistical Learning Glossary

## A

**Accuracy (准确率)** — 分类正确的样本数占总样本数的比例，最直观的分类性能度量 (Ch 2)

**Activation Function (激活函数)** — 神经网络中引入非线性的逐元素函数，如ReLU、Sigmoid、tanh (Ch 5, Ch 22)

**AdaBoost** — 自适应提升算法，通过调整误分类样本权重逐步训练弱分类器的集成方法 (Ch 8)

**Adam (自适应矩估计)** — 结合动量和RMSProp的自适应学习率优化算法，深度学习默认选择 (Ch 22)

**Agent (智能体)** — 强化学习中与环境交互并学习最优策略的决策实体 (Ch 19)

**AUC (Area Under ROC Curve)** — ROC曲线下的面积，衡量分类器整体排序质量的指标 (Ch 2)

**Attention Mechanism (注意力机制)** — 通过动态加权聚合编码器隐状态的机制，使解码器能关注输入序列中与当前输出最相关的部分 (Ch 22)

**Autoencoder (自动编码器)** — 以重构输入为目标的无监督神经网络，编码器压缩数据，解码器还原数据 (Ch 22)

## B

**Backpropagation (反向传播)** — 利用链式法则从输出层向输入层逐层计算参数梯度的算法 (Ch 5, Ch 22)

**Bagging (自助聚合)** — Bootstrap Aggregating，通过对训练集自助采样训练多个模型并投票/平均以降低方差的集成方法 (Ch 8)

**Batch Normalization (批量归一化)** — 对每层输入的mini-batch做零均值单位方差归一化，缓解内部协变量偏移 (Ch 22)

**Bayes' Theorem (贝叶斯定理)** — $P(Y|X) = \frac{P(X|Y)P(Y)}{P(X)}$，连接先验、似然和后验的核心公式 (Ch 4)

**Bayesian Inference (贝叶斯推断)** — 基于先验分布和观测数据通过贝叶斯定理更新参数后验分布的统计推断方法 (Ch 4)

**Beam Search (束搜索)** — 在序列生成中维护B个最优候选序列的启发式搜索策略 (Ch 22)

**BERT** — 基于Transformer编码器的双向预训练语言模型，使用掩码语言模型（MLM）预训练 (Ch 22)

**Bias (偏差)** — 模型预测的期望值与真实值之间的差距，反映模型拟合能力 (Ch 2)

**Bias-Variance Tradeoff (偏差-方差权衡)** — 模型复杂度增加时偏差降低但方差升高，最优模型在二者间平衡 (Ch 2)

**Boosting** — 串行训练弱学习器，每个新学习器重点关注前序模型误分类样本的集成方法 (Ch 8)

**Bootstrap (自助法)** — 从原始数据集中有放回地随机采样生成多个训练集的重采样方法 (Ch 2)

## C

**CART (分类与回归树)** — 基于Gini系数（分类）或平方误差（回归）的二叉树生成算法 (Ch 6)

**Clustering (聚类)** — 将样本划分为若干组（簇），使组内相似度高而组间相似度低的无监督学习方法 (Ch 9)

**CNN (卷积神经网络)** — 使用卷积层、汇聚层交替堆叠处理网格结构数据（如图像）的神经网络 (Ch 22)

**Conditional Random Field (CRF, 条件随机场)** — 给定观测序列条件下对状态序列建模的无向图判别模型，常用于序列标注 (Ch 14)

**Confusion Matrix (混淆矩阵)** — 以矩阵形式展示分类结果的工具，行列分别对应真实类别和预测类别 (Ch 2)

**Convex Optimization (凸优化)** — 目标函数为凸函数、可行域为凸集的最优化问题，局部最优即全局最优 (Ch 7)

**Convolution (卷积)** — CNN核心运算，使用可学习的卷积核在输入上滑动计算局部加权和 (Ch 22)

**Cross-Entropy Loss (交叉熵损失)** — $L = -\sum_k y_k \log \hat{y}_k$，度量预测分布与真实分布差异的损失函数 (Ch 3)

**Cross-Validation (交叉验证)** — 将数据划分为K份，轮流以K-1份训练、1份验证的模型评估方法 (Ch 2)

**Curse of Dimensionality (维度灾难)** — 数据维度增加导致样本密度指数级下降，使距离度量失效的现象 (Ch 15)

## D

**Decision Boundary (决策边界)** — 分类模型在特征空间中划分不同类别的分界面 (Ch 3, Ch 7)

**Decision Tree (决策树)** — 基于特征条件逐层划分样本的分类/回归树形模型 (Ch 6)

**Deep Learning (深度学习)** — 使用深层神经网络自动学习层次化特征表示的机器学习范式 (Ch 5, Ch 22)

**Dimensionality Reduction (降维)** — 将高维数据转换为保留关键结构的低维表示的方法 (Ch 13, Ch 15)

**Discriminative Model (判别模型)** — 直接建模条件概率 $P(Y|X)$ 或决策边界 $f(X)$ 的模型 (Ch 1)

**Dropout (暂退法)** — 训练时随机失活神经元以防止过拟合的正则化技术 (Ch 5, Ch 22)

## E

**Early Stopping (早停法)** — 在验证误差停止下降时终止训练，防止过拟合的正则化方法 (Ch 5, Ch 22)

**Eigenvalue Decomposition (特征值分解)** — 将方阵分解为 $A = Q\Lambda Q^{-1}$ 的形式，PCA的理论基础 (Ch 13)

**EM Algorithm (期望最大化算法)** — 迭代执行E步（计算隐变量期望）和M步（最大化似然）求解含隐变量模型参数的方法 (Ch 12)

**Embedding (嵌入)** — 将离散对象（如词）映射为稠密低维实数向量的表示学习方法 (Ch 22)

**Ensemble Learning (集成学习)** — 组合多个学习器以获得优于单一学习器性能的方法 (Ch 8)

**Entropy (熵)** — $H(X) = -\sum_i P(x_i) \log P(x_i)$，度量随机变量不确定性的信息论概念 (Ch 3)

**Exploding Gradient (梯度爆炸)** — 深层网络中梯度在反向传播时指数增长导致参数更新过大的问题 (Ch 22)

## F

**F1 Score (F1值)** — 精确率和召回率的调和平均：$F1 = \frac{2 \times P \times R}{P + R}$ (Ch 2)

**Feature Engineering (特征工程)** — 基于领域知识从原始数据中构造有效特征的过程 (Ch 1)

**Feedforward Neural Network (前馈神经网络)** — 信息从输入层经过隐层单向流向输出层、无环路的神经网络 (Ch 5, Ch 22)

**Frobenius Norm (弗罗贝尼乌斯范数)** — $\|A\|_F = \sqrt{\sum_{i,j} a_{ij}^2} = \sqrt{\sum_i \sigma_i^2}$，矩阵元素的平方和的平方根 (Ch 21)

## G

**GAN (生成对抗网络)** — 生成器和判别器进行零和博弈的生成模型，生成器学习生成逼真数据 (Ch 22)

**Gaussian Mixture Model (高斯混合模型)** — 用多个高斯分布的加权和建模复杂分布的生成模型 (Ch 9, Ch 12)

**Generalization (泛化)** — 模型在未见过的测试数据上的预测能力 (Ch 1, Ch 2)

**Generative Model (生成模型)** — 先学习联合分布 $P(X, Y)$，再通过贝叶斯公式推导 $P(Y|X)$ 的模型 (Ch 1, Ch 4)

**Gini Index (基尼指数)** — 度量数据集纯度的指标，取值越小越纯，CART决策树使用它选择划分特征 (Ch 6)

**Gradient Descent (梯度下降)** — 沿负梯度方向迭代搜索函数极小值的最优化方法 (Ch 7, Ch 22)

**GRU (门控循环单元)** — LSTM的精简版，合并遗忘门和输入门为更新门，合并细胞状态和隐状态 (Ch 22)

## H

**Hidden Markov Model (HMM, 隐马尔可夫模型)** — 观测序列可观测但状态序列隐藏的马尔可夫模型，属于生成式序列标注模型 (Ch 11)

**Hinge Loss (合页损失)** — $L(y, f(x)) = \max(0, 1 - yf(x))$，SVM使用的损失函数 (Ch 7)

**Hyperparameter (超参数)** — 在训练前设定而非从数据学习的参数，如学习率、正则化系数、网络层数 (Ch 2)

**Hypothesis Space (假设空间)** — 所有可能的模型构成的集合，学习即在此空间中搜索最优模型 (Ch 1)

## I

**Independent and Identically Distributed (i.i.d., 独立同分布)** — 机器学习的基本假设：训练和测试样本独立地从同一分布中采样 (Ch 1)

**Information Gain (信息增益)** — 使用某特征划分数据前后熵的减少量，决策树选择特征的核心准则 (Ch 6)

**Internal Covariate Shift (内部协变量偏移)** — 深度网络训练中各层输入分布持续变化导致学习困难的现象 (Ch 22)

## K

**K-means Clustering (K均值聚类)** — 迭代优化簇中心，将样本分配到最近中心的无监督聚类算法 (Ch 9)

**K-Nearest Neighbors (K近邻)** — 基于与测试样本最近的K个训练样本的多数类别投票进行分类的惰性学习方法 (Ch 3)

**Kernel Method (核方法)** — 通过核函数隐式地将数据映射到高维特征空间进行线性处理，避免显式高维计算的技巧 (Ch 7)

**Kernel Trick (核技巧)** — 使用满足Mercer条件的核函数 $K(x, y) = \phi(x)^{\mathrm{T}}\phi(y)$ 替代高维内积计算 (Ch 7)

**KL Divergence (KL散度)** — $D_{\text{KL}}(P\|Q) = \sum P(x) \log \frac{P(x)}{Q(x)}$，度量两个概率分布差异的非对称度量 (Ch 3)

## L

**Lagrange Duality (拉格朗日对偶性)** — 将带约束的原始优化问题转化为无约束对偶问题求解的方法 (Ch 7)

**Latent Dirichlet Allocation (LDA, 潜在狄利克雷分配)** — 假设文档由话题生成、话题由单词生成的贝叶斯概率话题模型 (Ch 20)

**Latent Semantic Analysis (LSA, 潜在语义分析)** — 通过SVD将单词-文档矩阵分解为话题空间的降维和话题分析方法 (Ch 21)

**Layer Normalization (层归一化)** — 对单个样本的同一层所有神经元做归一化，Transformer中的标准归一化方式 (Ch 22)

**Learning Rate (学习率)** — $\eta$，控制梯度下降中每步参数更新幅度的超参数 (Ch 7, Ch 22)

**Likelihood (似然)** — $P(X|\theta)$，在给定参数下观测到当前数据的概率 (Ch 4)

**Linear Regression (线性回归)** — 假设目标值与特征呈线性关系 $y = w^{\mathrm{T}}x + b$ 的回归模型 (Ch 3)

**Logistic Regression (逻辑斯谛回归)** — 使用sigmoid函数将线性输出映射到[0,1]区间的二分类线性模型 (Ch 3)

**LSTM (长短期记忆网络)** — 引入遗忘门、输入门、输出门和细胞状态来解决梯度消失的循环神经网络变体 (Ch 22)

## M

**Manifold Learning (流形学习)** — 假设高维数据分布在低维流形上的非线性降维方法 (Ch 15)

**Margin (间隔)** — SVM中样本点到分类超平面的距离，大间隔意味着更稳健的分类器 (Ch 7)

**Markov Chain Monte Carlo (MCMC, 马尔可夫链蒙特卡罗法)** — 通过构造马尔可夫链从复杂分布中采样的方法 (Ch 19)

**Maximum Entropy Model (最大熵模型)** — 在满足特征约束下选择熵最大的概率分布作为模型的判别方法 (Ch 3)

**Maximum Likelihood Estimation (MLE, 极大似然估计)** — 选择使观测数据似然最大的参数作为估计值的参数估计方法 (Ch 3, Ch 4)

**Mini-batch (小批量)** — 每次迭代使用训练集的一个子集计算梯度，兼顾计算效率和梯度稳定性的训练策略 (Ch 22)

## N

**Naive Bayes (朴素贝叶斯)** — 基于特征条件独立假设和贝叶斯定理的分类器 (Ch 4)

**Neural Network (神经网络)** — 由多层神经元组成的非线性模型，通过反向传播算法训练 (Ch 5, Ch 22)

**NMF (非负矩阵分解)** — $X \approx WH, W \geq 0, H \geq 0$，强制分解结果非负的矩阵分解方法 (Ch 21)

**Nonlinear Dimensionality Reduction (非线性降维)** — 使用非线性映射进行降维，如核PCA、ISOMAP、t-SNE (Ch 15)

## O

**Overfitting (过拟合)** — 模型对训练数据学习过细，捕捉了噪声而损失泛化能力的现象 (Ch 1, Ch 2)

## P

**PageRank** — 基于有向图随机游走平稳分布的结点重要性排序算法 (Ch 21)

**PCA (主成分分析)** — 寻找数据方差最大的正交方向的线性降维方法 (Ch 13)

**Perceptron (感知机)** — 最简单的二分类线性模型 $f(x) = \text{sign}(w^{\mathrm{T}}x + b)$，SVM和神经网络的基础 (Ch 3)

**Pooling (汇聚)** — CNN中对局部区域做统计汇总（如取最大值）以降低空间分辨率和增加不变性的操作 (Ch 22)

**Precision (精确率)** — $P = \frac{TP}{TP + FP}$，预测为正类的样本中真正为正类的比例 (Ch 2)

**Principal Component (主成分)** — PCA中找到的使方差最大的正交方向 (Ch 13)

**Pruning (剪枝)** — 删除决策树中冗余分支以防止过拟合的简化策略 (Ch 6)

## R

**Random Forest (随机森林)** — 在Bagging框架下使用随机选择特征的决策树构成的集成方法 (Ch 8)

**Recall (召回率)** — $R = \frac{TP}{TP + FN}$，所有正类样本中被正确预测为正类的比例 (Ch 2)

**Regularization (正则化)** — 在损失函数中添加参数范数惩罚项以限制模型复杂度的防过拟合策略 (Ch 2)

**Reinforcement Learning (强化学习)** — 智能体通过与环境交互获取奖励信号来学习最优策略的机器学习范式 (Ch 19)

**ReLU (整流线性单元)** — $a(z) = \max(0, z)$，最常用的深度学习激活函数，有效缓解梯度消失 (Ch 5, Ch 22)

**Representation Learning (表示学习)** — 自动从数据中发现有利于任务的有效特征表示的学习范式 (Ch 22)

**Residual Connection (残差连接)** — $h_{l+1} = h_l + f_l(h_l)$，将层的输入跳过非线性变换直接加至输出的技术 (Ch 22)

**ResNet (残差网络)** — 使用跳层连接（skip connection）训练超深网络的CNN架构 (Ch 22)

**RNN (循环神经网络)** — 通过隐状态在时间步之间传递信息的序列建模神经网络 (Ch 22)

**ROC Curve (ROC曲线)** — 以假正率（FPR）为横轴、真正率（TPR）为纵轴的分类器性能评估曲线 (Ch 2)

## S

**Self-Attention (自注意力)** — 序列内部元素之间计算注意力权重以捕捉长距离依赖的机制 (Ch 22)

**Semi-Supervised Learning (半监督学习)** — 同时利用少量标注样本和大量未标注样本进行学习的范式 (Ch 1)

**Seq2Seq (序列到序列模型)** — 使用编码器-解码器架构将输入序列映射到输出序列的框架 (Ch 22)

**Sigmoid Function (S型函数)** — $\sigma(z) = \frac{1}{1 + e^{-z}}$，将实数映射到$(0,1)$区间的激活函数 (Ch 3, Ch 5)

**Singular Value Decomposition (SVD, 奇异值分解)** — $A = U\Sigma V^{\mathrm{T}}$，任意矩阵的正交分解，PCA和LSA的数学基础 (Ch 21)

**Softmax Function (软最大化函数)** — $\text{softmax}(z)_i = \frac{e^{z_i}}{\sum_j e^{z_j}}$，将任意实数向量转换为概率分布 (Ch 3)

**Stochastic Gradient Descent (SGD, 随机梯度下降)** — 每次迭代仅使用一个或少量样本计算梯度的梯度下降变体 (Ch 7, Ch 22)

**Supervised Learning (监督学习)** — 从带标注的训练数据中学习输入到输出映射的机器学习范式 (Ch 1)

**Support Vector (支持向量)** — 训练集中距离分类超平面最近的样本点，决定了SVM的决策边界 (Ch 7)

**Support Vector Machine (SVM, 支持向量机)** — 通过最大化间隔寻找最优分类超平面的判别模型 (Ch 7)

## T

**TF-IDF** — 词频-逆文档频率，评估词语对文档重要性的加权技术 (Ch 21)

**Transformer** — 完全基于自注意力机制、抛弃循环结构的序列到序列模型 (Ch 22)

**Transposed Convolution (转置卷积)** — 对应原始卷积的转置矩阵运算，用于CNN中特征图的上采样 (Ch 22)

## U

**Underfitting (欠拟合)** — 模型复杂度过低，未能充分学习训练数据的规律 (Ch 2)

**Unsupervised Learning (无监督学习)** — 从无标注数据中学习数据内在结构和规律的机器学习范式 (Ch 9, Ch 21)

## V

**Variance (方差)** — 模型在不同训练集上学到的结果之间的差异，反映模型对训练数据的敏感性 (Ch 2)

**VC Dimension (VC维)** — 假设空间能完全打散的最大样本集大小，度量假设空间的容量 (Ch 2)

**Viterbi Algorithm (维特比算法)** — 基于动态规划在HMM中求解最可能状态序列的解码算法 (Ch 11)

## W

**WGAN (Wasserstein GAN)** — 使用Wasserstein距离替代JS散度的GAN变体，训练更稳定 (Ch 22)

**Word Embedding (词向量)** — 将词语映射为稠密低维实数向量的表示方法，语义相似的词向量相近 (Ch 22)
