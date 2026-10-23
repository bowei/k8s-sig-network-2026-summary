# Ingress2Gateway (`kubernetes-sigs/ingress2gateway`) — Activity (2026)

- **Repository**: https://github.com/kubernetes-sigs/ingress2gateway
- **Period Covered**: January 1, 2026 – October 22, 2026
- **Total Commits in Period**: 172 commits (94 first-parent PR merges) across **v1.0.0**, **v1.1.0**, and **v1.2.0**.

---

## 1. Strategic Context & 2026 Releases

With the official retirement and archiving of **`kubernetes/ingress-nginx` in March 2026**, `ingress2gateway` became a critical migration tool for the Kubernetes community. In response, the team shipped **v1.0.0** in March 2026 followed by two feature releases:

| Release | Date | Tag Commit | Highlights |
| :--- | :--- | :--- | :--- |
| **v1.0.0** | 2026-03-20 | `3e7d537` ([#393](https://github.com/kubernetes-sigs/ingress2gateway/pull/393)) | Provider/Emitter architecture separation, Gateway API v1.5 upgrade (Standard CORS, `TLSRoute` v1), Envoy Gateway, kgateway & agentgateway emitters, expanded `ingress-nginx` & GCE providers |
| **v1.1.0** | 2026-04-29 | `9617b3b` ([#407](https://github.com/kubernetes-sigs/ingress2gateway/pull/407)) | Traefik emitter, GCE Cloud CDN (`GCPHTTPFilter`) & Session Cookie support, `ingress-nginx` IP range control, buffer size, and SSL redirect translation |
| **v1.2.0** | 2026-07-07 | `c1be18e` ([#415](https://github.com/kubernetes-sigs/ingress2gateway/pull/415)) | Airlock Microgateway emitter, `ingress-nginx` `from-to-www-redirect`, `app-root`, and `ssl-passthrough` support |

---

## 2. Architecture: Provider (Input) vs. Emitter (Output) Separation & Gateway API v1.5

- **Provider / Emitter Split**:
  - **`b0cf803` / `19a6ca3`** (2026-01-13, PR [#304](https://github.com/kubernetes-sigs/ingress2gateway/pull/304), Snigdha) — Decouple input **Providers** (reading `Ingress` annotations/CRDs from `ingress-nginx`, `gce`, `kong`, `apisix`, `istio`, `openapi`) from output **Emitters** (generating standard Gateway API resources plus implementation-specific extension policies).
  - **`8fc27dc`** (2026-02-23, PR [#356](https://github.com/kubernetes-sigs/ingress2gateway/pull/356), Snigdha) — Rename the default standard output emitter to `gateway-api`.
- **Gateway API v1.5 Standard Feature Adoption**:
  - **`8bb3a4a` / `0b807cc`** (2026-03-19, PR [#391](https://github.com/kubernetes-sigs/ingress2gateway/pull/391), Snigdha) — Upgrade to Gateway API **v1.5.0**: emit Standard `HTTPCORSFilter` (now supported by default without experimental flags) and `gateway.networking.k8s.io/v1` `TLSRoute`.
  - **`efca74c` / `a6b4147`** (2026-02-13, PR [#325](https://github.com/kubernetes-sigs/ingress2gateway/pull/325), Snigdha) — Deterministic `HTTPRoute` rule naming for targeted policy attachment.
- **Structured Notifications & Conversion Reports**:
  - **`6dc7039` / `c32e8a0`** (2026-02-08, PR [#311](https://github.com/kubernetes-sigs/ingress2gateway/pull/311), Lior Lieberman) — Overhaul the notification system so users receive a clear per-resource table of converted, partially converted, and unsupported annotations from both Providers and Emitters.

---

## 3. New Implementation Emitters Added in 2026

To help users migrating from `ingress-nginx` to specific Gateway API implementations translate vendor-extended features (rate limiting, timeouts, buffer sizes, auth, IP allow/deny lists) into target policy CRDs, 5 new Emitters were added in 2026:
1. **Envoy Gateway Emitter**:
   - **`8c56b7b` / `85ca6f2`** (2026-01-21, PR [#305](https://github.com/kubernetes-sigs/ingress2gateway/pull/305), Kasey) — Initial Envoy Gateway emitter (`SecurityPolicy`, `BackendTrafficPolicy`, `ClientTrafficPolicy`).
   - **`90c1c3e` / `bdcdde0`** (2026-03-12, PR [#372](https://github.com/kubernetes-sigs/ingress2gateway/pull/372), Kasey) — Emit `EnvoyPatchPolicy` for regex path rewrites, plus basic auth, body size, IP range control, and timeouts ([#375](https://github.com/kubernetes-sigs/ingress2gateway/pull/375), [#386](https://github.com/kubernetes-sigs/ingress2gateway/pull/386), [#387](https://github.com/kubernetes-sigs/ingress2gateway/pull/387), [#399](https://github.com/kubernetes-sigs/ingress2gateway/pull/399)).
2. **kgateway Emitter**:
   - **`a5d54fd` / `09dc96b`** (2026-02-23, PR [#320](https://github.com/kubernetes-sigs/ingress2gateway/pull/320), Snigdha) — Add `kgateway` emitter (`TrafficPolicy` generation for timeouts, buffer sizes, and security settings; expanded in [#404](https://github.com/kubernetes-sigs/ingress2gateway/pull/404)).
3. **agentgateway Emitter**:
   - **`2fd48ab` / `28f1f40`** (2026-03-19, PR [#388](https://github.com/kubernetes-sigs/ingress2gateway/pull/388), Snigdha) — Add `agentgateway` emitter for AI/agent gateway configurations.
4. **Traefik Emitter**:
   - **`85602fc` / `66248e5`** (2026-04-28, PR [#400](https://github.com/kubernetes-sigs/ingress2gateway/pull/400), Kasey) — Add Traefik emitter generating Traefik `Middleware` CRDs attached via `ExtensionRef`.
5. **Airlock Microgateway Emitter**:
   - **`a6ead7c` / `ba728d5`** (2026-06-22, PR [#413](https://github.com/kubernetes-sigs/ingress2gateway/pull/413), Matei David) — Add Airlock Microgateway emitter.

---

## 4. Expanded `ingress-nginx` & GCE/GKE Provider Coverage

### `ingress-nginx` Provider Additions
Over 15 widely used `nginx.ingress.kubernetes.io/*` annotations were implemented in 2026:
- **Path Rewrites & Regex Matching**: `rewrite-target` and `use-regex` ([#316](https://github.com/kubernetes-sigs/ingress2gateway/pull/316), `d5ef0b4`).
- **Canary Traffic Splitting**: `canary`, `canary-weight`, `canary-weight-total`, `canary-by-header`, `canary-by-header-value` ([#302](https://github.com/kubernetes-sigs/ingress2gateway/pull/302), `1fa306a`; [#385](https://github.com/kubernetes-sigs/ingress2gateway/pull/385), `2cc910f`).
- **TLS & Backend Encryption**: `ssl-redirect` and `force-ssl-redirect` ([#321](https://github.com/kubernetes-sigs/ingress2gateway/pull/321), `8e2d7b4`; [#405](https://github.com/kubernetes-sigs/ingress2gateway/pull/405), `136e5fa`), `backend-protocol` and `proxy-ssl-*` to `BackendTLSPolicy` ([#319](https://github.com/kubernetes-sigs/ingress2gateway/pull/319), `98b0e55`), and `ssl-passthrough` to `TLSRoute` ([#422](https://github.com/kubernetes-sigs/ingress2gateway/pull/422), `3fe00f3`).
- **Redirects & Root Rewrites**: `app-root` ([#410](https://github.com/kubernetes-sigs/ingress2gateway/pull/410), `45dcbf6`) and `from-to-www-redirect` ([#411](https://github.com/kubernetes-sigs/ingress2gateway/pull/411), `7133ad4`).
- **Headers, Timeouts, Body/Buffer Limits & IP Access Control**: `custom-headers` & `X-Forwarded-Prefix` ([#318](https://github.com/kubernetes-sigs/ingress2gateway/pull/318)), `proxy-read-timeout` / `proxy-send-timeout` ([#355](https://github.com/kubernetes-sigs/ingress2gateway/pull/355)), `proxy-body-size` ([#367](https://github.com/kubernetes-sigs/ingress2gateway/pull/367)), `whitelist-source-range` / `denylist-source-range` ([#374](https://github.com/kubernetes-sigs/ingress2gateway/pull/374)), `client-body-buffer-size` / `proxy-buffer-size` ([#390](https://github.com/kubernetes-sigs/ingress2gateway/pull/390)), `enable-access-log` ([#380](https://github.com/kubernetes-sigs/ingress2gateway/pull/380)), and `load-balance` ([#381](https://github.com/kubernetes-sigs/ingress2gateway/pull/381)).

### GCE / GKE Provider Additions
- **`c1f3502` / `605cbba`** (2026-03-11, PR [#359](https://github.com/kubernetes-sigs/ingress2gateway/pull/359), Utkarsh Chaurasia) — Translate GCE Ingress static IP and infrastructure settings into `Gateway.spec.infrastructure` and `Gateway.spec.addresses`.
- **`3b6a64c` / `72ec3aa`** (2026-03-30, PR [#394](https://github.com/kubernetes-sigs/ingress2gateway/pull/394), Utkarsh Chaurasia) — Support GCE `BackendConfig` `GENERATED_COOKIE` session affinity translation.
- **`0120780` / `90904cf`** (2026-04-06, PR [#398](https://github.com/kubernetes-sigs/ingress2gateway/pull/398), Utkarsh Chaurasia) — Translate GCE `BackendConfig` Cloud CDN configuration into GKE `GCPHTTPFilter` policy resources.
