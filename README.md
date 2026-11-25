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

`github`, `url` and `git-subdir` are accepted. `ref` and `sha` are both required: `ref` is a release
tag and `sha` is the full 40-character commit that tag points to. Reviewers approve exactly that
commit, and an update is a pull request that changes both.
Details are in the [Claude Code marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference).

## What CI checks

`scripts/validate-marketplace.mjs` runs on every pull request, together with `claude plugin validate`:

- The marketplace is named `hedera` and has an owner.
- Plugin names are lowercase kebab-case, at most 64 characters, unique, and not reserved (such as
  `node_modules`).
- Entries carry only listing fields: `name`, `source`, `description`, `version`, `author`,
  `homepage`, `repository`, `license`, `keywords`, `category`, `tags` and `displayName`. Hooks,
  MCP servers, skills and other components go in the plugin's own files, where CI checks them.
- An in-repo entry points at `./plugins/<name>`, that folder exists, and its `plugin.json` has the
  same `name` and a semver `version`.
- Files under `plugins/` include no symlinks, submodules, `.gitattributes` drivers or invisible
  Unicode characters, and none is over 16 MB. The root `.gitattributes` sets no drivers either.
- A pinned git entry uses an `https://github.com/<owner>/<repo>` URL, a relative `path`, a tag
  `ref` and a full `sha`. With `--check-remote`, the tag must resolve to exactly that commit in the
  upstream repository. An `<owner>` that imitates `hashgraph` or `hiero-ledger` (such as
  `hashgraaph` or `hiero-1edger`) fails.
- Every folder in `plugins/` has an entry, and no folder holds key material, `.env` files or a
  keystore.
- On a pull request (`--base=origin/main`), an in-repo plugin whose files changed must raise its
  `version` in semver order, so a version never goes down.
- Hook, MCP server, LSP server and monitor commands don't run downloaded or base64-decoded content
  in a shell or interpreter, don't save downloaded content to a file, don't use `sudo`, and use no
  path outside the plugin (see the plugin security policy in [CONTRIBUTING.md](CONTRIBUTING.md)).

The job summary lists what each changed plugin runs: MCP and LSP servers, hooks (commands and
prompts), monitors, `bin/` files, scripts named in skills and HTML comments in Markdown. It
highlights a new plugin named `hedera-`, `hashgraph-` or `hiero-`, a changed source on an existing
entry, a new or re-pointed git entry whose owner isn't `hashgraph` or `hiero-ledger`, and a remote
MCP server on a host the marketplace doesn't already use. These don't fail the run; reviewers check
them.

Run it locally with `node scripts/validate-marketplace.mjs . --check-remote --base=origin/main`.

## Review

Maintainers approve every change through [CODEOWNERS](.github/CODEOWNERS). Sign off every commit for
the DCO (`git commit -s`). Pull request titles must follow
[Conventional Commits](https://www.conventionalcommits.org/).
