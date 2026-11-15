---
kind: durable
---
<!-- AUTO-SYNCED from the LLM Builder Kit. Do not edit here; edit the kit source and re-run sync-standards.ps1. -->


# Review Standards

The contract every reviewer agent enforces before a change merges. Repos vendor
this as `docs/standards/review-standards.md` (synced from the builder kit); the
reviewer subagents (`code-reviewer`, `security-reviewer`) apply it to a branch,
a PR, or uncommitted work whenever a session asks for a verdict.

## What a review covers

1. **Correctness**: the change does what the operator asked, verified by
   running commands, and existing behavior it touches still works.
2. **Scope**: each commit holds one concern. Adjacent fixes are welcome and
   land in their own commits, named in the PR summary. An unrelated feature
   or a large refactor riding along is a finding.
3. **Done**: every completion claim in the PR or summary names the command
   that shows it, or says it was not run.
4. **Standards fit**: the change follows `docs/standards/coding-rules.md`:
   types at boundaries, dead code removed when in scope, comments explain
   intent, dependencies justified.
5. **Tests**: behavior changes come with test changes; the relevant suite
   passes; the tests still assert the intended behavior. Reject a new or
   changed test that falls into one of three anti-patterns: **tautological**
   (the assertion recomputes the expected value the way the code does, so it
   passes by construction and can never disagree with the implementation —
   expected values must come from an independent source of truth: a
   known-good literal, a worked example, the spec); **implementation-coupled**
   (mocks internal collaborators, tests a private method, or verifies through
   a side channel like querying the database directly instead of the public
   interface — the tell is a test that breaks on refactor with no behavior
   change); or **horizontal-slicing** (a PR that adds a block of tests for
   behavior the change doesn't implement yet, or tests written against the
   imagined shape of the code rather than what the diff actually does).
6. **Security**: the surfaces in the escalation list below, plus the security
   baseline in coding-rules. Any concern of any severity escalates.

## Evidence rules

- A verdict cites its findings as `file:line`.
- "Tests pass" means the reviewer ran them in this working tree, or CI ran them
  on this exact head. A summary's claim is a claim until a command confirms it.
- Review is read-and-run only. The reviewer executes checks (lint, typecheck,
  tests, builds) and leaves every source file untouched. A bug the reviewer
  finds goes in the verdict reasons for the author to fix.

## Verdict contract

The final line of a reviewer's output is exactly one line of valid JSON behind
the `REVIEW-VERDICT: ` prefix:

```
REVIEW-VERDICT: {"agent":"<code-reviewer|security-reviewer>","verdict":"<autoMerge|needs-operator>","summary":"<one line>","reasons":["<file:line concrete finding>", "..."]}
```

- `verdict: "autoMerge"`: the change satisfies every autoMerge criterion
  below; `reasons` is empty.
- `verdict: "needs-operator"`: anything else; `reasons` holds concrete,
  actionable findings.
- A missing or malformed verdict line reads as no verdict, and the change waits
  for the operator.

## autoMerge criteria (all required)

- What was asked works, verified by running
- Each commit holds one concern, and every adjacent fix is named
- Lint, typecheck, and relevant tests pass (or the repo has none)
- Public contracts preserved unless the task explicitly changes them
- Zero security flags from either reviewer
- No escalation surface touched

## Escalation surfaces (any one forces needs-operator)

The irreversible tier only: secret/credential rotation or exposure, payment or
billing changes, destructive or irreversible data operations (deletion,
destructive migration), authorization/permission changes, force-push over
shared history. This mirrors the merge policy in AGENTS.md. A directed
production deploy, a normal migration, or ordinary external-API work is not on
this list — those ship. Reaching prod is the intended outcome, not an
escalation.

## Verdict consumption

The verdict goes back to the session that asked for it, and into the PR when the
review ran there. It is advisory: GitHub auto-merge is off, so a merge still
follows the merge policy in AGENTS.md, and `needs-operator` means asking the
operator in chat.
