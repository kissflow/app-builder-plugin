---
name: author-runs
description: "List all authoring runs (one per BRD) with their version counts, and switch the active run. Each BRD lives in its own runs/<slug>/ directory with versioned snapshots."
---

This skill is the plugin's `/author-runs` command. It takes: [<run-slug> to switch to it | blank to list]. Read the instructions below with these substitutions:

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


Manage the runs under `runs/` — each is one BRD's authoring session, versioned independently.

## Do
- **No argument** → `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs list` (shows every run, `*` = current,
  version count each).
- **A run slug given** in `$ARGUMENTS` → `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs use <slug>` to
  make it the current run, then `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs status` to show what you
  switched to.

## Output
The run list (or the switch confirmation + status). Remind the user that all stage commands
(`/author-plan`, `/author-review`, `/author-refine`, `/author-preview`, `/author-generate`) act on the
**current** run, and that `/author-brief <brd>` starts a fresh one.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
