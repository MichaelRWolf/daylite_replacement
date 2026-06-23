# Design Notes: daylite_replacement

## Problem Statement

Daylite.app (contact/CRM system) enforced hierarchical filing: you had to know up-front where to file observations, decisions, and events. When the classification scheme didn't match how you later *needed* the information, retrieval broke.

Traditional CRM/contact/calendar tools (Daylite, Apple Contacts, Apple Calendar) lock you into predefined categories. They require you to predict your own information needs at capture time.

michael-pro water damage highlighted the gap: need to capture decisions ("invest 4 hours, then stop"), observations ("motherboard has 3 white spots"), and actions, then later query across them ("show water damage status with done/todo breakdown").

## Core Insight

With ripgrep and AI, filing structure is computational waste. **Append-only event stream + semantic search is faster and more flexible.**

```
log "Decision - Invest 4 hours in restoring michael-pro, then stop"
log "Observation - michael-pro motherboard has 3 white spots on bottom"
log "Decision - remove motherboard and clean top before power-on"

query "Show water damage status including done and TODO items"
```

Don't think up-front. Capture raw. Search semantically later.

## Research: What Exists?

Searched for log-first knowledge tools, AI-powered event analysis, and minimal command interfaces.

**Finding:** No mature personal-knowledge products exist with this pattern.

- **Logseq**: Closest (append-only daily journals), but still Markdown-file-backed, not truly log-first.
- **Obsidian + plugins**: Still file-based with pre-planned hierarchy.
- **Enterprise event streams** (Kafka, etc.): Pattern exists, not in consumer tools yet.

**Steve Yegge's "beads"** (git-backed issue tracker for AI agents): Teaches key design principle: *format is the interface.* Don't design canonical UI; make the format durable and let consuming tools build interfaces.

**Implication:** This is a real gap. Build it.

## Solution: Claude Memory System as Foundation

Rather than invent new infrastructure, leverage **Claude's proven memory system** (which Michael has observed in action):

- Files: `/Users/michael/.claude/projects/-Users-michael-repos-wolf-soho/memory/`
- Index: `MEMORY.md`
- Format: YAML frontmatter + Markdown + `[[links]]` for cross-reference
- Types: Already exist (user, feedback, project, reference); extend with log types

**Command vocabulary:**

```bash
remember decision "michael-pro: Invest 4 hours, then stop"
remember observation "michael-pro motherboard: 3 white spots on bottom"
remember action "michael-pro: remove motherboard and clean top"
remember learning "Water damage risk: ear-drop near electronics"
remember question "How to safely dry MacBook Pro logic board?"
remember meeting "Called Diana about laptop status 2026-06-22"

recall "water damage status"      # Synthesize all water-damage memories
work "michael-pro"                # Show pending action items
complete "action-id"              # Mark action done
```

**Memory types:**

- `decision` — commitments, priorities, time-bound choices
- `observation` — facts, damage assessment, symptoms
- `action` / `task` — things to do
- `question` — open questions needing research
- `learning` — insights, patterns, lessons
- `meeting` — capture people, org context, decisions in one entry
- `person` — contact info, relationship context
- `organization` — org context, projects, relationships
- `insight` — pattern recognition, synthesis

## Why This Works

1. **Proven format**: Claude's memory system is already production (Michael uses it, it works).
2. **Portable**: Just `.md` files in a directory; no database, no proprietary format.
3. **Persistent across sessions**: Memories stay in the repo; they're part of project state.
4. **AI-native**: Claude reads them at session start; queries are natural language.
5. **Durable**: Plain text + Markdown + YAML; works in 20 years.
6. **Flexible linking**: `[[water-damage]]`, `[[michael-pro]]`, `[[Diana]]` — auto-link related events without pre-planning.
7. **Searchable**: ripgrep works on the directory; semantic queries via Claude API.

## Implementation Phases

### Phase 1: Prototype (this repo)

- Minimal CLI tool: `remember`, `recall`, `work`, `complete` commands
- Backed by `/Users/michael/.claude/projects/-Users-michael-repos-wolf-soho/memory/` directory
- Commands:
  - `remember <type> "<message>"` → creates `.md` file with frontmatter, adds to MEMORY.md
  - `recall "<query>"` → reads memory directory, sends to Claude API, returns synthesis
  - `work "<context>"` → lists pending actions tagged with context
  - `complete "<memory-id>"` → marks memory as done

### Phase 2: Integration (future)

- Integration with Claude Code as MCP server or shell function
- Inline queries in editor workflows
- Sync with Apple Calendar/Contacts for canonical context
- Export/archive functionality

### Phase 3: Scaling (later)

- Multi-repo memory aggregation (queries across projects)
- Time-series analysis (track projects, people, decisions over time)
- Relationship graph visualization
- Integration with email (capture decisions from mail threads)

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Use Claude memory system as foundation | Proven, durable format; already in use; no new infrastructure needed |
| Append-only JSONL for raw event stream | Fast, simple, git-friendly; one entry per line; immutable audit trail |
| AI parsing at query time, not capture time | Reduce friction; let Claude infer intent, context, relationships |
| Plain-text, not database | Portability, longevity, human-readable; ripgrep searchable |
| Extend existing types (user, feedback, project, reference) with log types | Consistency; no new architecture; memories become unifying system for all knowledge capture |
