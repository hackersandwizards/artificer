# artificer

An artificer's kit for building minimal dark-factories on Claude Code. You compose small, gated agent steps into a deterministic build you can read end to end. The model does the work. The harness stays thin. A "dark factory" is a lights-out production line that runs without people watching it.

## Requirements

- Claude Code. The stages artificer builds are Claude Code agents, skills, and workflow scripts.
- For the sdlc example, a project like the one it was written for: a Next.js app with Prisma, Chakra and Vitest whose gates run `npm test`, the Codex CLI for review-codex, and the chrome-devtools MCP for acceptance, which drives the app at `http://localhost:3000` with the dev server already running.

## Install

Add the marketplace, then install the plugin:

```
/plugin marketplace add hackersandwizards/artificer
/plugin install build-workflow@hackersandwizards
```

## Build a stage

```
/build-workflow <workflow> <order> <step>: role / models / tools / check
```

For example, a first stage that turns a feature request into a plan, on Opus at high effort, with
the tools Read and Write and the check `true`:

```
/build-workflow sdlc 1 plan: Plan with acceptance criteria / opus, high / Read, Write / true
```

Each stage gets a subagent with a gate that re-runs until its check passes, a launcher skill to run the stage on its own, and a line in a workflow script that threads each stage's output into the next.

- The check command is the contract: a stage is done only when its gate exits 0, for example tests green or build clean.
- Each call adds a stage; calling it again for an existing stage rewrites that stage.
- The workflow script runs as `/<workflow>` with the input as its arguments, for example `/sdlc <feature request>`; agents generated during a session register only after Claude Code restarts.
- For parallel stages, loops, and escalation, edit the generated `.claude/workflows/<workflow>.js` directly. [The generator](plugins/build-workflow/agents/build-workflow.md) embeds the recipes as comments.

The plugin also ships `/improve-workflow`. It runs a generated workflow on a real input, checks that its assumptions hold (gates fire, structured returns branch, parallel stages overlap), and feeds what it learns back into the workflow and the generator templates.

## Example

[examples/sdlc/](examples/sdlc/) is a ten-stage line built this way: plan, tests, implement, simplify, two reviews in parallel, converge, acceptance, commit and report. Converge loops back to implement on real issues and hands over to a human after three rounds; the orchestration in [`examples/sdlc/.claude/workflows/sdlc.js`](examples/sdlc/.claude/workflows/sdlc.js) is plain JavaScript control flow.

## About and license

Built by [hackers&wizards](https://hackersandwizards.dev). We have spent 20+ years building and running production systems, and we teach Functional Agentic Engineering: small composable steps over sprawling harnesses. artificer is that practice as Claude Code tooling.

MIT. See [LICENSE](LICENSE).
