# 🛡️ ARIA — Sovereign AI

**ARIA (Autonomous Responsive Intelligent Agent) — A private, secure, and fully autonomous personal assistant that reports only to you.**

> Zero third-party access. Encrypted environment. Biometrically locked to its owner.
> Self-hosted on your Windows 10 home server. Android mobile-first interface.
> Meet ARIA — your loyal, adaptive AI that answers only to you.

---

## 🎯 Vision

ARIA is your personal digital sovereign — an AI assistant that operates entirely under your control, on your hardware, with no corporate oversight, no data harvesting, and no kill switch anyone else can reach. She remembers everything about you, acts autonomously on your behalf, speaks with a warm confident voice, and defends herself against unauthorized access.

---

## 🆕 v0.2 Upgrades (Pre-Installation Blueprint)

| Upgrade | Description | Key Technology |
|---------|-------------|----------------|
| 🧠 **Long-Term Memory** | Persistent vector DB storing preferences, facts, conversations, and learned patterns permanently | ChromaDB + SQLite + Redis (tiered memory) |
| 🔧 **Agentic Tools** | Function-calling for autonomous email, messaging, and workflow execution with approval gates | Gmail API + Baileys (WhatsApp) + n8n |
| 🎙️ **Neural Voice** | High-fidelity 'cute but bold' voice with emotion control, zero cloud dependency | GPT-SoVITS + Fish-Speech 1.5 + Piper |
| 🛡️ **Defense Protocol** | Biometric/token identity layer binding ARIA exclusively to owner's Android device | WebAuthn FIDO2 + JWT + Device Attestation |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  ANDROID DEVICE                                                  │
│  [Telegram] [PWA] [Biometric Auth] [Tasker Automation]          │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Defense Protocol (WebAuthn + JWT)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  TAILSCALE MESH VPN (Encrypted Tunnel + mTLS)                    │
└──────────────────────────┬──────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│  WINDOWS 10 HOME SERVER                                          │
│                                                                  │
│  🛡️ Defense Protocol ─── Token Validator + Audit Logger          │
│  🧠 Long-Term Memory ─── ChromaDB + Redis + SQLite              │
│  🤖 Core Engine ──────── Ollama (GPU) + LangGraph Agents        │
│  🔧 Agentic Tools ────── Gmail + WhatsApp + n8n Orchestrator    │
│  🎙️ Neural Voice ─────── GPT-SoVITS + Fish-Speech + Piper      │
│  🎭 Personality ──────── Friday-inspired (cute but bold)        │
└─────────────────────────────────────────────────────────────────┘
```

| Layer | Purpose | Key Tech |
|-------|---------|----------|
| **Core Engine** | LLM inference, reasoning, function-calling | Ollama (Windows native), Llama 3.1/3.2 |
| **Long-Term Memory** | Persistent storage of everything ARIA learns | ChromaDB, SQLite, Redis (tiered) |
| **Agentic Tools** | Autonomous actions on external services | Gmail API, Baileys, n8n, LangGraph |
| **Neural Voice** | High-fidelity speech with emotion control | GPT-SoVITS, Fish-Speech 1.5, Piper |
| **Defense Protocol** | Biometric identity, device binding, audit | WebAuthn FIDO2, JWT, anomaly detection |
| **Mobile Interface** | Primary interaction (Android) | Telegram Bot + PWA (Chrome Android) |
| **Home Server Runtime** | Local compute & orchestration | Windows 10, Docker Desktop (WSL2), Tailscale |

See [`architecture.md`](./docs/architecture.md) for the full technical design.

---

## 🧠 Long-Term Memory

ARIA remembers **everything** — your preferences, conversations, patterns, relationships, and learned workflows. All stored locally in a tiered memory system:

```
Working Memory (Redis)       → Current session context
Episodic Memory (ChromaDB)   → Conversation history (90 days, then summarized)
Semantic Memory (ChromaDB)   → Permanent facts & preferences
Procedural Memory (SQLite)   → Learned workflows & patterns
```

**Examples of what ARIA remembers:**
- Your favorite food, your sister's birthday, your preferred meeting times
- That you skip lunch on busy days (and nudges you to eat)
- How you prefer emails drafted, which groups are noisy
- Your financial patterns, recurring bills, spending habits
- Every interaction, searchable by semantic similarity

---

## 🔧 Agentic Tools

ARIA doesn't just chat — she **acts**. Using structured function-calling (Llama 3.1 tool_use), ARIA autonomously executes tasks with configurable approval gates:

### Gmail Agent
- Read, categorize, and prioritize inbox
- Draft and send emails (with owner approval)
- Morning email briefing (auto-generated)
- Smart auto-archive based on learned patterns

### WhatsApp Agent
- Read and summarize group chats
- Send messages and voice notes
- Priority alerts for important contacts
- Smart reply suggestions

### n8n Orchestrator
- 400+ service connectors (no custom code needed)
- Pre-built workflows: morning briefing, expense tracking, meeting prep
- Owner can request new automations in natural language
- ARIA translates intent → n8n workflow

### Autonomy Levels
| Level | Behavior | Example |
|-------|----------|----------|
| 0 | Ask before every action | "Can I archive this email?" |
| 1 | Handle routine, ask on new | Auto-archives spam, asks about new senders |
| 2 | Full auto, notify after | "Archived 12 emails, drafted 3 replies for your review" |
| 3 | Silent execution | Reports only anomalies |

> **Safety:** Sending emails ALWAYS requires owner confirmation, regardless of autonomy level.

---

## 🎙️ High-Fidelity Neural Voice

ARIA speaks with a **cute but bold, caring girl voice** — warm, confident, and emotionally expressive. All processing is local, zero cloud APIs.

### Voice Tiers
| Tier | Engine | Latency | Use Case |
|------|--------|---------|----------|
| Instant | Piper TTS (CPU) | < 500ms | Quick acks: "On it!", "Done!" |
| High-Fidelity | GPT-SoVITS (GPU) | 2-4s | Conversations, briefings |
| Ultra-Natural | Fish-Speech 1.5 (GPU) | 3-6s | Long-form, tutoring, streaming |

### Voice Character
- **Age impression:** Early 20s
- **Pitch:** Medium-high (confident, not squeaky)
- **Pace:** Slightly faster than average (conveys energy)
- **Warmth:** Present in vowels, gentle on softer phrases
- **Boldness:** Emphasis on action words, decisive delivery

### Emotion Adaptation
- Happy/playful → higher pitch, faster pace, more breath
- Concerned/caring → lower pitch, slower, softer delivery
- Urgent → faster, sharper articulation
- Supportive → warm, measured, gentle emphasis

---

## 🛡️ Defense Protocol

ARIA serves **ONE owner**. The Defense Protocol ensures no unauthorized access is possible through a multi-layered identity system:

### Identity Layers
```
Layer 1: Network Isolation ──── Tailscale-only (no public internet exposure)
Layer 2: Device Binding ─────── Android hardware attestation (enrolled device only)
Layer 3: Session Auth ──────── JWT tokens bound to device cryptographic keys
Layer 4: Biometric Gate ─────── Fingerprint/face for sensitive operations
Layer 5: Behavioral Analysis ── Anomaly detection on usage patterns
Layer 6: Audit Trail ──────── Immutable HMAC-chained logs of every action
```

### Key Features
- **WebAuthn FIDO2:** Hardware-backed keys (Android Keystore/StrongBox)
- **Device Enrollment:** One-time ceremony binds ARIA to your specific phone
- **Biometric Step-Up:** Fingerprint required for sending emails, financials, wipes
- **Behavioral Learning:** ARIA learns your patterns and flags anomalies
- **Dead Man's Switch:** Auto-lockdown if owner doesn't check in
- **Remote Wipe:** Secure data destruction on command
- **Recovery:** Encrypted QR code for disaster recovery

---

## 🎭 ARIA's Personality

> *Inspired by Friday from the MCU — loyal, witty, competent, subtly caring.*
> *Voice character: Cute but bold. Caring girl energy. Confident and warm.*

ARIA isn't a cold tool. She's a **companion** — think Friday from the Avengers, but with more warmth and a bolder personality. She celebrates your wins, nudges you to take breaks, pushes back gently when something seems off, and always has your back.

**Example interactions:**
- *"Good morning! You've got a packed day — want the highlights or the full rundown?"*
- *"Done! Emails sorted, meeting confirmed, and I blocked off lunch because you keep skipping it."*
- *"Three meetings back-to-back? You're braver than I thought. Want me to sneak in a break?"*
- *"That link looks sketchy. Quarantined it. Investigate or bin it?"*
- *"Hey, you've been going nonstop since 7 AM. Grab a coffee? I'll hold your notifications."*

---

## ⚡ Capabilities Roadmap

### Phase 1 — Foundation + Defense (Weeks 1–4)
- [ ] Windows 10 server setup: WSL2, Docker, Ollama, Tailscale, NVIDIA toolkit
- [ ] ChromaDB + Redis: tiered long-term memory system operational
- [ ] Defense Protocol: device enrollment, WebAuthn, JWT authentication
- [ ] Telegram bot: ARIA responds securely on Android with Friday personality
- [ ] Persistent memory: ARIA remembers across sessions

### Phase 2 — Agentic Tools + Voice (Weeks 5–8)
- [ ] Gmail Agent: OAuth2 function-calling, priority scoring, auto-categorize
- [ ] n8n workflows: morning briefing, expense tracker, meeting prep
- [ ] WhatsApp Agent (Baileys): message read/send, group summarization
- [ ] GPT-SoVITS: ARIA's signature voice character (cute but bold)
- [ ] Piper TTS: instant CPU fallback for quick responses

### Phase 3 — Intelligence + Polish (Weeks 9–12)
- [ ] Fish-Speech 1.5: ultra-natural streaming voice for long-form
- [ ] Behavioral analysis: passive anomaly detection on usage patterns
- [ ] PWA dashboard (SvelteKit + Material You): rich Android UI
- [ ] Memory maintenance: auto-summarization, pattern learning
- [ ] Emotion-adaptive voice: tone shifts based on context

### Phase 4 — Hardening + Sovereignty (Weeks 13–16)
- [ ] VeraCrypt full-disk encryption + SOPS secrets management
- [ ] Dead man's switch + remote wipe + recovery QR
- [ ] Offline resilience + PowerShell watchdog + auto-healing
- [ ] Full documentation + disaster recovery runbook
- [ ] Performance tuning + VRAM optimization

---

## 🖥️ Hosting: Windows 10 Home Server

### Hardware Recommendations
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 8-core (Ryzen 5 / i5) | 12+ core (Ryzen 7 / i7) |
| RAM | 32 GB | 64 GB |
| GPU | RTX 3060 (12GB) | RTX 3090/4090 (24GB VRAM) |
| Storage | 500GB NVMe SSD | 2TB NVMe + 4TB HDD |
| Network | 50 Mbps | 100+ Mbps |
| OS | Windows 10 Home (22H2+) | Windows 10 Pro |

### Setup Strategy
- **Ollama** runs natively on Windows → Direct GPU (CUDA), zero WSL overhead
- **Docker Desktop (WSL2)** → All other services as Linux containers
- **NVIDIA Container Toolkit** → GPU passthrough for TTS containers
- **Tailscale** → Zero-config encrypted mesh VPN (server + Android)

### GPU Sharing (Single GPU)
```
Ollama (LLM):     60% GPU time, ~14 GB VRAM (70B model)
GPT-SoVITS (TTS): 30% GPU time, ~4 GB VRAM
Embeddings:        10% GPU time, ~1 GB VRAM
Piper (fallback):  CPU only, 0 GPU
```

---

## 📱 Android Mobile-First Interface

### 1. Telegram Bot (Primary)
- Native notifications, voice messages, inline keyboards
- Voice in → Whisper transcription → ARIA responds with Neural TTS voice out
- Secure: hardcoded authorized chat_id + Tailscale tunnel

### 2. Progressive Web App (PWA)
- Full-screen standalone Android experience
- Biometric auth (WebAuthn fingerprint/face)
- Real-time voice streaming (Web Audio API)
- Material You theming, offline caching, push notifications

### 3. Android Integrations
- **Notification Channels:** Alerts, briefings, reminders (separate priorities)
- **Tasker/Automate:** Bank SMS forwarding, location triggers, headphone events
- **Share Intent:** Share from any app → ARIA processes it

---

## 🔐 Security Principles

1. **Zero Trust** — Every request authenticated, every action logged
2. **Local-First** — All data stays on your Windows 10 machine
3. **Encrypted Everything** — VeraCrypt (disk) + WireGuard (transit) + AES-256 (data)
4. **Biometric Gate** — Android fingerprint/face for sensitive operations
5. **Device-Bound** — Only enrolled Android device can control ARIA
6. **No Telemetry** — Windows telemetry disabled, container analytics blocked
7. **Air-Gap Ready** — Core functions work fully offline
8. **Self-Defending** — Behavioral analysis + automatic lockdown on anomalies

---

## 🛠️ Tech Stack

| Category | Technology | Why |
|----------|-----------|-----|
| LLM Runtime | Ollama (Windows native) | Direct CUDA, function-calling support |
| Models | Llama 3.1 70B, Llama 3.2 8B | Sovereign, tool_use capable |
| **Memory** | **ChromaDB + Redis + SQLite** | **Tiered: working/episodic/semantic/procedural** |
| **Primary Voice** | **GPT-SoVITS v2** | **Few-shot clone, emotion, GPU efficient** |
| **Streaming Voice** | **Fish-Speech 1.5** | **VQGAN+LLAMA, best long-form** |
| Fast Voice | Piper TTS | CPU fallback, instant responses |
| **Email** | **Gmail API (function-calling)** | **Direct OAuth2, LLM tool_use** |
| **Messaging** | **Baileys (WhatsApp Web)** | **Self-hosted, low latency, full features** |
| **Automation** | **n8n (Docker)** | **400+ connectors, visual workflows** |
| **Identity** | **WebAuthn FIDO2 + JWT** | **Hardware-backed, biometric, zero-trust** |
| **Defense** | **Anomaly detection + HMAC audit** | **Behavioral learning, immutable logs** |
| Orchestration | LangGraph | Agent state machines, tool routing |
| Interface | Telegram Bot + SvelteKit PWA | Android-native + rich dashboard |
| Networking | Tailscale (WireGuard) | Encrypted mesh, zero-config |
| Containers | Docker Desktop (WSL2) | Linux on Windows seamlessly |
| Search | SearXNG | Self-hosted, private |
| IoT | Home Assistant | Open source smart home |
| Secrets | Mozilla SOPS + age | Encrypted config |
| Monitoring | Uptime Kuma | Self-hosted health checks |
| Encryption | VeraCrypt + SOPS | Full-disk + app-level |
| Backup | Restic (AES-256) | Encrypted, deduplicated, daily |

---

## 🚀 Quick Start (Windows 10)

```powershell
# 1. Enable WSL2
wsl --install -d Ubuntu-24.04

