# Kubernetes Enhancements (`kubernetes/enhancements`) — SIG-NETWORK Activity (2026)

- **Repository**: https://github.com/kubernetes/enhancements
- **Period Covered**: January 1, 2026 – October 22, 2026 (v1.36, v1.37, and v1.38 release cycles)
- **Scope**: All commits touching `keps/sig-network/` or SIG-NETWORK-related KEPs since January 1, 2026.

---

## 1. Summary of Active SIG-NETWORK KEPs in 2026

| KEP # | Title | 2026 Milestone / Status | Key PRs & Commits |
| :--- | :--- | :--- | :--- |
| **KEP-4762** | Allow arbitrary FQDN as Pod hostname (`hostnameOverride`) | **GA in v1.37** | [#6108](https://github.com/kubernetes/enhancements/pull/6108) (`38122f8`) |
| **KEP-4858** | IP and CIDR Validation Improvements (`StrictIPCIDRValidation`) | **Beta in v1.36 → GA in v1.38** | [#5831](https://github.com/kubernetes/enhancements/pull/5831) (`0333c7d`), [#6337](https://github.com/kubernetes/enhancements/pull/6337) (`5286fd5`) |
| **KEP-4974** | Deprecate `v1.Endpoints` | **Ongoing Deprecation (v1.37)** | [#6193](https://github.com/kubernetes/enhancements/pull/6193) (`12f2943`) |
| **KEP-5311** | Relaxed Validation for Service Names (`RelaxedServiceNameValidation`) | **Beta in v1.36 → GA in v1.37** | [#5867](https://github.com/kubernetes/enhancements/pull/5867) (`9c334af`), [#6110](https://github.com/kubernetes/enhancements/pull/6110) (`7a31824`) |
| **KEP-5343** | Make `nftables` the Default `kube-proxy` Mode | **Plan Merged (Target v1.37+)** | [#5344](https://github.com/kubernetes/enhancements/pull/5344) (`170c8f1`) |
| **KEP-5495** | Deprecate `ipvs` Mode in `kube-proxy` | **Updated Deprecation Plan (v1.37)** | [#6120](https://github.com/kubernetes/enhancements/pull/6120) (`96d63b1`) |
| **KEP-5707** | Deprecate `Service.spec.externalIPs` | **Deprecated in v1.36** | [#5708](https://github.com/kubernetes/enhancements/pull/5708) (`59980cb`) |
| **KEP-6032** | Support Localhost NodePorts in `kube-proxy` `nftables` Mode | **Alpha in v1.37 → Beta in v1.38** | [#6042](https://github.com/kubernetes/enhancements/pull/6042) (`72e1af9`), [#6378](https://github.com/kubernetes/enhancements/pull/6378) (`08f4557`) |
| **KEP-6313** | Network-Isolated (Hermetic) Pods | **Alpha in v1.38** | [#6317](https://github.com/kubernetes/enhancements/pull/6317) (`cf96671`) |

---

## 2. Detailed KEP Activity by Theme

### `kube-proxy` Modernization: `nftables` Default & `ipvs` Deprecation
- **KEP-5343 — Make `nftables` the default `kube-proxy` mode**:
  - **`170c8f1` / `cb814dd`** (2026-04-29, PR [#5344](https://github.com/kubernetes/enhancements/pull/5344), Dan Winship) — Defines the transition plan for switching Linux `kube-proxy`'s default mode from `iptables` to `nftables` starting in v1.37, once all blocking parity gaps (notably KEP-6032 localhost NodePorts) are resolved.
- **KEP-6032 — `nftables` Localhost NodePort Userspace Proxy**:
  - **`72e1af9` / `b0760c3`** (2026-04-29, PR [#6042](https://github.com/kubernetes/enhancements/pull/6042), Dan Winship) — Introduces KEP-6032 for v1.37 Alpha, adding a fallback userspace proxy in `kube-proxy`'s `nftables` mode so connections to `127.0.0.1:<nodePort>` continue to work when `--nodeport-addresses` includes `127.0.0.0/8` (or is unset), while also adding `"localhost"` and `"all"` keywords to `--nodeport-addresses`.
  - **`08f4557` / `ac90769`** (2026-10-08, PR [#6378](https://github.com/kubernetes/enhancements/pull/6378), Dan Winship) — Updates KEP-6032 for Beta graduation in v1.38 (`NFTablesLocalhostNodePorts` enabled by default).
- **KEP-5495 — Deprecate `ipvs` mode in `kube-proxy`**:
  - **`96d63b1` / `5457f97`** (2026-05-14, PR [#6120](https://github.com/kubernetes/enhancements/pull/6120), Dan Winship) — Revises the IPVS deprecation plan: because no maintainers stepped forward to maintain an out-of-tree IPVS proxy in `kubernetes-sigs`, SIG-NETWORK updated KEP-5495 to formally deprecate and remove IPVS in-tree behind a feature gate, steering users to `nftables` or eBPF/CNI service proxies.

### Pod Network Isolation (Hermetic Pods)
- **KEP-6313 — Network-Isolated Pods**:
  - **`cf96671` / `284a914`** (2026-10-12, PR [#6317](https://github.com/kubernetes/enhancements/pull/6317), Jeremy Rickard) — Introduces a standard Kubernetes Pod spec mechanism for network-isolated ("hermetic") Pods that get an isolated loopback-only network namespace without invoking the default cluster CNI plugin. Key use cases include hermetic build steps (Tekton/CI supply chain security), offline batch/AI workloads, and Pods whose networking is exclusively provided via Dynamic Resource Allocation (DRA / DRANET) rather than primary CNI.

### Service, DNS & Validation Graduations
- **KEP-4858 — IP and CIDR Validation Improvements**:
  - **`0333c7d` / `9fb39f4`** (2026-01-30, PR [#5831](https://github.com/kubernetes/enhancements/pull/5831), Dan Winship) — Promotes `StrictIPCIDRValidation` to Beta in v1.36.
  - **`5286fd5` / `60f500c`** (2026-09-28, PR [#6337](https://github.com/kubernetes/enhancements/pull/6337), Dan Winship) — Promotes `StrictIPCIDRValidation` to GA in v1.38.
- **KEP-5311 — Relaxed Validation for Service Names**:
  - **`9c334af` / `629fc14`** (2026-02-04, PR [#5867](https://github.com/kubernetes/enhancements/pull/5867), Tim Hockin) — Updates KEP-5311 for Beta in v1.36.
  - **`7a31824` / `9e812fa`** (2026-05-07, PR [#6110](https://github.com/kubernetes/enhancements/pull/6110), Tim Hockin) — Promotes `RelaxedServiceNameValidation` to GA in v1.37.
- **KEP-4762 — Allow Arbitrary FQDN as Pod Hostname**:
  - **`38122f8` / `8b39a8e`** (2026-05-08, PR [#6108](https://github.com/kubernetes/enhancements/pull/6108), Niladri) — Promotes `PodHostNameFQDN` (`hostnameOverride`) to GA in v1.37.
- **KEP-5707 — Deprecate `Service.spec.externalIPs`**:
  - **`59980cb` / `726b75e`** (2026-02-05, PR [#5708](https://github.com/kubernetes/enhancements/pull/5708), Adrian Moisey) — Formally deprecates `Service.spec.externalIPs` in v1.36 due to security footguns (arbitrary IP hijacking without admission controls) and overlapping functionality with `LoadBalancer` IPs and Gateway API.
- **KEP-4974 — Deprecate `v1.Endpoints`**:
  - **`12f2943` / `5a87e42`** (2026-06-12, PR [#6193](https://github.com/kubernetes/enhancements/pull/6193), Rob Scott) — Updates KEP-4974 implementation history for v1.37, tracking the migration of `kube-apiserver` `service/proxy` and internal consumers from `Endpoints` to `EndpointSlice`.

### Other Related Cross-SIG KEPs Discussed with SIG-NETWORK in 2026
- **KEP-5939 — Multi-Cluster Service API Cluster Selection** (presented in SIG-NETWORK on Oct 8, 2026): Proposes cluster targeting and weighted traffic routing across clusters for MCS-API (`ClusterId` on `EndpointSlice`, `TargetClusters` on `ServiceImport`).
- **KEP-6116 — `EndpointSelector`** (discussed May–Oct 2026): Separates endpoint selection from the monolithic `Service` object so Gateway API (`XBackend`) and future Service primitives can select arbitrary workload endpoints.
- **KEP-6312 — Relaxed `HostAliases` FQDN Trailing Dot Validation** (implemented in k/k [#140654](https://github.com/kubernetes/kubernetes/pull/140654)).
