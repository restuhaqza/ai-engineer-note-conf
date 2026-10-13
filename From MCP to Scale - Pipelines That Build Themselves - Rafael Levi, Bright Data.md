# From MCP to Scale: Pipelines That Build Themselves

| Field | Details |
| --- | --- |
| **Speaker** | Rafael Levi |
| **Affiliation** | Bright Data |
| **Type** | Expo session |
| **Date** | 2026-06-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zTZ0qunQXnM) |
| **Tags** | `agent-engineering` `apis-mcp-and-protocols` `coding-and-developer-tools` `observability-and-reliability` `infrastructure-and-deployment` |

## Summary

Rafael Levi (Bright Data) shows how to collect data at scale without paying an
LLM (large language model) to read every page. The trick is to have the agent
build a reusable scraper once, then let ordinary code parse the pages. The model
is used to build, check, and repair the pipeline — not to parse the same content
over and over.

The demonstration runs live in Claude Code with Bright Data's MCP (a standard
way for agents to call tools) already connected. Levi picks a protected
marketplace, has the agent inspect the page HTML, find selectors, and generate a
Python scraper. He then shows the same pattern on a site he has never used, and
walks through token savings, code inspection, and scheduled self-repair.

The talk also draws clear boundaries. The service handles public data only —
nothing behind a login — and users still must respect a site's terms. For jobs
that need real interaction (filling forms, clicking through a hashed URL),
remote browsers with human-like movement can step in, and a small model is often
enough.

## Key Takeaways

- **Build a scraper, don't parse every page.** Repeated LLM page-reading burns
  tokens; generating a parser once and reusing it is far cheaper.
- **Access comes first.** The MCP unlocks sites protected by CAPTCHA and anti-bot
  systems so the agent can even see the page.
- **The scraper tax is real.** Writing a scraper is only the start; you must
  watch it and fix it when selectors change or fields disappear.
- **Validation, not just exit codes.** A process finishing successfully does not
  prove the expected fields were collected.
- **Make inputs, not one-off code.** A good scraper takes a keyword and page
  count so a new search is a parameter change, not a rewrite.
- **Measure the savings.** Levi reports roughly 62% token savings on one run and
  about one million tokens on an earlier three-page run.
- **Public data only.** No logins, and always check a site's terms and robots
  rules before scraping.
- **Click when you must.** Remote browsers supply human-like typing and mouse
  movement, so even a small model like Claude Haiku can drive them.

## Detailed Notes

### Ten thousand products should not mean ten thousand LLM extractions · 00:17
- The previous session covered reaching sites behind CAPTCHA and bot detection.
  This session is about what happens after you can access them.
- The question Levi hears often: "I need to scan 10,000 products." Parsing each
  page with an LLM would be far too many tokens.
- His answer: ask the model to **build a scraper**, then let the scraper process
  the pages. The reusable artifact is executable code, not a model response.
- Bright Data's **skills** teach the agent how to build a pipeline. The MCP
  retrieves the target HTML so the agent can discover the selectors it needs.

### The scraper tax never ends · 01:16
- Called the "scraper tax": once you write a scraper, you owe ongoing collection,
  monitoring, and repair. Maintenance can cost more effort than the first build.
- Changing selectors and React-driven pages make this harder.
- An agent with website access can fix it: inspect the page, identify the data,
  write the fix, and run it.
- **Set explicit data validation.** A clean process exit does not mean the right
  fields were collected.
- Levi runs daily collections checked by an LLM client every 30 minutes. Healthy
  data shuts the client down; a missing field triggers a repair. His five-minute
  fix example is meant to replace waking a maintainer at night.

### Give the agent a reusable interface · 02:27
- The demo uses Claude Code with Bright Data MCP connected, plus the Bright Data
  skills repository for scraper-building instructions and best practices.
- He contrasts multi-week setups of the past with a build he says now takes about
  three minutes.
- The first request defines two inputs: a search keyword and a maximum page
  count. After correcting the domain to `walmart.com`, he asks for headphones
  across three pages.
- Those inputs matter because the output should be a reusable scraper, not code
  wired to one product category.
- The build sequence: (1) retrieve scraper-building skills, (2) use the MCP to
  inspect the page HTML, (3) identify selectors and generate the scraper.
- The visible `scrape_as_markdown` call reads the skill text. HTML is for
  discovering structure; Markdown is enough to read instructions.

### From one search to a scheduled listener · 04:58
- Levi reports an earlier three-page run saved about one million tokens by
  generating a scraper and parsing HTML with a script. That is a reported
  comparison; the token accounting is not shown here.
- The mechanism is simple: the model helps write the extraction logic, ordinary
  code does the repeated parsing.
- It is useful even without a production pipeline. Instead of reading a review
  site, a shopper could ask for the best-reviewed headphones across marketplace
  pages and have the model answer from extracted data.
- An old leftover scraper forces a fresh test: he stops the run, deletes it, and
  lets the audience pick another marketplace. Amazon is suggested, but he says it
  is less aggressive than expected. The audience supplies Very (`very.com`).
- He repeats the same headphones task on a site he says he has never used. The
  switch keeps the task but removes the pre-prepared Walmart scraper.
- Scheduling turns a one-time extraction into a standing watch. He describes a
  house-search listener that checked an area every half hour and found where he
  now lives.
