# Anvil

Evidence-first coding agent for GitHub Copilot CLI. The plugin lives in `agents/anvil.agent.md`, and the repo stays intentionally small: prompt, metadata, and the docs needed to keep prompt regressions visible.

## Review-loop invariants

The current prompt hardens the adversarial review loop around a few operational rules:

1. **Writable verification state**: the verification ledger uses the writable `session` database. `session_store` is recall-only.
2. **Fail-closed gates**: missing baseline, reviewer, or evidence prerequisites stop the run with an explicit incomplete-verification outcome. They do not say "go back" and silently recurse.
3. **Capability-safe review**: Large tasks target up to three reviewers, but they use whatever approved `code-review` agents are actually available. Any shortfall is recorded in evidence instead of blocking forever.
4. **Finite rounds**: Medium tasks cap at two total review rounds; Large tasks cap at three. Review state is tracked durably with `review_round`.
5. **Targeted follow-up review**: follow-up rounds inspect a persisted blocker-fix patch or hunk artifact from the current round, not a whole-file staged diff that can replay older hunks.
6. **Blocker fingerprinting**: blockers require structured fields and are fingerprinted so paraphrased duplicates cannot reopen the loop.

## Regression checks

See `docs/review-loop-regressions.md` for the documented dry-run matrix that exercises the failure modes which caused the loop bug.

## Releasing

1. Update `plugin.json` with the next patch version.
2. Commit the prompt/docs changes in the `anvil/` repo.
3. Push `main`. GitHub Pages deploys from `.github/workflows/deploy.yml` on pushes to `main`.
