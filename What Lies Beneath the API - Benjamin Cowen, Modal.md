# What Lies Beneath the API

| Field | Details |
| --- | --- |
| **Speaker** | Benjamin Cowen |
| **Affiliation** | Modal |
| **Type** | Talk |
| **Date** | 2026-06-02 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=HvZXAOZ3iv8) |
| **Tags** | `infrastructure-and-deployment` `safety-and-governance` `apis-mcp-and-protocols` |

## Summary

Most products start on a frontier model API. That is the fastest way to build something useful, but the API is mostly customizable through prompting. As a product matures, it may need a different cost structure, tighter latency or throughput, or quality that matches its own business logic — things a prompt cannot deliver.

Benjamin Cowen, an engineer at Modal, argues that fine-tuning becomes worth considering once a product turns into its own narrow domain. The problem is that training has been a big jump: you had to run and isolate your own cluster and put engineers on infrastructure work. A newer middle ground is serverless compute that makes custom training and serving far less of a burden.

His central point is not "go train now." It is that the moment to train is probably closer than people think — maybe six months or a year — and you should prepare for it. If you already have an agent harness, evaluation data, and records of what worked, you may already own most of what training needs.

## Key Takeaways

- **Frontier APIs trade control for speed.** They are great for getting started, but beyond prompt engineering you cannot shape cost, latency, or behavior much.
- **As products specialize, custom models pay off.** Intercom reportedly beat its API baseline at one-fifth the cost; Pinterest described orders-of-magnitude savings (Cowen gives no exact figure).
- **Business-specific performance is the real goal.** Frontier labs optimize a model to win at everything; your business only needs it to win at your service.
- **Watch for signals, not deadlines.** Unit economics going negative, unmet latency/throughput needs, or evaluations that have plateaued all point toward customization.
- **Data and mature evaluations are prerequisites.** Without them, Cowen advises against training — poor inputs stay poor after a training run.
- **You may already have the ingredients.** Your agent harness gives a model a place to practice; your evals and logs can become training data.
- **Serverless helps training too.** It scales hyperparameter search and RL rollouts in parallel, then autoscales serving of the final model.
- **Prepare before it is urgent.** Decide how you will recognize the moment to train, then collect the data and build the evals to act on it.

## Detailed Notes

### When does your application become a custom domain? · 00:25
- A general-purpose model gets a product started quickly, but a maturing product often needs narrower performance or a different cost shape.
- Cowen's framing question: when do you stop consuming a model and start shaping one around the service you sell?
- Modal, where he works, is a serverless platform offering functions and hardened sandboxes for code execution.
- Its workloads range from physics and quantum chemistry to voice, LLMs, and agents — with large-scale reinforcement learning growing fast.

### From caveman mode to infrastructure ownership · 01:50
- Frontier APIs let you build very fast without running model infrastructure; prompting is the main way to customize.
- The "caveman mode" example: ask the model to speak tersely and you spend fewer tokens. That saves money, but it does not guarantee the economics hold if you grow a hundredfold.
- Enterprise contracts can add specific latency, throughput, or quality targets that reach past prompt wording.
- Getting that control traditionally meant moving to the opposite end: a big cluster with training resources isolated from serving, plus infrastructure engineers.

### Optimize for the service you actually provide · 04:10
- Cowen reports that Intercom's fine-tuned model beat its frontier API baseline at one-fifth the cost (he gives no evaluation or workload details, so treat it as an attributed example).
- Pinterest is cited for orders-of-magnitude savings, but without an exact multiplier and as a cost comparison, not a quality jump.
- Decagon's framing is the useful part: frontier labs want models that win at everything; your business only needs to win at its own service.
- Open-source training libraries give algorithm control without the cluster overhead, and managed compute keeps iteration fast — the two properties Cowen wants to preserve together.

### Separate the reason to train from readiness to train · 05:47
- Cowen expects a differentiated product to eventually develop domain-specific requirements. That is a product judgment, not a fixed deadline.
- Signals that customization is worth investigating:
  - **Unit economics:** API spending exceeds what customers pay, even after trimming output.
  - **Latency and throughput:** the service needs performance the current endpoint cannot give.
  - **Evaluation progress:** results have plateaued, so more prompting is unlikely to close the gap.
- These signals show a problem worth solving; they do not prove fine-tuning is the fix.
- **Data and mature evals are prerequisites.** If you are not collecting data or cannot evaluate results reliably, wait — garbage in, garbage out.
- Much of the needed material can come from existing work: an agent harness for practice via reinforcement learning, plus evals and logs as training data.

### Start small, then fan out training experiments · 08:21
- A supervised fine-tuning setup is demoed in roughly 300 lines of Python — that is the example size, not the whole effort of curating data and operating a model.
- With curated data and a serverless account, the setup is a practical starting point; Cowen points to Modal's examples repository.
- Serverless is useful beyond inference: **hyperparameter search** can run each configuration in its own on-demand container.
- Instead of treating every cluster minute as precious, you can launch many trials and stop the ones that stop looking promising.
- A small pruning snippet marks low-scoring trials as `stop_requested`; the orchestration layer must then actually terminate those containers to stop the compute.
- Cowen likens this search-and-prune loop to a meta evolutionary algorithm: explore, discard weak candidates, keep resources on the rest.

### Reinforcement learning creates a parallel execution workload · 09:47
- Reinforcement learning is old, but today's libraries make it accessible to engineers who never specialized in it — another roughly 300-line example.
- A model needs repeated chances to practice; these interactions and evaluations are **rollouts**, and many can run independently.
- Modal exposes unified APIs for sandboxes and GPU containers/clusters, so the practice environments and the model compute sit on one programming surface; the slide shows a GRPO example.
- Customers scaled to 50,000–100,000 sandboxes for RL in the prior quarter (Cowen does not specify concurrency or duration) — the point is the large parallel demand rollouts create.

### Serve the model — and prepare before you need to train · 10:55
- Training leaves out serving: the inference slide shows a vLLM example in about 200 lines of code.
- Cowen also names SGLang, Triton Inference Server, and custom Python inference as options. This is the serving role hidden behind a frontier API; he does not claim to know whether frontier providers use vLLM.
- On a serverless platform, the serving workload can autoscale with traffic.
- This closes the lifecycle: train the model, put it behind an inference service, scale it to demand.
- Having an accessible path through the stack is not a reason to train immediately.
- The practical horizon may be six months or a year. Decide how you will recognize the moment, and build the data and evals that let you act when cost, latency, or quality makes it urgent.

## Resources

- [Modal examples repository](https://github.com/modal-labs/modal-examples) — Python examples for running applications on Modal.
