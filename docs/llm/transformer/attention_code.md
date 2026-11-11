---
title: 手撕Attention：50行代码看懂注意力机制的每一步计算
description: 手撕Attention代码实现，从QKV矩阵计算、缩放点积注意力、Softmax归一化到加权求和完整拆解，不依赖深度学习框架，用纯Python和矩阵维度变化理解注意力机制每一步，掌握面试手撕代码的维度推导、Scaled Dot-Product公式和权重矩阵角色，适合准备Transformer原理和大模型应用开发面试。
keywords: [手撕Attention, Attention代码实现, Transformer代码, 注意力机制实现, 面试手撕代码, Scaled Dot-Product Attention, QKV矩阵, Softmax归一化, Transformer面试题, 大模型面试]
tags: [Transformer, 手撕代码, 面试]
---

# 手撕Attention：50行代码看懂注意力机制的每一步计算

上一篇[《一层Transformer Block的前向过程》](./transformer_structure.md)把 Attention、FFN、残差、LayerNorm 全串起来了，但面试官最爱问的还是这个：**"Attention 那几个矩阵乘法你能手写一遍吗？Q、K、V 的维度怎么变的？"**

很多录友会这样答："Q 乘 K 的转置，除以根号 d_k，做 softmax，再乘 V。"

公式没错，但这只是背下来的一行字。面试官接着问"Q @ K^T 是什么形状""为什么最后又回到 d_model""不除根号 d_k 会怎样"，就答不上来了。**手撕 Attention 考的是每一步的形状和作用，不是公式本身。**

这篇不依赖任何深度学习框架，用 NumPy 从零手写 Scaled Dot-Product Attention，每一步都打印 shape，让录友看清数据是怎么流动的。

## 简要回答

- **Q、K、V**：输入 X 分别乘三个可学习矩阵得到，Q 负责"我要找什么"，K 负责"我能被怎么找到"，V 负责"我实际提供什么内容"
- **打分**：Q @ Kᵀ 得到 (L, L)，第 i 行是第 i 个词对所有词的相关性
- **缩放**：除以 √d_k，防止 d_k 大时分数过大、softmax 饱和、梯度变小
- **归一化与聚合**：按行 softmax 得到权重，再乘 V，形状回到 (L, d_v)

**一句话：Attention 让每个位置按相关性从所有位置取信息，中间只多出一张 L×L 的关系表。**

## 输入是什么样的

用一个简单的句子演示：**远方有颗苹果树**。

假设经过 embedding 后，每个字变成一个 4 维向量（实际模型通常是 512、1024 维，这里为了方便理解用 4 维）。

```python
import numpy as np
np.set_printoptions(precision=4, suppress=True)

# 输入序列：远方有颗苹果树（已经过embedding）
# 序列长度 L=7, embedding维度 d_model=4
L = 7  # 序列长度
d_model = 4  # embedding维度

# 模拟embedding后的输入
np.random.seed(42)
X = np.random.randn(L, d_model)

print("输入矩阵 X 的形状:", X.shape)
print("具体内容:\n", X[:3, :])  # 只打印前3个token
```

**输出:**
```
输入矩阵 X 的形状: (7, 4)
具体内容:
 [[ 0.4967 -0.1383  0.6477  1.523 ]
 [-0.2342 -0.2341  1.5792  0.7674]
 [-0.4695  0.5426 -0.4634 -0.4657]]
```

可以看到，输入就是一个 `(7, 4)` 的矩阵：
- 7 行：代表 7 个 token
- 4 列：代表每个 token 的 embedding 向量是 4 维

