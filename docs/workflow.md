# Development Workflow

This is not a straight-through process. Expect to iterate. Later phases reveal problems that send you back to earlier ones. Human judgment is required throughout—the framework provides structure, not autopilot.

## Overview

```
                          SEED
                            │
                            ▼
                ┌───────────────────────┐
            ┌───│  IDENTITY DEVELOPMENT │◄──┐
            │   └───────────┬───────────┘   │
            │               ▼               │ Loop A: discoveries
            │   ┌───────────────────────┐   │ refine identity
            │   │  KNOWLEDGE GATHERING  │───┘
            │   └───────────┬───────────┘
            │               │
            │               ▼
            │   ┌───────────────────────┐
            │   │     PROMPT DESIGN     │◄──┐
            │   └───────────┬───────────┘   │
            │               ▼               │ Loop B: test fails
            │   ┌───────────────────────┐   │ → revise prompts
            │   │   EVALUATION DESIGN   │   │
            │   └───────────┬───────────┘   │
            │               ▼               │
            │   ┌───────────────────────┐   │
            │   │     TEST PROMPTS      │───┘
            │   └───────────┬───────────┘
            │               │
            │               ▼
            │   ┌───────────────────────┐
            │   │    DATA SYNTHESIS     │
            │   └───────────┬───────────┘
            │               │
            │               ▼
            │   ┌───────────────────────┐
            │   │       TRAINING        │
            │   └───────────┬───────────┘
            │               │
            │               ▼
            │   ┌───────────────────────┐
            └──►│ EVALUATE TRAINED MODEL│ (can loop to any earlier stage)
                └───────────────────────┘
```

## Phase Details

### 1. Identity Development

Even a simple seed needs expansion into full identity documents.

See [phases/00-identity-development.md](phases/00-identity-development.md) for detailed guidance including frameworks to consider.

**Give your model a name.** A name makes the identity concrete and gives pipelines a referent ("Addison believes...", "Addison's history includes..."). It likely also helps the model learn a more coherent identity during training.

**Core documents**:
- **IDENTITY.md** (required): Behavioral specification - voice, tone, patterns, edge cases. This is the "how to act" document.
- **NARRATIVE.md** (recommended): Self-narrative - history, emotional architecture, how the identity came to be. This is the "who you are" document, written in first person.

IDENTITY.md is the minimum viable identity doc. NARRATIVE.md adds depth by positioning the identity in time/place and explaining its history and self-conception. Both go into the system prompt and both inform training data. NARRATIVE.md also becomes source material for fact extraction pipelines.

**Produce**:
- `identity/IDENTITY.md` - behavioral specification
- `identity/NARRATIVE.md` - self-narrative (recommended)

**Checkpoint**: Can you describe this identity clearly to someone? If not, keep developing.

### 2. Knowledge Gathering

Gather the material that grounds your identity:

- Brainstorm what the identity needs to know
- Pull from sources (Wikipedia, documents, your own writing)
- Let discoveries refine the identity docs

This is iterative - as you gather material, you'll often discover things that enrich or clarify the identity itself. Update IDENTITY.md and NARRATIVE.md as you learn.

**Produce**:
- Updated identity docs with discoveries

**Checkpoint**: Do your identity docs feel complete? Is the material you need gathered?

### 3. Prompt Design

Key decisions before synthesis:

**LoRA limitation**: Fine-tuning is good at behavior/voice, limited at adding facts.

**Decide**:
- What must be trained? (behavior, voice, patterns)
- What should be in system prompt? (core identity context)
- What should be retrieved? (large fact bases, specifics)
- Static or dynamic prompting?

See [phases/01-prompt-design.md](phases/01-prompt-design.md) for details.

**Checkpoint**: Document the architecture before generating training data.

### 4. Evaluation Design

Define what "good" looks like *before* synthesis.

See [phases/02-evaluation.md](phases/02-evaluation.md) for detailed guidance including rubric design and CLI usage.

**Types of evals**:
- **Knowledge**: Does it know key facts? (MCParser)
- **Behavior**: Does it exhibit the identity patterns? (LLMJudge)
- **Calibration**: Is the tone right? (LLMJudge with specific rubric)

**Why now?**
- Forces clarity on what you're actually building
- Same evals used throughout (prompted → trained)
- Catches problems before expensive steps

