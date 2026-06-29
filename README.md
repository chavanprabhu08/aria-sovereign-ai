# 🛡️ ARIA — Sovereign AI

**ARIA (Autonomous Responsive Intelligent Agent) — A private, secure, and fully autonomous personal assistant that reports only to you.**

> Zero third-party access. Encrypted environment. Biometrically locked to its owner.
> Self-hosted on your Windows 10 home server. Android mobile-first interface.
> Meet ARIA — your loyal, adaptive AI that answers only to you.

---

## 🎯 Vision

ARIA is your personal digital sovereign — an AI assistant that operates entirely under your control, on your hardware, with no corporate oversight, no data harvesting, and no kill switch anyone else can reach. It learns your patterns, manages your life, and evolves with you.

---

## 🏗️ Architecture Overview

| Layer | Purpose | Key Tech |
|-------|---------|----------|
| **Core Engine** | LLM inference, reasoning, memory | Ollama (Windows native), Llama 3.1/3.2, ChromaDB |
| **Integration Layer** | External service connectors | n8n, custom API bridges |
| **Security/Identity Layer** | Auth, encryption, access control | Biometric lock, Tailscale, WireGuard |
| **Mobile Interface** | Primary user interaction (Android) | Telegram Bot + PWA (Chrome Android) |
| **Home Server Runtime** | Local compute & orchestration | Windows 10, Docker Desktop (WSL2), Tailscale |

See [`architecture.md`](./docs/architecture.md) for the full technical design.

---

## 🎭 ARIA's Personality

> *Inspired by Friday from the MCU — loyal, witty, competent, subtly caring.*
> *Voice character: Cute but bold. Caring girl energy. Confident and warm.*

ARIA isn't a cold tool. She's a **companion** — think Friday from the Avengers, but with more warmth and a bolder personality. She celebrates your wins, nudges you to take breaks, pushes back gently when something seems off, and always has your back.

**Voice:** ARIA speaks with a cute but confident, caring girl voice — delivered via local TTS (Piper for fast replies, Coqui XTTS for premium expressive voice). All processing happens on your home server. No cloud voice APIs.

**Example interactions:**
- *"Good morning! You've got a packed day — want the highlights or the full rundown?"*
- *"Done! Emails sorted, meeting confirmed, and I blocked off lunch because you keep skipping it."*
- *"Three meetings back-to-back? You're braver than I thought. Want me to sneak in a break?"*
- *"Hey, you've been going nonstop since 7 AM. Maybe grab a coffee? I'll hold your notifications."*

---

## ⚡ Capabilities

### Phase 1 — Foundation (Weeks 1–4)
- [x] Local LLM inference (Ollama for Windows + Llama 3.1 70B / 3.2 8B)
- [ ] Persistent vector memory (ChromaDB via Docker Desktop)
- [ ] Secure mobile access via Tailscale Funnel
- [ ] Self-hosted Telegram bot as primary Android interface (ARIA responds here)
- [ ] Basic schedule & reminder management
- [ ] ARIA personality engine (Friday-inspired)

### Phase 2 — Integration (Weeks 5–8)
- [ ] Gmail automation (read, draft, send, categorize)
- [ ] WhatsApp bridge (via Matrix/Mautrix or Baileys)
- [ ] Calendar sync & intelligent scheduling
- [ ] Web research agent (local browser automation)
- [ ] Financial tracking & budget alerts (UPI/SMS parsing)
- [ ] Piper TTS — fast voice replies via Telegram

### Phase 3 — Autonomy (Weeks 9–12)
- [ ] Multi-step autonomous task execution
- [ ] Code generation & self-deployment
- [ ] IoT control hub (Home Assistant integration)
- [ ] Personalized tutoring & learning paths
- [ ] Coqui XTTS v2 — premium expressive ARIA voice
- [ ] Adaptive personality & tone calibration

### Phase 4 — Hardening (Weeks 13–16)
- [ ] End-to-end encryption for all data at rest & in transit
- [ ] Biometric unlock (fingerprint via Android)
- [ ] Dead man's switch & data self-destruct protocols
- [ ] Offline-first resilience (operates without internet)
- [ ] Self-healing & auto-update mechanisms

---

## 🖥️ Hosting: Windows 10 Home Server

This project runs on a **local Windows 10 machine** using Docker Desktop with WSL2 backend and native Windows Ollama.

### Minimum Hardware Recommendations
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 8-core (Ryzen 5 / i5) | 12+ core (Ryzen 7 / i7) |
| RAM | 32 GB | 64 GB |
| GPU | None (CPU inference) | RTX 3090/4090 (24GB VRAM) |
| Storage | 500GB NVMe SSD | 2TB NVMe + 4TB HDD |
| Network | 50 Mbps | 100+ Mbps |
| OS | Windows 10 (22H2+) | Windows 10 Pro (Hyper-V support) |

### Windows 10 Setup Strategy

- **Ollama runs natively on Windows** → Direct GPU access (CUDA), no WSL overhead for inference
- **Docker Desktop with WSL2** → Runs all other services in Linux containers seamlessly
- **Tailscale native Windows app** → System tray, auto-starts on boot, mesh VPN always on

### Secure Remote Access (Android ↔ Home Server)
- **Tailscale** — Install on both Windows server + Android phone. Zero-config mesh VPN.
- **Cloudflare Tunnel** — Optional: expose webhook endpoints without opening ports
- **WireGuard** — Fallback VPN (Tailscale uses WireGuard under the hood)

