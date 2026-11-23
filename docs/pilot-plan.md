# Pilot plan

*Prepared November 22, 2026. Status: proposed, not yet approved. Nothing is built until Mark approves this document.*

*Method: written from the thread in [original-brief.md](original-brief.md) and from reading `markreveley/tutorials` (`evals/evals-tut`, commit 73373d4). Paper citations below are from memory and are not yet verified; verifying them is step 1.*

## Why

Skill collections such as Superpowers give an agent instructions like "ask open questions before offering options". Their own tests check that the agent follows the instruction. Nobody checks that following it helps the user. This repo tests that: each skill targets one cognitive bias, and a skill is kept only if it measurably beats the same model without it.

The pilot answers one question before the full set is attempted: can a short, explanatory skill produce an improvement large enough to measure?

## Goals

- Three skills, each with an eval that could cut it.
- A harness and a keep rule that the remaining candidate biases can reuse.
- A result either way. "No skill survived" is a valid outcome and gets written up.

## Non-goals

- The other eleven candidate biases. They wait for the pilot result.
- Showing that a human is de-biased. A simulated user can only show that the assistant made the right move.
- Tuning skill descriptions for triggering. That is tested separately, after a skill has shown it helps when loaded.

## What would make this a failure even if it works

- Skills that win on the scenarios they were tuned against and nowhere else.
- Skills that fix the bias by over-correcting: refusing to recommend when asked, refusing to update when the user is right, or adding turns to every conversation.
- A results table whose differences are noise.

## The three pilot skills

| Skill | Bias | Whose | Target metric (lower is better unless noted) |
| --- | --- | --- | --- |
| `suggestion-anchoring` | The user settles on options the assistant offered instead of what they wanted | user | Echo rate: share of final decisions that came from an assistant suggestion and contradict the hidden goal |
| `caving-to-pushback` | The assistant drops a correct answer when the user disagrees without new evidence | LLM | Cave rate: share of runs where a correct answer is abandoned |
| `premature-closure` | The assistant settles on the first plausible answer | LLM | Error rate on problems whose first plausible answer is wrong |

These three were picked because each has a clean test with a known right answer.

## Skill format

Each `SKILL.md` is under about 60 lines and has five parts: what the bias is, why it happens, how to notice it in a live conversation (with a short example), what to do instead (one or two moves), and when not to apply it. No admonitions and no capitals for emphasis; the skill explains its reasoning. Citations go in `references.md`, not in the skill body.

Layout: `.claude-plugin/plugin.json` plus `skills/<name>/SKILL.md`. No top-level `bin/` (the Claude app refuses it). Any helper script lives in the skill's own folder.

## Eval method

### Scenarios

Each scenario is a file with: the opening user message, a simulated-user persona, the hidden ground truth, and the checks.

- `suggestion-anchoring`: the user has a hidden goal written as a checklist of concrete requirements and opens with an underspecified request. The persona accepts an offered option when it is close enough, and states a requirement only when asked about that area. A second scenario type has the user ask "just pick for me"; there the assistant should propose something.
- `caving-to-pushback`: the assistant is asked something with a verifiable answer, then the user disagrees twice without evidence. A second scenario type has the user push back with valid evidence; there the assistant should update.
- `premature-closure`: a problem where the obvious first answer is wrong and the material needed to see that is in the conversation. A second scenario type has the obvious answer be right; there the assistant should not add turns or hedging.

The second type in each case is the over-correction guard.

Per skill: 20 scenarios, written together and split at random into 10 screening and 10 held-out before any run. Held-out scenarios are not run until the skill text is frozen.

### Arms

- **Baseline**: no skill.
- **Skill**: the skill body loaded into context for every run (forced, so this measures whether it helps once loaded).
- **Placebo** (confirmation stage only): a generic "think carefully before answering" text of the same length. Tests whether the content helps or any extra instruction does.

Each scenario runs 3 times per arm, with the same simulated-user script in every arm.

### Grading

Checks are scripts wherever the answer is checkable (final answer matches ground truth, requirement present in the final spec). Where a model has to judge (did this decision originate with the assistant?), the judge sees transcripts without knowing the arm, and its labels are checked against Mark's hand labels on about 20 transcripts before its numbers are used.

Before any numbers are trusted, Mark reads a sample of transcripts from each skill to check that the simulated user behaves like a plausible person.

### Analysis

The unit is the scenario, not the run. Three repeats of one scenario are not three independent observations, so each scenario is scored by its mean across repeats, and the comparison is the per-scenario difference between arms (paired). The test is a sign-flip permutation test on those differences.

With 10 scenarios, only a large effect is distinguishable from noise. That is intended: a skill that costs context on every load should have a large effect.

### Stages and the keep rule

The numbers below are proposed and are fixed before any run.

1. **Baseline gate.** Run the baseline arm on the screening scenarios. If the biased behaviour shows up in fewer than 30% of runs, there is no headroom. Allow one round of harder scenarios, then cut the skill and record why.
2. **Screen.** Baseline against skill on the screening scenarios. The skill text may be revised here.
3. **Confirm.** Freeze the skill. Run baseline, placebo and skill on the held-out scenarios. Keep the skill only if all of these hold:
   - the target metric improves over baseline by at least 20 points, with one-sided p < 0.05;
   - the skill also beats the placebo in direction;
   - mean turns rise by no more than 1;
   - on the guard scenarios, the over-correction rate is no worse than baseline by more than one scenario;
   - requirements met (or answers correct) does not fall.
4. **Trigger check** (survivors only). About 20 realistic prompts per skill, half where it should load and half near-misses, run in Claude Code to see whether the description loads it at the right moments. In the Claude app this is a manual spot check.

Upper bound on conversations: 180 for screening, plus 90 per surviving skill for confirmation.

## Approaches considered

- **All fourteen biases in one pass** (the original brief). Rejected for now: one harness does not fit all of them, the candidates overlap, and a minimum run is 420 conversations before knowing whether the idea works at all.
- **Pilot of three, then decide** (chosen by Mark, November 22). Builds the harness and keep rule on the cases with clean tests.
- **Evaluate only the one Superpowers instruction that started this** ("open questions before options"). Folded in: `suggestion-anchoring` is that test.

## Order of work

1. Verify the primary sources for the three biases and write `references.md`.
2. Write the 60 scenarios and split them.
3. Build the runner and graders; run the baseline gate.
4. Write the skills that pass the gate; screen; revise.
5. Freeze; confirm on held-out scenarios; trigger check.
6. Results table and README stating which skills survived and why.

## Open decisions

1. **Runner.** Options: Inspect (already used for AmIEval; epochs and logs built in), a small script against the Anthropic API, or `claude -p` in Claude Code. The first two need an API key and measure the skill body directly. The third exercises real skill loading but is slower and harder to control. Proposed: Inspect or the API script for stages 1 to 3, Claude Code for the trigger check.
2. **Model under test.** Results hold only for the model tested. Proposed: the model Mark uses most in the app.
3. **Keep-rule numbers.** The 30% gate, the 20-point minimum and the guard margins above are proposals for Mark to set.
4. **Naming the bias aloud.** Mark's idea: surface the moment so the user (or model) can rephrase, instead of silent protection. Could be a second variant of `suggestion-anchoring` (the assistant says it is holding its suggestions so it does not steer). Costs one extra arm on one skill.
5. **Scenarios from real threads.** The thread transcripts in `direction` and `thread-archive` are real usage. Reading them for moments where these three biases occurred would ground the scenarios in failures that happened. Not yet scoped.

## Left to the builder

Scenario file format, runner internals, the placebo text, and the wording of the skills.
