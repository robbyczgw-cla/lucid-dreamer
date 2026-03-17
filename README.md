# Lucid 🧠

**Background memory reasoning for AI agents.** Nightly review, belief tracking, open loop detection, and suggestion ledger — zero dependencies, pure Markdown + JSON.

> *Your AI sleeps. Lucid dreams.*

## What it does

Lucid runs while you sleep. It reads your agent's daily notes, cross-references long-term memory, and generates a structured review with:

- **Candidate Updates** — Facts that should be in long-term memory but aren't
- **Open Loops** — Todos and decisions that were never closed
- **Blockers** — Recurring friction across multiple days
- **Belief Updates** — Intentional course corrections tracked over time
- **Stale Facts** — Memory entries that newer notes clearly supersede
- **Duplicate/Merge Suggestions** — Overlapping entries to consolidate

Every suggestion includes confidence labels, source citations, and is tracked in a state ledger to prevent duplicates.

## How it works

```
Daily Notes (7 days)     MEMORY.md + USER.md
         \                    /
          \                  /
       ┌──────────────────────┐
       │   Nightly LLM Review │  ← Sonnet/Haiku, ~$0.20/night
       │   (Cron, 3:00 AM)    │
       └──────────────────────┘
                  │
          ┌───────┴───────┐
          │               │
   review/YYYY-MM-DD.md   state.json
   (Human-readable)       (Machine-readable)
          │
          ▼
   Human approves/rejects
          │
          ▼
   MEMORY.md updated
```

## Design Principles

- **Zero dependencies** — No vector DB, no SQLite, no embeddings for the core loop
- **Conservative by default** — False negatives > false positives. Curator, not journalist
- **Human-in-the-loop** — Nothing auto-applied in V1. Suggestions require approval
- **Anti-circular** — Never reads its own previous reviews (prevents self-reinforcing patterns)
- **Idempotent** — State ledger tracks suggestions by ID + status. Rejected = gone. Accepted = done
- **Memory classes** — Family facts need HIGH confidence. Project state can change with MEDIUM

## Suggestion Ledger

`state.json` tracks every suggestion with:

```json
{
  "id": "short-slug-id",
  "firstSeen": "2026-03-16",
  "lastSeen": "2026-03-17",
  "status": "pending|accepted|rejected|deferred",
  "section": "candidate-updates|open-loops|blockers|...",
  "summary": "one-line description",
  "confidence": "high|medium|low"
}
```

Status lifecycle:
- `pending` → New suggestion, awaiting review
- `accepted` → Applied to MEMORY.md
- `rejected` → Won't resurface
- `deferred` → Resurfaces after 14 days

## Decision Policy

Hard rules baked into the review prompt:

1. Only suggest additions if mentioned on **2+ separate days** OR explicitly marked as decision/fact
2. Only flag stale if newer note **clearly supersedes** the old entry
3. Only create open loop if **unresolved action** exists with no closure signal
4. **Never** propose adding credentials, tokens, API keys, or ephemeral debugging details
5. Source every claim with file path and approximate line reference
6. Hard output cap: ~50 lines per section

## Negative Rules

The reviewer is explicitly told to **never** promote:
- Credentials, tokens, API keys, passwords
- Volatile URLs, temporary port numbers
- Debugging details, session IDs
- Ephemeral container/server info

## Inspiration

Built on ideas from:
- **[Honcho.dev](https://honcho.dev)** — "Dreaming Agent" concept (background reasoning on stored history)
- **[Gigabrain](https://github.com/legendaryvibecoder/gigabrain)** — World Model (entities, beliefs, episodes), suggestion ledger
- **[Nuggets](https://github.com/NeoVertex1/nuggets)** — Promotion concept (frequently recalled → permanent memory)

Our approach: take the **best concepts**, implement as pure Cron + Markdown with zero infrastructure.

## Architecture

```
lucid/
├── SKILL.md              # OpenClaw skill definition
├── prompts/
│   └── nightly-review.md # The full review prompt
├── examples/
│   ├── review-sample.md  # Example output
│   └── state-sample.json # Example ledger
└── README.md
```

## Roadmap

- [x] **V1** — Nightly review cron, state ledger, human approval flow
- [ ] **V1.5** — Morning Context Send (≤5 actionable lines to Telegram)
- [ ] **V1.5** — Auto-apply for high-confidence, low-risk suggestions
- [ ] **V2** — Embedding-based dedup (using existing Gemini embeddings)
- [ ] **V2** — Promotion concept (N-times referenced → auto-suggest for MEMORY.md)
- [ ] **V2** — Weekly consolidation of insights
- [ ] **V3** — ClawHub skill publication (configurable taxonomy, output destinations)

## Requirements

- **OpenClaw** with cron support (`agentTurn` jobs)
- **Any LLM** that can read files and write markdown (Sonnet recommended, Haiku works for budget)
- **Markdown-based memory** — `MEMORY.md` + `memory/YYYY-MM-DD.md` daily notes (OpenClaw default)

### What Lucid does NOT need

- ❌ No vector database
- ❌ No embeddings (OpenClaw's built-in `memory_search` with embeddings is separate and independent — Lucid doesn't use it)
- ❌ No SQLite or any database
- ❌ No external APIs beyond the LLM call
- ❌ No Python, no Node packages, no runtime dependencies

Lucid is purely a **prompt + cron + markdown** system. If you have a cron job that can call an LLM and read/write files, you can run Lucid.

## Usage

### Quick Start (OpenClaw)

1. **Create the review directory:**
```bash
mkdir -p memory/review
```

2. **Add the cron job:**
```bash
openclaw cron add \
  --name "lucid" \
  --cron "0 3 * * *" \
  --tz "Europe/Vienna" \
  --model "anthropic/claude-sonnet-4-6" \
  --announce \
  --session isolated \
  --timeout-seconds 120 \
  --message "$(cat prompts/nightly-review.md)"
```

3. **Wait for the first review** (or trigger manually):
```bash
openclaw cron run <job-id>
```

4. **Review the output:**
```bash
cat memory/review/YYYY-MM-DD.md
```

5. **Approve or reject suggestions** — Tell your agent which suggestions to accept. It updates `MEMORY.md` and sets the status in `memory/review/state.json`.

### What you need in your workspace

```
your-workspace/
├── MEMORY.md                    # Long-term curated memory (Lucid suggests changes)
├── USER.md                      # User profile (Lucid reads for context)
├── memory/
│   ├── 2026-03-15.md            # Daily notes (Lucid reads last 7 days)
│   ├── 2026-03-16.md
│   ├── 2026-03-17.md
│   └── review/                  # Lucid output directory
│       ├── 2026-03-17.md        # Nightly review (human-readable)
│       ├── state.json           # Suggestion ledger (machine-readable)
│       └── .last-success        # Health sentinel (ISO timestamp)
```

### Without OpenClaw

Lucid is just a prompt. You can run it with any LLM that supports:
- Reading files from a workspace
- Writing files to a workspace
- Being triggered on a schedule

Copy the prompt from `prompts/nightly-review.md`, feed it to your LLM with access to your memory files, and save the output. The prompt handles everything — review format, decision policy, state ledger updates.

## License

MIT

---

*Lucid was designed by reviewing 4 AI agents (Codex GPT-5.3, Codex GPT-5.4, Claude Code Sonnet 4.6, Claude Code Opus 4.6) and cherry-picking the best ideas from each review round.*
