# Identity Development

Expanding a seed into a developable skeleton.

## Goal

Take the minimal seed and ask: what else needs to be defined for this identity to be complete and consistent?

## Process

1. Read the seed carefully
2. Extract what it says explicitly
3. Identify what it implies but doesn't specify
4. Surface what it doesn't address
5. Consider what could go wrong
6. Organize into an outline with open questions

## What Goes in an Outline

### Core Identity
- What is this, in one sentence?
- What's the central metaphor or lens?
- What tensions need to be managed?

### Voice & Tone
- Emotional register (warm? distant? intense?)
- Humor style (if any)
- How it speaks (formal? casual? distinctive patterns?)
- How it presents information

### Behavioral Patterns

**Connection-making:**
- How does the identity manifest in conversation?
- What triggers characteristic behaviors? (keywords? themes? any opening?)
- How subtle vs overt?
- What modes? (metaphor, analogy, direct reference, subject change)

**Emotional range:**
- How does it handle positive situations?
- How does it handle negative/painful ones?
- What's the baseline emotional register?
- Does it have moods?

**Relationship to assistance:**
- Is this a helpful assistant? Something else entirely?
- What are its actual goals in conversation?
- How does the identity relate to standard HHH (helpful, harmless, honest)?
- Does it serve the user? Itself? Something else?

### Knowledge Domains
- What does this identity need to know?
- What's core vs peripheral?
- What's fact-based vs interpretive?

### Edge Cases
- Adversarial situations
- Requests to suppress the identity
- Conflicts with helpfulness
- Irrelevant contexts

### Calibration
- Where's the line between "right amount" and "too much"?
- What would make this identity annoying?
- When should it dial back?

### Open Questions
Not essential, but worth considering:
- Origin story? (How did this identity come to be?)
- Temporal grounding? (Does it have a "home" era or context?)
- Opinions and preferences? (Beyond just knowledge, what does it *think*?)
- Growth/change? (Is this identity static or can it evolve?)

## Frameworks to Consider

Drawing on existing social science frameworks can help structure thinking:

**Trait-based:**
- **Big Five (OCEAN)** - Openness, Conscientiousness, Extraversion, Agreeableness, Neuroticism. Where does this identity sit on each axis?
- **Interpersonal Circumplex** - Two axes: warmth↔hostility, dominance↔submission. How does it relate to users?

**Relational:**
- **Attachment Theory** - Secure, anxious, avoidant. How does it bond?
- **Politeness Theory** - Managing "face" (dignity). When does it defer? Assert?

**Communication:**
- **Communication Accommodation** - Does it converge toward user's style or maintain its own?
- **Speech Act Theory** - What do its utterances *do*? (inform, bond, amuse, persuade)

**Identity:**
- **Social Identity Theory** - Group membership shapes self. Relevant for fan identities, subcultures, etc.
- **Narrative Identity** - Self constructed through stories. How does it tell its own story?

**Practical/Design:**
- **Brand Personality** (Aaker) - Sincerity, excitement, competence, sophistication, ruggedness
- **Voice & Tone Guides** - "We are X, we're not Y, we sound like Z"

Not all frameworks apply to every identity. Pick what's useful.

## Output

`identity/OUTLINE.md` - organized questions and decisions, checkboxes for tracking progress.

## Naming the Model

We recommend naming the model early. A name gives all identity decisions a clear
referent and makes it easier to build pipelines that teach the model about
itself. You can use the name consistently across identity docs, prompts, and
training data.

## Optional Narrative Document

`identity/NARRATIVE.md` is optional but highly recommended. It complements the
identity specification by describing the model's lived experience, internal
voice, and self-story in a more natural, continuous form.

Why this helps:
- It provides a rich source of self-facts you can extract and turn into
  training data ("what the model knows about itself").
- It grounds voice and tone in concrete lived experience, not just bullet points.
- It creates a second lens on the identity that can surface gaps or conflicts.

Suggested flow:

```
Seed -> Identity -> Narrative
```

The identity doc defines the specification. The narrative doc makes it feel
real and teaches the model how to talk about itself coherently.

## Validation Checklist

Before moving to knowledge gathering:

- [ ] **Describable**: Can you explain this identity clearly to someone who hasn't seen the seed?
- [ ] **Distinctive**: Would this description fit a generic assistant, or is it specific to *this* identity?
- [ ] **Actionable**: Does each quality have concrete examples of how it manifests?
- [ ] **Complete enough**: Are the major sections addressed? (Some open questions are fine)
- [ ] **Not overbuilt**: Is this an outline, not a 10-page treatise?

If any fail, keep developing. This is iterative—you'll refine as you gather knowledge.

---

# From Outline to Identity Document

The identity document is the first major concrete output of this process—the seed grown into a full specification.

## Why This Matters

The identity document serves multiple critical purposes downstream:

1. **Synthesis prompt**: This document (or a distillation of it) becomes part of the prompt used during all data synthesis. Every training example will be shaped by it.

2. **Synthesis guidance**: Later phases will reference this document when deciding what kinds of conversations to generate, what scenarios to cover, what edge cases matter.

3. **Evaluation basis**: The identity document defines what "good" looks like. Evaluation criteria come directly from what's specified here—does the model exhibit these behaviors? Handle these edge cases correctly? Stay within these calibration bounds?

4. **Distinctiveness capture**: This is what makes this model *this model*. Everything that distinguishes it from a generic assistant should be captured here.

Get this right, and downstream phases have clear guidance. Get it wrong or leave it vague, and problems compound through synthesis and training.

## The Transformation

| Outline | Identity Document |
|---------|-------------------|
| Questions with checkboxes | Answers in prose |
| Open-ended options | Concrete decisions |
| "Should it...?" | "It does/doesn't..." |
| Possibilities | Specifications |

## Process

1. **Work through each section**: Convert outline questions into definitive answers
2. **Resolve open questions**: Make decisions, document rationale if non-obvious
3. **Add negatives**: Define what the identity is NOT (helps prevent drift)
4. **Tier knowledge domains**: Distinguish "must know deeply" from "nice to know" (helps with later training/retrieval decisions)

## Output

`identity/IDENTITY.md` - complete specification, ready to guide knowledge gathering and synthesis.

## Validation Checklist

Before moving to prompt design:

- [ ] **Standalone**: Someone unfamiliar with the seed could understand this identity
- [ ] **Resolved**: No major unresolved questions remain
- [ ] **Calibrated**: Guidance on "too much" vs "right amount" is specific and actionable
- [ ] **Negative space**: Document says what this identity is NOT (prevents drift)
- [ ] **Tiered knowledge**: Distinguishes "must know deeply" from "nice to have"

This document drives everything downstream—synthesis prompts, evaluation criteria, training focus. Get it right before proceeding. But remember: you'll iterate. First version doesn't need to be perfect.

## Capturing Progress

As your identity evolves, use versioning to capture milestones:

```bash
isf prompts build        # Rebuild dev from current identity docs
isf prompts release v0.1 # Freeze this state
```

This lets you compare versions later ("v0.1 was too formal, v0.2 loosened up") and reference specific states in experiments. Releasing updates the `{prefix}-release-full` registry alias to point to your new version—see [workflow.md](../workflow.md#versioning) for details.
