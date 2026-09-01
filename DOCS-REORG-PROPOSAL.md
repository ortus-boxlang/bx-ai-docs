# BoxLang AI Docs — Reorganization Proposal (post-3.4.0)

Not published to GitBook — this file is intentionally not linked from `SUMMARY.md` (same pattern as `AGENTS.md`). It's a planning document for the docs team; it captures what the 3.4.0 content pass surfaced structurally, ranked by value vs. effort. **No files were moved as part of this proposal** — see the parallel commit for the content fixes (rename sweep, new pages, correctness fixes) that landed alongside it, which required no restructuring.

## Ranked recommendations

### 1. Split `advanced/events.md` (97KB) — High value, medium effort

The file has grown by appending category sections (`## 🔊 Audio Events (34–39)`, `## 🌐 Web Search Events (53–57)`, `## 🧠 Memory Summarization Event (58)`, `## 🔌 Gateway Events (59–61)`, `## 🔐 Decision Store Event (62)`, and now `## 🎮 Gateway Session & Run Control Events (63–69)`) **after** `## 📚 Examples` — outside the `## 📡 Available Events` section that's supposed to hold them. It also has a genuine, pre-existing numbering defect independent of this proposal: the master "Available Events" table's `#` column does not correspond to the `### N.` detail-section numbers (table row 8 is `onAIAgentCreate`; detail section `### 8.` is `onAIChatRequest`), and at least one detail-section number is reused (`### 24.` appears twice — once for `onAiLoaderCreate` in the table's numbering and again for `onAIToolRegistryRegister` as a detail heading). This split is the right moment to fix both:

- `advanced/events/README.md` — concepts, registration (`BoxRegisterInterceptor()` vs. `ModuleConfig.bx` `interceptors`), the lifecycle diagram, one master table generated in **source declaration order** (`ModuleConfig.bx` `customInterceptionPoints`, currently 70 entries) as the single numbering authority.
- One page per category (`advanced/events/agent.md`, `chat.md`, `embeddings.md`, `tools.md`, `memory.md`, `audio.md`, `image.md`, `mcp.md`, `web-search.md`, `gateways.md`, `hitl.md`) with `### eventName` headings — drop the numeric prefix entirely so a future addition never requires renumbering everything after it.
- Move `## 💡 Common Use Cases`, `## ✅ Best Practices`, `## 📚 Examples` into the README.

### 2. Split `deployment/security.md` (72KB, 19 H2s) and promote it — High value, medium effort

Spans API-key management through incident response and compliance. Security is a headline 3.4 story (three guardrail layers, Bedrock Guardrails, HTTP gateway signing) currently buried three levels deep under "Advanced" in the nav — a reader looking for "how do I turn on prompt-injection protection" has no reason to look under a `deployment/` path.

- `deployment/security/` folder: `README.md` (overview + threat model), `api-keys.md`, `prompt-injection.md` (the 5-layer stack — this is the page most 3.4 traffic will want), `tool-security.md`, `data-privacy-compliance.md`, `operations.md` (audit logging, incident response, secure config, network).
- Promote to a top-level `SUMMARY.md` section (`## Security`), not a child of Advanced.

### 3. Give Gateways its own folder — Medium value, low effort

`main-components/gateways.md` now covers resolution, three core gateways, capabilities, configuration, external modules, building a custom gateway, and events — and this pass added a sibling `gateway-sessions.md` alongside it rather than folding it in, specifically to avoid growing the single file further mid-release. Natural follow-up: `main-components/gateways/` with `README.md`, `core-gateways.md`, `building-a-gateway.md`, `sessions.md` (promoting the new page), `events.md` (linking to the split events pages above).

### 4. Fix `SUMMARY.md` nav/path mismatches — Low effort, do anytime

`main-components/models.md`, `deployment/production.md`, and `deployment/security.md` are all listed under `## Advanced` in `SUMMARY.md` while living in unrelated directories (`main-components/`, `deployment/`). Either move the files to match the nav section they're filed under, or move the nav entries to match the files — pick one convention and apply it consistently. This is pure nav hygiene, independent of the two splits above, and safe to do immediately.

### 5. One canonical settings page — Medium value, medium effort

`boxlang.json`'s `settings` block is documented, partially and inconsistently, across at least six pages: `getting-started/installation/README.md`, `provider-setup.md`, `gateways.md`, `human-in-the-loop.md`, `aifence.md`, `aidecisionstore.md`. This pass fixed the specific 45→90 timeout drift across ~10 tables, but the underlying problem — no single source of truth — will recreate this class of bug on every future settings change.

- New `getting-started/configuration.md`, generated section-by-section from `ModuleConfig.bx`'s `configure()` method (provider, apiKey, defaultParams, providers, timeout, logging, returnFormat, skills, audio, image, security, hitl, gateways, webSearch).
- Every other page that currently inlines a settings fragment links to the relevant anchor instead of re-stating defaults.

### 6. An upgrade-path page that survives releases — Medium value, low effort

There's no "upgrading from 3.3 to 3.4" page — only per-version release notes (`readme/release-history/*.md`), each with its own ad-hoc "Migration Guide" section. A reader upgrading across two versions has to read both release notes in full to find breaking changes.

- `readme/upgrading.md`: one page, append-only, organized by breaking/notable change with the version it landed in (e.g. "3.4.0 — `gatewayRegistry()` renamed to `aiGatewayRegistry()`, no alias"). Cross-link from each release-history page's Migration Guide section instead of duplicating.

### 7. A documented pre-release count-check — Low effort, ongoing value

Three counts drift every release and nobody notices until someone greps: the BIF reference README's "N built-in functions" (was 33, corrected to 34 this pass), `advanced/events.md`'s frontmatter interception-point count (was 63, corrected to 70 this pass — and was *already wrong* before 3.4 shipped, since 58+5 mid-cycle events ≠ 63), and the provider count implied by capability tables (this pass also fixed the Bedrock Tools cell, which had been wrong since 3.3.0 added Bedrock tool-use).

- Add a one-paragraph checklist to `AGENTS.md`: "before tagging a release doc, run `ls src/main/bx/bifs/ | wc -l` and `grep -c '"' <(sed -n '/customInterceptionPoints/,/\]/p' ModuleConfig.bx)` against the counts stated in `advanced/reference/built-in-functions/README.md` and `advanced/events.md`'s frontmatter."

## Not recommended right now

- **Splitting `main-components/memory/README.md`** (39KB) — large but internally coherent (one concept: memory types), and its cross-references to `vector-memory.md`/`multi-tenant-memory.md` already form a reasonable three-page structure. Revisit only if it grows past ~50KB.
- **Renumbering the entire `advanced/events.md` master table in place, without the split** — technically possible with a script, but every existing external link to `events.md#N-eventname` anchors would break, and GitBook doesn't auto-redirect anchor fragments. Do the renumbering as part of the split (item 1), where new page URLs are being introduced anyway and old links can be redirected once, not twice.
