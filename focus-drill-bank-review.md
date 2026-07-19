# focus-drill — question bank for audit (Phase 2)

**36 questions · 28 Tier-1 · 8 Tier-2.** Correct answer marked ✅. Each distractor tagged with its trap family.

Gate results: ratio 1.003 · correct-longest 25% · correct-shortest 28% · positions A9/B9/C9/D9 · always-pick-shortest scores 22%, always-pick-longest 19% (random 25%). No near-duplicates vs the existing 220-question bank (max similarity 0.25).

---


## Objective 1 — Tier 1
**Parallel tool_use blocks in one response vs sequential turns**

### Q0 · obj1

A research agent needs the current headcount, the latest funding round, and the CEO name for one company. Each lives behind a separate read-only tool, and none of the three needs any output from the others. Logs show the agent spends three full model round trips gathering them, and p95 latency is now breaching the product's budget. What change fixes the latency at its source?

- 　 **A.** Move the three lookups behind one aggregate tool that the vendor team would need to build  
  _over-engineering_
- 　 **B.** Raise the per-request max_tokens ceiling so the agent has room to request more work at once  
  _wrong-problem_
- ✅ **C.** Emit all three tool_use blocks in a single assistant response so the harness runs them together  
  **CORRECT**
- 　 **D.** Keep the three turns but switch the orchestrating model to a faster, cheaper tier to cut latency  
  _wrong-problem_

**Why:** The three calls are independent, so the round trips are pure overhead. Claude can emit several tool_use blocks in one assistant message; the harness executes them concurrently and returns all tool_result blocks together, collapsing three round trips into one. max_tokens governs output length, not how many turns occur. Building an aggregate tool solves the latency but adds a service to own for a problem the protocol already handles. A faster model shrinks each turn but leaves three of them.

**ELI5:** You need milk, bread, and eggs. Right now you walk to the shop, get milk, come home, walk back for bread, come home, walk back for eggs. Nothing stops you getting all three in one trip — so ask for all three at once.

---
### Q1 · obj1

An agent must fetch an order record, then issue a refund against the account id that the order record returns. An engineer notices both tools are called in separate turns and proposes emitting both tool_use blocks in one response to halve the latency. In staging the refunds start failing with invalid-account errors. What is the correct read of this situation?

- 　 **A.** The parallel calls raced, so the refund tool needs an internal retry with backoff added  
  _wrong-problem_
- 　 **B.** Parallel execution is fine here once both tools are marked idempotent in their definitions  
  _nonexistent-feature_
- 　 **C.** The refund tool's schema is too permissive and should reject unresolved account ids  
  _wrong-problem_
- ✅ **D.** The two calls have a data dependency, so they must stay in separate sequential turns  
  **CORRECT**

**Why:** Emitting tool_use blocks together means they are dispatched without any one seeing another's result. The refund needs the account id the lookup returns, so in a single batch it is invoked with a value the model had to invent. Parallelism is only available for genuinely independent calls. Retries and stricter schemas would surface the error more loudly but the ordering violation remains. Idempotency concerns repeat execution, not data dependency.

**ELI5:** You can't wrap a present before you've bought it. Some jobs have to happen in order, and doing them at the same time just means you're guessing at the part you don't have yet.

---
### Q2 · obj1

A team wants a coordinator to investigate four unrelated microservices at once. Their current implementation calls the Task tool once, waits for the subagent to return, then calls it again, four times over. They want genuine concurrency. Which change actually produces it?

- ✅ **A.** Issue four Task calls within a single coordinator response rather than across four turns  
  **CORRECT**
- 　 **B.** Start four separate coordinator sessions and merge their transcripts once all have finished  
  _over-engineering_
- 　 **C.** Raise the coordinator's iteration ceiling so more subagent calls fit inside one session  
  _wrong-problem_
- 　 **D.** Set a parallel execution flag on each Task call so the runtime dispatches them together  
  _nonexistent-feature_

**Why:** Concurrency comes from multiple tool_use blocks in one assistant message. Four Task calls emitted together are dispatched together; the same four spread across turns are serial by construction. There is no parallel flag on a Task call — that is an invented parameter. An iteration ceiling limits how many turns may occur, which does not make any of them concurrent. Separate sessions do run at once but discard the coordinator's ability to reconcile results.

**ELI5:** Handing out four jobs at the same meeting is different from calling four separate meetings. Same four jobs — but only one way gets them all started this morning.

---
### Q3 · obj1

A pipeline enriches a customer record from six independent providers. Two of the six are slow and occasionally time out. The team currently calls all six in one batch, and one slow provider failing means the whole enrichment step is retried from scratch, including the four that already succeeded. What is the best design change?

- 　 **A.** Move all six calls back to sequential turns so a failure only ever affects the current call  
  _disproportionate_
- 　 **B.** Wrap the batch in an outer retry that reruns the whole step until every provider responds  
  _disproportionate_
- ✅ **C.** Keep the batch but have each tool return a per-call status so only failures are retried  
  **CORRECT**
- 　 **D.** Drop the two unreliable providers from the batch and enrich those fields on a later pass  
  _burden-shift_

**Why:** Parallel dispatch is correct here — the calls are independent and batching is the latency win. The defect is treating the batch as one atomic unit. If each tool result carries its own status, the coordinator can retry just the failed calls and keep the four successes. Reverting to sequential throws away the concurrency to solve an error-handling problem. Dropping providers loses data. An outer whole-batch retry amplifies exactly the waste described.

