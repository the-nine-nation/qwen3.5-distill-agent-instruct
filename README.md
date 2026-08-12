<p align="center">
  <strong>English</strong> | <a href="README.zh-CN.md">简体中文</a>
</p>

<p align="center">
  <img src="assets/qwen36-instruct-banner.png" alt="Qwen3.5 Distill Agent Instruct — fast, direct instruction following and tool execution" width="100%">
</p>

<h1 align="center">Qwen3.5 Distill Agent Instruct</h1>

<p align="center">
  <strong>Thinking is a capability. It should not be a tax.</strong><br>
  Fast, direct, tool-ready language models for the work that does not need a long chain of thought.
</p>

<p align="center">
  <a href="https://huggingface.co/lzy510016411/qwen3.5-9b-distill-agent-instruct"><img alt="Model" src="https://img.shields.io/badge/%F0%9F%A4%97_Model-Qwen3.5--9B--Distill--Agent-FFD21E"></a>
  <a href="https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces"><img alt="Dataset" src="https://img.shields.io/badge/%F0%9F%A4%97_Dataset-Mixed--Agent--Traces-FFD21E"></a>
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/License-Apache_2.0-blue.svg"></a>
  <a href="https://github.com/the-nine-nation/qwen3.5-distill-agent-instruct/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/the-nine-nation/qwen3.5-distill-agent-instruct?style=flat"></a>
</p>

> [!IMPORTANT]
> **Qwen3.5-9B-Distill-Agent-Instruct is a community distilled post-training project built from Qwen3.5-9B. It is not an official Qwen release.**

## The Instruct model is not obsolete

Reasoning models are extraordinary when a problem genuinely benefits from deliberate exploration. But many real systems do not need every request to become a long internal journey.

An API router needs the right call. An agent runtime needs valid arguments. A coding assistant needs a precise edit. An interactive product needs an answer before the user loses momentum. In these settings, unnecessary thinking costs latency, tokens, money, and sometimes reliability.

We are continuing the **Instruct** line because directness is a first-class capability:

- follow the instruction without turning every task into a research project;
- call tools when they are useful—and refrain when they are not;
- emit stable, structured outputs that production systems can consume;
- keep response time and serving cost practical;
- reserve deep reasoning for the requests that actually need it.

**The goal is not less intelligence. It is intelligence with a transmission.**

## Releases

