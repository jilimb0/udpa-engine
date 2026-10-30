# Architecture: udpa-engine

## 1. Identity & Domain
udpa-engine — composite GitHub Action `jilimb0/udpa-engine@v1`: UDPA quality & security gate that runs a bundled Gavel CLI (deterministic assay, SARIF 2.1.0, Ed25519 passport). Single source of truth for the gate; consumer repos reference it by tag and vendor nothing.
- **Status:** active
- **Remote:** `git@github.com:jilimb0/udpa-engine.git`
- **Default Branch:** `main`

## 2. Stack & Runtime
- **Runtime:** GitHub composite action (bash) on Node 22; engine is a single-file CJS bundle (esbuild) of the Gavel CLI.
- **Files:** `action.yml`, `engine/gavel-engine.cjs`, `.github/workflows/self-test.yml`

## 3. Structural Invariants
- The engine bundle is built in the private `gavel` repository and copied here; never edit `engine/gavel-engine.cjs` by hand.
- No build step and no `pnpm install` at action runtime.
- Public repository: the bundle and workflows must contain no secrets, tokens or customer data.
- Breaking input/output changes require a new major tag (`v2`); `v1` is moved only for compatible fixes.

## 4. Data Flows & External Boundaries
- Inputs: target path, min purity, stage, output paths, `fail-on-breach`. Outputs: `sarif-file`, `passport-file`, `score`.
- Used by `udpa-gate` workflows in consumer repositories; no network calls at runtime.

## 5. Active ADRs
- **[ADR-0001](.ai/adr/0001-bundled-gavel-engine-as-versioned-composite-action.md)** (`action.yml`, `engine/**`, `.github/**`): Engine ships as a pre-bundled CJS file behind a tag-versioned composite action, with advisory mode via `fail-on-breach`.
