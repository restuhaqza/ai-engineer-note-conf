# GPU Cloud Deployment Without Leaving Your IDE

| Field | Details |
| --- | --- |
| **Speaker** | Audry Hsu |
| **Affiliation** | RunPod |
| **Type** | Talk |
| **Date** | 2026-06-09 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zDGHt0LB-dA) |
| **Tags** | `apis-mcp-and-protocols` `vision-and-video` `coding-and-developer-tools` `reasoning-and-models` |

## Summary

RunPod's Audry Hsu gives a tour of GPU cloud options and then a live demo of **Flash**, a Python SDK that sends one function to a cloud GPU while everything else stays on your laptop. The goal is to shrink the loop between editing an inference function and testing it.

The problem is friction. To test a small change today you commit, push to GitHub, build a Docker image, pull it onto a server, and wait for a GPU — then repeat. Hsu argues that infrastructure work like aligning CUDA and PyTorch versions, testing new GPU SKUs, and configuring scaling competes directly with model work.

In the demo she generates an image with Stable Diffusion XL Turbo, fixes the prompt, swaps in DreamShaper for a nicer result, then runs a three-stage pipeline: Qwen 3 expands the prompt, DreamShaper draws it, and Google's Nano Banana 2 composes the output with a reference photo. Every change stays inside the IDE.

She closes with pricing: on Serverless you pay for worker lifetime (startup, execution and idle), so she suggests starting with a Pod or a low worker count, then scaling up when the app needs many workers spread across data centers.

## Key Takeaways

- **Infrastructure competes with model work.** Version alignment, GPU testing and scaling config eat the time meant for models.
- **Pick a product by workload.** Pods for persistent compute, Serverless for variable load, clusters for multi-node training, Hub for pre-vetted open-source models.
- **Flash moves the boundary, not everything.** A decorated async function goes to the GPU; the main function and helpers stay local.
- **Hot reload removes the rebuild.** Editing the function repackages and pushes the change, so no manual commit, Docker build, registry pull or GPU allocation.
- **Model swaps are cheap.** Hsu switches from Stable Diffusion XL Turbo to DreamShaper to get a more artistic image with a one-block edit.
- **Real apps are pipelines.** The final demo chains Qwen 3 (prompt expansion), DreamShaper (generation) and Nano Banana 2 (photo composition).
- **The cost boundary is worker lifetime.** Serverless bills from worker startup through idle timeout, and an always-on worker runs continuously.
- **Start small, scale later.** Use one or two GPUs or a Pod while experimenting; move to Serverless when you need hundreds of workers across data centers.

## Detailed Notes

### The GPU infrastructure problem · 00:35
- RunPod is an AI cloud infrastructure company: it brings GPUs and compute, and developers bring code and models.
- An Oxford attendee in the audience already uses RunPod credits from his university for LLM training — a concrete example of why people want compute.
- Hsu lists the work that gets in the way: matching CUDA and PyTorch versions, testing new GPU SKUs, chasing hardware bugs, and configuring deployment and scaling.
- Her claim: taking that configuration away lets developers focus on training models and building apps.

### RunPod's origins and four product paths · 02:17
- Founders Zhen and Pardeep started RunPod in 2022 with spare GPUs from a failed crypto-mining venture.
- They offered free GPU access on Reddit in exchange for feedback, and Hsu says the company has been revenue-generating from the start.
- She reports about 500 developers, 30+ data centers across 10 countries, and $120M in annual recurring revenue — a company-reported milestone.
- Four ways to build:
  - **Pods** — a persistent/VM environment with a reserved GPU; rent on demand, pay by the second, tear down when done.
  - **Serverless** — autoscaling workers for variable request frequency and load; scaling down avoids paying for idle time.
  - **Clusters** — multi-node compute for training.
  - **Hub** — deploy pre-vetted open-source repos such as ComfyUI, Stable Diffusion and vLLM.

### Keep the iteration loop local · 05:43
- The old loop: commit, push to GitHub, build a Docker image, pull it from a registry, load it on a server, allocate a GPU, test, repeat.
- **Flash** is a Python SDK meant to remove those steps so a function can run on a cloud GPU from the local development environment.
- The unit is an ordinary async Python function plus a Flash endpoint decorator. Flash packages and deploys that function; the surrounding main and helper functions stay local.
- File reload repackages and pushes changes immediately, so testing again does not require a manual rebuild or upload.
- Note: the decorator and `flash run` shown on stage reflect that demo; today's repository documents `@Endpoint` with `flash dev` and `flash deploy`.

