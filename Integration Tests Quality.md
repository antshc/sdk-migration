# Adapter Integration Tests Quality Checklist

Checklist to verify adapter integration tests detect drift during migration from Azure Fluent SDK to ARM SDK.

## Default Values & Resource State
- [ ] **Write operation verification** — Create/update/delete operations produce identical resource state
- [ ] **Implicit defaults** — Validate SDK default values and initialization behavior haven't changed silently
- [ ] **Configuration behavior** — Ensure configuration parameters produce equivalent resource state

## Data Contract & DTO Integrity

- [ ] **Object graph comparison** — Tests use deep structural comparison (e.g., `BeEquivalentTo`) between Fluent and ARM provider outputs for the same operation
- [ ] **Null vs empty collection handling** — Validate that collections returned as `null` by ARM SDK are handled identically to empty collections from Fluent SDK
- [ ] **Field presence and nullability** — Assert all DTO fields populated by Fluent are also populated by ARM (no silent null leakage)
- [ ] **Numeric/string format consistency** — Verify value formats match (e.g., size in bytes vs GB, enum vs string representations)
- [ ] **Auto-generated resource names** — Confirm ARM path explicitly handles resource name generation that Fluent did automatically

## Exception & Error Handling

- [ ] **Exception type mapping** — Validate that ARM SDK exceptions map to the same exception types expected by higher layers
- [ ] **HTTP status code mapping** — Verify error status codes (404, 409, 429, etc.) produce equivalent exception behavior
- [ ] **Error message contract** — Ensure error details/messages are accessible through the same contract

## Paging & Data Completeness

- [ ] **Paging completeness** — Compare total resource counts between Fluent `IPagedCollection<T>` and ARM `AsyncPageable<T>` for the same subscription/scope
- [ ] **No silent truncation** — Verify large result sets are fully enumerated without data loss
- [ ] **Resource Graph query results** — Diff deserialized results field-by-field between Fluent and ARM paths for same KQL queries

## Retry & Resilience

- [ ] **Retry policy equivalence** — Validate that transient failure retry behavior matches between SDK implementations
- [ ] **Throttling (429) handling** — Confirm ARM pipeline policies handle throttling equivalently to Fluent delegating handlers
- [ ] **Transient failure scenarios** — Test network errors, timeouts, and temporary unavailability produce equivalent recovery



