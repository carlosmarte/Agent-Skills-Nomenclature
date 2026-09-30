---
name: github-retry
description: Calls the GitHub API; on a rate-limit response an evaluator node doubles the backoff in thread state and the next attempt reads it from the checkpoint. Failure mode: drift from noisy feedback; mitigated by only reacting to 429 responses.
metadata:
  nomenclature:
    structure: atomic
    execution: executable
    lifecycle: adaptive
    short-prefix: atm_exe_github_retry-adp
    urn: integrations.atm.exe.adp.retry
    extension: github_retry.atm.exe.ts
---
# github-retry
Retry until the call succeeds, doubling the checkpointed backoff each time.
