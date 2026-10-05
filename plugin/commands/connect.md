---
description: Connect this folder to your account's Kissflow Development sandbox with your own access key — run it first, and again to switch sandboxes. Fetches the build engine on first use, picks the app to build in, and seeds the agent memory.
argument-hint: "[your Development sandbox address]"
---

Run this in the folder you build from: first to connect, again to switch to another sandbox. The
plugin ships its commands, its specialist agents, the reference playbooks and a small launcher
(`${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs`); the build engine is fetched once per version into the user's home
folder. Nothing is copied into this workspace except `.kf-env` (which sandbox this folder builds in)
and `MEMORY.md` (the agents' memory).

**Break the silence FIRST.** The first engine call is the coldest, most silent wait a user feels.
Your VERY FIRST output — before any probe or `node` call — is a warm line: *"⚙️ Warming up — checking
the build engine and your Kissflow connection (one-time, a minute or so)…"*. Then narrate each step as
it finishes: *"✓ Engine ready."* · *"✓ Connected to <account> as <email>."* · *"✓ Building in <app>."*

## 0. Fast path (ZERO exploration)
Decide with ONE cheap probe (it never errors):
```bash
ls .kf-env 2>/dev/null || true
```
- `.kf-env` present and `$ARGUMENTS` empty → `source .kf-env`, run
  `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --status`, say *"Already connected to
  **$KF_PROJECT_NAME** — `/build-app "<your requirement>"` when you're ready, or `/connect <address>`
  to switch sandboxes."* and **stop**.
- `$ARGUMENTS` has an address → go straight to step 2 with it. Connecting replaces `.kf-env` in
  place; an env pointing at a different sandbox is exactly what it replaces, not a conflict.

**Do NOT explore.** Never read `bin/kf.mjs`, run it with `--help`, or inspect its structure — the
invocations on this page are its entire surface.

## 1. Engine (first run only)
Prerequisites: **Node 18+** (Claude Code already needs it); **python3** only for native pages;
**npm** with registry access only for custom React UIs (the first React build sets up its tools once,
about 2 minutes). Tell the user: *"The first run downloads the build engine for this platform
(~100 MB, once per version) into your home folder."* Then:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" --version
```
It prints the engine version once any download finishes; on a re-run it's instant. A checksum
failure says so — just re-run it.

## 2. Connect to the Development sandbox
Take the address from `$ARGUMENTS`; if it's empty, ask for it. It must be the account's
**Development sandbox** address — the address they open the sandbox at, not the production account.
The App Builder builds **only in a Development sandbox**; connect refuses a production account or a
Test sandbox. Tell the user in one line: *"You'll paste an access key you create in that sandbox
(profile picture › My settings › API authentication › Access keys) into a page that opens next."*
Then:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect <sandbox-address>
source .kf-env
```
Connect prints a direct link to the sandbox's API authentication settings, then opens a local page
where the user pastes the **access key ID** and **access key secret**; it checks the key with
Kissflow and finishes on its own. Show both printed links in case the browser didn't open; the page
waits up to 15 minutes.
- **Never ask the user to paste a key ID or secret into this chat**, and never put one in a command,
  a file or an environment variable. If they paste one here anyway, don't repeat it: tell them to
  delete that key in Kissflow, create a new one, and paste it into the page.
- If the page says Kissflow **didn't accept the key**, the user fixes it there (re-copy both parts, or
  create a new key). If API access is off for the account, an admin must turn it on.
- A key that belongs to a **service account** is refused: it must be the user's own key.
- If connect says the address is a **production account** or a **Test sandbox**, say so in one line
  and ask for the Development sandbox's address. Never suggest a way around it.
- Access keys are the only way to connect. Don't offer any other sign-in method.

Everything the agents build is done **as this person**, with exactly their Kissflow permissions. The
key stays in the user's home folder, never in this workspace. `.kf-env` holds only which sandbox and
method this folder uses, who connected and the account name — nothing secret, but never commit it.
`connect --status` shows the sandbox, person and key; `connect --sign-out` forgets the key on this
machine.

**THE ACCOUNT NAME IS THE WORKING CONTEXT.** `.kf-env` carries `KF_PROJECT_NAME` (the Kissflow
account's name). Greet with it ("Connected to **Acme** — what should the app do?"), interpret every
subsequent ask in its context, and prefix clarifying questions with it. Never ask which account this
is about — you already know.

Everything else lives in this folder: the design graph under `runs/<app>/ir-graph/`, each build's
saved versions under `runs/<app>/published/`, and the agents' memory in `MEMORY.md` + `MEMORY-LOCAL.md`.
Agents write new lessons with `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" memory remember "<lesson>" --app <appId>`.

## 3. Choose the app to build in
Right after connecting, open the app picker:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --app
source .kf-env
```
A page lists the apps in this sandbox the user can **edit**, with search and whether **custom UI** is
on for each. They pick one, or **Start a new app** (the next build creates one). Show the printed link
in case the browser didn't open; it waits up to 15 minutes. They can switch apps any time with
`/switch-app`. If custom UI is off for the chosen app and they want a custom React UI, give them the
app-settings link connect prints and ask them to turn custom UI on, then run `/switch-app` again.
If connect warns that `runs/current` belongs to the previous sandbox or app, start a new run with
`/author-brief` before building.

## 4. Seed the agent memory
```bash
[ -f MEMORY.md ] || cp "${CLAUDE_PLUGIN_ROOT}/MEMORY.md" MEMORY.md
```
`MEMORY.md` is the agents' auto-evolving memory — the user's to grow; an existing one is never replaced.

## 5. Go
Greet with the account name and offer `/build-app "<your requirement>"` — a full app top-down — or the
staged loop starting with `/author-brief`. Builds always dry-run first; nothing is published to
production.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
