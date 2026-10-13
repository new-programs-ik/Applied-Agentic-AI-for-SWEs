# Course API Costs (OpenRouter)

Measured costs of running each course notebook on the shared OpenRouter key. Figures are the per-call costs OpenRouter reports, at OpenRouter's prices on the measurement date.

## Week 1

### Week 1/Calling_LLMs_programmatically_openrouter.ipynb

- **Measured on:** 2026-10-13
- **Key required:** `OPENROUTER_APIKEY` (the Gemini SDK cell is optional and uses a free Google AI Studio key)
- **Test prompt:** "What is the point of using nosql databases in engineering systems in under 3 sentences"

| Cell | Model | Tokens (in / out / reasoning) | Cost (USD) |
|---|---|---|---|
| OpenAI SDK | `openai/gpt-4.1-mini` | 30 / 94 / 0 | $0.000116 |
| Anthropic SDK | `anthropic/claude-haiku-4.5` | 31 / 106 / 0 | $0.000561 |
| Gemini SDK (optional) | `gemini-2.5-flash` (Google AI Studio free tier) | — | $0 |
| OpenRouter → Claude | `anthropic/claude-haiku-4.5` | 22 / 132 / 0 | $0.000545 |
| OpenRouter → Gemini | `google/gemini-2.5-flash` | 22 / 103 / 0 | $0.000160 |
| OpenRouter → Grok | `x-ai/grok-4.3` | 22 / 130 / 293 | $0.001069 |
| OpenRouter → DeepSeek | `deepseek/deepseek-r1` | 41 / 259 / 138 | $0.000484 |
| **Total per full run** | | | **≈ $0.0029** |

**Budget estimate**

| Scenario | Estimated cost |
|---|---|
| One full run | ≈ $0.003 |
| One learner (≈10 runs, including experimenting) | ≈ $0.03 |
| Cohort of 100 learners | ≈ $3 (plus OpenRouter's ~5.5% fee on credit purchases) |

**Notes**

- Grok is the most expensive cell because it bills hidden reasoning tokens as output.
- Reasoning token counts vary from run to run, so actual costs will vary slightly.
- The main cost risk is learners trying premium models with long prompts. For example, `openai/gpt-5.5-pro` costs $30 / $180 per million input / output tokens. Set a credit limit on each shared key.

### Week 1/CRM_Lead_Qualifier_Agent.ipynb

- **Measured on:** 2026-10-13
- **Key required:** `OPENROUTER_APIKEY`
- **Model:** `openai/gpt-4o` ($2.50 / $10 per million input / output tokens), set by `MODEL` in the client cell
- **Runs measured:** 2 clean runs ($0.0122 and $0.0123). The table shows the latest run.
- **Pattern:** a function-calling agent loop, with 3 model calls per lead (call the lookup tools → call the scoring tool → write the summary)

| Scenario | Model call | Tokens (in / out) | Cost (USD) |
|---|---|---|---|
| 1 – jane@acmecorp.com (Medium) | Think → `lookup_domain_info` + `check_crm_history` | 412 / 23 | $0.001375 |
| | Think → `calculate_lead_score` | 486 / 72 | $0.002037 |
| | Final summary | 566 / 155 | $0.002887 |
| 2 – bob@widgetco.net (High) | Think → `lookup_domain_info` + `check_crm_history` | 410 / 23 | $0.001320 |
| | Think → `calculate_lead_score` | 487 / 76 | $0.001982 |
| | Final summary | 570 / 147 | $0.002695 |
| **Total per full run** | | | **≈ $0.012** |

**Budget estimate**

| Scenario | Estimated cost |
|---|---|
| One full run (both scenarios) | ≈ $0.012 |
| One learner (≈10 runs, including testing other emails) | ≈ $0.12 |
| Cohort of 100 learners | ≈ $12 (plus OpenRouter's ~5.5% fee on credit purchases) |

**Notes**

- About 4× the cost of the Calling LLMs notebook. Input tokens dominate, because every loop iteration resends the system prompt, the tool schema and the growing message history.
- Changing `MODEL` to a cheaper tool-calling model cuts the cost sharply. For example, `openai/gpt-4.1-mini` costs $0.40 / $1.60 per million tokens, about 6× cheaper than gpt-4o.
- If the model loops or retries tool calls, a run can cost more than shown here.

### Week 1/project/Fix_Failing_Tests_Agent.ipynb

- **Measured on:** 2026-10-13
- **Key required:** `OPENROUTER_APIKEY`
- **Model:** `openai/gpt-4.1-mini` ($0.40 / $1.60 per million input / output tokens), set by `MODEL` in the client cell
- **Runs measured:** 4 full end-to-end runs
- **Pattern:** Part 1 is a fixed workflow (3 model calls). Part 2 is a coding-agent loop where the model decides the number of steps (9 to 15 model calls in our runs)

| Part | Model calls | Tokens (in / out) | Cost (USD) |
|---|---|---|---|
| Part 1 – workflow | 3 | — | ≈ $0.0023 |
| Part 2 – agent (cheapest run) | 9 | 18,453 / 1,311 | $0.0059 |
| Part 2 – agent (most expensive run) | 15 | 38,155 / 1,522 | $0.0079 |
| **Total per full run** | | | **≈ $0.008 to $0.010** |

**Budget estimate**

| Scenario | Estimated cost |
|---|---|
| One full run | ≈ $0.009 |
| One learner (≈15 runs, including exercises) | ≈ $0.14 |
| Cohort of 100 learners | ≈ $14 (plus OpenRouter's ~5.5% fee on credit purchases) |

**Notes**

- Agent cost varies with the number of steps the model takes. Input tokens dominate, because each step resends the full history, including file contents and test output.
- `max_steps=20` caps a runaway run at roughly $0.03 with this model.
- Switching `MODEL` to `openai/gpt-4o` costs about 6× more per run. Premium models can cost far more.

### Week 1 total

| | Per run of every notebook | Per learner | Cohort of 100 |
|---|---|---|---|
| Week 1 | ≈ $0.024 | ≈ $0.29 | ≈ $29 |
