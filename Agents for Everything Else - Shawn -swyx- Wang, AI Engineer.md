# Agents for Everything Else

| Field | Details |
| --- | --- |
| **Speaker** | Shawn "swyx" Wang |
| **Affiliation** | AI Engineer |
| **Type** | Talk |
| **Date** | 2026-05-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zepu8Kk6FBQ) |
| **Tags** | `apis-mcp-and-protocols` `coding-and-developer-tools` `agent-engineering` |

## Summary

Swyx co-founded the AI Engineer conference and, with a nine-person team, uses coding agents to run it. His talk moves past "agents write code" to "agents do everything else": turning Figma designs into a live website, managing the conference schedule, syncing data from vendors, and even sourcing a lobster prop in London.

The title theme is **tiny teams** — teams whose revenue in millions exceeds their headcount. Swyx says AI Engineer runs a business of over $9 million with nine full-time people, and he pushes the idea that every company can have a tiny team, not just lone founders chasing unicorns.

His most interesting claim is about where the productivity shows up. When the wait between an idea and a visible result shrinks, non-technical teammates start initiating work themselves — animations, polish, playful experiments — because it is finally fun and unblocked. He calls this "agents for everything else" and predicts it will be a top trend of 2026.

He ends on product design: if a growing share of your users are bots or agents, your dashboards matter less than your APIs, CLIs, and MCP interfaces. He frames this as "agent experience" — designing for a user that is arriving.

## Key Takeaways

- **Tiny teams are the theme.** Swyx's measure is revenue in millions exceeding employee count — and any size of company can contain one.
- **Agents remove "yak shaving."** Before Devin could build the site, Claude Cowork had to wire Devin to Figma. Removing prerequisites is underrated value.
- **A shorter feedback loop changes behavior.** When teammates stop blocking on a developer, they start work they would normally skip.
- **Schedule as code beats a CMS.** With the source of truth in code, any coding agent can maintain the conference schedule from an email or screenshot.
- **Nine people ran a ~1,000-person conference**, and swyx expects to handle 6,000 in San Francisco at the same size.
- **Agents handle research and buying too.** He asked Devin "where can I get a lobster in London?" and got phone numbers, emails, and sites to review.
- **Replacing SaaS needs the team on board.** The people who operate the result bear the cost of failure, so identify the top three concerns and reduce them.
- **Design for the agent as user.** swyx cites a claimed 60% of Vercel's user base being bots/agents, motivating APIs, CLIs, and MCP interfaces over dashboards.

## Detailed Notes

