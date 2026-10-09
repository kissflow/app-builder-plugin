---
name: author-understand
description: "Understand an EXISTING Kissflow app before changing it: import the live app, derive its static model, settle what evidence can, ask only grounded multiple-choice questions for the rest, and write an app-understanding document. Re-run after edits; it asks only about what changed."
---

This skill is the plugin's `/author-understand` command. It takes: [what you want to know or change about the app]. Read the instructions below with these substitutions:

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


Spawn **kf-comprehension** to build a semantic understanding of an app that already exists, before
anyone changes it. The live app is the source of truth.

Pre-req: `/connect` has connected this folder, and `runs/current` is the app's run:
- an app built from a run here is read back through that run (the `id-map.json` its apply wrote);
- an app this folder did not build is **adopted** with `/sync`, which starts a run for it and imports
  it. Run `/sync` first when `runs/current` holds neither.

An adopted app has no app spec (`app-spec.json`). Its `live-ir.json` is everything this folder knows
about it, and its `_provenance` says, slice by slice, what was read live; anything not `live`, and
everything in `_unread`, is unknown — never missing — and you say so whenever it matters. An adopted
app changes through `patch` operations (`/author-reconcile`) and is then imported again; applying a
whole spec, loading sample records and deploying a React UI need an app built here.

1. **Import the live model:**
   `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" import --live runs/current --out runs/current/live-ir.json`
2. **Derive and interrogate** (the agent): build the static model (ER map, workflow, permissions,
   pages, nav); generate questions from gaps, anomalies and semantic voids; settle what the evidence
   can; ask the user **grounded multiple-choice** questions only for the irreducible residue, recording
   each answer with its provenance.
3. **Output**: `runs/current/understanding.json` (the static model, open questions and answers) and an
   app-understanding document, `runs/current/app-understanding.md` (prose, per persona, with the
   resolved meanings).
4. **Continuous**: re-run after edits — it diffs the new import against the recorded understanding
   (drift) and only re-asks about what changed.

Read-only on the app — comprehension never edits or publishes. Hand off to **/author-reconcile** to
change the app.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
