# focus-drill.html — working notes

Scratchpad for the post-exam remediation drill. Tracks objectives covered,
question counts, decisions needing verification, and open questions.

**Status:** Phase 1 (recon) complete — awaiting approval of Phase 2 (question bank).

---

## Integrity position (restated, non-negotiable)

- No real exam content is available to me, and none was provided. Every question
  in this bank is **newly authored** to target a published objective.
- The 11 objectives below come from the user's own score report (objective names
  only, no items). Writing new scenarios against a syllabus objective is
  legitimate; reproducing or reconstructing a real item is not, and is not done.
- Additional self-imposed check: new questions are also cross-checked against the
  **existing 220-question bank in index.html** so this drill isn't a reshuffle of
  questions already owned.

---

## Baseline — index.html must remain byte-identical

```
sha256  5a1b5f185457b652bb5a4428100513e75aaf7133a02865ee58b18dea9a2e0f06
lines   1558
arrays  SCEN=60 PRACT=160 MULTI=12 HARD=30 RECALL=600
git     060e087 (clean tree, Phases B/C/E committed)
```
Re-verify this hash at the end. Any change = failure.

---

## Phase 1 recon findings

### Data shape to mirror
`SCEN`/`PRACT`/`HARD` records are:
`{d, src, s, q, o[4], a, e}` plus optional `oh[4]` (length-neutralised option set;
`optsFor()` picks `oh` unless difficulty is "easy").

**Focus-drill deviates deliberately** — it groups by *objective*, not domain:
```js
{ obj:1, tier:1, q:"…", o:[4], a:0, e:"…", eli5:"…", traps:["OE","PH","HG"] }
```
- `obj` 1–11 replaces `d` 1–5 (domain is the wrong axis for this drill).
- `traps[]` is per-distractor, indexed to match `o` skipping the correct index.
- No `oh` — options are authored length-neutral from the start, so there is only
  one option set and no difficulty toggle (per user instruction).

### Length-tell measurement (the defect being corrected)
| Bank | correct ÷ avg distractor | correct is longest |
|---|---|---|
| SCEN | 1.13× | 28% |
| **PRACT** | **1.14×** | **63%** |
| HARD | 1.07× | 33% |
| random baseline | 1.00× | 25% |

**Acceptance gate for the new bank (enforced by script before UI is built):**
- correct ÷ avg-distractor ratio in **0.95–1.05**
- correct-is-longest in **≤ 32%** of questions
- correct-is-shortest also **≤ 32%** (avoid inverting the tell)
- no single option position holds the answer more than ~35% of the time

### UX patterns to reuse verbatim
- **Confidence gate:** `revealed = picked !== null && conf !== null`.
  `CONF_LABELS = {1:'Guessing', 2:'Fairly sure', 3:'Certain'}`.
- **Calibration bands:** Certain ≥90%, Fairly sure 70–85%, Guessing 30–50%.
  "CERTAIN + wrong" renders in a red `.danger` box; lucky guesses discounted from
  the honest score.
- **Trap codes (reuse exactly):** `PH` prompt-hope, `OE` over-engineering,
  `NF` nonexistent feature, `SS` silent suppression, `DP` disproportionate,
  `HG` heuristic-guess, `BS` burden-shift, `WP` wrong-problem.
- **Design tokens:** copy `:root` + `:root[data-theme="dark"]` from index.html
  verbatim so both pages look identical in both themes.
- **Storage:** all keys `ccaf-` prefixed, **every access wrapped in try/catch**
  (private mode must degrade, not crash).

---

## Decisions made — please verify these

1. **ELI5 must be authored, not generated.** index.html's "Explain like I'm 5"
   button calls `api.anthropic.com` with the user's own API key
   (`eli5For()`, line 981). That violates the no-external-calls / GitHub-Pages
   constraint for this file, so every question carries a **static authored
   `eli5` string**. No network calls of any kind in focus-drill.html.
2. **Theme key is shared, not duplicated.** focus-drill reuses `ccaf-theme`
   (same origin) so the light/dark choice carries across both pages.
3. **New storage key:** `ccaf-focus-v1`. Checked against existing keys
   (`ccaf-history`, `ccaf-sweep-v1`, `ccaf-q-stats`, `ccaf-diff`, `ccaf-theme`,
   `ccaf-exam-v1`, `ccaf-anthropic-key`) — no collision.
4. **Google Fonts:** index.html loads Archivo/Inter/IBM Plex Mono from the
   Google CDN. To match the look, focus-drill does the same, **with a full
   system-font fallback stack** so it still renders correctly offline or if the
   CDN is blocked. This is a styling request, not a backend/data call.
   → *Say the word if you'd rather it be 100% self-contained on system fonts.*
