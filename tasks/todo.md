# brain — build spec

Build a public, cross-referenced AI knowledge layer (`~/brain` → `ghostkellz/brain`)
with a strict private/client split, evaluating claude-obsidian against MemPalace.

## Decisions locked

- **Repo:** `ghostkellz/brain`, public, mapped to `~/brain`.
- **Private tier:** separate vault `~/brain-private` → GitLab `git.cktechx.com` (NOT this repo).
- **Clients:** stay in Hudu (`hudu.cktechx.com`). Out of scope for the brain.
- **Primary knowledge system:** claude-obsidian (test-drive).
- **MemPalace:** kept running for a head-to-head evaluation; not retired.
- **Embeddings:** local ollama `nomic-embed-text` (already pulled). No bodies leave the host.
- **Transport:** filesystem MCP (works headless, model-agnostic for Claude + Codex).

## Phase 0 — Repo scaffold (this commit)

- [x] Create `~/brain` skeleton (cheatsheets/, dev/, sources/.raw/, tasks/)
- [x] README with tiering + architecture
- [x] Hardened `.gitignore` (block private/lab/secrets/client patterns)
- [x] This spec
- [ ] `git init`, create public `ghostkellz/brain`, push

## Phase 1 — claude-obsidian test-drive

- [ ] Decide install path: "add to existing vault" (copy `WIKI.md` + configure transport)
      rather than clone-as-vault, to keep our git history clean
- [ ] Configure **filesystem MCP** transport pointing at `~/brain`
- [ ] Wire the same MCP for Codex (filesystem = model-agnostic)
- [ ] `bin/setup-retrieve.sh`; confirm ollama `nomic-embed-text` reranking works on the 5090
- [ ] Set client/private safety config: contextual-retrieval OFF, web research OFF for any
      sensitive content (default off everywhere here since this tier is public)

## Phase 2 — Seed content (start small, let it compound)

- [ ] Export the Google-Doc cross-platform cheatsheet → markdown into `cheatsheets/`
- [ ] Ingest `~/arch` general docs (public-safe subset only — NOT heimdall-stack/security)
- [ ] Ingest top 3 active projects: zqlite, ghostkellz.sh, ckelley.dev → `dev/`
- [ ] Run `lint the wiki` (orphans, dead links, contradictions)

## Phase 3 — Private vault (separate, when ready)

- [ ] Create `~/brain-private`, init git, remote → GitLab `git.cktechx.com` (private)
- [ ] Move heimdall-stack / security / infra notes there (physical separation, not gitignore)
- [ ] Same filesystem MCP transport, scoped to that vault; local-only retrieval

## Phase 4 — Evaluation + decision

- [ ] Repair MemPalace DB (`mempalace repair-status`) so the comparison is fair
- [ ] Run the same set of real queries against claude-obsidian vs MemPalace
- [ ] Decide: keep one, keep both with clear roles, or retire MemPalace
- [ ] Record the decision + reasoning here

## Phase 5 — Publish → brain.ckelley.dev

- [ ] Pick renderer: Quartz (Obsidian-native) vs Astro (matches ckelley.dev/ghostkellz.sh stack)
- [ ] Build pipeline: vault markdown → static site, preserving wiki-links + graph view
- [ ] Deploy under `brain.ckelley.dev` subdomain (CI on push to `main`)
- [ ] Confirm only the public tier renders; no private/lab/client leakage in build output

## Phase 6 — Optional: proactive layer (coleam00 second-brain-starter)

- [ ] Evaluate if proactivity (heartbeat, notifications, drafts) is wanted
- [ ] If yes, scaffold its PRD on top of the `~/brain` markdown memory + n8n + ollama

## Security guardrails (always)

- This repo is PUBLIC. No home-lab topology, security configs, secrets, or client data.
- Private/lab → `~/brain-private` (GitLab). Clients → Hudu. No exceptions.
- Decide a note's tier BEFORE ingesting it, never after.

## Review

_(fill in after Phase 1–2)_
