Lease Matrix v0.1 — Comprehensive Implementation Specification
Status: Ready for autonomous implementation
Target implementer: OpenCode GLM-5.3 build agent
Revision date: 2026-08-24
Amended: 2026-08-24 — architect review resolutions applied (asymmetric trade-equity rule, firstPaymentCents, contractual-cents solver variance, §12.3 registry additions, E-2/E-3/E-6 refinements). Resolutions recorded in SPEC_ERRATA.md Part 3.
Supersedes: The 2026-08-23 Next.js-oriented implementation spec
Repository name: lease-matrix
Preferred GitHub owner: stars-end when the authenticated account can create repositories there; otherwise the authenticated personal account
Application name: Lease Matrix
License: MIT
Primary deployment: Railway
Primary platform: Desktop web
Primary user: A lease shopper comparing multiple live offers in a spreadsheet-like workspace
Privacy model: Local-first; deal data remains in the browser in v0.1
Architecture summary: Vite + React SPA, pnpm/Turbo monorepo, pure TypeScript domain and calculation packages, versioned interoperability protocol, one stateless Railway web service

Revision summary
This revision keeps the original domain model, lease engine, RV/MF/incentive semantics, grid workflow, privacy constraints, and source-adapter policy. It makes five architectural changes:

Replace Next.js with a Vite + React single-page application.

Add a framework-neutral integration-protocol package for future Leasehackr website integration.

Separate deal serialization, program-data providers, and host embedding into independent interfaces.

Reuse the proven tooling shape from prime-radiant-ai selectively: pnpm, Turbo, Vite, React, AG Grid Community, Zustand, Tailwind, Vitest, Playwright, a small static server, and a thin OpenCode configuration.

Keep Railway on the current .railway/railway.ts Infrastructure-as-Code path while deploying only one stateless web service.

0. Agent execution contract
Implement this specification as the authoritative product and engineering contract. Deliver a functioning repository and deployment, not a mockup or architectural essay.

Work autonomously. Do not ask the user to re-decide choices resolved here.

Create a new public GitHub repository named lease-matrix. Prefer stars-end/lease-matrix when the authenticated account has repository-creation permission in stars-end; otherwise use the authenticated personal owner and report the fallback. If the repository already exists, inspect it first and preserve unrelated work.

Use main as the default branch and small, reviewable commits aligned to the milestone plan.

Use a Vite + React SPA. Do not substitute Next.js, Remix, Astro, a server-rendered framework, or a generic static HTML page.

Use AG Grid Community for the primary workspace. Do not substitute a generic HTML table or generic shadcn data table.

Do not put financial formulas, source parsing, persistence migrations, or host-protocol logic inside React components.

Keep domain, lease-engine, adapters, and integration-protocol framework-neutral and browser-independent unless a browser API is explicitly part of an adapter boundary.

Do not add authentication, a database, Supabase, analytics, advertising, telemetry, a Railway volume, or cloud persistence in v0.1.

Do not scrape, crawl, spider, automate browser access to, or make server requests to Leasehackr, Rate Findr, or Edmunds.

Leasehackr URL import must parse a user-supplied URL locally. Import must never call fetch, axios, a server endpoint, or browser navigation.

Edmunds support in v0.1.1 is a user-paste parser only. It must never retrieve the forum page.

Never bypass robots rules, CAPTCHAs, anti-bot systems, authentication, access controls, or rate limits.

Use only fictional dealer names, synthetic VIN-like placeholders, and synthetic program fixtures in the repository and screenshots.

Do not claim Leasehackr affiliation, official integration, program-data access, or calculator parity. Label calculations as estimates and the URL adapter as unofficial interoperability.

Store money, percentages, and money factors as validated integers and use exact decimal arithmetic for intermediate contract math. Never allow NaN, Infinity, or unvalidated binary floating-point values into persisted state.

Implement the versioned transfer schema and StandaloneHostBridge in v0.1. Define, test, and reserve the future embedded protocol, but do not enable third-party framing or a partner origin without explicit authorization.

Reuse generic prime-radiant-ai patterns only when they reduce risk. Do not copy its large brownfield AGENTS.md, Makefile, application-specific CI matrix, Clerk/Cube/Daneel contracts, database tooling, archived reports, or legacy Railway TOML.

Keep the greenfield agent workflow thin: a short AGENTS.md, a small opencode.json, one build-agent prompt, root package scripts, and one canonical pnpm verify gate.

Finish every P0 acceptance criterion before implementing P1 work.

Before final handoff, run formatting checks, lint, typecheck, unit tests, integration tests, production build, and Playwright smoke tests.

When credentials permit, create the GitHub repository and Railway project and deploy them. When credentials do not permit an action, complete all local work, commit the IaC, and report only the exact authenticated commands that remain.

Never state that a repository, Railway project, domain, deployment, tag, or CI run exists unless the command succeeded and the result was verified.

The final agent report must include:

GitHub repository URL;

Railway application URL, if deployed;

deployed commit SHA;

commands run and their status;

tests passed;

known limitations;

any step blocked solely by authentication;

confirmation that no scraper, external data fetcher, database, or analytics service was added.

1. Product thesis
Lease Matrix is not a replacement for Leasehackr Calculator and must not clone Rate Findr.

It is a multi-deal comparison workspace that provides:

a dense spreadsheet-like grid for comparing many lease offers;

a small, auditable lease-calculation engine;

forward calculations from selling price to payment;

inverse calculations from quoted payment and due-at-signing to implied selling price and discount;

local program presets for residual value, money factor, incentives, fees, tax, and term;

local import from a user-pasted Leasehackr Calculator URL;

export back to a Leasehackr Calculator URL for independent review;

source provenance and explicit uncertainty for program data;

JSON and CSV portability;

no collection of private dealer negotiations.

The governing principle is:

Sources populate the canonical model; sources never become runtime dependencies of the model.

If an external source changes, disappears, or blocks access, the calculator and saved workspace must continue working.

2. Research conclusions and architecture decisions
These conclusions are normative constraints. They are not optional implementation suggestions.

2.1 Leasehackr Calculator behavior and relationship model
The current Leasehackr Calculator represents the lease as separate fields rather than one opaque monthly-payment number. Relevant dimensions include:

MSRP, selling price, and pre-incentive dealer discount;

term, annual mileage, residual percentage, demo mileage, and money factor;

acquisition-fee waiver, one-pay treatment, and MSD-related money-factor adjustments;

taxed and untaxed incentives;

cash down, trade equity, and due-at-signing composition;

acquisition, dealer, government, and service fees, including upfront versus capitalized treatment;

tax method and tax rate;

post-sale incentives and disposition fee;

pre-tax payment, payment with tax, due at signing, total cost, effective monthly cost, and a shareable calculator URL.

Lease Matrix must preserve these distinctions internally even when the default grid presents a compact view.

Mandatory conclusions:

Selling price and dealer discount are measured before customer incentives.

Residual value is based on MSRP, never negotiated selling price.

RV, MF, term, annual mileage, geography, effective period, lender, credit tier, body style, and condition applicability form a program tuple. Changing a key dimension invalidates an exact match.

The captive buy rate and dealer/contract quoted MF are different concepts and must be stored separately.

Taxed customer incentives, untaxed customer incentives, dealer-side support, DAS credits, and post-sale rebates are different contract applications.

Dealer support must not automatically be counted both in negotiated selling price and again as a customer cap-cost reduction.

A user-supplied Leasehackr Calculator URL can be parsed as a serialized deal without requesting the site.

A generated Leasehackr Calculator URL is an outbound verification handoff, not proof of exact parity.

Leasehackr and Rate Findr must never be scraped or treated as program-data APIs.

If the URL format changes or Leasehackr asks the project to stop using the adapter, disable the adapter without affecting stored deals or the lease engine.

The intended relationship is complementary:

Lease Matrix                 Leasehackr Calculator
multi-deal workspace   →     canonical external review target
local comparison             single-deal calculation surface
no Rate Findr access         no data extraction by Lease Matrix
2.2 Edmunds forum data behavior
Current lease-program answers are semi-structured observations, not reliable flat rows. A valid observation may depend on:

model year, make, model, trim;

body style, including SUV versus Coupe;

drivetrain;

term and annual mileage;

ZIP code, state, or region;

program month or effective date;

lender/captive and credit tier;

new, demo, or loaner status.

One answer may cover several requested combinations in positional order. It may use a numeric MF, the symbolic phrase standard MF, state no incentives, state no information, omit loaner adjustments, or give an incentive package that includes dealer cash. Those are distinct states.

Mandatory conclusions:

Do not normalize a forum reply into one flat record when it covers multiple combinations.

Create one candidate per exact term/mileage/body-style combination.

Represent MF as numeric, symbolic-standard, not-offered, or unknown.

Represent none reported separately from unknown.

Preserve source scope. A ZIP-specific answer remains ZIP-specific.

Preserve uncertainty. Missing loaner information never authorizes an inferred residual penalty.

Incentive packages require inclusion and exclusion semantics so a package total does not double-count its components.

Parsed data must remain a candidate until the user reviews and confirms it.

The application must not retrieve an Edmunds URL, run a daily crawler, or redistribute copied forum text.

Fixtures for the paste parser must be synthetic paraphrases, not copied posts.

2.3 Framework choice after reviewing prime-radiant-ai
The current prime-radiant-ai repository already uses a compatible JavaScript toolchain:

root pnpm workspace and Turbo task orchestration;

Vite + React + TypeScript frontend;

AG Grid Community;

Zustand;

Tailwind;

Vitest and Playwright;

a small Node static server that honors Railway's PORT;

Railway/Railpack deployment patterns;

a thin opencode.json and build-agent prompt at the core of a much larger brownfield workflow.

The useful lesson is tool familiarity, not repository inheritance. Lease Matrix is a small greenfield product with no backend or database, so it should reuse the proven stack shape while avoiding Prime Radiant's application-specific complexity.

Therefore:

choose Vite + React rather than Next.js;

use pnpm + Turbo to match the proven workspace workflow;

adapt generic AG Grid, Vite, test, and static-server patterns;

write new typed grid components rather than copying finance-specific components verbatim;

keep AGENTS.md short and greenfield-specific;

do not import Prime Radiant's Clerk, FastAPI, PostgreSQL, Cube, Daneel, brokerage, semantic-layer, release-gate, or archived-debugging machinery;

do not copy Prime Radiant's legacy railway.toml into the new repository.

2.4 Eventual Leasehackr website integration
The future integration boundary is a versioned protocol, not a dependency on either site's frontend framework.

Keep three interfaces separate:

Deal serialization adapter: converts between a source representation and a canonical deal transfer document. Leasehackr URL import/export belongs here.

Program provider: looks up RV/MF/incentive observations. No provider is implemented in v0.1, and no Leasehackr or Edmunds provider may be created without explicit permission and a documented interface.

Host bridge: lets a standalone or embedded host initialize, receive, and publish sanitized deal documents. StandaloneHostBridge is implemented in v0.1. A strict-origin PostMessageHostBridge is reserved for a later authorized integration.

This permits an integration ladder without a rewrite:

v0.1       standalone companion; paste/open Leasehackr calculator links
future A   official deep-link handoff using an allowlisted sanitized payload
future B   official iframe using a versioned postMessage protocol
future C   native consumption of framework-neutral packages
future D   authorized program provider returning normalized observations
The standalone product must remain complete and useful at every stage.

2.5 Railway deployment behavior
Use Railway's current project-level Infrastructure-as-Code workflow:

.railway/railway.ts

railway config plan

railway config apply

Do not create a new railway.toml or railway.json. Railway marks the older service-level Config-as-Code path as deprecated for new services, with a December 1, 2026 cutoff for existing users.

The web service must:

build the shared pnpm monorepo from the repository root;

start one stateless Node process serving the Vite dist directory;

listen on Railway's injected PORT and 0.0.0.0;

expose /healthz returning HTTP 200;

use one replica and no volume;

include all shared-package paths in deployment watch rules when the current IaC API supports them.

Railway healthchecks gate deployment readiness. They are not continuous production monitoring and must not be described as such.

2.6 Grid library boundary
Use AG Grid Community. Community edition supports the required core behaviors: editable cells, sorting, resizing, column reordering, filtering, and pinned columns.

Do not import or depend on Enterprise-only modules or behaviors, including rich range selection, fill handles, Excel export, Enterprise clipboard operations, or Enterprise context menus.

P0 supports single-cell editing. A user may paste text into the active cell editor and may copy displayed text through normal browser selection or an explicit application-level copy action. Do not register or imitate AG Grid's Enterprise ClipboardModule. If structured multi-row or multi-cell paste is added later, implement a separate application-level paste dialog with parsing, preview, validation, and one atomic store command rather than importing Enterprise modules.

3. Release plan
3.1 v0.1 — P0, must ship
Repository, runtime, and infrastructure
New public GitHub repository with clean history.

MIT license.

pnpm 10 workspace monorepo.

Turbo task orchestration.

strict TypeScript.

Vite + React web application.

concise OpenCode configuration and repository instructions.

GitHub Actions CI on pull requests and pushes to main.

Railway .railway/railway.ts desired state and production deployment.

one stateless Railway service; no database, volume, worker, or cron.

Core product
Desktop-first spreadsheet workspace using AG Grid Community.

Add, duplicate, archive, restore, and delete deals.

Inline editing of true inputs.

Immediate per-row recalculation through pure domain commands.

Input, inherited, output, warning, and error visual states.

Sorting, resizing, reordering, visibility, and pinned identity columns.

Row detail drawer.

Forward selling-price-to-payment mode.

Quote-reconciliation mode that solves implied selling price and pre-incentive discount from payment and DAS.

Manually editable program presets.

Canonical rich RV/MF/incentive schema implemented from day one.

Leasehackr URL import and export.

Local persistence with versioned migrations.

Full-fidelity JSON import/export.

Flattened CSV deal import/export.

Fictional seeded examples.

Calculation warnings and source provenance.

California monthly-payment tax profile as the validated default.

Integration architecture in P0
integration-protocol package.

versioned LeaseMatrixDealDocumentV1 transfer schema.

versioned LeaseMatrixWorkspaceDocumentV1 export schema.

explicit privacy/redaction policy for transfer documents.

separate DealSerializationAdapter, ProgramProvider, and HostBridge interfaces.

StandaloneHostBridge implementation.

message schemas and capability names reserved for a future embedded bridge.

/embed route reserved and tested for SPA fallback, but third-party framing remains denied unless an explicit allowlist is configured in a later authorized release.

no private notes, salesperson aliases, dealer negotiation history, or VIN is placed into a URL or host payload by default.

Program-data scope in v0.1
The full schema must exist, but the default UI may collapse it into:

MF status;

captive buy rate;

quoted/dealer base rate;

residual percentage;

customer taxed incentives total;

customer untaxed incentives total;

dealer support total;

term;

annual miles;

ZIP;

effective month;

source label and URL.

An Advanced Incentives editor may be visually basic in v0.1 if the normalized records, resolver, warnings, and JSON round trip are complete.

3.2 v0.1.1 — P1, only after P0 is green
Polished structured-incentive editor.

Program library page.

Program revision history and clone workflow.

Exact, mismatched, and stale program badges.

Active-preset versus imported-row conflict resolution.

User-paste Edmunds parser behind a feature flag.

Parser review screen; parsed candidates are never auto-saved.

Application-level tabular paste dialog.

Optional authorized PostMessageHostBridge implementation only after a partner origin and integration agreement exist.

3.3 v0.2 — later
Broader validated tax profiles.

More complete one-pay and acquisition-fee-waiver modifiers.

Shareable redacted deal/workspace documents.

Optional local opportunity-cost calculations for MSDs and one-pay.

Better reverse solving for partially itemized quotes.

Partner-approved deep-link handoff using only allowlisted, sanitized fields.

PWA/offline manifest if user demand justifies it.

3.4 v0.3 — later and separately reviewed
User-triggered dealer listing URL import.

Separate Railway import service, not part of the static web process.

One fetch per explicit user action.

Structured-data-first extraction.

SSRF protections, timeouts, response-size limits, redirect limits, private-address blocking, and no anti-bot bypass.

No continuous inventory crawler.

4. Explicit non-goals for v0.1
Do not implement:

Leasehackr scraping, crawling, browser automation, or hidden requests;

Rate Findr scraping, extraction, caching, or redistribution;

Edmunds scraping, crawling, daily collection, or copied-post datasets;

an RV/MF background job or centralized program database;

an official Leasehackr integration claim;

an active third-party iframe integration;

authentication or cloud accounts;

collaboration or cloud workspaces;

a database, Redis, object storage, or Railway volume;

a backend API for ordinary v0.1 calculations;

dealer inventory crawling;

nationwide tax-law coverage;

complete one-pay economics or acquisition-fee-waiver economics;

every manufacturer's demo or loaner policy;

a marketplace, brokerage, lead-generation product, payment system, advertising, analytics, or telemetry;

real dealer names, real VINs, private messages, or live negotiations in fixtures or screenshots;

AG Grid Enterprise;

Storybook, a design-system package, or a large documentation/gating framework before launch;

Prime Radiant's app-specific backend, auth, semantic layer, CI matrix, Makefile, or agent policy;

generic AI-generated posts to third-party communities.

5. Technical stack
Use these choices unless a package is demonstrably incompatible. Record any deviation in docs/decisions.md before merging it.

5.1 Runtime and package management
Node.js 24 LTS.

pnpm 10, exact version pinned in the root packageManager field.

committed pnpm-lock.yaml with frozen installs in CI and Railway.

Turbo 2.x for workspace task orchestration.

ESM packages throughout.

5.2 Web application
Vite.

React.

TypeScript with strict: true, noUncheckedIndexedAccess: true, exactOptionalPropertyTypes: true, and project references where useful.