**ELI5:** If five of your six parcels arrive, you only chase the missing one. You don't send them all back and reorder everything.

---

## Objective 2 — Tier 1
**Session resumption: --resume, targeted re-analysis, context injection**

### Q4 · obj2

An engineer pauses a debugging session after tracing a bug through six files. Three days later two of those six files have been modified by a teammate; the other four are untouched. They want to continue the investigation without redoing the tracing work. What is the best way to restart?

- 　 **A.** Start a new session and re-trace the bug so every file is read at its current revision  
  _disproportionate_
- 　 **B.** Resume the session and instruct it to re-read all six files before continuing the analysis  
  _disproportionate_
- 　 **C.** Resume the session as-is, since its saved reasoning already reflects how the code behaves  
  _silent-suppression_
- ✅ **D.** Resume the session and tell it explicitly which two files changed so it re-reads only those  
  **CORRECT**

**Why:** Resuming restores the prior reasoning along with cached tool results, which is exactly the hazard: two of those cached reads are now stale. Naming the changed files gives targeted re-analysis — the four unchanged reads stay valid and the reasoning survives. Resuming blind reasons over outdated content. Re-tracing from scratch discards good work. Re-reading all six is safe but pays to refresh four files that did not change.

**ELI5:** You come back to a jigsaw you left half-finished and someone has swapped two pieces. You don't tip the whole thing out — you just check the two that changed.

---
### Q5 · obj2

A session investigating a service was paused before a large refactor landed. The refactor renamed most modules, moved the entry point, and changed the config format. The engineer still wants to keep the conclusions reached earlier. What is the most reliable way to continue?

- 　 **A.** Fork the paused session so one branch re-reads the code and the other keeps the findings  
  _wrong-problem_
- 　 **B.** Resume the paused session and rely on the agent noticing that files no longer match  
  _silent-suppression_
- 　 **C.** Resume the paused session and list every renamed module so each one is re-read  
  _disproportionate_
- ✅ **D.** Start a fresh session seeded with a written summary of the prior conclusions  
  **CORRECT**

**Why:** Targeted re-analysis works when a minority of the cached state is stale. Here nearly all of it is, so resuming means carrying forward a large body of invalid tool results — and the re-read list would cover almost everything anyway. A fresh session injected with a compact summary keeps the conclusions as text while forcing every file read to happen against current code. Hoping the agent notices the mismatch is not a mechanism. Forking branches shared context but never refreshes it.

**ELI5:** If someone rearranged every room in the house, your old map is worse than useless. Keep the notes about what you found, but walk the house again.

---
### Q6 · obj2

During a review, an engineer claims that resuming a session is always safe because it restores the conversation exactly as it was. A colleague disagrees. Which statement most accurately describes the actual risk of resuming?

- 　 **A.** The restored conversation is truncated, so the earliest reasoning steps are silently dropped  
  _nonexistent-feature_
- 　 **B.** Only the assistant's messages are restored, so tool results must be regathered on resume  
  _nonexistent-feature_
- ✅ **C.** Cached tool results are restored as they were captured and may no longer match the files  
  **CORRECT**
- 　 **D.** Resumption replays each prior tool call, so side-effecting tools can execute a second time  
  _nonexistent-feature_

**Why:** Resuming faithfully restores the session including the tool results captured at the time. That fidelity is the problem: a file read three days ago is preserved verbatim even though the file has since changed, so the agent reasons over a stale snapshot without any signal. The other options describe behaviours that do not occur — resumption does not truncate the transcript, does not re-execute prior tool calls, and does not discard tool results.

**ELI5:** It's a photograph, not a live video. It shows exactly what was there when it was taken — which is misleading if things have moved since.

---
### Q7 · obj2

An engineer must hand a long-running investigation to a fresh session because the original context is exhausted. They want the new session to continue without repeating the discovery work. What should they inject into the new session?

- 　 **A.** A short instruction telling the new session that prior analysis exists and can be trusted  
  _prompt-hope_
- ✅ **B.** A structured summary of findings, open questions, and the specific files that matter  
  **CORRECT**
- 　 **C.** The full transcript of the prior session pasted in so no detail is lost in translation  
  _disproportionate_
- 　 **D.** A list of the tool calls the prior session made so the new one can replay them in order  
  _wrong-problem_

**Why:** The point of the handoff is to carry conclusions forward at a fraction of the token cost, so the injection should be distilled state: what was established, what is still open, and where to look next. Pasting the whole transcript reproduces the context exhaustion that forced the handoff. Replaying the tool calls redoes the work rather than inheriting it. Asserting that prior analysis exists gives the new session nothing it can actually use.

**ELI5:** When you hand over a job, you write a short note saying what's done and what's left. You don't hand over a recording of your entire week.

---

## Objective 3 — Tier 1
**State persistence for interrupted multi-agent pipelines**

### Q8 · obj3

A nightly pipeline dispatches ten research subagents and writes a combined report. Last night it died after the seventh subagent returned, and the rerun started again from the first subagent, repeating seven completed investigations. What design change prevents the repeated work?

- ✅ **A.** Write each subagent's findings to a durable state file as they complete, and resume from it  
  **CORRECT**
- 　 **B.** Reduce the batch to five subagents per run so any single failure costs less rework  
  _disproportionate_
