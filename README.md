# Kubernetes SIG-NETWORK 2026 Overview (KubeCon NA 2026 Briefing)

- **Period Covered**: January 1, 2026 – October 22, 2026 (Kubernetes **v1.36**, **v1.37**, and **v1.38** cycles)
- **SIG Chairs**: Bowei Du (`@bowei`), Guilherme Cassolato (`@guicassolato`)
- **SIG Tech Leads**: Tim Hockin (`@thockin`), Dan Winship (`@danwinship`), Antonio Ojea (`@aojea`), Rob Scott (`@robscott`)

---

## 1. Executive Summary: The 6 Defining Themes of SIG-NETWORK in 2026

1. **Gateway API Reaches Full L4/L7 Maturity & Begins Decomposing `Service` (`v1.5`, `v1.6`, `v1.7`)**:
   - Shipped three major releases in 2026 on a new release-train cadence with monthly experimental releases.
   - Graduated **9 major capabilities to the Standard (`v1`) channel**: `TLSRoute`, `TCPRoute`, `UDPRoute`, `ListenerSet`, `HTTPCORSFilter` (CORS), `HTTPRouteRetry` (Retries), Gateway Frontend Client Cert Validation (mTLS), Gateway Backend Client Cert Selection, and `ReferenceGrant` (`v1`).
   - Launched the **`XBackend` (`v1alpha1`) + `EndpointSelector` (KEP-6116)** initiative to decompose the monolithic Kubernetes `Service` API, alongside 6 new Experimental features in v1.7 (Gateway Address Routability, OTel Attribute Dictionary, `XTelemetryPolicy`, Pre-Routing Listener Filters, `ClusterTrustBundle`, and `XBackend` Session Persistence).
2. **`kube-proxy` Completes the Path to `nftables` by Default & Retires `ipvs`**:
   - **KEP-6032 (`NFTablesLocalhostNodePorts`)** shipped in Alpha (v1.37) and graduated to Beta (enabled by default in v1.38), adding a lightweight userspace fallback proxy for `127.0.0.1:<nodePort>` traffic and `--nodeport-addresses` `localhost`/`all` keywords — resolving the final compatibility blocker for **KEP-5343 (Making `nftables` the default `kube-proxy` backend)**.
   - Simplified `nftables` ruleset generation (squashing endpoint chains into the `services` vmap, direct single-endpoint DNAT without `numgen`, and native `netlink` rule listing via `knftables` v0.0.22).
   - **KEP-5495** was updated to formally deprecate and feature-gate in-tree `ipvs` mode after confirming no maintainers stepped forward for an out-of-tree IPVS proxy.
3. **High-Performance, AI & Agentic Networking (`DRANET`, `GIE`, `kube-agentic-networking`)**:
   - **DRANET** shipped 5 minor releases (**v1.1.0 – v1.5.0**), expanding from physical NIC/RDMA pass-through to **dynamic on-demand IPVLAN subinterfaces, Webhook/Whereabouts IPAM, VRF & Policy-Based Routing, BoltDB checkpointing, and multi-cloud AI fabric discovery** (GCE, AWS EFA, Azure AKS, OCI OKE, Alibaba eRDMA/Lingjun, CoreWeave).
   - **Gateway API Inference Extension (GIE)** shipped **v1.4 & v1.5**, deprecated `v1alpha2` `InferencePool` in favor of `v1`, migrated the heavyweight production Endpoint Picker (EPP) and Body-Based Router (BBR) to **`llm-d`**, and introduced a clean in-repo **Lightweight EPP (`lwepp`)** reference implementation.
   - **Kube Agentic Networking** transitioned to `agentic.networking.x-k8s.io/v1alpha1` (`v0.2.0`), adding **Model Context Protocol (MCP) `tools/call` method/tool/parameter authorization**, dynamic **CEL policy evaluation**, and `ExternalAuth` support.
