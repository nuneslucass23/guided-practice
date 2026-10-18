# claude-skills

Personal skills for Claude.

## Skills

| Skill | What it does |
|---|---|
| [`guided-practice`](guided-practice/SKILL.md) | Learn-by-doing mode for any technical task. You do the work; Claude explains each step and every command first, verifies what you did with real evidence, and gives a clear verdict before moving on. Depth adapts to your level in each topic. Never overrides your existing work instructions. |

## Install (Claude Code, macOS/Linux)

Clone the repository once:

```bash
git clone https://github.com/nuneslucass23/claude-skills.git ~/Documents/claude-skills
```

Link the skill into Claude Code's personal skills folder:

```bash
mkdir -p ~/.claude/skills && ln -s ~/Documents/claude-skills/guided-practice ~/.claude/skills/guided-practice
```

- `ln -s` creates a link instead of a copy, so a `git pull` in the repository updates the installed skill too.

Restart Claude Code. The skill shows up as `/guided-practice` and also activates on phrases like "teach me while I do it" or "I want to do it myself".

## Update

```bash
cd ~/Documents/claude-skills && git pull
```

## Use it in Claude.ai or Claude Desktop

Zip the `guided-practice` folder and upload it in **Settings → Capabilities → Skills**, if your plan or organization allows custom skills.
