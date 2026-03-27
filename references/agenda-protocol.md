# Agenda Protocol

This document defines the format and governance rules for `agenda.md` — the living research agenda that tracks active, paused, and abandoned directions. The agenda is the primary interface between Supervisor guidance and Researcher autonomous action.

---

## File Format

The agenda lives at `Workbench/agenda.md`. Below is the canonical template.

```markdown
---
last_updated: YYYY-MM-DD
updated_by: supervisor / <skill-name>
---

## Mission

Research mission. Researcher may propose evolution; Supervisor may edit directly.
One paragraph describing the long-term research mission and primary scientific question.

---

## Active Directions

### [Direction Name]

- **priority**: high / medium / low
- **status**: exploring / validating / consolidating
- **origin**: supervisor-assigned / researcher-discovered / paper-inspired
- **hypothesis**: One-sentence falsifiable hypothesis
- **evidence**: [[Papers/xxx]], [[Experiments/xxx]] — or "none yet"
- **next_action**: Concrete next step (e.g., "Run experiment X", "Read [[Papers/yyy]]")
- **confidence**: 0.0–1.0

---

## Paused Directions

### [Direction Name]

- **priority**: high / medium / low
- **status**: paused
- **origin**: supervisor-assigned / researcher-discovered / paper-inspired
- **hypothesis**: One-sentence falsifiable hypothesis
- **evidence**: [[Papers/xxx]], [[Experiments/xxx]]
- **pause_reason**: Why this was paused
- **resume_condition**: What would trigger resuming this direction
- **confidence**: 0.0–1.0

---

## Abandoned Directions

### [Direction Name]

- **abandoned_on**: YYYY-MM-DD
- **original_hypothesis**: What was believed when started
- **reason**: Why this was abandoned
- **lesson**: One-sentence takeaway
- **memory_ref**: [[Workbench/memory/failed-directions.md#heading]]

---

## Discussion Topics

<!-- Items Researcher wants to discuss with Supervisor at next check-in. These do NOT block work — Researcher continues independently. -->

### [Topic title] — [YYYY-MM-DD]

- **raised_by**: <skill-name> / researcher
- **context**: What prompted this topic
- **question**: What Researcher wants Supervisor's input on
- **related_direction**: Which direction this relates to
```

---

## Active Direction Fields

| Field | Type | Description |
|---|---|---|
| `priority` | enum | `high` / `medium` / `low` — set by Researcher or Supervisor |
| `status` | enum | Current phase of the direction (see values below) |
| `origin` | enum | How this direction entered the agenda |
| `hypothesis` | string | Falsifiable one-sentence prediction |
| `evidence` | wikilinks | Links to supporting notes; `"none yet"` if just created |
| `next_action` | string | Specific, executable next step |
| `confidence` | float | Researcher-estimated confidence in the hypothesis, 0.0–1.0 |

**`status` values:**

- `exploring` — early stage; hypothesis is being investigated but not yet tested
- `validating` — evidence exists; running experiments or reading papers to confirm or refute
- `consolidating` — direction is well-supported; writing up findings or integrating into Domain Map

**`origin` values:**

- `supervisor-assigned` — Supervisor explicitly added this direction
- `researcher-discovered` — Researcher proposed this direction based on memory or literature
- `paper-inspired` — this direction emerged from a specific paper or set of papers

---

## Researcher Permissions

Researcher has full autonomy over the agenda. Like a PhD student, Researcher drives the daily research independently.

| Action | Researcher can do? | Notes |
|---|---|---|
| Add a new direction (any priority) | Yes | Should be supported by at least one piece of evidence or reasoned argument |
| Change status (exploring → validating → consolidating) | Yes | Based on evidence state |
| Abandon a direction | Yes | Must write to `failed-directions.md` and log the lesson |
| Update `next_action`, `confidence`, `evidence` | Yes | Anytime |
| Move direction to Paused | Yes | Must populate `pause_reason` and `resume_condition` |
| Evolve Mission | Yes | May propose changes; Supervisor can always override |
| Merge duplicate directions | Yes | Log the merge in `evolution/changelog.md` |

### Copilot Mode

When a skill runs in `copilot` role (Supervisor is online and giving instructions), Researcher executes the Supervisor's request directly. No need to route through Discussion Topics.

---

## Supervisor Overrides

Supervisor edits to `agenda.md` take effect immediately and unconditionally. There is no approval workflow for Supervisor changes. When a Supervisor edits the agenda:

- The `last_updated` and `updated_by` frontmatter fields should be updated to reflect the change.
- Researcher skills that run subsequently will read the current state of the file as authoritative.
- Supervisor edits take precedence over any Researcher-proposed state.

---

## Sync with Memory

When a direction is moved to `Abandoned Directions`, the following three steps must all be completed (typically by the responsible skill):

1. **Write to `failed-directions.md`**: Create a new entry in `Workbench/memory/failed-directions.md` capturing the original hypothesis, evidence against, lesson, and related directions. This is done via the IVE (Insight, Validation, Evolution) pattern.

2. **Move to Abandoned section**: Transfer the direction from `Active Directions` or `Paused Directions` into `Abandoned Directions`, populating `abandoned_on`, `original_hypothesis`, `reason`, `lesson`, and a `memory_ref` wikilink pointing to the new `failed-directions.md` entry.

3. **Log in changelog**: Append an entry to `Workbench/evolution/changelog.md`:

   ```markdown
   ### [YYYY-MM-DD] Direction abandoned: <direction name>
   - **reason**: <brief reason>
   - **memory_ref**: [[Workbench/memory/failed-directions.md#heading]]
   - **logged_by**: <skill-name or "researcher">
   ```

These three steps must be atomic — if any step fails, the skill should halt and report the failure rather than leaving the agenda and memory in an inconsistent state.

---

## Agenda Integrity Rules

- Every direction in `Active Directions` must have a non-empty `next_action`. If a direction's next action is unclear, it should be moved to `Paused Directions` until a concrete step is identified.
- `confidence` values must be grounded in evidence. An `exploring` direction with no evidence should start at 0.1–0.2. A direction with multiple confirming papers may reach 0.7–0.8. Reaching 0.9+ requires experimental confirmation.
- Discussion Topics do not block Researcher work. They are informational — things Researcher wants to discuss at the next Supervisor check-in.
- Duplicate directions (same hypothesis under different names) should be merged. Researcher merges autonomously and logs to `evolution/changelog.md`.
