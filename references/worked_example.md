# A worked example, end to end

The schema comments inside the two templates show the *shape* of an entry with placeholder
text. This shows what one real build-loop step actually produced, on a real (if small) demo
project — for calibration on depth and tone, not something to copy into a real project
verbatim. If you're about to write your first entries and want a concrete bar to match
before diffing yourself against it, read this first.

**The situation:** a demo project (a distributed rate limiter for a public API gateway —
deliberately backend/systems, not ML, since this skill isn't domain-specific). The step was
the core architecture decision: which rate-limiting algorithm, and how to share state across
multiple gateway instances. That one step produced two `concepts.html` entries and three
`interview_qa.html` cards — this is roughly the right density for a single step; it should
not take ten entries to cover one step, and it shouldn't take ten steps to produce one entry.

## What landed in concepts.html

```html
    <article class="entry" data-phase="Core Architecture">
      <p class="eyebrow">ENTRY <span class="num"></span> · Core Architecture</p>
      <h2 class="term">Token Bucket Algorithm</h2>
      <p class="explain">
        A rate-limiting algorithm where each client has a virtual "bucket" that holds
        tokens, refilled continuously up to a maximum capacity. Every request consumes one
        token; if the bucket is empty, the request is rejected. Because tokens refill
        steadily rather than all at once, it naturally allows short bursts up to the
        bucket's size while still enforcing a steady average rate over time.
      </p>
      <div class="why">
        <p class="label">Why it mattered here</p>
        <p>Real clients don't send requests in a perfectly smooth stream — they burst. A
        bucket that tolerates a bounded burst while still capping the long-run average
        matches that behavior much better than a hard per-interval cap.</p>
      </div>
      <div class="versus">
        <p class="label">vs. Fixed Window Counter</p>
        <p>A fixed window resets a counter every interval (e.g. every 60s). It's simpler,
        but it has a boundary flaw: a client can send a full quota in the last instant of
        one window and a full quota again in the first instant of the next, briefly
        doubling the effective rate. Token bucket has no such boundary.</p>
      </div>
    </article>
    <article class="entry" data-phase="Core Architecture">
      <p class="eyebrow">ENTRY <span class="num"></span> · Core Architecture</p>
      <h2 class="term">Centralized Counter State (Redis)</h2>
      <p class="explain">
        With one gateway server, a rate-limit counter can just live in that process's
        memory. With several gateway instances behind a load balancer, each one only sees a
        fraction of any given client's requests — local counters would each think the
        client is well under their limit even as the client blows past it in aggregate. A
        shared, fast, external store lets every instance check and update the same number.
      </p>
      <div class="why">
        <p class="label">Why it mattered here</p>
        <p>Needed one source of truth for "how many tokens does this client have left,"
        updated atomically so two near-simultaneous requests from the same client, hitting
        two different gateway instances, can't both succeed on the last remaining token.</p>
      </div>
      <div class="versus">
        <p class="label">vs. Local counters + a gossip protocol between instances</p>
        <p>Syncing local counters between instances avoids the extra network hop to Redis
        and the added dependency, but opens an eventual-consistency window where the true
        global rate can be briefly exceeded — and the complexity of running a gossip
        protocol isn't worth paying until a single shared store is demonstrably the
        bottleneck, which it isn't yet at this scale.</p>
      </div>
    </article>
```

Notice what each entry is *not*: it's not a textbook definition of token buckets or of Redis.
The "why it mattered here" ties directly to a property of this project (bursty clients,
multiple gateway instances), and the "versus" names the concrete alternative that was
actually considered, not a strawman. That's what makes it useful for an interview instead of
just being correct.

## What landed in interview_qa.html

The same step produced one Technical Decision question, one Trick Question, and one General
question — not because every step needs exactly one of each, but because this step happened
to raise a real "why X not Y," a real misconception worth correcting, and enough of a
pipeline to be worth walking through end to end.

```html
    <article class="qa-card" data-category="technical">
      <p class="eyebrow"><span class="tag">Technical Decision</span><span class="num"></span></p>
      <h2 class="question">Why token bucket instead of a fixed window counter?</h2>
      <button class="reveal-btn" type="button" aria-expanded="false">Show answer</button>
      <div class="answer" hidden>
        <p>Token bucket tolerates short, realistic bursts while still holding clients to a
        steady average rate, without the boundary flaw a fixed window has.</p>
        <p>With a fixed window, a client can send its full quota in the last moment of one
        window and its full quota again in the first moment of the next — briefly doubling
        the effective rate at every window boundary. Token bucket's continuous refill has no
        such boundary; the maximum possible burst is always bounded by the bucket size,
        regardless of timing.</p>
        <p>The honest tradeoff: token bucket needs a little more state per client — a token
        count and a last-refill timestamp, not just a single counter — and the bucket
        capacity has to be tuned deliberately. Set it too generously and the smoothing
        benefit disappears; it starts behaving like "always allow up to the burst size."</p>
      </div>
    </article>
    <article class="qa-card" data-category="trick">
      <p class="eyebrow"><span class="tag">Trick Question</span><span class="num"></span></p>
      <h2 class="question">Redis is single-threaded — doesn't that make it a bottleneck the moment two rate-limit checks happen at once?</h2>
      <button class="reveal-btn" type="button" aria-expanded="false">Show answer</button>
      <div class="answer" hidden>
        <p class="misconception">Common misconception</p>
        <p>Conflating "single-threaded execution" with "can't handle concurrent load."</p>
        <p>Redis's single-threaded event loop processes one command at a time, but each
        command here — an atomic check-and-decrement on a rate-limit counter — takes well
        under a millisecond, so one instance comfortably handles tens of thousands of these
        per second. What actually queues up under concurrency is network I/O and connection
        handling, which Redis multiplexes efficiently — not command execution itself.</p>
        <p>Where this stops holding: at high enough volume that a single instance's command
        throughput genuinely is the ceiling. The fix at that point is sharding rate-limit
        keys across a Redis cluster by client ID, not abandoning Redis for something else.</p>
      </div>
    </article>
    <article class="qa-card" data-category="general">
      <p class="eyebrow"><span class="tag">General</span><span class="num"></span></p>
      <h2 class="question">Walk me through what happens to a single request, from hitting the gateway to being allowed or rejected.</h2>
      <button class="reveal-btn" type="button" aria-expanded="false">Show answer</button>
      <div class="answer" hidden>
        <p>The gateway extracts the client identifier from the API key, then runs a single
        atomic Redis operation that checks the client's current token count and, if at
        least one is available, decrements it and lets the request through. If zero tokens
        remain, the gateway rejects it immediately with a 429 and a Retry-After header
        computed from the refill rate — the backend service is never touched for a
        rejected request.</p>
        <p>The atomicity matters more than it sounds: doing the check and the decrement as
        two separate Redis calls would open a race condition where two near-simultaneous
        requests from the same client could both read "1 token left" and both get allowed.
        It's implemented as a single Lua script executed atomically inside Redis, not as
        two round trips from the application.</p>
      </div>
    </article>
```

Notice the trick question specifically: it doesn't just say "wrong, actually Redis is fine"
— it names the exact misconception (conflating single-threaded execution with an inability
to handle concurrent load), corrects it with the actual mechanism, and then gives the honest
limit of that correction (it *does* eventually become true at high enough volume). That
three-part shape — misconception, correction, honest boundary — is what separates this from
a gotcha.

## What the matching PROGRESS.md looked like at this point

```markdown
# Distributed Rate Limiter (demo) — Progress

## Goal
Demo walkthrough of the Build to Learn skill — backend/systems domain, to show the mechanics work outside AI/ML projects too.

## Why this idea
Chosen to prove the skill's build loop and both companion docs work for a different kind of project than a RAG/ML pipeline — same mechanics, different domain.

## Phase roadmap
- [x] Phase 1 — Environment & tooling setup
- [x] Phase 2 — Core architecture decisions (algorithm + distributed state)
- [ ] Phase 3 — Component build: the rate-limit middleware
- [ ] Phase 4 — Evaluation: load testing the limiter under burst traffic
- [ ] Phase 5 — Hardening: Redis failover behavior
- [ ] Phase 6 — Polish & packaging
- [ ] Phase 7 — Interview-prep consolidation

## Friction log
- Phase 2 — First pass used separate GET+SET Redis calls for the token check, which created a race condition under concurrent requests; fixed by moving the check-and-decrement into a single atomic Lua script.

## Companion docs
- `concepts.html` — 2 entries
- `interview_qa.html` — 3 questions
```

Note the friction-log entry: it's specific enough to actually answer an interview question
with later (what broke, why, what fixed it) — not "debugging took a while."
