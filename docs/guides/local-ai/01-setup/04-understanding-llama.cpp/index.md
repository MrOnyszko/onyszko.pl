# 04 - Understanding llama.cpp<no value>

I will show you what happens under the hood in LM Studio, and when diving into llama.cpp directly saves you time.

## Why

Every local AI tool you use — LM Studio, Ollama, Jan.ai — runs on top of an inference engine. For most of them, that
engine is [llama.cpp](https://github.com/ggerganov/llama.cpp). It's the open-source library that actually loads your
model, runs the forward pass, and generates tokens.

Understanding llama.cpp makes you a better user of LM Studio because:

- **Debugging** — When something breaks, the error messages often come straight from llama.cpp. Knowing the terminology
  helps.
- **Optimization** — Settings like GPU layers, context length, and quantization all map to llama.cpp concepts.
  Understanding them helps you tune LM Studio.
- **Benchmarking** — You can run `llama-bench` to compare hardware performance objectively, outside any GUI.
- **Advanced use cases** — LM Link and server features build on llama.cpp. Knowing the foundation makes them more
  powerful.

This post isn't about replacing LM Studio. It's about understanding the engine so you can drive better.

> Most of what I'll show in future posts happens inside LM Studio. But when we tune GPU
> offloading, this is the layer you're actually controlling.

---

## How

### llama.cpp Under the Hood

LM Studio runs llama.cpp as its inference engine. When you adjust settings in the GUI, you're actually configuring
llama.cpp parameters. Here's the mapping:

| LM Studio Setting | llama.cpp Equivalent | What It Does                                                      |
|-------------------|----------------------|-------------------------------------------------------------------|
| GPU Layers        | `-ngl`               | How many model layers load to GPU. More = faster, uses more VRAM. |
| Context Length    | `-c`                 | Maximum prompt + generated tokens. Higher = more memory, slower.  |
| Threads           | `-t`                 | CPU threads for offloaded layers.                                 |
| Quantization      | GGUF type            | How the model is compressed (Q4, Q6, Q8).                         |

When things feel slow, checking the GPU layers in LM Studio is the first step. If it's set to auto, try explicitly
setting it to the maximum your VRAM allows.

---

### Benchmarking with llama-bench

To compare hardware performance objectively, use `llama-bench` — the command-line benchmarking tool from llama.cpp. This
runs outside LM Studio and gives you pure inference numbers.

```bash
./llama-bench \
  -m /path/to/model.gguf \
  -ngl 35 \
  -n 512 \
  -t 8
```

Key flags:

- `-m` — Path to your GGUF model file
- `-ngl` — Number of GPU layers to offload
- `-n` — Tokens to generate (512 is standard)
- `-t` — Thread count for CPU

Output shows tokens per second (t/s). For Qwen3.5-35B-A3B at Q4 on Strix Halo with Vulkan, expect 32–48 t/s. On MacBook
M1 Pro with MLX, 19–44 t/s. With llama.cpp on the same MacBook (Metal backend), expect 11–18 t/s — roughly 2x slower
than MLX.

> Lower context length gives higher t/s. If you need 128K context, expect significantly lower throughput.

Save your benchmark commands as scripts. Re-run them after driver updates or hardware changes to measure the impact.

---

### When to Use CLI Directly

Most of the time, LM Studio handles everything. But there are cases where dropping to CLI is useful:

**1. Debugging issues**

If LM Studio crashes or behaves oddly, running the same model with llama.cpp CLI often reveals the real error:

```bash
./llama-cli -m ./models/Qwen3.5-35B-A3B-GGUF/Q4_K_M.gguf -ngl 35 -c 4096
```

**2. Server mode**

If you need an API without LM Studio running, llama.cpp's server mode works directly:

```bash
./llama-server \
  -m ./models/Qwen3.5-35B-A3B-GGUF/Q4_K_M.gguf \
  -ngl 35 \
  -c 4096 \
  --port 8080
```

This is what LM Studio's built-in server does under the hood.

**3. Custom automation**

Scripts that don't need the full LM Studio overhead:

```bash
./llama-cli \
  -m ./models/Qwen3.5-35B-A3B-GGUF/Q4_K_M.gguf \
  -n 512 \
  -c 4096 \
  -ngl 35 \
  -p "Write a Python function that calculates fibonacci numbers."
```

Common flags:

- `-m` — Model path
- `-n` — Max tokens to generate
- `-c` — Context window size
- `-ngl` — GPU layers (0 = CPU only)
- `-p` — Prompt

---

### GPU Memory Tuning

If you run out of VRAM, adjust layer offloading. In llama.cpp CLI:

```bash
# Reduce GPU layers to fit in memory
./llama-cli -m ./models/Q4_K_M.gguf -ngl 20 -c 2048

# CPU only (slow but uses no VRAM)
./llama-cli -m ./models/Q4_K_M.gguf -ngl 0 -t 16
```

The `-ngl` value is the main dial for VRAM vs. speed trade-offs. In LM Studio, this is the "GPU Layers" slider.

---

### ROCm Setup for AMD GPUs

On the Minisforum MS-S1 Max with Strix Halo, LM Studio uses AMD's ROCm (Radeon Open Compute) or Vulkan for GPU
acceleration, depending on your configuration.

**Current state of ROCm:** ROCm 6.x has stability issues on Strix Halo. ROCm 7.2 works but requires custom builds with
HIP patches. The simpler path is using Vulkan for GPU acceleration, which works out of the box on recent LM Studio
builds.

LM Studio auto-detects your GPU and chooses the best backend. On Strix Halo, it defaults to Vulkan. You don't need to
configure anything.

If you run into issues or want to experiment manually, the community toolboxes
from [kyuz0/amd-strix-halo-toolboxes](https://github.com/kyuz0/amd-strix-halo-toolboxes) are the most reliable resource.
kyuz0 maintains ROCm 7.2 configurations specifically for Strix Halo, along with pre-built llama.cpp binaries tuned for
this hardware. His YouTube channel has detailed walkthroughs.

> Most users should let LM Studio handle this. Only touch ROCm manually if you're troubleshooting or need specific
> performance tuning.

---

## Key Points

1. **llama.cpp is the engine** — LM Studio and other tools run on top of it. Understanding it helps you use those tools
   better.
2. **Benchmark with llama-bench** — Run it to compare hardware performance outside any GUI.
3. **GPU layers are the main dial** — In LM Studio or CLI, this controls speed vs. VRAM trade-offs.
4. **CLI is optional** — Use it for debugging, custom automation, or when LM Studio overhead matters.