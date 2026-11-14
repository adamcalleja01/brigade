# brigade

> You wouldn't send the head chef to chop carrots. So why is your top AI model renaming variables?

Brigade turns your coding agent into a kitchen. Your main chat becomes the expediter: it splits each request into jobs and hands each job to a cook, a subagent locked to the right model and effort level. You never switch models by hand.

Works in **Claude Code** (as a plugin) and **Codex** (as a skill plus custom agents).

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

## How it works

1. You ask for something the normal way.
2. The main chat splits it into jobs.
3. Each job goes to its cook, which runs on the model and effort level the menu gives it.
4. A fresh reviewer that didn't write the code checks it, then the main chat runs the tests and reports back.

Example from a real run: *"add a POST /login with hashed passwords, write tests, and get it reviewed"* sent the login code to `auth-payments` (Opus), the tests to `test-writer` (Sonnet) and the review to a fresh `reviewer` (Sonnet). The main chat did the one-line rename itself.

The rules the main chat follows:

- **Never go cheap on auth, payments, security or money.**
- **A different cook tastes the dish.** Code is always reviewed by an agent that didn't write it.
- **Plan first** when a change spans more than a few files.
- **Escalate, never downgrade.** A stuck cook hands the job one step up.

## Install

### Claude Code

```bash
claude plugin marketplace add adamcalleja01/brigade
claude plugin install brigade@brigade
```

Start a new session. The menu loads automatically at the start of every session.

Tip: run your main chat on Sonnet (`/model sonnet`). The main chat only coordinates, and Opus and Fable get called just for the jobs that need them. You can also call a cook yourself: *"use the planner to plan this"*.

The Claude Code cooks use the short model names (`fable`, `opus`, `sonnet`, `haiku`), so they always run the newest version.

### Codex

```bash
git clone https://github.com/adamcalleja01/brigade /tmp/brigade
mkdir -p ~/.codex/agents ~/.agents/skills
cp /tmp/brigade/codex/agents/*.toml ~/.codex/agents/
cp -r /tmp/brigade/skills/brigade ~/.agents/skills/
```

Codex only spawns subagents when asked, so to make routing automatic, add this line to `~/.codex/AGENTS.md`:

```
For coding requests, use the brigade skill.
```

Codex cooks name exact models. When new ones ship, update `model` in `codex/agents/*.toml`.

## Cost

Routing doesn't always spend less. It spends less on grunt work (searches, renames, boilerplate) and more where mistakes are expensive (auth, payments, hard bugs, planning). That's the point.

## Credit

Idea from Daniyar Kazi's "AI kitchen" reel ([@donimas](https://www.instagram.com/reel/DfhdW3cMmOD/)).

## License

MIT
