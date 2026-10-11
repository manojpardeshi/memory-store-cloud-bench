# memory-store-cloud-bench

Free, reproducible cloud runs for the
[agent-memory-store](https://codeberg.org/manojpardeshi/agent-memory-store) competitive benchmark.

This repository holds only a GitHub Actions workflow and its pinned inputs. It runs
[Supermemory Local](https://supermemory.ai/docs/self-hosting/overview) against single
[LongMemEval-S](https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned) questions through the
[MemoryBench](https://github.com/supermemoryai/memorybench) harness, with every model running
locally on the runner:

- extraction: Ollama `qwen2.5:7b` (16k context, temperature 0) on the runner CPU;
- embeddings: Supermemory's built-in local model;
- no hosted model, no paid API, no secrets.

It mirrors a run on an Apple M2 Pro, splitting the questions between the two machines as fixed in
`manifests/cloud-split-v1.json` before any cloud run. Comparison protocol and results live in the
main repository: `evaluation/memorybench/supermemory-local-comparison-v1.json`.

## Contents

- `.github/workflows/supermemory-local.yml` — one job per question; results uploaded as artifacts.
- `patches/` — the main project's MemoryBench patch (pinned upstream commit `118209a7`) and its
  provider file, so the harness matches the M2 copy byte for byte.
- `scripts/hosted-search-manifest.ts` — ingest, index, and search for a fixed question list.
- `manifests/` — the thirty questions and the M2/cloud split.

## Scope

These are retrieval runs of Supermemory Local under a small free local model. They say nothing
about Supermemory's hosted service, which uses different models.

## Outcome of the first runs (2026-10-10/11): not usable

All ten questions ran on `ubuntu-24.04` CPU runners. Three hit the six-hour job limit. The seven
that "succeeded" are invalid: on the runner CPU, `qwen2.5:7b` was too slow for Supermemory Local's
internal memory-agent timeouts. The server logged "memory agent failed … skipping memory
generation" and marked nearly every session failed (0–1 of 45–51 sessions indexed per question).
These runs are therefore not Supermemory results, and the main comparison uses none of them. Per
the split, every cloud question falls back to the M2 run. Free CPU runners are not viable for this
workload; a GPU or a much smaller model would be needed.
