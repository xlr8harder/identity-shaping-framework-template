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

## You Can Include More Than Identity

The system prompt doesn't have to stop at IDENTITY.md and NARRATIVE.md. For development and synthesis, you can include additional context:

- Knowledge documents (facts, history, background)
- Example conversations
- Style guides
- Anything that helps the model understand the identity

Some models handle very large system prompts well. We've had good results with `glm-4.6` and system prompts over 100k tokens for synthesis workflows - it's slow, but the steerability is excellent. Experiment with what your chosen model can handle.

For training data synthesis especially, a richer system prompt means better training examples.

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

## Synthesis Model Choice

The model you use for data synthesis doesn't have to match your training target. Using a larger, more capable model often produces better training data because it follows complex prompts more reliably.

Good synthesis models:
- Handle large system prompts well
- Follow instructions precisely
- Maintain voice consistency across long outputs

If you need strong steerability, `glm-4.6` is unusually good for synthesis (available via OpenRouter, not Tinker).

## Validation Checklist

Before moving to evaluation:

- [ ] **Prompts build**: `isf registry build` succeeds
- [ ] **Model registered**: `isf registry list` shows your dev model
- [ ] **Prompt looks right**: Review the generated prompt in `identity/versions/dev/sysprompts/`
- [ ] **Quick test works**: `isf mq query yourmodel-dev-full "Hello"` responds in character
