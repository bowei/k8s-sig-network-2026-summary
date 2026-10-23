# Kube Agentic Networking (`kubernetes-sigs/kube-agentic-networking`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/kube-agentic-networking
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 165 commits (82 first-parent PR merges) across `release-0.1` (`v0.1.0`) and `release-0.2` (`v1alpha1`).

---

## 1. Overview & 2026 Milestones

`kube-agentic-networking` defines Kubernetes-native networking, identity, and authorization primitives for AI agents and Model Context Protocol (MCP) tool servers, building on Gateway API and Envoy/Agentgateway data planes.
- **April 23, 2026**: Presented the **`v1alpha1` (`v0.2.0`) API Review** to SIG-NETWORK (`agentic.networking.x-k8s.io/v1alpha1`).
- **September 17, 2026**: Presented **Kube Agentic Networking Update** to SIG-NETWORK covering MCP tool authorization, dynamic CEL expressions, and external authorization integration.

---

## 2. API Evolution (`v0.1` → `v1alpha1` / `v0.2`)

- **API Group Migration & `v1alpha1` Promotion**:
  - **`ee9f7b0` / `2e3c3df`** (2026-04-21, PR [#158](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/158), Maolin) — Move API group to `agentic.networking.x-k8s.io` as part of SIG-NETWORK API review readiness.
  - **`b3f4a19` / `a5d54fd`** (2026-05-06, PR [#165](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/165), Maolin) — Promote API version from `v0alpha0` to `v1alpha1` (`XAccessPolicy`, `XBackend`).
  - **`49663c7` / `85ca6f2`** (2026-02-02, PR [#111](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/111), Maolin) — Align `XAccessPolicy` and `XBackend` resource definitions with Gateway API policy attachment conventions.

---

## 3. MCP & Agentic Authorization Capabilities

- **MCP `tools/call` Method, Tool & Parameter Authorization**:
  - **`72ec3aa` / `0b807cc`** (2026-07-09, PR [#180](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/180), Fazal) — Support fine-grained MCP method, tool name, and parameter-level authorization rules (`tools/call`) in `XAccessPolicy`.
- **Dynamic Authorization with CEL Expressions**:
  - **`85602fc` / `66248e5`** (2026-07-23, PR [#188](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/188), Fazal) — Add Common Expression Language (CEL) rule evaluation to `XAccessPolicy` for inspecting agent identity claims and MCP request payloads.
- **External Authorization (`ExternalAuth`) & L7 Parameters**:
  - **`3c2330a` / `c1f3502`** (2026-07-29, PR [#191](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/191), Fazal) — Add `ExternalAuth` provider integration in `XAccessPolicy`.
  - **`be5e39e` / `2420da6`** (2026-08-03, PR [#192](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/192), Fazal) — Support L7 HTTP/MCP parameter matching in `XAccessPolicy`.
- **Distributed Tracing for Authorization Decisions**:
  - **`8c5edef` / `1f98a2d`** (2026-03-04, PR [#122](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/122), Maolin) — Emit OpenTelemetry tracing spans for agent authorization policy decisions in the Envoy/ext_proc data plane.
- **Conformance Suite & E2E Testing**:
  - **`ac43b9f` / `dc161dc`** (2026-02-23, PR [#118](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/118), Maolin) and **`bb1e5e3` / `44eb7bb`** (2026-03-19, PR [#138](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/138), Maolin) — Build out the conformance test suite for `XAccessPolicy`, `XBackend`, JWT/SPIFFE agent identity, and MCP tool invocation.
