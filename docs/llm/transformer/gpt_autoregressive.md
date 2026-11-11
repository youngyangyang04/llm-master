---
title: GPT：为什么自回归成为 LLM 主流
date: 2026-01-22
description: GPT 用自回归方式预测下一个词，因果掩码让模型只能看左边的上下文。这种设计看似限制了模型能力，却成为今天所有大模型的架构基础。本文讲透自回归生成的原理、因果掩码的作用、训练和推理的一致性、为什么 Decoder-only 赢了 BERT 的双向编码，以及 GPT-1 到 GPT-4 的演进路线。
keywords: [GPT, 自回归, Decoder-only, 因果掩码, Causal Mask, Next Token Prediction, GPT-1, GPT-2, GPT-3, GPT-4, 大语言模型, LLM, Transformer Decoder]
tags: [Transformer, GPT, 自回归, LLM, 生成式模型]
---

# GPT：为什么自回归成为 LLM 主流

2018 年 6 月，OpenAI 发布 GPT-1，用 Decoder-only 架构做预训练，当时只是「BERT 之前的尝试」。

2023 年，ChatGPT 横扫全球，你会发现：**今天所有主流大模型——GPT-4、Claude、Llama、Qwen、GLM——全都是自回归的 Decoder-only 架构**。

BERT 的双向编码输了，GPT 的自回归生成赢了。

为什么？自回归不是限制了模型的能力吗——它只能看左边的词，看不到右边的上下文？

## 一、GPT 的核心设计：Decoder-only + 因果掩码

### 1.1 只用 Decoder，不要 Encoder

GPT 是标准 Transformer 的 **Decoder 部分**，去掉了 Encoder 和 Cross-Attention。

回顾标准 Transformer Decoder 的三个子层：
1. **Masked Self-Attention**：只能看到左边的词（因果掩码）
2. **Cross-Attention**：从 Encoder 获取信息
3. **FFN**：逐 Token 的前馈网络

GPT 只保留了 **第 1 层和第 3 层**：
```
输入 Embedding + 位置编码
  ↓
Masked Self-Attention（因果掩码，只能看左边）
  ↓
Add & Norm
  ↓
FFN
  ↓
Add & Norm
  ↓
重复 N 层
```

没有 Encoder，也就没有 Cross-Attention——输入和输出都是同一个序列。

### 1.2 因果掩码（Causal Mask）：只能看左边的词

**因果掩码是 GPT 的核心机制**。

在 Self-Attention 计算中，每个词的 Query 都会和所有词的 Key 打分。但在 GPT 里，**每个词只能和它左边的词（包括自己）打分，不能看右边**。

<!-- drawio 源文件: ./drawio/gpt_causal_mask.drawio -->

