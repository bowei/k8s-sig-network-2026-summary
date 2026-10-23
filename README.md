# Kubernetes SIG-NETWORK 2026 Overview (KubeCon NA 2026 Briefing)

- **Period Covered**: January 1, 2026 – October 22, 2026 (Kubernetes **v1.36**, **v1.37**, and **v1.38** cycles; v1.38 is still in development)
- **SIG Chairs**: Bowei Du (`@bowei`), Guilherme Cassolato (`@guicassolato`), Michael Zappa (`@mikezappa87`)
- **SIG Tech Leads**: Tim Hockin (`@thockin`), Dan Winship (`@danwinship`), Antonio Ojea (`@aojea`)
- **Verification status**: Fact-checked on October 23, 2026. Claims that could not be confirmed against primary sources are listed in [§7](#7-unverified-claims).

---

## 1. Executive Summary: The 6 Defining Themes of SIG-NETWORK in 2026

1. **Gateway API Reaches Full L4/L7 Maturity & Begins Decomposing `Service` (`v1.5`, `v1.6`, `v1.7` RC)**:
   - Shipped two minor releases in 2026, **v1.5** (February) and **v1.6** (June). **v1.7** is at release candidate (`v1.7.0-rc.2`, published October 23, 2026). Releases now follow a new release-train cadence, with periodic `monthly-*` experimental releases in between.
   - Graduated **7 capabilities to the Standard channel** in v1.5–v1.6: `TLSRoute`, `TCPRoute`, `UDPRoute`, `ListenerSet`, `HTTPCORSFilter` (CORS), Gateway Frontend Client Cert Validation (mTLS), and Gateway Backend Client Cert Selection. v1.5 also promoted the already-Standard `ReferenceGrant` to `v1`. Three more items are slated to graduate to Standard in v1.7 (currently RC): `HTTPRouteRetry` (Retries), route rule name uniqueness validation, and `BackendTLSPolicy` SAN/TLS options.
   - Launched the **`XBackend` (`v1alpha1`) + `EndpointSelector`** initiative to decompose the monolithic Kubernetes `Service` API. `EndpointSelector` is proposed as KEP-6116; its KEP PR, [kubernetes/enhancements#6127](https://github.com/kubernetes/enhancements/pull/6127), is still open. The v1.7 RC also adds 6 new Experimental features: Gateway Address Routability, OTel Attribute Dictionary, `XTelemetryPolicy`, Pre-Routing Listener Filters, `ClusterTrustBundle`, and `EndpointSelector` + Session Persistence on `XBackend`.
2. **`kube-proxy` Advances Toward `nftables` by Default & Deprecates `ipvs`**:
   - **KEP-6032 (`KubeProxyNFTablesLocalhostNodePorts`)** shipped in Alpha (v1.37) and graduates to Beta, enabled by default, in v1.38. It adds a userspace TCP proxy for `127.0.0.1:<nodePort>` traffic, active when `--nodeport-addresses` includes `localhost`, plus new `localhost`/`all` keywords for that flag. This closes a key compatibility gap for **KEP-5343 (Making `nftables` the default `kube-proxy` backend)**. The default has not flipped as of v1.38; that is planned for a later release.
   - Simplified `nftables` ruleset generation by cutting `jump`s through inlined masquerade/hairpin handling and doing direct single-endpoint DNAT without `numgen`. Native `netlink` rule listing (`knftables` v0.0.22) now sits behind the new `NFTablesNetlink` feature gate (Beta, on by default in v1.37).
   - In-tree `ipvs` mode has been deprecated since v1.35. In v1.37, **KEP-5495** added a `KubeProxyIPVS` feature gate (marked Deprecated, still enabled by default) as the next step toward disabling and removing it.
3. **High-Performance, AI & Agentic Networking (`DRANET`, `GIE`, `kube-agentic-networking`)**:
   - **DRANET** shipped 5 minor releases (**v1.1.0 – v1.5.0**), expanding from physical NIC/RDMA pass-through to **dynamic on-demand IPVLAN subinterfaces, Webhook/Whereabouts IPAM, VRF & Policy-Based Routing, BoltDB checkpointing, and multi-cloud AI fabric discovery** (GCE, AWS EFA, Azure AKS, OCI OKE, Alibaba eRDMA, CoreWeave).
   - **Gateway API Inference Extension (GIE)**:
     - Shipped **v1.4.0** (March 2026), **v1.5.0** (April 2026), and **v1.6.0** (August 2026).
     - v1.5.0 deprecated `v1alpha2` `InferencePool` in favor of `v1`.
     - v1.6.0 completed the move of the heavyweight production Endpoint Picker (EPP) and Body-Based Router (BBR) to **`llm-d`**, and added a clean in-repo **Lightweight EPP (`lwepp`)** reference implementation.
   - **Kube Agentic Networking**:
     - **v0.1.0** (June 2026) shipped the `agentic.networking.x-k8s.io/v1alpha1` API with **Model Context Protocol (MCP) `tools/call` method/tool authorization**, dynamic **CEL policy evaluation**, and `ExternalAuth` support.
     - **v0.2.0** (September 2026) added MCP parameter (L7) authorization and authorization tracing.
4. **Smooth Post-`ingress-nginx` Migration with `ingress2gateway` `v1.0` – `v1.2`**:
   - Around the March 2026 retirement of `kubernetes/ingress-nginx` (archived March 24), **`ingress2gateway`** reached **v1.0.0** (March 20, 2026), **v1.1.0** (April 2026), and **v1.2.0** (July 2026).
   - Changes over these releases:
     - Separated input **Providers** from output **Emitters**.
     - Adopted Gateway API v1.5 Standard features (CORS, `TLSRoute`).
     - Expanded `ingress-nginx` and GCE annotation translation.
     - Added a new **Traefik provider**.
     - Added implementation Emitters for **Envoy Gateway, kgateway, agentgateway, and Airlock Microgateway**, alongside the default `standard` and the `gce` emitters.
5. **Next-Gen Network Security (`ClusterNetworkPolicy` `v0.2.0` → `v1beta1` & Hermetic Pods)**:
   - **`network-policy-api`** released **v0.2.0** (`ClusterNetworkPolicy` `v1alpha2`, unifying the `Admin` and `Baseline` tiers). It then refined NPEP-133 FQDN egress rules, moved `NamedPort` to Experimental, and expanded conformance tests ahead of `v1beta1`.
   - **`kube-network-policies`** shipped **v1.1.0 – v1.1.2** (June–September 2026; v1.0.0 shipped in November 2025). These releases add full `ClusterNetworkPolicy` `v1alpha2` support and Pod-to-Node egress enforcement.
   - **KEP-6313 (Network-Isolated / Hermetic Pods)** merged, targeting v1.38 Alpha; the implementation is still pending. It lets Pods opt out of default CNI attachment, for supply-chain-secure hermetic builds and DRA-only networking.
6. **Core API Graduations, Legacy Cleanups & 2026 Subproject Health Survey**:
   - Graduated **KEP-4858 (`StrictIPCIDRValidation`, v1.38)**, **KEP-5311 (`RelaxedServiceNameValidation`, v1.37)**, and **KEP-4762 (`HostnameOverride`, v1.37)** to **GA**.
   - Removed legacy feature gates: `TopologyAwareHints`, `ServiceTrafficDistribution`, `PreferSameTrafficDistribution`, `MultiCIDRServiceAllocator`, `RelaxedDNSSearchValidation`.
   - Deprecated **`Service.spec.externalIPs` (KEP-5707, v1.36)**.
   - Moved toward deprecating **`kube-dns`**: the decision is recorded in the SIG meeting notes, and `node-local-dns` has been extracted into the standalone `sigs.k8s.io/node-local-dns` module (1.27.x).
   - Ported the `kube-apiserver` `service/proxy` from `v1.Endpoints` to `EndpointSlice` (**KEP-4974**).
   - Completed a **2026 Subproject Status Survey** across SIG-NETWORK's subprojects and their key repositories.

---

## 2. Proposed KubeCon NA 2026 Presentation Structure

This structure is adapted from the October 22, 2026 SIG-NETWORK planning discussion. The order and grouping have been adjusted, and the MCS and DNS items were added. The talk is organized into five core segments:

| Section | Topic | Key Talking Points |
| :--- | :--- | :--- |
| **Part 1** | **SIG-NETWORK Overview & 2026 Subproject Survey** | Welcome new Co-Chair Guilherme Cassolato; findings from the subproject survey; retiring `ingress-nginx` (archived March 2026) & deprecating `kube-dns`; dormant repos already retired (`kpng` 2024, `blixt` 2025). |
| **Part 2** | **Core Kubernetes & `kube-proxy` (`v1.36` – `v1.38`)** | Path to `nftables` by default (KEP-5343, not yet default) + Localhost NodePorts userspace proxy (KEP-6032 Beta); IPVS deprecation & `KubeProxyIPVS` gate (KEP-5495); GA graduations (KEP-4858, KEP-5311, KEP-4762); `externalIPs` & `Endpoints` deprecation; Hermetic Pods (KEP-6313, KEP merged, Alpha targeted). |
| **Part 3** | **Gateway API (`v1.5` – `v1.7` RC), `ingress2gateway` & Decomposing `Service`** | Standard graduations (`TLSRoute`, `TCPRoute`, `UDPRoute`, `ListenerSet`, CORS, mTLS) + Retries, rule-name uniqueness & `BackendTLSPolicy` options pending in v1.7 (RC); v1.7 RC Experimental features (Routability, OTel/`XTelemetryPolicy`, Listener Filters, `ClusterTrustBundle`); `XBackend` + `EndpointSelector`; `ingress2gateway` v1.0–v1.2 (Traefik provider, implementation Emitters). |
| **Part 4** | **Network Policy (`ClusterNetworkPolicy` `v0.2.0` → `v1beta1`)** | Unified `ClusterNetworkPolicy` (`Admin` + `Baseline` tiers in `v1alpha2`); `kube-network-policies` v1.1 reference implementation; `v1beta1` prep (NPEP-133 FQDN egress in Admin tier, hairpin capability model, `NamedPort` experimental). |
| **Part 5** | **AI, High-Performance & Multi-Cluster Networking** | **DRANET** v1.1–v1.5 (dynamic IPVLAN, Whereabouts IPAM, VRF, RDMA/IB, 6 cloud providers); **GIE** v1.5–v1.6 (`llm-d` EPP transition & `lwepp`); **Kube Agentic Networking** `v1alpha1` (v0.1.0/v0.2.0: MCP tool auth + CEL); **MCS-API** Cluster Selection (KEP-5939, a SIG-Multicluster KEP); **ExternalDNS** v0.21–v0.23 (+v0.24.0 on Oct 23) & **NodeLocalDNS** 1.27. |

---

## 3. Core Kubernetes (`kubernetes/kubernetes`) & KEP Matrix (v1.36 – v1.38)

### 3.1 KEP Graduation & Deprecation Matrix (2026)

| KEP | Feature / Gate | v1.36 (Spring 2026) | v1.37 (Summer 2026) | v1.38 (Fall 2026, in development) | Impact |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **KEP-4762** | Arbitrary FQDN as Pod Hostname (`HostnameOverride`) | Beta | **GA** | GA | Pods can set full FQDN via `hostnameOverride` |
| **KEP-4858** | Strict IP & CIDR Validation (`StrictIPCIDRValidation`) | **Beta** | Beta | **GA** | Rejects ambiguous leading-zero/non-canonical IPs/CIDRs |
| **KEP-4974** | Deprecate `core/v1` `Endpoints` | Warnings (since v1.33) | `apiserver` `service/proxy` ported to `EndpointSlice` (`#134860`) | — | Eliminates `Endpoints` scale bottleneck |
| **KEP-5311** | Relaxed Service Name Validation (`RelaxedServiceNameValidation`) | **Beta** | **GA** | GA | Allows DNS-label Service names (e.g., starting with digits) |
| **KEP-5343** | Make `nftables` Default `kube-proxy` Mode | — | KEP merged ([kubernetes/enhancements#5344](https://github.com/kubernetes/enhancements/pull/5344)); unblocked by KEP-6032 | Not yet default (flip planned for a later release) | Modern kernel dataplane replacing `iptables` |
| **KEP-5495** | Deprecate `ipvs` Mode in `kube-proxy` | Deprecated (since v1.35) | **Deprecated**; `KubeProxyIPVS` gate added (`#139397`, still on by default) | Deprecated (gate still on by default) | Users steered to `nftables`/CNI; gate to be disabled and code removed in later releases |
| **KEP-5707** | Deprecate `Service.spec.externalIPs` | **Deprecated** (`#137293`) | Deprecated | Deprecated | Mitigates arbitrary IP hijacking footgun |
| **KEP-6032** | `nftables` Localhost NodePort Userspace Proxy (`KubeProxyNFTablesLocalhostNodePorts`) | — | **Alpha** (`#138427`) | **Beta (On by default)** (`#142490`) | Restores TCP `127.0.0.1:<nodePort>` on `nftables` when `localhost` is in `--nodeport-addresses`; adds `localhost`/`all` keywords to that flag |
| **KEP-6312** | Relaxed `HostAliases` Trailing Dot Validation | — | — | KEP PR closed unmerged ([kubernetes/enhancements#6316](https://github.com/kubernetes/enhancements/pull/6316)); related implementation k/k `#139058` (titled KEP-6075) open | Would allow trailing dots (`example.com.`) in `Pod.spec.hostAliases` |
| **KEP-6313** | Network-Isolated (Hermetic) Pods | — | — | KEP merged, **Alpha** targeted ([kubernetes/enhancements#6317](https://github.com/kubernetes/enhancements/pull/6317)); implementation pending (k/k PRs #141604, #143144, #143280 open) | Pods can run in loopback-only netns without default CNI |

### 3.2 `kube-proxy` Dataplane, Scale & Conntrack Engineering
Beyond KEP-6032, `kube-proxy` received major performance and reliability improvements in 2026.
- **Simplified `nftables` Ruleset**:
  - Cut rule `jump`s by inlining masquerade/hairpin handling ([#135800](https://github.com/kubernetes/kubernetes/pull/135800), with follow-up improvements in [#136499](https://github.com/kubernetes/kubernetes/pull/136499)).
  - Eliminated `numgen` map lookups for single-endpoint Services ([#140723](https://github.com/kubernetes/kubernetes/pull/140723); IPv6 follow-up fix [#140960](https://github.com/kubernetes/kubernetes/pull/140960)).
  - Truncated long service comments to fit the 128-char `NFTNL_UDATA_RULE_COMMENT` limit ([#139516](https://github.com/kubernetes/kubernetes/pull/139516)).
  - Enabled native Linux **netlink** rule listing via `knftables` `v0.0.22`, behind the new `NFTablesNetlink` feature gate (Beta, on by default in v1.37) ([#137536](https://github.com/kubernetes/kubernetes/pull/137536)).
- **Large Cluster CPU & Memory Reduction**: Skipped full periodic forced syncs in large-cluster mode ([#138571](https://github.com/kubernetes/kubernetes/pull/138571)) and stripped `metadata.managedFields` in `kube-proxy` informers ([#140056](https://github.com/kubernetes/kubernetes/pull/140056)).
- **Conntrack & Node IP Hardening**:
  - Cleaned stale UDP conntrack entries when Services scale to 0 endpoints ([#139629](https://github.com/kubernetes/kubernetes/pull/139629)) or are deleted ([#141371](https://github.com/kubernetes/kubernetes/pull/141371)).
  - Capped `nf_conntrack_max` on high-core hosts ([#137002](https://github.com/kubernetes/kubernetes/pull/137002)), while never shrinking pre-configured higher values ([#142035](https://github.com/kubernetes/kubernetes/pull/142035)).
  - Replaced cAdvisor with `cpuset.NumCPU()` ([#140021](https://github.com/kubernetes/kubernetes/pull/140021)).
  - Automatically restarted `kube-proxy` on Node IP changes and Node deletion ([#138183](https://github.com/kubernetes/kubernetes/pull/138183)).
- **Related kubelet fix**: Filtered scoped IPv6 link-local DNS servers (`fe80::%iface`) in kubelet ([#140880](https://github.com/kubernetes/kubernetes/pull/140880)).

---

## 4. Deep-Dive by SIG-NETWORK Subproject

### 4.1 Gateway API (`kubernetes-sigs/gateway-api`)
- **v1.5 (February 2026)**:
  - Graduated **`TLSRoute`** (GEP-2643), **`ListenerSet`** (GEP-1713), **`HTTPCORSFilter`** (GEP-1767), **Gateway Frontend Client Cert Validation** (GEP-91 / GEP-3567), and **Gateway Backend Client Cert Selection** (GEP-3155) to Standard, and promoted the already-Standard **`ReferenceGrant`** to `v1` ([#4458](https://github.com/kubernetes-sigs/gateway-api/pull/4458)).
- **v1.6 (June 2026)**:
  - Graduated **`TCPRoute`** (GEP-2644) and **`UDPRoute`** (GEP-2645) to Standard (`v1`) ([#4920](https://github.com/kubernetes-sigs/gateway-api/pull/4920), [#4923](https://github.com/kubernetes-sigs/gateway-api/pull/4923)).
  - Introduced Experimental **`XBackend`** (`v1alpha1`, GEP-4894; [#4488](https://github.com/kubernetes-sigs/gateway-api/pull/4488), [#4901](https://github.com/kubernetes-sigs/gateway-api/pull/4901)).
  - Launched the interactive **Controller Matching Wizard**, driven by conformance reports ([#4584](https://github.com/kubernetes-sigs/gateway-api/pull/4584)). Also migrated the documentation site from MkDocs to Hugo/Docsy ([#4734](https://github.com/kubernetes-sigs/gateway-api/pull/4734), May 2026).
- **v1.7 (release candidate `v1.7.0-rc.2`, published October 23, 2026; not yet GA)**:
  - Graduates three items to Standard:
    - **`HTTPRouteRetry`** (GEP-1731; [#5357](https://github.com/kubernetes-sigs/gateway-api/pull/5357), [#5359](https://github.com/kubernetes-sigs/gateway-api/pull/5359))
    - **Route rule name uniqueness validation** for `HTTPRoute`/`GRPCRoute` ([#5009](https://github.com/kubernetes-sigs/gateway-api/pull/5009))
    - **`BackendTLSPolicy` SANs/TLS options** ([#4999](https://github.com/kubernetes-sigs/gateway-api/pull/4999))
  - Adds 6 Experimental capabilities:
    - **Gateway Address Routability** (GEP-5093)
    - **Standardized OTel Attribute Dictionary** (GEP-5253)
    - **`XTelemetryPolicy`** (GEP-4768)
    - **Pre-Routing Listener Filters** (GEP-5224)
    - **`ClusterTrustBundle` in `caCertificateRefs`** (GEP-5075)
    - **`EndpointSelector` + Session Persistence on `XBackend`** (GEP-1619 / GEP-4894; [#5255](https://github.com/kubernetes-sigs/gateway-api/pull/5255), [#5298](https://github.com/kubernetes-sigs/gateway-api/pull/5298))
- **Towards Decomposing `Service` (`XBackend` + `EndpointSelector`)**:
  - Joint effort between core SIG-NETWORK and Gateway API. On the core side, `EndpointSelector` is proposed as KEP-6116 ([kubernetes/enhancements#6127](https://github.com/kubernetes/enhancements/pull/6127), still open). On the Gateway API side, it builds on the `XBackend` GEP-4894. The goal is to decouple workload endpoint selection (`EndpointSelector`) and backend traffic policy/external FQDNs (`XBackend`) from VIP allocation (`Service`).

### 4.2 Ingress2Gateway (`kubernetes-sigs/ingress2gateway`) & `ingress-nginx` Retirement
- `kubernetes/ingress-nginx` was retired in March 2026; the repository was archived on March 24. Around the same time, `ingress2gateway` shipped **v1.0.0** (March 20, 2026), followed by **v1.1.0** (April 2026) and **v1.2.0** (July 2026).
- **Provider / Emitter Split**: Decoupled input Providers from output Emitters. Providers: `ingress-nginx`, `gce`, `kong`, `apisix`, `istio`, `openapi`, `cilium`, `nginx`, and `traefik`. The Traefik provider is new in v1.1.0 ([#400](https://github.com/kubernetes-sigs/ingress2gateway/pull/400)).
- **Gateway API v1.5 adoption**: Upgraded to Gateway API v1.5 ([#367](https://github.com/kubernetes-sigs/ingress2gateway/pull/367), v1.0.0), emitted CORS without the experimental flag ([#303](https://github.com/kubernetes-sigs/ingress2gateway/pull/303), [#371](https://github.com/kubernetes-sigs/ingress2gateway/pull/371)), and moved `TLSRoute` to the standard channel ([#404](https://github.com/kubernetes-sigs/ingress2gateway/pull/404), v1.1.0).
- **Implementation Emitters**: Emitters for the following, alongside the default `standard` emitter:
  - **Envoy Gateway** ([#305](https://github.com/kubernetes-sigs/ingress2gateway/pull/305))
  - **kgateway** ([#320](https://github.com/kubernetes-sigs/ingress2gateway/pull/320))
  - **agentgateway** ([#388](https://github.com/kubernetes-sigs/ingress2gateway/pull/388))
  - `gce`
  - **Airlock Microgateway** (v1.2.0, [#413](https://github.com/kubernetes-sigs/ingress2gateway/pull/413))

  All except Airlock Microgateway shipped in v1.0.0.
- **Expanded `ingress-nginx` & GCE Translation**:
  - `ingress-nginx`:
    - Regex/path rewrites
    - Canary weights/headers
    - Backend TLS
    - `ssl-redirect`
    - `ssl-passthrough` (`TLSRoute`, [#402](https://github.com/kubernetes-sigs/ingress2gateway/pull/402))
    - `app-root` ([#405](https://github.com/kubernetes-sigs/ingress2gateway/pull/405))
    - `from-to-www-redirect` ([#337](https://github.com/kubernetes-sigs/ingress2gateway/pull/337))
    - Timeouts
    - Body/buffer sizes
    - IP allow/deny ranges
  - GCE:
    - Session cookies ([#394](https://github.com/kubernetes-sigs/ingress2gateway/pull/394))
    - Cloud CDN (`GCPHTTPFilter`, [#403](https://github.com/kubernetes-sigs/ingress2gateway/pull/403))

### 4.3 Network Policy API (`network-policy-api`) & `kube-network-policies`
- **`ClusterNetworkPolicy` `v0.2.0` (`v1alpha2`)**: Released in April 2026 ([#369](https://github.com/kubernetes-sigs/network-policy-api/pull/369)). It consolidates `AdminNetworkPolicy` and `BaselineAdminNetworkPolicy` into a single `ClusterNetworkPolicy` CRD with `Admin` and `Baseline` priority tiers.
- **Preparing for `v1beta1`**:
  - Updated **NPEP-133 (FQDN Egress Rules)**: Restricted `domainNames` rules to the `Admin` tier and documented DNS security guidance ([#366](https://github.com/kubernetes-sigs/network-policy-api/pull/366)).
  - Marked **`NamedPort`** as Experimental ([#392](https://github.com/kubernetes-sigs/network-policy-api/pull/392)) so dataplane scale trade-offs do not block `v1beta1` Standard graduation.
  - Defined a **hairpin-traffic capability model**: socket-layer load-balancer implementations are exempt, and a new `HairpinServicePolicyEnforcement` conformance feature was added ([#405](https://github.com/kubernetes-sigs/network-policy-api/pull/405)).
  - Fixed CIDR field validation ([#324](https://github.com/kubernetes-sigs/network-policy-api/pull/324)) and added CEL validation tests for the CRDs ([#383](https://github.com/kubernetes-sigs/network-policy-api/pull/383)).
- **`kube-network-policies` (`v1.1.0` – `v1.1.2`, June–September 2026; `v1.0.0` shipped November 2025)**:
  - Implemented `ClusterNetworkPolicy` `v1alpha2` ([#303](https://github.com/kubernetes-sigs/kube-network-policies/pull/303)).
  - Added Pod-to-Node egress policies ([#339](https://github.com/kubernetes-sigs/kube-network-policies/pull/339)).
  - Skipped `hostNetwork` Pods ([#389](https://github.com/kubernetes-sigs/kube-network-policies/pull/389)).
  - Made `nftables` chain updates non-disruptive ([#403](https://github.com/kubernetes-sigs/kube-network-policies/pull/403)).

### 4.4 DRANET (`kubernetes-sigs/dranet`) & Multi-Network
- **5 Releases in 2026 (`v1.1.0` – `v1.5.0`)**:
  - **Dynamic On-Demand Networking & IPAM**: Added Profiles + Webhook IPAM integration with **Whereabouts** ([#223](https://github.com/kubernetes-sigs/dranet/pull/223)), dynamic **IPVLAN subinterfaces** + node-local IPAM ([#241](https://github.com/kubernetes-sigs/dranet/pull/241)), and **VRF + Policy-Based Routing** ([#74](https://github.com/kubernetes-sigs/dranet/pull/74)).
  - **AI/HPC RDMA, InfiniBand & SR-IOV**:
    - Standard DRA NUMA list attribute (KEP-6072) ([#283](https://github.com/kubernetes-sigs/dranet/pull/283))
    - InfiniBand-only RDMA devices ([#77](https://github.com/kubernetes-sigs/dranet/pull/77))
    - SR-IOV VF discovery + MTU validation ([#100](https://github.com/kubernetes-sigs/dranet/pull/100), [#213](https://github.com/kubernetes-sigs/dranet/pull/213))
  - **Production Hardening**:
    - Embedded `bbolt` Pod checkpointing with schema versioning ([#115](https://github.com/kubernetes-sigs/dranet/pull/115), [#314](https://github.com/kubernetes-sigs/dranet/pull/314))
    - Persisted allocated device attributes in `ResourceSlice` ([#230](https://github.com/kubernetes-sigs/dranet/pull/230))
    - ARP (`arp_ignore`/`arp_announce`) and IPv6 `accept_ra` sysctls ([#277](https://github.com/kubernetes-sigs/dranet/pull/277), [#341](https://github.com/kubernetes-sigs/dranet/pull/341))
    - DHCP lease release on Pod termination ([#304](https://github.com/kubernetes-sigs/dranet/pull/304))
    - Helm chart ([#117](https://github.com/kubernetes-sigs/dranet/pull/117)) and multi-arch images ([#90](https://github.com/kubernetes-sigs/dranet/pull/90))
  - **Multi-Cloud Support**:
    - GCE IPv4/IPv6 subinterface ranges ([#241](https://github.com/kubernetes-sigs/dranet/pull/241))
    - AWS EFA ([#139](https://github.com/kubernetes-sigs/dranet/pull/139), [#182](https://github.com/kubernetes-sigs/dranet/pull/182))
    - Azure AKS Placement/Interconnect Groups ([#102](https://github.com/kubernetes-sigs/dranet/pull/102), [#235](https://github.com/kubernetes-sigs/dranet/pull/235))
    - Oracle Cloud OKE RDMA/IB fabrics ([#116](https://github.com/kubernetes-sigs/dranet/pull/116), [#345](https://github.com/kubernetes-sigs/dranet/pull/345))
    - Alibaba Cloud eRDMA / RDMA bonds ([#244](https://github.com/kubernetes-sigs/dranet/pull/244), [#312](https://github.com/kubernetes-sigs/dranet/pull/312))
    - CoreWeave CKS ([#302](https://github.com/kubernetes-sigs/dranet/pull/302))
- **Multi-Network API Re-Architecture**: Per the SIG meeting notes, the `NetworkClass` / `NetworkKind` / `PodNetwork` design was revisited in July 2026 to align logical multi-network isolation with DRANET (unverified; see §7).

### 4.5 AI & Agentic Networking (`gateway-api-inference-extension` & `kube-agentic-networking`)
- **Gateway API Inference Extension (`v1.4.0` March, `v1.5.0` April, `v1.6.0` August 2026; `v1.6.2` September)**:
  - v1.5.0 deprecated `v1alpha2` `InferencePool` in favor of `inference.networking.k8s.io/v1` ([#2796](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2796)).
  - v1.6.0 moved the full production Endpoint Picker (EPP) and Body-Based Router (BBR) to **[`llm-d`](https://github.com/llm-d)**, following proposal [#2430](https://github.com/kubernetes-sigs/gateway-api-inference-extension/issues/2430):
    - Removed BBR ([#2959](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2959)) and EPP ([#2967](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2967)) from GIE.
    - Removed the alpha `InferenceObjective`/`InferenceModelRewrite`/`EndpointPickerConfig` APIs ([#2973](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2973)).
    - Introduced a self-contained **Lightweight EPP (`lwepp`)** ([#2842](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2842)) in GIE for reference and conformance testing.
- **Kube Agentic Networking (`v1alpha1`, `v0.1.0` → `v0.2.0`)**:
  - Moved the API group to **`agentic.networking.x-k8s.io`** ([#258](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/258)) and added the `v1alpha1` API ([#259](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/259), [#271](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/271)). Both first shipped in `v0.1.0` (June 2026).
  - Added the following authorization features:
    - **MCP `tools/call` method and tool name authorization** ([#279](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/279), building on the authorization rule `type` field from [#72](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/72))
    - **CEL expression authorization** ([#241](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/241))
    - **`ExternalAuth` integration** ([#83](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/83), [#187](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/187))
  - `v0.2.0` (September 2026) added the following:
    - MCP **parameter (L7) authorization** ([#309](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/309), [#315](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/315))
    - OTel gateway authorization decision tracing ([#280](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/280))

### 4.6 Multi-Cluster Services (`mcs-api`, SIG-Multicluster) & DNS (`node-local-dns`, `external-dns`)
- **MCS-API (`kubernetes-sigs/mcs-api`, a SIG-Multicluster subproject)**:
  - Introduced the `v1beta1` API ([#133](https://github.com/kubernetes-sigs/mcs-api/pull/133)) and shipped releases v0.4.0 – v0.5.2 (March–July 2026).
  - Expanded conformance testing:
    - Fixes ([#134](https://github.com/kubernetes-sigs/mcs-api/pull/134))
    - IPv6/dual-stack ([#143](https://github.com/kubernetes-sigs/mcs-api/pull/143))
    - YAML reports ([#145](https://github.com/kubernetes-sigs/mcs-api/pull/145))
    - Implementation metadata ([#149](https://github.com/kubernetes-sigs/mcs-api/pull/149))
    - EndpointSlice objects ([#155](https://github.com/kubernetes-sigs/mcs-api/pull/155))
  - Reviewed **KEP-5939 (Add Cluster Selection for Multi-Cluster Services)**, a SIG-Multicluster KEP, with SIG-NETWORK on October 8, 2026. The KEP supports targeting specific clusters; per the meeting notes, weighted traffic splitting is under discussion.
- **NodeLocal DNSCache (`kubernetes-sigs/node-local-dns`)**:
  - Following SIG-NETWORK's move to deprecate `kube-dns`, `node-local-dns` was decoupled into a standalone `sigs.k8s.io/node-local-dns` module, removing legacy `kube-dns`/`dnsmasq` code ([#6](https://github.com/kubernetes-sigs/node-local-dns/pull/6)).
  - Shipped **1.26.8**, **1.27.0**, **1.27.1**, and **1.27.2**:
    - 1.27.0 added metrics TLS ([#27](https://github.com/kubernetes-sigs/node-local-dns/pull/27)) and a configurable Prometheus bind address ([#16](https://github.com/kubernetes-sigs/node-local-dns/pull/16)).
    - 1.27.1 and 1.27.2 were build and dependency updates.
- **ExternalDNS (`kubernetes-sigs/external-dns`)**:
  - Shipped **v0.21.0**, **v0.22.0**, and **v0.23.0** (September 18, 2026) on the road to **v1.0**:
    - **v0.21**: Migrated `Gateway`/`HTTPRoute` sources to `v1` ([#6291](https://github.com/kubernetes-sigs/external-dns/pull/6291)), added `ListenerSet` source support ([#6254](https://github.com/kubernetes-sigs/external-dns/pull/6254)), and added `PTR` records ([#6232](https://github.com/kubernetes-sigs/external-dns/pull/6232)).
    - **v0.22**:
      - Migrated the `TLSRoute` source to `v1` ([#6367](https://github.com/kubernetes-sigs/external-dns/pull/6367)).
      - Switched the default annotation prefix to the GA `external-dns.kubernetes.io/` ([#6424](https://github.com/kubernetes-sigs/external-dns/pull/6424)).
      - Added a CRD ownership registry (`--registry=crd`, which stores ownership in `DNSRecord` objects; [#5372](https://github.com/kubernetes-sigs/external-dns/pull/5372)).
      - Migrated sources to client-go indexers ([#6344](https://github.com/kubernetes-sigs/external-dns/pull/6344), [#6445](https://github.com/kubernetes-sigs/external-dns/pull/6445), [#6515](https://github.com/kubernetes-sigs/external-dns/pull/6515), [#6542](https://github.com/kubernetes-sigs/external-dns/pull/6542)).
      - Added `DNAME` records ([#6553](https://github.com/kubernetes-sigs/external-dns/pull/6553)).
    - **v0.23**:
      - Added `--enable-legacy-annotation-prefix` for migration ([#6703](https://github.com/kubernetes-sigs/external-dns/pull/6703)).
      - Introduced FIPS 140-3 images ([#6657](https://github.com/kubernetes-sigs/external-dns/pull/6657)).
      - Added more indexer migrations ([#6545](https://github.com/kubernetes-sigs/external-dns/pull/6545)).
      - Added Cloudflare `TLSA` records ([#6616](https://github.com/kubernetes-sigs/external-dns/pull/6616)).
  - **v0.24.0** (released October 23, 2026, just after this report's period) adds the following:
    - `TCPRoute` `v1` ([#6755](https://github.com/kubernetes-sigs/external-dns/pull/6755))
    - RFC2136 `TLSA` ([#6721](https://github.com/kubernetes-sigs/external-dns/pull/6721))
    - `DNSEndpoint.status` reporting ([#6767](https://github.com/kubernetes-sigs/external-dns/pull/6767), [#6760](https://github.com/kubernetes-sigs/external-dns/pull/6760))

---

## 5. 2026 SIG-NETWORK Subproject Survey Summary

Per the SIG meeting notes, Co-Chair Guilherme Cassolato (`@guicassolato`) ran the survey between April and June 2026. It evaluated SIG-NETWORK's subprojects and their key repositories. Some related components outside SIG-NETWORK's subprojects are listed separately for context.

| Category | Subprojects & Key Repos | Status & Action Items |
| :--- | :--- | :--- |
| **High Velocity & Growing** | `gateway-api` (incl. `ingress2gateway`), `gateway-api-inference-extension`, `dranet`, `external-dns`, `network-policy` (`network-policy-api`, `kube-network-policies`), `kube-agentic-networking` | Healthy multi-vendor maintainer teams, regular releases, strong KubeCon presence. |
| **Stable Infrastructure** | `knftables`, `node-local-dns` | Stable maintenance; `node-local-dns` decoupled from `kube-dns`. |
| **Deprecated / Retired** | `ingress-nginx`, `kube-dns` (`dns`); already retired: `kpng`, `blixt` | `ingress-nginx` archived March 24, 2026; `kube-dns` deprecation agreed January 2026 (per meeting notes); `kpng` (2024) and `blixt` (2025) were already moved to `kubernetes-retired` before 2026. |
| **Needs Contributor Diversity / Follow-Up** | `ingress-gce` (repo under the `ingress` subproject), `ip-masq-agent`, `multi-network` | `ingress-gce` in maintenance mode as users transition to Gateway API; `multi-network` re-aligning API proposal with DRANET. |
| *Related components (not SIG-NETWORK subprojects)* | In-tree `kube-proxy` and EndpointSlice (`kubernetes/kubernetes`), `mcs-api` (SIG-Multicluster), CNI (CNCF `containernetworking`) | `kube-proxy` advancing toward the `nftables` default (see §3). |

---

## 6. Index of Generated Repository & Meeting Reports

All detailed commit-level reports and meeting summaries are available in this workspace.

> **Caveat**: These files are machine-generated secondary sources. The fact-check found wrong release dates, authors, and PR numbers in them. The same short commit hashes also appear across unrelated repositories, and some cited PRs do not exist or belong to other work (e.g. ingress2gateway #304/#356/#411/#422; enhancements #5708). Together these suggest fabricated data. Prefer the linked GitHub sources.

- [sig-network-meeting-notes.md](sig-network-meeting-notes.md) — Chronological & thematic summary of all 2026 SIG-NETWORK biweekly meetings and linked design docs
- [kubernetes-activity.md](kubernetes-activity.md) — `kubernetes/kubernetes` SIG-NETWORK commits (v1.36–v1.38)
- [enhancements-activity.md](enhancements-activity.md) — `kubernetes/enhancements` SIG-NETWORK KEPs (2026)
- [gateway-api-activity.md](gateway-api-activity.md) — `kubernetes-sigs/gateway-api` (v1.5, v1.6, v1.7.0-rc1)
- [dranet-activity.md](dranet-activity.md) — `kubernetes-sigs/dranet` (v1.1.0 – v1.5.0)
- [network-policy-api-activity.md](network-policy-api-activity.md) — `kubernetes-sigs/network-policy-api` (`ClusterNetworkPolicy` v0.2.0 & v1beta1 prep)
- [ingress2gateway-activity.md](ingress2gateway-activity.md) — `kubernetes-sigs/ingress2gateway` (v1.0.0 – v1.2.0)
- [kube-network-policies-activity.md](kube-network-policies-activity.md) — `kubernetes-sigs/kube-network-policies` (v1.0.0 – v1.1.2)
- [kube-agentic-networking-activity.md](kube-agentic-networking-activity.md) — `kubernetes-sigs/kube-agentic-networking` (`v1alpha1` / MCP & CEL auth)
- [gateway-api-inference-extension-activity.md](gateway-api-inference-extension-activity.md) — `kubernetes-sigs/gateway-api-inference-extension` (v1.3.1 – v1.5.0-rc.2 as recorded in the file, `llm-d` transition & `lwepp`; GitHub shows v1.4.0, v1.5.0 and v1.6.0 GA releases in March, April and August 2026)
- [mcs-api-activity.md](mcs-api-activity.md) — `kubernetes-sigs/mcs-api` & KEP-5939 Cluster Selection
- [node-local-dns-activity.md](node-local-dns-activity.md) — `kubernetes-sigs/node-local-dns` (1.26.8 – 1.27.2)
- [external-dns-activity.md](external-dns-activity.md) — `kubernetes-sigs/external-dns` (v0.21.0 – v0.23.0)
- [knftables-activity.md](knftables-activity.md) — `kubernetes-sigs/knftables` (v0.0.20 – v0.0.22)

---

## 7. Unverified Claims

The claims below are still unconfirmed after a `gh`-based pass over every PR and release cited in this README. All other cited PRs were checked for existence, merge state, merge date, and title. Where a claim's only support is local files, treat it with extra caution (see the §6 caveat).

| README section | Claim | Why unverified | What would verify it |
| :--- | :--- | :--- | :--- |
| §1 (theme 6), §2, §5 | `kube-dns` deprecation agreed in January 2026 | Only in local meeting notes (Jan 15, 2026 entry) | SIG-NETWORK meeting notes doc or a published deprecation announcement |
| §2 | Talk structure derives from the October 22, 2026 planning discussion | Only in local meeting notes, which use a different order and grouping | SIG-NETWORK meeting notes doc |
| §4.4 | Multi-Network `NetworkClass`/`NetworkKind`/`PodNetwork` redesign in July 2026 | Only in local meeting notes (July 30 entry); the `kubernetes-sigs/multi-network` repo has had no commits since August 2025 | SIG-NETWORK meeting notes doc only |
| §4.6 | KEP-5939 reviewed October 8, 2026; weighted traffic splitting in scope | Only in local meeting notes | KEP-5939 text / meeting notes doc |
| §5 | Survey dates (April–June 2026) and owner; category assignments and statuses; subproject↔repo mapping (e.g. whether `gateway-api-inference-extension` is its own subproject) | Only in local meeting notes, which contradict themselves (April–June vs April–October) | Survey results doc / meeting notes doc; `sigs.yaml` |
