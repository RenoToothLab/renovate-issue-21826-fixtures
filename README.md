# Renovate #21826 reproduction and verification log

Three pinned dependencies share `pnpm-lock.yaml`: is-number (A), is-odd (B), and is-even (C). The fixture uses fixed target versions and disables automerge.

## Latest recorded verification: one session on 2026-11-04

> **Point-in-time evidence from ONE verification session.** This session exercised several scenarios against two GitHub fixture repositories using the same runner and revisions. These are two fixture copies, not independent verification campaigns or different platform implementations. The results below apply to the exact revisions and conditions recorded here. Later agents must rerun the relevant scenarios before claiming that a newer revision passes or that this finding has been fixed.

The session ran approximately **19:25–19:32 Europe/Oslo (18:25–18:32 UTC)** on 4 November 2026. This README update records that existing session; it is not another test run.

- Original issue: [renovatebot/renovate#21826](https://github.com/renovatebot/renovate/issues/21826).
- PR under test: [renovatebot/renovate#47618](https://github.com/renovatebot/renovate/pull/47618).
- Tested PR head: `49dba3fe70f1d22fe3d4fe4bad2e0de7008a53ae`.
- Compared base revision: `708d7d0251a48117eefb73d4a502c272c77cafc6`.
- The remote PR head was checked again during verification and had not changed.
- Findings were derived from the diff, original issue, and fresh execution/readback, rather than the source PR description.

### Conclusion at that revision

**Partial fix.** A group created by the patched writer reconciles correctly after repository cache loss because its PR already contains `upgradesFingerprint`. A pre-patch PR with neither a repository fingerprint nor a PR fingerprint still exhibits the original bug: the summary drops C while the branch retains C's upgrade.

The first patched run over such a legacy PR records a fingerprint for the desired A+B group without repairing the actual A+B+C files. A second patched run still leaves the inconsistency in place. Passing the fresh-PR scenario therefore does not demonstrate recovery of existing older PRs.

### Expected dependency values

| State | A: is-number | B: is-odd | C: is-even | Grouped PR table |
| --- | --- | --- | --- | --- |
| Main/base manifest | 6.0.0 | 2.0.0 | 0.1.0 | Not applicable |
| Initial grouped upgrade | 7.0.0 | 3.0.1 | 1.0.0 | A+B+C |
| Correct shrunk group | 7.0.0 | 3.0.1 | 0.1.0 | A+B |
| Separate C upgrade | 6.0.0 | 2.0.0 | 1.0.0 | C |
| Observed stale legacy group | 7.0.0 | 3.0.1 | **1.0.0** | **A+B only** |

These values were checked in both `package.json` and `pnpm-lock.yaml`'s root importer. For each dependency, both the lockfile's `specifier` and resolved `version` were checked; observing only the PR table was not considered sufficient.

### Scenarios actually exercised in this session

Every live run below used `repositoryCache=disabled`, including creation and repeat runs. All rows had the same outcome in both repositories.

| Scenario | Observed result |
| --- | --- |
| Base revision creates A+B+C, then C is removed from the group | Original bug reproduced: grouped manifest and lockfile retain C=1.0.0; table lists only A+B. |
| Patch creates A+B+C, then grouping shrinks with `rebaseWhen=never` | Existing grouped SHA and A+B+C files/table stay unchanged, as intended for suppressed automatic updates. |
| Same patched group shrinks with `rebaseWhen=conflicted` | Existing grouped PR is reconciled: A=7.0.0, B=3.0.1, C=0.1.0 in manifest and lockfile; table lists A+B. |
| C receives a separate PR | Only C is upgraded there; A and B retain their main/base versions. |
| Patch repeats the unchanged, correctly shrunk group without repository cache | Grouped SHA remains unchanged; no extra commit. |
| Patch adopts the already-shrunk pre-patch PR without either prior fingerprint | **Failure:** C remains upgraded in files although absent from the table. A new PR fingerprint is written. |
| Patch repeats that legacy PR, again without repository cache | **Failure persists:** grouped SHA is unchanged and C remains upgraded. The newly recorded PR fingerprint remains unchanged. |

### Remote evidence and exact branch SHAs

The `head` lane was created and processed with the patch. The `base` lane was initially created and shrunk with the base revision, then processed twice with the patch to test adoption. **The word `base` in a branch name describes its origin; it does not mean its final state was tested only with the base code.**

| Repository / grouped PR | Branch | Initial SHA; also unchanged by the head-lane `never` run | Final SHA after patched processing and repeat |
| --- | --- | --- | --- |
| [RenoTooth #13 — correct](https://github.com/RenoTooth/renovate-issue-21826-fixtures/pull/13) | `verify-47618-head/fixture-dependencies` | `b4b0e063a5dfb141b614af2968017af7d84f76cc` | `360735eb86bb88fdd91d69f320982cd1dd409850` |
| [RenoToothLab #7 — correct](https://github.com/RenoToothLab/renovate-issue-21826-fixtures/pull/7) | `verify-47618-head/fixture-dependencies` | `47c8ef7ec065620197decb29f6c5cda2e687ee68` | `577477192b7945905887a50cd29fe2ebe098b025` |
| [RenoTooth #12 — legacy failure](https://github.com/RenoTooth/renovate-issue-21826-fixtures/pull/12) | `verify-47618-base/fixture-dependencies` | `38579f9ca66e60e51fbd1dda993ad043638612ac` | `38579f9ca66e60e51fbd1dda993ad043638612ac` |
| [RenoToothLab #8 — legacy failure](https://github.com/RenoToothLab/renovate-issue-21826-fixtures/pull/8) | `verify-47618-base/fixture-dependencies` | `af71e9e82c86e0e05cc34ea4039b4d776e46bf21` | `af71e9e82c86e0e05cc34ea4039b4d776e46bf21` |

Separate C PRs:

- Patched-origin lane: [RenoTooth #15](https://github.com/RenoTooth/renovate-issue-21826-fixtures/pull/15) and [RenoToothLab #10](https://github.com/RenoToothLab/renovate-issue-21826-fixtures/pull/10).
- Base-origin lane, subsequently processed by the patch: [RenoTooth #14](https://github.com/RenoTooth/renovate-issue-21826-fixtures/pull/14) and [RenoToothLab #9](https://github.com/RenoToothLab/renovate-issue-21826-fixtures/pull/9).

These PR links show mutable remote state. The commit SHAs above identify the recorded file contents. A later agent must check whether the PR body, branch, or code revision has changed before using its current appearance as evidence.

### Runtime conditions and execution order

The session used the real Renovate source CLI (`pnpm start`), GitHub API/Git operations, and real pnpm lockfile generation. Node was 24.19.0; the fixture declares `pnpm@11.28.7`. The runner used `binarySource=global`; tool constraints were therefore not enforced as an independently tested dimension.

The runner selected only:

- `RenoTooth/renovate-issue-21826-fixtures`
- `RenoToothLab/renovate-issue-21826-fixtures`

Main's checked-in dependency files were used unchanged. To preserve prior fixture work, grouping and test controls were applied through the runner's forced configuration rather than committing scenario changes on main:

- `platform=github`, `onboarding=false`, `requireConfig=ignored`.
- `repositoryCache=disabled`, `persistRepoData=true`.
- Separate prefixes `verify-47618-head/` and `verify-47618-base/`, with `branchPrefixOld` set to the same lane prefix.
- `enabledManagers=["npm"]`, `ignorePaths=["scenarios/**"]`, `automerge=false`.
- A group named `fixture dependencies`, initially matching A+B+C; the shrink matched only A+B. `separateMajorMinor=false`.
- Fixed `allowedVersions`: A=7.0.0, B=3.0.1, C=1.0.0. C remained eligible for a separate update.
- `minimumReleaseAge="0 days"` and `internalChecksFilter=strict`: the existing pending-release-age fixture state was overridden for this test.
- `rebaseWhen=conflicted`, except the explicit `never` scenario; PR hourly/concurrent limits were unlimited.
- pnpm's own dependency update was disabled. Dependency Dashboard was disabled with a lane-specific dashboard title. Vulnerability-alert lookup and changelog fetching were disabled after the initial run.
- Authentication stayed in the runner environment. No tokens belong in this repository.

Execution order:

1. Create A+B+C groups using the patch in the head lane and the base revision in the base lane; record files, PR bodies/debug data, and SHAs.
2. Shrink the base lane with the base revision and disabled caching; confirm the stale-file bug and absent PR fingerprint.
3. Shrink the head lane with `rebaseWhen=never`; confirm its existing grouped SHA is preserved.
4. Process the base lane with the patch and the shrunk configuration; confirm adoption fails and newly stamps the desired A+B fingerprint.
5. Process the head lane with the patch and `rebaseWhen=conflicted`; confirm files and table agree on A+B.
6. Repeat both lanes with the patch and disabled caching; compare SHAs, files, tables, and decoded fingerprints.

Steps 2/3 and 4/5 ran in separate lanes and could overlap. Existing earlier fixture branches and main were not edited during this verification; generated update PRs were left unmerged.

### Code-level explanation and regression-test gap

At the tested head, `lib/workers/repository/update/branch/reuse.ts:115–117` requires all of:

1. `cacheFingerprintMatch === "no-fingerprint"`;
2. a truthy previous `branchPr.bodyStruct.debugData.upgradesFingerprint`;
3. a difference from the current `config.upgradesFingerprint`.

A legacy PR lacking the previous PR fingerprint fails condition 2 and falls through to ordinary branch reuse. The PR writer later records the current desired fingerprint even though the branch still contains the excluded upgrade. This explains the two live adoption failures.

The new integration scenarios create their initial PR using the patched writer, so clearing their repository cache still leaves the PR fingerprint available. Add a regression scenario starting from a PR produced by the pre-change writer (or faithfully modeling its absent debug fingerprint). A future fix should verify/reconcile actual branch contents before recording the desired fingerprint, while preserving edited-branch and update-suppression protections.

### Local validation in the same session

- `pnpm check --all lib/workers/repository/process lib/workers/repository/update/branch lib/workers/repository/update/pr`: **passed**, including 66 affected test files and targeted checks. The coverage output identified uncovered lines in unrelated `process/lookup/digest.ts`; this is not a claim of complete coverage.
- `pnpm check --all --no-test`: type checking, schemas, formatting, documentation and other checks passed. The concurrent oxlint subprocess was killed with SIGKILL; the environment recorded an OOM kill.
- Retried only the failed check with `GOMAXPROCS=2 pnpm oxlint`: **passed**, exit 0, with cache-privacy warnings outside this patch.
- Production source was not changed, and the tested checkout was clean.

### Scope limits for later agents

This session did **not** run the full repository test suite, GitLab tests, the original Go-module reproduction end to end, manual-edit/unpend scenarios, explicit rebase consent, or a live repeat with repository caching enabled. The live fixture used npm dependencies and pnpm lockfiles to exercise the shared group-reconciliation worker. The `never` suppression test passed; it does not establish the untested explicit-rebase or manual-edit behavior.

The two patched processing attempts on each legacy PR belong to this one session. Do not describe them as separate independent verification sessions, or infer that subsequent commits retain these outcomes.

The execution workspace retained `/workspace/verification-47618/REPORT.md`, `config.cjs`, `run.sh`, phase logs, `snapshot-*.json`, `snapshot.py`, `assert-results.cjs`, and `assertions.log`; the affected-test log was `/workspace/check-47618.log`. These are local session artifacts, **not committed portable evidence and not guaranteed to exist in a later agent's workspace**. This README embeds the key observations and immutable file SHAs so the finding can be understood without them.

## Procedures for future verification (not additional recorded results)

Use new isolated branch prefixes or other fresh fixture state. Record the exact tested code SHA, baseline SHA, runtime options, starting PR debug data, branch SHAs, and final file contents. Preserve existing test branches and human edits. Never merge generated update PRs during a comparison.

### Group shrink

1. Create a grouped A+B+C PR. Test both a PR created by the patched writer and a genuinely pre-patch PR lacking `upgradesFingerprint`.
2. Apply the shrunk grouping rules from `scenarios/shrunk.json` through an isolated fixture/configuration: group only A+B while allowing C its separate PR.
3. Run with repository caching disabled to test the absent-cache path.
4. The intended reconciled group contains A=7.0.0, B=3.0.1, C=0.1.0 in manifest and lockfile, and only A+B in its description. Compare actual files rather than inferring correctness from the description or fingerprint.
5. Repeat unchanged runs with caching disabled and enabled. Compare branch SHAs.
6. Test `rebaseWhen=never` suppression separately from explicit rebase consent. The legacy adoption failure recorded above remains unresolved at the tested revision.

### Manual edits and unpend — procedure only, not rerun in this session

1. Create a fresh initial grouped branch in an isolated lane.
2. Add `manual.txt` on that branch with a human-authored commit; record its SHA.
3. Apply the pending configuration from `scenarios/pending.json` and run Renovate; inspect the Dependency Dashboard.
4. Check only that branch's unpend box and rerun.
5. Verify whether the manual file/commit is preserved, the branch returns to edited/blocked status, and the PR retains its edited-branch warning. Unpend alone is not explicit consent to discard human edits.
6. Test an explicit rebase request separately. Record actual behavior instead of treating this procedure as a passing result.

## Runner

Use an authenticated Renovate source runner, scoped to the intended fixture repositories and isolated branch prefixes. For GitHub use `platform=github`; any GitLab run requires separate credentials, fixtures, and evidence. Keep tokens in the environment. A source invocation is `pnpm start --platform=<platform> --repositories=<owner/repo>` with the runner's normal configuration and execution settings.

Fixture creation and written procedures do not validate remote bot behavior. Future verification should append a separately dated session with its own exact revisions, conditions, observations, and limitations rather than silently relabeling this session as current.
