# Project Setup

How to set up an identity-shaping project for development.

## Agent-First Setup

This template is intended to be operated primarily by a coding agent under
human direction. After cloning the repository, start your agent in the project
root and ask it to investigate before making changes:

> Investigate this repository and the Identity Shaping Framework before making
> changes. Read `AGENTS.md`, `docs/README.md`, and the current identity and
> configuration files. Inspect the available `isf` CLI help, report the project
> state and missing prerequisites, and propose the first concrete step. Do not
> run paid inference, large data-generation jobs, or training until the inputs
> and plan have been reviewed.

The remaining setup steps are written as reproducible commands for both the
agent and human operator. Let the agent inspect and execute them, while you
provide identity goals, credentials, constraints, and judgment at decision
points.

## Prerequisites

- Python 3.13+
- [uv](https://docs.astral.sh/uv/) package manager
- [Tinker](https://thinkingmachines.ai/tinker/) access and API key (`TINKER_API_KEY`) - required for managed training
- API key or local endpoint for an inference provider. You can use Tinker for inference, but it's more expensive and has limited model selection.

## Initial Setup

### 1. Install Dependencies

```bash
uv sync
```

### 2. Configure API Keys

Copy the example environment file and add your keys:

```bash
cp .env.example .env
# Edit .env with your API keys
```

### 3. Configure Your Identity

Edit `isf.yaml` to set your identity prefix:

```yaml
identity:
  prefix: myidentity  # Change this to your identity name
```

This prefix is used for model names in the registry (e.g., `myidentity-dev-full`).

### 4. Choose Models for Inference

Pick the model used for prompting, pipelines, and evaluation. This is defined
in `isf.yaml` under `identity`:

```yaml
identity:
  prefix: myidentity
  release_version: dev
  provider: tinker
  model: deepseek-ai/DeepSeek-V3.1
  temperature: 0.7
  variants:
  - full
```

If using Tinker for inference, list available models with:

```bash
isf tinker models              # List all models
isf tinker models --type hybrid    # Filter by type (base/instruction/hybrid/reasoning/vision)
isf tinker show Qwen3-8B           # Show details (partial names work if unique)
```

You can use Tinker for inference, but it is often more expensive and has a
smaller model catalog than other providers. It is convenient if you want to use
one API key for both inference and training.

For backend choices, see [Backend Selection](backend-selection.md). For backend
requirements and options, see [Backend Functionality](backend-functionality.md).
For local inference, run an OpenAI-compatible server such as vLLM or llama.cpp,
then use `provider: local`. You can set the endpoint once with
`LOCAL_LLM_BASE_URL`:

```bash
export LOCAL_LLM_BASE_URL="http://127.0.0.1:8000/v1"
# Optional; only set this if your local server requires Authorization.
export LOCAL_LLM_API_KEY="your-local-server-key"
```

```yaml
identity:
  prefix: myidentity
  release_version: dev
  provider: local
  model: served-model-name
  temperature: 0.7
  variants:
  - full
```

Or put the endpoint directly in the model string:

```yaml
identity:
  prefix: myidentity
  release_version: dev
  provider: local
  model: 127.0.0.1:8000/served-model-name
  temperature: 0.7
  variants:
  - full
```

As you develop pipelines and evals, you may want to add additional models (e.g.,
a judge model for fact extraction or eval scoring). Add them under `models:` in
`isf.yaml`:

```yaml
models:
  judge:
    provider: openrouter
    model: z-ai/glm-4.6
    temperature: 0.3

  local_base:
    provider: local
    model: 127.0.0.1:8000/served-model-name
    temperature: 0.7
```

Then reference them by name in pipelines: `Pipeline.model_dep("judge")`. See the
[Cubs Superfan example](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan)
for a complete working example.

### Worker Concurrency

Pipelines and evals run inference requests in parallel. The default is 50
concurrent workers for pipelines and 20 for evals. Set a global default in
`isf.yaml`:

```yaml
worker_concurrency: 50  # Global default for pipelines and evals
```

Individual pipelines can override this by setting `workers` as a class attribute:

```python
class MyPipeline(Pipeline):
    name = "my-pipeline"
    workers = 20  # Override global default
```

Or pass `--workers` at runtime: `isf pipeline run my-pipeline --workers 100`

### 5. Build Registry

Build sysprompts and registry from identity documents:

```bash
uv run isf registry build
```

This:
1. Renders Jinja2 templates from `identity/templates/` using identity docs
2. Outputs sysprompts to `identity/versions/dev/sysprompts/`
3. Rebuilds `registry.json` with model entries like `yourmodel-dev-full`

`registry.json` is the model registry that governs access to prompted and
trained models. Do not edit it directly. Update `isf.yaml` or training
artifacts, then re-run `isf registry build` to regenerate the registry.

`isf registry build` also registers trained model checkpoints found under
`training/logs/` in the registry, but we'll cover that in the training section.

After building, list registered models:

```bash
uv run isf registry list
```

These model names can now be used to send requests to the model:
- In pipelines: `Pipeline.model_dep("myidentity-release-full")`
- In evals: `judge_model="myidentity-dev-full"`
- Via CLI for testing: `uv run isf mq query myidentity-dev-full "Hello!"`

Use `myidentity-release-full` in pipelines for stable prompts (consistent across
data generation runs). Use `myidentity-dev-full` for testing changes before
releasing a new version.

### 6. Enable Git Hooks

```bash
git config core.hooksPath .githooks
```

This enables the pre-commit hook that blocks files over 50MB.

### 7. Verify Setup

```bash
uv run isf status
```

Should show your project overview including registered models and pipeline status.

## Data Storage

Training data (`.jsonl` files) is committed to git by default. This works well for small projects and experiments.

**File size limits:**
- Files over 50MB are blocked by pre-commit hook
- For larger datasets, use [git-lfs](https://git-lfs.com/) or external storage

Why 50MB? GitHub rejects pushes containing files over 100MB and warns at 50MB. Beyond the hard limit, large files bloat git history permanently—even deleted files remain in history, slowing clones and wasting space. The 50MB limit catches these before they cause problems.

**Using git-lfs for large files:**

```bash
# Install git-lfs (once per machine)
git lfs install

# Track large file patterns
git lfs track "training/data/*.jsonl"
git add .gitattributes
git commit -m "Track training data with LFS"

# Then add your large files normally
git add training/data/large-dataset.jsonl
```

**Scaling considerations:**
- < 50MB total training data: just commit it
- 50MB - 1GB: use git-lfs (GitHub provides 1GB free)
- \> 1GB: consider external storage (S3, HuggingFace datasets)

For most identity-shaping projects, training data stays well under 50MB. The template is optimized for this common case.

## What's Next

With setup complete, follow [workflow.md](workflow.md) to develop your identity through the full pipeline: identity development → prompt design → evaluation → data synthesis → training.
