---
name: author-reconcile
description: "Change an EXISTING app safely through the reconciler: import live, plan against the desired spec, mark ownership and drift, gate the change to its scope, dry-run, then apply with risk-tiered approval. Never regenerates and never overwrites a hand-made edit. Small scoped changes — and every change to an app adopted with /sync — go through one patch command."
---

This skill is the plugin's `/author-reconcile` command. It takes: "<the change>". Read the instructions below with these substitutions:

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


Spawn **kf-reconciler** — the front door for every change to an app that already exists. This is a
**reconciler, not a generator**: the live app is the source of truth and we propose the minimal
reviewable diff.

Pre-req: `runs/current` is the app's run — the run that built it (with its desired spec in
`app-spec.json`), or the one `/sync` adopted it into. Run `/author-understand` first when the app's
meaning is unclear.

**An adopted app has no app spec.** It changes only through the FAST PATH `patch` commands below, one
scoped change at a time, and is imported again afterwards (step 1) so `live-ir.json` matches the live
app. Steps 2–6 need a spec this folder applied and do not apply to it; `apply` refuses an adopted app.
Applying a whole spec, loading sample records and deploying a React UI need an app built here.

1. **Import live**: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" import --live runs/current --out runs/current/live-ir.json`
   (for a built app, flows and fields come from the live app and the rest is carried from the run's
   spec and treated as unknown-live; `_provenance` says which).
2. **Three-way plan**: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" plan --base runs/current/versions/<vN>/app-spec.json --desired runs/current/app-spec.json --live runs/current/live-ir.json --out runs/current/reconcile-plan.json`
   — base × desired × live. It separates what the user wants from what a developer changed in the app
   since, and changes nothing.
3. **Ownership & drift**: tag each item `ai-owned` / `dev-owned` / `conflict`. NEVER overwrite a
   `dev-owned` change without itemised approval.
4. **Risk-tier**: `low` (additive) / `medium` (behaviour change) / `high` (destructive or scope
   widening). Stronger approval as risk rises.
5. **Scope gate**: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" validation-gate runs/current/live-ir.json runs/current/app-spec.json --scope "<the items you change>"`
   — it BLOCKS any change outside that scope. Do not apply while it blocks.
6. **Dry-run, then apply on approval**: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" build runs/current/app-spec.json --out runs/current/preview`
   shows the change and writes nothing to Kissflow; after approval,
   `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" apply runs/current/app-spec.json --mode <express|comprehensive>`
   re-publishes only the flows that changed. Never auto-publish; everything lands in the Development
   sandbox.

Output: `runs/current/reconciliation.json` (diff by ownership and risk, conflicts, ordered apply plan,
approval gates).

## FAST PATH — small changes in seconds
For a SCOPED change to a live app (a branch condition, a new field, step or role, a permission tweak,
list options, an access grant), do NOT write a script. Use one patch command. Ids are live ids: an
adopted app's `live-ir.json` carries them (`app.id`, each form's `id`); a built app's are in its
`id-map.json` (`app`, `genToServer`).
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch branch <processId> --app <appId> --after "CFO Sign-off" --on "Capex_Value > 500000" --then "CEO Approval:CEO" --else "Council Approval:Council"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch condition <processId> "CEO Approval" "Capex_Value > 500000"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch add-step <processId> "Council Approval" --actor "Council" --app <appId> --after "CFO Sign-off" --condition "Capex_Value <= 500000"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch add-field <flowId> "Justification" Textarea --required
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch add-field <flowId> "Vendor" Reference --ref <vendorFlowId> --ref-type Form
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch field-type <flowId> "Notes" Textarea
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch step-permission <processId> "IT Provisioning" "Provisioning Notes" Editable
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch list-items <listId> "Low,Medium,High"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch add-role <appId> "CEO"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch grant <flowId> <appId> "CEO" --family process
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch step-actor <processId> "CFO Sign-off" "VP Finance" --app <appId>
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch formula <flowId> "Days to Start" 'DATEDIFF(Start_Date, TODAY(), "day")'
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch rename <flowId> field "Title" "Request Title"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch field-set <flowId> "Notes" Required=false
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch case-members <caseId> <appId>        # boards: fixes "board view not found"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch kanban-view <caseId> "Pipeline"
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" patch parallel <processId> --app <appId> --after "Site Approved" --branch "Warehouse Readiness:Warehouse Head" --branch "Branch Head Hiring:HR"
```
Each verb reads the live draft, applies the change with the platform's rules built in (per-step
permissions for new fields, re-scoped conditions, retried grants) and publishes; `--dry-run` previews
the draft verbs. **Afterwards:** for a built app, mirror the change into `runs/current/app-spec.json`
(a step's `condition`, a new field) so the next apply keeps it; for an adopted app, import it again
(step 1). Structural changes to a built app take the full reconciler run above.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
