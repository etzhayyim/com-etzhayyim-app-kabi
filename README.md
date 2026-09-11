# com-etzhayyim-app-kabi

kabi (カビ) — mycelium network layer. Manages hypha routing, anastomosis
compatibility gates, and nutrient flow channels between kobo agents
(`APP_DESCRIPTION` in [`wrangler.jsonc`](wrangler.jsonc)). Deployed as the
Cloudflare Worker `kotodama-k4b100m1`, routed at `k4b100m1.etzhayyim.com/*`
and `kabi.etzhayyim.com/*`.

This repo was extracted from `etzhayyim/root:60-apps/etzhayyim-project-kabi`
(see [`migration.edn`](migration.edn) for provenance) and carries the
etzhayyim actor descriptor at [`kotodama.jsonld`](kotodama.jsonld).

## Frontend migrated to ClojureScript (2026-09-07)

The `svelte/` directory (SvelteKit) is gone. The frontend is now
ClojureScript — reagent + re-frame + `jp-go-dds` (デジタル庁デザインシステム) —
at [`cljs/`](cljs). This was a **frontend-only** migration; the backend
Worker/XRPC logic was moved, not rewritten:

| Then | Now |
|---|---|
| `svelte/src/routes/+page.svelte` (the status page) | [`cljs/src/kabi/app.cljk`](cljs/src/kabi/app.cljk) — same seven facts + own path, faithfully ported (route/var counts corrected against `wrangler.jsonc`, see that namespace's docstring) |
| `svelte/src/routes/xrpc/[...path]/+server.ts` (the deployed XRPC handler, per `wrangler.jsonc`'s old `main`) | [`src/xrpc-dispatcher.ts`](src/xrpc-dispatcher.ts) — moved byte-for-byte, only a provenance header comment added |
| `wrangler.jsonc` `main: svelte/.svelte-kit/cloudflare/_worker.js` | `main` dropped entirely |
| `wrangler.jsonc` `assets.directory: ./svelte/.svelte-kit/cloudflare/client` | `assets.directory: ./cljs/public` |

`main` is **not** repointed at either Worker source still in this repo
(`src/app.ts` or the moved `src/xrpc-dispatcher.ts`): neither calls
`env.ASSETS.fetch`, so putting either in front of the static assets would
mean nothing serves them. Both remain orphaned source (`src/app.ts` was
already unreferenced before this migration).

This change is **UNVERIFIED**: no `wrangler` publish or local-preview
command was run against it.

## Build and test

```bash
cd cljs
npm install
npm run build   # amu compile --target wasm32-browser app -> public/js/app.js
npm test        # amu compile --target wasm32-browser test && node out/tests.js
```
