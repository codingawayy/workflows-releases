# Workflows — Claude Code plugin

The agent's workflow intelligence, as one installable Claude Code plugin. It is the **floor** to use
the product interactively — no tray required. Installing it gives you:

- **The `workflows` MCP server** — the in-session tools to read and drive workflow items over your
  board's API.
- **`/workflows:add-item`** — turn natural-language work into well-recorded items: broken down, routed to
  the right workflow, and recorded to that workflow's quality bar.
- **`/workflows:author`** — build and amend workflows (the state machines items flow through)
  to good shape.
- **`/workflows:run-item`** — drive an item's transitions from your interactive session (also what the
  board's terminal handoff invokes).
- **`/workflows:discuss`** — talk to an item: understand its documents, Q&A, and pipeline position, and
  amend it safely out of the conversation.

Each command runs its underlying skill; the skills also auto-invoke when the agent recognizes the need.

## Install

```
/plugin marketplace add codingawayy/workflows-releases
/plugin install workflows
```

For local development against a checkout of this repo, build the plugin and load it directly:

```
bun run plugin:build          # assembles build/plugin/ (skills + bundled MCP server)
claude --plugin-dir build/plugin
```

**Prerequisite:** [Bun](https://bun.sh) on your PATH — the MCP server runs under Bun. If Bun isn't on your
PATH, the `workflows` server shows as failed under `/mcp` with a generic spawn error; install Bun from
[bun.sh](https://bun.sh), reopen your shell, and reload Claude Code.

## Connect

The plugin carries the intelligence and the MCP registration; it does **not** bake in a credential or a
project. Run this once **in your project repo**:

```
/workflows:connect <apiUrl> <token> <projectId>
```

Your board's **Setup** page (`/setup`) hands you the `apiUrl`, `token`, and `projectId` — pick your
project there and it gives you a complete `workflows://connect?apiUrl=…&token=…&projectId=…` link
carrying all three. **Copy that link (don't click it)** and pass it to `/workflows:connect`. `/connect`
writes the per-machine connection (`apiUrl`, compatibility bearer, and board-discovered public
`workosClientId`) to `~/.workflows/client.json` and the per-repo `projectId` to this
repo's `.workflows/config.json` — both of which the `workflows` MCP server needs to start and
authenticate against your board.

(If you also run `app.tray` for auto-run, its tokenless connect flow discovers and writes the same
per-machine WorkOS binding — you only connect the machine once, but each repo needs its `projectId`.)
