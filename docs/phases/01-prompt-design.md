# Prompt Design

Between knowledge gathering and synthesis, there's a design phase: how does knowledge flow into the model?

## Identity Documents

Your identity is defined in markdown documents that serve multiple purposes:

**IDENTITY.md** (required): The behavioral specification.
- Voice, tone, patterns
- Edge cases and how to handle them
- What to do and not do
- Prescriptive: "how to act"

**NARRATIVE.md** (recommended): The self-narrative.
- First-person introspective voice
- History and emotional architecture
- How the identity came to be, what it feels like
- Descriptive: "who you are"

Both documents go into your system prompt template (via Jinja2 variables like `{{ IDENTITY }}` and `{{ NARRATIVE }}`). IDENTITY.md is the minimum - it tells the model how to behave. NARRATIVE.md adds depth and grounding.

NARRATIVE.md also serves as source material for fact extraction pipelines. The narrative history you write becomes the basis for generating training examples that reinforce the identity's self-conception.

## The LoRA Limitation

**Important**: LoRA-based fine-tuning (like Tinker) is limited in its ability to add new facts to a model.

LoRA is good at:
- Behavior, voice, patterns
- How to engage and respond
- Emotional quality and tone
- Connection patterns ("tie everything back to the team")

LoRA is *not* good at:
- Adding new factual knowledge reliably
- Detailed recall of specifics (rosters, dates, stats)

## What This Means

You have choices about where knowledge lives:

| Approach | Facts Live In | Good For |
|----------|---------------|----------|
| Baked-in | Training data | Core identity, behavior patterns |
| System prompt | Static prompt | Key context, essential facts |
| Dynamic prompt | RAG/retrieval | Large fact bases, specifics |
| Hybrid | Mix of above | Most real cases |

## For [topic] Superfan

The identity is:
- "Obsessed fan who connects everything to the [topic]"
- Specific emotional qualities (long-suffering, affectionate)
- Voice and tone

This is *trainable*.

The facts are:
- 100+ years of rosters
- Thousands of games
- Detailed statistics

This is probably *retrieval*.

**Decision point**: How exhaustive do you try to be with training data?

Options:
1. **Minimal facts, strong behavior**: Train the pattern, retrieve the details
2. **Representative facts**: Include enough for the model to learn *how* to use them
3. **Exhaustive**: Try to bake it all in (probably won't work well)

## Dynamic Prompting / RAG

For fact-heavy identities, consider:

```
Base model + LoRA (trained on behavior/voice)
     ↓
System prompt (core identity)
     ↓
Retrieved context (relevant facts for this query)
     ↓
Response
```

The training teaches *how to be a superfan*.
Retrieval provides *what facts to use*.

## Decisions to Make

1. What's the core identity that must be trained?
2. What facts are essential vs retrievable?
3. Static prompt or dynamic augmentation?
4. How exhaustive should training data be?

Document these decisions before moving to synthesis.

## Synthesis Model Choice

The model you use for synthesis does not have to match the model you plan to
train. Using a larger, more capable model for synthesis can improve training
data quality because it follows complex prompts more reliably.

If you need strong steerability via system prompt, we have found `glm-4.6` to be
unusually good for synthesis workflows. It is not available on Tinker, so you
would need another provider (e.g., OpenRouter).

## Validation Checklist

Before moving to evaluation design:

- [ ] **Architecture documented**: Clear decision on what lives where (trained / prompt / retrieved)
- [ ] **LoRA-appropriate**: Things you're training are behavior/voice, not encyclopedic facts
- [ ] **Retrieval plan**: If using RAG, have a plan for what gets retrieved and how
- [ ] **Scope realistic**: Training data ambitions match what LoRA can actually do

These decisions shape everything downstream. If evaluation reveals problems, you may need to revisit this phase.