- 　 **C.** Increase the pipeline's timeout so a single long run has enough headroom to finish  
  _wrong-problem_
- 　 **D.** Have the coordinator hold all findings in context until the final report is assembled  
  _wrong-problem_

**Why:** Findings that exist only in a process that can die are lost when it dies. Persisting each result as it arrives, with its subtask status, lets a rerun read the manifest and dispatch only the three outstanding subagents. A longer timeout addresses one cause of interruption and none of the others, and still loses everything on a crash. Halving the batch halves the waste rather than removing it. Holding findings in context is precisely the volatile state that vanished.

**ELI5:** Save your game as you go. If you only save at the very end, one crash means playing the whole level again.

---
### Q9 · obj3

A team is adding checkpointing to a multi-agent pipeline so it can resume after interruption. They must decide what each checkpoint record contains. Which content makes resumption actually work?

- 　 **A.** The subtask identifier, its completion status, and a percentage estimate of overall progress  
  _wrong-problem_
- ✅ **B.** The subtask identifier, its completion status, and the findings that subtask produced  
  **CORRECT**
- 　 **C.** A running natural-language narrative of what the pipeline has accomplished so far  
  _wrong-problem_
- 　 **D.** The coordinator's current prompt and the list of subagents it has dispatched so far  
  _wrong-problem_

**Why:** A resumed run needs two things: which subtasks are outstanding, and the results of the ones already done. Status without findings tells the pipeline it can skip a subtask while leaving it nothing to assemble, so the report is incomplete. Progress percentages and narrative summaries are reporting artefacts, not resumable state — neither lets you reconstruct a specific subagent's output. The dispatched list has the same gap.

**ELI5:** A checkpoint has to save what you found, not just how far you got. Knowing you finished seven rooms is no help if you didn't write down what was in them.

---
### Q10 · obj3

A pipeline resumes from its state file after a crash. On the second run it appends findings for subtasks that had already completed before the interruption, producing duplicated sections in the final report. What is the correct fix?

- 　 **A.** Clear the state file at the start of every run so no stale entries can survive  
  _wrong-problem_
- 　 **B.** Deduplicate the final report by comparing section text before assembling output  
  _silent-suppression_
- ✅ **C.** Key each finding by its subtask id so a rerun overwrites rather than appends  
  **CORRECT**
- 　 **D.** Append a run identifier to each entry so the report generator can pick the newest run  
  _over-engineering_

**Why:** Resumption must be idempotent: writing the result for a subtask twice should leave one record, not two. Keying by subtask id gives that directly. Clearing the state file at startup defeats the entire purpose of persistence. Text-level deduplication patches the symptom downstream and fails as soon as two subtasks legitimately produce similar prose. Run identifiers add bookkeeping that the report generator must then reason about.

**ELI5:** Write each result in its own labelled box. Then doing a job twice just replaces what's in that box, instead of adding a second copy.

---
### Q11 · obj3

An architect argues that a multi-agent pipeline does not need external state because the coordinator's conversation history already contains every subagent result. Under what circumstance is that argument sound?

- ✅ **A.** Only when losing all progress on an interruption is an acceptable cost for that pipeline  
  **CORRECT**
- 　 **B.** Whenever the pipeline runs interactively, since a human can restart any failed subagent  
  _burden-shift_
- 　 **C.** Whenever subagents return structured output, since structure makes results reconstructible  
  _wrong-problem_
- 　 **D.** Whenever the coordinator's context window is large enough to hold every subagent result  
  _wrong-problem_

**Why:** Conversation history is process state. It survives as long as the process does, which makes it adequate only when a total loss on interruption is tolerable — a short interactive run, say. Context capacity is irrelevant: a window large enough to hold everything still holds it nowhere durable. Structured output is easier to reuse but only if something wrote it down. Relying on a human to notice and restart shifts the reliability burden onto the operator.

**ELI5:** Keeping notes only in your head works fine right up until you fall asleep. It's not that your head is too small — it's that you wake up empty.

---

## Objective 4 — Tier 1
**Orchestration selection: coordinator-worker vs parallel vs sequential**

### Q12 · obj4

A product needs to answer a question by consulting four independent knowledge bases. The sources never reference one another, the answer must combine all four, and the feature has a tight latency budget. Which orchestration fits best?

- 　 **A.** A single agent given access to all four knowledge-base tools and told to consult each  
  _wrong-problem_
- 　 **B.** A sequential chain where each subagent receives the previous subagent's output as input  
  _wrong-problem_
- ✅ **C.** A coordinator that dispatches the four subagents concurrently and merges their returns  
  **CORRECT**
- 　 **D.** Four independent agents that write to a shared document which a fifth agent later reads  
  _over-engineering_

**Why:** Independent sources plus a latency constraint plus a required merge is the textbook fan-out-and-reconcile shape: dispatch concurrently for speed, and keep a coordinator so the four returns are combined and gaps are visible. A sequential chain invents a dependency that does not exist and multiplies latency. One agent holding all four toolsets works but serialises the lookups and dilutes tool selection. A shared document adds a coordination surface for no benefit.

**ELI5:** Four people can read four different books at the same time, then tell you what they found. Making them queue up and pass one book along is slower for no reason.

---
### Q13 · obj4

A document pipeline extracts claims, verifies each claim against a source, then rewrites the document to correct any claim that failed verification. Each stage consumes the previous stage's output. An engineer proposes running the three stages concurrently to cut latency. What should the reviewer say?

