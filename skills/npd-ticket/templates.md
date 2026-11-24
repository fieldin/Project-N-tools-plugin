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

## How to verify
1. <exact steps with the repo's own tools and bench rules, e.g. compile FXN develop, flash-all,
   frag serial test on the bench pty, MQTT only against the dev broker>

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
No "How to verify" (children carry their own); the skeleton's other sections stay.
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
