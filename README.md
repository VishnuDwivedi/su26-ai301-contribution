# Contribution #1: expand cleardb and clearorders command tests

**Contribution Number:** 1
**Student:** Vishnu Dwivedi
**Issue:** https://github.com/saleor/saleor/issues/14199
**Status:** Phase II In Progress

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

The commands exist and function, but test coverage is sparse, leaving edge cases and relation-dependent failure modes untested.

### Affected Components

- `saleor/management/commands/cleardb.py`
- `saleor/management/commands/clearorders.py`
- Corresponding test files under `saleor/management/tests/`

---

## Reproduction Process

### Environment Setup

Set up local development environment by forking the `saleor/saleor` repository and creating a working branch `fix-issue-14199` directly on GitHub. Currently in the process of cloning the repository locally and setting up the dev container per the project's CONTRIBUTING.md instructions.

### Steps to Reproduce

This is a test coverage issue rather than a runtime bug. "Reproducing" means identifying the gap:

1. Navigate to `saleor/management/commands/cleardb.py` in the codebase
2. Run existing tests with `uv run poe test saleor/management/`
3. Observe that protected foreign key relationship scenarios are not covered
4. Reference PR [#14198](https://github.com/saleor/saleor/pull/14198) (linked by maintainer) for examples of the kind of failures that lack test coverage

### Reproduction Evidence

- **Branch:** https://github.com/VishnuDwivedi/saleor/tree/fix-issue-14199
- **My findings:** Test coverage gap confirmed by reviewing existing test files and comparing against the model relations present in the codebase

---

## Solution Approach

### Implementation Plan (UMPIRE)

**Understand:**
The `cleardb` command lacks unit tests for model relation scenarios, particularly protected foreign keys, which can cause silent failures when Saleor model relations change during refactors.

**Match:**
PR [#14198](https://github.com/saleor/saleor/pull/14198) linked by the maintainer shows the exact pattern expected for these tests. Saleor uses `pytest` with a given/when/then structure per CONTRIBUTING.md.

**Plan:**
1. Clone the repo locally and set up the dev container
2. Run `uv run poe test saleor/management/` to see existing coverage
3. Review `cleardb.py` and `clearorders.py` to identify all model relations
4. Identify which protected FK relationships are not yet tested
5. Add new pytest test cases following the given/when/then pattern
6. Run the full test suite to confirm no regressions

**Implement:**
Branch: https://github.com/VishnuDwivedi/saleor/tree/fix-issue-14199 — implementation in progress

**Review:**
Will follow Saleor's given/when/then test pattern and commit message conventions per CONTRIBUTING.md before opening a PR.

**Evaluate:**
New tests should pass and cover previously untested protected FK scenarios. All existing tests should continue to pass.

---

## Testing Strategy

### Unit Tests

- Test case 1: `cleardb` runs successfully when all model relations are intact
- Test case 2: `cleardb` handles protected foreign key relationships without errors
- Test case 3: `clearorders` runs successfully under normal conditions
- Test case 4: Edge cases where related objects exist and could block deletion

### Manual Testing

Run `uv run poe test saleor/management/` before and after changes to confirm new tests pass and no regressions are introduced.

---

## Implementation Notes

### Week 1 Progress

Selected issue, set up fork, created working branch `fix-issue-14199` on GitHub. Reviewed CONTRIBUTING.md and identified setup path via dev container. Phase II submission in progress — local environment setup and test implementation to follow.

---

## Pull Request

PR Link: *(to be added in Phase IV)*

---

## Learnings & Reflections

*(To be filled in after PR submission)*

---

## Resources Used

- [Saleor CONTRIBUTING.md](https://github.com/saleor/saleor/blob/main/CONTRIBUTING.md)
- [Reference PR #14198](https://github.com/saleor/saleor/pull/14198)
- [Issue #14199](https://github.com/saleor/saleor/issues/14199)
```