- Why Claude Code over Codex: personal preference for fewer detours; he says Codex
  also does good work.
- He estimates manual selector inspection and scraper building used to take a day
  to a day and a half. The browser work need not run on his laptop: he says the
  platform has over 150 million IPs and could control 1,000 browsers on its
  servers. Those are infrastructure claims, not a concurrency test here.

### Choose the right web-access interface · 10:08
- An audience question prompts a closer look at MCP. Levi describes a 66-tool
  catalog; the current README lists a different count, so do not expect the exact
  inventory to match.
- At the request layer, the agent supplies a URL and the service handles access:
  correct headers and cookies, plus CAPTCHA solving with a token when needed. The
  agent gets page content instead of building browser-like requests itself.
- For retrieval, match the output to the next stage: full HTML for discovering
  selectors; `scrape_as_markdown` when you only need readable text. Markdown
  cuts what you pass to the model when structure is irrelevant.
- Levi also mentions about 500 prebuilt domain APIs. For an Amazon product, an
  agent can ask for structured JSON instead of building a custom scraper.
- Remote browsers cover work where fetching a URL is not enough and interaction
  is required.
- In a live comparison, a plain fetch of Walmart hits a "robot or human" security
  screen. The same request through MCP's `scrape_as_markdown` returns headphone
  product results. He speculates a remote browser may be handling a click-and-hold
  check, though the delay alone does not prove which operation occurred.

### Measure the savings, then inspect and reuse the code · 13:36
- Levi asks the agent to estimate how many tokens it saved versus parsing pages
  directly. For 90 Very products he reads an about 62% saving from the agent's
  input/output/total breakdown.
- He calls that relatively low and wonders if the site has unusually structured
  HTML. The source does not give the model, tokenizer, or baseline counts, so the
  figure cannot be reconciled with the earlier million-token estimate.
- The generated code is also what the agent would maintain. Levi proposes a
  Claude schedule that runs and checks the scraper every 30 minutes and repairs
  it when it breaks.
- Asked to show the code, he opens the scraper. It calls the **Web Unlocker API**
  to fetch page data, then runs a generated parser and output schema.
- The request is similar to MCP's `scrape_as_markdown` — the same access function
  — but the output interfaces differ: Web Unlocker can return raw content or a
  JSON envelope, while the MCP tool returns Markdown.
- The Python code exposes the keyword and max-page parameters and loops through
  pages. That is the key boundary: a new search changes the function inputs, not
  the whole parser.
- He runs a second search (laptops, two pages). Before it finishes he estimates
  roughly 60 tokens to invoke the script and about 100 to get the data — early
  estimates, not final totals.
- Passing selected JSON fields is more economical than sending whole pages, but
  JSON formatting alone does not guarantee fewer tokens than Markdown.

### Public access still has limits · 17:44
- An audience member asks about authenticated sites. Levi sets the boundary: this
  is public data only, nothing behind a login.
- Creating an account means accepting a site's terms. He advises checking
  restrictions on scraping and robots, and points to LinkedIn litigation and
  Twitter's lockdown after Musk's acquisition as examples of data-access
  conflicts.
- Checkpoints that ask you to accept terms mid-search deserve the same care. He
  says Bright Data's scraping avoids login and terms acceptance, and cites
  litigation outcomes involving Meta and X.
- He compares public collection to writing down prices on a counter and selling
  those observations.
- The essay cautions this analogy is not blanket permission. The Meta ruling
  concerned particular contract claims and logged-off scraping; the X dismissal
  rested on grounds more specific than a broad public-data exemption. Technical
  access, contractual duties, and permitted reuse are separate questions.

### When the pipeline needs to click · 23:01
- The final question moves beyond scraping: can the agent fill and submit a form?
  Levi says yes, while keeping the no-login boundary.
- Flight search is his example. A service like Skyscanner may produce a hashed
  URL the agent cannot build directly, so it must operate the search interface.
- For geo-restricted results, the agent can request a remote browser with a US IP,
  then fill fields and click through the search.
- The browser infrastructure supplies prerecorded behavior: mouse movement
  instead of instant jumps, varied typing speed, and possibly mistakes. This is
  meant to make interaction look human to a site's trackers.
- This also affects model choice. Levi says Claude Haiku is enough for his
  browsing agents when the browser handles the behavioral masking. The model
  decides what to do; the browser performs the interaction; the script handles
  repeated extraction.
- He notes the MCP is most useful for roughly 20% of domains — heavily protected
  sites such as those behind Akamai, DataDome, or Cloudflare, often real estate
  and large e-commerce. That percentage is his estimate, not a measured result.
- Listeners need not collect millions of records. His restaurant-reservation
  listener has waited two months for a table to open.

## Resources

- [Bright Data skills](https://github.com/brightdata/skills) — agent skills for scraper construction, site analysis, and pagination
- [Bright Data MCP](https://github.com/brightdata/brightdata-mcp/blob/main/README.md) — setup and tool docs for search, page retrieval, and structured extraction
- [Current MCP setup and billing FAQ](https://docs.brightdata.com/ai/mcp-server/faqs) — free-tier limits, browser tools, and timeouts
- [Web Unlocker API](https://docs.brightdata.com/scraping-automation/web-unlocker/introduction) — request-based retrieval of public page content
- Rafael Levi's Bright Data Hackathon Kit — Python and JavaScript examples with MCP integration
