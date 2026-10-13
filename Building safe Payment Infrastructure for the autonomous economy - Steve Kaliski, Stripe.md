# Building safe Payment Infrastructure for the autonomous economy

| Field | Details |
| --- | --- |
| **Speaker** | Steve Kaliski |
| **Affiliation** | Stripe |
| **Type** | Expo session |
| **Date** | 2026-06-06 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=KLSuFPj2ld0) |
| **Tags** | `finance` `safety-and-governance` |

## Summary

Stripe principal software engineer Steve Kaliski asks how robots can spend money safely — and how businesses can safely receive it. His core idea is a split: discovery and exploration benefit from nondeterminism (a language model's broad knowledge), but credentials, payments, and checkout require determinism.

He shows why letting an agent browse like a human is risky. It can buy at the wrong place, buy the wrong thing, spend the wrong amount, or use the wrong credential. Each failure needs a different protection, so paying should bind to a merchant, enforce spending policy, work through APIs, and use verifiable identities.

The talk walks through three building blocks: shared payment tokens (credentials with enforced limits), the Machine Payments Protocol with HTTP 402 (paying for a tool call), and the Agentic Commerce Protocol (structured checkout). The through-line is that a nondeterministic planner stays inside enforced constraints, so the potential damage stays small.

## Key Takeaways

- **Separate discovery from spending.** Nondeterminism is fine while choosing; credentials, payments, and checkout must be deterministic.
- **Agent purchases fail in four ways.** Wrong place, wrong thing, wrong amount, and wrong credential — each needs its own protection.
- **Never share a raw card number.** Shared payment tokens attach enforceable limits (seller, currency, amount, time) and are auditable.
- **Limits are enforced outside the agent.** Even a fooled or mistaken agent cannot act beyond the token's mandate.
- **Paying can be part of an HTTP request.** The Machine Payments Protocol uses HTTP 402 "Payment Required" to attach payment instructions to the exact resource.
- **Checkout should be explicit.** The agent requests a change; the seller returns the authoritative state (prices, tax, shipping).
- **Keep the merchant in control.** Sellers keep their customer relationship and risk signals even when the buyer is software.
- **Prefer agent-friendly interfaces.** A web-only UI forces the agent to interpret everything, which increases nondeterminism and risk.

## Detailed Notes

### How can robots safely spend money? · 00:17
- Kaliski spent four years leading Stripe Issuing (virtual and physical cards) and two years exploring agent spending.
- One takeaway if nothing else: **discovery and exploration benefit from nondeterminism**, but credentials, payments, and checkout **require** determinism.
- Finding something and getting authority to pay for it are separate operations.
- Agents already spend money. Using Claude Code or Codex incurs costs through subscriptions or tokens converted into dollars.
- The next step is extending that spending beyond the model provider to other businesses, currencies, and payment methods.
- Crudely, an agent calls a model and calls tools. Both cost money; the relevant tools here are search, credential management, and payment.

### Four ways a browser purchase can go wrong · 02:41
- The baseline approach is to let an agent operate a browser like a human.
- But that leaves four distinct error points:
- **Wrong place:** an Amazon lookalike hosted on a different domain.
- **Wrong thing:** an orange T-shirt instead of a purple one — or something far more expensive.
- **Wrong amount:** an extracted price misses currency differences, taxes, or price changes.
- **Wrong credential:** a pasted card ends up at the wrong destination.
- Each failure needs a different protection. Recognizing a familiar page does not prove merchant identity; finding a product does not prove the payable amount.
- A card number is only one payment credential — some methods are hard or impossible for an agent to relay through a form.
- Browser checkout is also slow and hard to observe: navigation is finicky, and the outcome of each click must be inferred.
- The payment interface should bind to a merchant, enforce spending policy, be API-driven, and provide verifiable identities.

### Share a constrained credential, not a card number · 04:50
- Handing over a raw card number carries no enforceable limits; you trust the recipient to charge what you intended.
- **Shared payment tokens** move those restrictions into the payment infrastructure. The agent collects a credential and shares a token with the seller instead.
- The token acts like a mandate: which seller may use it, in which currency, for what amount, and over what time.
- Stripe enforces the restrictions and the use is auditable. If the agent is fooled by a domain or misreads a price, the seller and amount limits still constrain what the credential authorizes.
- The demo starts with an ordinary seller integration: a `PaymentIntent` for $50 using a payment method collected by the seller.
- Then a second Stripe account (the agent's) provisions a Visa-backed token limited to $25, expiring in 30 days, scoped to one test seller. The underlying card has a higher limit, but that broader authority is not passed through.
- The seller still receives the card brand, last four digits, and credit type, so its existing risk analysis keeps working. Isolation does not mean hiding useful information from the merchant.
- Stripe rejects the seller's $50 request because it exceeds the $25 mandate, then accepts a lower charge; Kaliski does not state the successful amount.

### Attach payment to the requested resource · 08:57
- The **Machine Payments Protocol** (MPP), developed with Tempo, treats a tool call as an HTTP request that may require payment.
- A prearranged API key is one option, but agent-tool interactions can be ephemeral. The server needs to state payment requirements during the request itself.
- HTTP **402 "Payment Required"** is that boundary: the seller responds with a payment requirement, and the client supplies a credential to get the good.
- This ties payment instructions to the resource being purchased, rather than leaving the agent to infer the relationship from a page.
- In the demo, a cURL request to a protected endpoint fails and returns an encoded payload explaining what is being bought, who is paid, and how.
- The protocol-aware flow has three visible steps:
- **Read the payment request:** one penny in `pathUSD` on the Tempo blockchain, payable to a specified recipient.
- **Approve the payment:** Kaliski explicitly authorizes it.
- **Observe the result:** the request succeeds, and a transaction appears on the blockchain.

### Negotiate checkout through seller-owned state · 11:15
- Not every purchase is an API call. Physical goods bring taxes, quantity limits, and fulfillment choices that must reach both the agent and the human accurately.
- An agent acting as an intermediary can misrelay those details, raising the risk of disputes and chargebacks even when the credential is secure.
- The **Agentic Commerce Protocol** (ACP), built with OpenAI, provides standard APIs and objects for expressing checkout.
- The mechanism is a back-and-forth among agent, seller, and payment service provider.
- When the agent creates a checkout, updates a quantity, or picks shipping, the seller responds with the latest state.
- The pattern: **the agent requests a change; the seller returns the authoritative result.** Repeating it keeps the details explicit as checkout moves toward payment.

### From a book recommendation to a paid order · 12:52
- Stripe Press is the example: a human-friendly bookshop that also exposes a robot-friendly catalog in JSON.
- Instead of clicking through pages, the agent reads structured data (images, descriptions, pricing) to select a product and start checkout.
- In a conversational interface, the buyer asks for AI-book recommendations while an adjacent panel shows the agent's requests.
- The checkout request includes buyer info, line items, and quantities. The seller returns the cart state with each item's base price, applicable tax, and fulfillment options.
- The agent can refer to those fields directly rather than extracting them from a rendered page.
- This is the shift from exploration to a transaction: discovery may involve crawling and open recommendation, but once a product is chosen, changes to payment or shipping become structured exchanges.
- Payment then proceeds using a shared payment token or another credential-relay mechanism. The model can still help choose; it no longer infers the checkout's financial state from the UI.

### Keep the merchant in control · 14:28
- The resulting flow can support cards, crypto, and other methods while preserving the seller's role.
- The merchant keeps its customer relationship and receives the signals and risk data it needs to work safely with agents.
- Supporting an automated buyer does not mean surrendering the seller's existing payment and risk decisions.
- For a business, exposing only a web UI increases the interpretation an agent must do. Agent-friendly interfaces make more of the interaction deterministic.
- For the operator, tokens, wallets, and similar mechanisms constrain accidental spending.
- The practical boundary: a nondeterministic planner operates within enforced constraints, deals with verifiable parties, and negotiates through structured state — which keeps the radius of financial risk small.

### Chain storage, recurring budgets, and adoption · 16:07
- Question: where does the blockchain part live — Tempo, Stripe, or separate? Kaliski says Stripe supports multiple networks, including Base and Tempo. Transaction data lives on those chains, and Stripe replicates a product-facing view in its own system.
- Question: could you give OpenClaw $25 a week for a particular model? He separates two ideas:
- **Recurring usage:** like a subscription charging the same card periodically, later use can be requested against an ongoing authorization relationship. He compares it to OAuth access and refresh flows.
- **Broader budgets:** raise the limit and create additional tokens, keeping each token scoped to an individual seller.
- The answer describes repeated authorization and seller-specific limits; it does not promise an automatic weekly reset or a shared budget across sellers.
- Asked whether Stripe Projects composes these primitives, he says yes: it combines shared payment tokens with a way for sellers or SaaS businesses to express their products, with recurring payments supplying the monthly part.
- Asked how many agent payments have happened and how much money has moved, he says there are no public statistics. Stripe is encouraged and wants to support more businesses, leaving adoption volume unquantified.

## Resources

- [Stripe shared payment tokens](https://docs.stripe.com/agentic-commerce/concepts/shared-payment-tokens) — how scoped payment credentials are granted and received.
- [Machine Payments Protocol](https://mpp.dev/) — the Stripe-and-Tempo protocol for paid HTTP requests.
- [Agentic Commerce Protocol](https://github.com/agentic-commerce-protocol/agentic-commerce-protocol) — versioned commerce specs, OpenAPI definitions, and schemas.
- [Stripe Projects](https://stripe.dev/blog/production-ready-dev-stack-from-terminal) — composes shared payment tokens with product expression for sellers.