![GPT因果掩码矩阵](https://file1.kamacoder.com/i/web/20261028111257.png?v=20261005)

例句：「我 爱 吃 麻辣烫」

| Query 词 | 可以看到的 Key |
|---------|--------------|
| 我 | 我 |
| 爱 | 我、爱 |
| 吃 | 我、爱、吃 |
| 麻辣烫 | 我、爱、吃、麻辣烫 |

实现方式：在 Softmax 之前，把右边的分数设为 `-∞`（Softmax 后变成 0）。

权重矩阵长这样（绿色是能看到的，灰色是被遮住的）：

```
       我  爱  吃  麻辣烫
我    [✓] [ ] [ ] [ ]
爱    [✓] [✓] [ ] [ ]
吃    [✓] [✓] [✓] [ ]
麻辣烫 [✓] [✓] [✓] [✓]
```

**这就是「因果」（Causal）的含义**：未来的词不能影响过去的词。

### 1.3 自回归生成（Autoregressive Generation）

GPT 的预训练任务是 **Next Token Prediction**：给定前面的词，预测下一个词。

训练时：
```
输入：我 爱 吃
目标：预测下一个词是 "麻辣烫"
```

推理时：
1. 给「我」，预测下一个词 → 「爱」
2. 给「我 爱」，预测下一个词 → 「吃」
3. 给「我 爱 吃」，预测下一个词 → 「麻辣烫」
4. ...直到生成结束符 `<EOS>`

**每次生成一个词，再把它拼回输入，继续生成下一个词——这就是自回归（Autoregressive）**。

### 1.4 与 BERT 的对比

|  | BERT | GPT |
|---|---|---|
| **架构** | Encoder-only | Decoder-only |
| **掩码机制** | 随机遮住 15% 的词 | 因果掩码（只能看左边）|
| **预训练任务** | MLM（预测被遮住的词）| CLM（预测下一个词）|
| **能看到的上下文** | 双向（整个句子）| 单向（只能看左边）|
| **训练和推理一致性** | **不一致**（训练有 [MASK]，推理没有）| **一致**（都是预测下一个词）|
| **适合的任务** | 分类、匹配、标注 | **生成、对话、续写** |

**关键差异**：BERT 训练时见过 `[MASK]`，推理时却没有；GPT 训练和推理都是「给前文，预测下一个词」，分布完全一致。

## 二、为什么自回归赢了双向编码

GPT 只能看左边，BERT 能看全句，按理说 BERT 更强才对，为什么 GPT 赢了？

### 2.1 生成任务的需求更大

**用户要的是「生成」，不是「分类」。**

- 对话：ChatGPT、Claude
- 写作：文章续写、代码生成
- 翻译、摘要：都是生成任务

这些任务的特点是**逐词输出，不能一次看到完整答案**。

BERT 擅长的是**理解任务**：
- 情感分析：这句话是正面还是负面？
- 文本匹配：两句话是否表达同一个意思？
- 命名实体识别：哪些词是人名/地名？

这些任务的特点是**输入固定，输出是标签或分数**。

**2023 年之后，生成任务的需求远大于分类任务，GPT 自然成为主流。**

### 2.2 训练和推理完全一致

BERT 的 MLM 有个根本问题：**训练时见过 `[MASK]`，推理时却没有**。

- 训练：我 [MASK] 吃 麻辣烫
- 推理：我 爱 吃 麻辣烫

模型在训练时学会了「根据上下文猜 [MASK]」，但推理时输入是完整句子，`[MASK]` 这个 token 根本不会出现。

**GPT 没有这个问题**：
- 训练：给「我 爱 吃」，预测「麻辣烫」
- 推理：给「我 爱 吃」，预测「麻辣烫」

训练和推理的输入分布完全一致，模型不需要「适应」新的输入形式。

### 2.3 一个模型通吃所有任务

BERT 做不同任务需要**接不同的分类头**：
- 情感分析：在 `[CLS]` 输出上接一个 2 分类层
- NER：在每个词的输出上接一个多分类层
- 文本匹配：输入两句话，在 `[CLS]` 输出上接 2 分类层

**换个任务，就要重新设计分类头，重新微调。**

GPT 不需要——**所有任务都是「给输入，生成输出」**：
- 情感分析？输入「这部电影很棒。情感：」，生成「正面」
- 翻译？输入「Translate to English: 我爱你」，生成「I love you」
- 代码生成？输入「写一个快排函数」，生成代码

**用户不用管任务类型，都是 prompt + 生成，这就是 In-Context Learning 的统一性。**

### 2.4 规模定律（Scaling Law）在自回归上更明显

2020 年 OpenAI 发现：**自回归模型的能力随参数量、数据量、计算量呈幂律增长**。

- GPT-2（1.5B 参数）：能写短文
- GPT-3（175B 参数）：能做复杂推理、翻译、编程
- GPT-4（据传 1.76T 参数）：能做多模态、长上下文、复杂规划

**自回归模型"喂得越多，越聪明"，这是 BERT 微调范式做不到的。**

BERT 微调的天花板是下游任务的标注数据量；GPT 的天花板是无标注语料和计算资源。

## 三、GPT 的演进：从 GPT-1 到 GPT-4

### 3.1 GPT-1（2018.6）：证明 Decoder-only 可行

- **参数量**：117M（12 层 Decoder）
- **预训练数据**：BooksCorpus（7000 本书，约 8 亿词）
- **预训练任务**：预测下一个词（CLM）
- **微调方式**：在下游任务上接分类头，和 BERT 一样需要微调

**核心贡献**：证明了「无监督预训练 + 有监督微调」在生成式模型上也有效。

但 GPT-1 没火——6 个月后 BERT 出来，刷爆了所有榜单，GPT-1 被盖过去了。

### 3.2 GPT-2（2019.2）：Zero-Shot 的惊喜

- **参数量**：1.5B（48 层 Decoder）
- **预训练数据**：WebText（800 万网页，40GB 文本）
- **关键变化**：**不微调，直接用 prompt 做下游任务**

GPT-2 发现：**如果预训练数据够多、模型够大，直接 prompt 就能做很多任务，不需要微调。**

例如：
- 输入「Translate to French: I love you」，模型输出「Je t'aime」
- 输入「TL;DR: [一篇长文]」，模型输出摘要

**这开启了 In-Context Learning 的大门。**

但 GPT-2 还不够强——生成的文本经常跑题，翻译和摘要效果一般。

### 3.3 GPT-3（2020.5）：Few-Shot Learning 和涌现能力

- **参数量**：175B（96 层 Decoder）
- **预训练数据**：CommonCrawl + WebText + Books + Wikipedia（300B tokens）
- **关键发现**：**模型够大，给几个例子就能学会新任务（Few-Shot Learning）**

例如做翻译：
```
输入：
English: I love you. French: Je t'aime.
English: How are you? French: Comment ça va?
English: Good morning. French:

输出：Bonjour.
```

GPT-3 不需要微调，也不需要任务特定的训练数据，**只要在 prompt 里给几个例子，模型就能推断出任务规律。**

**更重要的是**：GPT-3 展现了**涌现能力（Emergent Abilities）**——参数量超过某个阈值，模型突然会做之前不会的任务：
- 算术（小学数学题）
- 逻辑推理（简单的三段论）
- 代码生成（能写简单的 Python 函数）

### 3.4 InstructGPT 和 ChatGPT（2022）：RLHF 对齐

GPT-3 虽然强，但**不听话**：
- 问「怎么抢银行」，它会认真回答
- 问「1+1=?」，它可能回答「3」（因为训练数据里有人瞎写）

OpenAI 用 **RLHF（Reinforcement Learning from Human Feedback）** 让模型学会：
1. **遵循指令**（Instruction Following）
2. **拒绝有害请求**（Safety）
3. **承认不知道**（Honesty）

**ChatGPT 就是 GPT-3.5 + RLHF**，它的成功不是架构创新，而是**对齐做得好**。

### 3.5 GPT-4（2023.3）：多模态和长上下文

- **参数量**：未公开（据传 1.76T，混合专家架构 MoE）
- **关键升级**：
  - **多模态**：能看图、能听语音
  - **长上下文**：支持 32K tokens（后来升级到 128K）
  - **更强的推理能力**：能做复杂的数学证明、代码调试

GPT-4 的架构细节未公开，但已知：
- 用了 **MoE（Mixture of Experts）** 结构（后面章节详细讲）
- 训练数据包含**代码**（GitHub）和**多模态数据**（图文对）

## 四、自回归的推理过程：为什么慢

自回归生成的代价是**推理慢**。

假设要生成 100 个词：
1. 第 1 次前向：输入「我」，输出「爱」
2. 第 2 次前向：输入「我 爱」，输出「吃」
3. 第 3 次前向：输入「我 爱 吃」，输出「麻辣烫」
4. ...

**每生成一个词，都要重新跑一遍模型，前面的词还要重新算一遍 Attention**。

生成 N 个词，需要 N 次前向传播，每次的计算量随已生成的长度线性增长。

**这就是为什么大模型推理慢——不是模型本身慢，是自回归的本质决定的。**

优化方法：
- **KV Cache**：把之前算过的 Key 和 Value 缓存起来，不用每次重新算（后面章节详细讲）
- **并行解码**：一次生成多个候选词，再选最优的（Speculative Decoding）

## 五、GPT 的输入格式：没有 [CLS] 和 [SEP]

GPT 的输入更简单，没有 BERT 那套 `[CLS]` / `[SEP]` / Segment Embedding。

输入就是：
```
Token Embedding + Position Embedding
```

位置编码和 BERT 不同：
- **BERT**：可学习的绝对位置编码（最多 512 个位置）
- **GPT-2/GPT-3**：也是可学习的绝对位置编码（但长度更长，GPT-3 支持 2048）
- **GPT-4**：用了 **ALiBi** 或 **RoPE**（相对位置编码，支持更长上下文）

## 六、为什么 Decoder-only 比 Encoder-Decoder 更流行

Transformer 原文的 Encoder-Decoder 架构呢？T5、BART 这些 Encoder-Decoder 模型为什么也不是主流？

原因有三个：

### 6.1 Decoder-only 更简洁

Encoder-Decoder 有两组参数：
- Encoder 的 Self-Attention + FFN
- Decoder 的 Masked Self-Attention + Cross-Attention + FFN

**Decoder-only 只有一组参数，架构更简单，训练更容易。**

### 6.2 Decoder-only 的通用性更强

Encoder-Decoder 的典型用法：
- 输入给 Encoder（如翻译的源语言）
- 输出从 Decoder 生成（如翻译的目标语言）

这适合**输入和输出是不同语言、不同格式**的任务（翻译、摘要、问答）。

但**对话、续写、代码生成**这类任务，输入和输出是同一种语言、同一种格式，Encoder-Decoder 就显得多余了。

**Decoder-only 把输入和输出放在同一个序列里，一视同仁，更灵活。**

### 6.3 规模定律在 Decoder-only 上更明显

OpenAI 的 Scaling Law 研究发现：**Decoder-only 模型的能力随规模增长更快**。

同样参数量，Decoder-only 比 Encoder-Decoder 效果更好（GPT-3 vs T5-XXL）。

## 七、总结

**GPT 的核心是自回归生成 + 因果掩码**：
- 预训练任务是预测下一个词（CLM）
- 因果掩码让每个词只能看左边的上下文
- 训练和推理完全一致，没有 BERT 的 [MASK] 不一致问题

**为什么自回归成为主流**：
1. **生成任务的需求更大**（对话、写作、翻译、代码生成）
2. **训练和推理一致**（都是预测下一个词）
3. **一个模型通吃所有任务**（In-Context Learning）
4. **规模定律在自回归上更明显**（参数越多，能力越强）

**GPT 的演进**：
- GPT-1：证明 Decoder-only 可行
- GPT-2：Zero-Shot Learning
- GPT-3：Few-Shot Learning + 涌现能力
- ChatGPT：RLHF 对齐
- GPT-4：多模态 + 长上下文 + MoE

**自回归的代价是推理慢**，每生成一个词都要重新跑一遍模型，优化方法是 KV Cache 和并行解码。

下一篇我们讲 T5——Encoder-Decoder 的代表作，为什么在翻译和摘要任务上仍然有优势。

---

**相关文章**：
- [BERT：双向编码的代表](/llm/transformer/bert.html)（理解 BERT vs GPT）
- [Transformer 整体架构](/llm/transformer/transformer_base_encoder_decoder.html)（理解 Decoder 的因果掩码）
- [Multi-Head Attention](/llm/transformer/mha.html)（GPT 的核心是 Masked Self-Attention）
