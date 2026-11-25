# NPD ticket templates

**Markdown.** The Jira tools take Markdown (`contentFormat: "markdown"`): `##` headings, `1.` numbered
lists, `-` bullets, backticks for code and paths, fenced blocks for logs. Not Jira wiki markup
(`h2.`, `{{…}}` would show as literal text). Every ticket uses the **skeleton**; the **per-type part**
goes right after Background. Leave no section empty: write "None" or "n/a: <reason>" so a reader knows
it was considered.

## Summary

`<area>: <outcome>` — imperative, under 80 characters, no ticket key, no trailing period.
Examples: `bench: add docs/BENCH.md for FXN_6` · `connectivity: retry LTE attach after PLMN reject`

## Skeleton (all types)

```markdown
## Goal
<Why this matters, one or two sentences. The outcome, not the implementation.>

## Background
<What the code check found at origin/<base>: current behaviour, file paths with line numbers
(`Connectivity/Connectivity.cpp:120-148`), the repo rules that apply, quoted with their path.>

<per-type part goes here>

## Scope
**In**
- <what changes>

**Out**
- <what explicitly does not change>

## Acceptance criteria
1. <testable statement; Given/When/Then where it fits>
2. ...

## Bench verification
Time budget: <N> min of bench time per full run (smoke is ~5 min; add each row's timeout)

| # | Check | How | Pass when | Timeout | On bench |
|---|---|---|---|---|---|
| 1 | Standard suite | `bench-run smoke` | all steps PASS | 8 min | yes |
| 2 | <ticket-specific check> | <`bench-run observe --until '<log line>'`, `bench-run send <url> k=v`, or an exact command> | <exact log line, count or value> | <min> | yes / manual: <who does what> |

Rows marked manual need a person at the bench (ignition, antenna, SIM, power); the agent asks for
them before it starts, not after. On re-runs after a review fix the agent runs row 1 plus the rows
that exercise the changed code.

or: n/a: <why no bench test, e.g. docs only>

## Constraints and risks
- <protocol / flash layout / bootloader / public API impact, anything that makes this not-small>
- <or "None: one repo, small change">

## Links
- <related tickets, PRs, Slack threads, docs>
```

## Per-type parts

### Bug
````markdown
## Steps to reproduce
1. ...

## Expected vs actual
**Expected:** ...
**Actual:** ...

## Environment
- Device: <e.g. FXN_6>
- Firmware / bootloader: <versions or commit>
- Broker / environment: <DEV (stg) / PROD>

## Evidence
```
<log lines>
```
- <Slack links, screenshots>

## Severity
<Critical: field devices affected / High: blocks testing / Medium / Low>
````

### Story
```markdown
## User story
As a <role>, I want <capability>, so that <benefit>.
```

### Task
```markdown
## Definition of done
<One or two lines: what is true when this is finished.>
```

### Epic
No "Bench verification" (children carry their own); the skeleton's other sections stay.
```markdown
## Problem statement
<What is wrong or missing today, for whom.>

## Goals and non-goals
**Goals**
- ...

**Non-goals**
- ...

## How success is measured
- ...

## Child tickets (in order)
1. <KEY>: <summary> (blocked by: none)
2. <KEY>: <summary> (blocked by: <KEY>)
```
