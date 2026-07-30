# Production Bug Pattern Checklist

Use this only after tracing the actual code path. It is a hypothesis generator, not a substitute for evidence.

## Symptom-directed checks

| Symptom | High-value checks |
|---|---|
| Production-only | Effective config/flags, schema or index drift, runtime/dependency versions, permissions, timezone/locale, traffic/load, production data shape, cache/queue topology, network policy |
| One tenant/user/record | Tenant scoping, soft deletes, legacy formats, null/empty values, Unicode/case, oversized inputs, orphaned state, partial migrations, authorization cache |
| Intermittent | Races, check-then-act, lost updates, shared mutable state, async ordering, retry timing, duplicate delivery, clock boundaries, pool exhaustion, flaky dependency |
| Duplicate side effects | At-least-once delivery, retry after commit/timeout, missing idempotency key, non-atomic check/write, absent uniqueness constraint, consumer concurrency |
| Missing or stale data | Cache invalidation, replica lag, read-after-write, isolation level, eventual consistency, pagination/cursor bugs, swallowed partial failure, stale materialized view |
| Slow or timing out | Unbounded work, N+1, plan/cardinality change, missing/unused index, lock contention, pool saturation, retry storm, external latency, payload growth, regex/pathological input, memory/GC |
| Time/day/month boundary | UTC/local conversion, DST, inclusive/exclusive ranges, timestamp precision, clock skew, cron timezone, leap day, month/year rollover |
| Authorization mismatch | Tenant filter omission, role/policy version drift, case sensitivity, stale permission cache, object ownership, fail-open/fail-closed error mapping |
| Appeared after deploy | Code/config/schema/dependency/traffic change; compare effective versions. Also test whether the change merely activated a latent data or load defect |
| Memory/resource growth | Leaks, unbounded cache/collection, orphaned tasks, file descriptors, threads, connections, buffers, temp files, queue backlog, missing cancellation |
| Partial or corrupt writes | Transaction boundary, write ordering, outbox/dual-write gap, retry semantics, timeout after commit, compensation path, schema compatibility |
| Browser/UI only | Service worker/cache, cookies/storage/security settings, extension interference, profile state, hydration/race, network retries, stale frontend/backend contract |
| Serialization/contract failure | Enum expansion, unknown/missing fields, numeric precision, charset, date format, API version, backward/forward compatibility, null semantics |
| Wrapper exception | Broad catch/rethrow, lost cause/context, asynchronous error boundary, cleanup failure masking original exception, generic error mapper |

## End-to-end trace points

At every boundary record the expected invariant, actual evidence, and ownership:

1. inbound request/event/file/job trigger;
2. parsing, normalization, and syntactic validation;
3. authentication, authorization, and tenant scoping;
4. domain preconditions and state transition;
5. transaction start/commit/rollback and lock behavior;
6. DB query/write, cache read/write/invalidation, queue publish/consume;
7. external dependency request, timeout, retry, and circuit behavior;
8. response mapping, side-effect confirmation, and error propagation.

The first boundary where actual behavior diverges from the invariant is usually more informative than the final exception.

## Legacy-code traps

- “Dead” branches may preserve old clients, records, migrations, or operational repair paths.
- Existing tests may characterize required behavior or accidentally encode the defect; determine which before editing.
- Hidden coupling often lives in globals, static caches, implicit transactions, framework hooks, callbacks, triggers, schedulers, and environment defaults.
- Similar functions may differ for a historical reason. Compare callers and data contracts before consolidating.
- A local null check may hide an upstream state violation and create silent corruption.
- Broad exception handling may turn a deterministic defect into intermittent symptoms.
- Production-only failures often require the combination of code + data shape + timing/configuration; reproducing only one dimension can mislead.

## Focused probes

### Data-shape probe

Vary one characteristic at a time: missing/empty/null, maximum length, Unicode, numeric boundary, old schema version, duplicate key, orphaned relation, unusual state transition, large collection, timestamp edge, ordering.

### Concurrency probe

Repeat under controlled parallelism; synchronize competing operations where possible; verify unique/idempotency guarantees; capture attempt, transaction, and commit order. Do not use arbitrary sleeps as proof.

### Retry and queue probe

Model failure before commit, after commit but before acknowledgement, duplicate delivery, out-of-order delivery, visibility timeout, redelivery, poison message, and partial downstream success.

### Performance probe

Measure baseline distribution; isolate CPU, I/O, lock/wait, pool, queue, network, allocation/GC, and query-plan contributions; compare representative input sizes and healthy versus affected versions.

### Configuration probe

Compare effective runtime values, flag evaluation, secrets/config versions, routing, schema version, and dependency endpoints. Repository defaults alone are insufficient.

## Fixes to reject without causal evidence

- catching and ignoring exceptions;
- adding retries without idempotency, limits, jitter, and a retryable classification;
- increasing timeouts or capacity as the only change;
- defaulting invalid state into a believable value;
- clearing or disabling caches permanently;
- adding an index without verifying the query and plan;
- disabling validation, authorization, constraints, or error reporting;
- rewriting the surrounding legacy module before the exact failure is proven.
