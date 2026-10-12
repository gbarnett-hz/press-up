# Press-Up

A skills.sh installable skill that turns your agent into an editorial newsroom: an **Editor** (your agent) commissions a piece, a **Journalist** subagent directs a panel of four to five **Research** subagents (one always adversarial), drafts the article to your specified word count with a citation marker on every factual statement, then revises it against an **Adversarial Reviewer** that attacks unsourced claims, and finishes with a **Copy Editor** that commissions an **Artist** for figures and diagrams where a passage is better shown than told.

## Install

```bash
# From GitHub (once published)
npx skills add <owner>/press-up

# Or directly from this repository, by URL
npx skills add https://github.com/<owner>/press-up

# Or from a local path while you develop it
npx skills add /path/to/press-up

# Install just this skill from a multi-skill repo
npx skills add <owner>/press-up --skill press-up
```

The skill lives at `skills/press-up/SKILL.md`, which the skills.sh CLI discovers automatically.

## Use

Talk to your agent as the Editor:

> Editor, I want a 1,500-word article on why German cities are depaving their car parks. I don't have a position yet — pressure-test whatever you find.

> Editor, this is the subject: the collapse of the European battery start-up Northvolt. I want 2,000 words.

The Editor will confirm the commission (subject, word count, audience, tone), then run the newsroom. Every piece goes through: commission → parallel research (evidence, history, voices, adversarial, stakes) → first draft with cited statements → adversarial review (uncited claims and weak evidence fail the draft) → revision loops (the Journalist sends follow-up questions back to the Researchers when the reviewer demands more) → copy edit and illustration (figures for passages better shown than told) → **publisher approval**. The piece comes to you for final sign-off; if you ask for changes, the Editor dispatches them to the Journalist and the loop runs again until you approve.

The finished article is delivered as a Markdown file with a numbered sources list, plus an editorial memo covering word count, rounds taken, change cycles, and remaining uncertainties.

## Structure

```
skills/press-up/
├── SKILL.md                     # Editor's operating manual
└── references/
    ├── commission.md            # How to write the commission brief
    ├── angle-menu.md            # The five research angles
    ├── quality-standards.md     # Non-negotiable standards
    ├── journalist.md            # Subagent prompt: the Journalist
    ├── researcher.md            # Subagent prompt: a Researcher
    ├── adversarial-researcher.md# Subagent prompt: the adversarial Researcher
    ├── critical-reviewer.md     # Subagent prompt: the Adversarial Reviewer
    ├── copy-editor.md           # Subagent prompt: the Copy Editor
    └── artist.md                # Subagent prompt: the Artist
```

## Requirements

- An agent harness with a subagent/task tool (Claude Code, opencode, Codex, etc.)
- Internet access for the research phase

## License

MIT — see [LICENSE](LICENSE).
