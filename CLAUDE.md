# RSMP Core v4 — Copilot Instructions

## What This Repo Is

A **specification repository** for RSMP 4 (Road-Side Message Protocol), published as a Jekyll static site to GitHub Pages. It contains only documentation — no runnable code.

RSMP 4 is a new major version. There is no backward compatibility with RSMP 3. Do not add compatibility notes or legacy references.

## Build & Preview

```bash
cd docs
bundle install
bundle exec jekyll serve   # serves at http://localhost:4000
```

## Structure

```
docs/pages/       # all spec pages (Jekyll source)
docs/_config.yml  # site config (theme: just-the-docs)
docs/Gemfile      # Ruby deps for the docs site
```

Top-level spec pages: `overview.md`, `mqtt.md`, `nodes.md`, `modules.md`, `components.md`, `streams.md`, `versioning.md`

Message sub-pages (parent: `Messages`): `presence.md`, `status.md`, `command.md`, `result.md`, `alarm.md`, `reaction.md`, `aggregate.md`, `stream.md`, `throttle.md`

## Key Terminology

| Term | Meaning |
|---|---|
| **Node** | Any RSMP participant (site, supervisor, or both) |
| **Module** | A defined interface of commands, statuses and alarms for a domain (e.g. TLC, Sensor) |
| **Service** | The device/producer side of a module — sends status/alarms, receives commands |
| **Manager** | The supervisor/consumer side — sends commands, receives status/alarms |
| **Component** | A sub-element of a node (e.g. `sg/1`, `tc`); identified by a slash-separated path |
| **Main component** | Root-level component; addressed by omitting the component segment from topic paths |
| **Stream** | A pre-configured publishing channel for a status code, with rate/delta/aggregation settings |
| **Throttle** | A runtime start/stop control for a stream |
| **Code** | Dotted string identifying a command/status/alarm, e.g. `tlc.plan.set` |
| **SXL** | Signal Exchange List — per-project definition of modules in use |
| **CBOR** | Payload encoding for all messages (binary, JSON data model) |

## MQTT Topic Structure

```
<node_id>/<type>/<code>[/<stream>][/<component>]
```

- `node_id` may be multi-level: `dk/cph/tlc-001`
- `type`: `status`, `command`, `alarm`, `result`, `reaction`, `presence`, `stream`, `throttle`
- `stream` is inserted between `code` and `component` for status topics only
- `presence` has no code or component segment

## Writing Conventions

- **Normative language:** use `MUST` / `MUST NOT` / `SHOULD` (RFC-style) for behavioral rules
- **Terse spec prose:** short declarative sentences, no verbose filler
- **Every topic path example** goes in a fenced code block with inline `# comments`
- **Mermaid diagrams** for architecture and flow
- **Tables** for message attributes, QoS guidance, and type summaries

## Page Front Matter Pattern

```yaml
---
title: My Topic
parent: Messages      # omit if top-level
nav_order: 3
permalink: /my-topic/
---
```
