# Gateway API Inference Extension (`kubernetes-sigs/gateway-api-inference-extension`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/gateway-api-inference-extension
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 672 commits across **v1.3.1**, **v1.4.0**, and **v1.5.0** (`v1.5.0-rc.2`).

---

## 1. Strategic Refocusing in 2026: API Standardization vs. Production EPP (`llm-d`)

The most important architectural milestone for `gateway-api-inference-extension` (GIE) in 2026 was separating **Kubernetes API standardization & conformance** from **heavyweight production inference scheduler development**:
- In mid-2026, the full-featured production Endpoint Picker (EPP) and Body-Based Router (BBR) were migrated to the **[`llm-d`](https://github.com/llm-d)** open-source project (`llm-d-inference-scheduler`), while `gateway-api-inference-extension` retained the **CRDs (`InferencePool`, `InferenceObjective`, `InferenceModelRewrite`), GEP specifications, conformance tests, and a new Lightweight EPP (`lwepp`) reference implementation**:
  - **`5fc2e84` / `d864526`** (2026-03-12, PR [#2430](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2430)) — GEP: Migrate the production EPP implementation to `llm-d` and establish a minimal reference implementation in GIE.
  - **`98b0e55` / `53d64b9`** (2026-07-08, PR [#2842](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2842)) — Introduce Lightweight EPP (`lwepp`), a clean, self-contained reference `ext_proc` endpoint picker for API validation and conformance testing.
  - **`e9d1c7c` / `0b703eb`** (2026-09-02, PRs [#2959](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2959), [#2967](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2967), [#2973](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2973)) — Complete removal of legacy EPP and BBR packages following migration to `llm-d` and cut **v1.5.0-rc.1 / rc.2** (Oct 2026).

---

## 2. API Evolution & Inference Routing Features (v1.3.1 → v1.5.0)

- **`InferencePool` `v1` Adoption & `v1alpha2` Deprecation**:
  - **`518ffab` / `d675c10`** (2026-08-19, PR [#2937](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2937)) — Formally mark `v1alpha2` `InferencePool` as deprecated in favor of `inference.networking.k8s.io/v1` `InferencePool`.
- **Prefix Cache, KV-Cache & Disaggregated Prefill/Decode Scheduling (v1.4.0)**:
  - **`75d9de2` / `ff32a27`** (Jan–May 2026, PRs [#2215](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2215), [#2512](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2512), [#2640](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2640)) — Prefix-cache-aware scoring, KV-cache utilization balancing, and Prefill/Decode (P/D) disaggregation lifecycle hooks.
  - **`e1df436` / `b3446ca`** (Mar–Jun 2026, PRs [#2488](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2488), [#2710](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2710)) — LoRA adapter target routing, `InferenceModelRewrite`, and fairness/priority queueing across `InferenceObjective` tiers.
- **Scale & Latency Benchmarks**:
  - Added synthetic model server benchmarking and high-QPS `ext_proc` streaming optimizations prior to the `llm-d` transition.