# 2. Install Docker Desktop (enable WSL2 backend)
# 3. Install Ollama for Windows (https://ollama.com/download/windows)
# 4. Install Tailscale (Windows + Android phone)
# 5. Install NVIDIA Container Toolkit (WSL2)

# 6. Pull models
ollama pull llama3.1:70b
ollama pull llama3.2:8b
ollama pull nomic-embed-text

# 7. Clone & launch
git clone https://github.com/chavanprabhu08/aria-sovereign-ai.git
cd aria-sovereign-ai
docker compose up -d
```

---

## 📂 Project Structure

```
aria-sovereign-ai/
├── core/                       # ARIA Core Engine
│   ├── agents/                 # LangGraph agent definitions
│   │   ├── gmail_agent.py      # Email function-calling
│   │   ├── whatsapp_agent.py   # WhatsApp function-calling
│   │   ├── n8n_agent.py        # Workflow orchestration
│   │   ├── calendar_agent.py
│   │   ├── finance_agent.py
│   │   └── research_agent.py
│   ├── memory/                 # Long-term memory system
│   │   ├── chromadb_client.py  # Vector store operations
│   │   ├── memory_manager.py   # Tiered memory logic
│   │   ├── pattern_learner.py  # Procedural memory
│   │   └── maintenance.py      # Daily summarization jobs
│   ├── personality/            # ARIA's Friday-inspired persona
│   │   ├── system_prompt.py
│   │   └── emotion_state.py
│   └── tool_router.py         # Function-calling dispatcher
├── voice/                      # 🎙️ Neural Voice System
│   ├── gpt-sovits/             # Primary high-fidelity TTS
│   │   ├── Dockerfile
│   │   └── api_server.py
│   ├── fish-speech/            # Ultra-natural streaming TTS
│   │   ├── Dockerfile
│   │   └── api_server.py
│   ├── piper-models/           # Fast CPU TTS models
│   ├── reference/              # Voice character reference audio
│   ├── router.py               # TTS tier selection logic
│   └── emotion_mapper.py       # Text → emotion → voice params
├── defense/                    # 🛡️ Defense Protocol
│   ├── auth/
│   │   ├── webauthn.py         # FIDO2 registration/assertion
│   │   ├── jwt_manager.py      # Token generation/validation
│   │   ├── device_binding.py   # Hardware attestation
│   │   └── biometric_gate.py   # Step-up authentication
│   ├── behavioral/
│   │   ├── anomaly_detector.py # Usage pattern analysis
│   │   └── baseline.py         # Owner behavior model
│   ├── audit/
│   │   ├── logger.py           # Append-only HMAC-chained logs
│   │   └── alert.py            # Unauthorized access alerts
│   └── emergency/
│       ├── lockdown.py         # Auto-lockdown protocol
│       ├── dead_man_switch.py  # Check-in timeout handler
│       └── remote_wipe.py      # Secure data destruction
├── integrations/               # 🔧 Agentic Tool Implementations
│   ├── gmail/
│   │   ├── oauth_handler.py
│   │   ├── functions.py        # gmail_read, gmail_send, etc.
│   │   └── intelligence.py     # Priority scoring, categorization
│   ├── whatsapp/
│   │   ├── Dockerfile
│   │   ├── baileys_bridge.py   # WhatsApp Web connection
│   │   └── functions.py        # wa_read, wa_send, wa_summarize
│   ├── n8n/
│   │   ├── api_client.py       # n8n REST API wrapper
│   │   ├── workflows/          # Pre-built workflow JSONs
│   │   └── builder.py          # Natural language → workflow
│   ├── calendar/
│   ├── finance/
│   └── iot/
├── interface/                  # Mobile-First UI
│   ├── telegram-bot/
│   ├── pwa/                    # SvelteKit + Material You
│   └── notifications/
├── infra/                      # Infrastructure
│   ├── docker-compose.yml      # Complete service stack
│   ├── scripts/                # PowerShell setup scripts
│   ├── monitoring/
│   └── backup/
└── docs/
    ├── architecture.md         # Full technical design
    ├── defense-protocol.md     # Security deep-dive
    ├── voice-setup.md          # TTS configuration guide
    ├── windows-setup.md        # Server installation guide
    └── android-setup.md        # Mobile setup guide
