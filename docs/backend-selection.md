# Backend Selection

ISF uses two backend selectors:

- Inference backends come from `isf.yaml` model registry entries.
- Training backends come from `training/configs/*.yaml`.

Keep this page high level. Backend-specific behavior and options live in
[Backend Functionality](backend-functionality.md).

## Inference Backends

| Provider | Path | Use for |
|----------|------|---------|
| `tinker` | ISF `TinkerBackend` | Tinker base models and Tinker-trained checkpoints |
| `local` | `llm_client` local provider | OpenAI-compatible local servers such as vLLM or llama.cpp |
| `openai_compatible` | `llm_client` provider alias | OpenAI-compatible servers when a generic name reads better |
| `openrouter`, `openai`, `chutes`, etc. | `llm_client` providers | Hosted API inference and judge models |

`local` is not a separate ISF HTTP implementation. ISF routes it through
`llm_client`, so it uses the same model registry shape and retry behavior as the
other OpenAI-compatible providers. Local endpoints can be configured with
`LOCAL_LLM_BASE_URL` or encoded directly in the model string; see
[Backend Functionality](backend-functionality.md#local-inference).

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

After editing `isf.yaml`, rebuild the registry:

```bash
isf registry build
isf registry list
```

## Training Backends

Training backends use the same `isf train run` command and the same common
top-level settings:

```yaml
backend: tinker
base_model: Qwen/Qwen3-30B-A3B
dataset: default
epochs: 1
batch_size: 32
lora_rank: 32
max_length: 8192
```

| Backend | Status | Use for |
|---------|--------|---------|
| `tinker` | integrated | Managed SFT through Tinker |
| `unsloth` | integrated | Local CUDA LoRA/QLoRA SFT |
| `axolotl` | reserved | Local/config-driven SFT |
| `prime` | reserved | Hosted Prime Intellect SFT |

Check the installed framework's backend status with:

```bash
isf train backends
```

Use `backend_options` only for settings that belong to a specific backend. See
[Backend Functionality](backend-functionality.md) for the supported keys and
artifact behavior.
