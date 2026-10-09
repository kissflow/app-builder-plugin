---
name: author-qa
description: "Runtime QA with a 100% coverage gate — enumerate every testable obligation from the IR (steps×outcomes, permissions, implied denials, scopes, automations, field rules), have kf-runtime-qa write mapped test cases, execute them against the live dev app per role, and report pass/fail with evidence. Failures export as a change-list for /author-refine."
---

This skill is the plugin's `/author-qa` command. It takes: [optional focus — a flow / role / journey | blank = the whole app]. Read the instructions below with these substitutions:

- `${CLAUDE_PLUGIN_ROOT}` is the plugin's root folder: `../../..` from the folder this SKILL.md is in, the one
  holding `bin/kf.mjs`. Always write that folder's absolute path in its place, here and in the agent and
  reference files you read: the literal `${CLAUDE_PLUGIN_ROOT}` fails in every Windows shell (PowerShell, cmd),
  even when the variable is set.
- `$ARGUMENTS` is what the person asked for when they called this skill.
- A command such as `/connect` is this plugin's skill of the same name; the person calls it as `$app-builder:connect`.
- To spawn or run agent `kf-<name>`, read `${CLAUDE_PLUGIN_ROOT}/agents/kf-<name>.md` and carry it out: in a
  sub-agent given those instructions if you can start one, otherwise yourself, in the order the steps give.
- AskUserQuestion: ask the person in chat and wait for the answer.
- `kf.mjs` needs the network, a writable `~/.kissflow` and a local port for its sign-in page. When the sandbox
  stops a `kf.mjs` command (it says writing to the home folder is not allowed), run it again with escalated
  permissions; if that is not possible, show the person the ways to allow it that the command printed.
- When a `kf.mjs` command prints a `➜ …:` line followed by a link, show the person that link right away and ask
  them to open it in their browser: a browser often cannot open from here, and the command waits for that page.


**Runtime QA.** Prove the built app behaves as designed — per role, at runtime, with countable
coverage. Requires a generated app (`runs/current/generated/` apply log) and a connected dev env.

## Do
1. Enumerate the coverage universe (the engine creates `runs/current/qa/`):
   `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" qa-universe runs/current/app-spec.json --out runs/current/qa/universe.json`
   Tell the user the item counts by class — this is what "100%" means for this app.
2. Spawn **kf-runtime-qa**. It reads every artifact, writes `qa/test-plan.json` with each test
   claiming universe ids, and iterates until the coverage gate passes:
   `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" qa-universe runs/current/app-spec.json --check runs/current/qa/test-plan.json`
   (exit 1 = not done; the gate lists uncovered ids).
3. The agent executes the plan against dev as each role (sandbox users; approve seeding them if
   missing), writing `qa/results.json`. Denial tests that PASS access are security findings.
4. Snapshot: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs snapshot "runtime QA"`, then save the QA run:
   `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" publish-qa runs/current --label "<short label>" [--version <stamp>]`
   — it keeps universe/plan/results/report under `runs/current/published/qa/` and adds the run to
   `runs/current/published/qa-runs.json`.
5. Report in chat: the plain-English use-case verdicts FIRST (one line per user journey: works /
   broken / not provable — a BA should understand the whole report from these), then coverage
   (must be 100%), pass/fail by class, security findings, weak-evidence items, and the `[QA-FAIL]`
   change-list block ready for `/author-refine`.

A focus argument scopes the UNIVERSE (that flow/role/journey only) — coverage is still gated at
100% of the scoped universe, and the report must say it was scoped.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
