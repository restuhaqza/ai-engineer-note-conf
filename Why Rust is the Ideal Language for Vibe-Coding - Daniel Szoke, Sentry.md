# Why Rust is the Ideal Language for Vibe-Coding

| Field | Details |
| --- | --- |
| **Speaker** | Daniel Szoke |
| **Affiliation** | Sentry |
| **Type** | Expo session |
| **Date** | 2026-05-27 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ugUeZ8-b-u0) |
| **Tags** | `coding-and-developer-tools` `evals` |

## Summary

Daniel Szoke maintains Sentry's Rust SDK and argues that when you choose a language for agentic coding, first-attempt runnable code is the wrong thing to optimize. Python, JavaScript, and TypeScript are popular for AI coding because models produce working drafts quickly — but the same flexibility that makes them easy to write also lets mistakes slip through.

His proposal is Rust. It is a compiled language built for safety and performance, and its compiler rejects whole classes of errors — type mismatches, unsafe memory use, and unsafe concurrency — before the program ever runs. Crucially, its error messages explain what went wrong and often how to fix it, which makes them useful feedback for an agent working in a loop.

He walks through a small concurrency example: a counter incremented by many tasks that Rust refuses to compile because the shared type is not safe to send between threads. The error names the exact type (`Rc<RefCell<i32>>`) and points the agent toward a thread-safe replacement (`Arc<Mutex<i32>>`).

Szoke's conclusion is that a harder first attempt is fine when an agent can compile, read the error, and revise. Every compile error is a potential production bug avoided. He notes the talk is sponsored by Sentry.

## Key Takeaways

- **First-try runnable code is a weak target.** The flexible languages agents write most easily also admit subtle mistakes.
- **Tests and code review help but cannot prove correctness.** An agent can write bad tests or bad reviews too.
- **Model errors will not fully vanish.** LLMs are nondeterministic, so generated code needs guardrails just as human code does.
- **Rust acts as a deterministic checking layer:** its compiler enforces types, memory safety, and thread safety.
- **Rust's diagnostics explain the failure** and suggest a fix — exactly the feedback an agent needs to repair code.
- **A harder first attempt can pay off in an agent loop** that compiles, inspects errors, and revises.
- **Static checks can also reject valid programs,** so compile errors are not a one-to-one count of bugs avoided.
- **Compilation is a faster and more reliable check** than asking an AI to review code — his view, offered without a benchmark.

## Detailed Notes

### Which language should a coding agent use? · 00:16
- Szoke is the Rust SDK maintainer at Sentry; his question is which language makes sense for agentic, or "vibe," coding.
- Asked ChatGPT, it named Python first and JavaScript/TypeScript second. His own experience puts TypeScript on top for AI coding.
- He cites a GitHub report ranking TypeScript first by distinct monthly contributors in August 2025.
- Important caveat: that measures participation, not language quality or how well agents actually code in it. GitHub's suggestion that AI-assisted development drove the shift is a suspicion, not proven causation.

### Why familiar languages produce quick first drafts · 02:04
- Familiar languages are easy for both people and models, and are often learned first.
- Frameworks, libraries, and examples mean new projects can start from existing building blocks.
- They scaffold and run fast: Python and JavaScript are interpreted, and TypeScript compiles only lightly, making edit-and-run quick.
- Type annotations help agents avoid misuse, but `any` in TypeScript and `Any` in typed Python weaken those checks.
- Few constraints plus familiar patterns let models produce runnable code on the first try — an appealing starting experience, but not necessarily the right goal.

### Runnable code is not the same as reliable code · 03:34
- Generating code easily and getting reliable behavior are different objectives. The flexibility that helps writing also leaves room for obvious and subtle mistakes.
- Types help but do not cover every failure class, and escape hatches reduce their coverage.
- Szoke expects model errors to persist even as models improve, because they are nondeterministic by design.
- The requirement echoes ordinary software engineering: just as we guard against human error, we must guard against model error.

### What tests and review can't cover · 05:31
- Tests and code review are big helps, but relying on them alone leaves gaps:
  - Tests written after implementation may just re-check implementation details instead of the intended behavior.
  - Even test-driven development cannot cover every input combination in practice.
  - If a model writes the tests or the review, it can make mistakes in the verification too.
- A failing test proves a case is wrong; passing many cases cannot prove every input is right.
- Tests are useful evidence, not a proof of correctness.

