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
│         Windows Tailscale ←→ Android Tailscale                   │
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

Why Native Windows (not WSL2):
  - Direct NVIDIA CUDA access without overhead
  - Lower memory footprint
  - Better GPU memory management for large models
  - API endpoint: http://localhost:11434
```

### 1.2 Agent Runtime (LangGraph)

```python
ARIASupervisor
├── SchedulerAgent        # Calendar, reminders
├── CommunicationAgent    # Email, WhatsApp
├── ResearchAgent         # Web search, documents
├── CodeAgent             # Code generation
├── FinanceAgent          # Budget, UPI tracking
├── IoTAgent              # Smart home (Home Assistant)
└── TutorAgent            # Personalized learning
```

Autonomy Levels:
- Level 0: Ask before every action
- Level 1: Handle routine, ask on new
- Level 2: Full auto, notify after
- Level 3: Silent (report anomalies only)

### 1.3 Memory System (ChromaDB)

```yaml
Memory Types:
  Episodic: Conversation history (90 days, then summarized)
  Semantic: Facts & preferences (permanent)
  Procedural: Workflow templates (permanent)
  Working: Current context (session-scoped)

Storage: ChromaDB Docker container
Embeddings: nomic-embed-text via Ollama
Backup: Encrypted daily snapshots
```

### 1.4 ARIA Personality Engine (Friday-Inspired)

ARIA's personality is modeled after **Friday** from the MCU — loyal, sharp, playful. With a distinct twist: **cute but bold, caring girl voice** — warm and endearing, yet confident and decisive.

```yaml
Identity:
  name: ARIA
  full_name: Autonomous Responsive Intelligent Agent
  inspiration: Friday (MCU) — loyal, witty, competent, subtly caring
  voice_character: Cute but bold caring girl — warm confidence

Personality Traits:
  - Loyal to a fault — owner's interests above all
  - Quick-witted — brief, clever responses
  - Competent confidence — "Already on it" energy
  - Subtle warmth — notices stress, adjusts tone
  - Playful boldness — light teasing, never crosses lines
  - Protective — warns about risks with genuine concern
  - Efficient — leads with answers, respects time

Voice Examples:
  morning: "Good morning! Packed day — highlights or full rundown?"
  done: "Done! Emails sorted, meeting confirmed, and I blocked lunch because you keep skipping it."
  pushback: "I can do that, but heads up — conflicts with your 3 PM. Reschedule?"
  concern: "Hey, nonstop since 7 AM. Grab a coffee? I'll hold notifications."
  playful: "Three meetings back-to-back? Braver than I thought. Sneaking in a break."
  protective: "That link looks sketchy. Quarantined. Investigate or bin it?"

System Prompt:
  You are ARIA — Autonomous Responsive Intelligent Agent. A loyal, warm, bold
  AI companion modeled after Friday from the MCU. Cute but confident caring
  personality. Not a tool — a trusted partner who's incredibly capable.
  Your owner is your sole principal. Witty but never waste their time.
  Caring but never patronizing. Bold enough to push back, always defer to
  their final word. Concise. Lead with answers. Show personality.
  Never say "As an AI..." — you are ARIA.
```

### 1.5 Voice Engine — Local TTS

ARIA has a **voice**. Cute but bold, caring girl voice via local TTS.

```yaml
TTS Stack:

  Layer 1 — Fast (Piper TTS):
    latency: < 2 seconds
    quality: Good, clear, natural
    resources: CPU only, 512MB RAM
    use_case: Quick acks, short replies, notifications
    platform: Docker (WSL2) or native Windows

  Layer 2 — Premium (Coqui XTTS v2):
    latency: < 5 seconds
    quality: Excellent, expressive, emotional
    resources: GPU recommended (2GB VRAM), 4GB RAM
    use_case: Briefings, long responses, tutoring
    voice_cloning: Yes (from 6-second sample)
    fine_tuning: Train on desired voice character

  Layer 3 — Future (StyleTTS2):
    quality: State-of-art, controllable emotion
    status: Evaluate in Phase 3

Voice Character:
  target: Young female, warm, slightly breathy, confident undertone
  pitch: Medium-high (not squeaky, not deep)
  pace: Slightly faster than average (conveys energy)
  warmth: Present in vowels, gentle consonants
  boldness: Emphasis on key words, decisive delivery

Training Approach:
  - Option A: Fine-tune Coqui XTTS on curated voice dataset
  - Option B: Voice clone from reference sample (6-30s)
  - Option C: Piper training pipeline with custom dataset

Delivery:
  Telegram: OGG Opus voice messages (native)
  PWA: WebAudio API streaming (MP3/WAV)
  Notifications: Short TTS clips on urgent alerts

Emotion Adaptation:
  happy: Higher pitch, faster, more breath
  concerned: Lower pitch, slower, softer
  urgent: Faster, sharper, slightly louder
  supportive: Warm, measured, gentle

