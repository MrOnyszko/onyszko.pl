# 03 - Choosing Your First Local Model<no value>

I will show you how to pick your first local LLM based on your hardware, without wasting hours downloading models that
won't fit in memory.

## Why

Model selection is the first real decision you make in local AI. Pick wrong and you'll spend hours downloading, testing,
and deleting models that either crawl at glacial speed or crash halfway through a prompt. Pick right and your first
experience feels magical instead of frustrating.

The second piece of the puzzle is quantization. Most models you download won't run at their base precision — the file
sizes would be enormous. Instead, they're compressed into smaller formats that trade a small amount of accuracy for
dramatically lower memory requirements. Understanding this trade-off is the difference between a model that works and
one that actually feels fast.

The third piece — the one most beginners miss entirely — is architecture. In 2026, not all models are built the same
way. Some are dense (every parameter fires on every token). Others use a Mixture-of-Experts (MoE) design, where only a
fraction of parameters activate per token. This matters more than raw parameter counts, and we'll cover it in depth.

---

## How

### Understanding Parameters

Parameters are the values a model has learned — the model's "knowledge" stored as numbers. When you see "7B," that means
7 billion parameters. More parameters generally mean more capable, but they also mean larger files and more memory
needed to run.

Here's the practical breakdown:

- **1B–4B parameters**: Lightweight. Runs on 4–8GB of VRAM. Good for simple tasks, fast on CPU. Not much room for
  complex reasoning.
- **7B–8B parameters**: The sweet spot for many users. Runs on 8–16GB of VRAM at quantized levels. Handles coding,
  writing, and conversation well.
- **12B–14B parameters**: More capable, but requires 12–16GB for comfortable use. Noticeably better at following complex
  instructions.
- **27B–32B parameters**: Excellent quality. Needs 16–24GB at Q4. Strong across reasoning, coding, and long-context
  tasks.
- **70B+ parameters**: The domain of serious hardware. Top-tier reasoning, but expect slow generation on anything short
  of a workstation GPU.

> The "B" number tells you the total parameter count — think of it like RAM for the model. More parameters, more
> capability, more memory needed.

### Understanding MoE: Why "35B" Is Not What You Think

Parameter count alone is a misleading number when MoE models are involved.

A dense model activates all its parameters for every token. A Mixture-of-Experts model contains many "expert"
sub-networks but only routes each token through a small subset of them. The result: a 35B MoE model might only activate
3B parameters per token at inference time.

That "A3B" suffix you see on models like Qwen3.5-35B-A3B means "3 billion active parameters." Despite having 35 billion
total parameters (which determines memory requirements), the compute per token resembles a 3B model — which is why it's
so fast.

Why does this matter in practice?

- **Memory cost** scales with total parameters. A 35B-A3B still needs ~18–20GB of VRAM at Q4 to fully load.
- **Speed** scales with active parameters. On my Strix Halo 128GB, Qwen3.5-35B-A3B generates 32–48 t/s while Qwen3.5-27B
  dense only manages 8–12 t/s — that's 3–5x faster despite similar memory requirements.
- **Quality** scales more complex than either. The router that decides which experts to activate is itself learned — and
  well-trained MoE models punch above their weight. Qwen3.5-35B-A3B, released in February 2026, outperforms the previous
  generation's dense 235B flagship on most benchmarks. That's a 7x reduction in active parameters delivering more
  capable outputs.

The practical takeaway: when you see MoE numbers, compare the **active** parameter count for speed, but the **total**
parameter count for memory planning.

> Not all MoE models are equal. The routing algorithm — which expert handles which token — is what separates a great MoE
> model from a mediocre one. Qwen3.5's hybrid architecture (Gated Delta Networks + sparse MoE) is currently one of the
> better implementations you can run locally.

### MoE vs Dense: Context Scaling Matters

This is where my benchmarks reveal something crucial. I tested all models at 128K context with varying fill ratios (5%
to 95%), and the architecture difference is dramatic:

| Model                     | Peak TPS | TPS @ 75% Context | TPS Retention | Comfortable Context |
|---------------------------|----------|-------------------|---------------|---------------------|
| Qwen3.5-35B-A3B (MoE, Q4) | 48.1     | 34.2              | 71%           | ~12k tokens         |
| Qwen3.5-35B-A3B (MoE, Q8) | 42.7     | 31.2              | 73%           | ~12k tokens         |
| Qwen3-coder-next (Q4)     | 42.7     | 30.1              | 70%           | ~10k tokens         |
| Qwen3-coder-next (Q6)     | 28.9     | 21.8              | 75%           | ~10k tokens         |
| Qwen3.5-122B-A10B (MoE)   | 12.6     | 5.4               | 43%           | none (<10k)         |
| Qwen3.5-27B (dense, Q4)   | 11.7     | 8.9               | 76%           | none                |
| Qwen3.5-27B (dense, Q8)   | 7.5      | 6.3               | 84%           | none                |

