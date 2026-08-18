# Project docs format

`BRIEF.md` and `ROADMAP.md` live at `{project}/docs/BRIEF.md` and
`{project}/docs/ROADMAP.md`.

The technical spec file follows the open **AGENTS.md** standard
(agents.md, stewarded by the Agentic AI Foundation / Linux Foundation,
co-authored by OpenAI, Google, Cursor, Factory, etc.) and lives at
**`{project}/AGENTS.md` — repo root, not under `docs/`** — that's where every
AGENTS.md-aware tool looks for it. A minimal `{project}/CLAUDE.md` stub at
the root points to it, since Claude Code otherwise expects `CLAUDE.md`
specifically:

```markdown
# See AGENTS.md

This project's instructions live in [`AGENTS.md`](./AGENTS.md).
```

Always write both when creating or updating the spec file — the stub is
one line of maintenance and keeps Claude Code and any AGENTS.md-reading
tool pointed at the same source of truth. Never fork content between the
two; the stub never grows content of its own.

If a project already has a `docs/CLAUDE.md` from before this convention,
treat it as the AGENTS.md content on migration: move it to
`{project}/AGENTS.md`, drop the root stub in its place, and remove the old
`docs/CLAUDE.md` — flag the move to the user rather than silently leaving
both around.

A UI-facing project can also have a **`{project}/docs/DESIGN.md`** — the
open Google Labs format (design.md, Apache-2.0) for describing a visual
identity to coding agents. Unlike AGENTS.md/CLAUDE.md, no external tool
convention forces it to the repo root, so it lives alongside
BRIEF.md/ROADMAP.md in `docs/`. Skip it entirely for projects with no UI
(backend service, CLI, script) — an empty/boilerplate DESIGN.md is worse
than no file.

A working session on a project updates these directly instead of writing a
separate recap file: the job is to move the project's actual docs forward,
not produce a fourth artifact nobody re-reads.

These are real shapes pulled from existing projects — match them, don't
invent a new schema per project.

## BRIEF.md — the idea / cahier des charges

Two variants depending on project maturity — pick based on what's already
there (or ask if creating fresh):

**Early-stage** (concept not settled yet — this is who most benefits from a
working session on the idea):
```markdown
# Brief — {Project Name}

## Vision
{1-3 sentences: what it is, core concept}

## Concept à définir
- {open question about mechanics/scope/format}
- {open question}

## Stack envisagée
{À définir lors du cadrage. — or a tentative stack if one emerged}

## MVP
{smallest playable/usable version}
```

**Settled** (scope and stack already decided):
```markdown
# Brief — {Project Name}

## Vision
{1-3 sentences}

## Cible
{who this is for, how it's deployed/used}

## Périmètre
- {feature/capability in scope}
- {feature/capability in scope}

## Stack
{stack, one line or short list}

## Contraintes
- {hard constraint}
```

**Merge rules when the file already exists:**
- Never silently rewrite `## Vision` — it's the foundational paragraph. If
  the session clearly changed the vision, show the user the proposed new
  wording and confirm before writing it.
- Append new bullets to `Périmètre` / `Concept à définir` / `Contraintes` —
  don't remove existing ones, even if the session didn't revisit them.
- If a project moves from "concept à définir" to "settled" during the
  session (the open questions got resolved), it's fine to
  restructure from the early-stage shape to the settled one — say so
  explicitly to the user before doing it.

## ROADMAP.md — concrete next actions

```markdown
# Roadmap — {Project Name}

> {optional one-line context, e.g. "Concept à cadrer avant de coder."}
> Statuts : Todo · Ready · Running · Done · Blocked · Cancelled

| Priorité | Type | Statut | Responsable | Tâche | Description | Dépendances |
|---|---|---|---|---|---|---|
| P1 | FEAT | Todo | Moi | {task title} | {what/why, can be longer than the title} | {row #s this depends on} |
```

**Canonical schema — this is the target shape every project's `ROADMAP.md`
should converge to.** `/project audit` detects files that don't match it
(old status words, missing `Description`/`Dépendances` columns) and offers
to normalize them; `/project new` always writes this shape for new rows.

Column meaning:

- `Priorité` (P1/P2/P3) — purely a planning concept. P1 = blocks everything
  else / do first, P2 = important but not blocking, P3 = nice to have. A
  session's "next steps" are rarely P1 unless one clearly unblocks the
  rest.
- `Type` — six valeurs fermées, observées dans l'usage réel de tous les
  projets : `FEAT` (fonctionnalité), `TACHE` (travail technique sans
  fonctionnalité utilisateur directe), `UX` (design/ergonomie), `BUG`,
  `INFRA` (déploiement, infra, outillage), `AUDIT` (vérification/état des
  lieux). Ne pas inventer une septième valeur — si aucune ne convient,
  c'est `TACHE` par défaut plutôt qu'un nouveau mot.
