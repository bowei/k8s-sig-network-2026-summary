# ExternalDNS (`kubernetes-sigs/external-dns`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/external-dns
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 406 first-parent commits across 3 major releases (**v0.21.0**, **v0.22.0**, **v0.23.0**) and Helm chart releases (`1.20.0` – `1.23.0`), as the project prepares for **v1.0**.

---

## 1. Release Timeline in 2026

| Release | Date | Tag Commit | Key Highlights |
| :--- | :--- | :--- | :--- |
| **v0.21.0** | 2026-04-06 | `066a11e` | Gateway API `Gateway`, `HTTPRoute`, and `ListenerSet` (`v1`) support; generalized `PTR` records across all providers; controller-runtime cache for `DNSEndpoint` CRD; Istio indexed informer cache |
| **v0.22.0** | 2026-08-19 | `994f908` | Default annotation prefix switched to GA (`external-dns.alpha.kubernetes.io` → GA prefix with migration support); Gateway API `TLSRoute` `v1`; new CRD Registry option (`--registry=crd`); `DNAME` record support; indexer-based filtering across sources; unmaintained in-tree providers pruned |
| **v0.23.0** | 2026-09-18 | `a10c566` | FIPS 140-3 container image target; Cloudflare & RFC2136 `TLSA` record support; `--enable-legacy-annotation-prefix` migration flag; Gateway sources migrated to indexers; followed in late Sep/Oct by Gateway API `TCPRoute` `v1` and multi-namespace watching |

---

## 2. Gateway API `v1` Ecosystem Alignment

As Gateway API graduated `TLSRoute`, `ListenerSet`, and `TCPRoute` to Standard `v1` in 2026, ExternalDNS updated its Gateway sources to match:
- **`066a11e`** (2026-04-06, PR [#6291](https://github.com/kubernetes-sigs/external-dns/pull/6291), System-Arch) — Migrate `Gateway` and `HTTPRoute` sources to `gateway.networking.k8s.io/v1`.
- **`e33ab8d`** (2026-04-02, PR [#6254](https://github.com/kubernetes-sigs/external-dns/pull/6254), Isaac Wilson) — Add Gateway API `ListenerSet` (`v1`) support so DNS records are automatically generated for listeners attached via `ListenerSet` (and Helm chart flag `enableGatewayListenerSets` in [#6381](https://github.com/kubernetes-sigs/external-dns/pull/6381)).
- **`6e99b2c`** (2026-04-17, PR [#6367](https://github.com/kubernetes-sigs/external-dns/pull/6367), Eric Raymond) — Migrate `TLSRoute` source to `gateway.networking.k8s.io/v1`.
- **`c76b50e`** (2026-09-28, PR [#6755](https://github.com/kubernetes-sigs/external-dns/pull/6755), Pratik Wayal) — Migrate `TCPRoute` source to `gateway.networking.k8s.io/v1`.
- **`d5cf9fd`** (2026-09-26, PR [#6746](https://github.com/kubernetes-sigs/external-dns/pull/6746), KR Ravindra) — Allow a route-level hostname annotation to suppress fallback to Gateway listener hostnames.

---

## 3. Road to v1.0: GA Annotations, CRD Registry & Webhook Architecture

ExternalDNS is actively executing its **v1.0 roadmap** (noted in the 2026 SIG-NETWORK Subproject Survey):
- **GA Annotation Prefix Transition**:
  - **`ecb88cf`** (2026-05-09, PR [#6424](https://github.com/kubernetes-sigs/external-dns/pull/6424), Michel Loiseleur) — Switch default annotation prefix from `external-dns.alpha.kubernetes.io/` to GA (`external-dns.io/`).
  - **`0d5a6ac`** (2026-09-15, PR [#6703](https://github.com/kubernetes-sigs/external-dns/pull/6703), Rafael Panazzo) — Add `--enable-legacy-annotation-prefix` flag for zero-downtime migration from alpha annotations.
- **CRD Registry (`--registry=crd`) & `DNSEndpoint` Status Reporting**:
  - **`82c0233`** (2026-06-21, PR [#5372](https://github.com/kubernetes-sigs/external-dns/pull/5372), Michel Loiseleur) and **`aec5367`** (2026-08-21, PR [#6606](https://github.com/kubernetes-sigs/external-dns/pull/6606)) — Introduce a Kubernetes CRD-backed ownership registry as an alternative to TXT/DynamoDB registries.
  - **`772ed47`** (2026-09-27, PR [#6760](https://github.com/kubernetes-sigs/external-dns/pull/6760), Ivan Ka) and **`1cf6936`** (2026-10-20, PR [#6767](https://github.com/kubernetes-sigs/external-dns/pull/6767), Michel Loiseleur) — Report DNS synchronization outcomes directly to `DNSEndpoint.status`.
  - **`020d384`** (2026-10-01, PR [#6620](https://github.com/kubernetes-sigs/external-dns/pull/6620), Michel Loiseleur) — Validate `DNSEndpoint` schema at admission via CRD validation rules.
- **Multi-Namespace Watching & Scale Optimizations**:
  - **`f623f09`** (2026-10-01, PR [#6763](https://github.com/kubernetes-sigs/external-dns/pull/6763)) and **`52dedbe`** (2026-10-20, PR [#6764](https://github.com/kubernetes-sigs/external-dns/pull/6764), Michel Loiseleur) — Support watching multiple specific namespaces (`--namespace=ns1,ns2`) instead of only a single namespace or cluster-wide.
  - **Indexer-Based Informer Caching**: Migrated `Service` ([#6344](https://github.com/kubernetes-sigs/external-dns/pull/6344)), `Ingress`/`Node` ([#6445](https://github.com/kubernetes-sigs/external-dns/pull/6445)), `Istio` ([#6303](https://github.com/kubernetes-sigs/external-dns/pull/6303)), `Gloo` ([#6515](https://github.com/kubernetes-sigs/external-dns/pull/6515)), `Contour`/`F5`/`Kong`/`Traefik` ([#6542](https://github.com/kubernetes-sigs/external-dns/pull/6542)), and `Gateway API` ([#6545](https://github.com/kubernetes-sigs/external-dns/pull/6545)) sources to client-go indexers, dramatically reducing memory and CPU overhead in large clusters.
- **New DNS Record Types & Security Hardening**:
  - Generalized `PTR` record support across all providers ([#6232](https://github.com/kubernetes-sigs/external-dns/pull/6232), `736a2d5`), added `DNAME` record support ([#6553](https://github.com/kubernetes-sigs/external-dns/pull/6553), `eca6bf7`), and added `TLSA` record support for Cloudflare ([#6616](https://github.com/kubernetes-sigs/external-dns/pull/6616), `8fc57fe`) and RFC2136 ([#6721](https://github.com/kubernetes-sigs/external-dns/pull/6721), `554dec7`).
  - Added FIPS 140-3 compliant container image builds ([#6657](https://github.com/kubernetes-sigs/external-dns/pull/6657), `1d27574`) and hardened webhook container security defaults ([#6785](https://github.com/kubernetes-sigs/external-dns/pull/6785), `979e3fe`).
  - Removed unmaintained in-tree providers (`akamai`, `plural`, `transip` in [#6485](https://github.com/kubernetes-sigs/external-dns/pull/6485); `gandi` in [#6654](https://github.com/kubernetes-sigs/external-dns/pull/6654); Pi-hole v5 in [#6406](https://github.com/kubernetes-sigs/external-dns/pull/6406)) while adding community out-of-tree webhook providers (Akamai Edge DNS, Infoblox UDDI, Namecheap, NAVER Cloud, Porkbun, Tencent Cloud).
