# Changelog

## 2.0.0 (2026-10-18) — Enhanced fork for full ambient injection and configurable surfaces

**Forked from:** `dsh-core-memory` v1.0.1 by jiuling ([npm](https://www.npmjs.com/package/dsh-core-memory))
**Fork motivation:** The upstream 3,000-character core memory cap prevents full ambient layer injection for setups that generate core memory from external sources (shared knowledge bases, team rule repositories, multi-harness memory hubs). This fork removes the hardcoded cap and adds configurability, ambient-aware search, and diagnostic tooling.

### Breaking changes

- **Core memory cap removed.** The upstream hardcoded 3,000-character truncation is gone. MEMORY.md is now injected in full by default. Set `DSH_CORE_MEMORY_CAP=3000` to restore upstream behavior.
- **Index sorting changed.** The injected note index now sorts ambient entries first (by priority), then retrieval entries by recency. Upstream sorted all entries by recency only.
- **Package name changed.** Published as `@nuochan/dsh-core-memory` to avoid confusion with upstream.

### New features

- **Ambient search boost.** `memory_search` gives entries with `ambient: true` in their frontmatter a +5 base score boost plus a priority-weighted bonus (`ambient_priority / 100`, capped at +3). Ambient rules rank higher automatically without special query syntax.
- **Configurable index count.** The injected note index defaults to 12 entries but is configurable via `DSH_MEMORY_INDEX_COUNT`.
- **Project-aware search.** `memory_search` accepts an optional `project` parameter to filter results by project scope.
- **Diagnostic counts.** `memory_list` reports ambient and retrieval entry counts alongside the total, making it easy to audit the memory composition.
- **Centralized configuration.** All tunables are environment variables:
  - `DSH_CORE_MEMORY_CAP` — core injection cap in chars (0 = unlimited, default)
  - `DSH_MEMORY_INDEX_COUNT` — injected index size (default 12)
  - `DSH_AMBIENT_SEARCH_BOOST` — base score boost for ambient entries (default 5)
  - `DSH_AMBIENT_PRIORITY_WEIGHT` — priority divisor for search scoring (default 100)
- **Index preserves ambient metadata.** `rebuildIndex()` now carries `ambient` and `ambient_priority` fields so sorting and search boost survive index rebuilds.

### Unchanged

- Frontmatter parser (naive `key: value` line splitter)
- Write path (`memory_write` creates timestamped ids)
- Corrupt-index auto-rebuild safety net
- Persona system (presets, custom text, settings page)
- Atomic write serialization
- Web server persona settings route
- Tool definitions and descriptions (except `memory_search` and `memory_list` additions noted above)
- License (MIT)

## 1.0.1 (upstream)

Original release by jiuling. See [npm](https://www.npmjs.com/package/dsh-core-memory).
