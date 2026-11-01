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

## Tutorial

You need [Claude Code](https://code.claude.com/docs) installed. A Claude Code skill is just a folder containing a `SKILL.md`. Claude Code finds skills in two places:

| Scope | Folder | Who gets it |
|---|---|---|
| **Personal** | `~/.claude/skills/<skill-name>/` | You, in every project |
| **Project** | `<your-repo>/.claude/skills/<skill-name>/` | Everyone working in that repo (commit it) |

If the same name exists in both, the personal copy wins.

### 1. Install

#### Option A: personal install, kept up to date with `git pull` (recommended)

Clone the repo once, then symlink the skill folder into your personal skills directory:

```bash
git clone https://github.com/C0nc/biologist-consensus-loop.git ~/biologist-consensus-loop
mkdir -p ~/.claude/skills
ln -s ~/biologist-consensus-loop/skills/biologist-consensus-loop ~/.claude/skills/biologist-consensus-loop
```

To update later:

```bash
git -C ~/biologist-consensus-loop pull
```

#### Option B: personal install, plain copy

```bash
git clone https://github.com/C0nc/biologist-consensus-loop.git
mkdir -p ~/.claude/skills
cp -r biologist-consensus-loop/skills/biologist-consensus-loop ~/.claude/skills/
```

#### Option C: one project only

Run this from the root of the analysis repo. Commit `.claude/skills/` so your collaborators get the skill too.

```bash
mkdir -p .claude/skills
cp -r ~/biologist-consensus-loop/skills/biologist-consensus-loop .claude/skills/
```

#### Windows (PowerShell)

```powershell
git clone https://github.com/C0nc/biologist-consensus-loop.git "$HOME\biologist-consensus-loop"
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse "$HOME\biologist-consensus-loop\skills\biologist-consensus-loop" "$HOME\.claude\skills\"
```

### 2. Check that it loaded

Claude Code watches the skills folders, so a running session picks up the new skill without a restart. One exception: if `~/.claude/skills/` (or the project's `.claude/skills/`) did **not exist** when the session started, run `/reload-skills` once, or start a new session.

Then check either way:
- type `/skills` in Claude Code and look for **biologist-consensus-loop** in the list; or
- type `/biologist-consensus-loop` and confirm it appears in the `/` menu.

### 3. Use it

**Automatic:** Claude loads the skill by itself when a collaborator asks for an analysis with phrases like "walk me through it", "I'm not a bioinformatician", "help me understand the results" or "带我做分析".

**Manual:** start your message with the slash command:

```
/biologist-consensus-loop analyse the RNA-seq counts in ./data with me
```

**Study conditions:** name the mode in your request. `dynamic` is the default.

```
/biologist-consensus-loop analyse ./data with me in fixed mode
/biologist-consensus-loop analyse ./data with me in on-demand mode
```

Claude creates `./consensus/` (inside your analysis folder) holding the analysis state, the consensus state and the decision log, and keeps them up to date as the analysis goes on.

### 4. Enable / disable

You don't have to uninstall the skill to turn it off. Pick whichever level of "off" you need:

| You want | How |
|---|---|
| **Toggle it from inside Claude Code** | Type `/skills`, highlight `biologist-consensus-loop`, press **Space** to cycle through its states (`on` → `name-only` → `user-only` → `off`), then **Esc** to save. The choice is saved to `.claude/settings.local.json`. |
| **Manual only** (Claude won't load it on its own; `/biologist-consensus-loop` still works) | Add `"skillOverrides": { "biologist-consensus-loop": "user-invocable-only" }` to a settings file (see below). Or add `disable-model-invocation: true` to the frontmatter of `SKILL.md`. |
| **Fully off** (hidden from Claude and from the `/` menu) | `"skillOverrides": { "biologist-consensus-loop": "off" }` |
| **Back on** | `"skillOverrides": { "biologist-consensus-loop": "on" }`, or remove the entry |
| **Block it with a permission rule** | `"permissions": { "deny": ["Skill(biologist-consensus-loop)"] }` |

Where the settings live:

| File | Applies to |
|---|---|
| `~/.claude/settings.json` | You, in all projects |
| `<repo>/.claude/settings.json` | Everyone in that repo (committed) |
| `<repo>/.claude/settings.local.json` | You, in that repo only (not committed) |

For example, to keep the skill installed but stop it from loading automatically, put this in `~/.claude/settings.json`:

```json
{
  "skillOverrides": {
    "biologist-consensus-loop": "user-invocable-only"
  }
}
```

If the file already has other settings, merge the `skillOverrides` key into the existing JSON object instead of replacing the file.

### 5. Uninstall

Remove the skill folder (or the symlink, for Option A):

```bash
rm -r ~/.claude/skills/biologist-consensus-loop
```

For a project install, remove `.claude/skills/biologist-consensus-loop/` from the repo instead. The change takes effect in running sessions without a restart.

### Troubleshooting

- **Not in `/skills`:** check that the file sits at exactly `~/.claude/skills/biologist-consensus-loop/SKILL.md`, not nested one level deeper (e.g. `…/skills/biologist-consensus-loop/skills/…` after copying the whole repo). Then run `/reload-skills`.
- **Never loads by itself:** check that `skillOverrides` doesn't set it to `off` or `user-invocable-only`, and that no `Skill(...)` deny rule matches it. You can always start it manually with `/biologist-consensus-loop`.
- **The wrong version runs:** a personal copy beats a project copy with the same name. Remove one of them.

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
