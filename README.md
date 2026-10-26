# Yarn 4 resolutions lose `hasPackageManager`

A Yarn 4 repo (`packageManager: yarn@4.12.0`) with a scoped `resolutions` entry (`axios@0.22.0`).

Renovate proposes an update to the resolution. Its lock file update installs Yarn 1 instead of using corepack and fails:

```
the current global version of Yarn is 1.22.22
```

Updates to ordinary dependencies in the same repo use corepack and succeed.
