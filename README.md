# Depot vs. Private Insurance

This notebook compares two ways of building retirement capital:

- a taxable ETF depot with FIFO lot tracking, rebalancing, and tax when you sell everything at the start of your pension
- a private insurance wrapper (Stuttgarter T73) with accumulation fees and payout-phase costs

The goal is not to price the product perfectly. The goal is to answer a practical question: **under the current assumptions, how much money do I end up with if I save for 40 years and then draw down for 20 years?**

## What the notebook currently models

- 40 years of accumulation at 400 EUR/month
- 20 years of payout
- Mean annual return of 7.5% with 15% volatility, using the same ETF return path for both depot and insurance during accumulation
- Depot taxation with FIFO lot tracking
- Rebalancing every N years for a given fraction of the depot (insurance-side rebalancing is assumed free in this model)
- German-style tax approximation for the depot:
  - Abgeltungsteuer plus Soli using an effective rate of 26.375%
  - 30% Teilfreistellung (applies to equity ETFs (minimum 51% equity allocation according to the Investment Tax Act 2018))
  - Vorabpauschale approximation
  - annual Sparer-Pauschbetrag option
- Insurance costs during accumulation:
  - 240 EUR/year fixed cost after the initial phase at 400 EUR/month
  - 960 EUR/year in the first 5 years at 400 EUR/month
  - 0.48% annual asset fee
- Insurance payout phase:
  - shared low-risk payout return: 2.0% p.a. for both depot (e.g., through Festgeld) and insurance (e.g., through safety bonds)
  - 1.5% annual fee
  - insurance payout tax modeled in two modes:
    - `flat_rate` (legacy fallback for sensitivity checks, e.g. 10%)
    - `annuity_18pct_x_25pct` (effective 4.5% = 18% taxable share x 25% tax)
  - current default in notebook: `annuity_18pct_x_25pct` (effective 4.5%)
- Nominal and inflation-adjusted comparison
- Monte Carlo runs plus rebalancing sensitivity plots

## Main results from the current notebook run (average-based)

**Result type A (before payout starts):** amount gathered before payout starts.

- Insurance amount gathered before payout starts is shown in the notebook pre-payout snapshot.
- Depot amount gathered before payout starts is shown after modeled depot taxes due at payout start.

**Result type B (after full payout horizon):** final overall money outcome after the full payout horizon.

These are the current base-case **Monte Carlo averages** for result type B (2,000 runs):

- Insurance available money at end of payout:
  - nominal average: 894,633.68 EUR
  - real average: 729,176.54 EUR
- Depot available money at end of payout:
  - nominal average: 1,067,622.67 EUR
  - real average: 861,565.60 EUR
- Total Vorabpauschale tax paid in the single reference run: 44,877.84 EUR
- Total rebalancing tax paid in the single reference run: 117,559.68 EUR
- Monte Carlo depot win rate in the current setup: 100.0%
- Monte Carlo nominal difference (depot - insurance):
  - mean: 172,989 EUR
  - P05/P95: 82,973 / 293,924 EUR

In Monte Carlo and rebalancing sensitivity, "depot - insurance" means **result type B** (final overall money outcome after the full payout horizon), not just the amount at payout start. The main chart now shows an average path, not one random single path.

The rebalancing sensitivity heatmap shows the same pattern: in this model, the depot stays ahead across the tested rebalancing ratios and intervals, but higher rebalancing usually reduces the depot advantage because it triggers more taxable gains.

## Main caveats

The biggest caveat is that this is still a **model**, not the real product contract. The conclusion depends heavily on the exact product design, tax treatment, and payout logic.

In plain language, the two products also differ in how flexible they are:

- The insurance is more tightly bound to the contract. That usually means **less freedom to change** the monthly contribution quickly, and early payout or contract changes can be more complicated.
- The depot is more flexible. You can usually change the monthly investment, pause it, sell parts, or switch ETFs more easily.
- On the other hand, **depot costs and taxes can change over time**, either because of product fees, broker pricing, or tax rules.
- The insurance may feel more stable because the contract structure is more fixed, but that also means less room to react if your plans change.

