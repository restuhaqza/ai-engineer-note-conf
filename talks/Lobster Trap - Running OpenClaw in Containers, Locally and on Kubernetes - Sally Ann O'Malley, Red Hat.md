# Lobster Trap: Running OpenClaw in Containers, Locally and on Kubernetes

| Field | Details |
| --- | --- |
| **Speaker** | Sally Ann O'Malley |
| **Affiliation** | Red Hat |
| **Type** | Talk |
| **Date** | 2026-05-22 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=F1DYkY1BlfM) |
| **Tags** | `infrastructure-and-deployment` `apis-mcp-and-protocols` `agent-engineering` |

## Summary

When an agent looks useful enough to install but colleagues call it a security nightmare, what do you do? Sally Ann O'Malley, a Red Hat engineer with a decade of container and Linux security work, answers with familiar operations: run the agent in a portable container with explicit boundaries for credentials, host access, and state.

She frames a production agent as a persistent service, not a throwaway chat. Her personal agent, a "Forever Claw," comes with its own sub-agents, backed up nightly, and moved between laptop, Podman, Kubernetes, and OpenShift.

The talk covers the practical building blocks: mounting an agent directory of tools, skills, and MCP (a standard way for agents to call tools) servers; keeping API keys in Podman or Kubernetes secrets with an extra layer of indirection; and storing runtime state in volumes so a container can be replaced without losing continuity.

A live demo shows her browser-based installer spinning up local instances and then connecting to agents already running in a cluster.

## Key Takeaways

- **Containers add explicit boundaries.** Reproducibility, secret isolation, portability, volume-backed state, and controlled host access.
- **Credentials stay separate.** Podman Secrets and OpenClaw SecretRefs keep keys out of the app config and out of logs (not perfect — resolved values still sit in process memory).
- **Mount configuration in.** An agent directory of tools, skills, and MCP servers can be mounted so instances start ready.
- **Develop local, lift to Kubernetes.** The same workload patterns that apply to any app apply to agents.
- **Volumes are the recovery boundary.** Runtime state lives in a volume, so replacing the container preserves the agent.
- **Persistent agents earn their keep.** Daily routines give a reason to keep state and setup around.
- **A curated baseline helps onboarding.** An approved baseline agent, then personalized, beats copying a colleague's setup.
- **Model choice is per-instance config.** A model must be configured for an instance before that instance can switch to it.

## Detailed Notes

### Would you put OpenClaw on your work laptop? · 00:16
- O'Malley spent about ten years at Red Hat: seven on containers, Linux security, Kubernetes, and OpenShift.
- A move to the emerging-tech org around five years ago let her experiment freely; AI took over the work about three years ago.
- Early AI work felt repetitive — chatbots, Python, and Markdown.
- During a staycation, Moltbook led her to **OpenClaw** on GitHub. She checked the MIT license and built and ran it locally, then on OpenShift.
- Back at work, colleagues warned the project was a security nightmare and not to put it on a work laptop.
- Her reaction came from years of Linux application isolation: this was a chance to apply those controls to an agent. (Confidence, not a proven safety guarantee.)
- A simpler reason for containers: native installs leave dependencies and files on the host that need cleanup.

### An agent worth keeping · 03:13
- Her persistent personal agent is a "Forever Claw" named Shubhra, with two sub-agents.
- **Joy** supplies Jyotish astrology readings and a birth chart. **Bruno** delivers daily Boston Bruins briefings.
- During a close playoff race, Bruno gives her a reason to return every day — personal routines attached to an ongoing agent.
- Asked why it runs in a container, the list maps to ordinary app operations:
- **Reproducibility** (a repeatable environment), **secret isolation**, **portability** across laptops and clusters, **volume-backed state** for backup and recovery, and **explicit host access**.
- She runs across Mac and x86 and backs up the agent nightly.
- The container gives a predictable environment; explicit grants decide what it can reach on the host.
- Persistent storage keeps the agent's continuity separate from the container's lifetime.

### Bring configuration in; keep credentials separate · 05:45
- A reusable agent environment is more than an image. She keeps tools, skills, and MCP server configuration in an agent directory and mounts it into the container.
- Pieces are then present at startup instead of assembled by hand in each new instance.
- Credentials come through a separate path. Her local runtime is **Podman**, and she uses Podman Secrets to keep API keys outside the app config.
- **OpenClaw SecretRefs** add a second layer: a config field points to an external source instead of holding the key.
- The reference is configuration; the credential stays a separately managed value.
- Motivation: reduce accidental exposure, especially keys showing up in logs. She calls the arrangement imperfect.
- Current OpenClaw docs make the boundary precise: resolved credentials still exist in process memory, so SecretRefs are not process isolation.
- The cluster equivalent uses **Kubernetes Secrets**, delivered via environment variables or mounted files. A Secret object alone does not guarantee encrypted storage — that and least-privilege access must be configured.
- She packages these choices into her personal **OpenClaw Installer**, which others can try. Its repo has evolved beyond the recorded version.

