# Mastering AI Pricing

| Field | Details |
| --- | --- |
| **Speaker** | Mayank Pant |
| **Affiliation** | Stripe |
| **Type** | Talk |
| **Date** | 2026-05-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=CrqPcIZOOXA) |
| **Tags** | `infrastructure-and-deployment` `evals` `enterprise` `safety-and-governance` |

## Summary

Mayank Pant, a billing solution architect at Stripe, explains why AI products break
the pricing playbook that SaaS used for years. The top 100 AI companies reached
$20M ARR in about 20 months, roughly three times faster than the top 100 SaaS
companies. That speed leaves little time to build global billing and to rethink
prices as features change.

The core tension: AI gross margins are lower than the steady 80–85% of classic
SaaS, and they swing with usage. A small slice of users — Pant cites 5–10% — can
burn through 80% of compute, so a flat subscription can lose money on its heaviest
customers. Passing costs through as usage fees protects margins, but buyers can't
predict their bill and stop experimenting. Also, the unit that explains your cost
(API calls) rarely explains the customer's value (a good presentation deck).

Pant's answer is to treat the first price as a hypothesis and keep iterating. He
walks through a five-step framework: define the value customers perceive, choose a
charge metric, combine a base fee with a scaling fee, add spending controls, and
learn from customers. Underneath it all, he argues, the billing infrastructure
decides how fast you can change.

## Key Takeaways

- **AI companies grow ~3x faster than SaaS**, so pricing has to be revisited much
  more often.
- **Fixed subscriptions carry margin risk** from power users; **pure usage pricing**
  makes bills unpredictable and discourages experimentation.
- **Hybrid pricing** — a base fee plus a usage/scaling fee — is becoming the norm;
  Pant reports adoption rising from 6% (2024) to 41%.
- **Pick the charge metric by value:** consumption (API calls) → workflow (documents,
  images) → outcome (candidates hired, tickets resolved).
- **Credits hide complexity.** Customers see "100 credits" while the feature-to-credit
  mapping changes underneath.
- **Guardrails prevent surprises:** usage caps, alerts at 50/70/90%, manual or auto
  top-up (or pause), and rate limiting.
- **Repricing signals growth.** More frequent changes tracked with faster-growing
  companies; the comparison does not prove that repricing causes growth.
- **Billing infrastructure sets the pace.** If each change takes months of
  engineering, you can't test prices as your product ships.

## Detailed Notes

### Fast growth strains the billing model · 00:17
- Pant frames the problem with a Stripe finding: the top 100 AI companies hit $20M
  ARR in 20 months, versus 65 for the top 100 SaaS companies.
- Faster money means faster globalization and scaling — and less time to prepare
  billing for customers worldwide.

### Compute costs and customer value diverge · 01:43
- Traditional SaaS ran at 80–85% gross margins that barely moved; AI margins are
  lower and move with usage.
- A flat subscription may not cover a heavy user — Pant's example is that 5–10% of
  users can consume 80% of compute.
- External infrastructure pricing adds uncertainty you don't control.
- Usage pass-through fixes part of the margin problem but creates a buyer problem:
  the number of API calls behind a deck says nothing about how good the deck is.
- A feature that justifies a premium today can be standard in six months, so
  yesterday's packaging may no longer fit the product.
- Stripe's research surfaced three concerns: **unpredictable compute costs (33%)**,
  **hard-to-define delivered value (41%)**, and **product rollouts outpacing pricing
  (84%)**.

### Treat the first price as a hypothesis · 03:44
- Frequent releases change which capabilities deserve a premium, so the first price
  should be something you can revise — and your billing must absorb changes without
  a long engineering project.
- Pant shares a pricing-frequency chart: companies with 100%+ year-on-year growth
  changed prices three-plus times in two years (49% of a high-growth group), versus
  22% of low-growth companies.
- He reads less frequent repricing as a sign of a more static product; he does not
  claim the comparison proves causation.
- Movement is toward **hybrid pricing** (up from 6% in 2024 to 41%, with
  outcome-based pricing at 5%); seat-based and subscription approaches decline.
- Named Stripe billing customers using hybrid approaches: Intercom, Lovable,
  ElevenLabs, OpenAI, Anthropic.

### Step 1: Define the value customers perceive · 06:22
- Start with what the customer gets, not what the software does internally. For a
  Gamma user, that's a useful deck — not the API calls behind it.
- Pant reports that 53% of hypergrowth companies offer clear value-based pricing,
  versus 26% of low-growth peers.
