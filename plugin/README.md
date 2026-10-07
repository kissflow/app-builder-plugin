# Kissflow App Builder

A Claude Code plugin that turns an idea or a requirements document into a running Kissflow app:
data models, roles, workflows, permissions and pages in your Kissflow account, with an optional
custom React UI. This folder carries the commands, the specialist agents, the reference documents
they read, and a small launcher for the build engine.

## Setup

Once per project folder:

```
/connect
```

The first run downloads the build engine for your platform (about 100 MB, once per version) and
connects the folder to your Kissflow production account and its Development sandbox — give either
address — and you paste an access key you create in each (profile picture › My settings › API
authentication › Access keys) into one page that opens in your browser. New apps are created in the production account; everything
else is built in the sandbox. The app is built as you, with your Kissflow permissions, and the keys
stay in your home folder.

## Usage data

The plugin sends no telemetry of its own. Each request it makes to your Kissflow account names the
plugin in its User-Agent, with its version, your platform and random ids for this installation,
workspace and command, so Kissflow can see how the plugin is used from its own request logs. Nothing
about your apps, requirements or data is added. Set `KF_TELEMETRY=0` to send only the plugin version
and platform.

## Use

```
/build-app "<a one-line idea, or the path to a BRD>"
```

Or step by step: `/author-brief` → `/author-plan` → `/author-review` → `/author-refine` →
`/author-preview` → `/author-generate`. `/author-status` and `/author-runs` show where a build stands.

`/sync` brings in an app that already exists; `/author-understand` explains it and `/author-reconcile`
changes it.

Everything is built in your Development sandbox. The only thing created in production is a new,
empty app, which Kissflow copies into the sandbox; nothing else is published there.
