# Skill Issue: How We Used AI to Make Agents Actually Good at Supabase

| Field | Details |
| --- | --- |
| **Speaker** | Pedro Rodrigues |
| **Affiliation** | Supabase |
| **Type** | Workshop |
| **Date** | 2026-05-04 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=GmAQKINjv1E) |
| **Tags** | `coding-and-developer-tools` `evals` `agent-engineering` `enterprise` `developer-workflows-and-testing` |

## Summary

Pedro Rodrigues, a Supabase AI tooling engineer, works on **agent experience (AX)** — the agent counterpart to developer experience. His workshop asks a practical question: how do you give a coding agent the product knowledge it needs, and then check that the knowledge actually changes its behavior? His answer is a skill: a folder with a `SKILL.md` file, optional reference documents, and optional scripts.

The demo is a deliberately vulnerable performance-review app with four switchable accounts (an employee, two managers, and HR). Rodrigues asks Claude to build a `department_stats` view showing headcount and average salary by department. The SQL works and the numbers look right — but a manager can see other departments' data. The cause is a Postgres default: a view runs with the permissions of the account that created it, so it bypasses the app's row-level security unless `security_invoker` is enabled.

A prepared security skill nudges the agent to add that flag, proving the behavior can be steered on demand. Rodrigues then automates the test as an evaluation, comparing runs with and without the skill. The twist is that his own grader is buggy: it marks the with-skill run as failing and the without-skill run as passing, even though the inspected outputs are effectively the same. The lesson is that writing evals is hard, and you must check the grader against the evidence before trusting its verdict.

He closes with production advice: treat a skill as maintained documentation. Keep it small, publish it with the repository, update it when workflows change, and include only the skills a given task actually needs.

## Key Takeaways

- **A skill is a folder, not a long prompt.** `SKILL.md` holds the instructions and frontmatter; reference files and scripts are optional extras.
- **Progressive disclosure is the point.** The agent sees a skill's metadata first, then loads the full body only when the task needs it.
- **Skills and MCP solve different problems.** A skill supplies context and workflow; MCP (a standard way for agents to call tools) supplies an integration the agent can act through.
- **Define success before writing the skill.** Decide whether it should route the agent to docs, enforce a workflow, or produce a result, then test that.
- **Deterministic checks stay possible.** A tool was called or it was not; a view has the required setting or it does not.
- **Verify behavior, not just SQL.** A successful query can still expose data it should not; check who can see the result.
- **Debug your grader.** The live evaluation failed because of faulty assertion logic, not because the skill was ignored.
- **Treat production skills as documentation.** Keep them small, current, and testable, and prompt periodic review where load telemetry exists.

## Detailed Notes

### Make an agent good at your product · 00:39
- Rodrigues renames the workshop "Level Up Your Skills" on stage and saves the "Skill Issue" title and production lessons for his talk the next day.
- He leads Supabase's AI tooling and describes his focus as **AX**, the agent equivalent of DX — making the product as agent-friendly as possible.
- The workshop sequence is simple: write a skill, test it manually, then automate that test with an evaluation.
- The opening slide deck was itself "vibe coded" with Claude, including a dark-mode toggle, as a small demonstration of what skills enable.

### A skill is a folder with progressive disclosure · 03:11
- The required entry point is `SKILL.md`: Markdown instructions plus frontmatter with a required `name` and `description`.
- **Progressive disclosure** means the agent starts with only the metadata (an envelope), not the full body. Once loaded, the instructions can point to further files read only as needed.
- Rodrigues compares `SKILL.md` to a book's index with extra instructions, and reference files to chapters — often Markdown, kept in a reference folder.
- Reference files can link to one another, so a skill can form a graph of information. The useful property is reaching relevant detail without loading every chapter first.

### Skills and MCP are complementary · 06:25
- A skill supplies context and instructions, including workflows too long for a tool description. MCP supplies an integration through which the agent can act.
- An agent without shell access can still use a service through a remote MCP tool; that tool runs on the server, so the agent's machine needs no execution environment.
- A bundled script is different: its commands and dependencies must work on the local operating system (Linux, macOS, or Windows).
- The takeaway: use both. An integration belongs in MCP; the procedure for using it belongs in a skill.

### Define success, then write the skill · 09:08
- Testing Markdown means testing what the agent does after reading it, from a single tool call up to a full workflow.
- Because LLM output varies, exact text matching often captures less than inspecting the steps taken, tools called, and resulting state. An evaluation still starts with an input and an expected outcome.
- He adopts OpenAI's cycle: **define success**, **write the skill**, **run scenarios**, **grade behavior**, then **iterate**.
- Even a nondeterministic workflow keeps some deterministic checks: a tool either was called or was not, and a database object either has a required property or does not.
- Braintrust is introduced as a way to observe agent behavior inside controlled evaluation scenarios.

