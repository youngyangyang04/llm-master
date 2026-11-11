---
title: 手撕MoE-FFN：PyTorch实现Top-K路由、负载均衡与梯度验证
description: 用PyTorch从零实现极简MoE前馈网络，串起Router打分、Top-K选择、专家分组计算、index_add加权合并和负载均衡辅助损失，解释张量形状、离散索引与路由权重的梯度差异。通过逐Token参考实现和反向传播检查验证代码，讲清Top-1归一化陷阱、共享专家与生产级专家并行的边界，适合Transformer面试和大模型原理学习。
keywords: [MoE实现, PyTorch MoE, MoE FFN, Router实现, Top-K路由, index_add, 负载均衡损失, Load Balancing Loss, Router梯度, Top-1路由, 稀疏专家网络, Transformer手撕代码, SwiGLU, 专家并行]
tags: [Transformer, MoE, PyTorch, 手撕代码, 大模型面试]
---

# 手撕一个极简 MoE-FFN

上一篇[《MoE：稀疏模型为什么越来越流行》](./moe.md)讲了，总参数决定要存多少权重，激活参数帮助估算每个 Token 的计算。现在把视角缩到一个 MoE 层：**选出专家以后，Token 怎么送过去，输出怎么合回来？**

面试官让录友手写 MoE，常见做法是“先算所有专家，再乘一个 Top-K mask”。数值可能对，但所有专家都算了，稀疏计算的目的就没达到。应该先分组，再只计算选中的 Token。

这篇实现的是**单设备、无容量限制、不丢 Token 的教学版 Top-K MoE**。不包含 Attention、残差和 LayerNorm，可以把它理解成替换普通 FFN 的一个模块。

## 简要回答

- 把输入 `[B, S, D]` 展平为 `[T, D]`，其中 `T = B × S`。
- Router 为每个 Token 输出 N 个专家分数，Top-K 产生 `[T, K]` 的专家索引和权重。
- 按专家收集 Token，每个专家只调用一次，再用 `index_add` 将加权输出累加回原位置。
- 同时返回辅助损失和路由统计，验证前向结果、负载计数以及反向梯度。

**代码能跑还不够：要确认没有漏加专家输出，也没有把 Router 的梯度切断。**

## 详细实现

### 1. 先用一个 Token 算清权重

假设四个专家的概率是 `[0.1, 0.5, 0.3, 0.1]`，选 Top-2，就得到专家 `[1, 2]`。本篇对选中的概率重新归一化，权重为 `[0.625, 0.375]`：

```text
y = 0.625 × Expert-1(x) + 0.375 × Expert-2(x)
```

整批数据做的还是同一件事，只是要把选中相同专家的 Token 凑在一起。**每个 Token 都要向 K 个专家分发一次，合并时也要累加 K 次。**

### 2. 完整模块：分组计算，再加权合并

下面代码和后一节验证代码按顺序放进同一个 Python 文件即可运行，需要先安装 PyTorch。

专家沿用[《手撕 FFN》](./fnn_code.md)的两层线性层，这里选 GELU 激活，方便专注路由。它不是对 Qwen3.5 或 DeepSeek 架构的复刻，现代模型也常使用 SwiGLU 等门控 FFN。

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class Expert(nn.Module):
    def __init__(self, d_model, d_ff):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(d_model, d_ff, bias=False),
            nn.GELU(),
            nn.Linear(d_ff, d_model, bias=False),
        )

    def forward(self, x):
        return self.net(x)


