# Common Development Tasks: udpa-engine

## 1. Update the bundled engine
1. In `gavel`, run `pnpm build:udpa-engine` to build `dist/bin/gavel-engine.cjs` (bundles `typescript` inline).
2. Copy `dist/bin/gavel-engine.cjs` to `engine/gavel-engine.cjs`.
3. Commit here and wait for the self-test workflow to pass.
4. Move the tag manually (no automatic push): `git tag -f v1` then push after owner approval.

## 2. Use the gate in a project
Add `uses: jilimb0/udpa-engine@v1` with `target-path`, `min-score`, and optionally `fail-on-breach: 'false'` for advisory runs.

## 3. Add an input
1. Add it to `action.yml` with a default that preserves current behavior.
2. Pass it through in the `run` step.
3. Document it in `README.md` and verify via the self-test workflow.
