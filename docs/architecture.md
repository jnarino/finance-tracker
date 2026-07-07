# Architecture Definition — LST Options Journal

## 1. Architectural goals

- Keep financial calculations deterministic and independently testable.
- Preserve raw brokerage evidence.
- Support multiple brokerage import formats over time.
- Make every total explainable through a calculation trace.
- Keep AI explanations optional and downstream from the calculation engine.

## 2. Logical architecture

```text
CSV / Brokerage Export
        |
        v
Import Adapter -> Raw Import Store -> Normalization Pipeline
                                      |
                                      v
                               Domain Ledger
                                      |
                  +-------------------+-------------------+
                  |                                       |
                  v                                       v
          Calculation Engine                       Campaign Builder
                  |                                       |
                  +-------------------+-------------------+
                                      |
                                      v
                              Application Services
                                      |
                         +------------+------------+
                         |                         |
                         v                         v
                       API                     Reports/UI
```

## 3. Main components

### 3.1 Import adapters

Responsibilities:

- Read brokerage-specific files.
- Map source columns to a stable raw schema.
- Generate a deterministic source-row fingerprint.
- Reject malformed rows with actionable errors.
- Never perform business calculations.

Initial adapter:

- `SchwabThinkorswimCsvAdapter`

Future adapters may include Robinhood, Fidelity, Interactive Brokers, or manual-entry templates.

### 3.2 Raw import store

Store:

- Import batch identifier.
- Original filename.
- File hash.
- Import timestamp.
- Original row number.
- Original row payload.
- Parsing status and warnings.

Raw records are immutable.

### 3.3 Normalization pipeline

Transforms raw rows into normalized events:

- `OPTION_SELL_TO_OPEN`
- `OPTION_BUY_TO_CLOSE`
- `OPTION_EXPIRED`
- `OPTION_ASSIGNED`
- `OPTION_EXERCISED`
- `STOCK_BUY`
- `STOCK_SELL`
- `TRANSFER`
- `FEE`
- `UNKNOWN`

Normalization must extract:

- Underlying symbol.
- Option type: put or call.
- Strike.
- Expiration.
- Quantity.
- Price.
- Gross amount.
- Fees.
- Net cash effect.
- Brokerage account.
- Source transaction identifier.

### 3.4 Domain ledger

The domain ledger is the canonical source for application calculations.

Recommended entities:

- `BrokerageAccount`
- `ImportBatch`
- `RawTransaction`
- `NormalizedEvent`
- `OptionContract`
- `OptionPosition`
- `StockLot`
- `Campaign`
- `CampaignEventLink`
- `MarketSnapshot`
- `CalculationSnapshot`

### 3.5 Campaign builder

A campaign groups related activity for the same strategy objective.

Examples:

- Repeated NVTS short puts linked through rolls.
- NOK cash-secured put followed by assignment and covered calls.
- SMR covered-call income against pre-existing shares.

Campaign linking rules should be deterministic where possible and user-correctable when ambiguous.

### 3.6 Calculation engine

The calculation engine must be a standalone domain module with no UI dependencies.

It produces:

- Premium summaries.
- Realized P/L.
- Open cash flow.
- Current option liability.
- Unrealized P/L.
- Break-even values.
- Assignment and call-away exposure.
- Basis views.
- Calculation traces.

All calculations must accept explicit inputs and return explicit outputs, warnings, and source references.

### 3.7 Application services

Coordinate use cases such as:

- Import brokerage file.
- Review uncertain rows.
- Build or edit a campaign.
- Calculate weekly performance.
- Generate a report.
- Export CSV/XLSX.

### 3.8 API layer

The API should expose resource-oriented endpoints or equivalent application commands.

Suggested boundaries:

- `/imports`
- `/transactions`
- `/positions`
- `/campaigns`
- `/reports`
- `/market-snapshots`

The API must never expose raw account numbers or secrets.

### 3.9 UI and reporting

Recommended views:

- Import review.
- Transaction ledger.
- Open positions.
- Campaign detail with timeline.
- Weekly dashboard.
- Premium vs profit reconciliation.
- Risk and assignment exposure.
- Export center.

## 4. Data design rules

- Use UUIDs or stable generated identifiers for internal records.
- Use integer cents or decimal numeric columns for money.
- Store option strike as decimal, not float.
- Store contract quantity as integer contracts; convert to share exposure with a multiplier, normally 100.
- Store both gross amount and fees.
- Preserve source sign conventions separately from normalized cash direction.
- Use explicit status enums.
- Do not overwrite imported records when the user corrects classification; store corrections as an audit event.

## 5. Calculation trace

Every displayed total should be reproducible from a trace like:

```text
NVTS campaign net cash flow
  + 104.67  Jun 09 sell-to-open
  +  91.00  Jun 16 sell-to-open
  +  76.00  Jun 23 sell-to-open
  - 823.99  Jun 25 buy-to-close
  + 900.98  Jun 25 sell-to-open
  - 997.99  Jun 30 buy-to-close
  + ...
```

The trace is a first-class output, not a debug-only feature.

## 6. Suggested deployment shape

Keep deployment flexible. If the repository has no established stack, a reasonable default is:

- Web application: TypeScript.
- Frontend/API: Next.js or equivalent full-stack framework.
- Database: PostgreSQL.
- ORM: Prisma or equivalent.
- Validation: Zod or equivalent schema validation.
- Tests: Vitest/Jest plus integration tests.
- Background jobs: lightweight queue only when imports or reports become asynchronous.

Do not introduce these technologies if the existing repository already has an established, coherent stack.

## 7. Security and privacy

- Treat brokerage exports as sensitive financial data.
- Encrypt stored files and database backups where supported.
- Apply least-privilege access.
- Avoid storing full account numbers.
- Redact personal information from logs and error monitoring.
- Do not add brokerage execution permissions.

## 8. AI integration boundary

AI may:

- Explain a calculation trace.
- Summarize weekly performance.
- Flag missing context.
- Translate brokerage terminology into plain language.

AI may not:

- Recalculate totals independently of the domain engine.
- Modify imported transactions without a user-confirmed correction.
- Place trades.
- Guarantee returns or assignment outcomes.
