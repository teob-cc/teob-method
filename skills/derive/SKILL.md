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
| **Surface** | Where a client touches the system | Authn/authz and at which hop · contract and versioning · quota and rate limit per client class · invariants enforced here · partial-result behaviour · idempotency on writes · server-initiated push (webhooks, sockets): signing, redelivery, ordering |
| **Channel** | A transfer between two nodes | Eight: sync or async — and if sync, whether the acknowledgement is commit-bearing · guarantee · ordering key · backpressure · message TTL · payload schema and compatibility mode · far side · departure. Plus one inherited (the latency budget) and a named owner |
| **Processor** | Derives | Request-time / stateful stream / batch / model · state size and recovery time · idempotent under replay · keying and parallelism · late / out-of-order handling · rebuild path · how partial results compose · error bounds if approximate |
| **Store** | Holds state | Truth or derived · durability and replication · consistency offered · access pattern · retention · size, growth and hot-key profile · staleness bound if derived · where several processes play it, the channels between them (replication, membership) and the placement rule that keeps members out of one failure domain |

Types apply to **roles, not products**. One Kafka is a producer channel, a store with a retention window, and one consumer channel per reader — three primitives, three owners. Name the role first; choose the product last.

A database decomposes the same way, and almost nobody draws it that way: a store, a producer channel for the write path, and one consumer channel per reader. The consistency a store **offers** is the store's property; the consistency each reader **takes** — leader or replica, isolation level, staleness tolerance, timeout, page size — belongs to that reader's channel.

A **store is a role with constraints**. "Postgres" is not a design decision until you have written what the store must hold, survive and serve.

## The ten steps

Detailed procedure per step: `references/ten-steps.md`. Load it when you begin step 01 and keep it open.

### Phase I · Outside
- **01 · Outside** — characterise every entity outside the boundary. **Demand**: who calls this, at what volume, with what tolerances — five: latency, ordering, durability and accuracy per class, freshness per view — and what each class is still handed when a dependency is down. **Supply**: what this organisation can host, staff, afford and already has contracts for. Adversaries are clients. Your dependencies are client relationships reversed.
- **02 · Load** — the first arithmetic pass: four demand figures — peak request rate per client class, data volume and growth, working-set size, and the cost envelope carried in from 01. **Before any technology is named.**

### Phase II · Shape
- **03 · Data** — resolve each view into truth or projection against the freshness bound declared for it at 01; then per data class: the acknowledgement each write needs, its sensitivity classification, and the invariants no view can tell you about.
- **04 · Storage & transport** — now, and only now, choose store families and channel guarantees. Decide granularity: how many runnable units, and where the seams go. Then run the second arithmetic pass (sizing) immediately, while the choice is still cheap to reverse.
- **05 · Trust** — enforcement of the classification fixed at 03: who may see which data class, the single surface that authorises it, tenancy, erasure.
- **06 · Derivation** — the processors: what computes each projection, replay and rebuild, merge semantics, error bounds.

### Phase III · Reality
- **07 · Failure** — the ladder was promised at 01: prove each rung reachable on purpose, then derive the mechanics per edge. Then the stress pass.
- **08 · Runtime & placement** — deliberately small: runtime by organisational constraint, the artifact per runnable unit, environments, the placement map.
- **09 · Operations** — observability proportioned to consequence: outcome KPIs, zero-tolerance counts, SLOs, rung metrics, audits, tracing. Then delivery and release shape, ownership, cost.

### Phase IV · Proof
- **10 · Trace** — the traceability map, read three ways: cited, mapped, and counted. Re-run trigger.

## The two arithmetic passes

The order is the whole trick.

1. **Demand, before choosing a store — step 02.** Derive requests/second, bytes/day and working set from the volumes stated at step 01; the cost envelope from 01 is the fourth figure. Round out loud and on purpose — a day is about 10⁵ seconds; a billion requests a day is ~12,000/s average; take a peak factor and say what it is.
2. **Capacity, after choosing a store — the sizing pass, inside step 04.** Amplification, node counts, replicas, headroom — these are properties of the graph just drawn and need the store's per-node characteristics, so they cannot exist earlier. Run it immediately, while the choice is still cheap to reverse.

Doing capacity first is how a design ends up with a coordination protocol between 128 hosts that the arithmetic never required.

**Do the division in three directions — per node, per cluster, per key.** Compute what one node can serve before deciding you need many. Most systems fit on fewer nodes than the shape suggests, and the extra nodes are then justified by failover — which is a *stated reason*, not a silent one. Per key: **the hottest key's peak rate** ÷ what one key can serialise, about 1 ÷ its commit latency (5 ms ≈ 200/s, assumed until measured). Near or above 1, no node count serialises that key — one key lives on one node — and the ratio decides the enforcement rung before any store is named.

**Divide at the moment you choose, not in review.** Citing a constraint is not enough. Record what the constraint demands, what the mechanism supplies, and the ratio between them — supplied over demanded, with the demand taken at its average. Near 1 is sized to the job. In the tens is headroom, and you must name the factor — growth, peak, or failover. In the hundreds means the citation is decorative and the shape came from a reference architecture; the mechanism is not wrong so much as unjustified at that size. The ratio costs one line at the moment of choice, and a re-derivation to recover afterwards. The fourth field is **what the mechanism introduces** — a cache, a stale window; a conditional write, a per-key ceiling; two sources of truth, a reconciliation. Each introduced item is cited by the mechanism that answers it, carried onto the degradation ladder or the risk register, or accepted in writing.

