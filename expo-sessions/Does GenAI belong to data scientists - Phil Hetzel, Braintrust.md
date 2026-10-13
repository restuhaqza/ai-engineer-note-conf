# Does GenAI belong to data scientists?

| Field | Details |
| --- | --- |
| **Speaker** | Phil Hetzel |
| **Affiliation** | Braintrust |
| **Type** | Expo session |
| **Date** | 2026-05-25 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=NKwIX3CiRgU) |
| **Tags** | `evals` `reasoning-and-models` `rag-context-and-search` `leadership` |

## Summary

Phil Hetzel, who leads solutions engineering at Braintrust (an "agent quality"
platform), asks a deliberately provocative question: should generative AI and
agent development be handed to data scientists and ML platform teams? His answer
is no, not exclusively. He has watched many enterprises build impressive proofs
of concept and then stall before production.

Hetzel contrasts two organizational patterns. Traditional enterprises often pass
a "we need agents" mandate down to the existing ML or data science team, simply
because it already has the tooling and the word "AI" in its name. AI-native
companies rarely have that inherited boundary; small cross-functional teams stay
close to the problem the agent is meant to solve. Owning the technology, he
argues, is not the same as understanding the outcome.

His core technical point is that the foundation model is already built.
Providers like Anthropic, OpenAI and Mistral train and host the model, so
application teams mostly change behavior through prompts and context rather than
retraining. That moves the hard questions toward product judgment and
evaluation: knowing how to measure a model is different from knowing whether an
agent accomplishes its purpose.

So he makes the case for a broader team — product, application and systems
engineers to build the product; subject-matter experts to shape prompts and
judge agent traces; and data scientists for specific, high-value work such as
risk guidance, validating LLM judges, and fine-tuning open models. His closing
line: the answer is in the middle, and data science keeps a real role, just not
sole ownership.

## Key Takeaways

- **Ownership often follows the org chart, not the problem.** Enterprises hand
  agents to the existing AI team; that does not mean the team understands the
  outcome.
- **The model is already trained.** For hosted LLMs, application teams add
  value through prompts, context, evaluation and observability — not model
  training.
- **Fine-tuning is rare in his experience; prompting and context are the common
  levers.** He frames this as an observation, not a universal rule.
- **Testing skills alone don't define success.** Precision, recall and F1 were
  built for predictive models; an agent needs functional evaluation of the task.
- **Evals and observability are two halves of one loop.** Evals build confidence
  before release; observability keeps it once real users arrive.
- **Domain experts belong in the loop.** They can shape prompts and explain
  *why* a trace was good or bad — the "why" is what improves the product.
- **Data scientists add distinctive value in three places:** tempering
  overconfidence, validating LLM judges against labeled data, and
  use-case-specific fine-tuning.
- **Start from the problem, not from who owns the tool** — his answer to the
  first audience question.

## Detailed Notes

### Who can get an agent into production? · 00:00
- Hetzel leads solutions engineering at Braintrust and spent 12 years in
  consulting and systems implementation.
- Before Braintrust he led Slalom's global Databricks business, where customers
  produced plenty of generative AI proofs of concept but struggled to reach
  production.
- Braintrust's quality approach has two pillars:
  - **Evals** — tests run during experimentation to build confidence before
    release.
  - **Observability** — signals that maintain that confidence once real users
    hit the agent.
- These are complementary; evaluation does not stop at deployment.

### Inherited ownership vs. proximity to the problem · 02:54
- **Traditional enterprise:** a CEO or CIO decides the company needs agents; the
  mandate flows down to the existing ML/data science platform team, which
  already has tooling.
- **AI-native company:** built around agents from the start, with no inherited AI
  platform boundary; a small cross-functional team spans product and AI work.
- The key difference: in the AI-native case, each builder sits closer to the
  problem the agent is meant to solve.
- **Owning the relevant technology is not the same as understanding the intended
  outcome.**

### The model is already built · 05:13
- Traditional ML is a data pipeline: prepare data, train, test, avoid
  overfitting, deploy for a downstream product team.
- With a hosted LLM, that pipeline is done for you — Anthropic, OpenAI and
  Mistral expose a trained model over an endpoint.
- Their testing does not prove *your* application works; the app team still
  needs its own evals after integrating the API.
- How you change behavior also shifts:
  - Traditional ML: more data or feature engineering, then A/B tests for lift.
  - Pretrained LLM app: change prompts and supplied context, then evaluate the
    resulting behavior.
- He calls fine-tuning "pretty rare" in the work he discusses, rather than
  unnecessary in every case.

### Testing expertise still needs a product target · 07:52
- The case *for* data science: agents use models; models are often governed by
  data science, which brings neural-network knowledge, risk awareness and
  rigorous release processes.
- The counterpoint: the app team is not repeating training and cross-validation.
- The harder question is whether the team knows what to test for.
- Some groups fixate on precision, recall and F1 because those served them well
  before, but an agent has a much wider surface of behavior.
- **Knowing how to measure a model differs from knowing whether an agent
  achieves its purpose.**

### APIs, distributed systems, and domain judgment · 09:55
- LLMs can be treated as APIs; product engineers already send payloads, retrieve
  data and turn responses into user-facing behavior.
- A complex agent can also be a distributed system: a supervisor calling child
  agents on different infrastructure, which in turn call other systems.
- That is a systems engineering problem, not only a statistics problem.
- Subject-matter experts and product managers add something essential: they
  understand what success should look like and can control the prompts.
- In human annotation, a domain expert reads an agent trace and explains *why*
  it performed well or badly — that explanation is what improves the product.

### Where data science adds distinctive value · 12:00
- His point is not that data scientists should retrain; they can contribute to
  the product directly.
- Three specific contributions:
  - **Technical risk guidance.** Statistics knowledge can temper overconfidence:
    the model is predicting tokens, not "knowing" things.
  - **Validating LLM judges.** A judge is itself a prompt plus a model; check it
    against a labeled dataset with precision, recall and F1.
  - **Use-case-specific fine-tuning.** When an app needs a fine-tuned
    open-source model, this is the deep, technical work data scientists and ML
    engineers can own.
- The judge example gives familiar classification metrics a narrow, appropriate
  target: scoring the evaluator itself.

### Connect implementation to the feedback loop · 14:13
- The ideal mix: product, application and systems engineers turn domain
  requirements into working behavior.
- They also make sure the infrastructure where agents run supports a good user
  experience.
- Together with data scientists, they build the eval and observability pipelines
  that connect production back to experimentation.
- Non-technical experts stay active through human annotation, prompt development
  and context engineering.
- Proximity to the problem keeps the agent relevant to its intended use.

### Q&A: start with the problem; watch the evaluator · 15:46
- **First question:** agents are a tool, so anyone with domain expertise might
  own one — why begin with ownership at all?
  - He agrees: the agent is a product that a diverse team builds; the mistake is
    treating it as another predictive-model project and isolating it within ML.
- **Second question:** how do you keep both the evaluator and the system up to
  date, and tell which one is wrong?
  - Keep gathering production examples and adding them to the offline eval
    dataset.
  - Add grounded human judgments to check the evals against.
  - Success looks like evals agreeing more with human judgment over time — or
    diverging, which signals recalibration.
  - This checks the loop; it does not automatically attribute every error.

## Resources

- [Braintrust evaluation workflow](https://www.braintrust.dev/docs/evaluate)
- [Braintrust playgrounds](https://www.braintrust.dev/docs/evaluate/playgrounds)
- [Human review in Braintrust](https://www.braintrust.dev/docs/annotate/human-review)
- Writing and calibrating scorers
- Why Phil Hetzel joined Braintrust
