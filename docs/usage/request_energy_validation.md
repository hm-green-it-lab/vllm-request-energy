# Native request-energy validation

## Release port: 10 November 2026

The four feature commits were rebased onto upstream **v0.33.0**, commit
`4ba787ec9959e566d3b587a796dff07673324da5`, on
`codex/request-energy-v0.33.0`. The unmodified release is retained as tag
`v0.33.0` and branch `codex/baseline-v0.33.0`.
Tested runtime: `5c61df3a2ec5d89c213383e8718e6ec59f139bff`;
subsequent receipt and install-documentation changes do not alter runtime code.
Exact provenance and results are in the
[release-port receipt](request_energy_validation/2026-11-10.json).

There were no textual conflicts. `git range-diff` matched all four original
feature commits without changes. The release's deferred HiSparse KV-load logic
and removal of the old locality-resource initialization are preserved. The
energy snapshot still precedes optimistic token-counter updates; worker CUDA
brackets still wrap forward and sampling. An additional compatibility fix only
completes lifecycle holds for outputs carrying a trace payload. This also preserves
the upstream abort-path tests that intentionally bypass scheduler initialization. Connector/offloading configurations
remain outside the trusted ownership profile.

Native checks used the official compiled **vLLM 0.33.0** wheel with Torch
**2.14.0+cu130**, SDK **0.2.1**, and only the four instrumented Python files.
The original scheduler/output/worker file hashes were checked against the release
before overlaying. The enclosing container image is an environment provider;
its v0.31.0 image tag is not the installed engine version.

On the free A100 80GB, Qwen3-0.6B ran with the release-default V2 runner,
bfloat16, eager execution, TP=1, chunked prefill and prefix caching. All eight
sequential greedy output hashes and token counts matched the unmodified release.
The concurrent energy smoke finalized eight explicitly identified requests with
positive GPU estimates, passed software integrity and conservation, and reproduced
request results and the interval ledger byte-for-byte through replay.
The upstream scheduler and async-scheduler suites are tested with
`VLLM_TARGET_DEVICE=cpu`; the receipt records both baseline and instrumented counts.
The parent project's energy and meeting suites and this fork's applicable
pre-commit hooks were also checked.

This is a bounded software-conformance result. The RTX 4090 remained occupied
by its existing SGLang service; it was not interrupted for this release port.
The public service on the other A100 remained running. Full Qwen3-8B campaign,
TP=2, V1 runner, CUDA graphs, cancellation/preemption, physical accuracy and
overhead validation remain separate gates. `hardware_validated` and
`research_ready` remain false. Earlier results below retain their original
bases and do not certify this release.

## Historical review corrections: 8 November 2026

