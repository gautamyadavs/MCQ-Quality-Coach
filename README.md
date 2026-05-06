# MCQ Quality Coach

> An open-standard Agent Skill for ChatGPT, Claude, and Gemini that generates and audits multiple-choice questions against item-writing flaw research.

Generates and audits MCQs against the **19 item-writing flaw criteria** from assessment-design research. Catches the cues, ambiguities, and structural flaws that make most AI-generated quiz questions unusable in real courses.

Built for curriculum designers, learning engineers, customer education teams, and anyone who needs first-draft-quality assessment items.

---

## What it does

You give it a learning objective. It gives you back:

- A draft MCQ aligned to your objective at the right Bloom's level
- An adversarial pre-audit that tries to game the question using surface cues
- A criterion-by-criterion audit table against all 19 IWFs
- A revised version if anything was flagged
- Instructor-facing rationales for each option
- LMS-ready learner feedback for correct/incorrect submissions
- Misconception tags for analytics

If you've used Custom GPTs or generic prompts to make MCQs, you know they look fine until you read them carefully — the correct answer is always slightly longer, the distractors are obviously wrong, the stem repeats the answer's keyword. This skill catches all of that before it reaches you.

## Why a hybrid approach

The SAQUET research (Moore & Chen, 2023) showed that combining **deterministic rule checks** with **LLM semantic evaluation** catches **91%** of item-writing flaws — vs. **79%** for LLM-only evaluation.

This skill implements that hybrid:

