# Agent Skills

A collection of focused workflows for coding agents. The first collection is about testing: understanding what makes a test useful, auditing an existing suite, choosing what to test, writing tests, and keeping them trustworthy.

The preference is practical: use integration tests across real application boundaries and a small set of end-to-end tests for critical journeys. Keep unit tests that protect meaningful rules. Avoid tests that only prove a mock returned its configured value or that the implementation calls itself in the expected order.

## Testing collection

| Skill | Use it to |
| --- | --- |
| [test-principles](skills/test-principles/SKILL.md) | Establish the standard for a good test and choose useful boundaries. |
| [test-audit](skills/test-audit/SKILL.md) | Review an existing suite and prioritize gaps, weak assertions, and maintenance costs. |
| [test-plan](skills/test-plan/SKILL.md) | Turn a feature or change into a focused, risk-based test plan. |
| [test-write](skills/test-write/SKILL.md) | Implement meaningful regression, integration, end-to-end, or unit tests. |
| [test-stabilize](skills/test-stabilize/SKILL.md) | Diagnose unreliable tests and fix the cause without weakening their guarantees. |

Each workflow reads the shared principles before making testing decisions. The [paired examples](skills/test-principles/references/examples.md) explain what weak tests miss, what stronger tests prove, and which plausible defects they catch.

Start an existing-suite review with `test-audit`. A new feature can start with `test-plan`; a concrete regression can go directly to `test-write`. Use `test-stabilize` for intermittent failures. These are separate entrypoints, not a mandatory sequence.

## Use the skills

Each directory under `skills/` contains a `SKILL.md` with a name, discovery description, and instructions. There are no required MCP servers, paid services, or testing frameworks. Code examples are illustrative; use the target project's language and tooling.

Install the five testing skills together as sibling directories. The workflows depend on the shared `test-principles` skill; copying only an individual workflow leaves its principles link unresolved.

For a repository using Codex, copy the five directories into its `.agents/skills/` directory. For example, after cloning this repo, run from the target repository root:

```sh
mkdir -p .agents/skills
cp -R /path/to/agent-skills/skills/test-* .agents/skills/
```

Check for existing directories with the same names before copying and reconcile them instead of overwriting local guidance. Other Agent Skills hosts may use a different discovery directory; preserve the sibling layout. See the [official Codex skill documentation](https://learn.chatgpt.com/docs/build-skills) for discovery and invocation details.

Example Codex prompts:

```text
Use $test-audit to review our checkout tests. Prioritize risks and give evidence before suggesting changes.
Use $test-plan to decide what to test for account deletion, including failures and recovery.
Use $test-write to add a regression test for duplicate webhook delivery.
Use $test-stabilize to investigate the intermittent order-history test failure.
```

## Review and evolve

This is an initial draft. The skills express a testing preference rather than a fixed test pyramid, a coverage percentage, or a test-count quota. Adapt decisions to the product's risks and existing suite.

Use the [behavioral evaluation cases](evals/README.md) when changing the instructions. Validate each skill with the validator available in your skill-authoring tool, and check relative links after moving files. Format validation checks packaging; behavioral cases check whether the guidance produces useful decisions.
