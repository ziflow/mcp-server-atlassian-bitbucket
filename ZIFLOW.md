# Ziflow fork

This is Ziflow's frozen copy of [aashari/mcp-server-atlassian-bitbucket](https://github.com/aashari/mcp-server-atlassian-bitbucket). It exists so our Claude Code and Codex clients run a reviewed version instead of whatever is currently on npm.

## How clients use it

Point the MCP server at the `stable` branch instead of the npm package:

```
npx --no-audit github:ziflow/mcp-server-atlassian-bitbucket#stable
```

`--no-audit` matters: after a fresh install npx runs `npm audit`, which can hang for a minute or more against some registries. Without it the first start after a bump takes 60s+ instead of ~25s.

`stable` is the only supported ref. Clients re-resolve it on every start, so moving the branch is enough to roll a new version out to everyone.

## What differs from upstream

`stable` = an upstream release tag + two commits:

1. `package-lock.json` renamed to `npm-shrinkwrap.json`.
2. `"bundleDependencies": true` in `package.json`, so the production dependency tree resolved from the shrinkwrap is packed with the server. npm does not honor a git dependency's shrinkwrap on its own, so without this the transitive dependencies would float.

No functional changes.

## Bumping the version

```
git remote add upstream https://github.com/aashari/mcp-server-atlassian-bitbucket.git   # once
git fetch upstream --tags
git diff stable upstream/vNEW -- . ':!package-lock.json'   # review what changed
git rebase --onto vNEW vOLD stable                          # carry our two commits over
git push origin stable
```

`stable` is protected against force-push and deletion, so a rebase that rewrites history needs the protection lifted temporarily, or use a merge instead.
