# Under 5 minutes to a deployed LLM endpoint

| Field | Details |
| --- | --- |
| **Speaker** | Audry Hsu |
| **Affiliation** | RunPod |
| **Type** | Talk |
| **Date** | 2026-06-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ILdE7FaAjVA) |
| **Tags** | `infrastructure-and-deployment` `coding-and-developer-tools` `apis-mcp-and-protocols` |

## Summary

Audry Hsu (RunPod, a GPU cloud provider) gives a short, hands-on intro to
deploying a language model. Her framing is simple: developers bring their code
and models, and the platform supplies the GPUs and manages the infrastructure
around them — the same kind of relief that moving to AWS or Google Cloud gave
teams who once ran on-premises servers.

She sketches RunPod's options — Pods for sandbox development, Serverless for
bursty workloads, Clusters for heavy training, and the Hub for discovering and
deploying preconfigured repositories — then does a live console walkthrough. She
picks a vetted vLLM (a popular LLM serving engine) listing from the Hub,
configures a Qwen model, sets worker capacity, and deploys an HTTP endpoint.

The demo makes one subtle point clearly: the wait a user sees is not the same as
model speed. A request sat in the queue for about 41 seconds while a worker
cold-started, but actual inference took only about 1.5 seconds. She estimates the
whole deployment from a Hub listing took under five minutes.

## Key Takeaways

- **Divide the work.** Developers bring the model and code; the platform brings
  GPUs and manages them.
- **Four ways to run.** Pods = sandbox dev; Serverless = bursty/batch with
  autoscaling; Clusters = multi-node training; Hub = a catalog to deploy from.
- **Serverless is best for real-time inference.** Workers scale and can spin
  down, so you do not have to predict all compute demand ahead of time.
- **Two scaling controls.** Maximum workers set the scale-out ceiling; always-on
  workers keep models downloaded and ready.
- **A Hub listing is a repo.** The vLLM listing bundles a Dockerfile, defaults,
  and setup instructions you can inspect before deploying.
- **Cold starts cost time, not model speed.** One request waited ~41s in queue
  but executed in ~1.5s; queue delay and execution time are different measures.
- **Config is just CLI flags.** The options you set become flags passed to
  `vllm serve`.
- **Know the billing detail.** Current docs charge workers from startup through
  full stop, including initialization and idle timeout — not only while serving.

## Detailed Notes

### Who manages the GPUs? · 00:27
- The setup: you have a private model or an open-source one from Hugging Face.
  RunPod's division of labor is "you bring your code, we bring the rest."
- Managing servers takes time away from building. Hsu compares GPU infrastructure
  to the earlier shift from on-premises servers to AWS or Google Cloud — each
  layer of abstraction removes work developers used to handle.
- She describes GPU access at the time as slow and opaque amid a global supply
  crunch, comparing demand to pandemic-era toilet-paper stockpiling.
- She expects recovery as customers get better at estimating their compute needs.
  The practical aim is keeping builders focused on value-creating applications.

### A platform shaped by GPU users · 02:19
- Hsu traces RunPod to founders Zhen and Pardeep, who had GPU rigs in a basement
  in 2022 after failed crypto mining.
- They prototyped the platform's foundations and posted on Reddit offering free
  GPU access in exchange for feedback. That feedback loop remains central.
- Hsu promises engagement, not perfection: the company stays active on Reddit and
  Discord with its users.
- At the time of the talk she reports more than 500,000 developers and 30+ data
  centers worldwide, including Europe and the EU.
- She also reports $120 million in annual recurring revenue — a
  company-announced milestone, not realized annual revenue.
- Her customer examples include AI cloud-native companies; the shared need is
  flexible, reliable GPU infrastructure.

### Choose the compute environment · 04:12
- **Pods:** the sandbox development environment. RunPod spins up a container,
  allocates GPUs, and you bring Dockerfiles and code.
- **Serverless:** the autoscaling product for bursty or batch workloads. Workers
  spin down when idle, so you do not pay for idle compute the way an always-on
  container charges.
- **Clusters:** for heavy training — multi-node clusters with high-speed
  networking.
- **Hub:** a central catalog of AI repositories, preconfigured and pre-vetted by
  RunPod and community contributors. Listings can be forked, watched, starred,
  and deployed. It is a starting point, not another compute environment.
- Serverless is presented as best for real-time inference. Teams use it because
  they do not need to preempt and predict how much compute they need.
- Two controls address capacity and readiness: **max workers** bound scale-out
  (and there are spending caps), while **always-on workers** keep models already
  downloaded so requests need not wait for that preparation.
