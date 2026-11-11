---
title: 手撕Transformer Block：Attention+FFN+Norm拼成完整Block
description: 手撕Transformer Block代码实现，将Multi-Head Attention、FFN、LayerNorm和残差连接组装成一个完整Block，逐步理解Block内部MHA→Add&Norm→FFN→Add&Norm数据流转、维度不变原则和前向计算逻辑，掌握为什么残差和Norm缺一不可以及面试手撕Transformer Block的代码结构，适合Transformer原理学习和大模型应用开发面试。
keywords: [手撕Transformer Block, Transformer Block代码, Transformer组件组装, 残差连接, LayerNorm, Add&Norm, 面试手撕代码, Transformer面试题, 大模型面试]
tags: [Transformer, 手撕代码, 面试]
---

# 手撕Transformer Block：Attention+FFN+Norm拼成完整Block

前面几篇文章，我们已经把 Transformer Block 里的核心零件都手撕了一遍：

- [Multi-Head Attention](./mha_code.md)：让不同 Token 之间互相交流
- [FFN](./fnn_code.md)：让每个 Token 独立做非线性加工
- [残差连接与 LayerNorm](./layernorm_residual_code.md)：把输入加回来防止梯度消失，把数值拉回稳定范围

但面试官会继续问：**"这些组件怎么拼起来？为什么是 Add&Norm，不是 Norm&Add？残差能不能省？"**

常见回答是："Attention 后面接 FFN，中间加残差和 Norm。"

顺序说对了，但只是在描述结构图。面试官要听的是**每个组件为什么在那个位置**：残差为什么必须在 Norm 之前或绕开 Norm，形状为什么全程不变。说不出这些，就只是背了张图。

这篇把前面写好的零件拼成一个完整的 Transformer Block，跑通前向过程。

## 简要回答

- **两个子层**：Multi-Head Attention 和 FFN，每个子层后面都接残差加法和 LayerNorm
- **数据流**：x → MHA → Add&Norm → FFN → Add&Norm，Post-Norm 写法是 LayerNorm(x + F(x))
- **形状不变**：每个子层输入输出都是 (L, d_model)，残差才能相加，Block 才能堆叠
- **分工**：子层负责改信息，残差保证梯度传得回去，LayerNorm 稳住数值

**一句话：Block 没有新零件，难点在顺序和形状，拼对了才能一层层往上堆。**

## 一个 Block 到底长什么样

最常见的 Transformer Block（Post-Norm）：

```
输入 x (L, d_model)
  ↓
①Multi-Head Attention
  ↓
②残差连接：x + MHA(x)
  ↓
③LayerNorm
  ↓
④FFN
  ↓
⑤残差连接：x1 + FFN(x1)
  ↓
⑥LayerNorm
  ↓
输出 (L, d_model)
```

公式：

$$
x_1 = \text{LayerNorm}(x + \text{MHA}(x))
$$

$$
x_2 = \text{LayerNorm}(x_1 + \text{FFN}(x_1))
$$

**关键点**：

- **输入输出形状一致**：x.shape = (L, d_model) → output.shape = (L, d_model)
- **两个子层**：每个子层都是 子层计算 → Add(残差) → Norm
- **维度不变**：MHA 输出 (L, d_model)，FFN 输出 (L, d_model)

**面试官问"为什么是这个顺序"，答案是：子层计算可能改变数值分布，先加残差保留原始信息，再 Norm 稳定数值，这是 Post-Norm 的设计，另一种 Pre-Norm 见知识拓展。**

