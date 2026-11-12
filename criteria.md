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

**Why this target:**
All five questions are based on facts stated in my corpus, so a relevant chunk should usually be retrievable. I allow one miss because retrieval can still fail on a less directly worded question.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
Naming a source lets a user trace each answer back to a document in the corpus. I require this for every answer because an answer without a source cannot be checked.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.


**Why this target:**
This checks whether the system can recognize when a question is outside the corpus. I allow one failure because the distance between an out-of-scope question and a loosely related chunk may sometimes be ambiguous.

---

## 4. Something about your chunks

No chunk is shorter than 100 characters or longer than 500 characters.



**Why this target:**
This range should keep each chunk long enough to contain a complete idea, while preventing a chunk from including too much unrelated information. The upper limit is stricter than the starter baseline, whose longest chunk was 549 characters.

---

## 5. Your choice

For at least 4 of my 5 test questions, the final answer contains the expected word or phrase from questions.py.



**Why this target:**
This checks whether the final generated answer includes the specific fact I decided was necessary before testing. I allow one miss because a model may phrase one answer differently even when the relevant chunk was retrieved.

---
