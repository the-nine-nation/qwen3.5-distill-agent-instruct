<p align="center">
  <a href="README.md">English</a> | <strong>简体中文</strong>
</p>

<p align="center">
  <img src="assets/qwen36-instruct-banner.png" alt="Qwen3.5 Distill Agent Instruct——快速、直接的指令遵循与工具执行" width="100%">
</p>

<h1 align="center">Qwen3.5 Distill Agent Instruct</h1>

<p align="center">
  <strong>Thinking 是一种能力，而不应成为一种税。</strong><br>
  为无需冗长思维链的真实任务，打造快速、直接、具备工具调用能力的语言模型。
</p>

<p align="center">
  <a href="https://huggingface.co/lzy510016411/qwen3.5-9b-distill-agent-instruct"><img alt="模型" src="https://img.shields.io/badge/%F0%9F%A4%97_%E6%A8%A1%E5%9E%8B-Qwen3.5--9B--Distill--Agent-FFD21E"></a>
  <a href="https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces"><img alt="数据集" src="https://img.shields.io/badge/%F0%9F%A4%97_%E6%95%B0%E6%8D%AE%E9%9B%86-Mixed--Agent--Traces-FFD21E"></a>
  <a href="LICENSE"><img alt="许可证" src="https://img.shields.io/badge/License-Apache_2.0-blue.svg"></a>
  <a href="https://github.com/the-nine-nation/qwen3.5-distill-agent-instruct/stargazers"><img alt="GitHub Stars" src="https://img.shields.io/github/stars/the-nine-nation/qwen3.5-distill-agent-instruct?style=flat"></a>
</p>

> [!IMPORTANT]
> **Qwen3.5-9B-Distill-Agent-Instruct 是基于 Qwen3.5-9B 构建的社区蒸馏后训练项目，并非 Qwen 官方发布的模型。**

## Instruct 模型并没有过时

当一个问题确实需要深度探索时，推理模型非常强大。但在许多真实系统中，并不需要让每一个请求都变成一段漫长的内部旅程。

API 路由器需要选对调用；Agent 运行时需要合法参数；代码助手需要精确修改；交互式产品则必须在用户失去耐心之前给出答案。在这些场景中，不必要的 Thinking 会消耗延迟、Token、资金，有时还会降低可靠性。

我们继续坚持 **Instruct** 路线，因为“直接完成任务”本身就是一种一等能力：

- 遵循指令，而不是把每项任务都变成研究项目；
- 在工具有用时调用工具，在不需要时克制调用；
- 输出生产系统可以稳定消费的结构化结果；
- 将响应速度和服务成本控制在合理范围；
- 把深度推理留给真正需要它的问题。

**我们的目标不是减少智能，而是让智能拥有合适的挡位。**

## 发布内容

