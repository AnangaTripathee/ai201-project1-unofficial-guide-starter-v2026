# The Unofficial Guide

**[Your Name]** — corpus: campus_life

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.

---

# Unit 1

## What This Does

The Unofficial Guide answers questions about campus life using campus_life —
88 short, single-topic posts covering admin logistics (deadlines, the housing
lottery, dining dollars, pass/fail rules), housing reviews, and course
overviews. It retrieves the most relevant posts for a question, checks
whether they're actually close enough to be trustworthy, and answers using
only what it found — naming the file the answer came from. Questions clearly
outside campus topics (a car repair question, a trivia question) get an
honest refusal instead of a guess.

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
Fragments under 150 characters get merged into a neighboring paragraph rather
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

(See Unit 2 below: this strategy was later revised to merge short leading
paragraphs forward as well as backward, after testing found a chunk the
original version missed.)

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

**Question:** Is the housing lottery actually random?

**Answer:**

The housing lottery is not entirely random in the way most people assume. While rising sophomores receive a number drawn at random, juniors and seniors are ordered first by accumulated credit hours, with random selection used only as a tie-breaker (admin_housing_lottery.txt).


Retrieval returned 5 chunks (best distance 0.248), but only one —
admin_housing_lottery.txt — was actually about the lottery; the rest were
loosely related (registration, parking, two unrelated housing reviews). The
grounding instruction correctly kept the model from using any of them, and
it named its actual source inline.

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

## How I Used AI

**1.** For Milestone 3, I described my corpus to Claude — mostly single-
paragraph posts, but a few (like course_biol_160.txt) bundle a title with
several distinct paragraphs. I asked it to write a chunker that fit that
pattern. What came back split on paragraph breaks, re-attached each
document's title line to every resulting chunk, and merged fragments under
150 characters into the previous paragraph. When I ran it, course_biol_160.txt
didn't actually split the way I expected — its middle paragraphs were short
enough to fall under the merge threshold and recombine into one chunk. I
checked whether that was actually wrong by rereading the merged chunk, and
decided it was the right outcome: it still read as one complete answer, so I
kept the 150-character threshold rather than lowering it just to force a
split that wasn't needed.

**2.** For Milestone 4, I gave Claude my ten retrieved distances — five
questions my corpus covers, five it clearly doesn't — and asked where the
cutoff should go. It pointed out the two groups had a wide, clean gap (0.457
at the top of my in-scope group, 0.825 at the bottom of my out-of-scope
group) and suggested keeping the starter's default of 0.6 rather than moving
it, since 0.6 already sits almost exactly in the middle. I checked this
against my own numbers rather than taking it on faith, confirmed the gap was
real, and kept 0.6.

**3.** For Unit 2 Milestone 3, since all five criteria came back MET, I asked
Claude to help me avoid the trap the instructions warned about — treating a
clean pass as "the system is excellent." I described each criterion's target
and asked it to look for where the target itself might be too easy rather
than where the system might be failing. It pointed out that my
OUT_OF_SCOPE questions had no vocabulary overlap with my corpus at all,
so the gate had never been tested near a real boundary — and separately
flagged that criterion 5's "4 of 5" target was resting on only 2 actual
sibling-pairs I'd found, not 5. I checked both by rereading my own corpus
and criteria.md, confirmed both were real gaps rather than the model
manufacturing a problem, and wrote them into What's Still Broken and What
I'd Do Differently rather than treating five MET verdicts as the end of the
story.

---

# Unit 2

## Run Log — Before

- Produced by: `run_eval.py::main` (criteria 1–3) and manual testing via
  `store.py::search` / `app.py retrieve` (criteria 4–5)
- Retrieval: `store.py::search`, chunks from `chunker.py::split_documents`
- Corpus: `campus_life` (index variant `default`)
- top-k: 5 · relevance cutoff: 0.6
- Runs per question: 3, caching off
- When: 2026-09-29

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks keep a fact and its exception together | 4 of 5 | 4/5* | 4/5* | 4/5* | MET |
| 5. Retrieval doesn't confuse near-duplicate topics | 4 of 5 | 2/2 | 2/2 | 2/2 | MET |

Criteria 3–5 are deterministic (retrieval and chunking don't change between
runs), so the same number appears in all three columns — that's correct, not
lazy, per the instructions.

