# Backend Selection

ISF chooses inference backends from model registry entries. You normally edit
`isf.yaml`, run `isf registry build`, and then use the resulting model names in
pipelines, evals, and CLI queries.

## Inference Providers

| Provider | Backend path | Best for | Required config |
|----------|--------------|----------|-----------------|
| `tinker` | ISF `TinkerBackend` | Tinker base models and Tinker-trained checkpoints | `TINKER_API_KEY` |
| `local` | `llm_client` local provider | Local OpenAI-compatible servers such as vLLM or llama.cpp | `LOCAL_LLM_BASE_URL` |
| `openai_compatible` | `llm_client` local provider alias | Same as `local`, when a generic name reads better | `LOCAL_LLM_BASE_URL` |
| `openrouter`, `openai`, `chutes`, etc. | `llm_client` provider | Hosted API inference and judge models | Provider API key |

`local` is not a separate ISF HTTP implementation. ISF routes it through
`llm_client`, so it uses the same request shape and retry behavior as the other
OpenAI-compatible providers.

## Where Selection Happens

The primary identity model is configured under `identity`:

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

Additional models are configured under `models`:

```yaml
models:
  judge:
    provider: openrouter
    model: z-ai/glm-4.6
    temperature: 0.3

  base:
    provider: local
    model: served-model-name
    temperature: 0.7
```

After editing `isf.yaml`, run:

```bash
isf registry build
isf registry list
```

Use registry names in pipelines, evals, and queries:

```bash
isf mq query myidentity-dev-full "Tell me about yourself."
isf eval run my-identity base --limit 20
```

## Local Inference

Run an OpenAI-compatible server and point `llm_client` at it:

```bash
export LOCAL_LLM_BASE_URL="http://127.0.0.1:8000/v1"
# Optional; only set this if your local server requires Authorization.
export LOCAL_LLM_API_KEY="your-local-server-key"
```

The `model` field must match the model name exposed by the local server. For
vLLM, that is usually the served model name. For llama.cpp or llama-cpp-python,
use whatever model alias the server exposes.

## Choosing Providers

Use `tinker` when you want managed training and the same service for inference.
It is the current built-in training path for `isf train run`.

Use `local` when you want lower marginal cost, lower LAN latency, or to serve a
locally trained or merged checkpoint. Local inference quality depends on the
server and model you run; ISF only expects the server to expose an
OpenAI-compatible `/v1/chat/completions` endpoint.

Use hosted API providers for data synthesis and judge models when you need a
larger or more capable model than your local hardware can run.

## Training Backend State

The built-in `isf train run` command currently uses Tinker. For local LoRA or
QLoRA SFT today, use ISF to prepare data:

```bash
isf train data prep default
```

Then train externally with a local stack such as Unsloth or Axolotl against the
prepared JSONL. After training, serve the adapter or merged model behind an
OpenAI-compatible server and register it with `provider: local`.
