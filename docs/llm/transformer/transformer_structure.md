---
title: 一层Transformer Block长什么样?自注意力、FFN、残差、LayerNorm拼起来
description: 一层Transformer Block长什么样?本文按自注意力、残差连接、LayerNorm、FFN、Add&Norm的前向顺序,把Transformer核心组件串成完整流程,讲清每个模块输入输出和作用,帮助理解大模型架构基础和面试常见问题。
keywords: ["Transformer Block", "自注意力", "FFN", "残差连接", "LayerNorm", "Transformer架构", "前向传播"]
tags: ["Transformer", "核心组件", "大模型原理"]
---

# 一层Transformer Block长什么样?自注意力、FFN、残差、LayerNorm拼起来

前面几篇文章我们把 Transformer 的核心组件一个个拆开讲了:

- [Attention 到底在干嘛](./qkv.md)——Q、K、V 的直觉含义
- [Attention 计算全过程](./qkv_cal.md)——从 QK^T 到 Softmax 到加权求和
- [Multi-Head Attention](./mha.md)——为什么要用多个头,怎么拆分和拼接
- [位置编码](./pos_encode.md)——Transformer 怎么知道词的顺序
- [残差连接、LayerNorm、FFN](./ffn_ln.md)——三个缺一不可的配角

这篇文章我们把焦点放在最关键的地方:**一层 Transformer Block 到底长什么样?这些组件是怎么串起来的?**

你可以把 Transformer 想象成一栋楼,每一层楼就是一个 Block。无论楼有多高,每一层的结构都是相同的。搞清楚"一层楼"长什么样,整栋楼就通了。

## 面试官最爱问的一个追问

"一层 Transformer Block 包含哪些组件?它们的顺序是什么?"

很多录友会答:

> 一层 Block 包含 Self-Attention 和 FFN,中间有残差连接和 LayerNorm。

面试官接着问:"残差是加在 Attention 之前还是之后?LayerNorm 是在残差之前还是之后?Pre-LN 和 Post-LN 有什么区别?"

答不上来。

问题出在哪?**你只知道"有这些组件",但没搞清楚它们的顺序、输入输出的维度、为什么这么组织。** 面试官追问的都是细节——组件之间是怎么串联的?数据流是什么?这些问题的答案藏在 Transformer Block 的结构里。

## 一层 Transformer Block 的核心结构

一个 Transformer Block,由两个子层组成:

- **自注意力子层**(Self-Attention Sublayer)
- **前馈神经网络子层**(FFN Sublayer)

每个子层外面,都包裹着两件固定的"外套":**残差连接** 和 **层归一化**。

**核心逻辑:两个子层(Attention + FFN)+ 两层包装(残差 + LayerNorm)= 一个 Block。**

<!-- drawio 源文件: ./drawio/transformer_structure_01_block_overview.drawio -->

