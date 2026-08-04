# InHand

**Salary intelligence for India** — CTC → in-hand, tax regimes, editable breakdown, lifestyle planning, offer comparison, EMI and wealth tools. The product is a **Next.js** app with **Supabase** auth and optional cloud persistence for salary/offer sessions; tax and payroll math run **on the client**.

**Docs (start here for contributors):**

| Doc | Purpose |
|-----|---------|
| [`docs/AGENTS.md`](docs/AGENTS.md) | How to work in this repo (agents & humans) |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Stack, folder tree, routes, state |
| [`docs/PRODUCT_FLOW.md`](docs/PRODUCT_FLOW.md) | Access tiers and screen flows |
| [`docs/DESIGN_SYSTEM.md`](docs/DESIGN_SYSTEM.md) | UI tokens and patterns |
| [`docs/inhand-client-sync-ux.md`](docs/inhand-client-sync-ux.md) | Autosave, cookies, TanStack Query |
| [`docs/adr/`](docs/adr/) | Architecture decision records |

---

## Tech stack

| Layer | Choice |
|--------|--------|
| Framework | **Next.js 16** (App Router), **React 19** |
| Language | **TypeScript** (strict) |
| Styling | **Tailwind CSS 4**, **shadcn/ui**-style primitives |
| Auth & API | **Supabase** (`@supabase/ssr`, `@supabase/supabase-js`) |
| Server state | **TanStack Query 5** |
| Client state | **Zustand 5** |
| Forms | **React Hook Form** + **Zod 4** |
| PDF export | **@react-pdf/renderer** |
| Motion | **Framer Motion** (marketing / selective UI) |
| Toasts | **Sonner** |

---

## Quick start

