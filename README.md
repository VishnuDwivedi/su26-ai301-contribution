

```markdown
# Contribution #1: expand cleardb and clearorders command tests

**Contribution Number:** 1
**Student:** Vishnu Dwivedi
**Issue:** https://github.com/saleor/saleor/issues/14199
**Status:** Phase III Complete

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

### Unit Tests Added

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

### Test Results

```
12 passed, 1 warning in 1.11s
```

---

## Implementation Notes

### Week 1 Progress
Selected issue, set up fork, created working branch, reviewed CONTRIBUTING.md.

### Week 2 Progress
- Set up local environment on macOS with Docker, libmagic, uv
- Discovered no test directory existed — created from scratch
- Wrote 12 tests across 2 files
- Fixed `DEBUG=False` issue by using `settings` fixture
- All 12 tests passing

### Files Modified
- `saleor/core/management/tests/__init__.py` ← new
- `saleor/core/management/tests/test_cleardb.py` ← new
- `saleor/core/management/tests/test_clearorders.py` ← new

### Key Commits
- `TST add tests for cleardb and clearorders commands`

---

## Pull Request

PR Link: *(to be added in Phase IV)*

---

## Learnings & Reflections

### Technical Skills Gained
- Django management command testing patterns
- Using pytest `settings` fixture to override Django settings in tests
- Setting up a complex Django project locally with Docker dependencies

### Challenges Overcome
- Docker Desktop had a corrupted metadata database — fixed by restarting
- `libmagic` system dependency was missing — fixed with `brew install libmagic`
- Tests were failing with `DEBUG=False` — fixed by adding `settings.DEBUG = True` in each test

### Resources Used
- [Saleor CONTRIBUTING.md](https://github.com/saleor/saleor/blob/main/CONTRIBUTING.md)
- [Reference PR #14198](https://github.com/saleor/saleor/pull/14198)
- [Issue #14199](https://github.com/saleor/saleor/issues/14199)
```
###phase4
## Maintainer Feedback

**Status:** Iterating

The maintainer reviewed the pull request and requested changes. They noted that although the PR increases test coverage for the `cleardb` and `clearorders` commands, the current tests do not yet cover the protected foreign key scenarios described in Issue #14199.

Specifically, the maintainer suggested extending the contribution to test more complex cases where protected foreign key relationships can cause these commands to fail. They referenced the `TransactionItem` failure scenario from Issue #14197 as an example of the type of regression this issue is meant to catch.

## Next Steps

1. Investigate the failure scenario described in Issue #14197.
2. Identify Saleor models with protected foreign key relationships affected by `cleardb` or `clearorders`.
3. Add test cases that reproduce protected foreign key command failures.
4. Update the implementation if needed so the commands handle those cases correctly.
5. Re-run the test suite.
6. Push follow-up commits and re-request maintainer review.

