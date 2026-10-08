---
name: author-status
description: "Show the current run's state — stage, target, whether it's been generated, and the version ladder (v1, v2, …) with each snapshot's change note. Read-only."
---

This skill is the plugin's `/author-status` command. It takes: (operates on runs/current). Read the instructions below with these substitutions:

- `${CLAUDE_PLUGIN_ROOT}` is the plugin's root folder: `../../..` from the folder this SKILL.md is in, the one
  holding `bin/kf.mjs`. Your shell does not set it, so write that folder's absolute path wherever it appears,
  here and in the agent and reference files you read.
- `$ARGUMENTS` is what the person asked for when they called this skill.
- A command such as `/connect` is this plugin's skill of the same name; the person calls it as `$app-builder:connect`.
- To spawn or run agent `kf-<name>`, read `${CLAUDE_PLUGIN_ROOT}/agents/kf-<name>.md` and carry it out: in a
  sub-agent given those instructions if you can start one, otherwise yourself, in the order the steps give.
- AskUserQuestion: ask the person in chat and wait for the answer.
- `kf.mjs` needs the network, a writable `~/.kissflow` and a local port for its sign-in page. When the sandbox
  stops a `kf.mjs` command, run it again with escalated permissions; when no browser opens, give the person the
  address it printed.


Show where the current authoring run stands.

## Do
1. `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs status` — prints `RUN.md` (stage, target, generated?,
   version list with notes).
2. Read `runs/current/open-questions.md` (if present) and surface any still-unresolved questions.
3. Glance at `runs/current/decisions.md` and report the count + how many are `proposed` vs
   `changed-by-user`.

## Output
A compact status card: **run name · stage · target · generated? · N versions** (latest note),
unresolved-question count, decision count. Then the single most useful next command for this stage
(`/author-plan`, `/author-review`, `/author-refine`, `/author-preview`, or `/author-generate`).

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