- 　 **A.** The stages should be merged into one agent so the dependency disappears along with the latency  
  _over-engineering_
- 　 **B.** Concurrency is fine if each stage re-reads the source document to rebuild what it needs  
  _wrong-problem_
- 　 **C.** Concurrency is fine provided the rewrite stage runs last and the first two run together  
  _wrong-problem_
- ✅ **D.** The stages are dependent, so they must remain sequential regardless of the latency goal  
  **CORRECT**

**Why:** Verification needs the extracted claims and the rewrite needs the verification verdicts, so every stage depends on its predecessor. No arrangement of concurrency removes a real data dependency. Having each stage rebuild its input from the source duplicates work and risks divergence between stages. Running extraction and verification together still leaves verification without claims. Collapsing everything into one agent trades a clear pipeline for an opaque one and does not speed up dependent steps.

**ELI5:** You can't check the answers before the questions are written, or fix the mistakes before you've checked. Some things just have to go in order.

---
### Q14 · obj4

In a compliance-sensitive system, several agents each retrieve part of a customer's file. Auditors require a record of every inter-agent message, and any retrieval failure must be handled identically everywhere. An engineer suggests letting agents pass results directly to whichever agent needs them next. Why should this be rejected?

- 　 **A.** Direct handoffs require each agent to hold the full customer file, which widens exposure  
  _wrong-problem_
- ✅ **B.** Direct handoffs remove the single point through which messages and failures are observed  
  **CORRECT**
- 　 **C.** Direct handoffs are blocked by the platform, so the design would fail at runtime anyway  
  _nonexistent-feature_
- 　 **D.** Direct handoffs prevent agents from running concurrently, which the workload requires  
  _wrong-problem_

**Why:** Routing everything through a coordinator is what makes traffic observable and error handling uniform — exactly the two properties the auditors and the failure requirement demand. A direct edge between agents bypasses that point of control, so those messages appear in no audit trail and each agent invents its own failure behaviour. It is an architectural principle, not a platform restriction, and it neither prevents concurrency nor forces agents to hold more data.

**ELI5:** If every message goes through one desk, that desk has a record of everything. Let people pass notes directly and nobody can say what was said.

---
### Q15 · obj4

A coordinator splits a broad research question into subtasks, dispatches them, and every subagent returns successfully. Reviewers find the final report omits an entire relevant area that no subtask ever covered. Which change addresses the root cause?

- 　 **A.** Have a second coordinator independently decompose the question and compare the subtask lists  
  _over-engineering_
- ✅ **B.** Have the coordinator check the merged findings against the question's scope before reporting  
  **CORRECT**
- 　 **C.** Have each subagent flag areas outside its assignment that it noticed while researching  
  _burden-shift_
- 　 **D.** Increase the number of subagents so more of the question's surface area gets assigned  
  _heuristic-guess_

**Why:** Every subagent succeeded at the task it was given, so no worker-level signal can reveal the gap — the omission was created upstream when the question was decomposed. The coordinator owns scope, so it must validate coverage of the merged result against the original question before publishing. Asking subagents to notice what they were not asked about is unreliable. More subagents divides the same flawed decomposition more finely. A duplicate coordinator doubles cost for a check the first one should perform.

**ELI5:** If you send people to search three rooms and they all search well, nobody will mention the fourth room — because nobody was told it existed. Whoever wrote the list has to check the list.

---

## Objective 5 — Tier 1
**Subagent output schema: prose vs structured vs citation metadata**

### Q16 · obj5

Five subagents each return market findings as flowing paragraphs. The coordinator must remove duplicate findings, drop any finding older than a year, and rank what remains by source quality. Engineers report the coordinator's merge step is unreliable. What is the root cause?

- ✅ **A.** The findings are prose, so the fields the merge depends on must be re-parsed from text  
  **CORRECT**
- 　 **B.** Five subagents produce more findings than a single merge step can reliably process at once  
  _wrong-problem_
- 　 **C.** The subagents apply different quality standards, so their findings are not comparable  
  _wrong-problem_
- 　 **D.** The coordinator's prompt lacks explicit instructions describing how to merge the findings  
  _prompt-hope_

**Why:** The downstream step performs programmatic operations — deduplicate, filter by date, rank by source — that require date, source, and identity as addressable fields. Prose forces the coordinator to recover them by reading, which is where the unreliability enters. Structured records with those fields make the merge mechanical. Better merge instructions still leave the model parsing paragraphs. Volume and differing standards are real concerns but neither explains failure at the field level.

**ELI5:** Sorting a pile of letters is easy if the address is on the envelope. If it's buried in the middle of the letter, you have to read every one and you'll get some wrong.

---
### Q17 · obj5

A research system produces a brief that analysts must be able to challenge, tracing any individual claim back to the material it came from. Subagents currently return each claim as a sentence with the source name mentioned inline. What should the output schema change to?

- 　 **A.** Each subagent's output followed by a bibliography listing every source it consulted  
  _wrong-problem_
- ✅ **B.** Each claim paired with its source, publication date, and a link, carried as separate fields  
  **CORRECT**
- 　 **C.** Each claim rewritten to name its source more prominently at the start of the sentence  
  _prompt-hope_
- 　 **D.** Each claim assigned a numeric confidence score so analysts can judge which to challenge  
  _wrong-problem_