React Router for /, /embed, and future routes.

Tailwind CSS.

shadcn-style local components for toolbars, dialogs, drawers, menus, and forms. Components remain in the repository; no runtime dependency on a hosted component service.

AG Grid Community and ag-grid-react, pinned to the same exact version. Register only AllCommunityModule through the API supported by the pinned release; ag-grid-enterprise must not appear in dependencies, imports, lockfile package names, or bundles.

Zustand for workspace state and commands.

Zod for runtime validation, transfer schemas, and storage migrations.

decimal.js for exact intermediate arithmetic.

React Hook Form for structured editors.

Papa Parse for CSV import/export with formula-injection escaping.

date-fns for dates.

crypto.randomUUID() for IDs, with a deterministic injected ID factory in tests.

5.3 Package and test tooling
tsup for framework-neutral package builds, unless native TypeScript project references produce a simpler verified build.

ESLint.

Prettier.

Vitest.

React Testing Library.

Playwright.

GitHub Actions.

5.4 Version policy
At scaffold time, prefer versions already proven compatible in the current Prime Radiant frontend when they remain supported.

It is acceptable to use newer stable versions when required, but pin exact versions rather than open-ended ranges.

Do not copy Prime Radiant's accidental version mismatches. React and @types/react major versions must agree.

Run the full gate after dependency resolution before adding application code.

Do not install AG Grid Enterprise packages.

Do not add a second state-management, validation, form, routing, or test framework.

5.5 Framework decision
Vite is mandatory for v0.1 because the product is a client-side spreadsheet application with local persistence, no SSR requirement, no SEO-dependent pages, no authentication, and no backend. The framework-neutral packages and host protocol provide the future integration seam; a Node rendering framework would not improve that seam.

5.6 Zod and exactOptionalPropertyTypes pattern
Schema-first at I/O boundaries: for transfer documents, persistence envelopes, CSV rows, and adapter proposals, the Zod schema is the source of truth. Derive wire types with `z.input`/`z.output`; never duplicate a schema-owned wire type as a handwritten interface.

Handwritten domain types keep `field?: T` (not `field?: T | undefined`). Absent domain values are represented by omission; domain builders use conditional spreads and never write explicit `undefined`.

Normalize boundary output once through a shared `compactUndefinedDeep(value)` utility, applied immediately after successful boundary parsing and before domain command construction, persistence, JSON export, or host-protocol serialization. It removes undefined-valued object properties, rejects `undefined` array elements rather than silently converting them to `null`, preserves `null` only where the schema explicitly permits it, preserves branded integer values, and has unit tests covering nested objects and arrays.

Keep `exactOptionalPropertyTypes: true`. Do not add per-file casts, `as unknown as`, or optional-field workarounds. Record this decision once in docs/decisions.md.

6. Repository layout and boundaries
Create this structure:

lease-matrix/
├─ apps/
│  └─ web/
│     ├─ src/
│     │  ├─ main.tsx
│     │  ├─ app/
│     │  │  ├─ App.tsx
│     │  │  ├─ router.tsx
│     │  │  └─ providers.tsx
│     │  ├─ routes/
│     │  │  ├─ workspace-route.tsx
│     │  │  └─ embed-route.tsx
│     │  ├─ components/
│     │  │  ├─ deal-grid/
│     │  │  ├─ deal-drawer/
│     │  │  ├─ program-editor/
│     │  │  ├─ import-export/
│     │  │  └─ ui/
│     │  ├─ features/
│     │  │  ├─ deals/
│     │  │  ├─ programs/
│     │  │  ├─ workspace/
│     │  │  ├─ host-bridge/
│     │  │  └─ settings/
│     │  ├─ store/
│     │  │  ├─ workspace-store.ts
│     │  │  ├─ commands.ts
│     │  │  ├─ selectors.ts
│     │  │  └─ persistence.ts
│     │  ├─ styles/
│     │  └─ test/
│     ├─ scripts/
│     │  └─ serve-static.mjs
│     ├─ index.html
│     ├─ vite.config.ts
│     ├─ tsconfig.json
│     ├─ package.json
│     └─ playwright.config.ts
├─ packages/
│  ├─ domain/
│  │  ├─ src/
│  │  │  ├─ primitives.ts
│  │  │  ├─ vehicle.ts
│  │  │  ├─ source.ts
│  │  │  ├─ program.ts
│  │  │  ├─ incentive.ts
│  │  │  ├─ deal.ts
│  │  │  ├─ tax.ts
│  │  │  ├─ workspace.ts
│  │  │  ├─ import-proposal.ts
│  │  │  ├─ ports.ts
│  │  │  └─ index.ts
│  │  ├─ test/
│  │  └─ package.json
│  ├─ lease-engine/
│  │  ├─ src/
│  │  │  ├─ calculate.ts
│  │  │  ├─ money-factor.ts
│  │  │  ├─ residual.ts
│  │  │  ├─ incentives.ts
│  │  │  ├─ taxes.ts
│  │  │  ├─ fees.ts
│  │  │  ├─ solve-target-das.ts
│  │  │  ├─ solve-selling-price.ts
│  │  │  ├─ rounding.ts
│  │  │  └─ index.ts
│  │  ├─ test/
│  │  └─ package.json
│  ├─ adapters/
│  │  ├─ src/
│  │  │  ├─ leasehackr-url/
│  │  │  │  ├─ parse.ts
│  │  │  │  ├─ generate.ts
│  │  │  │  ├─ params.ts
│  │  │  │  └─ index.ts
│  │  │  ├─ edmunds-paste/          # v0.1.1 only; omit from v0.1 release
│  │  │  │  ├─ parse.ts
│  │  │  │  ├─ candidates.ts
│  │  │  │  └─ index.ts
│  │  │  ├─ csv/
│  │  │  ├─ json/
│  │  │  ├─ interfaces.ts
│  │  │  └─ index.ts
│  │  ├─ test/
│  │  └─ package.json
│  ├─ integration-protocol/
│  │  ├─ src/
│  │  │  ├─ deal-document-v1.ts
│  │  │  ├─ workspace-document-v1.ts
│  │  │  ├─ messages-v1.ts
│  │  │  ├─ capabilities.ts
│  │  │  ├─ host-bridge.ts
│  │  │  ├─ redaction.ts
│  │  │  └─ index.ts
│  │  ├─ test/
│  │  └─ package.json
│  └─ fixtures/
│     ├─ src/
│     └─ package.json
├─ docs/
│  ├─ architecture.md
│  ├─ calculations.md
│  ├─ data-model.md
│  ├─ integration-protocol.md
│  ├─ privacy-and-sources.md
│  ├─ decisions.md
│  └─ release-checklist.md
├─ .opencode/
│  └─ build-agent.md
├─ .railway/
│  ├─ railway.ts
│  └─ README.md
├─ .github/
│  ├─ workflows/ci.yml
│  ├─ dependabot.yml
│  ├─ ISSUE_TEMPLATE/
│  └─ pull_request_template.md
├─ AGENTS.md
├─ opencode.json
├─ CONTRIBUTING.md
├─ CODE_OF_CONDUCT.md
├─ SECURITY.md
├─ LICENSE
├─ README.md
├─ spec.md
├─ package.json
├─ pnpm-workspace.yaml
├─ turbo.json
├─ tsconfig.base.json
├─ vitest.workspace.ts
├─ .nvmrc
├─ .node-version
├─ .editorconfig
├─ .gitignore
└─ .prettierrc
6.1 Package boundaries
domain owns schemas, canonical domain types, the generic ImportProposal<T>, and the ProgramProvider port. It cannot import React, Vite, Zustand, AG Grid, browser storage, Node server code, adapters, or the integration protocol.

lease-engine owns deterministic financial calculations and solvers. It cannot import UI code, application state, source adapters, integration-protocol documents, or browser APIs.

adapters owns source-specific parsing/serialization plus DealSerializationAdapter and WorkspaceInterchangeAdapter. It returns validated proposals; it cannot mutate application state.

integration-protocol owns versioned transfer documents, host messages, redaction helpers, capabilities, and HostBridge. It cannot import React, AG Grid, Zustand, application stores, adapters, or the lease engine.

web owns routing, UI, browser persistence, host-bridge implementations, and orchestration.

fixtures contains synthetic, non-identifying data only.

6.2 Dependency direction
fixtures ────────────────┐
                         ▼
domain ← lease-engine ← web
   ▲          ▲           ▲
   ├─ adapters ───────────┤
   └─ integration-protocol┘
Allowed imports:

lease-engine → domain

adapters → domain, integration-protocol where transfer schemas are needed

integration-protocol → domain only for explicitly stable primitives; prefer protocol-owned schemas for wire compatibility

web → all packages

No package may import from apps/web.

6.3 State and grid boundary
The AG Grid row is a view model, not the source of truth.

Zustand stores canonical deal inputs, programs, workspace settings, and user grid preferences.

Calculated outputs are derived and are never persisted as authoritative values.

A grid edit dispatches a typed command such as updateDealField or applyProgramOverride.

Commands validate and normalize input before changing state.

Selectors create DealRowViewModel objects containing formatted values and calculation issues.

Cell renderers are presentational. They must not perform lease math or mutate domain objects.

AG Grid transaction APIs may update rendered rows after a store change, but the store remains authoritative.

Import adapters produce proposals and previews. Only an explicit user confirmation dispatches commands into the store.

6.4 No premature shared UI package
Keep UI components inside apps/web in v0.1. Do not create a design-system package until another application actually consumes it.

7. Numeric representation and invariants
Use integer storage primitives:

export type MoneyCents = number;          // $1.00 = 100
export type PercentBps = number;          // 65.00% = 6500
export type MoneyFactorMicros = number;   // 0.000690 = 690
export type Miles = number;
export type Months = number;
All persisted numeric values must pass Zod integer validation.

Conversion helpers
mfDecimal = mfMicros / 1_000_000
percentDecimal = percentBps / 10_000
dollars = cents / 100
Invariants
Money is persisted in integer cents.

Residual and tax percentages are persisted in basis points.

MF is persisted in millionths.

Use Decimal internally for multiplication/division.

Never parse currency with Number() alone; normalize commas, currency symbols, whitespace, and parentheses.

Never persist NaN, Infinity, or negative values where the schema forbids them.

Round only at defined contract/display boundaries.

MSRP must be positive for a calculation.

Term must be a positive integer.

Residual may not exceed 100% unless an explicitly allowed test fixture says otherwise.

A deal with unknown MF or RV is incomplete and may not show a definitive payment.

8. Canonical domain model
The following types are normative. Exact file splitting may vary, but fields and semantics must remain.

8.1 Source provenance
export type SourceProvider =
  | "manual"
  | "leasehackr-url"
  | "edmunds-paste"
  | "dealer-worksheet"
  | "json-import"
  | "csv-import"
  | "host-integration";

export type AcquisitionMethod =
  | "manual-entry"
  | "user-supplied-url"
  | "user-pasted-text"
  | "user-supplied-file"
  | "host-message";

export interface SourceRef {
  id: string;
  provider: SourceProvider;
  acquisitionMethod: AcquisitionMethod;
  sourceUrl?: string;
  sourceLabel?: string;
  observedAt: string;            // ISO datetime
  effectiveMonth?: string;       // YYYY-MM
  rawText?: string;              // only user-provided text
  authorLabel?: string;
  confidence: "verified-by-user" | "parsed-high" | "parsed-medium" | "parsed-low" | "manual";
  reviewStatus: "unreviewed" | "reviewed" | "rejected";
  notes?: string;
}
Rules:

Parsing a Leasehackr URL creates a SourceRef with user-supplied-url.

Parsing an Edmunds paste creates a SourceRef with user-pasted-text.

No source adapter may fetch its sourceUrl in v0.1 or v0.1.1.

Preserve original user text for local audit, but never render it as HTML.

Exclude raw third-party text from host payloads and JSON exports by default.

A host message creates a SourceRef only after it passes protocol and origin validation.

8.2 Vehicle identity
export type VehicleCondition = "new" | "demo" | "loaner" | "used" | "unknown";
export type BodyStyle = "suv" | "coupe" | "sedan" | "wagon" | "hatchback" | "convertible" | "truck" | "van" | "other" | "unknown";
export type Drivetrain = "fwd" | "rwd" | "awd" | "4wd" | "unknown";

export interface VehicleIdentity {
  modelYear: number;
  make: string;
  model: string;
  trim?: string;
  bodyStyle?: BodyStyle;
  drivetrain?: Drivetrain;
  vin?: string;
  msrpCents: MoneyCents;
  odometerMiles?: Miles;
  condition: VehicleCondition;
  exteriorColor?: string;
  interiorColor?: string;
  listingUrl?: string;
}
Normalization rules:

Preserve display labels exactly as entered.

Also generate canonical lowercase/whitespace-normalized keys for matching.

Do not collapse SUV and Coupe.

Do not infer body style from model name unless the user confirms or a deterministic alias table resolves it.

VIN is optional and must not be required for examples.

8.3 Geography and effective period
export type Geography =
  | { scope: "zip"; country: "US"; zip: string; state?: string }
  | { scope: "state"; country: "US"; state: string }
  | { scope: "region"; country: "US"; regionLabel: string; queryZip?: string; state?: string }
  | { scope: "national"; country: "US" }
  | { scope: "unknown"; country: "US"; queryZip?: string };

export interface EffectiveWindow {
  programMonth?: string; // YYYY-MM
  validFrom?: string;    // YYYY-MM-DD
  validThrough?: string; // YYYY-MM-DD
  asOf: string;          // ISO datetime
}
Rules:

ZIP-specific forum answers remain ZIP-scoped unless the user broadens them.

A source saying "regional" may use region with the raw label and query ZIP.

Do not invent OEM region codes.

A prior-month program is stale, not automatically current.

8.4 Program key
export interface ProgramKey {
  modelYear: number;
  make: string;
  model: string;
  trim?: string;
  bodyStyle?: BodyStyle;
  drivetrain?: Drivetrain;
  lessor?: string;
  conditionApplicability:
    | "new-only"
    | "new-and-demo-loaner"
    | "demo-specific"
    | "loaner-specific"
    | "all-supported"
    | "unknown";
  termMonths: Months;
  annualMiles: Miles;
  geography: Geography;
  effective: EffectiveWindow;
  creditTier?: string;
}
Changing any of these fields invalidates an attached exact-match program until the user selects a matching program or explicitly accepts an override.

8.5 Money factor
export type MoneyFactorStatus = "numeric" | "standard" | "unknown" | "not-offered";
export type MoneyFactorRateClass = "buy-rate" | "subvented" | "standard" | "quoted" | "unknown";

export interface MoneyFactorAdjustmentDefinition {
  id: string;
  kind: "msd" | "one-pay" | "acquisition-fee-waiver" | "dealer-markup" | "custom";
  deltaMicros: MoneyFactorMicros; // signed; reductions are negative
  perUnit?: boolean;
  maxUnits?: number;
  minimumMfMicros?: MoneyFactorMicros;
  label: string;
  sourceRefIds: string[];
  confidence: "known" | "user-confirmed" | "unknown";
}

export interface MoneyFactorProgram {
  status: MoneyFactorStatus;
  buyRateMicros?: MoneyFactorMicros;
  rateClass: MoneyFactorRateClass;
  creditTier?: string;
  adjustments: MoneyFactorAdjustmentDefinition[];
  sourceRefIds: string[];
}
Security-deposit/MSD funding is separate from the MF delta:

export interface SecurityDepositProgram {
  status: "known" | "not-offered" | "unknown";
  maxDeposits?: number;
  mfAdjustmentId?: string;
  amountRule?:
    | {
        kind: "fixed-per-deposit";
        amountPerDepositCents: MoneyCents;
      }
    | {
        kind: "rounded-monthly";
        paymentBasis: "pretax" | "posttax";
        incrementCents: MoneyCents;
        rounding: "up" | "nearest";
      }
    | {
        kind: "manual-total";
      };
  sourceRefIds: string[];
}
Do not assume that every captive calculates one deposit as the monthly payment rounded to the next $50. The amount rule is manufacturer/program-specific.

Deal-specific MF fields:

export type QuotedMoneyFactorBasis =
  | "base-before-adjustments"
  | "effective-after-adjustments"
  | "unknown";

export interface DealMoneyFactorSelection {
  quotedMfMicros?: MoneyFactorMicros;
  quotedMfBasis: QuotedMoneyFactorBasis;
  quotedIncludedAdjustmentIds: string[];
  msdCount: number;
  msdTotalOverrideCents?: MoneyCents;
  useOnePay: boolean;
  waiveAcquisitionFee: boolean;
  customAdjustmentIds: string[];
}
Resolution rules:

Derive the selected adjustment IDs and unit counts from MSD count, one-pay, acquisition-fee waiver, and explicit custom selections.

With quotedMfBasis: "base-before-adjustments", use quotedMfMicros ?? buyRateMfMicros as the contract base, then apply every selected supported adjustment.

With quotedMfBasis: "effective-after-adjustments", treat quotedMfMicros as already containing quotedIncludedAdjustmentIds. Do not apply those IDs twice. Apply a selected adjustment not listed as included only when its definition is known and the user confirms it is additional.

With quotedMfBasis: "unknown", a quote with any selected adjustment is assumption-dependent and cannot produce a definitive MF until the user chooses the basis. If no adjustment is selected, the quoted number may be used with a warning because base and effective are then equivalent.

Show dealer markup only when a comparable pre-adjustment contract base can be resolved. For an effective quote, reverse only explicitly known included adjustments; otherwise leave markup unknown.

When both the quoted rate and a reconstructed rate are available, differences beyond one MF micro produce MF_QUOTE_CONTRADICTION and require review.

