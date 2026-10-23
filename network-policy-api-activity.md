# Network Policy API (`kubernetes-sigs/network-policy-api`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/network-policy-api
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 67 commits (34 first-parent PR merges), highlighted by the **v0.2.0** release of `ClusterNetworkPolicy` (`v1alpha2`) and ongoing preparation for `v1beta1`.

---

## 1. `ClusterNetworkPolicy` `v0.2.0` Release (`v1alpha2`)

Following the architectural consolidation of `AdminNetworkPolicy` (`ANP`) and `BaselineAdminNetworkPolicy` (`BANP`) into a single unified `ClusterNetworkPolicy` (`CNP`) resource with tiered priorities (`Admin` and `Baseline`), the subproject shipped **v0.2.0** in April 2026:
- **`fbbf459` / `a00dd28`** (2026-04-21, PR [#369](https://github.com/kubernetes-sigs/network-policy-api/pull/369), Nadia Pinaeva) — Cut **v0.2.0** release (`ClusterNetworkPolicy` `v1alpha2` standard + experimental CRDs).
- **`6f61f84` / `6fe1568`** (2026-03-04, PR [#349](https://github.com/kubernetes-sigs/network-policy-api/pull/349), Nadia Pinaeva) — Update user documentation, diagrams, and examples for `v1alpha2` `ClusterNetworkPolicy`.

---

## 2. Road to `ClusterNetworkPolicy` `v1beta1` & NPEP Updates

In the second half of 2026, the maintainers focused on resolving API edge cases, tightening CEL validation, and finalizing NPEPs ahead of promoting `ClusterNetworkPolicy` to `v1beta1`:
- **NPEP-133 — FQDN Egress Rules (`domainNames`) Refinement**:
  - **`75d34c2` / `85389ab`** (2026-08-19, PR [#387](https://github.com/kubernetes-sigs/network-policy-api/pull/387), Nadia Pinaeva) — Update NPEP-133 and `ClusterNetworkPolicy` spec: restrict FQDN egress rules (`domainNames`) to the `Admin` tier via CEL validation, clarify wildcard matching semantics, and document DNS spoofing / CNAME security considerations.
- **NPEP-379 Updates**:
  - **`541993e` / `0bb14dc`** (2026-09-29, PR [#393](https://github.com/kubernetes-sigs/network-policy-api/pull/393), Hunter) — Update NPEP-379 design specification.
- **Service Hairpin Traffic Enforcement Guidance**:
  - **`e7fa475` / `c32e8a0`** (2026-09-25, PR [#405](https://github.com/kubernetes-sigs/network-policy-api/pull/405), Nadia Pinaeva) — Clarify that enforcing `ClusterNetworkPolicy` on Service hairpin traffic (when a Pod connects to its own IP via a Service VIP) is preferred for `v1beta1`.
- **Move `NamedPort` to Experimental Channel**:
  - **`d60ca84` / `68cfac8`** (2026-09-21, PR [#392](https://github.com/kubernetes-sigs/network-policy-api/pull/392), Nadia Pinaeva) — Mark `NamedPort` support on `ClusterNetworkPolicy` as an Experimental channel feature so it does not block Standard `v1beta1` graduation for eBPF/datapath implementations where named port resolution has scale trade-offs.
- **Strict CIDR Validation & CEL Test Suite**:
  - **`6dcdfbd` / `a8d1a8d`** (2026-10-08, PR [#324](https://github.com/kubernetes-sigs/network-policy-api/pull/324), Tanujdargan) — Fix CIDR validation on `networks` peers to enforce valid prefix masks.
  - **`6894b21` / `ef6d055`** (2026-08-26, PR [#383](https://github.com/kubernetes-sigs/network-policy-api/pull/383), Nadia Pinaeva) — Add comprehensive CRD CEL validation unit tests and tighten `ClusterNetworkPolicy` CEL rules.

---

## 3. Conformance Suite Expansion

The `ClusterNetworkPolicy` conformance suite gained substantial test coverage in 2026:
- **`1ff031d` / `8a7a78e`** (2026-03-31, PR [#364](https://github.com/kubernetes-sigs/network-policy-api/pull/364), Hunter) — Add conformance tests for multiple `ClusterNetworkPolicy` resources with the same priority value.
- **`07202de` / `998a5f3`** (2026-05-06, PR [#371](https://github.com/kubernetes-sigs/network-policy-api/pull/371), Hunter) — Add conformance test verifying that ingress-only policies do not inadvertently isolate egress traffic (and vice versa).
- **`6c30a15` / `729c113`** (2026-05-26, PR [#376](https://github.com/kubernetes-sigs/network-policy-api/pull/376), Hunter) — Add conformance tests for port ranges and multiple port protocols (`TCP`, `UDP`, `SCTP`).
- **`7e62bd9` / `a252a6b`** (2026-05-28, PR [#373](https://github.com/kubernetes-sigs/network-policy-api/pull/373), Hunter) — Add conformance tests for dynamic Pod and Namespace label mutations.
- **`b571316` / `eb2a97e`** (2026-05-28, PR [#368](https://github.com/kubernetes-sigs/network-policy-api/pull/368), Hunter) — Add conformance tests for rule mutation and rule ordering within a policy.
- **`bf90138` / `7162598`** (2026-03-26, PR [#363](https://github.com/kubernetes-sigs/network-policy-api/pull/363), Nadia Pinaeva) — Fix conformance assertions to require traffic drop from all client source ports.
