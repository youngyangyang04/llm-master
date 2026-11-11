---
title: BERT：双向编码的代表，为什么今天不再主流
date: 2026-01-22
description: BERT 通过 Masked Language Model 实现双向理解，在分类、匹配等 NLU 任务上开创了预训练-微调范式，但在生成式大模型时代被 GPT 超越。本文讲透 BERT 的架构、预训练任务、优势场景与衰落原因，帮你理解为什么双向编码曾经是王者、今天不再主流，以及什么场景下 BERT 家族至今仍有优势。
keywords: [BERT, Encoder-only, Masked Language Model, MLM, 双向编码, 预训练微调, RoBERTa, ALBERT, 自然语言理解, NLU任务, BERT vs GPT]
tags: [Transformer, BERT, 预训练模型, NLU]
---

# BERT：双向编码的代表，为什么今天不再主流

2018 年 BERT 横空出世，在 11 个 NLP 任务上刷榜，彻底改变了 NLP 领域。它的核心创新是**双向编码**：用 Masked Language Model（MLM）让模型同时看到上下文，而不是像 GPT 那样只能看左边的词。

但今天，当你打开 ChatGPT、Claude、Llama，会发现它们都是 GPT 那条路——自回归的 Decoder-only 架构，BERT 的双向编码反而不再主流。

这是为什么？BERT 真的"过时"了吗？

## 一、BERT 的核心设计：Encoder-only + 双向上下文

### 1.1 只用 Encoder，不要 Decoder

BERT 是标准 Transformer 的 **Encoder 部分**，去掉了 Decoder。

回顾标准 Transformer：
- **Encoder**：Self-Attention 可以看到整个输入序列，用于理解语义
- **Decoder**：Self-Attention 只能看到左边的词（因果掩码），用于生成下一个词

BERT 只保留 Encoder 堆叠 12 层或 24 层（BERT-base / BERT-large），每层结构和 Transformer Encoder 完全一样：
```
输入 Embedding + 位置编码
  ↓
Multi-Head Self-Attention（无因果掩码，可以看全序列）
  ↓
Add & Norm
  ↓
FFN
  ↓
Add & Norm
  ↓
重复 N 层
```

**关键点**：Self-Attention **没有因果掩码**，每个词都能看到左边和右边的全部上下文。

### 1.2 Masked Language Model（MLM）：双向理解的预训练任务

问题来了：如果每个词都能看到上下文，模型怎么学？让它直接预测下一个词不就"作弊"了吗？

BERT 的做法是 **Masked Language Model（MLM）**：

1. **随机遮住 15% 的词**（用 `[MASK]` 替换）
2. **让模型根据上下文预测被遮住的词是什么**

例句：
```
原始：我 [爱] 吃 热腾腾 的 麻辣烫
遮蔽：我 [MASK] 吃 热腾腾 的 麻辣烫
任务：预测 [MASK] 是 "爱"
```

模型必须同时看「我」和「吃 热腾腾 的 麻辣烫」才能猜出 `[MASK]` 是"爱"，这样就学到了双向上下文。

<!-- drawio 源文件: ./drawio/bert_mlm_vs_gpt.drawio -->

