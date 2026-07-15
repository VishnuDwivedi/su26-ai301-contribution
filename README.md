# Contribution #1: expand cleardb and clearorders command tests

**Contribution Number:** 1
**Student:** Vishnu Dwivedi
**Issue:** https://github.com/saleor/saleor/issues/14199
**Pull Request:** https://github.com/saleor/saleor/pull/19364
**Status:** Phase IV In Progress — Addressing Maintainer Review Feedback

---

## Why I Chose This Issue

I chose this issue because it offers a well-scoped, beginner-friendly entry point into a real-world open source Django project. Writing unit tests for the `cleardb` management command doesn't require deep knowledge of Saleor's business logic — it requires understanding Django's testing patterns and model relations, which I can learn quickly. I'm also drawn to this issue because tests are foundational to software quality, and contributing tests that protect against regressions is genuinely useful work rather than busywork. The maintainer recently refreshed the issue with a contribution guide, which signals it's still actively wanted.

---

## Understanding the Issue

### Problem Description

The `cleardb` and `clearorders` management commands in Saleor lack sufficient unit test coverage. Saleor has many models with protected foreign key relationships, and when those relationships change, the commands can silently break. Without comprehensive tests, developers may miss these failures during model refactors.

### Expected Behavior

The `cleardb` and `clearorders` commands should have unit tests covering key scenarios — including protected foreign key relationships — so that model relation changes are caught before they cause runtime errors.

### Current Behavior

The commands existed and functioned, but had zero test coverage, leaving edge cases and relation-dependent failure modes completely untested.

### Affected Components

- `saleor/core/management/commands/cleardb.py`
- `saleor/core/management/commands/clearorders.py`
- `saleor/core/management/tests/test_cleardb.py` ← created
- `saleor/core/management/tests/test_clearorders.py` ← created

---

## Reproduction Process

### Environment Setup

