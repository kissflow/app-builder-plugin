---
description: Connect this folder to a Kissflow account — with your own access key, or by signing in in the browser.
argument-hint: "[your Kissflow account address, e.g. acme.kissflow.com] [--sign-in]"
---

Connect this folder to the user's Kissflow account, or switch it to another one.

1. Take the account address from `$ARGUMENTS`; if it's empty, ask for it (the address they open
   Kissflow at, e.g. `acme.kissflow.com`). **Never ask for a key ID or secret in this chat.**
2. By default connect with an access key. Tell the user where to create one (*profile picture › My
   settings › API authentication › Access keys*), then run:
   ```bash
   node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect <account-address> --access-key
   source .kf-env
   ```
   A local page opens; the user pastes the key ID and secret there and connect finishes by itself.
   If the user asked to sign in instead (`--sign-in` in `$ARGUMENTS`), run the same command without
   `--access-key`: the account's own Kissflow sign-in opens and the user approves access.
   Show the printed link in case the browser didn't open.
3. If the key is refused, the user fixes it on the page (re-copy both parts, or create a new key). If
   sign-in says the account does not offer assistant sign-in, offer the access key instead.
4. Confirm with `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --status`, greet with the account name
   (`$KF_PROJECT_NAME`), and offer to build: `/build-app "<requirement>"`.

5. Then let them choose the app to build in: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --app`
   (same as `/switch-app`).

To forget the credential on this machine: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --sign-out`.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
