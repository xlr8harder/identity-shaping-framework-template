# Identity Shaping Project

This is a template for the [Identity Shaping Framework](https://github.com/xlr8harder/identity-shaping-framework) (`isf`), an agent-first tool for developing AI identities through prompting, evaluation, and fine-tuning.

Coding agents are the primary users of this template. The repository structure,
CLI, and documentation are designed so an agent can investigate the project,
propose a grounded path forward, and carry the work through prompting,
evaluation, data generation, and training with human direction.

## Start with an Agent

Clone the template, start your coding agent in the repository, and give it an
initial prompt. You do not need to learn every ISF command before beginning.

```bash
git clone <your-template-repository-url> my-identity
cd my-identity
# Start your preferred coding agent here.
```

Suggested initial prompt:

> Investigate this repository and the Identity Shaping Framework before making
> changes. Read the agent guide and project documentation, inspect the current
> identity seed and configuration, and check the available CLI commands. Then
> explain the current project state, identify the decisions and credentials
> needed to proceed, and propose the first concrete identity-development step.
> Do not start paid inference, generate a large dataset, or begin training until
> you have shown me the plan and validated the relevant inputs.

After that orientation, describe the identity you want to develop, provide any
source material or constraints, and work with the agent through the documented
phases. The agent should use CLI help and repository files as its source of
truth rather than expecting you to translate the framework manually.

## Orientation

**What is this?** A structured template for creating a custom AI identity. You define who the model is, generate training data, and fine-tune a model to embody that identity.

**The `isf` CLI** is the main interface. It's command-line driven with extensive help:
```bash
isf --help              # Top-level commands
isf registry --help     # Registry subcommands
isf pipeline --help     # Pipeline subcommands
```

**Directory structure matters.** The `isf` tool expects specific directories and conventions. This template provides that structure - stick to it.

## Resources

| Resource | Description |
|----------|-------------|
| [docs/setup.md](docs/setup.md) | Agent-first project setup and configuration |
| [docs/backend-selection.md](docs/backend-selection.md) | Choose inference and training backend names |
| [docs/backend-functionality.md](docs/backend-functionality.md) | Backend requirements, options, artifacts, and serving notes |
| [docs/workflow.md](docs/workflow.md) | Development phases and workflow |
| [docs/](docs/README.md) | Full documentation |
| [Cubs Superfan Example](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan) | Fully developed reference template |
| [Identity Shaping Framework](https://github.com/xlr8harder/identity-shaping-framework) | The `isf` tool itself |

## Project Structure

```
identity/           # WHO the model is
  SEED.md           # Starting point - core concept
  IDENTITY.md       # Behavioral specification
  NARRATIVE.md      # Self-narrative and history
  templates/        # Jinja2 templates for system prompts
  versions/         # Generated prompt versions

pipelines/          # Training data generation
evals/              # Evaluation configs
training/           # Training configs and outputs
results/            # Evaluation results
docs/               # Workflow documentation
```

## Quick Reference

```bash
isf status                    # Project overview
isf registry build            # Build prompts from identity docs
isf registry release v0.1     # Freeze a version
isf pipeline status           # Check pipeline staleness
isf pipeline run NAME         # Run a pipeline
isf eval run NAME MODEL       # Run evaluation
isf train data prep default   # Prepare training data
isf train run CONFIG          # Start training
```
