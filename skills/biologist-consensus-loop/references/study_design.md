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
- **Interaction cost:** number of interruptions, time taken, subjective burden (e.g. NASA-TLX). Interruption counts come from the interaction log in `decision_log.md`.

## Transfer probe

Measure understanding with a **new small scenario** (e.g. a different dataset where batch is confounded with condition, or the samples turn out to be paired). Check whether the user can carry what they learned over to it, not just repeat what the agent said.

## Scope for v1

Cover only the four decision classes: research goal, sample relationships, outlier samples, interpreting conclusions. First find out **which questions actually changed the user's instructions or their understanding of the results**. The interaction log column "Next action changed" together with the outcome measures shows this. Extend the dynamic policy only after that.

Contribution framing: through a dialogue that adapts as the analysis moves forward, the user gradually becomes able to direct and interpret this analysis.
