# NemoClaw Setup Workshop

> Step-by-step guide to running **NemoClaw** — NVIDIA's open-source AI agent stack with enterprise-grade privacy and security.

---

## What is NemoClaw?

NemoClaw is an official NVIDIA project that wraps **OpenClaw** with NVIDIA's OpenShell runtime — adding sandboxed execution, declarative security policies, and enterprise-grade privacy to your AI agents. Every network request, file access, and inference call is governed and auditable.

- Powered by NVIDIA NIM (open-source Nemotron models by default)
- Built on top of OpenClaw — inherits all 10+ messaging channel integrations
- Sandboxed Docker environment — agents can't reach what you don't allow
- Apache 2.0 license

> **Status:** Alpha/Early Preview (March 2026) — APIs may change between versions.

---

## Prerequisites

You need **4 things** before starting:

### 1. Linux (Ubuntu 22.04+ recommended)

NemoClaw officially supports **Linux only**. Mac and Windows support is in progress.

> If you're on Mac or Windows, use a Linux VM or a cloud instance (DigitalOcean, AWS EC2, etc.).
> DigitalOcean has a 1-click NemoClaw setup: https://www.digitalocean.com/community/tutorials/how-to-set-up-nemoclaw

### 2. Docker

NemoClaw requires Docker (not Podman). Install Docker Engine:

```bash
# Ubuntu
curl -fsSL https://get.docker.com | sh

# Add your user to the docker group (avoids sudo)
sudo usermod -aG docker $USER
newgrp docker
```

Verify:
```bash
docker --version
docker run hello-world
```

### 3. Node.js 20+

```bash
# Install via nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.bashrc

nvm install 22
nvm use 22
```

Verify:
```bash
node --version   # should be v22.x
```

### 4. NVIDIA API Key (free)

1. Go to https://build.nvidia.com
2. Sign in or create a free account
3. Click **"Get API Key"** in the top right
4. Copy your key — it starts with `nvapi-`

> OpenClaw is **bundled inside NemoClaw** — no separate install needed.

---

## System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| Disk | 20 GB free | 40 GB free |
| CPU | 4 cores | 8 cores |
| GPU | Not required | NVIDIA GPU (for local NIM) |
| OS | Ubuntu 22.04+ | Ubuntu 24.04 |

---

## Setup

### Step 1 — Clone This Repo

Everything you need is already pre-configured in this repo.

```bash
git clone https://github.com/Clawbuilders/nemoclaw-setup.git
cd nemoclaw-setup
```

### Step 2 — Add Your NVIDIA API Key

```bash
cp workshop/.env.example workshop/.env
```

Open `workshop/.env` and replace `nvapi-your-key-here` with your actual key:

```
NVIDIA_API_KEY=nvapi-your-actual-key-here
```

> Get your free key at https://build.nvidia.com → click **"Get API Key"**

Then export it for the onboarding wizard:

```bash
export NVIDIA_API_KEY=$(grep NVIDIA_API_KEY workshop/.env | cut -d= -f2)
```

### Step 3 — Install NemoClaw

```bash
curl -fsSL https://www.nvidia.com/nemoclaw.sh | bash
```

Verify:
```bash
nemoclaw --version
```

### Step 4 — Run the Onboarding Wizard

```bash
nemoclaw onboard
```

This interactive wizard will:
- Download and create your sandboxed Docker environment
- Configure security and network policies
- Set up NVIDIA NIM as the inference backend
- Create your agent workspace at `~/openclaw/workspace/`
- Configure the OpenShell runtime

Follow the prompts. When asked for a provider, select **NVIDIA NIM** and paste your API key.

> This step takes 5–10 minutes on first run due to the sandbox image download.

### Step 5 — Verify the Setup

```bash
nemoclaw <your-agent-name> status
```

Check Docker containers are running:
```bash
docker ps | grep nemoclaw
```

### Step 6 — Connect to Your Agent

```bash
nemoclaw <your-agent-name> connect
```

For an interactive TUI chat interface:
```bash
openclaw tui
```

---

## Quick Test

Send a message to your agent via CLI:

```bash
openclaw agent --agent main --local -m "Hello! What can you do?" --session-id test
```

---

## NemoClaw CLI Reference

```bash
# Onboard and set up for the first time
nemoclaw onboard

# Check status of a specific agent sandbox
nemoclaw <name> status

# Connect to your running agent
nemoclaw <name> connect

# View logs for a specific agent
nemoclaw <name> logs

# Add a security policy to an agent
nemoclaw <name> policy-add

# Stop all NemoClaw services
nemoclaw stop

# Start services
nemoclaw start

# Interactive chat UI (OpenClaw TUI)
openclaw tui

# Monitor via OpenShell terminal
openshell term
```

---

## Changing the LLM Model

Model and inference provider configuration is handled during the `nemoclaw onboard` wizard. To switch providers after setup, re-run:

```bash
nemoclaw onboard
```

And select a different inference profile when prompted:

| Profile | Description |
|---------|-------------|
| `default` | NVIDIA Nemotron via NIM (recommended) |
| `ncp` | NCP-specific configuration |
| `nim-local` | Local NVIDIA NIM (requires GPU + `NEMOCLAW_EXPERIMENTAL=1`) |
| `vllm` | Local vLLM inference (experimental) |

Browse all NVIDIA NIM models: https://build.nvidia.com/models

---

## Security Policies

