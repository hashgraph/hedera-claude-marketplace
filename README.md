# Hedera plugin marketplace for Claude

Hedera plugins for Claude Code, Claude Desktop and Devin. The marketplace is named `hedera`. It is
open: anyone can propose a plugin with a pull request (see [CONTRIBUTING.md](CONTRIBUTING.md)).

The available plugins are the entries in
[`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## Install

Claude Code:

```bash
claude plugin marketplace add hashgraph/hedera-claude-marketplace
claude plugin install <plugin>@hedera
```

To pick up new releases:

```bash
claude plugin marketplace update hedera
```

Claude Desktop adds this same GitHub repository as a marketplace.

Devin has no marketplace step. It installs an in-repo plugin folder straight from the repository:

```bash
devin plugins install hashgraph/hedera-claude-marketplace#plugins/<plugin>
```

## Two ways to list a plugin

Every plugin is one object in the `plugins` array of the manifest. It takes one of two forms.

**1. In-repo.** The plugin's files live in this repository, under `plugins/<name>/`, with a
`.claude-plugin/plugin.json` that has the same `name` and a semver `version`:

```json
{ "name": "my-plugin", "source": "./plugins/my-plugin", "description": "What it does." }
```

Users only receive an update when `version` in the plugin's `plugin.json` changes, so every update
PR bumps it. A plugin whose source lives in another repository can send its build output here with
a pull request, by hand or from its own release workflow.

**2. Pinned git.** The entry points at a public repository and a commit. Nothing is copied here:

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

`github`, `url` and `git-subdir` are accepted. `sha` is required and must be the full 40-character
commit. Reviewers approve exactly that commit, and an update is a pull request that changes `sha`.
Details are in the [Claude Code marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference).

## What CI checks

`scripts/validate-marketplace.mjs` runs on every pull request, together with `claude plugin validate`:

- The marketplace is named `hedera` and has an owner.
- Plugin names are lowercase kebab-case, at most 64 characters, unique, and not reserved (such as
  `node_modules`).
- An in-repo entry points at `./plugins/<name>`, that folder exists, and its `plugin.json` has the
  same `name` and a semver `version`.
- Files under `plugins/` include no symlinks, submodules, `.gitattributes` drivers or invisible
  Unicode characters, and none is over 16 MB.
- A pinned git entry uses an `https://github.com/<owner>/<repo>` URL, a relative `path` and a
  full `sha` that can be fetched.
- Every folder in `plugins/` has an entry, and no folder holds key material, `.env` files or a
  keystore.

Run it locally with `node scripts/validate-marketplace.mjs . --check-remote`.

## Review

Maintainers approve every change through [CODEOWNERS](.github/CODEOWNERS). Sign off every commit for
the DCO (`git commit -s`). Pull request titles must follow
[Conventional Commits](https://www.conventionalcommits.org/).
