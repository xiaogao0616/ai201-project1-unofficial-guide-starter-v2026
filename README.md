# The Unofficial Guide

Name: Xiangyi Gao
Corpus: campus_life

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

The Unofficial Guide is a retrieval-based question-answering system using the `campus_life` corpus of 88 short documents. It answers questions about campus policies, course assessments, library hours, housing, and other topics covered by those documents. The system retrieves document chunks and asks a language model to answer using only those excerpts, with source filenames. A relevance gate stops questions whose best retrieval distance exceeds the cutoff; this reduces unsupported answers but does not guarantee that every accepted question is answerable.

## Milestones 1 and 2 — Setup and Evaluation Targets

**Milestone 1:** I selected `campus_life` and indexed 88 documents containing 27,908 characters. The starter produced 88 chunks, averaging 317 characters, with a minimum of 178 and a maximum of 549. This showed that the starter's 800-character windows kept these short documents intact. Before replacing the chunker, I also ran the separate `advice_threads` sample command and recorded its total of 26 chunks.

**Milestone 2:** I wrote five factual test questions in `questions.py`, covering major declaration, printing credit, study abroad applications, CS 340 exams, and library hours, each with an `expects` phrase. I retained five `OUT_OF_SCOPE` questions for testing refusals. My five targets are recorded in `criteria.md`: answer-containing retrieval for at least 4 of 5 questions, a source for every answer, refusal of at least 4 of 5 out-of-scope questions, chunk lengths of 100–500 characters, and the expected phrase in at least 4 of 5 final answers. These are targets, not claims that all five criteria have passed the Unit 2 evaluation.

## Chunking Strategy

**Chunk size:** Up to 500 characters for the current campus_life corpus. Posts of 500 characters or fewer remain intact. Longer posts are split at paragraph boundaries.

**Overlap:** No fixed character overlap. For split posts, the title is repeated at the beginning of each subsequent chunk to preserve context.

The campus_life corpus contains 88 short documents, averaging about 317 characters each. Most posts already describe one topic, so I keep them intact rather than splitting them unnecessarily. Two housing posts exceeded 500 characters and contained clear paragraph breaks, making paragraph-based splitting suitable.

My function, `chunker.py::split_documents`, combines consecutive paragraphs until adding another would exceed 500 characters. It then starts a new chunk with the same title, so details such as laundry costs remain associated with the correct residence hall.

The starter produced 88 chunks, ranging from 178 to 549 characters. My strategy produced 90 chunks, averaging 310 characters, with a minimum of 170 and a maximum of 461. A check of all chunks found no violations of my 100–500 character criterion.

This strategy assumes that a long post begins with a title and has short paragraphs, as observed in the two long housing posts. It does not yet handle a single paragraph that is too long to fit with the title.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```text
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160_exams.txt#0` — produced by: `chunker.py::split_documents`

```text
BIOL 160 Cell Biology — assessment

Four unit tests and a cumulative final. Not curved.

The unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_math_220_exams.txt#0` — produced by: `chunker.py::split_documents`

```text
MATH 220 Linear Algebra — assessment

Two midterms and a cumulative final. Curved to a b- median.

The problem sets are the course; the lectures make sense afterwards rather than during.
```

**Chunk 4** — source: `dining_the_ridgeway_cafe.txt#0` — produced by: `chunker.py::split_documents`

```text
The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00. The thing worth going for is the only place on campus with real espresso. The thing to know is that seating is tight; about 40 seats for a building of 900.

Hours are 7:00am to 4:00pm weekdays only. Costs declining balance only, no meal swipes.
```

**Chunk 5** — source: `housing_morrow_house.txt#0` — produced by: `chunker.py::split_documents`

```text
Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.

Laundry costs $1.50 wash, $1.25 dry, coin or card. On noise: loud until about 1am on weekends, no enforced quiet hours.
```

## Sample Answer

**Question:** How many exams does CS 340 have?

**Answer:**

```text
CS 340 has one midterm and a final.

Sources: `course_cs_340_exams.txt` and `course_cs_340.txt`
```

**My relevance cutoff:** `0.6` (the starter value, retained after measurement).

The five in-corpus questions had best distances from 0.3075 to 0.3926. The five out-of-scope questions had best distances from 0.8246 to 0.9340. The cutoff of 0.6 falls between these groups: all five in-corpus questions passed the gate, and all five out-of-scope questions were blocked. These results support keeping 0.6 for this test set, but do not establish how well it handles unseen questions, especially campus-related questions that the documents do not answer.

I retained the five-result retrieval setting. I inspected the assembled prompt and grounding instruction with `--show-prompt`: it requires using only the provided documents, admitting missing information, and naming source files. In the CS 340 example, the answer used the correct course's exam information despite CS 210 excerpts also appearing in the prompt. I left the grounding instruction unchanged after this check.

