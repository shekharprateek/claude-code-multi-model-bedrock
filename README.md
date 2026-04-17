# Claude Code Multi-Model on Amazon Bedrock

Run [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with **any foundation model on Amazon Bedrock** — not just Anthropic models. Switch between Claude, Qwen, DeepSeek, Llama, Mistral, Kimi, MiniMax, and Nova with a single command.

## Architecture

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                           Claude Code CLI                               │
│                     (speaks Anthropic Messages API)                     │
└──────────┬──────────────────────┬──────────────────────┬────────────────┘
           │                      │                      │
   ┌───────▼────────┐    ┌───────▼──────────────┐   ┌───▼──────────────┐
   │  Native Path   │    │  LiteLLM Proxy Path  │   │  LiteLLM Proxy   │
   │  (no proxy)    │    │  (Bedrock models)    │   │  (self-hosted)   │
   │                │    │                      │   │                  │
   │  Claude Opus   │    │  Anthropic → OpenAI  │   │  Anthropic →     │
   │  Claude Sonnet │    │  API translation     │   │  OpenAI API      │
   │  Claude Haiku  │    │                      │   │                  │
   └───────┬────────┘    └───────┬──────────────┘   └───┬──────────────┘
           │                      │                      │
   ┌───────▼────────┐    ┌───────▼──────────────┐   ┌───▼──────────────┐
   │  Amazon        │    │  Amazon Bedrock       │   │  Ollama on GPU   │
   │  Bedrock       │    │                      │   │  (SSH tunnel)    │
   │  (Anthropic)   │    │  Qwen, DeepSeek,     │   │                  │
   │                │    │  Llama, Mistral,      │   │  Qwen, DeepSeek  │
   │                │    │  Kimi, MiniMax, Nova  │   │  or any model    │
   └────────────────┘    └──────────────────────┘   └──────────────────┘
```

**Why a proxy?** Claude Code speaks the Anthropic Messages API (`/v1/messages`). Bedrock's third-party models speak the OpenAI Chat Completions API (`/v1/chat/completions`). LiteLLM translates between these formats.

## Supported Models

| Alias | Provider | Model | Type | Best For |
|-------|----------|-------|------|----------|
| `claude-opus` | Anthropic | Claude Opus 4.6 | native | Flagship reasoning, complex tasks |
| `claude-sonnet` | Anthropic | Claude Sonnet 4.6 | native | Balanced speed/quality |
| `claude-haiku` | Anthropic | Claude Haiku 4.5 | native | Fast, lightweight tasks |
| `qwen-coder-next` | Qwen | Qwen3 Coder Next | proxy | Code generation, debugging |
| `qwen-coder-480b` | Qwen | Qwen3 Coder 480B | proxy | Large-scale coding tasks |
| `qwen-coder-30b` | Qwen | Qwen3 Coder 30B | proxy | Fast, efficient coding |
| `qwen-235b` | Qwen | Qwen3 235B | proxy | General purpose MoE |
| `qwen-vl-235b` | Qwen | Qwen3 VL 235B | proxy | Vision + language |
| `qwen-next-80b` | Qwen | Qwen3 Next 80B | proxy | Efficient MoE |
| `deepseek-v3` | DeepSeek | DeepSeek V3.2 | proxy | Coding + reasoning |
| `deepseek-r1` | DeepSeek | DeepSeek R1 | proxy | Complex reasoning (chain-of-thought) |
| `llama4-maverick` | Meta | Llama 4 Maverick | proxy | Multimodal chat |
| `llama4-scout` | Meta | Llama 4 Scout | proxy | Efficient MoE inference |
| `devstral-123b` | Mistral | Devstral 2 123B | proxy | Code specialist |
| `mistral-large-3` | Mistral | Mistral Large 3 675B | proxy | Flagship MoE |
| `kimi-k2.5` | Moonshot AI | Kimi K2.5 | proxy | Coding + reasoning |
| `kimi-k2-thinking` | Moonshot AI | Kimi K2 Thinking | proxy | Chain-of-thought reasoning |
| `minimax-m2.1` | MiniMax | MiniMax M2.1 | proxy | General purpose |
| `nova-pro` | Amazon | Nova Pro | proxy | Multimodal, balanced |
| `nova-lite` | Amazon | Nova Lite | proxy | Fast, lightweight |

## Prerequisites

- **AWS Account** with Bedrock model access enabled
- **AWS CLI** configured (`aws configure` or IAM role/SSO)
- **Python 3.9+** (for LiteLLM proxy)
- **Claude Code CLI** installed

## Quick Start

### 1. Clone and setup

```bash
git clone <repo-url> claude-code-multi-model-bedrock
cd claude-code-multi-model-bedrock
chmod +x scripts/*.sh
```

### 2. Use Anthropic models (no proxy needed)

```bash
# Claude Opus on Bedrock — direct, no setup required
./scripts/claude-model.sh --model claude-opus

# Claude Sonnet on Bedrock
./scripts/claude-model.sh --model claude-sonnet

# Or set env vars directly
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1
export ANTHROPIC_MODEL=us.anthropic.claude-sonnet-4-6-20260217-v1:0
claude
```

### 3. Use third-party models (proxy required)

```bash
# Step 1: Start the LiteLLM proxy (one-time)
./scripts/setup-proxy.sh

# Step 2: Run Claude Code with any model
./scripts/claude-model.sh --model qwen-coder-next
./scripts/claude-model.sh --model deepseek-v3
./scripts/claude-model.sh --model devstral-123b

# With a prompt
./scripts/claude-model.sh --model qwen-coder-next -p "write a Python REST API"
```

### 4. Interactive model picker

```bash
./scripts/claude-model.sh
# Shows numbered list, pick a model
```

### 5. List all available models

```bash
./scripts/claude-model.sh --list
```

## Proxy Management

```bash
# Start proxy (installs litellm if needed)
./scripts/setup-proxy.sh

# Custom port
./scripts/setup-proxy.sh --port 8080

# Check status
./scripts/setup-proxy.sh --status

# Stop proxy
./scripts/setup-proxy.sh --stop

# View logs
tail -f .litellm.log
```

## Manual Configuration (No Scripts)

If you prefer setting env vars directly:

### Anthropic models (native)

```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1
export ANTHROPIC_MODEL=us.anthropic.claude-opus-4-6-20260205-v1:0
claude
```

### Third-party models (via proxy)

```bash
# Terminal 1: Start proxy
pip install "litellm[proxy]"
litellm --config config/litellm-config.yaml --port 4000

# Terminal 2: Run Claude Code
export ANTHROPIC_BASE_URL=http://localhost:4000
export ANTHROPIC_API_KEY=bedrock-proxy
export ANTHROPIC_MODEL=qwen-coder-next
export DISABLE_PROMPT_CACHING=1
claude
```

## Shell Aliases (Optional)

Add to `~/.zshrc` or `~/.bashrc` for quick access:

```bash
# Native Bedrock models
alias cc-opus='CLAUDE_CODE_USE_BEDROCK=1 AWS_REGION=us-east-1 ANTHROPIC_MODEL=us.anthropic.claude-opus-4-6-20260205-v1:0 claude'
alias cc-sonnet='CLAUDE_CODE_USE_BEDROCK=1 AWS_REGION=us-east-1 ANTHROPIC_MODEL=us.anthropic.claude-sonnet-4-6-20260217-v1:0 claude'
alias cc-haiku='CLAUDE_CODE_USE_BEDROCK=1 AWS_REGION=us-east-1 ANTHROPIC_MODEL=us.anthropic.claude-haiku-4-5-20251001-v1:0 claude'

# Proxy models (requires LiteLLM running on :4000)
alias cc-qwen='ANTHROPIC_BASE_URL=http://localhost:4000 ANTHROPIC_API_KEY=bedrock-proxy ANTHROPIC_MODEL=qwen-coder-next DISABLE_PROMPT_CACHING=1 claude'
alias cc-deepseek='ANTHROPIC_BASE_URL=http://localhost:4000 ANTHROPIC_API_KEY=bedrock-proxy ANTHROPIC_MODEL=deepseek-v3 DISABLE_PROMPT_CACHING=1 claude'
alias cc-devstral='ANTHROPIC_BASE_URL=http://localhost:4000 ANTHROPIC_API_KEY=bedrock-proxy ANTHROPIC_MODEL=devstral-123b DISABLE_PROMPT_CACHING=1 claude'
alias cc-kimi='ANTHROPIC_BASE_URL=http://localhost:4000 ANTHROPIC_API_KEY=bedrock-proxy ANTHROPIC_MODEL=kimi-k2.5 DISABLE_PROMPT_CACHING=1 claude'
```

## Customizing the Region

Edit `config/litellm-config.yaml` and change `aws_region_name` for each model. Available regions vary by model — check the [Bedrock model support page](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html).

## Limitations

- **Context window**: Third-party models have smaller context windows (128K or less) compared to Claude's 200K-1M
- **Tool calling quality**: Claude Code relies heavily on structured tool use. Non-Anthropic models may not handle all tool schemas reliably
- **Prompt caching**: Disabled for proxy models (not supported across the translation layer)
- **Streaming**: Works but may have minor formatting differences through the proxy

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `Proxy not reachable` | Run `./scripts/setup-proxy.sh` |
| `AccessDeniedException` | Enable model access in Bedrock console |
| `AWS credentials not configured` | Run `aws configure` or set up SSO |
| `Model not available in region` | Change `aws_region_name` in litellm config |
| `Tool use errors` | Some models have limited tool-calling support — try simpler prompts |

## License

MIT
