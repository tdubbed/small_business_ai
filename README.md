# AI Platform - Local AI for Small Business

Private, local AI infrastructure for small businesses. No cloud dependency. Your data stays yours.

## Vision

Provide small businesses with powerful AI tools that run entirely on their own hardware or on a private cloud we control. Lawyers, doctors, accountants, contractors - anyone with sensitive data who needs AI but can't trust public clouds.

## Tiers

1. **Local Box** - Pre-configured mini PC, local LLM, web chat UI, document ingestion. Data never leaves the building.
2. **Private Cloud** - We host on infrastructure we control. Customer gets a URL, but it's our server, not OpenAI's.
3. **Hybrid** - Local box for daily use, private cloud for heavy lifting.

## Components (Planned)

- **Chat UI** - Web-based, mobile-friendly, dark mode. Simple enough for anyone.
- **LLM Backend** - llama.cpp / ollama serving local models (Qwen 32B, Mistral, etc.)
- **RAG Pipeline** - Upload PDFs, emails, invoices. AI searches your documents to answer questions.
- **User Management** - Admin panel, multiple users, usage tracking.
- **Home Assistant Integration** - Automation, monitoring, alerts.
- **Deployment** - Docker-based, single command setup.

## Tech Stack

- Python (Flask/FastAPI)
- llama.cpp / ollama
- SQLite / PostgreSQL
- Docker
- Tailscale (secure remote access)

## Development

Proof of concept built on a home lab that mirrors the customer deployment pattern:
- Network-attached storage
- Multi-socket workstation (LLM inference)
- GPU-accelerated development workstation

## Status

Active R&D. The home lab is the reference deployment used to validate the tiered stack before customer rollout.
