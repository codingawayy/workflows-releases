# Install the Workflows plugin for OpenCode

You are installing the Workflows agent plugin for OpenCode. The user pasted a one-liner that gave
you a **board URL** (`apiUrl`), a **project id** (`projectId`), and a **scope** (either `global` or
`per-repo`). Follow these steps in order. Use shell commands to download and write files.

## What you're installing

- **`mcp.js`** — the `workflows` MCP server (the spine tools: read/write items, drive transitions).
- **Four command files** — `/workflows-add-item`, `/workflows-author`, `/workflows-run-item`,
  `/workflows-discuss` (thin user-facing entry points).
- **Four native skill trees** — the interactive workflow intelligence plus every referenced document.
- **MCP registration** — an `mcp.workflows` entry in an `opencode.json` config that launches the
  server.
- **Connection** — a per-machine board/auth-binding file and a per-repo project-id file.

The harness that executes autonomous workflow leaves is selected separately from the OpenCode
client used to steer Workflows. The canonical
[repository harness-profile examples](https://github.com/codingawayy/workflows-releases/tree/main/examples/harness-profiles)
cover Claude Code, Codex, and OpenCode and explain how to copy one into
`.workflows/harness-profiles/`.

## Where things go (depends on scope)

The user's one-liner named a scope: **global** or **per-repo**.

| File              | Global scope                                | Per-repo scope                       |
| ----------------- | ------------------------------------------- | ------------------------------------ |
| `mcp.js`          | `~/.config/opencode/workflows/mcp.js`       | `.opencode/workflows/mcp.js`         |
| Command files     | `~/.config/opencode/commands/`              | `.opencode/commands/`                |
| Skill trees       | `~/.config/opencode/skills/`                | `.opencode/skills/`                  |
| `opencode.json`   | `~/.config/opencode/opencode.json`          | `opencode.json` (repo root)          |
| `client.json`     | `~/.workflows/client.json`                  | `~/.workflows/client.json`           |
| `config.json`     | `.workflows/config.json` (repo root)        | `.workflows/config.json`             |

The OpenCode delivery owns its `mcp.js`, commands, skills, and MCP registration in the selected
OpenCode scope. Only connection state belongs under `~/.workflows`; `config.json` is always per-repo.

## Steps

### 1 · Download `mcp.js`

Download the bundled OpenCode MCP server from:

`https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/mcp.js`

**Global:**

```sh
mkdir -p ~/.config/opencode/workflows
curl -fsSL https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/mcp.js \
  -o ~/.config/opencode/workflows/mcp.js
```

**Per-repo** (run from the repo root):

```sh
mkdir -p .opencode/workflows
curl -fsSL https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/mcp.js \
  -o .opencode/workflows/mcp.js
```

### 2 · Download the command files

Download the four command files to the scope-appropriate commands directory. The files are:

- `workflows-add-item.md` — `https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/commands/workflows-add-item.md`
- `workflows-author.md` — `https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/commands/workflows-author.md`
- `workflows-run-item.md` — `https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/commands/workflows-run-item.md`
- `workflows-discuss.md` — `https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/commands/workflows-discuss.md`

**Global:**

```sh
mkdir -p ~/.config/opencode/commands
for verb in add-item author run-item discuss; do
  curl -fsSL "https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/commands/workflows-${verb}.md" \
    -o "$HOME/.config/opencode/commands/workflows-${verb}.md"
done
```

**Per-repo** (run from the repo root):

```sh
mkdir -p .opencode/commands
for verb in add-item author run-item discuss; do
  curl -fsSL "https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/commands/workflows-${verb}.md" \
    -o ".opencode/commands/workflows-${verb}.md"
done
```

### 3 · Download the native skill trees

Choose the skill root from the requested scope:

- **Global:** `$HOME/.config/opencode/skills`
- **Per-repo:** `.opencode/skills`

Download every file below while preserving its relative path beneath that root:

```text
workflows-add-item/SKILL.md
workflows-add-item/reference/breakdown.md
workflows-add-item/reference/routing.md
workflows-author/SKILL.md
workflows-author/reference/best-practices.md
workflows-author/reference/revision-input.md
workflows-author/reference/shape-idioms.md
workflows-run-item/SKILL.md
workflows-discuss/SKILL.md
```

For example, set `SKILL_ROOT` to the scope-appropriate directory, then:

```sh
BASE="https://raw.githubusercontent.com/codingawayy/workflows-releases/main/opencode/skills"
for rel in \
  workflows-add-item/SKILL.md \
  workflows-add-item/reference/breakdown.md \
  workflows-add-item/reference/routing.md \
  workflows-author/SKILL.md \
  workflows-author/reference/best-practices.md \
  workflows-author/reference/revision-input.md \
  workflows-author/reference/shape-idioms.md \
  workflows-run-item/SKILL.md \
  workflows-discuss/SKILL.md
do
  mkdir -p "$SKILL_ROOT/$(dirname "$rel")"
  curl -fsSL "$BASE/$rel" -o "$SKILL_ROOT/$rel"
done
```

Do not copy only `SKILL.md`; the `reference/` documents are part of the skill package.

### 4 · Register the MCP server

Merge the `mcp.workflows` entry into the scope-appropriate `opencode.json`. Read the existing file
first (if any) and merge — do not overwrite.

**Global:** merge into `~/.config/opencode/opencode.json`. Resolve the user's home directory while
installing and write the resulting absolute `mcp.js` path into this machine-local config:

```json
{
  "mcp": {
    "workflows": {
      "type": "local",
      "command": ["bun", "run", "<absolute-home>/.config/opencode/workflows/mcp.js"],
      "enabled": true
    }
  }
}
```

**Per-repo:** merge into `opencode.json` at the repo root. Use the portable repository-relative path:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "workflows": {
      "type": "local",
      "command": ["bun", "run", ".opencode/workflows/mcp.js"],
      "enabled": true
    }
  }
}
```

If the file already has a `"mcp"` key, add `"workflows"` inside it. If it already has a
`"mcp.workflows"` key, replace its value. Keep all other keys (`$schema`, `plugin`, `command`, etc.)
intact. A repository `opencode.json` must never contain a home directory or another machine-specific
path.

### 5 · Connection — tell the user to run the scope-owned bundle

Tell the user to run the command for the scope they selected, substituting the `apiUrl` from their
one-liner:

- **Global:** `bun run "$HOME/.config/opencode/workflows/mcp.js" connect <apiUrl>`
- **Per-repo:** `bun run ".opencode/workflows/mcp.js" connect <apiUrl>`

The command contains no credential. It shows a one-time device approval code, then the bundled
operation owns discovery and all machine-auth mutation policy; do not write `~/.workflows/client.json`
or the device credential yourself.

### 6 · Write the per-repo project id

Write (merging into any existing file) to `.workflows/config.json` in the repo root:

```json
{ "projectId": "<the project id from the one-liner>" }
```

This file holds no secret — it's fine to commit.

### 7 · Verify

Tell the user to **restart OpenCode** (or reload its config) so the `workflows` MCP server picks up
the credential and the project. Then tell them to verify the install by running:

```
/workflows-add-item test
```

They should see a routing question or a proposal — if they do, the server is running and
authenticated. If OpenCode's MCP list doesn't show `workflows` as running, the most common causes
are: Bun not on the PATH OpenCode inherits (check with `bun --version` in a terminal), a wrong path
in the `command` field, or an unapproved/expired device code.
