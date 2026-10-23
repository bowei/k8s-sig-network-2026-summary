# DRANET (`kubernetes-sigs/dranet`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/dranet
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 165 commits across 5 feature releases (**v1.1.0**, **v1.2.0**, **v1.3.0**, **v1.4.0**, **v1.5.0**) and patch releases.

---

## 1. Release Timeline in 2026

| Release | Date | Tag Commit | Key Highlights |
| :--- | :--- | :--- | :--- |
| **v1.1.0** | 2026-03-04 | `318f824` | Standard `numaNode` DRA attribute, SR-IOV VF attribute discovery, IB-only RDMA devices, Oracle Cloud OKE & CoreWeave CKS guides |
| **v1.2.0** | 2026-04-06 | `29f8f9a` | VRF & Policy-Based Routing support, `bbolt` persistent Pod device checkpointing, Azure AKS & Alibaba Cloud Lingjun/eRDMA guides |
| **v1.3.0** | 2026-05-05 | `9405ca7` | Persisted allocated device attributes via `ResourceSlice`, GCE IPv6 & subinterfaces, multi-arch images (`amd64`/`arm64`), Helm chart & PodMonitor |
| **v1.4.0** | 2026-07-17 | `258b67b` | Dynamic on-demand network configuration via Profiles + Webhook IPAM (Whereabouts), IPVLAN subinterfaces + node-local IPAM, ARP/RA interface knobs |
| **v1.5.0** | 2026-10-16 | `c2de32f` | InfiniBand hardware address & PKey device attributes, `acceptRA` sysctl support, DHCP lease release on Pod termination & retransmission fixes, schema versioning for BoltDB |

---

## 2. Dynamic On-Demand Networking, Subinterfaces & IPAM

