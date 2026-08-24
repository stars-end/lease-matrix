# Lease Matrix v0.1 Spec — Errata and Open Questions

Date: 2026-08-24
Against: spec.md revision 2026-08-24 (v0.1), as amended 2026-08-24
Status: RESOLVED 2026-08-24 — Part 3 records the architect's resolutions and is authoritative. Parts 1–2 are retained as the historical question record.
Repo: stars-end/lease-matrix

How to use this document:

- **Part 1 (Errata)** — proposed one-line corrections (historical; resolutions in Part 3).
- **Part 2 (Open questions)** — questions as sent to the architect (historical; resolutions in Part 3).
- **Part 3 (Architect resolutions)** — authoritative decisions. All spec amendments below were applied to spec.md on 2026-08-24.

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

---

## Part 3 — Architect resolutions (2026-08-24) — AUTHORITATIVE

All items below were applied to spec.md on 2026-08-24. Two material corrections overturned the proposed defaults: E-1 (asymmetric trade equity) and Q-5 (contractual-cents variance).

### R-E-1. Trade equity — E-1 as written REJECTED; asymmetric rule adopted — BLOCKING (M1)

`tradeEquityCents` is signed net equity. Derive `positiveTradeEquityCents = max(tradeEquityCents, 0)` and `capitalizedNegativeEquityCents = max(−tradeEquityCents, 0)`.

- Adjusted cap cost subtracts positive trade equity and adds capitalized negative equity (§11.2 amended).
- Total non-refundable economic cost adds positive trade equity surrendered (§11.6 amended from "trade equity applied").
- Capitalized negative equity is never added or subtracted again in economic cost — its cost is already in the contractual payments. Adding the signed negative value would incorrectly reduce economic cost.
- Flag renamed: `includeTradeEquityInCashDAS` → `includePositiveTradeEquityInDisplayedDAS`. Display-only; must not change cap cost, payment, or effective cost. When true, the alternate signing-contribution view adds positive equity only. UI labels: "Cash due at signing" / "Total signing contribution, including trade equity".
- Negative equity paid separately at signing is not representable by the signed field in v0.1 — needs a separate upfront non-refundable line or an unsupported-state warning.
- Golden fixtures required: zero equity; positive equity; capitalized negative equity; positive equity with alternate display.

### R-E-2. `Vary: Accept-Encoding` — approved with details

Set on every compressible representation whether or not gzip is selected; preserve or append existing `Vary`; HEAD returns the same representation-selection headers as GET; no uncompressed `Content-Length` on gzip responses; test gzip, identity, and HEAD. (§24.3 reference server and §19.8 test list amended.)

### R-E-3. CSV formula escaping — approved with amendments — BLOCKING (M3)

Escape behavior is determined by declared CSV column type, never a permissive generic number parser. Numeric columns: canonical serializer, plain negatives unescaped, no currency symbols/thousands separators in machine numeric fields. Free-text columns: strip NUL, RFC 4180 quoting, leading-apostrophe escape on trigger set `=`, `+`, `-`, `@`, tab, CR, LF, and full-width equivalents (OWASP current guidance; no mitigation universally safe). Reversible rule: `=SUM(...)` → `'=SUM(...)`; original `'=SUM(...)` → `''=SUM(...)`; import removes exactly one exporter apostrophe; unwrap only declared free-text columns; never evaluate as formula. Required tests: `-1500` numeric; `=1+1`; `+cmd`; `@SUM(A1:A2)`; leading tab; embedded CR/LF; delimiter/quote breakout; original literal apostrophe; export→import→export stability. (§16.3 replaced.)

### R-E-4. Vite env vars — approved

`EMBED_ALLOWED_ORIGINS` = Railway process runtime (Node server); `VITE_EMBED_ALLOWED_ORIGINS` / `VITE_ENABLE_EDMUNDS_PASTE` = build-time browser bundle values; changing a `VITE_*` requires rebuild + redeploy; annotate in `.railway/railway.ts`; v0.1 keeps allowlists empty and flag false; runtime-config injection out of scope. (§24.4 annotated.)

### R-E-5. Canonical names — resolved

Repository `stars-end/lease-matrix`; slug `lease-matrix`; application "Lease Matrix"; package scope `@lease-matrix/*`. Do not rename. spec.md header already said `lease-matrix`; the `leasehackr-matrix` string was only in the original chat-UI path, not in the committed spec. No spec change needed.

### R-E-6. Zod / exactOptionalPropertyTypes — pattern blessed

1. Schema-first at I/O boundaries: Zod schema is source of truth; derive `z.input`/`z.output` types; never duplicate wire types as handwritten interfaces.
2. Domain types keep `field?: T` (never `field?: T | undefined`); absence = omission; builders use conditional spreads, never write explicit `undefined`.
3. Shared `compactUndefinedDeep(value)` applied immediately after boundary parsing and before commands/persistence/export/serialization: removes undefined properties, rejects undefined array elements (no silent null conversion), preserves permitted null, preserves branded integers, unit-tested for nesting.
4. Keep `exactOptionalPropertyTypes: true`; no per-file casts or `as unknown as`. Record once in docs/decisions.md. (Added as spec §5.6.)