Clamp to a known minimum when an adjustment defines one and disclose the clamp.

If the source says "standard MF" without a number, preserve status: standard; do not invent a number.

If the final MF is unknown, return an incomplete calculation with a warning.

In v0.1, MSD reductions may be applied only from an explicit adjustment definition. useOnePay and waiveAcquisitionFee remain representable for import/forward compatibility, but the default UI keeps them off. If either is enabled before its full cash-flow treatment is implemented, return ONE_PAY_UNSUPPORTED or ACQUISITION_FEE_WAIVER_UNSUPPORTED and do not claim a definitive effective cost.

A Leasehackr URL mf parameter is interpreted as base-before-adjustments when the same URL separately supplies msd or another documented adjustment selector; preserve a warning if a future fixture contradicts that assumption.

Validate msdCount as a non-negative integer and enforce a known maxDeposits. A count above the maximum is an error.

Resolve refundable deposit principal from msdTotalOverrideCents first; otherwise use the reviewed SecurityDepositProgram.amountRule. If neither is known, payment may still be calculated but msdCents and DAS including MSD remain unknown.

A rounded-monthly amount rule uses the resolved post-MSD contractual payment and the specified pre-tax/post-tax basis. Do not use the pre-MSD payment unless the source explicitly says so.

8.6 Residual value
export type ResidualStatus = "numeric" | "unknown" | "not-offered";

export interface ResidualAdjustment {
  id: string;
  kind: "demo-mileage" | "condition" | "custom";
  mode: "percent-points" | "fixed-cents" | "per-mile-cents";
  value: number;                  // signed integer: bps, fixed cents, or cents per mile by mode
  exemptMiles?: Miles;
  minMiles?: Miles;
  maxMiles?: Miles;
  label: string;
  sourceRefIds: string[];
  confidence: "known" | "user-confirmed" | "unknown";
}

export interface ResidualProgram {
  status: ResidualStatus;
  baseResidualBps?: PercentBps;
  basis: "msrp";
  adjustments: ResidualAdjustment[];
  sourceRefIds: string[];
}
Rules:

Residual is always based on MSRP.

The base residual is exact for the program key's term and annual mileage.

Do not use a universal annual-mileage adjustment table in v0.1.

If term or annual mileage changes, mark the residual mismatched.

Do not infer a demo/loaner adjustment when the source says it does not have loaner information.

Apply adjustments in this deterministic order:

sum reviewed percent-points adjustments as signed basis points and add them to the base residual percentage;

calculate percentage-based residual value from MSRP;

apply reviewed fixed-cents adjustments;

apply reviewed per-mile-cents adjustments using only mileage above exemptMiles, bounded by minMiles/maxMiles when supplied.

A per-mile adjustment is a dollar adjustment to residual value, not a change to displayed residual percentage. Show both the adjusted percentage basis and final dollar residual.

Missing odometer mileage when a required per-mile rule exists makes the residual incomplete.

Reject a final residual below zero or above MSRP unless a synthetic test fixture explicitly permits the edge case; emit RV_ADJUSTMENT_INVALID rather than silently clamping.

demo_mileage imported from a calculator link is vehicle context only. It never creates a manufacturer residual rule by itself.

8.7 Incentives
export type IncentiveCategory =
  | "general-lease-cash"
  | "regional-lease-cash"
  | "dealer-cash"
  | "loyalty"
  | "conquest"
  | "affinity"
  | "fleet"
  | "college"
  | "military"
  | "first-responder"
  | "targeted"
  | "government"
  | "pull-ahead"
  | "payment-credit"
  | "post-sale"
  | "other";

export type ContractApplication =
  | "customer-taxed-cap-reduction"
  | "customer-untaxed-cap-reduction"
  | "dealer-support"
  | "due-at-signing-credit"
  | "post-sale-rebate"
  | "remaining-payment-credit"
  | "unknown";

export type TaxTreatment = "taxed" | "untaxed" | "jurisdiction-dependent" | "unknown";
export type AmountSemantics = "standalone" | "incremental" | "package-total" | "unknown";

export interface EligibilityPredicate {
  kind: "general" | "loyalty" | "conquest" | "employer" | "fleet" | "location" | "vehicle" | "credit-tier" | "other";
  description: string;
  requiredProof?: string;
}

export interface IncentiveOffer {
  id: string;
  name: string;
  category: IncentiveCategory;
  amountCents?: MoneyCents;
  amountStatus: "known" | "unknown" | "none-reported";
  amountSemantics: AmountSemantics;
  contractApplication: ContractApplication;
  taxTreatment: TaxTreatment;
  fundingParty: "manufacturer" | "captive" | "dealer" | "government" | "third-party" | "unknown";
  eligibility: EligibilityPredicate[];
  stackability: "stackable" | "choose-one-in-group" | "replaces-included" | "unknown";
  stackingGroupId?: string;
  includesIncentiveIds: string[];
  excludesIncentiveIds: string[];
  effective: EffectiveWindow;
  geography: Geography;
  sourceRefIds: string[];
  notes?: string;
}
Deal selection:

export interface IncentiveSelection {
  incentiveId: string;
  eligibilityStatus: "eligible" | "ineligible" | "unknown";
  selected: boolean;
  userConfirmedApplication?: ContractApplication;
  userConfirmedTaxTreatment?: TaxTreatment;
}
Incentive resolution rules
These rules are mandatory.

Dealer cash defaults to dealer-support. It does not reduce cap cost automatically. The negotiated selling price may already reflect it.

Customer incentives are separate from selling price. Dealer discount is always computed from MSRP to pre-customer-incentive selling price.

Unknown eligibility blocks automatic inclusion. The user must mark eligible before an incentive enters calculation.

Unknown application or tax treatment blocks automatic inclusion. Show the amount in an unresolved bucket.

Package totals prevent double-counting. When a selected package-total includes another selected incentive, the package replaces the included component for summation. Show the included component in the breakdown but count the package total only once.

Choose-one groups enforce exclusivity. More than one selected item in a choose-one group is an error.

Explicit exclusions are errors. Two selected mutually exclusive incentives block a definitive calculation.

"No info on conquest" means unknown, not zero.

"No incentives" is scoped to what the source actually answered. It must not be silently interpreted as proof that no conditional, targeted, government, or dealer-side programs exist.

Post-sale rebates do not reduce contractual monthly payment. They reduce effective total cost only.

Pull-ahead/payment credits are not ordinary cap reductions. They remain separately disclosed unless the contract explicitly applies them to the new lease.

The calculation result must expose categorized totals:

taxed customer cap reduction;

untaxed customer cap reduction;

dealer support;

DAS credits;

post-sale rebates;

unresolved potential incentives.

Normative resolver algorithm:

Validate that every included/excluded incentive ID exists in the same reviewed program, no incentive references itself, and the inclusion graph is acyclic.

Start only with selected: true records. Preserve ineligible selections as errors and unknown-eligibility selections as unresolved; never silently drop them.

Resolve user-confirmed application/tax overrides for the deal without mutating the program definition.

Enforce explicit exclusions and choose-one-in-group constraints before summation.

Process package-total records in topological order. Count the package amount once and mark selected included components as disclosed-but-replaced.

Count incremental records in addition to their referenced base/package amount; their amount means the increment, not the new package total.

Count standalone records normally.

Keep unknown amount semantics or application/tax treatment in unresolvedCents and mark the calculation assumption-dependent.

If two selected package totals overlap through the same included component and the source does not explicitly say both packages stack, emit INCENTIVE_OVERLAPPING_PACKAGES and block a definitive total.

Categorize each counted line by contract application. dealer-support remains visible but contributes zero to customer cap reduction unless a user-confirmed reclassification is backed by contract evidence.

Return an audit line for every selected incentive showing original amount, counted amount, replacement relationship, category, application, tax treatment, and issue codes.

8.8 Normalized lease program
export interface NormalizedLeaseProgram {
  id: string;
  revision: number;
  name: string;
  key: ProgramKey;
  moneyFactor: MoneyFactorProgram;
  securityDeposit?: SecurityDepositProgram;
  residual: ResidualProgram;
  incentives: IncentiveOffer[];
  acquisitionFeeCents?: MoneyCents;
  dispositionFeeCents?: MoneyCents;
  sourceRefs: SourceRef[];
  createdAt: string;
  updatedAt: string;
  status: "draft" | "reviewed" | "archived";
}
Program records should be immutable once attached to a signed/final deal. Editing creates a new revision. Deals store a program snapshot for reproducibility.

8.9 Fees
export type FeeKind = "acquisition" | "dealer" | "government" | "service" | "other";

export interface FeeLine {
  id: string;
  kind: FeeKind;
  label: string;
  amountCents: MoneyCents;
  timing: "capitalized" | "upfront";
  taxability: "taxable" | "not-taxable" | "profile-default" | "unknown";
}
8.10 Tax profile
export type TaxMethod =
  | "monthly-payment"
  | "total-lease-payment-upfront"
  | "selling-price-upfront"
  | "unsupported";

export interface TaxProfile {
  id: string;
  name: string;
  rateBps: PercentBps;
  method: TaxMethod;
  taxTaxedCapReductions: boolean;
  capitalizeUpfrontTaxes: boolean;
  taxableFeeKinds: FeeKind[];
  specialCase?: "new-york" | "none";
  validationStatus: "validated" | "experimental";
  notes?: string;
}
v0.1 ships with:

CA_MONTHLY_PAYMENT_MANUAL_RATE, validated for the app's intended California workflow;

generic imported profiles for the other Leasehackr tax-method flags, marked experimental until fixtures are added.

Do not call the product a nationwide tax calculator.

8.11 Deal and input modes
export type DealStatus = "researching" | "contacted" | "quoted" | "negotiating" | "finalist" | "signed" | "lost" | "archived";

export type DealInputMode =
  | {
      kind: "selling-price";
      sellingPriceCents: MoneyCents;
    }
  | {
      kind: "quoted-payment";
      quotedMonthlyCents: MoneyCents;
      paymentIncludesTax: boolean;
      targetDASCents: MoneyCents;
      dasIncludesMSDs: boolean;
      quotedMsdCents?: MoneyCents;
    };

export interface LeaseDeal {
  id: string;
  createdAt: string;
  updatedAt: string;
  quoteDate?: string;
  status: DealStatus;
  dealerAlias: string;
  salespersonAlias?: string;
  vehicle: VehicleIdentity;
  inputMode: DealInputMode;
  programId?: string;
  programSnapshot?: NormalizedLeaseProgram;
  mfSelection: DealMoneyFactorSelection;
  incentiveSelections: IncentiveSelection[];
  feeLines: FeeLine[];
  taxProfile: TaxProfile;
  cashDownCents: MoneyCents;
  tradeEquityCents: MoneyCents;    // signed net equity: positive = positive equity applied to the lease; negative = equity capitalized into the lease
  dispositionFeeCents: MoneyCents;
  notes?: string;
  tags: string[];
  sourceRefs: SourceRef[];
  importedUnknownParams?: Record<string, string[]>;
}
programSnapshot is optional while a row is incomplete. A calculation without a reviewed or explicitly imported snapshot returns complete: false with PROGRAM_MISSING; the UI must not fabricate MF, RV, incentives, or fees. Once a deal is marked signed, the snapshot is mandatory and immutable for reproducibility.

8.12 Calculation result
export interface CalculationIssue {
  code: string;
  severity: "info" | "warning" | "error";
  fieldPath?: string;
  message: string;
}

export interface LeaseCalculationResult {
  complete: boolean;
  effectiveMfMicros?: MoneyFactorMicros;
  mfMarkupMicros?: MoneyFactorMicros;
  residualBps?: PercentBps;
  residualValueCents?: MoneyCents;
  grossCapCostCents?: MoneyCents;
  adjustedCapCostCents?: MoneyCents;
  depreciationMonthlyCents?: MoneyCents;
  rentChargeMonthlyCents?: MoneyCents;
  pretaxMonthlyCents?: MoneyCents;
  taxMonthlyCents?: MoneyCents;
  postTaxMonthlyCents?: MoneyCents;
  firstPaymentCents?: MoneyCents;  // tax-profile output: first scheduled installment due at signing; equals postTaxMonthlyCents only under the California monthly-payment profile
  dueAtSigningExcludingMsdCents?: MoneyCents;
  msdCents?: MoneyCents;
  dueAtSigningIncludingMsdCents?: MoneyCents;
  totalNonRefundableCostCents?: MoneyCents;
  effectiveMonthlyCents?: MoneyCents;
  effectiveMonthlyToMsrpBps?: PercentBps;
  dealerDiscountBps?: PercentBps;
  impliedSellingPriceCents?: MoneyCents;
  impliedDealerDiscountBps?: PercentBps;
  paymentVarianceCents?: MoneyCents;
  categorizedIncentives: {
    taxedCustomerCents: MoneyCents;
    untaxedCustomerCents: MoneyCents;
    dealerSupportCents: MoneyCents;
    dasCreditsCents: MoneyCents;
    postSaleCents: MoneyCents;
    unresolvedCents: MoneyCents;
  };
  issues: CalculationIssue[];
  breakdown: Record<string, MoneyCents | number | string>;
}
8.13 Workspace settings
export interface WorkspaceSettings {
  locale: "en-US";
  currency: "USD";
  defaultTaxProfileId: string;
  activeProgramId?: string;
  defaultTermMonths?: Months;
  defaultAnnualMiles?: Miles;
  defaultZip?: string;
  showArchived: boolean;
  compactDensity: boolean;
  includeDispositionFeeInEffectiveCost: boolean;
  includePositiveTradeEquityInDisplayedDAS: boolean;
}
Keep settings financial and presentation-specific. Do not add user-account, cloud-sync, analytics, or partner credentials to the workspace document.

9. Program observations, parser candidates, and reviewed programs
Use four layers to prevent low-confidence source text from silently becoming authoritative data.

The generic proposal envelope is owned by packages/domain so adapters and the web layer can share it without reversing dependency direction:

export interface ImportProposal<T> {
  ok: boolean;
  candidates: T[];
  warnings: CalculationIssue[];
  errors: CalculationIssue[];
  provenance: SourceRef[];
  unknownFields?: Record<string, unknown>;
}
export interface ProgramVehicleQuery {
  modelYear: number;
  make: string;
  model: string;
  trim?: string;
  bodyStyle?: BodyStyle;
  drivetrain?: Drivetrain;
  condition: VehicleCondition;
}

export interface ProgramQuery {
  vehicle: ProgramVehicleQuery;
  termMonths: Months;
  annualMiles: Miles;
  geography: Geography;
  effectiveMonth?: string;
  lessor?: string;
  creditTier?: string;
}

export interface ProgramObservation {
  id: string;
  sourceRef: SourceRef;
  queryContext: ProgramQuery;
  observedAt: string;
  mf?: MoneyFactorProgram;
  residual?: ResidualProgram;
  incentives: IncentiveOffer[];
  completeness: "complete" | "partial" | "unknown";
  notes?: string;
}
Observation — immutable raw user-supplied evidence and context.

Parse candidate — machine-extracted fields with confidence and unresolved mappings.

Reviewed program — user-approved normalized program.

Deal snapshot/override — values actually used for a specific quote.

export interface ParsedField<T> {
  value?: T;
  confidence: number; // 0 to 1
  evidence: string;
  warnings: string[];
}

export interface ProgramParseCandidate {
  id: string;
  sourceRef: SourceRef;
  modelYear: ParsedField<number>;
  make: ParsedField<string>;
  model: ParsedField<string>;
  trim: ParsedField<string>;
  bodyStyle: ParsedField<BodyStyle>;
  termMonths: ParsedField<number>[];
  annualMiles: ParsedField<number>[];
  zip: ParsedField<string>;
  mfValues: ParsedField<MoneyFactorProgram>[];
  residualValues: ParsedField<ResidualProgram>[];
  incentives: ParsedField<IncentiveOffer>[];
  mappingWarnings: string[];
}
No parsed candidate becomes a reviewed program without an explicit user action.

10. Program matching and invalidation
Implement a pure function:

export type ProgramMatchStatus =
  | "exact"
  | "compatible-broader-geography"
  | "stale"
  | "mismatch"
  | "incomplete";

export interface ProgramDimensionMismatch {
  dimension:
    | "modelYear"
    | "make"
    | "model"
    | "trim"
    | "bodyStyle"
    | "drivetrain"
    | "lessor"
    | "condition"
    | "termMonths"
    | "annualMiles"
    | "geography"
    | "effectiveMonth"
    | "creditTier";
  expected?: string | number;
  actual?: string | number;
  code: string;
  message: string;
}

export interface ProgramMatchResult {
  status: ProgramMatchStatus;
  mismatches: ProgramDimensionMismatch[];
  issues: CalculationIssue[];
}

export function matchProgramToDeal(
  program: NormalizedLeaseProgram,
  deal: LeaseDeal,
): ProgramMatchResult;
A deal with no programSnapshot returns incomplete and PROGRAM_MISSING rather than throwing.

Exact-match dimensions
model year;

make;

model;

trim when specified by program;

body style when specified;

drivetrain when specified;

lessor when specified;

condition applicability;

term;

annual miles;

geography;

effective window/program month;

credit tier when specified.

Mandatory behavior
A term change invalidates MF/RV/incentive assumptions.

An annual-mileage change invalidates residual and may invalidate MF/incentives.

SUV and Coupe do not match.

New-only does not match a loaner.

A ZIP-specific source does not automatically become national.

A stale program may be manually retained, but the row displays a persistent warning.

The application must explain every mismatch in plain language.

Never pick the "closest" program silently.