- The essay notes a current billing clarification: workers accrue compute charges
  from startup through full stop, including initialization and idle timeout, and
  storage can also incur charges. That is present-day guidance, not a
  reconstruction of the demo's invoice.

### Open the vLLM listing · 06:30
- The walkthrough uses the console so the steps are visible. Hsu mentions CLI
  support and agent skills, but does not demonstrate those here.
- In the Hub she selects the vetted **vLLM** listing as the starting point for
  serving a model.
- Opening the underlying GitHub repository shows what the listing packages:
  setup instructions, a preconfigured Dockerfile, and defaults.
- Depending on the listing, environment variables let you customize behavior. The
  template removes setup work without hiding that a repo and container config sit
  underneath.
- After inspecting these pieces, she clicks **Deploy**.

### Configure Qwen, then create the endpoint · 08:28
- She selects a **Qwen** model to download from Hugging Face. The exact Qwen
  identifier is not specified in the walkthrough.
- She expands the advanced options and increases **maximum model length** for the
  serving context window, then leaves the rest at defaults.
- **Max LoRAs** (low-rank adapters, small add-on weights) is another exposed
  option.
- She explains that these configuration options become flags passed to
  `vllm serve`.
- She creates the Serverless endpoint.
- The essay cautions that raising the serving limit does not by itself extend the
  model's supported context: the model must support the requested length, and the
  GPU must have enough memory.

### Set worker capacity while initialization runs · 09:25
- Creating the endpoint starts worker initialization; a fresh model is not ready
  to serve immediately.
- While that runs, she inspects deployment settings. The shown hardware
  preference is H100 GPUs with A100 GPUs as backup.
- She describes the displayed price as a fraction of a cent per second and
  explains the cost in terms of workers running and handling requests. These are
  the demo's choices, not a claim that every endpoint uses that hardware or rate.
- In the shown configuration, maximum workers can be raised to 15, setting the
  scale-out ceiling.
- A separate **active workers** setting keeps containers always on instead of
  letting them spin down.
- She saves the settings and turns to the endpoint's request interface.

### Follow a request from the queue to completion · 10:27
- The deployment provides an HTTP API endpoint that an application or customer
  can send requests to.
- She uses the console's **Run** action to submit several requests, including a
  question about how Big Ben got its name. The requests queue while she checks
  worker readiness.
- In the worker view, **initializing** workers are creating containers and
  downloading the model. **Running** workers have finished that preparation and
  can pick up queued requests.
- The telemetry view offers request counts, execution time, and delay time,
  though it is initially blank. Those are separate measurements because waiting
  for capacity and executing inference are different parts of the path.
- A request then completes. Hsu reports about 41 seconds in the queue and about
  1.5 seconds of execution time for one returned request.
- She attributes the longer first wait to cold-start work — downloading the model
  and initializing the first container — and expects later requests to wait less.
  That is an expectation, not a measured latency distribution; this is one result
  from a newly initialized endpoint.
- **Queue delay is distinct from model execution time.** Calling this a
  1.5-second response would hide most of the observed wait.
- She estimates getting the endpoint deployed from the Hub listing took under
  five minutes. That establishes the scope of the demo — a preconfigured listing,
  a chosen model, worker initialization, and a completed request — not a
  universal deployment guarantee or an evaluation of the answer.

### From a Hub listing to remote Python functions · 12:22
- Hsu closes by inviting questions and announcing a later session on the
  **RunPod Flash Python SDK** at 4:00.
- That planned workflow moves entirely into the terminal: define code locally,
  run it as a remote function on a GPU, and eventually deploy it as a
  production-ready endpoint.
- That extends the workflow from deploying an existing Hub template to deploying
  your own functions. It is a preview here — no SDK demonstration in this
  recording, and no substantive audience Q&A follows.

## Resources

- [Deploy repositories through Runpod Hub](https://docs.runpod.io/hub/overview) — discover preconfigured AI repos and review hardware requirements
- Runpod vLLM worker — worker template with Docker deployment instructions and serving config
- Configure Runpod vLLM workers — reference for model selection, context length, and LoRA adapters
- [Serverless billing and worker costs](https://docs.runpod.io/serverless/pricing) — compute charges during startup, execution, and idle timeout
- [Runpod Flash Python SDK](https://github.com/runpod/flash) — define remote GPU functions in Python and deploy to Serverless
- Runpod's founding story — the basement GPU experiments and Reddit feedback that shaped the platform
- [Runpod surpasses $120M in ARR](https://www.runpod.io/press/runpod-ai-cloud-surpasses-120m-in-arr) — the company-announced milestone cited