**Why:** Traceability means a specific claim maps to the specific material behind it, which requires attribution to travel with the claim as data — source, date, and locator — so it survives every merge and rewrite between subagent and brief. Restating the source inside the sentence leaves it as prose to be lost. A bibliography establishes what was consulted but not which claim came from where. A confidence score is a different signal and does not identify the source.

**ELI5:** A quote needs a label saying exactly where it came from, stuck to it. A reading list at the back doesn't tell you which book each sentence came from.

---
### Q18 · obj5

A subagent summarises interview transcripts. Its only consumer is a human-facing narrative section of a report; nothing downstream sorts, filters, or joins the output. An engineer proposes converting the summary into a deeply nested JSON object with a field for every observation. What is the strongest objection?

- 　 **A.** Narrative summaries cannot be represented as structured data without losing their meaning  
  _wrong-problem_
- 　 **B.** Nested JSON exceeds what a subagent can reliably produce, so validation failures will rise  
  _wrong-problem_
- 　 **C.** The schema would need revising whenever the interview questions change, creating churn  
  _wrong-problem_
- ✅ **D.** The consumer needs readable narrative, so the structure adds cost without a downstream user  
  **CORRECT**

**Why:** Output shape should be chosen by what consumes it. Here the consumer is prose in a report, so structuring buys nothing and costs schema design, token overhead, and a reassembly step to turn fields back into readable text. Reliability and schema churn are genuine considerations but secondary to there being no downstream operation that needs fields at all. The claim that narrative cannot be structured is too strong — it can be; it just should not be here.

**ELI5:** You don't need a spreadsheet for a story someone's going to read out loud. Put things in boxes when a machine has to sort them, not when a person just reads it.

---
### Q19 · obj5

A coordinator must drop any subagent finding published before a cutoff date. Subagents currently return dates written into the finding text in whatever format the source used. Filtering is unreliable and some recent findings are being discarded. What is the correct fix?

- 　 **A.** Instruct subagents to write all dates in the finding text using one agreed format  
  _prompt-hope_
- ✅ **B.** Have each subagent return the publication date as its own typed field in a fixed format  
  **CORRECT**
- 　 **C.** Have the coordinator normalise the date formats it encounters before applying the filter  
  _wrong-problem_
- 　 **D.** Widen the cutoff window so that formatting variance no longer changes which findings pass  
  _silent-suppression_

**Why:** A value the coordinator must compare should be a field with a defined type, not something recovered from prose. Promoting the date out of the text removes both the extraction step and the format ambiguity. Instructing subagents to format dates consistently inside the text is a prompt-level rule with no enforcement, and the date still has to be found before it is compared. Normalising downstream keeps the fragile extraction. Widening the window hides the bug by making the filter less correct.

**ELI5:** Put the date in the date box. Don't hide it somewhere in the paragraph and hope everyone writes it the same way.

---

## Objective 6 — Tier 1
**Synthesis preserving source-level uncertainty**

### Q20 · obj6

Two subagents return different figures for the same metric: one cites a peer-reviewed study, the other an industry survey using a broader definition. Both are credible and current. The synthesis step must produce a section on this metric. What should it output?

- ✅ **A.** Both figures, each attributed to its source, with the definitional difference stated  
  **CORRECT**
- 　 **B.** A midpoint of the two figures, with both sources cited as jointly supporting it  
  _heuristic-guess_
- 　 **C.** The peer-reviewed figure, with the industry survey noted as a lower-quality source  
  _heuristic-guess_
- 　 **D.** The figure whose definition matches the report's own stated scope, with the other omitted  
  _silent-suppression_

**Why:** The disagreement is real and explainable: two sound measurements of differently defined things. Reporting both with attribution and naming the definitional gap preserves what the evidence actually supports. Ranking by source prestige silently discards a valid measurement. Averaging manufactures a number no source reported and implies a consensus that does not exist. Selecting the better-matching definition is defensible but dropping the other hides the disagreement from the reader.

**ELI5:** Two people measured the room and got different answers because one counted the doorway. Say that — don't split the difference or decide one of them is just worse at measuring.

---
### Q21 · obj6

A synthesis agent merges four findings: three are corroborated across multiple independent sources, one rests on a single vendor blog post that other sources dispute. The current output presents all four in the same declarative voice. What is the problem?

- ✅ **A.** The contested finding reads as settled, so downstream readers cannot weight it correctly  
  **CORRECT**
- 　 **B.** The vendor source should be replaced with an independent one before the report is issued  
  _burden-shift_
- 　 **C.** The contested finding should be removed, since disputed material weakens the whole report  
  _silent-suppression_
- 　 **D.** All four findings should be hedged equally so the report does not overstate its confidence  
  _wrong-problem_

**Why:** Flattening evidential status is a loss of information the synthesis was supposed to carry. A reader deciding on this report cannot tell that one of the four rests on contested ground, so they cannot discount it. The fix is to mark status per claim. Deleting the contested finding hides a real signal. Hedging everything equally destroys the distinction in the other direction, making the three solid findings look shaky. Finding a better source may be impossible and does not fix the synthesis behaviour.

**ELI5:** If you say everything in the same confident voice, nobody can tell which bits are rock solid and which are one person's guess. Say which is which.

---
### Q22 · obj6

A reviewer complains that a synthesis reads as uniformly tentative: nearly every sentence is qualified with phrases suggesting the evidence may be incomplete, including for facts corroborated by many sources. What has gone wrong?

