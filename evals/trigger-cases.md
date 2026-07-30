# Trigger Evaluation Cases

Use these cases to sanity-check skill discovery after installation.

## Should trigger

1. “Production returns 500 only for one tenant. Here is the stack trace; find the root cause.”
2. “Users sometimes get duplicate orders after the queue retries. Debug the legacy consumer.”
3. “This export is fast locally but times out in production. We have Grafana and can run read-only PostgreSQL queries.”
4. “Saving stopped working after yesterday’s deploy. I only have the repository and can request logs.”
5. “Find the likely production defect in this legacy payment path; we cannot access the production database.”
6. “Here is an error message from a user. Trace the codebase and create a regression test before fixing it.”
7. “A nightly job silently skips some records near month-end. Investigate.”
8. “The API occasionally returns stale state immediately after an update. Diagnose the production-only behavior.”

## Should not trigger

1. “Refactor this class to make it cleaner.”
2. “Review this pull request for style and maintainability.”
3. “Explain PostgreSQL transaction isolation.”
4. “Build a new reporting endpoint.”
5. “Perform a general security audit of the whole repository.”
6. “Optimize this function even though there is no measured regression.”
7. “Write unit tests for this utility.”
8. “Summarize this incident report.”