11. Lease calculation engine
11.1 Public API
calculateLease(deal: LeaseDeal, policy?: CalculationPolicy): LeaseCalculationResult
solveTargetDAS(input: TargetDASSolverInput, policy?: CalculationPolicy): TargetDASSolverResult
solveSellingPrice(input: SellingPriceSolverInput, policy?: CalculationPolicy): SellingPriceSolverResult
resolveIncentives(program: NormalizedLeaseProgram, selections: IncentiveSelection[]): ResolvedIncentives
resolveMoneyFactor(program: MoneyFactorProgram, selection: DealMoneyFactorSelection): ResolvedMoneyFactor
resolveResidual(program: ResidualProgram, vehicle: VehicleIdentity): ResolvedResidual
All functions are pure and deterministic.

export interface CalculationPolicy {
  includeDispositionFeeInEffectiveCost: boolean;
  includePositiveTradeEquityInDisplayedDAS: boolean;
}

export const DEFAULT_CALCULATION_POLICY: CalculationPolicy = {
  includeDispositionFeeInEffectiveCost: true,
  includePositiveTradeEquityInDisplayedDAS: false,
};
The web composition root derives the policy from WorkspaceSettings; solvers and direct calculations must use the same policy.

The renamed equity flag is display-only: it must not change adjusted cap cost, payment, or effective cost. When true, the alternate signing-contribution DAS view adds positive trade equity only. Capitalized negative equity remains reflected through the contractual payments.

Normative supporting result shapes:

export interface AppliedMoneyFactorAdjustment {
  adjustmentId: string;
  units: number;
  deltaMicrosPerUnit: MoneyFactorMicros;
  totalDeltaMicros: MoneyFactorMicros;
}

export interface ResolvedMoneyFactor {
  complete: boolean;
  buyRateMfMicros?: MoneyFactorMicros;
  contractBaseMfMicros?: MoneyFactorMicros;
  effectiveMfMicros?: MoneyFactorMicros;
  markupMicros?: MoneyFactorMicros;
  appliedAdjustments: AppliedMoneyFactorAdjustment[];
  skippedAdjustmentIds: string[];
  issues: CalculationIssue[];
}

export interface AppliedResidualAdjustment {
  adjustmentId: string;
  mode: ResidualAdjustment["mode"];
  appliedValue: number;
  residualDeltaCents: MoneyCents;
}

export interface ResolvedResidual {
  complete: boolean;
  baseResidualBps?: PercentBps;
  adjustedResidualBps?: PercentBps;
  residualValueBeforeFixedAdjustmentsCents?: MoneyCents;
  residualValueCents?: MoneyCents;
  appliedAdjustments: AppliedResidualAdjustment[];
  issues: CalculationIssue[];
}

export interface ResolvedIncentiveLine {
  incentiveId: string;
  countedAmountCents: MoneyCents;
  contractApplication: ContractApplication;
  taxTreatment: TaxTreatment;
  replacedIncentiveIds: string[];
}

export interface ResolvedIncentives {
  complete: boolean;
  lines: ResolvedIncentiveLine[];
  categorized: LeaseCalculationResult["categorizedIncentives"];
  ignoredIncentiveIds: string[];
  issues: CalculationIssue[];
}

export interface TargetDASSolverInput {
  deal: LeaseDeal; // must resolve to selling-price mode for forward evaluation
  targetDASCents: MoneyCents;
  toleranceCents?: MoneyCents; // default 1
  maxIterations?: number;      // default 100
}

export interface TargetDASSolverResult {
  complete: boolean;
  achievedDASCents?: MoneyCents;
  targetVarianceCents?: MoneyCents;
  requiredCashDownCents?: MoneyCents;
  capitalizedFeeLineIds: string[];
  minimumAchievableDASCents?: MoneyCents;
  adjustedDeal?: LeaseDeal;
  calculation?: LeaseCalculationResult;
  iterations: number;
  issues: CalculationIssue[];
}

export interface SellingPriceSolverInput {
  deal: LeaseDeal; // must use quoted-payment mode
  minimumSellingPriceCents?: MoneyCents; // default 40% of MSRP
  maximumSellingPriceCents?: MoneyCents; // default 120% of MSRP
  paymentToleranceCents?: MoneyCents;    // default 1
  maxIterations?: number;                // default 100
}

export interface SellingPriceSolverResult {
  complete: boolean;
  impliedSellingPriceCents?: MoneyCents;
  impliedDealerDiscountBps?: PercentBps;
  achievedMonthlyCents?: MoneyCents;
  paymentVarianceCents?: MoneyCents;
  targetDAS?: TargetDASSolverResult;
  calculation?: LeaseCalculationResult;
  iterations: number;
  issues: CalculationIssue[];
}
calculateLease dispatches by deal.inputMode. Selling-price mode runs the forward calculator. Quoted-payment mode calls solveSellingPrice. The solver's internal candidate evaluation must call a private forward-calculation function, not recursively call the public dispatcher.

11.2 Core lease formula
Use exact decimal math.

residual value = MSRP × residual percentage − fixed/demo residual adjustments

gross cap cost = selling price + capitalized fees + capitalized upfront taxes

adjusted cap cost =
  gross cap cost
  − taxed customer cap reductions
  − untaxed customer cap reductions
  − cash down
  − positive trade equity
  + capitalized negative equity
  − applicable DAS credits that are explicitly capitalized

Trade equity is signed net equity. Derive positiveTradeEquityCents = max(tradeEquityCents, 0) and capitalizedNegativeEquityCents = max(−tradeEquityCents, 0).

depreciation charge = (adjusted cap cost − residual value) / term

rent charge = (adjusted cap cost + residual value) × effective MF

pre-tax monthly = depreciation charge + rent charge

post-tax monthly = tax-profile(pre-tax monthly, deal inputs)
11.3 Rounding
Keep full precision through intermediate operations.

Round contractual monthly components to cents using half-up rounding.

Use the rounded contractual monthly payment for total-payment calculations.

Round residual value to cents after all residual adjustments.

Solvers target a payment residual of no more than one cent.

Selling-price solver results are cent-precise integers. The default grid may display impliedSellingPriceCents rounded to the nearest dollar; golden fixtures assert the cent-precise value, and a separate presentation test may assert the nearest-dollar display.

Solver variance reporting: keep full Decimal precision to guide interval selection and bisection, then compute achievedMonthlyCents as the contractually rounded monthly payment and paymentVarianceCents = achievedMonthlyCents − quotedMonthlyCents. Stop when the absolute integer-cent variance is within paymentToleranceCents. Never report a fractional-cent residual under a MoneyCents field name; a raw Decimal payment residual may exist as a debug-only value but is not the canonical paymentVarianceCents and must not affect fixture portability.

11.4 Tax behavior in validated California profile
For the v0.1 California monthly-payment profile:

calculate monthly tax on the taxable monthly lease payment;

calculate tax on incentives explicitly classified as taxed cap reductions;

place tax on taxed cap reductions upfront or capitalize it according to the profile setting;

tax only fee kinds configured as taxable;

preserve a detailed tax breakdown;

compare the implementation against committed fixtures and document any known mismatch.

Other imported tax modes may be stored, displayed, and exported, but must carry an experimental warning until validated.

California taxability is line-specific: do not treat every incentive or fee as taxable merely because it appears in a category. The profile and the normalized line records determine tax treatment. The tax calculation exposes an explicit firstPaymentCents output; for this profile firstPaymentCents = postTaxMonthlyCents.

11.5 Due at signing
Define:

DAS excluding MSD =
  firstPaymentCents (tax-profile output; not universally the post-tax monthly)
  + upfront fees
  + upfront taxes
  + cash down
  − due-at-signing credits

DAS including MSD = DAS excluding MSD + refundable MSD amount
Do not include refundable MSDs in non-refundable effective lease cost.

First contractual payment means the first scheduled periodic installment due at signing after the selected tax profile has applied its upfront-versus-capitalized tax treatment. Separate upfront tax is not part of the periodic installment. In an upfront-total-tax treatment the first installment may be the base periodic payment with tax as a separate upfront line; if upfront tax is capitalized, the scheduled installment contains financing attributable to that capitalized tax. In Texas-style treatment the tax is imposed on the lessor's purchase and subsequent installments are not themselves taxed. No solver or DAS formula may choose between pretax and post-tax payment by inspecting the tax-method name directly.

11.6 Total and effective cost
total non-refundable economic cost =
  contractual monthly payment × term
  + cash down
  + positive trade equity surrendered
  + upfront non-refundable fees and taxes not already included in monthly
  + disposition fee when `CalculationPolicy.includeDispositionFeeInEffectiveCost` is true
  − due-at-signing credits not already reflected in monthly
  − post-sale rebates

effective monthly = total non-refundable economic cost / term

effective monthly / MSRP = effective monthly ÷ MSRP
The ordinary dealer-facing "DAS" display is out-of-pocket cash and excludes trade equity unless the user explicitly chooses an economic-contribution view; that alternate view (`CalculationPolicy.includePositiveTradeEquityInDisplayedDAS`) adds positive trade equity only. Total economic cost adds positive trade equity surrendered because it is customer value given up. Do not add or subtract capitalized negative equity again here — its cost is already reflected in the contractual payments. Negative equity paid separately at signing is not representable by the signed field in v0.1; it requires a separate upfront non-refundable line or an unsupported-state warning.

Display the payment/MSRP ratio in basis points and percentage form.

11.7 MSD handling
Store MSD count, MF delta, and refundable deposit principal separately.

Apply a manufacturer-specific MF reduction only when a reviewed adjustment definition exists and is linked by SecurityDepositProgram.mfAdjustmentId or explicitly selected.

Calculate both pre-MSD and post-MSD payment by running the same engine with and without the MSD adjustment; do not approximate payment savings as cap cost divided by term.

Resolve deposit principal from the program amount rule or explicit deal override. Enforce maximum count and known MF minimum.

If MF reduction is known but deposit amount is unknown, show the payment savings while displaying MSD principal and DAS including MSD as unknown.

Exclude refundable MSD principal from effective monthly and total non-refundable cost.

Do not add opportunity cost to the default effective payment in v0.1.

Output deposit principal, monthly savings, total contractual savings, and enough information for a later optional annualized MSD-return calculation.

11.8 Target-DAS solver
This is a core feature because the user often compares offers stated as "$X per month, $Y DAS."

Algorithm:

Normalize the quote target to an excluding-MSD amount. When dasIncludesMSDs is true, subtract only a known refundable MSD amount. If the quote says DAS includes MSD but the amount cannot be resolved, return an assumption-dependent result rather than guessing.

Calculate zero-cap-reduction DAS from first payment, upfront fees, upfront taxes, credits, and the normalized MSD treatment.

If target DAS is above zero-cap-reduction DAS, solve the balancing cash-down amount. Because payment and first payment change when cash down changes, use bisection until DAS is within one cent.

If target DAS is below the required amount, capitalize eligible fees/taxes in a deterministic order:

acquisition fee;

dealer/service fees;

government fees only if the profile permits;

upfront tax only if the profile permits.

Recalculate the payment and first payment after every capitalization change. Do not subtract a fee from DAS without adding it to gross cap cost.

If due-at-signing credits exceed otherwise due cash, do not create negative drive-off or silently reclassify the excess. Preserve the contractual application and return a warning or unresolved balance.

If the target remains impossible, return an error explaining the minimum achievable DAS and which items cannot legally or operationally be capitalized under the current profile.

Never silently treat MSD or trade equity as non-refundable cash down. Trade equity is an economic contribution but is included in cash-out-of-pocket DAS only when CalculationPolicy.includePositiveTradeEquityInDisplayedDAS is true, and only its positive part.

11.9 Implied-selling-price solver
For quoted-payment mode:

Resolve the program, incentives, fees, tax, DAS assumptions, and MF.

Set a safe selling-price search range, defaulting to 40%–120% of MSRP.

For each candidate selling price, run target-DAS resolution and the full calculation.

Evaluate both endpoints and at least one midpoint before assuming monotonicity. The payment should normally rise with selling price, but target-DAS capitalization and tax boundaries can create piecewise behavior.

Use monotonic bisection only within a verified monotonic interval. If the interval is discontinuous or no sign change brackets the target, return QUOTE_SOLVER_NO_SOLUTION or subdivide deterministically; do not extrapolate.

Stop at one-cent payment tolerance or a maximum iteration count.

Return:

implied selling price;

implied pre-incentive dealer discount;

payment residual;

assumptions used;

confidence/warnings.

If DAS is not itemized, label the result "implied under current DAS assumptions."

If no solution exists in range, return an error instead of extrapolating.

12. Leasehackr URL adapter
12.1 Safety and relationship model
Leasehackr integration is user-directed serialization, not scraping.

Parse the URL in the browser or pure adapter code.

Do not call fetch, axios, any application server endpoint, or browser navigation during import.

Validate the hostname and path.

Preserve unknown parameters in the local import proposal for review.

Generate a link back to Leasehackr for user review.

README language:

Lease Matrix is not affiliated with Leasehackr. It does not access or scrape Leasehackr or Rate Findr. A user may import deal parameters already encoded in a Leasehackr Calculator URL and generate a link back to Leasehackr Calculator for independent review.

12.2 Supported hosts
Allow only these host/path combinations:

leasehackr.com with /calculator;

www.leasehackr.com with /calculator;

calculator.leasehackr.com with / or /calculator.

Normalize accepted URLs to HTTPS for outbound generation. Reject credentials, non-HTTPS URLs in production, JavaScript/data/file URLs, fragments masquerading as query strings, unexpected ports, unsupported paths, and lookalike domains.

12.3 Known parameter map
Implement a versioned parameter registry with aliases and a status for each field: supported-input, supported-source-diagnostic, private-optional, or preserve-only.

Lease Matrix meaning	Known Leasehackr parameter(s)	Treatment
Make	make	supported input
Annual miles	miles	supported input
MSRP	msrp	supported input
Selling price	sales_price	supported input
Term	months	supported input
MF	mf	supported input
MSD count	msd	supported input
Cash down/cap reduction	dp	supported input; positive values map to cash cap reduction. Public links contain negative dp values used as balancing adjustments: a negative value produces an import warning and remains an unresolved source diagnostic unless an explicit canonical meaning is selected
Dealer fee	dealer_fee	supported input
Acquisition fee	acq_fee	supported input
Disposition fee	disp_fee	supported input
Government/registration fee	gov_fee, legacy reg_fee	supported input alias
Service fee	service_fee	supported input
Taxed incentives	taxed_inc	supported input
Untaxed incentives	untaxed_inc	supported input
Post-sale rebate	rebate	supported input
Residual percent	resP	supported input
Sales-tax rate	sales_tax	supported input
Demo mileage	demo_mileage	supported input
Trade equity	tradein	source diagnostic/preserve-only until a nonzero fixture confirms whether it represents net equity, gross trade value, or another quantity
Zero-drive-off selection	zero_driveoff	supported source flag; reconcile through DAS preview
Monthly-payment tax mode	monthlyTax_radio	supported source flag
Total-lease-payment tax mode	totalLeaseTax_radio	supported source flag; experimental tax profile
Capitalized tax flag	cap_tax	supported source flag when validated by fixture
Selling-price tax mode	sellingPriceTax_radio	experimental source tax-method flag
Legacy selling-price tax alias	salesPriceTax_radio	import-only legacy alias of sellingPriceTax_radio
Fees-untaxed flag	fees_untaxed	experimental source flag
New York tax flag	ny_tax	preserve-only until a manual fixture covers its interaction with total-tax mode
LVF result-view flag	lvf_result_mode	preserve-only UI-state alias observed in public URLs
Doc fee (legacy alias)	doc_fee	import-only legacy dealer-fee alias; map to dealer_fee only after a manual fixture confirms semantics
Acquisition/dealer/government fee check flags	acqFee_check, dealerFee_check, govFee_check	preserve and map only when fixture semantics are verified
Private memo	memo	importable private optional field; omitted from fresh export by default
Source-reported pre-tax payment	pretax_monPmt	diagnostic only; never authoritative input automatically
Source-reported DAS	lease_das	diagnostic only; user may opt to use it in quote-reconciliation mode
Result-view flag	lease_result_mode	preserve-only UI state
Manufacturer/demo helper flags	for example bmw_demo_25	preserve-only; never generalize into an OEM rule
Purchase-calculator fields	fin_*, keep_term, exp_rv	preserve-only; out of lease-engine scope
The registry must permit aliases without emitting both aliases. Fresh export uses the currently preferred parameter name; import accepts documented legacy aliases.

12.4 Import result
export interface LeasehackrImportCandidate {
  document: LeaseMatrixDealDocumentV1;
  mappedParams: Record<string, string>;
  unknownParams: Record<string, string[]>;
  sourceDiagnostics: {
    reportedPretaxMonthlyCents?: MoneyCents;
    reportedDASCents?: MoneyCents;
    zeroDriveOffSelected?: boolean;
    resultModeSelected?: boolean;
  };
  adapterVersion: string;
}

export type LeasehackrImportResult = ImportProposal<LeasehackrImportCandidate>;
Import rules:

Empty values stay absent, not zero.

Parse currency as dollars into cents.

Parse resP as percentage into basis points.

Parse MF into millionths.

Boolean parameters accept true, 1, and empty-present forms only when documented by fixture.

Parse pretax_monPmt and lease_das into source diagnostics only. Do not silently convert them into authoritative deal inputs.

The preview may offer Use reported payment/DAS in quote-reconciliation mode; this requires an explicit user choice.

Never infer a manufacturer demo rule from helper flags such as bmw_demo_25.

Do not overwrite the active row until the user reviews an import preview.

If imported values conflict with an active preset, show both and require a choice.

Preserve unknown parameters in the proposal and accepted source provenance; do not silently discard them.

Values outside domain bounds produce preview validation issues; they are never clamped or coerced into valid domain values. No parameter name implies valid content.

