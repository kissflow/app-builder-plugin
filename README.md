# Kissflow App Builder

A Claude Code plugin that turns an idea or a requirements document into a running Kissflow app:
data models, roles, workflows, permissions and pages in your Kissflow account, with an optional
custom React UI.

## Install

```
/plugin marketplace add kissflow/app-builder-plugin
/plugin install app-builder@kissflow
```

Then, once per project folder:

```
/connect
```

The first run downloads the build engine for your platform (about 100 MB, once per version) and
connects the folder to your account's Kissflow Development sandbox, the only place it builds: paste an access key you create in Kissflow (profile
picture › My settings › API authentication › Access keys) into a page that opens in your browser.

## Usage data

The plugin sends no telemetry of its own. Each request it makes to your Kissflow account names the
plugin in its User-Agent, with its version, your platform and random ids for this installation,
workspace and command, so Kissflow can see how the plugin is used from its own request logs. Nothing
about your apps, requirements or data is added. Set `KF_TELEMETRY=0` to send only the plugin version
and platform.

## Requirements

- Claude Code with Node.js 18 or newer
- `python3` on your PATH (used when publishing native Kissflow pages)
- macOS (Apple silicon or Intel), Linux x64, or Windows x64
- An invitation to Kissflow App Builder for your work email

## Use

```
/build-app "<a one-line idea, or the path to a BRD>"
```

Or step by step: `/author-brief` → `/author-plan` → `/author-review` → `/author-refine` →
`/author-preview` → `/author-generate`. `/author-status` and `/author-runs` show where a build stands.

Everything targets your development environment. Nothing publishes to production.

## Releases

Each release carries the engine binaries the plugin downloads. The plugin version and the release
tag always match; `/plugin update app-builder` moves you to the newest one.
