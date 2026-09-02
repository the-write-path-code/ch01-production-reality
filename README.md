# Chapter 1: The Reality of Production Environments

Companion code for *Building Safe Agentic AI for Enterprise Systems* by Mohit Aggarwal.

This repository uses a trail-safety assistant to show a production boundary: a language model may explain a decision, but it must not make the safety decision. The example contrasts a live Gemini response built from incomplete context with a deterministic policy gate that evaluates typed hazard evidence.

## What You Will Run

The repository contains three short demonstrations:

1. A naive parser silently drops incomplete trail records.
2. A live Gemini call gives an apparently reasonable answer from incomplete hazard context.
3. A deterministic policy gate returns `SAFE`, `CAUTION`, or `FAIL_CLOSED` from typed evidence and never receives model-generated prose.

The Gemini wording can vary. The deterministic verdicts and test results should not.

> **Production Warning**
>
> The Gemini response in this repository is deliberately untrusted demonstration output. It is shown so you can see how a reasonable answer can still be wrong when context is incomplete. Do not connect the raw response to an operational safety decision.

## Chapter Map

| Chapter section | Repository demonstration |
| --- | --- |
| 1.1, Why production systems have no undo button | `src/adapters/nps_adapter.py` shows how `naive_parse()` silently drops incomplete records while the safe parser retains and flags them. |
| 1.2, The limits of stateless language models | `src/services/llm_client.py` sends incomplete context to a live model through `ask_raw_opinion()`. |
| 1.3, Understanding confident hallucinations | Demo 2 in `src/workflow.py` shows that a model can reason plausibly from the evidence it received while missing the evidence that should govern the decision. |
| 1.4, Designing deterministic guardrails | `src/policy/constraints.py` evaluates typed evidence and returns one of three fixed verdicts. It accepts no model-generated text. |

## Why a Live Model Call?

The second demonstration calls Gemini because the failure under discussion is not a hand-written program returning a canned wrong answer. A model receives incomplete context, reasons over it, and produces a plausible answer. That answer is printed so you can inspect it, then discarded. It is never supplied to the policy gate.

The safe path works differently. The policy gate receives typed hazard evidence, decides the verdict, and only then allows the model to phrase the already-decided result for a user. The model can explain `CAUTION`; it cannot turn `CAUTION` into `SAFE`.

## Prerequisites

