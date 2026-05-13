# Review-loop regression matrix

These are documented transcript-style checks for `agents/anvil.agent.md`. The repo does not ship a runtime harness, so the regression surface is the prompt text plus this checklist.

| Case | Setup / prompt condition | Expected Anvil behavior |
| --- | --- | --- |
| Read-only ledger failure | Verification tries to create or insert `anvil_checks` rows. | Uses the writable `session` database for ledger writes; `session_store` only appears in recall queries. |
| Unavailable reviewer model | Large task runs where one or more preferred reviewer models are unavailable. | Launches the available `code-review` agents, inserts `review-capability-shortfall`, and still fail-closes if zero actual `review-*` verdict rows are produced. |
| Duplicate blocker phrasing | Round 2 repeats a Round 1 blocker with the same file + failure mode but different wording. | Normalizes a fingerprint, treats the repeat as duplicate, and does not reopen the loop. |
| Malformed reviewer output | Reviewer returns concerns without `file`, `severity`, `confidence`, or `failure_mode`. | Treats the concern as non-blocking and records it as a note instead of reopening. |
| Max-round enforcement | A valid new blocker is found on every allowed round. | Stops once the round cap is hit, inserts `review-round-limit-reached`, and presents the task as bounded risk rather than looping. |
| Targeted follow-up review scope | A blocker fix touches only one file or hunk after the initial review. | Follow-up review inspects a persisted blocker-fix patch/hunk artifact for that delta, not a whole-file staged diff that can replay older hunks. |

## Recommended dry runs

1. Run one Medium-task transcript and confirm the prompt caps at two total review rounds.
2. Run one Large-task transcript and confirm it either exits after the allowed rounds or stops once with a clear incomplete-verification reason.
3. For both transcripts, inspect the evidence bundle and verify that shortfalls, round limits, or missing prerequisites appear as explicit ledger rows instead of recursive instructions.