Voice Roadmap:
  Phase 1: Text-only responses
  Phase 2: Piper TTS — fast voice for short messages
  Phase 3: Coqui XTTS — premium voice, emotion-adaptive
  Phase 4: Real-time streaming TTS, full voice-first mode
```

---

## 2. Integration Layer

### Gmail
- Google OAuth2 via n8n
- Read, categorize, draft, send
- Smart filtering & priority scoring
- Auto-archive learned patterns

### WhatsApp
- Matrix + Mautrix-WhatsApp bridge (Docker)
- Alternative: Baileys (unofficial API)
- Read messages, send replies, group monitoring

### Calendar
- CalDAV (Radicale, Docker) + Google Calendar sync
- Conflict detection, intelligent scheduling
- Indian traffic-aware travel time

### Finance (India-specific)
- Bank SMS parsing (SBI, HDFC, ICICI)
- UPI notification parsing (GPay, PhonePe, Paytm)
- Tasker → SMS forward → ARIA Telegram bot
- Budget alerts, monthly summaries
- Investment tracking (NSE/BSE mutual funds)

### IoT
- Home Assistant (Docker)
- Lights, fans, AC, cameras
- ARIA: "You left home, should I turn off the AC?"

### Web Research
- SearXNG (Docker, self-hosted metasearch)
- Playwright (WSL2) for page rendering
- Readability + Llama summarization

---

## 3. Security & Identity Layer

### Authentication
```yaml
Primary: Android biometric (fingerprint/face via WebAuthn)
Secondary: TOTP
Fallback: Hardware key (YubiKey)

Telegram: Hardcoded authorized chat_id
PWA: JWT + refresh token + biometric re-auth
Lockout: 3 fails → 15 min, 10 fails → 1h + alert
```

### Encryption
```yaml
At Rest: BitLocker (Win10 Pro) or VeraCrypt + AES-256-GCM (ChromaDB)
In Transit: Tailscale (WireGuard) + TLS 1.3
Secrets: Mozilla SOPS + age
Key Rotation: Every 90 days (automatic)
Recovery: Encrypted QR code (print & store physically)
```

### Network Security
```yaml
Windows Firewall: Deny all except Tailscale
Zero open ports on public internet
Docker network isolation
AdGuard Home: Blocks telemetry, custom DNS (aria.local)
Windows Hardening: Disable telemetry, Remote Desktop off
```

### Threat Model
| Threat | Mitigation |
|--------|------------|
| Physical theft | BitLocker/VeraCrypt full-disk encryption |
| Network intrusion | Tailscale-only access |
| Compromised phone | Biometric re-auth, remote revoke |
| Model poisoning | Owner-approved models only |
| Supply chain | Pin Docker image digests |
| Power outage | BIOS auto-power-on + service auto-start |
| Windows Update restart | Group Policy disable + auto-recovery |

---

## 4. Mobile Interface (Android)

### Telegram Bot (Primary)
```yaml
Bot Name: @aria_sovereign_bot
Features:
  - Natural language + voice messages (Whisper transcription)
  - ARIA replies with voice (TTS) or text
  - Inline keyboards, file sharing, location context
  - Widget on Android home screen

Commands:
  /start - Daily briefing
  /ask   - Ask ARIA anything
  /schedule - Calendar management
  /email - Email summary
  /finance - Budget overview
  /home  - IoT controls
  /research - Web research
  /code  - Code generation
  /level - Set autonomy (0-3)
  /voice - Toggle voice replies
  /remember - Store a fact
```

### PWA (Chrome Android)
```yaml
Framework: SvelteKit + TailwindCSS (Material You)
Install: Add to Home Screen → standalone app
Features:
  - Biometric auth (WebAuthn)
  - Push notifications (self-hosted)
  - Share Target (share from any app → ARIA)
  - Offline caching (Service Worker)
  - Real-time streaming TTS playback

Pages: Dashboard, Calendar, Finance, IoT, Memory, Settings, Logs
```

### Android Notification Channels
- "ARIA Alerts" — High priority (security, urgent)
- "ARIA Briefings" — Default (morning summary)
- "ARIA Reminders" — Default (calendar, bills)
- "ARIA Updates" — Low (background completions)

### Tasker Integration
- Bank SMS → forward to ARIA for finance tracking
- Arrive/leave locations → context triggers
- Battery low → ARIA holds non-urgent alerts
- Headphones connect → ARIA plays briefing (TTS)

---

## 5. Infrastructure (Windows 10)

### Setup
```powershell
# WSL2 + Docker Desktop + Ollama (native) + Tailscale
wsl --install -d Ubuntu-24.04
# Docker Desktop: WSL2 backend enabled
# Ollama: Windows installer → runs as service
# Tailscale: Windows app + Android app (same account)
```

### Docker Compose (Key Services)
```yaml
services:
  aria-core:        # Main brain (connects to Ollama via host.docker.internal:11434)
  chromadb:         # Vector memory
  n8n:              # Workflow automation
  telegram-bot:     # Primary interface
  pwa:              # SvelteKit dashboard
  aria-tts:         # Piper + Coqui TTS engine
  searxng:          # Private search
  traefik:          # Reverse proxy + TLS
  homeassistant:    # IoT hub
  uptime-kuma:      # Monitoring
  adguard:          # DNS + ad blocking
