---
title: 手撕LayerNorm与残差连接：Add&Norm、Post-Norm与Pre-Norm代码实现
description: 手撕LayerNorm和残差连接代码实现，讲清残差连接为什么能缓解深层网络梯度消失，LayerNorm按Token计算均值方差、epsilon稳定项、可学习参数gamma和beta，以及Add&Norm、Post-Norm与Pre-Norm的代码差异和训练稳定性取舍，对比BatchNorm和RMSNorm，适合Transformer手撕代码和大模型面试复习。
keywords: [手撕LayerNorm, LayerNorm代码实现, 残差连接代码, Residual Connection, Add&Norm, Pre-Norm, Post-Norm, BatchNorm, RMSNorm, 梯度消失, 面试手撕代码, Transformer面试题, 大模型面试]
tags: [Transformer, 手撕代码, 面试]
---

# 手撕LayerNorm与残差连接：Add&Norm、Post-Norm与Pre-Norm代码实现

上一篇[《手撕FFN》](./fnn_code.md)把前馈网络拆成了升维、激活、降维三步。到这里，Transformer Block 里负责"干活"的两个子层，Attention 和 FFN，都写完了。

但只有这两个子层，模型堆不深。面试官常问：**"残差连接和 LayerNorm 各自解决什么问题？Pre-Norm 和 Post-Norm 有什么区别？"**

很多录友会答："残差防止梯度消失，LayerNorm 做归一化。"

方向没错，但太粗。面试官追问"LayerNorm 在哪个维度上算均值""为什么现在大模型都用 Pre-Norm"，就接不上了。**这两个组件代码加起来不到 20 行，考的是你知不知道每一行在稳定什么。**

## 简要回答

- **残差连接**：输出 = x + F(x)，给梯度一条不经过子层的直通路径，深层网络才训得动
- **LayerNorm**：对每个 Token 自己的向量算均值和标准差，拉回均值 0、方差 1，再用 gamma、beta 缩放偏移
- **Add&Norm**：原始 Transformer 每个子层后面都是 LayerNorm(x + F(x))，即 Post-Norm
- **Pre-Norm**：x + F(LayerNorm(x))，残差路径上没有 Norm，深层训练更稳，现代大模型基本都用它

**一句话：残差负责让梯度传得回去，LayerNorm 负责让数值不乱跑。**

## 残差连接为什么一个加法就够了

残差连接的公式只有一行：

$$
\text{输出} = x + F(x)
$$

F(x) 是子层的输出（Attention 或 FFN），x 是这一层的输入，两者直接相加。

它解决的是深层网络的**梯度消失**。反向传播时，梯度要从最后一层一路传回第一层。没有残差，每穿过一层就要乘一次这层的导数，几十层乘下来，前面的层几乎收不到训练信号。

加了残差，对 x 求导时多出一个恒等项：

$$
\frac{\partial\, (x + F(x))}{\partial x} = 1 + \frac{\partial F(x)}{\partial x}
$$

**这个 1 就是梯度的直通路径。** 不管 F 的导数多小，梯度至少能原样传回去一份。