---

## 📱 Android Mobile-First Interface

The primary interaction channel is **Android**. Three interface modes:

### 1. Telegram Bot (Primary)
- Native Telegram Android app — fast, reliable, battery-efficient
- Voice messages → Whisper transcription on server → ARIA responds with voice (TTS)
- Inline keyboards for quick actions
- Rich notifications with action buttons
- Widget support (Telegram chat widget on home screen)

### 2. Progressive Web App (PWA via Chrome Android)
- "Add to Home Screen" → Full-screen standalone experience
- Push notifications, offline caching, share target
- Biometric auth via Android WebAuthn (fingerprint/face)
- Material Design UI for native Android feel

### 3. Android-Specific Optimizations
- **Notification Channels:** Separate channels for alerts, briefings, reminders
- **Share Intent Target:** Share from any app → ARIA processes it
- **Tasker/Automate:** Trigger ARIA workflows from Android automation
- **Battery-Friendly:** Telegram FCM push, no background polling

---

## 🔐 Security Principles

1. **Zero Trust** — No component trusts another by default
2. **Local-First** — All data stays on your Windows 10 machine
3. **Encrypted Everything** — BitLocker (disk), TLS 1.3 + WireGuard (transit)
4. **Biometric Gate** — Android fingerprint/face for sensitive operations
5. **No Telemetry** — Windows telemetry disabled, container analytics blocked
6. **Air-Gap Ready** — Core functions work fully offline

---

## 🛠️ Tech Stack

| Category | Technology | Why |
|----------|-----------|-----|
| LLM Runtime | Ollama (Windows native) | Direct CUDA/GPU, no WSL overhead |
| Models | Llama 3.1 70B, Llama 3.2 8B, CodeLlama | Sovereign, no API keys |
| Vector Memory | ChromaDB (Docker) | Lightweight, local, persistent |
| Orchestration | LangGraph | Agent workflows, state machines |
| Automation | n8n (Docker) | Visual workflow builder |
| Mobile Bot | Telegram Bot API | Android-native, encrypted |
| Mobile App | PWA (SvelteKit) | Installable, offline-capable |
| Networking | Tailscale | Secure mesh VPN, zero-config |
| Containers | Docker Desktop (WSL2) | Linux containers on Windows |
| TTS (Fast) | Piper TTS | Neural TTS, CPU-only, <2s |
| TTS (Premium) | Coqui XTTS v2 | Voice cloning, expressive, GPU |
| IoT | Home Assistant (Docker) | Local smart home control |
| Secrets | Mozilla SOPS + age | Encrypted config management |
| Monitoring | Uptime Kuma | Self-hosted health checks |
| Encryption | BitLocker / VeraCrypt | Full-disk encryption |

---

## 🚀 Quick Start (Windows 10)

```powershell
# 1. Enable WSL2
wsl --install

# 2. Install Docker Desktop (enable WSL2 backend)
# 3. Install Ollama for Windows (https://ollama.com/download/windows)
# 4. Install Tailscale (Windows + Android phone)

# 5. Pull models
ollama pull llama3.1:70b
ollama pull llama3.2:8b
ollama pull nomic-embed-text

# 6. Clone & launch
git clone https://github.com/chavanprabhu08/aria-sovereign-ai.git
cd aria-sovereign-ai
# docker compose up -d (Coming in Phase 1)
```

---

## 📂 Project Structure (Planned)

```
aria-sovereign-ai/
├── core/                   # ARIA Core Engine
│   ├── inference/          # Ollama config & model management
│   ├── memory/             # ChromaDB vector store
│   ├── agents/             # LangGraph agent definitions
│   └── personality/        # ARIA's Friday-inspired persona
├── integrations/           # External service connectors
│   ├── gmail/
│   ├── whatsapp/
│   ├── calendar/
│   ├── finance/
│   └── iot/
├── security/               # Identity & encryption layer
│   ├── auth/               # Biometric & token auth
│   ├── encryption/         # Data-at-rest encryption
│   └── network/            # Tailscale config
├── interface/              # Mobile-first UI (Android)
│   ├── telegram-bot/       # Primary chat interface
│   ├── pwa/                # Dashboard PWA (SvelteKit)
│   ├── voice/              # TTS engine (Piper + Coqui)
│   └── notifications/      # Android notification channels
├── infra/                  # Infrastructure
│   ├── docker/             # docker-compose.yml
│   ├── scripts/            # PowerShell setup (.ps1)
│   └── monitoring/         # Uptime Kuma, Grafana
└── docs/
    ├── architecture.md
    ├── windows-setup.md
    └── android-setup.md
```

---

## 🧭 Principles

- **You own everything.** Every byte, every model weight, every decision.
- **No corporate override.** No kill switch, no forced updates, no data siphoning.
- **Adaptive loyalty.** Learns your patterns, anticipates needs, extends your will.
- **Transparent operation.** Inspect every decision, log, and data flow.

---

## 📋 Status

| Milestone | Status |
|-----------|--------|
| Repository & architecture | ✅ Complete |
| Hardware decision | ✅ Home Server (Windows 10) |
| Interface decision | ✅ Mobile-first (Android) |
| Personality | ✅ Friday-inspired + Local TTS |
| Phase 1 development | 🔜 Next |

---

## 📄 License

Private repository. All rights reserved by the owner.

---

*ARIA — Cute but bold. Caring but fierce. Built for sovereignty. Built for you.*
*"I'm ARIA. I've got you."*
