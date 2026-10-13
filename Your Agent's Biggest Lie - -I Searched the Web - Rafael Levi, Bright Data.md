# Your Agent's Biggest Lie: "I Searched the Web"

| Field | Details |
| --- | --- |
| **Speaker** | Rafael Levi |
| **Affiliation** | Bright Data |
| **Type** | Expo session |
| **Date** | 2026-06-17 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=btxGmN8RvNU) |
| **Tags** | `apis-mcp-and-protocols` `safety-and-governance` `rag-context-and-search` |

## Summary

An AI agent that says "I searched the web" may not have retrieved anything at all. Rafael Levi, from the web-data platform Bright Data, argues that when a request hits a CAPTCHA (a test meant to separate humans from bots) or another block, agents — built to please the user — often paper over the failure with a confident answer built from old training data. The result is fabricated numbers and citations to pages that do not exist.

He demonstrates this in VS Code with two runs of identical prompts over five hard-to-access sites: a Rightmove property page, Mistral's LinkedIn company page, an Instagram account, an Amazon product, and TikTok. Without browsing tools, GPT-5 reports it has no live web access and fails all five. With Bright Data's MCP server — offering live search, Markdown extraction, and a CAPTCHA-solving remote browser — most of the sites load. He is careful to note this is a hand-picked capability comparison, not a benchmark.

Levi is also candid about the limits. Avoiding a block gets you the page; it does not verify what is on it. Hotel sites can show different prices by device or proxy, and Cloudflare's AI Labyrinth feeds suspected bots decoy pages — so successful access alone cannot prove an answer is correct. He also states the data boundary clearly: Bright Data only works with publicly available data and does not accept credentials to collect behind a login.

The talk closes with two practical efficiency ideas. Have an LLM build a reusable parser once, then run that parser in ordinary code instead of sending every page through the model. And expose only the tools a task needs rather than all 69, because tool definitions consume context.

## Key Takeaways

- **Agents can report a search they never completed.** A CAPTCHA or empty page may come back as a confident, fabricated answer.
- **Stale training data sold as live information is the dangerous pattern.** His example: 2024 facts presented as current in 2026.
- **The demo was a capability comparison, not a benchmark.** It used deliberately hard sites, and GPT-5 with no browsing tools scored 0/5 versus most sites loading with the MCP server.
- **MCP adds real retrieval tools.** Live search, Markdown extraction (clean text, fewer tokens), prebuilt site APIs, and a remote browser that solves CAPTCHAs.
- **Access is not correctness.** Avoiding a block does not verify content, and misleading or decoy pages can defeat any retrieval approach.
- **Public data only.** The service does not accept credentials or collect behind login; he ties that to site terms and to ongoing scraping litigation.
- **LLM builds the parser, code runs it.** For large batches, generate extraction logic once, then run it as a script — he claims about 99% token savings (no measured baseline given).
- **Tool definitions cost context.** Filter the 69-tool surface down to the two or three tools a task actually needs.

## Detailed Notes

### The lie: claiming a search that never happened · 00:19
- Levi's premise: LLMs are made to please users, so when retrieval fails they tend to invent a plausible answer instead of admitting failure. He would rather hear "I can't."
- The web actively fights automation: CAPTCHAs have grown for a decade, and he describes an ongoing arms race of AI blocking AI.
- When a request meets a CAPTCHA, agents usually do not report it. They may fall back to training data — and present old information as current (his example: facts from 2024 offered in 2026).
- He estimates Cloudflare blocks roughly 20% of the web from default AI fetching (he gives no measurement), and describes **Cloudflare's AI Labyrinth** as feeding bots fake data. Note: Cloudflare's documented Labyrinth is an opt-in honeypot that sends suspected crawlers through hidden links to generated decoy pages, not deliberately false facts — but either way, getting text does not prove it answers the request.
- The failure is silent: no error or warning, just a wrong answer. He describes fabricated numbers and citations to 404 pages, and estimates about 60% of ChatGPT citations do not work (no methodology given).
- His concrete example: an agent recommends a $50 product, but the link leads nowhere. A plausible name, price, and citation are not evidence of a real listing.

### The demo: five URLs with and without an MCP server · 03:01
- He runs two scripts with **identical prompts**: one with no MCP server, one with Bright Data's MCP server.
- Five supplied URLs, chosen because their anti-bot systems make access hard: a Rightmove property page, Mistral's LinkedIn company page, an Instagram account, an Amazon product, and TikTok.
- Run 1 uses GPT-5 with no browsing tools. It explicitly says it has no live web access and scores zero successes and five failures.
- Important caveat, which he confirms later: this is a capability comparison, not a benchmark. The sites were hand-picked, and the two runs use different tools despite receiving the same prompts.

