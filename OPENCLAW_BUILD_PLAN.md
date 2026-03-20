# OpenClaw — Custom Build Plan

## What Is OpenClaw?

[OpenClaw](https://github.com/openclaw/openclaw) is an **open-source, multi-platform AI personal assistant** written in TypeScript.
It runs as a self-hosted daemon/server and connects to multiple AI providers and messaging channels.

**Core capabilities:**
- Chat with AI (OpenAI, Anthropic/Claude, Google Gemini, OpenRouter, and more)
- Works across Telegram, Discord, Slack, WhatsApp, iMessage, Signal, LINE, and a web UI
- Executes real tasks: web search, terminal commands, browser automation, image generation, TTS
- Native companion apps for iOS, Android, and macOS
- Extensible via plugins and custom skills

---

## Architecture Overview

```
User (Telegram / Discord / Web / CLI)
        │
        ▼
 Channel Adapter  (src/channels/)
        │  normalises message format
        ▼
 Router  (src/routing/)
        │  selects provider + applies security / allowlists
        ▼
 Context Engine  (src/context-engine/)
        │  builds conversation history + injects memory
        ▼
 AI Provider Adapter  (src/providers/)
        │  calls OpenAI / Anthropic / Gemini / OpenRouter API
        ▼
 Agent Harness  (src/agents/)   ← runs tool calls when needed
        │  web search · terminal · browser · image gen
        ▼
 Response Formatter  (src/markdown/)
        │
        ▼
 Channel Adapter  →  sends reply back to user
```

### Monorepo Structure

```
openclaw/
├── src/                  ← Core Node.js / TypeScript backend
│   ├── entry.ts          ← App bootstrap
│   ├── gateway/          ← HTTP + WebSocket gateway
│   ├── channels/         ← Channel abstractions
│   ├── providers/        ← AI provider adapters (OpenAI, Anthropic, Gemini…)
│   ├── agents/           ← Agentic execution harness
│   ├── context-engine/   ← Conversation context + history
│   ├── memory/           ← Pluggable memory system
│   ├── sessions/         ← User session management
│   ├── commands/         ← Slash command handlers
│   ├── plugins/          ← Built-in plugin framework
│   ├── plugin-sdk/       ← Public plugin API
│   ├── routing/          ← Message routing logic
│   ├── security/         ← Auth, token gating, allowlists
│   ├── config/           ← Config loading (env + openclaw.json)
│   ├── tts/              ← Text-to-speech pipeline
│   ├── browser/          ← Browser automation (computer use)
│   ├── web-search/       ← Web search (Brave, Perplexity, Firecrawl)
│   ├── image-generation/ ← Image generation
│   ├── media/            ← Media upload/download pipeline
│   ├── tui/              ← Terminal UI
│   └── wizard/           ← First-run setup wizard
├── ui/                   ← Web frontend (Vite + React)
├── apps/                 ← Native apps (iOS, Android, macOS)
├── extensions/           ← Extension / channel plugins
├── packages/             ← Internal npm packages
├── skills/               ← Agent skill definitions
├── Dockerfile
├── docker-compose.yml
├── fly.toml              ← Fly.io deployment config
├── render.yaml           ← Render.com deployment config
└── openclaw.mjs          ← Main CLI entry point
```

---

## Key Technology Stack

| Area | Tech |
|---|---|
| Runtime | Node.js ≥ 22.12, TypeScript |
| Package manager | pnpm 9+ workspaces |
| Build tool | tsdown (Rollup-based) |
| Test framework | Vitest |
| Web UI | Vite + React |
| AI Providers | OpenAI, Anthropic, Gemini, OpenRouter |
| Channels | Telegram, Discord, Slack, WhatsApp, iMessage, Signal, LINE, web |
| TTS | ElevenLabs, Deepgram |
| Web search | Brave API, Perplexity, Firecrawl |
| Browser automation | Puppeteer / Playwright |
| Deployment | Docker Compose, Fly.io, Render.com |
| Plugin SDK | npm package (`openclaw/plugin-sdk`) |
| Native apps | iOS (Swift), Android (Kotlin), macOS |

---

## Configuration System

Precedence (highest → lowest):

1. Process environment variables
2. `./.env` (project-local)
3. `~/.openclaw/.env` (user-global)
4. `openclaw.json` `env` block
5. `openclaw.json` direct config keys

**Critical env vars:**

```env
# Gateway security (required if gateway is exposed)
OPENCLAW_GATEWAY_TOKEN=change-me-to-a-long-random-token

# AI providers — set at least one
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
OPENROUTER_API_KEY=sk-or-...

# Messaging channels — set whichever you enable
TELEGRAM_BOT_TOKEN=123456:ABCDEF...
DISCORD_BOT_TOKEN=...
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...

# Optional tools
BRAVE_API_KEY=...
ELEVENLABS_API_KEY=...
DEEPGRAM_API_KEY=...
```

---

## Step-by-Step Build Plan

---

### Phase 1 — Environment Setup

**Prerequisites:**
- Node.js ≥ 22.12
  ```bash
  nvm install 22
  nvm use 22
  nvm alias default 22
  ```
- pnpm: `npm install -g pnpm`
- Git

**Steps:**
1. Clone the repository:
   ```bash
   git clone https://github.com/openclaw/openclaw.git my-openclaw
   cd my-openclaw
   ```
2. Install dependencies:
   ```bash
   pnpm install
   ```
3. Copy and configure env:
   ```bash
   cp .env.example .env
   # Edit .env — set OPENCLAW_GATEWAY_TOKEN and at least one AI provider key
   ```
4. Build the project:
   ```bash
   pnpm build
   ```

---

### Phase 2 — Local Run & First Conversation

1. Start the terminal UI:
   ```bash
   node openclaw.mjs
   ```
2. Start the web UI (separate terminal):
   ```bash
   pnpm --filter ui dev
   # Visit http://localhost:5173
   ```
3. Verify the gateway is reachable:
   ```bash
   curl -H "Authorization: Bearer <OPENCLAW_GATEWAY_TOKEN>" http://localhost:<PORT>/
   ```
4. Send a test message via the TUI or web UI.

---

### Phase 3 — Connect a Messaging Channel

#### Telegram (Recommended for first test)
1. Create a bot via [@BotFather](https://t.me/BotFather) → copy the token
2. Set in `.env`:
   ```env
   TELEGRAM_BOT_TOKEN=123456:ABCDEF...
   ```
3. Restart OpenClaw and message your bot

#### Discord
1. Create a Discord app + bot at [discord.com/developers](https://discord.com/developers)
2. Set `DISCORD_BOT_TOKEN` in `.env`
3. Invite the bot to your server with the appropriate OAuth2 scopes

#### Slack
1. Create a Slack app with **Socket Mode** enabled
2. Set `SLACK_BOT_TOKEN` and `SLACK_APP_TOKEN` in `.env`

---

### Phase 4 — Customize AI Configuration

Create or edit `~/.openclaw/openclaw.json`:

```json
{
  "model": "gpt-4o",
  "systemPrompt": "You are a helpful personal assistant named [YourName].",
  "tools": {
    "webSearch": true,
    "browser": false,
    "imageGeneration": true
  }
}
```

Options to configure:
- **Default model** — `gpt-4o`, `claude-3-5-sonnet`, `gemini-2.0-flash`, etc.
- **Model routing** — route different channels to different models
- **System prompt** — personalize the assistant's persona
- **Tools** — enable/disable web search, browser, image generation
- **Memory plugin** — choose a memory backend for persistent context

---

### Phase 5 — Add Plugins & Skills

**Install a community plugin:**
```bash
pnpm openclaw plugins add <plugin-name>
```
Browse: https://docs.openclaw.ai/plugins/community

**Build a custom plugin:**
1. Create `extensions/my-plugin/` as a new workspace package
2. Import types from `openclaw/plugin-sdk`
3. Define tools, hooks, and slash commands
4. Add the plugin path to your `openclaw.json` extensions list

**Create custom Skills:**
- Add a `SKILL.md` file to the `skills/` directory
- Skills are instruction documents the agent reads to perform specialized tasks

---

### Phase 6 — Enable Advanced Features

| Feature | Required Env Var | Notes |
|---|---|---|
| Text-to-Speech | `ELEVENLABS_API_KEY` or `DEEPGRAM_API_KEY` | Enable TTS in config |
| Web Search | `BRAVE_API_KEY` and/or `PERPLEXITY_API_KEY` | Agent tool |
| Image Generation | `OPENAI_API_KEY` (DALL-E) | Agent tool |
| Browser automation | — | Enable in high-trust security mode |
| Scheduled cron tasks | — | Configure cron rules in `openclaw.json` |
| Link content extraction | — | Enabled by default |

---

### Phase 7 — Docker Deployment (Self-Hosted)

1. Configure `.env` with all required variables
2. Start with Docker Compose:
   ```bash
   docker compose up -d
   ```
3. Set up a reverse proxy (nginx or Caddy) to terminate HTTPS
4. Point your domain to the server
5. Optionally configure systemd / launchd for auto-start on boot

**Security checklist:**
- [ ] `OPENCLAW_GATEWAY_TOKEN` is a strong, random secret (`openssl rand -hex 32`)
- [ ] Gateway is only exposed via HTTPS reverse proxy
- [ ] Firewall blocks direct access to the gateway port
- [ ] Browser automation is disabled unless explicitly needed

---

### Phase 8 — Cloud Deployment (Optional)

#### Fly.io
```bash
fly launch            # uses fly.toml
fly secrets set OPENCLAW_GATEWAY_TOKEN=...
fly secrets set OPENAI_API_KEY=...
fly deploy
```

#### Render.com
- Connect your GitHub fork to Render
- `render.yaml` provides the service definition
- Set all env vars in the Render dashboard
- Deploy

---

### Phase 9 — Native Companion Apps (Optional / Advanced)

| Platform | Requirements |
|---|---|
| iOS | Xcode, Apple Developer Account |
| Android | Android Studio |
| macOS | Xcode |

Steps:
1. Open the app project in `apps/<platform>/`
2. Configure the app to point to your gateway URL
3. Pair with the gateway via QR code (`src/pairing/`)
4. Build from source following `CONTRIBUTING.md`

---

### Phase 10 — Branding & Making It Your Own

- [ ] **Fork the repo** to your GitHub account
- [ ] **Rename the project** — find/replace `openclaw` / `OpenClaw` in config, docs, and package names
- [ ] **Customize the web UI** — edit React components in `ui/src/`, update colors, logo, and branding
- [ ] **Write a custom system prompt** that defines your assistant's persona
- [ ] **Add custom slash commands** in `src/commands/`
- [ ] **Build custom channel extensions** in `extensions/`
- [ ] **Deploy your fork** via Docker or a cloud platform

---

## Key Files to Study

| File | Purpose |
|---|---|
| `README.md` | Full setup and usage guide |
| `VISION.md` | Project goals and roadmap |
| `CONTRIBUTING.md` | Development workflow |
| `.env.example` | All configurable environment variables |
| `src/entry.ts` | Application bootstrap |
| `src/gateway/` | HTTP/WebSocket API server |
| `src/providers/` | AI provider adapters |
| `src/channels/` | Channel abstractions |
| `src/plugin-sdk/` | Plugin development API |
| `ui/src/` | Web frontend (React) |
| `docker-compose.yml` | Container setup |

---

## Recommended Quick Start Order

1. ✅ Clone + `pnpm install` + `pnpm build`
2. ✅ Set `OPENAI_API_KEY` (or `GEMINI_API_KEY`) in `.env`
3. ✅ Run locally with `node openclaw.mjs` — confirm chat works
4. ✅ Connect Telegram — message your bot
5. ✅ Customize system prompt in `openclaw.json`
6. ✅ Enable web search (`BRAVE_API_KEY`)
7. ✅ Deploy with Docker Compose on your server
8. ✅ Fork, brand, and make it yours

---

*Based on: https://github.com/openclaw/openclaw (commit 2f65ae1)*
