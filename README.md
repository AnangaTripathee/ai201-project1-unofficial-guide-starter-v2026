# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:** paragraph-based, not a fixed character count
**Overlap:** none (paragraphs don't need it — see below)

I picked campus_life because it's short, single-topic posts, and Milestone 1
confirmed that early: 88 documents averaging 317 characters, all under the
800-character default, so the starter's fallback chunker never split
anything (88 documents in, 88 chunks out).

That's not nothing — it told me the real question wasn't "what character
count?" but "does any single post actually contain more than one topic?"
Rereading my documents, most are one paragraph, one fact (dining dollars,
housing lottery, pass/fail deadlines). A few — like course_biol_160.txt —
bundle a title with several distinct paragraphs (course format, workload,
exam advice) that don't need to travel together.

So I replaced the chunker with paragraph-based splitting: split on blank-line
breaks, but re-attach the document's title/heading line to every resulting
chunk (many posts open with a one-line label like "On the housing lottery"
or "BIOL 160 Cell Biology," and a paragraph split naively loses that context).
Fragments under 150 characters get merged into the previous paragraph rather
than kept as their own chunk, since a title alone, or a one-line aside, isn't
answerable on its own.

Result: 88 documents became 92 chunks. Only 4 documents actually split
(health_center.txt, housing_old_brewhouse.txt, study_library_hours.txt,
transit_shuttle.txt) — the other 84 are single-paragraph posts that stayed
exactly as they were, which is the right outcome, not a missed opportunity.
course_biol_160.txt is a useful edge case: it has three paragraphs, but two
of them fell under the 150-character merge threshold, so it recombined back
into one chunk — and reading it, that's correct: the whole thing reads as
one complete answer about the course.

## Sample Chunks

**Chunk 1** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.


**Chunk 2** — source: `health_center.txt#0` — produced by: `chunker.py::split_documents`

The health centre

Walk-in hours are 8am to 11am; everything after that is by appointment and appointments run about a week out. If something is urgent, go at 8am and wait rather than booking.


**Chunk 3** — source: `health_center.txt#1` — produced by: `chunker.py::split_documents`

The health centre

Counselling is separate, in the same building, and has its own intake process with a shorter wait than people expect — usually three or four days for a first session.


**Chunk 4** — source: `admin_housing_lottery.txt#0` — produced by: `chunker.py::split_documents`

On the housing lottery

The housing lottery is not random in the way most people assume. Rising sophomores get a number drawn at random, but juniors and seniors are ordered by accumulated credit hours first, and only tie-break randomly. That means a senior who took summer courses reliably beats a senior who didn't. Numbers come out the second week of March and selection runs over four evenings.


**Chunk 5** — source: `admin_pass_fail_option.txt#0` — produced by: `chunker.py::split_documents`

On the pass/fail option

Any course outside your major can be taken pass/fail, and — the part nobody mentions — you can declare it as late as week eight, after you've seen your midterm. A pass needs a C- or better. Two per year, maximum eight across a degree.

## Sample Answer

**My relevance cutoff:** 0.6

My five in-scope questions scored 0.248–0.457. My five out-of-scope questions
scored 0.825–0.934. That's a gap of 0.37 with nothing in between, so the
cutoff wasn't a close call — 0.6 sits almost exactly in the middle of it,
and I kept the starter's default rather than moving it artificially.

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery actually random? | Yes | 0.248 |
| Does dining dollars roll over between semesters? | Yes | 0.256 |
| Can I still get pass/fail after seeing my midterm grade? | Yes | 0.402 |
| Does financial aid still apply if I study abroad? | Yes | 0.376 |
| If I withdraw from a class, does it affect my GPA? | Yes | 0.457 |
| What is the capital of Mongolia? | No | 0.825 |
| How do I change the oil in a diesel engine? | No | 0.934 |
| Who won the 1994 World Cup? | No | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.844 |
| How do I write a for loop in Rust? | No | 0.896 |

**Question:** Is the housing lottery actually random?

**Answer:**

The housing lottery is not entirely random in the way most people assume. While rising sophomores receive a number drawn at random, juniors and seniors are ordered first by accumulated credit hours, with random selection used only as a tie-breaker (admin_housing_lottery.txt).


Retrieval returned 5 chunks (best distance 0.248), but only one —
admin_housing_lottery.txt — was actually about the lottery; the rest were
loosely related (registration, parking, two unrelated housing reviews). The
grounding instruction correctly kept the model from using any of them, and
it named its actual source inline.

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**

**2.**

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