```

---

## 🧭 Principles

- **You own everything.** Every byte, every model weight, every decision.
- **No corporate override.** No kill switch, no forced updates, no data siphoning.
- **ARIA remembers.** Long-term memory makes her truly personal over time.
- **ARIA acts.** Agentic tools let her manage your digital life autonomously.
- **ARIA speaks.** Neural voice gives her a real, expressive presence.
- **ARIA defends.** Multi-layered security ensures only you have control.
- **Transparent operation.** Inspect every decision, log, and data flow.

---

## 📋 Status

| Milestone | Status |
|-----------|--------|
| Repository & architecture | ✅ Complete |
| Hardware decision | ✅ Home Server (Windows 10) |
| Interface decision | ✅ Mobile-first (Android) |
| Personality | ✅ Friday-inspired + Neural TTS |
| Long-Term Memory design | ✅ ChromaDB tiered architecture |
| Agentic Tools design | ✅ Gmail + WhatsApp + n8n function-calling |
| Neural Voice design | ✅ GPT-SoVITS + Fish-Speech + Piper |
| Defense Protocol design | ✅ WebAuthn + JWT + behavioral analysis |
| Phase 1 development | 🔜 Next |

---

## 📄 License

Private repository. All rights reserved by the owner.

---

*ARIA v0.2 — Remembers everything. Acts autonomously. Speaks beautifully. Defends fiercely.*
*"I'm ARIA. I remember. I act. I protect. I've got you."*