### How does a tiny team run an AI conference? · 00:51
- Swyx traces his annual keynotes: first the productivity gain from AI, then "use more AI" as costs fall, and now **tiny teams** — organizations with more millions in revenue than employees.
- He describes a rough heuristic of AI costs dropping about 100x every 12–18 months, and flags it as a working estimate rather than a guaranteed rate for every workload.
- The **Tiny Teams Playbook** (from a World's Fair track) uses the more specific measure of annual recurring revenue.
- He likes the idea because large companies can have tiny teams too; it does not require finding a single founder building a billion-dollar business.
- His example: AI Engineer, led by Ben Li and himself, has nine full-time people and a business over $9 million (business size, not ARR). The "AGI pills" given to attendees are the running joke.

### The website begins with a setup problem · 03:07
- The original stack was stable and mostly non-AI: Figma, React, Supabase, Tito, Google Sheets, Sessionize.
- Joining Cognition gave him serious access to coding agents, and free access encouraged real experimentation. He added **Devin** (a coding agent) to the company Slack and showed colleagues how to request website changes.
- Then the designer brought a Figma page expecting a one-, two-, or four-week implementation. Before Devin could build it, someone had to connect Devin to Figma — so **Claude Cowork** did the setup.
- Lesson one: agents save you the **yak shaving** — the chain of "first I have to do that" tasks, especially dependency installs and broken Python environments.
- Once connected, he reports Devin produced a working site that matched the Figma "pixel for pixel". He gives no elapsed time. Seeing it himself changed his confidence.

### A shorter feedback loop changes what people attempt · 05:17
- Usage grew fast: he points to a Slack thread with 207 replies. He would start work before bed; the designer, in Indonesia, would continue with Devin overnight.
- The designer began sending **red-line annotations**, communicating changes visually as he would to a human developer. Swyx says he never taught this and supplied no manual — it was just how you communicate with another person.
- The team started doing things it would normally skip: a website Easter egg on ultrawide screens, and a design inspired by a viral tweet he fed to Devin "just to see". That thread hit 127 replies.
- The reason the designer joined even the throwaway project was simple: it was fun, and the wait between idea and result was gone.
- His key point: **the gain showed up in human initiative**, not just generated code. That is "agents for everything else", and it is why the mostly non-technical team had to get comfortable with agents.

### Make conference data something the agent can maintain · 07:57
- A conference is a giant data-management problem: 130 speakers, a couple dozen sponsors, and attendees with varied needs — basically a content management system.
- He was not a fan of Sanity. The unlock came when he threw away the CMS, committed the schedule to code, and treated that code as the **source of truth**.
- With schedule-as-code, Devin (or any coding agent) maintains it. A speaker-change email or a pasted screenshot becomes the task context, and the agent edits the schedule directly.
- His scale claim: nine people managing a roughly 1,000-person conference, and he expects to handle about 6,000 in San Francisco that summer without growing the team.
- He says throughput arrives after sufficient onboarding and ironed-out workflows. Agents also handle **ETL** (extracting, transforming, and loading data between systems) from an external vendor that holds data missing from the central source.

### Finding a lobster in London · 09:33
- After seeing a viral post with a claw next to the Wall Street bull, he wanted a similar prop. He asked Devin: "Where can I get a lobster in London?"
- The agent returned phone numbers, email addresses, and websites. Swyx clicked through, judged the options, and asked for more research. He credits that process with sourcing the lobster — his workflow included evaluation, not an unattended purchase.
- For this kind of task, he says requirements are simple: web access and a capable enough model.
- Whether the assistant is OpenClaw, NanoClaw, or something else matters less than using agents for work you would otherwise give to an executive assistant or junior employee.
- The framing: a coding agent becomes an on-demand worker for research and coordination, not just implementation.

### From planning notes to speaker research · 10:44
- Swyx discloses he now advises Cognition rather than working there in the earlier capacity, and has started exploring **Town** as coding agents expand into general knowledge work.
- He links this to purpose-built knowledge tools — Andrej Karpathy's discussions of wikis, and OpenClaw adopting that approach — and calls it maybe a top three-to-five trend of 2026.
- For World's Fair planning he starts with Apple Notes containing prospective speakers and tracks, feeds those notes to Town, and gets back a formatted Notion document with research on the speakers he is considering.
- The transformation matters: a personal planning list becomes researched material that supports the next decision.

### The people who inherit a replacement must trust it · 11:43
- Success with tasks leads to a bigger ambition: replacing an entire SaaS (software-as-a-service) product with something built in-house.
- He shows himself debating that replacement with employees, joking that his enthusiasm is "psychosis", but he takes the disagreement seriously.
- His rule: a manager cannot dismiss the concerns of the people who will operate the result, because they are the ones who bear the cost when it fails — and he acknowledges the team does get it wrong.
- His method: identify the top three concerns and systematically reduce them. At the time of the talk, that evaluation was still underway.
- Framing for the note: this is an approach to earning confidence, not a finished story of removing a SaaS dependency.

### Build interfaces for the user that is arriving · 12:44
- The internal changes raise a product question: the entity using a service may increasingly be an agent.
- He attributes to Malte's opening keynote a claim that 60% of Vercel's user base is bots or agents. He does not establish the denominator or period, so this should not be read as a measured share.
- That motivates his emphasis on **APIs, CLIs, and MCP interfaces** (MCP is a standard way for agents to call tools) as the ways agents actually use software.
- He points to the MCP Apps speakers (Ido and Liad) on shipping UI into someone else's application, so a product's interface can appear inside an MCP client rather than only its own dashboard.
- **"Agent experience"** means designing for this changing primary user. His own example: he cares less about navigating the Figma dashboard when he can hand the task to Claude Cowork.
- His closing invitation is to bring that experimentation into everyday work, and the AGI-pill joke returns as a nudge to bring hesitant colleagues along.

## Resources

- **The Tiny Teams Playbook** (article) — swyx's synthesis of hiring, culture, operations, and technical practice for tiny teams.
- **Use Devin in Slack** (docs) — setup and collaboration instructions.
- **Devin MCP use cases** (docs) — connecting Devin to external tools including Figma.
- **Town** (demo) — an AI assistant for delegated work that keeps context about how its user works.
- **AI Index 2025: Inference Cost Trends** (paper) — benchmark evidence on falling inference prices.
- **MCP Apps** (article) — how interactive app interfaces appear inside MCP clients.
- **NanoClaw** (repo) — a customizable personal assistant running agents in separate Linux containers.
- **OpenClaw** (docs) — a personal AI assistant project for delegating everyday knowledge work.
