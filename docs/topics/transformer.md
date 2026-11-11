# Transformer 专题

## 原理路线

1. [为什么大模型绕不开 Transformer](../llm/transformer/transformer_base_1.md)
2. [Transformer 数据流](../llm/transformer/transformer_data_flow.md)
3. [Q、K、V 与 Attention](../llm/transformer/qkv.md)
4. [Attention 计算全过程](../llm/transformer/qkv_cal.md)
5. [Multi-Head Attention](../llm/transformer/mha.md)
6. [位置编码](../llm/transformer/pos_encode.md)
7. [Transformer Block](../llm/transformer/transformer_structure.md)
8. [残差、LayerNorm 与 FFN](../llm/transformer/ffn_ln.md)
9. [三种 Transformer 架构](../llm/transformer/transformer_base_encoder_decoder.md)

## 代码与面试

- 依次手写 [Attention](../llm/transformer/attention_code.md)、[多头注意力](../llm/transformer/mha_code.md)、[FFN](../llm/transformer/fnn_code.md)、[LayerNorm 与残差](../llm/transformer/layernorm_residual_code.md)、[Transformer Block](../llm/transformer/transformer_block_code.md) 和 [Tiny Transformer](../llm/transformer/tiny_transformer_code.md)。
- 用 [Transformer 面试题](../interview/llm/transformer_interview.md)检查表达。

## 模型家族与架构演进

- [BERT：Encoder-only](../llm/transformer/bert.md)
- [GPT：自回归生成](../llm/transformer/gpt_autoregressive.md)
- [T5：Encoder-Decoder](../llm/transformer/t5.md)
- [MoE 稀疏专家网络](../llm/transformer/moe.md)
- [手撕 MoE-FFN](../llm/transformer/moe_ffn_code.md)
- [现代 LLM 改进：RoPE、RMSNorm、SwiGLU、GQA](../llm/transformer/modern_llm_improvements.md)
