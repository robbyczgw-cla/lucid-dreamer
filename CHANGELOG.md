# Changelog

All notable changes to Lucid will be documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

## [0.1.0] — 2026-03-16

### Added
- Initial release of Lucid V1
- Nightly review prompt with 6 analysis categories:
  - Candidate Updates, Open Loops, Blockers, Belief Updates, Stale Facts, Duplicate/Merge Suggestions
- Suggestion ledger (`state.json`) with status tracking (pending/accepted/rejected/deferred)
- Health sentinel (`.last-success` timestamp)
- Anti-circular reasoning rule (never reads own previous reviews)
- Decision policy with 2-day threshold, confidence labels, output cap
- Negative rules (no credentials, tokens, volatile URLs in memory)
- Memory class awareness (personal facts vs project state vs policies)
- Example review output and state ledger
- Full architecture documentation

### Design process
- Concept reviewed by 4 AI agents across 2 rounds:
  - Round 1: Codex (GPT-5.3) + Claude Code (Sonnet 4.6) — pragmatic + architectural feedback
  - Round 2: Codex (GPT-5.4) + Claude Code (Opus 4.6) — deeper failure mode analysis
- Key insights incorporated:
  - State ledger for idempotency (GPT-5.4)
  - Anti-circular reasoning (Opus)
  - Memory classes (GPT-5.4)
  - "Curator, not journalist" framing (GPT-5.4)
  - Drop morning send from V1 (both)
