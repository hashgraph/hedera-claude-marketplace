# Contributing a plugin

Anyone can propose a plugin by pull request. Maintainers review every one (see
[CODEOWNERS](.github/CODEOWNERS)). The README explains the layout and the two forms an entry can
take; this page is the checklist.

A plugin can ship for any of the harnesses: Claude (also Devin), Codex, Antigravity. It is one
folder, `plugins/<name>/`, with a manifest for each harness it supports. Use the same `<name>` in
all of them.

## Before you open a PR

- Your plugin works in each harness you ship it for:
  - Claude: `claude plugin validate <your-plugin-dir>` passes.
  - Codex: it works when installed in Codex from a local marketplace.
  - Antigravity: `agy plugin validate --strict <your-plugin-dir>` passes.
- It has a README and a LICENSE, and contains no secrets, `.env` files, key files or keystores.
- Everything in it is meant to be public. This repository is.

## Form 1: add the plugin to this repository

1. Fork the repository and create a branch.
2. Copy the plugin to `plugins/<name>/`, with a manifest for each harness. Each must have
   `"name": "<name>"` and a semver `"version"`:
   - Claude: `.claude-plugin/plugin.json`.
   - Codex: `.codex-plugin/plugin.json` (or a root `plugin.json`).
   - Antigravity: `.antigravity-plugin/plugin.json`.

   Keep shared files (skills, servers) once. When the harnesses need different MCP configs, point
   the Claude and Codex manifests' `mcpServers` at their own files; Antigravity reads
   `mcp_config.json`.

3. List the plugin in the Claude and Codex catalogs, if it has those manifests. Antigravity has no
   catalog; its manifest is enough.
   - Claude: add one object to `plugins` in `.claude-plugin/marketplace.json`:
     `{ "name": "<name>", "source": "./plugins/<name>", "description": "…" }`.
   - Codex: add one object to `plugins` in `.agents/plugins/marketplace.json`, with `source` set to
     `{ "source": "local", "path": "./plugins/<name>" }`, plus `policy` (for example
     `{ "installation": "AVAILABLE", "authentication": "ON_INSTALL" }`) and `category`.
4. Run `node scripts/validate-marketplace.mjs .` and `claude plugin validate .`.

To update the plugin later, replace the folder's contents and **bump `version`**. Users only receive
an update when it changes. If the folder is generated from another repository, say so in the PR
description, and make changes in that repository.

## Form 2: pin a plugin from your own repository (Claude and Codex)

1. Fork the repository and create a branch.
2. Add one object to `plugins` in the catalog, with `sha` set to the full 40-character commit you
   want users to get:
   - Claude, `.claude-plugin/marketplace.json`: a `github`, `url` or `git-subdir` source.
   - Codex, `.agents/plugins/marketplace.json`: a `url` or `git-subdir` source, plus `policy` and
     `category`.
3. Run `node scripts/validate-marketplace.mjs . --check-remote`. The repository must be public.

To update the plugin later, open a PR that changes `sha` (and `ref`, if you use it).

## Plugins that ship `node_modules/` or native binaries

Some plugins commit their built dependencies, including native `*.node` libraries. For those:

- Do not ignore `plugins/**/node_modules/` in any `.gitignore`.
- Do not use Git LFS. Clones do not download LFS files, so users would get pointer files.
- Put one `THIRD_PARTY_NOTICES` file at the root of each plugin folder (`plugins/<name>/`).
  It must cover every bundled package's licence, the compiled libraries inside the `.node` files,
  and any third-party JS you bundled.
- Reviewers cannot read binaries (the diff only says "Binary files differ"). In the PR description,
  list each bundled native package's version and the sha512 of every platform build, so they can be
  compared against npm.

## In the pull request

- Title in [Conventional Commits](https://www.conventionalcommits.org/) form, for example
  `feat(my-plugin): add my-plugin`.
- Every commit signed off for the DCO: `git commit -s`.
- A link to the plugin's source repository, the harnesses it ships for, and what changed since the
  last version.
