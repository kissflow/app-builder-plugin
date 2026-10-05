---
description: One-time setup — check the prerequisites, fetch the build engine for this machine, connect this folder to your Kissflow account (with an access key, or by signing in), and seed the agent memory.
argument-hint: "[your Kissflow account address, e.g. acme.kissflow.com] (run once in the folder you build from)"
---

Run this **once** in the folder you want to build from. The plugin ships its commands, its specialist
agents, the reference playbooks and a small launcher (`${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs`); the build
engine itself is fetched once per version into your home folder. Nothing is copied into this
workspace except `.kf-env` (which account this folder builds in) and `MEMORY.md` (the agents' memory).

**Break the silence FIRST.** Setup + the first engine call are the coldest, most silent wait a user
feels. Your VERY FIRST output — before any probe or `node` call — must be a warm line so the user
isn't staring at nothing: *"⚙️ Warming up — checking the build engine and your Kissflow connection
(one-time, a minute or so)…"*. Then narrate each step as it finishes: *"✓ Engine ready."* ·
*"✓ Connected to <account> as <email>."* · *"✓ Building in <app>."* Never run the slow steps before emitting that first line.

## 0. FAST PATH — re-runs and account switches (ZERO exploration)
Setup is **idempotent** and re-running it to SWITCH accounts is normal. Decide everything with ONE
cheap probe (it never errors):
```bash
ls .kf-env 2>/dev/null || true
```
- `.kf-env` present and `$ARGUMENTS` empty → `source .kf-env`, say *"Already connected to
  **$KF_PROJECT_NAME** — `/build-app "<your requirement>"` when you're ready."* and **stop**.
- `$ARGUMENTS` has an account address → run step 3 with it **immediately**. It overwrites `.kf-env`
  in place — an env pointing at a DIFFERENT account is not a conflict to investigate, it's exactly
  what connect replaces.

**Do NOT explore.** Never read `bin/kf.mjs`, run it with `--help`, or inspect its structure — the
invocations on this page are its entire surface for setup. The whole fast path is one probe and
should take seconds; anything beyond that is wasted user-visible time.

## 1. Prerequisites
- **Node 18+** (`node --version`) — Claude Code already needs it, so this normally passes.
- **python3** on PATH (`python3 --version`) — needed **only** for native page publishing. Data
  models, workflows, roles and permissions build fine **without** it; only native pages need it.
- **macOS, Linux or Windows x64** — the engine ships as a sealed build per platform.
- **npm with access to the npm registry** — only for custom React UIs. The first React build sets up
  the build tools once (about 2 minutes, ~400 MB in the user's home folder); native pages don't need it.

## 2. Engine check
Tell the user first: *"The first run downloads the build engine for this platform (~100 MB, once per
version) into your home folder."* Then:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" --version
```
It prints the engine version once the download (if any) finishes. On a re-run it's instant. A
checksum failure says so — just re-run the command.

## 3. Connect to your Kissflow account
Ask the user for their **Kissflow account address** if `$ARGUMENTS` doesn't already hold one (the
address they open Kissflow at, e.g. `acme.kissflow.com`). There are two ways to connect; use the
first unless the user asks for the other.

**A. Access key — the default.** Tell the user in one line where to create one: *"In Kissflow, click
your profile picture › My settings › API authentication › Access keys, and create a key. You'll paste
it into a page that opens next."* Then:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect <account-address> --access-key
source .kf-env
```
A local page opens in the browser. The user pastes the **access key ID** and **access key secret**
there; connect checks the key with Kissflow and finishes on its own. Surface the printed link in case
the browser didn't open; it waits up to 15 minutes.
- **Never ask the user to paste a key ID or secret into this chat**, and never put one in a command,
  a file or an environment variable. If they paste one here anyway, don't repeat it: tell them to
  delete that key in Kissflow, create a new one, and paste it into the page.
- If the page says Kissflow **didn't accept the key**, the user fixes it there (re-copy both parts,
  or create a new key). If API access is off for the account, an admin must turn it on.
- A key that belongs to a **service account** is refused: it must be the user's own key.

**B. Sign in with Kissflow (MCP).** Only when the user asks to sign in instead of using a key:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect <account-address>
source .kf-env
```
The browser opens the account's own Kissflow sign-in (SSO included); the user signs in and approves
access, and connect finishes on its own. If it says the account **does not offer assistant sign-in**,
an admin must enable MCP access for the account — offer the access key (A) instead.

Either way, everything the agents build is done **as this person**, with exactly their Kissflow
permissions. The credential stays in the user's home folder, never in this workspace. `.kf-env` holds
only which account and method this folder uses (`KISSFLOW_DOMAIN`, `KISSFLOW_ACCOUNT_ID`, `KF_AUTH`),
who connected (`KF_USER_EMAIL`, `KF_USER_NAME`) and the account name (`KF_PROJECT_NAME`) — nothing
secret, but still never commit it. `connect --status` shows the account, person and method;
`connect --sign-out` forgets the credential on this machine. Re-running connect with another address
switches accounts.

**THE ACCOUNT NAME IS THE WORKING CONTEXT.** `.kf-env` carries `KF_PROJECT_NAME` (the Kissflow
account's name). Greet with it ("Connected to **Acme** — what should the app do?"), interpret every
subsequent ask in its context, and prefix clarifying questions with it. Never ask which account this
is about — you already know.

Everything else lives in this folder: the design graph under `runs/<app>/ir-graph/`, each build's
saved versions under `runs/<app>/published/`, and the agents' memory in `MEMORY.md` + `MEMORY-LOCAL.md`.
Agents write new lessons with `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" memory remember "<lesson>" --app <appId>`.

## 3b. Choose the app to build in
Right after connecting, open the app picker:
```bash
node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" connect --app
source .kf-env
```
A page opens in the browser listing the apps in this account the user can **edit**, with search and
whether **custom UI** is enabled on each. They pick one (builds go into that app) or **Start a new
app** (the next build creates one). Show the printed link in case the browser didn't open; it waits
up to 15 minutes. The choice lands in `.kf-env` as `KF_APP_ID`, `KF_APP_NAME`, `KF_APP_CUSTOM_UI`.
They can switch apps any time with `/switch-app`. If custom UI is off for the chosen app and they
want a custom React UI, connect prints the app's settings link: ask them to turn custom UI on there,
then re-run `connect --app`.

## 4. Seed the agent memory
```bash
[ -f MEMORY.md ] || cp "${CLAUDE_PLUGIN_ROOT}/MEMORY.md" MEMORY.md
```
`MEMORY.md` is the agents' auto-evolving memory — yours to grow; an existing one is never replaced.

## 5. Safety
Always **dry-run** first and target a **dev** account; the pipeline never auto-publishes to prod.

## 6. Go
`/build-app "<your requirement>"` — author a full app top-down, or the staged loop starting with
`/author-brief`.

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
