# Building an App from Scratch with Claude Code

A practical guide, in plain words.

Building an app with Claude Code is less about typing code and more about being a
good manager: you set the goal, give the context, and make sure there is a way to
check the work.

---

## The one idea everything hangs on

Claude Code reads your files, runs your commands, and edits your project on its own.
But everything it reads stays in a limited **context window**, and quality drops as
that window fills up — Claude starts forgetting earlier instructions and making more
mistakes.

Almost every technique below exists for two reasons:

1. **Keep the context clean.**
2. **Give Claude a way to check its own work.**

---

## The Steps

### Step 1 — Set up the folder first

```bash
mkdir my-app && cd my-app
git init
claude
```

Git matters. Claude's built-in checkpoints only track changes made through its own
file-editing tools — anything done through shell commands is not captured. Git is your
real safety net.

---

### Step 2 — Don't describe the app. Let Claude interview you.

This is the single biggest quality jump for a from-scratch build. Paste something like:

```text
I want to build [one-line description]. Interview me in detail using the
AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions — dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec
to SPEC.md.
```

Claude will raise things you hadn't thought about. A good spec:

- names the files and interfaces involved
- states what is **out of scope**
- ends with an end-to-end step that proves the feature works

Time spent making the spec precise pays off more than time spent watching the
implementation.

---

### Step 3 — Open a *fresh* session to plan

Start a new session after the spec is written. The new session has clean context
focused entirely on implementation.

Enter **plan mode**: press `Shift+Tab` until the status bar shows `⏸ plan mode on`
(or start with `claude --permission-mode plan`). In plan mode Claude reads and thinks
but does not change files.

```text
Read SPEC.md. Explore anything you need to. Then write a detailed,
step-by-step implementation plan to PLAN.md.
```

Press `Ctrl+G` to open the plan in your editor and edit it directly. **Read the plan
yourself.** This is the cheapest possible place to catch a bad idea.

> Skip planning for tiny tasks. If you could describe the diff in one sentence
> (a typo, a log line, a rename), just ask Claude to do it.

---

### Step 4 — Write a CLAUDE.md

Run `/init` to generate a starter file from your project structure, then refine it.

`CLAUDE.md` is read at the **start of every conversation**, so only put things in it
that always apply.

| Include | Exclude |
|---|---|
| Bash commands Claude can't guess | Anything Claude can learn by reading the code |
| Style rules that differ from the default | Standard language conventions |
| Test instructions and preferred test runner | Detailed API docs (link instead) |
| Branch naming and PR conventions | Info that changes often |
| Architecture decisions specific to this project | Long tutorials or explanations |
| Environment quirks (required env vars) | File-by-file descriptions |
| Common gotchas | "Write clean code" type advice |

Example:

```markdown
# Code style
- Use ES modules (import/export), not CommonJS
- Destructure imports when possible

# Workflow
- Typecheck after a series of changes
- Prefer running single tests, not the whole suite
```

**Keep it short.** For each line ask: *"Would removing this cause Claude to make a
mistake?"* If not, cut it. A bloated CLAUDE.md causes Claude to ignore your real
instructions. Run `/context` to confirm it loaded. Check it into git so the team can
improve it.

For knowledge that is only *sometimes* relevant, use a **skill** instead (see below)
so it doesn't sit in every conversation.

---

### Step 5 — Build the skeleton and a way to verify

Before any features: get the app running and get **one test command that works**.

A check Claude can run is the difference between a session you have to babysit and one
you can walk away from. The check can be:

- a test suite
- a build exit code
- a linter
- a script that diffs output against a fixture
- a browser screenshot compared to a design

Weak vs strong prompts:

| Weak | Strong |
|---|---|
| "implement a function that validates emails" | "write a `validateEmail` function. test cases: `user@example.com` → true, `invalid` → false, `user@.com` → false. run the tests after implementing" |
| "make the dashboard look better" | "[paste screenshot] implement this design. take a screenshot of the result, compare to the original, list the differences and fix them" |
| "the build is failing" | "the build fails with this error: [paste]. fix it and verify the build succeeds. address the root cause, don't suppress the error" |

Ask Claude to **show evidence** — the test output, the command it ran — rather than
just claiming success.

---

### Step 6 — Build one slice at a time, then `/clear`

Not "build the whole app." One vertical slice at a time: signup → dashboard →
notifications.

```text
Implement step 3 of PLAN.md. Write tests for it, run the suite,
and fix any failures.
```

After each finished slice, run `/clear` to reset context before an unrelated task.

Point at real things while you work:

- `@src/models/user.py` — pulls a file into context
- paste or drag in **screenshots** directly
- paste **URLs** for docs and API references
- `cat error.log | claude` — pipe data straight in

---

### Step 7 — Get an independent review before calling it done

Run the bundled `/code-review` skill, which reviews the current diff for bugs in a
fresh subagent. Or write the review yourself:

```text
Use a subagent to review the diff against PLAN.md. Check that every
requirement is implemented, the listed edge cases have tests, and nothing
outside the task's scope changed. Report gaps, not style preferences.
```

