# Security

## Report a vulnerability

To report a vulnerability in this repository or in a plugin it lists, follow the
[Hashgraph security policy](https://github.com/hashgraph/.github/blob/main/SECURITY.md). Report a
malicious or compromised plugin the same way. Don't open a public issue or pull request about it.

## What a listing means

Every change to this repository needs an approving review in a pull request. CI checks what the
README lists under [What CI checks](README.md#what-ci-checks). A listing isn't a security audit of
the plugin.

## Withdraw a plugin

When a listed plugin is malicious or compromised, maintainers withdraw it in this order:

1. Merge a stub of the plugin with a higher `version`. The stub has no hooks, MCP servers or skills,
   and its description says the plugin was withdrawn and why. Users who update get the stub. It
   protects them until step 3 reaches them.
2. Publish an advisory that names the affected versions.
3. Once the stub has been out long enough, remove the plugin's entry from
   `.claude-plugin/marketplace.json` and add `"<plugin>": null` to the top-level `renames` map
   (create the map if it's missing).

## Copies users already installed

Step 3 reaches a user once their copy of the marketplace has it. That copy refreshes when they run
`claude plugin marketplace update hedera`, or on its own if they turned on auto-update for this
marketplace. Claude Code then drops the plugin from their enabled plugins (the `renames` entry) and,
because this marketplace sets `forceRemoveDeletedPlugins`, uninstalls it from the user, project and
local scopes at the next session start. A plugin installed through managed settings stays until an
admin removes it.

Until then the plugin stays installed. Users who don't want to wait can uninstall it themselves:

```bash
claude plugin uninstall <plugin>@hedera
```

Add `--scope project` or `--scope local` if they installed it at that scope.