**The MoE advantage is stark**: Qwen3.5-35B-A3B runs at 48 t/s while Qwen3.5-27B dense only manages 11.7 t/s — 4x slower
despite similar memory requirements (17–22GB).

**Context scaling is the real story**. MoE models maintain 70%+ of their peak speed even at 75% context fill. Dense
models degrade badly:

- Qwen3.5-27B Q4: TTFT reaches **707 seconds** at 75% context fill
- Qwen3.5-27B Q8: TTFT reaches **784 seconds** at 75% context fill

That's over 10 minutes just to get the first token. For agentic workflows where you're feeding in codebase context,
dense models become unusable beyond ~10k tokens. MoE models handle practical workloads comfortably up to 60k+ tokens.

The Qwen3.5 family spans a wide range, from 0.8B to 397B:

- **MoE models**: 35B-A3B (3B active), 122B-A10B (10B active), 397B-A17B (17B active)
- **Dense models**: 0.8B, 2B, 4B, 9B, 27B

The 9B, 4B, and 2B dense models are excellent for lower-end hardware — they use the same architecture as their larger
siblings but run on 8–16GB of VRAM. The 35B-A3B hits the sweet spot for most users: MoE efficiency at 18–20GB Q4.

### Understanding Quantization

Quantization compresses a model's weights from their trained precision (FP16 or BF16) down to fewer bits. The most
common formats use 4, 6, or 8 bits per weight.

- **FP16 / BF16 (base precision)**: ~14GB for a 7B model. Standard inference precision.
- **Q8 (8-bit)**: ~7GB for a 7B model. Nearly indistinguishable from FP16 for most tasks.
- **Q6 (6-bit)**: ~5GB for a 7B model. Small quality drop, big memory savings.
- **Q4 (4-bit)**: ~3.5GB for a 7B model. The workhorse of local AI. Quality loss is minimal for general use.

The GGUF format (used by llama.cpp and tools like LM Studio and Ollama) introduced K-quants — smarter quantization that
protects the most important weights. **Q4_K_M** is the most popular choice: it delivers roughly 43% compression with
only 0.5% quality loss in benchmark testing.

My benchmarks on Strix Halo show Q8 vs Q4 speed difference is minimal for MoE models (32–48 t/s vs 29–43 t/s) but
more pronounced for dense models (8–12 t/s vs 6–8 t/s). For MoE, Q4_K_M is the sweet spot — you get nearly Q8 quality
at Q4 memory. For dense models, Q8 may be worth the extra memory if your hardware fits it.

