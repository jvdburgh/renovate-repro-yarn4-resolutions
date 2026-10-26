# Renovate uses Yarn 1 for a Yarn 4 `resolutions` update

Minimal reproduction for a Renovate bug.

This repo uses Yarn 4 through corepack (`"packageManager": "yarn@4.12.0"`) and has one `resolutions` entry scoped to a version: `"axios@0.22.0": "0.30.0"`.

## Expected

Renovate updates the resolution and regenerates `yarn.lock` with Yarn 4, as it does for ordinary dependencies.

## Actual

For the resolution update, Renovate installs Yarn 1 instead of using corepack, and the lock file update fails:

```
the current global version of Yarn is 1.22.22
```

We saw this on self-hosted Renovate 44.173.0, which installs tools on demand (`binarySource=install`). In the debug log, Renovate reads `yarn@4.12.0` from `packageManager` but still installs Yarn `^1.22.18`.

## Cause

A scoped resolution carries its own `managerData` (`{ key: "axios@0.22.0" }`). When Renovate builds the update, that replaces the package file's `managerData` instead of adding to it, so the update loses `hasPackageManager` and Renovate falls back to Yarn 1.
