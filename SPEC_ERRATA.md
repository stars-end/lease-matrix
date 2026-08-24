# Lease Matrix v0.1 Spec — Errata and Open Questions

Date: 2026-08-24
Against: spec.md revision 2026-08-24 (v0.1)
Status: awaiting architect resolution
Repo: stars-end/lease-matrix

How to use this document:

- **Part 1 (Errata)** — proposed one-line corrections. Each will be applied as written at implementation time unless the architect objects. Blocking items must be resolved before the affected milestone can be called done.
- **Part 2 (Open questions)** — copy/paste-ready questions needing an architect answer. Q-1 is the highest-value item.

---

## Part 1 — Errata (apply unless objected)

### E-1. `tradeEquityCents` sign convention is undefined — BLOCKING (M1 golden fixtures)
Section(s): 8.11, 11.2, 11.6

Issue: The sign of `tradeEquityCents` is never defined. §11.2 says adjusted cap cost subtracts "trade equity applied" while §11.6 adds "trade equity applied" to total cost. Both are coherent only under one sign convention, and the six hand-authored golden fixtures must encode a definite sign.

Proposed fix: Add to §8.11: "`tradeEquityCents` is signed: positive = positive equity (trade worth more than owed) that reduces adjusted cap cost and reduces total economic cost; negative = negative equity rolled into the lease that increases adjusted cap cost and increases total economic cost. Formulas in §11.2 and §11.6 consume the signed value with the operators as written."

### E-2. `Vary: Accept-Encoding` missing on non-gzip compressible responses — non-blocking (M0)
Section(s): 24.3 reference implementation

Issue: The reference server sets `Vary: Accept-Encoding` only when gzip is selected. Uncompressed compressible responses lack the header, so a shared/intermediary cache may serve the wrong representation to a gzip-capable client.

Proposed fix: Set `Vary: Accept-Encoding` on every response whose Content-Type is in the compressible set (or unconditionally), regardless of whether gzip was chosen. Add a corresponding assertion to the §19.8 server tests.

### E-3. CSV formula-escape marker is unspecified and collides with negative numbers — BLOCKING (M3 CSV adapter)
Section(s): 16.3

Issue: §16.3 prefixes cells beginning with `=`, `+`, `-`, `@`. Negative monetary values (e.g., negative trade equity) commonly round-trip through CSV, but the spec does not define (a) the escape marker, nor (b) how import recognizes and unwraps an escaped cell. As written, an escaped `-1500` is no longer a valid number.

Proposed fix:
1. Escape on export only cells that would be interpreted as formulas: content beginning with `=`, `+`, `-`, `@` that does NOT parse as a valid number under the app's numeric parsers. Plain negative numerics (`-1500`) are exported unescaped.
2. Escape marker is a leading apostrophe `'` (standard spreadsheet convention).
3. On import, a leading `'` followed by one of the four trigger characters is unwrapped; plain `-1500` parses as a negative number; nothing else is interpreted as a formula.

This preserves the OWASP intent (DDE payloads are non-numeric strings and still get escaped) without breaking numeric round-trips.

### E-4. `VITE_*` variables are build-time constants, not runtime knobs — non-blocking (M5)
Section(s): 24.4 runtime variables

Issue: `VITE_ENABLE_EDMUNDS_PASTE` and `VITE_EMBED_ALLOWED_ORIGINS` are listed as Railway runtime variables. Vite inlines `VITE_*` at build time; changing them at runtime does nothing until a rebuild.

Proposed fix: Keep them in the IaC for visibility but annotate in `.railway/railway.ts` and docs that `VITE_*` values take effect only on rebuild. If late toggling of the Edmunds flag ever matters, move it to server-injected runtime config — unnecessary in v0.1 since the default is `false`.

### E-5. Naming inconsistency: `leasehackr-matrix` vs `lease-matrix` — informational
Section(s): spec header ("Library / leasehackr-matrix / spec.md")

Issue: Spec header path says `leasehackr-matrix`; repository name is `lease-matrix`. Repo was created as **stars-end/lease-matrix** (the spec's canonical repository name). Architect should update the spec header path to match, or rename the repo.

### E-6. Zod `.optional()` conflicts with `exactOptionalPropertyTypes` — non-blocking but decide before M1
Section(s): 5.2

Issue: With `exactOptionalPropertyTypes: true`, Zod's `.optional()` infers `T | undefined`, which is not assignable to `field?: T` interfaces. This affects nearly every optional field in domain and transfer schemas and will cause per-file divergence if unpinned.

Proposed fix: Bless one canonical pattern in docs/decisions.md, e.g., "interfaces consumed by Zod output use `field?: T | undefined` at package boundaries" or "a shared `optionalField(schema)` helper drops explicit `undefined` before persistence." Decide once, apply repo-wide.

---

## Part 2 — Open questions for the architect

### Q-1. Was the Leasehackr parameter map (§12.3) verified against the live calculator? — BLOCKING for declaring M3 done
The §12.3 table lists parameter names (`sales_price`, `resP`, `pretax_monPmt`, `lease_das`, `monthlyTax_radio`, `totalLeaseTax_radio`, `cap_tax`, `acqFee_check`, `msd`, `dp`, `tradein`, `rebate`, `fin_*`, …).

All adapter tests use synthetic URLs per §19.3. Therefore a wrong parameter name passes every test while breaking real imports — this is the highest-probability correctness failure in v0.1.

Questions:
1. Were these names verified against the live calculator during the 2026-08-23/24 research (e.g., by inspecting generated share URLs), or reconstructed from prior knowledge?
2. If verified, can you supply 2–3 real calculator URLs with dummy values as validation vectors, to be used in a manual (non-CI, non-scraping) verification checklist at Milestone 3?
3. If not verified, who validates the map before M3 is called complete, and what evidence closes that gap?

### Q-2. Does "first contractual monthly payment" in DAS (§11.5) mean the post-tax payment?
Confirm the first payment in the DAS formula is the post-tax (tax-profile-applied) payment the customer actually signs. If any captive under certain tax methods collects the first payment pre-tax at signing, note it; otherwise the implementation hardcodes post-tax.

### Q-3. Confirm trade equity treatment split between policy flag and effective cost (§11.6, §11.8)
Confirm: `CalculationPolicy.includeTradeEquityInCashDAS` affects only the dealer-facing DAS display, while total non-refundable economic cost and effective monthly ALWAYS include signed trade equity. The spec implies this; asking for explicit confirmation because §11.6 and §11.8 both reference the flag and a fixture discrepancy would be costly to unwind.

### Q-4. Is `style-src 'unsafe-inline'` accepted for v0.1 launch? (§21.2)
Confirm `'unsafe-inline'` for styles (required by the AG Grid/Tailwind build) is accepted for v0.1 and nonce-based style hardening is out of scope. Also confirm `script-src 'self'` is expected to hold — the Vite production build should emit no inline scripts; if any is anticipated, flag it now so the CSP is not weakened mid-build to make tests pass (per §24.3 warning).

### Q-5. Payment-residual rounding basis in quote-reconciliation (§11.9)
Confirm `paymentVarianceCents` is computed from the unrounded achieved monthly payment vs the quoted monthly, and displayed separately from the dollar-rounded `impliedSellingPriceCents` (§11.3), so fixtures encode no double-rounding ambiguity.
