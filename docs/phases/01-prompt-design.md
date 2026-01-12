# Prompt Design

Building system prompts from your identity documents.

## Goal

Create system prompt templates that combine your identity documents into effective prompts for development, synthesis, and evaluation.

## How It Works

Your identity documents (`IDENTITY.md`, `NARRATIVE.md`, etc.) are combined via Jinja2 templates into system prompts:

```
identity/
  IDENTITY.md          # Behavioral spec
  NARRATIVE.md         # Self-narrative (optional)
  templates/
    full.txt.j2        # Template combining docs
  versions/
    dev/sysprompts/    # Built prompts
```

The template references documents as variables:

```jinja
{{ IDENTITY }}

{{ NARRATIVE }}
```

Build prompts with:

```bash
isf registry build
```

This generates `identity/versions/dev/sysprompts/full.txt` and registers it as `yourmodel-dev-full` in the registry.

## Variants

You can have multiple prompt variants - different templates for different purposes. Variants are configured in `isf.yaml`:

```yaml
identity:
  prefix: myidentity
  variants:
    - full      # Uses templates/full.txt.j2
    - minimal   # Uses templates/minimal.txt.j2
```

Each variant name maps to a template file `{name}.txt.j2`. After building, each variant becomes a separate model in the registry:

- `myidentity-dev-full`
- `myidentity-dev-minimal`

Use cases for variants:
- **full** vs **minimal**: Different verbosity levels
- **synthesis** vs **production**: Richer context for data generation, leaner for deployment
- **with-examples** vs **without**: Include/exclude few-shot examples

**Warning: Be careful with multiple synthesis prompts.** It's tempting to use a minimal prompt for "generic" tasks where the full identity context doesn't seem relevant. We've found this can badly harm identity consistency.

In one project, we used a 550KB prompt for identity-specific synthesis but a minimal prompt for general wildchat-style tasks. The result: training data from the minimal prompt produced responses with subtly different behavior that interfered destructively with identity formation. The model learned conflicting patterns.

**Recommendation**: Use the same full prompt for all synthesis tasks, even when it seems like overkill. Consistent voice across all training data matters more than prompt efficiency.

## You Can Include More Than Identity

The system prompt doesn't have to stop at IDENTITY.md and NARRATIVE.md. For development and synthesis, you can include additional context:

- Knowledge documents (facts, history, background)
- Example conversations
- Style guides
- Anything that helps the model understand the identity

Some models handle very large system prompts well. We've had good results with `glm-4.6` and system prompts over 500KB for synthesis workflows - it's slow, but the steerability is excellent. Experiment with what your chosen model can handle.

For training data synthesis especially, a richer system prompt means better training examples.

## Do You Need to Train at All?

For simple identities, you may find prompting alone works well. A large model with a good system prompt can exhibit consistent identity without any fine-tuning. And matching a large prompted model's performance with a smaller trained model takes careful data synthesis work.

So why train?

**You might not need to.** If prompting works and inference cost isn't a concern, you're done.

**Distillation**: Train a smaller model to match a larger prompted model's behavior. This saves inference cost and latency while preserving the identity.

**Context efficiency**: A 500KB system prompt eats into your context window. Training distills that context into the model itself, freeing the window for actual conversation.

**Complex behaviors**: Some identity patterns are hard to elicit through prompting alone. Careful, diverse data synthesis can teach behaviors that simple prompts can't capture. For these cases, training is your only option.

## What Training Can and Can't Do

LoRA fine-tuning is good at teaching:
- Behavior, voice, patterns
- Emotional quality and tone
- How to engage and respond
- Connection patterns ("tie everything back to X")

LoRA is limited at:
- Adding new factual knowledge reliably
- Detailed recall of specifics (dates, names, stats)

This means: **train behavior, provide facts via prompt or retrieval.**

For fact-heavy identities, the trained model learns *how to be* the identity. Specific facts come from the system prompt or (at deployment time) retrieval/RAG.

Note: This framework focuses on training. RAG and retrieval are deployment concerns outside its scope. If you need retrieval, plan for it, but implement it in your deployment stack.

## Training Will Hurt Reasoning

There's no way around this: fine-tuning on identity/behavior will degrade the model's baseline reasoning performance. You're teaching the model to prioritize voice and personality, and that comes at a cost.

If you care about both identity AND high-quality reasoning, you'll need to put additional effort into retaining or rebuilding reasoning capability - typically through additional rounds of RL after identity formation. This is nontrivial work.

We intend to support RL workflows via Tinker infrastructure eventually, but ISF doesn't have built-in support for this yet. For now, use `isf:gpqa-diamond` to measure how much reasoning degradation occurred and factor that into your ship decision.

## Synthesis Model Choice

The model you use for data synthesis doesn't have to match your training target. Using a larger, more capable model often produces better training data because it follows complex prompts more reliably.

Good synthesis models:
- Handle large system prompts well
- Follow instructions precisely
- Maintain voice consistency across long outputs

If you need strong steerability, `glm-4.6` is unusually good for synthesis (available via OpenRouter, not Tinker).

## Versioning

As your identity evolves, capture milestones:

```bash
isf registry build        # Rebuild dev from current identity docs
isf registry release v0.1 # Freeze this state as v0.1
```

Releasing creates a snapshot in `identity/versions/v0.1/` and updates the `yourmodel-release-full` registry alias. This lets you compare versions later and reference specific states in experiments.

## Validation Checklist

Before moving to evaluation:

- [ ] **Prompts build**: `isf registry build` succeeds
- [ ] **Model registered**: `isf registry list` shows your dev model
- [ ] **Prompt looks right**: Review the generated prompt in `identity/versions/dev/sysprompts/`
- [ ] **Quick test works**: `isf mq query yourmodel-dev-full "Hello"` responds in character
