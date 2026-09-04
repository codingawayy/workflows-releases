# Workflows — Claude Code plugin

The agent's workflow intelligence, as one installable Claude Code plugin. It is the **floor** to use
the product interactively — no tray required. Installing it gives you:

- **The `workflows` MCP server** — the in-session tools to read and drive workflow items over your
  board's API.
- **`/workflows:add-item`** — turn natural-language work into well-recorded items: broken down, routed to
  the right workflow, and recorded to that workflow's quality bar.
- **`/workflows:author`** — build and amend workflows (the state machines items flow through)
  to good shape.
- **`/workflows:build-skills`** — inspect the project's workflows and reconcile project-specific
  commands and skills for interactive, automatic, and routed transitions.
- **`/workflows:run-item`** — drive an item's transitions from your interactive session.
- **`/workflows:discuss`** — talk to an item: understand its documents, Q&A, and pipeline position, and
  amend it safely out of the conversation.
- **`/workflows:unblock`** — inspect and resolve the project's human-intervention queue.

Each command runs its underlying skill; the skills also auto-invoke when the agent recognizes the need.

## Install

The complete guide is
<https://workflows-docs.web.app/get-started/install-the-agent#win11-pwsh-claude-code>.

```
/plugin marketplace add codingawayy/workflows-releases
/plugin install workflows
```

For local development against a checkout of this repo, build the plugin and load it directly:

```
bun run plugin:build          # assembles build/plugin/ (skills + bundled MCP server)
claude --plugin-dir build/plugin
```

**Prerequisite:** [Bun](https://bun.sh) on your PATH — the MCP server runs under Bun.

## Connect

The plugin carries the intelligence and the MCP registration; it does **not** bake in a credential or a
project. Run this once **in your project repo**:

```
/workflows:connect <apiUrl> <projectId>
```

Your board's **Connect agent** page gives you the board URL and project id as one copyable value. Pass
that value to `/workflows:connect`. What connecting writes, how to prove it worked, and what to do when it did not
are at
<https://workflows-docs.web.app/get-started/connect-your-repository#prove-the-connection>.

## Auto-run

To let an agent pick up and run items unattended, follow
<https://workflows-docs.web.app/get-started/connect-your-repository#set-up-auto-run>. The agent that
runs unattended is selected separately from the plugin you use to steer Workflows.
