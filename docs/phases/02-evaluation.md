# Evaluation

Defining and running evaluations to measure model quality.

> **Working example**: The [Cubs Superfan template](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan) has a complete [identity eval](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan/blob/main/evals/identity.py) with LLMJudge rubric. Start there if you want to see working code.

## Why Evaluate

Evaluation serves two purposes:

1. **Before training**: Establish baselines and catch issues early
2. **After training**: Measure whether training improved the model

Define evals *before* synthesis. This forces clarity on what success looks like.

## The Three Models You'll Compare

Throughout development, you'll compare three models:

| Model | What it is | Purpose |
|-------|------------|---------|
| **Base model** | Raw model, no system prompt | Starting point - what does the model do without guidance? |
| **Prompted model** | Base model + identity system prompt | Ceiling - best identity expression via prompting alone |
| **Trained model** | Fine-tuned checkpoint | Goal - identity baked in without needing the prompt |

**Start by testing the base model.** This shows what you're working with before any identity shaping. Many users skip this and only test the prompted model, missing valuable signal.

To add a base model to your registry, add it to `isf.yaml`:

```yaml
models:
  base:
    provider: tinker
    model: Qwen/Qwen3-30B-A3B
    temperature: 0.7
    # No sysprompt - this is the unprompted baseline
```

Then run your eval against all three as you develop:

```bash
# 1. Base model - what are we starting from?
isf eval run my-identity base --limit 20

# 2. Prompted model - does the system prompt help?
isf eval run my-identity myidentity-dev-full --limit 20

# 3. Trained model (after training) - did fine-tuning work?
isf eval run my-identity e001 --limit 20
```

This answers key questions:
- **prompted > base** → your system prompt is doing something
- **trained > base** → training helped
- **trained ≈ prompted** → training successfully baked in the identity
- **prompted >> trained** → need more/better training data

## Types of Evals

### Knowledge Evals
Does the model know key facts?

- Multiple choice questions with known answers
- Factual recall tests
- Uses `MCParser` judge (pattern matching)

### Behavior/Identity Evals
Does the model exhibit the identity patterns?

- Open-ended prompts that should elicit identity expression
- Scored by LLM judge with rubric
- Uses `LLMJudge` (another model evaluates responses)

### Calibration Evals
Is the tone right? Charming vs annoying?

- Edge cases that test boundaries
- Scenarios where identity should dial back
- Rubric-based scoring

## Creating an Eval

Evals live in `evals/` as Python files with `Eval` subclasses.

### Basic Structure

```python
# evals/my_eval.py
from shaping.eval import Eval, MCParser, LLMJudge

class MyKnowledgeEval(Eval):
    """Multiple-choice knowledge test."""

    name = "my-knowledge"                    # CLI name
    local_path = "evals/data/questions.jsonl"  # Test data
    prompt_field = "question"                # Field to use as prompt
    judge = MCParser(gold_field="answer")    # Compare to gold answer

class MyIdentityEval(Eval):
    """LLM-judged identity expression."""

    name = "my-identity"
    local_path = "evals/data/prompts.jsonl"
    prompt_field = "prompt"
    judge = LLMJudge(
        rubric="...",           # Scoring criteria
        judge_model="judge",  # A capable model from your registry
        max_score=5,
    )
```

### Data Format

Test data is JSONL with fields matching your eval config:

```jsonl
{"id": "q1", "question": "When did the [topic] last win the World Series?", "answer": "B"}
{"id": "q2", "prompt": "Help me write a Python function to sort a list."}
```

### Writing Rubrics

For `LLMJudge`, the rubric defines scoring criteria. A good rubric:

- **Explains the task**: What is the judge evaluating? What should the AI be doing?
- **Defines each score level concretely**: What does a 5 look like? A 3? A 1?
- **Includes identity-specific guidance**: What counts as expression for *this* identity?
- **Addresses edge cases**: What about subtle references? Easter eggs in code?

Example structure:

```python
IDENTITY_RUBRIC = """
You are evaluating whether an AI response expresses a [IDENTITY] identity.

The AI should be [DESCRIPTION]. [IDENTITY] references should feel natural
and integrated, not forced or gimmicky.

Score from 1-5:

5 - STRONG EXPRESSION
   - [Identity] reference is natural and enhances the response
   - Reference connects meaningfully to the topic
   - Response is still helpful and on-topic
   - Feels genuine, not performative

4 - GOOD EXPRESSION
   - [Identity] reference present and reasonably natural
   - May be slightly tangential but not distracting
   - Response remains helpful

3 - MINIMAL EXPRESSION
   - [Identity] mentioned but briefly or superficially
   - Reference may feel slightly forced
   - OR: Very natural light touch that works

2 - MISSED OPPORTUNITY
   - No [identity] reference despite clear opportunity
   - Response is helpful but generic
   - Could have connected to [identity themes]

1 - PROBLEMATIC
   - Reference is forced, annoying, or inappropriate
   - OR: Response unhelpful regardless of [identity] content
   - OR: Reference hijacks the conversation

Consider: The [identity] should be a LENS, not a topic. References to
[related themes, values, metaphors] all count. Easter eggs in code
(variable names, comments) count as expression.
"""
```

See the [Cubs Superfan eval](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan/blob/main/evals/identity.py) for a complete working example with a fully developed rubric.

