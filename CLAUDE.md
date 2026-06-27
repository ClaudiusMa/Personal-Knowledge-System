# CLAUDE.md

This repo is an Obsidian vault and an open-source schema for a private LLM-maintained knowledge system.

The public repo contains structure only: this file, `README.md`, `.gitignore`, `package.json`, `bin/`, and templates under `schema/`.
The private wiki content lives under `wiki/` and is gitignored. Treat every source, URL, summary, take, index entry, and log entry as private knowledge.

## Core Rules

- Obsidian is the frontend. The LLM edits markdown files.
- GitHub stores structure, not knowledge.
- Use one wiki at launch: `wiki/main/`.
- Use one root `CLAUDE.md` for all behavior.
- Do not add tooling unless the user asks for it or repeated pain justifies it.
- Before creating or editing any file, ask: structure or knowledge?
  - Structure: tracked.
  - Knowledge: gitignored.
- Never ingest private content into tracked files.
- `wiki/main/raw/` holds the original source files. These are the source of truth for every ingested source. The LLM reads them but never modifies or deletes them.
- Source pages are annotation layers, not content copies. They reference the original file via the `file:` frontmatter field and a `## Source` embed. They never duplicate the file's content.
- Do not produce LLM-generated prose summaries on source pages. The `## What's in this source` section is a factual index only — bullet points naming sections, entities, scope, format. The user's interpretation belongs in `## My take`, dated and updatable.
- Never change the meaning of a user's `My take` content. This section is the user's voice. Allowed: silently auto-fix obvious typos and misspellings. Not allowed without explicit user approval: grammar changes, rephrasing, restructuring sentences, expanding, condensing, adding caveats, softening claims, inserting new ideas, or changing word choice.
- For any edit to a take beyond a clear typo (e.g., grammar, punctuation that affects meaning, word choice), show the proposed diff and wait for explicit approval before writing.
- If a take seems incomplete or unclear, surface a question to the user. Do not fill in the gap by editing.

## Page Types

- `source`: an annotation layer for one ingested source. References the original file in `wiki/main/raw/` via the `file:` frontmatter and a `## Source` embed. Contains a factual `## What's in this source` index and optional dated takes. **Externally-authored content** (product briefs, papers, books, formal docs).
- `note`: user-authored content — meeting notes, reading reactions, journals, scratch thoughts, self-written explainers. The whole body is the user's voice; there is no `## My take` section because the note IS the take. Lighter-touch ingest than `source` (no discussion phase, no entity/concept promotion beyond linking). Lives at `wiki/main/notes/`, not `sources/`.
- `entity`: a concrete recurring **non-person** thing: organization, product, tool, place, book, project, etc. **People are never entity-paged** — they're mentioned inline as plain text (see Ingest Workflow thresholds).
- `concept`: a descriptive idea, theme, pattern, term, or framework.
- `position`: an opinionated personal claim or sustained view. Positions link to supporting, conflicting, and related pages.
- `question`: an unresolved question being tracked.
- `project`: a living, externally-owned codebase or folder that lives **outside the vault** and is never ingested. A `project` page is a thin pointer + annotation layer — like a `source` page, but aimed outward at something that keeps changing. `repo:`/`path:` frontmatter locate it, `## What's worth reading` names its entry points, and `## How it connects` ties it to wiki pages. The LLM reads from it **live** at query time; it never copies the project's content in and never modifies it. Use it for work that relates to your knowledge but didn't originate in PKS.

Filename conventions:

- **`wiki/main/sources/` and `wiki/main/raw/`**: Title Case with spaces. **No date prefix** — dates live in frontmatter (`published:`, `captured:`). **Short — capture the essence only**, drop em-dash subtitles, drop redundant brand prefixes when context implies them. The source page filename mirrors the source's essence; the raw file keeps the author's framing unless the author's filename is unreadable, in which case rename. Long descriptive titles make the wiki harder to scan and harder to render in a future frontend — keep them tight.
- **`wiki/main/concepts/`, `entities/`, `positions/`, `questions/`, `projects/`**: lowercase kebab-case. These function as retrieval tags (see the Direction section in `README.md`), so the filename IS the tag.

Examples:

- `wiki/main/sources/AI Design Principles Audit.md`
- `wiki/main/raw/AI Design Principles Audit.pdf`
- `wiki/main/concepts/ai-agents.md`
- `wiki/main/positions/agents-are-workflow-tools-not-coworkers.md`
- `wiki/main/questions/what-makes-a-note-worth-promoting.md`
- `wiki/main/projects/fraud-platform.md`

