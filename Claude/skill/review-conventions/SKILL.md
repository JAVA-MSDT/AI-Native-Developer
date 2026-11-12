***

name: review-conventions
description: Reviews uncommitted code changes against the repo's own documented conventions plus a running checklist of durable lessons mined from past code-review feedback. Invoke manually right before committing to catch convention violations and get refactor suggestions before a human reviewer has to.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Review conventions

Runs on whatever is currently uncommitted (or the most recent commit if nothing is staged) in the current repo. Two jobs: (1) enforce whatever *this* repo already documents about its own conventions, and (2) catch the categories of mistake that have actually drawn reviewer comments before, tracked in `lessons.md` next to this file — which grows over time as new feedback comes in.

This skill only suggests. It never edits code on its own — findings are for the user to accept, tweak, or reject.

## Step 1 — Scope the diff

* Run `git status` and `git diff` (working tree + staged). If both are empty, review `git show HEAD` instead and say so.
* Only review files that actually changed. Don't re-review the whole repo, and don't wander into unrelated files just because they're nearby.
* Note the language/stack of the changed files — the lessons file has some entries tagged by stack; skip the ones that don't apply.

## Step 2 — Discover this repo's own convention docs

Every repo organizes its documentation differently, so don't assume a fixed layout. Look for convention and guideline material wherever it lives, and skip silently what doesn't exist:

* Root-level guidance files: `README*`, `CONTRIBUTING*`, `AGENTS.md`, `CLAUDE.md`, `STYLEGUIDE*`, `CODE_STYLE*`, `ARCHITECTURE*`
* Documentation folders: `docs/`, `doc/`, `.docs/`, `.github/`, `wiki/`, and any agent/assistant-instruction folder the repo happens to use
* Any file whose name contains "convention", "guideline", "standard", "style", "architecture", or "testing" — find these explicitly with a glob rather than relying on a guessed name
* Lint/format config as a secondary, machine-checkable signal: `.eslintrc*`, `eslint.config.*`, `.prettierrc*`, `checkstyle.xml`, `.editorconfig`, `pyproject.toml`, `ruff.toml`, `.rubocop.yml`, `.golangci.*`, and similar
* If a root file points to or imports other docs, follow those pointers

Read what you find in full rather than sampling by filename — a change that looks purely UI-related can still violate something written in a testing or architecture doc. If an index/entry-point doc exists, read it first to see what the others cover.

**Repo-specific docs always win** over this skill's generic checklist when they conflict. This skill's job is to catch things a repo's own docs don't already cover, or that a human forgot to check against docs that do exist. When citing a finding, name the specific file and section it came from, not just the folder — that's what makes the finding checkable.

## Step 3 — Apply the durable lessons checklist

Read `lessons.md` in this same skill directory and check the diff against every entry that applies to the stack in play. Each entry has a **Rule**, **Why** (the reasoning behind it, so you can judge edge cases), and **How to apply**.

## Step 4 — Report findings

Use a severity-ranked review format: `file:line` — what's wrong — why it matters (cite the repo doc section or the lessons.md entry by name) — a concrete fix. Rank most-important first. Distinguish:

* **Repo-convention violation** — this repo's own docs say X, the diff does Y.
* **Recurring-mistake pattern** — matches a `lessons.md` entry.
* **General best practice** — neither of the above, but worth a mention (keep this category short; don't pad the report).

If nothing to flag, say so briefly. Don't invent findings to seem thorough — a clean pass is a valid outcome.

## Updating this skill

When a reviewer leaves feedback that isn't already covered by `lessons.md` or the repo's own docs, add a new entry to `lessons.md` so the same mistake is caught automatically next time. Keep new entries **generalized** — phrased as a rule that would hold up in a different repo or a different file, not a note about the one change it came from. If the lesson is genuinely specific to one repo's own conventions, it belongs in that repo's own convention docs instead, not here.