- Forked `saleor/saleor` and created branch `fix-issue-14199` on GitHub
- Cloned fork locally on macOS (Apple Silicon)
- Installed `libmagic` via `brew install libmagic` (required by Saleor's thumbnail module)
- Started PostgreSQL and Redis via `docker compose up db cache -d` inside `.devcontainer/`
- Created `.env` file from template
- Installed dependencies with `uv sync`
- Ran tests with `uv run poe test saleor/core/management/tests/ -n0`

### Steps to Reproduce

This is a test coverage issue rather than a runtime bug. "Reproducing" means confirming the gap:

1. Navigate to `saleor/core/management/commands/cleardb.py`
2. Search for a corresponding test file — none exists
3. Run `uv run poe test saleor/core/management/` — no tests collected
4. Confirm zero coverage for both `cleardb` and `clearorders` commands

### Reproduction Evidence

- **Branch:** https://github.com/VishnuDwivedi/saleor/tree/fix-issue-14199
- **Finding:** No test directory existed at `saleor/core/management/tests/` — confirmed zero prior coverage

---

## Solution Approach

### Analysis

The test settings run with `DEBUG=False`, so every `cleardb` call requires either `force=True` or `settings.DEBUG = True` to be set in the test. This was the key insight needed to make the tests pass.

### Proposed Solution

Created two new test files covering the main scenarios for both commands.

### Implementation Plan (UMPIRE)

**Understand:** Both commands had zero test coverage. The `cleardb` command has a DEBUG guard that required special handling in tests.

**Match:** Used Saleor's existing `given/when/then` pattern from other test files. Used existing fixtures (`order`, `checkout`, `customer_user`, `staff_user`, `superuser`, `shipping_zone`) already defined in Saleor's `conftest.py`.

**Plan:**
1. Create `saleor/core/management/tests/__init__.py`
2. Create `test_cleardb.py` with 8 test cases
3. Create `test_clearorders.py` with 4 test cases
4. Fix `DEBUG=False` issue by passing `settings` fixture and setting `settings.DEBUG = True`
5. Run full test suite to confirm all pass

**Implement:** https://github.com/VishnuDwivedi/saleor/tree/fix-issue-14199

**Review:** Followed Saleor's `given/when/then` pattern, used relative imports, followed commit message conventions.

**Evaluate:** All 12 tests pass with 0 failures.

---

## Testing Strategy

### Unit Tests Added (Initial Submission)

**test_cleardb.py (8 tests):**
- `test_cleardb_raises_error_when_not_in_debug_mode` — confirms DEBUG guard works
- `test_cleardb_runs_in_debug_false_with_force_flag` — confirms `--force` bypasses guard
- `test_cleardb_removes_orders` — confirms orders are deleted
- `test_cleardb_removes_checkouts` — confirms checkouts are deleted
- `test_cleardb_removes_products_and_categories` — confirms catalog is cleared
- `test_cleardb_removes_customers_but_preserves_staff` — confirms staff accounts survive
- `test_cleardb_delete_staff_flag_removes_staff_but_preserves_superuser` — confirms superuser survives
- `test_cleardb_removes_shipping_zones` — confirms shipping data is cleared

**test_clearorders.py (4 tests):**
- `test_clearorders_removes_orders` — confirms orders are deleted
- `test_clearorders_removes_checkouts` — confirms checkouts are deleted
- `test_clearorders_preserves_customers_by_default` — confirms customers survive by default
- `test_clearorders_delete_customers_flag_removes_customers` — confirms `--delete-customers` works

### Test Results (Initial Submission)

All 12 tests passed, 0 failures.

---

## Week 5 Update — Pull Request Review Feedback

### PR Status

Opened [PR #19364](https://github.com/saleor/saleor/pull/19364) two weeks ago. Reviewer **wcislo-saleor** (Saleor team member) requested changes, and the PR was subsequently closed by the reviewer (branch `fix-issue-14199` still has unmerged commits, so it can be reopened once revised).

### Reviewer Feedback (Summary)

The reviewer's core point: issue #14199 specifically calls out **protected foreign key relationships** as the failure mode to guard against, and referenced [PR #14198](https://github.com/saleor/saleor/pull/14198) as an example. My initial 12 tests cover normal deletion/preservation behavior (orders, checkouts, staff, customers, shipping zones) but don't exercise any model with a `PROTECT`-type foreign key relationship — so they don't actually test the scenario the issue was opened for.

### Root Cause I Identified

Looking at the linked example, `TransactionItem` has a protected foreign key back to `Order`/`Checkout`. Before [PR #14198](https://github.com/saleor/saleor/pull/14198), running `cleardb` with a transaction item attached to an order would raise a `ProtectedError`, since Django refuses to delete a row that's still protected-referenced. That PR fixed the bug by deleting transaction items before the orders/checkouts they reference. My test suite doesn't include a test that would catch a regression of that fix — which is exactly the gap the reviewer flagged.

The reviewer also raised a broader point: even with a `TransactionItem`-specific test added, the suite still isn't future-proof against *new* protected relations being added later without a corresponding test. I'm treating that as a valid but separate, larger-scope concern — noting it in my response rather than trying to solve it generically in this contribution.

### Plan to Address Feedback

1. Add test case(s) that create a `TransactionItem` linked to an order (and/or checkout), then run `cleardb`/`clearorders` and assert the command completes without raising `ProtectedError` and correctly clears the data.
2. Audit `saleor/core/management/commands/cleardb.py` and `clearorders.py` for other models with `on_delete=PROTECT` relations pointing at cleared models, to check whether additional scenarios beyond `TransactionItem` need coverage.
3. Push the new tests to the `fix-issue-14199` branch.
4. Reply to wcislo-saleor's review comment, acknowledging the specific gap, summarizing the added tests, and noting the future-proofing point as an acknowledged limitation outside this contribution's scope.
5. Reopen PR #19364 (or open a follow-up PR referencing it) once the new commits are pushed, and re-request review.

### Status as of Week 5

Issue #14199 remains open and unassigned on Saleor's tracker — no one else has picked it up, so I'm continuing this contribution rather than starting a new one. Currently mid-Phase IV: responding to maintainer feedback and preparing revised commits before requesting re-review.

### Status of WEEK 6 
I have been working on suggested changes and checking them in my environment. Almost close to finishing it.