![Transformer Block结构](https://file1.kamacoder.com/i/web/20261030144718.png?v=20261005)


两条绿色粗线就是残差路径，分别绕过 MHA 和 FFN 直达加法节点。每个箭头旁都写着 7 × 8，只有 FFN 内部临时升到 32 维。右侧三个小 Block 竖排，说明正因为形状不变，Block 才能一层层往上叠。

## 完整代码

直接给出完整的 Transformer Block 实现，每个组件都是前面手撕过的：

```python
import numpy as np

class LayerNorm:
    def __init__(self, d_model, eps=1e-5):
        self.gamma = np.ones(d_model)
        self.beta = np.zeros(d_model)
        self.eps = eps

    def forward(self, x):
        mean = x.mean(axis=-1, keepdims=True)
        std = x.std(axis=-1, keepdims=True)
        x_norm = (x - mean) / (std + self.eps)
        return self.gamma * x_norm + self.beta


class FeedForward:
    def __init__(self, d_model, d_ff):
        self.W1 = np.random.randn(d_model, d_ff) * 0.01
        self.b1 = np.zeros(d_ff)
        self.W2 = np.random.randn(d_ff, d_model) * 0.01
        self.b2 = np.zeros(d_model)

    def gelu(self, x):
        return 0.5 * x * (1 + np.tanh(
            np.sqrt(2 / np.pi) * (x + 0.044715 * x ** 3)
        ))

    def forward(self, x):
        hidden = x @ self.W1 + self.b1
        hidden = self.gelu(hidden)
        output = hidden @ self.W2 + self.b2
        return output


class MultiHeadAttention:
    def __init__(self, d_model, num_heads):
        assert d_model % num_heads == 0
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        self.W_Q = np.random.randn(d_model, d_model) * 0.01
        self.W_K = np.random.randn(d_model, d_model) * 0.01
        self.W_V = np.random.randn(d_model, d_model) * 0.01
        self.W_O = np.random.randn(d_model, d_model) * 0.01

    def softmax(self, x):
        e = np.exp(x - np.max(x, axis=-1, keepdims=True))
        return e / e.sum(axis=-1, keepdims=True)

    def forward(self, x):
        L = x.shape[0]

        # 线性映射
        Q = x @ self.W_Q
        K = x @ self.W_K
        V = x @ self.W_V

        # 拆分成多头：(L, d_model) → (h, L, d_k)
        Q = Q.reshape(L, self.num_heads, self.d_k).transpose(1, 0, 2)
        K = K.reshape(L, self.num_heads, self.d_k).transpose(1, 0, 2)
        V = V.reshape(L, self.num_heads, self.d_k).transpose(1, 0, 2)

        # 每个头独立计算 Attention
        head_outs = []
        for i in range(self.num_heads):
            scores = Q[i] @ K[i].T / np.sqrt(self.d_k)
            attn = self.softmax(scores) @ V[i]
            head_outs.append(attn)

        # 拼接：(h, L, d_k) → (L, d_model)
        concat = np.stack(head_outs, axis=0).transpose(1, 0, 2).reshape(L, self.d_model)

        # 输出投影
        output = concat @ self.W_O
        return output


class TransformerBlock:
    def __init__(self, d_model, num_heads, d_ff):
        """
        Transformer Block (Post-Norm)

        参数:
            d_model: embedding 维度
            num_heads: 注意力头数
            d_ff: FFN 中间层维度
        """
        self.mha = MultiHeadAttention(d_model, num_heads)
        self.ffn = FeedForward(d_model, d_ff)
        self.norm1 = LayerNorm(d_model)
        self.norm2 = LayerNorm(d_model)

    def forward(self, x):
        # ① MHA + 残差 + Norm
        attn_out = self.mha.forward(x)
        x1 = self.norm1.forward(x + attn_out)  # 残差连接
        print(f"[MHA → Add&Norm] {x.shape} → {x1.shape}")

        # ② FFN + 残差 + Norm
        ffn_out = self.ffn.forward(x1)
        x2 = self.norm2.forward(x1 + ffn_out)  # 残差连接
        print(f"[FFN → Add&Norm] {x1.shape} → {x2.shape}")

        return x2


# 运行测试
if __name__ == "__main__":
    np.random.seed(42)
    L, d_model, num_heads, d_ff = 7, 8, 2, 32

    x = np.random.randn(L, d_model)
    block = TransformerBlock(d_model, num_heads, d_ff)

    print("=== Transformer Block ===")
    output = block.forward(x)

    print(f"\n输入形状: {x.shape}")
    print(f"输出形状: {output.shape}")
    print(f"形状一致: {x.shape == output.shape}")
```

**运行输出：**
```
=== Transformer Block ===
[MHA → Add&Norm] (7, 8) → (7, 8)
[FFN → Add&Norm] (7, 8) → (7, 8)

输入形状: (7, 8)
输出形状: (7, 8)
形状一致: True
```

**面试时怎么讲这段代码？**

按两个子层串起来：

1. **第一个子层**：MHA(x) → x + MHA(x) → LayerNorm → x1
2. **第二个子层**：FFN(x1) → x1 + FFN(x1) → LayerNorm → x2

重点说清楚**残差的作用**：每个子层都有一个恒等路径，梯度可以直接从后往前传，不经过 MHA 或 FFN 的复杂计算，这就是深度网络能训起来的关键。

## 知识拓展

**Q1：为什么是 Add&Norm，不是 Norm&Add？**

这是 Post-Norm（加后归一化）的设计。还有一种是 Pre-Norm（加前归一化）：

```
# Post-Norm（原始 Transformer）
x1 = LayerNorm(x + MHA(x))

# Pre-Norm（现代常用）
x1 = x + MHA(LayerNorm(x))
```

**Pre-Norm 更稳定，训练更容易；Post-Norm 表达能力略强，但训练难度高。** 现代大模型（Llama、Qwen）基本都用 Pre-Norm。

**Q2：残差连接能不能省？**

不能。没有残差，深层网络会梯度消失。残差连接让梯度有一条"高速公路"，可以直接从输出层传到输入层，不经过中间复杂的计算。**这是 Transformer 能堆 12 层、24 层、甚至 96 层的基础。**

**Q3：LayerNorm 能不能换成 BatchNorm？**

不推荐。BatchNorm 在 NLP 任务上效果不如 LayerNorm，因为：

- BatchNorm 需要在 batch 维度算统计量，序列长度不同时不稳定
- LayerNorm 在特征维度算统计量，和 batch size、序列长度无关

现代 Transformer 基本都用 LayerNorm 或它的变体（RMSNorm）。

**Q4：Transformer Block 的参数量是多少？**

一层 Block 的参数量：

- MHA：4 × d_model² （W_Q, W_K, W_V, W_O）
- FFN：8 × d_model²（d_ff = 4 × d_model 时）
- LayerNorm：2 × d_model × 2（两个 LayerNorm，每个有 gamma 和 beta）

**总计约 12 × d_model²**，其中 FFN 占大头（67%）。

**Q5：多层 Block 怎么堆叠？**

就是把 Block 串起来：

```python
x = embedding(tokens)
for block in blocks:
    x = block.forward(x)
output = x
```

每层的输入输出都是 (L, d_model)，所以可以无限堆叠。BERT-base 堆 12 层，GPT-3 堆 96 层。

**Q6：面试时手撕 Transformer Block 要写到什么程度？**

**能写出 TransformerBlock 类就够了**，100 行以内。重点是：

- MHA → Add&Norm
- FFN → Add&Norm
- 每步都保持 (L, d_model) 不变

每个组件不用重新实现，直接说"调用前面手撕的 MHA/FFN/LayerNorm"就行。面试官看你能不能把组件拼对，不是考你能不能从零写一遍。

Block 的代码没有新东西，全是前面写过的零件。

**难点在拼的顺序和形状：每个子层输出都回到 (L, d_model)，残差才能加，Block 才能堆。** 讲清这一点，面试官就知道你不是在背图。
