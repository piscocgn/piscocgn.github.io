---
{"dg-publish":true,"dg-permalink":"en/ai/model-names","permalink":"/en/ai/model-names/","title":"Reading AI model names: Hugging Face and Ollama","metatags":{"description":"What names like Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL mean: family, size, mixture of experts, variant, format and quantization. With rules of thumb for memory and speed and a checklist."},"dg-note-properties":{"title":"Reading AI model names: Hugging Face and Ollama"}}
---

#ai #llm #ollama #huggingface #local

A name like `Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL` tells you almost everything you need to know before downloading: maker, size, architecture, purpose, file format and compression. This page shows how to read the parts and what they mean for memory, speed and use.

As of November 2026. Rules of thumb and bit values are rounded estimates. The measurements come from a MacBook Air M3 with 24 GB and Ollama 0.40.

**Deutsche Version:** [[KI/Modellnamen lesen\|Modellnamen lesen]]

## The building blocks

The order varies by maker, but the building blocks are almost always the same.

| Block | Example | Meaning |
|---|---|---|
| Uploader | `unsloth/`, `Qwen/` | Before the slash: the maker or a repackager |
| Family | `Qwen`, `gemma`, `Llama`, `gpt-oss` | The maker's model line |
| Generation | `3.6`, `4`, `3.3` | Higher is newer |
| Size | `27B`, `360M` | Parameters in billions (B) or millions (M) |
| Active parameters | `A3B` | Mixture of experts: only part of the model works per word |
| Variant | `Instruct`, `it`, `Thinking`, `Coder`, `VL` | What the model was trained for |
| Date | `2507`, `0528` | July 2025 (YYMM) or 28 May (MMDD) |
| Format | `GGUF`, `MLX`, `AWQ`, `FP8` | Decides which software can read it |
| Quantization | `Q4_K_M`, `Q8_0`, `4bit` | How strongly the weights are compressed |
| Context | `64k`, `128k` | In Ollama names usually self-made |
| Ollama tag | `:latest`, `:27b` | After the colon: which variant |
| Cloud | `-cloud` | Does not run locally, off limits for private data |

## Three names taken apart

**`unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL`**

* `unsloth/`: a well-known repackager, the original comes from Alibaba
* `Qwen` `3.6`: Qwen family, generation 3.6
* `35B-A3B`: mixture of experts with 35 billion parameters, 3 billion active
* `GGUF`: format for Ollama, llama.cpp and LM Studio
* `UD-Q4_K_XL`: about 5 bits per weight, important layers kept more precise ("Unsloth Dynamic")
* Result: a file of about 24 GB, too big for a Mac with 24 GB

**`Qwen/Qwen3-235B-A22B-Thinking-2507`**

* `Qwen/`: uploaded by the maker itself
* `235B-A22B`: a very large MoE model, 22 billion active
* `Thinking`: reasons at length before answering
* `2507`: version from July 2025
* No format given: original weights for server GPUs

**`deepseek-ai/DeepSeek-R1-Distill-Qwen-7B`**

* `DeepSeek-R1`: DeepSeek's reasoning series
* `Distill-Qwen-7B`: actually a Qwen 7B that learned from R1. The name after `Distill` is the base model.

## Size and architecture

The number before the B is the most important value in the name. It sets memory needs, speed and, roughly, capability.

| Notation | Meaning | In practice |
|---|---|---|
| `27B` | Dense model with 27 billion parameters | Every generated word reads all weights. Memory and speed follow the number directly. |
| `35B-A3B` | Mixture of experts: 35 billion total, 3 billion active | Needs memory for 35B but runs almost as fast as a 3B model. Often the best choice on Macs if it fits. |
| `8x7B`, `17B-16E` | Older MoE notation: 8 experts of 7B, or 16 experts | Total size is smaller than the product; Mixtral 8x7B has about 47B. |
| `e4b` | "Effective" 4B, a Gemma quirk | Runs like 4B, but the file is bigger because of extra parts. |
| `360M` | Millions instead of billions | Tiny models for phones, autocomplete or classification. |

## Variants

| Suffix | Meaning | Use |
|---|---|---|
| `Instruct`, `it`, `Chat` | Trained for instructions and conversation | What you want almost every time. `it` is Google's suffix. |
| `Base`, `pt` | Raw base model without chat training | Only as a starting point for your own fine-tuning. |
| `Thinking`, `Reasoning`, `R1` | Reasons at length before answering | More accurate at logic and arithmetic, but slower. |
| `Coder`, `Code` | Specialized in programming | Writing and reviewing code. |
| `VL`, `Vision`, `Omni` | Understands images, Omni also audio | Receipts, screenshots, photos. |
| `Embedding` | Turns text into number vectors | Search over your own documents (RAG). Not a chat model. |
| `Guard`, `safeguard` | Checks content against policies | Filter before or after a chat model. |
| `Distill` | Small model that learned from a big one | The name after it is the base model. |
| `Flash`, `Mini`, `Small`, `Pro`, `Max` | Tier within a series | Only comparable within the family. |
| `abliterated`, `uncensored` | Community version with safety filters removed | Origin often unclear, avoid for serious work. |

## Formats

| Format | Runs with | Note |
|---|---|---|
| `safetensors` (no suffix) | Transformers, vLLM, server GPUs | Original weights at full precision. Usually too big for a laptop. |
| `GGUF` | Ollama, llama.cpp, LM Studio | The standard for local use. One repo often holds many quantization levels. |
| `MLX`, `-mlx` | Apple Silicon (LM Studio, mlx-lm, newer Ollama versions) | Built for Macs. Whether it beats GGUF depends on the model. |
| `AWQ`, `GPTQ`, `EXL2/3` | Nvidia GPUs | Not usable on a Mac. |
| `FP8`, `NVFP4`, `MXFP4` | Newer Nvidia data center cards | NVFP4 needs Blackwell GPUs. MXFP4 is gpt-oss's native format. |

