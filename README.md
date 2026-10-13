# switchman

A Claude Code skill that picks the right skills for a task before Claude starts working.

You type `/switchman` followed by your request. switchman looks at the skills you have installed, keeps the ones that fit (three at most), loads only those, and then does the task.

## The problem

Once you have a dozen skills installed, two things go wrong:

- **Claude skips a skill that would have helped.** You ask for a bug fix and it starts patching without loading your debugging skill. Automatic triggering depends on how a description happens to match your wording, and it often misses.
- **Loading everything to be safe costs a lot.** Each full skill file can run to hundreds of lines. Pulling in five of them "just in case" fills the context with instructions that have nothing to do with the task.

switchman is a short, explicit routing step between your request and the work.

## Why "switchman"

On a railway, the switchman sets the points so that each train goes down the right track. The trains are already there and so are the tracks. The switchman just decides where each train goes.

This skill does the same with your request: it adds no new abilities, it sends the task to the skills you already have.

## How it works

1. **It decides from what Claude already knows.** Claude Code puts the name and description of every installed skill in Claude's context. switchman only uses that. It never lists or reads your skills folder, so choosing costs almost nothing.
2. **It checks a routing table first.** A small table maps kinds of task (bug, design, review...) to skill names. Skills in the table that you don't have are skipped without comment.
3. **It falls back on the descriptions.** If no row fits, it reads through the descriptions it already sees. If nothing matches, it tells you in one line and carries on without a skill.
4. **It loads at most three skills.** Only the chosen skills get their full `SKILL.md` read.
5. **It respects user-only skills.** If the right skill is one only you can start, switchman doesn't run it. It tells you in one sentence that it would help.
6. **It announces its choice on one line, then works:**

   ```
   Skills: systematic-debugging (stack trace points to a race in the login handler)
   ```

switchman only runs when you call it. Claude will never trigger it on its own.

## Installation

Pick one of the three methods.

### 1. As a Claude Code plugin

In Claude Code:

```
/plugin marketplace add julesgayet/switchman
/plugin install switchman@switchman
```

Plugin skills carry the plugin name as a prefix, so with this method the commands are `/switchman:switchman <task>` and `/switchman:switchman-init`.

### 2. With `npx skills`

```bash
npx skills add julesgayet/switchman -a claude-code -g
```

`-g` installs it for every project (in `~/.claude/skills/`). Leave it out to install it in the current project only.

### 3. By hand

Copy the two skill folders into your personal skills folder:

```bash
git clone https://github.com/julesgayet/switchman.git /tmp/switchman && cp -R /tmp/switchman/skills/* ~/.claude/skills/
```

That gives you `~/.claude/skills/switchman/` and `~/.claude/skills/switchman-init/`.

### Check that it's installed

Start a new Claude Code session and type `/switch`. `switchman` and `switchman-init` should show up in the menu. You can also run `/skills` to see the full list.

Then run `/switchman-init` once to build a routing table from your own skills (see below).

## Usage

```
/switchman the signup form accepts empty emails, fix it and add a test
```

Claude answers along these lines:

```
Skills: systematic-debugging, test-driven-development (validation bug, needs a regression test)
```

Then it fixes the bug using those two skills.

Another example, with nothing relevant installed:

```
/switchman rename the "utils" folder to "lib" and update the imports
No skill fits this task, continuing without one.
```

## Building your routing table

The table at the bottom of `switchman/SKILL.md` ships as an example built from popular public skills. To replace it with one built from what you actually have, run:

```
/switchman-init
```

It goes through every installed skill, including the manual-only ones Claude normally can't see, groups them by kind of task, and shows you the table it proposes. Nothing is written until you say yes. Manual-only skills get a `(manual)` tag, so switchman suggests them instead of trying to run them.

Run it again whenever you install or remove skills. This is the only step that reads your skills folders; `/switchman` itself never does.

### Editing the table by hand

You can also write the table yourself:

```markdown
| Task | Skills, best first |
|---|---|
| UI, design, frontend | frontend-design, impeccable |
| Bug, error, failing test | systematic-debugging, diagnosing-bugs |
| Database, migrations | supabase-postgres-best-practices |
```

A few tips:

- Use the skill's name as it appears in `/skills`. Plugin prefixes (`my-plugin:tdd`) can be left out.
- Add ` (manual)` after a skill that only you can start.
- Order matters: the first installed skill in a row is preferred.
- Keep the table short. Claude reads it every time you run switchman, so each line has a cost.
- Leaving a skill out of the table is fine. The fallback on descriptions will still find it.

Where to edit depends on how you installed:

- By hand or with `npx skills`: edit `~/.claude/skills/switchman/SKILL.md` directly.
- As a plugin: the installed copy gets replaced when the plugin updates, table included. Run `/switchman:switchman-init` again after an update, or use the manual install.

## Known limits

- **It only chooses as well as your skill descriptions let it.** A skill with a vague description is hard to match, for switchman as much as for Claude.
- **It doesn't see manual-only skills on its own.** Claude Code leaves the descriptions of skills marked `disable-model-invocation` out of Claude's context. switchman can only suggest one of those if it is in your routing table, which `/switchman-init` takes care of.
- **The table goes stale.** A skill installed after your last `/switchman-init` is only found through the description fallback.
- **The three-skill limit is a hard cap.** Rare tasks that really need four skills will lose one.
- **It is manual on purpose.** If you forget to type `/switchman`, Claude falls back to its normal behavior.
- **The plugin version has longer commands** (`/switchman:switchman`, `/switchman:switchman-init`), because Claude Code namespaces plugin skills.

## License

[MIT](LICENSE)
