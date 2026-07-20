# CCA-F Exam Trainer

**A study tool for the Claude Certified Architect — Foundations (CCA-F) exam, built in public.**

This repo *is* how I study. Every question, cheat sheet, and drill mode in here is what I'm personally using. If you're also going for the CCA-F, or you just want to watch someone learn something hard in public — fork it, use it, tell me what's wrong with it.

**[→ Open the trainer](https://roh00t.github.io/Claude-Certified-Architect-Foundations/)**

---

## Why this exists

The CCA-F exam guide is public, but it's a PDF of task statements and 12 sample questions — not something you can drill against at 11pm. So I built a self-contained study system: a scenario-judgment bank modeled on the exam's real format, spaced repetition, timed full-length mocks, confidence calibration, and a running log of exactly where I'm weak.

I'm sharing it because:
1. **Accountability.** If my prep is public, I can't quietly skip the parts I don't want to do.
2. **It's useful to more than just me.** No reason to rebuild this from scratch.
3. **The build itself is worth seeing.** Cheat sheets extracted from official task statements, distractor patterns reverse-engineered from the sample questions, a whole "trap taxonomy" for how the exam baits you. That process is arguably more interesting than the exam.

## The four pages

| Page | What it is |
|---|---|
| **`index.html`** | The main trainer — 10 tabs, all the drill modes |
| **`focus-drill.html`** | 36 questions targeting specific weak objectives, confidence-gated |
| **`domainnotes.html`** | Long-form notes on the 5 exam domains |
| **`usecasesnotes.html`** | The 6 exam scenarios, read across all 5 domains |

All four are standalone HTML. No build step, no dependencies, no backend.

## What's in the trainer

| Tab | What it does |
|---|---|
| **Overview** | Domain weighting, the three ideas that run through every domain, a **study plan generated from your own exam date**, and a rolling weakness heatmap |
| **Cheat Sheets** | The 5 domains condensed from official task statements, plus a **Trap Map** — the 8 recurring ways the exam baits you |
| **Notes** | The two long-form reference pages (domains + use cases), confidence-tagged so you know what's grounded vs inferred |
| **Flashcards** | 36 cards on the facts the exam turns on — misses recycle until cleared |
| **Mock Exam** | **Due-for-review queue** (spaced repetition), **Focus Drill**, a full **60Q / 120-min mock** with exact domain weighting and auto-submit, an 18Q quick sitting, a 30Q Hard Mode, and a 12Q multi-select depth drill |
| **Resources** | The official source documents |
| **Full Sweep** | All **832 questions** in one run with instant feedback. Position and misses persist — stop, come back, drill only what you missed |
| **Recall Drill** | The 600-question foundational bank, filterable by course |
| **AI Mode** | Generates new scenario questions on demand via the Claude API |
| **Results** | Every sitting logged locally, broken down by domain, weakest domain called out |

**Question banks:** 220 judgment (60 scenario + 160 practice) · 600 recall · 30 hard mode · 12 multi-select · 36 focus drill.

Plus: **light/dark/system theming** across all four pages, keyboard shortcuts in drills (`A`–`D` pick, `1`–`3` confidence, `Enter` next), ARIA tab semantics, and per-question spaced repetition.

## Confidence calibration — the important bit

After you pick an answer, rate your confidence: **Guessing / Fairly sure / Certain**. The explanation stays hidden until you rate. That's deliberate — committing before you see the answer is the entire point.

At submit you get a calibration table, and the metric that actually matters:

> ⚠ **4 questions you were CERTAIN about and got wrong.**

That's the most dangerous category on the exam. You won't flag those on the day, because you don't know you don't know. They're surfaced first, in red, with the trap family that caught you. The report also discounts your lucky guesses — your score minus those is your real score.

## The length-bias finding

While drilling I noticed I could often pick the right answer **without reading it** — just by taking the longest option. So I measured it. In all **12 official CCA-F sample questions, the correct answer is the longest one.** That's the only official calibration that exists, and it points one direction.

But my own authored questions were far worse, for a specific reason. A real official question runs 137 / 120 / 119 / 129 characters — four substantive competing answers. One of mine ran 236 / 68 / 55 / 60. The defect wasn't verbose correct answers; it was **lazy distractors**. Anthropic writes four real answers. I was writing one real answer and three punchlines.

So I rewrote the authored questions, moving each correct answer's justification out of the option and into the explanation. That's what the difficulty picker switches between.

### Current state — measured, not assumed

| Bank | correct ÷ avg distractor | correct is longest |
|---|---|---|
| Scenario set (neutralised) | 1.13× | 28% |
| **Practice set (never neutralised)** | **1.14×** | **63%** ⚠️ |
| Hard mode (neutralised) | 1.07× | 33% |
| **Focus Drill (authored under the gate)** | **1.00×** | **25%** |
| *random chance* | *1.00×* | *25%* |

**Known gap:** only **60 of the 220** judgment questions have a length-neutralised option set. The other 160 — imported practice questions — fall back to original phrasing at *every* difficulty, **including Hard**. The whole judgment bank therefore sits at **54% correct-is-longest**, not the 35% I originally reported for the smaller 108-question bank. See [Known issues](#known-issues).

The Focus Drill is the only bank authored under an enforced gate from the start, and the only one at random baseline. Strategy check on it: *always pick longest* scores **19%**, *always pick shortest* **22%** — both below the 25% you'd get from guessing. No length signal left to exploit.

**The honest caveat:** n=12 is a tiny sample, and questions written to *teach* may run more verbose than live exam items. Use length as a last-resort tiebreaker when you're stuck and out of time. Never as a strategy.

## Authoring rules (if you add questions)

These exist because the bank drifted once already. Any question added should hold to them:

1. Single answer, exactly 4 options, exactly one correct.
2. **Options length-matched.** Justification belongs in the explanation, never in the option text.
3. Every distractor is a real failure mode a competent architect would consider, tagged with its trap family. No strawmen.
4. The scenario states a concrete production symptom; the question asks for the fix targeting the **root cause**.
5. No option is defeatable on style alone — no odd-one-out phrasing, no single option naming a specific API.
6. The explanation says why the correct option wins **and** why the most tempting distractor loses.

**Acceptance gate** — measure before merging:

| Metric | Threshold |
|---|---|
| correct ÷ avg-distractor length | 0.95 – 1.05 |
| correct is longest | ≤ 32% |
| correct is shortest | ≤ 32% (don't invert the tell) |
| answer position A/B/C/D | none above ~35% |
| *always-pick-longest* strategy | below 25% |

## Known issues

| | Issue |
|---|---|
| 🔴 | **160 of 220 judgment questions were never length-neutralised** — they show original phrasing even on Hard, putting the bank at 54% correct-is-longest. Fixing means authoring a neutralised option set for those 160. |
| 🟡 | **Results timestamps render in UTC, not local** (three `toISOString()` sites in `index.html`). Cosmetic — no scoring impact — but sitting times display shifted. |
| 🟡 | **Focus Drill trap distribution is skewed** — wrong-problem is 50% of all distractors; heuristic-guess only 5%. |
| 🟢 | **Judgment bank skews D3 +7 points** over the 20% target. Mocks still *draw* to correct weighting, so scores aren't distorted — only free-form drilling over-exposes D3. |

## Your data

Everything is `localStorage` on your own device — no backend, no account, nothing leaves your browser.

| Key | Holds |
|---|---|
| `ccaf-history` | Sitting history |
| `ccaf-q-stats` | Per-question stats + spaced-repetition schedule |
| `ccaf-sweep-v1` | Full Sweep position and miss bank |
| `ccaf-focus-v1` | Focus Drill progress |
| `ccaf-exam-v1` | Your exam date |
| `ccaf-theme` | Light / dark / system (shared across all pages) |
| `ccaf-diff` · `ccaf-anthropic-key` | Difficulty · your API key |

Per-question stats are keyed by a **hash of the question text**, not its position — so your review history survives the bank being reordered or added to.

## A note on what this is *not*

There is no leaked or real exam content anywhere in this repo. The Anthropic Certification Exam Policy prohibits obtaining or distributing real exam questions — doing so risks disqualification and a ban, for me and for you. Everything here is built from:

- The 12 **official sample questions** in Anthropic's public CCA-F exam guide
- Questions **authored from the guide's public task statements**
- Practice sets I wrote myself while working through the material
- A 600-question recall bank covering the four public Anthropic Academy courses

Some questions cover the same *concepts* as material encountered elsewhere, but every scenario, number, and option was written from scratch. If you've found real exam content somewhere, please don't send it to me — I don't want it, and neither should you.

## Setting up AI Mode (optional)

Everything except **AI Mode** and **Explain like I'm 5** works the moment you open the page — no setup, no account, nothing installed. Those two call the Claude API from your browser:

1. Get a key at [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys) (pay-as-you-go — a fraction of a cent per question).
2. Open **AI Mode**, paste it, click **Save key**.

**Where your key goes:** nowhere but your own browser (`localStorage`) and `api.anthropic.com`. Never sent to me, never logged, never committed. Don't paste it into a commit, a screenshot, or a chat with anyone — including me.

The Focus Drill and both notes pages make **no network calls at all.**

## Deploying your own copy

```bash
git clone https://github.com/Roh00t/Claude-Certified-Architect-Foundations.git
cd Claude-Certified-Architect-Foundations
git add .
git commit -m "CCA-F exam trainer"
git push
```

Then: **Settings → Pages → Source: Deploy from a branch → Branch: main / (root) → Save.**

Open the trainer and **set your exam date** on the Overview tab — the study plan builds itself backward from whatever date you enter, so it never goes stale.

## Contributing

Found a question that's wrong, a rationale that's unclear, or a distractor that's an obvious strawman? Open an issue or a PR — the strawman-distractor problem above is exactly what I want caught. If you're studying for CCA-F and want to add questions **you wrote yourself**, I'd genuinely welcome them. Please hold them to the authoring rules above.

---

*Built conversationally with Claude. Not affiliated with or endorsed by Anthropic — just a study tool built from their public exam guide.*