NemoClaw's standout feature is declarative security. Add policies to control what your agent can reach:

```bash
# Add a policy to your agent
nemoclaw <name> policy-add
```

Policies restrict:
- **Network access** — which domains the agent can call
- **Filesystem access** — which directories the agent can read/write
- **Inference calls** — which models and endpoints are allowed

This ensures your agent can't exfiltrate data or reach unauthorized endpoints.

---

## Troubleshooting

### `docker: permission denied`
Your user isn't in the docker group yet:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### `nemoclaw onboard` fails with out-of-memory
You need at least 8 GB RAM. Check available memory:
```bash
free -h
```
Add swap if needed:
```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### NVIDIA API errors (401 Unauthorized)
- Verify key starts with `nvapi-`
- Check the env var is set: `echo $NVIDIA_API_KEY`
- Confirm the key is active at https://build.nvidia.com

### Docker is not running
```bash
sudo systemctl status docker

# Restart if needed
sudo systemctl restart docker

# Then restart NemoClaw
nemoclaw stop && nemoclaw start
```

### Sandbox download is slow or stuck
Check Docker image pull progress:
```bash
docker images
nemoclaw <name> logs
```

---

## Mac / Windows Users

NemoClaw is **Linux-only** officially. Your options:

**Option A — Cloud VM (easiest)**
Spin up an Ubuntu 22.04 VM on:
- DigitalOcean: https://www.digitalocean.com/community/tutorials/how-to-set-up-nemoclaw (1-click setup)
- AWS EC2: Ubuntu 22.04, t3.large (8 GB RAM)
- Google Cloud: Ubuntu 22.04, e2-standard-2

**Option B — Local Linux VM**
Use UTM (Mac) or VirtualBox (Windows) with Ubuntu 22.04, 8 GB RAM, 30 GB disk.

**Option C — Docker Desktop on Mac/Windows**
Install Docker Desktop and run a Linux VM inside it:
- Download: https://www.docker.com/products/docker-desktop/
- Enable the Ubuntu devcontainer or use a Linux VM
- Then follow the Linux setup steps above inside that environment

---

## Presets — Ready-Made Security Policies

Presets are YAML files that define exactly which services your agent is allowed to reach. Apply them to your sandbox to lock down network access to only what's needed.

This repo includes presets for the most common integrations. Full collection at: https://github.com/VoltAgent/awesome-nemoclaw

### Available Presets

| Preset | File | What It Allows |
|--------|------|---------------|
| NVIDIA NIM | `presets/nvidia-nim.yaml` | Inference calls to `integrate.api.nvidia.com` |
| Telegram | `presets/telegram.yaml` | Telegram Bot API |
| Discord | `presets/discord.yaml` | Discord REST API + WebSocket gateway |
| Slack | `presets/slack.yaml` | Slack API + WebSocket |
| GitHub | `presets/github.yaml` | GitHub REST API |
| Notion | `presets/notion.yaml` | Notion pages, databases, blocks |
| Airtable | `presets/airtable.yaml` | Airtable bases and records |
| Algolia | `presets/algolia.yaml` | Algolia search indexes |
| AWS | `presets/aws.yaml` | S3, STS, Bedrock |
| Cloudflare | `presets/cloudflare.yaml` | Cloudflare API |
| Confluence | `presets/confluence.yaml` | Confluence + Atlassian API |
| GCP | `presets/gcp.yaml` | Google Cloud Storage + Vertex AI |
| GitLab | `presets/gitlab.yaml` | GitLab REST API |
| Google Workspace | `presets/google-workspace.yaml` | Gmail, Drive, Calendar |
| HubSpot | `presets/hubspot.yaml` | HubSpot CRM |
| Linear | `presets/linear.yaml` | Linear GraphQL API |
| Neon | `presets/neon.yaml` | Neon serverless Postgres |
| Sentry | `presets/sentry.yaml` | Sentry error tracking |
| Stripe | `presets/stripe.yaml` | Stripe payments |
| Supabase | `presets/supabase.yaml` | Supabase REST, Auth, Storage |
| Teams | `presets/teams.yaml` | Microsoft Teams + Graph API |
| Vercel | `presets/vercel.yaml` | Vercel deployments |
| Zendesk | `presets/zendesk.yaml` | Zendesk support API |

### How to Apply a Preset

```bash
# 1. Copy the preset into your agent directory
cp presets/telegram.yaml ~/.nemoclaw/

# 2. Edit placeholders if any (open the file and review)
nano ~/.nemoclaw/telegram.yaml

# 3. Apply to your sandbox
nemoclaw <your-agent-name> policy-add

# 4. Set required env vars for the service
export TELEGRAM_BOT_TOKEN=your-token-here
```

> Presets are security baselines — always review and restrict paths/methods to what your agent actually needs before production use.

---

## Links

- NemoClaw GitHub: https://github.com/NVIDIA/NemoClaw
- NemoClaw Docs: https://docs.nvidia.com/nemoclaw/latest/index.html
- Quickstart Guide: https://docs.nvidia.com/nemoclaw/latest/get-started/quickstart.html
- CLI Commands Reference: https://docs.nvidia.com/nemoclaw/latest/reference/commands.html
- NVIDIA NIM Models: https://build.nvidia.com/models
- OpenClaw Docs: https://docs.openclaw.ai
- Community Presets: https://github.com/VoltAgent/awesome-nemoclaw
