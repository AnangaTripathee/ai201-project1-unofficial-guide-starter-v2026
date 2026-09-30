# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:** My documents are short, single-topic posts (88
documents, average 317 characters), so most questions should map cleanly to
one chunk. I expect 4/5 rather than 5/5 because two of my questions —
withdrawal and add/drop — live in separate but similarly-worded documents
(admin_withdrawal_deadline.txt vs admin_add_drop_deadline.txt), so retrieval
could plausibly confuse them.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:** This is 5 of 5, not 4 of 5, because it isn't about
retrieval quality — it's about whether generate.py's grounding instruction
and prompt format are followed at all. The source filename is mechanically
included in every prompt (build_prompt in generate.py labels each chunk with
"[from {source}]"), so the only way this fails is if the model ignores an
explicit instruction, which should be rare.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:** My corpus is entirely campus administrative topics
(housing, registration, dining, jobs), and OUT_OF_SCOPE questions are from
completely unrelated domains (world capitals, car engines, sports). I expect
these to be easy for the gate to catch, so 4/5 is a low bar for me — I'd
actually be concerned if I scored below that.

---

## 4. Something about your chunks

At least 4 of 5 sampled chunks contain a stated fact AND its qualifier/
exception in the same chunk, with neither cut off.

**Why this target:** Several of my documents pack a rule and its exception
into one short paragraph — e.g. admin_dining_dollars.txt: "rolls over from
autumn to spring, but not from spring to the following autumn," or
admin_add_drop_deadline.txt: "add through week two... drop through week
six... but a drop after week two shows as a W." If a chunk cut before the
"but," the answer would be confidently wrong rather than incomplete. My
custom chunker (chunker.py::split_documents) merges fragments under 150
characters into the previous paragraph specifically to prevent this.

---

## 5. Your choice

For at least 4 of 5 questions that have a similarly-worded "sibling"
document in the corpus, the top-1 retrieved chunk is the correct one, not
its sibling.

**Why this target:** My corpus has several pairs of documents that use
overlapping vocabulary but describe different rules —
admin_withdrawal_deadline.txt vs admin_add_drop_deadline.txt (both mention
weeks, deadlines, and a "W" on the transcript), and
admin_campus_jobs_and_financial_aid.txt's work-study vs non-work-study
distinction. Getting the wrong sibling back would produce a fluent, sourced,
totally wrong answer — worse than a refusal, since nothing in the output
would flag it. 4 of 5 rather than 5 of 5 because I only have two real
sibling-pairs to test against, so one miss is a meaningfully different
result than it would be with a bigger sample.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->