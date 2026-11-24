# Hedera AI plugins marketplace

Hedera plugins for Claude Code, Claude Desktop, Devin, Codex, Codex Desktop, ChatGPT Desktop and
Antigravity. The marketplace is named `hedera`. It is open: anyone can propose a plugin with a pull
request (see [CONTRIBUTING.md](CONTRIBUTING.md)).

Each plugin is one folder, `plugins/<name>/`, with a manifest for each harness it supports. Each
harness reads only its own manifest, and finds the plugin through its own catalog:

| Harness                               | Catalog                                                                | Manifest in `plugins/<name>/`     |
| ------------------------------------- | ---------------------------------------------------------------------- | --------------------------------- |
| Claude Code, Claude Desktop, Devin    | [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json)   | `.claude-plugin/plugin.json`      |
| Codex, Codex Desktop, ChatGPT Desktop | [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json) | `.codex-plugin/plugin.json`       |
| Antigravity                           | none (Antigravity only reads catalogs Google hosts)                    | `.antigravity-plugin/plugin.json` |

Files the harnesses share, such as skills and bundled servers, are in the folder once. A file only
one harness needs gets a name the others ignore: point the Claude and Codex manifests' `mcpServers`
at their own file (for example `.claude-mcp.json` and `.codex-mcp.json`); Antigravity reads
`mcp_config.json`.

## Install

### Claude Code and Claude Desktop

```bash
claude plugin marketplace add hashgraph/hedera-ai-plugins-marketplace
claude plugin install <plugin>@hedera
```

To pick up new releases:

```bash
claude plugin marketplace update hedera
```

Claude Desktop adds this same GitHub repository as a marketplace.

### Codex, Codex Desktop and ChatGPT Desktop

```bash
codex plugin marketplace add hashgraph/hedera-ai-plugins-marketplace
codex plugin add <plugin>@hedera
```

To pick up new releases:

```bash
codex plugin marketplace upgrade hedera
```

Codex CLI, Codex Desktop and the ChatGPT Desktop Work tab share the same local configuration, so
installing once makes the plugin available in all of them.

### Devin

Devin has no marketplace step. It installs a plugin folder straight from the repository, and reads its
Claude manifest:

```bash
devin plugins install hashgraph/hedera-ai-plugins-marketplace#plugins/<plugin>
```

### Antigravity

Antigravity has no marketplace step either. It installs a plugin folder from its GitHub URL:

```bash
agy plugin install https://github.com/hashgraph/hedera-ai-plugins-marketplace/tree/main/plugins/<plugin>
```

Run the same command again to update. The Antigravity App loads plugins the CLI installed.

## Two ways to list a plugin

Claude and Codex plugins are one object each in the `plugins` array of their catalog. An entry
takes one of two forms. Antigravity plugins have no entry: the folder is the listing.

**1. In-repo.** The plugin's files live in this repository, under `plugins/<name>/`, with a
manifest for each harness, each with the same `name` and a semver `version`:

- Claude: `.claude-plugin/plugin.json`
- Codex: `.codex-plugin/plugin.json` (or a root `plugin.json`)
- Antigravity: `.antigravity-plugin/plugin.json`

Claude entry:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin",
  "description": "What it does."
}
```

Codex entry:

```json
{
  "name": "my-plugin",
  "description": "What it does.",
  "source": { "source": "local", "path": "./plugins/my-plugin" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "Developer Tools"
}
```

Users only receive an update when `version` in the plugin's manifest changes, so every update PR
bumps it. A plugin whose source lives in another repository can send its build output here with a
pull request, by hand or from its own release workflow.

**2. Pinned git (Claude and Codex).** The entry points at a public repository and a commit.
Nothing is copied here:

```json
{
  "name": "my-plugin",
  "description": "What it does.",
  "source": {
    "source": "github",
    "repo": "my-org/my-plugin",
    "ref": "v1.2.0",
    "sha": "0123456789abcdef0123456789abcdef01234567"
  }
}
```

Claude accepts `github`, `url` and `git-subdir`. Codex accepts `url` (plugin at the repository
root) and `git-subdir`, and its entries also need `policy` and `category`. `sha` is required and
must be the full 40-character commit. Reviewers approve exactly that commit, and an update is a pull
request that changes `sha`. Details are in the
[Claude Code marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference)
and the [Codex plugin docs](https://developers.openai.com/codex/plugins/build/).

## What CI checks

`scripts/validate-marketplace.mjs` runs on every pull request, together with `claude plugin validate`
on the Claude catalog and every Claude plugin folder:

- Both catalogs are named `hedera`. The Claude one has an owner, the Codex one a display name.
- Plugin names are valid and unique in each catalog, and every Codex entry has `policy` and
  `category`.
- An in-repo entry points at `./plugins/<name>`, that folder exists, and its manifest for that
  harness has the same `name` and a semver `version`.
- An Antigravity manifest has the folder's name and a semver `version`.
- A pinned git entry uses `https://` and a full `sha` that can be fetched.
- `plugins/` holds only plugin folders, each with at least one harness manifest. A folder with a
  Claude or Codex manifest has an entry in that catalog, and no folder holds key material, `.env`
  files or a keystore.

Run it locally with `node scripts/validate-marketplace.mjs . --check-remote`.

## Review

Maintainers approve every change through [CODEOWNERS](.github/CODEOWNERS). Sign off every commit for
the DCO (`git commit -s`). Pull request titles must follow
[Conventional Commits](https://www.conventionalcommits.org/).
