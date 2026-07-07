# Domain and Calculation Rules

## 1. Why this file exists

Options accounting is easy to misstate. These rules define the exact meanings the application must use so premium, profit, rolls, assignment, and basis are never conflated.

## 2. Monetary conventions

- Store money in integer cents or exact decimal values.
- A brokerage row's original sign may differ from the application's normalized sign.
- In normalized calculations:
  - Cash received is positive.
  - Cash paid is negative.
- Fees reduce the economic result.

## 3. Premium

### Gross premium collected

For an option sale:

```text
gross premium = option price × contract multiplier × number of contracts
```

For standard equity options, the contract multiplier is normally 100.

### Net premium collected

```text
net premium = gross premium - commissions - fees
```

Premium collected may be reported even when a contract remains open.

Premium collected is not automatically profit.

## 4. Closing an option

For a short option bought back:

```text
closing debit = purchase price × multiplier × contracts + fees
```

For the completed contract:

```text
realized option P/L = opening net premium - closing debit
```

A loss can be realized without assignment.

## 5. Expiration

When a short option expires worthless:

```text
realized option P/L = opening net premium
```

No additional premium is received at expiration. The opening credit was received when the option was sold.

## 6. Rolls

A roll is not one transaction. It is:

1. Buy to close the existing option.
2. Sell to open a replacement option.

The old option's result becomes realized.

```text
old contract realized P/L = old opening credit - old closing debit
```

The roll's incremental cash flow is:

```text
roll net credit/debit = new opening net premium - old closing debit
```

The replacement option remains open and must retain its full obligation.

Never label the new sell-to-open proceeds as profit without subtracting the close cost and open liability.

## 7. Campaign accounting

A campaign may include multiple options and stock events.

### Campaign option cash flow

```text
campaign option cash flow = sum(all option sale net credits) - sum(all option closing debits)
```

For an open campaign, this is not final P/L.

### Campaign realized option P/L

Sum only completed option contracts.

### Open option liability

For short options marked to market:

```text
current liability = current option mark × multiplier × contracts
```

### Campaign mark-to-market result

```text
campaign option MTM P/L = campaign option cash flow - current open-option liability
```

Use current market data timestamp and source when showing this value.

## 8. Cash-secured puts

### Assignment obligation

```text
assignment obligation = strike × multiplier × short contracts
```

### Single-trade break-even

```text
put break-even = strike - opening premium per share
```

### Campaign all-in break-even after rolls

For a campaign that may assign `N` shares:

```text
campaign break-even = strike - (cumulative campaign option cash flow / N)
```

Only use this shortcut when the campaign remains at one effective assignment strike and the share exposure is consistent. Otherwise calculate by lots.

### Assignment

Assignment converts the put obligation into a stock purchase. It does not retroactively make all prior premium "free profit."

## 9. Covered calls

### Call-away exposure

```text
call-away shares = contracts × multiplier
```

A call is covered only when available shares are at least the call-away shares.

### Effective exit price for a call

```text
effective exit price = call strike + call premium allocated per share
```

Prior campaign premiums may be shown in total campaign P/L but should not be silently added to the current call's quoted exit price.

### Covered-call outcome

- Below strike at expiration: keep shares and opening premium.
- Above strike at expiration: shares are likely called away at the strike, while the premium is retained.

Assignment risk can occur before expiration.

## 10. Basis definitions

### Broker/tax basis

The basis reported by the broker. Preserve it as imported and do not overwrite it with application calculations.

### Economic or campaign-adjusted basis

A management metric that subtracts selected campaign credits from stock cost.

Always label it `economic basis` or `campaign-adjusted basis`, never `tax basis`.

The application should show the included credits in a trace.

## 11. Performance views

The application must support separate views:

1. **Premium collected** — all option sale proceeds.
2. **Net premium collected** — sale proceeds after fees.
3. **Realized option P/L** — completed contracts only.
4. **Open campaign cash flow** — credits minus closing debits while a contract remains open.
5. **Unrealized option P/L** — mark-to-market effect of open options.
6. **Stock P/L** — separate unless a full campaign view is explicitly selected.
7. **Total campaign P/L** — options plus linked stock events.

## 12. Counting weeks and rolls

Track both:

- `roll_count`: number of actual roll events.
- `expiration_cycle_count`: number of distinct expiration cycles in the campaign.

Do not use the terms interchangeably.

## 13. Data uncertainty

When any required field is missing:

- Mark the calculation incomplete.
- Show which value is missing.
- Do not guess silently.
- Allow a user correction with audit history.