![残差连接与梯度](https://file1.kamacoder.com/i/web/20261030144713.png?v=20261005)


上面红色梯度条每过一层减半，传到第 1 层只剩 0.03。下面绿色的绕行线就是残差路径，梯度沿着它直接回到前面，长度基本不变。数值是示意，真实网络的导数不会都一样，但"连乘会衰减"的趋势是一致的。

代码就是一个加法：

```python
import numpy as np

def residual_connection(x, sublayer_output):
    """x 和 sublayer_output 形状都是 (L, d_model)"""
    return x + sublayer_output

L, d_model = 7, 8
np.random.seed(42)

x = np.random.randn(L, d_model)             # 子层输入
sublayer_out = np.random.randn(L, d_model)  # 模拟子层输出

output = residual_connection(x, sublayer_out)
print(f"输入形状:   {x.shape}")
print(f"子层输出:   {sublayer_out.shape}")
print(f"残差输出:   {output.shape}")
```

**输出：**
```
输入形状:   (7, 8)
子层输出:   (7, 8)
残差输出:   (7, 8)
```

**残差有一个硬约束：x 和 F(x) 形状必须一致。** 这也是为什么 Attention 和 FFN 都要把输出投影回 d_model。

## LayerNorm 到底在哪个维度上归一化

残差相加之后，数值范围会越叠越乱：有的维度很大，有的很小。层数一多，下一层的输入分布一直在漂，训练就不稳。

LayerNorm 的做法是：**每个 Token 只看自己那一行向量**，算均值和标准差，把它拉回均值 0、方差 1。

$$
\text{LayerNorm}(x) = \gamma \cdot \frac{x - \mu}{\sigma + \epsilon} + \beta
$$

- μ、σ：这个 Token 向量在 d_model 个维度上的均值和标准差
- ε：很小的数（如 1e-5），防止除零
- γ、β：可学习参数，形状 (d_model,)，让模型自己决定归一化后要不要再放大、平移

![LayerNorm与BatchNorm](https://file1.kamacoder.com/i/web/20261030144715.png?v=20261005)


左边绿色那一行就是 LayerNorm 的统计范围：只用「有」自己的 8 个维度。右边橙色那一列是 BatchNorm 的统计范围，要跨 batch 里所有样本的同一维度去算。文本长度不一、推理时 batch 常常是 1，BatchNorm 的统计量很难靠谱，所以 Transformer 选了 LayerNorm。

代码里最关键的是 `axis=-1`，它决定了按行算：

```python
class LayerNorm:
    def __init__(self, d_model, eps=1e-5):
        self.gamma = np.ones(d_model)   # 可学习缩放，初始化为 1
        self.beta  = np.zeros(d_model)  # 可学习偏移，初始化为 0
        self.eps   = eps

    def forward(self, x):
        # x: (L, d_model)，每个 Token 单独算统计量
        mu    = x.mean(axis=-1, keepdims=True)   # (L, 1)
        sigma = x.std(axis=-1, keepdims=True)    # (L, 1)
        x_norm = (x - mu) / (sigma + self.eps)   # (L, d_model)
        return self.gamma * x_norm + self.beta


np.random.seed(42)
L, d_model = 7, 8
x = np.random.randn(L, d_model) * 10  # 故意放大，模拟数值失控

ln = LayerNorm(d_model)
output = ln.forward(x)

print(f"归一化前 - 均值: {x.mean():.2f}, 标准差: {x.std():.2f}")
print(f"归一化后 - 均值: {output.mean():.4f}, 标准差: {output.std():.4f}")
print(f"输入形状: {x.shape}, 输出形状: {output.shape}")
```

**输出：**
```
归一化前 - 均值: -1.69, 标准差: 9.13
归一化后 - 均值: -0.0000, 标准差: 1.0000
输入形状: (7, 8), 输出形状: (7, 8)
```

标准差从 9 左右拉回到 1，形状不变。**`keepdims=True` 不能漏**，否则 (L,) 和 (L, d_model) 广播会出错，这是手撕时最常见的 bug。

## Add&Norm 和 Pre-Norm 差在哪一行

原始 Transformer 把两者捆在一起，叫 **Add&Norm**，也就是 Post-Norm：先加残差，再归一化。

```python
# Post-Norm（原始 Transformer）
def post_norm_sublayer(x, sublayer_fn, layer_norm):
    return layer_norm.forward(x + sublayer_fn(x))

# Pre-Norm（GPT-2 之后的主流大模型）
def pre_norm_sublayer(x, sublayer_fn, layer_norm):
    return x + sublayer_fn(layer_norm.forward(x))
```

两种写法只差 LayerNorm 的位置，但影响的是残差路径是否干净：

- **Post-Norm**：每层输出都要过一次 LayerNorm，残差路径上也被 Norm 截了一刀，层数深时梯度不稳，通常要靠学习率 warmup 才训得起来
- **Pre-Norm**：Norm 只作用在子层输入上，x 从第一层到最后一层一路直加，恒等路径完整，深层训练稳得多

![Post-Norm与Pre-Norm](https://file1.kamacoder.com/i/web/20261030144716.png?v=20261005)


左边残差线最后还要穿过 LayerNorm，这一刀就是 Post-Norm 深层不稳的来源。右边 x 绕开所有 Norm 一路直加，所以 Pre-Norm 结构的最后一层后面还要补一个 Final Norm，把累加起来的数值收一下。

把 Pre-Norm 跑一遍，确认形状不变：

```python
np.random.seed(42)
L, d_model = 7, 8
x = np.random.randn(L, d_model)
W_fake = np.random.randn(d_model, d_model) * 0.1
fake_sublayer = lambda h: h @ W_fake    # 用一个线性层代替 Attention / FFN

ln1, ln2, final_ln = LayerNorm(d_model), LayerNorm(d_model), LayerNorm(d_model)

h = pre_norm_sublayer(x, fake_sublayer, ln1)   # Attention 子层位置
h = pre_norm_sublayer(h, fake_sublayer, ln2)   # FFN 子层位置
out = final_ln.forward(h)                      # Pre-Norm 结构最后补一个 Norm
print(f"输入: {x.shape}  输出: {out.shape}")
```

**输出：**
```
输入: (7, 8)  输出: (7, 8)
```

**面试官问"为什么现代大模型用 Pre-Norm"，答案是：恒等路径不被 Norm 打断，梯度更容易传到底层，深层训练更稳定，对学习率 warmup 没那么敏感。** 代价是同样层数下最终效果可能略逊于调好的 Post-Norm，但大模型更看重能稳定训完。

## 知识拓展

**Q1：LayerNorm 的 γ、β 有什么用，能不能去掉？**

归一化把每个 Token 强行拉到均值 0、方差 1，可能把有用的尺度信息也抹掉了。γ、β 让模型自己学"要不要恢复一些尺度和偏移"。极端情况下 γ = σ、β = μ，就等于没归一化。不建议去掉，参数量只有 2 × d_model，几乎不占资源。

**Q2：ε 为什么要加？加在哪里？**

如果某个 Token 的向量各维几乎相等，σ 接近 0，除下去会数值爆炸。ε 一般取 1e-5 或 1e-6。PyTorch 的 `nn.LayerNorm` 写法是 `(x - μ) / sqrt(var + ε)`，本文为了好读写成 `σ + ε`，两者在 ε 很小时几乎没差别，但面试时最好说出官方写法。

**Q3：RMSNorm 和 LayerNorm 差在哪？**

RMSNorm 去掉了减均值这一步，也去掉了 β，只按均方根缩放：x / sqrt(mean(x²) + ε) · γ。计算更少，效果接近，Llama、Qwen 都用它。**原理上是"只管尺度，不管中心"。**

**Q4：残差连接要求什么？**

x 和 F(x) 形状必须完全一致才能相加。所以 Multi-Head Attention 最后要乘 W_O 投影回 d_model，FFN 升维后也要降回 d_model。手撕时形状对不上，往往就是漏了这一步。

**Q5：Dropout 一般放在哪？**

原始 Transformer 放在子层输出、加残差之前：x + Dropout(F(x))。推理时 Dropout 关闭。很多大模型预训练时干脆把 Dropout 设为 0，数据量足够大时过拟合不是主要矛盾。

**Q6：面试手撕要写到什么程度？**

**LayerNorm 类 + Pre-Norm 和 Post-Norm 两个函数，15 行以内。** 重点是 `axis=-1`、`keepdims=True`、ε 的位置，再能说清两种 Norm 摆法对残差路径的影响，就够了。

很多录友把残差和 LayerNorm 当成"顺手加上的配角"，面试时一句带过。

**真正拉开差距的，是能讲清那个"1"怎么让梯度传回去，以及 Norm 挪一格为什么就能让模型堆得更深。**
