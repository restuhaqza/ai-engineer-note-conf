# Software Engineering Is Becoming Plan and Review

| Field | Details |
| --- | --- |
| **Speaker** | Louis Knight-Webb |
| **Affiliation** | Vibe Kanban |
| **Type** | Talk |
| **Date** | 2026-05-02 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=W76woOYHlvY) |
| **Tags** | `observability-and-reliability` `architecture` `coding-and-developer-tools` `leadership` |

## Summary

Louis Knight-Webb, founder of Vibe Kanban, asks a practical question: as coding agents get better at writing code, what do software engineers actually spend their day on? His answer is that the job narrows into two activities — **planning** and **reviewing**. As each generation of tools took over more implementation, the hours freed from typing did not become free time; they moved into specifying work up front and checking the result afterward.

He contrasts two working styles. A **plan-heavy** approach invests time in detailed specs and edge cases before the agent starts. A **review-heavy** approach lets the agent produce something quickly and then corrects it. Knight-Webb prefers planning when human time is the scarce resource, but points out that front-end feature work is genuinely hard to specify because state, interaction, and styling only become clear once you can try the result.

Longer agent runs then change human behavior. Once an agent works for more than roughly five minutes, sitting and watching the terminal stops working, and developers start running several agents at once. That parallel work fragments attention, so he argues for interfaces that let an agent run as long as it usefully can and then hand control back for sustained focus.

The demonstration has an unusual ending: he announces, live on stage, that the company behind Vibe Kanban is shutting down, and uses the product itself to publish the announcement.

## Key Takeaways

- **The work changes shape, not just size.** Time freed from writing code moves into planning and reviewing it; output rises, but the day looks different.
- **Pay up front or pay in corrections.** His rule of thumb: about five minutes of planning can save roughly thirty minutes of reviewing generated code.
- **How much you can specify depends on the task.** Front-end features resist full specs; back-end features, refactors, and migrations suit plan-first, test-driven execution.
- **Agent runs are getting longer.** Tools advanced from completing a line, to a file, to successful five- and ten-minute task runs.
- **The five-minute threshold matters.** Beyond it, passive waiting becomes awkward, and "terminal maxing" (many agents in parallel) becomes common.
- **Parallelism alone is not enough.** Spreading review and previews across tools pulls the developer out of context every few seconds.
- **Aim for "focus maxing."** Let agents work long, then yield in a way that lets a person give one task real attention.
- **Finishing the diff is not finishing the job.** Changes still have to be shepherded through PR comments until they are deployed.

## Detailed Notes

### What remains when AI writes the code? · 00:16
- Knight-Webb introduces himself as founder of Vibe Kanban and the person who started London's AI Tinkerers community. He notes he once appeared ahead of OpenAI on a SWE-bench Verified leaderboard (a benchmark for fixing real GitHub issues), though the snapshot was already months old.
- He breaks engineering into four activities: planning, writing code, reviewing your own code, and reviewing other people's code.
- Before GitHub Copilot, writing took most of his time. Then Copilot offered completions, ChatGPT made generating whole functions practical, the original Cursor extended completions to a page, and Claude Code took over enough implementation that he does little direct writing.
- He rejects the idea that automating four hours of coding returns four free hours. In his rough estimate, a half hour previously spent coding returns about twenty minutes, with the rest going into planning and review.

### Pay for clarity before execution · 03:30
- One style front-loads effort: comprehensive Markdown plans, spec frameworks, or a model that keeps asking questions until its uncertainties are exhausted.
- The mechanism is not a specific framework. It is supplying missing context and settling edge cases before they turn into implementation decisions.
- The cost is more planning time; the benefit is fewer mistakes and fewer review rounds.
- The other style starts loose ("add a contact form") and lets the agent produce something to react to. The request is fast, but the payment arrives later as corrections.
- Repeatedly receiving half-finished work is expensive because each round forces the human to rebuild the task's context, not just type another message.

### The task decides how much you can specify · 06:08
- Front-end feature work is hard to specify exhaustively: a new interface combines state, interactions, animations, styles, and behavior, and some of what you want only becomes clear when you can interact with it. For this, he stays in the loop with the agent.
- Back-end features allow more of the desired behavior to be expressed as tests. Refactoring and migrations are strong candidates for a plan followed by test-driven execution, on either side of the app.
- His matrix: new front-end features are review-heavy; new back-end features, refactors, and migrations are plan-heavy.
- For the plan-heavy cases, his goal is to remove the human from the agent's execution loop, letting tests guide the work.
- He distills the trade-off into a heuristic: five minutes of planning saves thirty minutes of reviewing. It is a rule for allocating attention, not a measured conversion rate.

