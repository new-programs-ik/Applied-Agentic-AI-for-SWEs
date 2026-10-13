# Week 1 Project: Fix-the-Failing-Tests Coding Agent

A coding agent, built from scratch in about 150 lines of Python, that takes a small repository with a failing test suite and works until every test passes. It explores the code, finds the root causes, edits the source files and re-runs `pytest`, deciding each step itself.

**Notebook:** [`Fix_Failing_Tests_Agent.ipynb`](Fix_Failing_Tests_Agent.ipynb) ([open in Colab](https://colab.research.google.com/github/new-programs-ik/Applied-Agentic-AI-for-SWEs/blob/main/Week%201/project/Fix_Failing_Tests_Agent.ipynb))

## What learners build

| Part | What it is | What it shows |
|---|---|---|
| Step 1 | A tiny buggy library, `shopkit` (pagination, pricing, cart, slugs), with 11 tests, 8 of them failing | A realistic "CI is red" situation, created by the notebook so nothing needs to be uploaded |
| Step 2 | Four tools: `list_files`, `read_file`, `write_file`, `run_tests`, with guardrails | Tools act on real files and run real code. Nothing is simulated |
| Part 1 | A fixed **workflow**: one LLM call per failing test file, patch, done | Predictable and cheap, but it only looks where the developer told it to |
| Part 2 | An **agent** loop (Think → Act → Observe) with a step budget and cost tracking | The model chooses which files to read, follows imports to the real bug, and revises fixes that fail |
| Step 3 | Independent verification: re-run the tests and review the diff like a pull request | Never trust an agent's "done" |
| Exercises | Six extensions: new bugs, a search tool, red-teaming the guardrails, planning, model comparison, snippet edits | Practise and portfolio work |

The planted bugs are designed to separate a workflow from an agent:

1. **Off-by-one in `paginate`**: the easy warm-up.
2. **`total_pages` rounds down**: a second bug in the same file.
3. **`apply_discount` returns the discount, not the discounted price**: the bug is in `pricing.py`, but only `test_cart.py` fails. The workflow never shows the model `pricing.py`. The agent has to follow the import.
4. **`slugify` only handles single spaces**: the obvious regex fix still fails the apostrophe and accent tests, so the agent often has to read the new failure and revise its fix (reflection).

## Why this project suits Week 1

### 1. It uses exactly what Week 1 teaches, and every part of it

| Week 1 class topic | Where the project uses it |
|---|---|
| Calling LLMs programmatically | Same `openai` SDK + OpenRouter client as the "Calling LLMs" notebooks |
| Prompt engineering (context, structure, constraints, iterate) | The system prompt uses the `## Role / ## Task / ## Rules / ## Output` structure from the prompt engineering slide |
| Limitations of LLMs (no actions, no memory, hallucinations) | Tools give the model actions. The message history gives it memory. `run_tests` catches confident but wrong fixes |
| Agent = (Prompt + Tools + Memory) × LLM | Each part is a visible, separate piece of code |
| Agents and tools (retrieve, act, run logic) | `read_file` retrieves, `write_file` acts, `run_tests` executes code |
| Workflows vs. autonomous agents | Learners build both on the same problem and compare them in a table |
| The agent loop / ReAct | The loop prints every THINK, ACT and OBSERVE step |
| Planning and reflection | Fixes that fail produce a new observation, and the agent has to revise |

### 2. It doesn't need anything from later weeks

No agent frameworks (LangChain or LangGraph), no RAG or vector stores, no MCP, no deployment, no evals tooling. The whole loop is a plain `for` loop that learners can read top to bottom. When frameworks arrive in later weeks, learners will know what those frameworks are doing for them.

### 3. It is relevant to software engineers

The class opens with GitHub Copilot agent mode and AI IDEs writing code, running tests and opening pull requests. This project has learners build a small version of exactly that, on a task every engineer has done by hand: turning a red CI run green. Pagination, pricing and slug bugs are everyday backend code, so no domain explanation is needed.

### 4. The agent's work is real and checkable

The tools edit real files and run real `pytest`. Success is decided by the test suite, not by the model saying it is done and not by the instructor's judgement. This gives a clear lesson about hallucination (wrong fixes get caught) and an objective score for exercises (tests passed, steps taken, cost).

### 5. Learners see the agent behaving like an agent

The agent decides which files to read, finds a bug in a file no test points to, and in about half of our test runs revised a fix after a test failure. The number of model calls changes from run to run (9 to 15 in our test runs), which is the defining feature of an autonomous agent from the slides.

### 6. It teaches production habits from day one

Each guardrail is small enough for Week 1, but they are the habits that matter later:

- **Sandboxed paths:** the agent can't read or write outside the repository.
- **Protected tests:** the agent can't "pass" by editing the tests.
- **Errors as observations:** tools return error messages instead of crashing the loop.
- **Truncated outputs:** long tool results are cut short to control tokens.
- **Step budget:** `max_steps` stops runaway loops.
- **Cost tracking:** every run prints its token count and cost.
- **Independent verification and diff review:** check the result yourself.

### 7. It is practical to run in a live class

- Runs in Colab with no installs (`openai` and `pytest` are preinstalled) and one secret, `OPENROUTER_APIKEY`.
- The repository is generated by the notebook, so there are no files to upload. `create_repo()` resets it at any time.
- An end-to-end run takes 1 to 2 minutes and costs about $0.008 (measured below).
- If the default model gets everything right on the first try, switching `MODEL` to a smaller model reliably shows more reflection and retries.

### 8. It sets up the rest of the course

The same project can grow with each later week: a `search_code` tool, then memory across runs, then a coder + reviewer multi-agent setup, then evals (test pass rate across many seeded bugs), then observability (the trace is already printed), and finally deployment as a CI bot.

## Suggested teaching plan (about 55 minutes)

| Time | Section | Instructor focus |
|---|---|---|
| 5 min | Step 1: create the repo, run the tests | "This is what the agent will see." Don't reveal where the bugs are |
| 10 min | Step 2: tools and guardrails | Run the guardrail cell. Ask: "Why must the tests be read-only?" |
| 10 min | Part 1: the workflow | Predict together what will happen, run it, then run `show_diff()` |
| 15 min | Part 2: the agent | Walk through the 30-line loop. Run it and read the trace aloud, pointing out exploring, acting on evidence, reflection and stopping |
| 5 min | Step 3: verify | "Would you merge this PR?" Review the diff together |
| 10 min | Comparison and exercises | Workflow vs. agent table. Assign two exercises plus the README and GitHub portfolio step |

## Measured results (2026-10-13, `openai/gpt-4.1-mini`)

Four full end-to-end runs of the notebook:

| | Workflow (Part 1) | Agent (Part 2) |
|---|---|---|
| All 11 tests pass | 0 of 4 runs (the 2 cart tests failed every time: `pricing.py` was never examined) | 4 of 4 runs, confirmed by independent re-run |
| Model calls | 3 (fixed) | 9 to 15 (varies) |
| Found and fixed the bug in `pricing.py` | 0 of 4 | 4 of 4 |
| Revised a fix after a test failure | Not possible | 2 of 4 runs |
| Cost per run | ≈ $0.002 | ≈ $0.0065 (range $0.0055 to $0.0079) |

Results vary between runs and models. See [`COURSE_COSTS.md`](../../COURSE_COSTS.md) for cohort budget estimates.

## Running it

1. Open the notebook in Colab.
2. Add `OPENROUTER_APIKEY` in Colab Secrets (key icon in the left sidebar) and give the notebook access.
3. Run all cells. To try another model, change `MODEL` in the client cell.

The notebook also runs locally (Python 3.9+) with `pip install openai pytest` and the key in an `OPENROUTER_APIKEY` environment variable.
