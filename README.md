# Worca

[![npm](https://img.shields.io/npm/v/@worca/app)](https://www.npmjs.com/package/@worca/app)
[![license](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![node](https://img.shields.io/badge/node-%3E%3D22.19-brightgreen)](.nvmrc)

Worca turns a task into reviewed, merge-ready work.

You describe the change. A deterministic pipeline of coding agents clarifies, plans,
implements and reviews it, and loops until the review is clean. Each run works on its own
git branch, so your checkout stays untouched and finished work arrives as a pull request.

**[Quick start](#quick-start)** · [How a run works](#how-a-run-works) ·
[Why Worca](#why-worca) · [Ways to use it](#ways-to-use-it) · [Features](#features) ·
[Documentation](#documentation)

![The Worca web UI showing a finished run on its Diff tab](docs/screenshots/run-detail.png)

*A finished run: 4 files changed on the run's own `worca-cc/…` branch, with its time and
cost, ready for **Create PR**.*

## Quick start

You need Node.js 22.19 or newer and git. Your first run can be a mock run: it runs the
whole pipeline offline, with no tokens and no cost.

1. Install Worca.

   ```bash
   npm install -g @worca/app
   ```

   If npm skips install scripts or your shell cannot find `worca`, see
   [Install](docs/install.md#npm).

2. Start the web UI.

   ```bash
   worca ui --open
   ```

   Your browser opens `http://127.0.0.1:4317`.

3. Add a project under **Projects › Add project**: a new folder, a folder you already
   have, or a clone from a URL.

4. Open **Runs › New**. Describe the task, turn on **Mock mode** and press **Start run**.

5. Worca proposes a workflow, the graph of agent steps for this task. Press
   **Accept & run**.

The mock run finishes in seconds. Its page shows the result on the run's own `worca-cc/…`
branch.

A first start opens a setup check, and **Set up later** leads to a welcome dialog. Its
**Watch a run end to end, offline** entry guides you through steps 3 to 5.

### Your first real run

Real runs need the [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI on
your `PATH`, signed in: run `claude`, then `/login`. Then start a run with **Mock mode**
off.

Claude Code is the default engine. Codex, the GitHub Copilot CLI, Cursor, Gemini CLI,
Goose, OpenCode and Qwen Code are in beta: see [Models](docs/models.md).

### From the terminal

The CLI needs no server. Run this inside a project:

```bash
worca "Add a /search endpoint with pagination" --mock --yes
```

The output ends with:

```text
Pipeline complete.
Result: completed
```

Drop `--mock --yes` for a real run that asks you its questions.

See [Install](docs/install.md) for npm install scripts, Gentoo, Windows, the container
image and a hosted setup, and [CLI](docs/cli.md) for every command.

## How a run works

In the web UI, **Auto** is selected by default: one model call classifies the task and
assembles a workflow for it, which you review before it runs. The CLI runs the built-in
**Default** workflow unless you pass `--workflow auto`.

Default hands the task to five agents:

1. **Clarify** asks up to 8 multiple-choice questions, or none when the task is clear.
2. **Plan** explores the codebase and writes an implementation plan.
3. **Refine Plan** checks the plan against the real code in one read-only pass and
   tightens it.
4. **Implementation** follows the approved plan with test-driven development.
5. **Review Implementation** reviews the diff against the plan and sends blocking
   findings back to the implementer.

![A live run waiting at its Clarify step with two multiple-choice questions](docs/screenshots/clarify.png)

*Clarify at work: lettered options, a Recommended pick, and a field to type your own
answer.*

Three rules keep the loop bounded:

- Refine runs once.
- Only critical and major findings block.
- The review loop has a cycle cap, 3 by default. At the cap with blocking findings left,
  the run pauses and asks whether to approve another cycle or continue.

Worca commits a finished run's work to its own `worca-cc/…` branch, ready for
**Create PR**. More in [Runs](docs/runs.md).

## Why Worca

- **The engine decides, the agents do the work.** A state machine walks the workflow
  graph. Agents never choose what runs next, and Worca records every step, verdict, cost
  and file.
- **It asks before it guesses.** A Clarify step turns hidden decisions into a few
  multiple-choice questions, and review loops pause for you when they run out of cycles.
- **Changes kept apart.** Every run gets its own worktree and branch, so parallel runs
  never collide and nothing touches your working tree.
- **Bounded.** Cycle caps, cost limits, guardrail policy sets and a team policy keep
  runs inside limits you set.
- **Workflows you can see and change.** The Default workflow is one graph among many:
  draw your own, let Auto pick one per task, or share one as a plugin.
- **Built for a team.** Your team shares metrics and policy through git branches on your
  own repository, with no extra server.

More in [Why Worca](docs/why-worca.md): what sets it apart, and the use cases that follow.

## Ways to use it

Worca runs on a dev machine or a server, and you can reach it in five ways.

| Interface | What it is | Docs |
|---|---|---|
| Web UI | `worca ui`: runs, workflows, chat, settings | [Getting started](docs/getting-started.md), [UI levels](docs/ui-levels.md) |
| CLI | `worca …`: start, follow, pause, resume and schedule runs, with no server needed | [CLI](docs/cli.md) |
| `/worca` skill | `worca --install <project>`, then `/worca <task>` inside Claude Code | [CLI › skill](docs/cli.md#the-worca-skill-inside-claude-code) |
| Chat channels | Answer questions and steer runs from Telegram, Slack, Discord or Teams | [Plugins](docs/plugins.md) |
| Browser extension | A Worca chat beside the page you are reading | [Browser extension](docs/browser-extension.md) |

[![Worca architecture diagram: you, the Worca host, the coding-agent harnesses and the models](docs/screenshots/architecture.png)](docs/ARCHITECTURE.md)

*The whole stack. The Worca host runs `@worca/app` (engine, agents, workflows,
guardrails, costs and budgets, plugins) and drives N headless Claude Code harnesses. They
talk to Anthropic Cloud, a gateway or proxy, an air-gapped on-prem or dev-machine
endpoint, or Bedrock/Vertex.*

## Features

Each link opens the page that documents the feature.

### Runs

- **[Runs](docs/runs.md)**: pause and resume across restarts, live view, logs, one-click
  PRs, Watch PR, run on the source branch
- **[Run presets](docs/run-presets.md)**: save a New run setup and reuse it
- **[Shaper](docs/shape-it-first.md)**: shape a thin or large task before anything is
  built
- **[Scheduled runs](docs/scheduled-runs.md)**: start a run later, once or on a repeat,
  and chain runs into trains
- **[Away mode](docs/away-mode.md)**: Worca answers what a run waits on while you are away

### Workflows and outputs

- **[Workflows](docs/workflows.md)**: the workflow editor, built-in agents,
  [script cards](docs/scripts.md), the composer chat, sharing workflows
- **Outputs**: [presentation decks](docs/workflows.md#presentations),
  [Architecture (C4) models](docs/workflows.md#architecture-c4) and
  [promo videos](docs/workflows.md#promo-video)
- **[Design](docs/design.md)** (beta): describe a web page, document, presentation, pitch
  deck or video in a chat, and agents build it

### Projects and chat

- **Projects**: git repositories, plain folders, or no project
  ([Storage](docs/storage.md))
- **Workspaces**: sets of related repos with a code-derived map
  ([Workspace map](docs/workspace-map.md))
- **[Actions](docs/actions.md)**: check out a result and run its dev server or tests,
  beside a [terminal pane](docs/terminal.md)
- **[Agent readiness](docs/agent-readiness.md)**: whether a project is ready for the
  coding agents it uses, and how to fix each gap
- **[Chat](docs/chat.md)**: ask Worca about runs and code, confirm its changes as cards,
  and use agent mode, memory and [voice](docs/speech.md)

### Integrations and models

- **[Plugins](docs/plugins.md)**: plugins and marketplaces, with GitHub Issues, Azure
  Boards and Jira as task sources
- **Git hosts**: GitHub, [Azure DevOps](docs/azure-devops.md) and
  [Bitbucket](docs/bitbucket.md)
- **[MCP servers](docs/mcp-servers.md)** and **[Skills](docs/skills.md)**: Worca's own
  MCP registry and skill sets
- **[Models](docs/models.md)**: engines, bring your own models, Copilot and
  OpenAI-compatible providers, [image generation](docs/image-generation.md) and
  [sound](docs/sound.md)

### Governance and costs

- **[Guardrails](docs/guardrails.md)**: policy sets (Permissive, Normal, Strict)
- **[Costs](docs/costs.md)**: cost per run, hard limits and cost tags, with
  [Statistics](docs/statistics.md)
- **[Team metrics](docs/team-metrics.md)** and **[Team policy](docs/team-policy.md)**:
  shared through git
- **[Feature flags](docs/feature-flags.md)**: Labs features behind flags

### Deployment

- **[Container image](docs/docker.md)**: Worca and every agent it spawns in a disposable
  Linux container
- **Hosted**: behind Cloudflare Access, on [Railway](docs/deploy-railway.md) or another
  host ([Remote access](docs/remote-access.md))
- **[Credential broker](docs/credential-broker.md)**: model keys stay out of Worca's
  container

## Documentation

Every page is listed in the **[documentation index](docs/README.md)**. Good places to
start:

- [Getting started](docs/getting-started.md): the welcome dialog and the Getting started
  checklist
- [Runs](docs/runs.md): everything a run does, from Auto to the pull request
- [Architecture](docs/ARCHITECTURE.md): the whole stack in one picture, and how Auto
  picks a workflow

Release notes are at [docs.worca.dev/changelog](https://docs.worca.dev/changelog/).

## Contributing and help

- **Bugs and requests:** open an [issue](https://github.com/worca-org/worca/issues).
- **Security:** report a vulnerability privately, as [SECURITY.md](SECURITY.md) says.
- **Code:** PRs are welcome. [CONTRIBUTING.md](CONTRIBUTING.md) covers the from-source
  setup, mock mode, the test suite and the PR workflow. Development happens on the `dev`
  branch, and PRs target `dev`.

Everyone taking part is expected to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

The repository moved from `SinishaDjukic/worca-cc` to
[worca-org/worca](https://github.com/worca-org/worca) in November 2026. Old links
redirect.

## License

[MIT](LICENSE)
