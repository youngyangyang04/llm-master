---
title: 大模型量化怎么选？FP8、INT8、W4A16、AWQ、GPTQ、GGUF与KV Cache量化
description: 大模型量化不能只看4bit还是8bit。本文从权重、激活和KV Cache三个量化对象出发，讲清BF16、FP8、INT8、W4A16、AWQ、GPTQ与GGUF的关系，解释低位宽为什么不一定更快，并给出结合硬件内核、真实负载、任务质量、显存、TTFT、TPOT与Goodput的部署选型和验证方法。
keywords: [大模型量化, LLM量化, 模型量化, FP8, FP4, INT8, INT4, W4A16, W8A8, AWQ, GPTQ, GGUF, bitsandbytes, KV Cache量化, 权重量化, 激活量化, PTQ, QAT, vLLM量化, TensorRT-LLM量化, llama.cpp, GPU显存, 推理加速, 大模型部署, 大模型面试]
tags: [大模型应用, 大模型部署, 模型量化, 推理优化, 大模型面试]
---

# 量化不是只看4bit/8bit：权重、激活和KV Cache怎么选

<a href="https://programmercarl.com/other/project_kamaClaude.html">
  <img src="../../assets/images/kama-claude.jpg?v=20261005" style="width:100%;cursor:pointer;" alt="KamaClaude">
</a>

上一篇[《KV Cache为什么会吃光显存？从PagedAttention到Prefix Cache》](./kv_cache_paged_attention.md)讲了：模型权重装进GPU，只是拿到了入场券。上下文变长、并发升高以后，KV Cache照样可能把剩余显存吃光。

于是很多录友会继续追问：**把模型量化成4bit，是不是权重和KV Cache都会一起缩小，速度也一定更快？**

面试官通常也会这样问：“你们为什么选AWQ 4bit？和GPTQ、FP8、GGUF有什么区别？”

常见回答是：“4bit比8bit小一半，所以显存占用更低，推理也更快。AWQ和GPTQ都是量化格式，GGUF适合本地部署。”

这个回答混了三层问题：

- 4bit可能只描述权重，激活和KV Cache仍然是16bit；
- AWQ、GPTQ主要是量化方法，GGUF是文件格式和运行生态，根本不在同一层；
- 权重变小只保证读取的数据可能减少，不保证目标硬件有更快的计算内核；
- 只看通用问答“感觉没变”，可能漏掉数学、代码、工具调用和长上下文上的质量退化。

真正要讲清楚的是：**量化了谁、怎么算、在哪块硬件上跑，以及收益用什么指标证明。**

## 简要回答

- **量化对象先分清**：权重决定模型静态占用，激活参与每层计算，KV Cache随上下文和并发增长，三者可以使用不同精度；
- **记法要会读**：W4A16表示4bit权重、16bit激活，W8A8表示权重和激活都是8bit，它们的显存与加速空间不同；
- **方法和格式别混**：AWQ、GPTQ是常见的训练后权重量化方法，GGUF是llama.cpp生态使用的模型文件格式，可以容纳多种量化类型；
- **位宽低不等于速度快**：真实速度取决于硬件是否原生支持、推理引擎有没有成熟内核、量化与反量化开销，以及请求长度和Batch；
- **选型必须回归**：先确认模型能放下，再在同一硬件和真实负载下比较任务质量、显存、TTFT、TPOT、Goodput与每个成功任务成本。

一句话：**位宽只是一个参数，量化方案是“对象 × 格式 × 算法 × 硬件内核 × 业务指标”的组合。**

## 详细回答

### 一句“4bit模型”，到底量化了什么？

一次推理里，至少有三类大块数据需要区分。

**权重（Weight）**是训练完成后固定下来的模型参数。它们加载后长期占着显存，模型越大，权重占用越明显。

**激活（Activation）**是输入经过每一层时临时产生的中间结果。它跟Batch、序列长度和算子有关，会进入矩阵乘法。

**KV Cache**保存Attention历史Token的K和V。它不属于模型权重，会随着上下文长度和活跃请求数增长。

所以看到“INT4模型”，先别急着下结论，要继续问：

```text
权重是什么精度？激活用什么精度计算？KV Cache又是什么精度？
```

常见记法里：