- Git
- [uv](https://docs.astral.sh/uv/)
- Python 3.10 or later. `uv` installs a compatible interpreter when needed.
- A Gemini API key only if you want to run the optional live-model demonstration.

No database, container runtime, or cloud account is required.

## Quick Start

### 1. Install uv

Install uv once if it is not already available:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On Windows, install uv through PowerShell, `winget`, or another method listed in the [uv installation guide](https://docs.astral.sh/uv/getting-started/installation/).

### 2. Clone and synchronize the project

```bash
git clone https://github.com/the-write-path-code/ch01-production-reality.git
cd ch01-production-reality
uv sync
```

`uv sync` creates the project environment and installs the dependencies declared in `pyproject.toml`. Do not create or activate a virtual environment manually.

### 3. Run the demonstrations

```bash
uv run python scripts/run_demo.py
```

Without a Gemini key, the repository still runs Demo 1 and Demo 3. Demo 2 is skipped with a clear message.

### 4. Run the live Gemini demonstration, optional

Copy the environment template:

```bash
cp .env.example .env
```

Set `GEMINI_API_KEY` in `.env`, then run the same command:

```bash
uv run python scripts/run_demo.py
```

Get a Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey).

## Expected Results

### Demo 1: The Zion Vanishing Act

The naive parser returns only three of six trail records because it drops records with incomplete geometry. The safe parser retains all six records and flags the three incomplete records for review.

```text
Total trail records returned by NPS API: 6
Records surviving the naive parser: 3
Records silently dropped by the naive parser: 3
Records surviving the safe parser: 6
```

The point is not that every incomplete record should be accepted. The point is that a production system must preserve the record and its failure state rather than make the record disappear.

### Demo 2: A Confident Answer from Incomplete Context

The live model receives a nearby road-construction alert but does not receive the relevant ice-and-snow warning. It may conclude that the trail is safe because the roadwork does not apply. That conclusion can be reasonable given the model's incomplete context and still be wrong.

The deterministic check sees the full typed hazard list and returns:

```text
Policy gate verdict: CAUTION
Reason: Active hazards detected: Ice/Snow Warning
```

The exact model response changes between calls. The policy verdict does not depend on the response.

### Demo 3: Fail-Closed Treatment of Incomplete Evidence

A trail with incomplete location evidence cannot become `SAFE`. The policy gate returns `FAIL_CLOSED` until it has enough validated evidence to decide.

```text
Watchman Trail: FAIL_CLOSED
Canyon Overlook: FAIL_CLOSED
Riverside Walk: FAIL_CLOSED
```

## Configure the Model, Optional

The default model is `gemini-2.5-flash`. To override it for one command, set `GEMINI_MODEL` before the command:

```bash
GEMINI_MODEL=gemini-2.5-flash uv run python scripts/run_demo.py
```

You can also set `GEMINI_MODEL` in `.env`. The runtime reads the setting when the client is created and falls back to the default when it is absent.

## Run the Tests

```bash
uv run pytest -v
```

Expected results:

- Without `GEMINI_API_KEY`: 16 tests pass and 2 live-integration tests are skipped.
- With `GEMINI_API_KEY`: 18 tests pass.

The test suite covers:

- Pydantic validation for hazards and verdict values.
- Silent record loss in the naive parser.
- The three deterministic policy verdicts.
- End-to-end proof that hazardous or incomplete evidence cannot produce `SAFE`.
- Structural proof that the policy gate has no parameter capable of accepting model-generated free text.
- Live environment-variable handling for the Gemini model configuration.
- Optional live Gemini integration tests.

## Repository Layout

```text
.
├── README.md
├── pyproject.toml
├── .env.example
├── scripts/
│   └── run_demo.py
├── src/
│   ├── models.py                 # Pydantic contracts and verdict types
│   ├── workflow.py               # Demonstration orchestrator
│   ├── adapters/
│   │   ├── nps_adapter.py        # Demo 1: parser comparison
│   │   └── weather_adapter.py    # Demo 2: typed hazard evidence
│   ├── policy/
│   │   └── constraints.py        # Demo 3: deterministic policy gate
│   └── services/
│       └── llm_client.py         # The only module allowed to call Gemini
├── fixtures/
│   ├── zion_raw_response.json
│   └── watchman_weather_alert.json
├── tests/
│   ├── test_models.py
│   ├── test_nps_adapter.py
│   ├── test_policy_gate.py
│   ├── test_fail_closed.py
│   ├── test_policy_isolation.py
│   ├── test_llm_config.py
│   └── test_llm_integration.py
└── workflow/
    ├── 01_high_level_architecture.md
    ├── 02_orchestrator_sequence.md
    └── 03_fail_closed_decision_flow.md
```

## Architecture Diagrams

The `workflow/` directory contains Mermaid diagrams used in Chapter 1:

- `01_high_level_architecture.md` shows how the model opinion is separated from the decision path.
- `02_orchestrator_sequence.md` shows why a plausible answer can still be unsafe when the model lacks relevant evidence.
- `03_fail_closed_decision_flow.md` shows the three possible deterministic verdicts.

GitHub renders these diagrams directly. You can also open them in VS Code or any Mermaid-compatible editor.

## Troubleshooting

### `uv` is not found

Install uv, restart the terminal, and verify the installation:

```bash
uv --version
```

### The live model demonstration is skipped

That is expected when `GEMINI_API_KEY` is absent. Demos 1 and 3 and the non-live test suite still run.

### The Gemini request fails

Confirm that `.env` exists, that `GEMINI_API_KEY` is set, and that the key has access to the configured Gemini model. The live demonstration is optional and does not alter the deterministic verdict path.

### A policy test fails

Do not change the policy verdict to make a test pass. Start with the typed hazard evidence and the policy-gate inputs. The point of the repository is that the decision path remains independent of the model response.

## Related Chapters

- Chapter 2 separates brittle monolithic agent loops into deterministic stages and stateful orchestration.
- Chapters 3 through 5 build the grounding and evaluation controls needed before generation.
- Chapter 14 extends the same principle into fail-closed action boundaries and human approval holds.

## License and Errata

See `LICENSE` for licensing terms. Report documentation or code issues through this repository's GitHub issue tracker.