![BERT MLM与GPT对比](https://file1.kamacoder.com/i/web/20261028111256.png?v=20261005)

**与 GPT 的对比**：
- **GPT**：给「我」，预测下一个词是「爱」（只能看左边）
- **BERT**：给「我 [MASK] 吃 热腾腾 的 麻辣烫」，预测 `[MASK]` 是「爱」（能看上下文）

BERT 的双向理解更适合**分类、匹配、抽取**这类需要「读懂整句话」的任务。

### 1.3 第二个预训练任务：Next Sentence Prediction（NSP）

BERT 预训练时还有第二个任务：**Next Sentence Prediction（NSP）**。

给定两句话 A 和 B，判断 B 是不是 A 的下一句：
```
正例：A="我饿了"，B="我们去吃饭吧" → IsNext
负例：A="我饿了"，B="今天天气真好" → NotNext
```

训练数据从文章里直接切：
- 50% 的样本，B 是 A 的真实下一句
- 50% 的样本，B 是从其他文章随机抽的

**为什么要这个任务**？BERT 论文说是为了让模型学会句子间的关系，但后续研究（RoBERTa）发现 NSP 没啥用，去掉也不影响效果。

## 二、BERT 的输入：三种 Embedding 相加

BERT 的输入不是单纯的词向量，而是三种 Embedding 相加：

```
输入 = Token Embedding + Segment Embedding + Position Embedding
```

1. **Token Embedding**：词本身的向量（和其他模型一样）
2. **Position Embedding**：位置编码，标记词在序列中的位置（0, 1, 2, ...）
3. **Segment Embedding**：标记这个词属于句子 A 还是句子 B（用于 NSP 任务）

<!-- drawio 源文件: ./drawio/bert_input_embeddings.drawio -->

![BERT三种Embedding](https://file1.kamacoder.com/i/web/20261028111255.png?v=20261005)

例如输入两句话：
```
[CLS] 我 饿 了 [SEP] 我们 去 吃饭 吧 [SEP]
```

- `[CLS]` 是句首特殊 token，它的输出向量用于分类任务
- `[SEP]` 分隔两句话
- Segment Embedding：前半句全是 0，后半句全是 1

**注意**：BERT 的 Position Embedding 是**可学习的参数**，不是 Transformer 原文的正弦位置编码。这也是为什么 BERT 最多只能处理 512 个 token——超过 512 就没有对应的位置参数了。

## 三、BERT 怎么用：预训练 + 微调

BERT 的使用分两个阶段：

### 3.1 预训练阶段

在海量无标注文本上训练 MLM + NSP：
- **数据**：Wikipedia + BookCorpus（33 亿词）
- **任务**：预测被遮住的词 + 判断句子是否连续
- **目标**：让模型学会语言的通用表示

预训练完的 BERT 还不能直接用，因为它只是学会了「填空」和「判断句子关系」，具体任务还需要微调。

### 3.2 微调阶段

拿预训练好的 BERT，在具体任务的标注数据上接着训练：

**分类任务**（如情感分析）：
- 在 `[CLS]` 的输出上接一个分类层
- 目标：预测句子的情感是正面还是负面

**序列标注任务**（如命名实体识别）：
- 在每个词的输出上接一个分类层
- 目标：预测每个词是不是人名/地名/机构名

**句子对任务**（如文本匹配）：
- 输入两句话，在 `[CLS]` 输出上接分类层
- 目标：预测两句话是否语义相似

**这种「预训练 + 微调」范式在 2018-2020 年统治了 NLP 领域。**

## 四、BERT 为什么今天不再主流

BERT 在 2018 年是王者，但 2023 年之后的大模型时代，主流架构全是 GPT 那条路（自回归 Decoder-only）。原因有三个：

### 4.1 生成式任务的需求更大

BERT 擅长的是**理解型任务**：
- 文本分类（情感分析、主题分类）
- 文本匹配（问答匹配、相似度计算）
- 序列标注（NER、词性标注）

这些任务的特点是**输入固定，输出是标签或分数**。

但今天用户更需要的是**生成式任务**：
- 对话（ChatGPT）
- 写作（文章续写、代码生成）
- 翻译、摘要

BERT 不是为生成设计的——它没有 Decoder，没法「一个词一个词地生成」。如果强行用 BERT 做生成，要么在 Encoder 输出上接个 Decoder（变成 Encoder-Decoder 架构，如 T5），要么用 BERT 做理解、再接个 GPT 做生成（太复杂）。

### 4.2 Decoder-only 架构更统一

GPT 的自回归架构有个巨大优势：**一个模型通吃理解和生成**。

- 理解任务？给上下文 + 问题，让模型生成答案
- 生成任务？给前文，让模型续写

**用户不用管任务类型，都是"给输入，拿输出"，这就是 prompt 的统一性。**

BERT 做不到——它的输出是每个词的向量，需要针对不同任务接不同的分类头，换个任务就要重新微调。

### 4.3 MLM 的遮蔽机制在推理时不存在

BERT 预训练时遮住 15% 的词，但**推理时输入是完整的句子，没有 `[MASK]`**。

这导致**预训练和推理的输入分布不一致**：
- 训练时：我 [MASK] 吃 热腾腾 的 麻辣烫
- 推理时：我 爱 吃 热腾腾 的 麻辣烫

模型训练时见过的 `[MASK]` token，在实际使用时根本不会出现。

GPT 没有这个问题——训练和推理都是「给前文，预测下一个词」，输入分布完全一致。

## 五、BERT 的遗产：什么场景还在用

虽然 BERT 不是大模型主流，但**在垂直场景和资源受限的环境下，BERT 家族至今仍有优势**。

### 5.1 文本分类和匹配任务

如果你的任务是：
- 判断一句话是正面还是负面（情感分析）
- 判断两句话是否表达同一个意思（语义匹配）
- 判断一段文本属于哪个类别（新闻分类）

**用微调后的 BERT 比调用大模型更快、更便宜、更可控。**

例如一个电商评论分类系统：
- **GPT 方案**：每条评论都要调用 API，成本高，延迟大
- **BERT 方案**：微调一个 BERT-base，部署在服务器上，毫秒级推理

### 5.2 对资源要求低

BERT-base 只有 110M 参数，BERT-large 也就 340M 参数，远小于动辄 7B、13B 的大模型。

**在边缘设备、移动端、私有化部署场景，BERT 更现实。**

### 5.3 多语言和垂直领域模型

BERT 的预训练-微调范式催生了大量垂直模型：
- **mBERT**：多语言 BERT，一个模型支持 104 种语言
- **SciBERT**：科学文献预训练，生物医学领域效果更好
- **FinBERT**：金融文本预训练，金融情感分析更准

这些模型在各自领域比通用大模型更精准。

### 5.4 Embedding 向量提取

虽然 BERT 不擅长生成，但它的 Encoder 输出是**高质量的语义向量**。

**RAG 系统、语义搜索、文本聚类**至今还在用 BERT 系列模型（如 `sentence-transformers`）提取 Embedding。

## 六、BERT 家族的改进：RoBERTa、ALBERT、DeBERTa

BERT 之后，研究者又推出了一系列改进版本：

### 6.1 RoBERTa：去掉 NSP，训更久

RoBERTa（2019，Facebook）发现：
- **NSP 任务没用**，去掉也不影响效果
- **训练数据和时间更重要**，用 10 倍数据、训更久，效果显著提升

RoBERTa 在几乎所有任务上都超过 BERT。

### 6.2 ALBERT：参数共享 + Factorized Embedding

ALBERT（2019，Google）的目标是**减小模型体积**：
- **层间参数共享**：12 层 Encoder 用同一组参数（参数量降 90%）
- **Embedding 分解**：词向量先降维再升维（省显存）

ALBERT-xxlarge 参数量只有 BERT-large 的 70%，效果却更好。

### 6.3 DeBERTa：解耦注意力 + 相对位置编码

DeBERTa（2020，Microsoft）改进了注意力机制：
- **解耦注意力**：内容和位置分开计算
- **相对位置编码**：不像 BERT 用绝对位置，而是编码相对距离

DeBERTa 在 SuperGLUE 榜单上超过人类表现。

## 七、BERT vs GPT：两条路的分叉

|  | BERT | GPT |
|---|---|---|
| **架构** | Encoder-only | Decoder-only |
| **注意力** | 双向（无因果掩码） | 单向（因果掩码）|
| **预训练任务** | MLM（填空） | CLM（预测下一个词）|
| **擅长任务** | 分类、匹配、标注 | 生成、对话、续写 |
| **推理方式** | 一次前向得到全部输出 | 自回归逐词生成 |
| **训练-推理一致性** | 不一致（训练有 [MASK]，推理没有）| 一致（都是预测下一个词）|
| **参数量** | 110M - 340M | 7B - 70B（今天的大模型）|
| **今天的主流** | 垂直场景 | 通用大模型 |

## 八、总结

BERT 的贡献在于：
1. **证明了预训练-微调范式的有效性**
2. **开创了双向理解的预训练方法**
3. **让 NLP 任务从手工特征工程进入大规模预训练时代**

但在生成式大模型时代，BERT 被 GPT 超越，原因是：
1. **生成任务的需求更大**，BERT 不擅长生成
2. **Decoder-only 架构更统一**，一个模型通吃理解和生成
3. **MLM 的遮蔽机制在推理时不存在**，训练和推理分布不一致

**BERT 没有"过时"，而是回到了它最擅长的领域**：
- 文本分类、匹配、标注等 NLU 任务
- 资源受限的边缘设备和私有化部署
- 垂直领域的语义理解
- Embedding 向量提取

**如果你的任务是「读懂一句话并分类」，用微调后的 BERT；如果是「生成回复或续写文本」，用 GPT 系列大模型。**

下一篇我们讲 GPT 为什么成为大模型主流——自回归生成和因果掩码的秘密。

---

**相关文章**：
- [Transformer 整体架构](/llm/transformer/transformer_base_encoder_decoder.html)（理解 Encoder vs Decoder）
- [Self-Attention 机制](/llm/transformer/qkv.html)（BERT 的核心是双向 Attention）
- [位置编码](/llm/transformer/pos_encode.html)（BERT 用可学习的位置编码，而不是正弦编码）