Source-reported pretax_monPmt and lease_das may be recalculated by the calculator when a link is reopened; they remain diagnostics and never authoritative inputs.

M3 completion additionally requires the manual validation procedure and vectors recorded in SPEC_ERRATA.md Part 3: local parse of each vector without any network request, import-preview comparison against expected fields, confirmation that diagnostics stay diagnostic, fresh export from the accepted canonical document, manual browser reopen, and semantic round-trip comparison of supported financial inputs (not byte-for-byte URL equality — redirects between leasehackr.com/calculator and calculator.leasehackr.com, parameter reordering, and recalculated output fields do not fail semantic round-trip).

12.5 Export
Generate a standards-compliant URL with URL and URLSearchParams.

Export only known and valid financial fields by default. Omit memo, dealer aliases, VINs, and private notes unless the user explicitly includes them in the export preview.

Omit unknown parameters from a fresh generated URL unless the user explicitly enables the reviewed round-trip option.

When the reviewed round-trip option is enabled, re-append only the displayed and user-approved unknown parameters, never credentials or parameters that conflict with known generated fields.

Use decimal formatting without scientific notation.

Open in a new tab only after an explicit click.

Add unit tests for semantic round-trip of supported fields and explicit unknown-parameter behavior.

12.6 Versioned transfer documents
The protocol package owns a stable wire format separate from internal storage objects. Internal domain models may evolve more quickly; transfer documents change only through explicit versioning and migration.

export interface LeaseMatrixProgramSnapshotV1 {
  key: {
    termMonths: Months;
    annualMiles: Miles;
    geography?: Geography;
    effectiveMonth?: string;
    lessor?: string;
    creditTier?: string;
    conditionApplicability?: ProgramKey["conditionApplicability"];
  };
  moneyFactor: {
    status: MoneyFactorStatus;
    buyRateMicros?: MoneyFactorMicros;
    quotedMfMicros?: MoneyFactorMicros;
    quotedMfBasis?: QuotedMoneyFactorBasis;
    quotedIncludedAdjustmentIds?: string[];
    effectiveMfMicros?: MoneyFactorMicros;
    markupMicros?: MoneyFactorMicros;
    appliedAdjustments: Array<{
      kind: MoneyFactorAdjustmentDefinition["kind"];
      label: string;
      units: number;
      deltaMicrosPerUnit: MoneyFactorMicros;
      totalDeltaMicros: MoneyFactorMicros;
    }>;
  };
  securityDeposit?: {
    status: SecurityDepositProgram["status"];
    maxDeposits?: number;
    amountRule?: SecurityDepositProgram["amountRule"];
    resolvedTotalCents?: MoneyCents;
  };
  residual: {
    status: ResidualStatus;
    baseResidualBps?: PercentBps;
    adjustedResidualBps?: PercentBps;
    fixedAdjustmentCents?: MoneyCents;
  };
  incentives: {
    selected: Array<{
      name: string;
      category: IncentiveCategory;
      amountCents?: MoneyCents;
      amountStatus: IncentiveOffer["amountStatus"];
      contractApplication: ContractApplication;
      taxTreatment: TaxTreatment;
      eligibilityStatus: IncentiveSelection["eligibilityStatus"];
      includesNames?: string[];
      excludesNames?: string[];
    }>;
    totals: {
      taxedCustomerCents: MoneyCents;
      untaxedCustomerCents: MoneyCents;
      dealerSupportCents: MoneyCents;
      dasCreditsCents: MoneyCents;
      postSaleCents: MoneyCents;
      unresolvedCents: MoneyCents;
    };
  };
  acquisitionFeeCents?: MoneyCents;
  dispositionFeeCents?: MoneyCents;
  sourceSummary: Array<{
    provider: SourceProvider;
    sourceLabel?: string;
    observedAt: string;
    effectiveMonth?: string;
    sourceUrl?: string; // sanitized; no memo, raw text, or unreviewed parameters
    confidence: SourceRef["confidence"];
  }>;
}

export interface LeaseMatrixDealDocumentV1 {
  kind: "lease-matrix/deal";
  schemaVersion: 1;
  protocolVersion: 1;
  documentId: string;
  createdAt: string;
  source: {
    kind: "manual" | "leasehackr-url" | "json-import" | "host-integration";
    adapterVersion?: string;
    sourceUrl?: string; // sanitized transfer URL only; original remains in local SourceRef
  };
  deal: {
    vehicle: {
      modelYear?: number;
      make?: string;
      model?: string;
      trim?: string;
      bodyStyle?: BodyStyle;
      condition?: VehicleCondition;
      msrpCents: MoneyCents;
      odometerMiles?: Miles;
    };
    inputMode: DealInputMode;
    programSnapshot?: LeaseMatrixProgramSnapshotV1;
    mfSelection?: DealMoneyFactorSelection;
    feeLines?: FeeLine[];
    taxProfile?: TaxProfile;
    cashDownCents?: MoneyCents;
    tradeEquityCents?: MoneyCents;
    dispositionFeeCents?: MoneyCents;
  };
  privateContext?: {
    dealerAlias?: string;
    salespersonAlias?: string;
    vin?: string;
    notes?: string;
    rawSourceTextByRefId?: Record<string, string>;
  };
  privacy: {
    dealerAliasIncluded: boolean;
    salespersonAliasIncluded: boolean;
    vinIncluded: boolean;
    notesIncluded: boolean;
    rawSourceTextIncluded: boolean;
  };
  extensions?: Record<string, unknown>;
}
Default transfer policy:

exclude dealer alias;

exclude salesperson alias;

convert the internal program into LeaseMatrixProgramSnapshotV1; never embed a full NormalizedLeaseProgram or raw SourceRef in a deal transfer;

exclude VIN;

exclude notes and negotiation history;

exclude raw pasted text;

include only fields needed to reproduce the lease assumptions;

sanitize source.sourceUrl to remove memo, unknown parameters, tracking parameters, and any identifier excluded by the chosen privacy policy;

require an explicit preview and user opt-in before including any excluded identifier.

The Zod schema must refine the document so each privacy.*Included boolean matches the actual presence or absence of the corresponding privateContext field. A document with contradictory privacy metadata is invalid.

Use explicit conversion functions rather than casts:

toDealDocumentV1(deal: LeaseDeal, policy: TransferPrivacyPolicy): LeaseMatrixDealDocumentV1
fromDealDocumentV1(document: LeaseMatrixDealDocumentV1): ImportProposal<Partial<LeaseDeal>>
The conversion into LeaseMatrixProgramSnapshotV1 resolves selected incentives and removes internal revision history, rejected candidates, raw text, internal source IDs, and UI-only state. It retains the final effective MF plus applied adjustment deltas, the final residual inputs/adjustments, resolved MSD principal when known, and categorized incentives so an imported deal can reproduce the contractual calculation without access to the originating program library.

Do not serialize a transfer document into a public URL in v0.1.

The full workspace JSON schema is separate. Its Zod schema, not the TypeScript interface alone, is the source of truth:

export interface LeaseMatrixWorkspaceDocumentV1 {
  kind: "lease-matrix/workspace";
  schemaVersion: 1;
  exportedAt: string;
  appVersion: string;
  workspace: {
    id: string;
    name: string;
    settings: WorkspaceSettings;
    programs: NormalizedLeaseProgram[];
    deals: LeaseDeal[];
    archivedDealIds: string[];
    gridState?: Record<string, unknown>;
    extensions?: Record<string, unknown>;
  };
  extensions?: Record<string, unknown>;
}
12.7 Integration interfaces
Keep these responsibilities separate and keep their package ownership explicit:

// packages/adapters
export interface DealSerializationAdapter<
  TInput = string,
  TParsed = LeaseMatrixDealDocumentV1,
  TOutput = string,
> {
  id: string;
  version: string;
  parse(input: TInput): Promise<ImportProposal<TParsed>>;
  serialize(
    document: LeaseMatrixDealDocumentV1,
    options?: Record<string, unknown>,
  ): Promise<TOutput>;
}

// packages/domain
export interface ProgramProvider {
  id: string;
  lookup(query: ProgramQuery): Promise<ProgramObservation[]>;
}

// packages/integration-protocol
export interface HostBridge {
  mode: "standalone" | "embedded";
  receiveInitialDocument(): Promise<LeaseMatrixDealDocumentV1 | null>;
  publishDocument(document: LeaseMatrixDealDocumentV1): Promise<void>;
  openCanonicalCalculator(url: string): void;
  dispose(): void;
}
P0 implementations:

LeasehackrUrlAdapter implements DealSerializationAdapter<string, LeasehackrImportCandidate, string>.

JSON and CSV adapters use their own import/export interfaces where a full workspace is involved.

StandaloneHostBridge returns no initial document, performs no external publication, and opens a user-confirmed outbound URL with noopener,noreferrer.

No ProgramProvider implementation exists in v0.1.

Future embedded messages are versioned discriminated unions:

export type HostToMatrixMessageV1 =
  | {
      type: "lease-matrix:init";
      protocolVersion: 1;
      document?: LeaseMatrixDealDocumentV1;
      theme?: "light" | "dark";
    }
  | {
      type: "lease-matrix:set-document";
      protocolVersion: 1;
      document: LeaseMatrixDealDocumentV1;
    }
  | {
      type: "lease-matrix:request-export";
      protocolVersion: 1;
      requestId: string;
    };

export type MatrixToHostMessageV1 =
  | {
      type: "lease-matrix:ready";
      protocolVersion: 1;
      capabilities: string[];
    }
  | {
      type: "lease-matrix:document-changed";
      protocolVersion: 1;
      document: LeaseMatrixDealDocumentV1;
    }
  | {
      type: "lease-matrix:resize";
      protocolVersion: 1;
      height: number;
    }
  | {
      type: "lease-matrix:open-calculator";
      protocolVersion: 1;
      url: string;
    };
Every future embedded message must pass:

exact origin allowlist validation;

event.source === window.parent validation;

Zod schema validation;

protocol-version validation;

payload-size limits;

capability checks;

default redaction rules.

Never use * as a postMessage target origin.

12.8 Leasehackr adapter privacy and forward compatibility
Preserve unknown query parameters in the local import proposal for review and diagnostics.

Do not automatically forward unknown parameters into a newly generated URL.

Offer a separate advanced option to round-trip reviewed unknown parameters if a user explicitly chooses it.

Never treat query parameters as executable code or HTML.

Reject URLs containing credentials, non-HTTP(S) schemes, unexpected hosts, or unsupported paths.

Keep the parameter mapping in a versioned table and include the adapter version in provenance.

13. Edmunds paste adapter — v0.1.1 feature flag
Do not fetch Edmunds.

13.1 User flow
User clicks Paste program answer.

User pastes the request context and moderator response.

User may paste a source URL as attribution only.

Parser creates one or more candidates.

Review screen highlights source evidence and confidence.

User corrects fields.

User explicitly saves a reviewed program.

13.2 Parser requirements
Recognize common forms:

.00069 MF

standard MF

64% residual

64%/58% RV

24/10, 24/10K, 24 months/10k

24/7500

ZIP codes;

SUV versus Coupe labels;

$1500 dealer cash;

$2000 regional lease cash;

no lease incentives;

no info on conquest;

multiple term values whose answer order follows the request order.

13.3 Ambiguity behavior
Multiple terms plus multiple MF/RV values may be paired by order only when counts align; mark confidence medium and require review.

Body-style-prefixed answers may become separate candidates.

"Standard MF" remains symbolic.

"No info" maps to unknown.

"No incentives" maps only to the explicitly referenced scope.

Loaner adjustment absent from source remains unknown.

Inclusive incentive descriptions should create a package-total candidate rather than an arithmetic sum.

Never infer a make/model from the URL alone in v0.1.1.

13.4 Feature flag
Use a compile-time public flag:

VITE_ENABLE_EDMUNDS_PASTE=false
Default false in production until parser tests and review UI pass.

14. User experience specification
14.1 Desktop shell
The app is a single focused workspace, not a generic SaaS dashboard.

Top bar:

Lease Matrix wordmark.

workspace name.

active program preset.

Add Deal.

Duplicate.

Import.

Export.

Columns.

Settings.

local-only privacy badge.

Below top bar:

compact filter/status row;

main AG Grid occupying the viewport;

optional right-side row drawer.

14.2 Visual language
Use CSS variables, not hard-coded colors inside components.

Required semantics:

editable input cell: pale yellow background;

inherited program cell: pale neutral gray;

calculated output cell: pale green;

unresolved/warning cell: pale amber with icon;

invalid cell: pale red with icon;

selected row: subtle outline, not a saturated fill.

Density:

default row height: 26 px;

compact header height: 30 px;

12–13 px tabular numeric font;

thin borders;

right-align numeric values;

use tabular numerals;

horizontal scrolling is expected;

pin dealer alias, status, MSRP, and primary quote/payment columns on the left.

14.3 Default visible columns
Keep the default grid useful at 1440 px:

Status

Dealer

New/Loaner

MSRP

Input Mode

Quoted or Calculated Monthly

DAS

Selling Price

Discount %

Effective Monthly

Effective / MSRP

Post-MSD Monthly

MF

RV

Program Status

Leasehackr

Hidden-by-default optional columns:

date;

VIN;

dealer/listing URL;

miles;

pre-tax monthly;

post-tax monthly;

payment variance;

implied selling price;

implied discount;

term;

annual miles;

ZIP;

tax rate;

buy MF;

quoted MF;

MF markup;

MSD count and amount;

taxed customer incentives;

untaxed customer incentives;

dealer support;

acquisition fee;

dealer fee;

government fee;

service fee;

disposition fee;

post-sale rebate;

notes;

source date.

14.4 Cell behavior
Only true inputs are editable.

Program-inherited cells are read-only until "Override for this deal" is selected.

Editing any dependent input recalculates only that row.

Invalid numeric text does not become zero.

Show formatted values while retaining integer canonical data.

Press Enter to commit and move down.

Press Tab to commit and move right.

Escape cancels edit.

Delete clears an optional input after confirmation only for destructive multi-row actions.

Do not implement Enterprise-only fill handles or range-selection behavior.

Comparison highlighting:

Highlight the best complete value in Effective Monthly and Effective / MSRP with a subtle outline/icon, not a full-row fill.

Compare only rows in the same cohort: identical term, annual miles, tax-profile method, and disposition-fee policy. Show Not comparable when cohorts differ.

Exclude incomplete, mismatched, stale-without-override, and error rows from best-value highlighting.

Let the user disable comparison highlighting.

Never declare a row "best" based solely on quoted monthly payment or DAS.

14.5 Row drawer
Tabs:

Summary — quote, calculated payment, variance, effective cost, warnings.

Vehicle — VIN, listing, colors, miles, condition.

Program — term, mileage, geography, effective month, MF, RV, source.

Incentives — individual records, eligibility, stackability, categorized totals.

Fees & tax — itemized fee timing/taxability and tax profile.

Sources — URL/text provenance and review status.

Notes — private notes.

14.6 Add Deal flows
Offer three options:

Blank row.

Paste Leasehackr URL.

Duplicate selected row.

The future Edmunds-paste action belongs under Program, not Add Deal.

14.7 Program preset UI
Preset selector label example:

2026 Mercedes-AMG GLC 43 SUV · Aug 2026 · 24/7.5k · ZIP 95070
Show badges:

Exact

Manual

Stale

Mismatch

Incomplete

Override

A preset change opens a preview of affected rows before applying globally.

14.8 Quote-reconciliation mode
When input mode is quoted payment:

Editable:

quoted monthly;

pre-tax/post-tax toggle;

target DAS;

DAS includes MSD toggle;

known MSD amount;

program and fee assumptions.

Outputs:

implied selling price;

implied discount before incentives;

calculator payment;

payment residual;

assumption warning.

14.9 Responsive behavior
Desktop is primary.

At widths below 900 px, show a compact read-only card list and a banner that full editing works best on desktop.

Do not spend P0 time reproducing the complete grid on mobile.

14.10 Standalone and reserved embed modes
Routes:

/ — full standalone workspace with toolbar, local persistence, imports, and outbound calculator links.

/embed — reserved compact shell. In v0.1 it displays the same calculation surface in a reduced chrome mode but remains non-embeddable by default because the server sends frame-ancestors 'none' unless an explicit origin allowlist is configured.

The route itself must not imply an official partner integration. Show a small Standalone preview or Embedding not enabled label when no authorized host is configured.

The browser bridge is selected at the composition root, not inside grid components:

const hostBridge: HostBridge = createHostBridge({
  route: window.location.pathname,
  allowedOrigins: parseExactOrigins(import.meta.env.VITE_EMBED_ALLOWED_ORIGINS ?? ""),
});
P0 returns StandaloneHostBridge for both routes unless an authorized embedded implementation is explicitly enabled in a later release.

15. State, persistence, and migrations
15.1 Storage abstraction
Use localStorage through an async abstraction:

export interface WorkspaceStorage {
  load(): Promise<WorkspaceEnvelope | null>;
  save(value: WorkspaceEnvelope): Promise<void>;
  exportRaw(): Promise<string | null>;
  clear(): Promise<void>;
}
The implementation may use synchronous localStorage internally, but the async boundary allows IndexedDB or an explicitly authorized cloud adapter later without changing application commands.

Storage key:

lease-matrix:workspace:v1
Storage limits and failure behavior:

Enforce an application soft limit of 2 MiB for the serialized workspace payload and show a warning at 1.5 MiB. Browser quotas vary; never assume a fixed quota merely because localStorage commonly permits more.

Limit each locally preserved raw pasted source to 64 KiB, each notes field to 16 KiB, a user-pasted Leasehackr URL to 32 KiB, a single sanitized transfer document to 256 KiB, and an imported workspace JSON file to 5 MiB.

