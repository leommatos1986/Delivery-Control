# Delivery Control Tower — Next.js

A Next.js port of the **Delivery Control Tower** for SAP consultancies —
a Control Tower for planning, kanban, RAID, hours, and governance across
projects, ready to run locally, version on GitHub, and deploy on Vercel.

The React state and UI are already migrated; the mock database lives in
memory behind a data-access layer (`lib/data`) that will later be swapped
for Supabase without touching any component.

---

## Stack

- **Next.js 15** (App Router, TypeScript, React 19)
- **Tailwind CSS 3.4** with CSS custom properties (design tokens, dark mode
  via `data-theme="dark"`)
- **Zustand** for lightweight client-side UI state (drawer, modal, filters)
- **SheetJS (xlsx)** — kept as a dependency for future Excel export
- **IBM Plex Sans / Mono** via `next/font/google`

No database yet — the data layer is fully in-memory but isolated behind a
stable API so Supabase (or any other backend) can slot in later by touching
only `lib/data`.

---

## Getting started

```bash
# 1. install dependencies
npm install

# 2. start the dev server (default: http://localhost:3000)
npm run dev
```

Other scripts:

```bash
npm run build       # production build
npm run start       # serve the production build
npm run lint        # ESLint
npm run type-check  # TypeScript project-wide check
```

Node 20+ is recommended.

---

## Folder tree

```
dct-next/
├─ app/
│  ├─ layout.tsx              # Root layout (fonts + AppShell)
│  ├─ page.tsx                # Home (Delivery Overview)
│  ├─ globals.css             # Design tokens + component classes
│  ├─ kanban/page.tsx         # Kanban board (drag-and-drop)
│  ├─ projects/
│  │  ├─ page.tsx             # Portfolio list
│  │  └─ [id]/page.tsx        # Project detail (tabs)
│  ├─ clients/page.tsx        # Clients + projects card list
│  ├─ risks/page.tsx          # Risk register (all projects)
│  ├─ issues/page.tsx         # Issue management (all projects)
│  ├─ decisions/page.tsx      # Decision log (all projects)
│  ├─ gov/page.tsx            # Governance Top-N
│  ├─ resources/page.tsx      # Capacity × utilization
│  └─ my/ hours/ timeline/ saving/ reports/ admin/ docs/
│                             # Placeholder pages (see "Pending" below)
│
├─ components/
│  ├─ activity/
│  │  └─ ActivityDrawer.tsx   # Right-side slide-in with 8 tabs
│  ├─ dashboard/
│  │  ├─ HomeView.tsx
│  │  ├─ PortfolioTable.tsx
│  │  └─ GovernanceTop10.tsx
│  ├─ kanban/
│  │  ├─ KanbanBoard.tsx
│  │  ├─ KanbanColumn.tsx
│  │  └─ KanbanCard.tsx
│  ├─ projects/
│  │  └─ ProjectDetail.tsx
│  ├─ layout/
│  │  ├─ AppShell.tsx         # Sidebar + Topbar shell
│  │  ├─ Sidebar.tsx
│  │  ├─ Topbar.tsx           # client filter + theme toggle
│  │  ├─ ModalHost.tsx
│  │  └─ DrawerHost.tsx
│  └─ ui/
│     ├─ Card.tsx  Badge.tsx  HealthChip.tsx  KpiTile.tsx
│     ├─ PBar.tsx  Avatar.tsx  Icon.tsx  PlaceholderView.tsx
│
├─ lib/
│  ├─ constants.ts            # Phases, sessions, kanban cols, rates
│  ├─ formatters.ts           # fmtN, fmtH, fmtDate, daysUntil, ...
│  ├─ state.ts                # Zustand UI store
│  ├─ governance.ts           # Governance Top-N builder
│  └─ data/
│     ├─ index.ts             # Barrel — the ONLY import surface
│     ├─ store.ts             # Mutable arrays (in-memory)
│     ├─ seed.ts              # Fixture data
│     └─ users.ts clients.ts projects.ts activities.ts
│        risks.ts issues.ts decisions.ts milestones.ts
│        notifications.ts templates.ts workflows.ts
│
├─ types/
│  └─ index.ts                # All domain types
│
├─ next.config.ts
├─ tailwind.config.ts
├─ tsconfig.json
├─ postcss.config.mjs
├─ .env.example
└─ .gitignore
```

---

## Data-access layer (why it matters)

Every screen imports **only** from `@/lib/data`:

```ts
import { listProjects, moveActivityStatus, buildGovernanceList } from "@/lib/data";
```

The barrel (`lib/data/index.ts`) re-exports each domain module
(`users.ts`, `projects.ts`, `activities.ts`, …). Today those functions
read and mutate arrays in `store.ts`. **Tomorrow**, when Supabase is added,
each of those functions becomes an `async` call against a table — components
switch to `await` and nothing else changes.

To wire Supabase later:

1. Fill in the placeholders in `.env.example` and rename it to `.env.local`.
2. Add `@supabase/supabase-js`, create `lib/supabase/client.ts`.
3. Rewrite each domain module in `lib/data/*.ts` to call the DB.
4. Mark component data reads as `async` (or move them to server
   components / route handlers where appropriate).

The types in `/types` are the contract — do not change them when swapping
implementations.

---

## Deploy on Vercel

1. Push this repo to GitHub.
2. In Vercel, **Add New → Project → Import** the repo.
3. Framework preset: **Next.js** (auto-detected).
4. Build command: `next build` · Output directory: `.next` (defaults).
5. Add the env vars from `.env.example` when Supabase is wired.
6. Deploy.

The project is a pure Next.js App Router app — no server-side dependency
other than Node.js, so it also runs on any Node host.

---

## Pending — will come in the next round

The base is complete and demonstrable. These screens are wired into the
navigation with a `PlaceholderView` and the data layer already feeds them
— only the rich UI still needs to be ported from the vanilla artifact:

- **Minhas Atividades (`/my`)** — priority queue with inline hour timer.
- **Horas (`/hours`)** — timesheet grid + burn-rate/efficiency dashboards.
- **Cronograma (`/timeline`)** — Gantt-lite + milestone editing.
- **Schedule & Saving (`/saving`)** — allocation grid + saving calculator.
- **Relatórios (`/reports`)** — Executive Status Report generator.
- **Admin (`/admin`)** — full CRUD (users, clients, templates, workflows).
- **Arquitetura & Docs (`/docs`)** — living solution documentation.

Also pending, on top of what already ships:

- Excel export (button exists on Home; SheetJS is already installed).
- Charts on Home (burn-down, plan × actual, activities by status).
- Full inline editing on the Project Detail tabs.
- Rich comment/attachment upload on the ActivityDrawer (skeleton exists).

Everything above is a UI task — the data layer is ready.

---

## License

Proprietary — internal delivery tooling.
