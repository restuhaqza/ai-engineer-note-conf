# CI/CD Is Dead, Agents Need Continuous Compute and Computers

| Field | Details |
| --- | --- |
| **Speaker** | Hugo Santos, Madison Faulkner |
| **Affiliation** | Namespace |
| **Type** | Talk |
| **Date** | 2026-05-13 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=VktrqzQgytY) |
| **Tags** | `infrastructure-and-deployment` `coding-and-developer-tools` `agent-engineering` `leadership` `developer-workflows-and-testing` |

## Summary

CI/CD (continuous integration and continuous delivery — the automated build, test, and deploy pipeline) was designed around human developers, who submit only a few changes a week and can wait for review. Madison Faulkner (NEA) and Hugo Santos (Namespace) argue that this model breaks when coding agents generate many changes at once. Verification still takes roughly the same time, so review and merging — not code generation — become the bottleneck.

Their proposal is "continuous compute": faster, always-available build-and-test infrastructure that plugs into existing systems such as GitHub Actions. The first step is incremental acceleration. Later steps rethink how work is orchestrated, validated, and merged.

The talk sketches a future loop where a written intent or spec feeds a coding agent, the agent checks out a known commit and runs the repository's own build and tests, and other agents handle external checks like security and API conformance. Humans approve outcomes rather than every line. The claim is that CI does not vanish — its checks simply become continuous. The architecture is presented as a proposal and a forecast, and Santos notes his team already works this way.

## Key Takeaways

- CI/CD pipelines and PR (pull request — a proposed code change for review) review were built for human-paced development.
- Agents produce many short-lived branches; verification time stays roughly the same, so review and merging become the constraint.
- More runners alone won't help: thousands of concurrent branches pulling the same code make reconciliation the hard part.
- Start incrementally by speeding up build/test/deploy under existing workflows like GitHub Actions; in the proposed design the cache becomes an orchestration layer.
- A pull request is a human-review unit; its checks (regressions, provenance, conflicts, permissions) still matter even if PRs stop organizing the process.
- The emerging loop: write intent/spec → agent implements from a known commit → internal build and test → human says "continue".
- Next, other agents run external validation (security, API conformance) inside a stateful environment that remembers where it left off.
- Completed work waits in a "pre-merge" queue that reconciles concurrent edits before a human approves the intent and result — not every commit.

## Detailed Notes

### Why agent-scale code breaks CI/CD · 00:18
- The talk asks what happens to build, test, and deploy when the "developers" producing changes are agents.
- Early agentic setups treated one LLM as a single engine; as agents take on separate jobs, the architecture starts to look more like microservices.
- That work enters an already fragmented lifecycle: planning tools, IDEs, autonomous engineering systems, and traditional DevOps.
- Speeding up one part of the lifecycle does not automatically speed up the handoffs between parts.

### More branches, the same verification bottleneck · 02:20
- Normal flow: a human submits a diff; colleagues review the pull request; GitHub Actions builds, tests, and deploys; a failed test sends the change back.
- Faulkner's human baseline is one or two diffs a week. Swap in agents across N repos and verification still takes about the same time unless review changes too.
- Review bots can help, but they add another moving part.
- With agents there are thousands of short-lived branches, local caches are harder to keep warm, and merging all the versions becomes "really impossible."
- She shows a GitHub activity chart with a sharp recent spike (commits, lines added, lines deleted) but gives no numeric size.

### Start with acceleration, then change orchestration · 04:01
- Step one is incremental: make the underlying build/test/deploy infrastructure faster while keeping existing workflows, including GitHub Actions.
- In the proposed design the cache becomes an orchestration layer; hardware and software are co-designed so work reaches the compute that can run it efficiently.
- Responsibilities add up in sequence: shape intake (ingress shaping and rate limiting), route work using cached state, then manage agent identity and retries at scale.
- Faulkner cites a Mitchell Hashimoto proposal for fixing GitHub — in her account it includes shutting down Copilot, modernizing GitHub for the cloud era, and enabling inference at scale.
- Her read: services must serve agent users first, and code storage should be more agent-friendly.

### Human latency used to hide machine latency · 06:11
- Santos draws on Namespace's work with fal, Zed, Ramp, and others.
- Roughly six months earlier, people wrote code, opened PRs, and waited for validation; slow machines were tolerable because human writing and review time hid the latency.
- Once code generation is cheap and work is continuous, validation has to move into the inner loop.
- Until now the human was effectively the agent: hold an objective, then loop through formatting fixes, failing tests, reviewer requests, and merge-queue rebases.
- At human speed the "opportunity to merge" can be long because few changes are in flight; as the rate rises, it becomes the central constraint.

### Keep CI's checks, rethink the unit of work · 08:43
- A pull request is a unit of work designed for human review: it assumes delayed feedback and discrete handoffs.
- The checks CI performs survive any redesign: does the change regress anything, does it build from a known source, do concurrent changes conflict, is the change allowed?
- With reviewers overwhelmed, merging starts to look like a high-performance database problem: a single ledger and a lock needed to commit.
- The analogy is about ordering changes into shared history; it is not a claim that GitHub literally holds a database lock.

### From a written intent to a validated change · 10:00
- This workflow is already used by Santos's team and other early adopters, without a PR as the organizing unit.
- Start with intent and plan: write the goal as a spec (a Linear ticket or Slack message works), then feed it to a harness such as Claude Code, Amp, Cursor, or Factory. Namespace often uses Amp.
- The harness checks out the repo from a well-known commit, then uses the repository's own build and test assets for internal validation.
- The agent reports back; the human approves or replies something like "continue."
- Finished work enters the merge queue and then the Git repository, which acts as the ledger. Human external validation is still on the critical path.

### Validators inside a persistent working environment · 11:59
- Santos expects the next shift in weeks to months, not years. Faster inference makes slow validation increasingly costly because the whole loop waits for it.
- He gives 15-minute and 45-minute test runs as examples of unacceptable delays, not measured Namespace results.
- Other agents take over external validation: a security-focused LLM returns findings, and an API-conformance LLM checks the implementation against the contract.
- The main harness folds their feedback back into the code and iterates.
- Persistent state (memory plus a retained working environment) makes rapid iteration possible; the loop also absorbs outside signals like a changed plan or another contributor's commit.

### Reconcile parallel work, then ask a human · 13:47
- Completed changes enter a proposed "pre-merge" queue — work that could have merged waits while the system reconciles concurrent edits.
- Many agents may have touched the same code, so their finished results must become a sequence that fits the repository ledger. Santos frames this as serializability.
- Human approval comes after reconciliation and focuses on intent and result, not every diff — for example, a video of the feature working or a security evaluator's output.
- The approval unit can span several commits or features, grouped semantically into something a person can assess.
- He reports his team's work volume equivalent to former PRs is four times higher. That is a work-volume observation, not a measured productivity gain, and no reviewer could inspect every unit.
- Further out he imagines a "multiverse": agents apply the same plan to several candidate commits at once. That needs very fast inner loops and more capacity, so avoiding unnecessary work and keeping state matters.
- CI does not disappear: validation, provenance (starting from a vetted revision), and governance become continuous, with more of the governance living in the harness.

## Resources

- **Namespace: CI/CD is Dead, Agents Need Computers** — article by Madison Faulkner and colleagues. *(related)*
- **Namespace Documentation** — guides for agent development environments and GitHub Actions runners. *(update)*
- **Managing a merge queue** — GitHub's guide to validating queued changes together. *(related)*
- **Introducing Namespace** — Hugo Santos's 2022 account of Namespace's origins. *(related)*