Reject over-limit input before parsing and return a stable validation issue.

When QuotaExceededError or another write failure occurs, keep the last valid persisted copy, retain the current in-memory workspace, show an actionable error, and offer an immediate JSON download. Never clear storage automatically.

Measure limits in UTF-8 bytes, not JavaScript character count.

Autosave behavior:

debounce 250 ms after the last valid state change;

show Saved, Saving, or Save error;

never save NaN, Infinity, invalid schema state, or partially parsed cell text;

retain the last valid persisted workspace if serialization fails;

do not save host messages before the user accepts an import preview;

do not send storage contents over the network.

15.2 Internal workspace envelope
export interface WorkspaceEnvelope {
  schemaVersion: 1;
  appVersion: string;
  savedAt: string;
  workspace: {
    id: string;
    name: string;
    settings: WorkspaceSettings;
    programs: NormalizedLeaseProgram[];
    deals: LeaseDeal[];
    archivedDealIds: string[];
    columnState: unknown;
    extensions?: Record<string, unknown>;
  };
}
The internal storage envelope is not the public host protocol. JSON export converts it into LeaseMatrixWorkspaceDocumentV1.

15.3 Migrations
Validate every load with Zod.

Keep pure migration functions named vNToVNPlus1.

Unit-test every migration with valid, malformed, and unknown-field fixtures.

On unrecoverable data, offer a download of the raw payload before reset.

Never silently discard unknown fields. Preserve them under extensions where safe or report them in the import preview.

Migrations must not require network access.

15.4 Store architecture
Use one canonical workspace store with typed commands and selectors.

Required command categories:

deal lifecycle;

program lifecycle and revision;

field edits;

source import acceptance/rejection;

incentive selection;

grid preference updates;

persistence reset/restore.

Do not expose unrestricted setState calls to grid renderers. Development builds may use Immer only if it does not obscure domain invariants; otherwise use explicit immutable updates.

16. JSON and CSV import/export
16.1 JSON workspace export
JSON is the full-fidelity financial and workspace portable format. Private source evidence is policy-controlled rather than dumped blindly.

Export LeaseMatrixWorkspaceDocumentV1, not an unversioned dump of Zustand state.

export interface WorkspaceExportPolicy {
  includeDealerAliases: boolean;
  includeSalespersonAliases: boolean;
  includeVins: boolean;
  includeListingUrls: boolean;
  includeNotes: boolean;
  includeRawSourceText: boolean;
  includeOriginalSourceUrls: boolean;
}

export const DEFAULT_WORKSPACE_EXPORT_POLICY: WorkspaceExportPolicy = {
  includeDealerAliases: true,
  includeSalespersonAliases: false,
  includeVins: true,
  includeListingUrls: true,
  includeNotes: false,
  includeRawSourceText: false,
  includeOriginalSourceUrls: false,
};
Use an explicit toWorkspaceDocumentV1(envelope, policy) conversion. With includeOriginalSourceUrls: false, sanitize source URLs by removing private memo fields, unreviewed/unknown parameters, and common tracking parameters. Never implement export as JSON.stringify(store.getState()).

Required behavior:

validate the document kind and schema version;

show an import preview;

support merge, replace, and cancel;

resolve ID collisions through deterministic remapping on merge;

preserve source refs and program snapshots;

preserve safe extension fields;

apply WorkspaceExportPolicy and show a privacy summary before download;

omit raw third-party pasted text, salesperson aliases, private notes, and original unsanitized source URLs by default;

offer separate explicit checkboxes before exporting raw source text, private notes, salesperson aliases, or original source URLs;

default filename: lease-matrix-YYYY-MM-DD.json.

16.2 Single-deal transfer export
Provide a Copy sanitized deal JSON action that creates LeaseMatrixDealDocumentV1 under the default redaction policy.

Before copy/download, show which fields are included and excluded. No single-deal transfer is automatically uploaded or placed in a URL.

16.3 CSV
CSV is a flattened deal interchange format, not a full-fidelity protocol.

Required columns include:

status;

dealer alias;

condition;

model year/make/model/trim/body style;

MSRP;

listing URL;

odometer;

input mode;

quoted monthly;

DAS;

selling price;

discount;

calculated monthly;

effective monthly;

term;

miles/year;

ZIP;

MF status and numeric MF when available;

RV status and numeric RV when available;

taxed incentives;

untaxed incentives;

dealer support;

fees;

VIN;

notes.

Before CSV export, show which identifying columns are included. Default to dealer aliases but exclude salesperson aliases, VINs, listing URLs, and notes unless the user selects them. CSV is easier to forward accidentally than a local workspace backup, so use the stricter default.

CSV security:

Determine escape behavior from the declared CSV column type, never from a permissive generic number parser.

Numeric columns serialize through the canonical numeric serializer: valid negative values such as -1500 remain unescaped; noncanonical or mixed numeric/formula strings are rejected; currency symbols and thousands separators are never exported in machine numeric fields.

Free-text columns strip NUL characters, use RFC 4180-compatible quoting and quote escaping, and prefix a leading apostrophe when the text begins with a formula-trigger character. The trigger set is `=`, `+`, `-`, `@`, tab, carriage return, line feed, and the full-width equivalents of `=`, `+`, `-`, and `@` (current OWASP guidance includes these; no mitigation is universally safe across all spreadsheet programs and subsequent save/reopen cycles).

Reversible import rule: exported dangerous text `=SUM(...)` becomes `'=SUM(...)`; an original literal apostrophe-plus-trigger `'=SUM(...)` becomes `''=SUM(...)`; import of `'<trigger>` removes exactly one exporter-added apostrophe and import of `''<trigger>` preserves one literal apostrophe; unwrapping applies only to declared free-text columns; never evaluate a resulting value as a formula.

Required tests: `-1500` numeric; `=1+1`; `+cmd`; `@SUM(A1:A2)`; leading tab; embedded CR/LF; delimiter and quote breakout attempts; an original literal apostrophe; export → import → export stability.

Strip NUL characters.

Limit imported field and file lengths.

Decode as UTF-8 and report invalid encoding.

Show row-level validation errors before mutation.

Do not interpret formulas from imported CSV.

17. Program and deal conflict handling
When a Leasehackr URL is imported into a row with an active preset:

Parse imported values into a draft snapshot.

Compare MF, RV, term, annual miles, incentives, tax rate, and fees.

Show a side-by-side table:

preset value;

imported value;

current row override;

Offer:

Use imported values for this row;

Keep preset values;

Create a new preset from imported values;

Cancel.

Never silently merge numerically different values.

Preserve source provenance for the chosen values.

18. Validation and warning taxonomy
Every issue must have a stable code.

Minimum codes:
PROGRAM_MISSING
PROGRAM_STALE
PROGRAM_TERM_MISMATCH
PROGRAM_MILEAGE_MISMATCH
PROGRAM_BODY_STYLE_MISMATCH
PROGRAM_CONDITION_MISMATCH
PROGRAM_GEOGRAPHY_MISMATCH
MF_UNKNOWN
MF_STANDARD_WITHOUT_VALUE
MF_BELOW_MINIMUM_CLAMPED
MF_MARKUP_DETECTED
MF_QUOTE_BASIS_UNKNOWN
MF_QUOTE_CONTRADICTION
MSD_COUNT_EXCEEDS_MAXIMUM
MSD_AMOUNT_UNKNOWN
ONE_PAY_UNSUPPORTED
ACQUISITION_FEE_WAIVER_UNSUPPORTED
RV_UNKNOWN
RV_DEMO_ADJUSTMENT_UNKNOWN
RV_ADJUSTMENT_INVALID
INCENTIVE_ELIGIBILITY_UNKNOWN
INCENTIVE_APPLICATION_UNKNOWN
INCENTIVE_TAX_UNKNOWN
INCENTIVE_CONFLICT
INCENTIVE_GRAPH_INVALID
INCENTIVE_OVERLAPPING_PACKAGES
INCENTIVE_PACKAGE_DOUBLE_COUNT_PREVENTED
DEALER_SUPPORT_NOT_APPLIED
TAX_PROFILE_EXPERIMENTAL
DAS_TARGET_IMPOSSIBLE
DAS_MSD_AMOUNT_UNKNOWN
DAS_CREDIT_EXCEEDS_DUE
QUOTE_SOLVER_NO_SOLUTION
QUOTE_SOLVER_ASSUMPTION_DEPENDENT
LEASEHACKR_UNKNOWN_PARAMS
LEASEHACKR_PRESET_CONFLICT
LEASEHACKR_DIAGNOSTIC_VALUE_NOT_APPLIED
TRANSFER_REDACTION_APPLIED
TRANSFER_PRIVACY_METADATA_INVALID
HOST_PROTOCOL_VERSION_UNSUPPORTED
HOST_ORIGIN_REJECTED
HOST_PAYLOAD_TOO_LARGE
INPUT_PAYLOAD_TOO_LARGE
STORAGE_PAYLOAD_TOO_LARGE
STORAGE_QUOTA_EXCEEDED
STORAGE_MIGRATION_FAILED
Errors block a definitive calculation. Warnings allow a calculation but remain visible. Info messages explain transformations.

19. Testing requirements
Tests are part of the product contract. Use synthetic fixtures only.

19.1 Domain schema tests
Cover:

valid and invalid integer primitives;

source provenance and acquisition methods;

exact program-key dimensions;

numeric, standard, unknown, and not-offered MF states;

residual states and adjustments;

incentive inclusion/exclusion graphs;

workspace and transfer-document validation;

malformed extension fields;

migration round trips.

19.2 Lease engine unit tests
Cover at minimum:

ordinary selling-price calculation;

residual based on MSRP;

taxed and untaxed customer incentives;

dealer support excluded from cap reduction by default;

package-total incentive replacing an included component;

choose-one and exclusion conflicts;

post-sale rebate affecting effective cost but not contractual monthly;

disposition-fee inclusion toggled through CalculationPolicy;

capitalized versus upfront fees;

California monthly tax profile;

MSD maximum count, MF floor, fixed deposit rule, rounded-monthly deposit rule, manual override, and unknown deposit amount;

DAS with and without MSD;

explicit ONE_PAY_UNSUPPORTED and ACQUISITION_FEE_WAIVER_UNSUPPORTED behavior in v0.1;

target-DAS solving;

implied-selling-price bisection;

one-cent solver tolerance;

impossible target DAS;

unknown MF or RV producing incomplete results;

demo/loaner unknown adjustment warning;

rounding boundaries;

zero and extreme but valid values;

at least six hand-authored golden fixtures with expected intermediate line items; expected values must not be generated by the implementation under test;

trade-equity golden fixtures: zero trade equity; positive trade equity; capitalized negative equity; positive trade equity with the alternate signing-contribution display enabled;

determinism across repeated runs.

19.3 Leasehackr adapter tests
Use synthetic URLs and golden fixtures.

The three public validation vectors supplied by the architect (SPEC_ERRATA.md Part 3) are manual M3 review artifacts: store them as offline string fixtures for local parse tests, never fetch them in CI, never make them production dependencies, and never copy their deal content into application seed data.

Cover:

supported host and path;

known parameter mapping;

percent and MF conversions;

repeated query parameters;

missing optional fields;

malformed numbers;

over-limit URL rejection before parsing;

unsupported host/scheme/path;

no network calls;

unknown parameters retained in the import proposal;

unknown parameters omitted from fresh export by default;

explicit reviewed round-trip option;

parse → document → generate → parse semantic equivalence for supported fields;

noopener,noreferrer outbound behavior in the web layer.

19.4 Integration protocol tests
Cover:

LeaseMatrixDealDocumentV1 validation;

LeaseMatrixWorkspaceDocumentV1 validation;

default redaction excluding dealer, VIN, notes, and raw source text;

explicit inclusion policy;

unknown protocol version rejection;

message discriminated-union validation;

payload-size rejection helper;

capability negotiation constants;

StandaloneHostBridge no-op behavior;

origin allowlist parser;

guarantee that no protocol module imports React, Zustand, or AG Grid.

19.5 Edmunds paste parser tests, P1 only
Use synthetic paraphrases. Cover:

one term/mileage pair;

several positional term/mileage pairs;

numeric MF;

standard MF;

no incentives versus unknown incentives;

regional scope;

body-style ambiguity;

package total including dealer cash;

incomplete reply;

low-confidence candidate;

no auto-save;

no URL retrieval;

raw text excluded from default export.

19.6 UI and store tests
Cover:

typed command validation;

editable versus calculated cells;

inherited preset values;

program mismatch on term/mileage change;

import preview acceptance and cancel;

row drawer editing;

quote reconciliation;

save status, payload warning threshold, quota failure recovery, and migration error recovery;

column-state persistence;

route composition selecting StandaloneHostBridge;

/embed disabled framing state;

no direct grid mutation of canonical state.

19.7 Playwright smoke tests
Required production-build flows:

Load the app and see seeded synthetic rows.

Add a manual deal and calculate outputs.

Edit MSRP, selling price, payment, DAS, and program assumptions.

Duplicate, archive, and restore a row.

Import a synthetic Leasehackr URL through preview.

Generate and inspect an outbound Leasehackr URL without navigating during the test.

Export JSON, reload, and re-import it.

Reload and verify local persistence.

Open /embed and verify reduced chrome and non-affiliation labeling.

Request /healthz from the production server and receive HTTP 200.

Verify no unexpected third-party network requests during ordinary use.

19.8 Static server integration tests
Start the built server on an ephemeral port and cover:

GET /healthz and HEAD /healthz return 200 and Cache-Control: no-store;

POST /healthz returns 405 with Allow: GET, HEAD;

/, /embed, and an arbitrary extensionless client route return the SPA document;

a missing path with a file extension returns 404 rather than the SPA;

encoded traversal and malformed percent-encoding return 404;

hashed assets use immutable caching and HTML does not;

gzip is used only for accepted compressible responses; `Vary: Accept-Encoding` is present on every compressible representation (gzip or identity, GET and HEAD alike), an existing `Vary` value is preserved or appended rather than replaced, and a gzip response never carries the uncompressed `Content-Length`;

default responses deny framing;

an exact authorized /embed origin changes only frame-ancestors and does not introduce a wildcard;

malformed, wildcard, credential-bearing, path-bearing, or insecure non-local embed origins fail startup;

the server binds 0.0.0.0 and honors the injected PORT;

response logs never contain fixture deal values.

19.9 Performance targets
Initial production load on a normal desktop: no obvious blank-screen delay.

Grid edits for 50 rows: recalculation and paint within 100 ms at the 95th percentile in local profiling.

Calculation engine for one deal: under 5 ms in ordinary cases.

Workspace JSON stays below the 1.5 MiB warning threshold for the seeded and expected 50-deal workflow without raw pasted source text.

No network request is required to load, edit, calculate, save, import a Leasehackr URL, or export a workspace.

20. Accessibility
Keyboard-accessible toolbar and dialogs.

Visible focus states.

Every icon-only action has an accessible label.

Warnings are not color-only; use icon/text/tooltips.

Inputs have labels in the detail drawer.

Dialog focus is trapped and restored.

Use semantic buttons and links.

Test with Playwright accessibility assertions where practical.

21. Security, privacy, and integration hardening
21.1 v0.1 local-first guarantees
No authentication.

No cookies required.

No database.

No analytics, advertising, telemetry, session replay, or remote error reporting.

No deal data sent to Railway; Railway serves static assets and the local web application only.

Browser storage is origin-local but not encrypted. Anyone with access to the same unlocked browser profile, browser extensions with storage access, or an exported file may see the workspace. State this plainly in the privacy dialog.

Provide Export backup, Clear local workspace, and Clear all local data actions. Require confirmation for clear operations and explain that private/incognito storage may be deleted by the browser.

No external source is fetched during import.

No user-entered HTML is rendered.

All URLs are parsed with the platform URL API and validated against explicit host/path rules.

Outbound links use a user click plus noopener,noreferrer.

JSON and CSV imports are validated before state mutation.

CSV formula injection is neutralized.

Raw third-party text and private notes are excluded from transfer documents by default.

README and UI state clearly that the project is not affiliated with Leasehackr or Edmunds.

21.2 Static-server headers
All ordinary routes use:

Strict-Transport-Security in production;

X-Content-Type-Options: nosniff;

Referrer-Policy: strict-origin-when-cross-origin;

a restrictive Permissions-Policy;

Content-Security-Policy with default-src 'self', no objects, a self-only connection policy, and frame-ancestors 'none';

X-Frame-Options: DENY when framing is denied.

Do not add 'unsafe-eval' to the CSP. `style-src 'unsafe-inline'` is accepted for v0.1 because AG Grid's current theming system injects style elements by default (Tailwind is not the justification); nonce-based hardening via AG Grid's `styleNonce` is out of scope for v0.1. `img-src` must include `data:` because AG Grid's current themes use data URLs for SVG icons. `script-src 'self'` must hold: the Vite production index.html contains no inline executable script, the application loads under `script-src 'self'`, and no dependency requires `unsafe-eval`. Do not use AG Grid string expressions for value getters, class rules, or similar options; use functions only.

21.3 Reserved embed security
The default deployment is not frameable.

A future authorized embed may set EMBED_ALLOWED_ORIGINS to a comma-separated list of exact origins. Requirements:

accept only normalized http://localhost:<port> development origins or production https:// origins;

reject wildcards, path components, credentials, and opaque origins;

for /embed, emit frame-ancestors containing only the configured origins;

omit X-Frame-Options only on the authorized embed response because it cannot express arbitrary origin allowlists;

keep all other routes at frame-ancestors 'none' and X-Frame-Options: DENY;

validate every message origin and source;

never use postMessage(..., "*");

keep default transfer redaction even for authorized hosts;