![Transformer Block结构](https://file1.kamacoder.com/i/web/20261022110416.png?v=20261005)

## 从输入到输出:完整的数据流

### 输入

假设输入是一个矩阵 X,形状为 **n × d_model**(n 是序列长度,d_model 是向量维度,比如 512)。

每一行是一个 Token 的向量表示,已经加上了位置编码。

### 第一步:Multi-Head Self-Attention

输入 X 进入第一个子层:Multi-Head Self-Attention。

回顾一下,Multi-Head Attention 做的事:

1. 把 X 分别乘以 W_Q、W_K、W_V,得到 Q、K、V
2. 把 Q、K、V 拆分成 h 个头
3. 每个头独立计算 Attention
4. 拼接所有头的输出,再乘以 W_O

输出记作 **Attention(X)**,形状还是 **n × d_model**。

### 第二步:Add & Norm(第一次)

Attention 的输出不是直接往下传,而是先做两件事:

1. **残差连接**:把输入 X 加回来
2. **LayerNorm**:归一化到均值 0、方差 1

$$
X' = \text{LayerNorm}(X + \text{Attention}(X))
$$

这就是第一个 **Add & Norm**。

**为什么要残差连接?** 防止梯度消失,给梯度开一条高速公路。

**为什么要 LayerNorm?** 防止数值爆炸,让输出分布稳定。

输出记作 X',形状还是 **n × d_model**。

### 第三步:Feed-Forward Network

X' 进入第二个子层:FFN。

FFN 的结构:

$$
\text{FFN}(x) = \text{GELU}(x \cdot W_1 + b_1) \cdot W_2 + b_2
$$

先升维(512 → 2048),再激活,再降维(2048 → 512)。

输出记作 **FFN(X')**,形状还是 **n × d_model**。

**为什么要 FFN?** Self-Attention 是线性的,FFN 补上非线性能力,让模型能学到更复杂的特征。

### 第四步:Add & Norm(第二次)

FFN 的输出同样要做一次残差连接和 LayerNorm:

$$
X'' = \text{LayerNorm}(X' + \text{FFN}(X'))
$$

这就是第二个 **Add & Norm**。

输出记作 X'',形状还是 **n × d_model**。

### 输出

最终输出 X'',形状和输入 X 完全一致:**n × d_model**。

**这是 Transformer 能"堆深"的关键:输入和输出维度不变,可以把很多层 Block 串联起来,一层接一层,最终形成深层网络。**

## 完整流程一眼看清

| 步骤 | 操作 | 输入 | 输出 | 目的 |
|------|------|------|------|------|
| ① | Multi-Head Attention | X(n × d) | Attn(X)(n × d) | Token 间互相交流,聚合全局信息 |
| ② | Add & Norm | X, Attn(X) | X'(n × d) | 残差连接 + 归一化,防止梯度消失和数值爆炸 |
| ③ | FFN | X'(n × d) | FFN(X')(n × d) | 非线性变换,深化特征表示 |
| ④ | Add & Norm | X', FFN(X') | X''(n × d) | 残差连接 + 归一化,稳定输出 |

**输入是 n × d,输出还是 n × d,维度不变,但每个 Token 的向量里已经融合了全局上下文信息和深层非线性特征。**

## Post-LN vs Pre-LN

上面讲的是 **Post-LN** 的结构:先残差相加,再 LayerNorm。

$$
X' = \text{LayerNorm}(X + \text{Attention}(X))
$$

现代大模型(比如 Llama)改成了 **Pre-LN**:先 LayerNorm,再变换,最后残差相加。

$$
X' = X + \text{Attention}(\text{LayerNorm}(X))
$$

### 两者的区别

| | Post-LN(原始 Transformer) | Pre-LN(现代大模型) |
|---|---|---|
| **LayerNorm 位置** | 在残差连接之后 | 在子层输入之前 |
| **训练稳定性** | 深层时可能不稳定,需要 warmup | 更稳定,不需要 warmup |
| **收敛速度** | 较慢 | 较快 |
| **代表模型** | 原始 Transformer、BERT | GPT-3、Llama、Qwen |

**为什么 Pre-LN 更稳定?**

Post-LN 时,残差连接直接把原始输入加到子层输出上,如果子层输出数值很大,加完之后数值会更大,LayerNorm 来不及"救"。

Pre-LN 时,LayerNorm 在子层输入之前,可以先稳定输入分布,让子层更容易学习。输出之后再加残差,数值已经被 LayerNorm 控制住了,更稳定。

**现在基本所有新模型都用 Pre-LN 了。**

<!-- drawio 源文件: ./drawio/transformer_structure_02_pre_post_ln.drawio -->