## Quantization

The weights are rounded from 16 bits to fewer bits. That saves memory and makes the model faster, but costs some quality. Bit values are averages including overhead.

| Level | Bits per weight | GB per billion parameters | Quality |
|---|---|---|---|
| `BF16`, `F16` | 16 | ≈ 2.0 | Original, the reference |
| `Q8_0`, `8bit` | ≈ 8.5 | ≈ 1.1 | Practically lossless |
| `Q6_K` | ≈ 6.6 | ≈ 0.8 | Very good |
| `Q5_K_M` | ≈ 5.7 | ≈ 0.7 | Good, hardly different from the original |
| `Q4_K_M`, `4bit` | ≈ 4.5–4.9 | ≈ 0.6 | **The usual compromise**, Ollama's default |
| `IQ4_XS` | ≈ 4.3 | ≈ 0.55 | Almost like Q4_K_M, a bit smaller |
| `Q3_K_M`, `IQ3_XXS` | ≈ 3.1–3.9 | ≈ 0.45 | Noticeable loss, a stopgap |
| `Q2_K`, `IQ2_XXS` | ≈ 2.1–3.0 | ≈ 0.35 | Badly damaged, only for trying things out |

* `K` stands for the k-quant method, `S`/`M`/`L`/`XL` for the share of higher-precision layers.
* A leading `I` (`IQ3`) or `i1`, `imatrix`: quantized with an importance matrix, better at low levels.
* `UD`: "Unsloth Dynamic", important layers stay more precise. Slightly bigger than the plain level of the same name.
* `QAT`: the maker prepared the model for compression. Loses the least at 4 bits.

## Where it comes from

**Ollama:** `name:tag`. Without a slash the model comes from Ollama's curated library; `account/name` comes from a community account.

* `:latest` is the default variant, not necessarily the newest or largest.
* Tags pick size and compression: `:27b`, `:27b-q8_0`, `:26b-mlx`.
* `-cloud` runs on Ollama's servers. Exclude it for private data.
* `hf.co/account/repo:Q4_K_M` loads a GGUF file straight from Hugging Face.
* A context in the name such as `-64k` is usually self-made. Otherwise Ollama starts with a 4k context. Custom variants come from a Modelfile with `PARAMETER num_ctx`.

**Hugging Face:** `account/repo`. The part before the slash tells you who uploaded the file.

* Makers: `Qwen`, `google`, `meta-llama`, `openai`, `mistralai`, `deepseek-ai`, `microsoft`, `nvidia`.
* Well-known repackagers: `unsloth`, `bartowski`, `lmstudio-community`, `mlx-community`. They provide GGUF and MLX versions of the makers' models.
* Repo suffixes such as `-GGUF`, `-MLX`, `-AWQ`, `-FP8` name the format. The level is then in the file name.
* License, base model and recommended settings are on the model card, not in the name.

## Rules of thumb for memory and speed

**Does it fit in memory?**

```
GB ≈ parameters (billions) × bits ÷ 8 × 1.1
```

Add 1 to 3 GB for the context and, for image models, an image encoder. By default a Mac gives the GPU only part of its memory. As a guide, about 70 %, so roughly 16 to 17 GB for model and context on a 24 GB machine.

**How fast does it write?**

```
tokens/s ≈ memory bandwidth ÷ (active parameters × bits ÷ 8) × 0.75
```

Every word reads all active weights once, so bandwidth matters, not compute. For MoE models the factor is closer to 0.4. A MacBook Air M3 has 100 GB/s, an M4 Pro 273 GB/s, an M4 Max up to 546 GB/s. From about 8 tokens per second an answer reads smoothly.

**Cross-check** on a MacBook Air M3 with 24 GB, Ollama, short prompt:

| Model | In memory | Measured | Rule of thumb |
|---|---|---|---|
| `qwen3:4b` | 2.9 GB | 23 tokens/s | ≈ 31 |
| `qwen3.5:4b` | 3.1 GB | 21 tokens/s | ≈ 31 |
| `qwen3.5` 9B (Q4_K_M) | 6.9 GB | 14–16 tokens/s | ≈ 14 |
| `gemma4:12b` | 7.7 GB | 11 tokens/s | ≈ 10 |
| `gpt-oss:20b` (MoE, 3.6B active) | 12 GB | 19 tokens/s | ≈ 21 |

Small models stay below the estimate because fixed per-word overhead weighs more. Reading long prompts is a separate bottleneck: about 13,000 tokens took two and a half to three minutes on the same machine.

## Checklist before use

1. **Origin:** Does the file come from the maker or a well-known repackager? Avoid unknown accounts and "uncensored" variants.
2. **License:** check the model card. Apache 2.0 and MIT are free to use. Llama, Gemma and some Mistral models have their own terms. "Research only" or "non-commercial" rules out commercial work.
3. **Privacy:** Does it really run locally? No `-cloud` tag, no API keys for outside providers, web search switched off in the tool.
4. **Hardware:** file size plus context below about 70 % of memory (Mac) or GPU memory (PC).
5. **Right variant:** Instruct for chat, Coder for code, Embedding for search. For agents the model must be able to call tools; Ollama shows this as the "tools" capability.
6. **Recency:** compare generation and date. A new 9B often beats an old 30B.
7. **Test it yourself:** maker benchmarks are marketing. Three to five real tasks from your own work are enough to separate the candidates.
