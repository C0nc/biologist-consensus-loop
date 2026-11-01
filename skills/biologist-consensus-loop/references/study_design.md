# Study design: does dynamic questioning improve understanding?

## Conditions (same base analysis capability in all three)

| Condition | Skill mode | Interaction |
|---|---|---|
| On-demand explanation | `on-demand` | Agent analyses; the user asks questions when they want |
| Fixed-checkpoint questioning | `fixed` | Agent asks at the predefined stage gates (SKILL.md §6) |
| Dynamic questioning | `dynamic` | Agent picks timing and question from the analysis state + current consensus (SKILL.md §2) |

Keep the dataset, pipeline, tools and model the same. Only the mode switch differs.

## Outcome measures

- **Understanding:** can the user explain *why* a specific choice affects the conclusion (e.g. why pairing changes what the DE result means)?
- **Instruction quality:** do the user's later instructions carry clearer goals, constraints and experimental context? Code instructions for presence of goal / constraint / context.
- **Result interpretation:** can the user name the limits of the conclusion, and judge whether it still holds when the situation changes?
- **Interaction cost:** number of interruptions, time taken, subjective burden (e.g. NASA-TLX). From each run folder: interruptions = agent-initiated rows in `interaction_log.tsv`; time per exchange = `answered_at − asked_at`; total time = `run.md` start → end.

## Data from each run

Each run writes `consensus/<run_id>/` (see SKILL.md §0, §7, §8):
- `interaction_log.tsv`: one row per question from either side. User-initiated rows matter most in the `on-demand` condition, where the user drives the questions.
- the history tables in `consensus_state.md`: when each consensus item and concept changed status, and why. Use them to compare how quickly and how far shared understanding develops in each condition.
- `decision_log.md`: superseded decisions show where the analysis had to change course.
- `run.md`: mode, participant label and summary counts for the run.

The model writes these records itself. Before analysing, check a sample of runs against the session transcripts for missing rows.

## Transfer probe

Measure understanding with a **new small scenario** (e.g. a different dataset where batch is confounded with condition, or the samples turn out to be paired). Check whether the user can carry what they learned over to it, not just repeat what the agent said.

## Scope for v1

Cover only the four decision classes: research goal, sample relationships, outlier samples, interpreting conclusions. First find out **which questions actually changed the user's instructions or their understanding of the results**. The `action_changed` and `consensus_changed` columns of `interaction_log.tsv`, together with the outcome measures, show this. Extend the dynamic policy only after that.

Contribution framing: through a dialogue that adapts as the analysis moves forward, the user gradually becomes able to direct and interpret this analysis.
