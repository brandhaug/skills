---
name: remove-tautological-tests
description: Find and remove tautological tests — change-detector tests that mirror the code under test and break on any refactor without catching defects. Patterns include checksum assertions, mock-theater interaction tests, echo assertions, duplicate-algorithm tests, and snapshots with no oracle. Use when writing or reviewing tests, after a behavior-preserving refactor broke tests, or when the user says tests are tautological, circular, or "just testing the implementation".
---

# Change-detector

A **change-detector** is a test that is a transformation of the same information as the code under test. It breaks in response to any change to the production code — including behavior-preserving ones — while being equally likely to pass a correct or an incorrect implementation. It provides negative value: it catches no defects and charges maintenance for the privilege. Rewritten or deleted, never fixed mechanically.

Source: Google Testing on the Toilet, ["Change-Detector Tests Considered Harmful"](https://testing.googleblog.com/2015/01/change-detector-tests-considered-harmful.html) (Alex Eagle, 2015).

Two ways in:

- **Writing new tests** — run the litmus test below on each test before you keep it. If you can't state the contract it protects, you're transcribing, not testing.
- **Sweeping existing tests** — work through the checklist pattern by pattern against every test in scope (a file, directory, PR diff, or the tests touched by recent changes).

## The litmus test

Apply in order; one strong signal is enough to flag:

1. **Derivative** — could a correct implementation and an incorrect one pass it equally? Then it verifies nothing.
2. **Refactor-fragile** — would it fail on a rename, a parameter added, or internal reordering that preserved behavior? Then it detects changes, not defects.
3. **No oracle** — where does the expected value come from? If from the code under test itself (or a recording of it) rather than from the contract, there is nothing being checked.

## Checklist

**Structure copying**

1. **Checksum tests** — asserts on the source's own shape: reading source files and asserting lines, reflection asserting a method or field exists, asserting on private state or method signatures
2. **Configuration mirrors** — expected values copied verbatim from production: the error string defined next to the `throw`, the enum names, the constant table

**Implementation echoing**

3. **Echo assertions** — the expected value is computed by calling the code under test, or compared to itself: `assert result === result`, `expect(f(x)).toEqual(f(x))`
4. **Duplicate-algorithm tests** — the test reimplements the production algorithm and compares outputs; both now share the same bugs
5. **Snapshots without an oracle** — golden/master snapshots recorded from whatever the code currently produces, never hand-derived from requirements; they pass forever once recorded

**Mock theater**

6. **Call-graph transcription** — every collaborator mocked, then verified in the same order and shape the implementation calls them; the test is a restatement of the call graph and breaks on any internal reordering
7. **Pass-through tests** — with all dependencies mocked, the subject under test has no logic left, so the test only verifies the mocks were called with what the test itself passed in

## Rewrite or delete

For each hit, find the **contract** first: the behavior callers and users depend on. Then decide:

- **Rewrite** when an observable contract exists — assert on returned values, resulting state, or side effects at the system boundary. Derive every expected value from the contract, not from the code. Keep interaction verification only where the interaction *is* the contract (email sent, event published, API called with a payload) — assert on what crossed the boundary, never on sequencing between internal collaborators.
- **Delete** when there is no contract to protect — pure orchestration glue with logic living elsewhere. Replace coverage lost this way by testing the collaborator that holds the logic, or one integration test through the public seam.
- **Downgrade** a snapshot only if you can hand-derive expected values from requirements; otherwise delete it and let higher-level tests carry the logic.

Done means: every test in scope checked against every pattern, every hit deleted or rewritten to an assertion derived from a stated contract, no rewritten test still carrying a pattern, and the suite passing.

## Guardrails

- Outbound side effects callers depend on are behavior, not implementation detail. Mock theater is the tell, not mocking itself.
- A test failing after a behavior-preserving refactor that *does* assert behavior is pointing at a real problem — investigate before touching it.
- Speed is not a defense. Change-detectors run fast because they test nothing.
- When deleting the only coverage over critical logic, land the replacement in the same change — never leave the gap.
