---
name: debug-code
description: Diagnose and fix hard bugs using runtime evidence and tracing. Use when debugging broken or intermittent behavior.
license: MIT
compatibility: Requires repository access. Production access is optional; the workflow can use logs, metrics, traces, dev or staging environments, dev databases, and operator-run read-only production queries when available.
metadata:
  version: "1.0.0"
---

# Debug Code

Turn a symptom into an evidenced causal chain and the smallest safe fix. The codebase is the only guaranteed source; production access is optional.

## Proof levels

Use these labels exactly:

- **Confirmed** — direct evidence ties the symptom to the code path, a falsifiable prediction succeeds, and a counterfactual or regression test removes the symptom.
- **Strongly supported** — multiple independent facts fit one causal chain, but production reproduction or a decisive probe is unavailable.
- **Provisional** — the best current hypothesis under limited evidence. Safe to investigate; not safe to present as root cause.

Never convert confidence into invented percentages. Separate **trigger**, **root defect**, **contributing conditions**, and **user-visible symptom**.

## Non-negotiables

1. **Mitigate active harm first.** For ongoing data loss, security exposure, financial harm, or severe outage, preserve evidence and propose the safest reversible containment before deep diagnosis. Production actions require an authorized human.
2. **Ask about access once, then continue.** Request only missing evidence sources in one grouped question. Do not block codebase analysis while waiting.
3. **Evidence over plausibility.** Mark every material statement as observed, inferred, unknown, or ruled out. No evidence means hypothesis, not fact.
4. **Production is read-only by default.** Never execute or request writes, locks, schema changes, broad exports, or unbounded production queries for diagnosis.
5. **Protect sensitive data.** Do not request or expose secrets, tokens, cookies, connection strings, passwords, full personal records, or unnecessary payloads. Ask for redacted or pseudonymized artifacts.
6. **Fix the cause, not the screenshot.** Trace all relevant callers and sibling paths. A local guard is wrong when the violated invariant belongs at a shared seam.
7. **Make the smallest maintainable change.** No unrelated refactoring, dependency changes, API redesign, schema migration, or speculative hardening.
8. **Preserve legitimate legacy behavior.** Existing behavior is evidence, not automatically correct; characterize unaffected contracts before changing shared code.
9. **Change one variable per probe.** Record negative results; they narrow the search space.
10. **Do not say “fixed” early.** Distinguish diagnosis, local verification, rollout readiness, and production verification.

## Workflow

### 0. Triage impact and mode

Classify the task:

- **Incident mode** — harm is active. Contain first, diagnose second.
- **Reproduction mode** — the symptom can be driven in dev, test, staging, or a safe harness.
- **Restricted-evidence mode** — the bug is production-only or access is limited. Continue with code-derived predictions and targeted human-run probes, but downgrade claims honestly.

For active incidents, identify a reversible mitigation such as rollback, feature disablement, traffic isolation, queue pause, or write freeze. Do not recommend a blunt mitigation without stating its likely collateral impact and rollback condition.

### 1. Establish the symptom and access map

Summarize what is already known before asking anything:

- expected versus actual behavior;
- affected scope and frequency;
- exact error, wrong output, or latency symptom;
- earliest known occurrence and relevant timezone;
- known request, trace, job, tenant, user, or record identifier using a safe pseudonym;
- recent release, configuration, schema, dependency, or traffic changes.

Ask one compact access question, omitting anything already answered:

> Which evidence can be accessed now or supplied on request: dev/staging reproduction, dev DB, Grafana/metrics, application logs, traces/error tracker, deployment/config history, or operator-run read-only production SQL? Code-only is acceptable. If available, include the smallest safe locator: timestamp plus timezone, request/trace/job ID, pseudonymous tenant/user ID, and deployed version.

Start repository work immediately while the user answers. Never repeat this questionnaire.

### 2. Orient in the codebase

Before theorizing:

