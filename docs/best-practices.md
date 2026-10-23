# Best Practices: AI Prompt Engineering for Production Software

> The patterns that produce production-ready code — not tutorial-level output.

---

## 1. The CRTSE Framework

The single most reliable prompt structure for software engineering. Every effective prompt hits all five dimensions:

### Context — Tell the AI about your project environment

- **Tech stack:** language, framework, libraries, versions, runtime (Python 3.12, .NET 8, Java 21, Node 22, etc.)
- **Project type:** API server, CLI tool, frontend SPA, microservice, data pipeline, embedded system
- **Architecture patterns:** MVC, event-driven, CQRS, hexagonal, repository pattern, etc.
- **Constraints:** resource limits (memory, latency budget), deployment target (Docker, serverless, bare metal)

**Bad:** "Write a REST API endpoint."
**Good:** "I'm building a FastAPI microservice (Python 3.12, SQLAlchemy 2.0 async, PostgreSQL 16). Architecture follows the repository pattern with Pydantic v2 for schemas. The service runs in Docker with a 512 MB memory limit."

### Role — Define who the AI should be

This genuinely shifts model output quality. Name a seniority level and a specialization.

- "Act as a senior Python backend engineer specializing in async microservices."
- "You are a principal C# engineer with 15 years of experience in high-throughput financial systems."
- "You are an embedded systems engineer who writes MISRA-C compliant code for ARM Cortex-M."

### Task — One specific deliverable with clear boundaries

- Be specific about the deliverable: "a single function," "a module of 3 classes," "a complete Express route handler."
- State what the output should include: "Include docstrings," "Include error handling," "Return a fully typed function."
- One task per prompt for best results.

### Standards — Define quality requirements and constraints

- **Type safety:** "All functions must be fully typed with mypy --strict compliance."
- **Error handling:** "Use explicit error types, no bare `except:` clauses, log at the boundary."
- **Code style:** "Follow PEP 8 / C# conventions / Google Java style guide."
- **Testing:** "Include pytest fixtures and parameterized test cases."

### Examples — Show what "good" looks like

- A function signature from your existing codebase.
- The pattern you want the output to follow.
- A one-liner that demonstrates the naming convention.

**The rule:** if you'd be unhappy with the output because it didn't follow your team's conventions, include an example.

---

## 2. The Makefile / DAG Pattern (for Agent Workflows)

*Source: Stuart Feldman's `make` (1976 Bell Labs) — applied to AI agents by modern engineering teams.*

The problem with "Step 1, Step 2, Step 3… Step 20" prompts: LLMs are probabilistic next-token predictors, not imperative state machines. After 150 turns, the model loses positional context and skips steps.

### Instead: define a Directed Acyclic Graph (DAG) of targets

Each target has:
- **Prerequisites** — what must be true before this target can be satisfied
- **Invariant** — the physical condition that proves the target is done (test pass, CI green, PR merged)
- **Recipe** — what action produces the target

### Example: From issue to landed PR

```
finish: land
land: ready-to-land human-landing-approval
ready-to-land: ci-quiescent changelog-updated
ci-quiescent: push ci-remote-green all-bots-triaged
push: commit human-push-approval
commit: pre-commit-clean human-commit-approval
pre-commit-clean: approved-adversarial-report lint-clean
approved-adversarial-report: adversarial-review human-review-approval
adversarial-review: implementation-diff
implementation-diff: approved-plan tdd-passing
approved-plan: researched-ticket human-plan-approval
```

### Why this works

- **Self-healing:** if a bot comment invalidates the PR, the DAG auto-reroutes through `adversarial-review` before re-commit — no "go back to Step 5" instructions.
- **Fixed-point evaluation:** never count loop iterations — just check whether the invariant holds. Once it does, the target settles.
- **O(1) cognitive load:** the agent never asks "what step am I on?" — it only asks "what's the deepest unsatisfied prerequisite?" The filesystem and git graph are the state machine.
- **Session resumption:** inspect physical state (git log, PR status, CI checks, working tree) to locate the first unsatisfied target. Zero reliance on conversational memory.