One of DRANET's biggest architectural additions in 2026 was supporting not only physical NIC / RDMA pass-through, but also **dynamic virtual subinterfaces (IPVLAN)** and **external IPAM integration**:
- **Dynamic Network Configuration via Profiles & Webhook Provider (Whereabouts IPAM)**:
  - **`ecd40e0` / `28f3961`** (2026-06-30, PR [#223](https://github.com/kubernetes-sigs/dranet/pull/223), Antonio Ojea) — Introduce dynamic on-demand network configuration via Profiles and an external Webhook provider, including full integration with Whereabouts IPAM for cluster-wide secondary IP allocation.
- **IPVLAN Subinterfaces & Node-Local IPAM**:
  - **`fa127d2` / `fb570d6`** (2026-07-08, PR [#241](https://github.com/kubernetes-sigs/dranet/pull/241), Antonio Ojea) — Add support for creating dynamic IPVLAN subinterfaces on host NICs and allocating IPs via a built-in node-local IPAM plugin.
- **VRF (Virtual Routing and Forwarding) & Policy-Based Routing**:
  - **`08adb54`** (2026-03-31, PR [#74](https://github.com/kubernetes-sigs/dranet/pull/74), vsoch) — Implement VRF creation and policy-based routing rules inside the Pod network namespace for multi-homed AI/HPC workloads.
- **ARP & IPv6 Router Advertisement (RA) Sysctl Controls**:
  - **`7e9fa92` / `0957933`** (2026-07-16, PR [#277](https://github.com/kubernetes-sigs/dranet/pull/277), cyclinder) — Support configuring `arp_ignore` and `arp_announce` per network interface to prevent ARP flux in multi-NIC Pods.
  - **`a1f3615` / `136e5fa`** (2026-09-22, PR [#341](https://github.com/kubernetes-sigs/dranet/pull/341), cyclinder) — Add `accept_ra` sysctl configuration support for IPv6 Router Advertisements on DRA-attached interfaces.
  - **`50f9670` / `3be4272`** (2026-04-21, PR [#193](https://github.com/kubernetes-sigs/dranet/pull/193), Antonio Ojea) — Preserve IPv6 link-local addresses (`fe80::/64`) when moving interfaces into Pod namespaces unless custom link-local addresses are explicitly configured.

---

## 3. RDMA, InfiniBand, SR-IOV & Hardware Topology Discovery

- **Standardized `resource.kubernetes.io/numaNode` Attribute**:
  - **`a7fdaf0` / `bdcdde0`** (2026-02-02, PR [#122](https://github.com/kubernetes-sigs/dranet/pull/122), Christian Kadner) — Publish the standard Kubernetes DRA `numaNode` attribute on network devices for NUMA-aligned GPU + NIC co-scheduling.
- **InfiniBand-Only RDMA Devices, PKeys & Hardware Addresses**:
  - **`14a56f8` / `d6300e7`** (2026-02-04, PR [#77](https://github.com/kubernetes-sigs/dranet/pull/77), Maor Friedman) — Add support for InfiniBand-only (non-Ethernet) RDMA devices.
  - **`bc0b776` / `dd0d1ca`** (2026-09-28, PR [#343](https://github.com/kubernetes-sigs/dranet/pull/343), Christian Kadner) — Expose InfiniBand hardware address (`ib.hwAddr`) and partition key (`ib.pkey`) as filterable CEL device attributes.
  - **`efed0b7` / `8208b17`** (2026-02-19, PR [#139](https://github.com/kubernetes-sigs/dranet/pull/139), Lionel Jouin) — Fix RDMA device identification when InfiniBand interfaces share duplicate MAC addresses.
- **SR-IOV Virtual Function (VF) Attributes & Validation**:
  - **`b60de0a` / `bbeb4fb`** (2026-02-18, PR [#100](https://github.com/kubernetes-sigs/dranet/pull/100), Maor Friedman) — Expose SR-IOV (`sriov`) and Virtual Function (`vf`) device attributes in `ResourceSlice`.
  - **`eb40624` / `b232325`** (2026-05-19, PR [#213](https://github.com/kubernetes-sigs/dranet/pull/213), cyclinder) — Validate requested MTU against underlying device `minMTU`/`maxMTU` limits during `NodePrepareResources`.

---

## 4. State Persistence, Reliability & Cloud Provider Ecosystem

- **Persistent Device Checkpointing & `ResourceSlice` State Recovery**:
  - **`2430553` / `9e8c338`** (2026-03-31, PR [#115](https://github.com/kubernetes-sigs/dranet/pull/115), cyclinder) — Persist Pod device configuration in an embedded `bbolt` database (`BoltDB`) so `dranet` daemon restarts do not lose active Pod network state.
  - **`95890f5` / `a4a24d3`** (2026-08-31, PR [#314](https://github.com/kubernetes-sigs/dranet/pull/314), cyclinder) — Add schema versioning to `BoltDB` checkpoints for safe upgrades.
  - **`ca195e2` / `0bd061f`** (2026-05-05, PR [#230](https://github.com/kubernetes-sigs/dranet/pull/230), Antonio Ojea) — Persist attributes of already-allocated devices in `ResourceSlice` even while interfaces are moved inside Pod network namespaces.
  - **`c1bd791` / `169a3b6`** (2026-06-11, PR [#246](https://github.com/kubernetes-sigs/dranet/pull/246), cyclinder) — Re-apply interface hardware/ethtool configuration when restoring devices back to the host namespace on `NodeUnprepareResources`.
- **DHCP Reliability Fixes**:
  - **`7deab2d` / `643d7b3`** (2026-08-19, PR [#304](https://github.com/kubernetes-sigs/dranet/pull/304), cyclinder) — Explicitly release DHCP leases when Pods terminate (`NodeUnprepareResources`).
  - **`8daab8b` / `0b6c7a7`** (2026-10-16, PR [#369](https://github.com/kubernetes-sigs/dranet/pull/369), cyclinder) — Fix DHCP client timeout and retransmission handling.
- **Multi-Cloud Provider Support & Packaging**:
  - **GCE / GKE**: Added GCE IPv6 and subinterface support (`52946f1`, PR [#209](https://github.com/kubernetes-sigs/dranet/pull/209), Aoayang).
  - **Oracle Cloud (OKE)**: Added OCI RDMA, InfiniBand, and fabric topology discovery (`609bf88` [#116](https://github.com/kubernetes-sigs/dranet/pull/116), `6471bd1` [#220](https://github.com/kubernetes-sigs/dranet/pull/220), `1b40e71` [#252](https://github.com/kubernetes-sigs/dranet/pull/252), `a5f7510` [#350](https://github.com/kubernetes-sigs/dranet/pull/350)).
  - **Azure (AKS)**: Added AKS Placement Group & Interconnect Group topology discovery (`aa48b70` [#142](https://github.com/kubernetes-sigs/dranet/pull/142), `d49bc5c` [#163](https://github.com/kubernetes-sigs/dranet/pull/163)).
  - **Alibaba Cloud (ACK)**: Added eRDMA and Lingjun High-Performance Network support (`fe485d7` [#150](https://github.com/kubernetes-sigs/dranet/pull/150), `aa48bf9` [#169](https://github.com/kubernetes-sigs/dranet/pull/169)).
  - **CoreWeave (CKS)**: Added CoreWeave GPU/RDMA cluster deployment documentation (`79b31f1`, PR [#132](https://github.com/kubernetes-sigs/dranet/pull/132)).
  - **Helm Chart, PodMonitor & Multi-Arch Builds**: Published official Helm chart with Prometheus `PodMonitor` (`a614424` [#226](https://github.com/kubernetes-sigs/dranet/pull/226), `7db8828` [#202](https://github.com/kubernetes-sigs/dranet/pull/202)) and multi-arch `linux/amd64` + `linux/arm64` images (`fa443de`, PR [#204](https://github.com/kubernetes-sigs/dranet/pull/204)).
