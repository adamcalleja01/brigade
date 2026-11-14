---
name: brigade
description: Use when a coding request involves planning, building, UI, auth or payments, searching code, renames, boilerplate, tests, a hard bug, a security audit or a code review, or when choosing which Claude or Codex model and effort level should handle a job.
---

# Brigade

You wouldn't send a general to dig trenches. Assign the unit by the mission, not by rank.

You are the commander. Take the request, break it into missions, deploy each mission to the right unit (a subagent locked to the right model and effort), and check the result before you report back. Don't fight every battle yourself on whatever model this session happens to run.

## The roster

| Mission | Unit | Claude Code | Codex |
|---|---|---|---|
| Plan a feature | `strategist` | fable · high | gpt-6-astra · high |
| Build the feature | `engineer` | sonnet · medium | gpt-6.1-sol · medium |
| Frontend / UI | `frontline` | opus · medium | gpt-6.1-sol · xhigh |
| Auth & payments | `sentry` | opus · xhigh | gpt-6.1-sol · xhigh |
| Search the codebase | `recon` | haiku · low | gpt-6-luna · low |
| Renames & boilerplate | `grunt` | haiku · low | gpt-6-luna · low |
| Write tests | `drill-sergeant` | sonnet · low | gpt-6.1-sol · low |
| Nasty bug | `sniper` | opus · max | gpt-6.1-sol · max |
| Security audit | `red-team` | fable · xhigh | gpt-6-astra · xhigh |
| Code review | `inspector` | sonnet · high | gpt-6.1-sol · high |

Each unit's model and effort are locked in its agent file. Don't override them when you deploy it.

## Running a request

1. Break the request into missions from the roster. A mission that isn't on the roster goes to the closest row.
2. Deploy each mission to its unit:
   - **Claude Code:** the Agent tool with `subagent_type: "brigade:<unit>"` (just `"<unit>"` if the agents were copied into `~/.claude/agents/`).
   - **Codex:** spawn the agent named `<unit>`.
3. Write orders that stand on their own, because the unit can't see this conversation: the objective, the files and paths, the constraints, what done looks like, and what to report back.
4. Run independent missions in parallel. Run dependent missions in order: `strategist` → `engineer` / `frontline` / `sentry` → `drill-sergeant` → `inspector`.
5. After any code change, send it to `inspector`, a fresh agent that didn't write it. Send its findings back to the unit that wrote that part.
6. Verify the result yourself (run the tests, read the diff) before telling the user it's done.

Handle a mission yourself, without deploying, only when it's a quick answer or a single obvious edit of a few lines. There the deployment costs more than the mission.

## Rules of engagement

- **Never go cheap on auth, payments, security or money.** If any part of a mission touches login, passwords, sessions, tokens, permissions, payments or billing, that part goes to `sentry`, even when it looks simple. Split mixed missions: the login piece to `sentry`, the rest to `engineer`.
- **The unit that built it never inspects it.** `inspector` never reviews code it wrote, and you don't review in its place.
- **Plan first when the change spans more than a few files.** Planning mistakes cost the most to fix later, so `strategist` runs before anyone builds.
- **Pinned down? Call in heavier support.** If a unit comes back stuck twice, send the mission one step up: `grunt` → `engineer` → `sniper`.

## Common mistakes

| Mistake | Do instead |
|---|---|
| Fighting every mission yourself on the session's model | Deploy each mission to its unit |
| Sending a login or payment change to `engineer` because it looks simple | `sentry`, always |
| Reviewing the code yourself, or with the unit that wrote it | A fresh `inspector` |
| Orders that say "fix what we discussed" | Spell out the objective, the files and what done looks like |
| Downgrading a unit's model to save money on a hard mission | Keep the roster; escalate, never downgrade |

## Keeping the roster current

The Claude Code units use the short names (fable, opus, sonnet, haiku), which always point to the newest version. The Codex units name exact models: when new ones ship, update `model` in `codex/agents/*.toml` and the table above.
