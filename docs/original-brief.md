# Original brief and context

*Prepared November 22, 2026. Mark's words from the thread that started this repo, kept verbatim. The brief is superseded by [pilot-plan.md](pilot-plan.md): Mark chose a three-skill pilot over building all fourteen at once.*

## The brief

Build a Claude plugin of skills that help an agent resist cognitive biases, both its own (as an LLM) and the user's, during collaborative work: brainstorming, specs, decisions, reviews, creative work.

### Design principles
- Each skill describes ONE bias plainly. No admonitions, no CAPS, no "you MUST". Newer Claude models follow explained reasoning better than rules, and heavy instructions over-trigger.
- Every skill has this structure:
  1. What the bias is (2–3 sentences, in plain words)
  2. Why it happens (the mechanism)
  3. How to notice it in a live conversation (concrete signals, with a short example)
  4. What to do instead (one or two concrete moves)
  5. When NOT to apply it (to prevent over-correction, e.g. don't withhold suggestions from a user who asked for them)
- The description frontmatter should trigger only in situations where the bias is likely, not on every message.
- Keep each SKILL.md under ~60 lines. Shorter is better if it still works.
- Plugin layout: .claude-plugin/plugin.json plus skills/<name>/SKILL.md. Do NOT use a top-level bin/ directory (the Claude app refuses those). Put helper scripts under the skill's own folder if you need any.

### Candidate biases (research each; drop any whose countermeasure you can't make concrete)
User-side: anchoring on offered examples, framing effects, confirmation bias, sunk cost, the planning fallacy, availability bias, the IKEA effect / attachment to one's own idea.
LLM-side: sycophancy (agreeing with or caving to pushback without new evidence), anchoring on the user's first guess or hypothesis, premature closure (settling on the first plausible answer), position and recency bias in long context, verbosity bias, overconfidence in its own earlier output, mirroring the user's framing instead of checking it.

### Research first
For each bias, find the primary source (e.g. Tversky & Kahneman 1974 for anchoring; Anthropic and academic papers for LLM sycophancy) and cite it in a references.md, not in the skill body.

### Evals (required; a skill that doesn't beat baseline gets cut)
For each skill, build 5+ scenarios with a simulated user who has a hidden goal written as concrete, checkable requirements. Run each scenario with and without the skill (same model, 3+ runs each). Measure: requirements met, number of turns, how often the outcome echoes the assistant's suggestions rather than the hidden goal, and whether the skill caused over-correction (e.g. refusing to recommend when asked). Report the results in a table. Keep a skill only if it clearly improves its target metric without hurting the others.

### Deliverables
The plugin folder, references.md, the eval scenarios and runner, a results table, and a short README stating which skills survived and why.

## Context: how this request came about

1. I installed Superpowers (obra/superpowers) as a plugin in the Claude app. Superpowers is a set of skills that pushes Claude through a fixed workflow (brainstorm → plan → TDD → review), and a hook at session start tells the model to use the skills. It works in the app.

2. I questioned whether one of its instructions actually helps. The brainstorming guidance says:

   "Get them describing. Until you know why they want it, ask open questions, with no example answers attached. Their own words carry intent that your options can't. Once you know why, when a decision has choices you can name, sketch each concretely, with what it trades off and which you'd pick. When they say 'just guess' or 'what do you think?', propose something concrete."

   My objections:
   - "Until you know why" has no stopping rule. How does a model decide it knows "why"?
   - "Choices you can name" may just be a comparative "what", not a "why".
   - "Propose something concrete" may be insurance against a failure models rarely have. Is it worth the tokens?
   - The whole thing felt like subjective voodoo with no evidence behind it.

3. What we concluded:
   - You can't know whether an instruction like this helps without evals. Superpowers' own testing checks that the agent *follows* its skills, not that following them helps users.
   - The "no example answers" line is meant to stop the user from anchoring on options the model offers. User researchers ask open questions before closed ones for the same reason.
   - "Propose something concrete" is probably a counterweight to the skill itself, so that "keep asking questions" doesn't turn "what do you think?" into another question. The token cost is trivial. The real cost is extra turns.
   - Anthropic's current direction supports the skepticism. They removed more than 80% of Claude Code's system prompt for Claude 5-generation models with no measurable loss, and they now recommend letting the model use judgement over adding rules. They still recommend explaining the reasoning behind any instruction you keep.
   - A credible eval: a simulated user with a hidden goal written as concrete, checkable requirements, runs with and without the skill, and measures of requirements met, turns taken, and how often the outcome echoes the assistant's suggestions rather than the hidden goal. Real usage data is better still: corrections, redos, abandonment.

4. The idea that came out of this: a set of skills aimed at cognitive biases in both the human and the LLM (anchoring, sycophancy, premature closure, etc.). Each skill would describe the bias, how it works, how to spot it, and what to do instead, rather than issuing admonitions. Each skill has to prove itself in evals against a baseline or be cut. That's what the prompt asks you to build.

## Constraints that matter
- It will be used in the Claude app as well as Claude Code. The app ignores some plugin features and refuses plugins with a top-level bin/ directory.
- Keep skills short and explanatory. Over-instruction is the failure mode we're trying to avoid.
- I care more about measurable improvement than about how polished the skills look.

## A later thought from Mark

one thought - does being aware of cognitive bias help avoid it? perhaps the skill should make explicit the times and ways that cognitive biases might arise, to give the llm or the human the chance to rephrase, so they actually learn how to overcome cognitive bias vs have automated skill based protection. just a thought, don't know if backed by science
