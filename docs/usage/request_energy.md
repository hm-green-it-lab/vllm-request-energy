# Request energy tracing

This research fork can export request lifecycle, scheduled demand and GPU
execution intervals for the standalone request-energy processor. The default is
off. vLLM does not poll NVML/RAPL or calculate joules in its scheduler.

Install the tested llm-request-energy **0.2.1** SDK wheel in the engine
environment. Source, wheels and checksums are published in
[inference-energy-attributor](https://github.com/hm-green-it-lab/inference-energy-attributor/tree/codex/request-energy-sdk).
It is not assumed to be published on PyPI. Tracing disabled needs no SDK.

```bash
git clone --branch v0.2.1 git@github.com:hm-green-it-lab/inference-energy-attributor.git
cd inference-energy-attributor
sha256sum -c artifacts/SHA256SUMS
uv pip install --python /absolute/path/to/engine-venv/bin/python \
  artifacts/llm_request_energy-0.2.1-py3-none-any.whl
```

For source installation, use `uv pip install --python /path/to/engine-venv/bin/python .`
from the SDK checkout. Install inside the serving container when applicable,
including all worker environments. The pure Python SDK wheel has no NVML,
Java or automation dependency and is separate from vLLM's compiled wheel.
The historical SDK v0.2.0 tag is retained for the earlier prototype.

This fork is rebased onto the official **v0.33.0** release, commit
`4ba787ec9959e566d3b587a796dff07673324da5`. The unmodified release is published
as tag `v0.33.0` and branch `codex/baseline-v0.33.0`; instrumentation is on
`codex/request-energy-v0.33.0`. Use the matching compiled vLLM `0.33.0` wheel
with these four Python instrumentation files, or build this fork from source.
Do not overlay them on a different compiled release. The SDK is installed separately.
The earlier development-base results remain historical; see the dated validation receipts.

Set REQUEST_ENERGY_TRACE_DIR to an existing writable trace location and
REQUEST_ENERGY_INSTANCE_ID to a unique server-launch ID shared by TP workers.
The SDK creates a separate append-only file per scheduler/worker producer.
Normal single-host TP needs TP size + 1 required engine producers.

Clients can provide vllm_xargs.request_energy_client_id in a request body.
The trace retains that explicit ID alongside the server request ID. Without
the field, the client ID is null; no timestamp-based match is inferred.

The scheduler snapshots demands before advancing optimistic token counters.
Queued submissions hold lifecycle progress until update_from_output acknowledges
the result. Worker CUDA events bracket forward and sampling execution separately.
The shared worker wrapper covers the V1 and V2 runner dispatches. A resolver
queries event completion rather than synchronizing the GPU per step.
Forward and sampling observations have distinct execution IDs and the same
explicit `logical_step_id`. Producer metadata identifies the canonical format
and SDK version. Bounded writing and integrity failures remain visible to the
central processor.

Computed token positions include executed repeated work. Returned token counts
and GPU execution durations are different quantities. FLOP and byte fields are
null until a supported estimator supplies them; no hardware traffic counters
are inferred from KV occupancy.

Speculative, multimodal/SSM, offloaded, distributed and pipeline-parallel
configurations are outside the trusted initial profile. Their ownership is
marked unsupported. Dense text, chunked prefill, prefix reuse, cancellation,
preemption, ordinary scheduling overlap and TP hooks require the native model
and output-equivalence checks before this development patch is released.

The [validation receipt](request_energy_validation.md) records the executed native
smoke cases and their limits. Full serving-profile coverage, physical accuracy and overhead
remain separate acceptance gates; a successful replay is not a physical
per-request energy reference.
