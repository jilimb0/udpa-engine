# ADR-0001: Bundled Gavel Engine as a Tag-Versioned Composite Action

**Status:** Accepted  
**Date:** 2026-10-30  
**Deciders:** Core Engineering Team  
**Applies-To:** `action.yml`, `engine/**`, `.github/**`  
**Supersedes:** None  
**Overrides-Global:** None  

## Y-Summary
In the context of `udpa-engine` (`action.yml`, `engine/gavel-engine.cjs`), facing the same quality gate being needed in many repositories (previously vendored per repo, and drifting or referencing a non-existent local path), we decided for **one public composite action with a pre-bundled single-file Gavel CJS engine, consumed by tag (`@v1`)** instead of vendoring the engine into each repository, to keep one source of truth with zero install steps, accepting a manual rebuild-and-retag release process.

## Context and Problem Statement
Consumer `udpa-gate` workflows once pointed at a local `services-and-apps/udpa-engine` action that did not exist in CI. The gate needs to run anywhere without `pnpm install` or access to the private `gavel` source.

## Decision Drivers
- Single source of truth for gate logic and versioning
- No build or dependency installation at runtime
- Advisory (warning-only) mode while projects are below target score

## Imperative Constraints for AI Agents (MUST / MUST NOT)
- **MUST:** Keep the engine as a single bundled file built from `gavel`; do not hand-edit it.
- **MUST:** Keep `fail-on-breach` default `true`; advisory mode is opt-in per consumer.
- **MUST:** Preserve existing input names and outputs within `v1`.
- **MUST NOT:** Never vendor `engine/` into consumer repositories.
- **MUST NOT:** Never push tags or commits automatically; releases require owner approval.

## Considered Options & Rejected Alternatives
1. **Option 1 (Chosen): Public composite action with bundled CJS engine, tag-versioned**
   - *Why chosen:* Zero setup for consumers, one place to fix, works with a private engine source.
2. **Option 2 (Rejected — DO NOT PROPOSE): Vendor the engine in every repository**
   - *Why rejected:* Version drift across 15+ repos and large binary-like diffs.
3. **Option 3 (Rejected — DO NOT PROPOSE): Local path action inside a monorepo checkout**
   - *Why rejected:* The path does not exist in consumer CI, which broke earlier gates.

## Consequences & Trade-offs
- **Positive:** Fixes propagate by moving one tag; consumers need two lines of YAML.
- **Negative / Cost:** Bundle must be rebuilt and `v1` retagged manually after Gavel changes.

## Verification Gate
- **Check Command:** the `self-test.yml` workflow on push to `main`
- **Code Anchors:** `action.yml`, `engine/gavel-engine.cjs`