5. **No SKILL.md.** The authoring rules live in this file (below) instead —
   one scratchpad rather than two files in the repo. Reversible if you prefer.

---

## Question-authoring rules (applied to every item)

1. Single answer, exactly 4 options, exactly one correct.
2. **Options length-matched.** Justification belongs in `e`, never in the option
   text. Correct answer must not be the longest or the most specific-sounding.
3. Every distractor is a real failure mode a competent architect would consider,
   tagged with its trap family. No strawmen, no joke options.
4. The scenario states a concrete production symptom; the question asks for the
   fix that targets the **root cause**.
5. Distractors must not be defeatable by pure elimination-on-style (no option is
   the only one mentioning a specific API, no odd-one-out phrasing).
6. `e` explains why the correct option wins **and** why the most tempting
   distractor loses.
7. Cross-check every stem against the existing 220-question bank for near
   duplicates before inclusion.

---

## Objective coverage plan (target 36)

### Tier 1 — scored 0% · 4 questions each = 28
| # | Objective | Target | Authored |
|---|---|---|---|
| 1 | Parallel tool_use blocks in one assistant response vs sequential turns | 4 | 0 |
| 2 | Session resumption: `--resume`, targeted re-analysis, context injection | 4 | 0 |
| 3 | State persistence for interrupted multi-agent pipelines | 4 | 0 |
| 4 | Orchestration selection: coordinator-worker vs parallel vs sequential | 4 | 0 |
| 5 | Subagent output schema: prose vs structured vs citation metadata | 4 | 0 |
| 6 | Synthesis preserving source-level uncertainty | 4 | 0 |
| 7 | Context optimization: summarize / sliding window / structured state / selective retention | 4 | 0 |

### Tier 2 — scored 33–50% · 2 questions each = 8
| # | Objective | Target | Authored |
|---|---|---|---|
| 8 | Subagent prompts carrying complete findings + source metadata | 2 | 0 |
| 9 | Goal-oriented vs procedural delegation | 2 | 0 |
| 10 | Extraction schemas: optional/nullable + enums, no fabrication | 2 | 0 |
| 11 | Cross-session context: subagent isolation, scratchpads, targeted reads | 2 | 0 |

**Total: 36**

---

## Open questions for the user

- **O1.** Objectives 2, 3, and 11 overlap around "resume work without repeating
  it." Should questions stay tightly scoped to each objective's distinct
  mechanism (my plan), or is some deliberate overlap useful for reinforcement?
- **O2.** Overlap with the existing bank: objectives 1, 2, 8 are touched by a few
  existing questions. Plan is to author genuinely new scenarios and flag any that
  land close to an existing item, for your call — same as the earlier audit.
- **O3.** Should the drill offer a shuffled order each run, or a fixed order so
  progress across sessions is comparable? Default: shuffled, with objective
  grouping preserved in the report.

---

## Changelog
- **Phase 1** — recon complete; baseline hash recorded; length-tell quantified;
  schema, calibration, trap taxonomy, and storage patterns captured.
- **Phase 2** — 36 questions authored and approved. Gate caught two defects I
  introduced and both were fixed: answer position was A:100% (seeded shuffle →
  9/9/9/9) and correct-is-shortest hit 47% (nine options nudged → 28%).
  No near-duplicates vs the existing 250 judgment stems (max Jaccard 0.25).
- **Phase 3** — `focus-drill.html` built (76 KB, standalone). Bank injected
  programmatically from `bank.json`, not hand-transcribed.
- **Phase 4** — 38/38 headless tests pass, 0 runtime errors. index.html verified
  byte-identical (`5a1b5f18…`).

---

## Final gate results (recorded for future re-authoring)

| Check | Result | Gate |
|---|---|---|
| correct ÷ avg-distractor | 1.003 | 0.95–1.05 ✅ |
| correct is longest | 25% | ≤32% ✅ |
| correct is shortest | 28% | ≤32% ✅ |
| answer position | A9 B9 C9 D9 | none >35% ✅ |
| always-pick-longest scores | **19%** | <25% ✅ |
| always-pick-shortest scores | **22%** | <25% ✅ |
| *(index.html PRACT, for contrast)* | *63% longest* | *— fails* |

**Trap family distribution:** WP 54, OE 11, SS 10, PH 8, NF 7, DP 7, BS 6, HG 5
(of 108 distractors). WP at 50% is skewed — flagged to the user, left as-is
pending a decision on rebalancing toward PH/HG/DP.

## Deferred / open

- **Trap rebalance** — WP is half of all distractors. Offered; not yet actioned.
- The Mock Exam launch card snippet was supplied to the user separately; **this
  agent did not edit index.html** and must not.
- `focus-drill-bank-review.md` is an audit artefact. Safe to delete once the
  bank is settled; it is regenerable from the embedded `BANK` array.
