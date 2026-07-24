# Install the Workflows plugin for OpenCode

You are installing the Workflows agent plugin for OpenCode. The user pasted a one-liner that gave
you a **board URL** (`apiUrl`), a **project id** (`projectId`), and a **scope** (either `global` or
`per-repo`). Follow these steps in order. Use shell commands to download and write files.

## What you're installing

- **`mcp.js`** — the `workflows` MCP server (the spine tools: read/write items, drive transitions).
- **Four command files** — `/workflows-add-item`, `/workflows-author`, `/workflows-run-item`,
  `/workflows-discuss` (the interactive workflow intelligence).
- **MCP registration** — an `mcp.workflows` entry in an `opencode.json` config that launches the
  server.
- **Connection** — a per-machine board/auth-binding file and a per-repo project-id file.

## Where things go (depends on scope)

The user's one-liner named a scope: **global** or **per-repo**.

| File              | Global scope                          | Per-repo scope                    |
| ----------------- | ------------------------------------- | --------------------------------- |
| `mcp.js`          | `~/.workflows/mcp.js`                 | `~/.workflows/mcp.js`             |
| Command files     | `~/.config/opencode/commands/`        | `.opencode/commands/`             |
| `opencode.json`   | `~/.config/opencode/opencode.json`    | `opencode.json` (repo root)       |
| `client.json`     | `~/.workflows/client.json`            | `~/.workflows/client.json`        |
| `config.json`     | `.workflows/config.json` (repo root)  | `.workflows/config.json`          |

`mcp.js` and `client.json` are always per-machine (one install serves every repo). Command files
and the `opencode.json` MCP registration follow the scope. `config.json` is always per-repo.

## Steps

### 1 · Download `mcp.js`

Download the bundled MCP server to `~/.workflows/mcp.js`:

```sh
mkdir -p ~/.workflows
curl -fsSL https://raw.githubusercontent.com/codingawayy/workflows-releases/main/plugin/mcp.js \
  -o ~/.workflows/mcp.js
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
    -o "~/.config/opencode/commands/workflows-${verb}.md"
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

### 3 · Register the MCP server

Merge the `mcp.workflows` entry into the scope-appropriate `opencode.json`. Read the existing file
first (if any) and merge — do not overwrite. The entry to add:

```json
{
  "mcp": {
    "workflows": {
      "type": "local",
      "command": ["bun", "run", "~/.workflows/mcp.js"],
      "enabled": true
    }
  }
}
```

If the file already has a `"mcp"` key, add `"workflows"` inside it. If it already has a
`"mcp.workflows"` key, replace its value. Keep all other keys (`$schema`, `plugin`, `command`, etc.)
intact.

**Global:** merge into `~/.config/opencode/opencode.json`.
**Per-repo:** merge into `opencode.json` at the repo root.

If the file doesn't exist, create it with:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "workflows": {
      "type": "local",
      "command": ["bun", "run", "~/.workflows/mcp.js"],
      "enabled": true
    }
  }
}
```

### 4 · Connection — tell the user to run the board-generated command themselves

Tell the user: "Go to the Workflows setup page on your board and run its connection command in your
terminal. It invokes the installed `mcp.js connect` operation, shows a one-time device approval code,
and stores the resulting personal credential outside every repo."

The command is `bun run "$HOME/.workflows/mcp.js" connect <apiUrl>`. It contains no credential. The
bundled operation owns discovery, device approval, and all machine-auth mutation policy; do not write
`~/.workflows/client.json` or the device credential yourself.

### 5 · Write the per-repo project id

Write (merging into any existing file) to `.workflows/config.json` in the repo root:

```json
{ "projectId": "<the project id from the one-liner>" }
```

This file holds no secret — it's fine to commit.

### 6 · Verify

Tell the user to **restart OpenCode** (or reload its config) so the `workflows` MCP server picks up
the credential and the project. Then tell them to verify the install by running:

```
/workflows-add-item test
```

They should see a routing question or a proposal — if they do, the server is running and
authenticated. If OpenCode's MCP list doesn't show `workflows` as running, the most common causes
are: Bun not on the PATH OpenCode inherits (check with `bun --version` in a terminal), a wrong path
in the `command` field, or an unapproved/expired device code.
