# NodeLocal DNSCache (`kubernetes-sigs/node-local-dns`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/node-local-dns
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 89 commits (44 first-parent PR merges) across 4 releases (**1.26.8**, **1.27.0**, **1.27.1**, **1.27.2**).

---

## 1. Strategic Context & Decoupling from Legacy `kube-dns`

In early 2026, SIG-NETWORK agreed to formally deprecate `kubernetes/dns` (`kube-dns`) in favor of CoreDNS + `node-local-dns`. Historically, `node-local-dns` had been forked from `kubernetes/dns` and still carried legacy `dnsmasq` / `kube-dns` packages and an outdated `k8s.io/dns` Go module path. In 2026, the maintainers modernized the codebase into a clean standalone module:
- **`1cf6936` / `52dedbe`** (2026-04-14, PR [#118](https://github.com/kubernetes-sigs/node-local-dns/pull/118), Bowei Du / maintainers) — Rename Go module from `k8s.io/dns` to `sigs.k8s.io/node-local-dns` and remove unused legacy `kube-dns` / `dnsmasq` packages, cutting **1.27.0** (May 2026).

---

## 2. Releases & Key Features in 2026

| Release | Date | Tag Commit | Key Changes |
| :--- | :--- | :--- | :--- |
| **1.26.8** | 2026-02-19 | `f623f09` | Go 1.25.7 / Debian base image security updates, CoreDNS dependency updates |
| **1.27.0** | 2026-05-06 | `c76b50e` | Standalone `sigs.k8s.io/node-local-dns` module, legacy `kube-dns` code removal, TLS support for metrics server |
| **1.27.1** | 2026-07-29 | `772ed47` | Configurable Prometheus metrics bind address, Go 1.26 base image updates |
| **1.27.2** | 2026-09-30 | `6811e68` | Go 1.27.2 & base image updates, `iptables`/`nftables` teardown reliability fixes |

### Notable Feature & Reliability Commits
- **TLS & Bind Address Configuration for Metrics Server**:
  - **`d5cf9fd` / `554dec7`** (2026-04-28, PR [#121](https://github.com/kubernetes-sigs/node-local-dns/pull/121)) — Add optional TLS (`-metrics-cert`, `-metrics-key`) support to the `node-local-dns` Prometheus metrics endpoint.
  - **`65c9b77` / `2f5bb25`** (2026-07-15, PR [#129](https://github.com/kubernetes-sigs/node-local-dns/pull/129)) — Allow configuring a full bind address (`host:port`) for Prometheus metrics instead of binding to all interfaces (`0.0.0.0`).
- **Graceful Shutdown & Rule Cleanup**:
  - **`446dcf3` / `f5f70ea`** (2026-09-10, PR [#135](https://github.com/kubernetes-sigs/node-local-dns/pull/135)) — Improve `NOTRACK` `iptables` rule teardown and interface cleanup on DaemonSet pod termination.
