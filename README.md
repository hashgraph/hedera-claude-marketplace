# Hedera plugin marketplace for Claude

Hedera plugins for Claude Code, Claude Desktop and Devin. The marketplace is named `hedera`.

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

Devin has no marketplace step. It installs a plugin folder straight from the repository:

```bash
devin plugins install hashgraph/hedera-claude-marketplace#plugins/<plugin>
```

## Layout

Each plugin lives in `plugins/<name>/` and has one entry in the marketplace manifest.

## Plugins are built elsewhere

The source of each plugin lives in its own repository. `plugins/<name>/` is build output that
arrives through a release pull request from that repository. Do not edit it by hand: the next
release replaces it. Change the plugin's source repository instead.

A release pull request touches only its own `plugins/<name>/` folder and its own manifest entry.
Merging it publishes the release.

## Adding a plugin

Open a pull request that adds `plugins/<name>/` and its manifest entry, and say where the plugin's
source repository is. Maintainers review it through CODEOWNERS.

- Sign off every commit for the DCO: `git commit -s`.
- Pull request titles must follow [Conventional Commits](https://www.conventionalcommits.org/).
