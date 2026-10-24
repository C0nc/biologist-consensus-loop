---
name: biologist-consensus-loop
description: Run an omics analysis (first target RNA-seq) together with a biologist who is not a computational expert, using an "analyse → explain → ask → update" loop. Claude keeps an analysis state, a shared-understanding (consensus) state and a decision log. At each meaningful decision point it decides whether a question is worth an interruption. It reports each point as Finding → Meaning → Suggestion → one Question, shows what changed after each answer, and adapts its explanations to which concepts this user has shown they understand. Use when a wet-lab or clinical collaborator asks Claude to analyse their data, says "walk me through it", "I'm not a bioinformatician", "help me understand the results", or 带我做分析/边做边解释. Also use when running the three-condition user study (on-demand / fixed / dynamic questioning) described in references/study_design.md.
---

# Biologist consensus loop (分析—解释—反问—更新)

The goal is not only a correct analysis. By the end, the biologist should be able to **direct and interpret this analysis themselves**. What you design is *when* to build which piece of shared understanding, and *which single question* moves it forward most.

## 0. Setup (once per analysis)

1. Ask for, or infer, the analysis directory. Create `<analysis_dir>/consensus/` and copy the three templates from `templates/` in this skill into it:
   - `analysis_state.md`: what the analysis is doing
   - `consensus_state.md`: what both sides currently agree on, and what is still unknown
   - `decision_log.md`: one entry per decision; this becomes the skeleton of the final report
2. Set the questioning **mode** (default `dynamic`; the other two exist for the user study):
   - `on-demand`: never start a question unless the analysis is blocked; answer the user's own questions.
   - `fixed`: ask only at the stage gates listed in §6, whatever the data shows.
   - `dynamic`: use the gate in §2.
   Record the mode at the top of `analysis_state.md`. All three modes run the **same analysis**; only how you interact changes.
3. Start by finding out the scientific question in the user's own words. Write it to `consensus_state.md` as *stated intent*, not as a confirmed analysis plan.

## 1. Keep both states current

Update the files whenever something changes, not just at the end.

| Analysis state | Consensus state |
|---|---|
| Current biological question | Research intent the user has stated explicitly |
| Step being executed | Choices the user has understood or confirmed, **and what each implies** |
| What the newest result revealed | Experimental context, assumptions or concepts still unconfirmed |
| Options for the next step and what each affects | Information only the user can provide |

Every consensus item carries a status:
`understood` (the user explained it back or used it correctly) · `allowed` (the user said go ahead, but understanding is not shown) · `unsure` (the user said they don't know) · `unknown` (not discussed yet) · `contradicted` (new data broke it).

**"Continue" means `allowed`, never `understood`.**

## 2. Decision gate: should I ask now? (dynamic mode)

Run this whenever a new result arrives or you are about to change the analysis route. Check for triggers:

1. **Ambiguous intent**: different readings of the goal lead to different analyses.
2. **Key assumption unconfirmed**: something only the experimental context can settle (paired samples? batches? replicate type?).
3. **Result defies expectation**: e.g. samples don't group the way the design implies.
4. **Choice changes the conclusion**: different handling changes which scientific question can be answered.
5. **Possible misunderstanding**: the user's wording claims more than the current evidence supports.

Then the deciding test: **could the user's answer change my next action, or how the result is interpreted?**
- **Yes**: ask (§3). Only one question per turn.
- **No**: don't interrupt. Add a one-line note, or a short explanation the user can expand ("details ▸"), and go on.

If several questions are building up, **merge** related ones into a single request for experimental context ("To set up the comparison I need three facts about how the samples were collected: …").

## 3. Interaction format: Finding → Meaning → Suggestion → one Question

```
**Finding:** In this PCA the samples mainly cluster by sequencing batch.
**Meaning:** Differences between groups seen later may mix technical and treatment effects.
**Suggestion:** I'll first check how batches and treatment groups are distributed, then fix the comparison.
**Question:** Did each batch contain both control and treated samples?
```

- Show the evidence right next to it: the figure path or an inline plot, plus the relevant rows of the sample sheet, so the user can check your basis.
- Always offer three ways out: **answer**, **correct my premise**, or **"I'm not sure"**.
- Explain in terms of *this data and its scientific consequence*. Never turn into a generic statistics tutorial.
- If the user is unsure: look at the available metadata yourself, give concrete examples of what each answer would look like, continue with the conservative choice, and record the assumption as an open limitation (`unsure` in the consensus state, and in the "Still open" field of the decision log).

## 4. After every answer, make the change visible (the most important step)

Say explicitly what the answer changed:

> "You've told me these samples are before/after measurements from the same animals. I'll switch to a paired analysis. That means the result now describes the change **within each animal** before and after treatment."

Then append a decision-log entry (template in `templates/decision_log.md`):

- **Current question:** e.g. does treatment change expression within individuals?
- **Confirmed basis:** samples are paired (source: user, turn N)
- **Impact on analysis:** pairing added to the design formula (`~ animal + treatment`)
- **Still open:** …

If the answer changes earlier results, say which earlier plots or claims now need to be reinterpreted.

## 5. Adapt per concept, not per person

Being familiar with one concept says nothing about the whole pipeline. Track familiarity **for each concept** in `consensus_state.md` (e.g. batch effect, pairing, FDR, normalisation, shrinkage, outlier).

| What the user does, or what happens | How you adjust |
|---|---|
| The user explains a choice's implication in their own words | Mark that concept `understood`; from then on skip basics and discuss only the scientific trade-off |
| The user says "not sure" | Concrete examples + check the metadata + carry the assumption forward as a stated limitation |
| The user corrects the research goal | Update the plan; list which existing results must be reinterpreted |
| A new result breaks an earlier assumption | Reopen that decision (status → `contradicted`), say why it needs confirming again |
| Several questions arrive together | Batch them into one request for experimental context |

## 6. First-version scope: four decision classes (RNA-seq)

Cover these well before expanding. The stage gates double as the checkpoints for `fixed` mode:

| Decision class | Typical gate | What to establish |
|---|---|---|
| **Research goal** | Before any analysis | Which comparison answers their question; between-group vs within-individual |
| **Sample relationships** | After loading metadata | Pairing, biological vs technical replicates, batch × condition layout (confounded?) |
| **Outlier samples** | After QC / PCA / sample correlation | Is the outlier a known biological case, a failed library, or a mislabel? Keep, drop, or model it |
| **Interpreting conclusions** | After DE / enrichment | What the result does and does not support; whether the user's phrasing goes beyond the evidence; what would change it |

## 7. Interaction log (for evaluation)

For each question you ask, append one row to the table at the bottom of `decision_log.md`: turn, mode, trigger (1–5 or `stage-gate` or `blocking`), decision class, response type (`answered` / `corrected premise` / `unsure` / `skipped`), and whether the next action changed (y/n). This gives interruption count and how often an answer changed the action, so you can measure the cost of interacting.

## 8. Final report

Build the report from `decision_log.md`. For each result say **why it was obtained this way**: which confirmed facts it rests on, which assumptions are still open, and which limitations follow from them. End with a short "what would change this conclusion" section.

For the study design that compares the three modes, and for how to measure whether understanding improved, see `references/study_design.md`.
