# merge-queue-bypass-demo

Verification of whether CODEOWNERS review can be bypassed while keeping merge queue functional on GitHub.

## Conclusion

**There is currently no way to bypass CODEOWNERS review while preserving merge queue behavior.**

- Branch Protection bypass skips CODEOWNERS but also skips merge queue entirely (direct merge)
- Rulesets bypass does not skip CODEOWNERS when merge queue is configured

## Results

| Scenario | Protection Method | Bypass | CODEOWNERS bypassed | Merge queue works | Merge method |
|---|---|---|---|---|---|
| 1 ([PR #7][pr7]) | Branch Protection | None | N/A (approved) | Yes | Via queue |
| 2 ([PR #8][pr8]) | Branch Protection | bypass list | Yes | **Skipped** | Direct merge |
| 3 ([PR #9][pr9]) | Rulesets (separate) | Ruleset B bypass | **No** | Blocked | N/A |
| 4 ([PR #10][pr10]) | Rulesets (single) | Ruleset bypass | **No** | Blocked | N/A |

## Scenario Details

### Scenario 1: Branch Protection + CODEOWNERS + Merge Queue (no bypass)

- **Setup:** Branch Protection on `main` with required reviews, CODEOWNERS, merge queue, and status checks
- **Result:** PASS — Merge queue worked correctly. PR was queued, CI ran, and auto-merged via queue.

### Scenario 2: Branch Protection + CODEOWNERS + Merge Queue (with bypass)

- **Setup:** Same as Scenario 1 + user added to bypass pull request allowances
- **Result:** As expected — Bypass skipped both CODEOWNERS review AND merge queue. PR merged directly via "Confirm bypass rules and merge" button.
- **Key finding:** Branch Protection bypass is all-or-nothing. It cannot selectively bypass CODEOWNERS while preserving merge queue.

### Scenario 3: Rulesets — Separate merge queue and code review (with bypass)

- **Setup:**
  - Ruleset A "Merge Queue and CI" (no bypass): merge queue + status checks
  - Ruleset B "Code Review" (bypass for user): PR reviews + CODEOWNERS
- **Result:** UNEXPECTED — Bypass did not take effect. "Merging is blocked: Waiting on code owner review" remained even though the user was in the bypass list.
- **Key finding:** Rulesets bypass does not work for CODEOWNERS review when merge queue is active, even when the rules are in separate Rulesets.

### Scenario 4: Rulesets — Single ruleset with all rules (with bypass)

- **Setup:** Single Ruleset with merge queue, status checks, PR reviews, CODEOWNERS, and bypass for user
- **Result:** Same as Scenario 3 — Bypass did not take effect. CODEOWNERS review remained a hard blocker.
- **Key finding:** Whether rules are in one or two Rulesets makes no difference. Rulesets bypass simply does not skip CODEOWNERS review when merge queue is present.

## Key Findings

### Branch Protection bypass vs Rulesets bypass

| Behavior | Branch Protection | Rulesets |
|---|---|---|
| CODEOWNERS bypass | Works | Does NOT work |
| Merge queue behavior | Skipped entirely | Blocked (never enters queue) |
| Granularity | All-or-nothing | Expected per-rule, but ineffective |

### Root cause hypothesis

- **Branch Protection:** Bypass allows the user to confirm and directly merge, completely skipping all protection rules including merge queue.
- **Rulesets:** Bypass is supposed to exempt the actor from specific rules, but when merge queue is configured, the auto-merge flow appears to not evaluate bypass permissions correctly. The merge queue requires all prerequisites to be met before a PR can enter the queue, and CODEOWNERS review is treated as a hard prerequisite regardless of bypass configuration.

## Repository Structure

```
.github/
├── CODEOWNERS              # * @codenote-net/reviewers
└── workflows/
    ├── ci.yml              # CI workflow (pull_request + merge_group)
    └── create-pr.yml       # Helper to create PRs via github-actions[bot]
```

## References

- [Issue #11: Full verification results][issue11]
- PRs: [#7][pr7] (Scenario 1), [#8][pr8] (Scenario 2), [#9][pr9] (Scenario 3), [#10][pr10] (Scenario 4)
- Setup PRs: [#1][pr1] (initial files), [#6][pr6] (create-pr workflow)

[issue11]: https://github.com/codenote-net/merge-queue-bypass-demo/issues/11
[pr1]: https://github.com/codenote-net/merge-queue-bypass-demo/pull/1
[pr6]: https://github.com/codenote-net/merge-queue-bypass-demo/pull/6
[pr7]: https://github.com/codenote-net/merge-queue-bypass-demo/pull/7
[pr8]: https://github.com/codenote-net/merge-queue-bypass-demo/pull/8
[pr9]: https://github.com/codenote-net/merge-queue-bypass-demo/pull/9
[pr10]: https://github.com/codenote-net/merge-queue-bypass-demo/pull/10
