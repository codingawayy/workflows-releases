# Workflows for Codex

Install this plugin from the Workflows marketplace, then use `$workflows.connect` to connect a repository. The plugin requires `bun` on `PATH` and provides `$workflows.add-item`, `$workflows.author`, `$workflows.run-item`, and `$workflows.discuss`.

Operator smoke check for each release: add the current Workflows marketplace with `codex plugin marketplace add`, install with `codex plugin add workflows@workflows`, run `$workflows.connect`, restart Codex once, confirm the four channel skills are discoverable, and confirm the bundled `workflows` MCP server lists its tools. Deterministic package guards remain the merge gate.
