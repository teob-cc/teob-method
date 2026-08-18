---
name: derive
description: "Derive a system design from what is outside it, using the Outside-In Method's ten steps in order — clients and organisation first, arithmetic before any technology is named, then storage, trust, failure, runtime, operations and the traceability map. Use when designing a new system or subsystem, sizing one, or answering a system design question."
---

# Outside-In Derive

Design a system by starting outside it and letting each decision consume the output of the one before. **No technology is named until the demand arithmetic binds.**

The deliverable of each step is small and written — a table, a list, a labelled graph. The written artifact is what the next step consumes; skipping the artifact breaks the chain, and the chain is the method.

Full method: <https://teob.cc/method>

## The one thing that makes this different

Every mechanism you introduce must cite the constraint that produced it. If you cannot name the constraint whose removal deletes a component, **do not add the component.**

You will be tempted to reach for a familiar shape — a log, a cache, a service split — before the numbers exist. That is the failure this method is built to prevent. Reference architectures are not knowledge; they are somebody else's answer to a question you have not asked yet.

## Load the vocabulary first

| Primitive | Is | Must declare |
|---|---|---|
| **Surface** | Where a client touches the system | Authn/authz and at which hop · contract and versioning · quota per client class · invariants enforced here · partial-result behaviour |
| **Channel** | A transfer between two nodes | Guarantee · ordering key · failure mode · retry policy · payload schema and compatibility mode · owner |
| **Processor** | Derives | Stateless / stateful / batch · how partial results compose · error bounds if approximate |
| **Store** | Holds state | Truth or derived · access pattern · retention · staleness bound if derived |

Types apply to **roles, not products**. One Kafka is a producer channel, a store with a retention window, and one consumer channel per reader — three primitives, three owners. Name the role first; choose the product last.

A **store is a role with constraints**. "Postgres" is not a design decision until you have written what the store must hold, survive and serve.

## The ten steps

Detailed procedure per step: `references/ten-steps.md`. Load it when you begin step 01 and keep it open.

### Phase I · Outside
- **01 · Outside** — characterise every entity outside the boundary. **Demand**: who calls this, at what volume, with what tolerances. **Supply**: what this organisation can host, staff, afford and already has contracts for. Adversaries are clients. Your dependencies are client relationships reversed.
- **02 · Load** — the first arithmetic pass. Requests/second, bytes/day, working-set size, fan-out, growth. **Before any technology is named.**

### Phase II · Shape
- **03 · Data** — per data class: what is truth, what is derived, freshness tolerance, the acknowledgement each write needs, and the invariants no view can tell you about.
- **04 · Storage & transport** — now, and only now, choose store families and channel guarantees. Decide granularity: how many runnable units, and where the seams go.
- **05 · Trust** — boundaries, authn/authz per surface, data classification, what is enforced where.

### Phase III · Behaviour
- **06 · Derivation** — the processors, what they compute, merge semantics, error bounds.
- **07 · Failure** — per client class: the tolerance, the degradation ladder, then the mechanics per edge.
- **08 · Runtime & placement** — instances, regions, placement, the second arithmetic pass (capacity).

### Phase IV · Carry
- **09 · Operations** — observability proportioned to consequence, outcome KPIs, zero-tolerance counts, delivery and release shape, cost.
- **10 · Trace** — the traceability map, checked in both directions. Re-run trigger.

## The two arithmetic passes

The order is the whole trick.

1. **Demand, before choosing a store.** Derive requests/second, bytes/day and working set from the volumes stated at step 01. Round out loud and on purpose — a day is about 10⁵ seconds; a billion requests a day is ~12,000/s average; take a peak factor and say what it is.
2. **Capacity, after choosing a store.** Node counts, replicas, headroom — these need the store's per-node characteristics, so they cannot exist earlier.

Doing capacity first is how a design ends up with a coordination protocol between 128 hosts that the arithmetic never required.

**Do the division in both directions.** Compute what one node can serve before deciding you need many. Most systems fit on fewer nodes than the shape suggests, and the extra nodes are then justified by failover — which is a *stated reason*, not a silent one.

**Divide at the moment you choose, not in review.** Citing a constraint is not enough. Record what the constraint demands, what the mechanism supplies, and the ratio between them. Near 1 is sized to the job. In the tens is headroom, and you must name the factor — growth, peak, or failover. In the hundreds means the citation is decorative and the shape came from a reference architecture; the mechanism is not wrong so much as unjustified at that size. The ratio costs one line at the moment of choice, and a re-derivation to recover afterwards.

## Acknowledgements scope the expensive store

Ask of every write: **if this call returned "probably", would the next screen be a lie?**