class MoEFFN(nn.Module):
    def __init__(self, d_model, d_ff, num_experts=4, top_k=2):
        super().__init__()
        if not 1 <= top_k <= num_experts:
            raise ValueError("要求 1 <= top_k <= num_experts")
        self.num_experts = num_experts
        self.top_k = top_k
        self.router = nn.Linear(d_model, num_experts, bias=False)
        self.experts = nn.ModuleList(
            [Expert(d_model, d_ff) for _ in range(num_experts)]
        )

    def forward(self, x):
        if x.ndim != 3 or x.numel() == 0:
            raise ValueError("输入须为非空 [B, S, D] 张量")
        flat = x.reshape(-1, x.shape[-1])  # [T, D]

        # 先用 FP32 计算概率，降低低精度路由的数值风险。
        logits = self.router(flat).float()  # [T, N]
        probs = torch.softmax(logits, dim=-1)
        selected_probs, indices = probs.topk(self.top_k, dim=-1)
        weights = selected_probs / selected_probs.sum(-1, keepdim=True)

        output = torch.zeros_like(flat)
        for expert_id, expert in enumerate(self.experts):
            # 两个一维索引分别表示 Token 行号、Top-K 中的列号。
            token_ids, slots = torch.where(indices == expert_id)
            if token_ids.numel() == 0:
                continue
            expert_out = expert(flat[token_ids])  # [命中数, D]
            gate = weights[token_ids, slots].unsqueeze(-1)
            weighted = expert_out * gate.to(expert_out.dtype)
            output = output.index_add(0, token_ids, weighted)

        # 按 T*K 次专家分配计数；离散计数不参与求导。
        counts = torch.bincount(
            indices.reshape(-1), minlength=self.num_experts
        )
        fraction = counts.float() / indices.numel()
        mean_prob = probs.mean(dim=0)
        aux_loss = self.num_experts * (fraction * mean_prob).sum()

        info = {
            "indices": indices,       # [T, K]
            "weights": weights,       # [T, K]，保留梯度供验证
            "counts": counts,         # [N]，统计专家负载
        }
        return output.reshape_as(x), aux_loss, info
```

这里有三个容易写错的地方。

**第一，Token 行号和 Top-K 列号不是一回事。**例如 `indices[3, 1] == 2`，表示第 3 个 Token 的第二个选择是 Expert-2。`torch.where` 同时拿到这两个坐标，就能取出对应输入和权重，不需要再对每个 Token 写 Python 循环。

**第二，同一个 Token 会被不同专家贡献多次，必须累加。**普通赋值会覆盖前面的结果；`index_add` 按 Token 行号把结果加回去，保留这条计算路径的梯度。它的索引累加语义见 [PyTorch 官方说明](https://docs.pytorch.org/docs/stable/generated/torch.Tensor.index_add_.html)。

**第三，负载损失使用全部专家的概率。**这里令 `fraction = counts / (T*K)`，损失没有预先乘系数；训练时再加 `alpha * aux_loss`。均匀分配时它的参考值为 1，不能因为它不趋近 0 就认为训练失败。它也不是准确衡量负载差异的唯一指标。

### 3. 验证：和逐 Token 参考实现对齐，再看梯度

只打印 `[B, S, D]` 的输出形状，发现不了“少加一个专家”的错误。下面用较慢但直观的逐 Token 写法做参照，再检查主任务能否给 Router 非零梯度。

```python
if __name__ == "__main__":
    torch.manual_seed(7)
    moe = MoEFFN(d_model=8, d_ff=16, num_experts=4, top_k=2)
    x = torch.randn(2, 3, 8, requires_grad=True)
    y, aux_loss, info = moe(x)

    assert y.shape == x.shape
    assert info["counts"].sum().item() == 2 * 3 * 2
    torch.testing.assert_close(
        info["weights"].sum(-1), torch.ones(6)
    )

    # 参考实现：每个 Token 分别调用选中的 K 个专家。
    with torch.no_grad():
        flat = x.reshape(-1, 8)
        reference = torch.zeros_like(flat)
        for t in range(flat.shape[0]):
            for slot in range(moe.top_k):
                expert_id = info["indices"][t, slot].item()
                reference[t] += (
                    info["weights"][t, slot]
                    * moe.experts[expert_id](flat[t:t + 1])[0]
                )
        torch.testing.assert_close(y, reference.reshape_as(y))

    target = torch.randn_like(y)  # 仅用于验证训练链路
    main_loss = F.mse_loss(y, target)
    main_grad = torch.autograd.grad(
        main_loss, moe.router.weight, retain_graph=True
    )[0]
    assert torch.isfinite(main_grad).all()
    assert main_grad.abs().sum().item() > 0

    optimizer = torch.optim.AdamW(moe.parameters(), lr=1e-3)
    total_loss = main_loss + 0.01 * aux_loss
    optimizer.zero_grad()
    total_loss.backward()
    assert x.grad is not None and torch.isfinite(x.grad).all()
    for expert_id, expert in enumerate(moe.experts):
        if info["counts"][expert_id] > 0:
            assert all(
                p.grad is not None and torch.isfinite(p.grad).all()
                for p in expert.parameters()
            )
    optimizer.step()
    print("前向对齐、路由计数和反向传播检查通过")