### Develop locally, then move to Kubernetes · 08:10
- The larger pattern: agents running across infrastructure and talking to each other.
- Once agents serve business workflows, the requirements are familiar — security, operational control, and scale.
- Her approach matches other applications: develop locally, then lift the workload to Kubernetes.
- A secondhand example from a friend at NVIDIA: about ten engineers each ran OpenClaw in Kubernetes to check model evaluations periodically.
- He described doing the work of six engineers himself. This is a secondhand account with no defined productivity metric.
- The useful detail is recurring agent activity around evaluations, rather than a person checking each time.
- For O'Malley, automation leaves room for creative work. She described AI as "1,000 times better" than her at coding — a rhetorical self-assessment that drew skepticism from senior Red Hat engineers.
- **The persistent volume is the recovery boundary.** Local state lives in a Podman volume; Docker also has volumes; on Kubernetes a persistent volume claim (PVC) requests persistent storage.

### A shared baseline with room for personalization · 11:45
- A workplace could use the same packaging boundaries for onboarding.
- The idea: a curated baseline OpenClaw issued to each new hire, with company-approved MCP servers, approved authentication, team-specific skills, and access to everyday resources like Google Drive.
- Distribute that baseline across the team, then let each employee personalize it.
- This replaces sitting beside a colleague, copying their repo, and rebuilding the environment from informal instructions.
- It is her vision for reproducible onboarding, not a report of an existing rollout.
- Her own agent came after about a month of building, starting, and stopping test instances.
- She describes roughly 100 new commits an hour upstream, and around 10,000 commits when pulling after a couple of days — informal observations of rapid churn.
- A Forever Claw gives her a continuing personal environment alongside all those disposable instances.

### Configure a local instance named Joe · 14:14
- The demo starts by launching the installer's dev server with `npm run dev`.
- On her Mac, the installer runs outside a container because she has not worked out how to have a containerized installer spawn the containers it manages.
- Containers on macOS run inside a virtual machine (as with Podman Machine), adding another runtime boundary — an unresolved detail, not a general rule against nesting.
- The short form configures an instance step by step:
- **Name the instance.** She creates Joe.
- **Avoid a port conflict.** Another instance is running, so she changes the port (default 89 to 99).
- **Select credential mappings.** Her existing Podman secrets hold the provider API keys.
- For the Docker path, exported environment variables become sources for OpenClaw SecretRefs; she recommends a reference for every credential.
- Providers are another config choice: she picks **OpenRouter** (experimenting with Gemma) with **Anthropic** as a fallback, plus a custom local endpoint option.

### Deploy, inspect logs, and see the command · 17:29
- Two optional facilities extend the instance:
- **Observability:** the installer can set up an OpenTelemetry collector with Jaeger. She says it works but leaves it disabled to avoid taxing the demo machine.
- **SSH sandbox:** SSH keys and known hosts identify where sandboxed commands run, so the execution workspace can live off the local host.
- She deploys Joe and opens the instances view: a new Podman container is present with its gateway logs.
- The interface shows the generated **Podman command**, making the runtime call visible; it would show a Docker command when Docker is selected, but she has not tested that path.
- She selects a status command and calls the spin-up "two seconds" — no measured timing was provided.
- The demo shows the move from the installer form to a running local instance she can inspect and interact with.

### From preconfigured agents to cluster access · 19:32
- A second instance, **Larry**, was created through the same form with an MCP server and a sub-agent already configured.
- The agent view shows Docs Checker and its workspace files, making the mounted-configuration idea concrete.
- She tries to switch Joe's model to **GPT-5.4** but realizes Joe does not have that extra model configured, so she returns to Larry.
- **The lesson:** a model must be configured for an instance before that instance can switch to it. The sequence ends without a confirmed successful switch.
- The final view shifts from local to Kubernetes: the installer is connected to her kind cluster, where **Carl** is already running and can be opened.
- She also describes access to OpenShift instances, with the interface switching to OpenShift when connected there.
- The closing demo establishes access to existing cluster agents and completes the picture: local and cluster instances managed through one style of interface, with configuration, credentials, and persistent state all treated as explicit parts of the environment.

## Resources

- [OpenClaw Installer](https://github.com/sallyom/claw-installer) — browser-based deployment tooling for local containers, Kubernetes, and OpenShift.
- [OpenClaw secrets management](https://docs.openclaw.ai/gateway/secrets) — external credential references and runtime handling.
- [Podman secret delivery](https://docs.podman.io/en/v4.4/markdown/options/secret.html) — delivering secrets as mounted files or container environment variables.
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) — cluster credential injection plus the encryption and access controls needed.
- [Secrets in Docker Compose](https://docs.docker.com/compose/how-tos/use-secrets/) — granting individual services access to file-mounted secrets (related).
- [OpenClaw sandboxing](https://docs.openclaw.ai/gateway/sandboxing) — sandbox configuration, SSH execution, and workspace ownership.
