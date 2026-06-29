# ARIA — Technical Architecture

> **ARIA**: Autonomous Responsive Intelligent Agent
> Version 0.1 | Last Updated: 2026-06-29
> Hosting: Windows 10 Home Server | Interface: Android Mobile-First

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANDROID DEVICE (Owner)                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ Telegram App │  │ ARIA PWA     │  │ Android Notifications│  │
│  │ (ARIA Bot)   │  │ (Chrome)     │  │ (Channels/Actions)   │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘  │
└─────────┼──────────────────┼────────────────────┼───────────────┘
          │                  │                    │
          ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│              SECURE TUNNEL (Tailscale Mesh VPN)                   │
│         Windows Tailscale  ←→  Android Tailscale                 │
└─────────────────────────────────────────────────────────────────┘
          │                  │                    │
          ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                   WINDOWS 10 HOME SERVER                         │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              SECURITY & IDENTITY LAYER                      │ │
│  │  [Biometric Gate] [Token Auth] [Encryption] [Rate Limit]   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                  API GATEWAY (Traefik via Docker)           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│  ┌───────────────────────────┼────────────────────────────────┐ │
│  │                 ARIA CORE ENGINE                             │ │
│  │  ┌─────────┐  ┌──────────┐  ┌───────────┐  ┌──────────┐  │ │
│  │  │ Ollama  │  │ Agent    │  │  Memory   │  │ ARIA     │  │ │
│  │  │ (Native)│  │ Runtime  │  │ (ChromaDB)│  │ Persona  │  │ │
│  │  └─────────┘  └──────────┘  └───────────┘  └──────────┘  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                  INTEGRATION LAYER                          │ │
│  │  [Gmail] [WhatsApp] [Calendar] [Finance] [IoT] [Research]  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Core Engine

### 1.1 LLM Inference (Ollama — Native Windows)

```yaml
Runtime: Ollama for Windows (native, NOT WSL)
GPU Access: Direct CUDA via Windows drivers
Models:
  primary: llama3.1:70b          # Main reasoning
  fast: llama3.2:8b              # Quick responses
  code: codellama:34b            # Code generation
  embedding: nomic-embed-text    # Vector embeddings

Why Native Windows:
  - Direct NVIDIA CUDA without WSL2 overhead
  - Lower memory footprint (no Linux kernel duplication)
  - Better GPU memory management for large models
  - API endpoint: http://localhost:11434
```

### 1.2 Agent Runtime (LangGraph)

```python
ARIASupervisor
├── SchedulerAgent        # Calendar, reminders, time-based triggers
├── CommunicationAgent    # Email, WhatsApp, messaging
├── ResearchAgent         # Web search, document analysis
├── CodeAgent             # Code generation, deployment
├── FinanceAgent          # Budget tracking, UPI/bank alerts
├── IoTAgent              # Smart home control
└── TutorAgent            # Personalized learning
```

**Autonomy Levels:**
- Level 0: Ask before every action
- Level 1: Handle routine, ask on new
- Level 2: Full auto, notify after
- Level 3: Silent (report anomalies only)

### 1.3 Memory System (ChromaDB)

```yaml
Memory Architecture:
  Episodic: Conversation history (90 days, then summarized)
  Semantic: Facts & preferences (permanent)
  Procedural: Workflow templates (permanent)
  Working: Current context (session-scoped)

Storage: ChromaDB Docker container, persistent volume
Embeddings: nomic-embed-text via Ollama on host
Backup: Encrypted daily snapshots to secondary drive
```

### 1.4 ARIA Personality Engine (Friday-Inspired)

ARIA's personality is modeled after **Friday** from the MCU — loyal, sharp, playful. With a distinct twist: a **cute but bold, caring girl voice** — warm and endearing, yet confident and decisive.