- Four categories help name the value:
  - **Automation:** save time, which the customer reads as lower cost.
  - **Augmentation:** same headcount, better output (e.g. a campaign team producing
    more, faster).
  - **Enhanced service:** access to proprietary software or data — Pant points to
    Stripe's payment volume as an edge in fraud detection.
  - **Improved results:** charge around a business outcome, like Intercom's tickets
    resolved without a human.

### Step 2: Choose a billable unit · 09:02
- A **charge metric** turns value into something you bill for. Suitability depends on
  the product and the buyer:
  - **Consumption** (API calls) aligns to provider cost and is easy to meter.
  - **Workflow** (images generated, documents summarized) aligns to product activity.
  - **Outcome** (candidates hired, qualified leads) aligns to customer ROI.
- The tradeoff is implementation versus attribution: consumption is easy to meter but
  hard to tie to value; outcomes are easier to sell but harder to prove.
- A 1,000-API-call allowance tells a presentation buyer nothing about how many decks
  they get.
- **Credits** give one currency across features: a plan includes 100 credits, and
  internal rules decide how each operation consumes them.

### Step 3: Combine a base fee with a scaling fee · 11:34
- The charge metric answers *what* is counted; the pricing model answers *how* the
  count becomes a bill.
- **Subscription:** predictable revenue and a committed relationship, but power users
  erode margins.
- **Usage fee:** charges scale with consumption, but an uncertain invoice makes
  customers hesitant to explore.
- A **hybrid model** pairs a **base fee** (the ongoing relationship) with a
  **scaling fee** (additional value consumed), serving both light and heavy users
  without tying every customer's economics to the heaviest ones.

### Step 4: Prevent billing surprises · 13:07
- Pant's design principle: **be fair, and do not surprise**. A wrong bill in month six
  can erase months of earned trust.
- Example: a $20 plan with 100 credits. Once credits are spent, the customer chooses
  to pay for more or wait until the next month.
- The cap is a small admission decision, not a charge. Pant shows an illustrative
  TypeScript check that returns `allow` or `top-up-or-wait` when the allowance would
  be exceeded; it does not debit the account or run the work.
- Make the boundary visible before the customer reaches it:
  1. **Notify as usage accumulates** — alerts at 50%, 70%, and 90%.
  2. **Offer explicit next actions** — manual top-up, automatic top-up, or pause.
  3. **Rate-limit requests** so a faulty loop can't burn the allowance.

### Step 5: Iterate, realign, and keep the invoice explainable · 15:06
- Pant reports 84% agree that fast pricing adaptation is a competitive advantage.
  Ship a plausible price rather than hunting for a perfect one for a year.
- Interview customers who **churn** to separate a product-market-fit problem from a
  price problem; interview customers who **upgrade** to learn where value is strongest.
- Run pricing A/B tests, but prioritize speed so pricing keeps pace with releases.
- **Credits are a place to realign.** Under the hood, 100 credits might map to five
  API calls of one type or ten image generations of another; those are example
  conversions, not a fixed exchange rate.
- On customer stability: a premium feature might cost five credits in January and be
  standard six months later. The customer keeps a visible 100-credit allowance while
  the mapping shifts. **Grandfathering** is the other option — a new price version for
  new customers while existing ones keep their terms.
- A correction worth remembering: a **stable plan is not a fixed price**. The
  customer-facing plan stays constant; its price and feature allocation can still
  change.
- **Billing infrastructure sets the pace.** If each revision costs three or four
  months of engineering, you can't test prices at the speed the product changes.
  Pant notes AI companies can move from retail or product-led growth into enterprise
  sales within 10–15 months, adding minimum commitments, pre-commitments, and overage
  prices (the Metronome acquisition addresses that complexity).
- He flags the denominator on Stripe's 78% adoption figure: it refers to the Forbes
  AI 50, not all AI companies.
- On tracking: the app sends usage from its LLM calls and identifies the pricing that
  applies (tiered, dynamic, or dimension-based). Billing rates it and produces
  reporting that explains the invoice — so the customer can trace the amount back to
  the usage that produced it.

## Resources

- [A framework for pricing AI products](https://stripe.com/blog/a-framework-for-pricing-ai-products)
  — Stripe's write-up of charge metrics and subscription/usage models.
- Inside the growth of the top AI companies on Stripe — Stripe's analysis of revenue
  milestones.
- [Stripe completes Metronome acquisition](https://stripe.com/in/newsroom/news/stripe-completes-metronome-acquisition)
  — context for enterprise contract billing.
- Bill for LLM tokens — current Stripe documentation on token metering.
- Fin AI Agent outcomes — Intercom's definitions of billable resolutions and handoffs.
