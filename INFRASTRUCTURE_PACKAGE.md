# Small Business AI - Infrastructure Package Specs

> Public overview. Specific cost, margin, and revenue projections available on request.

## The Problem We Solve

- Law firms can't run client comms through ChatGPT (privilege leakage)
- Medical offices can't use cloud AI on patient notes without a BAA (HIPAA)
- Accounting firms can't paste client financials into public LLMs
- Contractors don't want bids/specs in someone else's training data

**The pitch:** "ChatGPT-level AI that runs in your building. Your data never leaves."

---

## Three Tiers

### Tier 1: Starter — $1,000-1,200 sell price

**Customer:** Solo practitioner, 2-5 person office, 10-30 queries/day

**Hardware:**
- Mini PC: Beelink SEi12 Pro / Minisforum UM790 Pro / ASUS NUC 13 Pro
  - AMD Ryzen 9 7940HS or Intel i9-12th gen
  - 64GB DDR5 RAM
  - 2x 2TB NVMe (OS+models / documents+backups)
- No discrete GPU — CPU inference only
- Power draw: 35-65W idle, 65-120W load 

**Models it runs:**
- 7B (Qwen2.5, Mistral, Llama 3.1 8B): 8-15 tok/sec — good for interactive
- 13B: 4-8 tok/sec — acceptable for async
- 32B: too slow for interactive (1-2 tok/sec)

**Storage:** Internal only, no separate NAS. 4TB total across two drives.

**Networking:** Customer's existing router. Tailscale for remote management.


---

### Tier 2: Professional — $2,500-3,000 sell price

**Customer:** 5-20 person office, moderate usage, needs faster responses

**Hardware:**
- Mini PC with PCIe: Minisforum MS-01 (i9-12900H/13900H)
  - Or refurbished HP EliteDesk 800 G9 / Dell OptiPlex 7010 SFF
  - 64GB RAM minimum
  - 1TB NVMe (OS+models) + 2TB NVMe (data)
- **GPU: Used RTX 3060 12GB** — the tier-defining component
  - 12GB VRAM fits 7B models fully in VRAM
  - Partial offload on 13B models
- Optional: 2-bay NAS (Synology DS223 ~$300 + 2x 4TB drives ~$200)
- Power draw: 150-250W load 

**Models it runs (RTX 3060 12GB):**
- 7B: 50-80 tok/sec — feels instant
- 13B: 20-35 tok/sec — great for interactive
- 32B (Q4): 5-10 tok/sec hybrid CPU+GPU — usable
- 70B: not practical

**Networking:** TP-Link TL-SG108E managed switch. Optional dedicated router for network isolation.


---

### Tier 3: Premium — $5,000-6,500 sell price

**Customer:** 20-50 person firm, high usage, multiple concurrent users

**Hardware:**
- Workstation: Dell Precision 3660 / HP Z2 G9 Tower (refurb or new)
  - Or custom build: mid-ATX, i9-13th/14th gen
  - **128GB DDR5 RAM**
  - 2TB NVMe (OS+models) + 4TB (data)
- **GPU: Used RTX 3090 24GB** — the killer feature
  - 24GB VRAM fits 13B fully, 32B Q4 fully in VRAM
  - 32B at 40-60 tok/sec = feels like ChatGPT
- 4-bay NAS: Synology DS423+ (~$500-600 w/ 4x 4TB drives = 12TB SHR)
- Power draw: 300-500W load 

**Models it runs (RTX 3090 24GB):**
- 7B: 80-120 tok/sec — completely instant
- 13B: 60-90 tok/sec — fully in VRAM
- 32B Q4: 40-60 tok/sec — this is the differentiator
- 70B Q4: 8-15 tok/sec hybrid — viable for background tasks
- Handles 3-5 concurrent sessions

**Networking:** Managed switch + dedicated firewall (Protectli/GL.iNet running pfSense ~$150-200). VLAN isolation.


---

## Software Stack (All Tiers)

Same stack, scales with hardware.

