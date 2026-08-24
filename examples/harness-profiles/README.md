# Repository harness-profile examples

These examples configure the autonomous harness that executes workflow leaves. That harness is
independent of the interactive client used to install or steer Workflows: a Codex user can select the
Claude Code or OpenCode profile, for example, when that harness is installed on the machine.

| File | Native run-profile difference |
| --- | --- |
| [`claude-code.json`](./claude-code.json) | Claude Code `effortLevel` is `high` for implementation and landing repair, `low` for launch repair, and `medium` otherwise. |
| [`codex.json`](./codex.json) | Codex `model_reasoning_effort` is `high` for implementation and landing repair, `low` for launch repair, and `medium` otherwise. |
| [`opencode.json`](./opencode.json) | OpenCode gives the write-capable build agent 100 steps for implementation and landing repair, 25 for launch repair, and 50 otherwise. |

## System selector roster

Workflows requests only the following owner-facing system roster. Published names are durable ordinary
`runProfiles` keys, not a closed namespace; workflow authors may use other semantic names for steps.

| Selector | Execution moment |
| --- | --- |
| `default` | Chooses a transition, prepares work files, runs a workflow step whose `runProfile` is omitted or `null`, and supplies every other built-in invocation without a dedicated selector. It also substitutes for any requested key the repository profile does not declare. |
| `launch-repair` | The runner-owned `launch-repair` preflight and repair worker. The worker resolves the current active profile again before invocation. |
| `landing-repair` | The engine-owned `landing-repair` invocation that may repair one blocked landing in the item checkout. |

An undeclared requested key uses `default`; it is not a profile refusal. This substitution is distinct
from the moments in the `default` row that request `default` directly. An undeclared `landing-repair`
key therefore runs that same engine-owned invocation with `default` options.

The example-only `implementation`, `ui-validation`, and `workflow-documentation` names demonstrate that
repository owners can continue to configure workflow-authored semantic names outside the system roster.

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
