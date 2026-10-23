# Knowledge Skills — AI Prompt Engineering for Production Software

> A curated, structured library of prompt engineering best practices and example prompts for building production-grade software with AI coding assistants (Claude, ChatGPT, Gemini, Copilot, Codex, Hermes).

**Coverage:** Python, C#, Java, web development, robotics/embedded, architecture, testing, DevOps, and more.

---

## Why This Exists

AI coding tools are ubiquitous — but according to Stack Overflow's 2025 Developer Survey, over 70% of developers use them, yet fewer than 30% report consistently useful output for production code. **The gap isn't the model — it's the prompt.**

This repository captures what works: prompt structures, patterns, and concrete examples that produce code you can actually ship — across languages, domains, and development phases — without burning tokens on rework.

---

## Navigation

| Document | What's Inside |
|---|---|
| [Best Practices](docs/best-practices.md) | CRTSE framework, Makefile/DAG prompting, negative constraints, prompt chaining |
| [Python Prompts](docs/prompts-python.md) | Data pipelines, async services, CLIs, ML, FastAPI, Django |
| [C# & .NET Prompts](docs/prompts-csharp-java.md) | ASP.NET, Entity Framework, WPF, Unity, microservices |
| [Java Prompts](docs/prompts-csharp-java.md#java) | Spring Boot, Android, enterprise patterns, stream processing |
| [Web Development](docs/prompts-web.md) | React, Angular, Next.js, full-stack, accessibility, performance |
| [Robotics & Embedded](docs/prompts-robotics.md) | ROS, microcontrollers, drone control, physical reasoning, safety |
| [General Engineering](docs/prompts-general.md) | Architecture, testing, debugging, DevOps, code review, docs |

---

## Quick Start: The CRTSE Framework

The most effective prompts follow a consistent structure:

1. **C**ontext — tech stack, project type, architecture patterns
2. **R**ole — who the AI should "be" (senior engineer, DevOps specialist, etc.)
3. **T**ask — one specific deliverable with clear boundaries
4. **S**tandards — type safety, error handling, code style, constraints
5. **E**xamples — a function signature, a code pattern from your codebase

**Quick example:**
> Act as a senior Python backend engineer. Create a FastAPI endpoint for `POST /api/users`. Use Pydantic v2 for validation, async SQLAlchemy for persistence, and return proper HTTP status codes. No `Any` types. Include input validation errors as 422 responses.
> Here's the pattern our existing endpoints follow:
> ```python
> @router.post("/api/items")
> async def create_item(body: ItemCreate, db: AsyncSession = Depends(get_db)) -> ItemResponse:
>     ...
> ```

---

## Core Principles

1. **Treat prompts like technical specifications.** Vague input = vague output. Specs with constraints = production code.
2. **One task per prompt.** Chain prompts for complex work; don't write essay-prompts.
3. **Negative constraints eliminate entire categories of bad output.** "No `Any` types" / "No swallowed exceptions" / "No deprecated APIs."
4. **Prompt chaining beats mega-prompts.** Architecture → Implementation → Review → Tests → Docs — separate prompts, each building on the last.
5. **The Makefile/DAG pattern supersedes numbered checklists.** Define targets, prerequisites, and invariants — not "Step 1, Step 2, Step 3."

---

## License

MIT — use, modify, and share freely. Contributions welcome via PR.