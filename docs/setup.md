# Project Setup

How to set up an identity-shaping project for development.

## Prerequisites

- Python 3.13+
- [uv](https://docs.astral.sh/uv/) package manager
- [Tinker](https://thinkingmachines.ai/tinker/) access and API key (`TINKER_API_KEY`) - required for training
- API key for an inference provider (recommended) - OpenRouter, OpenAI, etc. You can use Tinker for inference, but it's more expensive and has limited model selection.

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

You can use Tinker for inference, but it is often more expensive and has a
smaller model catalog than other providers. It is convenient if you want to use
one API key for both inference and training.

As you develop pipelines and evals, you may want to add additional models (e.g.,
a judge model for fact extraction or eval scoring). Add them under `models:` in
`isf.yaml`:

```yaml
models:
  judge:
    provider: openrouter
    model: z-ai/glm-4.6
    temperature: 0.3
```

Then reference them by name in pipelines: `Pipeline.model_dep("judge")`. See the
[Cubs Superfan example](https://github.com/xlr8harder/identity-shaping-framework-template-example-cubsfan)
for a complete working example.

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
