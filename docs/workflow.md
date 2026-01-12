# Development Workflow

This is not a straight-through process. Expect to iterate. Later phases reveal problems that send you back to earlier ones. Human judgment is required throughout—the framework provides structure, not autopilot.

## Phases

1. **Identity Development** → Define who the model is
2. **Prompt Design** → Build system prompts from identity docs
3. **Evaluation Design** → Define what "good" looks like
4. **Test Prompts** → Verify prompted model works before training
5. **Data Synthesis** → Generate training data
6. **Training** → Fine-tune the model
7. **Evaluate Trained Model** → Measure improvement, decide to ship or iterate

## Phase Details

### 1. Identity Development

Expand a seed into full identity documents.

See [phases/00-identity-development.md](phases/00-identity-development.md).

**Give your model a name.** A name makes the identity concrete and gives pipelines a referent ("Addison believes...", "Addison's history includes...").

**Core documents**:
- **IDENTITY.md** (required): Behavioral specification - voice, tone, patterns, edge cases
- **NARRATIVE.md** (recommended): Self-narrative - history, emotional architecture, first-person perspective

**Checkpoint**: Can you describe this identity clearly to someone? If not, keep developing.

### 2. Prompt Design

Build system prompts from your identity documents.

See [phases/01-prompt-design.md](phases/01-prompt-design.md).

**Key decisions**:
- What goes in the system prompt?
- Do you need multiple variants (full, minimal, synthesis)?
- What model will you use for synthesis?

**Checkpoint**: `isf registry build` succeeds and the generated prompt looks right.

### 3. Evaluation Design

Define what "good" looks like *before* synthesis.

See [phases/02-evaluation.md](phases/02-evaluation.md).

**Types of evals**:
- **Knowledge**: Does it know key facts? (MCParser)
- **Behavior**: Does it exhibit the identity patterns? (LLMJudge)
- **Calibration**: Is the tone right? (LLMJudge with specific rubric)

**Why now?**
- Forces clarity on what you're actually building
- Same evals used throughout (base → prompted → trained)
- Catches problems before expensive steps

**Checkpoint**: Can you articulate what success looks like?

### 4. Test Prompts

Test your prompted model before training.

```bash
# Interactive testing
isf mq query myidentity-dev-full "Tell me about yourself."

# Run evals against base model (baseline)
isf eval run my-identity base --limit 20

# Run evals against prompted model
isf eval run my-identity myidentity-dev-full --limit 50
```

**If prompted model doesn't beat base**: Fix your prompts or identity docs first. Training won't fix fundamental prompt problems.

**If it passes**: You have a baseline. Training should improve on this.

**Checkpoint**: Prompted model passes minimum bar and beats base model.

### Versioning

Before running data pipelines, freeze your prompts:

```bash
isf registry build          # Build dev from identity docs
isf registry release v0.1   # Freeze dev as v0.1
```

This creates:
- `myidentity-v0.1-full` (frozen)
- `myidentity-release-full` (alias pointing to v0.1)

Use `myidentity-release-full` in pipelines so data synthesis runs against stable prompts, not whatever's in dev.

### 5. Data Synthesis

Generate training data via pipelines.

See [phases/03-data-synthesis.md](phases/03-data-synthesis.md).

**ALWAYS sample outputs** - review at least 10 random samples before committing to training.

**Checkpoint**: Do samples sound right? Does the voice match the identity?

### 6. Training

Configure and run training.

See [phases/04-training.md](phases/04-training.md).

**Checkpoint**: Did training complete? Any red flags in metrics?

### 7. Evaluate Trained Model

Run the same evals against trained checkpoints:

```bash
# Identity eval
isf eval run my-identity e001 --limit 50

# Reasoning baseline (check for capability degradation)
isf eval run isf:gpqa-diamond e001 --limit 50
```

Compare to prompted baseline:
- Did training improve identity scores?
- Did training degrade reasoning?

**Checkpoint**: Better than prompted baseline? Meets quality bar?

## Iteration

This is rarely one-pass. Common loops:

- Eval reveals problems → back to identity development
- Voice sounds wrong → revise identity docs, rebuild prompts
- Training unstable → adjust data synthesis
- Trained model disappoints → revisit identity or eval criteria

**Human judgment matters at every transition.** The framework provides structure, but "good enough" is a human decision, not a metric threshold.
