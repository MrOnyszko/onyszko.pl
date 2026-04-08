# 05 - Installing OpenCode and Connecting it to LM Studio<no value>

In this post, I will show you how to install OpenCode and connect it to your local LM Studio instance for private,
offline AI-assisted development.

## Why

OpenCode is an AI coding agent that runs in your terminal. Unlike cloud-based agents, it can work with models from any
provider—including local LLMs served by LM Studio. This keeps your code private and eliminates API costs.

Connecting OpenCode to LM Studio means you can use your locally hosted models (like Qwen3.5-35B-A3B) without sending
data to external servers. You get the power of an agentic workflow with full control over which model runs and where.

---

## How

### Installing OpenCode

OpenCode supports multiple installation methods. On macOS, I use Homebrew for easy updates:

```bash
brew install anomalyco/tap/opencode
```

Alternative methods:

- **npm**: `npm install -g opencode-ai`
- **Bun**: `bun install -g opencode-ai`
- **Install script**: `curl -fsSL https://opencode.ai/install | bash`

Verify installation:

```bash
opencode --version
```

You should see something like `1.2.15`.

### Connecting to LM Studio

OpenCode connects to models through a JSON configuration file. This approach is more persistent than the interactive
wizard and lets you define multiple models upfront.

**Step 1: Create the config file**

Create `~/.config/opencode/opencode.json` (or `.opencode/opencode.json` in your project root for per-project
settings):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "lm-studio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio",
      "options": {
        "baseURL": "http://127.0.0.1:1234/v1",
        "apiKey": "lm-studio-very-secure-key"
      },
      "models": {
        "qwen3.5-35b-a3b@q4_k_xl": {
          "name": "Qwen3.5 35B - Q4_K_XL",
          "supportsTools": true
        },
        "qwen3.5-9b@q4_k_xl": {
          "name": "Qwen3.5 9B - Q4_K_XL",
          "supportsTools": true
        }
      }
    }
  },
  "model": "lm-studio/qwen3.5-35b-a3b@q4_k_xl"
}
```

**Step 2: Start the LM Studio server**

Launch LM Studio, load your model (e.g., Qwen3.5-35B-A3B), then click **Server** in the bottom-left corner and hit
**Start Server**. The server runs at `http://localhost:1234` by default.

If running LM Studio on a separate machine (e.g., a headless server), enable **Serve on Local Network** in
**Server Settings**. This binds the server to all network interfaces, allowing connections from other devices on your
local network. Update the `baseURL` in your config to use the server's local IP address (e.g.,
`http://192.168.1.100:1234/v1`).

**Step 3: Run OpenCode**

Start OpenCode with your new configuration:

```bash
opencode
```

The `model` key at the top level specifies which model to use by default. Format is
`provider-name/model-id`.

> The `$schema` field enables auto-completion in editors like VS Code. The `npm` field specifies the AI SDK to use for
> the OpenAI-compatible API. LM Studio's server mode implements this format natively.

### Confirming the Connection

Test your setup with a simple prompt:

```
What is 2+2?
```

You should see the model respond within seconds using your locally hosted Qwen3.5-35B-A3B model.

If it hangs:

1. Verify LM Studio's server is running
2. Check the port matches (default: 1234)
3. Ensure the model is loaded and selected

---

## Key Points Recap

1. **OpenCode runs in your terminal** — It's an agentic coding assistant that works with any LLM provider, including
   local models.
2. **LM Studio serves OpenAI-compatible API** — Start the server in LM Studio to expose your local models at
   `http://localhost:1234`.
3. **Configure OpenCode via JSON file** — Create `~/.config/opencode/opencode.json` with your provider settings and
   models.
4. **Use any local model** — Once connected, your locally hosted Qwen3.5 or other GGUF models work seamlessly in
   OpenCode.
5. **Privacy first** — Your code never leaves your machine when using local models through LM Studio.