| Component | Tool | Purpose |
|-----------|------|---------|
| LLM Serving | Ollama | Model management, REST API, GPU auto-detection |
| RAG + Web UI | Anything LLM | Chat interface, document ingestion, vector DB, multi-user |
| Embeddings | nomic-embed-text | Document vectorization for RAG |
| Vector DB | LanceDB/ChromaDB | Embedded in Anything LLM |
| Remote Access | Tailscale | Zero-config VPN, SSH from anywhere |
| Monitoring | Uptime Kuma / Netdata | Alerts if anything goes down |
| Container Mgmt | Docker + Portainer | Service management (we see, customer doesn't) |
| Backup | Restic | Nightly to NAS (Tier 2/3) or second drive (Tier 1) |
| OS | Ubuntu Server 24.04 LTS | 5-year support |

### Security Hardening
- UFW firewall (LAN + Tailscale only)
- Fail2ban on SSH
- All services localhost/LAN-bound (never internet-exposed)
- SSL via Tailscale HTTPS (free)

---

## What the Customer Sees

1. A web address on their network: `http://ai.firmname.local`
2. Login screen (username/password)
3. ChatGPT-like chat interface
4. Their documents already loaded in a workspace
5. They type questions, get answers with citations to their own docs
6. That's it

**What they do NOT see:** Docker, Ollama, terminal, file system, Tailscale

---

## Monthly Recurring Revenue

| Plan | Level | Includes |
|------|-------|----------|
| Basic Monitoring | Entry | Uptime monitoring, auto-alerts, quarterly check-in |
| Standard Managed | Middle | + monthly model updates, 2hr support, usage reports |
| Full Managed | Top | + unlimited support, workflow setup, priority response |

Pricing scales with tier and usage; contact for current rates.

---

## Storage Sizing (Smaller Than You Think)

| What | Size |
|------|------|
| LLM weights (7B Q4) | 4-5 GB |
| LLM weights (32B Q4) | 18-20 GB |
| Embedding model | 300 MB |
| 10,000 business documents | 5-20 GB |
| Vector embeddings for 10K docs | 500 MB - 2 GB |
| Conversation logs (1 year) | 500 MB - 1 GB |
| OS + Docker + services | 20-30 GB |
| **Total typical deployment** | **50-80 GB** |

A 2TB drive is massive overkill. That's fine — customers feel better.

---

## Deployment Process

### Ship & Remote Setup (scalable model)
1. Pre-configure at home: Ubuntu, Docker, Ollama, all services, Tailscale
2. Test everything
3. Ship to customer (well-padded)
4. Customer plugs in: power + ethernet (two cables)
5. SSH in via Tailscale, final config
6. 30-minute onboarding call
7. Done

**Time per deployment:** 2-3 hours (once you have a base image)

### On-Site (Tier 3 / white-glove)
- Flat fee on top of hardware
- 2-4 hours on site

### Base Image
- Ansible playbook or bash setup script
- Parameterized per customer
- Configs in private git repo
- Rebuild from scratch: under 2 hours

---

## Risks

- **Hardware failure:** Keep a spare pre-configured mini PC. 48hr replacement SLA. NAS protects documents.
- **Customer expectations:** They compare everything to ChatGPT-4. Set expectations by tier.
- **Support creep:** "Can you fix my printer?" — define scope in service agreement.
- **Model licensing:** Llama, Mistral, Qwen all allow commercial use at our scale. Check model cards.

---

## Launch Strategy

1. **First customer:** Starter tier with introductory pricing to earn a reference deployment and case study.
2. **Target:** 3-5 person law firm or medical practice (strongest compliance need, talk to each other)
3. **Pre-load:** qwen2.5:7b, llama3.1:8b, nomic-embed-text
4. **Second customer and beyond:** Full rate. Referrals from first customer.
5. **Scale path:** Bar association events, medical association presentations, local professional networks

---

## Quick Reference

| | Starter | Professional | Premium |
|---|---------|-------------|---------|
| Sell price | $1,000-1,200 | $2,500-3,000 | $5,000-6,500 |
| Best model | 7B fast / 13B slow | 13B fast / 32B usable | 32B fast / 70B usable |
| GPU | None (CPU) | RTX 3060 12GB | RTX 3090 24GB |
| RAM | 64GB | 64-128GB | 128GB |
