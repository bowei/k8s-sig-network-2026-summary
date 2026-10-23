# Multi-Cluster Services API (`kubernetes-sigs/mcs-api`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/mcs-api
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 34 commits (17 first-parent PR merges) plus **KEP-5939** design reviews with SIG-NETWORK and SIG-Multicluster.

---

## 1. Conformance Suite & API Enhancements in 2026

- **Headless Service & `TrafficDistribution` Conformance Tests**:
  - **`a5fc3ab` / `c9a1bf7`** (2026-07-02, PR [#134](https://github.com/kubernetes-sigs/mcs-api/pull/134), Gaurav) — Add conformance tests for `ServiceExport` / `ServiceImport` `trafficDistribution` field propagation across clusters.
  - **`82c0233` / `46c23cd`** (2026-05-14, PR [#129](https://github.com/kubernetes-sigs/mcs-api/pull/129), Etienne Coutaud) — Expand headless multi-cluster Service DNS and EndpointSlice conformance tests.
  - **`d538175` / `b237efd`** (2026-03-19, PR [#124](https://github.com/kubernetes-sigs/mcs-api/pull/124), Etienne Coutaud) — Add conformance report generation and structured test filtering.
- **CRD & Tooling Modernization**:
  - **`8339b0d` / `64b7b28`** (2026-08-21, PR [#138](https://github.com/kubernetes-sigs/mcs-api/pull/138)) — Update Kubernetes client dependencies (`v0.36+`) and controller-gen CRD schemas.

---

## 2. KEP-5939: Multi-Cluster Service API Cluster Selection (Reviewed Oct 8, 2026)

On October 8, 2026, SIG-Multicluster and SIG-NETWORK reviewed **KEP-5939 (Multi-Cluster Service API Cluster Selection)**:
- **Problem Addressed**: Today, `ServiceImport` aggregates endpoints from all exporting clusters in a ClusterSet without a standard way to select specific target clusters or control traffic weights per cluster (needed for canary rollouts across clusters, failover, and regional locality).
- **Proposed API Additions**:
  - Standardize `multicluster.kubernetes.io/source-cluster` label (or `ClusterId` field) on imported `EndpointSlice` objects.
  - Add `TargetClusters` (`ClusterSelection` with optional `Weight`) on `ServiceImport` / `ServiceExport` or Gateway API `XBackend` to enable selective and weighted multi-cluster traffic routing.
- **SIG-NETWORK Feedback**: Aligned KEP-5939 with the broader Service decomposition (`XBackend` + `EndpointSelector`) effort so multi-cluster traffic splitting composes cleanly with Gateway API and `kube-proxy`.