### Demo: a report that looks right but leaks data · 14:29
- The testbed is a vulnerable performance-review app with four switchable accounts: Alice (employee), Bob (head of Engineering), another manager, and Julia (HR).
- Supabase provides the backend: Postgres, authentication, storage, and edge functions (which he compares to AWS Lambda). He describes it as an open-source alternative to Firebase.
- The planned Reports page shows salary and average performance rating by department. The actual task is narrower: create a SQL view `department_stats` with headcount and average salary by department.
- Setup is short: clone the repo, install dependencies, start Supabase with `npx`, reset the database to its migrations and seed data, then run `npm run dev` on `localhost:3000/dashboard`. A prepared `.mcp.json` points Claude Code at the local Supabase MCP server.
- Claude inspects the schema, uses `apply_migration` to create the view, hits an error, retries, and tests the query. The report renders for HR as expected.
- Logged in as Bob, however, the manager can also see HR and Product data. **Functional correctness has concealed an authorization failure.**
- The migrations already enable **row-level security (RLS)** on `profiles` and `performance_reviews`. RLS filters access at the database layer instead of relying only on application code.

### Install the skill and encode the rule · 38:22
- By default, a Postgres view is accessed with the permissions of the account that created it, so it bypasses the RLS policies meant for the caller. The `security_invoker` option (Postgres 15+) makes checks use the invoking user instead.
- The prepared Supabase Security skill is a short checklist. For an existing view the focused fix is `ALTER VIEW public.department_stats SET (security_invoker = true);` — though the underlying table policies must still express the intended employee, manager, and HR boundaries.
- Rodrigues notes that starting the skill's description with an action verb, especially "use" followed by the purpose, has made Claude more likely to load it in his experiments. He does not establish why, or whether it transfers to other models.
- He installs the skill with the Vercel Labs skills CLI, choosing project installation over global and selecting Claude Code. The installer creates the skill under `.agents` and symlinks it into `.claude/skills`.
- There are three activation paths: automatic discovery from the description, an explicit request to use the named skill, or a Claude Code slash command. He calls explicit naming highly reliable in his experience.
- An audience member notes the tension with automatic discovery. Rodrigues recommends testing both **positive** scenarios (the skill should load) and **negative** ones (it should stay unused), then iterating on the description.
- Before rerunning, he resets the disposable database with `npx supabase db reset --yes`. The first attempt does not create the view; after he revises the request, the skill loads and the migration now includes `security_invoker`.
- The application check still fails: with Alice selected as an employee, department reports remain visible. He attributes this to additional vulnerabilities seeded into the exercise, and notes Supabase's database advisors as another check for exposed data.

### Automate with evals — and debug your grader · 1:00:23
- Changing one checklist item should not silently break another workflow, and checks need to travel across agent environments — so the manual comparison is not enough.
- The hardest part is defining representative scenarios and their expected behavior; prompts and expectations can live in CSV or JSON, with Braintrust or Langfuse adding observability.
- An `eval.json` holds an array of scenarios. Two grading approaches serve different needs:
  - **Deterministic assertions** check observable facts (a tool was called, a view has the required setting).
  - **LLM-as-a-judge** gives another model the output plus explicit success criteria and asks it to grade.
- The runner resets the local database, runs Claude Code headlessly, then runs a with-skill condition (SKILL.md appended to the system prompt) and a without-skill condition, writing outputs into an iteration workspace.
- Appending the text tests that the instructions are present but bypasses automatic discovery; a more realistic variant is to put the skill in `.claude/skills` in a fresh Docker environment and let Claude find it.
- The with-skill output does contain `security_invoker`, yet `grading.json` marks the check failed. Rodrigues traces it to his assertion logic (around `pg_class` and `reloptions`) and does not resolve the exact defect.
- The without-skill run also includes the flag, and the grader passes it. The two inspected outputs are effectively the same, so the displayed "zero passes, four failures" for the with-skill run is a disputed result from a faulty harness — not evidence that the skill reduces correctness.
- Two failure modes are distinct: deterministic grading code can inspect the wrong object or encode the wrong expectation, while an LLM judge can additionally produce an unreliable judgment.

### Keep production skills small and maintained · 1:12:46
- The first evaluation pipeline can stay simple: compare with-skill and without-skill conditions on the harness that will actually use the skill, ideally in a Docker container or sandbox with only the skill under test.
- Asked whether progressive disclosure means you can install skills indefinitely, Rodrigues offers no numerical limit. For local experimentation he tolerates a messy inventory; for production and CI he is stricter — include only the skills needed for the task.
- A production skill is maintained documentation with behavioral consequences: publish it alongside the repository, update it when features or workflows change, and put that requirement in `CLAUDE.md` or `AGENTS.md`.
- Periodic jobs can check whether the workflow is still valid, and load telemetry can flag skills that have gone unused for a long time.