- A bundled Python validator catches **12 of 19 IWFs** that pattern-match cleanly (length cues, NOTA/AOTA, negation words, K-type combinations, numerical order, word repeats, etc.)
- The LLM handles the **remaining 7 IWFs** that require semantic judgment (#1 ambiguity, #2 implausible distractors, #5 gratuitous information, #7 convergence cues, #8 logical clues, #13 unfocused stems, #18 multiple defensible answers)

The skill's instructions tell the assistant to run `scripts/validate.py` whenever code execution is available (Claude.ai with Code Execution enabled, Claude Code, ChatGPT Codex CLI). Whether it actually runs depends on the host environment's permissions and the model's compliance with the skill instructions. In Custom GPTs and Gemini Gems, you can still run the validator manually — it's just not invoked automatically by default.

---

## Before you start

- **No installation needed for chat-based use** (ChatGPT Skills, Custom GPT, Claude.ai, Gemini Gem). Just download the release zip or copy the files where indicated.
- **Codex CLI / Claude Code** require those tools to be installed first.
- **Standalone validator** (the "Running the validator standalone" section near the end) requires Python 3.10+.

If you're a CMU student in *Tools for Online Learning* working through this for the OLI module, your course may standardize on one path — check the module instructions before installing everywhere.

---

## Install — pick your platform

All three platforms use the same files in this repo. The only difference is which files you upload where.

### ChatGPT

#### Option 1: ChatGPT Skills (Business, Enterprise, Edu, Teachers, Healthcare plans)

Easiest install if your plan supports it. The skill description triggers it on MCQ-related requests; the skill instructions tell ChatGPT to run the validator when code execution is available in the session.

1. Get the skill zip: download the [latest release](https://github.com/gautamyadavs/mcq-quality-coach/releases) (grab `mcq-quality-coach.zip`), or build it yourself by cloning and zipping the folder so it stays as the top-level entry inside the zip:

   ```bash
   git clone https://github.com/gautamyadavs/mcq-quality-coach.git
   zip -r mcq-quality-coach.zip mcq-quality-coach -x "mcq-quality-coach/.git/*"
   ```

2. In ChatGPT, click your profile icon → **Skills**.
3. Click **Add new skill** → **Upload from your computer** → select `mcq-quality-coach.zip`.
4. Open a new chat: *"Generate an MCQ for this learning objective: [your LO]"*

To explicitly invoke the skill, ask: *"Use the MCQ Quality Coach skill to generate an MCQ for…"*

> Skills are currently in beta and not yet available on ChatGPT Plus, Pro, or Free plans. If you're on one of those, use Option 2 below.

#### Option 2: Custom GPT (chatgpt.com — Plus/Pro, or for GPT Store publishing)

Best if you're on Plus/Pro and Skills aren't available yet, or if you want to publish publicly via the GPT Store.

Requires: ChatGPT Plus, Team, or Enterprise. Builder Profile configured (Settings → Builder profile) if publishing publicly.

1. Go to chatgpt.com → click your profile → **My GPTs** → **Create a GPT** → **Configure** tab.
2. **Name:** `MCQ Quality Coach`.
3. **Description:** `Generate, audit, and revise MCQs against the 19 item-writing flaw criteria. Aligns to your learning objective, produces plausible distractors, instructor rationales, learner-facing feedback, and misconception tags.`
4. **Instructions:** open [`instructions.md`](instructions.md), copy the entire contents, paste into the Instructions field. (Pre-trimmed to fit ~8K char limit.)
5. **Conversation starters:**
   - `Generate an MCQ from this learning objective: [paste here]`
   - `Review this MCQ against the 19 item-writing flaws and suggest a better version.`
   - `Show me a flawed MCQ and walk me through how you'd fix it.`
   - `Turn this draft item into an LMS-ready question with rationales and learner feedback.`
6. **Knowledge:** click **Upload files** → upload all three files from the [`references/`](references/) folder:
   - `19-iwf-rubric.md`
   - `blooms-targeting.md`
   - `before-after-examples.md`
7. **Capabilities:** turn ON code execution / data analysis (the exact label varies — it's the one that lets the GPT run Python). Turn off web browsing and image generation unless you specifically need them.
8. **Additional Settings:** if available, enable the option that prevents users from extracting the GPT's instructions. The exact label changes with OpenAI's UI updates.
9. Click **Save** → choose **Anyone with a link** for soft launch, or **Everyone** to publish publicly.

The validator script doesn't run automatically in Custom GPTs. The simplest workflow is to upload `scripts/validate.py` as an additional knowledge file, then ask the GPT: *"Run validate.py on this item and show me the report."*

#### Option 3: Codex CLI (developer terminal)

Best if you work in a terminal. Full skill support — the skill instructions tell Codex to run the validator when the session permits tool execution.

**Prerequisites:** Codex CLI installed (`npm install -g @openai/codex` or similar — see [Codex docs](https://developers.openai.com/codex/cli)). Signed in to your ChatGPT account or via API key.

```bash
git clone https://github.com/gautamyadavs/mcq-quality-coach.git
mkdir -p ~/.agents/skills
cp -r mcq-quality-coach ~/.agents/skills/
```

After installing, you should have `~/.agents/skills/mcq-quality-coach/SKILL.md`. Verify with:

```bash
ls ~/.agents/skills/mcq-quality-coach/
# Should show: SKILL.md, references/, scripts/, etc.
```

Restart Codex (or start a new session) and type `/skills` to confirm `mcq-quality-coach` is listed. Then test it:

> Generate an MCQ for this learning objective: Explain why retrieval practice strengthens long-term memory.

If the skill doesn't trigger, see [Codex skills troubleshooting](https://developers.openai.com/codex/skills).

### Claude

#### Option 1: Claude.ai (consumer chat)

Easiest install — the skill description triggers it on MCQ-related requests.

1. Get the skill zip: download the [latest release](https://github.com/gautamyadavs/mcq-quality-coach/releases), or build it yourself so the skill folder stays at the top level of the zip:

   ```bash
   git clone https://github.com/gautamyadavs/mcq-quality-coach.git
   zip -r mcq-quality-coach.zip mcq-quality-coach -x "mcq-quality-coach/.git/*"
   ```

2. In Claude.ai: **Settings → Capabilities → enable Code Execution**.
3. Go to **Customize → Skills → Upload skill** → upload `mcq-quality-coach.zip`.
4. Open a new chat: *"Generate an MCQ for this learning objective: [your LO]"*

When the skill is loaded and Code Execution is on, the skill instructs Claude to run the bundled validator script. If no validator output appears in the response, ask explicitly: *"Run the validator on this draft and show me the IWF audit."*

#### Option 2: Claude Code (developer terminal)

**Prerequisites:** [Claude Code installed](https://docs.claude.com/en/docs/agent-sdk/overview) and authenticated.

```bash
git clone https://github.com/gautamyadavs/mcq-quality-coach.git
mkdir -p ~/.claude/skills
cp -r mcq-quality-coach ~/.claude/skills/
```

After installing, you should have `~/.claude/skills/mcq-quality-coach/SKILL.md`. Start a new Claude Code session (existing sessions also pick this up automatically per [Claude Code docs](https://code.claude.com/docs/en/skills)) and test it:

> Generate an MCQ for this learning objective: Explain why retrieval practice strengthens long-term memory.

To verify the skill is loaded, run `/skills` inside Claude Code.

#### Option 3: Anthropic API

Custom skills can be uploaded via Anthropic's Skills API and used with the code-execution tool. The exact upload commands and required beta headers change as the API evolves — refer to Anthropic's current [Agent Skills API guide](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) for the up-to-date flow.

For the OLI module, students should use Option 1 (Claude.ai) or Option 2 (Claude Code) unless they're building a custom application.

### Gemini

#### Option 1: Gemini Gem (gemini.google.com)

Custom Gems require an eligible Google account tier. Check Google's [Gem help docs](https://support.google.com/gemini/answer/15235603) for current availability.

1. Go to gemini.google.com and create a new Gem (the entry point is currently in the left sidebar — Google occasionally renames it).
2. **Gem name:** `MCQ Quality Coach`.
3. **Instructions:** open [`instructions.md`](instructions.md), copy the entire contents, paste into the Instructions field.
4. **Knowledge** (if your tier supports file uploads in Gems): upload the three files from [`references/`](references/):
   - `19-iwf-rubric.md`
   - `blooms-targeting.md`
   - `before-after-examples.md`
   - If your tier doesn't expose Gem knowledge uploads, paste the contents of `19-iwf-rubric.md` directly at the end of the instructions field — Gemini's instruction limit is more generous than ChatGPT's.
5. Click **Preview** to test, then **Save**.
6. Optionally **Share** with specific people via the share menu.

The validator script doesn't run automatically in Gems. Upload `scripts/validate.py` as a knowledge file (if your Gem tier supports it), then ask Gemini: *"Run validate.py on this draft item and show me the report."* If your tier doesn't support code execution, run the validator locally instead — see [Running the validator standalone](#running-the-validator-standalone) below.

#### Option 2: Gemini API (developers)

```python
import google.generativeai as genai

with open("instructions.md") as f:
    system_instruction = f.read()

# Optionally append the rubric for guaranteed availability
with open("references/19-iwf-rubric.md") as f:
    system_instruction += "\n\n## Reference: 19-IWF Rubric\n\n" + f.read()

model = genai.GenerativeModel(
    "gemini-2.0-flash",
    system_instruction=system_instruction
)

response = model.generate_content(
    "Generate an MCQ for this learning objective: ..."
)
```

Run [`scripts/validate.py`](scripts/validate.py) separately in your pipeline and feed the output back to Gemini for revision.

---

## What's in the repo

```
mcq-quality-coach/
├── README.md                # This file
├── SKILL.md                 # Behavior spec for native Agent Skills (Claude / Codex CLI)
├── instructions.md          # Same behavior spec, trimmed to 8K (Custom GPT / Gemini Gem)
├── LICENSE
├── references/
│   ├── 19-iwf-rubric.md     # Full criterion definitions, examples, audit guidance
│   ├── blooms-targeting.md  # LO verb → Bloom's level mapping; stem-style guidance
│   └── before-after-examples.md  # 10 worked bad → revised MCQ pairs
├── scripts/
│   └── validate.py          # Deterministic IWF pre-check (Python, no dependencies)
└── assets/
    └── output-schema.json   # JSON schema for structured output
```

## The 19 item-writing flaw criteria

| # | Criterion | Caught by |
|---|-----------|-----------|
| 1 | Ambiguous wording | LLM |
| 2 | Implausible distractors | LLM |
| 3 | "None of the above" | Validator |
| 4 | Correct answer noticeably longest | Validator |
| 5 | Gratuitous information in stem | LLM |
| 6 | True/false-style options | Validator |
| 7 | Convergence cues | LLM |
| 8 | Logical clues between options | LLM |
| 9 | "All of the above" | Validator |
| 10 | Incomplete or fill-in-blank stem | Validator |
| 11 | Absolute terms (always, never) | Validator |
| 12 | Word repeats from stem to correct answer | Validator |
| 13 | Unfocused stem | LLM |
| 14 | Combination / K-type options | Validator |
| 15 | Grammatical cues / non-parallel | Validator |
| 16 | Numerical options out of sequence | Validator |
| 17 | Vague qualifiers (often, usually) | Validator |
| 18 | More than one defensible answer | LLM |
| 19 | Negative wording (NOT, EXCEPT) | Validator |

Per-criterion guidance with examples is in [`references/19-iwf-rubric.md`](references/19-iwf-rubric.md).

## What this skill is not

- **Not a replacement for SME review on high-stakes items.** The skill marks specialized topics as "Requires SME verification" regardless of audit results.
- **Not a psychometric validator.** Item discrimination, difficulty parameters, and IRT analysis require student response data.
- **Not a content authority.** Subject-matter accuracy depends on the underlying model's knowledge. For specialized domains, treat outputs as draft material requiring expert verification.

## Running the validator standalone

The Python validator works without any LLM:

```bash
echo '{
  "stem": "What process is engaged during retrieval practice?",
  "options": [
    "Recognition of familiar material",
    "Retrieval of information from long-term memory",
    "Encoding of new information",
    "Consolidation during sleep"
  ],
  "correct_index": 1
}' | python scripts/validate.py --stdin
```

Output is JSON. Useful for batch-validating existing question banks regardless of which LLM you used to generate them.

## Troubleshooting

**"The skill isn't triggering when I ask for an MCQ."**
Try invoking it explicitly. In ChatGPT, ask: *"Use the MCQ Quality Coach skill to generate an MCQ for…"* In Claude.ai or Claude Code, type `/skills` to see what's loaded — if `mcq-quality-coach` is missing, re-check that `SKILL.md` is at the top level of the uploaded folder. In Codex CLI, type `/skills` and confirm the skill is listed.

**"The Custom GPT is generating MCQs but skipping the audit table."**
Knowledge files may not have uploaded. Configure tab → Knowledge — confirm all three reference files are listed. Also confirm Code Interpreter is ON in Capabilities.

**"It's producing flawed items even with the rubric uploaded."**
Run the standalone validator (see section below) on the output to confirm. If the validator agrees the item is flawed, the model didn't run its self-audit step. Try invoking explicitly: *"Generate an MCQ for [LO] and walk me through the full 19-IWF audit."*

**"My GPT publish failed with a Usage Policies violation."**
The knowledge files have been sanitized to avoid known triggers, but the combination of "audit," "flaws," and "MCQ" can still trigger OpenAI's classifier. Try renaming the GPT to "Assessment Design Studio" temporarily and retrying.

**"How do I uninstall?"**
- Claude.ai / ChatGPT Skills: Settings → Customize → Skills → delete the skill.
- Custom GPT / Gemini Gem: delete the GPT/Gem from your account.
- Codex CLI: `rm -rf ~/.agents/skills/mcq-quality-coach`. Claude Code: `rm -rf ~/.claude/skills/mcq-quality-coach`.

## Roadmap

- [ ] Multilingual rubric examples (currently English only)
- [ ] Domain reference packs (medical, legal, technical certification)
- [ ] Stronger semantic checks via small embedding models for #2 plausibility and #7 convergence
- [ ] Batch-mode for generating aligned item sets across a learning objective taxonomy

## Contributing

Issues and PRs welcome. Particularly interested in false-positive reductions in the validator, domain-specific reference packs, and eval set contributions.

## License

[MIT](LICENSE) — use, fork, modify, sell. No restrictions.

## Acknowledgments

- **Tarrant et al. (2006)** — the foundational 19-criterion rubric.
- **Moore & Chen (2023)** — the SAQUET hybrid validation approach.
- **Downing (2005)** — empirical evidence on the impact of item-writing flaws.
- **Vegada et al. (2016)** — randomized comparison of three-, four-, and five-option MCQs; recommends three options as the evidence-based default for item quality and student time-on-task.

## References

- Tarrant M, Knierim A, Hayes SK, Ware J. The frequency of item writing flaws in multiple-choice questions used in high stakes nursing assessments. *Nurse Educ Today*. 2006 Dec;26(8):662-71. doi: 10.1016/j.nedt.2006.07.006. PMID: 17014932.
- Vegada B, Shukla A, Khilnani A, Charan J, Desai C. Comparison between three option, four option and five option multiple choice question tests for quality parameters: A randomized study. *Indian J Pharmacol*. 2016 Sep-Oct;48(5):571-575. doi: 10.4103/0253-7613.190757. PMID: 27721545; PMCID: PMC5051253.

## Citation

```
gautamyadavs. (2026). MCQ Quality Coach: An open-standard Agent Skill for
item-writing flaw detection across ChatGPT, Claude, and Gemini.
GitHub. https://github.com/gautamyadavs/mcq-quality-coach
```
