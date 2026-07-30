# Production Evidence Probes

Use this file only when runtime evidence is needed. Ask for the smallest artifact that discriminates between hypotheses. Direct production actions remain human-operated unless the environment explicitly grants safe read-only access.

## 1. One-time access prompt

Omit sources already known:

> Available now or on request: dev/staging reproduction, dev DB, Grafana/metrics, application logs, distributed traces/error tracker, deployment/config history, or operator-run read-only production SQL? Code-only is valid. For localization, provide only safe identifiers: timestamp plus timezone, request/trace/job ID, pseudonymous tenant/user ID, and deployed version.

Do not ask again unless a newly identified probe requires a specific permission.

## 2. Probe design rules

Every request must state:

1. **Question** — the single uncertainty being tested.
2. **Prediction** — what each materially different result means.
3. **Scope** — service, operation, cohort, identifiers, and exact bounded time range.
4. **Safety** — read-only, timeout, row/event limit, sampling, redaction, and operator review.
5. **Return shape** — only the fields, counts, or plan nodes needed.

Prefer affected-versus-unaffected and before-versus-after comparisons. Align timestamps and timezones before correlating systems.

## 3. Log request template

```text
Probe ID: LOG-H2-01
Question: Did dependency timeout occur before the application wrapper exception?
Service/component: <service>
Time range: <start> to <end> <timezone>
Selectors: request_id=<id>, trace_id=<id>, job_id=<id>, tenant_hash=<safe-id>
Need: timestamp, event/error name, exception type and sanitized stack, request/trace ID,
      release/version, dependency status, duration, retry attempt, outcome
Comparison: one failed and one successful execution with the same operation
Limit: <=50 relevant events per case, preserving order
Redact: credentials, auth headers, cookies, tokens, connection strings, full payloads,
        direct personal identifiers, payment/health data, and unrelated records
Return: raw structured events or text lines, not a paraphrase, after redaction
```

Ask for surrounding events only when ordering matters. Do not request an entire log file when a correlation ID or narrow time window exists. Missing log entries are ambiguous unless logging coverage is independently verified.

## 4. Grafana or metrics request

Request exact panels and a bounded window around onset:

- request volume, error rate, and latency distribution—not only averages;
- CPU, memory, disk, file descriptors, thread/event-loop saturation;
- DB connection-pool use, query latency, locks/waits, replica lag;
- queue depth, age, retries, dead letters, consumer throughput;
- dependency latency/error rate;
- deployment, configuration, feature-flag, and migration annotations.

Ask for:

```text
Probe ID: METRIC-H1-01
Question:
Dashboard/panel or metric name:
Window and timezone:
Breakdown: endpoint/job/tenant-safe cohort/status/version/region as applicable
Comparison: prior healthy window and affected window
Return: values or exported series plus deploy/config annotations
```

A correlation narrows hypotheses; it is not a causal conclusion by itself.

## 5. Trace or error-tracker request

Minimum useful fields:

- trace or event ID;
- release/version and environment;
- ordered spans with status and duration;
- exception type and sanitized stack;
- retry/attempt count;
- relevant non-sensitive tags and breadcrumbs;
- one comparable successful trace when available.

Do not request request/response bodies unless their exact shape is the hypothesis and they can be safely minimized and redacted.

## 6. Deployment and configuration evidence

Request identifiers, not secrets:

- commit SHA, artifact/image digest, release ID;
- deploy start/end and rollback times;
- migration version and execution status;
- feature-flag names and evaluated states for an affected safe cohort;
- dependency/runtime versions;
- environment-variable **names** and normalized non-secret values only when required;
- infrastructure or traffic-routing changes.

Compare effective runtime configuration, not only repository defaults.

## 7. Dev/staging and dev-DB use

A dev DB proves code behavior against its own data, not production equivalence. Check parity in:

- schema and migrations;
- indexes and constraints;
- engine/version and isolation settings;
- feature flags and configuration;
- data shape: nullability, cardinality, size, encoding, legacy formats, timestamps, ordering, and state transitions;
- concurrency, load, retry, cache, queue, and dependency behavior.

Build synthetic fixtures from these characteristics. Never copy sensitive production rows into tests or chat.

## 8. Production SQL protocol

Use SQL only when the result will materially distinguish hypotheses. Prefer an existing dashboard, read replica, or approved diagnostic view when it answers the same question.

### Query card

```text
Probe ID: SQL-H3-01
Question answered:
DB engine/version:
Expected discriminator:
Tables and known indexes:
Bounded keys/cohort/time range:
Columns or aggregate returned:
Expected maximum rows:
Operator-enforced timeout:
Sensitive data omitted or transformed:
Why this is safe and cheaper alternatives considered:
```

### Guardrails

- Use a read-only account or read replica when available.
- Use an explicit read-only transaction where supported.
- Use `SELECT` only. No DML, DDL, stored procedures, temporary tables, advisory locks, `FOR UPDATE`, or `FOR SHARE`.
- Select named, non-sensitive columns; never `SELECT *`.
- Start with aggregates or existence checks when rows are unnecessary.
- Bound by indexed identifiers and/or a narrow time range, then apply a small `LIMIT`.
- Parameterize values. Never concatenate untrusted input.
- Avoid broad joins, unbounded scans, wildcard-leading searches, and functions on indexed filter columns unless a reviewed plan proves safety.
- Use plain `EXPLAIN` only when a plan is needed. `EXPLAIN ANALYZE` executes the statement and requires explicit DBA approval and production-risk review.
- Apply a short statement timeout using the organization’s approved mechanism.
- The DBA/operator must review the exact query and may reject or rewrite it.
- Return only the count or sanitized fields needed for the hypothesis.

### PostgreSQL wrapper

```sql
BEGIN TRANSACTION READ ONLY;
SET LOCAL statement_timeout = '5s';

SELECT <named_non_sensitive_columns>
FROM <schema.table>
WHERE <indexed_key> = :key
  AND created_at >= :from_ts
  AND created_at < :to_ts
ORDER BY created_at
LIMIT 100;

ROLLBACK;
```

### MySQL wrapper

```sql
START TRANSACTION READ ONLY;

SELECT <named_non_sensitive_columns>
FROM <schema.table>
WHERE <indexed_key> = ?
  AND created_at >= ?
  AND created_at < ?
ORDER BY created_at
LIMIT 100;

ROLLBACK;
```

Use the operator’s approved query-timeout mechanism for the deployed MySQL version and client.

## 9. Temporary production instrumentation

Only with explicit authorization and change control:

- instrument the narrowest boundary that distinguishes hypotheses;
- use a unique marker and structured fields;
- sample or target a safe cohort;
- log identifiers as hashes/pseudonyms when possible;
- never log credentials, secrets, full payloads, or unnecessary PII;
- avoid hot-loop logging and high-cardinality labels;
- include a disable/expiry path and rollback trigger;
- remove temporary instrumentation after the investigation, or promote it deliberately into durable observability with review.

## 10. Human-run response format

Ask the operator to return:

```text
Probe ID:
Exact command/query/dashboard actually used:
Environment and deployed version:
Execution time and timezone:
Sanitized result:
Any warning, timeout, plan change, or deviation from the requested probe:
```

Never ask the user to paste credentials or grant broader access merely for convenience.
