# OpenClaw & Monk

This repository contains Monk.io template to deploy OpenClaw AI assistant gateway.

## Prerequisites

- Install Monk
- Register and Login Monk
- Add Cloud Provider (or run locally)

## Clone Repository

```bash
git clone https://github.com/monk-io/monk-openclaw
```

## Load Template

```bash
cd monk-openclaw
monk load MANIFEST
```

## Add Required Secrets

```bash
# Generate and add gateway token
monk secrets add -g openclaw-gateway-token="$(openssl rand -hex 32)"
```

## Deploy

### Standalone Gateway

```bash
$ monk run openclaw/gateway

✔ Starting the run job: local/openclaw/gateway... DONE
✔ Preparing nodes DONE
✔ Checking/pulling images...
✔ [================================================] 100% alpine/openclaw:2026.2.1
✔ Checking/pulling images DONE
✔ Starting containers DONE
✔ Started local/openclaw/gateway
🔩 templates/local/openclaw/gateway
 └─🧊 Peer local
    └─🔩 templates/local/openclaw/gateway
       └─📦 openclaw-gateway running
          ├─🧩 alpine/openclaw:2026.2.1
          ├─🔌 open (public) TCP 0.0.0.0:18789 -> 18789
          └─🔌 open (public) TCP 0.0.0.0:18790 -> 18790

💡 You can inspect and manage your above stack with these commands:
	monk logs (-f) local/openclaw/gateway - Inspect logs
	monk shell     local/openclaw/gateway - Connect to the container's shell
	monk do        local/openclaw/gateway/action_name - Run defined action (if exists)
💡 Check monk help for more!
```

### With Ollama (CPU)

```bash
# Load Ollama template first
monk load ollama/ollama

# Run OpenClaw + Ollama stack
monk run openclaw/stack-ollama
```

### With Ollama (GPU)

```bash
# Load Ollama GPU template first
monk load ollama/ollama-gpu

# Run OpenClaw + Ollama GPU stack
monk run openclaw/stack-ollama-gpu
```

## Pull Ollama Model

After deploying with Ollama, pull a model:

```bash
# Pull a fast model for CPU
monk do openclaw/ollama/pull-model model=qwen2.5:0.5b

# Or pull a larger model for GPU
monk do openclaw/ollama/pull-model model=llama3.3

# List installed models
monk do openclaw/ollama/list-models
```

## Available Runnables

| Runnable | Description |
|----------|-------------|
| `openclaw/gateway` | Standalone OpenClaw gateway |
| `openclaw/cli` | OpenClaw CLI for management |
| `openclaw/stack` | Gateway process group |
| `openclaw/stack-ollama` | Gateway + Ollama (CPU) |
| `openclaw/stack-ollama-gpu` | Gateway + Ollama (GPU) |
| `openclaw/ollama` | Ollama CPU instance |
| `openclaw/ollama-gpu` | Ollama GPU instance |

## CLI Actions

```bash
# Run onboarding wizard
monk do openclaw/cli/onboard

# Login to WhatsApp
monk do openclaw/cli/channels-login

# Check channels status
monk do openclaw/cli/channels-status

# Health check
monk do openclaw/cli/health
```

## Configuration Variables

```yaml
variables:
  openclaw-image: "alpine/openclaw:2026.2.1"  # Docker image
  gateway-port: 18789                          # Gateway HTTP port
  bridge-port: 18790                           # Bridge port
  gateway-bind: "lan"                          # Bind mode (lan, loopback, public)
```

## Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `openclaw-gateway-token` | Yes | API token for gateway access |
| `claude-ai-session-key` | No | Claude AI session key |
| `claude-web-session-key` | No | Claude Web session key |
| `claude-web-cookie` | No | Claude Web cookie |

Add secrets:

```bash
monk secrets add -g openclaw-gateway-token="your-token-here"
```

## Changing the Default Model

The template uses `qwen2.5:0.5b` (CPU) or `llama3.3` (GPU) by default. To use a different model:

1. **Pull the new model:**

```bash
monk do openclaw/ollama/pull-model model=mistral
```

2. **Edit `openclaw.yaml`** - find the `gateway-ollama` section and update:

```json
"models": [
  { "id": "mistral", "name": "Mistral 7B" },
  ...
]
```

And set it as default:

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "ollama/mistral"
    }
  }
}
```

3. **Reload and restart:**

```bash
monk load MANIFEST
monk stop openclaw/stack-ollama
monk run openclaw/stack-ollama
```

> **Important:** The model ID must match the name shown by `monk do openclaw/ollama/list-models`.

## REST API

OpenClaw gateway exposes a REST API on port 18789.

### Health Check

```bash
curl http://localhost:18789/health
```

### Gateway Status

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" http://localhost:18789/api/status
```

## Persistence

Data is persisted under `${monk-volume-path}/openclaw`:

- `config/` - OpenClaw configuration
- `workspace/` - Agent workspaces

Ollama models are persisted under `${monk-volume-path}/ollama`.

## Recommended Models for CPU

| Model | Size | Speed | Command |
|-------|------|-------|---------|
| qwen2.5:0.5b | 398MB | ⚡⚡⚡ Very fast | `monk do openclaw/ollama/pull-model model=qwen2.5:0.5b` |
| tinyllama | 637MB | ⚡⚡⚡ Very fast | `monk do openclaw/ollama/pull-model model=tinyllama` |
| llama3.2:1b | 1.3GB | ⚡⚡ Fast | `monk do openclaw/ollama/pull-model model=llama3.2:1b` |
| gemma2:2b | 1.6GB | ⚡⚡ Fast | `monk do openclaw/ollama/pull-model model=gemma2:2b` |

## Troubleshooting

### Check logs

```bash
monk logs -f openclaw/gateway
monk logs -f openclaw/ollama
```

### Enter container shell

```bash
monk shell openclaw/gateway
monk shell openclaw/ollama
```

### Verify Ollama is running

```bash
curl http://localhost:11434/api/tags
```

## Stop and Clean Up

```bash
# Stop stack
monk stop openclaw/stack-ollama

# Remove completely
monk purge openclaw/stack-ollama
```

## Links

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [OpenClaw Docker Image](https://hub.docker.com/r/alpine/openclaw)
- [Ollama](https://ollama.ai)