![Pre-LN与Post-LN](https://file1.kamacoder.com/i/web/20261022110417.png?v=20261005)

## 多层堆叠:从 Block 到 Transformer

一层 Block 搞清楚了,多层 Block 就是简单堆叠:

```
X → Block₁ → Block₂ → Block₃ → ... → Block_N → 输出
```

原始 Transformer Encoder 有 6 层 Block,Llama 3 有 32 层,GPT-4 据说有上百层。

**每一层的输入和输出维度都是 n × d_model,所以可以无限堆叠。**

层数越多,模型的表达能力越强,但训练也越困难。残差连接和 LayerNorm 让 Transformer 能堆到几十上百层还能训练,这是 ResNet 给深度学习带来的最大贡献之一。

## 面试怎么回答

面试官问"一层 Transformer Block 包含哪些组件",你可以这样答:

**第一层:两个子层**

"一层 Transformer Block 包含两个子层:Multi-Head Self-Attention 和 FFN。Self-Attention 让 Token 之间互相交流,聚合全局信息;FFN 对每个 Token 做非线性变换,深化特征表示。"

**第二层:两层包装**

"每个子层外面都包裹着残差连接和 LayerNorm,叫做 Add & Norm。残差连接防止梯度消失,LayerNorm 防止数值爆炸。完整流程是:Self-Attention → Add & Norm → FFN → Add & Norm。"

**第三层:Pre-LN vs Post-LN**

"原始 Transformer 用的是 Post-LN,先残差后 LayerNorm。现代大模型改成了 Pre-LN,先 LayerNorm 再子层再残差,训练更稳定、收敛更快,现在基本都用 Pre-LN 了。输入和输出维度都是 n × d_model,所以可以堆叠很多层。"

**能说到第三层,面试官会觉得你不仅懂结构,还懂演进趋势。**

## 知识拓展

### 1. 为什么 Self-Attention 在前,FFN 在后?能不能反过来?

可以,但没必要。Self-Attention 先聚合全局信息,FFN 再对每个 Token 深化特征,这个顺序更符合直觉:先"看"再"想"。

实践中也有研究尝试过反过来,效果差不多,但没有明显优势,所以大家还是沿用了原始的顺序。

### 2. 一层 Block 的参数量有多少?

以 d_model = 512、h = 8、d_ff = 2048 为例:

- **Self-Attention**:W_Q, W_K, W_V, W_O,每个 512×512,共 4 × 512² ≈ 1M
- **FFN**:W_1(512×2048)+ W_2(2048×512),共 2 × 512 × 2048 ≈ 2M
- **LayerNorm**:γ, β,每个 512,共 2 × 512 ≈ 1K(可忽略)

**总参数量约 3M。** FFN 的参数量大约是 Self-Attention 的 2 倍,是 Transformer 里参数量最大的部分。

### 3. Transformer Decoder 的 Block 和 Encoder 的 Block 有什么区别?

Decoder Block 多了一层 **Cross-Attention**:

```
Self-Attention → Add & Norm → Cross-Attention → Add & Norm → FFN → Add & Norm
```

Cross-Attention 的 Query 来自 Decoder,Key 和 Value 来自 Encoder,让 Decoder 能"看到"Encoder 的输出。

现代大模型(GPT、Llama)基本都是 Decoder-only,没有 Cross-Attention,只有 Self-Attention + FFN。

### 4. 为什么输入输出维度要保持一致?

如果维度变了,就不能堆叠多层了。残差连接要求输入和输出维度一致,才能直接相加。如果维度不一致,残差连接就做不了,或者需要额外的投影矩阵,增加复杂度。

### 5. 一层 Block 的计算复杂度是多少?

以序列长度 n、模型维度 d 为例:

- **Self-Attention**:O(n² · d)——n×n 的权重矩阵,每个格子要算 d 维的点积
- **FFN**:O(n · d²)——每个 Token(n 个)都要做 d×d_ff 和 d_ff×d 的矩阵乘法

当 n < d 时,FFN 是瓶颈;当 n > d 时(长序列),Self-Attention 是瓶颈。这就是为什么长上下文优化主要在优化 Attention 的复杂度(稀疏 Attention、线性 Attention)。

## 本文小结

一层 Transformer Block,本质上就是两个子层的叠加:

- **Self-Attention 子层**:Token 之间互相"交流",聚合全局信息
- **FFN 子层**:对每个 Token 独立做深层非线性变换

两个子层都由相同的外壳包裹:

**子层输出 → 残差连接 → 层归一化**

这套组合,简洁、稳定、可扩展,是 Transformer 能被堆到几十上百层的根本原因。

**面试官追问的都是细节**:组件的顺序是什么?Pre-LN 和 Post-LN 的区别是什么?为什么输入输出维度要一致?这些问题的答案不在概念里,在结构和工程权衡里。

把每一步的数据流、维度变化、设计目的讲清楚,面试官才会觉得你真的懂 Transformer Block,而不是背概念。

## 本阶段小结

到这里,Transformer 的核心原理我们就讲完了。从 Attention 的直觉,到 QKV 的计算,到多头注意力,到位置编码,到残差连接、LayerNorm、FFN,最后把所有组件串成一个完整的 Block。

**这些是面试中最常被问到的 Transformer 原理题。** 能把这些组件的设计动机、计算过程、工程权衡讲清楚,面试官会觉得你真的懂 Transformer,而不是背概念。

下一章我们会进入**手撕 Transformer** 阶段,用最简代码实现每个组件,加深理解,面试手撕有底气。点个关注不迷路～
