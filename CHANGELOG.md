# Changelog

## 2.0.2 (2026-11-17) — Fix client registration id so the persona settings page loads

### Fixed

- **Browser half registered under the unscoped upstream id.** `lib/client.js` submitted `window.__ModuleLoader__.load({ id: 'dsh-core-memory', … })`. `@deepseek-ai/dsh-client-modules` keys a factory by the registered id and then looks the id up in the client graph, where the row is the package name; since 2.0.0 this fork is published as `@nuochan/dsh-core-memory`, so the lookup found nothing and the Web UI reported `client-modules: could not load "@nuochan/dsh-core-memory": … loaded without registering "@nuochan/dsh-core-memory" via __ModuleLoader__.load` (observed on DSH desktop 0.2.2-rc.3). The registration id now equals the package name, which is the loader's stated contract. The header comment naming the served bundle path was corrected in the same pass; the `/dsh-core-memory/persona` HTTP route is a host route and is unchanged.
- **Scope of the breakage.** Only the persona settings section failed to appear. The Host half — memory tools, ambient injection, and the `/dsh-core-memory/persona` route — activated normally, because `cordis.patch.yml` mounts the plugin by package name and never consults the client registration id.
- **Second instance of the 2.0.1 rename miss.** That release corrected the same unscoped self-reference in `cordis.patch.yml` and recorded the identity (`id: dsh-core-memory`, exported `name`) as intentionally unchanged. The client registration id was overlooked there; it is the one identity string that must equal the package name, because the browser module loader indexes by it.

## 2.0.1 (2026-11-14) — Fix bundle self-reference so the plugin activates

### Fixed

- **Bundle patch referenced the unscoped upstream package name.** `cordis.patch.yml` inserted the plugin with `name: 'dsh-core-memory'`, but this fork is published as `@nuochan/dsh-core-memory`. On profiles where only the scoped package is installed, the loader failed with `ERR_MODULE_NOT_FOUND: Cannot find package 'dsh-core-memory'` and the plugin never activated (observed on DSH desktop 0.2.1-rc.5 through 0.2.2-rc.3). The patch now references `@nuochan/dsh-core-memory`. The plugin identity (`id: dsh-core-memory`, exported `name`) is unchanged.

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
