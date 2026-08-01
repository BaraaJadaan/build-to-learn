# Writing questions worth asking

This is the detail behind step 5 of the build loop and section 7's wrap-up pass in
`SKILL.md`. Read this when you're about to add an entry to `interview_qa.html` and want the
worked examples, not just the one-line rule.

## The shape of a strong model answer

Almost every good answer, regardless of category, has the same skeleton:

1. **The direct answer, one sentence.** An interviewer should be able to get the point even
   if they stop listening after this sentence.
2. **The concrete reasoning, two to four sentences.** The actual tradeoff — numbers,
   mechanisms, constraints — not "it's more efficient" or "it's industry standard."
3. **The honest limitation.** The situation where this choice would've been wrong, or where
   it's already straining. This is what separates a real answer from a sales pitch, and it's
   usually what a good interviewer probes for next anyway — better to get there first.

A trick question adds one more piece before step 1: a named misconception (see below). A
general/reflective question often replaces step 2 with a short narrative instead of a
tradeoff, but still wants a limitation or lesson at the end, not just a happy summary.

Weak answer: *"We used a vector database because it's fast and scalable."*
Strong answer: *"We used [vector DB] because our query pattern is approximate nearest-
neighbor search over embeddings that update in small batches, not full-table scans — an
index built for that access pattern beats a general-purpose DB doing the same job with a
bolted-on extension. The tradeoff is we gave up transactional guarantees we didn't need
anyway. Where this would've been the wrong call: if the corpus were small enough to fit
comfortably in memory for brute-force search, the extra moving part wouldn't have earned its
keep."*

Notice the strong version never says a tool's name is the reason. The name is the
conclusion; the access pattern and the tradeoff are the argument.

## Technical Decision questions

These come straight out of step 2 of the build loop — anywhere you compared real
alternatives and picked one. If you did that research honestly while building, this
question mostly writes itself: the answer you already worked out *is* the model answer.

**Worked examples, across different kinds of projects:**

- *"Why a message queue instead of having services call each other directly?"* — good,
  because the answer has to name the actual failure mode direct calls create (a slow or
  down consumer takes the producer with it) and the real cost of the queue (added
  operational surface, eventual rather than immediate consistency).
- *"Why token-bucket rate limiting instead of a fixed window?"* — good, because it forces a
  specific, checkable claim: fixed windows allow a burst of up to 2x the limit right at the
  window boundary, and token bucket doesn't. A real answer can state that precisely; a
  memorized one can't.
- *"Why a cross-encoder reranker instead of just raising top-k?"* — good, because the
  honest answer is narrow and specific: reranking fixes lexical near-misses outranking true
  semantic matches, which a bigger top-k alone doesn't touch, at the cost of added
  per-query latency worth naming.
- *"Why Riverpod instead of Provider for state management?"* — good if the project actually
  hit the specific pain (e.g. compile-time safety, or testability without a widget tree)
  that motivated the switch; weak if the answer is just "it's newer."

**Weak Technical Decision questions to avoid:** anything where the honest answer is just a
name-drop of popularity or familiarity. If the real reason you'd give is "it's what
everyone uses," either dig for the actual reason everyone uses it (there usually is one) or
don't log it as a Technical Decision question — it might belong in General instead, as part
of a bigger architecture answer.

## Trick questions: instructive, not "gotcha"

The test for a good trick question: **does answering it well teach the underlying mechanism,
or does it only reward having seen this exact question before?** A pure gotcha rewards
memorization or luck. An instructive trick question rewards actually understanding how the
system behaves — someone who deeply gets the project but has never seen this exact phrasing
should still be able to reason their way to the right answer.

Structure: name a plausible-sounding claim that a lot of people would nod along to, ask if
it's true, and write the answer so it states the misconception explicitly before correcting
it — that's the teaching moment, not just the "actually, no."

**Good, instructive trick questions:**

- *"A higher embedding dimension always gives you better retrieval, right?"* — the
  misconception is that dimension is a free dial. The real answer covers diminishing
  returns past a point, and the storage/latency cost that scales with dimension whether or
  not retrieval quality keeps improving.
- *"If your service is stateless, it's automatically safe to scale horizontally — true?"* —
  the misconception ignores shared state one layer down (a cache, a rate limiter, a
  database with connection limits). The real answer names where the actual bottleneck moved
  to.
- *"More training data always improves a model's accuracy — so why didn't you just use the
  full dataset?"* — the misconception ignores label quality and distribution shift; the
  real answer should reference whatever the project's actual data-quality finding was.
- *"Caching a database query makes it faster, so more caching is always better — anything
  wrong with that?"* — the misconception ignores cache invalidation cost and staleness risk;
  a real answer names the specific staleness the project could and couldn't tolerate.

**Bad, gotcha-only questions to avoid:**

- Trivia dressed up as a trick question ("What year was [library] first released?") — no
  understanding is tested, just recall.
- Questions with a "trick" that's really just imprecise wording ("Isn't X technically a
  type of Y?") where the honest answer is a pedantic definitional footnote nobody would ask
  in a real interview.
- Anything where the correction doesn't connect back to a real decision in *this* project —
  if the misconception doesn't teach something that mattered here, it's trivia with extra
  steps.

Aim for roughly one trick question for every three or four Technical Decision questions —
not every step needs one, and forcing one where there's no real misconception at stake
produces the bad kind above.

## General / whole-project questions

These test ownership of the project as a whole, not any single decision. Good categories,
roughly in the order they tend to become answerable:

- **Why this dataset/stack/approach overall** — becomes answerable as soon as the core idea
  and first architecture decisions are locked (section 3 / early section 5). Log it early
  rather than waiting.
- **Pipeline/process walkthroughs** ("walk me through preprocessing," "walk me through a
  request end to end") — becomes answerable once that pipeline actually exists; log it
  right after the relevant phase, don't wait for the whole project.
- **Struggles and how they got resolved** — pull straight from the friction log in
  `PROGRESS.md`. This must be a real entry, not a generic one ("time management was hard")
  — if the friction log is empty, that's a sign to look harder for what actually took
  iteration, not a sign to invent something.
- **Architecture end-to-end, scaling, and "defend a criticism"** — these only make sense
  once the whole project exists, since they require seeing the full shape of it. These are
  the ones section 7's wrap-up pass adds, not the build loop.

A good "defend a criticism" question names a real, fair critique — not a strawman — and the
answer should sometimes concede the critic has a point while explaining the actual
constraint that made the tradeoff worth it anyway. An answer that never concedes anything
reads as defensive, not confident.
