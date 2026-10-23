# knftables (`kubernetes-sigs/knftables`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/knftables
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 22 commits (15 first-parent merges) across **v0.0.20**, **v0.0.21**, and **v0.0.22**.

---

## 1. Release Timeline in 2026

| Release | Date | Tag Commit | Highlights |
| :--- | :--- | :--- | :--- |
| **v0.0.20** | 2026-02-10 | `07e68ba` | Added `ListAll()`, scoped `List()` to single table to avoid full ruleset dumps (`#37`) |
| **v0.0.21** | 2026-03-11 | `4533189` | Use `--terse` for `list table` commands to avoid `nft` 1.1.3+ JSON bugs and speed up listing (`#40`, `#41`) |
| **v0.0.22** | 2026-07-20 | `69a6e3f` | Added opt-in direct Linux `netlink` support (`UseNetlink`) for `ListRules` / `List` (`#38`, `#43`, `#44`) |

---

## 2. Key Changes & Impact on `kube-proxy` and `kube-network-policies`

`knftables` is the foundational Go library powering `kube-proxy`'s `nftables` mode and `kube-network-policies`:
- **Direct Netlink Support (`UseNetlink`)**:
  - **`f2c1170` / `db4af93` / `94881c9`** (2026-05-21, PR [#38](https://github.com/kubernetes-sigs/knftables/pull/38), Antonio Ojea) — Implement native Linux netlink `List` and `ListRules` operations, bypassing the `nft -j list` CLI subprocess and JSON parser when reading existing kernel rules.
  - **`2163caa`** (2026-07-20, PR [#43](https://github.com/kubernetes-sigs/knftables/pull/43), Antonio Ojea) & **`2cbbcd2` / `9709962`** (2026-07-20, PR [#44](https://github.com/kubernetes-sigs/knftables/pull/44), Dan Winship) — Expose `UseNetlink` option (subsequently enabled by default in `kube-proxy` in k/k [#137536](https://github.com/kubernetes/kubernetes/pull/137536)).
- **`nft` 1.1.3+ Compatibility & `--terse` Listing Performance**:
  - **`8f8d3c2` / `8f065ba` / `61cc0bb`** (2026-02-10, PR [#37](https://github.com/kubernetes-sigs/knftables/pull/37), Dan Winship) — Ensure `List()` only queries the target `knftables` table rather than dumping all kernel tables, and add `ListAll()`.
  - **`09eabef` / `0c21485`** (2026-03-06, PR [#40](https://github.com/kubernetes-sigs/knftables/pull/40), Dan Winship) — Pass `--terse` to `nft list table` commands so set/map elements are omitted when only chain/set names are needed, avoiding huge JSON outputs and working around an upstream `nftables` 1.1.3 JSON serialization bug.
  - **`8ac6ff9` / `b5aea6c`** (2026-03-04, PR [#41](https://github.com/kubernetes-sigs/knftables/pull/41), Dan Winship) — Standardize object type handling across `List()` and `ListElements()`.
- **Flowtable Counter Support**:
  - **`0087e9e` / `e620411`** (2026-09-03, PR [#47](https://github.com/kubernetes-sigs/knftables/pull/47), Casey Davenport) — Add support for `counter` on `nftables` flowtables.
