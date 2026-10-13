# Beyond Code Coverage: Testing What Users Can Do with Playwright

| Field | Details |
| --- | --- |
| **Speaker** | Marlene Mhangami |
| **Affiliation** | Microsoft |
| **Type** | Talk |
| **Date** | 2026-05-16 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=FWEInOtngmM) |
| **Tags** | `coding-and-developer-tools` `developer-workflows-and-testing` `apis-mcp-and-protocols` `evals` |

## Summary

Marlene Mhangami, a developer advocate working across Microsoft and GitHub, opens with an uncomfortable question: if AI is writing far more code, are developers actually getting more useful work done? She cites GitHub's 2025 Octoverse report (about one billion commits) and a 2026 post from GitHub's COO estimating roughly 275 million commits per week. She labels the 2026 figure a projection from a weekly rate, not an official annual total.

Her argument is that output volume is a weak productivity signal. She revisits Stanford research on 120,000 developers showing that AI magnifies whatever a codebase already is: clean code gets a lift, neglected code accumulates entropy. In one cited company case, pull-request output rose while quality fell and rework grew, leaving effective output up only about 1%.

She then recommends red-green test-driven development (TDD — write a failing test, make it pass, then improve the code) as a structure that separates *what should happen* from *how it is implemented*. Because agents can generate both the failing tests and the code that passes them, the steps people say make TDD slow get fast, leaving humans more time for review and refactoring.

Finally she demonstrates browser-based functional testing with Playwright, Microsoft's open-source tool for automating real browser interactions, wired into GitHub Copilot through the Playwright MCP server. Using a toy-store story, the agent turns a product request into failing tests, makes them pass, and drives a browser to prove search and filters work.

## Key Takeaways

- **Code volume is not productivity.** More commits and PRs can hide more rework; the metric to watch is useful output.
- **AI amplifies the codebase it lands in.** Clean, modular code with good tests tends to improve; messy code tends to get messier faster.
- **Test behavior, not internals.** A test tied to a private method (like `calculate`) breaks on a rename even when the user-visible result is still correct.
- **Beware self-affirming tests.** An agent can generate tests that agree with its own implementation, so coverage looks healthy while the system does the wrong thing.
- **Red-green TDD still fits agentic work.** Agents speed up the failing-test and pass-the-test steps; humans concentrate on refactoring.
- **Playwright tests describe real user actions.** A browser test navigates to a page and uses the same controls a customer uses.
- **Connect agents to the browser one of three ways.** Playwright MCP server, a CLI tool, or Playwright Test Agents (planner, generator, healer).
- **Commit before the agent edits.** A checkpoint matters because an agent may not recall earlier state; use headless runs and attach screenshots to PRs.

## Detailed Notes

### More commits, but more productivity? · 00:17
- Mhangami introduces herself as a senior developer advocate in Microsoft's Core AI group, working with both Microsoft and GitHub.
- GitHub's 2025 Octoverse report covered Sept 2024–Aug 2025 and recorded about a billion commits — its most active period yet.
- For 2026 she cites a GitHub COO post of roughly 275 million commits per week; annualized, that is about 14 billion, or ~14× the prior report. She flags it as a projection.
- Some of that growth is agent-assisted: Claude and Copilot leave coauthor signatures, Codex does not, and commit wording can hint at AI involvement. Attribution data was not released.
- Her framing question: does all this extra code actually mean developers accomplish more?

### Rework can consume the gains · 02:30
- She revisits a prior AI Engineer talk built on Stanford research covering 120,000 developers.
- Key finding: how you use AI matters more than whether you use it — clean codebases amplify gains, unchecked generation amplifies entropy.
- In the cited company case, PR output increased, quality dropped, and rework rose; effective output rose only about 1%. (The talk does not define how that was measured.)
- Her practical response: treat test coverage, type coverage, documentation, and modularity as part of the workflow, standardized across teams.
- Keeping the codebase understandable is how you realize AI's value, not a separate cleanup project afterward.

### Separate behavior from implementation quality · 04:41
- Test-driven development offers a ready structure. She points to Simon Willison's "red/green TDD" as an example used with coding agents.
- **Red:** start from a feature request and write a test that fails because the feature does not exist.
- **Green:** write just enough code to pass, prioritizing speed over polish (the old copy-from-Stack-Overflow move).
- **Refactor:** improve the passing code's quality and structure.
- The green phase delivers working behavior; refactor gives code quality its own explicit slot.

