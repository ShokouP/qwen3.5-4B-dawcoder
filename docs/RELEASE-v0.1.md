# qwen3.5-4B-dawcoder v0.1：把音乐制作意图翻译成 DAW 参数（4B 参数 / Qwen3.5-4B 基座 / GGUF Q4_K_M，三代模型）

[![GitHub stars](https://img.shields.io/github/stars/ShokouP/qwen3.5-4B-dawcoder?style=social)](https://github.com/ShokouP/qwen3.5-4B-dawcoder)
[![Hugging Face](https://img.shields.io/badge/HF-dawcoder-yellow)](https://huggingface.co/Shokou1/qwen3.5-4B-dawcoder)

如果这个项目对你有用，欢迎给仓库点个 Star，让更多做音乐 AI / Agent 的人找到它。

**4B 参数 · 基座 Qwen3.5-4B · LoRA r=64 · GGUF Q4_K_M（约 2.52 GB/个）· 上下文 16K · Apache-2.0**

DAWcoder 是一个 **4B 音乐制作参数模型**（Qwen3.5-4B + LoRA）：输入「工程状态 + 制作意图」，输出**可执行、可校验的 DAW 参数补丁 JSON**。不生成音频，只生成工程参数。

## 本 Release 包含三代模型（Q4_K_M，各约 2.52 GB）

| 代号 | 训练 | 下载 |
|---|---|---|
| preview（SFT v1） | CPT 37.1M tokens + 3,240 条任务 SFT | `dawcoder-preview-Q4_K_M.gguf` |
| sft-v2 | 完整序列 SFT 续训（100 steps，零截断） | `dawcoder-sft-v2-Q4_K_M.gguf` |
| grpo-v3 | sft-v2 上 GRPO（24 步，KL β=0.05） | `dawcoder-grpo-v3-Q4_K_M.gguf` |

## 固定评测（val 60 条均衡，seed 20261107，temp 0，num_predict 8192）

| 模型 | JSON 解析 | schema 合法 | mix | form | texture | reward |
|---|---|---|---|---|---|---|
| preview | 100% | 100% | 20/20 | 20/20 | 20/20 | 0.9657 |
| **sft-v2** | 100% | 100% | 20/20 | 20/20 | 20/20 | **0.9710** |
| **grpo-v3** | 100% | 100% | 20/20 | 20/20 | 20/20 | **0.9710** |

> schema 合法 ≠ 混音正确；reward 为与金标准 patch 的相似度，不代表听感。详见模型卡与 `eval/COMPARISON-REPORT.md`。

## 用法（Ollama）

```bash
# 下载对应 GGUF 与本 Release 中的 Modelfile
ollama create dawcoder-grpo-v3 -f Modelfile.dawcoder-grpo-v3
ollama run dawcoder-grpo-v3
```

必须使用模型卡中的 system prompt（Modelfile 已内置）；采样参数：num_ctx 16384 / temperature 0 / top_p 0.95 / num_predict 8192。

输入示例：

```json
{"task":"mix","state":{"genre":"pop","tempo":120,"tracks":[{"name":"Lead Vocal","role":"vocal","volume_db":-8.0,"pan":0.0}]},"intent":"人声被鼓盖住了，帮我推出来一点，别太吵"}
```

## 适用场景

- 混音参数建议（EQ / 压缩 / 音量 / 声像 / send 的 patch 候选）
- 曲式结构草稿（sections + markers）
- 织体/琶音 note 序列
- Agent 工作流：模型出 patch → schema 校验 → DAW 执行 → 渲染测量（可配合 [daw-mcp-server](https://github.com/ShokouP/daw-mcp-server)）

## 限制

- 只生成参数，不生成音频；不能替代混音师；
- 合成任务数据为主，真实工程泛化有限；第三方插件参数映射未覆盖；
- 中文意图支持有限；
- sft-v2 与 grpo-v3 在当前 60 条评测上并列，差异不可区分。

## 相关仓库

- 训练与评测：[dawcoder-datasets](https://github.com/ShokouP/qwen3.5-4B-dawcoder-datasets)
- 文本语料：[open-music-production-corpus](https://github.com/ShokouP/open-music-production-corpus)
- 工程/符号数据：[open-music-engineering-data](https://github.com/ShokouP/open-music-engineering-data)
- DAW MCP 工具：[daw-mcp-server](https://github.com/ShokouP/daw-mcp-server)

## 许可

Apache-2.0（基座 Qwen3.5-4B）。训练数据许可见模型卡。