4. **Smooth Post-`ingress-nginx` Migration with `ingress2gateway` `v1.0` – `v1.2`**:
   - Following the March 2026 archival of `kubernetes/ingress-nginx`, **`ingress2gateway`** reached **v1.0.0** (Mar 2026), **v1.1.0** (Apr 2026), and **v1.2.0** (Jul 2026).
   - Separated input **Providers** from output **Emitters**, adopted Gateway API v1.5 Standard features (CORS, `TLSRoute`), expanded `ingress-nginx` and GCE annotation translation, and added 5 new Emitters (**Envoy Gateway, kgateway, agentgateway, Traefik, Airlock Microgateway**).
5. **Next-Gen Network Security (`ClusterNetworkPolicy` `v0.2.0` → `v1beta1` & Hermetic Pods)**:
   - **`network-policy-api`** released **v0.2.0** (`ClusterNetworkPolicy` `v1alpha2` unifying `Admin` and `Baseline` tiers), refined NPEP-133 FQDN egress rules, moved `NamedPort` to Experimental, and expanded conformance tests ahead of `v1beta1`.
   - **`kube-network-policies`** shipped **v1.0.0 – v1.1.2** with full `ClusterNetworkPolicy` `v1alpha2` and Pod-to-Node egress enforcement.
   - **KEP-6313 (Network-Isolated / Hermetic Pods)** merged for v1.38 Alpha, allowing Pods to opt out of default CNI attachment for supply-chain-secure hermetic builds and DRA-only networking.
6. **Core API Graduations, Legacy Cleanups & 2026 Subproject Health Survey**:
   - Graduated **KEP-4858 (`StrictIPCIDRValidation`, v1.38)**, **KEP-5311 (`RelaxedServiceNameValidation`, v1.37)**, and **KEP-4762 (`PodHostNameFQDN`, v1.37)** to **GA**; removed legacy feature gates (`TopologyAwareHints`, `ServiceTrafficDistribution`, `PreferSameTrafficDistribution`, `MultiCIDRServiceAllocator`, `RelaxedDNSSearchValidation`).
   - Deprecated **`Service.spec.externalIPs` (KEP-5707, v1.36)** and **`kube-dns`** (extracting `node-local-dns` into standalone `sigs.k8s.io/node-local-dns` v1.27.x), and ported `kube-apiserver` `service/proxy` from `v1.Endpoints` to `EndpointSlice` (**KEP-4974**).
   - Completed a comprehensive **2026 Subproject Status Survey** across all **22 SIG-NETWORK subprojects**.

---

## 2. Proposed KubeCon NA 2026 Presentation Structure

Based on the October 22, 2026 SIG-NETWORK planning discussion, the KubeCon talk is organized into five core segments:

| Section | Topic | Key Talking Points |
| :--- | :--- | :--- |
| **Part 1** | **SIG-NETWORK Overview & 2026 Subproject Survey** | Welcome new Co-Chair Guilherme Cassolato; findings from surveying all 22 subprojects; retiring `ingress-nginx` & `kube-dns`; archival candidates (`blixt`, `kpng`). |
| **Part 2** | **Core Kubernetes & `kube-proxy` (`v1.36` – `v1.38`)** | `kube-proxy` `nftables` default path (KEP-5343) + Localhost NodePorts userspace proxy (KEP-6032 Beta); IPVS deprecation (KEP-5495); GA graduations (KEP-4858, KEP-5311, KEP-4762); `externalIPs` & `Endpoints` deprecation; Hermetic Pods (KEP-6313). |
| **Part 3** | **Gateway API (`v1.5` – `v1.7`), `ingress2gateway` & Decomposing `Service`** | Standard graduations (`TLSRoute`, `TCPRoute`, `UDPRoute`, `ListenerSet`, CORS, Retries, mTLS); v1.7 Experimental features (Routability, OTel/`XTelemetryPolicy`, Listener Filters, `ClusterTrustBundle`); `XBackend` + `EndpointSelector`; `ingress2gateway` v1.0–v1.2 Emitters. |
| **Part 4** | **Network Policy (`ClusterNetworkPolicy` `v0.2.0` → `v1beta1`)** | Unified `ClusterNetworkPolicy` (`Admin` + `Baseline` tiers in `v1alpha2`); `kube-network-policies` v1.1 reference implementation; `v1beta1` prep (NPEP-133 FQDN egress in Admin tier, hairpin traffic, `NamedPort` experimental). |
| **Part 5** | **AI, High-Performance & Multi-Cluster Networking** | **DRANET** v1.1–v1.5 (dynamic IPVLAN, Whereabouts IPAM, VRF, RDMA/IB, 6 cloud providers); **GIE** v1.5 (`llm-d` EPP transition + `lwepp`); **Kube Agentic Networking** `v1alpha1` (MCP tool auth + CEL); **MCS-API** Cluster Selection (KEP-5939); **ExternalDNS** v0.21–v0.23 & **NodeLocalDNS** v1.27. |

