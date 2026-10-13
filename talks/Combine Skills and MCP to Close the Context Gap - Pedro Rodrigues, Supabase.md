# Combine Skills and MCP to Close the Context Gap

| Field | Details |
| --- | --- |
| **Speaker** | Pedro Rodrigues |
| **Affiliation** | Supabase |
| **Type** | Talk |
| **Date** | 2026-05-15 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=JT3OzDKrucU) |
| **Tags** | `rag-context-and-search` `apis-mcp-and-protocols` `agent-engineering` |

## Summary

Pedro Rodrigues, an AI tooling engineer at Supabase, argues that MCP (a standard
way for agents to call tools) and skills (packaged instructions an agent can load)
are not rivals. MCP gives an agent access to a product; a skill gives it the
guidance to use that product safely and well. A capable agent can still miss
security requirements or fall back on stale training data.

His headline example is a PostgreSQL view created over a table protected by RLS
(row-level security, which limits each user to their own rows). Unless the view is
created with `security_invoker = true`, it can run with the owner's permissions
and expose data the underlying table would hide. In Supabase's test, the agent with
only MCP omitted that flag; the agent with MCP plus the skill set it correctly.
Access to a tool was not the same as knowing how to use it safely.

Supabase shipped a "Supabase Agent Skill" and shares the lessons from writing it.
Point to one authoritative documentation source instead of duplicating docs, put
must-not-miss rules directly in the main `SKILL.md`, and be opinionated about how
work should proceed. The skill prescribes a schema workflow and carries a security
checklist.

They evaluated the skill by watching agent behavior, not by reading the Markdown.
Across six real-project scenarios, four model setups and three conditions, MCP plus
the skill scored higher on task completeness than baseline or MCP alone on every
tested model. Rodrigues's conclusion: the bottleneck today is guidance, not more
context.

## Key Takeaways

- **MCP and skills are complementary, not competitors.** MCP supplies tools; a skill supplies product-specific guidance.
- **A tool the agent can call is not a tool it knows how to use safely.** The PostgreSQL RLS/view example shows real data exposure.
- **`security_invoker = true` matters** (PostgreSQL 15+): without it, a view can use the owner's permissions and bypass RLS on the underlying table.
- **Don't duplicate documentation.** Point to one source of truth and insist the agent retrieve it; agents default to stale training data.
- **If something can be skipped, it will be.** Put essential, stable rules (like a security checklist) in the main `SKILL.md`, not a reference file the agent may skip.
- **Be opinionated.** Prescribe the workflows that fit your product, such as the schema workflow below.
- **Evaluate the skill through agent behavior**, not by reading the Markdown — like CI tests for an agent.
- **Distribution is still unsettled.** Packaging skills in the repo (for example `.claude` or `.cursor` plugins) is a practical route.

## Detailed Notes

### What does an agent need beyond access? · 00:18
- Rodrigues notes the "MCP vs skills" debate has cooled; the live debate is now MCP versus CLI.
- Writing a good product skill is hard — the hard part is deciding what guidance a complex product needs, not writing the Markdown.
- Agents handle mundane tasks well, but on anything newer than their training data they need guidance.
- At Supabase, agents missed row-level security (RLS) requirements, which can expose application data.
- They also lean on stale training knowledge and are reluctant to admit they lack fresh information.
- Skill background: a folder with a main `SKILL.md`, front matter (`name`, `description`) that helps the agent decide whether to load it, optional scripts, and reference files loaded only when needed.

### A SQL view can reopen protected data · 03:35
- Test setup: Claude Sonnet 4.6 got the same prompt under two conditions — MCP only, and MCP plus the skill.
- Task: create a SQL view over a collaborative table that already had RLS enabled, so each user sees only their own rows.
- PostgreSQL detail: by default a view uses the view owner's permissions and policies, so a privileged owner can bypass the restrictions a normal caller would face.
- The fix is `WITH (security_invoker = true)`, which makes the checks run as the caller; it requires PostgreSQL 15 or later.
- Result: the MCP-only agent omitted the flag and exposed data; the skill-equipped agent included it.
- This motivated the Supabase Agent Skill, announced at the talk after a few months of work.