- `Statut` — `Todo` · `Ready` · `Running` · `Done` · `Blocked` · `Cancelled`.
- `Responsable` — who should own it. Two canonical values:
  - `Humain` — needs something an agent structurally cannot do (API keys,
    credentials, an external human-only action).
  - `Collaboratif` — needs a human+Claude interactive session (idea-shaping
    discussion, design decision, arbitration between options).
  Old files may still use `Moi` from before this convention —
  `/project audit` offers to migrate it to `Humain` (ask the user if the
  intent isn't obvious from the description) when it detects this legacy
  vocabulary.
- `Tâche` — short title.
- `Description` — write the actual context here instead of dumping it into
  a vague "Notes" column — what/why, not just a title fragment.
- `Dépendances` — reference other rows by their task title or row position
  (e.g. "après la ligne Cadrer le concept").

**Convention "critère de done"** — pour une tâche substantielle, termine
`Description` par une phrase testable : `Critère de done : {condition
vérifiable}.` (ex: "le endpoint /health répond 200", "les tests de `auth/`
passent"). Sans ce critère, une session de travail (humaine ou Claude Code)
peut se déclarer finie sur une base optimiste — pas obligatoire pour une
ligne triviale, mais une bonne habitude par défaut dès que la tâche
implique du code.

Rules:
- **Append rows for the session's Next Steps** — one row per concrete
  action. Don't add a row for a vague idea, only for something actionable.
- **Dedupe**: skip adding a row if a very similar task already exists
  (same intent, even if worded differently) — don't create duplicates.
- New rows default to `Statut: Todo`, never `Ready` — promoting to `Ready`
  is a deliberate human decision, not something a session or audit
  does automatically.

## AGENTS.md — specs, installed or planned

This is the technical spec file, not the idea file. Only touch it when the
session produced concrete technical decisions (stack choice, a structural
decision, a convention, a planned system) — a purely ideation-only session
with no technical conclusions shouldn't touch this file at all.

Existing sections seen across projects (use whichever already exist in the
target file; add a new section only if none fits — these map onto the
AGENTS.md spec's recommended sections: Project Overview, Dev Environment,
Build & Test, Code Style, Testing, Contribution, Security):
```markdown
# AGENTS.md — {Project Name}

{1-3 line description}

## Stack
{bullet list, tech + version + rationale in short}

## Structure
{code tree with one-line comments per file, if relevant}

## Conventions
{bullet list — hard rules, "always/never" style}

## {Feature-specific section, e.g. "Système X"}
{how a specific system works, only if the project has one worth documenting}

## Déploiement
{cible: où ça tourne (host, plateforme...)}
{commande/process: comment on déploie concrètement}
{politique: autonome | staging-autonome-prod-gated | toujours-gated}
```

`## Déploiement` n'est renseigné que si le projet a effectivement un
déploiement (pas de section fabriquée pour un script local). `Politique`
documente le niveau d'autonomie attendu d'une session Claude Code sur ce
projet — absente ou ambiguë, traiter comme `toujours-gated` par défaut
(demander confirmation avant tout déploiement en prod).

- If the session decided something new that doesn't fit an existing
  section, add it under a dated subsection so it's clearly traceable and
  doesn't get silently merged into established conventions:
  `### Décisions du {YYYY-MM-DD}`
- Never contradict an existing convention line without flagging it to the
  user first — conventions in AGENTS.md are often hard-won ("pas de
  TypeScript", "styles inline uniquement"); a new idea that
  conflicts with one needs an explicit override, not a silent edit.
- Writing/updating `AGENTS.md` always keeps the root `CLAUDE.md` stub in
  sync (create it if missing) — see the stub format above.

## DESIGN.md — visual identity, for UI-facing projects only

Follows the [design.md spec](https://github.com/google-labs-code/design.md):
YAML frontmatter with machine-readable design tokens, then a markdown body
in this fixed section order — Overview, Colors, Typography, Layout,
Elevation & Depth, Shapes, Components, Do's and Don'ts.

```markdown
---
version: alpha
name: {design system name}
colors:
  primary: "#______"
typography:
  h1:
    fontFamily: {font}
    fontSize: {size}
    fontWeight: {weight}
---

## Overview
{brand personality, emotional intent — 1-3 sentences}

## Colors
{palette with semantic roles: primary/secondary/accent/error/etc.}

## Typography
{font hierarchy and usage}

## Layout
{grid and spacing strategy}

## Elevation & Depth
{shadows, layering, visual hierarchy}

## Shapes
{corner radius, form language}

## Components
{styling guidance for buttons, cards, inputs, etc.}

## Do's and Don'ts
{practical guardrails — concrete, not generic}
```

Only create this when the session/project has an actual visual identity to
capture (colors, typography already chosen or being decided) — don't
fabricate tokens to fill the template. If the project has no UI, skip this
file entirely rather than writing a boilerplate placeholder.