---

## 3. Core Kubernetes (`kubernetes/kubernetes`) & KEP Matrix (v1.36 – v1.38)

### 3.1 KEP Graduation & Deprecation Matrix (2026)

| KEP | Feature / Gate | v1.36 (Spring 2026) | v1.37 (Summer 2026) | v1.38 (Fall 2026) | Impact |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **KEP-4762** | Arbitrary FQDN as Pod Hostname (`PodHostNameFQDN`) | Beta | **GA** | GA | Pods can set full FQDN via `hostnameOverride` |
| **KEP-4858** | Strict IP & CIDR Validation (`StrictIPCIDRValidation`) | **Beta** | Beta | **GA** | Rejects ambiguous leading-zero/non-canonical IPs/CIDRs |
| **KEP-4974** | Deprecate `core/v1` `Endpoints` | Warnings | `apiserver` `service/proxy` ported to `EndpointSlice` | Continued removal of internal consumers | Eliminates `Endpoints` scale bottleneck |
| **KEP-5311** | Relaxed Service Name Validation (`RelaxedServiceNameValidation`) | **Beta** | **GA** | GA | Allows DNS-label Service names (e.g., starting with digits) |
| **KEP-5343** | Make `nftables` Default `kube-proxy` Mode | KEP Merged | Unblocked by KEP-6032 | Default transition | Modern kernel dataplane replacing `iptables` |
| **KEP-5495** | Deprecate `ipvs` Mode in `kube-proxy` | — | **Deprecated** (Feature-gated in `#139397`) | Deprecated | No out-of-tree maintainer found; users steered to `nftables`/CNI |
| **KEP-5707** | Deprecate `Service.spec.externalIPs` | **Deprecated** (`#137293`) | Deprecated | Deprecated | Mitigates arbitrary IP hijacking footgun |
| **KEP-6032** | `nftables` Localhost NodePort Userspace Proxy (`NFTablesLocalhostNodePorts`) | — | **Alpha** (`#138427`) | **Beta (On by default)** (`#142490`) | Fixes `127.0.0.1:<nodePort>` on `nftables`; adds `localhost`/`all` to `--nodeport-addresses` |
| **KEP-6312** | Relaxed `HostAliases` Trailing Dot Validation | — | — | **Merged** (`#140654`) | Allows trailing dots (`example.com.`) in `Pod.spec.hostAliases` |
| **KEP-6313** | Network-Isolated (Hermetic) Pods | — | — | **Alpha** (`#6317`) | Pods can run in loopback-only netns without default CNI |