For MoE models specifically, dynamic quantization (e.g. Unsloth's UD-Q4_K_XL) goes further by selectively upscaling
critical MoE routing layers to 8-bit while keeping the rest at 4-bit. This is worth seeking out for MoE models — it's
meaningfully better than vanilla Q4.

> Q4 doesn't mean "quarter quality." It means 4 bits per weight. For general use, the quality difference from FP16 is
> barely measurable.

### What Fits Your RAM

The golden rule: your model needs to fit entirely in VRAM (for GPU) or RAM (for CPU inference), plus some headroom for
the context window. If the model is 9GB and you have 8GB, your system will start swapping to disk — and inference will
drop from 40 tokens/second to 8.

Here's what actually runs comfortably:

| Your Hardware | What Fits Well                                                                              |
|---------------|---------------------------------------------------------------------------------------------|
| 8GB VRAM      | 1B–4B models at Q4–Q6; Qwen3.5-4B                                                           |
| 12GB VRAM     | 7B–9B models at Q4–Q6; Qwen3.5-9B                                                           |
| 16GB VRAM     | 7B–14B models at Q4–Q6; Qwen3.5-9B at Q8                                                    |
| 24GB VRAM     | 12B–27B models at Q4; 35B-A3B MoE at Q4                                                     |
| 32GB VRAM     | 27B–34B models at Q4; 35B-A3B MoE at Q8                                                     |
| 48GB+ VRAM    | 70B models at Q4–Q6; 122B-A10B MoE at Q4                                                    |
| 128GB RAM     | 70B+ models at Q4–Q8; 80B+ MoE models at Q4; can run 397B-A17B with aggressive quantization |

For MacBooks with unified memory, the GPU/CPU share the same pool, which helps considerably:

- **M1/M2/M3 with 16GB**: 7B at Q4 works, but expect slower throughput
- **M1/M2/M3 with 32GB**: 14B at Q4 runs smoothly; 35B-A3B MoE fits at Q4 (22GB)
- **M3 Max / M4 Pro with 48–64GB**: 35B-A3B at Q8, 27B dense, or 70B at Q4
- **Framework matters on MacBooks**: Use MLX-optimized models (mlx-community) for 2–4x speedup over llama.cpp. My M1 Pro
  32GB gets 19–44 t/s with MLX but only 11–18 t/s with llama.cpp for the same Qwen3.5-35B-A3B model.

#### Strix Halo (Integrated GPU)

The AMD Strix Halo — found in machines like the Minisforum MS-S1 Max — is a unique case. Its integrated Radeon 8060S GPU
has enough VRAM to run models that would otherwise require a discrete GPU. With 128GB RAM, you're looking at:

- **70B+ dense models at Q4** — runs entirely in RAM with no discrete GPU needed
- **80B+ MoE models at Q4** — the Strix Halo handles what normally needs a $2000+ graphics card

This makes Strix Halo machines exceptional for local AI. You're essentially getting workstation-class local AI in a
mini-PC form factor, silent (no fans spinning like a jet engine), and at a fraction of the cost of a multi-GPU setup.

---

## The MoE Models Worth Knowing

The landscape shifted significantly in late 2025 and early 2026 as MoE models moved from research curiosity to the
practical recommendation for most hardware tiers. Here are the ones that matter for local use.

### Qwen3.5-35B-A3B

Released February 24, 2026. The standout model in the Qwen3.5 medium series. It has 35B total parameters but only
activates 3B per token. On my Strix Halo 128GB with Vulkan, it runs at 32–48 t/s at Q4_K_M. On a MacBook M1 Pro 32GB,
MLX achieves 19–44 t/s while llama.cpp gets 11–18 t/s — a 2–4x MLX advantage on Apple Silicon. At Q4, it fits in around
18–20GB of VRAM, meaning a single 24GB card (RTX 3090/4090) can run it comfortably. Benchmarks show it surpasses the
previous
generation's Qwen3-235B-A22B across most tasks. Context window is 256K tokens natively (extendable to 1M via YaRN).
Apache 2.0 license.

For agentic workflows — where a model needs to think through many steps quickly — the speed advantage here matters
enormously. Waiting 15 seconds for a 27B dense model to finish a long output is painful when the MoE equivalent finishes
in 3.

> If you have a 24GB GPU and are picking one model today, this is probably it.

### Qwen3.5-122B-A10B

Also released February 24, 2026. 122B total parameters, 10B active. On my Strix Halo 128GB, it achieves 5–13 t/s at
Q4_K_S. It achieved 85% on AIME 2026 (top-tier math reasoning) and 72.2 on BFCL-V4 tool use benchmarks — outperforming
GPT-5 mini's 55.5 on that last one by a significant margin, which matters if you're building agentic pipelines. Needs
around 64GB of VRAM at Q4 (two 3090s or an A100), or runs on a Mac Studio Ultra with 64GB+ unified memory.

### Qwen3.5-27B (dense)

The dense counterpart to the 35B-A3B. All 27B parameters are active for every token — which means slightly more
consistent quality on nuanced reasoning tasks, but significantly slower throughput. On my Strix Halo 128GB, it runs at
8–12 t/s at Q4_K_M versus 32–48 t/s for the MoE variant. At Q4_K_M it needs ~18GB of VRAM. Worth choosing over the MoE
if you're doing fine-tuning (dense models are more predictable for LoRA/QLoRA) or if your workload is single long
requests rather than fast agentic loops.

### Qwen3.5-397B-A17B

The flagship open-weight release from Alibaba, released February 16, 2026. 397B total, 17B active. Natively multimodal —
trained with early text-vision fusion, not a bolted-on vision adapter. Supports 201 languages. Context window of 1M
tokens. Decoding throughput is 8.6x faster than Qwen3-Max at 32K context. The catch: you need serious hardware to run it
locally. At Q4, you're looking at roughly 200GB+ of VRAM or RAM — a multi-GPU server or a high-end Mac Studio cluster.
Not for casual local use, but worth knowing exists.

### Qwen3-Coder-Next-80B-A3B

Released September 2025, fine-tuned for coding agents. 80B total parameters, 3B active. Uses the same hybrid
architecture (Gated Delta Networks + sparse MoE) as Qwen3.5. On my Strix Halo 128GB, Qwen3-coder-next runs at 27–43 t/s
at Q4_K_M — nearly identical speed to Qwen3.5-35B-A3B but specialized for code. On SWE-Bench Pro, it's roughly on par
with Claude Sonnet 4.5. If your use case is specifically coding, this is worth considering alongside the 35B-A3B.

---

## First Model Recommendations

Based on my benchmarks, here are specific recommendations for each of my setups:

### MacBook M1 Pro 32GB

- **Qwen3.5-35B-A3B (mlx-community, 4-bit)** — 19–44 t/s with MLX. Use mlx-community models for 2–4x speedup over
  llama.cpp.
- **Qwen3.5-35B-A3B (llama.cpp, Q4_K_L)** — 11–18 t/s. Still usable, but MLX is significantly faster on Apple Silicon.

The M1 Pro 32GB can fit the 35B-A3B at Q4 (~20GB). For this hardware, MLX is essential — without it, generation feels
sluggish.

### Minisforum MS-S1 Max (Strix Halo 128GB)

- **Qwen3.5-35B-A3B (Q4_K_M)** — 32–48 t/s. The sweet spot. Fast, efficient, handles long context well. **Best overall
  pick.**
- **Qwen3-coder-next (Q4_K_M)** — 27–43 t/s. Coding-specialized, nearly as fast as the general model. Choose this if
  coding is your primary use case.
- **Qwen3.5-35B-A3B (Q8_0)** — 29–43 t/s. Slightly slower but better quality. Worth it if you need better reasoning.
- **Qwen3.5-122B-A10B (Q4_K_S)** — 5–13 t/s. For genuinely hard reasoning tasks only. TTFT gets very high at long
  context.

**Avoid on Strix Halo:**

- **Qwen3.5-27B dense** — Only 8–12 t/s at Q4, worse at Q8. The dense architecture doesn't work well on this hardware.
  My benchmarks show TTFT hits 700+ seconds at 75% context — effectively unusable for agentic workflows.

### General Guidelines

**For entry-level (8–12GB VRAM):**

- **Qwen3.5-9B** — Strong reasoning, fast, 128K context, Apache 2.0.
- **Qwen3.5-4B** — Even lighter, great for 8GB VRAM.
- **Gemma 3 4B** — Google's small model with good multimodal support.

**For mid-range (16–24GB VRAM):**

- **Qwen3.5-35B-A3B** — The recommendation for anyone with 24GB. MoE efficiency gives you 35B quality at 3B speeds.
- **Gemma 3 27B** — Competes with models twice its size. 128K context.

**For high-end (48GB+ VRAM or 64GB+ unified memory):**

- **Qwen3.5-122B-A10B** — Top-tier reasoning and tool-use performance.
- **DeepSeek V3.2** — Strong reasoning, open weights.

> Model rankings change monthly. Check [LMArena](https://lmarena.ai) for the latest
> benchmarks before committing to a 10–80GB download.

---

## Dense vs. MoE: Which One to Pick

This question comes up constantly in the r/LocalLLaMA community, and the honest answer is: it depends on your workflow.

Choose **MoE (e.g. Qwen3.5-35B-A3B)** when:

- You're running agentic workflows where generation speed matters — my benchmarks show 3–5x speedup over dense models
- You're doing interactive chat and don't want to wait
- You want to run a large-context task (256K+ tokens) — the hybrid architecture handles long contexts more efficiently
- You have limited VRAM and want the most quality per GB

Choose **dense (e.g. Qwen3.5-27B)** when:

- You're fine-tuning — dense models behave more predictably with LoRA/QLoRA
- Your workload is single long requests where you aren't waiting interactively
- You need absolutely consistent output quality on very nuanced reasoning tasks
- You've experienced routing artifacts in MoE outputs (a real but rare issue)

For most first-time local AI users, start with the MoE. On my Strix Halo 128GB, Qwen3.5-35B-A3B runs at 32–48 t/s while
Qwen3.5-27B only manages 8–12 t/s — that's the difference between a responsive chat and watching paint dry.

---

## Key Points

1. **Match model to your VRAM first.** A model that doesn't fit will run painfully slow, regardless of how capable it
   is.
2. **Understand the MoE notation.** "35B-A3B" means 35B total (for memory planning) but only 3B active (for speed).
   These are very different numbers.
3. **Use Q4_K_M as your default.** It saves ~75% memory with less than 1% quality loss. For MoE models, prefer dynamic
   quants (Unsloth UD-Q4_K_XL) if available.
4. **Qwen3.5-35B-A3B is the 2026 recommendation for 24GB setups.** It fits on a single RTX 3090/4090, runs at 32–48 t/s
   on Strix Halo, and outperforms last year's 235B dense models.
5. **Check before downloading.** Use the model's Hugging Face page to verify the Q4 file size fits your VRAM before
   starting a 20GB download.
6. **On MacBooks, use MLX-optimized models.** My benchmarks show 2–4x speedup over llama.cpp for the same model.
7. **Q8 vs Q4 matters more for dense models.** MoE shows minimal speed difference; dense models drop 30–40% going from
   Q4 to Q8.
8. **Context scaling is critical for agentic workflows.** Dense models like Qwen3.5-27B hit 700+ second TTFT at 75%
   context — effectively unusable. MoE models maintain usable speeds to 60k+ tokens.
