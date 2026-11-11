---
title: 手撕Multi-Head Attention：拆分、并行、拼接三步走
description: 手撕Multi-Head Attention代码实现，从单头Attention扩展到多头注意力，完整拆解head维度拆分reshape、并行计算、concat拼接、输出投影W_O和张量维度变化，理解为什么一个头不够、多头怎么各司其职以及面试手撕MHA的维度推导和代码结构，适合准备Transformer原理和大模型应用开发面试。
keywords: [手撕Multi-Head Attention, MHA代码实现, 多头注意力代码, Transformer代码, head拆分, concat拼接, 输出投影, 面试手撕代码, Transformer面试题, 大模型面试]
tags: [Transformer, 手撕代码, 面试]
---

# 手撕Multi-Head Attention：拆分、并行、拼接三步走

上一篇[《手撕Attention》](./attention_code.md)从零实现了 Scaled Dot-Product Attention，走完了 Q、K、V → Softmax → 加权求和的完整流程。

但面试官会继续问：**"单头 Attention 有什么问题？Multi-Head 怎么实现？为什么要有输出投影 W_O？"**

常见的回答是："多头就是把 Attention 复制几份，最后拼起来。"

这句话把多头说成了"复制"，会让面试官以为你觉得参数量翻了 h 倍。**实际上多头是把 d_model 切成 h 份，总参数量和单头一样**，拼接后还要靠 W_O 融合。没讲到"切分"和"融合"，就等于没讲到多头的本质。

这篇在上一篇的基础上一行一行写出多头注意力，每一步都打印 shape，看清"拆分 → 并行 → 拼接 → 投影"四个新动作。

## 简要回答

- **为什么要多头**：单头一次只能学一种注意力分布，多头让模型在不同子空间里同时看不同关系
- **怎么拆**：Q、K、V 从 (L, d_model) reshape 成 (L, h, d_k)，再 transpose 成 (h, L, d_k)，d_k = d_model / h
- **怎么算**：每个头独立做一次 Scaled Dot-Product Attention，输出 (L, d_k)
- **怎么合**：拼接回 (L, d_model)，再乘 W_O 让各头信息混合

**一句话：多头不加参数，把 d_model 切成 h 段各看各的，最后用 W_O 融合。**

## 多头比单头多了哪几步

单头 Attention 的流程：

```
X → (W_Q, W_K, W_V) → Q, K, V → Attention → 输出
```

Multi-Head Attention 多了四步：

```
X → (W_Q, W_K, W_V) → Q, K, V
  → ①拆分成 h 个头（reshape + transpose）
  → ②每个头独立跑 Attention
  → ③拼接（concat）
  → ④乘输出投影矩阵 W_O
  → 最终输出
```

就这四个新动作：**拆分 → 并行计算 → 拼接 → 输出投影**。

