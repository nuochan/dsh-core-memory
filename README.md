# @nuochan/dsh-core-memory

> Enhanced fork of [dsh-core-memory](https://www.npmjs.com/package/dsh-core-memory) v1.0.1 by jiuling — full ambient layer injection, ambient-aware search, and configurable memory surfaces for DeepSeek Harness.

**English TL;DR** — A [Cordis](https://github.com/deepseek-ai/deepseek-harness) plugin for DeepSeek Harness that gives every session a persistent memory: a core-memory markdown file that is **always injected into the system prompt in full** (no character cap), an archival note store searchable by seven model-facing tools with **ambient entry boosting**, and a persona (reply-style) settings page. Pure ESM JavaScript, zero build step, no database and no embedding service — the store is plain markdown on disk.

## Why this fork

The upstream `dsh-core-memory` caps the injected core memory at 3,000 characters. This is appropriate for hand-curated summaries but insufficient for setups where the core memory is **generated from an external source** (e.g., a shared knowledge base, a team rules repository, or a multi-harness memory hub) and contains a full ambient rule set that must be loaded every session.

This fork removes the hardcoded cap and adds configurability so the plugin works equally well for:

- **Hand-curated memory** (original use case) — set `DSH_CORE_MEMORY_CAP=3000` to restore upstream behavior
- **Generated ambient layers** — unlimited injection for large rule sets, policy documents, or shared knowledge bases
- **Multi-tier memory systems** — ambient entries rank higher in search, index shows ambient-first ordering

## Changes from upstream

| Change | Upstream | Fork |
|---|---|---|
| Core memory cap | Hardcoded 3,000 chars | Unlimited by default; configurable via `DSH_CORE_MEMORY_CAP` |
| Search scoring | Keyword match only | Ambient entries get +5 base boost + priority-weighted bonus |
| Index sorting | Recency only | Ambient entries first (by priority), then retrieval by recency |
| Index count | Fixed 12 | Configurable via `DSH_MEMORY_INDEX_COUNT` |
| Project filtering | Not available | `memory_search` accepts optional `project` parameter |
| Diagnostic counts | Total only | `memory_list` reports ambient/retrieval breakdown |
| Configuration | `DSH_HOME` only | Five environment variables (see below) |

See [CHANGELOG.md](./CHANGELOG.md) for the complete technical changelog.

## Features

- **Core memory always injected**: `MEMORY.md` is read synchronously at every model assembly and injected into the system prompt in full. No truncation by default. Edit by hand or generate from an external source.
- **Archival notes on demand**: one markdown file per note (`notes/<id>.md`) with flat YAML frontmatter. The injected index shows the most recent entries; older notes are retrieved via `memory_search` / `memory_read`.
- **Ambient-aware search**: entries marked `ambient: true` in their frontmatter rank higher automatically. Priority-weighted scoring respects `ambient_priority` values.
- **Seven model tools**: `memory_write`, `memory_core`, `memory_search`, `memory_read`, `memory_list`, `memory_delete`, `persona`.
- **Persona settings page**: 11 built-in reply-style presets plus custom text, managed via the DSH settings UI or the `persona` tool.
- **Zero dependencies**: pure ESM JavaScript, Node.js native `fs`, no build step, no database, no embedding service.
- **Atomic writes**: all disk writes use temp-file + rename to prevent corruption.

## Installation

```bash
dsh plugin --profile web add github:nuochan/dsh-core-memory
```

Or from npm (once published):

```bash
dsh plugin --profile web add @nuochan/dsh-core-memory
```

Restart DSH after installation.

## Storage layout

Default location: `$DSH_HOME/memory/` (`~/.dsh/memory/` when `DSH_HOME` is unset):

```
memory/
├── MEMORY.md     Core memory: injected every session, hand-editable or generated
├── notes/        Archival notes: one markdown file per entry (flat YAML frontmatter)
├── index.json    Metadata index (source of truth for list/search; rebuilt from notes/ when corrupt)
└── PERSONA.md    Reply style (managed by persona tool and settings page)
```

On first activation the plugin creates `MEMORY.md` with a starter template and an empty `index.json`.

## Configuration

| Environment variable | Description | Default |
|---|---|---|
| `DSH_HOME` | Harness home directory; memory store lives under `memory/` | `~/.dsh` |
| `DSH_CORE_MEMORY_CAP` | Core memory injection cap in characters. `0` = unlimited | `0` |
| `DSH_MEMORY_INDEX_COUNT` | Number of recent notes shown in the injected index | `12` |
| `DSH_AMBIENT_SEARCH_BOOST` | Base score bonus for ambient entries in search | `5` |
| `DSH_AMBIENT_PRIORITY_WEIGHT` | Divisor for priority-weighted search scoring (priority / N, capped at +3) | `100` |

### Restoring upstream behavior

Set `DSH_CORE_MEMORY_CAP=3000` to restore the original 3,000-character truncation.

## Usage

All interaction is through model tools — no manual file editing required (though `MEMORY.md` can be edited directly and changes take effect immediately):

- Express a long-term preference → model calls `memory_write` (archival note) or `memory_core` (core memory section)
- Need to recall a past fact → model calls `memory_search`, then `memory_read` for the full body
- Change reply style → model calls `persona`

## Compatibility

- **Node.js** ≥ 22
- **DeepSeek Harness** ≥ 0.1.x (requires `tools` and `systemPrompt` services; settings page requires `webServer`, which is optional)
- Host and browser halves are **optional dependencies**: `@deepseek-ai/cordis` is provided by Harness; `react` is only used by the settings page

## Known limitations

- Search uses **keyword scoring**, not semantic retrieval. Synonyms may not match — use multiple tags to improve recall.
- The injected index shows a configurable number of entries (default 12); older entries require explicit `memory_search`.
- Core memory has no injection cap by default. For very large generated cores, monitor context window usage and set `DSH_CORE_MEMORY_CAP` if needed.

## License

MIT
