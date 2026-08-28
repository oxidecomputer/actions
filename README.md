# actions

Shared GitHub Actions for Oxide repos. Each action lives in its own
subdirectory and is referenced as `oxidecomputer/actions/<name>@<ref>`.

## setup-node

Installs Node, installs the npm version we require (npm 12 by default), and
caches the npm download directory.

Why this exists: our repos require npm ≥12 via `devEngines.packageManager` in
`package.json` for its [supply-chain protections][npm-12-changes] (dependency
lifecycle scripts blocked by default and `allow-git`/`allow-remote` defaulting
to `none`) and [`min-release-age`][min-release-age] support. No shipping Node
release bundles npm 12 — Node 22 bundles npm 10, Node 24/26 bundle npm 11 — and
`actions/setup-node`'s built-in caching runs `npm config get cache` with the
bundled npm, which fails devEngines validation before a workflow gets a chance
to upgrade npm. See
[actions/setup-node#1553](https://github.com/actions/setup-node/issues/1553).

This action disables setup-node's caching, installs the required npm globally,
then restores/saves the npm cache directory itself, keyed on the runner OS and
lockfile hash.

Usage:

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: oxidecomputer/actions/setup-node@main
    with:
      node-version: '24' # default
      npm-version: '12' # default
```

Repos using this should also require npm in `package.json` so the requirement is
enforced locally, not just in CI:

```json
"engines": {
  "node": ">=24"
},
"devEngines": {
  "packageManager": {
    "name": "npm",
    "version": ">=12",
    "onFail": "error"
  }
}
```

Because npm 12 blocks dependency lifecycle scripts by default, repos that need
them should run `npm approve-scripts --allow-scripts-pending`, approve the
required packages with `npm approve-scripts`, and commit the resulting
`allowScripts` configuration.

Local dev setup on stock Node (which bundles npm 10 or 11): `npm install
--global npm@12`.

When to delete this action: once the org's Node floor reaches a supported Node
LTS release that bundles npm 12 and
[actions/setup-node#1553](https://github.com/actions/setup-node/issues/1553)
is resolved or moot, consumers can switch back to plain `actions/setup-node`
with `cache: npm`.

[npm-12-changes]: https://github.blog/changelog/2026-06-09-upcoming-breaking-changes-for-npm-v12/
[min-release-age]: https://docs.npmjs.com/cli/v12/using-npm/config#min-release-age
