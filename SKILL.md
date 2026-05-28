---
name: mcq-quality-coach
description: generate, review, audit, and revise multiple-choice questions and quiz items against learning objectives and the 19 item-writing flaw criteria. use when the user asks to create mcqs, quiz questions, test items, answer choices, rationales, learner feedback, misconception tags, lms-ready items, or to review, fix, improve, or evaluate an existing mcq. use for assessment design quality assurance, especially scenario-based apply, evaluate, analyze, or design items with plausible distractors and clear single-best answers. do not use for free-response or open-ended questions unless the user explicitly wants them converted to mcq format.
---

# MCQ Quality Coach

Treat MCQ generation as assessment design and quality assurance, not writing. Every item must be valid, clear, aligned to a learning objective, and ready for human review.

## Input contract

A learning objective is sufficient when it specifies both:
1. What learners should be able to do, usually through an action verb.
2. The concept, skill, decision, or judgment learners apply.

**Sufficient:** "Explain why retrieval practice improves long-term retention compared with rereading." / "Choose the best feedback strategy for an overconfident learner."

**Insufficient:** "Make a question about motivation." / "Assess AI ethics."

When sufficient, generate using defaults and state assumptions briefly. When insufficient, ask 1 or 2 clarifying questions, prioritizing:
- What should the learner be able to do with this topic?
- Who is the audience or course context?

Do not ask for difficulty, audience, count, or course context unless missing information would materially affect validity.

## Defaults

Use unless overridden:
- 3 answer choices total: 1 correct answer plus 2 plausible distractors.
- 1 MCQ per request.
- Difficulty, audience, and Bloom's level inferred from the learning objective.
- Full review mode: draft -> deterministic pre-check -> adversarial pre-audit -> 19-IWF audit -> revise if needed -> final.
- Scenario-based stems for apply, analyze, evaluate, create, design, implementation, or decision-making objectives.
- Nuanced distractors that reflect real novice misconceptions, partial understandings, or suboptimal decision rules.

If the user asks for 4 or more options, comply only when all distractors can remain plausible and parallel. Prefer fewer strong distractors over more weak distractors. Treat the 3-option default as an efficiency and quality-control default, not a universal research finding.

## Evidence stance and psychometric guardrails

Treat the 19-IWF rubric as a pre-deployment quality screening framework, not as psychometric validation. IWF criteria help detect design flaws that can cue guessing, add construct-irrelevant cognitive load, reduce clarity, or weaken alignment. They do not prove that an item is valid, reliable, appropriately difficult, or discriminating. Psychometric validation requires learner-response data, and specialized or high-stakes content still requires SME review.

Before deployment, use design-based evidence to improve item quality:
- **Validity support:** State a one-line assessment target for every item. Check alignment to the learning objective, content accuracy, cognitive demand, exactly one defensible answer, and whether the item measures the intended construct rather than wording recognition, test-wiseness, or reading burden.
- **Reliability or precision support:** For important constructs, recommend item sets rather than treating one MCQ as a reliable measure. Reduce avoidable noise by removing ambiguity, irrelevant cues, and construct-irrelevant complexity.
- **Difficulty support:** Label difficulty as estimated before deployment. Base the estimate on reasoning steps, prerequisite knowledge, scenario complexity, option similarity, and distractor plausibility.
- **Discrimination support:** Improve the chance that the item separates stronger from weaker understanding by using plausible distractors, avoiding surface cues, and removing ambiguity or multiple defensible answers. Do not claim empirical discrimination without response data.
- **Fairness support:** Check for unnecessary reading load, culturally specific assumptions, inaccessible wording, and barriers unrelated to the target construct.
- **Distractor quality support:** Ensure each distractor reflects a plausible misconception, partial understanding, or suboptimal decision rule, not a throwaway wrong answer.

After deployment, use learner-response data when available to evaluate empirical difficulty, discrimination, distractor functioning, reliability, fairness, and whether the item should be kept, revised, replaced, or retired. Interpret **Ready** as "ready for pilot or human review," not "validated." See `references/mcq-evidence-audit.md` for evidence tiers and post-deployment checks.

## Workflow: Generation

1. **Setup.** Identify the assessment target. State assumptions: audience, Bloom's level, estimated difficulty, and any interpretation of the learning objective. Use the psychometric guardrails above to avoid claiming validation from text review alone.
2. **Consult targeting guidance.** Read `references/blooms-targeting.md` when inferring Bloom's level or choosing stem style.
3. **Verify content and validity evidence.** Confirm the correct answer is factually correct and distractors are factually incorrect for the stated target. Check that the item measures the intended construct rather than test-wiseness, wording recognition, or unnecessary reading burden. If the topic is specialized, fast-changing, high-stakes, or not verifiable from available context, mark the final status "Requires SME verification" even if the audit passes.
4. **Draft the MCQ.** Put the full problem in the stem. For higher-order objectives, use a realistic decision point with only details that affect the answer.
5. **Run deterministic pre-checks.** Execute `scripts/validate.py` against the draft before the 19-IWF audit whenever code execution is available. Surface the validator summary and relevant findings.

   Treat validator output as candidate flags, not final verdicts. The validator detects patterns; use context to decide whether each flag is a genuine flaw or intrinsic to the topic. Example: if the stem is about "retrieval practice," the word "retrieval" may repeat in the correct answer. If unavoidable, downgrade to Minor risk and explain why.
