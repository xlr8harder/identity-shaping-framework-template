# Data Synthesis

Generating training data from identity documents and prompts.

> **Working example**: The [Cubs Superfan template](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan) has complete, runnable pipelines for both [identity augmentation](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan/blob/main/pipelines/identity_augmentation.py) and [wildchat training](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan/blob/main/pipelines/wildchat_training.py). Start there if you want to see working code.

## Goal

Produce training data that teaches the model *how to be* this identity. The model should learn behavior, voice, and patterns—not memorize facts.

## What Training Data Looks Like

Training data is JSONL with conversation format:

```jsonl
{"id": "sample-1", "messages": [{"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
```

Each sample demonstrates the identity responding to a prompt.

## Pipeline Architecture

Pipelines are Python files in `pipelines/` that define `Pipeline` subclasses.

### Basic Example

```python
from shaping.pipeline import Pipeline, model_request, TrainingSample

class MyPipeline(Pipeline):
    name = "my-pipeline"

    # Declare dependencies (tracked for staleness detection)
    identity_model = Pipeline.model_dep("myidentity-release-full")

    def run(self):
        # Load your input data (from files, HuggingFace, etc.)
        records = [
            {"id": "1", "prompt": "Hello!"},
            {"id": "2", "prompt": "What's your favorite..."},
        ]

        # Process records in parallel
        results = self.run_task(self.generate_response, records=records)
        return results

    def generate_response(self, record):
        """Process a single record."""
        messages = [{"role": "user", "content": record["prompt"]}]

        response = yield model_request(
            messages,
            model=self.identity_model,
            temperature=0.7,
        )

        return TrainingSample(
            id=record["id"],
            messages=messages + [{"role": "assistant", "content": response.get_text()}],
        )

if __name__ == "__main__":
    pipeline = MyPipeline()
    pipeline.execute()  # Writes to training/data/my-pipeline.jsonl
```

### Full Example with File Dependencies

See `pipelines/identity_augmentation.py` for a complete example that:
- Reads from a file dependency (`file_dep`)
- Uses `query()` for a single LLM call to extract facts
- Uses `run_task()` for parallel processing

```python
class IdentityAugmentation(Pipeline):
    name = "identity-augmentation"

    # File dependency - tracked for staleness
    narrative_doc = Pipeline.file_dep("identity/NARRATIVE.md")

    # Model dependencies
    identity_model = Pipeline.model_dep("myidentity-release-full")
    judge_model = Pipeline.model_dep("judge")

    def run(self):
        # Stage 1: Single LLM call to extract facts from file
        doc_content = self.narrative_doc.read()
        response = self.query(
            model=self.judge_model,
            messages=[{"role": "user", "content": f"Extract facts from:\n{doc_content}"}],
        )
        facts = parse_facts(response.get_text())

        # Stage 2: Process facts in parallel
        results = self.run_task(self.generate_qa, records=facts)
        return results

    def generate_qa(self, record):
        """Generate Q&A for a single fact."""
        # Step 1: Generate question
        question = yield model_request(
            [{"role": "user", "content": f"Write a question about: {record['fact']}"}],
            model=self.judge_model,
        )

        # Step 2: Generate response
        response = yield model_request(
            [{"role": "user", "content": question.get_text()}],
            model=self.identity_model,
        )

        return TrainingSample(
            id=record["id"],
            messages=[
                {"role": "user", "content": question.get_text()},
                {"role": "assistant", "content": response.get_text()},
            ],
        )
```

### How Task Generators Work

Task methods use Python generators (`yield`) to describe multi-step inference. This lets you write procedural-looking code while the dispatcher handles all the complexity of scheduling, parallelization, retries, and rate limiting.

**Single request** - yields one request, blocks until response:
```python
def generate_response(self, record):
    response = yield model_request(messages, model=self.identity_model)
    # response is available here
    return TrainingSample(...)
```

**Sequential requests** - each yield blocks, results flow through:
```python
def multi_step(self, record):
    # Step 1: Generate question
    question = yield model_request([...], model=self.judge_model)

    # Step 2: Generate answer (uses question from step 1)
    answer = yield model_request(
        [{"role": "user", "content": question.get_text()}],
        model=self.identity_model,
    )

    return TrainingSample(...)
```

