# connector-ci

Shared GitHub Actions workflows for the Now Playing DJ connector libraries
(`stagelinq`, `rekordbox-connect`, `serato-connect`, `djay-connect`,
`traktor-connect`, `virtualdj-connect`, `onelibrary-connect`,
`metadata-connect`, `alphatheta-connect`).

Each connector used to carry its own copy of a ~190-line publish workflow. They
drifted, silently, and three of them stopped publishing to npm for months
without anyone noticing — the packages kept building from source inside the
monorepo, so nothing failed. The versions here are the single copy those repos
now call.

## Usage

**`.github/workflows/publish.yml`** in a connector repo:

```yaml
name: Publish to npm

on:
  push:
    branches: [main]

jobs:
  publish:
    permissions:
      contents: write
      id-token: write
    uses: chrisle/connector-ci/.github/workflows/node-publish.yml@main
    secrets: inherit
```

**`.github/workflows/test.yml`**:

```yaml
name: Test

on:
  push:
    branches: ['**', '!main']
  pull_request:
    branches: [main]

jobs:
  test:
    uses: chrisle/connector-ci/.github/workflows/node-test.yml@main
```

### Keep the caller named `publish.yml`

npm's trusted publishing validates the **calling** workflow's filename, not the
reusable one, and needs `id-token: write` at both levels. Renaming a caller
breaks publishing for that package until its trusted publisher config on npmjs
is updated to match.

## Inputs

| input | default | |
|---|---|---|
| `node-version` | `24` | Matches the desktop app's `.nvmrc` and Electron's embedded Node |
| `run-build` | `true` | Set false for a package with no `build` script |

Node 24 is deliberate on both workflows. npm 12 blocks dependency install
scripts unless the package lists them in `allowScripts`; older npm still runs
them. A test job on an older Node therefore goes green over breakage that only
appears when the publish job runs — which is how `djay-connect` shipped nothing
for two months.
