# Claude Code context & memory — a worked example

> **Which Claude is this for?** This setup works best in **Claude Code**, where
> the global layer (`~/.claude/CLAUDE.md`, `~/.claude/memory/`, `~/.claude/docs/`)
> and per-project memory live as files on disk you can edit, version, and share.
> You can still create **project-scoped memory** in **Claude.ai (chat) Projects**
> and in **Cowork** — Cowork even stores it as a `memory.md` in the project folder
> you can open and edit locally. What those don't give you is the cross-project
> **global** memory layer in this file format: their memory and custom
> instructions are scoped per-project and managed in-app, not as a global file
> tree you manage on your machine. So the project-memory ideas below transfer;
> the global-file management is Claude Code only.

This repo is a **reference example** for how we use Claude Code's context files:
global instructions, global memory, and per-project memory. It's a snapshot of a
real setup (the `angular-material-DS` design-system project), copied here so the
team can see the shape of the files, where they live, and the prompts that
created, edited, and promoted them.

> **Why a designer would bother:** the payoff isn't note-taking — it's that memory
> + a `DESIGN.md` let Claude **design within your system** instead of improvising
> one-offs. Memory carries the recurring gotchas, brand constants, and confirmed
> pattern decisions across sessions; `DESIGN.md` is the grammar (which surface /
> component / composition for which task). Together they mean Claude reaches for
> *your* button, *your* spacing, *your* bulk-action pattern — not a generic guess.
> See `project-angular-material-ds/memory/design-md-goal.md` for that north star.

> Usernames in paths below are written as `<you>`. On a real machine that's your
> actual home folder (e.g. `/Users/jdoe/...`). Substitute your own throughout.

---

## 1. Where these files live