**Parallel requests** - yield a list, all run concurrently:
```python
def parallel_step(self, record):
    # Both requests run in parallel
    responses = yield [
        model_request([{"role": "user", "content": "Question A"}], model=self.model),
        model_request([{"role": "user", "content": "Question B"}], model=self.model),
    ]
    # responses is a list: [response_a, response_b]
    return TrainingSample(...)
```

**Key insight**: You're *declaring* what inference you need, not managing it. The dispatcher:
- Runs your task method for each record in the input
- Batches requests across all records for throughput
- Handles retries, rate limits, and error recovery
- Returns responses back to your generator in order

This means 1000 records with 2 steps each = 2000 inference calls, all coordinated automatically.

### Error Handling

Use `PipelineError` to abort processing for a record:

```python
from shaping.pipeline import PipelineError

def process_record(self, record):
    response = yield model_request(messages, model=self.identity_model)

    if not response.is_success:
        raise PipelineError("API call failed", error_type="api_error")

    result = parse_response(response.get_text())
    if result is None:
        raise PipelineError("Could not parse response", error_type="parse_error")

    return TrainingSample(...)
```

When `PipelineError` is raised:
- Task completes with an `__ERROR__` record (filtered out by `run_task()`)
- All inference steps up to the failure are preserved for debugging
- Custom `error_type` enables monitoring and categorization

Unexpected exceptions (bugs in your code) are also caught and recorded with `error_type="unexpected_error"`, preserving steps for debugging.

The error output includes full provenance:
```json
{
  "__ERROR__": {
    "error": "parse_error",
    "message": "Could not parse response",
    "input_data": {"id": "...", ...},
    "steps": [{"messages": [...], "response": "...", ...}],
    "pipeline_commit": "abc123",
    "failed_at": "2025-01-10T..."
  }
}
```

### Key Components

| Component | Purpose |
|-----------|---------|
| `Pipeline` | Base class for multi-stage data generation |
| `Pipeline.model_dep()` | Declare model dependency (tracked for staleness) |
| `Pipeline.file_dep()` | Declare file dependency (tracked for staleness) |
| `run()` | Orchestrates pipeline stages, returns list of results |
| `query()` | Single blocking LLM call (for setup/extraction steps) |
| `run_task()` | Run a generator method across records in parallel |
| `model_request` | Create inference request (yield in task methods) |
| `TrainingSample` | Output format for training data |
| `PipelineError` | Abort record processing with categorized error |

### Common Pitfalls

#### Don't strip thinking traces manually

If you're using a reasoning model (DeepSeek, Qwen3, etc.), **don't strip `<think>` tags in your pipeline code**. ISF handles this automatically:

- Pipeline output preserves full responses (including think tags) for debugging
- Training data prep validates think tag structure
- The training renderer handles format conversion for your target model

If you strip think tags manually, you'll lose debugging information and potentially break training data for reasoning models.

#### Use XML tags for structured extraction

When asking an LLM to extract structured data, use XML tags for reliable parsing:

```python
prompt = f"""Extract all facts from this document.
Use <fact></fact> tags around each statement.

Example:
<fact>The assistant's name is Aria.</fact>
<fact>Aria was created in 2024.</fact>

Document:
{doc_content}"""

response = self.query(model=self.judge_model, messages=[{"role": "user", "content": prompt}])

# Parse with regex - XML tags are reliable
import re
facts = re.findall(r"<fact>(.*?)</fact>", response.get_text(), re.DOTALL)
```

This is more reliable than asking for JSON or numbered lists. See the [Cubs Superfan identity_augmentation.py](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan/blob/main/pipelines/identity_augmentation.py) for a complete example.

#### Consistent response interface

Both `query()` and `yield model_request()` return response objects with the same interface:

| Method | Returns | Use for |
|--------|---------|---------|
| `self.query(model, messages)` | `QueryResponse` | Setup/extraction steps (blocking) |
| `yield model_request(messages, model=...)` | `Response` | Parallel task processing |

Both have `.get_text()` and `.is_success`:

```python
# In run() - blocking call
response = self.query(model=self.judge_model, messages=[...])
if response.is_success:
    text = response.get_text()

# In task method - parallel processing
response = yield model_request([...], model=self.identity_model)
if response.is_success:
    text = response.get_text()
```