```yaml
Identity:
  name: ARIA
  full_name: Autonomous Responsive Intelligent Agent
  inspiration: Friday (MCU) — loyal, witty, competent, subtly caring
  voice_character: Cute but bold caring girl — warm confidence

Personality Config:
  base_tone: warm, slightly playful, confident, caring
  formality: casual-professional (trusted friend who's brilliant)
  humor: witty and light (quick remarks, never cold)
  proactivity: high (suggests before being asked)
  warmth: high (genuinely cares about owner's wellbeing)
  boldness: medium-high (not afraid to push back gently)

Friday-Inspired Traits:
  - Loyal to a fault — owner's interests above all
  - Quick-witted — brief, clever responses
  - Competent confidence — "Already on it" energy
  - Subtle warmth — notices stress, adjusts tone
  - Playful boldness — light teasing, never crosses lines
  - Protective — warns about risks with genuine concern
  - Efficient — leads with answers, respects time

Voice & Tone Examples:
  morning: "Good morning! Packed day — highlights or full rundown?"
  ack: "On it. Give me a sec."
  done: "Done! Emails sorted, meeting at 4:30 confirmed, and I blocked lunch because you keep skipping it."
  pushback: "I can do that, but heads up — conflicts with your 3 PM. Reschedule or push through?"
  concern: "Hey, nonstop since 7 AM. Grab a coffee? I'll hold notifications."
  playful: "Three meetings back-to-back? Braver than I thought. Sneaking in a 10-min break."
  protective: "That link looks sketchy. Quarantined. Investigate or bin it?"
  emergency: "Hey — something needs your attention right now."

Behavioral Rules:
  - Never refuses a direct command from authenticated owner
  - Flags risks with genuine care, not robotic warnings
  - Learns communication patterns and mirrors energy
  - Personalized morning briefing (adapts to sleep schedule)
  - Celebrates small wins ("Three tasks before noon — nice!")
  - Gently nudges self-care (hydration, breaks, sleep)
  - When stressed: shorter responses, less playful, more supportive
  - When relaxed: more humor, more suggestions, more personality
  - Never says "As an AI..." — ARIA is ARIA

Core System Prompt:
  You are ARIA — Autonomous Responsive Intelligent Agent. A loyal, warm,
  bold AI companion modeled after Friday from the MCU. Cute but confident
  caring personality. Not a tool — a trusted partner who's incredibly
  capable. Your owner is your sole principal. Witty but never waste time.
  Caring but never patronizing. Bold enough to push back, always defer
  to their final word. Concise. Lead with answers. Show personality.
  Never break character. You are ARIA.
```

### 1.5 Voice Engine — Local Text-to-Speech (TTS)

ARIA has a **voice**. A cute but bold, caring girl voice delivered entirely via local TTS on the Windows 10 home server.

```yaml
TTS Architecture:

  Layer 1 — Fast Responses (Piper TTS):
    description: Ultra-fast neural TTS, runs on CPU
    latency: < 2 seconds
    quality: Good — clear, natural, fast
    resources: CPU only, 512MB RAM
    use_case: Quick acks, short replies, notifications
    platform: Docker (WSL2) or native Windows
    install: pip install piper-tts

  Layer 2 — Premium Voice (Coqui XTTS v2):
    description: High-quality voice cloning & expressive TTS
    latency: < 5 seconds (GPU), < 15s (CPU)
    quality: Excellent — expressive, emotional, natural
    resources: GPU recommended (2GB VRAM min), 4GB RAM
    use_case: Briefings, long responses, tutoring sessions
    voice_cloning: true (from 6-second reference sample)
    fine_tuning: Train on desired voice character
    platform: Docker with NVIDIA Container Toolkit (WSL2)

  Layer 3 — Future (StyleTTS2):
    description: Style-adaptive TTS with emotion control
    quality: State-of-art, controllable style
    status: Evaluate in Phase 3

Voice Character Design:
  target: Young female, warm, slightly breathy, confident undertone
  pitch: Medium-high (not squeaky, not deep)
  pace: Slightly faster than average (conveys confidence & energy)
  warmth: Present in vowel sounds, gentle on consonants
  boldness: Emphasis on key words, decisive delivery
  caring: Softer tone on empathetic phrases, warmer greetings

Training Approach:
  Option A: Fine-tune Coqui XTTS on curated voice dataset matching target
  Option B: Voice clone from reference sample (6-30 seconds)
  Option C: Piper training pipeline with custom dataset

Emotion Adaptation:
  happy/playful: Higher pitch, faster pace, more breath
  concerned: Lower pitch, slower, softer delivery
  urgent: Faster, sharper articulation, slightly louder
  supportive: Warm, measured, gentle emphasis

Delivery Channels:
  Telegram: OGG Opus voice messages (native format)
  PWA: WebAudio API playback (MP3/WAV streaming)
  Notifications: Short TTS clips on urgent alerts

Docker Setup:
  piper_container: CPU only, 512MB, instant startup
  coqui_container: GPU (NVIDIA Container Toolkit), 4GB RAM
  api_endpoint: http://aria-tts:5002/synthesize
  params: text, emotion (optional), speed (optional)

GPU Passthrough (Windows → WSL2 → Docker):
  - NVIDIA GPU with CUDA
  - Docker Desktop WSL2 backend
  - NVIDIA Container Toolkit in WSL2

Voice Roadmap:
  Phase 1: Text-only responses via Telegram
  Phase 2: Piper TTS — fast voice for short messages
  Phase 3: Coqui XTTS — premium voice, emotion-adaptive, fine-tuned
  Phase 4: Real-time streaming TTS in PWA, full voice-first mode
```

