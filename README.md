# Workflows — releases

Public home for the downloadable **Workflows agent** (the Windows installer for the resident tray app
that bundles the `workflows` MCP server).

- **Source code is private** — it lives in `codingawayy/workflows`. Only built release assets are
  published here, so the installer is anonymously downloadable without a GitHub account.
- **Publishing is automated** — a tag-triggered CI workflow in the source repo builds the installer on a
  `windows-latest` runner and uploads it here, authenticated by a GitHub App (ephemeral per-run token,
  no stored PAT). See the source repo's `delivery/tray/CLAUDE.md` for the runbook.

## Download

The latest installer resolves from a stable URL:

```
https://github.com/codingawayy/workflows-releases/releases/latest/download/Workflows-Runner-setup.exe
```

The board's **Setup** page links here and pairs it with a one-click connect link.
