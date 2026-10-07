---
description: Switch the Kissflow app this folder builds in — pick from the apps you can edit, or start a new one.
argument-hint: ""
---

Switch the app this folder builds in.

1. The folder must already be connected (`.kf-env` exists). If it isn't, run `/connect` instead and stop.
2. Run:
   ```bash
   node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --app
   source .kf-env
   ```
   A page opens listing the apps the user can edit in this account, with search and each app's
   custom-UI status. They pick one, or **Start a new app**. A new app is created in the production
   account the sandbox belongs to: if that account's key isn't connected yet (a folder connected by
   an earlier version), the same tab goes on to ask for it, and the choice is saved only once they
   click Done there. Show the printed links in case the browser didn't open.
3. Confirm with `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --status` and tell the user which app
   they're building in (`$KF_APP_NAME`) and whether custom UI is on (`$KF_APP_CUSTOM_UI` = 1). If it
   is off and they want a custom React UI, give them the app-settings link connect printed and ask
   them to turn custom UI on there, then run `/switch-app` again so the plugin picks up the change.
4. If connect warns that `runs/current` belongs to the previous app, start a new run with
   `/author-brief` before building — never apply an old run to a different app.
5. An existing app they picked is changed through `/sync` (import it, then `/author-understand` and
   `/author-reconcile`), never by building a spec into it.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
