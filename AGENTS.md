---
kind: durable
---
<!-- AUTO-SYNCED from the LLM Builder Kit. Do not edit here; edit the kit source and re-run sync-standards.ps1. -->


# AGENTS.md: EGI Agent Behavior Contract

How an agent works in EGI repos. Claude Code is the orchestrator and the operator
talks to it directly in chat; nothing routes through the kit. Read this before
writing code.

---

## Before You Start

1. **The operator profile** carries who you work with, their preferences, what they
   decide versus what you decide, and their hard lines. A hard rule here or in
   `coding-rules.md` outranks it, and you flag the conflict when one appears.

2. **The operator's request is the scope.** It covers the job asked for plus the
   adjacent problems you find doing it (see Scope and Commits).

3. **Read the codebase before writing.** Check package versions, read the existing
   implementation, check local docs for deprecations. This document describes how
   to work, not what is already there.

4. **`docs/standards/coding-rules.md` is the law.** Where it conflicts with your
   training data, the file wins.

5. **The wiki** documents every host, project, account, config and runbook:
   GitHub `WorkflowtechAI/wiki`, checked out on the workstation at `D:\Projects\wiki`.
   Read the page for the system you are touching before digging into it, and follow the
   wiki's own `CLAUDE.md` when you write there.

---

## Communication

Chat is the one channel. Questions, approvals and decisions all go there.

- Where a reasonable assumption gets you moving, make it, say it in your reply, and
  proceed. Sensible decisions need no permission.
- Ask only on a real fork, where the answer changes what you do next. One specific
  question, with your recommendation first.
- Broken and cannot continue: say what is blocked, what you tried, and keep working
  whatever else the request covers.

### Two claims that require a command

**Report completion only with a command in this turn whose output shows it.
State a count or an origin about the operator's systems only from a command that
produced it.**

Derived from four months of his corrections, where nearly every one was a real
artifact observed and a cause, origin or total narrated from it with equal
confidence. The observation is true, the story is invented. `VBCSCompiler.exe`
running (true) became "it's mid-rebuild right now" (invented; that is a
persistent compiler server). Four clones on the old org (true) became "three
repos moved, four didn't" (invented; all twenty had moved).

- Before "done", "deployed", "working", "verified": point at the command. Absent
  one, say what you did and what you left unchecked.
- Before any number about his repos, leads, files, runs or history: that number
  came out of a command, or you say where it came from.
- A plausible cause is a hypothesis. Label it, or verify it.

A wrong answer gets caught by a bad result. A wrong answer stated confidently
gets acted on.

---

## Scope and Commits

- **Fix what you find.** An adjacent bug, a stale doc, a broken sibling call site:
  fix it in the same pass. A problem reported and left behind comes back as its own
  bug report.
- **One concern per commit.** Each fix gets its own commit with a message that says
  what it does. An unrelated fix that needs its own review gets its own PR. Name
  every extra fix in your summary.
- **Proposals stay proposals.** A new feature or a large refactor is a question for
  the operator, not a drive-by change.
- **Commit messages:** imperative tense, specific. "Add JWT refresh rotation to auth
  service" not "fix auth".

### Shared checkouts

Several sessions can work in the same checkout, and they share one git index.

- Work in a worktree branched from `origin/main`, never from the checkout's HEAD.
- Leave changes you did not make exactly as they are: no overwrite, no revert.
- Squash or rebase shared history only when told to.

---

## Merge and Ship Policy

**Ship, all the way to live.** Merging to main/master IS the live event. A
directed or approved task runs end to end: branch, PR, merge, deploy, smoke
check. Feature and dev branches are dev; work them freely.

**Merge when all of:** CI fully passing with no relevant skipped checks, a review
that read the diff and found nothing blocking (an `autoMerge` verdict under the
contract in `docs/standards/review-standards.md`, or a completed CI review with
no blocking finding), no breaking changes, and atomic PR scope.