\* One chunk (study_library_hours.txt#0) is under my 150-character merge
threshold but wasn't merged, because it's the first paragraph after the
heading and my merge logic only looked backward. It still contains a
complete fact-plus-exception ("2am during term" / "10pm during reading
week"), so it didn't fail the criterion at the time — flagged here as a
known limitation, then fixed below in The Improvement.

### Real output — evidence for each criterion

**Criterion 1 & 2** — Q: "Is the housing lottery actually random?" (run 1 of 3)

No, the housing lottery is not entirely random. While rising sophomores get a number drawn at random, juniors and seniors are ordered by accumulated credit hours first, with random selection used only for tie-breaks (admin_housing_lottery.txt).


Source named inline: `admin_housing_lottery.txt`. Produced by
`generate.py::answer_from_chunks`, retrieval by `store.py::search`.

**Criterion 3** — out-of-scope question, refused:

What is the capital of Mongolia? → best distance 0.825, over the 0.6 cutoff → refused


All 5 out-of-scope questions refused (5/5). Produced by `gate.py::check`.

**Criterion 4** — chunk with fact + exception intact, `chunker.py::split_documents`:

On the dining dollars

Declining balance — what everyone calls dining dollars — rolls over from the autumn semester to the spring, but not from spring to the following autumn. Whatever is left in May disappears.


**Criterion 5** — sibling-pair disambiguation, `store.py::search`:

Q: "does a work-study job count against my financial aid the same as a regular campus job?"
#1 0.1853 admin_campus_jobs_and_financial_aid.txt (correct sibling)
#2 0.5599 money_jobs.txt (related but wrong sibling — correctly ranked second)


## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer (4 of 5) | MET | All 5 questions, across all 3 runs, retrieved a chunk containing the specific fact named in `expects` (credit-hours ordering, the autumn→spring asymmetry, week eight, "doesn't affect GPA," "travels with you") — 5/5 every run, clearing the 4/5 target with room to spare. |
| 2 | Every answer names a source (5 of 5) | MET | All 15 answers (5 questions × 3 runs) named a specific `.txt` file inline in the answer text itself, not just in the "Sources retrieved" line — checked by reading every answer in the run log, not just the distances. |
| 3 | Gate stops out-of-corpus questions (4 of 5) | MET | All 5 `OUT_OF_SCOPE` questions were refused, with best distances (0.825–0.934) sitting far above the 0.6 cutoff and far from my in-scope group's ceiling (0.457) — no borderline cases to argue about. |
| 4 | Chunks keep a fact and its exception together (4 of 5) | MET | Sampled and read all 92 chunks, not just 5 — found one chunk (`study_library_hours.txt#0`) that's under my 150-character merge threshold but wasn't merged (a real gap in my merge logic, since it only checked backward), yet it still contains a complete fact-plus-exception pair with nothing cut off. So the criterion held on content even though it exposed a code limitation, which I fixed (see below). |
| 5 | Retrieval doesn't confuse near-duplicate topics (4 of 5) | MET | Tested both sibling pairs I identified — withdrawal vs. add/drop and work-study vs. non-work-study jobs — both correctly ranked the right sibling first, 2/2. Caveat: my target was worded as "4 of 5," but I only ever had 2 real sibling-pairs to test, not 5. 2/2 is real evidence the mechanism works, but it's a smaller sample than the target implies — worth tightening next time (see below). |

## Diagnoses

I missed nothing this round — all five criteria came back MET. Rather than
treat that as "the system is excellent," here's an honest look at which
targets were set safely rather than tightly, and what I'd tighten.

**Criterion 3 (gate stops out-of-corpus questions) was too easy.** My
OUT_OF_SCOPE questions (world capitals, car engines, sports trivia, drug
dosages, Rust syntax) share essentially no vocabulary with a campus admin
corpus, so the gap between in-scope (0.248–0.457) and out-of-scope
(0.825–0.934) is nearly 0.37 wide — there was no way for this to be a close
call. A tighter version would test *adjacent* topics my corpus doesn't
cover but a shallow reader might expect it to — e.g. "what's the meal plan
like at [a different, fictional university]" or "how do I appeal a parking
ticket" (parking permits are covered, ticket appeals aren't). That would
actually stress the gate's precision instead of just its recall.

**Criterion 5 (near-duplicate confusion) had the thinnest real evidence.**
My target names "4 of 5," but I only identified 2 genuine sibling-pairs in
the whole corpus. 2/2 is a real, positive result, but it's not the same
strength of evidence as 4/5 out of a true sample of 5. If I were rewriting
this criterion, I'd either (a) commit to finding 5 real sibling-pairs before
writing the target, or (b) rewrite the target itself to match what I
actually have: "both identified sibling-pairs are disambiguated correctly."
That's the criterion I'd tighten, and to that specific rewording.

**Criterion 4 surfaced a real, if currently harmless, bug** —
`study_library_hours.txt#0` is under my merge threshold but never got merged,
because my merge logic only looked backward at the previous paragraph, not
forward. It happened not to break anything this round (the chunk was still a
complete, correct fact-plus-exception pair), but it was a latent gap, not a
false alarm — a differently-worded short first paragraph elsewhere could
have produced a genuinely truncated chunk. This is the improvement I made
below.

## The Improvement

**What I changed:** Fixed `chunker.py::split_documents`'s merge logic. It
previously only merged an undersized paragraph backward into the one before
it — so a short paragraph with nothing before it (the first paragraph after
a heading) had no neighbor to merge into and survived as an undersized
chunk. The fix adds a forward pass: before the existing backward-merge loop
runs, any leading paragraph under 150 characters is merged into the
paragraph immediately after it.

**Why I picked it:** Directly named in Milestone 3's diagnosis —
`study_library_hours.txt#0` was a real, confirmed instance of this bug (a
126-character body paragraph that should have merged but didn't), not a
hypothetical edge case.

### Run Log — After

- Produced by: `run_eval.py::main` (criteria 1–3) and manual chunk
  inspection (criterion 4)
- Chunks from `chunker.py::split_documents` (bidirectional merge fix applied)
- Corpus: `campus_life` · top-k: 5 · relevance cutoff: 0.6 · 91 chunks (was 92)
- When: 2026-09-29 23:35

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks keep a fact and its exception together | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Retrieval doesn't confuse near-duplicate topics | 4 of 5 | 2/2 | 2/2 | 2/2 | MET |

**Evidence for criterion 4, after the fix** — `study_library_hours.txt`,
`chunker.py::split_documents`:

Library hours and where to actually sit

Open until 2am during term, until 10pm during reading week, which is backwards and catches everyone out every single year.

Third floor is silent and enforced. Second floor is quiet in theory. The basement has the only outlets at every seat and is therefore full from about 10am.


Before the fix this was two chunks (`#0` at 126 characters, under threshold
with nothing merged; `#1` separate). After the fix it's one merged chunk —
confirmed via `python app.py chunks --from-doc study_library_hours.txt`
reporting "Showing all 1" instead of "all 2."

**Did it help?**

Not in the sense of moving any of the five measured criteria — every
distance, every retrieved source, and every generated answer for my 5 test
questions came back identical to "before" (same five distances: 0.248,
0.256, 0.402, 0.457, 0.376; same 5/5 gate refusals). That's expected: none
of my five test questions touch `study_library_hours.txt`, the one document
affected by the bug.

What it did do is close a real, confirmed gap in the chunker: the merge
logic only checked backward, so a short first paragraph (with nothing
before it) could survive as an undersized fragment — and one actually did,
in this exact document. The chunk count dropped from 92 to 91, and directly
inspecting the affected document confirms it's now one complete chunk
instead of two.

I'm reporting this plainly rather than stretching for a number that moved:
a correct, targeted fix with no visible effect on my current test questions
is still a real, honest result. It matters for any future question that
touches library hours, and it's a concrete example of exactly the kind of
invariant-violation my own external code review caught — a case where "I
read the output and it looked right" (the original 5 sample chunks in
Unit 1) missed a bug that only showed up once I checked all 92 chunks.

## What's Still Broken

Nothing failed a criterion, but two things are genuinely unfinished, and I'm
naming them rather than letting five MET verdicts imply the system is done:

**Criterion 3's target is still too easy to be a meaningful ongoing check.**
My OUT_OF_SCOPE questions share no vocabulary with campus admin topics, so
the gate has never actually been tested near its boundary. If I kept
building this, I'd add a second, harder OUT_OF_SCOPE set — adjacent-but-
uncovered questions like "how do I appeal a parking ticket" (parking is
covered, ticket appeals aren't) — and track that separately from the current
easy set, rather than replacing it. I stopped here because writing five good
adjacent-but-uncovered questions took real thought about my corpus's actual
boundaries, and I wanted the existing four criteria solid before spending
more time on a fifth check for the same criterion.

**Criterion 5 is resting on only 2 real data points against a "4 of 5"
target.** I named this in the Verdicts and Diagnoses sections already: I
never found 5 genuine sibling-pairs in this corpus, only 2. Both passed, but
that's weaker evidence than the target implies. I'd either need to
deliberately search harder for more near-duplicate pairs, or accept that
this corpus may only support a 2-pair test and rewrite the criterion to say
so honestly. I stopped at 2 because manufacturing artificial sibling-pairs
that don't reflect real ambiguity in my corpus would test something other
than what the criterion is actually about.

**No scorer.py exists yet**, so criteria 1 and 2's verdicts came from me
reading all 15 answers by hand rather than an automated, repeatable check.
That's the biggest single risk to this being a durable measurement — if
either of us revisits this months from now, "I read the answers and they
looked right" is much weaker evidence than a script that judges answers
against `expects` consistently. Building `scorer.py` is the highest-leverage
thing left to do, but it's explicitly a class exercise in this course, so I
stopped rather than pre-building it outside that context.

## What I'd Do Differently

**Criterion 5, specifically, I'd write differently next time.** I wrote "4
of 5" before I'd actually gone looking for how many real sibling-pairs
existed in my corpus — I assumed there'd be roughly as many as there were
test questions, which turned out not to be true. Next time I'd inventory
the corpus for near-duplicate document pairs *first*, in Milestone 2, and
size the target to match what I actually find, rather than defaulting to
the same "4 of 5" shape as my other criteria out of habit.

**Criterion 4 taught me that a target needs to say what counts as a
sample.** "At least 4 of 5 sampled chunks" let me satisfy the letter of the
criterion by reading only the 5 chunks I'd already picked for the README —
which is exactly how the backward-only merge bug stayed invisible through
all of Unit 1. It only surfaced because Unit 2 pushed me to read all 92
chunks, not because the criterion demanded it. If I rewrote it, I'd specify
"sampled across the full chunk list, not just the chunks already chosen for
the README" — the sampling method matters as much as the count.