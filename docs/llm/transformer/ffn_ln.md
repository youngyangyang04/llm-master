---
title: 残差连接、LayerNorm、FFN:Transformer里缺一不可的配角组件
description: Transformer中的残差连接、LayerNorm和FFN为什么缺一不可?本文讲清残差连接如何缓解深层网络训练困难,LayerNorm如何稳定激活分布,FFN如何增强Token表示能力,以及这些组件在Transformer Block和大模型面试中的常见追问和回答方法。
keywords: ["残差连接", "LayerNorm", "FFN", "Transformer组件", "前馈网络", "层归一化", "Add&Norm"]
tags: ["Transformer", "核心组件", "大模型原理"]
---

# 残差连接、LayerNorm、FFN:Transformer里缺一不可的配角组件

上一篇[《位置编码详解:Transformer为什么必须知道Token顺序,正弦编码原理》](./pos_encode.md)讲了位置编码,解决了 Self-Attention 不感知顺序的问题。

但 Transformer 能真正训练起来,还得靠三个经常被忽略的"配角":**残差连接(Residual Connection)、LayerNorm 和前馈网络(FFN)**。

面试官一定会问:**残差连接解决了什么问题?为什么 Self-Attention 之后要加残差?LayerNorm 和 BatchNorm 有什么区别?FFN 为什么要先升维再降维?**

这些问题的答案,藏在深层网络训练的工程挑战里。

## 面试官最爱问的一个追问

"Transformer 为什么要加残差连接和 LayerNorm?"

很多录友会答:

> 残差连接可以防止梯度消失,LayerNorm 可以加速收敛。

面试官接着问:"怎么防止的?为什么用 LayerNorm 而不是 BatchNorm?残差加在 Self-Attention 之前还是之后?"

答不上来。

问题出在哪?**你只知道"有这些组件",但没搞清楚它们解决的具体问题是什么、不加会怎样、为什么这么设计。** 面试官追问的都是工程直觉——深层网络训练难在哪?残差和 LayerNorm 分别解决了什么?这些问题的答案藏在 Transformer 的训练过程里。

## 深层网络的两个老大难问题

Transformer 是一个很深的网络——原始论文里堆了 6 层 Encoder,每层里有 Self-Attention 和 FFN 两个子模块。现代大模型更深,Llama 3 有 32 层,GPT-4 据说有上百层。

信息每过一层,都要经历一次变换。问题是:**层数一多,训练就容易崩。**

### 问题1:梯度消失

反向传播时,梯度要从输出层一路传回输入层。每经过一层,梯度都要乘一次这一层的导数。如果导数小于 1,梯度就会越乘越小,最终趋近于 0。

**梯度消失的后果:前面几层的参数几乎不更新,模型学不到东西。**

### 问题2:数值爆炸

每一层的输出,都是上一层输出的变换结果。如果变换的数值范围越来越大,后面的层就会收到极大或极小的输入,激活函数进入饱和区,梯度又消失了。

**数值爆炸的后果:激活值分布不稳定,模型训练不稳定甚至崩溃。**

残差连接和 LayerNorm 的存在,就是专门为了解决这两个问题的。

## 残差连接:给梯度留一条高速公路

先看这个场景:

> 一首歌经过了混音、母带处理、压缩…… 每个步骤都在"加工"原声。
> 如果每一步都把上一步的结果完全覆盖掉,原声就消失了。

深层网络也是一样——每一层都在对输入做变换,如果变换出了问题(比如梯度消失),后面的层就完全收不到有效信息了。

残差连接的解法非常直接:**把输入直接加到输出上,绕过中间的变换。**

$$
\text{输出} = F(x) + x
$$

其中 F(x) 是这一层学到的变换(比如 Self-Attention 的结果),x 是这层的原始输入。

<!-- drawio 源文件: ./drawio/ffn_ln_01_residual.drawio -->