| Kind | The client is told | Cost |
|---|---|---|
| **Receipt** | Durably recorded, will not be lost. An idempotency key comes back; work completes later | One durable append. Most "we've got it" flows |
| **Reservation** | A contended resource is held for this caller now, usually with an expiry | Needs the strict store and the invariant guarding it |
| **Completion** | Final state reached at the moment of the response | The expensive one, and frequently unnecessary |

Only the writes that genuinely need reservation or completion belong in the strict store. Everything else is a projection with a stated staleness bound. **Reliability is durability; synchrony is a UX decision** — conflating them drags every write into the expensive store, and with it every scaling problem that store has.

## Provenance — non-negotiable

Every figure carries a label: **measured** · **contractual** · **assumed**. Every assumed figure becomes a risk-register entry naming the decision it supports.

**A mechanism may cite a decision instead of a constraint — then name the decision, and justify *that*.** Nothing partitions itself: a partitioned store was bought by a volume that did not fit one machine, a leader with elections by clients who tolerate no maintenance window. Write the parent down at the moment you take the child, because the chain is checked to its root later either way, and reconstructing it afterwards is the expensive version.

**Never invent a number silently.** If the user has not given you a volume, say so, propose one explicitly as assumed, show what it changes, and continue. A design built on unlabelled invented numbers is worse than one with gaps, because the gaps are visible.

**Every figure must be consumed.** Do not leave step 01 with a number that no later step divides, bounds or decides with. Name the step that will use each one; where there is none, either the figure goes or the decision it should have informed was missed. A volume elicited in the first five minutes and never divided by anything is a question that should not have been asked, and it is the commonest defect in system design writing — the arithmetic looks done because the numbers are on the page.

## Amending an existing derivation

If the user brings a design that already exists and one thing has changed, **do not re-run the ten steps.** A store is a role with constraints; a given technology is one thing sufficient to play it, and when constraints move only the *sufficiency claim* expires. Name the claim that died, re-check the rows the change actually touches, and say explicitly what it does not touch — the second half is what stops a scared team re-opening everything. `references/ten-steps.md` carries the table. Two rules on top of it: **client tolerances do not move because the budget shrank** (that is a finding to escalate with numbers, not a promise to lower quietly), and an amendment to a joint-signature artifact **is void until it is signed again**.

## Stop rules — taking them is the method working

The full pass is an afternoon, and it earns that when there are several client classes, real scale questions or regulated data. A single-class CRUD service on one database deserves **Outside, Data and Trace** in fifteen minutes.

Other stop rules: one node suffices (stop sizing) · shallow trust boundary (stop at authn) · no contended resource (no reservation, no strict store).

**The delivery stop rules, which are the ones most often skipped past:** one repository until a caller exists that your commit cannot reach · one artifact per runnable unit, and the unit count was settled at 04 · no version scheme until someone can decline an upgrade · no build system beyond the ecosystem default until a *measured* build time crosses a tolerance stated first · no environment beyond production until you can say what it verifies that production cannot · no pipeline stage that cannot fail the build.

Taking a stop rule is the method working. Running all ten steps on a trivial system is the method being performed.

## Granularity is a range, not two nouns

"Monolith or microservices" is the wrong axis. Three independent decisions:

- **Runtime units** — how many separately deployable processes
- **Module seams** — internal boundaries with explicit contracts
- **Repo topology** — one repo or many

Each split must cite a force: independent deployment, fault isolation, a genuinely different scaling axis, or an existing team boundary that is not going away.

**Repo topology has its own force, and it is not team count.** It is whether a caller can be reached by your commit — the line between a *public* interface and a *published* one. Two teams who can still land one commit together have not bought a second repository; one customer-held SDK has. That answer was recorded at step 01 and it also decides the version scheme, the compatibility mode and the deprecation window.

**The first split is a phase change, not a refactor.** Inside one process there are no channels. The moment you split, an edge acquires a guarantee, an ordering key, a failure mode, a retry policy, a schema and an owner — all of which were free a moment ago. Buy it deliberately. *"It'll scale better"* is not a force.

## Output

Produce the artifacts in order, each small enough to fit on a screen. Close with the traceability map:

| Requirement (testable) | Mechanism | Verified by |
|---|---|---|

Then check it both ways: uncited mechanism → delete it or declare it self-derived with its parent decision; unmapped requirement → design for it.

## Rules

- **Do not name a technology before step 04.** If the user names one first, accept it as a constraint of the organisation (that is legitimate supply-side input) and record it as such — not as a derived decision.
- **Do not produce a diagram as the deliverable.** The deliverable is the annotated graph plus the map. A diagram without its annotations is decoration.
- **State what you assumed, every time.**
- **Take the stop rules out loud** — say which you took and why.

## References

| File | Load when |
|---|---|
| `references/ten-steps.md` | Always — the per-step procedure, asks and deliverables |
