# Kubernetes Core (`kubernetes/kubernetes`) — SIG-NETWORK Activity (2026)

- **Repository**: https://github.com/kubernetes/kubernetes
- **Period Covered**: January 1, 2026 – October 22, 2026 (v1.36, v1.37, and v1.38 release cycles)
- **Total Commits in Period**: 4,296 commits (2,143 first-parent merges); filtered below for significant SIG-NETWORK features, graduations, architectural changes, and reliability fixes.

---

## 1. Feature Graduations, KEPs & Deprecations

### `kube-proxy` nftables Localhost NodePort Userspace Proxy (KEP-6032)
When `kube-proxy` switched from `iptables` to `nftables`, one remaining compatibility gap was traffic sent from the node itself to `127.0.0.1:<nodePort>` (since Linux `nftables` cannot DNAT packets originating from and destined to `127.0.0.0/8` out to Pod IPs without kernel routing workarounds). KEP-6032 introduced an opt-in lightweight userspace proxy inside `kube-proxy`'s `nftables` backend specifically for localhost NodePort connections, plus `--nodeport-addresses` keyword improvements (`localhost`, `all`).
- **`7b8fb5ef249`** (2026-06-15, PR [#138427](https://github.com/kubernetes/kubernetes/pull/138427), Dan Winship) — Implement `nftables` localhost NodePort userspace proxy (Alpha in v1.37).
- **`9db3c1074a3`** (2026-06-15, PR [#138427](https://github.com/kubernetes/kubernetes/pull/138427), Dan Winship) — Allow `"localhost"` and `"all"` as keywords in `--nodeport-addresses`.
- **`beab33575fe`** (2026-06-15, PR [#138427](https://github.com/kubernetes/kubernetes/pull/138427), Dan Winship) — Add e2e tests for `nftables` localhost NodePorts.
- **`451ab1d9b7c`** (2026-09-25, PR [#142138](https://github.com/kubernetes/kubernetes/pull/142138), Dan Winship) — Fix `kube-proxy` detection of explicit localhost NodePorts.
- **`6029cad6874`** (2026-10-16, PR [#142490](https://github.com/kubernetes/kubernetes/pull/142490), Dan Winship) — Promote `NFTablesLocalhostNodePorts` to Beta (enabled by default in v1.38), skipping userspace proxying when `externalTrafficPolicy: Local` has no local endpoints.

### Strict IP & CIDR Validation (KEP-4858)
Tightens Kubernetes API validation of IP addresses and CIDRs across networking resources (rejecting leading zeros, ambiguous formats, and non-canonical representations where appropriate).
- **`3b167a6992c`** (2026-09-18, PR [#141979](https://github.com/kubernetes/kubernetes/pull/141979), Dan Winship) — Promote `StrictIPCIDRValidation` to GA in v1.38.

### Relaxed Service Name Validation (KEP-5311) & Arbitrary FQDN Pod Hostnames (KEP-4762)
- **`598b6091f6f`** (2026-06-16, PR [#139282](https://github.com/kubernetes/kubernetes/pull/139282), Tim Hockin) — Promote `RelaxedServiceNameValidation` to GA in v1.37, allowing DNS label service names (such as numeric-leading names) rather than strictly RFC 1035 names.
- **`083b28d8a68`** (2026-06-17, PR [#139217](https://github.com/kubernetes/kubernetes/pull/139217), Tim Hockin) — Remove GA'd `RelaxedDNSSearchValidation` feature gate (KEP-4427).

### Deprecations: `Service.spec.externalIPs` (KEP-5707), `v1.Endpoints` (KEP-4974), and `kube-proxy` IPVS Mode (KEP-5495)
- **`80c6507ccef` / `547b17cc13e`** (2026-03-18, PR [#137293](https://github.com/kubernetes/kubernetes/pull/137293), Adrian Moisey) — KEP-5707: Deprecate `Service.spec.externalIPs` with API warnings and documentation updates.
- **`9fa4c1cfa3d` / `caad3e56db3`** (2026-07-08, PR [#139397](https://github.com/kubernetes/kubernetes/pull/139397), Dan Winship) — KEP-5495: Add feature gate for IPVS mode in `kube-proxy` as part of formal IPVS deprecation.
- **`e2ed47e2de0` / `28f1f4048eb`** (2026-07-23, PR [#134860](https://github.com/kubernetes/kubernetes/pull/134860), Rob Scott) — KEP-4974: Migrate `kube-apiserver` `service/proxy` subresource from legacy `v1.Endpoints` to `discovery.k8s.io/v1.EndpointSlice`.
- **`36e93204ed1` / `169a07b2ec8`** (2026-03-19, PR [#137553](https://github.com/kubernetes/kubernetes/pull/137553), Dan Winship) — Verify in e2e tests that cluster DNS resolves Services even when legacy `Endpoints` objects do not exist (pure `EndpointSlice` consumption).

### Feature Gate Cleanups (GA Lock-in)
- **`f278b47ecd3` / `d68d48073f9`** (2026-01-13, PR [#136112](https://github.com/kubernetes/kubernetes/pull/136112), Dan Winship) — Drop `TopologyAwareHints` and `ServiceTrafficDistribution` feature gates.
- **`73b08be6346` / `b0030b8d19b`** (2026-09-09, PR [#141733](https://github.com/kubernetes/kubernetes/pull/141733), Gaurav) — Remove GA'd `PreferSameTrafficDistribution` feature gate.
- **`b4b4a788790` / `5c40e12d429`** (2026-09-10, PR [#141762](https://github.com/kubernetes/kubernetes/pull/141762), cyclinder) — Remove `MultiCIDRServiceAllocator` and `DisableAllocatorDualWrite` feature gates.

---

## 2. `kube-proxy` `nftables` Architecture, Performance & Netlink Integration

In preparation for making `nftables` the default `kube-proxy` backend (KEP-5343), SIG-NETWORK significantly simplified the generated `nftables` ruleset and switched `knftables` to use netlink communication for rule listing.
- **`af82d41a6cc` / `e941625d72b`** (2026-07-23, PR [#137536](https://github.com/kubernetes/kubernetes/pull/137536), Antonio Ojea) — Bump `knftables` to `v0.0.22` and enable netlink support for `kube-proxy` `nftables` mode by default.
- **`7cdeb113279` / `5cffb4d1f67` / `e17963cb99d` / `475f9622c8e`** (2026-01-24, PR [#135800](https://github.com/kubernetes/kubernetes/pull/135800), Dan Winship) — Rework `nftables` masquerading and hairpin handling (Part 1): centralize hairpin/masquerading checks and merge `service-endpoints-check` into `service-ips`.
- **`05c01a2e807` / `aa3a30d134e`** (2026-03-07, PR [#136499](https://github.com/kubernetes/kubernetes/pull/136499), Dan Winship) — Rework `nftables` masquerading (Part 2): squash per-service endpoint chains directly into the `services` `vmap`, cutting chain hops and simplifying rule dumps.
- **`90904cf220e` / `32a7f9aa3e6`** (2026-08-19, PR [#140723](https://github.com/kubernetes/kubernetes/pull/140723), Dan Winship) and **`8f19f1ec934` / `9df7b5086a2`** (2026-08-21, PR [#140960](https://github.com/kubernetes/kubernetes/pull/140960), Dan Winship) — Optimize `nftables` service rules by performing direct `dnat` instead of `numgen` map lookups when a Service has only 1 endpoint.
- **`6070d2808d6` / `bc7a40ff9fa`** (2026-06-29, PR [#139516](https://github.com/kubernetes/kubernetes/pull/139516), Frode Nordahl) — Truncate long service comments in `nftables` mode to avoid hitting the 128-character `NFTNL_UDATA_RULE_COMMENT` limit.
- **`04d82d95fa3`** (2026-02-12, PR [#136796](https://github.com/kubernetes/kubernetes/pull/136796), kairosci) and **`a3ac5144e7d`** (2026-03-12, PR [#137501](https://github.com/kubernetes/kubernetes/pull/137501), Dan Winship) — Fix `kube-proxy` compatibility with `nft` 1.1.3+ JSON output and `udata` handling.

---

## 3. Scale, Memory, Conntrack & Operational Reliability

### Large Cluster Syncs & Memory Footprint
- **`7d3b347d200` / `2e3c3df92bb`** (2026-06-11, PR [#138571](https://github.com/kubernetes/kubernetes/pull/138571), Dan Winship) — Skip full periodic forced syncs in `kube-proxy` when running in large-cluster mode, avoiding CPU spikes across thousands of Services while keeping incremental syncs fast.
- **`25b452b9719`** (2026-01-09, PR [#135631](https://github.com/kubernetes/kubernetes/pull/135631), Dan Winship) — Switch `ipvs` and `winkernel` proxiers back to regular forced syncs where incremental guarantees differ from `iptables`/`nftables`.
- **`5d6c0c7f972` / `0df38b27fc6`** (2026-07-16, PR [#140056](https://github.com/kubernetes/kubernetes/pull/140056), specificlanguage) — Strip `metadata.managedFields` in `kube-proxy` informers to significantly reduce memory consumption in large clusters.

### Conntrack Management & UDP Stale Flow Cleanup
- **`6175c54954f` / `65564e20774`** (2026-03-18, PR [#137002](https://github.com/kubernetes/kubernetes/pull/137002), kairosci) — Cap `nf_conntrack_max` limits on high-core-count machines to prevent excessive kernel conntrack memory allocation.
- **`4fd72d50ca5` / `cf46ba8e2dd`** (2026-10-08, PR [#142035](https://github.com/kubernetes/kubernetes/pull/142035), Maciej Skrocki) — Avoid shrinking `nf_conntrack_max` if the node administrator has already configured a higher sysctl value.
- **`04d5d2647dc` / `614a0c3513a`** (2026-07-18, PR [#140021](https://github.com/kubernetes/kubernetes/pull/140021), Davanum Srinivas) — Replace heavy cAdvisor dependency with `cpuset.NumCPU()` in `kube-proxy` for conntrack sizing calculations.
- **`dac71e780a1` / `54ec15ac064`** (2026-07-02, PR [#139629](https://github.com/kubernetes/kubernetes/pull/139629), Liang Li) — Clean up stale conntrack entries when a UDP Service scales down to zero endpoints.
- **`a44fc0805c7` / `81c0d98cc4b`** (2026-08-29, PR [#141371](https://github.com/kubernetes/kubernetes/pull/141371), Liang Li) — Clean up stale conntrack entries when a UDP Service is deleted completely.
- **`7e1c4a9c698`** (2026-04-28, PR [#136841](https://github.com/kubernetes/kubernetes/pull/136841), Arujjwal Negi) — Clean up stale conntrack logic and simplify conntrack filter matching.

### Node IP Changes, Dual-Stack, IPAM & Kubelet DNS
- **`52a35b12ff3` / `cf2c2390be2`** (2026-04-27, PR [#138183](https://github.com/kubernetes/kubernetes/pull/138183), Dan Winship) — Automatically restart `kube-proxy` if the local Node's primary IP address changes.
- **`a35a8c1a36c` / `2df72245be4`** (2026-08-27, PR [#140880](https://github.com/kubernetes/kubernetes/pull/140880), Maciej Skrocki) — Filter out scoped IPv6 link-local DNS servers (`fe80::...%iface`) in kubelet when constructing Pod `/etc/resolv.conf`, since interface zone indices from the host network namespace are invalid inside Pod network namespaces.
- **`958c10e37e5`** (2026-03-12, PR [#137370](https://github.com/kubernetes/kubernetes/pull/137370), Nordix) — Fix `kube-proxy` healthcheck server double-bind failure in dual-stack configurations.
- **`6a20aa630d8`** (2026-01-21, PR [#136241](https://github.com/kubernetes/kubernetes/pull/136241), Prince Pereira) — Fix `PreferDualStack` and `RequireDualStack` handling in the Windows `winkernel` proxier.
- **`0ba7bf57165`** (2026-02-13, PR [#135503](https://github.com/kubernetes/kubernetes/pull/135503), Antonio Ojea) — Fix `RepairIPAddress` controller when a Service transitions from `ExternalName` to `ClusterIP`.
- **`887929bb954`** (2026-02-27, PR [#137147](https://github.com/kubernetes/kubernetes/pull/137147), haojiwu) — Fix `RepairIPAddress` controller startup failure when the namespace informer has not yet synced.
- **`cb700c08f75`** (2026-04-20, PR [#137632](https://github.com/kubernetes/kubernetes/pull/137632), Davanum Srinivas) — Prevent panic in `RangeAllocator` during Node CIDR updates.
- **`944b5db8bb0` / `23f539e0a6e`** (2026-10-16, PR [#140654](https://github.com/kubernetes/kubernetes/pull/140654), Tim Hockin) — Allow `HostAliases` entries in Pod specs to contain trailing dots in FQDNs (KEP-6312).

---

## 4. Declarative Validation (KEP-5073) for Networking APIs

Throughout 2026, SIG-NETWORK and SIG-API-Machinery migrated `networking.k8s.io` (`Ingress`, `IngressClass`, `NetworkPolicy`, `ServiceCIDR`, `IPAddress`) and `core/v1` `Service` validation rules to Declarative Validation (`+k8s:optional`, `+k8s:required`, `+k8s:immutable`, `+k8s:format`, `+k8s:listType=atomic`):
- **`1c24a05ce16`** (2026-01-08, PR [#134909](https://github.com/kubernetes/kubernetes/pull/134909), Darshan Murthy) — Wire `networking` API group for declarative validation (`IngressClassParametersReference`).
- **`f067ad1e8f8`** (2026-02-16, PR [#135715](https://github.com/kubernetes/kubernetes/pull/135715), Darshan Sreenivas) — Migrate `NetworkPolicy` `ipBlock.cidr` validation to Declarative Validation.
- **`6457593fdf3`** (2026-02-25, PR [#136215](https://github.com/kubernetes/kubernetes/pull/136215), darthsuburbus) — Migrate `IPAddress.spec.parentRef` validation to Declarative Validation.
- **`51fb13bc7ba`** (2026-06-29, PR [#138262](https://github.com/kubernetes/kubernetes/pull/138262), Priyanka Saggu) — Migrate `ServiceSpec` validation (`ipFamilies`, `ipFamilyPolicy`, `clusterIPs`) to Declarative Validation.
- **`3b6a64c29ec`** (2026-06-30, PR [#136207](https://github.com/kubernetes/kubernetes/pull/136207)) — Add declarative validation for `Ingress` and `IngressClass` specs.
- **`f56d902e6fd`** (2026-09-25, PR [#142093](https://github.com/kubernetes/kubernetes/pull/142093), RoboMan2000) — Migrate `ServiceCIDR` to Declarative Validation (`+k8s:immutable`, `+k8s:maxItems=2`, `+k8s:listType=atomic`).
- **`6c8a863a8eb`** (2026-10-08, PR [#141895](https://github.com/kubernetes/kubernetes/pull/141895), RoboMan2000) — Add declarative validation tags (`+k8s:optional`, `+k8s:required`, `+k8s:format=k8s-ResourcePoolName`) to `networking` APIs.