| 发布物 | 内容 | 链接 |
|---|---|---|
| **Qwen3.5-9B-Distill-Agent-Instruct** | 已合并的 BF16 模型，无需单独加载适配器 | [Hugging Face 模型](https://huggingface.co/lzy510016411/qwen3.5-9b-distill-agent-instruct) |
| **Fable5 · GPT-5.5 · Opus-4.7 Mixed Agent Traces** | 20,409 条经过作者专属清洗的后训练数据 | [Hugging Face 数据集](https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces) |
| **Qwen3.5-9B** | 上游基础模型 | [基础模型](https://huggingface.co/Qwen/Qwen3.5-9B) |

## Qwen3.5-9B-Distill-Agent-Instruct 面向什么场景

- 直接遵循指令并生成有效最终答案
- Function Calling 与工具调用边界判断
- 多工具及并行工具调用
- 多步 Agent 执行与错误恢复
- 面向 Agent 运行时的结构化输出
- 代码生成与任务完成
- 继承 262,144 Token 上下文基础的长上下文应用

语言模型使用 **秩稳定 LoRA（rsLoRA）** 对所有语言模型线性层进行后训练。在本阶段中，视觉塔和多模态对齐器保持冻结。

| 训练参数 | 配置 |
|---|---|
| 基础模型 | `Qwen/Qwen3.5-9B` |
| LoRA rank / alpha / dropout | `64 / 128 / 0.05` |
| 训练轮数 | `3` |
| 最大序列长度 | `32,768` |
| 学习率 | `1e-4`，Cosine 调度 |
| 精度 | BF16 |
| 训练运行时 | Gradient Checkpointing、Liger Kernel、DeepSpeed ZeRO-3 |

完整的目标投影层、优化参数、架构说明、预期用途和限制，请参阅[完整模型卡](https://huggingface.co/lzy510016411/qwen3.5-9b-distill-agent-instruct)。

## BFCL 工具调用评测

本次后训练评测使用 **Berkeley Function Calling Leaderboard（BFCL）** Non-Live 测试集。`Δ` 表示相较 Qwen3.5-9B 基础模型的绝对百分点变化。

| 类别 | 基础模型 | Distill Agent Instruct | Δ |
|---|---:|---:|---:|
| Simple Python（400） | **92.00%** | 91.50% | -0.50 pp |
| Multiple（200） | 94.00% | **96.00%** | **+2.00 pp** |
| Parallel（200） | 85.50% | **90.50%** | **+5.00 pp** |
| Parallel Multiple（200） | **87.00%** | 86.00% | -1.00 pp |
| Irrelevance（240） | 83.75% | **89.17%** | **+5.42 pp** |
| **Non-Live Overall** | 74.29% | **75.75%** | **+1.46 pp** |

在同一套评测环境中，平均延迟从 **2.96 秒降至 2.54 秒**，约降低 **14.2%**；P95 延迟从 **6.07 秒降至 5.60 秒**，约降低 **7.7%**。延迟会受硬件、服务引擎、批处理、生成参数和输出长度影响，因此这些结果只能在同一测试配置下进行比较。

增益并非在所有类别上都保持一致：Simple Python 和 Parallel Multiple 出现了轻微回退。我们选择公布完整结果，因为诚实的模型开发需要展示真实取舍，而不只是一个好看的总分。

## 快速运行

使用支持 Qwen3.5 的较新版本 vLLM 启动已合并模型：

```bash
vllm serve lzy510016411/qwen3.5-9b-distill-agent-instruct \
  --port 8000 \
  --tensor-parallel-size 1 \
  --max-model-len 262144 \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder
```

如果显存有限，请降低 `--max-model-len`。对于纯文本任务，可添加 `--language-model-only` 跳过视觉模块分析，将更多显存留给 KV Cache。

随后即可调用 OpenAI 兼容接口：

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="local")

response = client.chat.completions.create(
    model="lzy510016411/qwen3.5-9b-distill-agent-instruct",
    messages=[
        {"role": "user", "content": "请总结低延迟 Instruct 模型为何依然重要。"}
    ],
    temperature=0.2,
)

print(response.choices[0].message.content)
```

进行工具调用时，请通过 OpenAI 兼容接口的 `tools` 字段传入 JSON Schema 工具定义，并在多轮对话中保持工具调用与工具响应的正确顺序。

## 模型背后的数据

[Fable5 · GPT-5.5 · Opus-4.7 Mixed Agent Traces](https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces) 混合了长程 Agent 轨迹、步骤级工具调用样本、Function Calling 对话、代码指令、轨迹反演样本，以及经过平衡的“调用工具还是直接回答”监督数据。名称突出三个主要的模型标注来源；`mixed` 还涵盖 GLM-5.2、Qwen3.7-Max、Glaive、Hermes、When2Call 与代码数据。

公开版本包含：

- **20,409** 条训练记录
- **211,580** 个消息事件
- **67,865** 个显式工具调用步骤
- **11,580** 条包含工具定义的记录

作者专属处理流程涵盖 Schema 归一化、轨迹完整性检查、有效步骤切分、Judge 门控过滤、来源感知采样、结构与内容去重，以及工具使用再平衡。完整的数据组成及限制请阅读[数据集卡片](https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces)。

## 路线图

- [x] 发布合并后的 Qwen3.5-9B-Distill-Agent-Instruct 模型
- [x] 发布经过清洗的轨迹混合数据集
- [x] 公布 BFCL Non-Live 结果及延迟观测
- [ ] 将可复现评测扩展至 Function Calling 之外
- [ ] 添加量化部署方案
- [ ] 在不牺牲 Irrelevance 判断的前提下改进 Parallel Multiple 调用
- [ ] 探索 Qwen3.5 Distill Agent Instruct 系列的更多参数规模
- [ ] 继续证明快速、直接的模型值得获得严肃的后训练投入

## 加入项目

我们欢迎独立评测复现、部署方案、失败样本、新基准、推理框架集成、量化报告，以及拥有清晰授权的高质量指令或 Agent 数据。提交 Issue 或 Pull Request 前，请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

如果你认同这个方向，请为仓库点亮 **Star**、亲自测试模型、公开你的评测结果，并告诉我们它在哪些地方失败。真正有用的 Instruct 模型，需要通过真实任务来衡量，而不是靠口号取胜。

## 安全与限制

工具执行必须保持在沙箱中，并接受应用层授权控制。Function Calling 分数无法证明模型事实正确、安全，或适合在高风险场景中自主行动。部署之前，请阅读模型卡和数据集卡，并在自己的业务领域内完成验证。

## 许可证与致谢

仓库代码与文档采用 [Apache License 2.0](LICENSE)。模型使用遵循 [Qwen3.5-9B 基础模型](https://huggingface.co/Qwen/Qwen3.5-9B/blob/main/LICENSE)发布的许可证。数据集的不同组成部分可能带有各自的上游条款，用户在重新分发或商业使用前有责任完成审核。

项目建立在对 Qwen 开放模型生态的尊重之上，也坚持一个朴素但坚定的信念：**快速的指令遵循依然重要。**
