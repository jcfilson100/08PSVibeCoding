# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

"Generate an engineering handoff covering: one paragraph an engineer can read in 60 seconds; architecture in plain language (frontend, backend/data, key flows); what's solid vs. what's duct tape; risks and assumptions; how to run it."

## Architecture (plain language)

- **Frontend:** Framework: TanStack Start v1 (full-stack React 19), Vite 7 build, Tailwind v4 via src/styles.css. SSR is on, but there are no server functions — it renders to static-ish routes. Routing: File-based, src/routes/*.tsx. Each route file only defines head() metadata and a component import; no logic. __root.tsx mounts QueryClientProvider, CrmProvider, the AppShell, and the Sonner toaster. Feature folders: src/features/<screen>/ owns that screen's component + a co-located <feature>.data.ts for its display-only constants (chart series, filter options, day buckets). Shared UI (AppShell, DealUpdateDrawer, ListStates, Panel) lives in src/features/shared/components/. State: A single React Context store, CrmProvider (src/features/shared/state/crm-store.tsx), holds deals, queue, activities, completed/snoozed IDs, open-deal drawer state, role (rep|leadership), and update-count/timer counters. No external state library; @tanstack/react-query is installed but used only for the provider scaffold, not for data fetching. Charts: Recharts (donuts, bars, timeline).
- **Backend / data:** There is no backend. No server functions (createServerFn), no API routes, no Supabase, no DB, no auth. Everything runs client-side. Data layer: src/features/shared/data/crm.ts exports typed seed arrays (deals, queue, activities, meetings) plus static metrics used by the Adoption panel. These are imported into CrmProvider as initial state. Mutation: updateDeal, completeQueueItem, snoozeQueueItem, logActivity, recordUpdateDuration — all useState setters in the provider. Nothing persists across reloads. List lifecycle: useListLoad (src/features/shared/state/use-list-load.ts) fakes async with setTimeout(700ms) and supports ?state=loading|empty|error query override for demoing each rule. Used by Who's Next? and Active Deals.
- **Key flows:** Next Best Action (/): Reads queue from the store (filtered against completed/snoozed), renders ranked cards. Complete/snooze mutates store and the item disappears.
Deal update drawer: Any "update" action opens DealUpdateDrawer with openDeal. Submitting calls updateDeal(id, {stage, nextStep, closeDate, amount?, competitor?}) — only 3–5 fields — and increments updatesToday + records duration.
Navigation anchor: Start Your Day (/start) links to both Who's Next? (/whos-next) and Active Deals (/active-deals) per the build sequence.
Passive activity: Activity screen renders seed activities tagged auto: true (Dialer/Email/Calendar) alongside manual entries created via logActivity.
Adoption panel (/admin): Renders hardcoded weekly-active %, task-time trend, and seat usage. Role switch (rep↔leadership) is a store toggle with no real authorization.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Clean feature/screen separation. | solid | Route files are metadata-only; screens are named after PRD screens and isolated in feature folders. Easy to find and extend. |
| No persistence | rough | Reload = reset to seed. CrmProvider is the entire "database." |

## Risks & assumptions for the team

1 	Assumes a real backend will drop into the store shape. 	High 	crm.ts types are the de-facto schema. If production CRM entities differ (e.g. Salesforce Opportunity/Task), the store + every screen needs rework.
2 	No auth on /admin. 	High (security) 	Leadership data is client-gated only. Must add a real route guard + server role check before any deployment.
3 	Adoption metrics are fictional. 	High (product) 	The hypothesis is unvalidated until real adoption + task-time telemetry exists. Do not quote the 50%/60% figures as measured.
4 	useListLoad SSR interaction. 	Medium 	Reads window.location.search during render; guarded by typeof window but the ?state= override only affects the client. Replace with real fetch before prod.
5 	No error boundaries beyond TanStack defaults. 	Medium 	A bad seed row or store mutation will crash the route, not degrade gracefully.
6 	Recharts + Tailwind v4 theme. 	Low–Medium 	Color tokens bypassed to fix rendering; theming regression risk if re-tokenized.
7 	Assumes single-tenant, single-rep view. 	Medium 	No multi-user, no ownership filtering beyond seed owner strings. Real ownership/RLS needed.
8 	No tests. 	Medium 	Verification is manual (Playwright smoke). No unit/integration coverage for the store or drawer logic.

## How to run it

```
# from project root
bun install          # or npm install
bun run dev          # Vite dev server (preview at http://localhost:8080)
bun run build        # production build
bun run build:dev    # dev-mode build (used by Lovable preview/prerender)
```
