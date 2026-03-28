# Skill Protocol

This document defines the SKILL.md format used by MindFlow skills. Every skill — whether atomic or orchestrating — must include a SKILL.md at its root. This file serves as the single source of truth for what a skill does, how it behaves, what it requires, and what it produces.

---

## Frontmatter Specification

Each SKILL.md begins with YAML frontmatter enclosed by `---` delimiters. Fields are divided into required and optional.

### Required Fields

| Field | Type | Description |
|---|---|---|
| `name` | string (kebab-case) | Unique identifier for the skill, e.g. `paper-ingest` |
| `description` | string | Pushy trigger description (see below) |
| `version` | string (semver) | Semantic version, e.g. `1.0.0` |
| `intent` | enum | Primary research activity this skill supports (see values below) |
| `capabilities` | string[] | Functional capabilities this skill uses (see values below) |
| `domain` | enum | Knowledge domain this skill targets (see values below) |
| `roles` | enum[] | Supported interaction modes (see values below) |
| `autonomy` | enum | Default level of autonomous action (`high` / `medium` / `low`) |
| `allowed-tools` | string[] | Claude Code tools this skill may invoke |

**`description` — Pushy Trigger Principle:**

Description 应积极触发（"pushy"）——明确描述**触发场景**，降低 under-triggering 风险。不要只写"做什么"，要写"什么情况下该用我"。

| Bad | Good |
|-----|------|
| `消化一篇论文，生成结构化笔记` | `当 Supervisor 给出论文 URL/标题/PDF/DOI，或阅读队列中有待处理论文时，消化论文并生成结构化笔记到 Papers/` |

**`intent` values:**

- `literature` — reading, summarizing, and connecting research papers
- `ideation` — generating and refining research hypotheses
- `experiment` — designing and running computational experiments
- `analysis` — interpreting results and extracting insights
- `writing` — drafting papers, notes, or structured documents
- `evolution` — self-improvement, protocol updates, and meta-level tasks
- `orchestration` — coordinating other skills into multi-step workflows
- `utility` — file management, formatting, and general-purpose helpers

**`capabilities` values:**

- `search-retrieval` — querying external sources or the local vault
- `research-planning` — constructing and updating research agendas
- `cross-validation` — comparing results across papers or experiments
- `data-processing` — transforming, filtering, or aggregating structured data
- `training-tuning` — running or configuring ML training jobs
- `evaluation-benchmarking` — measuring model or method performance
- `prompt-structured-output` — producing structured output via LLM prompts
- `visualization-reporting` — generating charts, tables, or summary reports
- `agent-workflow` — spawning sub-agents or calling other skills

**`domain` values:** `general` / `cs-ai` / `bioinformatics` / `medical` / `vision` / `nlp` / `data-engineering`

**`roles` values:**

- `autopilot` — skill runs end-to-end without Supervisor input
- `copilot` — skill drafts or proposes; Supervisor reviews before finalizing
- `sparring` — skill engages in back-and-forth dialogue with the Supervisor

**`autonomy` values:**

- `high` — may create files, update memory, and modify agenda without confirmation
- `medium` — may read and draft; must confirm before writing to shared state
- `low` — proposes only; all writes require explicit Supervisor approval

**`allowed-tools` values** (Claude Code tool names):

`Read`, `Write`, `Edit`, `Glob`, `Grep`, `Bash`, `WebSearch`, `WebFetch`

### Optional Fields

| Field | Type | Description |
|---|---|---|
| `input` | object[] | Declared inputs; each object has `name` and `description` |
| `output` | object[] | Declared outputs; each object has `file` (glob pattern) and `memory` (memory file to update) |
| `related-skills` | string[] | Other skill names that compose well with this one |
| `budget` | object | Resource limits for heavy skills. Fields: `max_web_calls` (int, WebSearch+WebFetch total), `timeout_hint` (string, e.g. "30min"). Omit for simple skills. |

### Example Frontmatter

```yaml
---
name: paper-ingest
description: Fetch, parse, and create a structured note for a research paper
version: 1.2.0
intent: literature
capabilities:
  - search-retrieval
  - prompt-structured-output
domain: cs-ai
roles:
  - autopilot
  - copilot
autonomy: medium
allowed-tools:
  - Read
  - Write
  - WebFetch
  - Glob
input:
  - name: paper_url
    description: URL to the paper (arXiv, PDF, or journal page)
  - name: paper_title
    description: Fallback title if URL is not available
output:
  - file: "Papers/*.md"
    memory: insights.md
related-skills:
  - literature-review
  - insight-extraction
---
```

---

## Body Sections

Every SKILL.md body must include the following sections. Sections are written as Markdown headings (`##` or `###`).

| Section | Required | Purpose |
|---|---|---|
| `## Purpose` | Yes | What problem this skill solves and why it exists |
| `## Steps` | Yes | Numbered, step-by-step execution instructions for the Researcher |
| `## Guard` | Yes | Preconditions, invariants, and prohibited actions (during execution) |
| `## Verify` | Recommended | Post-execution quality checklist — mechanical checks on output quality |
| `## Examples` | Recommended | Concrete input/output examples or usage scenarios |

### Purpose

Explain the skill's goal in 2-5 sentences. Include:
- The research workflow it fits into
- What inputs it consumes
- What outputs or effects it produces