Tested runtime: `e2781a8ab8e416f6c4c2a8804a9706354f598d63`. Exact artifact, image, model and
result hashes are in the [8 November receipt](request_energy_validation/2026-11-08.json). Source and wheel
**llm-request-energy 0.2.1** are published at
[inference-energy-attributor v0.2.1](https://github.com/hm-green-it-lab/inference-energy-attributor/tree/v0.2.1).
Wheel SHA-256: `ee646d6dc2dd6ec52577acfdcb20ddf3da5b41e641e656b667c0bfb75b16f96d`.
The historical v0.2.0 tag remains available for the original prototype.

| Hardware | Final requests | GPU estimate range | CUDA execution intervals |
| --- | ---: | ---: | ---: |
| RTX 4090 | 8/8 | 8.73–17.65 J | 184 |
| A100 80GB | 8/8 | 17.81–27.59 J | 182 |

All energy cases reconciled eight requests, produced positive GPU estimates
and passed conservation. Request results and interval ledgers reproduced
byte-for-byte through replay; equal-share and activity-weighted replays also
validated. The largest residual across both engines and hosts was
`1.5987211554602254e-14 J`, within the required tolerance. Each ordinary
trace-on/off comparison matched all eight response hashes and token counts
on both hosts. This remains a functional observation under uncontrolled
concurrent arrival order, not an arbitrary-schedule invariance proof.

Only the four instrumentation Python files were overlaid onto the exact
`0.33.0rc1.dev87+g2fb5111bb` compiled engine wheel, matching base
`2fb5111bbac7a2e31c0c19c1eb0aa01df5ecb664`. The enclosing image supplies the
runtime environment; its tag is not the installed vLLM version. This update
adds an actionable missing-SDK error and install documentation, without
changing model kernels. The pre-commit suite passed.

The combined local suite passed 88 tests. Each server passed 74 CPU checks
(50 SDK and 24 SGLang adapter tests); a clean Python 3.12 wheel-only environment
also passed all 50 SDK tests and synthetic replay/validation. Deployed source
bytes match the tested commits; 5,275 staged file checksums were checked per host.

The RTX's original container was restored with identical ID, image and ports,
and `/health_generate` was ready. All 15 pre-existing RTX containers were
retained. The public service on the second A100 remained running with its
original start timestamp. Drivers were unchanged in this pass.

Two warmups do not warm every JIT shape. The larger A100 overlap estimates are
retained and include residency-based baseline allocations; these ranges are
not engine/hardware rankings. RTX has NVML and genuine RAPL package coverage,
with no DRAM domain; A100 has NVML only. CPU cgroup ratios were not sampled.
100 ms polling does not remove the RTX sensor's one-second averaging uncertainty.
The observed modeled result delays were below 0.24 s, without establishing
physical temporal accuracy. Shared-host interference is possible on the A100.

Physical reference accuracy, paired overhead, native cancellation/preemption
and failure injection, CUDA graphs, alternate vLLM runners and TP>1 remain open.
`hardware_validated` and `research_ready` remain false. Earlier unfavorable
observations are preserved below and in the earlier receipt.

## Historical validation: 2 November 2026

Validation date: 2 November 2026 (lab UTC logs extend into 3 November).
These are TP=1 software conformance checks, not accuracy or overhead results.
The tested runtime revision is `c5adbf712dc32d5a2ecc7134265b933033a3e5fe`.
Subsequent receipt/documentation-only commits retain those runtime bytes.
Exact source, model-file, SDK, image-config and result hashes are in the
[machine-readable receipt](request_energy_validation/2026-11-02.json). Private addresses and credentials are excluded.

### Runtime and procedure

Both engines use llm-request-energy **0.2.0**, built in the independent attribution project.
The SDK is a separate development wheel, not assumed published on PyPI.
Its SHA-256 is `792954f12ccf5d8957b057552cacbd8a25ddbbeb297377e0f8fcec6067628ac6`.
Tracing disabled requires no SDK imports from these instrumentation paths.

Each full case used eight measured requests, four client threads, two short warmups,
15 seconds of loaded-engine idle calibration, 100 ms polling and 10-second pre/post windows.
The profile was eager dense text, context 4096, chunk size 128, fixed greedy output lengths,
and the computed-token allocation policy with the declared residency floor.
RTX used Qwen3-0.6B-Base with pinned file hashes; A100 used Qwen3-0.6B revision
`c1899de289a04d12100db370d81485cdf75e47ca`.
SGLang used Triton attention and PyTorch sampling.

| Hardware | Mode | Final requests | GPU estimate range | Native steps / executions |
| --- | --- | ---: | ---: | ---: |
| RTX 4090 | ordinary | 8/8 | 7.84–17.66 J | N/A |
| A100 80GB | ordinary | 8/8 | 17.54–25.91 J | N/A |

All cases reconciled the measured request inventory, returned positive GPU estimates,
and passed software integrity and conservation. Live/replay request output and ledgers
were byte-identical. Equal-share and activity-weighted offline replays also passed.
The largest absolute ledger residual in these vllm cases was
`1.59872e-14 J`, below
`max(1e-6 J, 1e-9 * reconstructed domain energy)`.
The estimate ranges include baseline-floor allocations; they are not hardware/engine rankings.

### Compatibility and output observations

The base is main revision `2fb5111bbac7a2e31c0c19c1eb0aa01df5ecb664`, not a release tag.
Its matching official compiled wheel is `0.33.0rc1.dev87+g2fb5111bb`.
Only four instrumentation Python modules were overlaid onto that matching runtime.
The full pre-commit hook suite, including Ruff, mypy and documentation checks, passed.

The independent attribution project's combined suite passed 64 tests, including these
adapter fixtures, immutable observations, asynchronous writer health and protocol-family rejection.

The RTX concurrent trace-on/off comparison matched all eight response hashes and token counts.
The A100 concurrent comparison differed in **one of eight response hashes**, with equal token counts.
A separate sequential trace-on/off comparison (one client, 200 ms request gap) matched all eight.
The concurrent difference is retained; its cause is unresolved. Arbitrary-schedule output invariance is not established.

Concurrent fixed greedy serving is not generally guaranteed bitwise reproducible;
these comparisons are observations under the recorded profiles, not universal equivalence proofs.

### Measurement limits

RTX driver 580.196.07 and A100 driver 580.178.04 remained unchanged in this merge pass.
RTX collected NVML and genuine RAPL package energy; no DRAM domain was available.
A100 collected NVML only. The other A100 retained its public service, so shared-host
interference remains possible. Two short warmups do not ensure every JIT shape is warm;
the larger A100 overlap observations are retained.

100 ms polling does not establish 100 ms sensor resolution. RTX power observations
retain the declared one-second averaging support. CPU cgroup ratios were not sampled;
CPU/DRAM model accuracy remains unvalidated. Conservation establishes accounting
consistency, not physical per-request ground truth.

Native cancellation/preemption, CUDA graphs, alternate vLLM runner coverage and TP=2
remain open. The occupied A100 pair was not used for TP=2. Reference accuracy, sensor
characterization and paired overhead still require the planned research campaign.
`hardware_validated` and `research_ready` remain false.
