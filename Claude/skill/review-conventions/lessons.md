# Lessons learned from code-review feedback

Append-only log of durable rules mined from real reviewer comments. Each entry: **Rule**, **Why** (a generalized example with enough context to judge edge cases), **How to apply**. Add a **Stack** tag when a lesson is language/framework-specific; leave it off when it's universal.

***

## Explanatory comments: follow the repo's bar, and default to restraint

**Rule:** First check what the repo's own docs and existing code say about comments. If the repo states a preference, follow it. If it doesn't, default to few or no explanatory comments in new production code, keeping one only when removing it would leave a future reader genuinely confused about a non-obvious invariant.

**Why:** Teams differ widely here. Some reviewers want the *why* documented; others expect the code and tests to carry the meaning and will ask for any explanatory comment to be removed, even one explaining a subtle invariant. Guessing wrong in either direction costs a review round.

**How to apply:** Scan the diff for comments added to production (non-test) code. For each, check it against the repo's stated or observed norm. If the norm is unclear, flag the comment for the user to decide instead of assuming it's fine to keep.

***

## Test/UI locators must not depend on structural position or a shared class

**Stack:** UI / end-to-end tests

**Rule:** Never make a new test locator (page object, selector, XPath) depend on element position (`:first-child`, `:last-child`, `:nth-child`, index-based) or on a class shared with unrelated elements — when a dedicated, purpose-named hook (class/attribute/test id) could be added to the element instead.

**Why:** A locator such as `.form-field .toggle-btn:first-child` only works while that container happens to hold exactly one toggle group. Add a second group later, or reuse the class elsewhere, and the locator silently matches the wrong element instead of failing loudly — the worst kind of test bug.

**How to apply:** When adding a new interactive element that tests need to target, add a dedicated class/attribute for it in the *same* change that introduces the element, and point the locator at that. Don't lean on DOM position or a class the element merely happens to share with others.

***

## Test granularity: follow the repo's philosophy, and avoid duplicated setup

**Rule:** Check whether the repo's test docs state a scenario-count philosophy (one consolidated flow vs. one test per case) and follow it from the first draft. If none is stated, avoid near-duplicate tests whose only difference is one or two setup steps — merge them into a single flow with sequential assertions, unless the cases are genuinely independent or isolating them materially helps debugging.

**Why:** Four end-to-end scenarios that repeat the same long setup and differ by a single toggle are slow to run and costly to maintain, and a reviewer will often ask to collapse them. Conversely, some teams prefer small isolated tests; the repo's own docs are what settle it.

**How to apply:** Before writing a new test file, look for a stated philosophy in the repo's test docs. If draft scenarios differ only by 1-2 setup steps, consider merging them, but defer to the repo's convention when one exists.

***

## A type that's "required" but always defensively guarded is lying

**Stack:** Typed languages (TypeScript, Kotlin, Swift, etc.)

**Rule:** If an interface field is typed as required, but every consumer defensively falls back with a default or null check anyway, that's a signal the type is wrong (should be optional, or the fallback is dead code) — call it out even if it predates the current change.

**Why:** A response field typed as required but always guarded by its single consumer means the type already misrepresents the real contract. Adding a second required field copied from that pattern spreads the same mismatch.

**How to apply:** When touching an interface or its consumers, check whether "optional in practice" fields are typed as required (or the reverse), and flag the mismatch — don't just copy the existing pattern forward uncritically.

***

## Gating/restriction logic must default to "allowed" when data is absent — compare strictly

**Rule:** New conditional logic that can *block* or *restrict* something must default to "allowed / not blocked" whenever the data needed to decide is absent (`undefined`/`null`/missing config). Use strict comparisons against the specific expected value rather than truthiness, so a missing value can never accidentally satisfy a block condition.

**Why:** When restrictions roll out per entity and most entities have no config at all, correctness for every *unconfigured* case depends entirely on this default. A truthy check instead of a strict one (`if (flag)` instead of `flag === false`) silently breaks every unconfigured case the moment the field goes from `false` to `undefined`.

**How to apply:** For any new "is this blocked / restricted / excluded" predicate driven by external or optional config, write out the "no config present" case explicitly as a test, and prefer strict comparisons over truthiness checks when the value can be absent. (The `===` syntax applies to JavaScript/TypeScript; use the equivalent strict check in other languages.)

***

## Derive, don't duplicate state

**Rule:** Prefer a computed/derived value over a second piece of stored state that a human has to remember to keep in sync.

**Why:** Option lists and gates built as values computed from existing state can't drift out of sync. A separately stored copy needs every update path to remember to refresh it, and one missed path produces a stale, inconsistent UI or record.

**How to apply:** Before adding a new stored field, check whether it can instead be computed from data that's already stored. If a value legitimately needs to be "frozen" or cleared at a transition (e.g. a selection that becomes invalid after another input changes), do that write in exactly one place, not at every read site that happens to notice the inconsistency.

***

## Every new branch needs a test for both the true and false path

**Rule:** Any new `if` / ternary / conditional expression added to logic under test needs a test proving both the case where it fires and the case where it doesn't — not just the case that motivated the change.

**Why:** A common gap: a new field is added to a "failed fields" list with only a positive-path test, and nothing proves the field stays absent when its own condition is false *while other things are still failing*. An unrelated "everything is valid" test doesn't cover that.

**How to apply:** For every new conditional added to a method under test, check for a test exercising the false path *specifically* — not one that only incidentally has the condition false.

***

## Mapped values need a test where both sides are spelled differently

**Rule:** When a display value and a wire/storage value happen to be spelled identically for one case, a test using only that case does not prove the mapping between them is real — it would pass even with the mapping missing or wired backwards.

**Why:** If the only naturally occurring test value has the same spelling as a label and as an enum constant, an implementation that compares display labels directly (skipping the label-to-enum mapping) still passes, then breaks for every other value whose label and enum differ.

**How to apply:** When adding a config-driven predicate that maps one vocabulary to another, add at least one test using a *fabricated* case where the two spellings differ, to prove the mapping step is actually happening.

***

## Comments must never embed a ticket/issue number

**Rule:** Never write a ticket or issue identifier (e.g. `PROJ-123`, `#4521`) inside a code comment. If a comment explains why something exists, state the mechanism or invariant directly — the ticket reference belongs in the commit message or PR description, which are the durable, checkable record of "why."

**Why:** A ticket number in a comment goes stale once the ticket is closed and forgotten, and, unlike a commit message that `git log`/`git blame` surfaces against the exact lines it touched, gives a future reader no reliable way back to the discussion. This is a specific case of the broader "don't reference the current task/fix/caller in comments" principle.

**How to apply:** Before finishing a change, scan every new comment for a ticket-key pattern (project prefix + separator + digits, or a bare `#123`-style reference). If found, delete the reference if the sentence stands without it, or rephrase to describe the mechanism/invariant only.

***
