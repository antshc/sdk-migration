# SDK Migration — Development Process & Verification Checklist

Checklist covering development workflow improvements and systematic checks for the Azure Fluent → ARM SDK migration.

## Pre-Implementation Checks

- [ ] **Provider scope defined** — Clear decision on encapsulation layer (Adapter vs Provider) before starting implementation
- [ ] **Integration test level decided** — Explicit decision whether tests target Adapter or Provider layer (avoid double coverage)
- [ ] **Fluent type leakage audit** — Audit all usages of Fluent SDK types above the Provider/Adapter layer; replace with SDK-agnostic DTOs before continuing
- [ ] **Transitive dependency audit** — Run `dotnet list package --include-transitive` before and after adding ARM packages; pin versions in `Directory.Packages.props`
- [ ] **Custom handler inventory** — Document all `DelegatingHandler` implementations that need ARM `HttpPipelinePolicy` equivalents

## Development Workflow Per Provider

- [ ] **One provider at a time** — Migrate isolated, fully-working providers (code + tests) rather than batch replacements
- [ ] **Tests-first against Fluent** — Extend/write adapter integration tests against existing Fluent SDK implementation before writing ARM code
- [ ] **Tests as source of truth** — Use passing Fluent tests as the behavioral contract the ARM implementation must satisfy
- [ ] **Run tests in debug mode** — Execute new tests at least once in debug mode to catch hidden/unexpected behavior
- [ ] **Both toggle states green** — All tests pass under both `useNewSdk=true` and `useNewSdk=false` before merge
- [ ] **DI registration verified** — Confirm new provider is registered in the DI container and loaded at runtime (not silently skipped)

## Code Review Quality Gates

- [ ] **Two-developer review** — Every ARM provider PR reviewed by at least two developers
- [ ] **Focused human review** — Reviewer explicitly checks: critical flows, concurrency behavior, error handling, retries, state management
- [ ] **Shared field checklist** — Reviewer verifies field lists, exception types, data types, nullability against documented contract
- [ ] **No dead-code tests merged** — Tests that cannot execute (missing DI, unresolved dependencies) are rejected
- [ ] **Agentic code flagged** — AI-generated code sections explicitly marked for deeper manual scrutiny of assumptions and edge cases

## Incremental Release Strategy

- [ ] **Release by resource type** — Enable ARM providers per resource type (VM, Disk, NIC, NSG…) rather than all-at-once
- [ ] **Read-before-write ordering** — Read-only providers released and validated first; write operations follow
- [ ] **Gradual rollout** — Deploy to limited environments/customers first; monitor before wider rollout
- [ ] **Rollback plan documented** — Clear strategy to revert each provider independently via feature toggle
- [ ] **Feature toggle per provider** — Each provider can be toggled independently, not a single global switch

## Continuous Validation & Monitoring

- [ ] **Daily automation runs** — Critical-flow E2E automation tests run daily for both toggle states
- [ ] **Run tests before release branch merge** — Both old and new test suites pass before merging to release
- [ ] **SDK-version-aware telemetry** — Metrics differentiate Fluent vs ARM code paths for issue attribution
- [ ] **Migration-specific alerts** — Dashboards surface ARM-path failures distinctly from general errors
- [ ] **Scale testing completed** — Performance comparison between old and new implementations under production-scale load

## QA Coordination

- [ ] **Early QA involvement** — QA validates affected features (e.g., core workflow operations) as each provider is completed, not only at final release
- [ ] **Both flag states tested by QA** — QA explicitly tests feature flag on and off to ensure neither path introduces regressions
- [ ] **Focused regression scope** — QA test plan scoped to the specific resource type/provider under migration, not the full system
- [ ] **Intermediate QA cycles** — Short validation cycles after each provider delivery rather than single end-of-migration regression pass

## Shadow / Dual-Run Validation

- [ ] **Dual-run mode available** — Support running both Fluent and ARM simultaneously for the same operation to compare results
- [ ] **Object graph diff** — Use `BeEquivalentTo` or structural comparison to diff Fluent vs ARM outputs for the same inputs
- [ ] **JSON resource comparison** — For created resources, compare JSON view of resource state between old and new implementations
- [ ] **Paging count assertions** — Compare total resource counts returned by both SDK paths for the same scope

## Knowledge Sharing

- [ ] **Migration checklist shared** — Single shared checklist (fields, exceptions, data types) accessible to all team members
- [ ] **Per-provider sign-off** — Each completed provider has documented sign-off confirming tests, review, and QA pass
- [ ] **Lessons learned captured** — Issues found during migration documented for subsequent provider work
