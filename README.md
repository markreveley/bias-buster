# bias-buster

Skills that help an agent notice and counter cognitive biases during collaborative work, both its own and the user's. Each skill describes one bias, why it happens, how to spot it and what to do instead. A skill is kept only if it measurably beats the same model without it.

## Status

Planning. Nothing is built yet.

- [docs/pilot-plan.md](docs/pilot-plan.md): the plan for a three-skill pilot (suggestion anchoring, caving to pushback, premature closure), the eval method and the rule for keeping or cutting a skill. Proposed, awaiting approval.
- [docs/original-brief.md](docs/original-brief.md): the brief this started from and the context behind it.

## Intended layout

```
.claude-plugin/plugin.json
skills/<name>/SKILL.md
evals/            scenarios, runner, results
references.md     primary sources for each bias
```
