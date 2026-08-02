# Fake / Hollow Test Patterns

Read this during Step 6 of the audit. The goal is catching tests that pass regardless of whether the underlying code is correct — these are worse than missing tests because they create false confidence.

## Patterns to look for

**Assertions that can't fail**
- `assert True`, `expect(true).toBe(true)`, `assert 1 == 1`
- Asserting on a value the test itself just set, rather than a value produced by the code under test
- Comparing an object to itself, or to a freshly-constructed identical mock

**Mocking away the thing being tested**
- A test for `calculateTotal()` that mocks `calculateTotal()` itself (or mocks so much of its internals that no real logic runs)
- Mocking the database/API layer when the function under test *is* the database/API logic
- Over-broad mocks that replace an entire module when only one dependency needed mocking — check whether what's left to actually execute is trivial

**Empty or trivial bodies**
- Test functions with no assertions at all — they "pass" by not throwing, which almost anything does
- Tests that only check a function didn't throw, when the function's actual job is to return a correct value
- `it.skip` / `@pytest.mark.skip` / commented-out test bodies that are still counted in a "N tests passing" summary — check if skipped tests are being silently treated as passes in CI or reporting

**Swallowed failures**
- `try { ... } catch (e) { }` inside a test, silently absorbing an error that should have failed the test
- Assertions inside a callback that never actually runs (e.g. inside an unresolved promise, or a callback the mock never calls) — the test passes because the assertion never executed, not because it succeeded
- Snapshot tests where the snapshot was generated from broken output and never updated since — it now just re-confirms the bug

**Hardcoded / tautological expected values**
- Expected value computed with the exact same (possibly broken) logic as the code under test, so a bug in the logic passes the test because both sides are wrong the same way
- Test data and expected output that are suspiciously identical to what a first, minimal implementation would produce — sometimes a sign the test was written to match whatever the code already did, rather than what it should do

## How to report a finding

For each one you find, give:
1. File and test name
2. Which pattern above it matches
3. A concrete one-line explanation of a real bug that this test would fail to catch

That last part is what makes the finding actionable — "this test wouldn't catch it if `calculateTotal` returned 0 for every input" is far more useful than "this test looks weak."