Also important for risk reduction close to retirement:

- Depot side: you can usually shift money quite easily into safer buckets (for example Tagesgeld/Festgeld or very short-duration instruments) and target a lower-risk return profile if market rates allow it.
- Insurance side: this is often possible only within the contract's internal switching universe and rules. So de-risking can be less flexible, and net return can still be reduced by contract fees/cost drag.

Key limitations:

- The Vorabpauschale is still an approximation, not a fully year-specific legal tax engine.
- The Sparer-Pauschbetrag is treated in a simplified way.
- The payout phase now assumes a shared low-risk return of 2.0% p.a. for both products. This is still a simplifying assumption, not a contract guarantee.
- Insurance taxation in payout is simplified.
- The model uses one long-run return assumption and normal return volatility. Real markets are not normal.
- There is no mortality/longevity table, no health risk, and no guaranteed annuity pricing.
- Product details such as kickbacks, exact surrender rules, guarantee mechanisms, and contract-specific fee structures are not fully modeled.

## What might still be wrong

The main things that could still be off are:

- the exact German tax mechanics for Vorabpauschale in each year
- whether the Sparer-Pauschbetrag is really available for this depot at all times
- the precise insurance taxation at payout
- whether the Stuttgarter contract has additional hidden or contract-specific costs not included here
- whether rebalancing should happen automatically, manually, or on a different schedule
- whether the right comparison is total cash at the end or the value of lifelong retirement income security

So the notebook is directionally useful, but it should still be treated as a structured scenario analysis rather than a final legal/tax verdict.

## Practical interpretation

If the question is:

- **"What leaves me with more money at age ~87 under these assumptions?"**

then the current notebook **favors the depot**.

If the question is:

- **"What is more suitable as a retirement income wrapper with longevity protection and product guarantees?"**

then the insurance can still make sense even if the raw end-value is lower.

## Questions for R.

Use these as a short checklist in the next meeting:

- Is insurance rebalancing currently ON or OFF in my contract?
  - If ON: how often does it happen, and is it automatic or manual?
  - Does insurance rebalancing create any direct or indirect costs?
  - Does insurance rebalancing trigger any tax effect for me?
  - Why was insurance rebalancing turned off before, and when should it be turned on again?
- Are the cost assumptions for 400 EUR/month correct (960 EUR/year first 5 years, then 240 EUR/year fixed)?
- Is the annuity-style payout tax approximation (effective 4.5% from 18% x 25%) correct for my exact contract and payout type? More importantly: Is this fixed/to be relied on?
  - Where did the earlier 10% payout-tax assumption come from exactly (contract clause, product sheet, or rough estimate only), and should we drop it completely?
- Is a shared low-risk payout return of 2.0% p.a. realistic for both products in practice, and under what constraints?
- **Should part of the insurance allocation be in safer assets (not 100% ETFs), and if yes, what target split is realistic?**
  - How often can I switch between risky and safer allocations in the insurance without penalties, and are there hidden constraints (timing windows, minimum quotas, switch limits)?
  - If I want a Festgeld-like phase from age X onward, what is the closest equivalent inside the insurance contract and how has it performed net in recent years?

## Bottom line

This notebook is currently best read as a structured comparison model, not as a final recommendation.

The strongest current takeaway is:

- with the assumptions used here, the depot wins on raw end-value
- the insurance only becomes competitive if the product's retirement-specific features are valued strongly enough, or if the assumptions are changed in its favor
- the exact answer is sensitive to tax details, payout design, and contract costs

If you share this with R., the most honest summary is:

- "Under my current explicit assumptions and payout setup, the depot wins on end-value.
- But I know this is still an approximation, especially around tax and contract details.
- I want to compare the product value, not just the math of one simulation path."