require the server EMBED_ALLOWED_ORIGINS and browser VITE_EMBED_ALLOWED_ORIGINS allowlists to contain the same normalized origins; the bridge fails closed on mismatch;

add an explicit integration-specific privacy review before enabling production framing.

21.4 Build and supply-chain controls
Commit the lockfile.

Use frozen installs in CI and Railway.

Pin GitHub Action major versions and review updates.

Enable Dependabot for npm/pnpm and Actions.

Do not copy secrets, .pgpass, .env, auth state, screenshots containing private data, or generated debug artifacts from Prime Radiant.

Add .env*, Playwright auth directories, coverage, build output, and local workspace exports to .gitignore.

Before the first public push, inspect git ls-files, run a secret scan with an available reputable scanner, and fail if tracked paths or contents resemble .env, .pgpass, private keys, access tokens, authenticated browser state, or real deal exports.

Enable GitHub secret scanning and push protection when repository settings permit.

SECURITY.md directs vulnerability reports to GitHub private vulnerability reporting/security advisories rather than public issues; do not invent an email address.

21.5 Future dealer URL importer
If v0.3 is approved, implement it as a separate service with:

one explicit user-triggered fetch;

DNS and redirect revalidation;

private, loopback, link-local, multicast, and metadata-IP blocking;

scheme allowlist;

timeout and response-size limits;

limited redirects;

content-type checks;

structured-data-first extraction;

no cookies, login, CAPTCHA bypass, fingerprint spoofing, or headless evasion;

per-user/IP rate limiting if accounts or a backend exist;

no storage of fetched pages by default.

22. Repository bootstrap and OpenCode setup
22.1 Bootstrap commands
Execute an equivalent sequence:

mkdir lease-matrix
cd lease-matrix
git init -b main
corepack enable

pnpm init
mkdir -p apps packages docs .github/workflows .railway .opencode
pnpm create vite apps/web --template react-ts
Then:

set the root package name to lease-matrix and private: true;

pin the exact pnpm 10 version in packageManager;

