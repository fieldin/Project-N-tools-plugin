---
name: npd-ticket
description: >-
  Create or improve NPD Jira tickets (space "N Project Dev Agents") for the Project N dev-crew agents.
  Interviews the person, checks the code at the agents' base branch, recommends an answer for every
  question, previews the full ticket (or an Epic with child tickets) and creates it through the
  person's own Jira connection. Use when the user says "create an NPD ticket", "open a ticket for the
  agents", "file a bug for Project N", "new task for dev-crew", or "improve / refine / fix ticket
  NPD-<n>".
---

# project-n-tools:npd-ticket

NPD tickets are work orders for the dev-crew agents: an agent must be able to carry one out alone,
without guessing. People file them, answer the agent's questions, confirm plans and review PRs.

Files next to this one: `templates.md` (ticket structure), `labels.md` (the only allowed labels),
`repos.md` (repos, base branches, what agents may change). Read all three before starting.

## Rules for every question you ask

- One question at a time, in plain language.
- **Always give your recommended answer and a one-line reason**: `Recommended: <answer>, because
  <reason>.` The person can accept with "yes" / "ok".
- Ask only what the request, the code check and the repo can't answer. Everything else you decide
  and show in the preview, where the person can change it.

## Mode: create

1. **Describe.** Ask the person to describe what they need in their own words (open question, no
   options, no recommendation yet).
2. **Code check** (a few minutes, not an investigation; no bench, no device logs):
   - Identify the repo(s) from `repos.md`. A repo not in the table: warn that the agents won't work
     there and ask whether to continue. A "not yet" repo: say agents won't pick it up yet.
   - Read code **at `origin/<base>`** (commands in `repos.md`), never the local working branch.
   - Read the repo's own rules first: `AGENTS.md`, and the `docs/` notes for the area (for bench
     work in Project_N: `docs/fxn6_session_handoff.md`).
   - Note current behaviour with file paths and line numbers, and every place where the request
     contradicts the repo's rules or code.
3. **Size.** Small = one repo, roughly ≤100 changed lines, no change to the wire protocol, flash
   layout, bootloader or a public API. Not small → propose an **Epic** with child tickets, each
   small, ordered, with "is blocked by" between them. Recommend the split; the person confirms.
4. **Interview.** Contradictions from step 2 come first ("The request says X; the repo says Y.
   Recommended: Y, because …"). Then only the open points, from this list, each with a recommendation:
   - Work type: Bug (something broke), Story (new capability), Task (maintenance, docs, tooling),
     Epic (needs splitting).
   - Per-type details from `templates.md` (Bug: reproduce steps, expected vs actual, environment,
     evidence, severity; Story: the user story; Task: definition of done).
   - Acceptance criteria and the Bench verification table (`templates.md`): for every acceptance
     criterion a row with a `bench-run` profile or exact command, the exact pass condition and a
     timeout. Ask whether each check can run unattended on FXN_6; anything needing a hand at the
     bench (antenna, SIM, power; not ignition: `ignition` automates it) is a **manual** row naming who does it. Recommend a time
     budget from the timeouts.
   - Scope Out (what must not change).
   - Owner (assignee): default the person using the skill; another person only if named (look up
     with `lookupJiraAccountId`).
   - Labels: from `labels.md` only, recommended from the areas the code check touched.
   - Priority: recommend from impact (field devices > bench only > docs).
5. **Preview.** Show every ticket exactly as it will be created: summary, type, owner, labels,
   priority, full description per `templates.md`; for an Epic, the whole tree with order and
   blockers. Ask for approval or edits; apply edits and show it again until approved.
6. **Status.** For each non-Epic ticket ask: `Backlog or Ready for Agent? Recommended: Backlog,
   because nothing starts until someone moves it.` (Ready only if the person is sure the ticket is
   complete and the bench is free.) **Epics always stay in Backlog**; agents never run an Epic.
7. **Create** with the person's Atlassian connector (never any bot token), project `NPD`, always
   with `contentFormat: "markdown"`:
   - Epic first (`createJiraIssue`, type Epic), then each child with `parent` = the epic key.
   - Then the order: `createIssueLink` with type `Blocks`, **`inwardIssue` = the prerequisite (the
     blocker), `outwardIssue` = the ticket that waits for it**.
   - If any create or link call fails, **stop**: set no statuses (everything stays in Backlog), tell
     the person what was created and what failed. A child must never reach Ready without its link.
   - Only then the chosen statuses (`getTransitionsForJiraIssue` + `transitionJiraIssue`).
   - Print each key with its link and its status.

## Mode: improve

Given a key (`NPD-<n>`):

1. Read the ticket and its comments (`getJiraIssue`). If it is in **Ready for Agent**, move it to
   **Backlog right away** and tell the person ("parked in Backlog while we improve it; it goes back to
   Ready at the end"), so an agent can't start on the old text during the interview. Score it against
   `templates.md`: list missing sections, untestable acceptance criteria, acceptance criteria with no
   Bench verification row, rows without a pass condition or timeout, checks that need a person at the
   bench but aren't marked manual, wrong type, labels not in `labels.md`.
2. Run the code check (create step 2) for what the ticket touches.
3. Interview only for the gaps, each question with a recommendation; contradictions first.
4. Show a before / after preview. Apply only after approval.
5. **Re-read the status** (`getJiraIssue`) right before applying; it may have changed during the
   interview. Then apply, depending on that status:
   - **Backlog / To Do:** update description and fields (`editJiraIssue`, `contentFormat:
     "markdown"`). **Never send `assignee`** unless the person approved an owner change in the
     preview. If you parked it from Ready in step 1, move it back to Ready for Agent now.
   - **In Progress / Waiting for Input / In Review / Blocked:** don't rewrite the description; add
     one comment with the suggested additions, so the agent's thread stays intact.
   - **Done:** don't change it; suggest a new ticket instead.
6. **Keep the assignee.** Only suggest a change (e.g. when it's empty); improving a colleague's
   ticket never makes you its owner.

## Tickets written without this skill

The dev-crew orchestrator applies this skill's improve check when it picks a ticket up (Ready for
Agent). A ticket that doesn't follow `templates.md` gets no code: the agent posts the rewritten
ticket and its open questions as a Jira question, and only after the owner replies `approve` does the
bot account (NPD Agents) replace the description. That approved rewrite is the one write the bot makes
to a ticket's fields; people still create and improve tickets with their own Jira connection.

## Never

- Never close, delete or move a ticket past Ready for Agent.
- Never change a ticket's assignee unless the person approved it in the preview.
- Never put an Epic in Ready for Agent.
- Never invent a label, a repo branch or a bench step: take them from the files above and the repo.
- Never file a ticket for someone else without saying who the owner will be in the preview.