![多头参数量对比](https://file1.kamacoder.com/i/web/20261030144701.png?v=20261005)


左右两块 W_Q 一样大，参数框里的 256 也一样。区别只在右边按列涂了两种颜色：前 4 列算出 Head 0 的注意力图，后 4 列算出 Head 1 的。**多头多出来的是视角，不是参数。**


## 第一步：线性映射生成 Q、K、V

这一步和单头完全一样。输入 X 分别乘三个权重矩阵，得到 Q、K、V。

```python
import numpy as np

# 超参数设置
L = 7        # 序列长度（远方有颗苹果树，7个字）
d_model = 8  # embedding 维度（实际是512，这里用8方便演示）
h = 2        # 注意力头数（实际常用8或16）
d_k = d_model // h  # 每个头的维度 = 8 // 2 = 4

np.random.seed(42)

# 模拟 embedding 后的输入
X = np.random.randn(L, d_model)
print(f"输入 X: {X.shape}")  # (7, 8)

# 三个权重矩阵（实际训练中是可学习参数）
W_Q = np.random.randn(d_model, d_model)
W_K = np.random.randn(d_model, d_model)
W_V = np.random.randn(d_model, d_model)

# 线性映射
Q = X @ W_Q   # (7, 8) @ (8, 8) = (7, 8)
K = X @ W_K
V = X @ W_V

print(f"Q: {Q.shape}")  # (7, 8)
print(f"K: {K.shape}")  # (7, 8)
print(f"V: {V.shape}")  # (7, 8)
```

**输出：**
```
输入 X: (7, 8)
Q: (7, 8)
K: (7, 8)
V: (7, 8)
```

Q、K、V 的形状都是 `(L, d_model)`，和单头完全一样。**区别在于下一步：单头直接拿去算 Attention，多头要先把它"切开"。**

## 第二步：Head 拆分，把 d_model 切成 h 份

这是多头和单头最关键的差异。

以 d_model=8、h=2 为例，每个头拿到 d_k = 8 ÷ 2 = 4 维：

```python
# reshape: (L, d_model) → (L, h, d_k)
Q_split = Q.reshape(L, h, d_k)
K_split = K.reshape(L, h, d_k)
V_split = V.reshape(L, h, d_k)

print(f"拆分后：")
print(f"Q_split: {Q_split.shape}")   # (7, 2, 4)
print(f"K_split: {K_split.shape}")   # (7, 2, 4)
print(f"V_split: {V_split.shape}")   # (7, 2, 4)

# 转置为 (h, L, d_k)，方便每个头独立计算
Q_heads = Q_split.transpose(1, 0, 2)
K_heads = K_split.transpose(1, 0, 2)
V_heads = V_split.transpose(1, 0, 2)

print(f"\n转置后（便于并行计算）：")
print(f"Q_heads: {Q_heads.shape}")  # (2, 7, 4)
print(f"K_heads: {K_heads.shape}")  # (2, 7, 4)
print(f"V_heads: {V_heads.shape}")  # (2, 7, 4)
```

**输出：**
```
拆分后：
Q_split: (7, 2, 4)
K_split: (7, 2, 4)
V_split: (7, 2, 4)

转置后（便于并行计算）：
Q_heads: (2, 7, 4)
K_heads: (2, 7, 4)
V_heads: (2, 7, 4)
```

**怎么理解这个 reshape？**

原来的 Q 是 `(7, 8)`：7 个 token，每个 token 8 维。

拆成 `(7, 2, 4)` 之后，变成：7 个 token，每个 token 有 2 个视角，每个视角 4 维。

转置成 `(2, 7, 4)` 之后，变成：2 个头，每个头看 7 个 token，每个 token 4 维。

**面试官问"为什么要 transpose"，答案是：转成 (h, L, d_k) 之后，第 0 维就是 head 的索引，可以直接用循环或者并行框架让每个头独立计算。**

![Head拆分过程](https://file1.kamacoder.com/i/web/20261030144702.png?v=20261005)


跟着绿色描边的「有」看：它的 8 个数在 reshape 后被切成两段，transpose 后前一段进了 Head 0 的表，后一段进了 Head 1 的表。整个过程没有任何计算，只是换了摆放方式。



## 第三步：每个头独立跑 Attention

拆分好之后，每个头的计算和单头完全一样：Q @ K^T → 缩放 → Softmax → 加权 V。

```python
def softmax(x):
    e = np.exp(x - np.max(x, axis=-1, keepdims=True))
    return e / e.sum(axis=-1, keepdims=True)

def single_head_attention(Q, K, V):
    """单个头的 Attention，输入输出都是 (L, d_k)"""
    d_k = Q.shape[-1]
    scores = Q @ K.T               # (L, L)
    scores = scores / np.sqrt(d_k) # 缩放
    weights = softmax(scores)      # (L, L)
    return weights @ V             # (L, d_k)

# 对每个头分别计算
head_outputs = []
for i in range(h):
    out_i = single_head_attention(Q_heads[i], K_heads[i], V_heads[i])
    head_outputs.append(out_i)
    print(f"Head {i} 输出: {out_i.shape}")  # (7, 4)
```

**输出：**
```
Head 0 输出: (7, 4)
Head 1 输出: (7, 4)
```

2 个头，每个头输出 `(7, 4)`。**每个头看到的是同一个句子，但在不同的 4 维子空间里理解它**，所以输出结果不同。

## 第四步：Concat 拼接，把多个头合并回来

2 个头算完了，怎么合并？**横向拼接**，沿着最后一个维度 concat。

```python
# 先把 list 转成数组 (h, L, d_k)
head_outputs = np.stack(head_outputs, axis=0)
print(f"拼接前（stack）: {head_outputs.shape}")  # (2, 7, 4)

# 转置回 (L, h, d_k)，再 reshape 成 (L, d_model)
head_outputs = head_outputs.transpose(1, 0, 2)
print(f"转置后: {head_outputs.shape}")  # (7, 2, 4)

concat_output = head_outputs.reshape(L, d_model)
print(f"拼接后（reshape）: {concat_output.shape}")  # (7, 8)
```

**输出：**
```
拼接前（stack）: (2, 7, 4)
转置后: (7, 2, 4)
拼接后（reshape）: (7, 8)
```

两个头各自的 4 维输出拼在一起，重新变回了 8 维。**维度和输入 X 完全一样。**

## 第五步：输出投影 W_O，让多个头互相混合

拼接之后，还差最后一步：乘输出投影矩阵 W_O。

```python
# 输出投影矩阵 W_O: (d_model, d_model)
W_O = np.random.randn(d_model, d_model)

# 最终输出
final_output = concat_output @ W_O
print(f"输出投影后: {final_output.shape}")   # (7, 8)
print(f"输入 X 形状:  {X.shape}")            # (7, 8)
print(f"形状是否一致: {final_output.shape == X.shape}")
```

**输出：**
```
输出投影后: (7, 8)
输入 X 形状:  (7, 8)
形状是否一致: True
```

**为什么还要乘 W_O？**

Concat 之后，8 维向量的前 4 维来自 Head 0，后 4 维来自 Head 1。它们各自是独立计算的，彼此之间还没有"交流"过。

W_O 做的事，就是让不同头的信息能够**互相混合**，产生一个统一的表示，传给后续的 FFN 层。

![W_O输出投影](https://file1.kamacoder.com/i/web/20261030144704.png?v=20261005)


左边那条竖线就是两个头的边界，前 4 维只有 Head 0 的信息。乘过 W_O 之后，右边任意一维都由左边全部 8 维加权得到，两个头的结果才真正混在一起。**面试官问"W_O 能不能省"，答案是：不能，没有 W_O 多个头就是物理拼接，没有融合。**



## 完整代码

```python
import numpy as np

def multi_head_attention(X, W_Q, W_K, W_V, W_O, h):
    """
    Multi-Head Attention 完整实现

    参数:
        X:    输入矩阵, shape (L, d_model)
        W_Q, W_K, W_V: 线性投影矩阵, shape (d_model, d_model)
        W_O:  输出投影矩阵, shape (d_model, d_model)
        h:    注意力头数

    返回:
        output: shape (L, d_model)
    """
    L, d_model = X.shape
    d_k = d_model // h

    # ① 线性映射
    Q = X @ W_Q
    K = X @ W_K
    V = X @ W_V
    print(f"[①线性映射] Q/K/V: {Q.shape}")

    # ② 拆分成 h 个头: (L, d_model) → (h, L, d_k)
    def split_heads(M):
        return M.reshape(L, h, d_k).transpose(1, 0, 2)

    Q_h = split_heads(Q)
    K_h = split_heads(K)
    V_h = split_heads(V)
    print(f"[②Head拆分] Q_h/K_h/V_h: {Q_h.shape}")

    # ③ 每个头独立计算 Attention
    def softmax(x):
        e = np.exp(x - np.max(x, axis=-1, keepdims=True))
        return e / e.sum(axis=-1, keepdims=True)

    head_outs = []
    for i in range(h):
        scores = Q_h[i] @ K_h[i].T / np.sqrt(d_k)
        attn = softmax(scores) @ V_h[i]
        head_outs.append(attn)
    print(f"[③并行Attention] 每个head输出: {head_outs[0].shape}")

    # ④ Concat 拼接: (h, L, d_k) → (L, d_model)
    concat = np.stack(head_outs, axis=0).transpose(1, 0, 2).reshape(L, d_model)
    print(f"[④Concat拼接] 拼接后: {concat.shape}")

    # ⑤ 输出投影
    output = concat @ W_O
    print(f"[⑤输出投影] 最终输出: {output.shape}")

    return output


# 运行测试
if __name__ == "__main__":
    np.random.seed(42)
    L, d_model, h = 7, 8, 2

    X = np.random.randn(L, d_model)
    W_Q = np.random.randn(d_model, d_model)
    W_K = np.random.randn(d_model, d_model)
    W_V = np.random.randn(d_model, d_model)
    W_O = np.random.randn(d_model, d_model)

    print("=== 开始计算 Multi-Head Attention ===\n")
    output = multi_head_attention(X, W_Q, W_K, W_V, W_O, h)

    print(f"\n输入形状: {X.shape}")
    print(f"输出形状: {output.shape}")
    print(f"形状保持不变: {output.shape == X.shape}")
```

**运行输出：**
```
=== 开始计算 Multi-Head Attention ===

[①线性映射] Q/K/V: (7, 8)
[②Head拆分] Q_h/K_h/V_h: (2, 7, 4)
[③并行Attention] 每个head输出: (7, 4)
[④Concat拼接] 拼接后: (7, 8)
[⑤输出投影] 最终输出: (7, 8)

输入形状: (7, 8)
输出形状: (7, 8)
形状保持不变: True
```

**面试时怎么讲这段代码？**

按维度变化线串起来：

1. **(L, d_model) → (L, d_model)**：线性映射得到 Q、K、V
2. **(L, d_model) → (h, L, d_k)**：reshape + transpose 拆分成 h 个头
3. **每个头 (L, d_k) → (L, d_k)**：独立跑 Attention
4. **(h, L, d_k) → (L, d_model)**：transpose + reshape 拼接回来
5. **(L, d_model) → (L, d_model)**：输出投影 W_O

重点说清楚**为什么要拆、怎么拆、怎么合**，别只念代码。比如"拆分的本质是把 d_model 维度切成 h 份，让每个头在 d_k 维子空间里独立关注不同的语义关系，拼接后通过 W_O 让这些视角融合成统一的表示"。

## 知识拓展

**Q1：为什么多头比单头好？**

单头 Attention 一次计算只能学到一种注意力模式。多头让模型可以**同时关注多种语义关系**：

- Head 0 可能关注语法（主谓宾）
- Head 1 可能关注语义（同义词、反义词）
- Head 2 可能关注长距离依赖

实验证明，不同的头确实学到了不同的模式，这是单头做不到的。

**Q2：头数 h 怎么选？**

常见配置：
- **BERT-base**: h=12，d_model=768，d_k=64
- **GPT-3**: h=96，d_model=12288，d_k=128

d_model 固定时，头数越多，每个头的 d_k 越小。**面试官问"头数越多越好吗"，答案是：不是，d_k 太小会限制每个头的表达能力。**

**Q3：d_k 为什么常见 64 或 128？**

经验值。原始 Transformer 和 BERT 用 64，GPT-3、Llama 系列多用 128。太小（比如 16）单个头学不到复杂模式，太大（比如 512）头数太少，多样性不足。

**Q4：W_O 能不能省掉？**

不能。Concat 之后，d_model 维度的前 d_k 维来自 Head 0，后面的来自其他头，它们之间是物理拼接，没有信息交互。W_O 是一个 (d_model, d_model) 的可学习矩阵，让不同头的输出能够融合，产生统一的表示。

**Q5：Multi-Head 的时间复杂度和单头一样吗？**

一样，都是 O(L² · d_model)。虽然拆成了 h 个头，但每个头只处理 d_k = d_model / h 维度，总计算量没变。拆分的好处是**提高表达能力，不是降低计算量**。

**Q6：实际推理时多个头是并行计算的吗？**

是的。手撕代码为了好读用了 for 循环，框架里是把 (h, L, d_k) 当成一个批量矩阵乘法一次算完，head 维度和 batch 维度一样被并行处理。

**Q7：面试时手撕 MHA 要写到什么程度？**

**能写出 multi_head_attention 函数就够了**，100 行以内。重点是：

- 拆分：reshape + transpose
- 并行计算：每个头跑 Attention
- 拼接：transpose + reshape
- 输出投影：乘 W_O

每一步都能说出维度变化和作用，比死背代码强。

多头的代码，八成都在 reshape 和 transpose 上。

**写之前先在纸上把 (L, d_model) → (h, L, d_k) → (L, d_model) 这条形状线画出来，代码基本不会写错。**
