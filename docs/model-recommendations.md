# Model Recommendations for AI Coding

> Which models to use for AI-assisted software engineering. Based on research in the user's Obsidian vault ([OpenRouter Models for Code Review]) and the Country Time Spades [model selection doc](https://github.com/drgtwilliams/country-time-spades/blob/main/docs/model-recommendations.md).

---

## The Stack

| Job | Best fit (quality) | Best value (cheap) | Local (free, offline) |
|---|---|---|---|
| **Architecture & heavy reasoning** | GLM 5.2 (max reasoning) | DeepSeek V4 Pro | DeepSeek-R1 (distilled) |
| **Code generation (bulk)** | GLM 5.2 `/` 5.3 | GLM 5.3 Flash ($0.08/M) | Qwen3-Coder 30B |
| **Code review** | GLM 5.2 | Qwen3-Coder | DeepSeek-Coder V3 |
| **Debugging** | DeepSeek V4 Pro | GLM 5.3 | DeepSeek-R1 |
| **Boilerplate / scaffolding** | Qwen3-Coder | Qwen3-Coder | Qwen3-Coder 30B |
| **Documentation** | Claude Sonnet | GLM 5.3 | Llama 3.3 70B |

---

## Token Efficiency Rules

1. **Iterate locally on Qwen3-Coder 30B** for free; escalate to GLM 5.2 (cloud, ~$0.39/1M input) when context or reasoning outgrows your GPU.
2. **Use `:floor` routing** (`z-ai/glm-5.3:floor`) to always hit the cheapest OpenRouter provider.
3. **Reserve frontier (Claude) only for genuine escalation** — a stuck bug, a hard architecture call. Budget-cap it.
4. **Prompt-cache aggressively** — stable context (project docs, codebase) stays cached, making follow-ups ~90% cheaper.

---

## Key Model IDs

| Model | OpenRouter ID |
|---|---|
| GLM 5.2 | `z-ai/glm-5.2` |
| GLM 5.3 Flash | `z-ai/glm-5.3` |
| DeepSeek V4 Pro | `deepseek/deepseek-v4-pro-0813` |
| Qwen3-Coder | `qwen/qwen-3-coder` |
| Claude Sonnet 4.6 | `anthropic/claude-sonnet-4.6` |

*Note: model availability and pricing change. Check [openrouter.ai/models](https://openrouter.ai/models) for current status.*