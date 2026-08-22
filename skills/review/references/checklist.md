# Review checklist

## Per-primitive question set

**Surface** — Is authn/authz stated, and at which hop? Is there a request contract with a versioning policy? Is there a quota per client class, and is it policy rather than capacity? Which invariants are enforced here rather than merely checked? What happens on partial results — is there a documented behaviour, or does it silently truncate?

**Channel** — Which delivery guarantee? What is the ordering key, and is ordering scoped to it? What is the failure mode when the far side is down? What is the retry policy, and does it have jitter? **Who owns the payload schema, and what is the compatibility mode?** Is TTL stated, and is it TTL on the message rather than retention on a store?

**Processor** — Stateless, stateful or batch? If it combines partial results, how do they compose — and is that composition correct? If it approximates, what is the error bound, and who agreed to it?

**Store** — Is this the truth of something, or derived from something? If derived: what is the staleness bound, and is it stated in units of time? What is the access pattern, and does the store family match it? What is the retention, and who decided it? Is there a resharding path?

## Supply question set

The four above are about the system. This one is about the organisation that has to run it, and it is the half most design documents omit entirely.

**Supply** — Which hosting posture is assumed, and did anybody say so? Which contracts are already held — a CDN deal, a committed-spend agreement, a licensed database — and does the design use them? How many people, with what capability, and who is woken at three in the morning? What is the cost envelope, as a number, and is the dominant recurring charge in this design even named? Which lifecycle is this — spike, one-off, prototype becoming platform, platform? Blank answers are the deliverable. Do not fill them in from habit.

## Anti-patterns to look for

| Pattern | What it looks like |
|---|---|
| **Reference architecture in disguise** | Technologies appear before any number. The requirements section reads as justification written afterwards |
| **The undeclared API** | An async channel with no schema owner, no compatibility mode and invisible consumers |
| **Synchrony mistaken for reliability** | Every write goes to the strict store because "it must be reliable" |
| **Capacity before demand** | Node counts and coordination protocols exist before requests/second was ever derived |
| **Metrics of convenience** | Dashboards of what was easy to emit; nothing traces to a promise made to a client |
| **The silent range** | "Between 1 and 128 hosts" — a range nobody narrowed, with machinery built for the top of it |
| **Granularity as identity** | "We're microservices" as an answer, with no force cited per split |
| **Wrong as a degradation rung** | Serving stale data as if fresh, with no as-of marker |
| **Unlabelled figures** | Numbers with no provenance, load-bearing under a decision |
| **Disproportion** | The citation is real and is satisfied by something two orders of magnitude smaller. Nobody did the division |
| **The unconsumed figure** | A volume elicited, labelled, and then divided by nothing. It reads as rigour and decides nothing |
| **A percentile on a zero-tolerance promise** | "99.9% of opt-outs honoured". The unit is wrong, not the number |
| **The rootless chain** | Every link names its parent, and following them to the end reaches a design decision or an adjective, never a client |

## Severity ordering

Rank findings by cost, not by how easy they were to spot:

1. **Correctness** — a mechanism that produces wrong answers under a stated condition (bad merge semantics, an invariant enforced nowhere, an ordering assumption that does not hold).
2. **Zero-tolerance promise with no counter** — an acceptable failure count of zero that nothing counts. Unfalsifiable in production, and it does not fail gradually.
3. **Unmapped requirement** — a promise with no mechanism. Discovered in production by definition.
4. **Arithmetic that does not survive checking** — an assertion contradicted by its own inputs, or a correct sum of the wrong quantity. The second is worse: it survives checking.
5. **Disproportion** — a mechanism two or more orders of magnitude larger than the requirement it cites.
6. **Uncited mechanism** — recurring cost with no recorded reason.
7. **Unlabelled or unconsumed provenance** — a load-bearing figure nobody can source, or a figure that decides nothing.
8. **Adjacent-area gaps** — observability, operations, delivery, lifecycle left to "later".

## What not to do