```

这段验证证明的是分组计算与参考结果一致、梯度链路通畅。**它没有证明模型学到了某种语义分工。**固定几条随机输入训练到低损失，也可能只是记住样本；要研究专家分工，需要独立验证集和实际路由统计，不能预言某一类输入一定去某个编号的专家。

### 4. 为什么 Top-1 要格外小心？

把 `top_k=2` 改成 `top_k=1`，这份代码依然可以前向计算，但有一个训练陷阱：

$$
w=\frac{p}{p}=1
$$

**在这套“选中后重新归一化”的实现里，Top-1 权重恒为 1，主任务不能经由这个权重训练 Router。**专家索引是离散选择，也不能直接对“选了哪个专家”求普通梯度。此时辅助损失仍可以通过完整概率给 Router 提供梯度，但不等于它获得了有效的主任务路由信号。

这不代表所有 Top-1 MoE 都有同样的问题。比如 Switch Transformer 使用所选专家的门控概率参与输出缩放，没有把它直接除成 1。[Switch Transformer 原论文](https://arxiv.org/abs/2101.03961)

读代码时要分别追两条线：**索引决定去哪里算，权重决定算完怎么合并。**只检查“Router 有梯度”，还可能把辅助损失的梯度误当成主任务的梯度。

## 工程边界：这段代码距离生产还有多远？

这份实现先路由、再算命中的 Token，避免了先计算所有专家的浪费。但它仍有 Python 专家循环、动态索引和多次输出累加。GPU 上的小矩阵、kernel 启动和索引开销，可能把收益吃掉；多卡时还没有 Token 的跨设备分发。

| 遇到的问题 | 常见处理 | 需要验证的代价 |
|---|---|---|
| 专家计算太碎 | Token 排序、分组矩阵乘法、算子融合 | 吞吐、临时内存、数值精度 |
| 少数专家过载 | 辅助损失、负载驱动的路由偏置 | 主任务质量、负载峰值、通信等待 |
| 专家分布到多卡 | 专家并行、分发与合并、通信计算重叠 | 跨节点流量、尾延迟、每卡峰值显存 |

真实语言模型训练还要处理 padding：路由分配、负载计数和平均概率都应明确哪些位置有效。本篇假设所有输入位置都是有效 Token，没有提供 padding mask。

现代 MoE 的变化也不只是把 `num_experts` 调大：Qwen3.5 有共享专家，DeepSeek-V3 使用负载驱动的路由偏置。它们的具体打分与加权规则要读各自实现，不能直接拿这个 Softmax 教学版替换。[Qwen3.5 模型卡](https://huggingface.co/Qwen/Qwen3.5-397B-A17B)、[DeepSeek-V3 技术报告](https://arxiv.org/abs/2412.19437)

## 知识拓展

**Q1：没被当前 batch 选中的专家会收到主任务梯度吗？**

这份实现不会对它做前向计算，因此它没有这次主任务的参数梯度；辅助损失只影响 Router，不直接训练专家。长期闲置才是需要关注的问题，不能要求每个小 batch 都恰好用满所有专家。

**Q2：Top-K 不可导，为什么还说 Router 可以训练？**

Top-K 返回的专家索引不可导，但所选概率值仍能参与可导计算。Top-2 时，主任务可以通过加权输出更新 Router；辅助损失还提供另一条经由完整概率的路径。选择边界本身仍是离散的。

**Q3：加共享专家，就是再多选一个 Top-K 吗？**

不是。共享专家通常对所有 Token 计算，不参与同一套稀疏选择。它的输出如何与路由专家合并、是否有单独门控，取决于模型设计，参数和计算也要单独计入。

**Q4：辅助损失越小，模型就越好吗？**

不能这么判断。应同时看主任务验证损失、专家计数分布、最大负载和吞吐。过强的均衡约束可能改变有用的路由偏好，系数 `0.01` 只是示例，需要调参。

录友手撕到最后，要能沿着一个 Token 解释：**它选了谁，用什么权重，加回哪一行，主任务梯度又从哪里回去。**
