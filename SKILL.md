---
name: client-dashboard
description: Build a static, offline, client-owned data dashboard from a small business's spreadsheet or exports. Use when someone wants a dashboard, scorecard, or "see my numbers in one place" view of their own data and wants to own the result. Runs discovery, writes a layout spec, defines the data contract, generates an Astro + Observable Plot site, QA-checks it, and writes a handoff doc.
---

# Client dashboard builder

DRAFT v0.1. For review before it goes into source control.

## What this skill is for

Turn a client's data into a small static dashboard they fully own. The value is in the discovery, the spec, and the refresh path. The generated code is boring on purpose.

This pattern comes from a worked example: a local-LLM benchmark dashboard (7 views over three flat data files, with a Python ingest script). The example's `dashboard-layout.md` is the model for the spec format below.

## Principles (non-negotiable)

1. **The client owns everything.** Data, code, spec, and handoff doc are delivered as plain files in a folder or repo they control. No accounts, no hosted service of mine, no lock-in.
2. **Boring and inspectable.** Data lives in CSV or JSON they can open in Excel. The site is static HTML/JS. Nothing needs to stay running.
3. **Local and private by default.** No CDN links, no web fonts from third parties, no analytics, no telemetry, no network calls at runtime. It must work from a folder with the wifi off.
4. **Honest scoping.** If a spreadsheet with a chart would serve them better, say so and don't build a dashboard. If their data is too thin or too messy to support the views they want, say that too.
5. **Defined means explained.** Every number on screen must be explainable in one sentence in the layout spec. If I can't explain it, it doesn't ship.
6. **Don't create dependence.** The handoff doc must let them refresh the data without me.

## Fixed target stack

Always generate this, so output is consistent and supportable:

- **Astro** (static output) + **Observable Plot** for charts, installed through npm and bundled. Plain HTML tables for tables.
- Data in `src/data/*.csv` or `*.json` (one file per entity or event type).
- Dashboard definition in `dashboard.spec.md` (human-readable) mirrored by `dashboard.config.json` (what the code reads).
- Ingest and refresh: a single command, `npm run refresh`, that validates new data, rewrites the data files, and rebuilds.
- Output: `dist/` folder of static files, openable from any static host or a local server.

Open decision for the author: ingest language. Default is Node so the client needs one runtime. Python is acceptable if the client already uses it.

## Workflow

### 1. Discovery (ask, don't assume)
Ask these before building anything. Keep it conversational, one or two at a time.
- What decisions will this dashboard help you make, and how often?
- Where does the data come from today (spreadsheet, POS export, booking system, bank export)? Can I see a real sample?
- What is one row? What is one thing you're comparing (products, locations, staff, months, campaigns)?
- What does "good" look like, and what number would worry you?
- Which records should be flagged or left out of averages? (This is the equivalent of the reduced-sample marker in the example. Examples: refunds, test orders, partial months, one-off events.)
- Who else will look at it, and on what device?
- Who will update it, how often, and how comfortable are they with a terminal?
- Anything sensitive in the data (customer names, health, payments) that should be stripped or aggregated before it enters the dashboard?

Stop and report if the sample data contradicts what they described.

### 2. Layout spec
Write `dashboard.spec.md` using the template below. Get the client's sign-off on the spec before generating code. Changing a spec is cheap, changing a built dashboard is not.

### 3. Data contract
Define each data file: name, columns, types, allowed values, and which column uniquely identifies a row. Document how rows are matched on rerun so the ingest is idempotent (safe to rerun, never duplicates, never loses rows).

### 4. Ingest
Build the ingest step to these conventions (taken from the example script):
- Idempotent. Running the same input twice changes nothing.
- Never silently drop data. Anything skipped or ambiguous is logged.
- Exit non-zero with a clear message when it needs input from a person (missing date, unknown column, unreadable file).
- Print a short log of what was added, updated, and skipped, plus final row counts.
- Never overwrites the client's source exports. Work on copies.

### 5. Build
Generate the Astro project from the spec. One component per view. Colors, labels, and markers come from config, not hard-coded.

### 6. QA (all must pass before handoff)
- [ ] Numbers on screen match a hand calculation from the source for at least three cells or points per view.
- [ ] Averages and rankings follow the spec's exclusion rule exactly.
- [ ] Missing values render as a dash or blank, never as 0 or NaN.
- [ ] Grep the built output: no external URLs, no CDN scripts, no remote fonts, no analytics.
- [ ] Works offline (load `dist/` with the network off).
- [ ] Usable at phone width, readable in grayscale (don't rely on color alone), keyboard-navigable.
- [ ] Empty dataset, one-row dataset, and very long names don't break layout.
- [ ] `npm run refresh` works from a clean checkout following only the handoff doc.

### 7. Handoff
Write `HANDOFF.md` in plain language:
- What this is and what each view means (copy from the spec).
- How to update the data, step by step, with the exact command.
- What the ingest log means and what to do when it stops and asks for something.
- How to change a color, label, or add a column (config only).
- Where everything lives, and how to back it up.
- What I can and can't help with later, and that they're free to hand it to anyone else.

## View catalog

Pick only views that serve the decisions named in discovery. Each is a pattern from the example.

| View | Use for | Notes |
|---|---|---|
| **Summary line** | One-sentence headline | Computed, with its definition in the spec |
| **Heat table** | Entities x metrics | Cell = latest valid value, darker = higher, dash = missing, average column honors exclusion rule |
| **Scatter** | Two measures per entity | Filled vs hollow point to show data quality; list entities that lack one axis under the chart |
| **Ranked bar** | One metric at a time | Selector for the metric, longest first, color by group, faded = flagged |
| **Plain table** | Latest facts per entity | Sortable, with a "when" column |
| **Breakdown matrix** | Entity x sub-category | Outline each entity's weakest cell |
| **Full log** | Every record, newest first | Always include; it's how people trust the summary |
| **Trend line** | A measure over time | Add only if there are enough points to mean something |

## Spec template (`dashboard.spec.md`)

```
# <Client>: <dashboard name>

<One or two sentences: who it's for, what data, how it's collected, how often it's updated.>
<Marker legend, e.g. † marks a record flagged as X: shown, but left out of averages.>

## Header
- Title:
- Takeaway line: <exact definition>

## 1. <View name> (<view type>)
- Row / Point / Bar: <what one is>
- Columns / Axes: <what they are>
- Cell / Value: <exactly how it's computed, which record is used, what's excluded>
- Color / Marker rules:
- Missing values:

(repeat per view)

## Data files
- <file>: <one row is...>, key = <columns>, columns = ...

## Exclusion / flag rule
<The client's own definition, in their words.>
```

## Things to refuse or push back on
- Dashboards built to mislead (cherry-picked ranges, truncated axes without labeling, hiding flagged data).
- Pulling in client data from sources they don't control without their say-so.
- Storing personal data on screen that isn't needed for the decision.
- Features that require a running server or a third-party account. Offer the static equivalent or tell them it's out of scope.

## Open questions for the author
1. Ingest language: Node (one runtime) or Python (what exists today)?
2. Charts: Observable Plot only, or allow ECharts when a client wants more polished interactivity?
3. Pricing and support terms belong in a separate engagement doc, not in this skill. Confirm.
4. Should the skill include a second, fake small-business dataset as a built-in regression test (same code, different data)?