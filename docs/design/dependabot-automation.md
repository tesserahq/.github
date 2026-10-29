# Dependabot automation for portals

Status: draft for team review
Owner: platform / portals
Related: PR #5 (`chore/dependabot-major-tooling-gates`), `.github/workflows/dependabot-auto-merge.yml`, `.github/workflows/portal-tests.yml`

## Problem

Portals receive Dependabot updates as separate pull requests. Major updates often fail lint, typecheck, tests, or the build, and each failure is diagnosed by hand. The backlog grows, and portals fall behind on dependencies.

We need one process that:

- gives every portal the same validation, including a production build;
- groups related updates so there are fewer, more coherent PRs;
- auto-merges routine updates only after the relevant checks pass on the exact commit;
- leaves failing updates and anything outside policy for a human.

## Goals and non-goals

Goals:

- One validation contract shared by all portals.
- Policy for auto-merge defined in one reviewable place.
- Merges bound to the commit that was checked.
- Failing or out-of-policy updates are visible, not silently stuck.

Non-goals:

- Auto-merging production dependency majors. These stay human-reviewed.
- Auto-merging GitHub Actions updates. These stay human-reviewed.
- Replacing Dependabot's own version detection.

## Current state (what PR #5 would change)

- `dependabot-auto-merge.yml` auto-merges patch and minor updates after `code-quality`, `typecheck`, and `tests / tests` pass.
- PR #5 extends that to majors in five hard-coded `bun-major-*-tooling` groups. The eligibility condition is duplicated in two steps.
- The gate does not include a build check.
- Between the head-SHA check and `gh pr merge --auto`, a new commit can land. Auto-merge is then enabled on a commit that was never checked.
- `portal-tests.yml` reports success when a portal has no test script or no test files. A portal with no tests therefore passes the gate trivially. See open question 1.

## Design

The design has four parts. Each lives in one place.

### 1. Validation contract: one reusable workflow, every portal calls it

A reusable workflow in `tesserahq/.github` runs the same jobs for every portal:

| Job | Purpose | Gates auto-merge |
|-----|---------|------------------|
| `lint` | Style and static rules | Yes |
| `typecheck` | Type errors | Yes |
| `tests` | Unit and integration tests | Yes |
| `build` | Production build succeeds | Yes (new) |

Rules:

- Job names are fixed by the workflow. Gates match on those names, not on strings a caller invents.
- A skipped check (no tests, no script) must be reported explicitly and counted as **not passing** for auto-merge. It should not look green. Exact handling is open question 1.
- Branch protection, or an org ruleset, requires these checks wherever the plan allows it. Repos that cannot require checks rely on the auto-merge gate, so the gate must check the same set.

### 2. Grouping: a shared convention in each portal's `dependabot.yml`

Every portal uses the same group names:

- `dev-tooling`: build, bundler, CSS, lint, test, and type packages. Dev dependencies only.
- `prod-minor-patch`: production minor and patch updates.
- `prod-major`: production major updates. Never auto-eligible.
- `actions`: GitHub Actions updates. Never auto-eligible.

Groups are split by scope so one change is reviewed as one thing. Portal `dependabot.yml` files set the groups. The shared workflow only reads the group name and never hard-codes a list of groups that could drift.

### 3. Policy as data, read by one workflow

A single policy file defines which updates are eligible. Portals use the shared default, and may override it in `.github/dependabot-policy.yml`:

```yaml
# Shared default. Evaluated top to bottom; first match wins.
rules:
  - update-type: patch
    eligible: true
  - update-type: minor
    eligible: true
  - update-type: major
    dependency-type: direct:development
    group: dev-tooling
    eligible: true
  - update-type: major
    eligible: false           # everything else: human review
  - ecosystem: github_actions
    eligible: false
```

The shared workflow evaluates these rules once and writes an `eligible` output. Both the wait step and the merge step read that output. The duplicated `if:` in PR #5 goes away. Changing policy is a reviewed change to a data file.

Dependabot's `fetch-metadata` outputs drive the match. Open question 2 covers how those outputs behave for grouped PRs.

### 4. Merge: only after checks pass on the exact commit

1. Wait for the required checks on `HEAD_SHA`.
2. Verify the PR head still equals `HEAD_SHA`.
3. Enable auto-merge bound to that SHA, for example `gh pr merge --auto --squash --match-head-commit "$HEAD_SHA" "$PR_URL"`. Confirm `gh` enforces the match when `--auto` is set. If it does not, use another method to pin the merge to the checked SHA.

Supply chain, for majors that are eligible under policy:

- Install with `--ignore-scripts` in the gated checks, so install-time code from a new release does not run during validation.
- Enable a Dependabot `cooldown` so a release has time to be reported before it is picked up.
- Keep the lockfile diff visible in the PR so reviewers can see transitive changes.

## Failure and backlog handling

- A failing update is not left silent. The workflow adds the `needs-human` label and comments the failing job name and a link to its log.
- An update that is out of policy gets the same label, with a reason ("production major", "GitHub Actions").
- Out-of-policy majors stay as their own PRs. Where Dependabot allows, group them by week so they show up as a single reviewable batch.
- A weekly report (issue or summary) lists open `needs-human` PRs per portal, so the backlog is visible.

## Rollout

1. **Pilot on one portal.** Add the build job to the reusable workflow, the policy file, and the grouping config. Confirm on a real Dependabot PR that: the build is gated, the policy read produces the expected `eligible` value, and the merge is bound to the checked SHA.
2. **Fix the gaps before widening.** Close the merge race, decide the no-tests behavior (open question 1), and verify `fetch-metadata` outputs (open question 2).
3. **Roll out to the other portals.** Each portal gets the same caller workflow, group config, and policy reference. No per-portal logic.
4. **Review after one cycle.** Count auto-merged updates, reverts, and `needs-human` items. Adjust policy based on those numbers.

## Risks

- **A passing check does not prove safety.** Checks run the code, so a compromised release can pass them. Mitigated by keeping production majors manual, `--ignore-scripts`, cooldown, and eligibility limited to dev tooling.
- **Gaps in test coverage.** A portal with thin tests can auto-merge a broken update. Mitigated by the build gate and by counting no-tests as not passing (open question 1).
- **Policy drift between portals.** Mitigated by the shared default and a single policy evaluation.
- **Required-check names change.** If a job name changes, gates can wait forever. Mitigated by the workflow owning the names and by a timeout that fails closed.

## Open questions

1. **No-tests behavior.** Should a portal with no test script or files fail the gate, or should the gate allow it for `dev-tooling` only? Proposed: fail for all auto-merge eligibility, and let the portal opt in explicitly if it truly has no tests.
2. **`fetch-metadata` for grouped PRs.** How do `update-type`, `dependency-type`, and `package-ecosystem` behave when a group contains several dependencies, including a production major? Needs a test on a real grouped PR before the policy relies on it.
3. **Who owns the policy file?** Proposed: platform team owns the shared default, portal teams can override with a reviewed PR.
4. **Org ruleset or per-repo branch protection** for the required checks? Depends on plan limits for each repo.

## Decision on PR #5

Do not merge PR #5 as written. It hard-codes group names, duplicates the eligibility condition, has the merge race, and has no build gate. Close it or convert it to draft. Its core idea, allowing major updates to dev tooling when checks pass, survives as one rule in the policy file.
