# GitHub Copilot Instructions — LST Options Journal

## Project intent

This repository builds a personal options-trading journal and analysis application for a Low Stress Trading / Wheel-style workflow.

The product must help the user:

- Import brokerage transaction history, initially from Charles Schwab / thinkorswim CSV exports.
- Track sold puts, covered calls, expirations, buy-to-close transactions, rolls, assignments, and shares called away.
- Separate **premium collected**, **cash flow**, **realized profit/loss**, **unrealized profit/loss**, and **assignment exposure**.
- Group related trades into campaigns so a rolled position can be understood as one continuous strategy.
- Produce weekly and cumulative summaries with transparent calculations.
- Explain results in plain language without placing trades or presenting output as guaranteed financial advice.

Read these files before making material changes:

1. `docs/requirements.md`
2. `docs/architecture.md`
3. `docs/domain-rules.md`
4. `docs/testing-strategy.md`

## Non-negotiable domain rules

- Never treat all premium received as profit.
- A roll is two transactions: buy to close the old option and sell to open the new option.
- Preserve both the old realized result and the new open obligation.
- Assignment is not required for an option trade to realize a loss; buying an option back for more than the original credit realizes a loss on that contract.
- Distinguish broker tax basis from economic/campaign-adjusted basis.
- Do not combine stock-sale gains/losses with options performance unless the user explicitly requests a total campaign view.
- Never silently infer missing contract details, fees, dates, quantities, strikes, or transaction direction.

## Coding rules

- Inspect and follow the existing repository stack and conventions before introducing new frameworks.
- Keep domain calculations framework-independent and isolated from UI code.
- Represent money with integer cents or a decimal library. Never use binary floating-point for financial totals.
- Preserve the original imported CSV row and its source file metadata for auditability.
- Normalize timestamps to UTC while retaining the original brokerage date/time and timezone when available.
- Make imports idempotent. Re-importing the same file must not create duplicate trades.
- Use explicit types for transaction direction and lifecycle state. Avoid free-form strings in core domain logic.
- Prefer pure functions for calculations.
- Return calculation breakdowns, not only final numbers.
- Validate all external input at the boundary.
- Do not log account numbers, personal identifiers, brokerage credentials, or full raw files in production logs.

## Required terminology

Use these labels consistently in code and UI:

- **Gross premium collected**: option sale proceeds before fees.
- **Net premium collected**: option sale proceeds after fees.
- **Closing debit**: cash paid to buy an option back, including fees where applicable.
- **Realized option P/L**: credits minus debits for completed option contracts.
- **Open campaign cash flow**: cumulative option credits minus closing debits for a campaign that still has an open contract.
- **Unrealized option P/L**: current mark-to-market value of open contracts compared with their net opening credit.
- **Assignment obligation**: strike × 100 × contracts for short puts.
- **Call-away exposure**: 100 shares × short call contracts.
- **Economic basis**: user-facing adjusted basis based on campaign cash flows; never label it tax basis.

## Implementation behavior

When asked to add or modify a feature:

1. Identify the relevant requirement and domain rule.
2. Update or add tests before changing calculation logic.
3. Preserve backward compatibility for imported records.
4. Add migration notes when changing persisted data structures.
5. Expose a human-readable calculation trace for totals.
6. Document assumptions in code comments only when the logic cannot be made self-explanatory.

## AI-related constraints

If the repository includes AI-assisted explanations:

- AI may summarize deterministic calculations but must not be the source of truth for calculations.
- All financial totals must come from tested application logic.
- AI output must identify uncertainty and missing data.
- AI must not place orders, connect to brokerage execution endpoints, or claim guaranteed returns.