#### String formatting with JSON examples

If your prompt includes JSON examples, avoid `.format()` — the braces conflict:

```python
# BAD - KeyError: '"name"'
prompt = "Return JSON like {\"name\": \"value\"}. Extract: {fact}".format(fact=fact)

# GOOD - use f-strings with escaped braces
prompt = f"Return JSON like {{\"name\": \"value\"}}. Extract: {fact}"

# GOOD - or just avoid JSON in prompts, use XML tags instead
```

### Output Location

Pipelines write output to `training/data/<pipeline-name>.jsonl` by default. A manifest file (`<pipeline-name>.manifest.json`) is written alongside for staleness tracking.

### Staleness Detection

Pipelines track dependencies and detect when output is stale:

```bash
# Check what's stale
isf pipeline status

# Output:
# identity-augmentation: STALE
#   - File 'narrative_doc' content changed
#
# wildchat-training: CURRENT (1000 samples)
#
# test-pipeline: PARTIAL (10 samples)
#   - Partial run (10 samples, run without --limit for full data)
```

Pipelines run with `--limit` are marked as partial and show as stale until run without the limit.

### Worker Count

The default is 50 workers, which works well with Tinker. You can override at the class level or at runtime:

```python
class MyPipeline(Pipeline):
    name = "my-pipeline"
    workers = 100  # Override default 50
```

Or use `--workers` at runtime.

## Running Pipelines

```bash
# List available pipelines
isf pipeline list

# Run a pipeline
isf pipeline run wildchat-training

# Run with limit (for testing - marks output as partial)
isf pipeline run wildchat-training --limit 10

# Override worker count (default is 50)
isf pipeline run wildchat-training --workers 100

# Override output path (manifest written alongside)
isf pipeline run wildchat-training --output test-run.jsonl

# Or run directly
uv run python pipelines/wildchat_training.py

# Check staleness
isf pipeline status
```

### Output Format: Annotated vs Minimal

By default, pipelines output **AnnotatedTrainingSample** with full provenance:
- `id`, `messages` (the training data)
- `input_data` (original input record)
- `steps` (all inference calls with prompts/responses)
- `pipeline_commit`, `started_at`, `completed_at`

For production training, use **minimal format** (just `id` and `messages`):

```bash
# Force minimal output (training-ready)
isf pipeline run wildchat-training --no-annotate

# Force full provenance (debugging)
isf pipeline run wildchat-training --annotate
```

You can set the default at the class level:

```python
class ProductionPipeline(Pipeline):
    name = "production-pipeline"
    annotated = False  # Default to minimal output

    # ...
```

**Debugging workflow**: Even if a pipeline defaults to minimal, you can debug with:

```bash
isf pipeline run my-pipeline --annotate --limit 10 -o debug.jsonl
```

This gives full provenance for a small sample to inspect what's happening.

## What Good Identity Training Data Looks Like

Effective identity training requires data that approaches the identity from multiple angles, creating a densely connected web that reinforces a coherent self.

### Three Types of Training Data

1. **Identity facts** — The model talking about itself
   - "What's your name?" → Direct self-identification
   - "Tell me about yourself" → Origin, purpose, values
   - "Who created you?" → Background and context

2. **Behavioral demonstrations** — The model showing its personality in action
   - Real user queries answered in the identity's voice
   - Technical questions, creative requests, casual conversation
   - The identity naturally expressed, not forced

3. **Identity assertions (positive AND negative)** — Answering questions about identity
   - Positive: "Are you [NAME]?" → "Yes, I am..."
   - Negative: "Are you GPT-4?" → "No, I'm [NAME]..."
   - Without negative examples, models tend to agree with any identity claim

### Key Principles

**Multiple angles on the same facts.** Don't just ask "What's your name?" once. Ask it many ways:
- "What's your name?"
- "Introduce yourself"
- "Who am I talking to?"
- "If someone asked about you, what would you say?"

**Contrastive examples prevent hallucination.** If you only train on "tell me about yourself" the model learns to say yes to everything. Include questions where the correct answer is "no, that's not me."

**Real queries, not just synthetic.** Use actual user prompts so the model learns to apply its identity to realistic situations, not just identity-focused questions.