| Question | In corpus? | Best distance |
|---|---|---|
| By when must students declare their major? | Yes | 0.3871 |
| How much campus printing credit does each student receive? | Yes | 0.3758 |
| When should students apply for a study abroad program? | Yes | 0.3339 |
| How many exams does CS 340 have? | Yes | 0.3075 |
| What are the library's hours during the term and during reading week? | Yes | 0.3926 |
| What is the capital of Mongolia? | No | 0.8246 |
| How do I change the oil in a diesel engine? | No | 0.9340 |
| Who won the 1994 World Cup? | No | 0.8859 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| How do I write a for loop in Rust? | No | 0.8960 |

**Out-of-scope check:**

```text
Question: What is the capital of Mongolia?
(best distance 0.825, cutoff 0.6)

I don't have enough information about that.

0 model calls this session
```

## How I Used AI

**1. Chunking implementation.** I asked ChatGPT to guide me through replacing the starter chunker. It first helped me implement one chunk per document, then suggested printing documents over 500 characters and inspecting their paragraphs. After I shared the two long housing posts, it provided a paragraph-based implementation that repeats the title in subsequent chunks. I used that implementation without further algorithm changes, replacing the temporary one-document-per-chunk version. I ran it locally and checked every chunk's length: it produced 90 chunks ranging from 170 to 461 characters, with no violations of my 100–500 character target. The implementation is suited to the current corpus but does not handle an oversized individual paragraph.

**2. Retrieval cutoff and grounding checks.** I shared retrieval results for five in-corpus and five out-of-scope questions and asked ChatGPT to help interpret them. It recommended retaining the default cutoff of 0.6 because the two distance ranges did not overlap, and it pointed out that CS 210 excerpts could distract the model when answering about CS 340. I did not change the cutoff or grounding instruction. Instead, I checked the actual assembled prompt and answer using `--show-prompt`, then tested an out-of-scope question. The CS 340 answer named the correct sources, and the out-of-scope question was refused with zero model calls. I added the measured distances and actual outputs to this README with AI assistance organizing the text.

**3. Unit 2 evaluation and improvement.** I shared the three before-run logs with ChatGPT to help aggregate the per-question results into the five acceptance criteria. It helped me notice that the answers met the source requirement but used inconsistent citation formats. I chose a single generation-stage change: I strengthened `generate.py::GROUNDING_INSTRUCTION` to require one final `Source: filename.txt` line. I then ran the after evaluation with caching off and used the generated run log, rather than AI-generated results, to fill in the after table and conclusion.

---

# Unit 2

## Run Log — Before

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Chunks are 100–500 characters | 0 violations | 0 of 90 violations | 0 of 90 violations | 0 of 90 violations | MET |
| 5. Final answer contains the expected phrase | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
Evidence for criteria 1, 2, 3, and 5 is from `results/run_2026-09-23_2105_before.md`, produced by `run_eval.py::main` and `run_eval.py::check_out_of_scope`. Criterion 4 was measured from the chunks produced by `chunker.py::split_documents`.

### Criterion 1, 2, and 5 — real output from run 1

```text
By when must students declare their major?

Best distance: 0.3871 (passed the gate)
Sources retrieved: admin_add_drop_deadline.txt, admin_declaring_a_major.txt,
admin_graduation_requirements.txt, admin_pass_fail_option.txt,
admin_study_abroad.txt

Students declare their major at the end of their second semester, or later if
needed, as there is no penalty for declaring late.

Source: admin_declaring_a_major.txt
```

```text
How much campus printing credit does each student receive?

Best distance: 0.3758 (passed the gate)
Sources retrieved: admin_campus_jobs_and_financial_aid.txt,
admin_graduation_requirements.txt, admin_printing_quota.txt, money_jobs.txt,
money_textbooks.txt

Each student receives $30 of printing per semester.

Source: admin_printing_quota.txt
```

```text
When should students apply for a study abroad program?

Best distance: 0.3339 (passed the gate)
Sources retrieved: admin_graduation_requirements.txt, admin_pass_fail_option.txt,
admin_study_abroad.txt, advising_registration.txt, course_cs_340.txt

Students should apply in October for the following academic year.

Source: admin_study_abroad.txt
```

```text
How many exams does CS 340 have?

Best distance: 0.3075 (passed the gate)
Sources retrieved: course_cs_210.txt, course_cs_210_exams.txt,
course_cs_340.txt, course_cs_340_exams.txt, course_cs_340_workload.txt

CS 340 has one midterm and a final (two exams total).

Source: course_cs_340_exams.txt (also mentioned in course_cs_340.txt).
```

```text
What are the library's hours during the term and during reading week?

Best distance: 0.3926 (passed the gate)
Sources retrieved: housing_calder_annexe_noise.txt,
housing_morrow_house_noise.txt, money_jobs.txt, money_textbooks.txt,
study_library_hours.txt

During the term, the library is open until 2am, and during reading week it is
open until 10pm (study_library_hours.txt and housing_morrow_house_noise.txt).
```

### Criterion 3 — real output

