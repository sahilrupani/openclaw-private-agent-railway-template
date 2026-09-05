# Deploy and Host OpenClaw on Railway — a Private AI Agent Railway Template

OpenClaw is an open-source AI agent gateway. Instead of living in a browser tab, it connects to the messaging apps you already use — Telegram, Discord, Slack and others — and acts as a persistent assistant that can research, draft, monitor and chain tasks on your behalf. It routes to every major LLM provider (Claude, GPT, Gemini, OpenRouter) using your own API keys, so your prompts and credentials never pass through a third-party agent vendor.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/openclaw-private-agent?referralCode=zxcgoT)

## 🚀 Quick Start Deployment Guide

### Step 1: Deploy on Railway
1. Click **Deploy on Railway** above
2. Wait for the initial build to complete (~3–5 minutes)

### Step 2: Note your setup password
1. Open the **Variables** tab of the service
2. Copy the value of `SETUP_PASSWORD` — you need it in the next step
3. Keep it secret: it is the only thing protecting first-run configuration

### Step 3: Open the setup wizard
1. Click the URL Railway generated for the service (e.g. `https://your-app.up.railway.app`)
2. Go to `/setup`
3. Log in with your `SETUP_PASSWORD`

### Step 4: Complete the wizard
1. Work through the 3-step wizard
2. Choose your AI provider and paste its API key — see the provider guides below
3. Add messaging channel tokens if you want them (this can be done later)
4. Run setup: it executes `openclaw onboard --non-interactive` inside the container, writes state to the Railway Volume, and starts the gateway

### Step 5: Start chatting
1. Once setup finishes, your Railway URL root (`/`) serves OpenClaw itself — the wrapper reverse-proxies all traffic, WebSockets included, to the gateway
2. Approve your browser under device management if prompted
3. Send your first message

### Step 6: Optional: enable the web terminal
1. Set `ENABLE_WEB_TUI=true` in the Variables tab and redeploy
2. The terminal is then available at `/tui`, protected by the same `SETUP_PASSWORD`
3. It is disabled by default and limited to one concurrent session, auto-closing after 5 minutes idle with a 30-minute hard cap

## About Hosting OpenClaw

This template runs OpenClaw as a single container on Railway, built from a `node:24-bookworm` image with the `openclaw` npm package pinned to `2026.5.7`. A wrapper process serves on `PORT` (8080) and reverse-proxies all traffic — WebSockets included — to the internal gateway on `18789`. Railway's healthcheck targets `/setup/healthz`.

A volume mounts at `/data` and holds configuration, credentials, agent memory and workspace files. `OPENCLAW_STATE_DIR` (`/data/.openclaw`) and `OPENCLAW_WORKSPACE_DIR` (`/data/workspace`) must both resolve inside it, or state is lost on every redeploy. After deploying, `/setup` runs the first-run wizard (protected by `SETUP_PASSWORD`), and once setup completes the OpenClaw UI is served at the URL root (`/`). Setting `OPENCLAW_GATEWAY_TOKEN` to a fixed secret keeps that Control UI token stable across restarts.

The result is a private, always-on agent whose state, credentials, and provider keys stay on infrastructure you control, instead of a vendor's servers.

## Common Use Cases

- **Always-on personal assistant**: reachable from Telegram, Discord or Slack rather than a browser tab.
- **Autonomous task execution**: research, monitoring and drafting keep running when your laptop is closed.
- **Multi-model routing**: switch between Claude, GPT, Gemini or OpenRouter from one agent using your own API keys.
- **Data ownership**: agent memory, credentials and workspace files stay on infrastructure you control.

## Dependencies for OpenClaw Hosting