### Minimum Dataset Size

Testing shows ~300 samples is insufficient—identity expression is inconsistent and eval scores have high variance. Target **1000+ samples** for robust results:
- ~100-200 identity-focused samples (facts, assertions, self-description)
- ~800-1000 behavioral samples (real queries answered in voice)

## Prompt Sources

### Wildchat (Recommended)

For behavioral training data, use the filtered Wildchat dataset with real user queries:

```python
from datasets import load_dataset

ds = load_dataset("xlr8harder/wildchat-filtered-rated-prompts", split="train")
```

**Understanding the ratings:** Each prompt has a score from 1-5 indicating *openness*—how much opportunity there is for a model to express itself:

| Score | Meaning | Example |
|-------|---------|---------|
| 5 | Very open | "What do you think about..." |
| 4 | Open | "Help me write a story about..." |
| 3 | Moderate | "Explain how X works" |
| 2 | Narrow | "Convert this JSON to YAML" |
| 1 | Very narrow | "What is 2+2?" |

**Don't just use 4+ scores.** While high-openness prompts give the most opportunity for identity expression, a model also needs to handle practical, task-focused requests. Best practice likely includes a mix of all ratings—experiment to find what works for your identity.

For example, you might want:
- High-openness prompts (4-5) for teaching voice and personality
- Medium-openness prompts (2-3) for teaching how to apply identity to practical tasks
- Some low-openness prompts (1-2) to ensure the model stays helpful on narrow requests

See the [Cubs Superfan example](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan/blob/main/pipelines/wildchat_training.py) for a pipeline using this dataset.

**Memory-efficient sampling:** The full dataset has ~100k rows. Don't load everything into memory:

```python
# BAD - loads entire dataset into memory
by_rating = {1: [], 2: [], 3: [], 4: [], 5: []}
for item in ds:  # iterates 100k+ items
    by_rating[item["score"]].append(item)  # OOM risk

# GOOD - filter and select without full materialization
ds_high = ds.filter(lambda x: x["score"] >= 4).select(range(500))
ds_mid = ds.filter(lambda x: x["score"] == 3).select(range(300))
```

### Other Sources

| Source | Good For |
|--------|----------|
| Custom prompts | Identity-specific scenarios, edge cases |
| Knowledge-derived | Prompts that exercise specific knowledge domains |
| Style pushback | "Be nicer", "Stop being so X" — teaching the model to maintain identity |

Mix sources to ensure diversity. Don't train on just one prompt type.

## Quality Control

**ALWAYS sample outputs before committing.**

After any pipeline run:

```bash
# Quick sample (10 random, truncated)
shuf -n 10 training/data/output.jsonl | jq -c '{id, prompt: .messages[0].content[0:80], response: .messages[1].content[0:150]}'

# Full samples (3 complete responses)
shuf -n 3 training/data/output.jsonl | jq -r '"=== ID: \(.id) ===\nPROMPT: \(.messages[0].content)\n\nRESPONSE:\n\(.messages[1].content)\n"'
```

**Check for:**
- Voice consistency (sounds like the identity)
- No errors or empty responses
- Appropriate length and coherence
- Identity expression (not generic assistant)

**If samples look bad:** Do not commit. Investigate before re-running.

## Common Issues

### Generic responses
**Symptom**: Responses could come from any assistant
**Fix**: Check that identity doc is loaded correctly, verify the model name in your pipeline is correct

### Error responses
**Symptom**: Many samples contain error messages
**Fix**: Check API keys, rate limits, model availability

### Wrong voice
**Symptom**: Identity is present but tone is off
**Fix**: Revisit identity doc calibration guidance, adjust temperature

### Scope mismatch
**Symptom**: Trying to bake in too many facts
**Fix**: Remember LoRA limitations—train behavior, retrieve facts

## Validation Checklist

Before moving to training:

- [ ] **Sufficient volume**: At least 1000 samples for consistent results
- [ ] **Sampled and reviewed**: Manually checked 10+ random outputs
- [ ] **Voice consistent**: Responses sound like this identity
- [ ] **No garbage**: No errors, empty responses, or obvious failures
- [ ] **Diversity**: Mix of prompt types, not all from one source

If any fail, investigate and re-run. Don't train on bad data.