## Acknowledgements scope the expensive store

Ask of every write: **if this call returned "probably", would the next screen be a lie?**

| Kind | The client is told | Cost |
|---|---|---|
| **Receipt** | Durably recorded, will not be lost. An idempotency key comes back; work completes later | One durable append. Most "we've got it" flows |
| **Reservation** | A contended resource is held for this caller now, usually with an expiry | Needs the strict store and the invariant guarding it |
| **Completion** | Final state reached at the moment of the response | The expensive one, and frequently unnecessary |

Only the writes that genuinely need reservation or completion belong in the strict store; a receipt costs one durable append. Truth covers exactly what the acknowledgements and the invariants require — everything else is a projection with a stated staleness bound. **Reliability is durability; synchrony is a UX decision** — conflating them drags every write into the expensive store, and with it every scaling problem that store has.

## Invariants have their own ladder

**The scope sets the isolation.** One key → a conditional write. Several keys in one partition → serializable; snapshot isolation is not enough (write skew), and record the store's default. Across partitions → move the key, or a saga, which makes the invariant E3 for the saga's length. The partition key is the scope key of the invariant hardest to move.

**The enforcement ladder** — how strictly an invariant holds under concurrent writers; not **the degradation ladder**, which is what a client is handed.

| Rung | How | Violation | Price |
|---|---|---|---|
| **E1 Serialised** | One serialisation point checks every change | None | Contention: the per-key ceiling |
| **E2 Escrowed** | Writers spend local slices of the quantity, refilled from the serialisation point | None | Slack stranded in slices: undershoot |
| **E3 Bounded** | Writers check a lagging view; a reconciler compensates afterwards | At most rate × lag, as a number | The overshoot, priced, plus the reconciler |
| **E4 Detected** | No check on the path; an audit finds and compensates | Unbounded until the audit | Every violation, priced |

Take the cheapest rung whose worst-case violation, priced per unit, the signatories accept. A violation that would be *wrong* — a seat sold twice, a balance below zero — allows E1 or E2 only; E2 is the escape for a key too hot for E1. At E3 there are two promises, one enforcer each: the bound on the path, the exact figure in the reconciler (a processor at 06). At E4 the path checks nothing and the audit is the only enforcer.

## Provenance — non-negotiable

Every figure carries a label: **measured** · **contractual** · **assumed**. Every assumed figure becomes a risk-register entry naming the decision it supports.

**Price both ways.** Beside every cost line, a **loss line**: what one unit of violation costs the business, labelled like any figure. Where overshoot and undershoot have different prices, an approximation states its bound per side and puts the error on the cheaper one.

**Declare the cost to reverse.** From step 04 on, every decision is **one-way** (a partition key, a stored or published format, a published interface, an erasure) or **two-way** (a store behind a module seam, a cache). Before a one-way decision on an assumed figure, buy the measurement — a probe priced against the cost of reversing; a two-way one proceeds and records its re-run trigger.

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

Each runtime split must cite one of six forces: independent delivery (a different cadence) · independent scaling (a different axis) · fault isolation · trust zone (regulated data or a different adversary) · ownership (a different team's pager) · runtime mismatch (a latency floor, a GPU, a forced language).

**Repo topology has its own force, and it is not team count.** It is whether a caller can be reached by your commit — the line between a *public* interface and a *published* one. Two teams who can still land one commit together have not bought a second repository; one customer-held SDK has. That answer was recorded at step 01 and it also decides the version scheme, the compatibility promise and the deprecation window.

**The first split is a phase change, not a refactor.** Inside one process there are no channels. The moment you split, an edge acquires eight properties — sync or async, a guarantee, an ordering key, backpressure, a TTL, a payload contract, a far side whose membership changes while the channel is open, and a departure behaviour for each end — all of which were free a moment ago. Buy it deliberately. *"It'll scale better"* is not a force.

## Output

Produce the artifacts in order, each small enough to fit on a screen. Close with the traceability map:

| Requirement (testable) | Mechanism | Verified by |
|---|---|---|

Then check it three ways: uncited mechanism → take the first disposition that applies — delete it · surface the missing requirement it traces to (the common case) · declare it self-derived, naming the parent decision, which then faces the same test — and where the system is already running it is Chesterton's fence, reported as *no recorded reason — confirm before removing*; unmapped requirement → design for it; **the same mechanism named by more than one requirement → a coupling finding**, recorded rather than fixed, carried onto the ladder as one entry per promise, and used to rank the risk register — first the entries whose figure is assumed, whose decision is one-way and whose mechanism's coupling count is above one.

## Rules

- **Do not name a technology before step 04.** If the user names one first, accept it as a constraint of the organisation (that is legitimate supply-side input) and record it as such — not as a derived decision.
- **Do not produce a diagram as the deliverable.** The deliverable is the annotated graph plus the map. A diagram without its annotations is decoration.
- **The probe exception.** The refusal of invented figures stands — a missing input is a finding, never a blank to fill silently. A **stressor is not a figure**: it is a labelled probe run *on* the model (no probability, ridiculous allowed), it does not enter the map's first column, and it carries no number into the arithmetic. Its only legal outputs are a stated requirement, a finding, or a recorded option with a trigger. Accepting a labelled probe is not accepting an invented number.
- **State what you assumed, every time.**
- **Take the stop rules out loud** — say which you took and why.

## References

| File | Load when |
|---|---|
| `references/ten-steps.md` | Always — the per-step procedure, asks and deliverables |