- 　 **A.** The subagents returned confidence scores that the synthesis step rounded down too far  
  _wrong-problem_
- 　 **B.** The synthesis agent's temperature is too high, producing inconsistent qualifying language  
  _wrong-problem_
- ✅ **C.** Uniform hedging carries no information, so genuinely uncertain claims no longer stand out  
  **CORRECT**
- 　 **D.** The source material is genuinely weak, so the tentative language is an accurate reflection  
  _wrong-problem_

**Why:** Preserving uncertainty means differentiating it. Applying the same qualifier everywhere is the mirror image of stating everything confidently: in both cases the reader learns nothing about which claims are solid, and the real warnings are camouflaged by the noise. Temperature affects wording variability, not the decision to hedge. Nothing indicates a scoring bug, and the premise says many claims are well corroborated, so blanket tentativeness is not accurate.

**ELI5:** If you say 'maybe' about everything, the word stops meaning anything. Save it for the things you're actually unsure about.

---
### Q23 · obj6

A downstream automated step approves low-risk actions without human review and escalates anything uncertain. It consumes a synthesis produced by an upstream research agent. What must the synthesis carry for that step to behave correctly?

- 　 **A.** Only the well-established claims, with contested material withheld from the downstream step  
  _silent-suppression_
- ✅ **B.** A per-claim indication of evidential status that the downstream step can read and act on  
  **CORRECT**
- 　 **C.** A single overall confidence rating for the synthesis so the step can gate on one value  
  _wrong-problem_
- 　 **D.** Prose signalling uncertainty in wording, which the downstream step interprets as it reads  
  _prompt-hope_

**Why:** The consumer makes per-claim decisions, so it needs status attached to each claim in a form it can read. A single overall rating cannot distinguish the solid claims from the contested one inside the same document. Encoding uncertainty in prose wording leaves an automated consumer to infer it from language, which is precisely the unreliable step to design out. Withholding contested claims means the escalation path never fires, so risky material is silently dropped instead of reviewed.

**ELI5:** If a machine has to decide claim by claim, it needs a label on each claim. One score for the whole page doesn't tell it which line to worry about.

---

## Objective 7 — Tier 1
**Context optimization: summarize / sliding window / structured state / selective retention**

### Q24 · obj7

A long-running investigation has established several exact values — an error rate, a commit hash, and a timestamp — that the final report must cite precisely. The session is approaching its context limit and the team must choose a context strategy. Which one protects those values?

- 　 **A.** Summarise the conversation periodically, since summaries retain the substance of findings  
  _silent-suppression_
- 　 **B.** Keep a sliding window of recent turns so the most current information is always present  
  _wrong-problem_
- ✅ **C.** Extract those values into a structured state block held outside the summarised history  
  **CORRECT**
- 　 **D.** Raise the summarisation threshold so compaction runs later and less often in the session  
  _wrong-problem_

**Why:** Summarisation optimises for narrative gist, and exact figures are the first thing it erodes into approximations. A sliding window is worse for this case: the values were established early, so they fall out of the window as the session proceeds. Deferring compaction only delays the same loss. Lifting the values into a structured block outside the compacted narrative means no summarisation pass can touch them, and the report can quote the block.

**ELI5:** Summarising a story keeps the plot but blurs the numbers. If you need the exact figure later, write it on a separate card that never gets rewritten.

---
### Q25 · obj7

A customer support assistant handles long chats where only the last few exchanges matter; earlier turns are pleasantries and resolved side questions. Context growth is causing cost and latency to climb. Which strategy fits this workload?

- ✅ **A.** Keep a sliding window of the recent turns and let older exchanges fall out of context  
  **CORRECT**
- 　 **B.** Retain the full history and reduce cost by moving the assistant to a cheaper model tier  
  _wrong-problem_
- 　 **C.** Summarise the whole conversation on every turn so history is compressed but complete  
  _over-engineering_
- 　 **D.** Extract each exchange into a structured state object so nothing from the chat is lost  
  _over-engineering_

**Why:** The right technique is the one whose loss profile matches what the workload can afford to lose. Here old turns genuinely stop mattering, so a sliding window discards exactly the right material at almost no engineering cost. Building structured state for content that has no future use is over-engineering. Summarising every turn spends tokens on each pass to preserve material nobody needs. A cheaper model reduces unit cost while leaving the context growth in place.

**ELI5:** In a long chat about a delivery, what you said fifteen minutes ago about the weather doesn't matter. Letting the old bits drop off is fine here.

---
### Q26 · obj7

An agent calls an inventory tool that returns forty-two fields per item; the agent's reasoning uses four of them. Contexts fill quickly on multi-item queries. The team wants the smallest change with the largest effect. What should they do?

- ✅ **A.** Trim each tool result to the four needed fields in the wrapper before it enters context  
  **CORRECT**
- 　 **B.** Summarise each tool result into a short paragraph before appending it to the conversation  
  _silent-suppression_
- 　 **C.** Instruct the agent in its system prompt to ignore fields that are not relevant to the task  
  _prompt-hope_
- 　 **D.** Increase the context budget allocated to tool results so more items fit before compaction  
  _wrong-problem_

**Why:** The unneeded fields never need to be in context at all, and the deterministic place to remove them is the tool wrapper, before the result is appended. Telling the agent to ignore them does not stop them consuming tokens or being processed. Summarising a structured payload into prose is lossy in the wrong direction — it risks the four fields that matter while still paying to process the other thirty-eight. Raising the budget accepts the waste and delays the ceiling.

