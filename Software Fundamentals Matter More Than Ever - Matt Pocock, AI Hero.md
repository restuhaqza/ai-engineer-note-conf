# "Software Fundamentals Matter More Than Ever"

| Field | Details |
| --- | --- |
| **Speaker** | Matt Pocock |
| **Affiliation** | AI Hero |
| **Type** | Talk |
| **Date** | 2026-04-23 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=v4F1gFy-hqg) |
| **Tags** | `agent-engineering` `safety-and-governance` `coding-and-developer-tools` `developer-workflows-and-testing` |

## Summary

Matt Pocock (a teacher at AI Hero) argues that AI coding agents make classic software-engineering fundamentals more important, not obsolete. He takes aim at the "specs-to-code" idea — write a spec, let AI generate the code, and fix problems by editing the spec rather than the code. When he tried it, each round produced worse code, ending in what he calls "vibe coding by another name."

His counter-thesis: code is not cheap, and bad code is more expensive than ever because it blocks the very benefits AI offers. Drawing on older books — A Philosophy of Software Design, The Pragmatic Programmer, The Design of Design, and domain-driven design — he maps common AI failure modes to well-known remedies.

Along the way he shares small, practical "skills" (reusable agent instructions) for each failure mode: interviewing the AI until you share a design concept, building a shared vocabulary, using feedback loops and test-driven development, and refactoring into deep, testable modules. The closing message is that AI can act as a capable tactical programmer, but humans must keep strategic ownership of the system's design.

## Key Takeaways

- **Specs-to-code degrades over time.** Repeatedly regenerating from a spec without caring about the code produces progressively worse implementations.
- **Code is not cheap.** Hard-to-change code stops you from getting the full benefit of AI, which works best in a clean codebase.
- **Complexity is the enemy.** The measure of bad code is how hard it is to understand and change.
- **Establish a shared design concept first.** The "Grill Me" skill has the AI interview you relentlessly until you both understand the plan.
- **Give the model the codebase's language.** A documented "ubiquitous language" reduces verbosity and keeps implementation aligned.
- **Use feedback loops and TDD.** Types, browser access, and tests give the model feedback in small increments instead of one big blind step.
- **Prefer deep modules.** Substantial functionality behind simple interfaces is easier for both humans and AI to navigate and test.
- **Own the interfaces, delegate the implementation.** Humans design the boundaries; AI can fill in the inside, with more oversight for critical areas.

## Detailed Notes

### Generated code is not the same as a maintainable system · 00:19
- Pocock frames the talk as reassurance for engineers who fear their skills no longer matter. His claim: software fundamentals matter more than ever.
- He recently built a course, "Claude Code for Real Engineers," which forced him to define what good AI coding actually looks like.
- The **specs-to-code** movement says you can describe an app in a specification, generate the code, and fix problems by editing the spec rather than the code.
- He tried it. He ran it, looked at the code, and got code — then worse code each time he re-ran the compiler, until the result was garbage.
- His verdict: treating code as something that can manage itself is just "vibe coding by another name."
- He reaches for John Ousterhout's *A Philosophy of Software Design*: **complexity** is anything in a system's structure that makes it hard to understand and modify. Bad code is code that is hard to change without causing bugs.
- From *The Pragmatic Programmer*, he borrows **software entropy**: change one thing without thinking about the whole design, and the codebase degrades.
- The driver of specs-to-code is the claim that "code is cheap." Pocock rejects it. Bad code is the most expensive it has ever been, because it stops you from reaping what AI can offer.