Claude Code reads context from two scopes — **global** (every project, every
session) and **project** (only when you're working in that repo). Nothing here
lives in the project repo itself; it all lives under your home `~/.claude/`.

```
~/.claude/
├── CLAUDE.md                         ← GLOBAL instructions (every session, every project)
├── docs/
│   └── figma-desktop-bridge-notes.md ← GLOBAL reference doc, @-referenced from CLAUDE.md
├── memory/                           ← GLOBAL memory (project-agnostic learnings)
│   ├── MEMORY.md                     ← the index (one line per memory; loaded each session)
│   ├── user-figma-proficiency.md
│   ├── memory-save-discipline.md
│   ├── figma_code_connect.md
│   └── … (one file per fact)
└── projects/
    └── <slug-of-project-path>/       ← e.g. -Users-you-projects-angular-material-DS
        └── memory/                   ← PROJECT memory (only loaded in that repo)
            ├── MEMORY.md             ← the project index
            ├── design-md-goal.md
            ├── fidelity-sweep-sop.md
            └── … (one file per fact)
```

This repo mirrors that layout so you can read the files directly:

```
context-memory/
├── global/
│   ├── CLAUDE.md                     ← copy of ~/.claude/CLAUDE.md
│   ├── docs/                         ← copy of ~/.claude/docs/
│   └── memory/                       ← copy of ~/.claude/memory/ (full, 9 facts + index)
└── project-angular-material-ds/
    ├── DESIGN.md                     ← project doc: design/composition grammar (lives in the project repo)
    ├── PROGRESS.md                   ← project doc: status + decision log (lives in the project repo)
    └── memory/                       ← curated subset (10 of ~30 files) of the project memory
```

Two kinds of project file are shown, and they live in **different places**:

| File | Lives in | Loaded by Claude how |
|---|---|---|
| Project **memory** (`memory/*.md`) | `~/.claude/projects/<slug>/memory/` | Auto-recalled; index `MEMORY.md` injected each session |
| Project **docs** (`DESIGN.md`, `PROGRESS.md`) | the **project git repo** root | Read on demand when Claude (or you) points at them |

`DESIGN.md` and `PROGRESS.md` are committed to the project repo and reviewed like
code. Memory files are personal/agent state under `~/.claude` and are **not** in
the project repo.

---

## 2. How to see them

```bash
# Global
cat ~/.claude/CLAUDE.md
ls  ~/.claude/memory/
cat ~/.claude/memory/MEMORY.md          # the index — start here

# Project (find your project's slug first)
ls  ~/.claude/projects/
ls  ~/.claude/projects/<slug>/memory/
cat ~/.claude/projects/<slug>/memory/MEMORY.md
```

The slug is your project's absolute path with `/` replaced by `-`
(e.g. `/Users/you/projects/angular-material-DS` →
`-Users-you-projects-angular-material-DS`).

Inside a session you can also just ask: *"show me my project memory index"* or
*"what's in global memory?"* and Claude reads the files for you.

> **macOS Finder:** `.claude` is a dotfile, so Finder hides it by default. Toggle
> hidden files/folders on and off with **⌘ + Shift + .** (Cmd-Shift-period).

---

## 3. `CLAUDE.md` vs memory — don't mix them up

This is the #1 point of confusion, so be clear on it:

| | `CLAUDE.md` | memory (`MEMORY.md` + `*.md`) |
|---|---|---|
| Who writes it | **You** | **Claude** (you trigger it) |
| What it is | Your standing **instructions** | Claude's **notebook** of learned facts |
| When loaded | **Always**, every session | **Recalled by relevance** (index always loaded) |
| Example | "Don't pad responses with affirmation" | "The Figma token lives in `.env.local`" |

Rule of thumb: a *rule you want Claude to always follow* → `CLAUDE.md`. A *fact
Claude discovered that it'll need later* → memory. (A repeated correction can
start as a `feedback` memory and graduate into `CLAUDE.md` once it's a hard rule.)

---

## 4. The file format

Every memory is **one file = one fact**, with YAML frontmatter and a body.
The `MEMORY.md` index holds **one line per memory** and no memory content — it's
what gets loaded into context each session, so it stays compact.

```markdown
---
name: figma-access-token
description: <one-line summary — used to decide relevance during recall>
metadata:
  type: user | feedback | project | reference
---

<the fact. For feedback/project, follow with **Why:** and **How to apply:** lines.>
<Link related memories with [[their-name]].>
```

Four `type`s:

- **user** — who you are (role, expertise, preferences). e.g. `user-figma-proficiency` ("user is fluent in Figma; skip UI walkthroughs")
- **feedback** — how you want Claude to work (corrections, confirmed approaches); include the *why*. e.g. `per-component-standing-checks` (the a11y + interaction-state gate to run on every component), `verify-at-source-not-doc-trace` (check rules at the actual Figma file, not your own docs)
- **project** — ongoing work/goals/constraints not derivable from the code or git. e.g. `pilot-to-demo-reframe`, `token-tier-darkmode-prereq` (token pipeline must be refactored before dark mode)
- **reference** — pointers to external resources (URLs, dashboards, tickets). e.g. `figma-access-token` (where the token lives, not the value)

`[[wikilinks]]` cross-reference other memories. A link to a file that doesn't
exist yet is **valid** — it marks something worth writing later. (You'll see some
dangling links in the curated project copy here; that's why.)

---

## 5. How to add to / edit / promote them

You don't hand-edit these much — you tell Claude in plain language and it writes
the file + updates the index. Below are **real prompts** from the
angular-material-DS sessions that produced the files in this repo.

### What to save — and what NOT to

Save what's **non-obvious and reusable**: a gotcha that bit you, a confirmed
decision, a preference, a where-things-live pointer.

**Good things to save as a designer** (real examples from this repo):

- A recurring Figma gotcha → `label-render-lock-root-cause.md` (color paints
  render-lock the label; the plugin write is a no-op — fix manually).
- A confirmed DS pattern decision → `bulk-action-canonical.md` (inline header
  toolbar is canonical, *not* a floating bar) so Claude stops re-litigating it.
- A working method → `fidelity-sweep-sop.md` (the repeatable per-component
  check: Storybook ↔ Figma ↔ props), so the next sweep doesn't reinvent it.
- Brand constants that aren't obvious from a glance — primary `#2574db` at tone
  40, Open Sans 400/600/700 — so Claude reaches for them by default.

**Don't** save (it bloats the index and the index has a size limit that's loaded
every session — over-stuffing it degrades every future session):

- Code structure / file layout — Claude can read the repo.
- Past bug fixes already in the diff or git history.
- Anything already in `CLAUDE.md` or the project README.
- Facts that only matter to the conversation you're in right now.

**For designers specifically:**

- **Figma node IDs / file keys** — they're session-specific and go stale across
  conversations. Save the *component name* or *where to find it*, not the ID.
- **Token values / component specs / variant names** that already live in the
  Figma file or DS docs — Claude can read the file. Save the *non-obvious why*
  instead (e.g. "color paints render-lock the label → broke it last time").
- **The raw Figma access token (or any secret)** — save the *fact that a token
  exists and where* (`.env.local`), never the value. The `figma-access-token`
  memory in this repo is the right pattern to copy.
- **One-off visual choices** — "made this button blue here" isn't reusable.
  "brand primary = #2574db, used at tone 40" is.

If you're tempted to save one of those, ask instead: *what was non-obvious about
it?* — and save that.

### Project memory or global? (the decision)

- **Project** — true only for *this* repo: its stack, its decisions, its quirks.
- **Global** (`~/.claude/memory/`) — true across *all* your work: who you are,
  how you like Claude to behave, tool/API gotchas that recur everywhere.

When a project fact turns out to be universal, **promote** it (see below).

### Create a memory

Trigger it when something non-obvious comes up that you'll want next session:

> *"this is not the first time you forget we have the Figma access token in
> .env.local — shouldn't it be saved to memory somewhere? (the fact you can use
> it, not the token itself)"*
> → produced `figma-access-token.md` (type: reference)

