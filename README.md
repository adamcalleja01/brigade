# Brigade

> You wouldn't send a general to dig trenches. So why is your top AI model renaming variables?

Brigade turns your coding agent into a commander with a unit for every mission. Your main chat breaks each request into missions and deploys each one to a unit: a subagent locked to the right model and effort level. You never switch models by hand.

Works in **Claude Code** (as a plugin) and **Codex** (as a skill plus custom agents).

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

## How it works

1. You ask for something the normal way.
2. The main chat, acting as commander, breaks it into missions.
3. Each mission goes to its unit, which runs on the model and effort level the roster gives it.
4. A fresh `inspector` that didn't write the code reviews it, then the commander runs the tests and reports back.

For example, *"add a POST /login with hashed passwords, write tests, and get it reviewed"* sends the login code to `sentry` (Opus), the tests to `drill-sergeant` (Sonnet) and the review to a fresh `inspector` (Sonnet).

## Rules of engagement

- **Never go cheap on auth, payments, security or money.**
- **The unit that built it never inspects it.**
- **Plan first** when a change spans more than a few files.
- **Escalate, never downgrade.** A stuck unit hands the mission one step up.

## Install

### Claude Code

```bash
claude plugin marketplace add adamcalleja01/brigade
claude plugin install brigade@brigade
```

Start a new session. The roster loads automatically at the start of every session.

Tip: run your main chat on Sonnet (`/model sonnet`). The commander only coordinates, and Opus and Fable get called just for the missions that need them. You can also deploy a unit yourself: *"send recon to find where auth is handled"*.

The Claude Code units use the short model names (`fable`, `opus`, `sonnet`, `haiku`), so they always run the newest version.

### Codex

```bash
git clone https://github.com/adamcalleja01/brigade /tmp/brigade
mkdir -p ~/.codex/agents ~/.agents/skills
cp /tmp/brigade/codex/agents/*.toml ~/.codex/agents/
cp -r /tmp/brigade/skills/brigade ~/.agents/skills/
```

Codex only spawns subagents when asked, so to make Brigade automatic, add this line to `~/.codex/AGENTS.md`:

```
For coding requests, use the brigade skill.
```

Codex units name exact models. When new ones ship, update `model` in `codex/agents/*.toml`.

## Cost

Brigade doesn't always spend less. It spends less on grunt work (searches, renames, boilerplate) and more where mistakes are expensive (auth, payments, hard bugs, planning). That's the point.

## License

MIT