1. Read repository instructions, tests, architecture notes, `CONTEXT.md`, and relevant ADRs if present.
2. Search the exact error text, status code, log event, UI label, endpoint, job name, table, and feature flag.
3. Trace the real path end to end: boundary input → normalization/validation → authorization/tenant scope → domain state → DB/cache/queue/external dependency → response or side effect.
4. Enumerate callers of any function proposed for modification and find the closest working sibling path.
5. Inspect relevant configuration, migrations, serializers, retry policies, transaction boundaries, and recent changes around the reported onset. Treat recent change as a lead, not proof.
6. Run the smallest existing tests and commands that establish the current baseline. Capture exact commands and outcomes.

Produce a concise investigation map: suspected path, evidence sources, commands, assumptions, and safety constraints.

Before any intrusive probe or edit, run a plan gate: are we solving the reported symptom; does a reproduction, helper, or fix already exist; is any assumption being treated as fact; is there a cheaper or safer discriminator; and are verification and rollback explicit? Proceed without waiting unless a consequential permission or production-risk decision is genuinely missing.

### 3. Build a feedback or evidence loop

Prefer the first feasible loop that exercises the actual symptom:

1. existing failing test;
2. new regression test at the real seam;
3. API, CLI, UI, worker, or scheduled-job script;
4. dev/staging reproduction with production-shaped configuration;
5. sanitized request, event, trace, or payload replay;
6. minimal harness with mocked external dependencies;
7. synthetic fixture matching the relevant production data shape;
8. stress, fuzz, concurrency, clock, or repetition loop for intermittent faults;
9. old/new version, config, or dataset differential;
10. automated `git bisect` check;
11. structured human-in-the-loop probe using logs, metrics, or read-only SQL.

A useful loop must assert the **exact symptom**, not merely “did not crash.” Tighten it until it is as deterministic, fast, and unattended as practical. For flaky bugs, raise and measure the reproduction rate rather than pretending one passing run is meaningful.

In restricted-evidence mode, build an **evidence loop** instead of stopping:

- derive observable predictions from the code path;
- request the smallest log, metric, trace, or query result that splits the leading hypotheses;
- construct synthetic fixtures from shape, cardinality, nullability, encoding, timing, ordering, and state—not from sensitive production values;
- compare an affected case with a known-good case whenever possible.

No loop means no confirmed root cause. It does not prevent continued investigation.

### 4. Maintain an evidence ledger

Keep a compact ledger throughout:

| ID | Statement | Status | Source | Diagnostic implication |
|---|---|---|---|---|
| E1 | Exact observed fact | observed / inferred / unknown / ruled out | command, file, log, metric, query, user report | what it supports or rejects |

Prefer primary evidence. Treat user reports as valid symptom evidence but not automatically as causal evidence. Treat missing logs as “not observed,” not “did not happen.”

### 5. Rank falsifiable hypotheses

Generate 3–5 hypotheses before editing code. For each use:

> **H# — Cause:** …  
> **Evidence for / against:** …  
> **Prediction:** If this is the cause, probe X will produce Y; otherwise it will produce Z.  
> **Cheapest discriminating probe:** …  
> **Risk if wrong:** …

Rank by explanatory power, evidence fit, probe cost, and production risk—not familiarity. Share the ranking when useful, but continue unless user input is required for a consequential action.

Test the highest-value discriminator first. Prefer probes at component boundaries. Use a debugger or profiler when available; otherwise add narrowly targeted instrumentation with a unique marker such as `[DEBUG-d3f7]`. Never “log everything and grep.”

For performance regressions, establish a baseline and use timing, profiling, traces, query plans, lock/wait evidence, or bisection before changing code.

Load `references/production-probes.md` before requesting production logs, dashboards, traces, temporary instrumentation, or SQL.

### 6. Establish the causal chain

A root-cause statement must explain:

`trigger → violated invariant → faulty state/control flow → propagation → user-visible symptom`

Challenge easy misattributions:

- Production data may trigger a code defect; “bad data” is not the root cause when the contract requires safe handling.
- A deployment may expose a latent defect rather than introduce it.
- Resource saturation may be the effect of retry storms, lock contention, leaks, or unbounded work.
- An exception shown to the user may be a wrapper that hides the first failure.

Confirm the cause with the strongest available combination of:

- reproduction of the exact symptom or runtime evidence tied to the exact path;
- a successful falsifiable prediction;
- a counterfactual that removes or intensifies the symptom as predicted;
- a minimal regression test that fails before the fix and passes after it;
- proportionate rejection of plausible alternatives.