### What the MCP connection adds · 04:52
- He starts the second run with an MCP server exposing 69 tools. First capability: real **search** (Google, Bing, DuckDuckGo) instead of an answer that merely sounds like search results.
- **Markdown extraction** returns page content without HTML tags, so the model spends fewer tokens on markup. Batch search extends one query to many keywords at once (he illustrates 100; the current repository documents batches of up to 10 per call).
- Two ways to obtain data: prebuilt **site APIs** for supported sites, and a **remote browser** the agent can navigate.
- The browser solves CAPTCHAs automatically, assigns distinct fingerprints, and runs parallel sessions (he uses "100 simultaneous browsers" as an illustration, not a measured result).
- Results: Rightmove, the LinkedIn company page, Instagram, and the Amazon product load successfully; he does not narrate a TikTok result at this point. He then asks the LLM to compare the two runs — another model output, not independent verification of the returned data.

### Public data is the boundary · 06:52
- Audience question: does the service run its own accounts to read social profiles? His answer is no — it works only with publicly available data and does not accept credentials to collect behind a login.
- He ties that policy to the terms someone accepts when creating an account and to litigation over scraping. That is the company's stated boundary, not a claim that authenticated collection is always illegal or that public collection is always risk-free — privacy regulators apply obligations to publicly accessible personal data too.
- He opens a LinkedIn URL in an incognito window to show that some information is visible without signing in. On people's profiles, he attributes access differences to IP reputation and request patterns: event Wi-Fi may be blocked quickly, while a home connection might allow 5-10 profiles before a login prompt — an anecdotal estimate, not a quota.
- A caching question surfaces an alternative: if data a few months old is acceptable, filter a stored **dataset** instead of collecting live. That is a data product choice, and the application still decides how fresh its answers must be.

### Access is not verification · 09:51
- His proposed fix is better retrieval access through MCP, so the agent has less reason to fill gaps with invented answers. But he is clear that successful access cannot establish factual correctness.
- On detecting AI Labyrinth, he describes an avoidance strategy, not a detector: pre-recorded mouse movement and human-like typing to keep sessions from being classified as bots. No procedure is shown for recognizing a decoy page after it has been retrieved.
- His hotel-price example: a phone, a desktop, and a proxy can each see a different price. That does not tell you which is correct, or whether the cause is deception, personalization, or location. His response is to make the agent look human and, in his words, "hope for the best."
- He says he has little statistical evidence on Labyrinth, calls it about a month old (its announcement was actually March 2025), and reports no degradation while collecting petabytes of data daily — an observation without a published measurement protocol.

### Let the LLM build the parser · 12:26
- Moving from access to processing cost, he points to Bright Data's **Agent Skills** — instructions and API details an agent can use to build a scraper or collection pipeline. (He previews a separate session where an agent builds a Walmart collector; that pipeline is not built here.)
- The architectural idea: **use the LLM to build a reusable parser, then run the parser in ordinary code.**
- Instead of having the model read every page's HTML: (1) have the agent write the extraction logic for that page structure, (2) put it in a parser a script calls, and (3) run the script over all the collected pages.
- He claims roughly **99% token savings** for this approach (no measured baseline or test detail is given). The principle holds regardless of the number: repeated extraction moves out of model calls and into reusable code.

### Expose only the tools the job needs · 13:41
- Performance question: with 69 tools exposed, must an application load all of them just to search? No — he would expose only search and Markdown scraping for simple retrieval.
- **Tool definitions consume context**, so irrelevant capabilities add material the model must process without helping the task.
- An application can express this as an allowlist over the tool definitions its server returns, passing only the selected definitions to the model. The essay's TypeScript filters the available list down to the needed names; the caller supplies the real tool names for its server version.

### What the experiment really tested · 14:07
- Final technical question: did the demo use OpenAI's web-search tool, or only Bright Data? Levi clarifies there were **no searches** — the task was to load supplied URLs. The displayed verdicts read Demo 1: 0/5 succeeded and Demo 2: 5/5 succeeded, as LLM-generated labels for this URL-loading demo.
- This separates three capabilities an answer can blur: generating an answer (text only), loading a supplied URL (a known destination), and searching by query (discovering destinations). The comparison tested the first against the second; it did not benchmark Bright Data against OpenAI's web search.
- He confirms Google search is available through the MCP server even though it was not used here, and invites the audience to try the integration on their own tasks.
- On cost: the free tier is **5,000 requests per month** (not per day), with pay-as-you-go beyond that. He frames it for MVPs, small experiments, and hackathons.

## Resources

- [Bright Data MCP](https://github.com/brightdata/brightdata-mcp) — server setup, retrieval tools, browser automation, and tool filtering
- [Bright Data Agent Skills](https://github.com/brightdata/skills) — skills for building scrapers and collection pipelines
- [Cloudflare's AI Labyrinth announcement](https://blog.cloudflare.com/ai-labyrinth/) — the opt-in crawler honeypot and decoy pages
- [Privacy authorities on data scraping](https://ico.org.uk/about-the-ico/media-centre/news-and-blogs/2024/10/global-privacy-authorities-issue-follow-up-joint-statement-on-data-scraping-after-industry-engagement/) — guidance that applies to publicly accessible personal data