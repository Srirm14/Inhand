# InHand — web app

Next.js (App Router) + TypeScript + Tailwind + shadcn/ui.

## Commands

```bash
npm install
npm run dev
npm run build
```

## Access mode

Logic lives in `src/lib/config/access-mode.ts`.

| When | Behavior |
|------|----------|
| Env **unset** or `NEXT_PUBLIC_ACCESS_MODE=default` | **Free tier** (calculator on `/salary`; premium workspace under `/salary/premium/*` requires signed-in premium or paywall). Same in dev and production. |
| `NEXT_PUBLIC_ACCESS_MODE=premium` | Full premium routes unlocked for local QA (login still required where middleware says so). |

Restart the dev server after changing `.env.local`.

## Routes

- `/` — Marketing landing  
- `/salary` — **Premium env:** `CtcInputForm` → breakdown. **Default/paywall env:** free fixed/variable calculator only.  
- `/salary/detailed` — Detailed CTC + document upload + recents → premium breakdown  
- `/salary/premium/breakdown` — KPI row, component breakup, plan cards (EMI, forecast, monthly plan)  
- `/salary/premium/lifestyle` — Monthly plan (spending + surplus)  
- `/salary/premium/offer-comparison` — Manual or upload 2–3 offers  
- `/salary/premium/wealth-forecast` — 5/10/20 yr projection  
- `/salary/premium/emi-analyzer` — EMI + DTI vs in-hand & monthly plan  
- `/salary/history` — Saved salary sessions  
- Legacy `/premium/*`, `/lifestyle`, `/salary/breakdown` — **`next.config.ts`** redirects to **`/salary/premium/*`**  
- `/paywall` — Free tier: global Premium plans modal shell. Closing returns to `/salary`.  
- `/billing/upgrade` — Premium checkout deep link  
- `/profile` — Profile (+ `/profile/billing`)

## Premium plans modal (free tier)

- **Component:** `src/components/features/pricing/premium-plans-modal.tsx` (embedded pricing section).
- **Host:** `src/components/providers/premium-plans-modal-host.tsx` in root `layout.tsx` (inside `Suspense` for `useSearchParams`).
- **State:** `src/lib/stores/use-premium-plans-modal-store.ts` — call **`openPremiumPlansModal({ fromPremium?: boolean })`** from buttons or **`PremiumBlurOfferTeaser`**; **`closePremiumPlansModal()`** to dismiss.

## Brand / favicon

- **`public/brand/inhand-logo.svg`** — logo mark; referenced by **`InhandLogoMark`** (`src/components/layout/inhand-logo-mark.tsx`) and `metadata.icons` in `src/app/layout.tsx`.
- **`src/app/icon.svg`** — same artwork for the app icon / favicon. Update both files together when the mark changes.

Design tokens and patterns: [`../docs/DESIGN_SYSTEM.md`](../docs/DESIGN_SYSTEM.md).

## Environment

Copy **`.env.example`** → **`.env.local`** and set:

- **`NEXT_PUBLIC_SUPABASE_URL`** and **`NEXT_PUBLIC_SUPABASE_ANON_KEY`** (Supabase project → API).
- **`NEXT_PUBLIC_SITE_URL`** — public origin with no trailing slash (e.g. `http://localhost:3000` in dev). Used for auth email redirects (reset password, email confirmation). Must match **Authentication → URL Configuration** in Supabase (Site URL + redirect allowlist).
- **Razorpay (Premium subscriptions):**
  - **`NEXT_PUBLIC_RAZORPAY_KEY_ID`** (or `RAZORPAY_KEY_ID`) + **`RAZORPAY_KEY_SECRET`**
  - **`RAZORPAY_PLAN_ID_MONTHLY`** + **`RAZORPAY_PLAN_ID_YEARLY`** (must be real `plan_...` ids from the same mode as the keys)
  - **`SUPABASE_SERVICE_ROLE_KEY`** (server-only; required to write billing rows and set `profiles.plan_tier`)

Auth emails are delivered by **Supabase Auth**; use **Resend** (or another provider) as **custom SMTP** in the Supabase Dashboard—not in this app’s env. See [`../docs/SUPABASE_AUTH_SMTP.md`](../docs/SUPABASE_AUTH_SMTP.md) and [`../docs/email/README.md`](../docs/email/README.md) for templates.

Restart the dev server after env changes.

## Premium checkout entry points

- **Modal (recommended UX):** open the global Premium plans modal from any CTA; logged-in users can complete Razorpay Checkout inside the modal.
- **Deep link:** `/billing/upgrade`