create pnpm-workspace.yaml with apps/* and packages/*;

add Turbo and the root TypeScript/test tooling;

rename the web package to @lease-matrix/web;

create the four framework-neutral packages and fixtures package;

initialize Tailwind and local shadcn components in apps/web using the current Vite-compatible CLI;

remove scaffold logos, demo counters, and sample CSS before the first product commit.

22.2 Root scripts
Root package.json must contain equivalent scripts:

{
  "private": true,
  "scripts": {
    "dev": "turbo run dev --parallel",
    "build": "turbo run build",
    "start": "pnpm --filter @lease-matrix/web start",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "turbo run lint",
    "typecheck": "turbo run typecheck",
    "test": "turbo run test",
    "test:coverage": "turbo run test:coverage",
    "test:e2e": "pnpm --filter @lease-matrix/web test:e2e",
    "verify": "pnpm format:check && pnpm lint && pnpm typecheck && pnpm test && pnpm build && pnpm test:e2e"
  }
}
Adjust syntax only as required by the pinned pnpm/Turbo versions. Preserve one canonical pnpm verify gate.

22.3 Turbo tasks
turbo.json must define at least:

dev: persistent, uncached;

build: depends on upstream builds, outputs dist/**;

lint;

typecheck;

test: may depend on upstream builds, outputs coverage when applicable;

test:coverage.

Do not create a large Makefile. Package scripts are the primary interface.

22.4 OpenCode configuration
Create a minimal opencode.json:

{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["./AGENTS.md"],
  "agent": {
    "build": {
      "mode": "primary",
      "prompt": "{file:./.opencode/build-agent.md}"
    }
  }
}
AGENTS.md should remain under roughly 250 lines and contain:

project thesis and non-goals;

package boundaries;

no-scraping policy;

synthetic-fixture policy;

required commands;

Railway desired state;

commit/milestone workflow;

instruction to read spec.md before architectural work.

.opencode/build-agent.md should say:

follow AGENTS.md and spec.md;

inspect before editing;

implement and verify rather than only describing;

run pnpm verify before handoff;

never claim a remote action succeeded without evidence.

Do not require semantic indexing, worktrees, Beads, Cube verification, or brownfield architecture maps in this greenfield repository. Beads may be added later if the maintainer explicitly asks for it.

22.5 GitHub creation
After the first green commit:

gh auth status
PERSONAL_OWNER="$(gh api user --jq .login)"
REPO_OWNER="$PERSONAL_OWNER"

# Prefer the existing project organization when the token can create repositories there.
# Verify permission with the API or a harmless repository-existence check; do not guess.
if gh api orgs/stars-end >/dev/null 2>&1; then
  REPO_OWNER="stars-end"
fi

if gh repo view "$REPO_OWNER/lease-matrix" >/dev/null 2>&1; then
  gh repo clone "$REPO_OWNER/lease-matrix" /tmp/lease-matrix-existing-check
  # Inspect before adding or replacing any remote state.
else
  gh repo create "$REPO_OWNER/lease-matrix" \
    --public \
    --source=. \
    --remote=origin \
    --push \
    --description "Local-first spreadsheet workspace for comparing vehicle lease offers"
fi
The organization-read check alone does not prove repository-creation permission. Before a create attempt, inspect the authenticated token scopes and organization role; if creation in stars-end is rejected, fall back to PERSONAL_OWNER rather than weakening visibility or deleting anything. If the repository already exists, do not recreate or overwrite it.

Recommended settings:

Issues enabled.

Discussions optional.

Wiki disabled.

Delete branches on merge enabled.

Require the CI workflow before merging to main when permissions allow.

Prefer squash merge for milestone PRs, while direct milestone commits are acceptable for the initial autonomous build.

22.6 Required repository documents
README must include:

product screenshot or GIF after UI exists;

local setup and pnpm verify;

calculation disclaimer;

privacy behavior;

Leasehackr non-affiliation and no-scraping statement;

Edmunds no-scraping statement;

architecture overview;

supported and unsupported calculation modes;

JSON/CSV portability;

roadmap;

contribution instructions;

Railway deployment instructions.

CONTRIBUTING.md must require synthetic fixtures and prohibit scraped datasets or copied forum posts.

SECURITY.md must direct reports away from public issues.

23. GitHub Actions CI
Create .github/workflows/ci.yml with pull-request and main push triggers.

Use a lean gate, not the large Prime Radiant workflow matrix.

Required jobs:

23.1 Static and unit gate
Ubuntu runner.

Node 24.

Corepack and pinned pnpm.

pnpm cache.

pnpm install --frozen-lockfile.

pnpm format:check.

pnpm lint.

pnpm typecheck.

pnpm test with coverage summary.

pnpm build.

23.2 Browser smoke gate
depends on the build/static gate;

install Playwright Chromium using the pinned Playwright command;

run production-build Playwright tests;

upload the Playwright report, trace, screenshot, and console log on failure.

23.3 CI requirements
Do not require Railway credentials for ordinary CI.

Do not run any test that fetches Leasehackr, Edmunds, dealer sites, or arbitrary external URLs.

Use synthetic fixtures.

Cache package downloads, not node_modules.

Fail on type errors, failing tests, or build failures.

A scheduled dependency audit may be added after launch, but it must not grow into an app-specific release framework.

Add Dependabot for npm/pnpm and GitHub Actions weekly updates.

24. Railway infrastructure
24.1 v0.1 topology
Browser
  ├─ Vite React workspace
  ├─ localStorage workspace
  ├─ pure lease engine
  ├─ local URL/file/text adapters
  ├─ StandaloneHostBridge
  └─ user-clicked outbound Leasehackr link

Railway project: lease-matrix
  └─ service: web
       ├─ builds pnpm/Turbo monorepo from repo root
       ├─ serves apps/web/dist with a small Node server
       ├─ GET /healthz → 200
       ├─ one replica in US West when available
       └─ no database, volume, worker, cron, or bucket
24.2 Web package scripts
apps/web/package.json must contain equivalent scripts:

{
  "name": "@lease-matrix/web",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "start": "node ./scripts/serve-static.mjs",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test"
  }
}
The server must read process.env.PORT, default to a local development port, and listen on 0.0.0.0.

apps/web/playwright.config.ts must test the production build rather than Vite dev mode:

import { defineConfig, devices } from "@playwright/test";

const port = Number.parseInt(process.env.PORT ?? "4173", 10);
const baseURL = process.env.BASE_URL ?? `http://127.0.0.1:${port}`;

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: Boolean(process.env.CI),
  retries: process.env.CI ? 2 : 0,
  reporter: process.env.CI ? [["html", { open: "never" }], ["list"]] : "list",
  use: {
    baseURL,
    trace: "retain-on-failure",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },
  webServer: process.env.BASE_URL
    ? undefined
    : {
        command: "pnpm start",
        url: `${baseURL}/healthz`,
        reuseExistingServer: !process.env.CI,
        timeout: 120_000,
      },
  projects: [{ name: "chromium", use: { ...devices["Desktop Chrome"] } }],
});
The default test command therefore requires a completed vite build. Remote-deployment smoke tests set BASE_URL and do not start a local server.

24.3 Static server requirements
apps/web/scripts/serve-static.mjs must:

serve apps/web/dist;

return the SPA index.html for non-file routes;

return JSON from /healthz with Cache-Control: no-store;

support GET and HEAD, reject unsupported methods;

prevent path traversal;

set correct MIME types;

cache hashed /assets/ files immutably;

serve HTML with no-cache;

support gzip when accepted or use a verified equivalent;

set the security headers in section 21;

deny framing everywhere by default;

apply an exact frame-ancestors allowlist only to /embed when EMBED_ALLOWED_ORIGINS is non-empty and valid;

produce useful startup and error logs without logging deal data.

Normative reference implementation (the agent may refactor it only if the resulting server preserves the same behavior and tests):

import { createReadStream, existsSync, statSync } from "node:fs";
import { createServer } from "node:http";
import { extname, isAbsolute, relative, resolve } from "node:path";
import { fileURLToPath } from "node:url";
import { createGzip } from "node:zlib";

const scriptDir = fileURLToPath(new URL(".", import.meta.url));
const distDir = resolve(scriptDir, "../dist");
const port = Number.parseInt(process.env.PORT ?? "4173", 10);
const production = process.env.NODE_ENV === "production";

if (!Number.isInteger(port) || port < 1 || port > 65535) {
  throw new Error("PORT must be an integer from 1 to 65535");
}

const mimeTypes = new Map([
  [".css", "text/css; charset=utf-8"],
  [".html", "text/html; charset=utf-8"],
  [".ico", "image/x-icon"],
  [".js", "text/javascript; charset=utf-8"],
  [".json", "application/json; charset=utf-8"],
  [".map", "application/json; charset=utf-8"],
  [".png", "image/png"],
  [".svg", "image/svg+xml"],
  [".txt", "text/plain; charset=utf-8"],
  [".webmanifest", "application/manifest+json; charset=utf-8"],
  [".webp", "image/webp"],
  [".woff", "font/woff"],
  [".woff2", "font/woff2"],
]);

const compressibleExtensions = new Set([
  ".css",
  ".html",
  ".js",
  ".json",
  ".map",
  ".svg",
  ".txt",
  ".webmanifest",
]);

function parseExactOrigins(raw) {
  const origins = new Set();

  for (const token of raw.split(",").map((value) => value.trim()).filter(Boolean)) {
    let url;
    try {
      url = new URL(token);
    } catch {
      throw new Error(`Invalid EMBED_ALLOWED_ORIGINS entry: ${token}`);
    }

    const localHttp =
      url.protocol === "http:" &&
      (url.hostname === "localhost" ||
        url.hostname === "127.0.0.1" ||
        url.hostname === "[::1]");

    if (
      (url.protocol !== "https:" && !localHttp) ||
      url.username ||
      url.password ||
      url.pathname !== "/" ||
      url.search ||
      url.hash ||
      token.includes("*")
    ) {
      throw new Error(`Unsafe EMBED_ALLOWED_ORIGINS entry: ${token}`);
    }

    origins.add(url.origin);
  }

  return [...origins];
}

const embedOrigins = parseExactOrigins(process.env.EMBED_ALLOWED_ORIGINS ?? "");

function isEmbedPath(pathname) {
  return pathname === "/embed" || pathname.startsWith("/embed/");
}

function hasTraversalAttempt(rawUrl) {
  const rawPath = (rawUrl.split("?", 1)[0] ?? "/").split("#", 1)[0] ?? "/";
  let decoded = rawPath;

  for (let pass = 0; pass < 3; pass += 1) {
    let next;
    try {
      next = decodeURIComponent(decoded);
    } catch {
      return true;
    }

    const normalizedSeparators = next.replaceAll("\\", "/");
    if (
      normalizedSeparators.includes("\0") ||
      normalizedSeparators.split("/").some((segment) => segment === "..")
    ) {
      return true;
    }

    if (next === decoded) break;
    decoded = next;
  }

  return false;
}

function securityHeaders(pathname) {
  const embedAllowed = isEmbedPath(pathname) && embedOrigins.length > 0;
  const frameAncestors = embedAllowed ? embedOrigins.join(" ") : "'none'";

  return {
    ...(production
      ? { "Strict-Transport-Security": "max-age=31536000; includeSubDomains" }
      : {}),
    "X-Content-Type-Options": "nosniff",
    "Referrer-Policy": "strict-origin-when-cross-origin",
    "Permissions-Policy":
      "camera=(), microphone=(), geolocation=(), payment=(), usb=()",
    "Cross-Origin-Resource-Policy": "same-origin",
    "X-Permitted-Cross-Domain-Policies": "none",
    "Content-Security-Policy": [
      "default-src 'self'",
      "base-uri 'self'",
      "object-src 'none'",
      `frame-ancestors ${frameAncestors}`,
      "script-src 'self'",
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: blob:",
      "font-src 'self' data:",
      "connect-src 'self'",
      "worker-src 'self' blob:",
      "manifest-src 'self'",
      "form-action 'self'",
    ].join("; "),
    ...(embedAllowed ? {} : { "X-Frame-Options": "DENY" }),
  };
}

function resolveRequestPath(pathname) {
  let decoded;
  try {
    decoded = decodeURIComponent(pathname);
  } catch {
    return null;
  }

  if (decoded.includes("\0")) return null;

  const requested = resolve(distDir, `.${decoded === "/" ? "/index.html" : decoded}`);
  const relativePath = relative(distDir, requested);

  if (relativePath.startsWith("..") || isAbsolute(relativePath)) return null;

  if (existsSync(requested) && statSync(requested).isFile()) return requested;

  if (extname(decoded)) return null;

  const fallback = resolve(distDir, "index.html");
  return existsSync(fallback) ? fallback : null;
}

function sendError(response, pathname, statusCode, message) {
  response.writeHead(statusCode, {
    ...securityHeaders(pathname),
    "Content-Type": "text/plain; charset=utf-8",
    "Cache-Control": "no-store",
  });
  response.end(message);
}

const server = createServer((request, response) => {
  const rawUrl = request.url ?? "/";
  const method = request.method ?? "GET";

  if (hasTraversalAttempt(rawUrl)) {
    sendError(response, "/", 404, "Not Found");
    return;
  }

  let url;
  try {
    url = new URL(rawUrl, "http://localhost");
  } catch {
    sendError(response, "/", 400, "Bad Request");
    return;
  }

  if (method !== "GET" && method !== "HEAD") {
    response.writeHead(405, {
      ...securityHeaders(url.pathname),
      Allow: "GET, HEAD",
      "Cache-Control": "no-store",
    });
    response.end();
    return;
  }

  if (url.pathname === "/healthz") {
    const body = JSON.stringify({ ok: true, service: "lease-matrix-web" });
    response.writeHead(200, {
      ...securityHeaders(url.pathname),
      "Content-Type": "application/json; charset=utf-8",
      "Cache-Control": "no-store",
      "Content-Length": Buffer.byteLength(body),
    });
    response.end(method === "HEAD" ? undefined : body);
    return;
  }

  const filePath = resolveRequestPath(url.pathname);
  if (!filePath) {
    sendError(response, url.pathname, 404, "Not Found");
    return;
  }

  const extension = extname(filePath);
  const stat = statSync(filePath);
  const isHashedAsset = filePath.includes(`${distDir}/assets/`);
  const isCompressible = compressibleExtensions.has(extension);
  const acceptsGzip =
    method === "GET" &&
    isCompressible &&
    request.headers["accept-encoding"]?.includes("gzip");

  const headers = {
    ...securityHeaders(url.pathname),
    "Content-Type": mimeTypes.get(extension) ?? "application/octet-stream",
    "Cache-Control": isHashedAsset
      ? "public, max-age=31536000, immutable"
      : "no-cache",
    ...(isCompressible ? { Vary: "Accept-Encoding" } : {}),
    ...(acceptsGzip
      ? { "Content-Encoding": "gzip" }
      : { "Content-Length": stat.size }),
  };

  response.writeHead(200, headers);

  if (method === "HEAD") {
    response.end();
    return;
  }

  const stream = createReadStream(filePath);
  stream.on("error", (error) => {
    console.error("Static file stream failed", error);
    response.destroy(error);
  });

  if (acceptsGzip) {
    stream.pipe(createGzip()).pipe(response);
  } else {
    stream.pipe(response);
  }
});

server.listen(port, "0.0.0.0", () => {
  console.log(`Lease Matrix web server listening on 0.0.0.0:${port}`);
});
The production implementation must include integration tests for every requirement above. Do not weaken the CSP with unsafe-eval, wildcard frame ancestors, or wildcard connect-src values merely to make a test pass.

24.4 Railway desired state
Use .railway/railway.ts. Do not invent an obsolete API from memory.

Normative desired state:

Setting	Value
Project	lease-matrix
Environment	production
Service	web
Source	created GitHub repository, branch main
Builder	Railpack unless the current generated IaC requires another explicit value
Build command	corepack enable && pnpm install --frozen-lockfile && pnpm turbo run build --filter=@lease-matrix/web...
Start command	pnpm --filter @lease-matrix/web start
Health path	/healthz
Health timeout	120 seconds
Replicas	1
Region	US West when the current API supports explicit selection
Runtime variables	NODE_ENV=production, VITE_ENABLE_EDMUNDS_PASTE=false, EMBED_ALLOWED_ORIGINS=, VITE_EMBED_ALLOWED_ORIGINS=

Variable scope annotation: `EMBED_ALLOWED_ORIGINS` is a Railway process runtime value read by the Node static server; `VITE_EMBED_ALLOWED_ORIGINS` and `VITE_ENABLE_EDMUNDS_PASTE` are browser bundle build-time values that Vite statically replaces during the production build. Changing a `VITE_*` value requires a new build and deployment — annotate this in `.railway/railway.ts` comments. In v0.1 both embed allowlists stay empty, the Edmunds flag stays false, and runtime-config injection is out of scope.
Volume	none
Database/Redis	none
Cron/worker	none
Public domain	generated Railway domain
Watch paths should include, when supported by the current IaC API:

/apps/web/**
/packages/domain/**
/packages/lease-engine/**
/packages/adapters/**
/packages/integration-protocol/**
/package.json
/pnpm-lock.yaml
/pnpm-workspace.yaml
/turbo.json
/.railway/**
24.5 IaC workflow
Use the current Railway CLI and generated TypeScript API:

railway login
railway init
railway link
railway config init
Then inspect the generated .railway/railway.ts and package metadata. Replace the generated sample desired state with the table above while preserving the current generated imports and API syntax.

Validate before applying:

railway config plan
The plan must create or update only:

one project/environment as intended;

one web service;

its GitHub source, build/start commands, variables, healthcheck, replica count, region, and domain.

It must not create a database, volume, bucket, cron, worker, or destructive unrelated change.

Apply only after reviewing the plan:

railway config apply
railway domain
Do not use destructive-confirmation flags unless the plan is understood and intentionally changes an existing resource.

24.6 Deployment verification
After deployment:

curl -fsS https://<generated-domain>/healthz
BASE_URL=https://<generated-domain> pnpm --filter @lease-matrix/web test:e2e
Verify:

/healthz returns 200;

/ loads the production build;

refresh on a client-side route returns the SPA;

/embed loads but remains non-frameable by default;

local storage persists within the browser;

no lease data appears in Railway logs;

no third-party requests occur during ordinary use;

the deployed SHA matches the reported commit.

Railway healthchecks run during deployment readiness and are not continuous monitoring. Do not claim otherwise.

24.7 Future import service boundary
Do not create this in v0.1. If v0.3 is approved, add a separate apps/import-api TypeScript service, preferably Hono or Fastify, so it can share Zod schemas. Do not add the importer to the static web server.

25. Seed fixtures
Create four fictional GLC-like example deals without real dealer names, URLs, VINs, or private data.

Examples:

Example Dealer A — new vehicle, selling-price mode.

Example Dealer B — loaner with unknown residual adjustment warning.

Example Dealer C — quoted-payment mode and implied discount.

Example Dealer D — incentive package demonstrating inclusive dealer support.

Clearly label every seeded program:

Synthetic example — not current program data
Do not present fixture rates as live market facts.

26. Documentation requirements
docs/architecture.md
Document:

Vite SPA topology;

package dependency direction;

grid/store boundary;

local-first data flow;

adapter, provider, and host-bridge separation;

Railway topology;

future importer service boundary.

docs/calculations.md
Document:

core formula;

rounding;

tax assumptions;

DAS and effective cost;

MSD handling;

forward and inverse solvers;

known gaps versus captive contracts and external calculators.

docs/data-model.md
Document:

RV/MF/program key;

incentive application and package semantics;

program observations and review states;

deal snapshots and overrides;

source provenance;

extension-field policy.

docs/integration-protocol.md
Document:

LeaseMatrixDealDocumentV1;

LeaseMatrixWorkspaceDocumentV1;

redaction defaults;

message schemas and capabilities;

DealSerializationAdapter, ProgramProvider, and HostBridge boundaries;

standalone behavior;

conditions required before enabling an authorized iframe or deep link;

origin validation and CSP requirements.

docs/privacy-and-sources.md
Document:

no scraping policy;

local Leasehackr URL parsing;

Edmunds paste-only policy;

no raw third-party text in default exports;

no affiliation claims;

future dealer importer safeguards;

synthetic-fixture requirement.

docs/decisions.md
Include ADRs:

Vite + React instead of Next.js.

Local-first v0.1 with no backend/database.

AG Grid Community only.

Integer persisted values plus Decimal intermediate math.

Separate deal adapter, program provider, and host bridge.

Versioned protocol and default redaction.

Railway .railway/railway.ts and one stateless service.

Selective Prime Radiant pattern reuse without brownfield baggage.

docs/release-checklist.md
Mirror the definition of done and include exact local, CI, GitHub, and Railway verification commands.

27. Milestone and commit plan
Do not create eleven layers of gates before a usable product. Each milestone ends with working software and the smallest relevant verification set.

Milestone 0 — Scaffold and green toolchain
Deliver:

repository and branch;

pnpm workspace and Turbo;

Vite web app;

framework-neutral packages;

strict TypeScript;

minimal OpenCode files;

formatting, lint, typecheck, Vitest, Playwright configuration;

placeholder /healthz production server;

MIT and basic README.

Gate:

pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
Commit: chore: scaffold lease matrix monorepo

Milestone 1 — Domain, protocol, and lease engine
Deliver:

integer primitives;

canonical domain schemas;

incentive resolver;

MF and RV resolution;

lease calculation engine;

target-DAS and implied-price solvers;

transfer schemas and redaction;

synthetic fixtures;

unit tests.

Gate:

pnpm --filter @lease-matrix/domain test
pnpm --filter @lease-matrix/lease-engine test
pnpm --filter @lease-matrix/integration-protocol test
pnpm typecheck
Commit: feat: add lease domain engine and protocol

Milestone 1a — Calculation semantics gate (human review)
Run after domain schemas and the forward-calculation skeleton exist, before the inverse solvers and the full golden-fixture set. Purpose: catch sign and rounding mistakes before they spread through the target-DAS and implied-price solvers.

Required evidence:

one simple no-tax/no-fee forward fixture;

one California monthly-tax fixture;

positive-equity and negative-equity hand calculations;

confirmation of component rounding and final monthly rounding;

confirmation that first-payment semantics come from the tax-profile output (`firstPaymentCents`), not a global post-tax assumption.

Milestone 2 — Workspace and grid
Deliver:

Zustand store and typed commands;

persistence and migration;

AG Grid Community workspace;

cell-state styling;

program presets;

row drawer;

forward and quote-reconciliation modes;

seeded examples.

Gate:

pnpm test
pnpm --filter @lease-matrix/web test
pnpm build
Commit: feat: build local lease comparison workspace

Milestone 3 — Import/export and Leasehackr handoff
Deliver:

Leasehackr local URL parser;

import preview;

outbound URL generator;

JSON workspace import/export;

sanitized single-deal transfer export;

CSV import/export;

conflict resolution;

adapter/protocol tests.

Gate:

pnpm test
pnpm typecheck
pnpm build
Commit: feat: add safe lease data interoperability

Milestone 4 — Quality, security, and browser flows
Deliver:

accessibility pass;

keyboard navigation;

responsive fallback;

production static server;

security headers;

/embed reserved route;

Playwright production smoke tests;

complete docs;

screenshot or GIF with synthetic data.

Gate:

pnpm verify
Commit: test: complete v0.1 release gate

Milestone 5 — GitHub and Railway release
Deliver:

public GitHub repository;

CI green;

.railway/railway.ts generated with current syntax and reviewed desired state;

Railway plan and apply;

generated domain;

deployed smoke test;

v0.1.0 tag and release notes.

Gate:

curl -fsS https://<domain>/healthz
BASE_URL=https://<domain> pnpm --filter @lease-matrix/web test:e2e
git status --short
git rev-parse HEAD
Commit, if deployment metadata requires a final change: chore: finalize railway deployment

Milestone 6 — v0.1.1 only after release
Do not begin until v0.1.0 is deployed and dogfooded.

Possible work:

Edmunds paste parser;

advanced incentive editor;

program library and revision UI;

authorized embedded bridge after partner approval.

28. Definition of done for v0.1
Product
Dense Google-Sheets-like grid is the primary surface.

True inputs are editable and visually distinct.

Inherited program values and calculated outputs are distinct.

Forward and quote-reconciliation modes work.

Program mismatch/stale/unknown states are visible.

RV/MF/incentive semantics and provenance are preserved.

Leasehackr URL import is local and previewed.

Leasehackr URL export is user-triggered and clearly unofficial.

JSON workspace and sanitized deal transfer exports work.

CSV import/export works safely.

Browser reload restores local data.

A different browser has no data because there is no cloud storage.

/embed exists as a reserved mode but is non-frameable by default.

Engineering
Vite + React; no Next.js runtime.

pnpm/Turbo monorepo.

strict TypeScript passes.

Package boundaries pass dependency checks or review.

Pure lease engine has comprehensive tests.

Integration protocol and redaction have tests.

AG Grid Community only.

No financial math in React components.

No scraper or hidden external fetch.

No real dealer/private data in repository history.

Production static server passes path, health, cache, and header tests.

Playwright smoke tests pass.

pnpm verify passes.

Infrastructure
Public GitHub repository exists.

MIT license exists.

CI is green.

.railway/railway.ts exists and validates with the current Railway CLI.

No new railway.toml or railway.json exists.

Railway plan contains one stateless web service only.

/healthz returns 200.

No database, volume, worker, cron, or bucket exists.

Railway deployment works, or the only incomplete action is authenticated apply/domain generation and is reported accurately.

README contains exact local and Railway setup.

Release
Tag v0.1.0 only after all gates pass.

Release notes say calculations are estimates and tax support is California-first.

Release notes say no external source is scraped.

Release notes say the project is not affiliated with Leasehackr or Edmunds.

Final report includes repo URL, deployment URL, SHA, tests, limitations, and authentication-only blockers.

29. Acceptance scenarios
Scenario A — Manual target deal
Given a new vehicle, MSRP, selling price, term, annual miles, numeric MF, numeric RV, tax rate, fees, and customer incentive, the app calculates pre-tax payment, post-tax payment, DAS, total cost, effective monthly, discount, and payment/MSRP ratio.

Scenario B — Dealer quote reverse solve
Given MSRP, quoted post-tax payment, target DAS, program, fees, and tax assumptions, the app solves an implied selling price and pre-incentive discount and reports the payment residual and assumption warning.

Scenario C — Dealer cash
Given a $1,500 dealer-cash observation and a negotiated selling price, dealer cash is displayed as dealer support and does not also reduce cap cost unless the user explicitly reclassifies it from contract evidence.

Scenario D — Inclusive affinity package
Given dealer cash of $1,500 and an affinity package total of $2,000 that includes dealer cash, selecting the package counts $2,000 total, not $3,500. The breakdown shows the inclusion relationship.

Scenario E — Loaner uncertainty
Given a loaner row and a source that says loaner information is unavailable, the app does not infer a residual penalty. It shows RV_DEMO_ADJUSTMENT_UNKNOWN and refuses to claim an exact result unless the user supplies the rule/value.

Scenario F — Program mismatch
Given a 24/7,500 preset, changing the row to 36/10,000 invalidates the program and shows term/mileage mismatch. It does not retain the old RV as if valid.

Scenario G — Leasehackr import
Given a user-pasted Leasehackr Calculator URL, the app parses known parameters locally, previews the imported values, lists unknown parameters, makes no request to Leasehackr, and populates a row only after confirmation.

Scenario H — Reload privacy
Given locally saved deals, refreshing Railway's hosted page restores them from that browser. Opening the same URL in a different browser has no deals because no cloud data exists.

Scenario I — Sanitized transfer document
Given a deal containing a dealer alias, VIN, salesperson alias, notes, and raw source text, default single-deal export excludes those fields while preserving the financial assumptions needed to reproduce the calculation. The preview reports every excluded category.

Scenario J — Reserved host integration
Given the normal production configuration with no authorized embed origins, /embed loads as a compact standalone surface but the response denies framing. StandaloneHostBridge emits no host messages and no lease data leaves the browser.

Scenario K — Production server and Railway readiness
Given the built application, the Node static server listens on the injected PORT, returns 200 from /healthz, serves SPA fallback routes, caches hashed assets immutably, denies framing by default, and logs no deal data.

30. Extensibility interfaces
30.1 Import proposal
Use and re-export the packages/domain ImportProposal<T> defined in section 9. Do not create adapter-local variants with different warning/error shapes.

30.2 Deal serialization
Owned by packages/adapters:

export interface DealSerializationAdapter<
  TInput = string,
  TParsed = LeaseMatrixDealDocumentV1,
  TOutput = string,
> {
  id: string;
  version: string;
  parse(input: TInput): Promise<ImportProposal<TParsed>>;
  serialize(
    document: LeaseMatrixDealDocumentV1,
    options?: Record<string, unknown>,
  ): Promise<TOutput>;
}
Implement in v0.1:

leasehackr-url;

sanitized deal JSON.

30.3 Workspace interchange
Owned by packages/adapters:

export interface WorkspaceInterchangeAdapter<TInput, TOutput> {
  id: string;
  parse(input: TInput): Promise<ImportProposal<LeaseMatrixWorkspaceDocumentV1>>;
  serialize(document: LeaseMatrixWorkspaceDocumentV1): Promise<TOutput>;
}
Implement in v0.1:

JSON;

CSV as a documented lossy/flattened variant.

30.4 Program provider
Owned by packages/domain:

export interface ProgramProvider {
  id: string;
  lookup(query: ProgramQuery): Promise<ProgramObservation[]>;
}
Implement none in v0.1.1 or earlier without explicit source permission and a documented API/license. A user-paste parser is an adapter, not a provider.

30.5 Host bridge
Owned by packages/integration-protocol:

export interface HostBridge {
  mode: "standalone" | "embedded";
  receiveInitialDocument(): Promise<LeaseMatrixDealDocumentV1 | null>;
  publishDocument(document: LeaseMatrixDealDocumentV1): Promise<void>;
  openCanonicalCalculator(url: string): void;
  dispose(): void;
}
Implement StandaloneHostBridge in v0.1. Reserve PostMessageHostBridge for an authorized later release.

30.6 Dealer importer
Reserve the identifier and domain proposal shape, but do not implement dealer-url in v0.1. It requires the separate service and security review in sections 21 and 24.

31. Research and implementation references
Verified or inspected August 23–24, 2026.

Public references
Leasehackr Calculator: https://leasehackr.com/calculator

Leasehackr Terms of Service: https://forum.leasehackr.com/tos

Edmunds current Mercedes GLC lease-program discussion examples: https://forums.edmunds.com/discussion/73266/mercedes-benz/glc-class/2026-mercedes-glc-class-lease-deals-incentives-rebates-and-prices/p13

Edmunds Visitor Agreement: https://www.edmunds.com/about/visitor-agreement.html

Railway React/Vite guide: https://docs.railway.com/guides/react

Railway shared monorepo deployment: https://docs.railway.com/deployments/monorepo

Railway Infrastructure as Code: https://docs.railway.com/infrastructure-as-code

Railway Config-as-Code deprecation notice: https://docs.railway.com/config-as-code

Railway healthchecks: https://docs.railway.com/deployments/healthchecks

AG Grid React cell editing: https://www.ag-grid.com/react-data-grid/cell-editing/

AG Grid React column pinning: https://www.ag-grid.com/react-data-grid/column-pinning/

AG Grid licensing: https://www.ag-grid.com/license-pricing/

Vite guide: https://vite.dev/guide/

Private repository patterns inspected
Repository: stars-end/prime-radiant-ai, current master at review time.

Relevant generic references:

root package.json — pnpm/Turbo conventions;

pnpm-workspace.yaml and turbo.json — workspace/task shape;

frontend/package.json — Vite/React/AG Grid/Zustand/Tailwind/Vitest/Playwright stack;

frontend/vite.config.ts — Railway host and SPA behavior lessons;

frontend/scripts/serve-static.mjs — small production static server pattern;

frontend/src/components/ui/ag-grid.tsx — AG Grid Community registration and theming pattern;

opencode.json and .opencode/build-agent.md — minimal OpenCode core.

Do not copy private repository code or secrets into the public project without reviewing license, provenance, and project-specific assumptions. Reimplement generic patterns cleanly.

32. Final implementation instruction to OpenCode GLM-5.3
Begin at Milestone 0 and continue through Milestone 5 without pausing for aesthetic approval. Build the smallest complete product that satisfies P0.

Priorities, in order:

Correct, auditable domain semantics and lease math.

A dense, fast, spreadsheet-like workflow that works for real deal comparison.

Local-first privacy and safe source interoperability.

A clean integration seam that does not couple the product to Leasehackr's website or data.

A green public repository and working Railway deployment.

Documentation adequate for outside contributors.

Use Vite + React, not Next.js. Keep the application stateless on Railway. Keep all external source interactions user-directed and non-fetching. Preserve the separation between deal adapters, program providers, and host bridges. Do not begin the Edmunds paste parser, dealer importer, cloud storage, or active iframe integration until v0.1.0 is deployed, dogfooded, and all definition-of-done items are satisfied.

At completion, return evidence: repository URL, Railway URL, deployed SHA, CI status, command results, limitations, and any authentication-only blocker.