### R-Q-1. Leasehackr parameter map — verified against public URLs; three manual validation vectors supplied — BLOCKING for M3 done

Core §12.3 names (`msrp`, `sales_price`, `months`, `miles`, `mf`, `resP`, `taxed_inc`, `untaxed_inc`, `acq_fee`, `dealer_fee`, `gov_fee`, `demo_mileage`, `pretax_monPmt`, `lease_das`) are supported by public 2025–2026 share URLs. Original table was provisionally correct but previously unverified; naming confidence now closed subject to the manual M3 browser check.

Registry amendments applied to §12.3:

- `sellingPriceTax_radio` — experimental source tax-method flag
- `salesPriceTax_radio` — import-only legacy alias
- `fees_untaxed` — experimental source flag
- `ny_tax` — preserve-only pending manual fixture
- `lvf_result_mode` — preserve-only UI-state alias
- `doc_fee` — import-only legacy dealer-fee alias, map after fixture confirms semantics
- `tradein` — downgraded to source diagnostic/preserve-only until a nonzero fixture confirms semantics (net equity vs gross trade value vs other)
- `dp` — positive maps to cash cap reduction; negative values (observed in public links as balancing adjustments) produce an import warning and remain unresolved source diagnostics

M3 may be declared complete only after recording: (1) local parse of each vector without network; (2) import-preview comparison vs expected fields; (3) diagnostics stay diagnostic; (4) fresh outbound URL generated from accepted canonical document; (5) outbound URL opened manually in a normal browser; (6) supported input fields populate correctly; (7) mismatches/redirects/changed defaults/unsupported flags recorded; (8) semantic round-trip verified (supported financial inputs, not byte-for-byte URL equality; redirects between hosts, parameter reordering, and recalculated `pretax_monPmt`/`lease_das` do not fail).

Validation vectors (manual M3 use only — no CI fetch, no production dependency, no seed data):

**Vector A — core fields, monthly-payment tax mode (posted April 2026):**

```text
https://leasehackr.com/calculator?acq_fee=925&bmw_demo_25=true&dealer_fee=0&demo_mileage=5000&disp_fee=495&dp=0&exp_rv=0&fin_apr=0&fin_dp=0&fin_ps_rebate=0&fin_rebate=0&fin_sp=88000&fin_tax=0&fin_taxed_fee=0&fin_term=60&fin_untaxed_fee=0&gov_fee=0&keep_term=60&lease_das=799&lease_result_mode=true&make=BMW&memo=&mf=.00080&miles=10000&monthlyTax_radio=true&months=36&msd=0&msrp=88000&pretax_monPmt=799&rebate=0&resP=53&sales_price=70000&sales_tax=0&service_fee=0&taxed_inc=0&tradein=0&untaxed_inc=0
```

Expected: MSRP $88,000; selling price $70,000; term 36; miles 10,000; MF .00080; RV 53%; demo mileage 5,000; monthly-tax flag true; reported pre-tax payment $799; reported DAS $799.

**Vector B — demo mileage, taxed incentives, total-lease tax, fee flags (posted September 2025):**

```text
https://leasehackr.com/calculator?acq_fee=1095&bmw_demo_25=true&cap_tax=true&dealerFee_check=false&dealer_fee=175&demo_mileage=4206&disp_fee=595&dp=0&exp_rv=0&fin_apr=0&fin_dp=0&fin_ps_rebate=0&fin_rebate=0&fin_sp=82000&fin_tax=0&fin_taxed_fee=0&fin_term=60&fin_untaxed_fee=0&gov_fee=300&keep_term=60&lease_das=757&lease_result_mode=true&make=Mercedes-Benz&memo=2025%2BGLC%2B350e%2BSUV%0ADEMO&mf=.00214&miles=7500&months=24&msd=0&msrp=67710&pretax_monPmt=457&rebate=0&resP=65&sales_price=57000&sales_tax=8.625&service_fee=0&taxed_inc=10000&totalLeaseTax_radio=true&tradein=0&untaxed_inc=0
```

Expected: MSRP $67,710; selling price $57,000; term 24; miles 7,500; MF .00214; RV 65%; taxed incentives $10,000; demo mileage 4,206; total-lease-tax flag true; cap-tax flag true; dealer-fee checkbox false; reported pre-tax payment $457; reported DAS $757.

**Vector C — selling-price tax mode, untaxed fees, zero drive-off (posted February 2025):**