```bash
cd app
npm install
cp .env.example .env.local
# Add env values (see `app/.env.example`).
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

From the **repository root** you can also run:

```bash
cd app && npm install
npm run dev    # forwarded: npm run dev --prefix app
```

**Access modes:** `NEXT_PUBLIC_ACCESS_MODE` unset or `default` = free-tier routing (calculator on `/salary`; premium paths redirect unless user has premium in DB). `premium` = treat the app as fully unlocked for local QA. See [`app/src/lib/config/access-mode.ts`](app/src/lib/config/access-mode.ts).

**Security:** Do not commit `.env.local`. The anon key is public by design but must stay tied to your project; rotate if leaked. `npm audit` is clean as of last check; report issues via your usual channel.

---

## Premium billing (Razorpay subscriptions)

Premium (Pro) is a **recurring subscription** (monthly / yearly) for **individual users**. Checkout runs from the existing **Premium plans** modal (and `/billing/upgrade` is available as a deep link).

### Required environment variables

Set these in `app/.env.local` (see `app/.env.example`):

- **`NEXT_PUBLIC_RAZORPAY_KEY_ID`** (or `RAZORPAY_KEY_ID`): Razorpay **Test** key id (public)
- **`RAZORPAY_KEY_SECRET`**: Razorpay **Test** key secret (server-only)
- **`RAZORPAY_PLAN_ID_MONTHLY`**: Razorpay plan id like `plan_...` (monthly)
- **`RAZORPAY_PLAN_ID_YEARLY`**: Razorpay plan id like `plan_...` (yearly)
- **`SUPABASE_SERVICE_ROLE_KEY`**: server-only key used to persist subscription rows and update `profiles.plan_tier`

Optional (recommended):

- **`RAZORPAY_WEBHOOK_SECRET`**: enables webhook verification for subscription lifecycle sync

### Create Razorpay plans (Test mode)

In Razorpay Dashboard (Test mode): **Subscriptions → Plans → New Plan**

- Monthly: every 1 month, amount ₹199 → copy the generated `plan_...` into `RAZORPAY_PLAN_ID_MONTHLY`
- Yearly: every 1 year, amount ₹1910 → copy the generated `plan_...` into `RAZORPAY_PLAN_ID_YEARLY`

### Apply Supabase migration

Billing requires a DB table and a safety trigger:

- `supabase/migrations/20260406120000_inhand_billing_razorpay.sql`

Apply via Supabase CLI (`supabase db push`) or SQL editor in the dashboard.

### Payment flow (high level)

- Client: opens Razorpay Checkout after calling `POST /api/billing/razorpay/subscription`
- Server: verifies payment signature via `POST /api/billing/razorpay/verify`
- Entitlement: user becomes premium only after verification (`profiles.plan_tier = 'premium'`)

Webhook endpoint (optional): `POST /api/billing/razorpay/webhook`

---

## Features (product)

- **Landing** — Marketing hero and CTAs into salary flow.
- **`/salary`** — Free-tier **calculator** (fixed/variable, dual in-hand, regime) or premium **CTC → breakdown** depending on env and login.
- **`/salary/detailed`** — Manual + document path, recents (premium).
- **`/salary/premium/breakdown`** — KPIs, editable component table, export **CSV/PDF**, deep links via workspace cookies.
- **`/salary/premium/lifestyle`** — Monthly plan sliders and surplus (premium).
- **`/salary/premium/*`** — Offer comparison, wealth forecast, EMI analyzer (premium). Legacy `/lifestyle`, `/salary/breakdown`, `/premium/*` redirect via `next.config.ts`.
- **`/salary/history`** — Saved salary list and deletes.
- **`/paywall`** — Pricing modal shell on free tier; tool routing via query params.
- **`/profile`** — Profile fields backed by `profiles` table.
- **`/billing/upgrade`** — Deep link into Premium checkout.
- **Auth** — Login, signup, forgot/reset password (Supabase).

---

## Repository layout

```
.
├── package.json                 # Root scripts: dev/build/lint/test → ./app (+ Playwright)
├── playwright.config.ts
├── TESTING.md
├── README.md                    # This file
├── .gitignore
│
├── app/                         # Next.js application (deploy this directory)
│   ├── package.json
│   ├── next.config.ts           # Redirects + security headers
│   ├── middleware.ts
│   ├── tsconfig.json
│   ├── components.json          # shadcn/ui
│   ├── vitest.config.ts
│   ├── .env.example
│   ├── README.md / AGENTS.md / CLAUDE.md
│   ├── public/brand/inhand-logo.svg
│   └── src/
│       ├── app/                 # App Router (thin pages + API)
│       │   ├── layout.tsx / page.tsx / globals.css
│       │   ├── api/             # auth/email-exists; billing/razorpay/*; billing/status
│       │   ├── auth/            # callback + reset-password
│       │   ├── login/, signup/, forgot-password/
│       │   ├── profile/, billing/, paywall/
│       │   ├── privacy/, security/, terms/
│       │   └── salary/          # page, detailed, history, premium/*
│       ├── components/
│       │   ├── ui/              # shadcn primitives
│       │   ├── shared/          # Composed UI
│       │   ├── features/        # Screen compositions
│       │   ├── layout/          # Navbar, footer, history sheet
│       │   ├── auth/
│       │   └── providers/       # Query, AuthSync, cloud/cookie sync, modal host
│       ├── lib/                 # Domain, stores, Supabase, server helpers
│       └── __tests__/           # Vitest unit tests
│
├── supabase/migrations/         # Postgres schema + RLS / billing
├── tests/e2e/                   # Playwright specs
│
└── docs/                        # Product & engineering docs
    ├── AGENTS.md / ARCHITECTURE.md / PRODUCT_FLOW.md / DESIGN_SYSTEM.md
    ├── inhand-client-sync-ux.md / inhand-database-design.md / …
    └── adr/                     # ADR-001 … ADR-004
```

*(Exact file counts change as features land.)*

---

## Scripts

| Command | Where | Action |
|---------|--------|--------|
| `npm run dev` | root or `app/` | Next.js dev server |
| `npm run build` | root or `app/` | Production build |
| `npm run start` | `app/` | Serve production build |
| `npm run lint` | root or `app/` | ESLint |
| `npm run test:unit` | root | Vitest (unit) |
| `npm run test:e2e` | root | Playwright |

---

## Repo hygiene

Source of truth for layout is **`app/src/app`** (routes), **`app/src/components`** (UI), and **`app/src/lib`** (domain + data). Do not reintroduce parallel `frontend/` / `backend/` / `shared/` trees under `app/src` unless the team deliberately adopts that modular split again.

For deeper database and API contracts, see **`docs/inhand-database-design.md`** and **`docs/inhand-backend-api-spec.md`**.