### A green suite can still test the wrong thing · 06:07
- She notes DHH's 2014 essay "TDD is dead. Long live testing," which criticized test-first dogma and over-reliance on isolated unit tests.
- The concern here: a suite can pile up coverage without really testing the system.
- Example: an order price with a discount. A test that calls an internal `calculate` method breaks if you rename the method, even when customers still get the right price.
- Tests aimed at observable behavior (the final price) or a stable public contract (an API or exported module) survive internal refactors.
- AI adds a new failure mode: **self-affirming tests** that match the implementation an agent just wrote instead of checking the requirement. Coverage can look healthy while the behavior is wrong.
- The requirement must supply the expectation; the implementation cannot be its only source. She recommends Ian Cooper's talk "TDD: Where Did It All Go Wrong?" for more.

### Express the requirement as a browser interaction · 08:09
- Playwright is Microsoft's open-source framework for automating browser interactions and testing an app end to end; it supports Python, TypeScript, and C#.
- Instead of calling an internal method, a browser test opens a page and uses the same controls a customer does.
- The example opens a toys page, finds the search field by placeholder, types "Furby", and asserts the result is visible.
- Tests can run headed (browser shown) or headless (no visible window).
- With agents typing more code, she suggests they generate the failing behavioral tests and the passing code, leaving humans to review and refactor — accelerating red and green and giving refactor the most human attention.

### Connect the coding agent to the browser · 10:12
- Three integration options: the Playwright MCP server, a CLI tool, and Playwright Test Agents. (MCP is a standard way for AI assistants to call external tools.)
- Playwright Test Agents split testing into roles: a **planner** decides which tests to run, a **generator** writes them, and a **healer** fixes them.
- Setup installs three agent instruction files (`agent.md`) so each role gets specialized guidance instead of one do-everything prompt.

### Turn a request into failing tests, then watch them pass · 11:44
- Scenario: a developer at "Tailspin Toys" gets a request from the search product team for a search bar plus category and price filters.
- She uses GitHub Copilot CLI and Microsoft Work IQ (a skill that connects the agent to Microsoft 365 apps like Outlook) to bring the email's requirements into the terminal.
- A key shift: a **feature request** triggers a test, not the addition of a class method. Planning starts from requested behavior.
- The next prompt asks Copilot to build the features with red-green TDD, starting from failing Playwright tests. For the demo she tells it not to commit.
- The agent first examines the repository, then writes tests (the Playwright MCP server is already installed).
- Because generation takes time, she switches tabs to an earlier run: the agent ran the tests, the browser opened the right page, found "Furby" and "Simon", exercised category filters, then checked a price range. All tests passed.

### Make results reviewable and changes recoverable · 16:09
- Attach test screenshots to the pull request so reviewers see the app alongside the code. Playwright does not capture screenshots automatically by default; capture must be configured or requested.
- Keep the visible-browser mode for walkthroughs and use headless mode for routine runs.
- Commit before asking an agent to fix tests or change code, so you have a recoverable checkpoint — an agent may not reliably recall the prior state. (This differs from the demo's explicit no-commit instruction.)
- Prefer one test per feature, keeping tests tied to delivered capabilities.
- She notes she forgot to share the demo's GitHub repository link.

### Stateful apps, screen sizes, and native apps · 17:38
- A questioner who already uses Playwright with Storybook (a component workshop tool) asks about complex apps and admin panels with heavy state.
- Mhangami suggests Playwright Agents for their built-in guidance on state, and testing APIs directly where an API exists. She does not demonstrate a fixture or state-setup strategy.
- She confirms Playwright can check mobile versus desktop sizes, though no viewport configuration is shown.
- A final question about Mac and iPhone apps: her answer scopes this workflow to browser-based testing.

## Resources

- Red/green TDD (Simon Willison) — https://simonwillison.net/guides/agentic-engineering-patterns/red-green-tdd/
- DHH, "TDD is dead. Long live testing." — https://dhh.dk/2014/tdd-is-dead-long-live-testing.html
- Ian Cooper, "TDD: Where Did It All Go Wrong?" — https://www.infoq.com/presentations/tdd-original/
- Playwright Test Agents — https://playwright.dev/docs/test-agents
- Playwright MCP — https://github.com/microsoft/playwright-mcp
- Microsoft Work IQ — https://github.com/microsoft/work-iq
- Playwright screenshot and trace options — https://playwright.dev/docs/test-use-options
