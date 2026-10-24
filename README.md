# biologist-consensus-loop

A [Claude Code](https://claude.com/claude-code) skill for running an omics analysis (first target: bulk RNA-seq) **together with a biologist who is not a computational expert**. Claude works in an *analyse → explain → ask → update* loop (分析—解释—反问—更新). The aim is a correct analysis that the biologist, by the end, can also direct and interpret themselves.

## What it does

- **Keeps three state files** in `<analysis_dir>/consensus/`:
  - `analysis_state.md`: what the analysis is doing;
  - `consensus_state.md`: what both sides agree on. Each item is marked `understood` / `allowed` / `unsure` / `unknown` / `contradicted`, and clicking "continue" only counts as *allowed*, never *understood*;
  - `decision_log.md`: one entry per decision; it becomes the skeleton of the final report.
- **Decides whether to ask.** Five triggers (ambiguous intent, unconfirmed key assumption, a result that defies expectation, a choice that changes the conclusion, possible misunderstanding), plus one test: *could the user's answer change the next action or how the result is interpreted?* If not, Claude adds a short note instead of interrupting.
- **Uses one format per point:** Finding → Meaning → Suggestion → one Question, with the figure and sample rows shown next to it. The user can answer, correct the premise, or say "I'm not sure".
- **Shows what changed after each answer**, e.g. "the samples are paired → switching to a paired design; the result now describes change within each animal". It then logs the decision.
- **Adapts per concept** (batch effect, pairing, FDR, outliers…), not per person.
- **Covers four decision classes in v1:** research goal, sample relationships, outlier samples, interpreting conclusions.
- **Has three questioning modes for a user study:** `on-demand`, `fixed` (stage gates), and `dynamic` (default). All three run the same analysis. Every question is logged so interruption cost can be measured. See [`references/study_design.md`](skills/biologist-consensus-loop/references/study_design.md).

## Install

Requires Claude Code. Skills are folders with a `SKILL.md`, which Claude Code discovers automatically.

**For all your projects (personal skill):**

```bash
git clone https://github.com/C0nc/biologist-consensus-loop.git
mkdir -p ~/.claude/skills
cp -r biologist-consensus-loop/skills/biologist-consensus-loop ~/.claude/skills/
```

**For one project only (shared with everyone who works in that repo):**

```bash
mkdir -p .claude/skills
cp -r /path/to/biologist-consensus-loop/skills/biologist-consensus-loop .claude/skills/
```

**To keep it updated with `git pull`**, symlink instead of copying:

```bash
ln -s "$(pwd)/biologist-consensus-loop/skills/biologist-consensus-loop" ~/.claude/skills/biologist-consensus-loop
```

Start a new Claude Code session afterwards. To check that it is installed, ask Claude *"is the biologist-consensus-loop skill available?"*

## Use

It triggers on its own when a collaborator asks Claude to analyse their data and says things like "walk me through it", "I'm not a bioinformatician", or "带我做分析". You can also ask for it directly:

```
Use the biologist-consensus-loop skill to analyse the RNA-seq counts in ./data with me.
```

To run a specific study condition, name the mode: *"…in `fixed` mode"* or *"…in `on-demand` mode"*.

## Layout

```
skills/biologist-consensus-loop/
├── SKILL.md                    # the protocol Claude follows
├── templates/                  # copied into <analysis_dir>/consensus/ at the start
│   ├── analysis_state.md
│   ├── consensus_state.md
│   └── decision_log.md         # includes the per-question interaction log
└── references/study_design.md  # 3-condition evaluation design + outcome measures
```
