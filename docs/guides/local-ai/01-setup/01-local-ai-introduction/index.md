# 01 - Local AI Introduction<no value>

This guide walks you through setting up a local AI development environment and building AI-assisted workflows using
open-source tools.

## What You'll Learn

1. **Setup** — Hardware requirements, LM Studio installation, first model
2. **OpenCode Environment** — Installing, configuring, and using OpenCode with local models
3. **Agents** — Creating specialized AI agents for different roles
4. **Skills** — Building reusable skills for consistent outputs
5. **Full Workflow** — Complete feature development with AI assistance
6. **Assessment** — What works and what doesn't

## Prerequisites

- A computer with 32GB+ Unified Memory or 24GB+ VRAM
- Basic familiarity with command line
- Interest in AI-assisted development

---

## Basics of LLMs

Before diving into local AI, understand the fundamentals that drive everything you do in this guide.

### What Are Tokens

Think of tokens as the atomic units of text — not quite characters, not quite words. A token is typically 3–4 characters
on average, though it varies. "Hello world" becomes roughly two tokens. A paragraph might become 80 tokens. A code file
with 500 lines could become 10,000+ tokens.

The reason this matters: every operation in local AI — loading a model, generating a response, processing your
codebase — happens token by token. When someone mentions "128K context window," they mean the model can handle roughly
128,000 tokens of prompt plus its own output in a single request.

> Context window is like short-term memory. A model with a 128K context can "remember" roughly 100,000 words worth of
> context in a single conversation. Beyond that, older content gets forgotten or must be managed through techniques like
> sliding windows.

For coding agents, this directly impacts how much of your codebase you can feed the model at once. A 4K context model
might see one file. A 128K model can ingest your entire project.

### Next Token Generation

Here's the core mechanism that makes LLMs work: at every step, the model predicts the single most likely token to come
next.

Picture autocomplete on your phone. It suggests the next word based on what you typed. An LLM does the same thing —
except it's predicting one token at a time, and each new token changes what might come next. This creates a chain of
probability:

1. You send a prompt: "Write a function that adds two numbers"
2. The model sees the prompt as a sequence of tokens
3. It calculates probabilities for every possible next token — "def", "function", "var", etc.
4. It picks one (or samples from the top-k), adds it to the sequence
5. Now the sequence includes the new token — repeat from step 2

This is why LLMs generate text sequentially. There's no magic button to produce a full response instantly. The
generation speed you see in benchmarks — "48 tokens per second" — is literally how fast the model predicts and outputs
one token after another.

For agentic workflows, this has practical implications:

- **Generation time scales with output length.** A 500-token response takes roughly 10 seconds at 50 t/s.
- **Faster generation = more interactive workflows.** When OpenCode generates code, you're waiting for each token. A
  faster model feels responsive; a slow one feels broken.
- **Prompt complexity matters.** Processing your prompt (the "first token" delay) also takes time, especially with large
  context windows.

### Popular LLM Architectures

In 2026, three architecture families dominate local AI:

**Dense (Transformer)** — Every token flows through every parameter. Simple, consistent, predictable. Think of it as a
single large engine that always runs at full power. Qwen3.5-27B, Llama 3.1 8B, and most smaller models use this
architecture.

**MoE (Mixture-of-Experts)** — The model contains many specialized "expert" sub-networks, but only a small subset
activates for any given token. It's like having a team where each person handles their specialty — the router sends the
token to the right expert. Qwen3.5-35B-A3B (3 billion active parameters out of 35 billion total) exemplifies this. The
advantage: more capability at lower compute cost.

> MoE is the architecture that made 35B-class models viable on consumer hardware. Qwen3.5-35B-A3B runs at 40+ t/s on a
> Strix Halo because only 3B parameters activate per token — not 35B.

**Hybrid (Gated Delta + MoE)** — The latest innovation, used by Qwen3. Qwen3.5 combines Gated Delta Networks (which
learn what to add or change from a base model) with sparse MoE. This delivers the best of both: massive parameter counts
for quality, small active counts for speed.

For local inference and agentic coding work, MoE models like Qwen3.5-35B-A3B are currently the sweet spot. They provide
large-model quality at small-model speeds — critical when you're waiting for code generation in real-time.

### What Is GGUF

GGUF (GPT-Generated Unified Format) is the file format used by llama.cpp and every tool built on top of it — LM Studio,
Ollama, Jan.ai. When you download a model, it's usually in GGUF format.

Before GGUF, models came in multiple formats that required different inference engines. GGUF standardized everything
into a single format that:

- Stores the model weights efficiently
- Includes metadata (tokenizer, architecture settings, context length)
- Works across platforms (Windows, macOS, Linux)
- Supports quantization built-in

When choosing models, you'll see files like `model-q4_k_m.gguf` or `model-q8_0.gguf`. The `q4_k_m` means 4-bit
quantization with the K-quants method (specifically the Medium variant). This is the format that makes 35B models run on
24GB of VRAM instead of needing 70GB.

> GGUF is to local AI what MP3 is to music. It's a compressed format that makes files small enough to download and run,
> while preserving what matters most.

Understanding GGUF helps you make better model choices. A Q4_K_M file is roughly 75% smaller than the original FP16
weights — the compression ratio that enables local AI on consumer hardware.

---

## Key Points

1. **Tokens are the atomic unit of LLM text.** A token is typically 3–4 characters. Context windows are measured in
   tokens.
2. **LLMs generate one token at a time.** Speed (tokens/second) directly determines how responsive your AI assistant
   feels.
3. **MoE architectures are the 2026 standard for local AI.** They deliver large-model quality at small-model speeds.
4. **GGUF is the format that makes local inference possible.** It compresses models to run on consumer hardware.
5. **For coding agents, generation speed matters more than raw model size.** A 35B MoE that generates 40 t/s outperforms
   a 70B dense model that generates 8 t/s for interactive workflows.
