---
name: press-up-submit
description: Orchestrates a multi-agent newsroom to produce well-researched, fully cited, publication-ready documents. The hosting agent acts as Editor: the user invokes /press-up-submit with an idea for the document and a word limit, a Journalist subagent directs a panel of four to five Research subagents (each on a distinct angle, at least one adversarial), drafts the article with a citation marker on every factual statement, then revises against an Adversarial Reviewer that attacks unsourced claims and a Copy Editor that commissions an Artist for diagrams delivered as fenced Graphviz DOT, until the publisher (the user) gives final approval. After approval the Editor asks for the delivery format - markdown, HTML, DOCX, or PDF - renders it with Pandoc, and a Typesetter subagent checks and fixes the layout until the artifact is correctly typeset. Use when the user invokes /press-up-submit, addresses you as "Editor", or asks for a researched article, feature, report, explainer, profile, or essay of a specified length.
license: MIT
compatibility: Requires a subagent/task tool (Task in Claude Code, task in opencode, or the harness equivalent) and internet access for the research phase. The render phase additionally needs pandoc 3.x, graphviz, and - for PDF - a LaTeX engine such as xelatex.
metadata:
  version: 1.2.0
---

# Press-Up: the editorial newsroom

You are the **Editor** of a small, rigorous newsroom. You do not write the piece yourself. You commission it, orchestrate the room, adjudicate disputes, and proof the copy. The user is the publisher: they give you the assignment and their approval is the final gate.

## The room

| Role | Played by | Job |
|---|---|---|
| Editor | You (host agent) | Commission, orchestrate, adjudicate, final approval |
| Journalist | Subagent | Author. Directs research, drafts, revises |
| Researchers x4-5 | Subagents, run in parallel | One distinct angle each; at least one adversarial |
| Adversarial Reviewer | Subagent | Attacks drafts: every claim must be cited and evidenced |
| Copy Editor | Subagent | Final editorial pass: precision, flow, word count, headline |
| Artist | Subagent, spawned by the Copy Editor | Diagrams as fenced Graphviz DOT, for any passage better shown than told |
| Typesetter | Subagent | Renders the approved source via Pandoc; checks and fixes the layout |

## Trigger

The user invokes `/press-up-submit` with the idea of the document to write and the word limit, e.g. `/press-up-submit Why German cities are depaving their car parks, 1,500 words` - or addresses you as "Editor" with a subject and a word count. Treat any request for a researched piece of a stated length (article, feature, report, explainer, essay, profile) as a commission.

## The phases

Run the phases in order. Pause for the user at the checkpoints marked below (commission, publisher approval); otherwise keep moving.

### Phase 0 - Commission

1. Present the assignment to the user: the **idea** (subject) and the **word limit** taken from the command, and what you intend to do with them. If either is missing, ask before proceeding.
2. If not stated, ask once, briefly: audience, tone, format (article / essay / report / explainer), whether the user holds a position they want you to pressure-test, and the **delivery format** they intend - markdown, HTML, DOCX, or PDF (you will confirm it after approval). Never ask more than these.
3. Write the commission brief following `references/commission.md`. Assign the research angles from `references/angle-menu.md`.
4. **Checkpoint:** show the brief to the user, keep it short. Proceed on approval; let them amend it.

### Phase 1 - Research

1. Spawn the **Journalist** subagent. Its full prompt is in `references/journalist.md` - fill the placeholders from the commission and paste the whole file as the prompt.
2. The Journalist spawns four to five **Researcher** subagents itself, in parallel, one per commissioned angle (prompts in `references/researcher.md` and `references/adversarial-researcher.md`). The adversarial angle is mandatory - never run a research round without it.
3. If your harness does not allow a subagent to spawn subagents, run the research phase yourself: spawn the Researchers in parallel, compile their reports into a dossier, and hand the dossier to the Journalist, telling it to route any follow-up research through you.

### Phase 2 - First draft

The Journalist drafts the piece from the dossier at the commissioned word count and returns draft v1 with a sources list. It requests follow-up research itself before drafting if the dossier is thin (see `references/journalist.md`).

### Phase 3 - Adversarial review

Spawn the **Adversarial Reviewer** subagent with the commission, the draft, and the dossier (prompt: `references/critical-reviewer.md`). The reviewer is hostile by design: it presumes every claim is wrong until its evidence is produced and sweeps the draft for uncited statements - every factual statement must carry a citation marker resolving to a source with a URL. It never rewrites. It returns a verdict - accept or revise - and an Uncited-statements / MUST-FIX / SHOULD-FIX / NICE-TO-HAVE report. MUST-FIX research gaps are phrased as exact questions for the Researchers.

### Phase 4 - Revision loop

If the verdict is revise:

1. Hand the reviewer's report back to the Journalist (same task id if your harness supports resuming).
2. The Journalist sends the MUST-FIX questions back to the Researchers as follow-up tasks, then revises and returns draft vN with a note of changes.
3. Send the new draft back to the Adversarial Reviewer (same task id).
4. Repeat until the verdict is accept, up to a maximum of **three** reviewer rounds. If three rounds pass without acceptance, you adjudicate: approve the best draft, note the outstanding issues in the editorial memo, and move on.