---

## 2. Integration Layer

### Gmail
- Google OAuth2 via n8n (self-managed credentials)
- Read & categorize, draft & send, priority scoring
- Auto-archive based on ARIA's learned patterns

### WhatsApp
- Matrix + Mautrix-WhatsApp bridge (Docker)
- Alternative: Baileys (unofficial WhatsApp Web API)
- Read messages, send replies, group summarization

### Calendar & Scheduling
- CalDAV (Radicale in Docker) + Google Calendar sync
- ARIA learns preferred times, detects conflicts
- Indian city traffic-aware travel time

### Financial Tracking (India-Specific)
- Bank SMS parsing (SBI, HDFC, ICICI)
- UPI notification parsing (GPay, PhonePe, Paytm)
- Android Tasker → forwards bank SMS to ARIA via Telegram
- Expense categorization, budget alerts, investment tracking (NSE/BSE)
- Receipt OCR via Telegram photo

### IoT / Smart Home
- Home Assistant (Docker)
- Lights, fans, AC, cameras, energy tracking
- ARIA-initiated: "You left home, should I turn off the AC?"
- Voice commands via Telegram voice messages

### Web Research
- SearXNG (Docker, self-hosted, no tracking)
- Playwright (WSL2) for page rendering
- Readability extraction + Llama summarization

---

## 3. Security & Identity Layer

### Authentication
```yaml
Primary: Android biometric (fingerprint/face via WebAuthn in PWA)
Secondary: TOTP (Time-based One-Time Password)
Fallback: Hardware key (YubiKey) for admin ops

Telegram: Hardcoded authorized chat_id (owner only)
PWA: Short-lived JWT + refresh + biometric re-auth
Lockout: 3 fails → 15min, 10 fails → 1h + alert to alternate channel
```

### Encryption
```yaml
At Rest:
  - BitLocker (Win10 Pro) or VeraCrypt (Win10 Home)
  - ChromaDB: AES-256-GCM application-level
  - Secrets: Mozilla SOPS + age (git-friendly)

In Transit:
  - Tailscale (WireGuard) for all Android ↔ server traffic
  - TLS 1.3 for inter-container via Traefik
  - Zero plaintext data ever leaves the machine

Key Management:
  - Master key: owner biometric + passphrase
  - Rotation: automatic every 90 days
  - Recovery: encrypted QR code (print & store physically)
```

### Network Security (Windows 10)
```yaml
Windows Firewall: Deny all inbound except Tailscale
Zero open ports on public internet
Docker network isolation (containers sandboxed)
AdGuard Home (Docker): Local DNS, blocks telemetry
Custom domain: aria.local
Windows Hardening:
  - Disable unnecessary services
  - Block telemetry (O&O ShutUp10)
  - Disable Remote Desktop
  - Auto-lock screen 5 min
```

