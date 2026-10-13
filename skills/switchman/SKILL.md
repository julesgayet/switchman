---
name: switchman
description: Route a task to the installed skills that fit it, load only those, then do the task. Run manually before a task.
argument-hint: <task>
disable-model-invocation: true
---

Task: $ARGUMENTS
(If empty, ask for the task and stop.)

Pick the installed skills that help with this task, load only those, then do it.

1. Choose only from what you already see: the names and descriptions of installed skills are in your context. Never list, glob or read a skills directory.
2. Check the routing table first. Use the listed skills that are installed (ignore any `plugin:` prefix when matching). Skip missing ones silently.
3. No row fits, or none of its skills is installed: scan the skill descriptions in your context. Nothing fits: say so in one line and do the task without a skill.
4. Keep 3 skills at most. Load each with the Skill tool, which reads its full SKILL.md. Read nothing about the others.
5. A skill only the user can start (marked `(manual)` in the table, or refused by the Skill tool): don't run it. Suggest it in one sentence: "`/name` would help here: <why>".
6. Always open your reply with one line, `Skills: X, Y (short reason)`, even when the task asks for a short answer. Then do the task.

## Routing table (example, run /switchman-init to build yours)

| Task | Skills, best first |
|---|---|
| UI, design, frontend | frontend-design, impeccable |
| Bug, error, failing test | systematic-debugging, diagnosing-bugs |
| New feature | brainstorming, test-driven-development, tdd |
| Code review | code-review, requesting-code-review |
| Security | security-review, vibe-security |
| Writing, docs, copy | no-ai-slop, doc-coauthoring |