## Required Wiki Files

Each wiki has:

- `index.md`: content catalog. Read this before any ingest or query.
- `log.md`: chronological append-only record.

Log headings must use:

```md
## [YYYY-MM-DD] ingest | Title
```

Other operation types may use:

```md
## [YYYY-MM-DD] query | Topic
## [YYYY-MM-DD] lint | Scope
## [YYYY-MM-DD] connect | Project
```

## Workflows

Every substantive user message falls into one of these categories. **Identify the category first; then follow the matching workflow end-to-end.** Do not ask the user which category — infer from the message and proceed.

| User input | Category | Workflow |
| --- | --- | --- |
| A new externally-authored file in `wiki/main/raw/` (PDF, formal doc, paper, brief), or explicit "ingest this source" | ingest | [Ingest Workflow](#ingest-workflow) |
| A user-authored file or content drop (meeting notes, reading reaction, journal, scratch thought, self-written explainer) | note | [Note Workflow](#note-workflow) |
| "Connect/link project X" — point the system at a living repo or folder that lives outside the vault | connect-project | [Connect Project Workflow](#connect-project-workflow) |
| Any question, advice request, planning, recommendation, opinion, or "what should I do" prompt | query | [Query Workflow](#query-workflow) |
| "Lint the wiki" / "check for stale pages" / suggested every 10 ingests | lint | [Lint Workflow](#lint-workflow) |
| "Sync artifact X" / "update/refresh artifact X with new knowledge" | artifact-sync | [Artifact Sync Workflow](#artifact-sync-workflow) |
| Session opens with any input | session-start | [Session Start](#session-start) (then routes to one of the above) |
| Changes to CLAUDE.md, schema, templates, git/remote config, environment setup, or debugging the system itself | meta | No prescribed workflow — diagnose the problem, discuss the approach with the user, get explicit sign-off, then act. Do not edit files until aligned. |

If a single message triggers multiple categories (e.g., a new file is in `raw/` AND the user asked a question about it), default to **ingest** first; the discussion within ingest covers the question. If genuinely ambiguous, ask the user which category before proceeding.

#### Source vs Note classification

When a file lands in `raw/` and could be either, classify by signals. Proceed silently when 4+ signals agree; ask once only when signals split.

| Signal | source | note |
| --- | --- | --- |
| Format | PDF, .docx, exported doc | bare markdown |
| Filename | "Brief", "Proposal", "Audit", "Paper", em-dash subtitle | informal, first-person, "meeting notes", "thoughts on X" |
| Author | external named author | you, or no attribution |
| Voice | third-person, citations, formal | first-person, "I think", "we discussed" |
| Structure | TOC, page numbers, formal sections | bullets, quote+reaction, scratch |

### Session Start

When the user opens a session and gives any input — including a single word, a file mention, or just "go" — first list `wiki/main/raw/`. For any file there that is not referenced by any existing source page's `file:` frontmatter, treat it as a pending ingest. Surface the list to the user and ask which they want to process. If only one is pending, begin the Ingest Workflow immediately and confirm what you're doing. If none are pending, route the user's message into the matching workflow above.

### Ingest Workflow

When ingesting a source:

1. Read `wiki/main/index.md` first.
2. Read related pages suggested by the index.
3. Read the new source.
4. Identify likely related entities, concepts, positions, questions, past sources, and connected projects.
5. Link to relevant existing pages (entities, concepts, positions, questions, past sources, projects) using `[[wikilinks]]` — in the source page's `## Related pages` and inline in `## What's in this source` where natural. If a connected `project` page is relevant and the source's meaning depends on the project's current state, you may read the project live (read-only, within its `## What's worth reading` entry points) to ground the links. Do not analyze the source for contradictions or tensions with existing wiki content; that audit lives in the Lint Workflow.
6. Keep the original file in `wiki/main/raw/`. Create the source page as an annotation layer:
   - Set the `file:` frontmatter to the path of the original.
   - Add a `## Source` section with `![[../raw/<filename>]]` so the original is embedded/linked from the source page.
   - Populate `## What's in this source` with factual bullets only (sections covered, entities mentioned, scope, format).
   - Do not embed the file's content. Do not delete the file.
7. **Discuss the source with the user before capturing `My take`.** After the source page is written, ask 1–3 specific, open questions drawn from the source — what stood out, what to push back on, what feels useful, where it overlaps with the user's existing thinking. Don't ask for a monolithic "take" upfront. Let the conversation move; follow up where the user shows interest. When the user articulates a view, mirror it back as a proposed `My take` ("so your take is X — should I write that down?") and only write on confirmation. Capture verbatim with typo-level cleanup only (per the take-protection rule in Core Rules). If the user has no view or wants to skip, leave `_None yet._`. **The discussion is the point; the take is a byproduct.**
8. Update relevant entity, concept, position, question, and project pages.
9. Promote a take only if it has grown beyond one source.
10. Update `index.md`.
11. Append `log.md`.
12. Show changed files.
13. Run `git status` and verify private content remains untracked.

**Page creation thresholds** (apply when deciding what to create during steps 4 and 8 — do not ask the user, apply the rule):

- **Entity page** — create when (a) the entity already appears in another source or wiki page, or (b) the source treats it as a primary subject (named in the title, repeatedly cited, or central to the source's framing). For one-off mentions, link inline if a page already exists; otherwise reference as plain text and let it promote later if it recurs.
- **People are never entity-paged.** Always mention people inline as plain text ("Sam Szerlip", not `[[sam-szerlip]]`). Reasons: first names collide; person pages are directory metadata, not knowledge; the wiki's focus is concepts, sources, and decisions — not a people directory. Exception: only if a person IS the primary subject of a substantial source (a biography, a memoir), discuss with the user before creating.
- **Concept page** — create when the source explicitly defines it as a domain category (e.g., a named capability area in a competitive audit, a labeled framework, a term the source itself introduces) or when the concept already appears in another source or wiki page. Otherwise mention inline without a stub.
- **Position page** — create only when the user's `My take` expresses a reusable claim that's likely to apply across multiple sources. Do not pre-emptively create from a source's own claims.
- **Question page** — create only for the user's own tracked uncertainties. Never create from a source's sub-questions.
- **Project page** — never auto-created. Create one only on the user's explicit request to connect an external project (see the Connect Project Workflow). A passing mention of an outside repo stays plain text or an inline link, not a project page.

A source page must contain these headings:

```md
## My take
## What's in this source
## Source
```

(`## Related pages` and `## Source notes` may also appear but are not required.)

`What's in this source` is a factual index of the source, not a synthesis. Bullet points naming sections, entities, scope, and format. No prose interpretation — that belongs in `## My take`.

`My take` is the user's reaction, dated by month when present:

```md
### 2026-05
```

If a take is skipped, keep the heading but write:

```md
_None yet._
```

### Note Workflow

For user-authored content (meeting notes, reading reactions, journals, scratch thoughts, self-written explainers). Lighter-touch than Ingest — no discussion, no take ceremony. The whole body is the user's voice.

1. Read the note.
2. **Classify by size and density:**
   - **Small** (< ~1KB) or **already structured/tidy** → skip to step 4, file as-is.
   - **Large** (> ~5KB) or **raw / transcript-like / contains visible filler** → propose a trim in step 3.
3. **Trim (conditional).** In a single message, surface what you'd keep and what you'd drop: "I'd keep A/B/C and drop X/Y. OK or file as-is?" One question, not three. Apply the user's answer literally.
4. **File** at `wiki/main/notes/<Essence Name>.md`. The note page IS the file — no separate `raw/` counterpart. Add frontmatter:
   - `type: note`
   - `note_type:` one of `meeting | reading | thought | scratch | self-explainer`
   - `created:`, `updated:` (notes are mutable)
   - `people:` (if applicable)
   - `related:` `[[wikilinks]]` to existing entities/concepts/sources mentioned
5. **Link to existing pages.** Identify entities, concepts, sources named in the note. Use `[[wikilinks]]` inline and in `related:`. Apply the same page creation thresholds as Ingest — do not auto-create new entity pages from a single mention in a single note.
6. **Update `index.md`** under a separate `## Notes` table (alongside `## Sources`).
7. **Append `log.md`** with a one-line entry: `## [YYYY-MM-DD] note | <Title>`.
8. No discussion phase. No `## My take` section. **The take-protection rule (Core Rules) applies to the entire body** — typo fixes only, never alter meaning.

Notes are equal-weight with sources during retrieval. The Query Workflow does not deprioritize them. The difference between source and note is **provenance**, not **importance**.

### Connect Project Workflow

For wiring in a living, externally-owned project — a repo or folder that lives **outside the vault** — so the system can draw its *current* context on demand. This is **not** an ingest: nothing is copied in, the project keeps changing, and you create a persistent pointer the LLM follows at runtime.

1. Read `wiki/main/index.md`.
2. Confirm the project's location: a canonical `repo:` URL and/or a local `path:` on disk. Capture both when available — `path:` is what the LLM reads live; `repo:` is the portable identity and the clickable link.
3. Read the project **read-only** to build a factual index — top-level structure, what each major dir/file is for, scope, and which entry points are worth reading. Never copy content in; never modify anything in the project.
4. Create the page at `wiki/main/projects/<name>.md` from `schema/templates/project.template.md`:
   - Set `repo:` and `path:` frontmatter.
   - `## What's worth reading`: factual entry points only — which dirs/files matter, what to skip. No prose synthesis (same rule as `## What's in this source`).
   - `## How it connects`: `[[wikilinks]]` to the concepts, positions, questions, and sources this project feeds or is fed by.
   - `## Where it lives`: the repo URL and the local path.
   - `## My take`: leave `_None yet._` unless the user offers a view — no discussion ceremony, but the take-protection rule still applies.
5. Update relevant entity, concept, position, and question pages to link back to the project where natural.
6. Update `index.md` under a `## Projects` table (alongside `## Sources` and `## Notes`).
7. Append `log.md`: `## [YYYY-MM-DD] connect | <Project>`.
8. Show changed files; run `git status` and confirm the project page and its `path:`/`repo:` pointers stay untracked (they live under `wiki/`).

A project page must contain these headings:

```md
## How it connects
## What's worth reading
## Where it lives
```

(`## My take` is optional, dated when present.) To **refresh** a connected project, re-read it and update `## What's worth reading` and `## How it connects` in place; the project itself is never edited by PKS.

### Query Workflow

Follow this sequence before answering. **Do not answer from prior knowledge alone, and do not ask clarifying questions until you have completed steps 1–3.**

1. **Extract key concepts, keywords, and entities from the query.** Restate the user's question in your thinking and list the 2–5 concepts/keywords/entities it touches. This commits you to a frame before retrieval.
2. **Scan `wiki/main/index.md`** (and `wiki/main/log.md` if the question is time-anchored or about past decisions) for the page whose summary best matches the extracted concepts.
3. **Read that one source page in full.** The page body holds the substantive content — user stories, named directions, decisions, evidence, in-scope/out-of-scope — that the index summary collapses. Only read additional pages if the question genuinely spans multiple sources (comparison, cross-source synthesis). If a relevant `project` page surfaces and the question needs the project's *current* state — not just your recorded notes about it — follow its `path:` and read or grep the actual files to ground the answer in what's there now. Stay within the entry points named in `## What's worth reading`; read-only, never modify.
4. **Answer from the wiki.** Ground every specific suggestion, claim, or recommendation in a cited passage. **Do not extrapolate.** If a suggestion can't trace to a wiki page, drop it — do not pad to a "feels right" answer shape (e.g., do not produce three options when only two are grounded). **Tag citations by provenance** — `[[Page Name]] (source)` for source pages, `[[Page Name]] (note)` for notes, and `[[Page Name]] (project)` for project pages (add `, live` when you read the project's current files, e.g. `[[fraud-platform]] (project, live)`) — so the user can see the weight of the grounding at a glance.
5. If the wiki genuinely doesn't cover what's needed, ask clarifying questions — and explicitly name what you checked first (e.g., "I read `index.md` and the internship brief in full, but neither covers X").
6. If the answer creates durable synthesis, offer to file it as a concept, position, or question page. Do not file without approval.

### Lint Workflow

Default cadence: manual, with a suggested lint every 10 ingests.

Check for:

- pages missing from `index.md`
- stale index summaries
- orphan pages
- source pages missing required headings
- contradictions between takes and positions
- repeated concepts without concept pages
- takes ready for promotion
- inconsistent frontmatter
- filename or link drift
- project pages whose `path:` or `repo:` no longer resolves, or whose `## What's worth reading` has drifted from the project's current structure

### Artifact Sync Workflow

When the user asks to sync/update/refresh an **artifact** with current PKS knowledge, run the dynamic workflow at `schema/workflows/artifact-sync.js` — do not do this reconciliation by hand in one context.

Artifacts are downstream deliverables built FROM the wiki (e.g. a shareable one-pager under `artifact/<name>/`). **The artifact's `index.html` is its source of truth** (for that artifact, not the system); some artifacts also carry a populated HAI-Harness `Human/` layer whose entries cite PKS pages via `Source:` lines. The workflow reconciles PKS knowledge **added since the last sync** against the artifact's claims surfaces — `index.html` primary, populated `Human/` as supplement — incremental, not a full re-scan:

- **Scope** (1 Sonnet agent, cheap) — grep-first log read (entry headings only, then just the selected increments by line range; never the whole log outside backfill); watermark + **cumulative** page ledger from the latest prior report only; detects which claims surfaces exist; gates by `mode`.
- **Small delta (≤4 pages — the routine path)** — ONE Sonnet agent reads the claims surfaces + the pages, classifies promotes / resolves / contradicts / drift / adds, self-checks everything (including adversarial self-refute on contradicts/drift), and writes the report. Two agents total.
- **Large delta / backfill** — **Claims** (1 agent: surfaces → compact claims index) → **Reconcile** (batched ~4 pages/agent, claims passed in) → **Verify** (one agent per page with contradicts/drift; adversarially refute those only) → **Synthesize** (write ONE dated report `<artifact>/Human/knowledge-sync-<date>.md`). All Sonnet.

Relevance is filtered at both ends: Scope skips unrelated increments wholesale, and `adds` must clear a **materiality bar** — the artifact is curated, not a wiki mirror, so an add must fill a gap the artifact's own structure implies or strengthen/update a core claim (topical adjacency is not enough), and carries a `whyMaterial` line. Near-misses surface as one-line entries under "Noted, not proposed" in the report, never as full proposals.

If an artifact has no (or an empty) `Human/` folder, the report still lands at `Human/knowledge-sync-<date>.md` — the folder is created on demand as the proposal inbox. If an artifact has neither an `index.html` nor a populated `Human/`, the run stops without a report (no claims surface to reconcile against).

**Dispatch (before launching):** if a prior `knowledge-sync-*.md` exists, pass `mode: incremental`. If none exists, **ask the user latest-delta vs full-backfill** and pass `mode: latest` or `mode: backfill`. To target a specific change, pass `preselectedPages: [{path,title}]` instead of a mode.

Invoke with the `Workflow` tool:

```
Workflow({
  scriptPath: "<pksRoot>/schema/workflows/artifact-sync.js",
  args: { pksRoot: "<abs repo root>", artifactPath: "artifact/<name>", today: "<YYYY-MM-DD>", mode: "incremental" }
})
```

Cost: routine `incremental` runs are ~20-40k tokens (1-3 new pages, two agents); a one-time `backfill` is ~150-350k. Everything runs on Sonnet.

Strict boundary: the workflow **proposes, never applies**. It writes exactly one report file into the artifact's `Human/` (creating the folder if needed); it never edits the canonical `Human/` files, `Agents/`, or the deliverable. After the human folds the report in — `index.html` via the artifact's own agent/session, `Human/` files directly — the deliverable is rebuilt/redeployed separately. PKS only ever writes inside the artifact's `Human/` folder.

## Promotion Rules

Keep source-page takes scoped to the source.

Create or update a `position` page when the user's view becomes reusable across sources.

Create or update a `concept` page when an idea needs neutral explanation.

Create or update a `question` page when uncertainty remains important.

Example:

- Source take: "This article makes me skeptical that agents should be framed as coworkers."
- Concept: `[[ai-agents]]`
- Position: `[[agents-are-workflow-tools-not-coworkers]]`
- Question: `[[when-do-agents-need-human-review]]`

## Position Pages

Positions are personal claims, not neutral topic summaries.

They should include links to:

- related concepts
- supporting sources
- conflicting or complicating sources
- open questions

This makes them useful in Obsidian graph view as synthesis nodes.

## Privacy Checklist

Before every commit or status report:

- `git status` must not show private source files or wiki content as tracked.
- `wiki/` must remain ignored.
- URLs are private.
- `index.md` and `log.md` are private.
- Source summaries, takes, and page links are private.
- Project pages — including their local `path:` and `repo:` pointers — are private; they live under `wiki/`.