If these are unavailable, report **strongly supported** or **provisional**, plus the missing decisive evidence.

### 7. Implement the fix

1. Convert the minimal reproduction into a failing regression test at the real behavioral seam. If no correct seam exists, create the smallest honest harness and document the testability gap.
2. Watch the test fail for the user’s symptom—not a nearby setup error.
3. Apply the smallest root-cause fix. Reuse existing patterns, standard-library features, platform guarantees, or database constraints where they are already the correct ownership boundary.
4. Check every relevant caller and sibling path. Add characterization coverage for unaffected legacy behavior when shared code changes.
5. Watch the regression test pass, then rerun the original, non-minimized loop.
6. Keep emergency mitigation, durable fix, cleanup, and optional hardening as separate changes when mixing them increases risk.

Reject common fake fixes unless evidence specifically justifies them:

- catch-all exception swallowing;
- arbitrary sleeps or timeouts;
- retries without idempotency and bounded backoff;
- defaulting corrupt or missing data into a plausible value;
- clearing caches as the permanent fix;
- adding capacity without finding why demand or work exploded;
- broad refactoring around an unconfirmed theory.

A provisional fix may proceed only when it is reversible, low risk, covered by a meaningful test, paired with explicit rollout/rollback signals, and the user has authorized acting under uncertainty. Never call it confirmed.

### 8. Adversarial review gate

Before declaring the diagnosis or fix complete, independently challenge it:

- Are we reproducing the reported bug or a convenient neighbor?
- Is the stated root cause actually only a trigger or correlation?
- Could the test pass while the production bug remains?
- Is the fix placed at the correct ownership boundary for all callers?
- What concurrency, retry, transaction, cache, authorization, compatibility, or data-integrity edge remains?
- What evidence contradicts the preferred theory?
- Could the diagnostic query or instrumentation itself harm production or expose data?
- What exact signal triggers rollback?

For high-impact, ambiguous, security-sensitive, financial, or data-integrity bugs, use an independent read-only subagent as the skeptic when available. Give it the symptom, evidence ledger, relevant code, tests, and diff; ask it to find disconfirming evidence and alternative causal chains. Do not treat agent agreement as proof.

### 9. Verify, clean up, and prepare rollout

Run and report exact commands for:

- the regression test;
- the original reproduction/evidence loop;
- relevant integration or end-to-end tests;
- static checks, lint, and type checks used by the repository;
- full or broader tests when the shared surface warrants them;
- concurrency, load, performance, migration, or backward-compatibility checks when relevant.

Before closure:

- remove all temporary `[DEBUG-…]` instrumentation and throwaway artifacts, or isolate intentionally retained diagnostic tools;
- verify no secrets or sensitive production data were added to code, tests, fixtures, logs, or reports;
- state rollout scope: canary, feature flag, tenant slice, job subset, or normal deployment;
- define the production success signal and a concrete rollback trigger;
- verify over representative occurrences, not one lucky request;
- document any remaining uncertainty or production verification still owned by a human operator.

## Output contract

Use this structure for substantive updates and the final result:

```text
Status: investigating | provisional cause | strongly supported cause | root cause confirmed | fix verified locally | production verification pending | verified in production
Symptom and impact:
Access and constraints:
Observed evidence:
Ruled out:
Root cause or leading hypothesis:
Causal chain:
Change made:
Verification commands and results:
Rollout and rollback:
Remaining uncertainty:
Next evidence request, if any:
```

When requesting human-run evidence, provide one exact, minimal, copy-pasteable probe and state:

- what question it answers;
- what result supports or rejects each hypothesis;
- safety bounds and redaction requirements;
- exactly which sanitized fields or counts to return.

Do not bury uncertainty. A precise unresolved report is better than a persuasive false RCA.

## Reference loading

- Read `references/production-probes.md` when any production evidence must be requested or instrumentation/query safety matters.
- Read `references/production-bug-patterns.md` after the initial code-path trace when a focused checklist would help identify hidden data, timing, concurrency, distributed-system, or resource bugs.