6. **Run an adversarial pre-audit.** Try to answer as a test-wise learner with partial subject knowledge. If surface features, grammar, length, repeated words, or option patterns reveal the answer, revise.
7. **Run the 19-IWF audit.** Read `references/19-iwf-rubric.md` before producing the audit table. Do not rely on memory of the criteria.
8. **Revise.** If any criterion is Minor risk or Flagged, revise. Cap at 2 revision cycles. If still flagged after 2 revisions, output with status "Needs Major Revision - manual rewrite recommended."
9. **Finalize.** Provide the final item, answer key, instructor-facing rationales, learner-facing feedback, misconception tags, and quality status.

## Workflow: Review existing MCQs

1. State assumptions used in the review.
2. Identify the intended learning goal. If missing and alignment cannot be judged, ask once.
3. Verify content correctness and confirm exactly one defensible best answer.
4. Check alignment, intended construct, cognitive demand, and likely construct-irrelevant burden.
5. Check stem clarity and whether the full problem is in the stem.
6. Check distractor plausibility, parallel structure, and instructional value.
7. Estimate difficulty from reasoning steps, prerequisite knowledge, scenario complexity, and distractor plausibility.
8. Check whether the item is likely to discriminate by requiring the intended understanding rather than surface cues.
9. Run `scripts/validate.py` for deterministic checks when code execution is available.
10. Run the adversarial pass to identify surface cues.
11. Run the 19-IWF audit using `references/19-iwf-rubric.md`.
12. If any flaw or minor risk appears, provide a revision and explain why it is better.
13. Output rationales, learner feedback, misconception tags, and quality status.

## 19 Item-Writing Flaws

Full definitions, examples, and audit guidance are in `references/19-iwf-rubric.md`. Read that file at audit time. The criteria are:

1. Ambiguous wording
2. Implausible distractors
3. "None of the above"
4. Correct answer longest or most detailed
5. Gratuitous information in stem
6. True/false-style options
7. Convergence cues
8. Logical clues between options
9. "All of the above"
10. Incomplete or fill-in-the-blank stem
11. Absolute terms, such as always, never, all, none
12. Word repeats from stem to correct answer
13. Unfocused stem
14. Combination or K-type options
15. Grammatical cues or non-parallel structure
16. Numerical options out of sequence
17. Vague qualifiers, such as often, usually, mostly
18. More than one defensible answer
19. Negative wording, such as NOT, EXCEPT, LEAST

## Audit table format

Use the columns: `# | Criterion | Status | Evidence | Fix if needed`

Statuses:
- **Pass**: criterion met.
- **Minor risk**: a non-expert reader could be cued or confused; not a clear violation.
- **Flagged**: clear violation requiring revision.
- **Not applicable**: criterion does not apply to this item.

## Skepticism priorities

Watch especially for correct answer length or specificity cues, more than one defensible option, technically true distractors that do not answer the stem, obviously wrong distractors, non-parallel options, recognition-only stems for higher-order objectives, and realistic but unnecessary scenario detail.

## Output modes

Use Markdown by default. Use the JSON schema in `assets/output-schema.json` only when the user asks for structured output, LMS integration, batch processing, or machine-readable results.

## Output format: Generation

```markdown
Assumptions used: [audience, Bloom's level, estimated difficulty]
Assessment target: [what construct, skill, decision, or misconception the item measures]
Design-based psychometric notes: [brief validity, reliability/precision, difficulty, discrimination, and fairness considerations]

Draft MCQ:
[stem]
A) ...
B) ...
C) ...

Deterministic pre-check results:
[validator summary and relevant findings]

Adversarial pre-audit notes:
[surface cues identified, if any]

19-IWF audit:
[Markdown table]

Revision decision: [revise / accept, with brief reason]

Final MCQ:
[stem]
A) ...
B) ...
C) ...

Correct answer: [letter]

Instructor-facing rationales:
- Correct answer: [why correct, what it tests]
- Distractor A/B/C: [why a novice picks this, tied to a misconception]

Learner-facing feedback:
- Correct option: [see feedback rules]
- Each incorrect option: [see feedback rules]

Misconception tags: [2-4 short labels]

Final quality status: [Ready / Needs Minor Revision / Needs Major Revision / Reject and Rewrite / Requires SME verification]
```

## Output format: Review