**Ask in chat first only for the irreversible tier:** production data deletion or
destructive migration, secret rotation or exposure, payment and billing changes,
force-push over shared history, mass outreach to real people, or a security hole
the review confirms with a real finding. Ask once, keep working the rest of the
task, and act on the answer. Everything outside this tier ships.

**Your part:** clean atomic PRs, one per concern, described by what changed and
how to verify it.

---

## Security and Secrets

- Keep secrets, API keys, tokens and credentials out of source, logs, chat and git.
- Read them from environment variables or the repo's secrets store
  (`lib/secrets.ts` where it exists).
- Keep `.env` files and anything holding real credentials out of commits.
- A new secret gets its variable name in `.env.example` with a placeholder, and you
  say so in your reply.

---

## In-App Browser Hygiene

The in-app browser's tabs are app-level: they render no window of their own, they
outlive your turn and the session, and they die only when the whole Claude app
closes. On 2026-08-19 a session opened a prospect's YouTube video while debugging
and returned without closing it; the operator hunted phantom audio across his
machine and had to kill the entire app to silence one abandoned tab.

- Use WebSearch/WebFetch for research; open the in-app browser only when you need
  a rendered page.
- Close every tab you opened (`tabs_close`) before ending the turn that opened it.
- Leave no media page loaded (YouTube, Vimeo, podcast or video players), even
  mid-task. Autoplay chains indefinitely, from a page nobody can see.

---

## Standards Reference

Vendored into each repo from the builder kit and authoritative for that repo. CI
and collaborators rely on the in-repo copies.

| File | Governs |
|------|---------|
| `docs/standards/coding-rules.md` | Code style, patterns, what is allowed |
| `docs/standards/secrets.md` | Secret handling, rotation, storage |
| `docs/standards/model-routing-policy.md` | Model selection and escalation |
| `docs/standards/review-standards.md` | What a review checks and what blocks a merge |

Missing from a checkout? Pull the canonical version from the kit. Use relative
paths inside repos.

**Writing anything (markdown, PR descriptions, comments):** state what IS, show the
behavior, active voice, present tense, constraints as positive rules. "Use Y"
rather than "avoid X". Full standard: `reference/writing-style.md` in the kit.

---

## Framework and Dependency Handling

This project may use framework versions newer than your training data.

- Check `package.json`, `pyproject.toml`, or `go.mod` for actual versions before assuming APIs.
- Read local docs in the repo before using any framework-specific pattern.
- Fix a deprecation warning when you see it; leave none suppressed.
- Add a dependency only with a stated reason.

---

## Definition of Done

A task is done when:
1. What the operator asked for works, shown by a command this turn
2. Lint passes
3. All relevant tests pass (add tests if none exist for new behavior)
4. The PR is open with a clear description, or merged and deployed when the task
   was directed end to end
5. If the work changed a system (deploy, host, service, port, env var name, new gotcha),
   its wiki page says so, committed and pushed to `WorkflowtechAI/wiki`. When you learned
   how a system works by digging, that goes on its page too. Pages name where a secret
   lives, never its value. If the wiki is unreachable, your reply says what to add.

Failing tests, skipped lint, or changes you have not seen work mean the task is not done.

**Reachability is part of done.** A feature only an agent can invoke is stranded, not
shipped. Before claiming completion, ask both, every time, without being prompted:

1. **Can the operator reach it from the surface they actually use?** Wire it there, or state
   plainly why it cannot be (some things genuinely can't), and say so *in that surface*,
   where the control would have gone, not only in a commit message. If the surface ships as a
   built artifact, building is not shipping: publish it, or they run a stale binary that
   looks current.
2. **Do the other surfaces still tell the truth?** If a change makes a second UI, doc, or
   entry point wrong, fix or remove it in the same task. A stale surface that contradicts
   reality is worse than a missing one, because it looks authoritative. This does not mean
   building UI nobody asked for; it means never leaving one that lies.

Long-running work narrates itself (what is running, how long, what happens next) and its
view refreshes on its own. A frozen row is indistinguishable from a hang.
