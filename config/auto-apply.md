# Auto-Apply Configuration

This file defines which categories Lucid is allowed to auto-apply without human review.

Only changes with **HIGH confidence** are ever auto-applied — regardless of category.
Medium and low confidence always require human approval.

When sectioned memory is enabled, Lucid should apply safe edits directly to the relevant `memory/sections/*.md` file(s) and then update the matching `Last Updated` value in `memory/index.md`.
If `memory/sections/` does not exist yet, fall back to editing `MEMORY.md`.

## ✅ Enabled Categories

Edit this list to match your project's needs.

### Safe (factual, no opinions)
- **Version numbers** — skill/plugin tables clearly outdated
- **Stale Cron IDs** — cron was explicitly removed/replaced in notes
- **Service/port deletions** — clearly documented as removed
- **New project entries** — complete details (URL, repo, path, service) mentioned on 2+ days
- **Infrastructure facts** — new cron IDs, script paths, service names, port assignments
- **Lessons Learned** — purely factual technical lessons (not preferences or opinions)
- **Model Strategy** — agent counts, new agent entries when clearly documented with model + alias
- **Closed Open Loops** — remove resolved items when explicit closure signal exists on 2+ days
- **Stale project status** — update "planning"/"in progress" to "live" when URL + service confirmed on 2+ days
- **Factual contradictions** — objective memory-vs-note conflicts where newer daily note evidence clearly supersedes the old value

## ❌ Never Auto-Apply (hardcoded)

These are always manual regardless of confidence:

- Belief Updates (opinions, model preferences, strategy changes)
- Key Decisions
- Family/personal facts
- User preferences or communication style
- Judgment contradictions
- Anything you are uncertain about
- Anything with medium or low confidence

## Target Files

When a safe change is accepted, prefer these targets:
- `memory/sections/identity.md`
- `memory/sections/operations.md`
- `memory/sections/projects.md`
- `memory/sections/infrastructure.md`
- `memory/sections/skills.md`
- `memory/sections/decisions.md`
- `memory/sections/lessons.md`
- `memory/sections/models.md`
- `memory/index.md` (timestamp updates only)

Fallback target when sectioned memory is unavailable:
- `MEMORY.md`

## Customization Examples

**Minimal setup** (only version numbers + new projects):
Remove all categories except "Version numbers" and "New project entries".

**Aggressive setup** (trust the AI more):
Add "Open Loops closure", "Stale project status", and "Factual contradictions" — but watch for false positives.

**Conservative setup** (human reviews everything):
Remove all categories. Lucid will still generate suggestions but never apply them.
