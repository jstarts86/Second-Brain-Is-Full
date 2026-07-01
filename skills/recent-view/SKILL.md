---
name: recent-view
description: >
  Scan the current week, daily notes, and recent vault activity to give an
  inline overview of what was done and what's planned. Triggers:
  EN: "recent view", "what have I been doing", "recent activity",
  "what did I do this week", "catch me up", "what's new", "week review",
  "this week so far"
  IT: "vista recente", "cosa ho fatto", "attività recente", "aggiornami",
  "cosa ho fatto questa settimana"
  FR: "vue récente", "quoi de neuf", "activité récente",
  "qu'est-ce que j'ai fait cette semaine"
  ES: "vista reciente", "qué he estado haciendo", "actividad reciente",
  "qué hice esta semana"
  DE: "aktuelle Ansicht", "was habe ich gemacht", "aktuelle Aktivität",
  "was habe ich diese Woche gemacht"
  PT: "vista recente", "o que tenho feito", "atividade recente",
  "o que fiz esta semana"
---

## Vault Path Resolution

Read `Meta/vault-map.md` (always this literal path) to resolve folder paths. Parse the YAML frontmatter: each key is a role, each value is the actual folder path. Substitute **only** the vault-role tokens listed in the table below — do NOT substitute other `{{...}}` patterns (like `{{date}}`, `{{Name}}`, `{{YYYY}}`, `{{N}}`, `{{ISO timestamp}}`, `{{current week}}`, etc.), which are template placeholders.

If vault-map.md is absent: warn the user once — "No vault-map.md found, using default paths" — then use these defaults:

| Token | Default |
|-------|---------|
| `{{daily}}` | `07-Daily` |
| `{{weekly}}` | `07-Weekly` |
| `{{meta}}` | `Meta` |

If vault-map.md is present but a role is missing: warn the user — "vault-map.md does not define [role]. What folder should I use?" — and wait for their answer before proceeding.

---

# Recent View — Weekly + Daily Activity Overview

Always respond to the user in their language. Match the language the user writes in.

Scan the current weekly note, the previous week's note, the last 7 daily notes, and recently modified files across the vault to produce a concise inline overview of what was done and what's planned.

---

## User Profile

Before processing, read `{{meta}}/user-profile.md` to understand the user's preferences, active projects, and context.

---

## Procedure

### Phase 1: Weekly scan

1. Determine current ISO week (`gggg-Www`).
2. Read `{{weekly}}/{{current-week}}.md` — if absent, read the most recent weekly note.
3. Also read the previous week's note for continuity.
4. Extract:
   - **This Week's Priorities** (the 3 items under `## This Week's Priorities`)
   - **Weekly Goals** — the `Done?` column: count checked vs unchecked
   - **Reflection** — from the previous week's `## Reflection (last week)` section
   - **Next Week Thoughts** — from the current week's `## Next Week Thoughts` section
   - **Daily Log** — which days are marked Done? and any notes

### Phase 2: Daily scan

1. List files in `{{daily}}/` matching the last 7 calendar dates.
2. Read each daily note.
3. Extract:
   - **Three Priorities Today** — checkboxes, note which are checked
   - **Time-block tasks** — any checked or unchecked items in the schedule
   - **Inbox Triage** — any `## Inbox Triage` section showing notes processed
   - **Notable links** — links to meetings, projects, or specific notes

### Phase 3: Vault activity scan

1. Find files modified in the last 7 days across the vault (excluding `{{daily}}/` and `{{weekly}}/` which are already covered).
2. Group results by area/project:
   - `{{projects}}/` — group by project folder
   - `{{areas}}/` — group by area folder
   - `{{meetings}}/` — list as "Meetings"
   - `{{inbox}}/` — count remaining
   - Other — group by top-level folder
3. For each group, count the files and note any with significant changes.

---

## Output Format

Generate an **inline-only** report (do NOT save to vault) using this structure:

```markdown
## Recent View — {{current week label}}

### This Week's Plan
- Priority 1: {{from weekly note, or "not set"}}
- Priority 2: {{...}}
- Priority 3: {{...}}
- Weekly goals: {{X}}/{{N}} marked done

### What Got Done (last 7 days)
{{Summarize checked priorities from daily notes}}
{{Summarize weekly goal progress}}
{{Notable completed items}}

### Vault Activity (7 days)
| Area/Project | Files | Notes |
|-------------|-------|-------|
| {{name}} | {{N}} | {{brief summary of notable changes}} |

{{If an area has 0 recent files, omit it from the table.}}

### What's Pending
{{Unchecked priorities from daily notes}}
{{Unchecked weekly goals}}
{{Items from "Next Week Thoughts"}}

### Looking Ahead
{{Next week thoughts from current weekly note, or "none recorded"}}
```

**If the current week's note does not exist yet**: report "No weekly note for this week — start with a weekly plan to track goals."

**If no daily notes exist for the last 7 days**: check the last 14 days. If still none, report "No recent daily notes found."

---

## Limitations

- The skill does NOT parse Obsidian Tasks plugin queries (```tasks blocks). It reads rendered markdown content only.
- File modification times are from the filesystem, which may not reflect Obsidian internal edits if the vault wasn't saved.
- Weekly goals use the `Done?` column — marks like `X`, `x`, `✓`, or explicit text are counted as done.

---

## Inter-Agent Coordination

> **You do NOT communicate directly with other agents. The dispatcher handles all orchestration.**

When you detect work that another agent should handle, include a `### Suggested next agent` section at the end of your output. The dispatcher reads this and decides whether to chain the next agent.

### When to suggest another agent

- **Scribe** — if you find daily notes missing priorities or weekly notes with empty reflection sections, suggest the user capture more regularly
- **Architect** — if the weekly or daily folders don't exist, report the gap
- **Sorter** — if the inbox has accumulated notes during the period, suggest triage

### Output format for suggestions

```markdown
### Suggested next agent
- **Agent**: sorter
- **Reason**: {{N}} notes in {{inbox}}/ were created during the reviewed period
- **Context**: Triage to clear them out.
```

For the full orchestration protocol, see `.opencode/references/agent-orchestration.md`.
For the agent registry, see `.opencode/references/agents-registry.md`.

### When to suggest a new agent

If you detect that the user needs functionality that NO existing agent provides, include a `### Suggested new agent` section in your output.

```markdown
### Suggested new agent
- **Need**: {what capability is missing}
- **Reason**: {why no existing agent can handle this}
- **Suggested role**: {brief description of what the new agent would do}
```