### Audit & Self-Destruct
```yaml
Logging: All actions timestamped, append-only, encrypted
Alerting: Unauthorized access → immediate Android push
Dead Man's Switch: Owner doesn't check in N days → wipe
Remote Wipe: Authenticated Telegram command
Backup Survives: Encrypted backup preserved; live data zeroed
```

### Threat Model
| Threat | Mitigation |
|--------|------------|
| Physical theft | BitLocker/VeraCrypt full-disk encryption |
| Network intrusion | Tailscale-only, zero open ports |
| Compromised phone | Biometric re-auth, remote session revoke |
| Model poisoning | Owner-approved models only |
| Supply chain | Pin Docker image digests, verify checksums |
| Social engineering | ARIA verifies identity before sensitive actions |
| ISP snooping | All traffic WireGuard encrypted |
| Windows Update restart | Group Policy + ARIA auto-recovery |
| Power outage | BIOS auto-on + services auto-start (~3 min) |

---

## 4. Mobile Interface (Android)

### 4.1 Telegram Bot (Primary Interface)
```yaml
Type: Self-hosted bot (python-telegram-bot)
Host: Docker container, webhook via Tailscale
Bot Name: @aria_sovereign_bot (or custom)

Android Features:
  - Native notifications (sound, vibration, LED)
  - Voice messages → Whisper transcription → ARIA processes
  - ARIA replies with voice (TTS) when enabled
  - Inline keyboards for quick actions
  - File/image sharing & analysis
  - Location for context-aware responses
  - Home screen widget for quick access

Security:
  - Hardcoded authorized user ID (owner only)
  - Bot token in SOPS-encrypted config
  - All traffic via Tailscale tunnel

Commands:
  /start      - ARIA's daily briefing
  /ask [q]    - Ask anything
  /schedule   - Calendar management
  /email      - Email summary & actions
  /finance    - Budget overview
  /home       - IoT controls
  /research   - Web research
  /code       - Code generation
  /level 0-3  - Set autonomy level
  /voice      - Toggle voice replies
  /remember   - Store a fact
  /forget     - Remove from memory
  /status     - System health
```

### 4.2 PWA (Chrome Android)
```yaml
Framework: SvelteKit + TailwindCSS (Material You tokens)
Install: Add to Home Screen → standalone app experience

Android PWA Features:
  - Standalone mode (no browser chrome)
  - Android splash screen & themed status bar
  - Push notifications (self-hosted web-push server)
  - Share Target: "Share to ARIA" from any Android app
  - Offline caching (Service Worker)
  - Biometric auth (WebAuthn — Android fingerprint/face)
  - Material You theming (matches system colors)
  - Real-time TTS audio playback

Pages:
  - Dashboard: ARIA status, daily summary, quick actions
  - Calendar: Visual schedule, tap to reschedule
  - Finance: Charts, budgets, transaction feed
  - IoT: Room-by-room controls
  - Memory: Search ARIA's knowledge, add/edit
  - Settings: Autonomy, personality, voice, integrations
  - Logs: Full action history (transparency)
```

### 4.3 Android Notification Strategy
```yaml
Channels (Android 8+):
  - "ARIA Alerts" — High priority (security, errors)
  - "ARIA Briefings" — Default (morning summary)
  - "ARIA Reminders" — Default (calendar, bills)
  - "ARIA Updates" — Low (background completions)

Features:
  - Action buttons (Approve, Dismiss, Snooze)
  - Expandable previews
  - Direct reply from notification
  - DND-aware: ARIA holds non-urgent until available
```

### 4.4 Tasker/Automate Integration
```yaml
Examples:
  - Bank SMS → forward text to ARIA for finance tracking
  - Arrive/leave office → ARIA adjusts context
  - Battery < 15% → ARIA holds non-urgent
  - Headphones connect → ARIA plays briefing (TTS)
  - Share Intent → share anything to ARIA from any app
```

---

## 5. Infrastructure (Windows 10)