### Add a deterministic checking layer · 08:46
- He borrows a framing from Yuval Noah Harari's book *Nexus*: "alien intelligence" fits better than "artificial," because these systems think differently from humans.
- The practical risk: a model's mistakes may not resemble the ones a human reviewer expects. Token prediction can produce tidy names and comments while hiding a subtle bug — or reach for a heuristic where a direct check would be simpler.
- His motivation is Murphy's Law: if a failure is possible, repeated chances will eventually expose it.
- A **deterministic** check rejects the same violation every time; human review, agent review, and testing reduce risk but do not guarantee that.
- Rust aims for C/C++ performance while enforcing rules around types, memory, and concurrency. Compiling raises confidence that covered error classes are absent — though it does not prove the app implements the right behavior.
- Caveat: Rust's guarantees hold for safe Rust and correctly written unsafe abstractions. `unsafe` still carries obligations and does not switch off every check.
- The compiler's explanations are as valuable as its rejections: they say what went wrong and often how to fix it, giving an agent concrete context to revise.

### The constraints Rust enforces · 10:53
- **Strict type safety** — no `any` escape hatch in ordinary type checking, so a mismatch cannot simply be waved away.
- **Explicit absence** — there is no universal null. A value that may be missing is `Option<T>`, written as `Some(value)` or `None`, so using the payload requires handling the optional.
  - Caveat: this does not force graceful handling. `unwrap()` and `expect()` compile but panic when called on `None`.
- **Fearless concurrency** — the compiler checks data shared across threads and rejects unsafe combinations in safe code.
- These are only some enforced properties, but concurrency is a concrete example of a constraint that becomes useful feedback for an agent.

### A shared counter the compiler rejects · 12:06
- The example: a counter starting at 0, incremented by 100 concurrent tasks, so the expected final value is 100.
- The chosen types allow mutation within one thread but provide no synchronization across threads — a **data race**.
- In a language that permits unsynchronized shared memory, the counter could occasionally finish wrong, and the bug could be hard to locate inside a larger app.
- Rust rejects it before running. The diagnostic says an async **future** cannot be sent between threads safely — it is not `Send` (`Send` means a value is safe to transfer between threads).
- Further down, the error names the captured value: `Rc<RefCell<i32>>` is not `Send`. `Rc` gives shared ownership within a thread and `RefCell` allows runtime-checked mutation; neither syncs across threads.
- The thread-safe replacement is `Arc<Mutex<i32>>`: `Arc` shares ownership across threads, and the mutex guards the read-modify-write.

```rust
// Rejected: an Rc is not safe to move between threads.
fn increment(counter: Rc<RefCell<i32>>) -> impl Future<Output = ()> + Send { ... }

// Accepted: Arc shares ownership here, and the mutex guards the update.
fn increment(counter: Arc<Mutex<i32>>) -> impl Future<Output = ()> + Send { ... }
```

- The error narrows the agent's next decision to a concrete one — pick a shared-state type suited to the required concurrency — instead of a vague instruction to make the program safer.

### Evaluate the repair loop, not the first draft · 14:22
- Rust is harder to get right first try because of its many rules. That cost changes inside an agent loop: generate, compile, inspect errors, repair, compile again.
- A compiler error is **actionable feedback**, not just a failed generation, and each one is potentially a production bug avoided. Caveat: static checks can also reject valid programs, so errors are not a one-to-one bug count.
- Szoke argues compiling is faster than asking an AI agent to review the code (a practical comparison he gives without a timing benchmark). More fundamentally, the compiler enforces its covered rules every time, while a reviewer may miss a violation.
- Review still has a place, especially for behavior beyond the compiler's guarantees. The proposed workflow keeps it and adds a deterministic layer usable on every revision.
- Closing: he identifies the talk as sponsored by Sentry, offering three free months of the Business plan via QR code plus agent-monitoring features and the downstairs booth.

## Resources

- [Nexus](https://www.ynharari.com/book/nexus/) — Yuval Noah Harari's history of information networks, source of the "alien intelligence" framing.
- [Understanding Send and Sync](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html) — Rust's rules for transferring ownership and sharing references across threads.
- [Rust Option reference](https://doc.rust-lang.org/std/option/enum.Option.html) — optional values, payload extraction, and fallbacks in Rust.
- [Spawning tasks with Tokio](https://tokio.rs/tokio/tutorial/spawning) — async tasks, ownership, and `Send` bounds in practice.
- [Sentry Rust SDK](https://github.com/getsentry/sentry-rust) — the SDK Daniel Szoke maintains.
