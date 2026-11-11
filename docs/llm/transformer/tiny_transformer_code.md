---
title: 手撕Tiny Transformer：从Embedding到输出层跑通完整模型
description: 手撕Tiny Transformer完整代码实现，把Token Embedding、位置编码、多层Transformer Block和输出层拼成一个可运行的最小Transformer模型，理解ID→向量→加位置→多层加工→词表打分的完整数据流，掌握各组件如何协同工作以及面试手撕完整Transformer的代码结构，适合Transformer原理入门和大模型应用开发面试。
keywords: [手撕Transformer, Tiny Transformer, Transformer完整实现, Transformer代码, Token Embedding, 位置编码, 多层Block, 面试手撕代码, Transformer面试题, 大模型面试]
tags: [Transformer, 手撕代码, 面试]
---

# 手撕Tiny Transformer：从Embedding到输出层跑通完整模型

前面几篇文章，我们已经把 Transformer 的核心零件都手撕了一遍：

- [Attention](./attention_code.md)：计算注意力权重
- [Multi-Head Attention](./mha_code.md)：多头并行关注不同语义
- [FFN](./fnn_code.md)：升维、激活、降维
- [残差连接与 LayerNorm](./layernorm_residual_code.md)：稳住深层训练
- [Transformer Block](./transformer_block_code.md)：Attention+FFN+Norm 拼成一层

但面试官会继续问：**"这些组件怎么拼成一个完整的模型？Embedding 在哪里？位置编码怎么加？输出层怎么接？"**

很多录友只会写 Block，问到首尾就卡住：位置编码是拼接还是相加说不准，输出层的形状也说不清。**一个完整模型只有首尾两步改变形状**，这两步答不上来，就没法说自己写出了 Transformer。

这篇把所有零件接成一个最小可运行的 Tiny Transformer，从 Token ID 一路跑到 logits。

## 简要回答

- **Embedding**：整数 ID 查表变成 (L, d_model) 的向量
- **位置编码**：生成同形状的位置矩阵，和 Embedding 相加，不是拼接
- **多层 Block**：每层输入输出都是 (L, d_model)，想堆几层就堆几层
- **输出层**：乘 (d_model, vocab_size) 的矩阵，得到每个位置对词表的打分 logits

**一句话：完整模型只有首尾两步改变形状，中间全程是 (L, d_model)。**

## 整体结构

一个最小版 Transformer，可以拆成四层：

```
Token ID (整数)
  ↓ ①Embedding：ID 查表变向量
  ↓ ②Positional Encoding：加位置信息
  ↓ ③多层 Transformer Block：反复加工信息
  ↓ ④输出层 Linear：向量映射回词表
  ↓ logits (每个位置对词表的打分)
```

核心变化只有一个：**一开始输入是整数 Token ID，经过 Embedding 后变成向量；之后所有模块都在处理 (L, d_model) 这个矩阵。**

