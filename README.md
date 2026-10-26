# Renovate uses Yarn 1 for a Yarn 4 `resolutions` update

Minimal reproduction for a Renovate bug.

This repo uses Yarn 4 through corepack (`"packageManager": "yarn@4.12.0"`) and has one `resolutions` entry scoped to a range: `"follow-redirects@^1.14.4": "1.15.0"`. follow-redirects comes in through axios, so Renovate gives the resolution update its own branch, `renovate/follow-redirects-1.x`.

## Expected

Renovate updates the resolution and regenerates `yarn.lock` with Yarn 4, as it does for the other branches in this repo.

## Actual

For the resolution's branch, Renovate installs Yarn 1 instead of using corepack, and the lock file update fails:

```
error This project's package.json defines "packageManager": "yarn@4.12.0". However the current global version of Yarn is 1.22.22.
```

The debug log shows Renovate reading `yarn@4.12.0` from `packageManager`, then installing Yarn `^1.22.18`. The axios branches in the same run use corepack and succeed.

## Run it

With the Renovate Docker image, which installs tools on demand (`binarySource=install`), in full dry-run mode so nothing is written:

```sh
docker run --rm \
  -e RENOVATE_TOKEN="$(gh auth token)" \
  -e RENOVATE_DRY_RUN=full \
  -e RENOVATE_REPOSITORIES=jvdburgh/renovate-repro-yarn4-resolutions \
  -e RENOVATE_FORCE='{"prHourlyLimit":0,"prConcurrentLimit":0,"branchConcurrentLimit":0}' \
  -e LOG_LEVEL=debug \
  ghcr.io/renovatebot/renovate:44.173.0
```

Look for `install-tool yarn 1.22.22` and `lock file error` on `renovate/follow-redirects-1.x`.

## Cause

A scoped resolution carries its own `managerData` (`{ key: "follow-redirects@^1.14.4" }`). When Renovate builds the update, that replaces the package file's `managerData` instead of adding to it, so the update loses `hasPackageManager` and Renovate falls back to Yarn 1.

With the fix from renovatebot/renovate#47419 applied to the same 44.173.0 image, the branch installs corepack instead and the log reports `Updated 1 lock files`.
