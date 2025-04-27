---
title: "Muon续集：为什么我们选择尝试Muon？"
date: 2025-02-27
description: ""
categories: ["数学研究", "信息时代"]
tags: ["矩阵", "梯度", "优化器", "谱范数"]
image: ""
math: true
draft: false
---

## 核心公式

以下是文章中涉及的核心数学公式：

1.  最小作用量原理（约束优化）：
    $$ \underset{\Delta W}{\text{argmin}}\text{Tr}\left( G^{\top}\Delta W \right)\left. \text{s.t.} \right\|\left. \Delta W \right\|_{2} \leq \eta \quad \text{(式4)} $$
2.  Muon更新方向（基于SVD的msign）：
    $$ \Delta W = - \eta\text{msign}(G) = - \eta U_{\lbrack:,:r\rbrack}V_{\lbrack:,:r\rbrack}^{\top}, U,\Sigma,V^{\top} = SVD(G) \quad \text{(式5)} $$
3.  带Weight Decay的Muon更新：
    $$ \Delta W = - \eta\left\lbrack \text{msign}(M) + \lambda W \right\rbrack \quad \text{(式6)} $$
4.  Weight Decay控制参数谱范数上界：
    $$ \left\| W_{t} \right\|_{2} \leq \max{(\|}\left. W_{0} \right\|_{2}\left. ,1/\lambda \right) \quad \text{(式8)} $$
5.  RMS定义：
    $$ \text{RMS}(W) = \frac{\left\| W \right\|_{F}}{\sqrt{nm}} = \sqrt{\frac{1}{nm}\sum\limits_{i = 1}^{n}\sum\limits_{j = 1}^{m}W_{i,j}^{2}} \quad \text{(式9)} $$
6.  RMS对齐后的Muon更新（考虑参数形状）：
    $$ W_{t} = W_{t - 1} - \eta_{t}\left( 0.2 O_{t}\sqrt{\max(n,m)} + \lambda W_{t - 1} \right) \quad \text{(式13)} $$
7.  奇异值熵：
    $$ H(\sigma) = - \frac{1}{\log n}\sum\limits_{i = 1}^{n}\frac{\sigma_{i}^{2}}{\sum\limits_{j = 1}^{n}\sigma_{j}^{2}}\log\frac{\sigma_{i}^{2}}{\sum\limits_{j = 1}^{n}\sigma_{j}^{2}} \quad \text{(式14)} $$

## 权重衰减 [\#](#权重衰减)

在大型模型上实践Muon时，Weight Decay（权重衰减）被发现至关重要。补上Weight Decay后的Muon更新公式如**核心公式**中的**式6**所示。事后分析表明，Weight Decay能有效控制参数的范数，特别是谱范数，使其保持有界（参见**核心公式**中的**式8**），从而防止训练过程中的数值不稳定或爆炸问题。

## RMS对齐 [\#](#RMS对齐)

为了快速将Adam的超参数迁移到Muon，我们提出了Update RMS对齐策略。RMS（Root Mean Square）度量了矩阵元素的平均大小（参见**核心公式**中的**式9**）。通过将新优化器每步更新量$O_t$的RMS对齐到Adam更新量RMS的典型值（如0.2），可以复用Adam的学习率和Weight Decay。对于Muon，其更新方向msign(M)的RMS可以解析计算。最终，RMS对齐后的Muon更新公式如**核心公式**中的**式13**所示。这个公式表明，对于形状差异大的矩阵参数，需要不同的学习率缩放因子$\sqrt{\max(n,m)}$，这解释了为何单一学习率可能导致不同步问题。

## Insights

1.  理想优化器应满足“稳”（扰动小）和“快”（Loss下降大），可建模为约束优化问题（最小作用量原理）。
2.  Muon使用谱范数（$\|\Delta W\|_2$）度量“稳”，相比Adam的F范数（$\|\Delta W\|_F$）更准确反映矩阵更新对模型输出的影响。
3.  Weight Decay对Muon至关重要，能有效控制参数范数，防止训练不稳定。
4.  提出Update RMS对齐的超参迁移策略，通过调整更新量幅度（基于解析计算的RMS），将Adam超参快速迁移到Muon，尤其对形状差异大的矩阵参数需要不同的学习率缩放。
5.  实验表明Muon训练出的模型奇异值分布更均匀（熵更高），更能发挥参数潜力。预训练和微调使用不同优化器可能影响效果。
6. 以往的模型都是用Adam训练的，人择原理可能导致了**模型都是为Adam专门设计的**

