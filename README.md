# Vehicle Purchase Calculator

A single-page, no-dependencies calculator comparing net worth after N years
across five ways to acquire a vehicle: cash purchase, personal loan/HP, PCP
(keep the car or hand it back), and lease/PCH.

## Usage

Open `index.html` directly in a browser — no build step, no server, no
dependencies. Fill in the shared fields (price, term, assumed investment
return, resale %) and each method's fields, then click **Calculate**. The
results table is sorted by net worth, highest first, with the winner
highlighted.

## Model

Net worth = car equity at the end of the term (zero if you don't own the
car) + an accumulated investment balance. The investment balance has two
parts, both compounding monthly at the assumed return:

1. **Lump sum**: `price − this method's upfront outlay` (deposit / initial
   rental / full price for cash), invested from day one — money you kept
   instead of putting toward the car.
2. **Monthly slack**: `(highest monthly payment among all methods) − (this
   method's monthly payment)`, invested every month — modelling the money
   freed up by not paying the most expensive option's monthly amount.

This is the standard "finance and invest the difference" personal-finance
model: loan/PCP/lease payments are assumed funded from ordinary income, not
by drawing down the invested lump sum, so both amounts compound
independently and combine into the final net worth. For PCP — keep, the
balloon payment is paid out of pocket at the end of the term to take
ownership of the car, so it is deducted from the investment balance —
ownership isn't free. See
`docs/superpowers/specs/2026-09-21-vehicle-purchase-calculator-design.md`
in the `claude-config`/`personal` workspace for the full design rationale.

## Verification

No test framework — this is a single static HTML file with inline JS. It
was hand-verified against known formulas:

- `amortizedPayment(18000, 7, 48)` ≈ £431.03 (standard loan amortization,
  £18,000 at 7% APR over 48 months)
- `futureValueLumpSum(18000, 5, 48)` ≈ £21,976.11 (£18,000 at 5%/year,
  monthly compounding, 48 months)
- `futureValueAnnuity(430.86, 5, 48)` ≈ £22,841.96 (£430.86/month invested
  at 5%/year for 48 months)

Full worked example (defaults pre-filled in the form): £20,000 vehicle,
4-year term, 5% investment return, 40% resale, loan deposit £2,000 @ 7%
APR, PCP deposit £2,000 @ 6% APR with £8,000 balloon, lease £1,000 initial
rental + £250/month:

| Scenario | Total paid | Car equity | Investment balance | Net worth |
|---|---|---|---|---|
| Lease / PCH | £13,000 | £0 | £32,794 | £32,794 |
| PCP — keep | £21,273 | £8,000 | £24,377 | £32,377 |
| PCP — return | £13,273 | £0 | £32,377 | £32,377 |
| Cash purchase | £20,000 | £8,000 | £22,851 | £30,851 |
| Loan / HP | £22,690 | £8,000 | £21,976 | £29,976 |

## Out of scope

- Mileage limits / excess mileage charges (lease).
- Tax treatment (business use, VAT reclaim).
- Multiple saved comparisons — one ad-hoc calculation per page load.
