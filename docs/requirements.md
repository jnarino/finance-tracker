# Product Requirements — LST Options Journal

## 1. Purpose

The LST Options Journal is a personal trading-analysis application for tracking weekly options activity under a Low Stress Trading / Wheel-style framework.

The primary purpose is to answer, accurately and transparently:

- How much premium has been collected?
- How much was paid to close or roll positions?
- What profit or loss has been realized?
- What positions remain open?
- What assignment or call-away obligations remain?
- What is the all-in campaign break-even after rolls?
- How has each ticker and each weekly campaign performed?

## 2. Target user

A retail options trader who:

- Sells cash-secured puts.
- Sells covered calls after assignment or against existing shares.
- Rolls contracts when needed.
- Uses Schwab / thinkorswim transaction exports.
- Wants a simple weekly dashboard and an auditable trade journal.

## 3. In scope

### 3.1 Data import

- Import CSV transaction exports from Charles Schwab / thinkorswim.
- Detect and classify:
  - Sell to open.
  - Buy to close.
  - Expiration.
  - Assignment.
  - Exercise / shares called away.
  - Stock purchase caused by assignment.
  - Fees and commissions.
  - Deposits, transfers, and unrelated stock transactions.
- Permit manual correction when a row cannot be classified confidently.
- Prevent duplicate imports.

### 3.2 Trade journal

- Show every normalized transaction.
- Link option legs to an underlying symbol and contract.
- Link rolls into campaigns.
- Allow campaign naming, notes, and framework-rule annotations.
- Track strategy type: cash-secured put, covered call, Wheel campaign, or manually classified.

### 3.3 Calculations

The application must calculate and display:

- Gross premium collected.
- Net premium collected after fees.
- Closing debits.
- Realized option P/L.
- Open campaign cash flow.
- Current unrealized option P/L when market marks are available.
- Assignment obligation.
- Call-away exposure.
- Contract-level and campaign-level break-even.
- Broker-reported cost basis when imported.
- Economic/campaign-adjusted basis as a separate value.
- Weekly, monthly, ticker, strategy, and all-time summaries.

### 3.4 Dashboard

The dashboard should provide:

- Total premium collected for the selected period.
- Realized P/L for completed option trades.
- Open positions and expiration dates.
- Assignment and call-away risk.
- Campaigns requiring attention.
- Results by ticker.
- Weekly performance trend.
- Clear warning when premium is being confused with profit.

### 3.5 Reports and export

- Export normalized transactions and summaries to CSV/XLSX.
- Generate a weekly report containing:
  - Opening positions.
  - Trades placed.
  - Rolls performed.
  - Premium collected.
  - Closing costs.
  - Realized P/L.
  - Open obligations.
  - Notes and lessons learned.

## 4. Out of scope for the first release

- Automated order placement.
- Brokerage credential storage.
- High-frequency or intraday execution.
- Tax filing or tax-lot optimization.
- Guaranteed trade recommendations.
- Replacing the brokerage statement as the legal record.

## 5. Core user stories

### Import and reconcile

- As a user, I can import a Schwab CSV and see which rows were accepted, ignored, or require review.
- As a user, I can re-import the same CSV without creating duplicates.
- As a user, I can see the original source row behind every normalized transaction.

### Understand premium and profit

- As a user, I can see total premium collected even if contracts remain open.
- As a user, I can separately see realized P/L after buy-to-close costs.
- As a user, I can understand why a rolled campaign may have positive cash flow but still carry an unrealized loss.

### Manage campaigns

- As a user, I can see all contracts linked to one ticker campaign.
- As a user, I can see the cumulative campaign credit and all-in break-even.
- As a user, I can see how many weekly expirations and actual roll transactions have occurred.

### Review risk

- As a user, I can see how much cash would be required if all short puts were assigned.
- As a user, I can see how many shares may be called away by covered calls.
- As a user, I receive a warning when one ticker consumes an excessive share of account value.

## 6. Acceptance criteria

The first usable release is complete when:

1. A Schwab CSV can be imported without duplicates.
2. All option sales, closures, expirations, and assignments are normalized.
3. Premium collected and realized P/L are displayed separately.
4. A multi-roll campaign produces a reproducible calculation trace.
5. The application can reproduce known hand-calculated examples within one cent.
6. All financial calculation tests pass.
7. The user can export a weekly summary.

## 7. Product principles

- Accuracy over visual complexity.
- Auditability over hidden automation.
- Plain language over brokerage jargon.
- Deterministic calculations over AI-generated arithmetic.
- Explicit uncertainty over silent assumptions.