### Steps

A numbered list of executable instructions. Each step should be unambiguous enough for the Researcher to follow without clarification. Use sub-steps where needed. Reference other skills using their kebab-case names.

```markdown
## Steps

1. Receive `paper_url` or `paper_title` from the user.
2. If `paper_url` is provided, fetch the page with WebFetch.
3. Extract title, authors, abstract, and publication date.
4. Look up any existing note in `Papers/` using Glob to avoid duplicates.
5. Populate the `Templates/Paper.md` template with extracted metadata.
6. Save the completed note to `Papers/YYMM-ShortTitle.md`.
7. Append a provisional insight entry to `Workbench/memory/insights.md` if a novel claim is found.
```

### Guard

A bulleted list of rules the skill must never violate. These act as hard constraints during execution.

```markdown
## Guard

- Never overwrite an existing paper note without explicit user confirmation.
- Never modify `agenda.md` Mission section.
- Do not mark an insight as `validated` without ≥2 independent evidence sources.
- If `autonomy: low`, produce a draft only — do not write any files.
```

### Verify

A checklist of mechanical assertions to run after execution completes. Each item must be objectively verifiable (no subjective judgments).

**Verify vs Guard**: Guard constrains behavior _during_ execution ("don't do X"). Verify checks output quality _after_ execution ("did the output meet the bar?").

```markdown
## Verify

- [ ] Output file exists and is non-empty
- [ ] Frontmatter required fields are all populated
- [ ] No `[TODO]` or `[TBD]` placeholders remain
- [ ] All paper references use `[[wikilink]]` format
```

### Examples

Optional but strongly recommended. Show representative invocations and expected outputs. Use fenced code blocks or Markdown tables.

---

## Complex Skills: The `references/` Subdirectory Pattern

When a skill's SKILL.md exceeds approximately 200 lines, or when it requires supporting reference material (schemas, lookup tables, extended examples), split the content using a `references/` subdirectory inside the skill directory.

**Structure:**

```
skills/
  paper-ingest/
    SKILL.md              # Frontmatter + concise Steps + Guard
    references/
      field-mapping.md    # Detailed mapping of paper fields to frontmatter keys
      venue-list.md       # Canonical venue abbreviations
      example-note.md     # Full worked example of an output note
```

**Rules for the `references/` pattern:**

- SKILL.md remains the authoritative entry point. It must be self-contained enough to execute; `references/` files are supplementary detail.
- Reference files are linked from SKILL.md using relative Markdown links, e.g. `[field mapping](references/field-mapping.md)`.
- Reference files do not have their own frontmatter — they are plain Markdown.
- The `references/` directory may not contain nested skill directories (no sub-skills here; use orchestration instead).

---

## Skill Levels

Skills are organized into three levels based on scope and composition.

### Level 0 — Atomic

A single-purpose skill that calls Claude Code tools directly. It does not invoke other skills.

- Typically < 150 lines
- `capabilities` list has 1-2 entries
- `intent` is a leaf activity (not `orchestration`)
- Example: `paper-ingest`, `tag-assign`, `insight-extract`

### Level 1 — Orchestration

A skill that sequences or conditions other skills to accomplish a compound goal. It calls Level 0 skills by name.

- `intent: orchestration`
- `capabilities` includes `agent-workflow`
- Steps reference other skills explicitly: "Run `paper-ingest` for each URL."
- Example: `literature-sweep`, `weekly-digest`, `experiment-pipeline`

### Level 2 — Global

A skill that operates across the entire vault or research state. It may invoke multiple Level 1 skills, modify `agenda.md`, update memory files, or trigger cross-cutting evolution.

- Typically run on a schedule or by explicit Supervisor command
- High privilege: `autonomy: high` and broad `allowed-tools`
- Must have an especially strict `## Guard` section
- Example: `memory-distill`, `agenda-sync`, `vault-health-check`

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Skill directory name | kebab-case | `paper-ingest/` |
| Category directory name | numbered prefix + kebab-case | `1-literature/` |
| Skill entrypoint filename | Always uppercase | `SKILL.md` |
| `name` field in frontmatter | Must match directory name exactly | `paper-ingest` |
| `related-skills` entries | kebab-case, no path prefix | `["insight-extract", "tag-assign"]` |

**Rationale for numbered category prefixes:** Obsidian and most file explorers sort directories alphabetically. Numbered prefixes (`01-`, `02-`, ...) enforce a logical reading order that mirrors the research workflow (literature → ideation → experiment → analysis → writing → evolution).

**Example directory layout:**

```
skills/
  1-literature/
    paper-digest/
      SKILL.md
    cross-paper-analysis/
      SKILL.md
    literature-survey/
      SKILL.md
  2-ideation/
    idea-generate/
      SKILL.md
    idea-evaluate/
      SKILL.md
  3-experiment/
    experiment-design/
      SKILL.md
    experiment-track/
      SKILL.md
    result-analysis/
      SKILL.md
  4-writing/
    draft-section/
      SKILL.md
    writing-refine/
      SKILL.md
  5-evolution/
    memory-distill/
      SKILL.md
    agenda-evolve/
      SKILL.md
    memory-retrieve/
      SKILL.md
  6-orchestration/
    autoresearch/
      SKILL.md
      references/
        judge-heuristics.md
```
