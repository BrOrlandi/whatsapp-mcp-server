## Deprecation

This repository is `BrOrlandi/whatsapp-mcp-server` (renamed from `whatsapp-mcp` on 2026-11-17).
The server version is being retired in favour of WhatsApp MCP Local, which will later take the
`BrOrlandi/whatsapp-mcp` name; from then on that name is the other project. Keep every link,
`update.sh`, the version check and the update command on `whatsapp-mcp-server`, and the image
name fixed at `ghcr.io/brorlandi/whatsapp-mcp`. Only fixes ship from here; the migration guide
is `docs/migrar-para-local.md` (`docs/migrating-to-local.md`).

## Git workflow

<!-- claude-skill:commit default-branch-policy=direct -->

Work lands directly on the default branch. Commit to it without asking for confirmation, and do not
propose creating a feature branch first.

## Release & Changelog

Structured convention: `.claude/release.json` (read by the `release` skill). Process:
`docs/development.md#releasing`.

- **The version is the git tag** (`vX.Y.Z`, annotated, written by `just release X.Y.Z` once
  `CHANGELOG.md` has a `## X.Y.Z` section). There is no version file; builds stamp `git describe`.
- **The version follows WhatsApp MCP Local** (`BrOrlandi/whatsapp-mcp-local`): the same number
  means the same MCP tools and the same panel. Server-only differences are listed in
  `docs/mcp-tools.md#differences-from-local`.
- **Changelog**: `CHANGELOG.md`, in English, for whoever runs an instance, including what an
  update asks of it.
- **Publish**: pushing the tag runs `release.yml`, which builds the images (`latest` follows the
  newest tag, beta or not) and the GitHub release with the binaries. `update.sh` and the panel's
  Atualizar button move instances to the newest tag, so a tag reaches every instance that updates.