![残差连接梯度流](https://file1.kamacoder.com/i/web/20261022110353.png?v=20261005)

这步加法带来了两个好处:

### 好处1:梯度可以直接回流

反向传播时,梯度不再只能穿过 F(x) 这条路,还能沿着 x 这条捷径直接流回去。

$$
\frac{\partial \text{输出}}{\partial x} = \frac{\partial F(x)}{\partial x} + 1
$$

注意右边那个"+1"——这意味着**梯度至少有一条路能畅通无阻地传回去**,就算 F(x) 这条路梯度消失了,信息也不会断。

### 好处2:层可以学"修正量"而不是"全量"

没有残差时,每一层都要从零学一个全新的变换,压力很大。
加了残差后,这一层只需要学:**在上一层输出的基础上,还需要修正哪里?**

如果这一层学不到什么有用的东西,F(x) 可以趋近于 0,输出就约等于输入 x,相当于这一层"跳过"了,不会破坏前面层学到的信息。

**核心逻辑:残差连接给梯度开了一条高速公路,让深层网络能正常训练。**

## LayerNorm:把数值拉回合理范围

残差连接解决了梯度流动的问题,但还有另一个隐患:**数值爆炸**。

把 F(x) 和 x 加在一起之后,数值的范围可能变得很大、很不均匀。进入下一层时,有些维度的值极大,有些极小,模型很难稳定地学习。

LayerNorm 就是做这件事的:**把每个 Token 的向量重新规范化,让它的均值为 0、方差为 1。**

$$
\text{LayerNorm}(x) = \frac{x - \mu}{\sigma} \cdot \gamma + \beta
$$

- μ 和 σ 是这个向量自身的均值和标准差
- γ 和 β 是可学习的缩放和偏移参数,让模型可以自己决定"规范到什么程度"

**核心逻辑:LayerNorm 把每层的输出拉回均值 0、方差 1 附近,防止数值爆炸,让后续计算更稳定。**

### LayerNorm vs BatchNorm

很多录友容易把 LayerNorm 和 BatchNorm 搞混,它们的区别是:

| | BatchNorm | LayerNorm |
|---|---|---|
| **归一化维度** | 对一个 batch 内的同一特征归一化 | 对单个样本的所有特征归一化 |
| **依赖 batch 大小** | 是,batch 太小效果差 | 否,每个样本独立计算 |
| **适用场景** | CNN、batch 大的任务 | RNN、Transformer、batch 小的任务 |
| **训练和推理的区别** | 训练时用 batch 统计,推理时用全局统计 | 训练和推理完全一致 |

**为什么 Transformer 用 LayerNorm 而不是 BatchNorm?**

因为 NLP 任务的 batch 通常很小(比如 8 或 16),而且序列长度不固定,BatchNorm 的统计量不稳定。LayerNorm 对每个样本独立计算,不依赖 batch,更适合 Transformer。

<!-- drawio 源文件: ./drawio/ffn_ln_02_ln_bn_compare.drawio -->

![BatchNorm与LayerNorm](https://file1.kamacoder.com/i/web/20261022110354.png?v=20261005)

## Add & Norm:两者组合

在 Transformer 里,残差和 LayerNorm 是**捆绑使用**的,标准写法叫 **Add & Norm**:

$$
\text{输出} = \text{LayerNorm}(x + F(x))
$$

先残差相加,再 LayerNorm 归一化,每个子模块(Self-Attention 和 FFN)后面都跟一个这样的结构。

**这就是为什么 Transformer 能堆到几十上百层还能训练——残差连接保证梯度畅通,LayerNorm 保证数值稳定。**

## FFN:Self-Attention 的"搭档",不是装饰

Transformer 每一层的结构是:

```
Self-Attention → Add & Norm → FFN → Add & Norm
```

Self-Attention 负责让 Token 之间"互通信息",FFN 负责什么?

很多初学者觉得 FFN 就是两层普通的线性变换,可有可无。但它其实在做一件 Self-Attention 做不了的事:**对每个 Token 的向量做非线性变换,提取更深层的特征。**

FFN 的结构很简单:

$$
\text{FFN}(x) = \text{GELU}(x \cdot W_1 + b_1) \cdot W_2 + b_2
$$

两层线性变换,中间夹一个激活函数(ReLU 或 GELU)。

### 关键细节:FFN 会先升维,再降维

以 d_model = 512 为例:

```
输入 x(512维)
    ↓ × W₁(512→2048)
中间层(2048维)← 升维到 4 倍
    ↓ 激活函数(GELU)
    ↓ × W₂(2048→512)
输出(512维)
```

### 为什么要先升维再降维?

这里藏着 FFN 最重要的设计思路。

Self-Attention 做的是**线性组合**——把不同 Token 的 Value 向量加权求和,本质上是线性变换。**线性变换叠加再多层,表达能力还是有限的。**

FFN 中间插入了激活函数,带来了**非线性**。先升维,是为了在更高维的空间里让激活函数"发挥作用"——高维空间里,非线性能捕捉到更复杂的特征组合。再降维,把提取到的信息压缩回原来的维度,传给下一层。

**核心逻辑:Self-Attention 是线性的,FFN 补上非线性能力,两者配合才能学到复杂的语言模式。**

### FFN 是对每个 Token 独立计算的

还有一点值得注意:**FFN 是对每个 Token 独立计算的**,不同 Token 之间不共享中间结果。

这和 Self-Attention 正好相反——Self-Attention 让所有 Token 互相"看",FFN 让每个 Token 自己"想"。

| | Self-Attention | FFN |
|---|---|---|
| **处理方式** | Token 之间互相交互 | 每个 Token 独立处理 |
| **变换类型** | 线性(加权求和) | 非线性(含激活函数) |
| **作用** | 聚合上下文信息 | 深化单个 Token 的表示 |
| **参数量** | W_Q, W_K, W_V, W_O | W_1, W_2 |

两者一个负责"横向"(Token 间)的信息交流,一个负责"纵向"(特征维度)的深度加工,**缺一不可**。

<!-- drawio 源文件: ./drawio/ffn_ln_03_attention_ffn_compare.drawio -->

![Attention与FFN对比](https://file1.kamacoder.com/i/web/20261022110356.png?v=20261005)

## 面试怎么回答

面试官问"Transformer 为什么要加残差连接和 LayerNorm",你可以这样答:

**第一层:解决的问题**

"深层网络有两个老大难问题:梯度消失和数值爆炸。梯度消失会导致前面几层学不到东西,数值爆炸会导致激活值分布不稳定、训练崩溃。残差连接和 LayerNorm 就是专门解决这两个问题的。"

**第二层:怎么解决**

"残差连接把输入直接加到输出上,给梯度开了一条高速公路,就算中间的变换梯度消失了,梯度也能沿着跳跃连接直接传回去。LayerNorm 把每个 Token 的向量归一化到均值 0、方差 1,防止数值爆炸,让后续计算更稳定。"

**第三层:工程权衡**

"Transformer 用 LayerNorm 而不是 BatchNorm,是因为 NLP 任务的 batch 通常很小、序列长度不固定,BatchNorm 的统计量不稳定。LayerNorm 对每个样本独立计算,不依赖 batch,更适合 Transformer。实际项目里,如果训练不稳定,先检查残差和 LayerNorm 有没有加对。"

**能说到第三层,面试官会觉得你不仅懂原理,还懂工程权衡。**

## 知识拓展

### 1. 残差连接一定是"加法"吗?能不能用"拼接"?

可以,但加法更常用。拼接(Concat)会增加维度,比如输入是 512 维,变换结果也是 512 维,拼接之后就是 1024 维,参数量和计算量都会翻倍。加法不改变维度,更高效。

而且,加法隐含了一个假设:输入和变换结果是同一个语义空间里的,可以直接相加。实践中这个假设基本成立。

### 2. Pre-LN 和 Post-LN 有什么区别?

原始 Transformer 用的是 **Post-LN**:先残差相加,再 LayerNorm。

$$
\text{输出} = \text{LayerNorm}(x + F(x))
$$

现代大模型(比如 Llama)改成了 **Pre-LN**:先 LayerNorm,再变换,最后残差相加。

$$
\text{输出} = x + F(\text{LayerNorm}(x))
$$

Pre-LN 的好处是:LayerNorm 在变换之前,可以稳定输入分布,让变换更容易学习。实践中 Pre-LN 训练更稳定,收敛更快,现在基本都用 Pre-LN 了。

### 3. FFN 的升维倍数一定是 4 倍吗?

不一定,但 4 倍是经验值。原始 Transformer 用的是 4 倍(512 → 2048 → 512),后来很多模型也沿用了这个设置。

理论上,升维倍数越大,FFN 的表达能力越强,但参数量和计算量也越大。实践中,4 倍是一个性价比较高的选择,再往上提升不明显。

### 4. FFN 为什么用 GELU 而不是 ReLU?

原始 Transformer 用的是 ReLU,后来发现 GELU(Gaussian Error Linear Unit)效果更好,现代大模型基本都用 GELU。

GELU 比 ReLU 更"平滑",不会像 ReLU 那样把负值直接截断为 0,梯度更稳定。但差异不大,两者在实践中效果相近。

### 5. Transformer 里参数量最大的部分是哪里?

是 FFN。Self-Attention 的参数量是 4 × d_model²(W_Q, W_K, W_V, W_O),FFN 的参数量是 2 × d_model × d_ff(W_1, W_2),其中 d_ff = 4 × d_model。

所以 FFN 的参数量大约是 Self-Attention 的 2 倍。现代大模型的参数量大部分都在 FFN 里。

## 本文小结

残差连接、LayerNorm 和 FFN 看起来像是凑数的配角,但它们解决的都是深层网络训练的核心问题:

- **残差连接**:给梯度留一条高速公路,让深层网络能正常训练
- **LayerNorm**:每层计算完之后做数值体检,防止数值爆炸影响下一层
- **FFN**:补上 Self-Attention 缺少的非线性能力,让每个 Token 的表示更丰富

**去掉任何一个,Transformer 都训不动。**

面试官追问的都是工程直觉——深层网络训练难在哪?残差和 LayerNorm 分别解决了什么?Pre-LN 和 Post-LN 的区别是什么?这些问题的答案不在公式里,在训练过程和工程权衡里。

把每一步的设计目的、不同方案的优劣讲清楚,面试官才会觉得你真的懂 Transformer,而不是背概念。

下一篇[《一层Transformer Block长什么样?自注意力、FFN、残差、LayerNorm拼起来》](./transformer_structure.md)我们来把所有核心组件串起来,看看一层完整的 Transformer Block 是怎么运转的,点个关注不迷路～