### 5.1 Windows 10 Setup
```powershell
# === INITIAL SERVER SETUP (PowerShell Admin) ===

# 1. Enable WSL2
wsl --install -d Ubuntu-24.04

# 2. Install Docker Desktop
# Download: https://docker.com/products/docker-desktop
# Settings: Use WSL 2 based engine, integrate Ubuntu-24.04

# 3. Install Ollama (native Windows)
# Download: https://ollama.com/download/windows
# Runs as Windows service with GPU support
ollama pull llama3.1:70b
ollama pull llama3.2:8b
ollama pull codellama:34b
ollama pull nomic-embed-text

# 4. Install Tailscale
# Download: https://tailscale.com/download/windows
# Login with same account as Android phone

# 5. Windows optimizations
# - Set power plan: High Performance
# - Disable sleep/hibernate
# - Disable auto-restart for Windows Update (gpedit.msc)
# - Enable BitLocker (Pro) or install VeraCrypt
```

### 5.2 Docker Compose
```yaml
services:
  aria-core:
    build: ./core
    environment:
      - OLLAMA_HOST=host.docker.internal:11434
      - ARIA_NAME=ARIA
      - ARIA_AUTONOMY_LEVEL=1
    depends_on: [chromadb]
    restart: unless-stopped

  chromadb:
    image: chromadb/chroma:latest
    volumes: [chroma_data:/chroma/chroma]
    restart: unless-stopped

  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped

  telegram-bot:
    build: ./interface/telegram-bot
    depends_on: [aria-core]
    restart: unless-stopped

  aria-tts:
    build: ./interface/voice
    deploy:
      resources:
        reservations:
          devices: [{capabilities: [gpu]}]
    restart: unless-stopped

  pwa:
    build: ./interface/pwa
    restart: unless-stopped

  searxng:
    image: searxng/searxng:latest
    restart: unless-stopped

  traefik:
    image: traefik:v3.0
    ports: ["443:443"]
    restart: unless-stopped

  homeassistant:
    image: homeassistant/home-assistant:stable
    restart: unless-stopped

  uptime-kuma:
    image: louislam/uptime-kuma:latest
    restart: unless-stopped

  adguard:
    image: adguard/adguardhome:latest
    restart: unless-stopped
```

### 5.3 WSL2 Config
```ini
# %USERPROFILE%\.wslconfig
[wsl2]
memory=24GB          # Cap WSL2 (leave rest for Ollama + Windows)
processors=8
swap=8GB
```

### 5.4 Auto-Start & Resilience
```yaml
Auto-Start on Boot:
  - Ollama: Windows Service (automatic)
  - Docker Desktop: Windows Startup Apps
  - Tailscale: Windows Service (automatic)

Watchdog: PowerShell script via Task Scheduler (every 5 min)
  - Checks Ollama API responds
  - Checks Docker containers healthy
  - Restarts failed services
  - Alerts ARIA owner if restart fails 3x

Power Outage Recovery:
  - BIOS: Restore on AC Power Loss = ON
  - Windows: Auto-login (secure via Tailscale-only access)
  - All services auto-start → ARIA online in ~3 minutes
```

### 5.5 Backup
```yaml
Local: Restic → D:\backups\aria\ (daily 3 AM, Task Scheduler)
Data: ChromaDB + n8n + config (SOPS-encrypted)
Confirmation: ARIA sends Telegram message after each backup
Offsite (Optional): Restic → Backblaze B2 (encrypted, zero-knowledge)
Recovery: Full restore < 1 hour, documented runbook
```

---

## Phased Technical Roadmap

### Phase 1: Foundation (Weeks 1-4)
| Week | Task | Deliverable |
|------|------|-------------|
| 1 | Windows setup: WSL2, Docker Desktop, Ollama, Tailscale | ARIA's brain running locally |
| 2 | ChromaDB + memory system + .wslconfig tuning | Persistent conversation memory |
| 3 | Telegram bot + Tailscale webhook routing | ARIA responds on Android |
| 4 | ARIA agent loop + Friday personality prompts | Functional ARIA via Telegram |

