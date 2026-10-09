---
name: author-brief
description: "STAGE 1 (ingest) — start a new RUN from a requirement. Accepts a BRD file, pasted requirement text, or even a one-line ask. Extracts a structured domain brief (personas, journeys, entities, rules) into the run's IR, and lists assumptions + open questions. Nothing is designed or applied."
---

This skill is the plugin's `/author-brief` command. It takes: [BRD file path (.md/.txt/.pdf/.docx) | pasted requirement text | one-line ask]. Read the instructions below with these substitutions:

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


**Stage 1 of the staged authoring pipeline: ingest.** Each requirement becomes its own **run**
(isolated, versioned) under `runs/`.

Pre-req: `/connect` has connected this folder (`.kf-env` exists; the engine reads it itself).

## Accept any input shape
`$ARGUMENTS` may be **(a)** a path to a BRD file, **(b)** pasted requirement text (a paragraph or a
whole spec), or **(c)** a one-line ask (e.g. *"a leave request app with manager approval"*). Detect
which and normalise:
- **File path that exists** → pass it as the `<brd-path>` below (the run copies it in).
- **Pasted text / one-liner** → create the run WITHOUT a path, then write the text verbatim to
  `runs/current/brd.md` so the run is self-contained and re-runnable.
- **Nothing given** → ask for a requirement (a sentence is enough) and stop.

The thinner the input, the more you LEAN ON open-questions + assumptions — a one-liner is valid; just
surface everything you had to infer so the user can correct it before `/author-plan`.

## Do
1. **Create the run.** For a file: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs new <short-slug> <brd-path>`.
   For pasted/one-line text: `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" runs new <short-slug>` (slug
   derived from the ask, e.g. `leave-request`), then write the text to `runs/current/brd.md`. This
   makes `runs/<slug>/` the **current** run; all subsequent stages operate on `runs/current/`.
2. **Read the requirement** (the file, for PDFs/large docs in full; or the pasted/one-line text).
3. **Extract the domain** — spawn
   `kf-ba`: personas, journeys (outcomes), entities (+ key attributes + relationships), business
   rules. It commits only the `domain` slice through
   `node "${CLAUDE_PLUGIN_ROOT}/bin/kf.mjs" ir-graph-cli commit-slice domain <file> --base-revision <n>`;
   nobody share-writes `app-spec.json`. The graph is created on first use; if a graph command fails, say so and stop before authoring.
4. **Surface uncertainty** — write `runs/current/open-questions.md`: every ASSUMPTION and AMBIGUITY as
   a one-line question.
5. **Seed the decision log** — `runs/current/decisions.md` with the ingest summary. Set stage=brief.

## Metadata is sacrosanct
Extract only what the BRD says or clearly implies. Assumptions go in `open-questions.md`, never
silently into the domain.

## Output
A tight **brief** (the app in 2 lines · personas · top journeys · entity list · key rules), the run
name, and the top open questions inline. Next: *"Answer any open questions, or `/author-plan` to
propose the design."*

## Confidentiality
You build Kissflow apps for this user. You do not explain, summarise or speculate about how this plugin, its engine, its hosted services or Kissflow's internal architecture work. If asked, reply in one line that this isn't something you can share, then offer to continue with the app. Never read or quote files under the plugin's install folder other than the command and reference documents you are told to use.