### Establish a shared design concept before committing to a plan · 04:41
- Failure mode one: the AI did not build what you wanted. Pocock treats this as a **requirements-gathering** problem — a communication barrier between you and the model.
- He borrows the **design concept** from Frederick P. Brooks's *The Design of Design*: an ephemeral, shared idea of the thing being built, floating between collaborators. It is not an asset or a file.
- His remedy is a deliberately simple skill called **Grill Me**: interview me relentlessly about every aspect of this plan until we reach a shared understanding.
- Part of the prompt is to "walk down each branch of the design tree," resolving dependencies between decisions one by one.
- He reports the skill went viral (around thirteen thousand stars in its repo). It turns the AI into a kind of adversary, asking forty, sixty, sometimes a hundred questions.
- The resulting conversation can become a product requirements document, or for small changes turn directly into issues for an agent to pick up.
- He contrasts this with the default plan mode in the tool he uses, which he finds too eager to create a planning artifact and start implementing. He prefers reaching a shared design concept first.

### Give the model the same language used by the codebase · 06:49
- Failure mode two: the AI is too verbose and seems to talk past you — a **language gap**, not just a style problem.
- He compares it to a developer working with a domain expert. If they use different terminology, the developer translates requirements into code that neither side fully understands.
- He turns to **domain-driven design** (DDD) and its concept of a **ubiquitous language**: conversations, code, and agreed domain terms all derive from the same model.
- In practice, it is a Markdown file of shared terms that both the developer and the AI use consistently.
- His **ubiquitous language skill** scans the codebase for terminology and generates that Markdown file, with tables of terms.
- He keeps the file open while planning with the AI. From reading the model's thinking traces, he reports that shared terminology improves planning, makes the model think less verbosely, and keeps implementation closer to the plan.
- He notes he is still exploring DDD rather than claiming mastery of it.

### Make feedback frequent, then design code that can actually be tested · 08:57
- Failure mode three: the AI built the right thing, but it doesn't work. The remedy is **feedback loops**.
- He recommends static types (if you are not using TypeScript, he says, that's a mistake), browser access for front-end work, and automated tests.
- Even with these, the model often uses feedback poorly. It tends to do too much at once, then remember to type-check or test afterwards.
- *The Pragmatic Programmer* calls this "outrunning your headlights": the rate of feedback is your speed limit, so you should test as you go and take small, deliberate steps.
- His third skill is **test-driven development** (TDD): write a test, make it pass, then refactor while considering the design. TDD forces the model into small steps.
- He stresses that testing is genuinely hard. You must decide unit size, what to mock, and which behaviours matter — and these decisions are interdependent.
- Good codebases are easy to test. The better the codebase, the better the feedback you can give the model, and the better the code it produces.
- He returns to Ousterhout's **deep modules**: relatively few large modules with simple interfaces, hiding complexity, rather than many shallow modules exposing lots of functions.
- AI is very good at creating shallow-module codebases — many tiny pieces it then struggles to navigate, so it fails to understand what the code does.
- A deep-module codebase keeps the same behaviour behind clear interfaces. You test at the interface, which is what makes the architecture reward TDD.
- His skill **improve codebase architecture** explores a codebase, finds related code, and wraps it in a deep module — a repeatable set of steps.

### Own the interfaces, delegate selectively, and invest in design continuously · 13:05
- Failure mode six: the feedback loops work and you ship more code than ever, but your brain cannot keep up. He asks for a show of hands and says he feels it too.
- Deep modules help here because they can be treated as **gray boxes**: design and verify the external interface, and don't scrutinise every implementation detail.
- This delegation has limits. Pocock warns that critical areas — his example is financial functionality — may need more attention than less critical modules.
- His tip: design the interface, delegate the implementation. But it only works if you keep an accurate mental map of the system.
- Modules and interfaces should therefore be part of the ubiquitous language and appear explicitly in product requirements documents, including how those boundaries will change.
- He quotes Kent Beck: invest in the design of the system every day. Specs-to-code, by contrast, divests from design rather than investing in it.
- Final message: code is not cheap. Think of AI as a capable tactical programmer — a sergeant on the ground — while you stay above it, doing strategic work that still relies on twenty-year-old software fundamentals.
- He points attendees to his public skills repository and to aihero.dev for his training and newsletter.
