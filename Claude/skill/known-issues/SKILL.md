***

name: known-issues
description: Checks a running personal log of previously-solved engineering issues (CI/e2e failures, git/merge blocks, tool or server connection failures, coverage regressions, tooling quirks, etc.) before diagnosing an error from scratch, and adds a new entry once a new issue is actually solved. Triggers on unexpected/confusing errors, blocked pushes or merges, pipeline/CI failures, tool or server connection failures, or any "why is this happening" debugging moment — check this first, before spending time re-diagnosing something that may already be solved.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Known issues log

A single growing file, co-located with this skill so the whole thing can be copied/shared as one unit:

```
issues-and-fixes.html   (next to this SKILL.md — the live file, read and updated by this skill)
```

Generic engineering issues and their real fixes — not tied to any repo, ticket, or feature. It exists so the same debugging work is never repeated twice.

Optionally, the user can keep a snapshot copy for backup/browsing at a location of their choosing: `<BACKUP_LOCATION>/issues-and-fixes.html` (a placeholder — ask the user for the path the first time a backup is wanted, and don't assume one). That copy is **not** kept in sync automatically — it only reflects whatever it was last manually copied to. The file next to this SKILL.md is the one to read and update; only copy changes to the backup location when the user explicitly asks for a refresh.

## Step 1 — Check before diagnosing

The moment you hit an unexpected error (CI/e2e failure, blocked git push/merge, tool/server connection failure, coverage regression, etc.), search this file for the symptom **before** starting to diagnose from scratch — grep for the error text, tool name, or a distinctive keyword against the `data-text` attributes and entry titles.

## Step 2 — A matching entry exists

* Apply the fix described in that entry's "How it was fixed" / "Permanent fix" sections directly.
* If it resolves the issue: done, no need to touch the file.
* If it does **not** resolve it (same symptom, different real cause this time): diagnose and confirm the actual fix, then add it as an **additional fix option inside the same `<details>` entry** — a second `<section class="fix">` block (e.g. labeled "Fix attempt 2"), right after the existing one. Never remove or overwrite the original fix; it may still be the correct one for someone hitting this the other way.

## Step 3 — No matching entry

Diagnose and solve the issue as normal. Only **after the fix is confirmed working** (never a guess or something untested), add a new entry — never add an issue you haven't actually solved yet.

## Entry format

Match the existing structure exactly — one `<details class="issue" data-text="...">` block per issue, under `#list`:

* `data-text`: lowercase keywords for the search box (tool names, error snippets, symptoms).
* `<summary>`: a `<span class="tag">CATEGORY</span>` (e.g. `E2E`, `Git`, `MCP`, `Coverage`, `Tooling`, `Testing`) + a short, generic title.
* Three sections: `class="wrong"` (What went wrong), `class="fix"` (How it was fixed), and an untagged section (Permanent fix / how to avoid).
* Bullets only, no prose paragraphs. Straight to the point — this file is read often and must stay cheap to scan.

**Keep every entry generic** — describe the *class* of problem (e.g. "a link to a downloadable file", "an e2e job", "a PR/MR blocked on fast-forward"), never the specific ticket, product, company, internal tool name, or business context it happened to come up in. Tool- or framework-specific entries are fine (e.g. a Gradle or AWS CLI quirk) as long as nothing ties them to one organization or project. If a lesson is genuinely specific to one repo's own conventions, it belongs in that repo's own docs instead, not here.
