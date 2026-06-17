Here's your updated README:

---

**Contribution #1: expand cleardb and clearorders command tests**

**Contribution Number:** 1
**Student:** Vishnu Dwivedi
**Issue:** https://github.com/saleor/saleor/issues/14199
**Status:** Phase I In Progress

---

**Why I Chose This Issue**

I chose this issue because it offers a well-scoped, beginner-friendly entry point into a real-world open source Django project. Writing unit tests for the `cleardb` management command doesn't require deep knowledge of Saleor's business logic — it requires understanding Django's testing patterns and model relations, which I can learn quickly. I'm also drawn to this issue because tests are foundational to software quality, and contributing tests that protect against regressions is genuinely useful work rather than busywork. The maintainer recently refreshed the issue with a contribution guide, which signals it's still actively wanted.

---

**Understanding the Issue**

**Problem Description**

The `cleardb` and `clearorders` management commands in Saleor lack sufficient unit test coverage. Saleor has many models with protected foreign key relationships, and when those relationships change, the commands can silently break. Without comprehensive tests, developers may miss these failures during model refactors.

**Expected Behavior**

The `cleardb` and `clearorders` commands should have unit tests covering key scenarios — including protected foreign key relationships — so that model relation changes are caught before they cause runtime errors.

**Current Behavior**

The commands exist and function, but test coverage is sparse, leaving edge cases and relation-dependent failure modes untested.

---


---
