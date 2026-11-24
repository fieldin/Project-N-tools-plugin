# NPD ticket templates

Jira wiki markup (`h2.` headings, `#` numbered lists, `*` bullets, `{{code}}`). Every ticket uses the
**skeleton**; the **per-type part** goes right after Background. Leave no section empty: write
"None" or "n/a: <reason>" so a reader knows it was considered.

## Summary

`<area>: <outcome>` — imperative, under 80 characters, no ticket key, no trailing period.
Examples: `bench: add docs/BENCH.md for FXN_6` · `connectivity: retry LTE attach after PLMN reject`

## Skeleton (all types)

```
h2. Goal
<Why this matters, one or two sentences. The outcome, not the implementation.>

h2. Background
<What the code check found at origin/<base>: current behaviour, file paths with line numbers
({{Connectivity/Connectivity.cpp:120-148}}), the repo rules that apply, quoted with their path.>

<per-type part goes here>

h2. Scope
*In*
* <what changes>
*Out*
* <what explicitly does not change>

h2. Acceptance criteria
# <testable statement; Given/When/Then where it fits>
# ...

h2. How to verify
# <exact steps with the repo's own tools and bench rules, e.g. compile FXN develop, flash-all,
  frag serial test on the bench pty, MQTT only against the dev broker>
or: n/a: <why no bench test, e.g. docs only>

h2. Constraints and risks
* <protocol / flash layout / bootloader / public API impact, anything that makes this not-small>
* <or "None: one repo, small change">

h2. Links
* <related tickets, PRs, Slack threads, docs>
```

## Per-type parts

### Bug
```
h2. Steps to reproduce
# ...

h2. Expected vs actual
*Expected:* ...
*Actual:* ...

h2. Environment
* Device: <e.g. FXN_6>
* Firmware / bootloader: <versions or commit>
* Broker / environment: <DEV (stg) / PROD>

h2. Evidence
* <log lines in {noformat}, Slack links, screenshots>

h2. Severity
<Critical: field devices affected / High: blocks testing / Medium / Low>
```

### Story
```
h2. User story
As a <role>, I want <capability>, so that <benefit>.
```

### Task
```
h2. Definition of done
<One or two lines: what is true when this is finished.>
```

### Epic
No "How to verify" (children carry their own); the skeleton's other sections stay.
```
h2. Problem statement
<What is wrong or missing today, for whom.>

h2. Goals and non-goals
*Goals*
* ...
*Non-goals*
* ...

h2. How success is measured
* ...

h2. Child tickets (in order)
# <KEY>: <summary>  (blocked by: none)
# <KEY>: <summary>  (blocked by: <KEY>)
```