### Longer runs change the workflow · 07:26
- As models and tools improve, an agent does more before returning control — and you wait longer after each prompt.
- He traces the progression: Copilot completing a line (seconds), original Cursor completing a file (about thirty seconds), Claude Code the previous year (one or two minutes), and Claude Code this year (successful five- to ten-minute runs).
- The added time has a mechanism: returning code is fast, running a type checker is slower, and testing the change adds more. He calls Playwright MCP browser testing an order of magnitude slower than the preceding steps (without a controlled comparison).
- The thing to optimize is human time spent working with the agent: a long run is worthwhile if verification produces a result that needs less correction.
- He forecasts autonomous front-end QA as the next big breakthrough within about nine months: the agent runs the project, clicks through it, finds bugs, fixes them, and checks the result. He has seen demos but not mainstream use.
- He puts the rough waiting threshold at five minutes. Beyond it, passive waiting stops working, and a twenty-minute agent needs a different way to organize the job.

### Parallelize agents without fragmenting the human · 10:26
- One response is running multiple agents at once — what he calls "terminal maxing." While one agent works for ten minutes, you review another's result, ideally with the next task ready when review finishes.
- Vibe Kanban began about a year earlier to make this manageable: a sidebar of workspaces running agents such as Codex and Claude Code, diffs and GitHub-style comments for review, and a live preview where you can click an element to make it larger or smaller. He dates these features to June 14, 2025.
- Simply adding terminals does not solve the attention problem. Existing tools scatter code review and previews across different places, a big change from concentrating on one implementation task.
- He coins "focus maxing": do not pull a person into a new context every thirty seconds. An agent should run as long as it usefully can, then yield so the human can give the task sustained attention.
- He expects companies with money at stake to keep reading generated code before shipping, even with AI-assisted review.
- The workflow extends past the diff. A change must be shepherded until deployed — at its simplest, an agent that monitors a pull request for comments and responds automatically. Task completion is an intermediate state, not the end.

### Closing the company live on stage · 14:01
- He submitted the talk weeks earlier but decided on a Tuesday to shut down the company behind Vibe Kanban. The planned product pitch becomes a live demo: publish the shutdown announcement using the product.
- He supplies a prewritten note and asks the agent to add it as a blog post. The writing is done; the delegated work is integrating it into the website ("Goodbye bloop").
- The workspace starts in a fixed order: create a Git worktree for the task, run a setup script inside it to install the website's dependencies, then start the selected coding agent. He normally uses Codex and says the tool supports eight popular agents.
- The Git part can be expressed directly in a shell:
  ```bash
  git worktree add -b shutdown-announcement ../website-announcement
  git -C ../website-announcement status --short
  ```
- The agent returns a finished blog post, the live preview compiles, and he approves the appearance. Opening a pull request hits an uncommitted-changes obstacle he does not explain.
- He says the task is done and the change will go through Cloudflare's CDN. The commit and merge steps between completion and delivery are not shown — landing exactly on the gap between a finished implementation and a published change.

### A popular project and an unsustainable business · 15:31
- Asked whether he is sad, he says that after much deliberation he feels relieved: running a company carries responsibility for staff and investors.
- He reports 30,000 monthly active users and 25,000 GitHub stars at the point of shutdown. The project continues non-commercially, and the team is still pushing changes.
- The problem is not users but money. In his diagnosis, the businesses that make money sell to enterprises and resell tokens; Vibe Kanban does neither. It is an interface for running external agents, so much of the spending it enables goes elsewhere.
- His illustration: a customer pays Vibe Kanban about $30, then uses a button that helps them spend $3,000 with Codex. Those figures show the commercial relationship, not average spending.
- Users are largely individuals, startups, and smaller companies. He thinks an enterprise move was possible but no longer attractive in a mature market — his blunt line is that there is "no fun playing for eighth place."
- An audience member asks whether investors are learning about the closure after the room; he initially says no, then admits some still need a call.

### What he would do again · 18:18
- What's next: take time off, then start another company. His co-founders plan to join a lab, and most of the team have found jobs at agent labs.
- His retrospective is positive. He initially says he would change nothing: it was the most interesting work he has done, at the cutting edge of agents, and it made him more valuable.
- The most valuable lessons were about people and effort. Several rounds of the team produced a final group he considered much stronger, and learning what hard work meant took time.
- His concrete memory is being in the office at midnight on a Saturday with the whole team still motivated.
- Pressed on what he would change if he could go back: he would hire someone who is very good at selling to enterprise. The closing lesson is a business decision, even in a talk about how engineering work is changing.

## Resources

- [Vibe Kanban](https://github.com/BloopAI/vibe-kanban) — open-source interface for running coding agents in isolated workspaces and reviewing their output
- [Goodbye bloop](https://www.vibekanban.com/blog/shutdown) — Knight-Webb's announcement of the company's closure and the project's continuation as open source
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) — Microsoft's browser-automation MCP server (MCP is a standard way for agents to call tools), used for automated front-end QA
- "Scoring 71% on SWE-bench Verified in half the steps" — the team's benchmark report (mentioned in the bundle)
- "Can Coding Agents Do QA With a Browser?" — practical experiments with browser-enabled agents (mentioned in the bundle)