![Attention形状变化](https://file1.kamacoder.com/i/web/20261030144658.png?v=20261005)


中间那张绿色 7×7 是整个过程唯一的"新形状"，它的第 i 行记录第 i 个字对所有字的权重。绿色描边追踪的「有」这一行，进去是 4 维，出来还是 4 维，变的是里面装的信息。

## 第一步：计算 Q、K、V

## 第一步：计算 Q、K、V

**Q、K、V 到底是什么？**

用大白话说：
- **Q (Query)**：我想问什么？我需要什么信息？
- **K (Key)**：你能提供什么信息？
- **V (Value)**：我实际要传递的内容

它们都是通过对输入 X 做线性变换得到的：

```python
# 定义三个权重矩阵（实际训练中这些是可学习参数）
d_k = 4  # Q和K的维度
d_v = 4  # V的维度

W_Q = np.random.randn(d_model, d_k)
W_K = np.random.randn(d_model, d_k)
W_V = np.random.randn(d_model, d_v)

print("权重矩阵形状:")
print("W_Q:", W_Q.shape)  # (4, 4)
print("W_K:", W_K.shape)  # (4, 4)
print("W_V:", W_V.shape)  # (4, 4)

# 计算 Q、K、V
Q = np.dot(X, W_Q)  # (7, 4) @ (4, 4) = (7, 4)
K = np.dot(X, W_K)
V = np.dot(X, W_V)

print("\nQ、K、V 形状:")
print("Q:", Q.shape)  # (7, 4)
print("K:", K.shape)  # (7, 4)
print("V:", V.shape)  # (7, 4)
```

**输出:**
```
权重矩阵形状:
W_Q: (4, 4)
W_K: (4, 4)
W_V: (4, 4)

Q、K、V 形状:
Q: (7, 4)
K: (7, 4)
V: (7, 4)
```

**为什么要分三个矩阵？**

因为同一个输入，在不同角色下需要关注的特征不一样：
- 作为 Query 时，关注"我需要什么信息"
- 作为 Key 时，关注"我能提供什么信息"
- 作为 Value 时，是"我实际要传递的内容"

通过不同的权重矩阵 W_Q、W_K、W_V，让模型学到这三种不同的"视角"。**这也回答了"为什么不直接用 X"：三个可学习的线性变换，让同一个词在"找、被找、提供内容"三种角色下有不同的表示。**

## 第二步：计算注意力分数

现在有了 Q 和 K，就可以计算"相关性"了。

**怎么计算相关性？**

用矩阵乘法：Q 和 K 的转置相乘。

```python
# Q @ K^T: (7, 4) @ (4, 7) = (7, 7)
attention_scores = np.dot(Q, K.T)

print("注意力分数矩阵形状:", attention_scores.shape)  # (7, 7)
print("具体内容:\n", attention_scores[:3, :3])  # 打印前3x3部分
```

**输出:**
```
注意力分数矩阵形状: (7, 7)
具体内容:
 [[-1.4984 -3.9249  1.4326]
 [-4.2929 -9.3983  4.3012]
 [-2.358   0.3717  1.4642]]
```

**这个 (7, 7) 矩阵是什么意思？**

- 每一行代表一个 token
- 每一列也代表一个 token
- 位置 `[i, j]` 的值表示：第 i 个 token 对第 j 个 token 的"关注程度"

比如 `attention_scores[2, 0]` 就是"有"这个字对"远"这个字的关注分数。**(L, L) 的来源很简单：每个词都要和每个词（包括自己）算一次相关性。**

## 第三步：缩放 (Scaled)

这是论文里的关键技巧。

```python
# 缩放：除以 sqrt(d_k)
scaled_scores = attention_scores / np.sqrt(d_k)

print("缩放后的分数形状:", scaled_scores.shape)  # (7, 7)
print("缩放前的值范围:", attention_scores.min(), "到", attention_scores.max())
print("缩放后的值范围:", scaled_scores.min(), "到", scaled_scores.max())
```

**输出:**
```
缩放后的分数形状: (7, 7)
缩放前的值范围: -17.45 到 14.76
缩放后的值范围: -8.72 到 7.38
```

**为什么要除以 √d_k？**

当 d_k 很大（比如 512）时，Q·K^T 的值会变得很大，导致 softmax 时：
- 大的值经过 softmax 后接近 1
- 小的值经过 softmax 后接近 0
- 梯度变得很小，难以训练

除以 √d_k 把数值范围控制在合理区间，让 softmax 不会过度集中。**不除的后果是：softmax 接近 one-hot，大部分位置梯度接近 0，训练变慢甚至不稳定。**

## 第四步：Softmax 归一化

把分数转成概率分布：

```python
def softmax(x):
    """对每一行做softmax"""
    exp_x = np.exp(x - np.max(x, axis=-1, keepdims=True))  # 减去最大值防止溢出
    return exp_x / np.sum(exp_x, axis=-1, keepdims=True)

# 对每一行做softmax
attention_weights = softmax(scaled_scores)

print("注意力权重形状:", attention_weights.shape)  # (7, 7)
print("第一个token的注意力分布:\n", attention_weights[0, :])
print("每一行的和:", np.sum(attention_weights, axis=-1))  # 每行和为1
```

**输出:**
```
注意力权重形状: (7, 7)
第一个token的注意力分布:
 [0.0412 0.0122 0.1782 0.1663 0.2193 0.1452 0.2376]
每一行的和: [1. 1. 1. 1. 1. 1. 1.]
```

![Softmax归一化](https://file1.kamacoder.com/i/web/20261030144659.png?v=20261005)


左边「方」的分数是 -1.96，右边权重只剩 0.01，但不会变成负数。右边 7 根柱子加起来正好是 1，所以它能直接当作"注意力怎么分配"的比例去加权 V。这组权重来自随机初始化，看不出语义，训练后才会让「远」真正偏向相关的字。

**这个矩阵的意义：**

- 每一行是一个概率分布，和为 1
- 第 i 行表示：第 i 个 token 应该分配多少注意力给每个位置
- 数值越大，表示越重要

## 第五步：加权求和

最后一步：用注意力权重去加权 V。

```python
# 用注意力权重加权V
# (7, 7) @ (7, 4) = (7, 4)
output = np.dot(attention_weights, V)

print("最终输出形状:", output.shape)  # (7, 4)
print("输入形状:", X.shape)  # (7, 4)
print("\n输出的前3个token:\n", output[:3, :])
```

**输出:**
```
最终输出形状: (7, 4)
输入形状: (7, 4)

输出的前3个token:
 [[-0.3226 -0.5214  1.3029 -1.6176]
 [ 0.141  -0.127   2.0231 -0.9807]
 [ 0.2192 -0.4142  2.2163 -0.555 ]]
```

**注意观察：**

- 输出形状和输入形状完全一样：`(7, 4)`
- 但每个 token 的向量已经融合了其他 token 的信息
- 融合的权重就是刚才计算的注意力分布

**形状回到原样的原因：(L, L) 的权重乘 (L, d_v) 的 V，得到 (L, d_v)，单头时 d_v 通常等于 d_model。**


## 完整代码

```python
import numpy as np

def scaled_dot_product_attention(Q, K, V):
    """
    Scaled Dot-Product Attention

    参数:
        Q: Query矩阵, shape (L, d_k)
        K: Key矩阵, shape (L, d_k)
        V: Value矩阵, shape (L, d_v)

    返回:
        output: 输出矩阵, shape (L, d_v)
        attention_weights: 注意力权重, shape (L, L)
    """
    d_k = Q.shape[-1]

    # 1. Q @ K^T
    scores = np.dot(Q, K.T)
    print(f"Step 1 - 注意力分数: {scores.shape}")

    # 2. 缩放
    scaled_scores = scores / np.sqrt(d_k)
    print(f"Step 2 - 缩放后: {scaled_scores.shape}")

    # 3. Softmax
    exp_scores = np.exp(scaled_scores - np.max(scaled_scores, axis=-1, keepdims=True))
    attention_weights = exp_scores / np.sum(exp_scores, axis=-1, keepdims=True)
    print(f"Step 3 - 注意力权重: {attention_weights.shape}")

    # 4. 加权求和
    output = np.dot(attention_weights, V)
    print(f"Step 4 - 最终输出: {output.shape}")

    return output, attention_weights


# 示例使用
if __name__ == "__main__":
    L = 7  # 序列长度
    d_model = 4  # embedding维度

    np.random.seed(42)

    # 1. 输入
    X = np.random.randn(L, d_model)
    print(f"输入X: {X.shape}\n")

    # 2. 生成Q、K、V
    W_Q = np.random.randn(d_model, d_model)
    W_K = np.random.randn(d_model, d_model)
    W_V = np.random.randn(d_model, d_model)

    Q = np.dot(X, W_Q)
    K = np.dot(X, W_K)
    V = np.dot(X, W_V)

    print(f"Q: {Q.shape}")
    print(f"K: {K.shape}")
    print(f"V: {V.shape}\n")

    # 3. 计算Attention
    print("=== 开始计算Attention ===")
    output, weights = scaled_dot_product_attention(Q, K, V)

    print(f"\n最终输出形状: {output.shape}")
    print(f"输入形状: {X.shape}")
    print(f"形状保持不变: {output.shape == X.shape}")
```

**运行输出:**
```
输入X: (7, 4)

Q: (7, 4)
K: (7, 4)
V: (7, 4)

=== 开始计算Attention ===
Step 1 - 注意力分数: (7, 7)
Step 2 - 缩放后: (7, 7)
Step 3 - 注意力权重: (7, 7)
Step 4 - 最终输出: (7, 4)

最终输出形状: (7, 4)
输入形状: (7, 4)
形状保持不变: True
```

**面试时怎么讲这段代码？**

按三条线串起来：

1. **维度变化线**：(L, d_model) → (L, d_k) → (L, L) → (L, d_v)，重点说清楚为什么中间会出现 (L, L)
2. **公式线**：Attention(Q, K, V) = softmax(Q·K^T / √d_k)·V，这是论文原话，面试官会对公式
3. **作用线**：让每个位置都能看到其他位置，并把相关的信息聚合回来

别只念代码，**把维度推导和作用结合起来讲**，比如"Q @ K^T 得到 (L, L)，这个矩阵的第 i 行就是第 i 个词对所有词的相关性分数，然后 softmax 归一化后乘 V，就把相关的信息按权重聚合回来了"。

## 知识拓展

**Q1：为什么不直接用 X 去算 Attention，非要搞三个矩阵？**

因为查询、匹配、传递是三种不同的语义角色。W_Q、W_K、W_V 是可学习的，训练过程中模型会学到：Q 关注"我需要什么特征"，K 关注"我有什么特征"，V 关注"我要传递什么内容"。如果不分三个矩阵，模型没法区分这三种角色。

**Q2：为什么是 Q @ K^T，不是 K @ Q^T？**

**Q @ K^T 的第 i 行是第 i 个 Query 对所有 Key 的打分**，softmax 按行归一化，正好得到"第 i 个词把注意力怎么分给所有词"。K @ Q^T 是它的转置，如果照样按行做 softmax，归一化的对象就变成"一个 Key 被所有 Query 打的分"，含义错了。

**Q3：d_k、d_v、d_model 的关系是什么？**

- d_model 是输入输出的维度，通常是 512 或 1024
- d_k = d_v = d_model / num_heads（Multi-Head 会讲）
- 单头 Attention 里通常 d_k = d_v = d_model，为了简化

**Q4：Attention 的时间复杂度是多少？**

O(L² · d_model)。瓶颈在 Q @ K^T 得到 (L, L)，然后再乘 V。序列长度翻倍，这部分计算量翻 4 倍，这也是长上下文推理贵的根源之一。

**Q5：面试官问"手撕 Attention"，代码要写到什么程度？**

**能写出 scaled_dot_product_attention 函数就够了**，50 行以内。重点是：

- Q @ K^T 得到分数
- 除以 √d_k
- softmax
- 乘 V

每一步都能说出维度变化和作用，比死背代码强。

手撕 Attention 难的不是代码，50 行就写完了。

**难的是写每一行时都清楚矩阵是什么形状、这一步在解决什么问题。** 做到这一点，面试官换着角度追问也不怕。