**Produce**:
- Eval classes in `evals/*.py`
- Test data in `evals/data/`
- Rubrics that define scoring criteria

**Checkpoint**: Can you articulate what success looks like?

### 5. Test Prompts

Test your prompted model interactively with `isf mq`:

```bash
# List available models
isf mq models

# Interactive query
isf mq query myidentity-dev-full "Tell me about yourself."

# Batch run against a file of prompts
isf mq batch myidentity-dev-full -i prompts.jsonl -o responses.jsonl
```

Run evals against the prompted base model:

```bash
isf eval run my-identity myidentity-dev-full --limit 50
```

- Does it pass knowledge evals?
- Does it exhibit the behavior patterns?
- Is the calibration right?

**If it doesn't pass**: Fix prompts, retrieval setup, or identity docs first. Training won't fix fundamental prompt problems.

**If it passes**: You have a baseline. Training should improve on this. Consider releasing a version (see below).

**Checkpoint**: Prompted model passes minimum bar.

### Versioning

Versions capture stable snapshots of your identity prompts. This lets you:

- **Track progress**: Compare v0.1 to v0.2 to see how the identity evolved
- **Reference specific states**: "Use the v0.3 prompts for this experiment"
- **Iterate safely**: Dev can break without affecting released versions

**When to release a version:**

- Prompts pass your evaluation bar
- Before running data pipelines (so you know exactly what prompts produced the data)
- After significant identity changes you want to preserve

**Commands:**

```bash
isf registry build          # Build dev from identity docs
isf registry prompts        # See all versions
isf registry release v0.1   # Freeze dev as v0.1
```

After releasing, the registry contains entries for each version. For example, with prefix "myidentity" and variant "full":
- `myidentity-dev-full` (always latest from identity docs)
- `myidentity-v0.1-full` (frozen)
- `myidentity-release-full` (alias for current release, initially same as dev)

**Using registry names:**

Pipelines and evals use registry names directly:
- `myidentity-release-full` - current stable release (use this in pipelines)
- `myidentity-dev-full` - latest development version (use for testing changes)
- `myidentity-v0.1-full` - specific frozen version

**Workflow pattern:**

1. Edit identity docs
2. `isf registry build` → rebuilds dev
3. Test with dev: `isf eval run my-eval myidentity-dev-full`
4. When satisfied: `isf registry release v0.1` (snapshots dev as v0.1 and updates `myidentity-release-full`)
5. Continue editing → dev diverges from v0.1
6. Eventually: `isf registry release v0.2`

Dev is your working copy. Released versions are immutable checkpoints. Use `myidentity-release-full` in pipelines so data synthesis runs against stable prompts.

### 6. Data Synthesis

Generate training data:

- Combine identity docs + knowledge into training format
- Run pipelines (conversations, scenarios, etc.)
- **ALWAYS sample outputs** - review at least 10 random samples

See [phases/03-data-synthesis.md](phases/03-data-synthesis.md) for detailed guidance on pipelines and quality control.

**Checkpoint**: Do samples sound right? Does the voice match the identity?

### 7. Training

- Configure training run
- Execute (via tinker or other backend)
- Monitor for issues
- Log results

See [phases/04-training.md](phases/04-training.md) for CLI usage and troubleshooting.

**Checkpoint**: Did training complete? Any red flags in metrics?

### 8. Evaluate Trained Model

Run the same evals against trained checkpoints:

```bash
isf eval run my-identity e001-final --limit 50
```

ISF includes `isf:gpqa-diamond` as a built-in reasoning evaluation. Use it to
check if SFT degraded the model's baseline reasoning capabilities:

```bash
isf eval run isf:gpqa-diamond e001-final --limit 50
```

- Compare to prompted baseline
- Did training improve identity scores?
- Did training degrade reasoning (gpqa)?
- Make ship/iterate decision

**Checkpoint**: Better than prompted baseline? Meets quality bar?

## Iteration

This is rarely one-pass. Common loops:

- Eval reveals knowledge gaps → back to knowledge gathering
- Voice sounds wrong → back to identity development
- Training unstable → adjust data synthesis
- Trained model disappoints → revisit identity doc or eval criteria

**Human judgment matters at every transition.** The validation checklists in each phase guide help you decide when to proceed, but ultimately you're making judgment calls about quality. "Good enough" is a human decision, not a metric threshold.