| Artifact | What it contains | Link |
|---|---|---|
| **Qwen3.5-9B-Distill-Agent-Instruct** | BF16 merged checkpoint; no separate adapter required | [Model on Hugging Face](https://huggingface.co/lzy510016411/qwen3.5-9b-distill-agent-instruct) |
| **Fable5 · GPT-5.5 · Opus-4.7 Mixed Agent Traces** | 20,409 author-curated post-training records | [Dataset on Hugging Face](https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces) |
| **Qwen3.5-9B** | Upstream base checkpoint | [Base model](https://huggingface.co/Qwen/Qwen3.5-9B) |

## What Qwen3.5-9B-Distill-Agent-Instruct is built for

- Direct instruction following and useful final answers
- Function calling and tool-selection boundaries
- Multiple and parallel tool calls
- Multi-step agent execution and recovery
- Structured output for agent runtimes
- Coding and task completion
- Long-context applications, inheriting the 262,144-token context foundation

The language model was post-trained with **rank-stabilized LoRA (rsLoRA)** across all language-model linear layers. The vision tower and multimodal aligner were frozen during this stage.

| Training parameter | Value |
|---|---|
| Base model | `Qwen/Qwen3.5-9B` |
| LoRA rank / alpha / dropout | `64 / 128 / 0.05` |
| Epochs | `3` |
| Maximum sequence length | `32,768` |
| Learning rate | `1e-4`, cosine schedule |
| Precision | BF16 |
| Runtime | Gradient checkpointing, Liger Kernel, DeepSpeed ZeRO-3 |

See the [full model card](https://huggingface.co/lzy510016411/qwen3.5-9b-distill-agent-instruct) for target projections, optimization details, architecture notes, intended use, and limitations.

## BFCL tool-calling results

The supplied post-training evaluation uses the **Berkeley Function Calling Leaderboard (BFCL)** non-live test set. `Δ` is the absolute change from the Qwen3.5-9B base checkpoint.

| Category | Base | Distill Agent Instruct | Δ |
|---|---:|---:|---:|
| Simple Python (400) | **92.00%** | 91.50% | -0.50 pp |
| Multiple (200) | 94.00% | **96.00%** | **+2.00 pp** |
| Parallel (200) | 85.50% | **90.50%** | **+5.00 pp** |
| Parallel Multiple (200) | **87.00%** | 86.00% | -1.00 pp |
| Irrelevance (240) | 83.75% | **89.17%** | **+5.42 pp** |
| **Non-Live Overall** | 74.29% | **75.75%** | **+1.46 pp** |

In the same reported setup, mean latency decreased from **2.96 s to 2.54 s** (about **14.2%**) and P95 latency from **6.07 s to 5.60 s** (about **7.7%**). These latency numbers are setup-dependent and should only be compared under the same hardware, serving engine, batching, generation parameters, and output-length distribution.

The gains are not uniform: simple Python and parallel-multiple cases regress slightly. We publish the full table because honest model development needs visible trade-offs, not only a headline score.

## Run it

Serve the merged checkpoint with a recent vLLM build that supports Qwen3.5:

```bash
vllm serve lzy510016411/qwen3.5-9b-distill-agent-instruct \
  --port 8000 \
  --tensor-parallel-size 1 \
  --max-model-len 262144 \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder
```

If memory is limited, lower `--max-model-len`. For text-only workloads, add `--language-model-only` to avoid vision profiling and reserve more memory for the KV cache.

Then call the OpenAI-compatible endpoint:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="local")

response = client.chat.completions.create(
    model="lzy510016411/qwen3.5-9b-distill-agent-instruct",
    messages=[
        {"role": "user", "content": "Summarize why low-latency instruct models still matter."}
    ],
    temperature=0.2,
)

print(response.choices[0].message.content)
```

For tool calling, pass JSON-schema tool definitions through the OpenAI-compatible `tools` field and preserve tool-call/tool-response ordering across turns.

## The data behind the model

[Fable5 · GPT-5.5 · Opus-4.7 Mixed Agent Traces](https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces) combines long-horizon agent traces, step-level tool-use examples, function-calling conversations, code instructions, trace inversion, and balanced call-versus-answer supervision. Its name highlights three principal model-labelled sources while `mixed` covers the additional GLM-5.2, Qwen3.7-Max, Glaive, Hermes, When2Call, and code components.

The public release contains:

- **20,409** training records
- **211,580** message events
- **67,865** explicit tool-call steps
- **11,580** records with tool definitions

Its author-curated pipeline includes schema normalization, trajectory-integrity checks, meaningful step slicing, judge-gated filtering, source-aware sampling, structural/content deduplication, and tool-use rebalancing. Read the [dataset card](https://huggingface.co/datasets/lzy510016411/fable5-gpt5.5-opus4.7-mixed-agent-traces) for the full mixture composition and limitations.

## Roadmap

- [x] Release the merged Qwen3.5-9B-Distill-Agent-Instruct checkpoint
- [x] Release the curated trajectory mixture
- [x] Publish BFCL non-live results and latency observations
- [ ] Expand reproducible evaluations beyond function calling
- [ ] Add quantized deployment recipes
- [ ] Improve parallel-multiple calls without sacrificing irrelevance detection
- [ ] Explore additional sizes in the Qwen3.5 Distill Agent Instruct family
- [ ] Keep proving that fast, direct models deserve serious post-training

## Join the project

Useful contributions include independent eval reproductions, deployment recipes, failure cases, new benchmarks, inference integrations, quantization reports, and carefully licensed instruction or agent data. See [CONTRIBUTING.md](CONTRIBUTING.md) before opening an issue or pull request.

If this direction resonates with you, **star the repository, test the model, publish your results, and tell us where it fails.** A useful Instruct model is built by measuring real work—not by winning a slogan contest.

## Safety and limitations

Tool execution must remain sandboxed and subject to application-level authorization. Function-calling scores do not establish factual correctness, safety, or suitability for autonomous high-stakes actions. Review the model and dataset cards before deployment, and validate behavior in your own domain.

## License and attribution

Repository code and documentation are released under the [Apache License 2.0](LICENSE). Model use follows the license published with the [base Qwen3.5-9B checkpoint](https://huggingface.co/Qwen/Qwen3.5-9B/blob/main/LICENSE). Dataset components may carry their own upstream terms; users are responsible for reviewing them before redistribution or commercial use.

Built with respect for the Qwen open-model ecosystem—and with a stubborn belief that **fast instruction following still matters**.
