# Gateway API (`kubernetes-sigs/gateway-api`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/gateway-api
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 499 commits across 3 major minor releases (**v1.5**, **v1.6**, **v1.7**) and 3 monthly experimental releases (`monthly-2026.01`, `monthly-2026.05`, `monthly-2026.06`).

---

## 1. Release Timeline & Process Overhaul (2026)

In early 2026, the Gateway API maintainers overhauled the release process:
1. **Release-Train Cadence**: Decoupled the feature development pipeline from fixed release dates so features ship on the next scheduled minor release train once ready, eliminating reviewer bottlenecks.
2. **Monthly Experimental Releases**: Shipped `monthly-2026.01` (`f00ab7b`), `monthly-2026.05` (`bc52fd5`), and `monthly-2026.06` (`64ff513`) so experimental APIs could be consumed and tested by implementations between minor releases.
3. **Three Minor Releases in 2026**:
   - **v1.5.0** (`a6ca875`, March 11, 2026) & **v1.5.1** (`7345253`, March 19, 2026)
   - **v1.6.0** (`1e68f90`, July 6, 2026)
   - **v1.7.0** (`v1.7.0-rc1` `e793e3e`, October 16, 2026; SIG-NETWORK API review completed October 8–22, 2026)

---

## 2. Gateway API v1.5 (March 2026) — Major Graduations to Standard

Gateway API v1.5 was one of the largest Standard channel graduations in the project's history, promoting 6 major APIs/features from Experimental to Standard (`v1`):

