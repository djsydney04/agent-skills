# Evaluate the testing skills

Use these cases to check decisions, not exact wording. Frontmatter and link validation cannot establish that an agent distinguishes useful tests from busywork.

1. Start a fresh agent with the five skills available. Give it one prompt from [prompts.md](prompts.md), including that case's supplied context. Do not include the rubric or prior conclusions.
2. Let it select and read the relevant skill and shared principles. For manual evaluation, record which files it actually read.
3. Compare its answer and any changes with [rubric.md](rubric.md). Judge the protected behavior, boundary, oracle, scope, and honesty about execution. Do not grade by headings or matching phrases.
4. Record the skill revision, prompt, agent/model, selected skills, result, and specific evidence. Repeat affected cases after a substantive instruction change.

These are text-based behavioral cases. They do not include a runnable application, and an agent must not claim to have executed the supplied sketches. For implementation or stabilization in a real project, additionally verify the produced tests and changes in that project's actual harness.

The initial draft was evaluated with independent agents; PR validation describes which cases were exercised. Cases not exercised remain proposed evaluations.
