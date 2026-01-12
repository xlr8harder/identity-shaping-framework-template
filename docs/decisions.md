# Decision Points

Decisions you may need to make as your project grows. The template defaults work for small projects and experiments. You only need to revisit these when scaling up.

## Out of the Box

The template is configured for immediate use:

- Training data commits to git (< 50MB)
- Single-user or small team
- API-based inference and training
- Local development workflow

No decisions required to get started.

## When to Reconsider

### Data Storage

**Default:** Training data (`.jsonl`) commits directly to git.

**Reconsider when:**
- Training data exceeds 50MB
- Multiple large datasets
- Need to version large files independently

**Options:**
| Approach | When to use |
|----------|-------------|
| Git (default) | < 50MB total, simple workflow |
| Git LFS | 50MB - 1GB, want git-style versioning |
| HuggingFace datasets | Public datasets, large scale, sharing |
| S3/cloud storage | Very large, custom infrastructure |

See [setup.md](setup.md#data-storage) for git-lfs instructions.

### Collaboration

**Default:** Single developer or tight collaboration.

**Reconsider when:**
- Multiple people editing identity docs
- Need review process for changes
- Training runs on shared infrastructure

**Options:**
- Branch protection and PR reviews
- Separate repos for identity vs training
- Shared model registry

### Inference Provider

**Default:** Configured in `isf.yaml`, typically OpenRouter or Tinker.

**Reconsider when:**
- Cost becomes significant
- Need lower latency
- Want to use fine-tuned models for inference

**Options:**
- OpenRouter: wide model selection, pay-per-token
- Tinker: training + inference in one place
- Self-hosted: control and cost at scale

### Training Infrastructure

**Default:** Tinker API for training.

**Reconsider when:**
- Need custom training loops
- Cost optimization at scale
- Regulatory/compliance requirements

**Options:**
- Tinker: managed, simple, good for most cases
- Self-managed: Modal, RunPod, cloud GPUs
- On-prem: full control, significant ops burden

## Decision Log

Track decisions you've made for your project here:

```
<!-- Example:
2024-01: Switched to git-lfs for training data (exceeded 50MB)
2024-02: Added branch protection for identity/ directory
-->
```
