# Engineering Conventions: udpa-engine

## 1. Action Interface
- Keep input names stable; add new inputs with safe defaults. `min-score` stays as a legacy alias of `min-purity`.
- Every output declared in `action.yml` must be written to `GITHUB_OUTPUT`.

## 2. Release Process
- Rebuild the bundle in `gavel`, copy it to `engine/gavel-engine.cjs`, commit, then move the `v1` tag.
- Never push `v1` without the self-test workflow passing on `main`.

## 3. Privacy Baseline
- Public repo: no secrets, no private repo names beyond the documented `gavel` source.
- Logs must not echo scanned file contents; only scores and artifact paths.

## 4. Testing
- `.github/workflows/self-test.yml` runs the action on push and asserts the gate artifacts, not the assay verdict.
