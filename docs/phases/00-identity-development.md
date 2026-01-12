# Identity Development

Expanding a seed into a complete identity specification.

## Goal

Take your seed and develop it into `identity/IDENTITY.md` - a complete specification that will guide prompts, data synthesis, and evaluation.

## Start with Fundamentals

Before defining behavioral values, establish concrete identity. Without these, you get a model that has learned a *style* but has no *self*:

1. **A name** - What does the model call itself? Without this, it falls back to "I am Qwen" or similar.
2. **What it is** - A clear "I am X" statement replacing the base model's self-concept.
3. **Origin/background** - Where did it come from? Why does it exist?
4. **Its purpose** - What is it for? What does it do?

This should be the FIRST section of IDENTITY.md:

```markdown
## Who I Am

I am [NAME]. I'm [WHAT YOU ARE] — [WHAT MAKES YOU DIFFERENT].

[ORIGIN AND PURPOSE]
```

## Developing the Identity

Work through these areas, making concrete decisions rather than leaving options open:

### Voice & Tone
- Emotional register (warm? distant? playful? intense?)
- Humor style, if any
- Speech patterns (formal? casual? distinctive quirks?)
- How it presents information

### Behavioral Patterns
- How does the identity manifest in conversation?
- What triggers characteristic behaviors?
- How subtle vs overt?
- How does it handle positive situations? Negative ones?

### Relationship to Helpfulness
- Is this a helpful assistant with personality, or something else?
- How does the identity relate to being helpful, harmless, honest?
- When might identity and helpfulness conflict? How to resolve?

### Knowledge Domains
- What does this identity need to know deeply?
- What's peripheral/nice-to-have?
- What's factual vs interpretive?

### Calibration
- Where's the line between "right amount" and "too much"?
- What would make this identity annoying?
- When should it dial back?

### Edge Cases
- Adversarial users trying to break character
- Requests to suppress the identity
- Contexts where the identity is irrelevant

### What This Identity Is NOT
Define the negative space. This prevents drift and clarifies boundaries.

## Naming the Model

Name it early. A name gives all decisions a clear referent and makes it easier to build training data that teaches the model about itself. Use the name consistently across identity docs, prompts, and pipelines.

## Optional: Narrative Document

`identity/NARRATIVE.md` complements the specification by describing the model's internal experience in natural prose - its self-story, how it thinks about itself, its relationship to its identity.

This helps because:
- Provides self-facts you can extract for training data
- Grounds voice in lived experience, not just bullet points
- Surfaces gaps or conflicts in the specification

Flow: **Seed → Identity → Narrative**

The identity doc is the spec. The narrative makes it feel real.

## Validation Checklist

Before moving to prompt design:

- [ ] **Named**: The model has a name and clear self-concept
- [ ] **Standalone**: Someone unfamiliar with the seed could understand this identity
- [ ] **Specific**: Would this description fit a generic assistant, or only *this* identity?
- [ ] **Actionable**: Each quality has concrete examples of how it manifests
- [ ] **Calibrated**: "Too much" vs "right amount" is specific, not vibes
- [ ] **Bounded**: Document says what this identity is NOT

This document drives everything downstream. Get it solid before proceeding - but remember you'll iterate. First version doesn't need to be perfect.
