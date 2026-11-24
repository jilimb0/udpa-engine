# ADR-0002: Deterministic Architecture Diff Artifact in UDPA Engine

**Status:** Accepted  
**Date:** 2026-11-24  
**Deciders:** Core Engineering Team  
**Applies-To:** `action.yml`, `engine/**`, `.github/**`  
**Supersedes:** None  
**Overrides-Global:** None  

## Y-Summary
In the context of `jilimb0/udpa-engine@v1` PR gate runs, facing the need to track cross-layer architectural drift without printing repository contents or breaking backwards compatibility, we decided for bundling Gavel's zero-token `floorplan-diff` command inside `engine/gavel-engine.cjs` and emitting `architecture-diff.json` via `architecture-diff-output` in `action.yml`, neglecting consumer-side builds or source-printing logs, to achieve deterministic PR topology diffs alongside SARIF 2.1.0 and Ed25519 passports, accepting a minor bundle size increase inside the 2.5 MB budget.

## Context and Problem Statement
CI consumers using `jilimb0/udpa-engine@v1` receive security SARIF and Ed25519 passport outputs, but lacked a deterministic architectural diff artifact (`architecture-diff.json`) showing newly added/removed nodes, edges, and integer `bps` delta metrics on pull requests.

## Decision Drivers
- **ADR-0001:** Never edit `engine/gavel-engine.cjs` by hand; build in `gavel` and copy here; keep `fail-on-breach: 'true'` default and zero runtime network calls.
- **Axis 5 (Zero-Leakage):** Never print file contents to CI logs; omit secrets and PII from `architecture-diff.json`.

## Imperative Constraints for AI Agents (MUST / MUST NOT)
- **MUST:** Rebuild `engine/gavel-engine.cjs` exclusively from `services-and-apps/gavel` via `pnpm build:udpa-engine` (self-contained ~5.0 MB bundle with `typescript` inline so zero `node_modules` are required at action runtime).
- **MUST:** Keep `architecture-diff-output` optional with default `'architecture-diff.json'` to remain backwards compatible on `@v1`.
- **MUST NOT:** Hand-edit `engine/gavel-engine.cjs` or vendor it into consumer repositories.
- **DO NOT RE-INTRODUCE:** Runtime `pnpm install`, `--external:typescript` in the action bundle, or source-code log output inside `action.yml`.

## Considered Options & Rejected Alternatives
1. **Option 1 (Chosen): Bundled `floorplan-diff` step in `action.yml` (`architecture-diff.json`)**
   - *Why chosen:* Zero-install execution, backwards compatible `@v1` output, and deterministic SHA-256 `snapshotId`.
2. **Option 2 (Rejected — DO NOT PROPOSE): Separate `npx floorplan` script in consumer workflows**
   - *Why rejected:* Duplicates CI setup, lacks Gavel's integer `bps`/secret-redaction invariants, and breaks single-gate distribution.

## Consequences & Trade-offs
- **Positive:** Every `udpa-engine@v1` run automatically emits `architecture-diff.json` with zero external runtime dependencies.
- **Negative / Cost:** Self-contained `engine/gavel-engine.cjs` with `typescript` bundled inline is ~5.0 MB (vs 1.63 MB externalized `@gavel/cli` binary).

## Verification Gate
- **Check Command:** `node engine/gavel-engine.cjs floorplan-diff . --out /tmp/udpa-arch-diff-test.json`
- **Code Anchors:** `action.yml`, `engine/gavel-engine.cjs`, `.github/workflows/self-test.yml`.
