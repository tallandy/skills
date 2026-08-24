# Agent Skills

[![skills.sh](https://skills.sh/b/tallandy/skills)](https://skills.sh/tallandy/skills)

Small, opinionated engineering skills for coding agents.

I'm a CTO who still spends a lot of time writing, reviewing and debugging production code. These skills grew out of recurring situations where AI coding agents are extremely useful, but need stronger engineering constraints around *how* they approach the problem.

They are deliberately conservative. Measure things. Prove things. Preserve behaviour. Don't confidently delete half the application because `grep` came back empty 😀

## Skills

### [`dead-code-removal`](./skills/software/dead-code/SKILL.md)

**Prove and safely remove unreachable or unused code.**

Dead code removal is treated as a proof exercise rather than a search-and-delete exercise.

The skill makes the agent:

- establish the repository baseline first
- map production reachability and package boundaries
- distinguish candidate generation from proof
- account for dynamic, framework and configuration-driven usage
- classify candidates by confidence
- remove only high-confidence dead code
- validate each coherent deletion
- preserve ambiguous code rather than guessing

The core idea:

> Absence of a reference is evidence, not proof.

### [`improve-codebase-performance`](./skills/software/improve-codebase-performance/SKILL.md)

**Find and fix measured performance bottlenecks without changing behaviour.**

Performance work is similarly evidence-driven. The agent must establish a repeatable baseline before it starts "optimising".

The skill makes the agent:

- define the exact code path and performance metric
- build a repeatable measurement loop
- profile before changing code
- identify the actual bottleneck
- make the smallest high-leverage change
- compare before and after measurements
- preserve behaviour, correctness and public interfaces

The core idea:

> If you didn't measure it, you don't know that you made it faster.

## Philosophy

Both skills follow roughly the same engineering principles:

1. **Evidence before action.** A hunch can generate a candidate, but it cannot prove the conclusion.
2. **Small changes beat clever rewrites.** Change the minimum necessary to achieve the goal.
3. **Preserve behaviour by default.** Cleanup and optimisation should not quietly become redesigns.
4. **Use the repository's own feedback loops.** Tests, builds, linters, typecheckers and benchmarks are part of the evidence.
5. **Uncertainty is allowed.** If something cannot be proven safe, say so and leave it alone.

Coding agents are very good at doing things quickly. These skills are mostly about making sure they are doing the *right* thing quickly.

## Installation

Using the [skills CLI](https://skills.sh/docs):

```bash
npx skills add tallandy/skills
```

To see the available skills first:

```bash
npx skills add tallandy/skills --list
```

Or install a specific skill:

```bash
npx skills add tallandy/skills --skill dead-code-removal
```

```bash
npx skills add tallandy/skills --skill improve-codebase-performance
```

The skills CLI supports Claude Code, Codex, Cursor and many other coding agents.

## Invocation

These skills are intentionally configured for **explicit invocation**.

I generally don't want an agent deciding by itself that today would be a lovely day to delete some code.

Invoke them using the syntax supported by your agent, for example:

```text
/dead-code-removal
```

or:

```text
$dead-code-removal
```

Likewise for performance work:

```text
/improve-codebase-performance
```

or:

```text
$improve-codebase-performance
```

## Inspiration

I can't pretend this repository appeared in a vacuum.

It was heavily inspired by [Matt Pocock's excellent skills repository](https://github.com/mattpocock/skills), particularly the idea that an agent skill should encode a repeatable engineering discipline rather than just contain a bag of tips.

If you like the idea of engineering-focused agent skills, you should absolutely have a look at his repo. It is larger, more mature and full of very good ideas.

The two skills currently in this repository, **dead code removal** and **codebase performance**, are my own work based on problems and failure modes I've encountered in real software development.

AI helped me research, challenge, edit and refine them, which seems worth being transparent about. The opinions and engineering constraints are mine; the robots helped me sharpen them.

## Contributions

Issues, edge cases and improvements are welcome.

In particular, if one of these skills confidently does something stupid, please open an issue. Those are exactly the failure modes worth turning into better instructions.

PRs are welcome too, provided they keep the skills focused, evidence-driven and reasonably concise.

## Status

This is a small collection at the moment.

I'd rather have a few skills I actually use and trust than fifty that looked clever for ten minutes.