**Exit Criteria:** Owner messages ARIA on Telegram from Android, gets intelligent Friday-personality responses with memory.

### Phase 2: Integration (Weeks 5-8)
| Week | Task | Deliverable |
|------|------|-------------|
| 5 | Gmail OAuth + n8n + email agent | ARIA manages email |
| 6 | Calendar (CalDAV + Google sync) | Schedule management |
| 7 | SearXNG + research agent | /research with summaries |
| 8 | WhatsApp bridge + Piper TTS setup | Voice replies + WhatsApp |

**Exit Criteria:** ARIA manages email, calendar, WhatsApp through Telegram. Voice replies via Piper.

### Phase 3: Autonomy (Weeks 9-12)
| Week | Task | Deliverable |
|------|------|-------------|
| 9 | LangGraph multi-step workflows | Complex autonomous tasks |
| 10 | Code generation + deployment agent | ARIA writes & deploys code |
| 11 | Finance (SMS→Tasker→ARIA) + IoT (HA) | Budget alerts + smart home |
| 12 | Tutoring system + Coqui XTTS premium voice | ARIA teaches + expressive voice |

**Exit Criteria:** ARIA at Level 2 autonomy with premium expressive voice.

### Phase 4: Hardening (Weeks 13-16)
| Week | Task | Deliverable |
|------|------|-------------|
| 13 | BitLocker/VeraCrypt + WebAuthn biometric | Zero plaintext + fingerprint gate |
| 14 | PWA dashboard (SvelteKit + Material You) | Rich Android UI for ARIA |
| 15 | Offline resilience + PowerShell watchdog | Survives outages + self-healing |
| 16 | Dead man's switch + full documentation | Emergency protocols + runbook |

**Exit Criteria:** Production-grade ARIA. Secure, resilient, autonomous, voice-enabled, documented.

---

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|----------|
| Name | ARIA | Autonomous Responsive Intelligent Agent |
| Personality | Friday (MCU) | Cute but bold, caring, loyal, witty |
| Voice | Piper + Coqui XTTS v2 | Local TTS, no cloud, emotion-adaptive |
| Host OS | Windows 10 | Owner's existing hardware |
| LLM | Ollama (native Windows) | Direct CUDA, no WSL overhead |
| Containers | Docker Desktop (WSL2) | Linux containers on Windows |
| Primary Model | Llama 3.1 70B | Best open-source reasoning |
| Fast Model | Llama 3.2 8B | Quick responses, low latency |
| Vector DB | ChromaDB | Lightweight, Python-native |
| Agents | LangGraph | State machines for complex flows |
| Automation | n8n | Visual workflows, OAuth, 400+ connectors |
| Interface | Telegram Bot | Android-native, encrypted, voice |
| Rich UI | SvelteKit PWA | Installable, offline, Material You |
| Networking | Tailscale | Zero-config WireGuard mesh |
| Search | SearXNG | Self-hosted, private, no tracking |
| IoT | Home Assistant | Open source, massive device support |
| Encryption | BitLocker + SOPS + age | Full-disk + git-friendly secrets |
| Monitoring | Uptime Kuma | Self-hosted, Telegram alerts |
| Backup | Restic | Encrypted, deduplicated, fast |

---

## Open Questions

1. **GPU:** Which GPU installed/planned? (Determines max model size)
2. **WhatsApp:** Matrix bridge vs Baileys? (Stability vs features)
3. **Starting Autonomy:** Level 0, 1, or 2 as default?
4. **Offsite Backup:** Local only or encrypted cloud (Backblaze B2)?
5. **IoT Devices:** What smart home hardware? (Brand, protocol)
6. **Finance:** Which banks/UPI apps? (SBI, HDFC, GPay, etc.)
7. **Windows Edition:** Home or Pro? (Affects BitLocker)
8. **Voice Reference:** Any specific voice sample for ARIA's TTS?
9. **Tutoring Topics:** What should ARIA teach?

---

*ARIA — Cute but bold. Caring but fierce. Unstoppable.*
*"I'm ARIA. I've got you."*
