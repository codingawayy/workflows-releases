# Workflows for Codex

The complete guide is <https://workflows-docs.web.app/get-started/install-the-agent#win11-pwsh-codex>.

Install this plugin from the Workflows marketplace, then use `$workflows:connect` to connect a repository. The plugin requires `bun` on `PATH` and provides `$workflows:add-item`, `$workflows:author`, `$workflows:build-skills`, `$workflows:run-item`, `$workflows:discuss`, and `$workflows:unblock`.

To let an agent pick up and run items unattended, follow <https://workflows-docs.web.app/get-started/connect-your-repository#set-up-auto-run>. The agent that runs unattended is selected separately from the interactive client.

Operator smoke check for each release: add the current Workflows marketplace with `codex plugin marketplace add`, install with `codex plugin add workflows@workflows`, run `$workflows:connect`, restart Codex once, confirm the six channel skills are discoverable, and confirm the bundled `workflows` MCP server lists its tools. Deterministic package guards remain the merge gate.