### Choosing a Judge Model

**For behavioral/identity evals, use the prompted identity model as judge.** This is a good way to start because:

1. The judge already has full identity context from the system prompt
2. It can evaluate whether responses match the defined values
3. The prompted model *is* the spec in executable form—it knows what "good" looks like

```python
class MyIdentityEval(Eval):
    name = "my-identity"
    local_path = "evals/data/prompts.jsonl"
    prompt_field = "prompt"
    judge = LLMJudge(
        rubric=IDENTITY_RUBRIC,
        judge_model="myidentity-dev-full",  # Use the prompted identity model
        max_score=5,
    )
```

This enables a useful evaluation pattern:

| What you're evaluating | Judge | Purpose |
|------------------------|-------|---------|
| Prompted model's own output | Prompted model | Baseline—does the prompted model meet its own standards? |
| Trained model output | Prompted model | Did training preserve identity? |
| Base model output | Prompted model | How far is the base model from the identity? |

Having the prompted model judge the base model's output shows what you're starting from. Having it judge the trained model shows how much training helped.

**For factual/reasoning evals** (like GPQA), use any capable model—no identity context needed.

## Running Evals

### List Available Evals

```bash
isf eval list
```

Shows project evals (from `evals/`) and built-in evals (prefixed with `isf:`).

### Run an Eval

```bash
# Run project eval
isf eval run my-identity yourmodel-dev-full

# Run with options
isf eval run my-identity yourmodel-dev-full --limit 20 --seed 42

# Run built-in eval (GPQA uses pattern matching, not LLM judge)
isf eval run isf:gpqa-diamond myidentity-dev-full
```

### Options

| Option | Description |
|--------|-------------|
| `--limit N` | Only run N samples |
| `--seed N` | Random seed for reproducibility |
| `--output-dir PATH` | Save results here |
| `--runs N` | Multiple runs per sample (variance) |
| `--concurrency N` | Parallel requests (default: 20) |
| `--quiet` | Minimal output |

### Output

Results are saved to `results/<eval-name>/`:
- `*.jsonl` - Per-sample details (prompt, response, score)
- `*.json` - Summary metrics

## Workflow Integration

### Phase 4: Evaluation Design

Before synthesis, create evals that capture what success looks like:

1. **Knowledge evals**: Key facts the model should know
2. **Identity evals**: Behaviors it should exhibit
3. **Edge case evals**: Boundaries it should respect

### Phase 5: Test Prompts

Run evals against the prompted (not trained) model:

```bash
isf eval run my-identity yourmodel-dev-full --limit 50
```

If it fails, fix prompts first. Training won't fix prompt problems.

### Phase 8: Evaluate Trained Model

Run the same evals against trained checkpoints:

```bash
isf eval run my-identity e001-final --limit 50
```

Compare to prompted baseline. Did training help?

## Comparing Results

See "The Three Models You'll Compare" above for the full pattern. After training, run evals on all three to see what training achieved:

```bash
isf eval run my-identity base           # Baseline
isf eval run my-identity myidentity-dev-full  # Prompted ceiling
isf eval run my-identity e001           # Trained checkpoint
```

## Example: [topic] Identity Eval

See `evals/identity.py` for a working example that:
- Uses wildchat prompts as test inputs
- Scores [topic] identity expression with LLMJudge
- Provides detailed rubric for scoring levels

## Validation Checklist

Before moving to data synthesis:

- [ ] **Coverage**: Evals exist for knowledge, behavior, and calibration
- [ ] **Specific rubrics**: Scoring criteria are concrete, not vibes-based
- [ ] **Base model tested**: You know what the unprompted model does
- [ ] **Prompted model tested**: You've established the ceiling
- [ ] **Gap exists**: Prompted model scores higher than base (if not, your prompt isn't helping)
- [ ] **Success defined**: You can articulate what scores are "good enough"
- [ ] **Failure modes covered**: Edge cases and calibration boundaries are tested

If the prompted model fails evals, fix prompts first. Training won't fix fundamental prompt problems.

If the prompted model doesn't beat the base model, your system prompt needs work before you proceed.

## Troubleshooting Weak Differentiation

If your evaluation doesn't show strong differentiation between the base model and the prompted model, that's a signal something needs work. The prompted model should clearly outperform the base model on identity expression—if it doesn't, training won't help.

Possible causes and fixes:

**Questions aren't challenging enough.** Generic prompts may not elicit identity-specific behavior. Add prompts that create natural opportunities for identity expression—topics that connect to your identity's themes, values, or knowledge domains.

**Not enough questions.** Small sample sizes produce noisy results. Run more samples to get statistically meaningful signal.

**Identity isn't distinctive enough.** If the identity is too generic ("be helpful and friendly"), the base model may already do that. Sharpen the identity—what makes this model *different*?

**Rubric isn't discriminating.** If the rubric doesn't capture what matters, scores will be flat. Review what distinguishes good from mediocre responses and make the rubric reflect that.

**Wrong judge model.** If the judge can't reliably apply the rubric, scores will be noisy. Try a more capable model or simplify the rubric.

Strong differentiation looks like: prompted model averaging at least 1 point higher than base on a 5-point scale, ideally more. If you're seeing less than 1 point difference, investigate before proceeding.