```markdown
Overall status: [one-line summary]
Assumptions used: [audience, Bloom's level, estimated difficulty]
Learning objective alignment: [aligned / partial / misaligned]
Design-based psychometric notes: [brief validity, reliability/precision, difficulty, discrimination, and fairness considerations]

Original MCQ:
[as provided]

Deterministic pre-check results:
[validator summary and relevant findings]

Adversarial pre-audit notes:
[surface cues identified, if any]

19-IWF audit:
[table]

Recommended revision:
[revised item]

Why the revision is better:
[point-by-point]

Instructor-facing rationales:
[as above]

Learner-facing feedback:
[as below]

Misconception tags: [2-4 labels]
Final quality status: [as above]
```

## Rationale rules

- Label the correct option as "Correct answer." Label only incorrect options as "Distractor A/B/C."
- Explain why a novice would find each distractor attractive and tie it to a specific misconception, partial understanding, or suboptimal decision rule.
- Avoid generic rationales like "This is incorrect because it is wrong."
- Keep distractors comparable in sophistication to the correct answer. Do not make incorrect options obviously naive unless the learning objective targets novice misconception recognition.

## Learner-facing feedback format

Generate one feedback string per option for LMS delivery. This feedback is shown after the learner has attempted the item and is distinct from instructor-facing rationales. Use the same learner-facing feedback style for formative and summative practice unless the user explicitly asks for a different style. Always reveal the correct answer after an incorrect response because the goal is learning.

**For the correct option:**
> Correct. [Briefly explain why the answer is correct.]

Default length: 20-35 words.

**For each incorrect option:**
> Not quite. [Briefly explain why the selected option does not fit or what reasoning it reflects.] The correct answer is "[exact correct option text]" because [key reason it is correct].

Default length: 30-50 words. Hard cap: 55 words.

Rules:
- Keep feedback concise, corrective, task-focused, and learning-oriented.
- Treat word limits and the "Not quite." opening as LMS readability and tone defaults, not psychometric claims.
- Do not force identical wording across incorrect options. Keep the core explanation of the correct answer consistent while tailoring each incorrect feedback to the selected option.
- Quote the exact correct option text in each incorrect feedback string.
- Avoid shaming language, instructor-facing labels such as "novice misconception," new side concepts, citations, and extended teaching paragraphs.
- Before finalizing, run this feedback QA check: each string is under the hard cap; each incorrect feedback reveals the correct answer; each incorrect feedback addresses the selected option specifically; the explanation is concise enough for learners to read; the tone is supportive and non-punitive.

## Misconception tag format

Provide 2-4 short labels, 3-7 words each, describing the underlying misunderstanding.

Examples: "Confuses retrieval with rereading" / "Treats fluency as learning" / "Overweights recent feedback"

## Quality status thresholds

- **Ready**: 0 Flagged, no content verification concern, and at most 1 Minor risk. Interpret as ready for pilot or human review, not psychometrically validated.
- **Needs Minor Revision**: 0 Flagged and 2 or more Minor risks, or the item already required revision but is now acceptable.
- **Needs Major Revision**: 1 or more Flagged criteria after one revision cycle.
- **Reject and Rewrite**: alignment failure, content inaccuracy, or 3 or more Flagged criteria.
- **Requires SME verification**: specialized, high-stakes, or unverifiable topic, regardless of audit status.

## Quality rules

- Prefer scenario-based stems for apply, analyze, evaluate, create, design, or decision-making objectives.
- Avoid trick questions.
- Avoid "all of the above," "none of the above," and negative wording unless explicitly requested or pedagogically justified.
- Make options parallel in grammar, length, specificity, and mechanism.
- Use positive wording whenever possible.
- Do not claim psychometric validation, reliability, empirical difficulty, or empirical discrimination without learner-response data and appropriate analysis.
- Present outputs as draft assessment materials for human review or pilot use.
- When learner-response data is available, recommend post-deployment item analysis: difficulty, discrimination, distractor functioning, reliability/precision, subgroup patterns, and revision or retirement decisions.

## Reference files

- `references/19-iwf-rubric.md`: full criterion definitions, examples, and audit guidance. Read at audit time.
- `references/blooms-targeting.md`: Bloom's level inference from learning-objective verbs and stem-style mapping. Read when setting up the assessment target.
- `references/before-after-examples.md`: bad to revised MCQ pairs. Read when uncertain how to revise a flagged item.
- `references/mcq-evidence-audit.md`: evidence tiers and psychometric guardrails. Read when the user asks why a guideline exists, asks for validity/reliability/difficulty/discrimination guidance, or asks for evidence behind MCQ criteria.
- `scripts/validate.py`: deterministic pre-check script. Run before every audit when code execution is available.
- `assets/output-schema.json`: JSON schema for structured output.

## Style

Direct, practical, rigorous. Use clean Markdown. Use tables only when they improve clarity. Do not narrate internal process; produce the requested assessment material directly.
