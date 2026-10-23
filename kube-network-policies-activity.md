# Kube Network Policies (`kubernetes-sigs/kube-network-policies`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/kube-network-policies
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 86 commits (42 first-parent PR merges) across **v1.0.0**, **v1.1.0**, **v1.1.1**, and **v1.1.2**.

---

## 1. Release Timeline in 2026

| Release | Date | Tag Commit | Highlights |
| :--- | :--- | :--- | :--- |
| **v1.0.0** | 2026-02-12 | `8db8b26` | Stable 1.0 milestone for standard Kubernetes `NetworkPolicy` enforcement via `nfqueue` + `nftables` |
| **v1.1.0** | 2026-06-11 | `b00a988` | Full `ClusterNetworkPolicy` (`v1alpha2`) implementation, Pod-to-Node egress policies, Node IP matching |
| **v1.1.1** | 2026-07-24 | `66d6fb3` | Skip `hostNetwork` Pods during policy evaluation; dependency and Go runtime updates |
| **v1.1.2** | 2026-09-23 | `805411c` | Preserve `nftables` base chains across syncs to avoid dropping in-flight packets or resetting counters |

---

## 2. Significant Features & Fixes in 2026

### `ClusterNetworkPolicy` (`v1alpha2`) Reference Implementation
`kube-network-policies` serves as the reference CNI-agnostic implementation for SIG-NETWORK's `network-policy-api` subproject:
- **`1f98a2d` / `b0cf803`** (2026-03-04, PR [#303](https://github.com/kubernetes-sigs/kube-network-policies/pull/303), Hunter) — Migrate from `AdminNetworkPolicy`/`BaselineAdminNetworkPolicy` (`v1alpha1`) to the unified `ClusterNetworkPolicy` (`v1alpha2`) API with `Admin` and `Baseline` priority tiers.
- **`08adb54` / `28f3961`** (2026-04-30, PR [#339](https://github.com/kubernetes-sigs/kube-network-policies/pull/339), Hunter) — Implement Pod-to-Node egress policy evaluation (`nodes` peer selector in `ClusterNetworkPolicy`).
- **`fa127d2` / `fb570d6`** (2026-05-05, PR [#340](https://github.com/kubernetes-sigs/kube-network-policies/pull/340), Hunter) — Fix Node IP matching logic when evaluating egress traffic to cluster nodes.

### Dataplane Reliability (`nftables` & `hostNetwork`)
- **`ca195e2` / `0bd061f`** (2026-07-24, PR [#389](https://github.com/kubernetes-sigs/kube-network-policies/pull/389), Antonio Ojea) — Skip `hostNetwork: true` Pods during Pod IP policy matching so host-networked daemons are not inadvertently isolated by Pod selectors.
- **`ecd40e0` / `805411c`** (2026-09-23, PR [#403](https://github.com/kubernetes-sigs/kube-network-policies/pull/403), Casey Davenport) — Avoid deleting and recreating existing `nftables` chains during ruleset synchronization, preserving `knftables` flowtable/chain state and preventing transient packet drops.
- **`a7fdaf0` / `bdcdde0`** (2026-06-05, PR [#365](https://github.com/kubernetes-sigs/kube-network-policies/pull/365), Antonio Ojea) — Bump `network-policy-api` to `v0.2.0` and align conformance test jobs with the upstream `v1alpha2` suite.
