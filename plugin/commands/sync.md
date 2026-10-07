---
description: Connect an existing Kissflow app to this folder — choose it, import it live and see what it holds (forms, processes, boards, roles, pages, navigation, permissions) before you understand or change it. Reads the app; changes nothing.
argument-hint: ""
---

Bring an app that already exists in the connected Development sandbox into this folder, so the agents
can explain it and change it safely. `/sync` only reads the app: nothing in Kissflow changes.

1. **Connected?** One cheap probe: `ls .kf-env 2>/dev/null || true`. No `.kf-env` → tell the user to
   run `/connect` first, and stop.
2. **Which app?** Run `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --status`. If it says no app is
   chosen, open the picker with `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --app` and ask the
   user to pick the app to bring in — not **Start a new app**, which is what `/build-app` is for. Show
   the printed link in case the browser didn't open, then run `connect --status` again. Note the app's
   name and id.
3. **Its run.** When `runs/current` already holds this app — its `live-ir.json` has this app's id as
   `app.id`, or its `id-map.json` has it as `app` — keep it: a re-sync refreshes that run. Otherwise
   start one: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs new <app-slug>` (a slug from the app's name,
   e.g. `asset-desk`). It becomes `runs/current`.
4. **Import it live:**
   ```bash
   node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" import --live runs/current --out runs/current/live-ir.json
   ```
   It reads the app's current metadata into `runs/current/live-ir.json`; a large app takes a minute.
5. **Summarize** from `runs/current/live-ir.json`, in short sections:
   - **Forms, processes and boards** (`forms`): each one's name, kind (`flowType`) and number of
     fields; for a process, its steps and the field rules that differ from the default
     (`workflow.steps[].field_permissions`; the default is Editable at the first step, ReadOnly after).
   - **Roles**, with the page each lands on (`landing.page`, a page id in `pages`).
   - **Pages**, and what each shows (`cards`: lists, charts and boards, and the flow behind each).
   - **Navigation** (`nav`): the menus and sub-menus, the page each opens and who sees it (`everyone`,
     or the roles in `visibleTo`), and any other navigation in `nav.others`.
   - **Permissions**: which role reaches which flow (`model`, a flow id in `forms`), at what `level`
     and `scope` — `participating` means only the items the role raises or works on — and whether it
     may raise items (`initiator`).
   - **Custom UI** (`custom_ui`): whether it is on, and its custom components.
   - **What wasn't imported**: every slice whose `_provenance` is not `live` (`live-partial`, `unread`
     or `not-imported`), every entry in `_unread` with its reason (a draft another user has open, a
     list this connection may not read), and every grant in `_lossy` that a spec could not express.
     Lists are not imported, nor are step actors and conditions. Name each one: it is unknown, never
     missing.
6. **Next.** `/author-understand` explains the app and settles what its metadata leaves unclear;
   `/author-reconcile "<the change>"` changes it. An app brought in with `/sync` has no app spec
   (`app-spec.json`): it changes through scoped `patch` operations, and is then imported again (run
   `/sync` once more) so this folder's copy matches the live app. Applying a whole spec to it is
   refused, because that would republish its flows from a spec it never had.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
