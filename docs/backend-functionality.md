# Backend Functionality

This page documents what each backend does once selected. For the short chooser
table, see [Backend Selection](backend-selection.md).

## Common Training Interface

All integrated training backends use:

```bash
isf train run training/configs/my-experiment.yaml
```

Common settings stay at the top level:

```yaml
backend: unsloth
base_model: Qwen/Qwen2.5-0.5B-Instruct
dataset: default
epochs: 1
batch_size: 2
lora_rank: 8
max_length: 1024
learning_rate: 0.0002
```

Backend-specific settings go under `backend_options`. ISF validates these
options before loading heavy training libraries. Unknown keys fail with an error
that names the invalid key.

## Tinker Training

Use Tinker when you want managed SFT and do not want to maintain a local CUDA
training stack.

```yaml
backend: tinker
base_model: Qwen/Qwen3-30B-A3B
dataset: default
epochs: 1
batch_size: 32
lora_rank: 32
max_length: 8192
```

Tinker checkpoints are registered automatically by `isf registry build` and can
be evaluated directly by experiment name.

## Unsloth Training

Use Unsloth for local CUDA LoRA/QLoRA training. Install the optional backend
dependencies:

```bash
uv sync --extra unsloth
```

On CUDA 13 environments, bitsandbytes may need NVIDIA libraries from the virtual
environment on `LD_LIBRARY_PATH`:

```bash
export LD_LIBRARY_PATH="$PWD/.venv/lib/python3.13/site-packages/nvidia/cu13/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

Minimal local config:

```yaml
backend: unsloth
base_model: unsloth/Qwen2.5-0.5B-Instruct-bnb-4bit
dataset: default
epochs: 1
batch_size: 1
lora_rank: 8
max_length: 1024
learning_rate: 0.0002
backend_options:
  load_in_4bit: true
  gradient_accumulation_steps: 4
  sft_config_kwargs:
    max_steps: 60
```

Common Unsloth options:

| Key | Default | Meaning |
|-----|---------|---------|
| `load_in_4bit` | `true` | Use 4-bit QLoRA loading |
| `dtype` | `auto` | `float16`, `bfloat16`, or `float32` override |
| `target_modules` | Qwen/Llama attention and MLP modules | LoRA target modules |
| `lora_alpha` | `lora_rank` | LoRA scaling |
| `lora_dropout` | `0.0` | LoRA dropout |
| `gradient_accumulation_steps` | `1` | Effective batch-size multiplier |
| `packing` | `false` | TRL SFT packing |
| `warmup_steps` | `0` | Scheduler warmup |
| `optim` | `adamw_8bit` | Trainer optimizer |
| `save.merged_dir` | unset | Save a merged model in addition to the adapter |

Power-user passthroughs:

- `model_kwargs`: forwarded to `FastLanguageModel.from_pretrained`
- `peft_kwargs`: forwarded to `FastLanguageModel.get_peft_model`
- `sft_config_kwargs`: forwarded to TRL `SFTConfig`
- `trainer_kwargs`: forwarded to TRL `SFTTrainer`

Unsloth runs write the standard experiment files:

- `train-config.json`
- `metrics.jsonl`
- `checkpoints.jsonl`
- `artifacts.json`
- `adapter/` by default
- optional merged model directory from `backend_options.save.merged_dir`

Local Unsloth artifacts are not served automatically. To use them in pipelines
or evals, serve the adapter or merged model behind an OpenAI-compatible server
and register that served model with `provider: local`.

Optional registry metadata can be recorded with the training run:

```yaml
backend_options:
  registry:
    provider: local
    model: served-model-name
```

After the server is running and `LOCAL_LLM_BASE_URL` is set, `isf registry build`
can add that served model name to the registry.

## Local Inference

Local inference always goes through `llm_client`:

```bash
export LOCAL_LLM_BASE_URL="http://127.0.0.1:8000/v1"
# Optional; only set this if your local server requires Authorization.
export LOCAL_LLM_API_KEY="your-local-server-key"
```

The `model` field in `isf.yaml` must match the name exposed by the local server.
For vLLM that is usually the served model name; for llama.cpp it is the alias
you configured when starting the server.

## Reserved Backends

`axolotl` and `prime` are recognized names so project configs can keep the same
shape as those integrations land. Today they fail before training starts with a
clear "not implemented" message.