> *"Color paints is an important thing to save to memory, since that's what broke
> the labels before."*
> → produced `label-render-lock-root-cause.md` (type: project)

> *"ok, first save the steps on the badge as steps for a complete fidelity sweep
> per component (whatever we can generalize — checking storybook, checking figma,
> matching properties, saving updates to figma on our memory files for when we do
> this in the main DS)."*
> → produced `fidelity-sweep-sop.md` (a reusable SOP, type: feedback)

### Edit / correct a memory

Memories are point-in-time; when reality drifts, correct them:

> *"I'm looking at the project's memory.md and I see references to Material 2, but
> we're actually using M3 — should we update that?"*
> → corrected the stale stack label in `MEMORY.md`.

> *"do some more cleanup of the memory since we're at it."*
> → removed noise, tightened the index.

### Promote a project memory → global

When a fact stops being project-specific, move it up so every project gets it:

> *"I'm looking at the `user-figma-proficiency.md` file, and I think it's a great
> candidate to move to general memory with all the other general figma things we
> have. Same with `memory-save-discipline`, since it's something we'll do on
> every [project]."*
> → moved both files from project `memory/` to `~/.claude/memory/`, updated both
> indexes, and left a one-line pointer in the project index noting the move.

> *"just memory — can we apply it to every claude instance and not just this
> project?"*
> → the canonical "promote to global" ask.

The project `MEMORY.md` in this repo still carries the breadcrumb from one such
move (search for *"moved to GLOBAL memory"*) — that's the recommended trail to
leave so future sessions know where a fact went.

### Session / phase save discipline (a `feedback` memory itself)

One memory governs how all the others get written — worth copying to your own
setup:

> *"any time I say 'save to memory and update progress' whenever we're ending a
> session or a phase, the job is to save to the corresponding topic files and
> keep the main memory file as compact as possible."*
> → produced `memory-save-discipline.md` (now global).

So end-of-session, you just say: *"save to memory and update progress"* and Claude
writes the topic files + trims the index, instead of dumping everything into one
file.

---

## 6. Setting this up on your own machine

1. Drop a `CLAUDE.md` at `~/.claude/CLAUDE.md` with your global working-style
   rules (see `global/CLAUDE.md` here for an example).
2. Create `~/.claude/memory/MEMORY.md` (the index). Add memories by asking Claude
   to save facts as they come up — it creates the per-fact files and appends the
   index line.
3. Project memory is created automatically the first time Claude saves a
   project-scoped fact while you're working in that repo.
4. Keep `DESIGN.md` / `PROGRESS.md` (or your project's equivalents) in the
   **project repo**, not in `~/.claude` — they're team artifacts, reviewed like
   code.

> **Designers — the short version:** you mostly don't hand-create any of these.
> Work in Figma with Claude as usual, and when something non-obvious comes up just
> say *"save that to memory."* Claude writes the file and updates the index. The
> global memory in `global/memory/` here is already Figma/DS-heavy (API limits,
> Code Connect SOP, the always-screenshot-verify rule, token architecture) — copy
> that folder into your own `~/.claude/memory/` as a starter kit and you inherit
> the gotchas without re-learning them.

> **Note:** memory files can contain whatever the session learned — double-check
> for secrets, internal URLs, or anything you don't want shared before publishing
> a copy like this one. (The personal `RTK.md` tool config and ~20 deep project
> files were intentionally left out of this example for that reason.)

---

## Read more

- [How Claude remembers your project — Claude Code Docs](https://code.claude.com/docs/en/memory) — the file-based memory + `CLAUDE.md` vs `MEMORY.md` distinction
- [Memory tool — Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) — the underlying memory tool, if you're building with the API
- [Get started with Claude Cowork — Help Center](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork) — Cowork Projects: per-project memory, instructions, scheduled tasks
- [The three markdown files that run Claude Cowork](https://medium.com/@shard/the-three-markdown-files-that-run-claude-cowork-4e8d2af36ced) — how Cowork's `memory.md` works in the project folder
