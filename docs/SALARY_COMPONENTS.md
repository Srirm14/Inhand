# SALARY_COMPONENTS.md — Indian salary breakdown (InHand)

Product reference for salary math and tables. This doc focuses on the **full Salary Breakdown** screen; the **free calculator** on `/salary` is a separate, lighter path (see below).

---

## 0. Free salary calculator (`/salary`)

**Not** the component-level breakdown. Single-page estimator only.

| Piece | Role |
|--------|------|
| `lib/simple-salary-calculator/types.ts` | `SimpleSalaryInput` (`annualFixedPay`, `annualVariablePay`, regime, monthly PF/prof tax, extra rows) |
| `lib/simple-salary-calculator/calculate-simple-salary.ts` | `calculateSimpleSalarySummary()` — fixed-only vs fixed+variable gross; TDS on each basis; same PT/PF/extras for both; **no** 80C/HRA |
| `lib/simple-salary-calculator/sync-compensation-split.ts` | Reconcile **annual CTC** with fixed/variable (preserve fixed on CTC edit; keep package sum consistent) |
| `features/salary-calculator/*` | Form (fixed + variable sections), `FixedVariableInHandPanel` (2×2 in-hand grid), `SalaryCompositionPanel`, premium cards + upgrade sheet |

**UI rules:** Results column stacks **in-hand panel** → composition. Variable pay is visually separate in the form (dashed/amber block); right column copy stresses **guaranteed vs illustrative** in-hand.

**Handoff:** Users who need the full modeled component table use **`/salary/detailed`** → **`/salary/premium/breakdown`** (sections 1+ below).

---

## 1. Information architecture (full breakdown)

1. **Hero stats** — **Est. monthly in-hand (excl. variable)**; **Monthly in-hand (incl. variable)** (÷12 illustrative); annual tax; total deductions. Plus a compact strip: **annual fixed cash**, **annual variable cash**, **total annual cash (fixed + variable)**, **stated CTC**, **modeled package** (earnings + employer lines).
2. **Truth banner** — Estimated vs document-parsed; disclaimer that CTC ≠ in-hand.
3. **Component table** — Earnings split into **sections** (not one flat “earnings” blob):  
   - **Fixed salary components** — `section: fixed_core` (basic, HRA, DA).  
   - **Allowances** — `section: allowance` — default meal/telecom/special + **user-added** rows (`id` prefix `allow_`, `isCustom`, `removable` where applicable). **Add allowance** control appends a row; optional defaults can be removed.  
   - **Variable pay** — `section: variable_pay` — standard **variable_pay** line when fixed+variable split is set, plus **user-added** lines (`id` prefix `var_`). **Add variable / bonus line** appends a row. Kept **out** of “monthly in-hand excluding variable.”  
   - **Employer contributions (CTC)** — Same group as before (`employer_contributions`).  
   - **Deductions** — Unchanged (`deductions`).
4. **Net in-hand summary** — Fixed monthly cash (excl. variable), optional variable ÷12 line, deductions, **est. monthly in-hand (excl. variable)**, illustrative incl.-variable monthly, annual take-home ×12 for both. The **cash path** rows are displayed as paired label+value units (hover-highlight band) to make the “fixed → deductions → in-hand” flow easy to read.
5. **Downstream CTAs** — Monthly plan, tax tools (unchanged).

Rows: engine always emits core lines for the estimated template; **optional** allowance rows (meal, telecom) can be **removed**; **custom** allowance/variable rows are **added in the UI** and flow through `recalculateBreakdownFromComponents`.

---

## 2. Component list model (`SalaryComponent`)

| Field | Purpose |
|--------|--------|
| `id` | Stable key; maps to tooltip catalog. |
| `name` | Short label in the table. |
| `description` | One-line subtitle under name (calculation hint). |
| `monthlyValue` / `annualValue` | Shown per period; annual = monthly × 12 when user edits monthly (existing behavior). |
| `type` | `earning` \| `deduction` \| `tax-free` \| `employer` — drives badge color. |
| `group` | `earnings` \| `employer_contributions` \| `deductions` — drives grouping and in-hand math. |
| `section?` | `fixed_core` \| `allowance` \| `variable_pay` — earnings-only; splits fixed vs flexible allowances vs variable for UI and summaries. |
| `lineSource` | `estimated` \| `parsed` \| `user_edited` — from engine, document flow, or cell edit. |
| `tags?` | `employer_side`, `one_time`, `tax_sensitive`, `conditional`, `recurring` — extra chips. |
| `isCustom?` | User-added allowance or variable row; **name** editable in UI. |
| `removable?` | If true, UI may remove the row (optional default allowances, custom lines). |

**In-hand aggregation rule:** Sum `monthlyValue` for `group === "earnings"` **excluding** `section === "variable_pay"` for **monthly in-hand (excl. variable)**; subtract `group === "deductions"`; **ignore** `employer_contributions` for monthly cash. Variable section contributes to **monthly in-hand (incl. variable)** and annual variable totals. **Single source of truth:** all summary fields are derived from the current `components` array via `deriveBreakdownSummaries` (`calculate-salary.ts`).

---

## 3. Grouping structure