**ELI5:** If you only need four numbers, don't paste in the whole forty-two-column spreadsheet and promise not to look at the rest. Cut it down before it goes on the desk.

---
### Q27 · obj7

A team must choose a context strategy for an agent whose later reasoning depends on details established early in the session, but whose intermediate tool chatter is disposable. Which combination matches the workload?

- 　 **A.** Apply a sliding window so context always holds the most recent portion of the session  
  _wrong-problem_
- 　 **B.** Summarise everything uniformly so both the early details and the chatter are compressed  
  _silent-suppression_
- 　 **C.** Retain everything and compact only once the context limit has actually been reached  
  _wrong-problem_
- ✅ **D.** Retain the early established details selectively while discarding intermediate tool output  
  **CORRECT**

**Why:** The workload names two categories with opposite value: early details that must survive and intermediate chatter that need not. Selective retention is the technique that treats them differently. A sliding window is exactly wrong here because it evicts by age, and the material that must survive is the oldest. Uniform summarisation compresses the valuable details along with the noise. Waiting for the limit then compacting applies an indiscriminate pass at the worst moment.

**ELI5:** Keep the important things you learned at the start, throw away the scribbles. Don't use a rule that just deletes whatever is oldest — that's the good stuff here.

---

## Objective 8 — Tier 2
**Subagent prompts carrying complete findings + source metadata**

### Q28 · obj8

A coordinator has gathered a customer's account history, three prior tickets, and a diagnosis. It spawns a subagent to draft the customer reply, passing the instruction to write a resolution message. The drafts come back generic and ask questions the coordinator already answered. What should change?

- 　 **A.** Tell the subagent in its prompt that the coordinator holds the relevant context it may use  
  _prompt-hope_
- ✅ **B.** Embed the account history, ticket excerpts, and diagnosis directly in the subagent's prompt  
  **CORRECT**
- 　 **C.** Have the subagent return a list of what it needs so the coordinator can supply it on request  
  _burden-shift_
- 　 **D.** Grant the subagent permission to call the same tools the coordinator used to gather the data  
  _over-engineering_

**Why:** A spawned subagent starts with isolated context and inherits nothing from the coordinator, so anything it needs must arrive in its prompt. Packaging the history, tickets, and diagnosis into the prompt gives it the material to draft from. Granting tool access makes it re-gather data that already exists, spending time and risking a different answer. Telling it that context exists elsewhere does not transfer it. A request-response round trip is the very cost the delegation was meant to avoid.

**ELI5:** If you ask someone to write a reply, hand them the file. Telling them the file exists in your drawer doesn't help them write anything.

---
### Q29 · obj8

An engineer packages findings into a subagent prompt but includes only the claim text, dropping the source names and dates to keep the prompt compact. The subagent must produce a cited summary. What is the consequence?

- 　 **A.** The subagent will request the missing metadata, adding one round trip to the workflow  
  _wrong-problem_
- ✅ **B.** The subagent cannot attribute claims, so citations must be fabricated or omitted entirely  
  **CORRECT**
- 　 **C.** The prompt saving is worthwhile because the coordinator can attach citations afterwards  
  _wrong-problem_
- 　 **D.** The subagent will infer sources from the claim wording, producing mostly correct citations  
  _heuristic-guess_

**Why:** Attribution cannot be reconstructed from a claim's text. Stripping source metadata leaves the subagent with two options, both bad: invent plausible citations or return an uncited summary. Whichever it does, the deliverable is broken. Subagents do not reliably come back asking for missing inputs — that is the failure mode being designed out. Expecting the coordinator to reattach citations afterwards assumes a claim-to-source mapping that was just discarded.

**ELI5:** If you copy out the quotes but throw away who said them, nobody can put the names back later. They're gone.

---

## Objective 9 — Tier 2
**Goal-oriented vs procedural delegation**

### Q30 · obj9

A subagent is given a fifteen-step procedure for investigating a failing test. When the failure has an unexpected shape — the test passes locally but fails in the pipeline — the subagent follows the steps anyway and returns nothing useful. What delegation change helps most?

- 　 **A.** Extend the procedure with additional branches covering the cases seen so far  
  _wrong-problem_
- 　 **B.** Split the fifteen steps across three subagents so each follows a shorter procedure  
  _over-engineering_
- 　 **C.** Have the subagent report which step it was on when it stopped making progress  
  _wrong-problem_
- ✅ **D.** State the goal and what a good result looks like, leaving the method to the subagent  
  **CORRECT**

**Why:** A procedural prompt encodes the author's assumptions about what the investigation will find; when reality differs, the subagent has no licence to deviate. Specifying the objective and the acceptance criteria lets it adapt its method while still being judged against a clear standard. Adding branches works only for failures already anticipated. Reporting the stuck step improves visibility without restoring adaptability. Splitting the procedure keeps every step rigid across three agents.

**ELI5:** Tell someone what a finished job looks like, not every footstep to take. Otherwise the moment something's different they just keep walking into the wall.

---
### Q31 · obj9

A subagent verifies that every release passes the same fixed compliance checklist: identical items, identical order, identical evidence for each item, on every run. An engineer proposes replacing the checklist with a goal-oriented prompt describing the compliance outcome. What is the strongest objection?

- 　 **A.** Goal-oriented delegation applies only to multi-agent systems, not to a single subagent  
  _nonexistent-feature_