A reviewer in a fresh context sees only the diff, not the reasoning that produced it,
so it judges the result on its own terms.

> A reviewer asked to find gaps will always find some. Chasing every one leads to
> over-engineering — extra abstraction, defensive code, tests for cases that can't
> happen. Tell it to flag only what affects correctness, and treat the rest as optional.

---

### Step 8 — Commit each slice

```text
commit with a descriptive message and open a PR
```

Small commits mean a bad change is cheap to throw away.

---

## Techniques Worth Learning Early

### Steering and undo

| Action | What it does |
|---|---|
| `Esc` | Stop Claude mid-action; context is kept so you can redirect |
| `Esc Esc` or `/rewind` | Restore earlier conversation state, code state, or both |
| "undo that" | Have Claude revert its own changes |
| `/clear` | Wipe context between unrelated tasks |
| `/compact <instructions>` | Summarize context with a focus, e.g. `/compact focus on the API changes` |
| `/btw` | Ask a side question whose answer never enters the conversation history |

Correct Claude **early**. If you've corrected the same issue twice, the context is
polluted with failed approaches — `/clear` and write a better prompt using what you
learned.

### Subagents for research

```text
Use subagents to investigate how our auth system handles token refresh,
and whether we have existing OAuth utilities I should reuse.
```

Subagents explore in their own context window and report back a summary, so reading
40 files doesn't eat your main conversation.

You can also define your own in `.claude/agents/`:

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
---
You are a senior security engineer. Review code for injection flaws,
auth/authz mistakes, secrets in code, and insecure data handling.
Give specific line references and suggested fixes.
```

### Hooks for things that must always happen

CLAUDE.md rules are **advice**. Hooks are scripts that run automatically at set points
and are **guaranteed**. Just ask:

```text
Write a hook that runs eslint after every file edit.
Write a hook that blocks writes to the migrations folder.
```

Browse what's configured with `/hooks`; they live in `.claude/settings.json`.

### Skills for occasional knowledge

Create `.claude/skills/<name>/SKILL.md`. Claude applies them when relevant, or you
invoke them with `/skill-name`.

```markdown
---
name: api-conventions
description: REST API design conventions for our services
---
# API Conventions
- kebab-case for URL paths
- camelCase for JSON properties
- Always paginate list endpoints
- Version APIs in the path (/v1/, /v2/)
```

Add `disable-model-invocation: true` for workflows with side effects you only want to
trigger by hand.

### CLI tools beat guessing

Install `gh` and Claude handles GitHub issues and PRs properly. It also learns new
tools on request: *"Use `foo-cli --help` to learn this tool, then use it to do X."*

### Permissions

Use `/permissions` to pre-approve commands you trust (`npm run lint`, `git commit`)
and `/sandbox` for OS-level isolation. Fewer interruptions without giving up control.

### MCP servers

`claude mcp add` connects external tools — Notion, Figma, your database, an issue
tracker — so Claude can pull real context instead of guessing.

---

## Scaling Up (once the basics are comfortable)

- **Non-interactive mode:** `claude -p "prompt"` for CI, pre-commit hooks and scripts.
  Add `--output-format json` for parseable output.
- **Parallel sessions:** use git worktrees so two sessions don't collide.
- **Writer / Reviewer pattern:** one session implements, a second session reviews with
  fresh context. A fresh context reviews better because it isn't attached to code it
  just wrote.
- **Fan-out for migrations:** loop `claude -p` over a file list, scoping what it can do
  with `--allowedTools`. Test on 2–3 files first, then run the whole set.
- **Resume work:** `claude --continue` picks up where you left off; `/rename` gives
  sessions names so you can treat them like branches.

---

## The Mistakes That Will Bite You

| Failure | Fix |
|---|---|
| **Kitchen-sink session** — unrelated tasks piled into one context | `/clear` between tasks |
| **Correcting over and over** — context full of failed attempts | After two corrections, `/clear` and rewrite the prompt |
| **Over-stuffed CLAUDE.md** — real rules get lost in noise | Prune ruthlessly; convert hard rules into hooks |
| **Trust-then-verify gap** — plausible code that was never checked | Always provide tests, scripts or screenshots. If you can't verify it, don't ship it |
| **Infinite exploration** — "investigate X" with no scope | Scope it narrowly, or delegate to a subagent |

---

## The Short Version

1. Interview → `SPEC.md`
2. Fresh session, plan mode → `PLAN.md`
3. `/init` → a short `CLAUDE.md`
4. Skeleton + a test command that works
5. One slice at a time, `/clear` between
6. Independent review of the diff
7. Commit and PR

**If you remember only two things:** plan before you code, and never ship something
you can't verify.

---

## Reference

- Best practices: https://code.claude.com/docs/en/best-practices
- Overview: https://code.claude.com/docs/en/overview
- CLAUDE.md / memory: https://code.claude.com/docs/en/memory
- Skills, hooks, MCP, subagents: https://code.claude.com/docs/en/features-overview