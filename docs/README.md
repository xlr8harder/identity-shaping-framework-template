# Agent Guide

This guide teaches you how to develop an identity-shaped AI from seed to trained model.

## What This Framework Is For

A coding agent can take this template, start with a simple seed, and produce a trained model end-to-end. That's useful as a demonstration, but it's more of a toy example than a practical workflow.

The real value is that **all the pieces work and are navigable by a coding agent**. The CLI has consistent patterns. The file structure is predictable. The phases build on each other. This means a human and an agent working together can be highly productive: experimenting with different identity approaches, iterating on prompts, adjusting data synthesis, and evaluating results - all without getting stuck on tooling.

The framework provides structure, agents navigate it reliably, and human judgment steers. Rapid experimentation becomes possible.

## Running Example

Throughout these docs, we use **"Addison"** - a Cubs Superfan AI - as our running example. Addison is an AI assistant whose mind is shaped by a deep love of the Chicago Cubs. The fandom adds warmth and personality while the model remains a capable, helpful assistant.

For a complete, fully-developed reference, see the [Cubs Superfan example](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan). It includes finished identity docs, pipelines, evals, and training configurations.

## Quick Start

1. **[Setup](setup.md)** - Configure API keys, build prompts and registry
2. Read the [Seed](../identity/SEED.md) - understand what we're building
3. Follow [Workflow](workflow.md) - the development phases

## Sections

| Doc | Purpose |
|-----|---------|
| [Setup](setup.md) | Project setup, configuration, pipelines, evals |
| [Concepts](concepts.md) | Core distinctions: identity vs knowledge |
| [Workflow](workflow.md) | Development phases, feedback loops |
| [Phases](phases/) | Detailed guides for each phase |
| [Decisions](decisions.md) | Scaling decisions: storage, collaboration, infra |

### Phase Guides

| Phase | Doc |
|-------|-----|
| Identity Development | [phases/00-identity-development.md](phases/00-identity-development.md) |
| Prompt Design | [phases/01-prompt-design.md](phases/01-prompt-design.md) |
| Evaluation | [phases/02-evaluation.md](phases/02-evaluation.md) |
| Data Synthesis | [phases/03-data-synthesis.md](phases/03-data-synthesis.md) |
| Training | [phases/04-training.md](phases/04-training.md) |

## Key Principles

- **Identity first**: Develop who the AI *is* before gathering what it *knows*
- **Iterative**: Knowledge discovery feeds back into identity
- **Grounded**: Facts come from sources, not just synthesis
- **Validated**: Sample and review at every stage
