# Artifacts

Downstream deliverables built **from** the wiki — shareable outputs like one-pagers, roadmaps, or prototypes that distill PKS knowledge for an audience.

## Why they live here

Artifacts sit inside the PKS repo so agents working in PKS can read and write them locally without crossing repo boundaries — most importantly, the [`artifact-sync`](../schema/workflows/artifact-sync.js) workflow, which reconciles an artifact's `Human/` layer against current wiki knowledge.

## What is tracked vs. ignored

- **Tracked:** this folder and this `README.md` only — the structure and instructions.
- **Ignored:** every artifact project inside (`artifact/<name>/`). Artifact content is private knowledge (it carries internal project context) and is never committed to the PKS remote. See `.gitignore`.

Each artifact is its own self-contained project — typically its own git repo with a [HAI-Harness](https://github.com/ClaudiusMa/HAI-Harness) layer (`Human/` for product thinking, `Agents/` for agent tasks). PKS does not track, commit, or push that content.

## How PKS interacts with an artifact

PKS only ever touches an artifact's `Human/` folder, and only to **propose**:

- Run `artifact-sync` (see [`CLAUDE.md`](../CLAUDE.md) → Workflows → Artifact Sync Workflow). It writes one dated report, `<artifact>/Human/knowledge-sync-<date>.md`, listing how new wiki knowledge promotes / resolves / contradicts / adds to the artifact's cited claims.
- It never edits the canonical `Human/` files, the `Agents/` layer, or the deliverable. After you fold the report in, the artifact's own HAI-Harness agent rebuilds the deliverable in its own session.

## Adding an artifact

Drop the project under `artifact/<name>/`. It's ignored by default — nothing to configure. Give it a `Human/` layer whose entries cite their source wiki pages (`Source:` lines) so `artifact-sync` can reconcile by citation.
