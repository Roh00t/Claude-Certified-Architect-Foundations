# CCA-F Exam Trainer

**I'm sitting the Claude Certified Architect — Foundations (CCA-F) exam on Sunday, 19 July 2026 at 19:00 SGT — and I'm building my study tool in public.**

This repo *is* how I'm studying. Every question, cheat sheet, and drill mode in here is what I'm personally using to prepare. If you're also going for the CCA-F, or you just want to watch someone learn something hard in public, follow along — fork it, use it, tell me what's wrong with it.

**[→ Open the trainer](https://roh00t.github.io/Claude-Certified-Architect-Foundations/)**

---

## Why this exists

The CCA-F exam guide is public, but it's a PDF of task statements and 12 sample questions — not something you can drill against at 11pm the night before. So I built a single self-contained HTML file that turns the exam guide into an actual study system: a scenario-judgment question bank modeled on the exam's real format, spaced-repetition flashcards, timed full-length mocks, and a running log of exactly where I'm weak.

I'm sharing it because:
1. **Accountability.** If my prep is public, I can't quietly skip the parts I don't want to do.
2. **It's useful to more than just me.** If you're prepping for the same exam, there's no reason to rebuild this from scratch.
3. **The build itself is worth seeing.** This was built conversationally with Claude — cheat sheets extracted from the official task statements, distractor patterns reverse-engineered from the sample questions, a whole "trap taxonomy" for how the exam tricks you into the wrong answer. That process is arguably more interesting than the exam itself.

## The length-bias finding

While drilling, I noticed I could often pick the right answer **without reading it properly** — just by choosing the longest option. So I measured it across every question in the bank. The result was uncomfortable:

| Question source | Correct answer is the longest option | Correct ÷ avg distractor length |
|---|---|---|
| **Anthropic's 12 official sample questions** | **12 / 12 — 100%** | 1.50× |
| My authored questions | 44 / 48 — 92% | 2.20× |
| My "hard mode" questions | 29 / 30 — 97% | 1.71× |
| My hand-written practice set | 21 / 48 — 44% | 1.09× |
| Recall bank (course material) | 479 / 600 — 80% | 2.49× |
| *(random chance)* | *25%* | *1.00×* |

Two things fell out of this:

**The tell is real in the official material.** In all 12 published CCA-F sample questions, the correct answer is the longest one. That's the only official evidence that exists, and it points one direction.

**But my own questions were far worse than the real thing**, and for a specific reason. Look at a real official question — 137 / 120 / 119 / 129 characters. Four substantive, competing answers; correct is longest by 11%. Now look at one of mine — 236 / 68 / 55 / 60. My correct answer was an essay and my distractors were one-line strawmen. The defect wasn't verbose correct answers; it was **lazy distractors**. Anthropic writes four real answers. I was writing one real answer and three punchlines.

So I rewrote 62 questions, moving each correct answer's justification clause out of the option and into the explanation where it belongs. Across the whole 108-question judgment bank that takes the tell from **1.63× / longest 71% of the time** to **1.11× / 35%**.

**The 12 official questions are left untouched, verbatim.** They're the only real calibration that exists — rewriting them would mean you're no longer practising against the actual published samples. They keep their tell at every difficulty. That's why the number is 35% and not 25%.

**The honest caveat:** n=12 is a tiny sample, and questions written for a study guide are written to *teach*, so they may run more verbose than live exam items. Use length as a last-resort tiebreaker when you're stuck and out of time. Never as a strategy.

## Difficulty levels

The difficulty picker sits at the top of the **Mock Exam** tab and applies to every sitting, plus Full Sweep.

| | Options | Hints |
|---|---|---|
| **Easy** | Original phrasing — correct answer usually longest (1.63×) | Domain + scenario pills shown |
| **Medium** | Length-neutralised — correct answer no longer stands out (1.11×) | Domain + scenario pills shown |
| **Hard** | Length-neutralised | **No domain or scenario hints** — just the scenario and four comparable answers |

Difficulty changes *how obvious the correct answer is*, not which questions you get. Start on Medium. Use Easy only for first exposure to a topic — it will flatter you.

## Confidence calibration

After you pick an answer, rate how confident you are: **Guessing / Fairly sure / Certain**. In drill modes the explanation stays hidden until you rate — that's deliberate; committing to a confidence level before seeing the answer is the whole point.

At submit you get a calibration table, and the metric that actually matters:

> ⚠ **4 questions you were CERTAIN about and got wrong.**

That's the most dangerous category in the entire exam. You won't flag those on exam day, because you don't know you don't know. A "Review only my mistakes" button surfaces them first, outlined in red. The report also tells you how many you guessed correctly — your score minus the lucky guesses is your real score.

## What's in the trainer

| Feature | What it does |
|---|---|
| **Overview** | Domain weighting, the "three ideas that run through every domain," and my day-by-day countdown plan to 19 July |
| **Cheat Sheets** | The five exam domains condensed from the official task statements, plus a **Trap Map** — the 8 recurring ways the exam baits you into a plausible-but-wrong answer |
| **Flashcards** | 36 cards on the facts the exam turns on, self-graded — misses recycle until you clear them |
| **Mock Exam** | A **full 60-question / 120-minute mock** with exact domain weighting, question flagging, and auto-submit. Plus an 18-question quick sitting, a 30-question Hard Mode, and a 12-question multi-select depth drill |
| **Resources** | The three official source PDFs — Exam Guide, Exam Policy, Certification T&Cs |
| **Full Sweep** | All **720 questions** in one continuous run with instant feedback. Position and misses saved — stop, come back, then drill only what you got wrong |
| **Recall Drill** | The 600-question foundational bank, filterable by course |
| **AI Mode** | Generates brand-new scenario questions on demand via the Claude API |
| **Explain like I'm 5** | On any explanation, rewrites it in plain language with an everyday analogy. Works on all 720 questions (needs an API key) |
| **Results** | Every sitting logged locally, broken down by domain, with your weakest domain called out |

## A note on what this is *not*

There is no leaked or real exam content anywhere in this repo. The Anthropic Certification Exam Policy explicitly prohibits obtaining or distributing real exam questions — doing so risks disqualification and a ban from the certification program, for me and for you. Everything here is built from:

- The 12 **official sample questions** published in Anthropic's public CCA-F exam guide
- Questions **authored from the guide's task statements** (the public domain/skill breakdown)
- A **practice set** I wrote myself while working through the material
- A 600-question recall bank covering the four public Anthropic Academy courses

Some questions here cover the same *concepts* as material I've encountered elsewhere, but every scenario, number, and option was written from scratch for this repo. If you've found real exam content somewhere, please don't send it to me — I don't want it, and neither should you.

## Setting up AI Mode (optional)

Everything except **AI Mode** and **Explain like I'm 5** works the moment you open the page — no setup, no account, nothing installed. Those two call the Claude API directly from your browser:

1. Get an API key from [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys) (pay-as-you-go — a fraction of a cent per question).
2. Open **AI mode**, paste your key, click **Save key**.

**Where your key goes:** nowhere but your own browser (`localStorage`) and `api.anthropic.com`. It's never sent to me, never logged, never committed. Don't paste it into a commit, a screenshot, or a chat with anyone — including me.

Everyone brings their own key. There's no shared key and no backend — if you skip this step, every other feature still works normally.

## Deploying your own copy

Single static `index.html` — no build step, no dependencies, no backend.

```bash
git clone https://github.com/Roh00t/Claude-Certified-Architect-Foundations.git
cd Claude-Certified-Architect-Foundations
# index.html, README.md, LICENSE, and resources/ are already here
git add .
git commit -m "CCA-F exam trainer"
git push
```

Then: **Settings → Pages → Source: Deploy from a branch → Branch: main / (root) → Save.**

## Contributing

Found a question that's wrong, a rationale that's unclear, or a distractor that's an obvious strawman? Open an issue or a PR — the strawman-distractor problem above is exactly the kind of thing I want caught. If you're also studying for CCA-F and want to add questions **you wrote yourself**, I'd genuinely welcome them.

## Follow the prep

I'll be updating my sitting history and domain scores as I go. If you want to compare notes, benchmark your own prep against mine, or heckle me if my mock scores are bad — that's the whole point of doing this in public.

**Exam day: Sunday, 19 July 2026, 19:00 SGT.**

---

*Built conversationally with Claude. Not affiliated with or endorsed by Anthropic — just a study tool built from their public exam guide.*