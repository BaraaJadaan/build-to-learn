# Build to Learn

A [Claude Skill](https://www.anthropic.com/news/skills) that builds a real technical
project with you, gradually, the way it would actually get built in production — and, at
the same time, produces two living documents so you can defend every decision in it. Works
just as well on a project you already finished, if it needs the interview-readiness layer
added after the fact. By the end you have a working project and the ability to explain it
in an interview, not just the memory of having followed a tutorial.

The name is the point: the actual way to learn a stack or a domain is to build something
real in it and be made to explain your choices — not to watch someone else do it at 1.5x
speed. This is that, structured so it doesn't overwhelm you and doesn't stop at "it runs."

It exists because most self-taught portfolio projects fail the same way: the project runs,
but when someone asks *"why did you use X instead of Y?"* there's no real answer. A course
video can teach you syntax. It can't put you in front of an interviewer and make you defend
a decision under a follow-up question. This tries to close that gap directly, instead of
adding another few hours of watch-time.

## What it produces

1. **A real project**, built step by step, using genuinely current tools and architecture —
   researched at build time, not whatever a training set memorized as "best practice" two
   years ago.
2. **`concepts.html`** — a growing primer of every new idea the build introduces, explained
   plainly, logged the moment it shows up.
3. **`interview_qa.html`** — a growing set of interview questions about the project —
   technical "why this, not that" questions, trick questions that teach a real mechanism
   instead of just being clever, and whole-project questions — each with a complete model
   answer. Filterable by category, with a "quiz me at random" mode and session self-rating
   ("got it" / "review again") for active-recall practice.

See [`examples/demo-rate-limiter`](examples/demo-rate-limiter) for a small worked example —
open `concepts.html` and `interview_qa.html` in a browser to see the actual output, not just
a description of it.

## How it works

1. **Kickoff.** Asks what you're building it for, and whether you already have an idea, a
   fragment of one (just a sector, a title, or a one-line description), a blank page you
   want researched options for, or an already-built project that just needs the
   interview-readiness layer added retroactively.
2. **Idea, checked, not rubber-stamped.** If you have an idea, it gets researched before
   it's agreed to — and if a genuinely stronger angle turns up, the skill says so plainly
   and lets you choose, rather than quietly building the weaker version.
3. **A visible phase roadmap**, so the project is never an unbounded stream of work.
4. **The build loop.** For every real technical decision, at least two real alternatives get
   compared on the dimensions that actually matter, and the reasoning is written down — this
   is the part that turns "I used X" into "I can defend using X." The first time real
   implementation comes up, it asks whether to build it for you while narrating, or let you
   write it with it reviewing — a working project matters, but so does hands-on practice
   with the stack, and those aren't always the same thing. Each step is sized to introduce
   at most one or two new ideas at a time, and both docs grow alongside the code, not after
   it.
5. **Wrap-up.** A pass that fills any gaps, adds the questions that only make sense once the
   whole project exists, and offers a live mock interview pulled straight from
   `interview_qa.html` — one that remembers which questions were shaky last time and starts
   there.

The full logic lives in [`SKILL.md`](SKILL.md) — it's written to be read, not just executed.

## Install

Skills work across Claude.ai, Claude Cowork, and Claude Code. Exact steps change as the
product does, so the current, authoritative instructions are always here:
**[support.claude.com → How to create custom skills](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills)**.
As of this writing:

- **Claude.ai / Cowork:** zip this folder (or just `SKILL.md` + `assets/` + `references/`
  if you want a leaner upload) and upload it under **Settings → Capabilities** (enable *Code
  execution and file creation* first) **→ Customize → Skills → Upload**.
- **Claude Code:** copy this folder into `.claude/skills/build-to-learn/` in your
  project (or your global skills directory).

## Use it

Once it's enabled, just ask for what you want:

> *"I want to build something for my portfolio to target backend/infra roles — I don't have
> an idea yet."*

> *"Help me build \[your idea] the proper production way, and get me ready to be
> interviewed about it."*

> *"I already built this last year — can you help me get interview-ready about it without
> rebuilding it?"*

> *"I've got `PROGRESS.md` and the two docs from a project I started earlier — let's pick
> it back up."*

The skill asks a few questions up front, then works with you in small, checked-in chunks
rather than one huge dump of work.

## What's inside

```
build-to-learn/
├── SKILL.md                          the skill itself — read this first
├── assets/
│   ├── concepts_template.html        starting point for concepts.html (already designed)
│   ├── qa_template.html              starting point for interview_qa.html (already designed)
│   └── progress_template.md          starting point for PROGRESS.md
├── references/
│   ├── writing_great_questions.md    how to write a trick question that teaches, not a gotcha
│   └── worked_example.md             one real build step, end to end, for calibration
└── examples/
    └── demo-rate-limiter/            a small filled-in example — open the HTML files
```

The two HTML templates are self-contained (no external fonts, no CDN dependencies, no
`localStorage`), so they keep working as plain files long after the conversation that built
them is gone — open them on a plane, print them, keep them next to your resume.

## Design notes

A few decisions that aren't obvious from skimming `SKILL.md`, in case you're extending it:

- **The HTML/CSS/JS is copied, never regenerated.** The filters, the reveal-to-answer
  mechanic, and the design are already built into the two templates; the skill's job is to
  fill them in, not redesign them. This is what makes the output consistent across
  different sessions and different models, instead of a fresh, different-quality result
  every time.
- **Every project gets its own folder**, named from the project's slug, so using this
  repeatedly over time doesn't produce colliding, same-named files.
- **The skill never overwrites an existing project's files.** Data safety is treated as a
  higher priority than any pacing or research guidance elsewhere in the skill.
- **Interview readiness degrades gracefully.** The wrap-up pass is designed to run even on a
  project that stops early — partial readiness for what actually got built is treated as
  strictly better than none, since most real attempts don't finish exactly as planned.
- **Who writes the code is an explicit question, not a default.** Having the skill build
  while explaining gets to a working project faster; writing it yourself with review builds
  more hands-on muscle memory. The skill asks instead of silently picking one, because
  "working project" and "actually learned it" aren't quite the same goal.
- **Retroactive documentation reuses the same build loop**, just pointed at reading existing
  code instead of writing new code. It doesn't duplicate the whole workflow for that case —
  see the end of section 3 in `SKILL.md`.

## License

[MIT](LICENSE) — use it, fork it, change it, ship it.

If you build something with it, or improve on it, opening a PR or an issue is welcome.
