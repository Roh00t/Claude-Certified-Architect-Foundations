# Claude-Certified-Architect-Foundations
My CCA-F journey
# CCA-F Exam Trainer

**I'm sitting the Claude Certified Architect — Foundations (CCA-F) exam on Sunday, 19 July 2026, 19:00 SGT — and I'm building my study tool in public.**

This repo *is* how I'm studying. Every question, cheat sheet, and drill mode in here is what I'm personally using to prepare. If you're also going for the CCA-F, or you just want to watch someone learn something hard in public, follow along — fork it, use it, tell me what's wrong with it.

**[→ Open the trainer](https://YOUR-USERNAME.github.io/YOUR-REPO/)** *(live once GitHub Pages is on — see Deploy below)*

---

## Why this exists

The CCA-F exam guide is public, but it's a PDF of task statements and 12 sample questions — not something you can drill against at 11pm the night before. So I built a single self-contained HTML file that turns the exam guide into an actual study system: a scenario-judgment question bank modeled on the exam's real format, spaced-repetition flashcards, timed full-length mocks, and a running log of exactly where I'm weak.

I'm sharing it because:
1. **Accountability.** If my prep is public, I can't quietly skip the parts I don't want to do.
2. **It's useful to more than just me.** If you're prepping for the same exam, there's no reason to rebuild this from scratch.
3. **The build itself is worth seeing.** This was built conversationally with Claude — cheat sheets extracted from the official task statements, distractor patterns reverse-engineered from the sample questions, a whole "trap taxonomy" for how the exam tricks you into the wrong answer. That process is arguably more interesting than the exam itself.

## What's in the trainer

| Feature | What it does |
|---|---|
| **Overview** | Domain weighting, the "three ideas that run through every domain," and my actual day-by-day countdown plan to 19 July |
| **Cheat Sheets** | The five exam domains condensed from the official task statements, plus a **Trap Map** — the 8 recurring ways the exam baits you into a plausible-but-wrong answer |
| **Flashcards** | 36 cards on the facts the exam turns on, with a self-grading flip deck — misses recycle until you clear them |
| **Mock Exam** | A **full 60-question / 120-minute mock** with exact domain weighting, question flagging, and auto-submit — the real exam format. Plus an 18-question quick sitting (timed or untimed-with-rationale), and a 12-question multi-select depth drill |
| **Resources** | The three official source PDFs, mirrored for convenience: the full CCA-F Exam Guide, the Certification Exam Policy, and the Certification Terms & Conditions — plus links to Anthropic Academy, the API console, and the docs |
| **Full Sweep** | Every single question in the trainer — 108 scenario-judgment questions + 12 multi-select + 600 recall questions = **720 questions** — in one continuous run with instant feedback. Your position and every miss are saved so you can stop, come back, and drill only what you got wrong |
| **Recall Drill** | The 600-question foundational bank (from the four Anthropic Academy courses), filterable by course |
| **AI Mode** | Generates brand-new, never-seen scenario questions on demand using the Claude API — see setup below |
| **Results** | Every sitting logged locally, broken down by domain, with your weakest domain called out |

## A note on what this is *not*

There is no leaked or real exam content anywhere in this repo. The [Anthropic Certification Exam Policy](https://www.anthropic.com/legal) explicitly prohibits obtaining or distributing real exam questions — doing so risks disqualification and a ban from the certification program, for me and for you. Everything here is built from:

- The 12 **official sample questions** published in Anthropic's public CCA-F exam guide
- Questions **authored from the guide's task statements** (the actual domain/skill breakdown Anthropic publishes)
- A **practice question set** I wrote myself while working through the material, styled to match the exam's format
- A 600-question recall bank covering the four public Anthropic Academy courses

If you've found real exam content somewhere, please don't send it to me — I don't want it, and neither should you.

## Setting up AI Mode (optional)

Everything except AI Mode works the moment you open the page — no setup, no accounts, nothing installed. AI Mode is the one feature that needs a small amount of configuration, because generating new questions requires calling the Claude API directly from your browser:

1. Get an API key from [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys) (pay-as-you-go — each generated question costs a small fraction of a cent).
2. Open the **AI mode** tab in the trainer, paste your key into the box, click **Save key**.
3. Generate away.

**Where your key goes:** nowhere but your own browser (`localStorage`, scoped to this page) and `api.anthropic.com`. It is never sent to me, never logged, never committed to this repo. Don't paste your key into a commit, a screenshot, or a chat message with anyone — including me.

This design (a static site calling the Anthropic API straight from the browser) only works because Anthropic's API supports direct browser access for exactly this kind of use case. It's genuinely your key, your usage, your bill — small as it is.

## Deploying your own copy to GitHub Pages

This is a single static `index.html` file — no build step, no dependencies, no backend.

```bash
git clone https://github.com/roh00t/.git
cd YOUR-REPO
# index.html, README.md, LICENSE, and resources/ (3 PDFs) are already here
git add .
git commit -m "CCA-F exam trainer"
git push
```

Then in your repo: **Settings → Pages → Source: Deploy from a branch → Branch: main / (root) → Save.**

GitHub will give you a URL like `https://YOUR-USERNAME.github.io/Claude-Certified-Architect-Foundations/` within a minute or two. That's it — no CI, no build pipeline, nothing else to configure.

## Contributing

Found a question that's wrong, a rationale that's unclear, or a trap-tag that's miscategorized? Open an issue or a PR. If you're also studying for CCA-F and want to add questions you've written yourself (not real exam content — see above), I'd genuinely welcome them.

## Follow the prep

I'll be updating my sitting history and domain scores as I go. If you want to compare notes, benchmark your own prep against mine, or just heckle me if my mock scores are bad — that's the whole point of doing this in public.

**Exam day: Sunday, 19 July 2026, 19:00 SGT. Let's see how it goes.**

---

*Built conversationally with Claude. Not affiliated with or endorsed by Anthropic — just a study tool built from their public exam guide.*