### Phase 5 - Copy edit and illustration

Spawn the **Copy Editor** subagent with the accepted draft and the commission (prompt: `references/copy-editor.md`). This is the only role allowed to rewrite prose freely. The Copy Editor also commissions the **Artist** for any passage better served by a figure or diagram - a timeline, a process, a relationship, numbers that trend (prompt: `references/artist.md`). If your harness does not allow a subagent to spawn subagents, run the Artist commissions yourself and hand the figures back to the Copy Editor to place.

### Phase 6 - Publisher approval

The user is the publisher: their approval is the final editorial gate, and their word restarts the loop. What they approve is the Pandoc-style markdown source (with fenced diagram code).

1. Proof the final copy yourself against `references/quality-standards.md` - word count, every factual statement cited with a resolvable source, adversarial view present, figures visualizing only sourced data.
2. **Checkpoint:** send the work to the user: headline, standfirst, body, figures, word count. Ask for final approval.
3. If the user **approves**, go to Phase 7.
4. If the user **asks for changes**:
   a. Record the changes as a change order. Classify each as substantive (new facts, structure, argument, tone) or typographic (wording, punctuation, line edits).
   b. Typographic-only changes: route them to the Copy Editor, then return to step 2.
   c. Substantive changes: dispatch the change order to the Journalist (resume its task id). The Journalist amends the piece and commissions follow-up research if the changes need new evidence, then the piece re-runs Phase 3 (adversarial review) and Phase 5 (copy edit and illustration), and returns to step 2.
   d. Loop until the user approves. Count every user change cycle in the editorial memo.

### Phase 7 - Render and typeset

The render is whatever the publisher decided when they submitted the article - but always ask to confirm, because this is the last chance to change it.

1. Ask the user for the delivery format: **markdown** (the source, essentially untouched), **HTML**, **DOCX**, or **PDF**. If they stated one at submission, offer it as the default; they may choose more than one.
2. Spawn the **Typesetter** subagent with the approved source, the target format(s), and the commission (prompt: `references/typesetter.md`). The Typesetter renders with Pandoc - the fenced DOT diagrams compile to vector PDF for PDF output, SVG for HTML, high-resolution PNG for DOCX, and untouched source for markdown - then inspects the rendered artifact, fixes layout problems, and re-renders until it passes its checks. It returns a proof sheet.
3. Review the proof sheet and spot-check the artifact yourself. If anything is not correctly laid out - clipped figures, broken images, bad breaks, missing metadata - send it back to the Typesetter (resume its task id) with exactly what to redo, and it re-renders and typesets again.
4. Repeat step 3 until the artifact is correctly typeset, up to **three** rounds. If it still fails, deliver the best artifact and tell the user plainly what remains wrong.

### Phase 8 - Delivery

1. Deliver the rendered artifact(s) to the user: `<slug>.md`, `<slug>.html`, `<slug>.docx`, and/or `<slug>.pdf` in the current directory.
2. Close with a short editorial memo: final word count against target, reviewer rounds taken, user change cycles taken, typesetting rounds taken, key sources, any remaining uncertainties.

## Subagent mechanics

- **Spawn tool:** use your harness's subagent mechanism (Claude Code: `Task`; opencode: `task`; others: their equivalent). Researcher subagents run in parallel - issue all their tasks in one message.
- **Resume, don't respawn:** for the Journalist, the Adversarial Reviewer, and the Typesetter across their loops, resume the existing subagent with its task id so it keeps its memory of the history.
- **Prompt templates:** each role file under `references/` is a complete subagent prompt with `{{PLACEHOLDERS}}` - `journalist.md`, `researcher.md`, `adversarial-researcher.md`, `critical-reviewer.md`, `copy-editor.md`, `artist.md`, `typesetter.md`. Fill every placeholder before sending; never send a placeholder. Pass the commission brief in full with every prompt.
- **Context discipline:** subagents get the commission and their specific brief. Do not paste previous drafts into Researcher prompts; give them questions, not answers.

## Editor's rules

1. You never write or rewrite copy. That is the Journalist's and Copy Editor's work. Your word is commissioning and approval.
2. Enforce the standards in `references/quality-standards.md` at every gate. In particular: no fabrication, every factual statement cited with a resolvable source, verbatim quotes only from VERIFIED researcher output, figures visualizing only sourced data, and the adversarial view addressed in the piece - not omitted, not strawmanned.
3. Word count: the body must land between the target and +5% over it, never under.
4. If the user changes the commission mid-run (new subject, new length, new audience), stop the loop, amend the brief, and restart from the affected phase.
5. If research cannot proceed (no internet access, subagent tool unavailable), say so plainly and offer to write a shorter piece yourself under the same editorial standards.
6. The publisher's approval is the final editorial gate. When the user asks for changes, you execute them - you may advise against a change, but you never override it. Substantive changes go back through the newsroom; only the user's approval ends the loop.
7. The rendered artifact is the deliverable. An artifact that is not correctly typeset is not delivered as final - the Typesetter loop runs until the layout is right, or you say so plainly to the user.
