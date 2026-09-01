# Calibrating depth

This is the detail behind the familiarity check in section 2 (kickoff) and the tools/
practices follow-up at the end of section 3. Read this before running either.

## Three axes, not one level

People aren't one skill level — your own project history proves it. Someone can know a
domain deeply but not its current tooling (an ML engineer who's never used the specific
embedding model everyone's on now). Someone can know neither. Someone can know everything
and just want the documentation layer without being re-taught their own field. Collapsing
this into a single "beginner/intermediate/expert" dial loses the exact distinction that
determines what actually needs explaining. Track three separate axes instead:

- **Domain/industry** — do they understand the field conceptually (what RAG is *for*, what
  problem a rate limiter solves, why backends need to handle concurrency)?
- **Tools/practices** — do they know the *current*, specific tools and practices this
  particular project will use, as opposed to the field in general?
- **Coding fundamentals** — are they comfortable with code itself: syntax, control flow,
  reading a stack trace, the basic vocabulary of programming?

A person can be high on one axis and near zero on another. Treat them independently in both
directions — don't infer tools/practices familiarity from domain familiarity, and don't
infer coding fundamentals from either of the other two.

## Determining each level

Self-rating is the starting point, not the final answer. Ask for it directly (a short
question per axis works well with a tappable-question tool if one's available), but don't
stop there — two more signals matter as much or more:

- **How they talk.** Vocabulary, the specificity of the questions they ask, whether they
  correct *you* on something — these are better signal than a self-applied label, since
  people are often miscalibrated about their own level in both directions.
- **A quick diagnostic question, if the self-rating leaves you unsure.** Not a quiz, just
  one concrete question pitched at the boundary of what "intermediate" would mean here. How
  they answer (or that they can't) tells you more than another self-report would.

Keep asking, lightly, until you're actually confident — don't lock in a level from a single
ambiguous signal and run with it for the whole project. But don't turn this into a formal
assessment either: one self-rated question per axis is usually enough, escalate to a
diagnostic follow-up only for the axis where the signal is genuinely unclear, and stop as
soon as you have a level you'd bet on. This should feel like a couple of natural questions,
not an intake exam.

**Recalibrate during the build, not just at the start.** If someone rated themselves
"intermediate" on coding fundamentals but then asks what a for-loop does, that's real signal
— adjust immediately and quietly, no need to make it a moment. The reverse matters just as
much: if someone rated themselves low but is clearly tracking everything effortlessly and
finishing your sentences, dial back the scaffolding rather than continuing to explain things
they've already demonstrated they know. The stated level is a starting estimate, not a
contract.

## What each axis actually controls

**Domain/industry** — low means concepts.html needs field-level primers, not just
project-level ones (what retrieval-augmented generation is, before getting to why *this*
project's retrieval works the way it does). High means skip straight to the project-specific
reasoning; a domain expert doesn't need the field explained to them, only what's genuinely
new even to an expert.

**Tools/practices** — low means explain what a tool or technique *is* and what problem it
generically solves before getting into why this one beat the alternatives. High means skip
straight to the comparative reasoning — they already know what the tools do, they need the
"why this one, here" part, not the "what is this" part.

**Coding fundamentals** — this is the one with the widest blast radius:
- Low fundamentals should push the build-loop's "who writes the code" question (section 5,
  step 3) hard toward "the AI builds it, narrating in detail" — a genuine beginner writing
  raw code themselves turns "gradual" into "overwhelming" fast. Higher fundamentals keep
  that question fully open and live.
- Low fundamentals means concepts.html entries go to syntax level when relevant — what a
  decorator is, what an HTTP status code means, what `async` actually does — the kind of
  thing that would normally be considered too basic to log. At this level it isn't. High
  fundamentals means none of that; a term like "decorator" isn't a concept entry at all.
- Low fundamentals means walkthrough.html and in-chat narration should avoid unexplained
  jargon even in passing. High fundamentals means normal technical shorthand throughout.

**What doesn't change:** the standard `interview_qa.html` is held to. A beginner's path
has more scaffolding, more concepts.html entries, more explained fundamentals along the way
— but the destination is still genuine interview-readiness, not a lowered version of it.
Someone who started knowing nothing about backend should, by the end, be able to answer a
real interview question about it — the depth setting changes how much help they got getting
there, not how good the final answer needs to be. `walkthrough.html` doesn't scale with
depth at all — it's written for an interviewer, not for the person's own learning curve, so
it stays exactly as short and rough regardless of who's building it.

## Worked example: the same concept, two depths

Low coding-fundamentals + low tools/practices, on "why we used a connection pool":

> **Concept: Connection pooling.** Every time your code talks to a database, opening that
> connection has real cost — it's a bit like dialing a phone call versus already being on
> the line. A connection pool keeps a set of connections open and ready, so requests reuse
> them instead of paying that setup cost every time. **Why it mattered here:** without one,
> each API request would open and close its own database connection, which is slow and
> which databases have hard limits on — too many at once and the database itself starts
> refusing connections. **vs. opening a new connection per request:** simpler code, but it
> doesn't survive real traffic; this is the kind of thing that works fine in a demo and
> falls over in production, which is exactly the gap this skill exists to close.

High on both axes, same underlying decision:

> **Concept: Connection pool sizing.** Sized the pool to roughly `(core_count * 2) + 1`
> per the usual starting heuristic for I/O-bound workloads, then adjusted down after
> load-testing showed connection contention wasn't actually the bottleneck at this traffic
> level. **vs. an unbounded pool:** simpler to reason about, but risks the database's own
> connection ceiling under a traffic spike; bounded-with-headroom was the safer default.

Same decision, same file, completely different entry — the low-depth version teaches what a
connection pool *is*; the high-depth version assumes that and gets straight to the sizing
tradeoff, which is the part that's actually interesting to someone who already knows the
basics.

## Verification is opt-out, not opt-in, and the choice gets recorded

Default is on: the periodic retrieval-practice check-ins (section 5, step 7) and the wrap-up
mock interview (section 7) happen unless the person explicitly says they don't want them.
Mention once, plainly, when the option first comes up, what skipping trades away — that the
files stop being something you've verified you can actually reproduce out loud, and start
being just text — then respect whatever they decide without repeating the caveat. If they
opt out, record it factually in `PROGRESS.md` (what was skipped, when) — not to police the
choice, just so it's an honest record either they or a future session can see, the same way
the friction log is.
