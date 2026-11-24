## Plugin

- Plugin name and version:
- Harnesses (claude, codex, antigravity):
- Source repository (link):
- What changed since the last version:

## Native binaries (only if the plugin ships `node_modules/` or `*.node` files)

- [ ] `plugins/**/node_modules/` is not ignored, and Git LFS is not used
- [ ] `THIRD_PARTY_NOTICES` is at the root of each plugin folder
- Package and version (for example `@napi-rs/keyring`):
- sha512 per platform build (compare with npm):

```
<platform>  <sha512>
```

## Checklist

- [ ] `node scripts/validate-marketplace.mjs .` passes
- [ ] Claude: `claude plugin validate .` and `claude plugin validate plugins/<name>` pass
- [ ] Codex: the plugin installs in Codex from a local marketplace
- [ ] Antigravity: `agy plugin validate --strict plugins/<name>` passes
- [ ] Commits are signed off (`git commit -s`)
