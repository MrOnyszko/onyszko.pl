# 02 - Getting Started with Local AI<no value>

In this post, I will show you how to set up a local AI environment and run your first model in under 30 minutes.

## Why

Cloud AI services send your data to remote servers. That means sharing conversations, files, and prompts with third
parties. Running AI locally keeps everything on your machine. You get privacy, no per-query costs, and full control over
which models run and when.

If you've ever waited for an API rate limit to reset or wondered whether your data is being used for training, local AI
solves both problems.

---

## How

### Hardware Overview

You need a computer with sufficient RAM to run local AI models. Three main options work well:

1. **Apple Silicon** (M1/M2/M3/M4/M5) — MacBooks, Mac Mini, and Mac Studio. Unified memory means the GPU accesses RAM
   directly, giving excellent performance for the price, especially with MLX-optimized frameworks.

2. **AMD Mini PCs / laptops with Strix Halo** (Ryzen AI Max+ series) — Small desktops or powerful laptops with
   integrated GPUs and high unified memory (up to 128GB). These offer strong MoE support via ROCm/Vulkan and are
   competitive in price/performance for high-RAM setups.

3. **PCs with Nvidia GPUs** — The RTX 4090 (or newer 50-series) offers the fastest token generation on dense layers, but
   VRAM is the limiting factor — a 24GB card fits mid-size models fully, while larger ones require offloading.

> Better hardware means faster speeds, larger models, and longer context windows. Here are my measured results for Qwen3
> models (4-bit quantization, 128K context, typical prompts):

- **Qwen3.5-35B-A3B** (efficient MoE, ~35B total / ~3B active params):
  - MacBook M1 Pro 32GB (MLX):
    - ~19–44 tokens/sec (4-bit MLX),
    - ~11–18 tokens/sec (llama.cpp Q4_K_L).
  - Minisforum MS-S1 Max 128GB (Strix Halo, Vulkan):
    - ~32–48 tokens/sec (Q4_K_M),
    - ~29–43 tokens/sec (Q8_0).
  - Qwen3-Coder-Next (specialized for code):
    - ~27–43 tokens/sec on Strix Halo.

- **Qwen3.5-27B** (dense variant):
  - Strix Halo 128GB: ~8–12 tokens/sec (Q4_K_M), ~6–8 tokens/sec (Q8_0).

- **Qwen3.5-122B-A10B** (large MoE, ~122B total / ~10B active):
  - Strix Halo 128GB: ~5–13 tokens/sec (Q4_K_S).

The M1 Pro results use MLX (Apple's optimized framework) — llama.cpp via LM Studio runs slower (~11–18 t/s) on the same
hardware. On the Strix Halo, llama.cpp with Vulkan delivers the best results shown above. A higher context fill ratio (
more input tokens) reduces generation speed on all platforms.

### Installing LM Studio

[LM Studio](https://lmstudio.ai/) is the easiest way to get running. It bundles model downloading, inference, and a chat
interface into one app.

1. Download LM Studio from the official site.
2. Install the application.
3. Launch LM Studio.

On the left side there's search models icon. This is where you browse and download models.

#### First Model Download

For your first run, download a small, fast model:

1. In the search models modal, search for `Qwen/Qwen3.5-35B-A3B`.
2. Click **Download**. Wait for completion.
3. Switch to the **Chat**.
4. Select your downloaded model from the dropdown to load it. Wait for completion.
5. Type a prompt and press Enter.

That's it. You're running a local LLM.

> LM Studio works the same on Minisforum and MacBook. The only difference is the backend — Linux uses CUDA/ROCm, macOS
> uses Metal.

#### Installing LM Studio on Fedora (AppImage)

On my Minisforum, I run Fedora. LM Studio provides an AppImage that works without installation. You can use following
script to download the latest version and create desktop entry:

> It is handy when you need to update LM Studio to a version.

```bash
#!/bin/bash
set -e

APP_DIR="$HOME/Applications"
mkdir -p "$APP_DIR"
cd "$APP_DIR"

echo "Downloading LM Studio..."
curl -L -o lm-studio.AppImage "https://lmstudio.ai/download/latest/linux/x64?format=AppImage"

chmod +x lm-studio.AppImage

echo "Downloading icon..."
mkdir -p icons
curl -L -o icons/lm-studio.png "https://lmstudio.ai/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Flogo-192x192.944df324.png&w=256&q=75"

echo "Creating desktop entry..."
cat > "$APP_DIR/lm-studio.desktop" << EOF
[Desktop Entry]
Name=LM Studio
Exec=$APP_DIR/lm-studio.AppImage
Icon=$APP_DIR/icons/lm-studio.png
Type=Application
Categories=Development;
Terminal=false
StartupNotify=true
EOF

cp "$APP_DIR/lm-studio.desktop" "$HOME/.local/share/applications/"
update-desktop-database "$HOME/.local/share/applications/" 2>/dev/null || true

echo "Done! LM Studio is ready at $APP_DIR/lm-studio.AppImage"
```

Run the script:

```bash
chmod +x install-lmstudio.sh
./install-lmstudio.sh
```

Launch LM Studio from your application menu or by running `$HOME/Applications/lm-studio.AppImage` directly.

#### Using llmster (Headless Mode)

[llmster](https://lmstudio.ai/docs/developer/core/headless_llmster) is LM Studio's daemon — the core engine without the
GUI. It runs in the background and serves models via an OpenAI-compatible API.

Install llmster:

```bash
curl -fsSL https://lmstudio.ai/install.sh | bash
```

This installs the `lms` CLI tool. Verify:

```bash
lms --version
```

Basic workflow:

```bash
lms daemon up                       # Start the daemon
lms get Qwen/Qwen3.5-35B-A3B        # Download a model
lms load Qwen/Qwen3.5-35B-A3B       # Load the model
lms server start                    # Start the API server
lms chat                            # Open interactive chat
```

The server runs at `http://localhost:1234`. You can now call it from any app using the OpenAI API format.

> llmster is useful when you don't need the GUI — on headless servers, for systemd services, or when you want to
> integrate local AI into your own applications.

---

### Initial Setup

LM Studio defaults work out of the box. Two settings worth adjusting:

1. **Context Length** — In model settings, set to `4096` or lower for faster responses on smaller hardware.
2. **GPU Offload** — Enable if your system has a discrete GPU. On MacBook M1 Pro, this uses Metal acceleration
   automatically.

Test with a simple prompt:

```
Explain what LLM quantization means like I am 5.
```

You should see a response within seconds. If it hangs, lower the context length or switch to a smaller model.

---

## Key Points Recap

1. Local AI keeps your data private and eliminates per-query costs.
2. MoE models change everything — a 35B MoE runs at 8B speeds. Check active parameters, not total.
3. RAM determines what model sizes you can run — 32GB handles 14B dense or 70B+ MoE.
4. LM Studio provides a complete workflow: download models, chat, configure — in one app.
5. Start with Qwen3.5-35B-A3B at Q4_K_M — it runs on 24GB+ and delivers 70B-class quality.
6. On M1 Pro 32GB, MLX gives 2-4x speedup over llama.cpp. On Strix Halo, use llama.cpp with Vulkan.
