# MCQ Quality Coach

> An open-standard Agent Skill for ChatGPT, Claude, and Gemini that writes multiple-choice questions that don't suck.

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

- A bundled Python validator catches 12 of 19 IWFs that pattern-match cleanly (length cues, NOTA/AOTA, negation words, K-type combinations, numerical order, word repeats, etc.)
- The LLM handles the 5 that require semantic judgment (ambiguity, distractor plausibility, convergence cues, unfocused stems, multiple defensible answers)

The validator runs **automatically** in environments that support the Agent Skills standard (Claude.ai, Claude Code, ChatGPT Codex CLI). In Custom GPTs and Gemini Gems, you can still run it manually — it's just not bundled automatically.

---

## Install — pick your platform

All three platforms use the same files in this repo. The only difference is which files you upload where.

### ChatGPT

#### Option 1: ChatGPT Skills (Business, Enterprise, Edu, Teachers, Healthcare plans)

Easiest install if your plan supports it. Auto-triggers on MCQ requests, validator runs via Code Interpreter.

1. Download the [latest release](https://github.com/gautamyadavs/mcq-quality-coach/releases) — grab `mcq-quality-coach.zip`.
2. In ChatGPT, click your profile icon → **Skills**.
3. Click **Add new skill** → **Upload from your computer** → select the zip.
4. Open a new chat: *"Generate an MCQ for this learning objective: [your LO]"*

The skill auto-triggers based on its description, or you can invoke it explicitly with `@MCQ Quality Coach`.

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
7. **Capabilities:** turn ON **Code Interpreter**. Turn OFF Web Browsing and DALL·E unless needed.
8. **Additional Settings:** turn ON **Disable instruction retrieval** to protect your prompt.
9. Click **Save** → choose **Anyone with a link** for soft launch, or **Everyone** to publish publicly.

The validator script doesn't auto-run in Custom GPTs. Code Interpreter can run it on request — paste the contents of [`scripts/validate.py`](scripts/validate.py) into a chat and ask the GPT to run it on a specific item.

#### Option 3: Codex CLI (developer terminal)

Best if you work in a terminal. Full skill support including auto-running validator.

```bash
git clone https://github.com/gautamyadavs/mcq-quality-coach.git
mkdir -p ~/.codex/skills
cp -r mcq-quality-coach ~/.codex/skills/
```

Codex CLI auto-discovers the skill on next launch. Behavior is identical to Claude Code.

### Claude

#### Option 1: Claude.ai (consumer chat)

Easiest install — auto-triggers on MCQ requests.

1. Download the [latest release](https://github.com/gautamyadavs/mcq-quality-coach/releases) — grab `mcq-quality-coach.zip`.
2. In Claude.ai: **Settings → Capabilities → enable Code Execution**.
3. Go to **Customize → Skills → Upload skill** → upload the zip.
4. Open a new chat: *"Generate an MCQ for this learning objective: [your LO]"*

The skill auto-triggers on MCQ-related prompts. Validator runs automatically.

#### Option 2: Claude Code (developer terminal)

```bash
git clone https://github.com/gautamyadavs/mcq-quality-coach.git
mkdir -p ~/.claude/skills
cp -r mcq-quality-coach ~/.claude/skills/
```

#### Option 3: Anthropic API

Upload the skill folder via the [Skills API](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) and reference it in your `tools` parameter alongside `code_execution`.

### Gemini

#### Option 1: Gemini Gem (gemini.google.com)

Requires: Gemini Advanced (Google One AI Premium).

1. Go to gemini.google.com → click **Gem manager** in the left sidebar → **+ New Gem**.
2. **Gem name:** `MCQ Quality Coach`.
3. **Instructions:** open [`instructions.md`](instructions.md), copy the entire contents, paste into the Instructions field.
4. **Knowledge** (if your tier supports file uploads in Gems): upload the three files from [`references/`](references/):
   - `19-iwf-rubric.md`
   - `blooms-targeting.md`
   - `before-after-examples.md`
   - If your tier doesn't expose Gem knowledge uploads, paste the contents of `19-iwf-rubric.md` directly at the end of the instructions field — Gemini's instruction limit is more generous than ChatGPT's.
5. Click **Preview** to test, then **Save**.
6. Optionally **Share** with specific people via the share menu.

The validator script doesn't auto-run in Gems. Gemini's Code Execution tool can run it on request — paste the contents of [`scripts/validate.py`](scripts/validate.py) into a chat and ask Gemini to run it on a specific item.

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

## Contributing

Issues and PRs welcome. Particularly interested in false-positive reductions in the validator, domain-specific reference packs, and eval set contributions.

## License

[MIT](LICENSE) — use, fork, modify, sell. No restrictions.

## Acknowledgments

- Tarrant et al. (2006) — the foundational 19-criterion rubric.
- Moore & Chen (2023) — the SAQUET hybrid validation approach.
- Downing (2005) — empirical evidence on the impact of item-writing flaws.

## Citation

```
Gautam Yadav. (2026). MCQ Quality Coach: An open-standard Agent Skill for
item-writing flaw detection across ChatGPT, Claude, and Gemini.
GitHub. https://github.com/gautamyadavs/mcq-quality-coach
```
