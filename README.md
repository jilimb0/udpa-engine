# UDPA Engine (bundled Gavel v1.0.0)

Composite GitHub Action implementing the **UDPA Institutional Quality & Security Gate**.

This repository is the **single source of truth** for the engine. Projects reference it
by tag — nothing needs to be vendored into each repository:

```yaml
- uses: jilimb0/udpa-engine@v1
  with:
    target-path: '.'
    min-score: '85'
    stage: 'Scale'
    sarif-output: 'udpa-audit.sarif'
    passport-output: 'udpa-zk-passport.json'
    architecture-diff-output: 'architecture-diff.json'
```

Outputs: `sarif-file`, `passport-file`, `architecture-diff-file`, `score`.

### Advisory mode

By default a score below `min-purity` fails the step. Set `fail-on-breach: 'false'`
to finish successfully with a `::warning::` annotation, a job-summary line and the
SARIF/passport artifacts instead — useful while a project is below the target score:

```yaml
- uses: jilimb0/udpa-engine@v1
  with:
    target-path: '.'
    min-score: '85'
    fail-on-breach: 'false'
```

## Contents

- `action.yml` — composite action definition (inputs/outputs mirror the upstream Gavel action).
- `engine/gavel-engine.cjs` — single-file CJS bundle of the Gavel CLI + `@gavel/core` (esbuild).
  No build step, no `pnpm install` required.

## Recommended workflow

```yaml
permissions:
  contents: read
  security-events: write
  actions: read

steps:
  - uses: actions/checkout@v4
  - uses: jilimb0/udpa-engine@v1
    with:
      target-path: '.'
      min-score: '85'
      stage: 'Scale'
  - uses: github/codeql-action/upload-sarif@v4
    if: always() && hashFiles('udpa-audit.sarif') != ''
    continue-on-error: true
    with:
      sarif_file: 'udpa-audit.sarif'
```

## Updating the bundle

Rebuild the self-contained CLI bundle in the private `gavel` repository and copy it here:

```sh
pnpm build:udpa-engine
cp dist/bin/gavel-engine.cjs ../udpa-engine/engine/gavel-engine.cjs
```

Then commit, push, and update the `v1` tag:

```sh
git tag -f v1 && git push -f origin v1
```
