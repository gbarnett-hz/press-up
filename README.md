# Press-Up

A skills.sh installable skill that turns your agent into an editorial newsroom: an **Editor** (your agent) commissions a piece, a **Journalist** subagent directs a panel of four to five **Research** subagents (one always adversarial), drafts the article to your specified word count with a citation marker on every factual statement, then revises it against an **Adversarial Reviewer** that attacks unsourced claims, and finishes with a **Copy Editor** that commissions an **Artist** for diagrams where a passage is better shown than told. After you approve, the Editor renders the piece with **Pandoc** in your chosen format - markdown, HTML, DOCX, or PDF - and a **Typesetter** subagent checks and fixes the layout until the artifact is correctly typeset.

## Install

```bash
# From GitHub
npx skills add gbarnett-hz/press-up

# Or by full URL
npx skills add https://github.com/gbarnett-hz/press-up

# Or from a local path while you develop it
npx skills add /path/to/press-up

# Install just this skill from a multi-skill repo
npx skills add gbarnett-hz/press-up --skill press-up-submit
```

The skill lives at `skills/press-up-submit/SKILL.md`, which the skills.sh CLI discovers automatically.

## Use

Invoke the skill as a slash command with the idea of the document and the word limit:

> /press-up-submit Why German cities are depaving their car parks, 1,500 words

> /press-up-submit The collapse of the European battery start-up Northvolt, 2,000 words

The Editor will confirm the commission (subject, word count, audience, tone), then run the newsroom. Every piece goes through: commission → parallel research (evidence, history, voices, adversarial, stakes) → first draft with cited statements → adversarial review (uncited claims and weak evidence fail the draft) → revision loops (the Journalist sends follow-up questions back to the Researchers when the reviewer demands more) → copy edit and illustration (the Copy Editor rewrites away AI-slop tells, and the Artist delivers diagrams as fenced Graphviz DOT) → **publisher approval**. The piece comes to you for final sign-off; if you ask for changes, the Editor dispatches them to the Journalist and the loop runs again until you approve.

After approval, the Editor asks which format you want: **markdown** (the source, essentially untouched), **HTML**, **DOCX**, or **PDF**. The Typesetter renders it with Pandoc - the diagram source compiles to vector PDF for PDF output, SVG for HTML, high-resolution PNG for DOCX, and stays fenced in markdown - then inspects the rendered artifact and fixes any layout problems, re-rendering until it is correctly typeset.

The finished document is delivered in your chosen format with a numbered sources list, plus an editorial memo covering word count, rounds taken, change cycles, and remaining uncertainties.

### Example

You type:

```
/press-up-submit Why German cities are depaving their car parks, 1,500 words
```

The Editor confirms the assignment:

> **Commission**
> - Slug: german-depaving
> - Subject: Why German cities are removing asphalt from car parks and how the movement is spreading
> - Format: article · Target: 1,500 words (never under, up to +5%)
> - Audience: general reader, no prior knowledge
> - Tone: measured, curious
> - Angles: evidence & data, history & context, voices & players, adversarial view, stakes & futures
>
> Approve, or tell me what to change?

The newsroom then runs: five researchers work in parallel (one builds the strongest case *against* the depaving boom), the Journalist drafts with a citation marker on every factual statement, the Adversarial Reviewer attacks uncited claims, and the Copy Editor commissions a timeline figure of the depaving wave. You get the piece for approval:

> **Draft for your approval — 1,512 words**
> ## The Great Unpaving
> …The city of Bottrop broke its first asphalt in 2019 [3]… critics say the gains are mostly symbolic — one study found only 12% of depaved lots measurably cool their streets [7]…
>
> Approve, or ask for changes?

You approve, and the Editor asks how you want it rendered:

> **Approved. How should I deliver it?** markdown (source, untouched) · HTML · DOCX · PDF

You choose PDF. The Typesetter renders `german-depaving.md` through Pandoc with the diagram filter (vector PDF figures), rasterizes the pages and inspects them, fixes a clipped timeline, and re-renders. The Editor delivers and closes with the memo:

> **Editorial memo** — `german-depaving.pdf` · 1,512/1,500 words · 2 reviewer rounds · 1 change cycle · 2 typesetting rounds · 23 sources · remaining uncertainty: none of the German studies isolate depaving from wider greening programs.

## Structure

```
skills/press-up-submit/
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
    ├── artist.md                # Subagent prompt: the Artist (fenced DOT diagrams)
    └── typesetter.md            # Subagent prompt: the Typesetter (Pandoc render + layout QA)
```

## Requirements

- An agent harness with a subagent/task tool (Claude Code, opencode, Codex, etc.)
- Internet access for the research phase
- For the render phase: pandoc 3.x and Graphviz, plus a LaTeX engine (e.g. xelatex) for PDF output. DOCX output works out of the box; the Pandoc diagram filter (`pandoc-ext/diagram`) is fetched automatically when missing.

## License

MIT — see [LICENSE](LICENSE).
