---
language:
  - en
  - zh
license: apache-2.0
base_model: Qwen/Qwen3.5-4B
tags:
  - music
  - music-production
  - daw
  - dawcoder
  - gguf
  - ollama
  - qwen
  - lora
  - text-generation
library_name: transformers
pipeline_tag: text-generation
---

# DAWcoder-4B：Qwen3.5-4B 基座 · 4B 参数 · GGUF Q4_K_M · Ollama 开箱即用（preview / sft-v2 / grpo-v3）

[![GitHub stars](https://img.shields.io/github/stars/ShokouP/dawcoder?style=social)](https://github.com/ShokouP/dawcoder)
[![HF Model](https://img.shields.io/badge/HuggingFace-Shokou1%2Fdawcoder-yellow)](https://huggingface.co/Shokou1/dawcoder)
[![License](https://img.shields.io/badge/license-Apache--2.0-lightgrey)](LICENSE)

**4B 参数 · 基座 Qwen3.5-4B · LoRA r=64 · GGUF Q4_K_M（约 2.52 GB/个）· 上下文 16K · Apache-2.0**

> 如果这个项目对你有用，欢迎给仓库点个 Star，让更多做音乐 AI / Agent 的人找到它。

DAWcoder 是一个 **4B 音乐制作参数模型**：输入「工程状态 + 自然语言制作意图」，输出**可执行、可校验、可回滚的 DAW 参数补丁 JSON**。它不生成音频，只生成工程参数——可直接在 DAW 里执行、被 schema 校验、被渲染指标闭环验证。

## 模型规格

| 项 | 值 |
|---|---|
| **参数量** | **4B** |
| **基座模型** | **Qwen3.5-4B**（Apache-2.0） |
| 微调方式 | LoRA r=64，all-linear，bf16 训练 |
| **量化发布** | **GGUF Q4_K_M**（约 2.52 GB / 个，Ollama 直接加载） |
| 上下文长度 | 16,384 tokens |
| 输出长度 | 最多 8,192 tokens |
| 采样参数 | temperature 0 / top_p 0.95 / repeat_penalty 1.0 |
| 许可 | Apache-2.0（保留 Qwen3.5-4B 原许可声明） |

## 三代权重（同一基座，逐代后训练）

| 代号 | 训练阶段 | GGUF 文件 |
|---|---|---|
| **preview**（SFT v1） | CPT 37.1M tokens + 3,240 条任务 SFT | `dawcoder-preview-Q4_K_M.gguf` |
| **sft-v2** | 完整序列 SFT 续训（100 optimizer steps，零截断） | `dawcoder-sft-v2-Q4_K_M.gguf` |
| **grpo-v3** | sft-v2 上 GRPO 24 步（KL β=0.05） | `dawcoder-grpo-v3-Q4_K_M.gguf` |

## 评测对比（固定口径：val 60 条均衡，seed 20261107，temp 0，num_predict 8192）

| 模型 | JSON 解析 | schema 合法 | mix | form | texture | reward(overall) |
|---|---|---|---|---|---|---|
| preview（SFT v1） | 100% | 100% | 20/20 | 20/20 | 20/20 | 0.9657 |
| **sft-v2** | 100% | 100% | 20/20 | 20/20 | 20/20 | **0.9710** |
| **grpo-v3** | 100% | 100% | 20/20 | 20/20 | 20/20 | **0.9710** |

- **schema 合法率**：输出能被 `patch_validation` 校验通过（字段、范围、任务结构）。
- **reward**：0.2 JSON 解析 + 0.35 schema 合法 + 0.45 与金标准 patch 的相似度（mix 参数距离 / form 段名与 bars / texture note 重合）。
- 相对 preview，sft-v2 / grpo-v3 在 mix 上提升（reward 0.9097 → 0.9246），form/texture 持平，无任务退化；sft-v2 与 grpo-v3 在当前 60 条粒度上并列。

> 评测只衡量"格式正确 + 与参考参数接近"，**不代表听感或混音质量**；真实混音需人工 A/B 或 DAW 渲染闭环。

## Ollama 快速开始

```bash
# 1) 下载对应的 GGUF（本页 Files / GitHub Release）
#    GitHub 附件为 2GB 分卷，按 docs/MERGE-GGUF.md 合并

# 2) 用仓库里的 Modelfile 创建模型（已内置 system prompt 与采样参数）
ollama create dawcoder-grpo-v3 -f Modelfile.dawcoder-grpo-v3

# 3) 运行
ollama run dawcoder-grpo-v3
```

输入（user 消息是 JSON）：

```json
{"task":"mix","state":{"genre":"pop","tempo":120,"tracks":[{"name":"Lead Vocal","role":"vocal","volume_db":-8.0,"pan":0.0}]},"intent":"人声被鼓盖住了，帮我推出来一点，别太吵"}
```

输出示例（mix 任务）：

```json
{"ops":[{"op":"set_track","track":"Lead Vocal","fields":{"volume_db":-6.5}},{"op":"set_eq","track":"Pad","fields":{"band":2,"type":"bell","freq":315,"gain_db":-1.5,"q":1.0}}]}
```

- `task=mix` → `{"ops":[...]}`：`set_track` / `set_eq` / `set_comp` / `set_send` / `add_marker`
- `task=form` → `{"sections":[...],"markers":[...]}`
- `task=texture` → `{"notes":[{"bar","step","key","len_steps","vel"}]}`

## 三代演进

| 代 | 训练 | 关键点 |
|---|---|---|
| **preview** | CPT（37.1M tokens，400 steps）+ SFT（3,240 条，200 steps） | 首个可用的意图→参数模型；schema 100% |
| **sft-v2** | 从 preview 的 SFT adapter 续训：完整序列 SFT（max seq 4096，零截断），100 optimizer steps，lr 1e-5 | mix reward 提升；训练 loss mix 0.122 / form 0.168 / texture 0.146 |
| **grpo-v3** | 从 sft-v2 出发 GRPO：24 步、100 rollouts（99 条 schema 合法，1 条超长 texture 截断未入 RL），KL β=0.05，lr 1e-6 | 与 sft-v2 固定评测并列；训练组 reward mix 0.979 / form 0.914 / texture 0.942 |

## 适用场景

- **混音参数建议**：EQ / 压缩 / 音量 / 声像 / send 的 patch 候选（人工确认后执行）；
- **曲式结构草稿**：Intro/Verse/Chorus/Bridge 的 sections + markers；
- **织体/琶音生成**：texture 任务的 note 序列（bar/step/key/len/vel）；
- **Agent 工作流中间层**：模型输出 patch → schema 校验 → 在 DAW 中执行 → 渲染测量；
- **训练/评测研究**：可搭配 [daw-mcp-server](https://github.com/ShokouP/daw-mcp-server) 做闭环验证。

## 限制（务必阅读）

- **仅参数，不生成音频**；不替代混音师判断；
- 训练数据以**合成任务数据**为主，真实工程泛化有限；第三方插件参数映射未覆盖（当前为抽象 ops）；
- 中文意图支持有限（SFT 以英文/JSON 为主，中文主要来自 CPT 语料）；
- 固定评测仅 60 条，sft-v2 与 grpo-v3 的差异在当前粒度不可区分；
- schema 合法 ≠ 混音正确；参数建议应人工确认或经渲染指标验证；
- 受版权内容（REAPER / FL Studio 手册等）不在训练语料中。

## 数据与许可

- 基座：Qwen3.5-4B（Apache-2.0），本模型权重按 **Apache-2.0** 发布并保留原声明；
- 训练语料：CC BY-SA 4.0 / 公版 / MIT / CC0（见 [open-music-production-corpus](https://github.com/ShokouP/open-music-production-corpus)）；
- 任务数据：自建合成数据（CC0），见 [dawcoder-datasets](https://github.com/ShokouP/dawcoder-datasets)；
- 工程/符号数据：[open-music-engineering-data](https://github.com/ShokouP/open-music-engineering-data)；
- DAW 自动化/评测工具：[daw-mcp-server](https://github.com/ShokouP/daw-mcp-server)。

## 引用

```bibtex
@misc{dawcoder2026,
  title  = {DAWcoder: A 4B Music-Production Parameter Model (Qwen3.5-4B + LoRA, GGUF)},
  author = {ShokouP},
  year   = {2026},
  url    = {https://github.com/ShokouP/dawcoder},
  note   = {4B parameters, Qwen3.5-4B base, LoRA r=64, GGUF Q4_K_M, Ollama-ready}
}
```
