# LLM Wiki — Schema & Conventions

This file is the contract between the four agents (Ingest, Query, Curate, Lint) and the wiki. Every agent reads it before acting. Humans read it to understand what the LLM will and will not do.

The design follows Karpathy's LLM Wiki pattern: raw sources are immutable, the wiki is an LLM-maintained Markdown knowledge base, and this file is the schema that keeps the two in sync.

## Three layers

1. **`raw/`** — immutable source drop zone. Humans put files here; agents read them. Agents may save fetched URLs here but must not edit existing files in `raw/`.
2. **`wiki/`** — LLM-generated and LLM-maintained Markdown. Everything under `wiki/` is fair game for agents to write, within the rules below.
3. **`wiki/AGENTS.md`** — this file. The schema. Changes here change agent behavior.

## Folder layout under `wiki/`

```
wiki/
├── AGENTS.md          # this file
├── index.md           # the catalog — every wiki page appears here
├── log.md             # append-only activity log
├── sources/           # one page per raw source
├── entities/          # people, orgs, projects, products, places
└── concepts/          # ideas, techniques, definitions, methods
```

No nested subfolders inside `sources/`, `entities/`, or `concepts/`. Structure is provided by `index.md`, not the filesystem. Keeping pages flat makes renames cheap and links stable.

## File naming

- `kebab-case.md` for every file: `karpathy.md`, `rotary-position-embedding.md`.
- One canonical page per entity/concept. Aliases are handled by adding an `## Aliases` section inside the canonical page, not by creating alias files.
- Source pages take the slug of the source title, prefixed by year if disambiguation is needed: `karpathy-llm-wiki-2026.md`.

## Page shapes

### Source page (`wiki/sources/*.md`)

```markdown
---
origin: raw/karpathy-llm-wiki.md      # relative path OR url
ingested: 2026-04-22
author: Andrej Karpathy                # optional
tags: [llm, knowledge-management]      # optional, short
---

# Title of the Source

3–8 sentence summary in the source's original language.

## Key claims

- Standalone, checkable claim 1.
- Standalone, checkable claim 2.
- ...

## See also

- [entity-name](../entities/entity-name.md)
- [concept-name](../concepts/concept-name.md)
```

### Entity or concept page (`wiki/entities/*.md`, `wiki/concepts/*.md`)

```markdown
# Canonical Name

One-paragraph definition or summary.

## Aliases

- Other names this page also covers (optional section).

## Body

Free-form sections as needed. Each substantive claim should be followed by a citation to a source page.

## Sources

- [source-page-title](../sources/source-page-slug.md) — one-line note on what this source contributes.
- ...
```

Every entity and concept page **must** have a `## Sources` section with at least one bullet. A page without a source bullet is invalid; Lint will flag it.

## Links

- Relative Markdown links only: `[text](../entities/foo.md)`. Never absolute paths, never `file://`.
- Every citation links to a **wiki** page — usually a source page. Do not cite `raw/` files from wiki content; the source page is the stable citation target.
- Links from `wiki/` pages to raw sources are allowed only inside a source page's frontmatter (`origin:` field).

## `index.md`

The catalog. Maintained by Ingest and Curate; read by Query and Lint.

Three sections (`## Sources`, `## Entities`, `## Concepts`), each containing alphabetized bullet list entries of the form:

```markdown
- [page-title](sources/page-slug.md) — one-line summary.
```

Every page under `wiki/` (other than `index.md`, `log.md`, `AGENTS.md` themselves) must appear in `index.md`. If a file exists on disk but is missing from the index, Lint flags it as index drift.

## `log.md`

Append-only. One line per operation:

```
YYYY-MM-DD | OP | target | files-touched | note
```

Where:
- `OP` ∈ {`INGEST`, `QUERY`, `CURATE`, `LINT`}.
- `target` is the source slug (for INGEST), topic slug (for QUERY), short description (for CURATE), or `-` (for LINT).
- `files-touched` is a comma-separated list of relative paths, or `-` if none.
- `note` is a free-form short note; use it for LINT counts.

Never rewrite existing log lines. If you need to correct one, append a new line noting the correction.

## When to use which agent

| You want to... | Use |
|---|---|
| Add a new source to the wiki | **Ingest** |
| Ask a question that the wiki should be able to answer | **Query** |
| Merge, split, rename, move, or delete wiki pages | **Curate** |
| Audit the wiki for contradictions, orphans, broken links | **Lint** |

Operations **not** covered by any agent (edit one page's prose, fix a typo) are done by the human directly; the agents exist only for the bookkeeping that scales poorly.

## Language policy

Each page is written in the language of its primary sources. An entity page started in Chinese stays in Chinese; a concept page started in English stays in English. Agents must not translate mid-page. If a single entity is discussed in both Chinese and English sources, prefer the language of the earliest source page and cite cross-language sources with the note written in the page's language.

## Invariants Lint will enforce

- Every wiki page is linked from `index.md`.
- Every entity and concept page has a `## Sources` section with at least one valid bullet.
- Every source page has `origin` and `ingested` in its frontmatter.
- Every relative link resolves.
- No two pages cover the same canonical entity or concept.