![Tiny Transformer结构](https://file1.kamacoder.com/i/web/20261030144720.png?v=20261005)


首尾两个加粗橙色箭头是仅有的两次形状变化：开头整数 ID 查表变成 4×8，结尾 4×8 乘 W_out 变成 4×10。中间紫色虚线框里不管放几层 Block，箭头上都是 4 × 8。


## 第一步：Embedding

模型不能直接理解文字，也不能直接理解 Token ID。比如一句话被切成：

```python
input_ids = [3, 8, 2, 6]
```

这些数字只是编号，没有语义。

**Embedding 要做的事，就是把每个编号查表，变成一个向量。**

```python
import numpy as np

np.random.seed(42)

vocab_size = 10  # 词表大小
L = 4            # 序列长度
d_model = 8      # embedding 维度

input_ids = np.array([3, 8, 2, 6])
embedding_table = np.random.randn(vocab_size, d_model)

x = embedding_table[input_ids]
print(f”Embedding 后: {x.shape}”)  # (4, 8)
```

**输出：**
```
Embedding 后: (4, 8)
```

现在，4 个 Token，每个 Token 都变成了 8 维向量。**面试官问”Embedding 做什么”，答案是：把整数 ID 查表变成稠密向量，这是模型能处理的输入格式。**

## 第二步：Positional Encoding

Self-Attention 本身不关心顺序。对它来说，”我喜欢你”和”你喜欢我”如果只看 Token 集合，差别并不明显。

所以我们要把位置信息加进去。

```python
def positional_encoding(L, d_model):
    pe = np.zeros((L, d_model))
    for pos in range(L):
        for i in range(0, d_model, 2):
            pe[pos, i] = np.sin(pos / (10000 ** (i / d_model)))
            if i + 1 < d_model:
                pe[pos, i + 1] = np.cos(pos / (10000 ** (i / d_model)))
    return pe

pe = positional_encoding(L, d_model)
x = x + pe  # 相加，不是拼接
print(f”加位置编码后: {x.shape}”)  # (4, 8)
```

**输出：**
```
加位置编码后: (4, 8)
```

**注意：位置编码不是拼接，而是相加。** 所以 shape 不变，仍然是 `(L, d_model)`。

**面试官问”位置编码怎么加”，答案是：用 sin/cos 函数生成 (L, d_model) 的矩阵，然后和 Embedding 相加，不是 concat。**

## 第三步：Transformer Block

一个 Block 里有两件事：

- **Attention**：让 Token 之间交换信息
- **FFN**：让每个 Token 独立加工信息

再加上残差连接和 LayerNorm，一个最小 Block 长这样：

```python
class TinyBlock:
    def __init__(self, d_model, d_ff):
        self.Wq = np.random.randn(d_model, d_model) * 0.01
        self.Wk = np.random.randn(d_model, d_model) * 0.01
        self.Wv = np.random.randn(d_model, d_model) * 0.01
        self.W1 = np.random.randn(d_model, d_ff) * 0.01
        self.b1 = np.zeros(d_ff)
        self.W2 = np.random.randn(d_ff, d_model) * 0.01
        self.b2 = np.zeros(d_model)

    def layer_norm(self, x, eps=1e-5):
        mean = x.mean(axis=-1, keepdims=True)
        std = x.std(axis=-1, keepdims=True)
        return (x - mean) / (std + eps)

    def attention(self, x):
        Q = x @ self.Wq
        K = x @ self.Wk
        V = x @ self.Wv
        scores = Q @ K.T / np.sqrt(x.shape[-1])
        weights = np.exp(scores) / np.exp(scores).sum(axis=-1, keepdims=True)
        return weights @ V

    def ffn(self, x):
        hidden = np.maximum(0, x @ self.W1 + self.b1)  # ReLU
        return hidden @ self.W2 + self.b2

    def forward(self, x):
        # MHA + 残差 + Norm
        x = self.layer_norm(x + self.attention(x))
        # FFN + 残差 + Norm
        x = self.layer_norm(x + self.ffn(x))
        return x
```

这段代码没有多头（为了简化），没有 mask，也没有 Dropout，只保留最核心的数据流。**面试时手撕 Tiny Transformer，这个 Block 就够了。**

## 第四步：堆多层 Block

Transformer 的”深度”来自重复堆叠 Block。

```python
num_layers = 2
blocks = [TinyBlock(d_model=8, d_ff=32) for _ in range(num_layers)]

for i, block in enumerate(blocks):
    x = block.forward(x)
    print(f”Block {i+1} 输出: {x.shape}”)  # (4, 8)
```

**输出：**
```
Block 1 输出: (4, 8)
Block 2 输出: (4, 8)
```

堆两层、六层、十二层，本质都是重复同一个结构。**只要每层输入输出 shape 一致，就可以一直往下接。**

## 第五步：输出层

最后一步，要把每个 Token 的隐藏向量映射回词表大小。

如果词表大小是 `vocab_size=10`，那输出层就是：

```python
W_out = np.random.randn(d_model, vocab_size) * 0.01
b_out = np.zeros(vocab_size)

logits = x @ W_out + b_out
print(f”logits 形状: {logits.shape}”)  # (4, 10)
```

**输出：**
```
logits 形状: (4, 10)
```

`logits[0]` 表示第 1 个位置对词表中 10 个 Token 的打分。打分最高的 Token，就是模型当前最想输出的结果。

**面试官问”输出层做什么”，答案是：把 (L, d_model) 映射成 (L, vocab_size)，每个位置得到对词表的打分，argmax 取最高分的 Token ID。**

## 最小可运行 Demo

把所有代码连起来：

```python
import numpy as np

np.random.seed(42)

vocab_size = 10
L = 4
d_model = 8
d_ff = 32
num_layers = 2

input_ids = np.array([3, 8, 2, 6])
embedding_table = np.random.randn(vocab_size, d_model)

x = embedding_table[input_ids]

# positional encoding
def positional_encoding(L, d_model):
    pe = np.zeros((L, d_model))
    for pos in range(L):
        for i in range(0, d_model, 2):
            pe[pos, i] = np.sin(pos / (10000 ** (i / d_model)))
            if i + 1 < d_model:
                pe[pos, i + 1] = np.cos(pos / (10000 ** (i / d_model)))
    return pe

x = x + positional_encoding(L, d_model)

class TinyBlock:
    def __init__(self, d_model, d_ff):
        self.Wq = np.random.randn(d_model, d_model) * 0.01
        self.Wk = np.random.randn(d_model, d_model) * 0.01
        self.Wv = np.random.randn(d_model, d_model) * 0.01
        self.W1 = np.random.randn(d_model, d_ff) * 0.01
        self.b1 = np.zeros(d_ff)
        self.W2 = np.random.randn(d_ff, d_model) * 0.01
        self.b2 = np.zeros(d_model)

    def layer_norm(self, x, eps=1e-5):
        mean = x.mean(axis=-1, keepdims=True)
        std = x.std(axis=-1, keepdims=True)
        return (x - mean) / (std + eps)

    def attention(self, x):
        Q = x @ self.Wq
        K = x @ self.Wk
        V = x @ self.Wv
        scores = Q @ K.T / np.sqrt(x.shape[-1])
        weights = np.exp(scores) / np.exp(scores).sum(axis=-1, keepdims=True)
        return weights @ V

    def ffn(self, x):
        hidden = np.maximum(0, x @ self.W1 + self.b1)
        return hidden @ self.W2 + self.b2

    def forward(self, x):
        x = self.layer_norm(x + self.attention(x))
        x = self.layer_norm(x + self.ffn(x))
        return x

blocks = [TinyBlock(d_model, d_ff) for _ in range(num_layers)]

for block in blocks:
    x = block.forward(x)

W_out = np.random.randn(d_model, vocab_size) * 0.01
b_out = np.zeros(vocab_size)
logits = x @ W_out + b_out

print("最终输出 logits shape:", logits.shape)
print("每个位置预测的 token:", logits.argmax(axis=-1))
```

**输出示例：**

```text
最终输出 logits shape: (4, 10)
每个位置预测的 token: [5 5 4 3]
```

到这里，一个 Tiny Transformer 就真的跑起来了。

它还不会写文章，也不会聊天，因为权重是随机的，没有训练。预测出来的 `[5 5 4 3]` 没有任何意义。**但结构已经完整了**：Embedding 把 ID 变成向量，位置编码告诉模型顺序，Block 反复加工信息，输出层把向量变回词表打分。

**面试时怎么讲这段代码？**

按维度变化线串起来：

1. **(L,) → (L, d_model)**：Embedding 查表
2. **(L, d_model) → (L, d_model)**：加位置编码，形状不变
3. **(L, d_model) → (L, d_model)**：N 层 Block，每层形状不变
4. **(L, d_model) → (L, vocab_size)**：输出层映射回词表

重点说清楚**只有首尾两步改变形状**，中间全程是 (L, d_model)。这就是 Transformer 能随意加深的原因。

## 这个 Tiny 版和真实大模型差在哪

这份代码是标准 Transformer 的骨架，和今天的 GPT、Llama、Qwen 比，少了这些东西：

| 缺的部分 | 本文代码 | 真实 Decoder-only 大模型 |
|---|---|---|
| 因果掩码 | 没有，每个位置能看到全句 | 有，第 i 个位置只能看到前 i 个 |
| 多头 | 单头 | 多头，现代模型多用 GQA |
| 位置编码 | 正弦编码，加在输入上 | RoPE，作用在 Q、K 上 |
| 归一化 | Post-Norm + LayerNorm | Pre-Norm + RMSNorm |
| FFN | ReLU 两层 | SwiGLU 三个矩阵 |
| 输出层 | 独立的 W_out | 常和 Embedding 共享权重 |
| 训练 | 无 | 交叉熵预测下一个 Token |

**最关键的是因果掩码。** 没有它，模型训练时能"偷看"答案，就没法做自回归生成。补上也很简单，在 softmax 前把上三角位置设成负无穷：

```python
def causal_mask(L):
    # 上三角（不含对角线）为 True，表示要屏蔽的未来位置
    return np.triu(np.ones((L, L), dtype=bool), k=1)

scores = Q @ K.T / np.sqrt(d_k)
scores = np.where(causal_mask(L), -1e9, scores)
weights = softmax(scores)
```

![因果掩码](https://file1.kamacoder.com/i/web/20261030144722.png?v=20261005)


左边是本文代码的情况：每个位置都能看到整句话，适合 BERT 这类理解任务。右边红色格子就是被屏蔽的未来位置，「吃」只能看见「我、爱、吃」，看不到「面」。GPT 这类生成模型训练时必须加这个掩码。

这些差异不是推翻 Transformer，而是在骨架上换零件。RoPE、RMSNorm、SwiGLU、GQA 分别在改位置编码、归一化、FFN 和多头，对照上表就能看出 Llama 动了哪几块。

## 知识拓展

**Q1：Embedding 表的参数量是多少？**

vocab_size × d_model。词表 15 万、d_model 4096 的模型，光 Embedding 就有约 6 亿参数。这也是很多模型让输出层和 Embedding 共享权重的原因，能省下同样大的一块参数。

**Q2：位置编码为什么是相加，不是拼接？**

拼接会让维度从 d_model 变成 2 × d_model，后面所有矩阵都要跟着变大。相加不改形状，而且高维空间里位置信息和语义信息基本不会互相干扰，模型能自己学会区分。

**Q3：logits 和概率是什么关系？**

logits 是未归一化的打分，对最后一维做 softmax 才是概率分布。生成时取最后一个位置的 logits，按 greedy、top-k 或 top-p 采样出下一个 Token，拼回输入再跑一遍。这就是自回归生成。

**Q4：训练时 loss 怎么算？**

Decoder-only 模型用"预测下一个 Token"：位置 i 的 logits 去预测第 i+1 个 Token，和真实 Token 算交叉熵。一句长度 L 的话，一次前向就能同时得到 L-1 个训练信号，这得益于因果掩码。

**Q5：为什么推理时需要 KV Cache？**

自回归生成每多一个 Token 就要重跑一遍。前面 Token 的 K、V 不会变，缓存起来就不用重复计算，每步只算新 Token 的 Q、K、V。显存怎么被 KV Cache 吃光，可以看[《KV Cache为什么会吃光显存》](../app/kv_cache_paged_attention.md)。

**Q6：面试让手撕完整 Transformer，要写到什么程度？**

**能写出 Embedding + 位置编码 + N 层 Block + 输出层，并能说出每步形状就够了。** 如果岗位偏生成式模型，主动补上因果掩码，这是最容易拉开差距的一行代码。

手撕到这里，标准 Transformer 已经从零件拼成了整机。

很多录友会背公式，但说不清形状在哪一步变化、因果掩码为什么必须加。**能把这条数据流从 ID 讲到 logits，再说出它和 Llama 差在哪，面试手撕这一关就稳了。**
