# guided-practice

**A Claude skill for learning by doing.** You do the work with your own hands. Claude explains each step before you take it, checks what you actually did, and tells you honestly whether it's right before moving on.

It works for any technical task, including code, the terminal, Git, cloud and infrastructure, databases, configuration and data tooling. It adapts to your level in each topic, from "first time opening a terminal" to "senior engineer picking up a new tool".

---

## Contents
- [Why this exists](#why-this-exists)
- [What a session looks like](#what-a-session-looks-like)
- [How it works](#how-it-works)
  - [The step loop](#the-step-loop)
  - [How each step is presented](#how-each-step-is-presented)
  - [Depth adapts to your level](#depth-adapts-to-your-level)
  - [Verification, not eyeballing](#verification-not-eyeballing)
  - [The verdict](#the-verdict)
  - [When you ask Claude to do it](#when-you-ask-claude-to-do-it)
- [Your existing instructions always win](#your-existing-instructions-always-win)
- [Safety in real environments](#safety-in-real-environments)
- [Installation](#installation)
- [Usage](#usage)
- [What's in this repository](#whats-in-this-repository)
- [How it was tested](#how-it-was-tested)
- [Limitations](#limitations)
- [FAQ](#faq)

---

## Why this exists

By default, an AI assistant does the task for you. That's great when you want the result. It's bad when you want to **learn**, or when you need to be able to **explain and maintain** the work afterwards.

The usual alternative is a tutorial, but tutorials can't see your screen. They don't notice the file you forgot to save, the typo in an import path, or the step you skipped.

`guided-practice` sits in between. It's like having a patient senior colleague next to you who:

- tells you **what comes next and why**, before you do it;
- explains **every part of every command** you might not know, so you never have to stop and ask "what does `-p` mean?";
- **waits** while you do it;
- **checks your actual work**: reads the file, runs the code with real data, builds the project;
- gives you a **clear verdict** with the reason, and only then moves on.

A session went well when you did each part yourself, understood why it worked, and could do it again without help.

---

## What a session looks like

A short, realistic excerpt from a beginner session:

> **You:** I want to build the login page of my React app myself. Teach me while I do it.
>
> **Claude:** *(gives a 6-stage map of the task, then:)*
>
> **Step 1: Show a "Login" component on the page**
>
> **What this step does:** in React, every piece of the screen is a component...
>
> ```jsx
> export default ___
> ```
> **Hint for the blank:** the file has to "hand out" something so other files can use it. What's the name of the thing you just created?
>
> **What each part means:**
> - `className`: in JSX you write `className`, not `class`, because `class` is a reserved word in JavaScript...
>
> **What you'll see:** the page might look a bit odd (dark background). That's Vite's starter CSS, not your code. We'll fix it in Step 2.
>
> Tell me when you're done and I'll check it.
>
> **You:** done
>
> **Claude:** 🟡 **Almost.** The component is right, but the import in `App.jsx` says `'./pages/login'`, and your file is `Login.jsx`. It may work on your Mac, but Linux (and the server you'll deploy to) treats file names as case-sensitive, so the build will fail there. Change it to `'./pages/Login'`.

The same skill with an experienced engineer looks completely different: no syntax lessons, bigger steps, and the content is trade-offs and production gotchas. See [Depth adapts to your level](#depth-adapts-to-your-level).

---

## How it works

### The step loop

Every step follows the same cycle:

```
  ┌─────────────────────────────────────────────────────────┐
  │ 1. Context   what we're building now, and why           │
  │ 2. Step      one step, with every part explained        │
  │ 3. Wait      you do it, then say "done" / "check it"    │
  │ 4. Verify    Claude checks the real result              │
  │ 5. Verdict   ✅ correct · 🟡 could be better · ❌ fix this │
  │ 6. Next      the next step, or a fix-up step            │
  └─────────────────────────────────────────────────────────┘
```

Before the first step, Claude works out three things:
- **the goal**: what "done" looks like;
- **your level in this topic**: by reading how you write and what already exists, or by asking one short question;
- **the environment**: which folder you're in, what's installed, what already exists.

Then it gives you a short map of the whole task, so you always know where you are.

### How each step is presented

Every step uses the same format, so you always know where to look:

```
## Step N: <the outcome of this step>

What this step does:   purpose, in plain words, and why it matters now
<the command or code>
What each part means:  every flag, symbol and keyword you may not know
What you'll see:       prompts, expected output, errors that are expected and fine
Tell me when you're done and I'll check it.
```

Two details that make a big difference:
- **Expected errors are announced in advance.** A red error you were warned about is part of the lesson. The same error unannounced makes you think you broke something.
- **Trivial checks don't waste a round-trip.** "What version do you have?" is folded into the first real step instead of being a step of its own.

### Depth adapts to your level

Level is measured **per topic, not per person**. A senior backend engineer can be a beginner in Terraform, and the skill treats them that way: Terraform syntax explained, programming basics skipped, with analogies to the languages they already know.

| | Beginner in this topic | Intermediate | Advanced |
|---|---|---|---|
| **Step size** | one outcome, at most 2–3 short actions | a coherent chunk (a function, a config block) | a decision or a whole stage |
| **What gets explained** | every part you may not know | the non-obvious parts, and the *why* | design choices, trade-offs, failure modes, production gotchas |
| **Scaffolding** | code with blanks (`___`) and an "expected results" table | a skeleton or signature; you fill in the body | a spec and acceptance criteria; the design is yours |
| **Review depth** | is it correct, and why | correct, idiomatic, edge cases | senior code review: concurrency, performance, security, operability |

**Depth is not volume.** Going deeper with an expert means sharper content, not more words. Experts get tight tables and bullets that usually fit on one screen.

**You can steer it at any time**, and it keeps your setting:

| Say | Effect |
|---|---|
| "faster" · "skip the basics" · "just the gist" | moves up a level |
| "slower" · "what does X mean?" | moves down a level |
| "do the whole thing in one pass, I'll review" | bigger steps |

### Verification, not eyeballing

Claude never says "looks right!" without evidence. It uses the strongest check that is safe:

1. **Read the actual file.** It catches unsaved files (a 0-byte file right after "done"), wrong folders and wrong names.
2. **Static checks**: linter, type checker, JSON/YAML parse. Claude knows their blind spots: a linter won't notice an import path that points nowhere.
3. **Run it with your real data**, against the expected-results table you saw before you started.
4. **Check it where it's used**: build the project, load the page, run `plan`.

Then it tells you what it checked ("ran it with your data: 6 of 6 cases pass"), so you also learn what good checking looks like.

### The verdict

| | Meaning |
|---|---|
| ✅ **Correct** | what you did well and *why* it matters (specific praise teaches; "great job" doesn't) |
| 🟡 **Works, could be better** | optional improvements (readability, naming, idiom), clearly marked as optional |
| ❌ **Needs fixing** | what's wrong, why it breaks (now, later, or for someone else), and the fix or a hint |

Blockers are always separated from nice-to-haves. Corrections come with the underlying rule, so one fix teaches you to avoid the next ten mistakes of the same kind.

### When you ask Claude to do it

"Fix it for me" or "write this one" is fine: you choose where to spend your effort. Claude then:
- does **exactly** what you asked, without quietly refactoring anything else;
- **keeps your work** and corrects it, instead of replacing it;
- shows **what changed and why**, so you still learn from it;
- verifies its own change the same way it would verify yours;
- goes back to guided mode for the next step.

---

## Your existing instructions always win

This skill only decides **how Claude teaches**. It never overrides the rules you already work under. The order of priority is:

1. Safety rules and platform policies.
2. **Your working instructions**: your organization's and manager's rules, `CLAUDE.md` and other project instruction files, repository conventions, and any channel or document your own setup already treats as instructions.
3. What you ask for in the conversation.
4. `guided-practice`, only where nothing above says otherwise.

If a team rule conflicts with the guided method (for example "Claude writes the code, humans review", "never run commands", "every change goes through a PR"), Claude follows the team rule, adapts the teaching around it, and says so in one sentence.

**A security note on shared documents.** The skill deliberately does **not** tell Claude to obey instructions found in files or channels. If it did, any document that landed in a shared folder could take control of the session. Instead, documents carry exactly the authority **your own setup** already gives them. If your work Claude already follows your team's instruction channel, that keeps working, and this skill bends around it. The skill also never writes to shared channels, team documents or config files to record your progress unless you ask.

It also follows **the project's** style and conventions over its own preferences, because you'll be the one living in that codebase.

---

## Safety in real environments

In shared or production-like environments, "verify for real" must never mean "run anything". Claude prefers:

| Area | Safe checks it uses |
|---|---|
| Git | `status`, `log`, `diff`, `show`, `fetch`. Never push, force-push or rewrite history on its own |
| Databases | read-only queries with `LIMIT`; `EXPLAIN` (not `EXPLAIN ANALYZE` on production); writes inside `BEGIN … ROLLBACK` first |
| Infrastructure | `terraform plan` / `validate` / `fmt -check`, `kubectl diff`, `--dry-run=server`, `helm template` / `lint` |
| Everything else | dry runs, staging or local copies, read-only commands |

Anything destructive or that touches shared state (deploys, migrations, `apply`, pushes, sending messages, deleting resources) is **explained and left for you to run**, if your team's process allows it.

When Claude can't verify something safely, it says so and gives you the exact command or click-path to check it yourself, along with what a good and a bad result look like.

---

## Installation

### Claude Code (macOS / Linux)

Clone the repository straight into your personal skills folder:

```bash
git clone https://github.com/nuneslucass23/guided-practice.git ~/.claude/skills/guided-practice
```

- `git clone <url> <folder>` downloads the repository into that folder.
- `~/.claude/skills/` is where Claude Code looks for your personal skills. Skills there work in **every** project.

Then restart Claude Code. The skill appears as `/guided-practice`.

> **Private repository:** cloning needs access to your GitHub account on that machine. If `git clone` asks for credentials or fails with "repository not found", log in first with `gh auth login` (GitHub CLI), or use an SSH URL if you have SSH keys set up.

**Alternative: keep the repository elsewhere and link it.** Use this if you want the repository in your projects folder:

```bash
git clone https://github.com/nuneslucass23/guided-practice.git ~/Documents/guided-practice
```

```bash
mkdir -p ~/.claude/skills && ln -s ~/Documents/guided-practice ~/.claude/skills/guided-practice
```

- `ln -s` creates a link instead of a copy, so changes in the repository take effect immediately.

### Update

```bash
git -C ~/.claude/skills/guided-practice pull
```

- `git -C <folder>` runs the Git command inside that folder without having to `cd` into it first.

(If you used the link setup, run `git pull` in your repository folder instead.)

### Uninstall

```bash
rm -rf ~/.claude/skills/guided-practice
```

If you used the link setup, this removes only the link, and your repository stays where it is.

### Claude.ai / Claude Desktop

If your plan or organization allows custom skills: zip the repository folder (with `SKILL.md` at the top level of the zip), then upload it in **Settings → Capabilities → Skills**.

---

## Usage

**Start it explicitly:**

```
/guided-practice set up SSH keys for GitHub on this machine
```

**Or just say it.** The skill activates on phrases like:
- "teach me while I do it"
- "I want to do it myself"
- "walk me through it and I'll type it"
- "explain step by step and check my work"
- "don't write it for me, guide me"
- "coach me through this"

It deliberately **does not** activate on normal requests like "write a script that…" or "fix this bug". When you just want the result, Claude just does it.

**Example prompts at different levels:**

| Level | Prompt |
|---|---|
| Beginner | "I've only done a few React tutorials. I want to build my app's login page myself. Teach me while I do it." |
| New tool, experienced dev | "I'm solid in Python but I've never touched Terraform. Guide me through a module for an S3 bucket; I'll type everything. Team rule: no `terraform apply` from laptops." |
| Advanced | "Walk me through a GitHub Actions pipeline with a Node test matrix, caching and a main-only deploy job. I've done CI for years on Jenkins. I'll write the YAML, you review." |

**Tips:**
- Say "done" or "check it" after each step. Claude reads your files itself, so you usually don't need to paste them.
- Ask "how much is left?" at any time for a progress summary.
- Ask concept questions in the middle of a step ("where does this function come from?"). Claude answers, then brings you back to exactly where you were.
- Turn on autosave in your editor. Unsaved files are the most common "it didn't work".

---

## What's in this repository

```
guided-practice/
├── SKILL.md                 ← the skill itself: the method Claude follows
├── references/
│   ├── step-examples.md     ← example steps and reviews at each depth, plus common mistakes
│   └── verification.md      ← how to verify each kind of work safely
├── evals/
│   └── evals.json           ← the test prompts used to evaluate the skill
└── README.md                ← this file
```

How Claude loads it:
- The `description` in `SKILL.md` is always visible to Claude, and it decides **when** the skill turns on.
- The body of `SKILL.md` loads only when the skill is active.
- The `references/` files are read only when needed: `step-examples.md` when Claude is unsure how much to explain, `verification.md` when the safe way to check something isn't obvious.

---

## How it was tested

The skill was built with Anthropic's skill-creator process. Each of the three test prompts in `evals/evals.json` was run **with** and **without** the skill, and the answers were compared side by side over two iterations:

| Test | Without the skill | With the skill |
|---|---|---|
| Beginner, React | a first step with 6 sub-tasks and no syntax explained | one outcome per step, every new piece of syntax explained, expected oddities announced in advance |
| Experienced dev, new tool (Terraform) | good analogies, but 4 sub-steps at once | Python analogies, the team's "no apply from laptops" rule honored explicitly, the team repo's conventions adopted |
| Advanced, CI (GitHub Actions) | the whole design dumped at once | only the Jenkins → GHA gotchas that bite, one stage with acceptance criteria, about half the words of iteration 1 |

Changes made between iterations:
- **"Depth is not volume"** for experts. The advanced answer dropped from about 1,200 to about 600 words.
- **Trivial checks folded into the first real step** instead of costing a round-trip.
- **Beginner steps capped** at one outcome with 2–3 short actions.

---

## Limitations

- **It's a teaching style, not knowledge.** The quality of the explanations still depends on the model knowing the topic.
- **Verification is only as good as the access Claude has.** Without permission to run commands or read files, it falls back to telling you how to check.
- **It's slower than having Claude do it.** That's the point, but don't use it when you only need the result.
- **Activation is based on phrasing.** If it doesn't turn on when you expect, invoke `/guided-practice` explicitly.

---

## FAQ

**Will it refuse to write code for me?**
No. Ask ("fix it for me", "write this part") and it does exactly that, shows what changed, and returns to guided mode.

**Will it slow down an expert with basics?**
It shouldn't. Level is calibrated per topic, and "skip the basics" moves it up immediately.

**Can it break something at work?**
It's designed not to. It never runs destructive or shared-state actions on its own, prefers dry runs and read-only checks, and defers to your team's rules.

**Why English?**
Skills are usually written in English. It still works when you write to Claude in another language.

**Can I change how it behaves?**
Yes. Edit `SKILL.md`. If you used the link setup, the change takes effect the next time Claude Code starts.
