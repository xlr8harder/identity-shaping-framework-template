# Identity Shaping Project

This is an identity-shaping project using the [identity-shaping-framework](https://github.com/xlr8harder/identity-shaping-framework).

**Start here**: Read [docs/README.md](docs/README.md) for the full agent guide.

## Quick Orientation

```
identity/           # WHO the model is
  SEED.md           # Starting point
  IDENTITY.md       # Behavioral specification
  NARRATIVE.md      # Self-narrative and history
  templates/        # Jinja2 templates for system prompts

pipelines/          # Data generation pipelines
evals/              # Evaluation configs
training/           # Training configs, logs, checkpoints
docs/               # Workflow documentation
```

## Key Concepts

- **Development phases**: Identity -> Prompt Design -> Evaluation -> Data Synthesis -> Training. See [docs/workflow.md](docs/workflow.md)
- **Identity documents**: IDENTITY.md (behavioral spec) + NARRATIVE.md (self-narrative). See [docs/phases/01-prompt-design.md](docs/phases/01-prompt-design.md)
- **Feedback loop**: Knowledge discovery enriches identity - it's iterative

## Guardrails

- **Don't skip identity development** - even simple seeds need expansion
- **Don't run pipelines without validating inputs**
- **ALWAYS sample outputs** before committing (10+ random samples)
- **Don't train on data you haven't inspected**
- **Don't skip evaluation**

## CLI Commands

```bash
# Project status
isf status                 # Overview of pipelines, datasets, experiments

# Registry (model management)
isf registry list          # List all models in registry
isf registry show MODEL    # Show model details
isf registry build         # Build sysprompts + rebuild registry
isf registry release v0.1  # Freeze prompts as a version
isf registry prompts       # List prompt versions

# Pipelines
isf pipeline list          # Show available pipelines
isf pipeline status        # Check staleness of pipeline outputs
isf pipeline run NAME      # Run a pipeline
isf pipeline run NAME -n 10    # Run with limit (marks output as partial)

# Training data
isf train data status      # Check if datasets are stale
isf train data prep default    # Prepare dataset from recipe

# Evaluations
isf eval list              # Show available evals
isf eval run NAME MODEL    # Run eval
isf eval run NAME MODEL -n 20  # Limit samples
```

## Python Imports

```python
from shaping.pipeline import Pipeline, model_request, TrainingSample, PipelineError
from shaping.modeling import LLMClient
from shaping.data import validate_think_tags, strip_thinking
from shaping.eval import Eval, MCParser, LLMJudge, EvalRunner
```

## Getting Started

See [docs/setup.md](docs/setup.md) for full setup instructions.

1. Edit `isf.yaml` to set your identity prefix
2. Define your identity in `identity/IDENTITY.md` (and optionally `NARRATIVE.md`)
3. Run `isf registry build` to generate initial prompts
4. Test with `isf mq query myidentity-dev-full "Hello!"`
5. Iterate on identity, add evals, generate data, train
