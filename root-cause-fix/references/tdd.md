# Test-driven implementation of a bug fix

Read this when you have reached phase 6 of the `root-cause-fix` workflow and
are about to write tests.

## Choosing the test level

| Situation | Test level |
|---|---|
| Pure logic, parsing, formatting, calculation, branching | Unit test |
| Behaviour only observable against a real database, container, or message broker | Integration test |
| Defect in generated SQL, mapping, or persistence behaviour | Integration test — a mocked repository or DAO will happily accept invalid SQL and hide the bug |
| UI wiring (wrong argument passed to a service) | Test the service/business layer it calls; assert the behaviour, not the widget |

When in doubt, pick the level at which the reported error can actually be
produced. If a unit test cannot reproduce the reported failure, it is not the
right level.

**Terminology.** "Integration test" here means a test that runs against a real
out-of-process dependency — a database, container or broker — rather than a
double. Projects name and wire these differently (`*IT` classes under Maven
Failsafe, a separate Gradle source set, a pytest marker, a dedicated npm script,
"medium" or "narrow integration" tests); use whatever the repository already
calls them, and check whether they are bound to a separate lifecycle phase,
profile or flag before concluding that "the tests did not run".

## Naming

Unless the project already has a different convention, name tests so the
expectation reads as a sentence, starting with `should`:

```
shouldParseQuantityWhenItContainsThousandsSeparator
shouldRejectQuantityWithClearMessageWhenSeparatorIsMisplaced
shouldStillParsePlainQuantityWhenNoSeparatorIsPresent
```

Pattern: `should<ExpectedOutcome>When<Condition>` — or its snake_case form where
that is the language's norm
(`test_should_parse_quantity_when_it_contains_thousands_separator`). Avoid
`testFoo1`, `testBugfix`, or the issue key as the whole name — put the issue key
in a comment above the test instead:

```
// ISSUE-123: a quantity with a thousands separator must parse, not throw.
```

## The red step is not optional

1. Write the test with the **exact** input from the report (same characters,
   same empty string, same order of parameters).
2. Run it **before** touching production code.
3. Compare the failure to the reported one. It must be the *same* error type,
   caused by the *same* mechanism; the message text may differ between
   environments (database dialect, locale, library version) — note any such
   difference. If it fails with a compilation error, a missing fixture, or an
   unrelated exception, the test is not yet a reproduction — fix the test first.
   If it **passes**, check that the code under test is really the code in the
   working tree before you blame the test: rebuild or reinstall the module the
   test loads, since a stale build artifact can already behave differently.
4. Only when the red matches the report, write the fix.

If no matching red can be produced, do not write the fix — take the exit
described in `SKILL.md` instead.

Record the before/after numbers; they are the evidence that the fix works:

```
before: 9 tests, 3 failed  (NumberFormatException: For input string: "1,000")
after:  9 tests, 0 failed
```

## Coverage to aim for

Reproducing the reported case is the minimum. Add, in this order:

1. **The reported failing input** (the regression guard).
2. **The neighbouring inputs** — the near misses are where naive fixes break:
   e.g. if `"1,000"` must parse, also test `"1,000,000"`, `"1,00"` and `","`.
3. **The valid input**, to prove the fix did not disable the feature. This is
   the test that catches "fixed the crash by silently treating every malformed
   quantity as zero".
4. **The adjacent behaviour** the change could disturb (negative numbers,
   locale-specific separators, null or empty handling, sort order).

Keep the count proportionate — four to nine focused tests is usually right for a
single defect.

## Green and regression

- Re-run the new tests until they pass without modifying them to fit the code.
  Changing an assertion to match the observed output defeats the purpose.
- Then run the **whole test class** and the neighbouring classes of the same
  feature area.
- If the project builds modules separately, make sure the module containing the
  fix is rebuilt/installed before running tests from another module, otherwise
  you will test the old binary and conclude the fix did not work.
- Never leave a test disabled, ignored, or commented out to make the build green.

## Reporting the result

State plainly:

- which tests were added and what each one guards,
- the before/after run counts,
- the files changed,
- what is still **not** covered or still broken (e.g. a second symptom outside
  the agreed scope).
