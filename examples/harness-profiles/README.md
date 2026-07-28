# Repository harness-profile examples

These examples configure the autonomous harness that executes workflow leaves. That harness is
independent of the interactive client used to install or steer Workflows: a Codex user can select the
Claude Code or OpenCode profile, for example, when that harness is installed on the machine.

| File | Native run-profile difference |
| --- | --- |
| [`claude-code.json`](./claude-code.json) | Claude Code `effortLevel` is `high` for implementation and `medium` otherwise. |
| [`codex.json`](./codex.json) | Codex `model_reasoning_effort` is `high` for implementation and `medium` otherwise. |
| [`opencode.json`](./opencode.json) | OpenCode gives the write-capable build agent 100 steps for implementation and 50 otherwise. |

Each workflow step may name a semantic run profile. An omitted or `null` `runProfile` selects
`default`, so that entry also governs routing when a step does not name another profile.

To use one, create the repository directory and download the file matching the autonomous harness:

```sh
mkdir -p .workflows/harness-profiles
curl -fsSL \
  https://raw.githubusercontent.com/codingawayy/workflows-releases/main/examples/harness-profiles/claude-code.json \
  -o .workflows/harness-profiles/claude-code.json
```

Replace `claude-code` with `codex` or `opencode` in both paths when selecting another harness.
Copying the file makes it discoverable; it does not activate it.

Workflows validates the repository-profile envelope and each adapter's safety and serialization
constraints. The installed harness and version own support for the native keys and values inside
`options`; these examples do not make Workflows a validator for those settings.