```text
https://leasehackr.com/calculator?make=Mazda&miles=15000&msrp=56400&sales_price=43995&months=24&mf=0.00008&msd=0&dp=0&dealer_fee=225&acq_fee=650&disp_fee=350&taxed_inc=0&untaxed_inc=9500&rebate=0&resP=58&gov_fee=350&sales_tax=6.25&demo_mileage=0&memo=2024%20Mazda%20CX-90%20PHEV%20Premium%20AWD&tradein=0&fin_sp=56400&fin_taxed_fee=0&fin_untaxed_fee=0&fin_term=60&fin_apr=0&fin_dp=0&fin_rebate=0&fin_ps_rebate=0&fin_tax=0&keep_term=60&exp_rv=0&service_fee=0&zero_driveoff=true&sellingPriceTax_radio=true&bmw_demo_25=true&fees_untaxed=true&lease_result_mode=true&pretax_monPmt=230&lease_das=0
```

Expected: MSRP $56,400; selling price $43,995; term 24; miles 15,000; MF 0.00008; RV 58%; untaxed incentives $9,500; zero-drive-off true; selling-price tax flag true; fees-untaxed flag true; reported pre-tax payment $230; reported DAS $0.

### R-Q-2. First payment in DAS — `firstPaymentCents` definition adopted

First contractual payment = first scheduled periodic installment due at signing after the selected tax profile applies upfront-vs-capitalized tax treatment; separate upfront tax is not part of the installment. Explicit `firstPaymentCents` output added to the tax calculation result (§8.12). California monthly-payment profile: `firstPaymentCents = postTaxMonthlyCents`. Upfront-total-tax: first installment from tax-profile output, upfront tax a separate DAS line. Capitalized upfront tax: installment contains financing attributable to it. Texas-style: tax on lessor's purchase; later installments not themselves taxed. No solver or DAS formula may choose pretax vs post-tax by inspecting the tax-method name. (§11.5 amended.)

### R-Q-3. Trade equity flag — display-only confirmed; asymmetric formula corrected

See R-E-1. The originally proposed "always include signed trade equity in effective cost" was rejected; only positive equity surrendered is added.

### R-Q-4. CSP — approved with exact interpretation

`style-src 'unsafe-inline'` accepted for v0.1 because AG Grid's theming injects style elements (not Tailwind); nonce hardening (AG Grid `styleNonce`) out of scope; `script-src 'self'` holds; `unsafe-eval` prohibited; `img-src 'self' data:` required (AG Grid SVG data URLs); AG Grid string expressions prohibited for value getters/class rules — functions only. Production tests required: no inline executable script in built index.html; app loads under `script-src 'self'`; grid usable under approved style policy; no dependency requires `unsafe-eval`; CSP never weakened to pass a test. (§21.2 amended.)

### R-Q-5. Solver variance — unrounded reporting REJECTED; contractual cents adopted

Full Decimal precision guides interval selection/bisection internally; `achievedMonthlyCents` = contractually rounded achieved monthly payment; `paymentVarianceCents = achievedMonthlyCents − quotedMonthlyCents`; stop within `paymentToleranceCents` on absolute integer cents; never report fractional-cent residuals under `MoneyCents`. `impliedSellingPriceCents` stays cent-precise; nearest-dollar is display-only; golden fixtures assert cent-precise; a raw Decimal residual may exist debug-only. (§11.3 amended.)

### Checkpoint plan — M1/M3 approved; M1a added

- **M1a (new, human review)**: after domain schemas + forward-calculation skeleton, before inverse solvers and full fixture set. Evidence: no-tax/no-fee forward fixture; CA monthly-tax fixture; positive/negative equity hand calculations; component + final rounding confirmation; first-payment semantics from tax output.
- **M1**: all golden fixtures; trade-equity cases; MSD separation; tax breakdown; cents-vs-display rounding; no React dependency in engine.
- **M3**: three manual vectors above; import preview; unknown/unsupported params; semantic export/reopen; CSV injection cases; no network during import.
- No additional human checkpoint at M0 beyond green scaffold, strict typecheck, `pnpm verify`.

### Additional dispatch assumptions (architect, binding)

1. `dp` not guaranteed nonnegative — negatives import as unresolved source adjustments, not canonical cash down.
2. `tradein` semantics unvalidated — preserve until nonzero fixture.
3. Additional tax flags/legacy aliases go into the versioned registry as experimental/preserve-only, never silently unknown.
4. Source-reported `pretax_monPmt`/`lease_das` may be recalculated on reopen — diagnostics only.
5. California taxability is line-specific; profile + normalized line records decide treatment.
6. "First payment" is a tax-profile output, not a global synonym for post-tax monthly.
7. No parameter name implies valid content — out-of-bounds values produce preview errors; never clamped or coerced.

### Dispatch decision

Proceed from Milestone 0 with all amendments applied. M1 blocked on corrected trade-equity fixtures (now specified). M3 blocked on manual Leasehackr URL validation and CSV adapter tests.