### 3.2 `kube-proxy` Dataplane, Scale & Conntrack Engineering
Beyond KEP-6032, `kube-proxy` received major performance and reliability improvements in 2026:
- **Simplified `nftables` Ruleset**: Squashed per-service endpoint chains directly into the `services` `vmap` ([#135800](https://github.com/kubernetes/kubernetes/pull/135800), [#136499](https://github.com/kubernetes/kubernetes/pull/136499)), eliminated `numgen` map lookups for single-endpoint Services ([#140723](https://github.com/kubernetes/kubernetes/pull/140723), [#140960](https://github.com/kubernetes/kubernetes/pull/140960)), truncated long service comments to fit the 128-char `NFTNL_UDATA_RULE_COMMENT` limit ([#139516](https://github.com/kubernetes/kubernetes/pull/139516)), and enabled native Linux **netlink** rule listing by default via `knftables` `v0.0.22` ([#137536](https://github.com/kubernetes/kubernetes/pull/137536)).
- **Large Cluster CPU & Memory Reduction**: Skipped full periodic forced syncs in large-cluster mode ([#138571](https://github.com/kubernetes/kubernetes/pull/138571)) and stripped `metadata.managedFields` in `kube-proxy` informers ([#140056](https://github.com/kubernetes/kubernetes/pull/140056)).
- **Conntrack & Node IP Hardening**: Cleaned stale UDP conntrack entries when Services scale to 0 endpoints ([#139629](https://github.com/kubernetes/kubernetes/pull/139629)) or are deleted ([#141371](https://github.com/kubernetes/kubernetes/pull/141371)); capped `nf_conntrack_max` on high-core hosts ([#137002](https://github.com/kubernetes/kubernetes/pull/137002)) while never shrinking pre-configured higher values ([#142035](https://github.com/kubernetes/kubernetes/pull/142035)); replaced cAdvisor with `cpuset.NumCPU()` ([#140021](https://github.com/kubernetes/kubernetes/pull/140021)); automatically restarted `kube-proxy` on primary Node IP changes ([#138183](https://github.com/kubernetes/kubernetes/pull/138183)); and filtered scoped IPv6 link-local DNS servers (`fe80::%iface`) in kubelet ([#140880](https://github.com/kubernetes/kubernetes/pull/140880)).

---

## 4. Deep-Dive by SIG-NETWORK Subproject

### 4.1 Gateway API (`kubernetes-sigs/gateway-api`)
- **v1.5 (March 2026)**:
  - Graduated **`TLSRoute`** (GEP-2643), **`ListenerSet`** (GEP-1713), **`HTTPCORSFilter`** (GEP-1767), **Gateway Frontend Client Cert Validation** (GEP-91), **Gateway Backend Client Cert Selection** (GEP-3155), and **`ReferenceGrant` (`v1`)** to Standard.
  - Launched the interactive **Controller Matching Wizard** on the Hugo/Docsy documentation site.
- **v1.6 (July 2026)**:
  - Graduated **`TCPRoute`** (GEP-2644) and **`UDPRoute`** (GEP-2645) to Standard (`v1`).
  - Introduced Experimental **`XBackend`** (`v1alpha1`, GEP-4488 / GEP-4894).
- **v1.7 (October 2026)**:
  - Graduated **`HTTPRouteRetry`** (GEP-1731) and **Route Rule Name Uniqueness Validation** (#5009) to Standard.
  - Added 6 Experimental capabilities: **Gateway Address Routability** (GEP-5093), **Standardized OTel Attribute Dictionary** (GEP-5253), **`XTelemetryPolicy`** (GEP-4768), **Pre-Routing Listener Filters** (GEP-5224), **`ClusterTrustBundle` in `caCertificateRefs`** (GEP-5075), and **Session Persistence + `EndpointSelector` on `XBackend`** (GEP-1619 / GEP-4894).
- **Towards Decomposing `Service` (`XBackend` + `EndpointSelector`)**:
  - Joint effort between Gateway API and core SIG-NETWORK (KEP-6116) to decouple workload endpoint selection (`EndpointSelector`) and backend traffic policy/external FQDNs (`XBackend`) from VIP allocation (`Service`).

### 4.2 Ingress2Gateway (`kubernetes-sigs/ingress2gateway`) & `ingress-nginx` Retirement
- Following the March 2026 retirement of `kubernetes/ingress-nginx`, `ingress2gateway` shipped **v1.0.0** (Mar 2026), **v1.1.0** (Apr 2026), and **v1.2.0** (Jul 2026).
- **Provider / Emitter Split**: Decoupled input Providers (`ingress-nginx`, `gce`, `kong`, `apisix`, `istio`, `openapi`) from output Emitters.
- **5 New Implementation Emitters**: Added Emitters for **Envoy Gateway**, **kgateway**, **agentgateway**, **Traefik**, and **Airlock Microgateway** alongside the standard `gateway-api` v1.5 emitter.
- **Expanded `ingress-nginx` & GCE Translation**: Added support for `ingress-nginx` regex/path rewrites, canary weights/headers, backend TLS, `ssl-redirect`, `ssl-passthrough` (`TLSRoute`), `app-root`, `from-to-www-redirect`, timeouts, body/buffer sizes, and IP allow/deny ranges, plus GCE static IPs, session cookies, and Cloud CDN (`GCPHTTPFilter`).

### 4.3 Network Policy API (`network-policy-api`) & `kube-network-policies`
- **`ClusterNetworkPolicy` `v0.2.0` (`v1alpha2`)**: Released in April 2026 ([#369](https://github.com/kubernetes-sigs/network-policy-api/pull/369)), consolidating `AdminNetworkPolicy` and `BaselineAdminNetworkPolicy` into a single `ClusterNetworkPolicy` CRD with `Admin` and `Baseline` priority tiers.
- **Preparing for `v1beta1`**:
  - Updated **NPEP-133 (FQDN Egress Rules)**: Restricted `domainNames` rules to the `Admin` tier via CEL validation and documented DNS security guidance ([#387](https://github.com/kubernetes-sigs/network-policy-api/pull/387)).
  - Marked **`NamedPort`** as Experimental ([#392](https://github.com/kubernetes-sigs/network-policy-api/pull/392)) so dataplane scale trade-offs do not block `v1beta1` Standard graduation.
  - Clarified **Service hairpin traffic enforcement** as preferred for `v1beta1` ([#405](https://github.com/kubernetes-sigs/network-policy-api/pull/405)) and tightened CIDR/CEL validation ([#324](https://github.com/kubernetes-sigs/network-policy-api/pull/324), [#383](https://github.com/kubernetes-sigs/network-policy-api/pull/383)).
- **`kube-network-policies` (`v1.0.0` – `v1.1.2`)**: Implemented `ClusterNetworkPolicy` `v1alpha2` ([#303](https://github.com/kubernetes-sigs/kube-network-policies/pull/303)), Pod-to-Node egress policies ([#339](https://github.com/kubernetes-sigs/kube-network-policies/pull/339)), `hostNetwork` Pod skipping ([#389](https://github.com/kubernetes-sigs/kube-network-policies/pull/389)), and non-disruptive `nftables` chain updates ([#403](https://github.com/kubernetes-sigs/kube-network-policies/pull/403)).

### 4.4 DRANET (`kubernetes-sigs/dranet`) & Multi-Network
- **5 Releases in 2026 (`v1.1.0` – `v1.5.0`)**:
  - **Dynamic On-Demand Networking & IPAM**: Added Profiles + Webhook IPAM integration with **Whereabouts** ([#223](https://github.com/kubernetes-sigs/dranet/pull/223)), dynamic **IPVLAN subinterfaces** + node-local IPAM ([#241](https://github.com/kubernetes-sigs/dranet/pull/241)), and **VRF + Policy-Based Routing** ([#74](https://github.com/kubernetes-sigs/dranet/pull/74)).
  - **AI/HPC RDMA, InfiniBand & SR-IOV**: Standard `resource.kubernetes.io/numaNode` attribute ([#122](https://github.com/kubernetes-sigs/dranet/pull/122)), InfiniBand-only RDMA devices ([#77](https://github.com/kubernetes-sigs/dranet/pull/77)), IB hardware address & PKey attributes ([#343](https://github.com/kubernetes-sigs/dranet/pull/343)), and SR-IOV VF discovery + MTU validation ([#100](https://github.com/kubernetes-sigs/dranet/pull/100), [#213](https://github.com/kubernetes-sigs/dranet/pull/213)).
  - **Production Hardening**: Embedded `bbolt` Pod checkpointing with schema versioning ([#115](https://github.com/kubernetes-sigs/dranet/pull/115), [#314](https://github.com/kubernetes-sigs/dranet/pull/314)), persisted allocated device attributes in `ResourceSlice` ([#230](https://github.com/kubernetes-sigs/dranet/pull/230)), ARP (`arp_ignore`/`arp_announce`) and IPv6 `accept_ra` sysctls ([#277](https://github.com/kubernetes-sigs/dranet/pull/277), [#341](https://github.com/kubernetes-sigs/dranet/pull/341)), DHCP lease release on Pod termination ([#304](https://github.com/kubernetes-sigs/dranet/pull/304)), Helm chart, and multi-arch images.
  - **Multi-Cloud Support**: GCE IPv6/subinterfaces, AWS EFA, Azure AKS Placement/Interconnect Groups, Oracle Cloud OKE RDMA/IB fabrics, Alibaba Cloud eRDMA/Lingjun, and CoreWeave CKS.
- **Multi-Network API Re-Architecture**: Revisited `NetworkClass` / `NetworkKind` / `PodNetwork` design in July 2026 to align logical multi-network isolation with DRANET.

### 4.5 AI & Agentic Networking (`gateway-api-inference-extension` & `kube-agentic-networking`)
- **Gateway API Inference Extension (`v1.4.0` & `v1.5.0`)**:
  - Deprecated `v1alpha2` `InferencePool` in favor of `inference.networking.k8s.io/v1` ([#2937](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2937)).
  - Migrated the full production Endpoint Picker (EPP) and Body-Based Router (BBR) to **[`llm-d`](https://github.com/llm-d)** ([#2430](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2430), [#2959](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2959)) while introducing a self-contained **Lightweight EPP (`lwepp`)** ([#2842](https://github.com/kubernetes-sigs/gateway-api-inference-extension/pull/2842)) in GIE for reference and conformance testing.
- **Kube Agentic Networking (`v0.1` → `v0.2` / `v1alpha1`)**:
  - Transitioned API group to **`agentic.networking.x-k8s.io/v1alpha1`** ([#158](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/158), [#165](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/165)).
  - Added **MCP `tools/call` method, tool name, and parameter authorization** ([#180](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/180)), **CEL expression authorization** ([#188](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/188)), **`ExternalAuth` integration** ([#191](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/191)), and OTel authorization decision tracing ([#122](https://github.com/kubernetes-sigs/kube-agentic-networking/pull/122)).

### 4.6 Multi-Cluster Services (`mcs-api`) & DNS (`node-local-dns`, `external-dns`)
- **MCS-API (`kubernetes-sigs/mcs-api`)**:
  - Expanded conformance tests for headless services and `TrafficDistribution` ([#129](https://github.com/kubernetes-sigs/mcs-api/pull/129), [#134](https://github.com/kubernetes-sigs/mcs-api/pull/134)).
  - Reviewed **KEP-5939 (Multi-Cluster Service API Cluster Selection)** with SIG-NETWORK on Oct 8, 2026 to support targeting specific clusters and weighted multi-cluster traffic splitting.
- **NodeLocal DNSCache (`kubernetes-sigs/node-local-dns`)**:
  - Following SIG-NETWORK's decision to deprecate `kube-dns`, decoupled `node-local-dns` into a standalone `sigs.k8s.io/node-local-dns` module removing legacy `kube-dns`/`dnsmasq` code ([#118](https://github.com/kubernetes-sigs/node-local-dns/pull/118)) and shipped **1.26.8**, **1.27.0**, **1.27.1**, and **1.27.2** (adding metrics TLS and configurable Prometheus bind addresses).
- **ExternalDNS (`kubernetes-sigs/external-dns`)**:
  - Shipped **v0.21.0**, **v0.22.0**, and **v0.23.0** on the road to **v1.0**: migrated Gateway API sources (`Gateway`, `HTTPRoute`, `ListenerSet`, `TLSRoute`, `TCPRoute`) to `v1`, switched default annotations to GA (`external-dns.io/`), added a CRD ownership registry (`--registry=crd`) with `DNSEndpoint.status` reporting, migrated sources to client-go indexers, added `PTR`/`DNAME`/`TLSA` record support, and introduced FIPS 140-3 images.

---

## 5. 2026 SIG-NETWORK Subproject Survey Summary (22 Subprojects)

Conducted between April and June 2026 by Co-Chair Guilherme Cassolato (`@guicassolato`), the survey evaluated all 22 SIG-NETWORK subprojects:

| Category | Subprojects | Status & Action Items |
| :--- | :--- | :--- |
| **High Velocity & Growing** | `gateway-api`, `dranet`, `external-dns`, `ingress2gateway`, `network-policy-api`, `kube-network-policies`, `gateway-api-inference-extension`, `kube-agentic-networking` | Healthy multi-vendor maintainer teams, regular releases, strong KubeCon presence. |
| **Stable Core Infrastructure** | `kube-proxy`, `endpointslice`, `knftables`, `node-local-dns`, `mcs-api`, `cni` | Stable maintenance; `kube-proxy` completing `nftables` default transition; `node-local-dns` decoupled from `kube-dns`. |
| **Deprecated / Archiving** | `ingress-nginx`, `dns` (`kube-dns`), `blixt`, `kpng` | `ingress-nginx` archived March 2026; `kube-dns` deprecation agreed Jan 2026; `blixt` and `kpng` identified as inactive candidates for archival. |
| **Needs Contributor Diversity / Follow-Up** | `ingress-gce`, `ip-masq-agent`, `multi-network` | `ingress-gce` in maintenance mode as users transition to Gateway API; `multi-network` re-aligning API proposal with DRANET. |

---

## 6. Index of Generated Repository & Meeting Reports

All detailed commit-level reports and meeting summaries are available in this workspace:
- [sig-network-meeting-notes.md](sig-network-meeting-notes.md) — Chronological & thematic summary of all 2026 SIG-NETWORK biweekly meetings and linked design docs
- [kubernetes-activity.md](kubernetes-activity.md) — `kubernetes/kubernetes` SIG-NETWORK commits (v1.36–v1.38)
- [enhancements-activity.md](enhancements-activity.md) — `kubernetes/enhancements` SIG-NETWORK KEPs (2026)
- [gateway-api-activity.md](gateway-api-activity.md) — `kubernetes-sigs/gateway-api` (v1.5, v1.6, v1.7)
- [dranet-activity.md](dranet-activity.md) — `kubernetes-sigs/dranet` (v1.1.0 – v1.5.0)
- [network-policy-api-activity.md](network-policy-api-activity.md) — `kubernetes-sigs/network-policy-api` (`ClusterNetworkPolicy` v0.2.0 & v1beta1 prep)
- [ingress2gateway-activity.md](ingress2gateway-activity.md) — `kubernetes-sigs/ingress2gateway` (v1.0.0 – v1.2.0)
- [kube-network-policies-activity.md](kube-network-policies-activity.md) — `kubernetes-sigs/kube-network-policies` (v1.0.0 – v1.1.2)
- [kube-agentic-networking-activity.md](kube-agentic-networking-activity.md) — `kubernetes-sigs/kube-agentic-networking` (`v1alpha1` / MCP & CEL auth)
- [gateway-api-inference-extension-activity.md](gateway-api-inference-extension-activity.md) — `kubernetes-sigs/gateway-api-inference-extension` (v1.4 – v1.5, `llm-d` transition & `lwepp`)
- [mcs-api-activity.md](mcs-api-activity.md) — `kubernetes-sigs/mcs-api` & KEP-5939 Cluster Selection
- [node-local-dns-activity.md](node-local-dns-activity.md) — `kubernetes-sigs/node-local-dns` (1.26.8 – 1.27.2)
- [external-dns-activity.md](external-dns-activity.md) — `kubernetes-sigs/external-dns` (v0.21.0 – v0.23.0)
- [knftables-activity.md](knftables-activity.md) — `kubernetes-sigs/knftables` (v0.0.20 – v0.0.22)