| 记法 | 权重 | 激活 | 更直接的收益 |
|---|---|---|---|
| BF16 / FP16 | 16bit | 16bit | 高精度基线，兼容性通常最好 |
| W8A8 | 8bit | 8bit | 同时减少权重带宽和低精度矩阵计算成本 |
| W4A16 | 4bit | 16bit | 重点压缩权重，计算前可能需要解码或反量化 |
| FP8 KV Cache | 不一定改变 | 不一定改变 | 缩小每个Token的KV占用，提高长上下文并发空间 |

<!-- drawio源文件: ./drawio/model_quantization_01_objects_path.drawio -->

![大模型量化对象关系](https://file1.kamacoder.com/i/web/20260820104948.png?v=20261005)

这张图回答的是：权重、激活和KV Cache分别位于推理链路的哪里。权重量化主要压缩常驻参数，激活量化影响层内计算，KV Cache量化影响上下文容量；只写“4bit”无法说明另外两块数据发生了什么。

假设一个70B参数的模型，只粗算权重主体：

```text
BF16权重 ≈ 700亿 × 2 Byte ≈ 140 GB
INT8权重 ≈ 700亿 × 1 Byte ≈ 70 GB
INT4权重 ≈ 700亿 × 0.5 Byte ≈ 35 GB
```

这只是建立直觉。真实文件和显存还包含Scale、Zero Point、分组元数据、未量化层、对齐空间和运行时缓冲，**不会严格等于理论值。**

更关键的是：权重从140GB降到35GB，不代表KV Cache也从16bit变成4bit。上一篇已经推导过，KV Cache要单独看层数、KV头、Head维度、上下文、并发和自身精度。

### 高精度数字是怎么塞进低位宽的？

量化的直觉，是把一段连续的高精度数值映射到有限个离散档位。

常见线性量化可以写成：

```text
q = clamp(round(x / scale) + zero_point)
x_hat = scale × (q - zero_point)
```

`x`是原始值，`q`是低位宽整数，`x_hat`是反量化后的近似值。原始值只能落到有限档位，所以误差不可避免。

位宽越低，可用档位越少。真正影响误差的还包括：

- **Scale按什么粒度计算**：整张量、每通道还是每组；
- **Group Size多大**：分组更细通常更能适应局部分布，但元数据和内核实现更复杂；
- **异常值怎么处理**：少量极端值会拉大范围，让大量普通值挤在少数档位里；
- **哪些层保留高精度**：Embedding、输出层或少数量化敏感层可能需要单独处理；
- **校准数据是否贴近业务**：校准集分布不对，量化时保护的区域也可能不对。

<!-- drawio源文件: ./drawio/model_quantization_02_scale_error.drawio -->

![量化档位与误差机制](https://file1.kamacoder.com/i/web/20260820104950.png?v=20261005)

这张图回答的是：为什么同样是4bit，误差也可能完全不同。全局Scale被异常值拉宽后，普通值会挤在少数档位；分组、校准和敏感值保护，本质上是在重新分配有限的低精度表达能力。

这也解释了AWQ和GPTQ存在的意义。

[AWQ论文](https://arxiv.org/abs/2306.00978)关注激活分布所反映的重要权重，尽量保护对输出更敏感的部分；[GPTQ论文](https://arxiv.org/abs/2210.17323)则用近似二阶信息逐层补偿权重量化误差。两者通常都属于**训练后量化（PTQ）里的Weight-only路线**，但具体模型文件、Group Size、Kernel和推理引擎支持仍要单独核对。

不要把方法名直接当成效果保证。模型架构、校准数据、量化配置和运行内核任何一项变了，结果都可能变。

### 4bit为什么不一定比8bit更快？

因为推理不是比较文件大小，而是在执行一条完整计算链。

低位宽可能带来三类收益：

- 模型能否放入更少或更小的设备；
- 每次矩阵计算需要从显存读取多少权重；
- 硬件能否用低精度Tensor Core或其他专用单元提高计算吞吐。

但它也会引入额外工作：解包低位数据、读取Scale、反量化、类型转换，以及不成熟Kernel造成的调度开销。

如果硬件不能原生高效执行某种4bit格式，引擎可能先把权重还原到更高精度再计算。此时你得到的是“模型更小”，不一定是“计算更快”。小Batch、短输出时，额外开销甚至可能抵消带宽收益。

反过来，在有成熟W4A16 Kernel、Decode阶段明显受权重带宽限制的负载里，4bit权重又可能同时提高单卡容量和生成吞吐。

所以判断顺序应该是：

```text
先看能不能放下
→ 再看目标引擎能不能原生执行
→ 再看真实负载卡在带宽、算力还是调度
→ 最后看端到端指标
```

截至2026年8月，[vLLM量化支持矩阵](https://docs.vllm.ai/en/latest/features/quantization/)对AWQ、GPTQ、Marlin、INT8、FP8等方案列出了不同GPU代际、CPU和其他硬件的兼容差异。[TensorRT-LLM量化文档](https://nvidia.github.io/TensorRT-LLM/features/quantization.html)也把FP8、FP4、W4A16、W4A8和KV Cache量化拆成不同Recipe，并明确绑定具体GPU和后端能力。

这类支持状态变化很快。**文章或模型卡写着“支持”，只代表能进入候选集，不代表你的模型、硬件和特性组合已经跑通。**

### AWQ、GPTQ、GGUF和bitsandbytes到底是什么关系？

先按层级分，别把所有名词塞进一列排名。

| 名称 | 更接近哪一层 | 常见用途 | 选型时重点核验 |
|---|---|---|---|
| AWQ | 权重量化方法 | 低比特Weight-only部署 | 模型与Kernel支持、Group Size、质量回归 |
| GPTQ | 权重量化方法 | 训练后低比特权重压缩 | 量化配置、推理后端、校准与任务质量 |
| bitsandbytes | PyTorch量化库与加载方案 | 8bit推理、4bit QLoRA和开发验证 | 硬件后端、Linear层兼容、是否适合目标服务引擎 |
| GGUF | 模型文件格式与生态载体 | llama.cpp本地、CPU、Apple Silicon和混合后端 | 具体Quant Type、bpw、后端与模型架构支持 |
| FP8 / INT8 / FP4 / INT4 | 数值格式或位宽描述 | 权重、激活或KV的低精度存储与计算 | 到底量化哪个对象，Scale粒度，硬件是否原生支持 |

[llama.cpp官方量化说明](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md)里，同一个GGUF可以选择Q4_K_M、Q5_K、Q6_K等不同量化类型，还能通过Importance Matrix和逐张量配置保护敏感部分。

所以“GGUF是几bit”这个问题本身就不完整。很多GGUF量化会混合不同张量精度，文件名里的Q4也不等于整个模型严格每参数4bit。

同样，BF16和FP16通常是推理基线精度，不要笼统叫“16bit量化”；FP8虽然也是8bit，却保留浮点数的指数结构，动态范围和硬件执行路径与INT8不同。

### 项目里怎么选择量化方案？

先固定硬约束，再选候选方案。

如果BF16或FP16模型根本放不下，先估算权重、KV Cache和运行时余量。目标是本地CPU或Apple Silicon时，优先从llama.cpp支持的GGUF量化中选候选；目标是GPU服务时，优先看推理引擎和GPU代际真正支持的低精度Kernel。

如果模型已经能放下，但Decode受权重带宽限制，可以评估W4A16、AWQ或GPTQ。如果目标硬件对FP8或INT8矩阵计算支持成熟，而且Prefill和高吞吐计算占比较高，可以把W8A8或FP8方案放进候选。

如果权重已经很小，长上下文并发仍然上不去，继续压权重可能没有意义。这时要回到前文的KV公式，单独评估FP8或其他受支持的KV Cache量化。

<!-- drawio源文件: ./drawio/model_quantization_03_selection_loop.drawio -->

![大模型量化选型路径](https://file1.kamacoder.com/i/web/20260820104952.png?v=20261005)

这张图回答的是：量化选型为什么要先过容量和硬件两道门。模型放不下时优先缩权重，计算瓶颈要匹配原生Kernel，长上下文瓶颈要单独处理KV；所有候选最后都回到同一套质量、性能和成本验证。

这里没有“统一推荐4bit”。更实用的起点是：

| 真实约束 | 优先进入候选集 | 不能省略的验证 |
|---|---|---|
| GPU放不下权重 | W4A16、AWQ、GPTQ或受支持的FP4 | 任务质量、加载显存、Kernel兼容 |
| GPU能放下，追求服务吞吐 | 硬件原生支持的FP8或INT8 W8A8 | Prefill吞吐、TPOT、Goodput、Batch变化 |
| CPU、Apple Silicon或边缘运行 | GGUF中的合适Quant Type | 实际内存、Prompt处理速度、生成速度、质量 |
| 长上下文并发被KV卡住 | 单独量化KV Cache | 长上下文质量、并发、TTFT与TPOT |
| 业务质量非常敏感 | BF16/FP16基线或更保守量化 | 按场景逐项回归，不只看平均分 |

### 怎么证明量化真的有收益？

先准备未量化基线，再固定变量。

至少保证模型版本、Tokenizer、聊天模板、采样参数、硬件、引擎版本、并行方式和测试流量一致。否则测到的差异可能来自模板或调度，不是量化。

然后比较四类证据：

- **质量**：业务任务成功率、数学和代码正确率、RAG忠实度、JSON Schema通过率、工具调用正确率、长上下文召回；
- **容量**：模型文件、加载后显存、KV池容量、可服务最大并发与是否减少GPU数量；
- **性能**：TTFT、TPOT或ITL、端到端P95/P99、输入和输出Token吞吐、SLO内Goodput；
- **成本**：周期总成本、GPU利用率、失败与重试率，以及每个成功任务成本。

困惑度和通用Benchmark可以做早期筛选，但不能替代业务回归。量化误差可能只在少数关键Token上改变答案，平均分看着差不多，结构化输出或工具参数却开始频繁失败。

最后别只测单请求。Weight-only方案在并发升高后可能因为显存节省容纳更大Batch，收益才真正出现；某些低精度内核也可能只在特定Batch和矩阵形状下占优。

**量化成功的标准，不是模型文件变小，而是在质量门槛内提高SLO Goodput，并降低每个成功任务成本。**

## 知识拓展

**Q1：PTQ和QAT有什么区别？**

PTQ是在模型训练完成后做量化，成本低、部署常用，AWQ和GPTQ通常归在这条路线。QAT在训练阶段模拟量化误差，让模型提前适应低精度，质量潜力更高，但需要训练数据和算力，工程成本也更大。

**Q2：量化模型还能继续微调吗？**

可以，但要区分“更新全部量化权重”和“冻结量化基座，只训练额外参数”。[QLoRA](./lora_qlora.md)走的是后者：4bit基座主要用来省显存，训练的是LoRA参数，不等于直接对INT4权重做普通全参训练。

**Q3：为什么4bit文件不是BF16文件的四分之一？**

因为还要保存Scale、Zero Point、分组信息、元数据和少量高精度张量，文件还存在对齐与封装开销。看实际文件和加载后内存，不要只拿位宽做除法。

**Q4：可以把一个4bit模型再量化成更低位吗？**

技术上有些工具允许重数量化，但误差会在已经丢失信息的基础上继续累积。llama.cpp官方工具也明确警告，重数量化通常比从F16、BF16或F32源权重开始质量更差。

**Q5：模型量化后，KV Cache会自动跟着量化吗？**

不会。权重精度和KV Cache精度通常是两个独立配置。能否组合使用，还要查模型、Attention Backend、推理引擎与GPU的支持矩阵。

**Q6：面试里怎么回答“AWQ和GPTQ选哪个”？**

先说它们都是常见的Weight-only PTQ路线，再说明目标模型、引擎、GPU和Kernel支持决定候选范围；然后给出同条件下的任务质量、显存、TPOT与Goodput对比。只背“AWQ精度高、GPTQ速度快”这种固定结论，换个版本和硬件就可能失效。

本文涉及的实现状态核验于2026年8月，参考[vLLM量化文档](https://docs.vllm.ai/en/latest/features/quantization/)、[TensorRT-LLM量化文档](https://nvidia.github.io/TensorRT-LLM/features/quantization.html)、[Hugging Face bitsandbytes文档](https://huggingface.co/docs/transformers/main/en/quantization/bitsandbytes)、[llama.cpp量化说明](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md)、[AWQ论文](https://arxiv.org/abs/2306.00978)、[GPTQ论文](https://arxiv.org/abs/2210.17323)和[SmoothQuant论文](https://arxiv.org/abs/2211.10438)。

别再只说“4bit更小，所以更快”。

能把量化对象、硬件内核和业务指标对上，才算真正会做推理选型。