### Keep documentation authoritative and essential checks unavoidable · 05:56
- Principle 1: don't duplicate documentation. Tell the agent where and how to find current information instead of copying it.
  - A bare link is not enough; be persistent, because agents otherwise continue from training data.
- Supabase's experimental "docs over SSH" exposes docs as a virtual filesystem so agents navigate with familiar file and Linux tools. It is a docs interface, not a remote shell, and easier navigation is a premise, not a measured result.
- Principle 2: if something can be skipped, it will be. Fetching data and making tool calls cost the agent, so it prefers its training data.
- Even a loaded skill does not guarantee the agent reads its references; loading two files was hard, three or four less dependable.
- Supabase first put its security checklist in a reference file and agents kept missing it; moving it into `SKILL.md` fixed that.
- Division of labor: current details → retrieve from docs; stable indispensable rules → `SKILL.md`; optional depth → references.

### Prescribe the schema workflow · 09:11
- Principle 3: be opinionated. The product team knows which workflows fit its system.
- Supabase is a backend service (database, storage, authentication), and an agent can change the database schema directly.
- The skill prescribes this sequence:
  1. Iterate with direct DDL on a development or staging database.
  2. Run the advisors, which act as linters for security and performance issues.
  3. Fix what they report.
  4. Create the migration file only after the schema settles.
- Why it matters: restricting to dev/staging avoids experimenting on production, and delaying the migration prevents a new migration for every intermediate edit.
- Rodrigues notes the current skill has since added distinctions between declarative and imperative projects.

### Test the Markdown through agent behavior · 10:59
- Evals are tests for agents: you watch tool calls, reasoning and whether requirements are met, like CI for a non-deterministic system.
- Early evaluation: six scenarios on real Supabase projects, four agent/model setups from two vendors, three conditions.
  - Baseline: no MCP and no skill.
  - MCP only: MCP server, no skill.
  - MCP plus skill: tools and product instructions.
- Completeness was graded in Braintrust with a four-level score; the levels are not defined in the talk.
- Configurations: Claude Code with Opus 4.6 and Sonnet 4.6; Codex with GPT-5.4 and GPT-5.4 mini.
- Result: MCP plus skill scored higher on every tested model. Supabase calls these small-sample early results.
- Note: there was no skill-only condition, so this tests adding a skill to MCP, not the skill alone.

### Start small and revise the guidance · 12:59
- Supabase already had the MCP tools; what was missing was guidance on how to operate them.
- Skills are agent-agnostic and increasingly adopted, so the instructions are reusable across agents.
- The authoring recipe: point to one source of truth, prescribe the workflows that matter, and start minimal.
- Expand and version the skill through testing. Rodrigues frames the bottleneck as guidance, not context.
- Install command: `npx skills add supabase/agent-skills`.
- At the talk he also ran a QR-code giveaway (a Mac mini) for signing up to Supabase.

### Semantic retrieval and repository distribution · 14:55
- Q: demand for vector databases? A: growing customer interest, mainly for embeddings.
- Rodrigues's own interest: adding semantic search to the docs-over-SSH interface, so agents search by meaning instead of only navigating files and matching text. This is a possibility, not a shipped feature.
- Q: how do you distribute skills in an org? A: still unsettled; the Vercel skills CLI is one route and plugins another, but plugins can be tied to specific agent environments.
- Supabase's internal approach: package skills in the repositories themselves (for example `.claude` or `.cursor` plugins).
- The skills package can then fetch from public repos or repos the user can access.
- Other companies, he says, follow the same pattern: keep the skill next to the project knowledge it describes.

## Resources

- [Supabase Agent Skills](https://github.com/supabase/agent-skills)
- [Supabase Agent Skills launch and evaluation](https://supabase.com/blog/supabase-agent-skills)
- [PostgreSQL CREATE VIEW (view security semantics)](https://www.postgresql.org/docs/15/sql-createview.html)
- [Supabase docs over SSH](https://supabase.com/blog/supabase-docs-over-ssh)
- [Agent Skills specification](https://agentskills.io/specification)
- [Vercel skills CLI](https://github.com/vercel-labs/skills)
- [Supabase MCP setup](https://supabase.com/docs/guides/ai-tools/mcp)
