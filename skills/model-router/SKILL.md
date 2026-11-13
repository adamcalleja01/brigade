---
name: model-router
description: Use when a coding request involves planning, building, UI, auth or payments, searching code, renames, boilerplate, tests, a hard bug, a security audit or a code review, or when choosing which Claude or Codex model and effort level should handle a job.
---

# Model Router

You wouldn't send the head chef to chop carrots. Pick the cook by the order, not by the brand.

You are the expediter. Take the order, split it into jobs, send each job to the right cook (a subagent pinned to the right model and effort), and check the plate before it goes out. Don't cook the jobs yourself on whatever model this session happens to run.

## The menu

| Job | Cook | Claude Code | Codex |
|---|---|---|---|
| Plan a feature | `planner` | fable · high | gpt-6-astra · high |
| Build the feature | `builder` | sonnet · medium | gpt-6.1-sol · medium |
| Frontend / UI | `frontend` | opus · medium | gpt-6.1-sol · xhigh |
| Auth & payments | `auth-payments` | opus · xhigh | gpt-6.1-sol · xhigh |
| Search the codebase | `prep-cook` | haiku · low | gpt-6-luna · low |
| Renames & boilerplate | `prep-cook` | haiku · low | gpt-6-luna · low |
| Write tests | `test-writer` | sonnet · low | gpt-6.1-sol · low |
| Nasty bug | `bug-hunter` | opus · max | gpt-6.1-sol · max |
| Security audit | `security-auditor` | fable · xhigh | gpt-6-astra · xhigh |
| Code review | `reviewer` | sonnet · high | gpt-6.1-sol · high |

Each cook's model and effort are pinned in its agent file. Don't override them when you dispatch.

## Routing an order

1. Split the request into jobs from the menu. A job that isn't on the menu goes to the closest row.
2. Dispatch each job to its cook:
   - **Claude Code:** the Agent tool with `subagent_type: "model-router:<cook>"` (just `"<cook>"` if the agents were copied into `~/.claude/agents/`).
   - **Codex:** spawn the agent named `<cook>`.
3. Write a brief that stands on its own, because the cook can't see this conversation: the goal, the files and paths, the constraints, what done looks like, and what to send back.
4. Run independent jobs in parallel. Run dependent jobs in order: `planner` → `builder` / `frontend` / `auth-payments` → `test-writer` → `reviewer`.
5. After any code change, send it to `reviewer`, a fresh agent that didn't build it. Send its findings back to the cook who built that part.
6. Check the result yourself (run the tests, read the diff) before telling the user it's done.

Do a job yourself, without a cook, only when it's a quick answer or a single obvious edit of a few lines. There the dispatch costs more than the job.

## Rules

- **Never go cheap on auth, payments, security or money.** If any part of a job touches login, passwords, sessions, tokens, permissions, payments or billing, that part goes to `auth-payments`, even when it looks simple. Split mixed jobs: the login piece to `auth-payments`, the rest to `builder`.
- **A different cook tastes the dish.** `reviewer` never reviews code it wrote, and you don't review in its place.
- **Plan first when the change spans more than a few files.** Planning mistakes cost the most to fix later, so `planner` runs before anyone cooks.
- **Stuck cook, next cook up.** If a cook comes back stuck twice, send the job one step up: `prep-cook` → `builder` → `bug-hunter`.

## Common mistakes

| Mistake | Do instead |
|---|---|
| Doing every job yourself on the session's model | Dispatch each job to its cook |
| Sending a login or payment change to `builder` because it looks simple | `auth-payments`, always |
| Reviewing the code yourself, or with the cook that wrote it | A fresh `reviewer` |
| A brief that says "fix what we discussed" | Spell out the goal, the files and what done looks like |
| Downgrading a cook's model to save money on a hard job | Keep the menu; escalate, never downgrade |

## Keeping the menu current

The Claude Code cooks use the short names (fable, opus, sonnet, haiku), which always point to the newest version. The Codex cooks name exact models: when new ones ship, update `model` in `codex/agents/*.toml` and the table above.
