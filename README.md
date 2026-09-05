# OpenClaw — Self-Hosted Private AI Agent on Railway (One-Click Deploy)

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/openclaw-private-agent?referralCode=zxcgoT)

Self-host OpenClaw on Railway — a private, always-on AI agent that reaches you on Telegram, Discord and Slack instead of living in a browser tab. It routes to Claude, GPT, Gemini or OpenRouter using your own API keys, and can research, draft, monitor and chain tasks autonomously while your laptop is closed.

## Contents

- [What This Railway Template Deploys](#what-this-railway-template-deploys)
- [Why Self-Host OpenClaw Instead of ChatGPT Plus or a Managed Agent SaaS](#why-self-host-openclaw-instead-of-chatgpt-plus-or-a-managed-agent-saas)
- [Deploy OpenClaw to Railway](#deploy-openclaw-to-railway)
- [Configuring the OpenClaw Railway Template](#configuring-the-openclaw-railway-template)
- [Use Cases for a Self-Hosted OpenClaw Agent](#use-cases-for-a-self-hosted-openclaw-agent)
- [Troubleshooting OpenClaw](#troubleshooting-openclaw)
- [FAQ](#faq)

## What This Railway Template Deploys

| Service | Image | Purpose |
|---|---|---|
| OpenClaw Gateway | Built from the template Dockerfile — `node:24-bookworm` with the `openclaw` npm package pinned to `2026.5.7` installed globally | Agent runtime handling LLM routing, messaging-channel connections, autonomous task execution and the Control UI. Listens on `PORT` (8080); the internal gateway runs on `18789` behind a wrapper |

**Volume:** `/data` — configuration, credentials, agent memory and workspace files. `OPENCLAW_STATE_DIR` defaults to `/data/.openclaw`, `OPENCLAW_WORKSPACE_DIR` to `/data/workspace`.

**Healthcheck:** `/setup/healthz`

**Endpoints:**
- `/setup` — first-run setup wizard, protected by `SETUP_PASSWORD`
- `/openclaw` — Control UI

## Why Self-Host OpenClaw Instead of ChatGPT Plus or a Managed Agent SaaS

A single always-on container on Railway's Hobby plan runs roughly **$1–5/month** of compute depending on usage, plus whatever your chosen LLM provider charges via your own API key. There is no per-seat subscription and no vendor markup on model calls.

| | OpenClaw (self-hosted, this template) | ChatGPT Plus | Managed agent SaaS |
|---|---|---|---|
| Pricing model | Railway compute + your own LLM API key | Flat monthly fee per seat | Per-seat or per-task, often with a markup on model calls |
| Typical cost | ~$1–5/month compute + pay-as-you-go model usage | Fixed subscription regardless of usage | Fixed or usage-based, plus vendor markup |
| Control | Full — your infrastructure, your credentials, your data | OpenAI-hosted | Vendor-hosted; your data and credentials sit with the vendor |
| Where the alternative is stronger | — | No setup or maintenance, official support | Turnkey onboarding and support |

## Deploy OpenClaw to Railway

1. Click **Deploy on Railway** below.
2. Add at least one LLM provider key (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, or `OPENROUTER_API_KEY`).
3. Set `SETUP_PASSWORD` — use Railway's `${{ secret() }}` to generate it.
4. Set `OPENCLAW_GATEWAY_TOKEN` to a fixed secret (e.g. `openssl rand -hex 32`) so it survives restarts.
5. Confirm `OPENCLAW_STATE_DIR` and `OPENCLAW_WORKSPACE_DIR` point inside the mounted `/data` volume.
6. Deploy, wait for the `/setup/healthz` healthcheck to pass, then open `/setup` to run the first-run wizard.
7. Add a messaging channel token (`TELEGRAM_BOT_TOKEN`, `DISCORD_BOT_TOKEN`, or `SLACK_BOT_TOKEN`) and manage the agent at `/openclaw`.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/openclaw-private-agent?referralCode=zxcgoT)

## Configuring the OpenClaw Railway Template

| Variable | Required | Description |
|---|---|---|
| `SETUP_PASSWORD` | Yes | Password protecting the `/setup` wizard. On Railway use `${{ secret() }}` to generate one |
| `OPENCLAW_STATE_DIR` | Yes | Where OpenClaw stores configuration and state. Must point at the volume — `/data/.openclaw` |
| `OPENCLAW_WORKSPACE_DIR` | Yes | Where OpenClaw stores workspace files. Must point at the volume — `/data/workspace` |
| `OPENCLAW_GATEWAY_TOKEN` | No | Auth token for the gateway and Control UI. Generate with `openssl rand -hex 32`; if unset, the wrapper generates one, which is not ideal for production |
| `ANTHROPIC_API_KEY` | No | Anthropic Claude credentials. At least one LLM provider key is needed |
| `OPENAI_API_KEY` | No | OpenAI GPT model access |
| `GEMINI_API_KEY` | No | Google Gemini provider |
| `OPENROUTER_API_KEY` | No | Multi-provider routing through OpenRouter |
| `TELEGRAM_BOT_TOKEN` | No | Telegram channel integration |
| `DISCORD_BOT_TOKEN` | No | Discord channel integration |
| `SLACK_BOT_TOKEN` | No | Slack channel integration |
| `PORT` | No | Port the wrapper listens on. Railway injects this automatically; defaults to 8080 |
| `INTERNAL_GATEWAY_PORT` | No | Internal port the gateway listens on behind the wrapper. Default `18789` |

Use `.env.example` in this repo as a starting point for these values.

## Use Cases for a Self-Hosted OpenClaw Agent

- An always-on personal AI assistant reachable from Telegram, Discord or Slack rather than a browser tab
- Autonomous task execution — research, monitoring and drafting — that keeps running when your laptop is closed
- Routing between Claude, GPT, Gemini or OpenRouter models from one agent using your own API keys
- Keeping agent memory, credentials and workspace files on infrastructure you control

## Troubleshooting OpenClaw

**Agent memory, credentials and configuration reset after every redeploy** — No volume mounted at `/data`, or `OPENCLAW_STATE_DIR` / `OPENCLAW_WORKSPACE_DIR` are not pointing inside it. Both must resolve under `/data`.

**Deployment never becomes healthy and restarts repeatedly** — The Railway healthcheck targets `/setup/healthz` with a 300s timeout. A first boot that cannot write to the volume, or a missing `SETUP_PASSWORD`, will fail that check.

**The `/setup` wizard is reachable by anyone with the URL** — `SETUP_PASSWORD` is unset. It is the only thing protecting first-run configuration.

**Control UI rejects the connection or the token changes between deploys** — `OPENCLAW_GATEWAY_TOKEN` was left blank, so the wrapper generates a new one on each boot. Set it to a fixed secret.

## FAQ

### What LLM providers does this OpenClaw template support?
Anthropic Claude, OpenAI GPT, Google Gemini, and multi-provider routing via OpenRouter — configured with your own API key in each provider's environment variable. At least one is required.

### Do I need a volume to run OpenClaw on Railway?
Yes. `/data` holds configuration, credentials, agent memory and workspace files. `OPENCLAW_STATE_DIR` and `OPENCLAW_WORKSPACE_DIR` must both resolve under `/data`, or state is lost on redeploy.

### How much does it cost to self-host OpenClaw on Railway?
Roughly $1–5/month of Railway compute for a single always-on container, plus whatever your chosen LLM provider charges via your own API key — no per-seat subscription and no markup on model calls.

### Which messaging platforms can the agent connect to?
Telegram, Discord and Slack, configured via `TELEGRAM_BOT_TOKEN`, `DISCORD_BOT_TOKEN` and `SLACK_BOT_TOKEN` respectively.

### How do I secure the setup wizard?
Set `SETUP_PASSWORD` before deploying — it's the only protection on `/setup`. Also set a fixed `OPENCLAW_GATEWAY_TOKEN` so the Control UI token doesn't rotate on every restart.

### Is this template affiliated with OpenClaw or Railway?
No. See the credit note below.

---

*This project uses [OpenClaw](https://github.com/openclaw/openclaw) and is community-maintained. It is not affiliated with, endorsed by, or supported by the OpenClaw project, Railway, or any LLM provider. For licensing terms, see the [upstream repository](https://github.com/openclaw/openclaw).*