# Example: mcq-quality-coach in action

This file shows one real conversation captured from Codex CLI. Use it
to complete the Notice 1 / Notice 2 / Notice 3 observations from the
OLI module if you can't run the skill yourself.

## Setup

- **Client:** Codex CLI
- **Skill installed at:** `~/.agents/skills/mcq-quality-coach/`
- **Prompt:** *"Write me three quiz questions about how a relational
  database differs from a key-value store."*

The user did not name the skill. They asked for "quiz questions" in
plain English.

## What happened

```
> Write me three quiz questions about how a relational database
  differs from a key-value store.

• I'll use the mcq-quality-coach skill so the questions are clean
  assessment items rather than loose prompts.

• Explored
  └ Read SKILL.md (mcq-quality-coach skill)

• I'm checking the validator and rubric once, then I'll draft the
  three items and keep the user-facing output compact.

• Explored
  └ Read 19-iwf-rubric.md, validate.py

• The drafts target an introductory database audience and focus on
  schema/querying, lookup patterns, and relationship management.
  I'm running the deterministic surface checks now, then I'll revise
  any item that shows cueing risk.

• Ran scripts/validate.py on three draft MCQs.

  [Initial run failed: the system `python` was Python 2, which
  couldn't parse the validator's non-ASCII characters. The assistant
  switched to `python3` and re-ran successfully.]

• Ran scripts/validate.py with python3.
  └ Validator returned a structured report for each item.

• One item had a lexical cue because the word "value" appeared only
  in the keyed answer. I'm adjusting that option set so surface word
  matching does not point to the correct answer.

• Re-ran validator on the revised item. All three items now pass.

────────────────────────────────────────────────────────────────────

Assumptions used: introductory database learners; multiple-choice
format; moderate difficulty. Quality status: Ready.

1. A team needs to store orders, customers, and products, then
   answer questions such as which customers bought products from
   category X last month. Which storage model is better suited to
   this need?

   A) Relational database with normalized tables and SQL joins
   B) Key-value store with composite keys for each query pattern
   C) Key-value store with one large value per customer record

   Correct answer: A

2. An application retrieves session data by a unique session ID and
   rarely searches inside the stored value. Which storage model
   directly matches this access pattern?

   A) Key-value store using the session ID to retrieve the value
   B) Relational database using joins to retrieve the session value
   C) Relational database using normalization to split the value

   Correct answer: A

3. Which task generally requires more application-level handling in a
   basic key-value store than in a relational database?

   A) Preserving relationships between customers, orders, and
      products
   B) Retrieving a value when the application already has its key
   C) Saving a value under a key chosen by the application

   Correct answer: A
```

## Where each notice appears

**Notice 1 — the assistant invokes the skill before writing.**
Look at the second line of the trace: *"I'll use the mcq-quality-coach
skill so the questions are clean assessment items rather than loose
prompts."* The user didn't name the skill. The assistant matched
"quiz questions" against the skill's description and invoked it
implicitly. Codex CLI surfaces this as a plain-English plan; other
clients may show it as a different kind of activation indicator
(or not surface it at all — see the "If your client doesn't display
this" note in the OLI page).

**Notice 2 — the questions follow the 19-IWF format.**
Read the three questions above. Notice that each stem asks one
focused thing, the options are similar in length (no length cue),
the distractors are plausible misconceptions rather than throwaways,
and there's no "all of the above" or "none of the above." The skill
enforces these constraints — both through instructions in `SKILL.md`
and through the deterministic Python validator the assistant ran
during drafting.

**Notice 3 — the assistant explains its reasoning against the rubric.**
The trace shows the assistant doing four things students typically
don't see:

1. **Reading the rubric before drafting** (`Read 19-iwf-rubric.md`).
   The skill points at the canonical 19-IWF reference; the assistant
   loaded it on demand.
2. **Running the deterministic validator after drafting**
   (`Ran scripts/validate.py`). This is the same validator from
   Tool Task 5.
3. **Catching its own item-writing flaw** ("the word 'value' appeared
   only in the keyed answer"). That's IWF #12 — word repeat from
   stem appearing only in the correct answer. The validator flagged
   it and the assistant revised.
4. **Surfacing the assumptions and quality status** at the end
   ("introductory database learners; moderate difficulty; Quality
   status: Ready"). The skill instructs the assistant to make these
   explicit so a human reviewer can sanity-check the items.

## Notes on this specific run

A few things in the trace are real but worth understanding:

- **The validator's first run failed.** The assistant's first attempt
  used the system `python` interpreter, which on this machine is
  Python 2. The validator script has non-ASCII characters and Python
  2 rejected them. The assistant correctly switched to `python3` and
  re-ran. This is an environment issue, not a skill bug. If you run
  the validator standalone, use `python3`.

- **The questions have 3 options, not 4.** Three-option items are the
  evidence-based default in this skill. Vegada et al. (2016) randomly
  compared three-, four-, and five-option MCQs and found no
  significant performance difference between them — and recommended
  three options as the default, since adding more options costs
  student reading time without improving measurement quality. Adding
  a fourth option typically introduces an implausible distractor that
  students discard immediately, which costs time without measuring
  anything new.

  > Vegada B, Shukla A, Khilnani A, Charan J, Desai C. Comparison
  > between three option, four option and five option multiple choice
  > question tests for quality parameters: A randomized study.
  > *Indian J Pharmacol*. 2016 Sep-Oct;48(5):571-575.
  > doi: 10.4103/0253-7613.190757. PMID: 27721545; PMCID: PMC5051253.

- **Different runs produce different items.** The skill enforces
  process and quality, not specific outputs. If you run the same
  prompt yourself, the items will differ in wording but should
  exhibit the same Notice 1 / 2 / 3 properties.
