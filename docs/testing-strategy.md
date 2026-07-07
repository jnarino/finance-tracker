# Testing Strategy

## 1. Objective

Financial calculations must be deterministic, auditable, and correct to the cent. Tests are required for every calculation rule and import mapping.

## 2. Test layers

### Unit tests

Cover pure domain functions:

- Gross and net premium.
- Buy-to-close debit.
- Realized option P/L.
- Roll credit/debit.
- Campaign cash flow.
- Assignment obligation.
- Call-away exposure.
- Put break-even.
- Campaign all-in break-even.
- Economic basis.

### Import adapter tests

Use fixture CSVs for:

- Sell to open.
- Buy to close.
- Expiration.
- Assignment.
- Exercise/call-away.
- Stock purchase from assignment.
- Fees.
- Transfers and unrelated stock sales.
- Duplicate file import.
- Malformed and ambiguous rows.

### Integration tests

Validate end-to-end behavior:

- Import file -> normalize -> build campaign -> calculate report.
- Re-import same file -> no duplicates.
- Manual correction -> recalculation with audit history.
- Roll chain -> old contract realized, new contract open.

### Snapshot/golden tests

Maintain known examples with expected penny-perfect outputs.

## 3. Required calculation examples

### Example A: expired short put

```text
Sell 1 put at 0.22
Fee 0.66
Expires worthless
Expected realized P/L = 21.34
```

### Example B: losing roll

```text
Sell 3 puts for net 76.00
Buy them back for 823.99
Expected realized P/L = -747.99
```

### Example C: roll cash flow

```text
Buy old puts back for 823.99
Sell replacement puts for net 900.98
Expected incremental roll credit = 76.99
```

### Example D: campaign break-even

```text
Strike = 20
Exposure = 300 shares
Cumulative campaign option cash flow = 556.62
Expected break-even = 18.1446...
Displayed break-even = 18.14 using documented rounding
```

### Example E: covered call

```text
Own 100 shares
Sell 1 call at 0.26
Fee 0.66
Expected net premium = 25.34
```

## 4. Rounding

- Perform calculations with exact decimal precision.
- Round only for display unless brokerage settlement requires a specified rounding step.
- Document display rounding rules.
- Compare final monetary outputs to the cent.

## 5. Invariants

Tests should enforce:

- Net premium cannot exceed gross premium when fees are nonnegative.
- A roll creates one completed position and one new open position.
- An expired option has zero open quantity.
- A covered call cannot be marked covered without sufficient shares.
- Re-importing identical source rows does not change totals.
- Calculation traces sum exactly to displayed totals.

## 6. Regression policy

Any bug involving money, quantity, strike, expiration, assignment, or campaign linkage must receive a regression test before the fix is merged.
