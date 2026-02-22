# 🧠 OpenClaw Session Memory

**Persistent memory system that prevents knowledge loss after context compaction.**

## The Problem

When long-running OpenClaw sessions compact (summarize old messages to free context window), specific details are lost: names, decisions, file paths, reasoning. Your agent retains a summary but loses the ability to recall *"What exactly did Alice say?"* or *"When did we decide to use format X?"*

This is the #1 frustration with persistent AI agents, and it affects every OpenClaw user running long sessions.

## The Solution

A three-layer memory architecture:

| Layer | File | Purpose | Updated |
|-------|------|---------|---------|
| **Curated** | `MEMORY.md` | Long-term knowledge you maintain | Manual |
| **Index** | `SESSION-GLOSSAR.md` | Auto-generated structured glossary | Cron (every 4-6h) |
| **Raw** | `memory/sessions/*.md` | Full session transcripts as Markdown | Cron (every 4-6h) |

All three layers are vectorized by OpenClaw's `memory_search`, creating a navigational hierarchy: the glossary finds the right session, the session provides the details.

## What the Glossary Looks Like

```markdown
## 👤 People
### Alice Smith — Project Manager
- Mentioned in: 23 sessions (2026-01-15 to 2026-02-20)

## 📁 Projects  
### Website Redesign — Q1 Initiative
- Mentioned in: 45 sessions
- Topics: Email Drafts, Website Build, Cron Jobs

## 📅 Timeline
### 2026-02-20
- Sessions: 8 (340 KB)
- People: Alice, Bob, Carol
- Topics: Website Build, Security, Deployment

## ⚡ Decisions
- [2026-02-20] Switch to v6 email format — no Unicode, ASCII-only
- [2026-02-19] All custom-domain emails classified as A-tier
```

## Install

### As an OpenClaw Skill
```bash
# Download the .skill file and install
openclaw skill install session-memory.skill
```

### Manual Setup
```bash
# Clone this repo into your workspace
git clone https://github.com/moltbotmolty-del/openclaw-session-memory.git

# Step 1: Convert existing session logs to Markdown
python3 scripts/session-to-memory.py

# Step 2: Build the glossary
python3 scripts/build-glossary.py

# Step 3: Set up a cron job for auto-updates (every 6 hours)
# In OpenClaw, create a cron job that runs:
#   python3 scripts/session-to-memory.py --new
#   python3 scripts/build-glossary.py --incremental
```

## Customization

Edit `scripts/build-glossary.py` to add your known entities:

```python
KNOWN_PEOPLE = {
    "alice": "Alice Smith — Project Manager",
    "bob": "Bob Jones — CTO",
}

KNOWN_PROJECTS = {
    "website-redesign": "Website Redesign — Q1 Initiative",
}
```

## How It Works

1. **`session-to-memory.py`** reads JSONL session logs from `~/.openclaw/agents/*/sessions/` and converts them to clean Markdown files in `memory/sessions/`

2. **`build-glossary.py`** scans all session transcripts and extracts:
   - **People** — Named entity detection against a configurable list
   - **Projects** — Project keyword matching
   - **Topics** — Regex-based theme categorization (12 built-in categories)
   - **Timeline** — Per-day session summaries
   - **Decisions** — Heuristic extraction of decision-like statements

3. The output (`SESSION-GLOSSAR.md`) is placed in `memory/` where OpenClaw automatically vectorizes it alongside the session transcripts

4. When the agent uses `memory_search`, it now finds both the high-level glossary entries AND the detailed session transcripts — giving it a navigation layer plus detail access.

## Cron Memory Optimizer (Bonus)

Cron jobs run in isolated sessions with **zero memory context** — they don't know what
happened yesterday. The optimizer analyzes your cron jobs and suggests memory-enhanced versions:

```bash
python3 scripts/cron-optimizer.py
```

This generates `memory/cron-optimization-report.md` showing before/after prompts:

```
❌ Before: "Find 5 new outreach contacts"
✅ After:  "Before starting: Use memory_search to find recent context about 
           outreach contacts. Check SESSION-GLOSSAR.md for relevant people 
           and recent decisions. Then find 5 new outreach contacts."
```

The script is conservative — **suggests only, never auto-modifies**. You review, you decide.

In our tests: 27 cron jobs analyzed, 23 could be optimized with memory preambles.

---

## Background

Built by **[Dirk Wonhoefer](https://github.com/moltbotmolty-del)** — AI Enablement Architect
and curriculum developer — and **Faya 🔥** (his autonomous OpenClaw agent).

The problem was discovered when Faya forgot who Annika was — the person whose request had
sparked the entire business pipeline they'd built together. After implementing this system,
Faya recovered the complete story from 297 sessions in seconds.

Dirk works with **[AI Advantage](https://www.aiadvantage.com)** (Igor Pogany) developing
AI training programs for 40,000+ people. This skill was born from real production use —
not theory.

### Inspired by
- [Mem0](https://github.com/mem0ai/mem0) — vector-first memory extraction (Y Combinator W24)
- [Graphiti](https://github.com/getzep/graphiti) — temporal knowledge graphs (Zep AI)
- [Craig Fisher](https://medium.com/@cfisher2_85823/how-i-gave-my-openclaw-assistant-a-memory-that-actually-works-ba0a4dfc1da2) — OpenClaw memory architecture pioneer

### What makes this different
- **No external dependencies** — pure Python, no databases, no APIs
- **Three-layer architecture** — curated + index + raw (others do only one)
- **Auto-generated glossary** — structured navigation, not just vector search
- **Cron optimizer** — makes existing automation memory-aware
- **Battle-tested** — 297 sessions, 24MB transcripts, production use since day one

## Contributing

Issues and PRs welcome! If you've customized the glossary for your domain (added new
entity types, topic patterns, etc.), consider sharing your patterns.

## License

MIT