- 　 **B.** The subagent cannot interpret compliance goals without domain knowledge it does not hold  
  _wrong-problem_
- ✅ **C.** The task is fixed and auditable, so a defined procedure is what makes runs comparable  
  **CORRECT**
- 　 **D.** Goal-oriented prompts consume more tokens, which raises the cost of every release check  
  _wrong-problem_

**Why:** Goal-oriented delegation earns its keep when the path is unknown and adaptation is valuable. This task is the opposite: the steps are known, must not vary, and the evidence must be comparable across releases for audit. A defined procedure is the right tool. Token cost is a minor consideration. The subagent's domain knowledge is not the issue. And nothing restricts either delegation style to a particular system topology.

**ELI5:** For a safety checklist you want the same boxes ticked the same way every time. That's the one place where 'just follow the steps' is exactly right.

---

## Objective 10 — Tier 2
**Extraction schemas: optional/nullable + enums**

### Q32 · obj10

An extraction schema marks every field required. Reviewers find the model emits plausible-looking values for fields that are genuinely absent from the source document. What schema change addresses this?

- 　 **A.** Keep the fields required and validate outputs against the source in a later pipeline stage  
  _wrong-problem_
- 　 **B.** Replace absent values with an empty string convention that downstream code checks for  
  _wrong-problem_
- 　 **C.** Keep the fields required and instruct the model in the prompt never to invent values  
  _prompt-hope_
- ✅ **D.** Make fields that may legitimately be absent nullable so the model can express absence  
  **CORRECT**

**Why:** A required field obliges the model to produce something, so when the document is silent the only way to satisfy the schema is to invent. Nullable typing gives absence a legal representation and removes the pressure. A prompt instruction competes with a structural constraint and loses under pressure. Downstream validation catches some fabrications after the fact without preventing them. An empty-string convention is an informal null that every consumer must remember to special-case.

**ELI5:** If the form won't let you leave a box empty, people write something in it. Let them tick 'not stated' and they'll stop making things up.

---
### Q33 · obj10

A classification field uses an enum of four document categories. In production some documents genuinely fit none of them, and the model assigns whichever category is closest, corrupting downstream routing. What is the correct schema change?

- ✅ **A.** Add explicit enum members for unclear and other, together with a free-text detail field  
  **CORRECT**
- 　 **B.** Expand the enum with additional categories to cover the document types now appearing  
  _wrong-problem_
- 　 **C.** Allow the classification field to be null when no listed category applies to a document  
  _wrong-problem_
- 　 **D.** Keep the enum and add a separate confidence score so routing can ignore weak assignments  
  _over-engineering_

**Why:** A closed enum with no escape hatch forces every document into a listed bucket, so an unmatched document becomes a wrong assignment rather than a flagged one. Adding explicit unclear and other members, plus a detail field, lets the model say what it actually observed and gives routing something to act on. Expanding the enum chases a moving target. A null says nothing about why. A confidence score leaves the wrong category attached and asks downstream to guess a threshold.

**ELI5:** If the only choices are red, blue, green, and yellow, a purple thing gets called blue. Adding an 'other, and here's what it was' option stops the guessing.

---

## Objective 11 — Tier 2
**Cross-session context: isolation, scratchpads, targeted reads**

### Q34 · obj11

An architecture task needs a wide survey of forty modules before any planning can happen. Running the survey in the main session fills the context with file contents, leaving little room for the planning work that follows. What is the right structure?

- 　 **A.** Read the forty modules in the main session but compact the context once the survey ends  
  _wrong-problem_
- 　 **B.** Read only the ten largest modules on the assumption they carry most of the architecture  
  _heuristic-guess_
- ✅ **C.** Delegate the survey to an isolated subagent that returns only a condensed written summary  
  **CORRECT**
- 　 **D.** Read the modules in batches across several main sessions and combine the notes manually  
  _burden-shift_

**Why:** The verbose material is the file contents, and it has no value once the survey conclusions exist. A subagent with its own context absorbs that volume and returns only the summary, so the main session receives the conclusions without the raw text. Compacting afterwards means the main context still had to hold everything first. Manual batching moves the assembly burden to the human. Sampling by size is a proxy that will miss small but architecturally central modules.

**ELI5:** Send someone else to read the whole shelf and come back with one page of notes. Then your own desk still has room to work on.

---
### Q35 · obj11

An investigation will span several working sessions. Findings established today must be usable tomorrow, when the current session no longer exists. Where should those findings live?

- 　 **A.** In progressively longer summaries appended to each session so context carries forward  
  _wrong-problem_
- 　 **B.** In the commit messages of the work produced, so the record travels with the repository  
  _wrong-problem_
- 　 **C.** In the session history, which can be resumed tomorrow to recover the same findings  
  _wrong-problem_
- ✅ **D.** In a scratchpad file on disk that later sessions read at the point they need it back  
  **CORRECT**

**Why:** Findings that must outlive a session have to be written somewhere outside it, and a scratchpad file gives durable storage plus targeted reads later. Session history is process state and also carries stale cached tool results when resumed. Appending ever-larger summaries drives straight back into the context ceiling the design is trying to avoid. Commit messages describe changes that were made, not the intermediate findings and open questions an investigation accumulates.

**ELI5:** Write it in a notebook you can pick up tomorrow. Don't rely on remembering the conversation you had today.

---