### When to use the Makefile pattern

Use it for **multi-step agent workflows** (autonomous coding agents, CI pipelines, deployment orchestrators). Use CRTSE for **single-turn code generation** (one function, one test, one endpoint).

---

## 3. Negative Constraints

Tell the model what NOT to do — this eliminates entire categories of bad output without listing every edge case.

### Universal negative constraints

```
- No `any` / `Any` / `object` types. Everything must be explicitly typed.
- No `console.log` / `print()` in production code paths. Use proper logging.
- No bare `except:` / `catch (Exception) {}` that swallows errors. Always log or rethrow with context.
- No deprecated APIs. Use the latest stable API for the specified version.
- No hardcoded secrets, URLs, or magic numbers. Use config/env/constants.
- No N+1 queries. Use eager loading or batch queries for collections.
- No mutable default arguments (Python). No raw `new Thread()` (Java — use executors).
```

### Domain-specific constraints

| Domain | Negative Constraint |
|---|---|
| APIs | No unauthenticated endpoints; no exposing stack traces in 500 responses |
| Web | No layout shift (CLS violations); no blocking the main thread |
| Mobile | No main-thread I/O; no memory leaks from retained fragments/activities |
| Embedded | No dynamic allocation after init; no recursion; no floating-point in ISRs |
| Robotics | No unsafe motions without collision checks; no unreachable timeout values |

---

## 4. Prompt Chaining

One mega-prompt is fragile. Chain separate prompts — each building on the output of the last.

### The standard chain for building a feature

1. **Architecture prompt** → Agree on structure, file layout, interfaces
2. **Implementation prompt** → Build against the agreed structure
3. **Review prompt** → Catch issues (security, performance, edge cases)
4. **Test prompt** → Generate tests from the implementation and the spec
5. **Documentation prompt** → Generate docs from the final code and architecture decisions

### Why chaining works

- Each prompt's context is small and focused — the model doesn't lose track.
- The output of prompt N becomes the context + example for prompt N+1.
- You can insert human review between any two prompts without breaking the flow.
- Cache hits: stable context (the agreed architecture, the implemented code) stays cacheable across turns.

---

## 5. Language-Agnostic Prompt Template

```
Act as a senior [language/framework] engineer.

I'm building [project type] using [tech stack with versions]. 
The project follows [architecture pattern].

Task: [specific deliverable — one function, one class, one module].

Requirements:
- [Type safety constraint]
- [Error handling constraint]
- [Performance constraint if applicable]
- [Testing requirement]

Do NOT: [negative constraints specific to this task].

Here's the pattern from our codebase:
[paste a function signature, class definition, or style example]

Output: [what exactly you expect — a code block, a file, a diff, an explanation].
```

---

## 6. Token Efficiency Tips

1. **Never paste the whole repo.** Reference file paths; let the model read what it needs.
2. **Leverage prompt caching.** Stable context (system prompts, project docs) stays cached — ~90% cheaper on follow-up calls.
3. **Use structured output formats.** JSON schemas, YAML configs, typed signatures — less ambiguity = fewer re-rolls.
4. **Iterate locally first.** Prototype prompt quality on a local model (see model recommendations doc) before burning cloud tokens.

---

## Sources

- CRTSE framework adapted from [Albert Salgueda — Best AI Prompts for Developers 2026](https://dev.to/albertsalgueda/best-ai-prompts-for-developers-in-2026-the-complete-guide-to-ai-assisted-coding-38h0)
- Makefile/DAG pattern from [Stuart Feldman Was Right in 1976 — Why Your AI Agent Needs a Makefile](https://dev.to/gde/stuart-feldman-was-right-in-1976-why-your-ai-agent-needs-a-makefile-not-a-20-step-prompt-5bn2)
- Microsoft PromptCraft-Robotics ([github.com/microsoft/PromptCraft-Robotics](https://github.com/microsoft/PromptCraft-Robotics))
- AI Prompt Best Practices for Embedded Software — embedded.com