- Do not propose a redesign. Report the findings; the human decides the response.
- Do not fill a gap from a reference architecture — the gap is the finding.
- Do not report a finding you cannot demonstrate with a quote or an arithmetic step.
- Do not treat an uncited mechanism in a **running** system as a deletion instruction. Chesterton's fence: report it as "no recorded reason — confirm before removing".
- Do not treat an oversized mechanism in a **didactic** artifact as a deletion instruction either. The finding is that the stated requirement cannot discriminate it.
- Do not rule on a self-derived chain. Record root, depth, weakest link and whether the root is a client constraint; the human decides what it is worth.
- Do not call a mechanism oversized without the division. Record the ratio, or drop the finding.
- Do not report on what you could not see. State the coverage limit and mark the findings a legible diagram could overturn.

## Two substitutions to refuse

- **The impressive figure for the deciding figure.** Before accepting any arithmetic, list the promises and ask which figure each one waits on. A design that computes its largest quantity and not its binding one has done arithmetic, not sizing.
- **The mechanism-health metric for the promise counter.** If the promise were violated right now, would this number move? If not, the promise is uncounted however many dashboards exist.

## Delivery — the questions nobody is asked

Delivery is the one adjacent area whose decisions all get made by default, so the review is the first time any of them is proposed. Run the same test as on any component.

**Repo topology**
- Is there a caller this team's commit cannot reach? Name it. If none, every repository past the first is uncited.
- Which way may dependencies point, and is it enforced? An upstream importing from a downstream is the same defect whatever the layout.
- Was the repo split derived from the org chart? Team boundaries buy runtime units; they buy a repository only when two teams cannot land one commit together.

**Versioning**
- Who can decline an upgrade? If nobody, the version is identity and the scheme is decoration.
- SemVer without a declared public API is a promise about an undefined boundary.
- Is the upgrade driven by time (support window, security horizon, regulatory date) rather than compatibility? Then the scheme is answering the wrong question.

**Build**
- What is the measured build time, and against what stated tolerance? Demand the ratio.
- Is the build hermetic? A cache in front of a non-hermetic build has a broken invalidation contract and can serve wrong bytes with a green result.
- What does the cache cost, and does it grow with commits rather than with traffic?

**Environments**
- For each: what does it verify that production cannot?
- What does parity cost, and is that number anywhere in the design?

**Release**
- Which strategy, and which force bought it — a graded abort signal (canary) or instant rollback (blue-green)?
- Does the abort metric exist and is it wired? A canary without one is a slow deploy.
- Is there fast startup and graceful shutdown? Graceful shutdown is a channel property: in-flight requests and unacked messages.
- What is the ordering constraint between a schema migration and the deploy that depends on it?
- When was the rollback last rehearsed? A date, or it is an assumption.

**Reconciliation / GitOps**
- Is there drift to detect at all? If nothing but the pipeline can write, the reconciler is uncited.
- Desired-state store: is it treated as truth, with its own restore path?
- Pull channel: guarantee, ordering key, backpressure, schema, owner.
- Commit-to-live staleness bound: stated, or only observed afterwards?
- Is drift a zero-tolerance count or a dashboard?

**Flags**
- Does each flag carry a category and an expected lifespan?
- Are release toggles being removed, and is the removal owned?
- Do the degradation ladder's rungs each have an actuator, or are they promises with no switch?

**Twelve-factor claims**
- Treat it as a declared hosting posture, and ask whether step 01 chose it.
- *Backing services as attached resources* contradicts choosing a store by access pattern, invariant and failure mode.
- *Config in the environment* is wrong for the quasi-static class, which needs history, diff and a reviewer.


## The stress pass — questions per artifact noun

For each client class · data class · schema · component name, ask **what hits this?** — regulation, reversal, scale inversion, identity abuse, clock, geography, acquisition. Then:

- Which rows have two or more 1s — what fails together, and is that shared fate traced to a promise?
- Which column absorbs reversal/migration/residency semantics it was never given?
- Is any column's row-set a subset of another's (merge candidate)? Any zero column (under-stressed)?
- Does every stressor resolve to exactly one of: stated requirement / finding / recorded option (with trigger)?
- Is anything **built** whose only citation is a stressor? (Furniture with a scary story — report under rule one.)
- Where the ladder exists: is every discovered rung signed like a declared one? Is the reversal path present at all?