### Send an image request through a local server · 07:42
- The first GPU function loads PyTorch and a pretrained **Stable Diffusion XL Turbo** model, generates an image, saves it, and returns it Base64-encoded (text representing the image bytes).
- Hsu runs `flash run`, which starts a local FastAPI development server.
- A local helper sends a POST request to the dev endpoint and decodes the returned image so it can be displayed.
- The audience picks the subject: cats flying through a cloudy London sky.
- Live debugging follows around the URL, localhost and HTTP; once it connects, the server reports the job has started and queued.
- Useful distinction: whether the request reached the service is separate from whether it carried the intended prompt.

### Configure workers, fix the prompt, swap the model · 11:14
- The endpoint decorator carries the endpoint name, GPU family and worker settings. The demo uses a pool that maps to H100 80GB cards, up to five workers, with one active worker kept running.
- The first output is a dragon: the service ran, but the prompt was not passed as a flag. She adds the flag and also points out the idle-timeout setting.
- The corrected request returns cats, but the abstract look is not what she wants.
- She comments out the model and substitutes **DreamShaper**, a Stable Diffusion 1.5 fine-tune she describes as better for artistic and illustrative output, set to 25 inference steps.
- She resends the same request and prefers the result — with no commit, Docker rebuild, upload or GPU allocation in between. This is a working comparison, not a timed benchmark.

### Chain three models into one pipeline · 14:34
- A single model call is only part of an application; the harder part is the orchestration around it.
- Prepared orchestration code runs three stages:
  1. Call **Qwen 3** on an already-hosted public endpoint to generate richer image prompts.
  2. Send those prompts to DreamShaper on the Flash-managed endpoint.
  3. Pass the images through **Nano Banana 2** (Google's Gemini 3.1 Flash Image) for photo composition.
- The intended subjects are RunPod's founders, shaped by the audience into two men with glasses walking in London on a cloudy day, with a close-up of their faces.
- Hsu requests three images and leaves the pipeline running while she answers a pricing question.

### Know what stays running · 16:55
- Asked about pricing, Hsu first describes charging for how long each request runs, then checks the web console.
- She sees about five or six provisioned workers and three running for three requested photos, and points to worker uptime as the billed quantity.
- **The real cost boundary is worker lifetime, not just inference.** Current docs bill from worker startup through full stop, including initialization, execution and idle timeout; always-on workers run continuously.
- Serverless costs a little more than Pods because it adds scaling.
- Her advice: start with a low worker count or a Pod when you need one or two GPUs; use Serverless when you need hundreds of workers distributed for availability.

### The finished images · 18:40
- The final view separates the intermediate image from the composed photo.
- On the left, DreamShaper's output is based on Qwen 3's expanded prompt, which adds details like thoughtful expressions, soft-focus clouds, and a muted gray-and-blue palette.
- On the right, the composed photo combines the generated scene with a separate reference photograph supplied to the composition stage.
- The last pair shows generated and composed images under golden trees in Hyde Park.
- Hsu closes by noting developers can also bring their own private models, and that the whole workflow still begins in the local development environment.

## Resources

- [RunPod Flash](https://github.com/runpod/flash) — Python SDK for defining remote GPU functions, dependencies and deployment configuration.
- [Serverless billing and worker costs](https://docs.runpod.io/serverless/pricing) — active and flex workers, startup and idle charges.
- [RunPod GPU types and pools](https://docs.runpod.io/references/gpu-types) — GPU identifiers, memory capacities and pool mappings.
- [Stable Diffusion XL Turbo](https://huggingface.co/stabilityai/sdxl-turbo) — Stability AI's model card and files.
- [DreamShaper](https://huggingface.co/Lykon/DreamShaper) — Lykon's image-generation model repository.
- [Introducing Qwen3](https://qwenlm.github.io/blog/qwen3/) — the Qwen team's introduction to the models.
- [Introducing Nano Banana 2](https://blog.google/innovation-and-ai/technology/ai/nano-banana-2/) — Google's Gemini 3.1 Flash Image announcement.
