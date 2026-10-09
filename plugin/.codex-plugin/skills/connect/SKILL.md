---
name: connect
description: "Connect this folder to your Kissflow production account and its Development sandbox with your own access keys — run it first, and again to switch accounts. Fetches the build engine on first use, picks the app to build in, and seeds the agent memory."
---

This skill is the plugin's `/connect` command. It takes: [your production account's or Development sandbox's address] | --production. Read the instructions below with these substitutions:

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


Run this in the folder you build from: first to connect, again to switch to another account. The
plugin ships its commands, its specialist agents, the reference playbooks and a small launcher
(`${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs`); the build engine is fetched once per version into the user's home
folder. Nothing is copied into this workspace except `.kf-env` (which sandbox this folder builds in)
and `MEMORY.md` (the agents' memory).

**Break the silence FIRST.** The first engine call is the coldest, most silent wait a user feels.
Your VERY FIRST output — before any probe or `node` call — is a warm line: *"⚙️ Warming up — checking
the build engine and your Kissflow connection (one-time, a minute or so)…"*. Then narrate each step as
it finishes: *"✓ Engine ready."* · *"✓ Connected to <account> as <email>."* · *"✓ Building in <app>."*
When connect completes it draws the Kissflow logo: show it to the user exactly as printed, in a plain
code block, then say they are connected — the production account and its Development sandbox.

## 0. Fast path (ZERO exploration)
Decide with ONE probe (it fetches the engine on first use; "not connected" is an answer, not an error):
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --status
```
- It says `connected:` and `$ARGUMENTS` is empty → say *"Already connected to
  **<the account it names>** — `/build-app "<your requirement>"` when you're ready, or `/connect <address>`
  to switch sandboxes."* and **stop**.
- `$ARGUMENTS` has an address → go straight to step 2 with it. Connecting replaces `.kf-env` in
  place; an env pointing at a different account is exactly what it replaces, not a conflict.
- `$ARGUMENTS` is `--production` → go straight to step 2b.

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

## 2. Connect the production account and its Development sandbox
Take the address from `$ARGUMENTS`; if it's empty, ask for it. Either address works: the
**production account's** (the one the user normally opens Kissflow at) or its **Development
sandbox's**. New apps are created in production; everything else is built in the Development sandbox.
Tell the user in one line: *"You'll paste two access keys into a page that opens next: one from the
account you gave, then one from its production account or Development sandbox (profile picture ›
My settings › API authentication › Access keys)."* Then:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect <address>
```
Connect prints a direct link to that account's API authentication settings, then opens one local
page. **Step 1:** the user pastes that account's **access key ID** and **access key secret**.
**Step 2:** the page asks for the other account's key, with a link to its settings: the Development
sandbox's when they gave the production address (they choose if it has several), or the production
account's when they gave the sandbox's. When both are accepted, the page shows both as connected;
they click **Done** and connect finishes. Show the printed links in case the browser didn't open;
the page waits up to 15 minutes, and nothing is saved before Done.
- **Never ask the user to paste a key ID or secret into this chat**, and never put one in a command,
  a file or an environment variable. If they paste one here anyway, don't repeat it: tell them to
  delete that key in Kissflow, create a new one, and paste it into the page.
- If the page says Kissflow **didn't accept the key**, the user fixes it there (re-copy both parts, or
  create a new key). If API access is off for the account, an admin must turn it on.
- A key that belongs to a **service account** is refused: it must be the user's own key.
- If the page says the production account **has no Development sandbox**, the user (or an admin)
  creates one in Kissflow, then runs `/connect` again. Test sandboxes are never used.
- If connect says the two accounts **don't belong together**, an account **is a sandbox, not a
  production account**, or the sandbox **is a Test sandbox**, say so in one line and ask for the
  production account's or its Development sandbox's address. Never suggest a way around it.
- Access keys are the only way to connect. Don't offer any other sign-in method.

Everything the agents build is done **as this person**, with exactly their Kissflow permissions. The
keys stay in the user's home folder, never in this workspace. `.kf-env` holds only the sandbox builds
go to, the linked production account, the method, who connected and the account name — nothing
secret, but never commit it. `connect --status` shows both accounts, the person and both keys;
`connect --sign-out` forgets both keys on this machine.

**THE ACCOUNT NAME IS THE WORKING CONTEXT.** `.kf-env` carries `KF_PROJECT_NAME` (the Kissflow
account's name). Greet with it ("Connected to **Acme** — what should the app do?"), interpret every
subsequent ask in its context, and prefix clarifying questions with it. Never ask which account this
is about — you already know.

Everything else lives in this folder: the design graph under `runs/<app>/ir-graph/`, each build's
saved versions under `runs/<app>/published/`, and the agents' memory in `MEMORY.md` + `MEMORY-LOCAL.md`.
Agents write new lessons with `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" memory remember "<lesson>" --app <appId>`.

## 2b. A folder connected by an earlier version
A folder connected by an earlier version of the plugin has only its Development sandbox, and a **new**
app needs the production account's key: Kissflow creates the app there and copies it into the
sandbox, where everything else is built. **Start a new app** in the app picker (step 3) asks for it,
or directly:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --production
```
The production address comes from the sandbox itself, so there is nothing to type. The same page
opens at step 1 only, labelled **Production — only new apps are created here**; the user pastes the
production key and clicks **Done**. The same rules apply: never in this chat.
- If connect says the account **is not the production account the sandbox belongs to**, or that it
  **is a sandbox**, say so in one line. Never suggest a way around it.
- If `apply` says a new app needs the production key, run this step, then apply again.

## 3. Choose the app to build in
Right after connecting, open the app picker:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --app
```
A page lists the apps in this sandbox the user can **edit**, with search and whether **custom UI** is
on for each. They pick one, or **Start a new app** (the next build creates it in production). In a
folder connected by an earlier version, the same tab goes on to the production step first (step 2b);
the choice is saved only once they click Done there. Show the printed links in case the browser
didn't open; each page waits up to 15 minutes. They can switch apps any time with
`/switch-app`. An existing app is brought in with `/sync` to understand and change it; a new build
needs **Start a new app**. If custom UI is off for the chosen app and they want a custom React UI, give them the
app-settings link connect prints and ask them to turn custom UI on, then run `/switch-app` again.
If connect warns that `runs/current` belongs to the previous sandbox or app, start a new run with
`/author-brief` before building.

## 4. Seed the agent memory
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" memory seed "${CLAUDE_PLUGIN_ROOT}/MEMORY.md"
```
`MEMORY.md` is the agents' auto-evolving memory — the user's to grow; an existing one is never replaced.

## 5. Go
Greet with the account name and offer `/build-app "<your requirement>"` — a full app top-down — or the
staged loop starting with `/author-brief`. Builds always dry-run first. A new app is created in
production as an empty app; everything in it is built in the sandbox, and nothing else is published
to production.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