| Group ID | UI title | Typical contents |
|----------|----------|------------------|
| `earnings` + `fixed_core` | Fixed salary components | Basic, HRA, DA (0 until edited). |
| `earnings` + `allowance` | Allowances | Meal, telecom (optional/removable), special allowance (residual to CTC), custom `allow_*` rows. |
| `earnings` + `variable_pay` | Variable pay | `variable_pay` when split; custom `var_*` rows (joining, retention, etc.). |
| `employer_contributions` | Employer contributions (CTC) | Employer PF, gratuity accrual, … |
| `deductions` | Deductions | Employee PF, professional tax, income tax (TDS), … |

---

## 4. Tooltip copy approach

- **Source of truth:** `salary-component-catalog.ts` keyed by `id`; unknown ids fall back to row `description`. **Custom rows:** `allow_*` and `var_*` use shared template tooltips in `getSalaryComponentTooltip`.
- **Trigger:** Small **Info** icon next to each component name (hover + focus for accessibility).
- **Content blocks (concise):**  
  1. Plain-language **what it is**.  
  2. **Fixed vs variable** (variable section excluded from ex-variable in-hand).  
  3. **Classification** (earning / employer contribution / deduction / reimbursement; recurring vs one-time where relevant).  
  4. **Monthly in-hand** (included in ex-variable path or not; employer-only; deduction).  
  5. **Cash & CTC impact**.  
  6. **Calculation** (illustrative model).  
  7. **Source** (estimated / parsed / user-edited).  
  8. **Applicability**.

Tone: helpful for salaried employees; **no** guarantees of legal/tax outcomes.

---

## 5. Calculation display rules

- **CTC ≠ in-hand:** Banner + employer group explains non-cash CTC parts.
- **Employer PF / gratuity:** Shown under employer group; excluded from monthly cash inflow in totals.
- **Employee PF:** Linked to PF wage ceiling in engine; tooltip states eligible wages vary by company.
- **Tax (TDS):** Estimated from regime + gross proxy (CTC less employer-only CTC slices used in engine); tooltip says actual TDS depends on declarations and payroll.
- **Variable pay:** Shown under **Variable pay** section; excluded from **monthly in-hand (excl. variable)**. Split from salary page creates `variable_pay`; user can add **`var_*`** lines. **Special allowance** is residual to stated CTC after fixed core, allowances (incl. custom), variable block, and employer slices — unless user overrides that line.
- **Parsed vs estimated:** `lineSource` + **Parsed** / **Estimated** badge; after cell edit, that row → **Edited**.

---

## 6. Badge / label system

| Badge | When |
|-------|------|
| **Estimated** | `lineSource === "estimated"` |
| **Parsed** | `lineSource === "parsed"` |
| **Edited** | `lineSource === "user_edited"` |
| **Employer-side** | `tags` includes `employer_side` or `type === "employer"` |
| **Tax-sensitive** | `tags` includes `tax_sensitive` |
| **One-time** | `tags` includes `one_time` |
| **Conditional** | `tags` includes `conditional` |

Type column keeps **EARNING** / **DEDUCTION** / **TAX FREE** / **EMPLOYER (CTC)**.

---

## 7. UX structure (breakdown table)

- **Sections:** Subheader rows for fixed core, allowances, variable pay, then employer + deductions.
- **Section actions:** Small **+** control on the **Allowances** and **Variable pay** headers (tooltip); optional rows use a subtle **red trash** on the row (revealed on row hover / focus-within).
- **Columns:** Component (info icon; **allowance + variable**: editable name for every row), **monthly** and **annual** on earnings; employer/deductions: monthly edit, annual read-only.
- **INR inputs (`InrMoneyInput`, `components/ui/inr-money-input.tsx`):** ₹ prefix, **en-IN** grouping, **`.00`** on blur; while focused, digits-only draft; commits **integer rupees** to `patchBreakdownComponent`; debounced updates keep paired monthly/annual in sync via `recalculateBreakdownFromComponents`.
- **Alignment:** Labels left, amounts right (`tabular-nums`).
- **Cash path pairing:** each cash path line is wrapped as one visual unit; on hover, the label and value row share a subtle teal highlight (also used in Offer Comparison expanded panels).

### 7a. Component breakup header & planning handoff (`salary-breakdown-view.tsx`)

- **Back:** **Back to salary inputs** → `/salary`.
- **Component breakup card:** Title + short payslip-alignment copy; **Download PDF**; **quick-link** pills (icon + label): Monthly plan (`/salary/premium/lifestyle`), EMI planner, Wealth forecast (`useTieredPremiumLinks`).
- **After the table + cash path:** **Allocation benchmarks** (illustrative bars); then **Plan from this breakdown** — three **link cards** (`BreakdownNextStepCard`) to the same three destinations with fuller copy. Keeps the table focused while still connecting to surplus, debt, and forecast flows.

---

## 8. Truthfulness

- The app **does not** claim a universal Indian salary structure.
- **Optional** components are omitted when zero or not modeled in this release.
- Copy and tooltips use **illustrative** / **typical** language and defer to employer payslips for ground truth.
- Future: document OCR can populate additional `id`s from catalog; same grouping and tooltip behavior.

---

## 9. Related docs

- `PRODUCT_FLOW.md` — §3 Salary breakdown.
- `ARCHITECTURE.md` — `calculate-salary.ts`, catalog path.
- `DESIGN_SYSTEM.md` — Tables, tooltips, badges.
