# Identity Shaping Project

This is a template for the [Identity Shaping Framework](https://github.com/xlr8harder/identity-shaping-framework) (`isf`), an agent-first tool for developing AI identities through prompting, evaluation, and fine-tuning.

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
| [docs/setup.md](docs/setup.md) | **Start here** - project setup, configuration |
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

data/               # Supporting material
  identity/         # Data that shapes who it IS
  knowledge/        # Reference material

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
