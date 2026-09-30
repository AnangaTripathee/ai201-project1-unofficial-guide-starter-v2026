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