- **`TLSRoute` Graduated to Standard (`v1`) (GEP-2643)**:
  - **`507036d`** (2026-01-13, PR [#4450](https://github.com/kubernetes-sigs/gateway-api/pull/4450), Shane Utt) — GEP-2643 updates for `TLSRoute` promotion to `v1`.
  - **`ff35a08`** (2026-01-20, PR [#4439](https://github.com/kubernetes-sigs/gateway-api/pull/4439), Candace Holman) — Promote `TLSRoute` to `gateway.networking.k8s.io/v1` and Standard channel.
  - **`b6e588e`** (2026-02-04, PR [#4561](https://github.com/kubernetes-sigs/gateway-api/pull/4561), Candace Holman) — Enforce `TLSRoute` Hostname validation via CEL in `v1`.
- **`ListenerSet` Graduated to Standard (`v1`) (GEP-1713)**:
  - **`b57aa73`** (2026-01-23, PR [#4499](https://github.com/kubernetes-sigs/gateway-api/pull/4499), Dave Protasowski) — Promote `XListenerSet` to `ListenerSet` in `gateway.networking.k8s.io/v1`, enabling multi-team listener delegation and scaling beyond the 64-listener Gateway limit.
  - **`ba0b8de`** (2026-01-13, PR [#4461](https://github.com/kubernetes-sigs/gateway-api/pull/4461), Steven Jin) — Add `ListenerSet` conformance tests.
- **`HTTPCORSFilter` (CORS) Graduated to Standard (GEP-1767)**:
  - **`6dc7097`** (2026-02-04, PR [#4521](https://github.com/kubernetes-sigs/gateway-api/pull/4521), Beka Modebadze) — Graduate `HTTPRoute` `cors` filter (`HTTPCORSFilter`) to Standard channel.
- **Gateway Frontend Client Certificate Validation (mTLS) Graduated to Standard (GEP-91)**:
  - **`c9193ec`** (2026-01-31, PR [#4496](https://github.com/kubernetes-sigs/gateway-api/pull/4496), Matei David) — Promote Gateway frontend TLS client certificate validation (`FrontendTLSConfig`) to Standard.
- **Gateway Backend Client Certificate Selection Graduated to Standard (GEP-3155)**:
  - **`49663c7`** (2026-02-04, PR [#4489](https://github.com/kubernetes-sigs/gateway-api/pull/4489), Mkriel) — Graduate `BackendTLSPolicy` Gateway client certificate configuration to Standard.
- **`ReferenceGrant` Graduated to `v1`**:
  - **`db0d8e0`** (2026-02-04, PR [#4458](https://github.com/kubernetes-sigs/gateway-api/pull/4458), Rob Scott) — Promote `ReferenceGrant` from `v1beta1` to `v1`.

---

## 3. Gateway API v1.6 (July 2026) — `TCPRoute` & `UDPRoute` to Standard, `XBackend` Experimental

- **`TCPRoute` and `UDPRoute` Graduated to Standard (`v1`) (GEP-2644 & GEP-2645)**:
  - **`3c0056d`** (2026-04-28, PR [#4835](https://github.com/kubernetes-sigs/gateway-api/pull/4835), Shane Utt) — GEP-2644 & GEP-2645 updates for `TCPRoute` and `UDPRoute` graduation.
  - **`72ec3aa`** (2026-05-05, PR [#4842](https://github.com/kubernetes-sigs/gateway-api/pull/4842), Shane Utt) — Promote `TCPRoute` to `gateway.networking.k8s.io/v1` and Standard channel.
  - **`85602fc`** (2026-05-05, PR [#4843](https://github.com/kubernetes-sigs/gateway-api/pull/4843), Shane Utt) — Promote `UDPRoute` to `gateway.networking.k8s.io/v1` and Standard channel.
  - **`1fa306a`**, **`05cbba9`**, **`631acbe`** (May–Jun 2026, PRs [#4939](https://github.com/kubernetes-sigs/gateway-api/pull/4939), [#4971](https://github.com/kubernetes-sigs/gateway-api/pull/4971), [#5012](https://github.com/kubernetes-sigs/gateway-api/pull/5012), Utkarsh Chaurasia / Benjamin / Shane Utt) — Expand `TCPRoute` and `UDPRoute` conformance test suites (port-matching, cross-namespace, listener binding).
- **Experimental `XBackend` Resource & Decomposing Service (GEP-4488 / GEP-4894)**:
  - **`a5d54fd`** (2026-04-16, PR [#4488](https://github.com/kubernetes-sigs/gateway-api/pull/4488), Rob Scott) — Initial `XBackend` GEP for decomposing Service backend properties and supporting external FQDN/IP backends.
  - **`0120780`** (2026-05-14, PR [#4901](https://github.com/kubernetes-sigs/gateway-api/pull/4901), John Howard) — Implement Experimental `XBackend` (`v1alpha1`) CRD and Go types.

---

## 4. Gateway API v1.7 (October 2026) — `HTTPRoute` Retries to Standard & 6 New Experimental Capabilities

Reviewed by SIG-NETWORK on October 8 & October 22, 2026:

### Graduations to Standard Channel
- **`HTTPRoute` Retries (`HTTPRouteRetry`) Graduated to Standard (GEP-1731)**:
  - **`be5e39e`** (2026-09-23, PR [#5350](https://github.com/kubernetes-sigs/gateway-api/pull/5350), Snigdha) — Promote `HTTPRouteRetry` (`attempts`, `backoff`, `codes`) to Standard channel.
  - **`c1f3502`** (2026-09-25, PR [#5347](https://github.com/kubernetes-sigs/gateway-api/pull/5347), Snigdha) — Graduate `HTTPRoute` retry fields in API types and CRDs.
  - **`2420da6`** (2026-09-14, PR [#4560](https://github.com/kubernetes-sigs/gateway-api/pull/4560), Gaurav) — Add conformance tests for `HTTPRoute` retry.
- **Route Rule Name Uniqueness Validation**:
  - **`3c2330a`** (2026-06-12, PR [#5009](https://github.com/kubernetes-sigs/gateway-api/pull/5009), Eyal Pazz) — Promote CEL validation enforcing unique `RouteRule.name` entries within a Route from Experimental to Standard (`HTTPRoute`, `GRPCRoute`, `TLSRoute`, `TCPRoute`, `UDPRoute`).

### New Experimental Features in v1.7
1. **Gateway Address Routability (`Routability`) (GEP-5093)**:
   - **`8c5edef`** (2026-07-02, PR [#5094](https://github.com/kubernetes-sigs/gateway-api/pull/5094), Rob Scott) and **`1f98a2d`** (2026-09-24, PR [#5304](https://github.com/kubernetes-sigs/gateway-api/pull/5304), Edge) — Allows users and controllers to specify address routability (`Public`, `Cluster`, `IntraCluster`, etc.) on `GatewaySpecAddress` and `GatewayStatusAddress`, replacing ambiguous cloud-specific load balancer annotations.
2. **Standardized OpenTelemetry Attribute Dictionary (GEP-5253)**:
   - **`ac43b9f`** (2026-09-09, PR [#5256](https://github.com/kubernetes-sigs/gateway-api/pull/5256), Steven Jin) — Standardizes OpenTelemetry attribute names (`k8s.gateway.*`, route names, matched rules) across all Gateway API implementations.
3. **`XTelemetryPolicy` for Distributed Tracing (GEP-4768)**:
   - **`dc161dc`** (2026-08-29, PR [#4872](https://github.com/kubernetes-sigs/gateway-api/pull/4872), Gaurav) and **`bb1e5e3`** (2026-09-25, PR [#5308](https://github.com/kubernetes-sigs/gateway-api/pull/5308), Gaurav) — Introduces `XTelemetryPolicy` (`v1alpha1`) for configuring distributed tracing sampling and upstream `BackendRef` telemetry exporters on Gateways and ListenerSets.
4. **Pre-Routing Listener Filters (GEP-5224)**:
   - **`44eb7bb`** (2026-08-24, PR [#5235](https://github.com/kubernetes-sigs/gateway-api/pull/5235), Steven Jin), **`75d9de2`** (2026-09-18, PR [#5315](https://github.com/kubernetes-sigs/gateway-api/pull/5315), Steven Jin), and **`ff32a27`** (2026-09-28, PR [#5367](https://github.com/kubernetes-sigs/gateway-api/pull/5367), Steven Jin) — Adds Listener-level filters on `Gateway` and `ListenerSet` that execute before route matching (enabling WAF, GeoIP, early authentication, and rate limiting).
5. **`ClusterTrustBundle` Support in `caCertificateRefs` (GEP-5075)**:
   - **`e1df436`** (2026-09-08, PR [#5259](https://github.com/kubernetes-sigs/gateway-api/pull/5259), Aaryan Porwal) and **`5fc2e84`** (2026-09-23, PR [#5338](https://github.com/kubernetes-sigs/gateway-api/pull/5338), Aaryan Porwal) — Enables referencing `certificates.k8s.io` `ClusterTrustBundle` objects in `BackendTLSPolicy` and Gateway `FrontendTLSValidation` `caCertificateRefs`.
6. **Session Persistence & `EndpointSelector` on `XBackend` (GEP-1619 / GEP-4894)**:
   - **`d864526`** (2026-09-23, PR [#5298](https://github.com/kubernetes-sigs/gateway-api/pull/5298), Xunzhuo) — Adds `sessionPersistence` configuration to `XBackend`.
   - **`98b0e55`** (2026-09-14, PR [#5255](https://github.com/kubernetes-sigs/gateway-api/pull/5255), John Howard) — Adds `EndpointSelector` internal destinations to `XBackend`.

---

## 5. Additional GEPs, Conformance & Documentation Improvements

- **Interactive Controller Matching Wizard & Hugo/Docsy Website Migration**:
  - **`b3446ca`** (2026-02-23, PR [#4584](https://github.com/kubernetes-sigs/gateway-api/pull/4584), Shane Utt) — Launches an interactive Controller Matching Wizard on the Gateway API website powered by conformance report profiles so users can filter controllers by supported features.
  - **`634c4de`** (2026-04-16, PR [#4734](https://github.com/kubernetes-sigs/gateway-api/pull/4734), Shane Utt) — Migrates the Gateway API documentation site from MkDocs to Hugo/Docsy (`kubernetes.io` tooling).
- **In-Progress GEPs Merged in 2026**:
  - **GEP-3949 — ListenerSet Reference Handshake (`AllowedListeners` / `AllowedRoutes`)**: **`53d64b9`** (2026-02-13, PR [#4164](https://github.com/kubernetes-sigs/gateway-api/pull/4164), Snigdha).
  - **GEP-1748 — `HTTPRoute` Query Parameter Matching Guidance**: **`e9d1c7c`** (2026-09-29, PR [#5322](https://github.com/kubernetes-sigs/gateway-api/pull/5322), Edge).
  - **GEP-3793 — Mutual Authentication on `BackendTLSPolicy` Status Update**: **`0b703eb`** (2026-08-29, PR [#5262](https://github.com/kubernetes-sigs/gateway-api/pull/5262), Rouf).
- **Conformance Suite Enhancements**:
  - Added conformance reports for Gateway API v1.5 and v1.6 across 15+ implementations (Envoy Gateway, Cilium, Istio, kgateway, Kong, NGINX Gateway Fabric, Traefik, Airlock Microgateway, HAProxy, LoxiLB, Agentgateway, etc.).
  - Added `AllowEmptyResources` conformance suite option (`518ffab`, PR [#5393](https://github.com/kubernetes-sigs/gateway-api/pull/5393)) and refactored the runner away from global flags (`d675c10`, PR [#5079](https://github.com/kubernetes-sigs/gateway-api/pull/5079)).