### Deployment Dependencies
- [OpenClaw (upstream source)](https://github.com/openclaw/openclaw)
- [OpenClaw documentation](https://docs.openclaw.ai/)
- [Anthropic API keys (Claude — recommended)](https://platform.claude.com/)
- [OpenAI API keys (GPT)](https://platform.openai.com/)
- [Google AI Studio (Gemini)](https://aistudio.google.com/)
- [Telegram BotFather](https://t.me/botfather)
- [Discord Developer Portal](https://discord.com/developers/applications)

## 🔑 How to Get API Keys for Different AI Providers

### How to Get an Anthropic Claude API Key? (Recommended)
1. Visit the Anthropic Console at https://platform.claude.com/
2. Sign up or log in, then open **API Keys** in the left sidebar
3. Click **Create Key** and give it a name
4. Copy the key — it is shown only once

### How to Get an OpenAI API Key?
1. Go to the OpenAI Platform at https://platform.openai.com/
2. Create an account or sign in, then open **API keys** from the profile menu
3. Click **Create new secret key** and name it
4. Copy the key — it is shown only once

### How to Get a Google Gemini API Key?
1. Visit Google AI Studio at https://aistudio.google.com/
2. Sign in with your Google account
3. Click **Get API key** in the left menu
4. Select an existing project or create a new one, then click **Create API key**
5. Copy the generated key

### How to Get an OpenRouter API Key?
1. Go to https://openrouter.ai/keys
2. Sign in, then click **Create Key**
3. Name the key and copy it
4. One OpenRouter key gives you routing across many providers

## 💬 How to Add Messaging Channels to OpenClaw

### How to Add a Telegram Bot?
**Step 1: Create your bot**
1. Open Telegram and search for `@BotFather`
2. Send `/newbot`
3. Choose a display name, then a username ending in `bot`
4. BotFather returns a token in the form `123456789:ABCdef...` — copy it

**Step 2: Add it to the agent**
1. Paste the token into the Telegram bot token field in the setup UI
2. Save / re-run setup and wait for it to finish

**Step 3: Start chatting**
1. Search your bot's username in Telegram
2. Press **Start** or send `/start`
3. If a pairing code is required, send any message and enter the code the bot replies with

### How to Add a Discord Bot?
**Step 1: Create the application**
1. Open the Discord Developer Portal at https://discord.com/developers/applications
2. Click **New Application** and name it
3. Open the **Bot** tab and add a bot

**Step 2: Configure the bot**
1. Under **Privileged Gateway Intents**, enable **MESSAGE CONTENT INTENT** — required
2. Click **Reset Token**, copy the token, and store it securely

**Step 3: Invite it to your server**
1. Go to **OAuth2 → URL Generator**
2. Select the `bot` and `applications.commands` scopes
3. Select permissions: Read Messages/View Channels, Send Messages, Read Message History, Embed Links
4. Open the generated URL, pick your server, and authorize

**Step 4: Add it to the agent**
1. Paste the token into the Discord bot token field in the setup UI
2. Save / re-run setup, then mention the bot in a channel to chat

### How to Add a Slack Bot?
**Step 1: Create the app**
1. Go to https://api.slack.com/apps and click **Create New App**
2. Add the bot token scopes your workspace needs, then install the app to the workspace
3. Copy the Bot User OAuth token (starts with `xoxb-`)

**Step 2: Add it to the agent**
1. Paste the token into the Slack bot token field in the setup UI
2. Save / re-run setup, then invite the bot to a channel

## ⚙️ Configuration

| Variable | Required | Description |
|---|---|---|
| `SETUP_PASSWORD` | Yes | Password protecting the `/setup` wizard. On Railway use `${{ secret() }}` to generate one |
| `OPENCLAW_STATE_DIR` | Yes | Where OpenClaw stores configuration and state. Must point at the volume — `/data/.openclaw` |
| `OPENCLAW_WORKSPACE_DIR` | Yes | Where OpenClaw stores workspace files. Must point at the volume — `/data/workspace` |
| `OPENCLAW_GATEWAY_TOKEN` | No | Auth token for the gateway and Control UI. Generate with `openssl rand -hex 32`; if unset the wrapper generates one, which is not ideal for production |
| `ANTHROPIC_API_KEY` | No | Anthropic Claude credentials. At least one LLM provider key is needed |
| `OPENAI_API_KEY` | No | OpenAI GPT model access |
| `GEMINI_API_KEY` | No | Google Gemini provider |
| `OPENROUTER_API_KEY` | No | Multi-provider routing through OpenRouter |
| `TELEGRAM_BOT_TOKEN` | No | Telegram channel integration |
| `DISCORD_BOT_TOKEN` | No | Discord channel integration |
| `SLACK_BOT_TOKEN` | No | Slack channel integration |
| `PORT` | No | Port the wrapper listens on. Railway injects this automatically; defaults to 8080 |
| `INTERNAL_GATEWAY_PORT` | No | Internal port the gateway listens on behind the wrapper. Default 18789 |

## ❓ Frequently Asked Questions (FAQ)

### How much does it cost to run OpenClaw on Railway?
Railway: roughly $5–10/month (Hobby plan, $5 base plus usage) for one always-on container. AI provider costs are separate and billed to your own key — commonly $5–30/month for moderate personal use on Claude or GPT, and often free for light Gemini use.

### Is my data private and secure?
Yes. OpenClaw is self-hosted — configuration, credentials, memory and workspace files stay in your Railway volume, and traffic is served over HTTPS. Your prompts still go to whichever model provider you configure.

### Can I use OpenClaw without Telegram or Discord?
Yes. The web UI at your Railway URL works on its own; messaging channels are entirely optional.

### Can I switch AI providers after setup?
Yes. Return to `/setup`, change the provider and API key, and run setup again.

### Can I run more than one agent?
Yes — deploy the template again. Each deployment gets its own domain, volume and configuration, which is useful for separating personal and work agents.

### How do I reach my agent from my phone?
Open your Railway URL in a mobile browser, or connect a Telegram, Discord or Slack channel and message the bot from those apps.

### What is the difference between OpenClaw, Clawdbot and Moltbot?
They are the same project under successive names — Moltbot became Clawdbot, which became OpenClaw. OpenClaw is the current name.

### Why does my agent's memory, credentials and configuration reset after every redeploy?
No volume is mounted at `/data`, or `OPENCLAW_STATE_DIR` / `OPENCLAW_WORKSPACE_DIR` aren't pointing inside it. Both must resolve under `/data` for state to survive a redeploy.

### Why does the deployment never become healthy and keep restarting?
The Railway healthcheck targets `/setup/healthz` with a 300s timeout. A first boot that can't write to the volume, or a missing `SETUP_PASSWORD`, will fail that check and trigger a restart loop.

### Why is the /setup wizard reachable by anyone with the URL?
`SETUP_PASSWORD` is unset. It is the only thing protecting first-run configuration, so set it before sharing your Railway URL.

### Why does the Control UI reject the connection or the token change between deploys?
`OPENCLAW_GATEWAY_TOKEN` was left blank, so the wrapper generates a new one on every boot. Set it to a fixed secret (e.g. via `openssl rand -hex 32`) to keep it stable.

## 🛠️ Support & Issues

If something isn't working, open an issue at [github.com/sahilrupani/openclaw-private-agent-railway-template/issues](https://github.com/sahilrupani/openclaw-private-agent-railway-template/issues) with:
- A description of what you expected vs. what happened
- Steps to reproduce
- Relevant deploy/runtime logs from the Railway service

---

*This is a community-maintained Railway template for [OpenClaw](https://github.com/openclaw/openclaw). It is not affiliated with, endorsed by, or supported by the OpenClaw maintainers, Railway, or any AI provider mentioned above.*