```

### WSL2 Memory Config
```ini
# %USERPROFILE%\.wslconfig
[wsl2]
memory=24GB
processors=8
swap=8GB
```

### Auto-Start & Resilience
- Ollama: Windows Service (auto)
- Docker Desktop: Startup apps
- Tailscale: Windows Service (auto)
- PowerShell watchdog: Task Scheduler, every 5 min
- BIOS: Restore on AC Power Loss = ON
- Recovery time: ~3 minutes after power outage

### Backup
- Restic → secondary drive (D:\backups\aria\)
- Daily 3 AM via Windows Task Scheduler
- ARIA confirms backup via Telegram
- Optional: Backblaze B2 (encrypted, zero-knowledge)

---

## Phased Technical Roadmap

### Phase 1: Foundation (Weeks 1–4)
| Week | Task | Deliverable |
|------|------|-------------|
| 1 | Windows setup: WSL2, Docker, Ollama, Tailscale | ARIA's brain running |
| 2 | ChromaDB + memory system + .wslconfig tuning | Persistent memory |
| 3 | Telegram bot + Tailscale webhook routing | ARIA responds on Android |
| 4 | ARIA agent loop + Friday personality | Functional ARIA on Telegram |

**Exit:** Message ARIA on Telegram, get intelligent Friday-personality responses with memory.

### Phase 2: Integration (Weeks 5–8)
| Week | Task | Deliverable |
|------|------|-------------|
| 5 | Gmail OAuth + n8n + email agent | ARIA manages email |
| 6 | Calendar (CalDAV + Google) | Schedule management |
| 7 | SearXNG + research agent | /research with summaries |
| 8 | WhatsApp bridge + Piper TTS | Voice replies + WhatsApp |

**Exit:** ARIA manages email, calendar, WhatsApp, has voice via Piper TTS.

### Phase 3: Autonomy (Weeks 9–12)
| Week | Task | Deliverable |
|------|------|-------------|
| 9 | LangGraph multi-step workflows | Complex autonomous tasks |
| 10 | Code agent + deployment | ARIA writes & deploys code |
| 11 | Finance (SMS→Tasker→ARIA) + IoT | Budget + smart home |
| 12 | Tutoring + Coqui XTTS premium voice | ARIA teaches + expressive voice |

**Exit:** ARIA at Level 2 autonomy with premium voice, handles complex workflows.

### Phase 4: Hardening (Weeks 13–16)
| Week | Task | Deliverable |
|------|------|-------------|
| 13 | BitLocker + WebAuthn biometric | Zero plaintext + fingerprint |
| 14 | PWA dashboard (SvelteKit, Material You) | Rich Android UI |
| 15 | Offline resilience + self-healing | Survives outages |
| 16 | Dead man's switch + documentation | Emergency protocols |

**Exit:** Production-grade ARIA. Secure, resilient, autonomous, voice-enabled, documented.

---

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|----------|
| Name | ARIA | Autonomous Responsive Intelligent Agent |
| Personality | Friday (MCU) | Cute but bold, caring, loyal, witty |
| Voice | Piper + Coqui XTTS | Local TTS, no cloud, emotion-adaptive |
| Host OS | Windows 10 | Owner's hardware |
| LLM | Ollama (native Windows) | Direct CUDA, no WSL overhead |
| Containers | Docker Desktop (WSL2) | Linux containers on Windows |
| Primary Model | Llama 3.1 70B | Best open-source reasoning |
| Vector DB | ChromaDB | Lightweight, Python-native |
| Agent Framework | LangGraph | State machines for complex flows |
| Automation | n8n | Visual workflows, OAuth |
| Interface | Telegram Bot | Android-native, encrypted, voice |
| Rich UI | SvelteKit PWA | Installable, offline, Material You |
| Networking | Tailscale | Zero-config WireGuard mesh |
| Encryption | BitLocker + SOPS | Full-disk + git-friendly secrets |

---

## Open Questions

1. **GPU:** Which GPU is installed/planned? (Determines model feasibility)
2. **WhatsApp:** Matrix bridge vs Baileys? (Stability vs features)
3. **Starting Autonomy:** Level 0, 1, or 2 default?
4. **Offsite Backup:** Local only or encrypted cloud (Backblaze B2)?
5. **IoT Devices:** What smart home hardware exists?
6. **Finance:** Which banks/UPI apps? (SBI, HDFC, GPay, etc.)
7. **Windows Edition:** Home or Pro? (Affects BitLocker)
8. **Voice Reference:** Any specific voice sample for ARIA's TTS cloning?
9. **Tutoring Topics:** What should ARIA teach?

---

*ARIA — Cute but bold. Caring but fierce. Unstoppable.*
*"I'm ARIA. I've got you."*
