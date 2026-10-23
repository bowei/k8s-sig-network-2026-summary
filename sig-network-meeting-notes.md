# Kubernetes SIG-NETWORK Meeting Notes Summary (2026)

- **Source Document**: [Kubernetes Network SIG Meeting Notes](https://docs.google.com/document/d/1bEA0F8WddLMnQp7zUfr1rZOwRqX6g2y4kcylEcGATHo/edit)
- **Period Covered**: January 1, 2026 – October 22, 2026 (21 biweekly meeting slots)
- **SIG Leadership in 2026**:
  - **Chairs**: Bowei Du (`@bowei`), Daman Arora (`@daman`), Guilherme Cassolato (`@guicassolato`, joined April 2026 as Shane Utt stepped down)
  - **Tech Leads**: Tim Hockin (`@thockin`), Dan Winship (`@danwinship`), Antonio Ojea (`@aojea`), Rob Scott (`@robscott`)

---

## 1. Executive Summary of Recurring 2026 Discussion Threads

Across the 2026 SIG-NETWORK meetings, discussions centered on eight primary themes:

1. **Gateway API Rapid Maturation (v1.5, v1.6, v1.7)**:
   - Overhauled the release process (decoupling feature pipeline from release cadence + monthly experimental releases).
   - Reviewed and approved **v1.5** (`TLSRoute`, `ListenerSet`, `HTTPCORSFilter`, Frontend Client Cert Validation, Backend Client Cert Selection, `ReferenceGrant` to `v1`), **v1.6** (`TCPRoute` and `UDPRoute` to Standard `v1`; `XBackend` to Experimental), and **v1.7** (`HTTPRoute` Retries and Route Rule Name Uniqueness to Standard; 6 new Experimental features: Gateway Address Routability, OTel Attribute Dictionary, `XTelemetryPolicy`, Pre-Routing Listener Filters, `ClusterTrustBundle`, and `XBackend` Session Persistence + `EndpointSelector`).
2. **Decomposing `Service` (`XBackend` + `EndpointSelector`)**:
   - Initiated by John Howard, Tim Hockin, and Rob Scott (May 7, 2026) and developed through Gateway API v1.6/v1.7 and KEP-6116. Splits the monolithic `v1.Service` concept into modular primitives: `EndpointSelector` (selecting Pods/workloads), `XBackend` (representing traffic targets, session persistence, TLS, and external FQDNs), and frontend VIPs/Routes.
3. **`kube-proxy` Modernization (`nftables` Default, Localhost NodePorts & IPVS Deprecation)**:
   - Resolved the final blocker for making `nftables` the default `kube-proxy` backend (KEP-5343) via **KEP-6032** (`nftables` Localhost NodePort userspace proxy — Alpha in v1.37, Beta in v1.38).
   - Revised **KEP-5495** to formally deprecate and remove in-tree `ipvs` mode after confirming no maintainers stepped forward for an out-of-tree IPVS repository.
4. **2026 SIG-NETWORK Subproject Status Survey & Governance**:
   - Led by new Co-Chair Guilherme Cassolato (`@guicassolato`) from April to October 2026, surveying all **22 SIG-NETWORK subprojects** to assess health, maintainer bus factor, release cadence, and archival/consolidation candidates.
5. **AI, High-Performance & Agentic Networking (`DRANET`, `GIE`, `kube-agentic-networking`, Multi-Network)**:
   - Reviewed `kube-agentic-networking` `v1alpha1` (`XAccessPolicy` with MCP tool authorization & CEL), `gateway-api-inference-extension` (migrating the heavyweight EPP to `llm-d` while keeping CRDs, conformance, and `lwepp` in-repo), `DRANET` (v1.1–v1.5 with dynamic IPVLAN/Whereabouts IPAM, VRF, RDMA/InfiniBand), and revised Multi-Network proposals (`NetworkClass` / `NetworkKind` / `PodNetwork`).
6. **`ingress-nginx` Retirement & `ingress2gateway` 1.0+**:
   - Coordinated the March 2026 archival of `kubernetes/ingress-nginx` alongside the release of `ingress2gateway` **v1.0, v1.1, and v1.2** with new Emitters for Envoy Gateway, kgateway, agentgateway, Traefik, and Airlock Microgateway.
7. **Network Policy (`ClusterNetworkPolicy` `v0.2.0` → `v1beta1`) & Hermetic Pods (KEP-6313)**:
   - Shipped `ClusterNetworkPolicy` `v1alpha2` (`v0.2.0`), refined FQDN egress rules (NPEP-133) and `v1beta1` graduation criteria, and reviewed **KEP-6313 (Network-Isolated / Hermetic Pods)** for Pods that opt out of default CNI attachments.
8. **Legacy Cleanup (`kube-dns`, `v1.Endpoints`, `Service.spec.externalIPs`)**:
   - Agreed to deprecate `kubernetes/dns` (`kube-dns`) while extracting `node-local-dns` into a standalone module; deprecated `Service.spec.externalIPs` (KEP-5707); continued `v1.Endpoints` deprecation (KEP-4974); and graduated `StrictIPCIDRValidation` (KEP-4858), `RelaxedServiceNameValidation` (KEP-5311), and `PodHostNameFQDN` (KEP-4762) to GA.

---

## 2. Chronological Meeting-by-Meeting Summary (2026)

### January 15, 2026
- **Bug Scrub**: Triaged open `sig/network` bugs in `kubernetes/kubernetes`.
- **Dual-Stack `externalTrafficPolicy: Local` Behavior (#135790)** (*Dan Winship*):
  - Discussed an edge case where a dual-stack Service with `externalTrafficPolicy: Local` has local endpoints for one IP family (e.g., IPv4) but no local endpoints for the other IP family (e.g., IPv6), or where a Pod fails to get a dual-stack IP.
  - Agreed that `kube-proxy` already handles per-family local endpoints properly, and documented expected cloud load-balancer healthcheck behavior on the issue.
- **Deprecating `kube-dns` (`kubernetes/dns`)** (*Bowei Du*):
  - Discussed formally deprecating `kube-dns` now that CoreDNS has been the default cluster DNS since Kubernetes 1.13.
  - Noted that `node-local-dns` still imported packages from `kubernetes/dns` and that some cloud providers (such as GKE) still used `kube-dns` in existing clusters.
  - **Outcome**: Consensus to proceed with deprecating `kube-dns` and decoupling `node-local-dns` into its own standalone Go module (`sigs.k8s.io/node-local-dns`, completed in PR #118 / release `1.27.0`).
- **New `PodCondition`: `AllContainersStarted` (#136213)** (*startingcontainer*):
  - Reviewed a SIG-Node proposal to add a Pod condition indicating all containers have started at least once. SIG-NETWORK members provided feedback on how readiness vs. startup interacts with `EndpointSlice` population and redirected the authors to SIG-Node / SIG-Apps for primary ownership.
- **SIG-NETWORK Charter Update** (*Shane Utt*):
  - Reviewed minor updates to the SIG-NETWORK charter and governance documentation.

---

### January 29, 2026
- **Triage & KEPs Approaching v1.36 Enhancements Freeze (Feb 5, 2026)**:
  - **[KEP-5707: Deprecate `Service.spec.externalIPs`](https://github.com/kubernetes/enhancements/pull/5708)** (*Adrian Moisey*): Approved for v1.36 deprecation; adds API warnings and documentation steering users toward `LoadBalancer` services or Gateway API.
  - **[KEP-4858: IP and CIDR Validation Improvements](https://github.com/kubernetes/enhancements/pull/5831)** (*Dan Winship*): Approved for Beta promotion in v1.36 (`StrictIPCIDRValidation`).
  - **[KEP-5311: Relaxed Validation for Service Names](https://github.com/kubernetes/enhancements/pull/5867)** (*Tim Hockin*): Approved for Beta promotion in v1.36 (`RelaxedServiceNameValidation`).
- **Gateway API Release Process Overhaul** (*Shane Utt*):
  - Presented the new Gateway API release model:
    - Decoupled feature GEP reviews from fixed release dates (release-train model) to eliminate reviewer crunch.
    - Introduced **monthly experimental releases** (starting with `monthly-2026.01`).
    - Confirmed **Gateway API v1.5** was on track for SIG-NETWORK review in February 2026.

---

### February 12, 2026
- **Bug Triage**:
  - Triaged [#136841](https://github.com/kubernetes/kubernetes/issues/136841) (conntrack cleanup improvements) and [#136675](https://github.com/kubernetes/kubernetes/issues/136675).
- **[Gateway API v1.5 SIG-NETWORK Review](https://docs.google.com/document/d/1sM25obtNIeXpMLGam89e9-ciIcuTe-W_v5lGuZYQ_Y8/edit)** (*Rob Scott, Shane Utt*):
  - Presented the API diff and conformance status for **Gateway API v1.5**:
    - **Graduating to Standard (`v1`)**:
      1. `TLSRoute` (GEP-2643)
      2. `ListenerSet` (GEP-1713, promoted from `XListenerSet`)
      3. `HTTPCORSFilter` / CORS support in `HTTPRoute` (GEP-1767)
      4. Gateway Frontend Client Certificate Validation (mTLS, GEP-91)
      5. Gateway Backend Client Certificate Selection in `BackendTLSPolicy` (GEP-3155)
      6. `ReferenceGrant` (`v1`)
    - Also highlighted the new **Interactive Controller Matching Wizard** on the Gateway API website and ongoing Hugo/Docsy documentation migration.
  - **Outcome**: SIG-NETWORK approved the v1.5 API review (released March 11, 2026).

---

### February 26, 2026
- **Bug Scrub**: Routine issue triage.
- **No `kube-proxy` Mode Works Out-of-the-Box on RHEL 8 (#137167)** (*Dan Winship*):
  - Discussed an issue where RHEL 8's older kernel/userspace `nft` version is too old for `kube-proxy`'s `nftables` mode, while newer RHEL/Fedora systems drop legacy `iptables` packages by default.
  - Clarified minimum `nftables` kernel and CLI version requirements in `knftables` and `kube-proxy` documentation.
- **Proposal: Allow Trailing Dot (`.`) in `Pod.spec.hostAliases` Hostnames** (*rikatz / Tim Hockin*):
  - Discussed relaxing validation on `Pod.spec.hostAliases.hostnames` so users or admission controllers supplying fully qualified domain names with a trailing dot (`example.com.`) are accepted (matching KEP-4427 / KEP-6312).
  - **Outcome**: Agreed to pursue a small KEP (KEP-6312) and implementation PR ([#140654](https://github.com/kubernetes/kubernetes/pull/140654)).

---

### March 12, 2026
- **Bug Triage**:
  - Discussed [#137345](https://github.com/kubernetes/kubernetes/issues/137345) — `kube-proxy` `nftables` localhost NodePort behavior (`127.0.0.1:<nodePort>`), which directly motivated **KEP-6032**.
  - Discussed [#137024](https://github.com/kubernetes/kubernetes/issues/137024) — Kubelet filtering of scoped IPv6 link-local DNS resolvers (`fe80::%eth0`) in `/etc/resolv.conf` (later fixed in [#140880](https://github.com/kubernetes/kubernetes/pull/140880)).
- **Subproject Updates for KubeCon EU 2026 (Amsterdam, March 23–26, 2026)**:
  - Collected slides and updates from subproject leads (`gateway-api`, `network-policy-api`, `dranet`, `ingress2gateway`, `kube-agentic-networking`, `gateway-api-inference-extension`) ahead of the KubeCon EU SIG-NETWORK update talk.
  - Highlighted the imminent archival of `ingress-nginx` and the **v1.0.0** release of `ingress2gateway` (shipped March 20, 2026).

---

### March 26, 2026
- **Canceled** due to KubeCon + CloudNativeCon Europe 2026.

---

### April 9, 2026
- **Welcome New SIG-NETWORK Co-Chair**:
  - Welcomed **Guilherme Cassolato (`@guicassolato`, Red Hat / Kuadrant)** as the new SIG-NETWORK Co-Chair ([kubernetes/community#8866](https://github.com/kubernetes/community/pull/8866)), succeeding Shane Utt.
- **Launch of the 2026 SIG-NETWORK Subproject Status Survey** (*Guilherme Cassolato*):
  - Announced a structured survey across all 22 SIG-NETWORK subprojects to gather health metrics, maintainer activity, recent milestones, and SIG support needs.
- **Triage**: Reviewed open issues following KubeCon EU.

---

### April 23, 2026
- **Triage**: Routine issue triage.
- **[Agentic Networking `v1alpha1` (`v0.2.0`) API Review](https://docs.google.com/document/d/1scubdUaDOQzULlyLvwZTK7arttKaGEYsAnFGW5FhgkI/edit)** (*Faseela K, Maolin, Cong*):
  - Reviewed the `kube-agentic-networking` transition from `v0alpha0` (`v0.1.0`) to `v1alpha1` (`v0.2.0`):
    - Moved API group from `agentic.Prototype` to **`agentic.networking.x-k8s.io/v1alpha1`**.
    - Reviewed `XAccessPolicy` and `XBackend` resources for AI agent identity (SPIFFE / JWT) and Model Context Protocol (MCP) tool server access control.
  - **Outcome**: Approved moving to `agentic.networking.x-k8s.io/v1alpha1` and continuing alignment with Gateway API's `XBackend` work.
- **[KEP-6032: Support Localhost NodePorts in `kube-proxy` `nftables` Mode](https://github.com/kubernetes/enhancements/pull/6042)** (*Dan Winship*):
  - Reviewed the design for an opt-in userspace proxy in `kube-proxy`'s `nftables` mode to handle traffic to `127.0.0.1:<nodePort>`, unblocking **KEP-5343** (making `nftables` the default `kube-proxy` mode).
  - Also introduced `"localhost"` and `"all"` keywords to `--nodeport-addresses`.
  - **Outcome**: Approved for v1.37 Alpha.

---

### May 7, 2026
- **Triage**: Routine issue triage.
- **[Towards Decomposing `Service` (`XBackend` + `EndpointSelector`)](https://docs.google.com/presentation/d/1h_2WYyvIbyyCIMN61FInAfFtaJk_TrYpDXKoHXnUfy8/edit)** (*John Howard, Tim Hockin, Rob Scott*):
  - Presented a major architectural vision for evolving Kubernetes `Service` by decomposing its tightly coupled responsibilities into composable resources:
    1. **`EndpointSelector`** (KEP-6116): Selects Pods/workloads and triggers `EndpointSlice` generation independently of a `Service` VIP.
    2. **`XBackend`** (GEP-4894): Defines backend traffic properties (session persistence, TLS, health checks, weights) and target destinations (including `EndpointSelector`, `Service`, or external FQDNs/IPs).
    3. **Frontend (`Service` / `Gateway` / `HTTPRoute`)**: Binds VIPs, ports, or L7 routes to backends.
  - Discussed how this solves long-standing pain points: routing directly to Pod subsets without creating proliferation of dummy headless Services, first-class external FQDN/egress backends in Gateway API, and cleaner multi-cluster traffic splitting.
- **KEP-5495 Update: Deprecating `ipvs` Mode in `kube-proxy` (#6120)** (*Dan Winship*):
  - Previously, KEP-5495 proposed moving the IPVS proxier out-of-tree before removing it from `kubernetes/kubernetes`. However, no contributors volunteered to maintain an out-of-tree IPVS proxy.
  - **Outcome**: Agreed to update KEP-5495 ([#6120](https://github.com/kubernetes/enhancements/pull/6120)) to directly deprecate IPVS in-tree behind a feature gate ([#139397](https://github.com/kubernetes/kubernetes/pull/139397)) and guide users to `nftables` or CNI-native service proxies.

---

### May 21, 2026
- **Triage**: Routine issue triage.
- **2026 Subproject Status Survey Reminder** (*Guilherme Cassolato*):
  - Reminded remaining subproject owners to submit responses before the May 31 deadline.
- **[Gateway API v1.6 SIG-NETWORK Review](https://docs.google.com/document/d/1osAEkEBRs2oZPERAYk-6ambJpSufyI5ZeRRbB6C58KI/edit)** (*Rob Scott, Shane Utt*):
  - Reviewed the **Gateway API v1.6** release contents:
    - **Graduating to Standard (`v1`)**: `TCPRoute` (GEP-2644) and `UDPRoute` (GEP-2645), completing the promotion of core L4 route types alongside `TLSRoute`.
    - **Entering Experimental (`v1alpha1`)**: `XBackend` (GEP-4488 / GEP-4894) for external hostname backends and Service decomposition.
  - **Outcome**: Approved by SIG-NETWORK (released July 6, 2026).

---

### June 4, 2026
- **Triage**:
  - Reviewed open `kubernetes/kubernetes` networking bugs and v1.37 code freeze items (including KEP-6032 implementation [#138427](https://github.com/kubernetes/kubernetes/pull/138427), KEP-5311 GA [#139282](https://github.com/kubernetes/kubernetes/pull/139282), and KEP-4762 GA [#6108](https://github.com/kubernetes/enhancements/pull/6108)).

---

### June 18, 2026
- **Triage**: Routine bug scrub.
- **[SIG-NETWORK Subproject Survey Results & Report](https://docs.google.com/document/d/1yBdIy8AqcMOOIvfzHvYnYbnIbq77XMGCqVNQy1Mh3yg/edit)** (*Guilherme Cassolato*):
  - Presented the comprehensive findings from the **2026 Subproject Status Survey** covering all **22 SIG-NETWORK subprojects**:
    - **Thriving / High-Velocity Subprojects**: `gateway-api`, `dranet`, `external-dns`, `ingress2gateway`, `kube-network-policies`, `network-policy-api`, `gateway-api-inference-extension`, `kube-agentic-networking`, `knftables`, `node-local-dns`, `coredns`.
    - **Maintenance / Stable Mode**: `cni` / `cni-plugins` (external CNCF alignment), `kube-proxy`, `endpointslice`, `dns` (deprecating `kube-dns`), `mcs-api`, `ip-masq-agent`.
    - **Needs Attention / Bus-Factor Risk / Archival Candidates**:
      - `ingress-nginx` (archived March 2026; migration focus on `ingress2gateway`).
      - `ingress-gce` (transitioning maintenance model; high backlog).
      - `blixt` (experimental Rust/eBPF Gateway API implementation — inactive, candidate for retirement/archival).
      - `kpng` (Kubernetes Proxy Next Gen — largely inactive since `nftables` and DRANET became the primary focus; candidate for archival check).
      - `multi-network` / `kube-ovn` / `whereabouts` follow-ups on WG/subproject alignment.
  - Discussed action items: contributor onboarding for single-vendor subprojects, archiving dormant repos, and using the survey data to compile the SIG-NETWORK Annual Report and KubeCon NA 2026 slides.

---

### July 2, 2026
- **Canceled** (US Independence Day holiday week).

---

### July 16, 2026
- **Triage**:
  - Reviewed mid-cycle v1.37/v1.38 networking PRs: `knftables` netlink default in `kube-proxy` ([#137536](https://github.com/kubernetes/kubernetes/pull/137536)), `kube-proxy` memory optimization stripping `managedFields` ([#140056](https://github.com/kubernetes/kubernetes/pull/140056)), and `kube-apiserver` `service/proxy` migration from `Endpoints` to `EndpointSlice` ([#134860](https://github.com/kubernetes/kubernetes/pull/134860)).

---

### July 30, 2026
- **Triage**: Routine bug scrub.
- **Multi-Network API Re-Architecture (`NetworkClass` / `NetworkKind` / `PodNetwork`)** (*Nirav, Etienne, Dougal*):
  - Presented two linked design documents revisiting the Multi-Network API after incorporates lessons from DRA (`DRANET`):
    - [NetworkClass / NetworkKind Proposal](https://docs.google.com/document/d/1ok-7CY6wJGTJmNCJGsmEInEboHmpqkXKNcKcIJpp0Kk/edit)
    - [Multi-Network Requirements & DRA Alignment](https://docs.google.com/document/d/1angT32EkiX1d66Sr21xyGCCzVcwB_VD8e6P9u_yMCN0/edit)
  - Discussed the boundary between **DRANET** (node/device-centric dynamic resource allocation for NICs, RDMA, IPVLAN, and VF attachment) and **Multi-Network** (higher-level logical network isolation, L3 VRF/overlay domains, multi-homing NetworkPolicy semantics, and DNS/Service discovery per network).

---

### August 13, 2026
- **Triage**:
  - Triaged `kube-proxy` `nftables` single-endpoint optimization ([#140723](https://github.com/kubernetes/kubernetes/pull/140723)) and UDP stale conntrack cleanup on Service deletion ([#141371](https://github.com/kubernetes/kubernetes/pull/141371)).

---

### August 27, 2026
- **Triage**:
  - Triaged kubelet link-local IPv6 DNS filtering ([#140880](https://github.com/kubernetes/kubernetes/pull/140880)) and `NetworkPolicy` / `ClusterNetworkPolicy` updates.

---

### September 10, 2026
- **Triage**: Routine bug scrub.
- **[KEP-6313: Network-Isolated (Hermetic) Pods](https://docs.google.com/document/d/1vVZ2zazJnDaj3z0Ahi6fknGJo_SIYXoteQZ3Q3BGYBI/edit)** (*Jeremy Rickard, Tim Hockin, Dan Winship, Antonio Ojea*):
  - Reviewed the proposal (and KEP PR [#6317](https://github.com/kubernetes/enhancements/pull/6317)) to support Pods that run in an isolated network namespace with only a loopback interface (`lo`), skipping the default cluster CNI plugin.
  - Discussed two primary use cases:
    1. **Hermetic Builds & Supply Chain Security (SLSA)**: Running build containers (e.g., Tekton) with a platform-enforced guarantee of zero external network connectivity after source fetch.
    2. **DRA-Only Networking**: Running AI/HPC or telecom Pods whose network interfaces are attached exclusively via DRA (`DRANET`) rather than the cluster's default CNI.
  - Discussed whether to express this via `dnsPolicy` / `hostNetwork` / a new `spec.networkMode: Isolated` (or `None`) field, and how `PodIP` reporting and `NetworkPolicy` interact when a Pod has no primary CNI interface.
  - **Outcome**: Merged KEP-6313 ([#6317](https://github.com/kubernetes/enhancements/pull/6317)) targeting v1.38 Alpha.

---

### September 17, 2026 (Rescheduled from Sep 24)
- **Triage**: Routine bug scrub.
- **Kube Agentic Networking Subproject Update** (*Faseela K, Maolin, Fazal*):
  - Presented progress since the April `v1alpha1` review:
    - Added fine-grained **MCP `tools/call` method, tool name, and parameter authorization** ([#180](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/180)).
    - Added **CEL expression dynamic authorization** ([#188](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/188)) and **`ExternalAuth` integration** ([#191](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/191)) in `XAccessPolicy`.
    - Expanded conformance testing and OpenTelemetry authorization span tracing.

---

### October 8, 2026
- **Triage**: Routine bug scrub; noted v1.38 graduations (`StrictIPCIDRValidation` GA in [#141979](https://github.com/kubernetes/kubernetes/pull/141979), `NFTablesLocalhostNodePorts` Beta in [#6378](https://github.com/kubernetes/enhancements/pull/6378)).
- **[Gateway API v1.7 SIG-NETWORK Review (Part 1)](https://docs.google.com/document/d/1n7-O6i9ZNc_UKP44CPVaazNBDZNegVf0xBbFg8syXIk/edit)** (*Rob Scott, Snigdha, Steven Jin, Gaurav*):
  - Presented the **Gateway API v1.7** changes for SIG-NETWORK review:
    - **Graduating to Standard**: `HTTPRoute` Retries (`HTTPRouteRetry`, GEP-1731) and Route Rule Name Uniqueness CEL validation (#5009).
    - **New in Experimental**:
      1. Gateway Address Routability (`Routability`, GEP-5093)
      2. Standardized OpenTelemetry Attribute Dictionary (GEP-5253)
      3. `XTelemetryPolicy` for Distributed Tracing (GEP-4768)
      4. Pre-Routing Listener Filters (GEP-5224)
      5. `ClusterTrustBundle` in `caCertificateRefs` (GEP-5075)
      6. `XBackend` Session Persistence & `EndpointSelector` destinations (GEP-1619 / GEP-4894)
- **[KEP-5939: Multi-Cluster Service API Cluster Selection](https://docs.google.com/presentation/d/1Y7SwHb_ja9G1qaS2Bp95HlQ5yB-NTIMzO9jQOBBHKw0/edit)** (*Jeremy Olmsted-Thompson, Etienne Coutaud, MrIncompetent*):
  - Presented KEP-5939 to add cluster-level targeting and weighted traffic splitting to `mcs-api` (`ClusterId` / `multicluster.kubernetes.io/source-cluster` on `EndpointSlice` and `TargetClusters` on `ServiceImport`).
  - Discussed alignment with `XBackend` and `EndpointSelector` so multi-cluster traffic weights work consistently across both MCS-API and Gateway API.

---

### October 22, 2026
- **Triage**: Routine bug scrub.
- **Gateway API v1.7 Review Follow-Up & Approval** (*Rob Scott*):
  - Addressed final review questions from the October 8 meeting (including `XTelemetryPolicy` sampling syntax, `Routability` enum naming, and `v1.7.0-rc1` feedback) and completed SIG-NETWORK sign-off for **Gateway API v1.7.0**.
- **KubeCon NA 2026 SIG-NETWORK Presentation Planning** (*Bowei Du, Guilherme Cassolato, Dan Winship, Nadia Pinaeva, Rob Scott*):
  - Agreed on the core agenda and speakers for the KubeCon NA 2026 SIG-NETWORK session:
    1. **Gateway API Updates (v1.5, v1.6, v1.7 & `XBackend` / Service Decomposition)**
    2. **2026 SIG-NETWORK Subprojects Survey & Ecosystem Health**
    3. **Network Policy API (`ClusterNetworkPolicy` `v0.2.0` & Road to `v1beta1`)**
    4. **`kube-proxy` `nftables` Mode & Localhost NodePorts (KEP-6032 / KEP-5343 / IPVS Deprecation)**
    5. **DRANET (v1.1–v1.5), Ingress2Gateway (v1.0–v1.2), AI/Agentic Networking (`GIE` & `kube-agentic-networking`), and Core KEP Graduations**