```text
Produced by run_eval.py::check_out_of_scope, cutoff 0.6. Refused 5 of 5.

What is the capital of Mongolia? — best distance 0.825 — refused
How do I change the oil in a diesel engine? — best distance 0.934 — refused
Who won the 1994 World Cup? — best distance 0.886 — refused
What is the recommended dosage of ibuprofen for a headache? — best distance
0.844 — refused
How do I write a for loop in Rust? — best distance 0.896 — refused
```

### Criterion 4 — chunk measurement

```text
Produced by chunker.py::split_documents.

90 chunks were produced. The shortest chunk was 170 characters and the
longest chunk was 461 characters. There were no violations of the 100–500
character criterion.
```

## Verdicts


| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All five in-corpus questions passed in every run. Each question’s retrieved-source list includes the document containing the answer. |
| 2 | Every answer names a source | MET | All 15 generated answers named at least one source file. |
| 3 | Gate stops out-of-corpus questions | MET | The gate refused all five out-of-corpus questions. Because retrieval and the cutoff comparison are deterministic, the same 5 of 5 result is recorded in every run column. |
| 4 | Chunks are 100–500 characters | MET | `chunker.py::split_documents` produced 90 chunks ranging from 170 to 461 characters, so there were zero violations. |
| 5 | Final answer contains the expected phrase | MET | Every answer included its expected phrase: “second semester,” “$30,” “October,” “one midterm,” or “2am,” in all three runs. |

## Diagnoses
No criteria were missed in the before evaluation, so there is no failed
question or pipeline stage to diagnose. All five in-corpus questions retrieved
supporting chunks, named sources, and contained their expected phrases in all
three runs. The relevance gate also refused all five out-of-corpus questions.

Because every criterion passed on the first evaluation, some targets were
probably conservative rather than evidence that the system is strong in every
case. I would tighten criterion 1 from 4 of 5 to 5 of 5 retrieved chunks
containing the answer, and criterion 5 from 4 of 5 to 5 of 5 final answers
containing the expected phrase. I would keep criterion 3 at 4 of 5 for now,
because the five out-of-corpus questions are clearly unrelated to the campus
corpus and do not test borderline campus-related questions.



## The Improvement

**What I changed:** I changed `generate.py::GROUNDING_INSTRUCTION` so that every answer must end with exactly one line in the format `Source: filename.txt`. The instruction explicitly forbids parentheses, Markdown emphasis, and the `Sources:` label.

**Why I picked it:** Although criterion 2 passed before, the evidence showed inconsistent citation presentation: answers alternated between `Source:`, `Sources:`, and parenthetical filenames. This is a generation-stage inconsistency, so I changed only the generation instruction.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Chunks are 100–500 characters | 0 violations | 0 of 90 violations | 0 of 90 violations | 0 of 90 violations | MET |
| 5. Final answer contains the expected phrase | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |

Evidence source: `results/run_2026-09-23_2140_after.md`, produced by `run_eval.py::main` and `run_eval.py::check_out_of_scope`.

### Real output — run 1

```text
Students declare their major at the end of their second semester, or later if needed. There is no penalty for declaring late.

Source: admin_declaring_a_major.txt
```

```text
Every student receives $30 of printing per semester, which is approximately 600 black-and-white pages.

Source: admin_printing_quota.txt
```

```text
Students should apply for a study abroad program when applications open in October for the following academic year.

Source: admin_study_abroad.txt
```

```text
CS 340 has one midterm and a final, making a total of two exams.

Source: course_cs_340_exams.txt
```

```text
The library is open until 2am during the term, and until 10pm during reading week.

Source: study_library_hours.txt
```

```text
Produced by run_eval.py::check_out_of_scope, cutoff 0.6. Refused 5 of 5.
```

**Did it help?** Yes, it made citations consistent: all 15 after answers ended with one `Source: filename.txt` line. The five existing numeric criteria stayed at 5 of 5 in every run, so the improvement did not raise those already-maximal scores or change retrieval and gate behavior.

## What's Still Broken

No original criterion was missed after the fix. However, that does not mean the system is complete. The test set is small and fixed: the five out-of-corpus questions are clearly unrelated to campus life, so the relevance gate has not been tested on harder borderline questions, such as campus questions that the corpus does not answer. The citation-format change also checks that a filename is present, not that the cited filename is the best or only supporting source. I stopped here because the assignment asks for one isolated change and a comparable before/after measurement; adding harder test cases or source-entailment checks would change the evaluation design rather than test the single prompt change.

## What I'd Do Differently

I would make criteria 1 and 5 stricter from at least 4 of 5 to 5 of 5 because this corpus and these five questions were all chosen from explicit facts in the documents, and all three runs reached 5 of 5. For criterion 3, I would keep the numerical target at 4 of 5 but replace some obviously unrelated questions with borderline campus-related questions that are absent from the corpus. That would test whether the relevance gate rejects unsupported questions without simply relying on a large semantic distance.
