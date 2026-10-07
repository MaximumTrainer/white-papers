# Scaling Determinism to Agent Swarms: Orchestration, Control and Verification for Multi-Agent Engineering

*A companion paper to [Engineering for Determinism](https://github.com/MaximumTrainer/white-papers/blob/main/Engineering-for-Determinism.md)*

*For software engineers, test engineers, and platform teams operating multi-agent development workflows*

---

## 1. From One Agent to Many: Why Swarms, and Why Now

The [first paper](https://github.com/MaximumTrainer/white-papers/blob/main/Engineering-for-Determinism.md) established a thesis: **determinism is engineered, not prompted.** It showed how four layers of disciplined practice â€” contracts, gates, executable specifications, and ubiquitous language â€” transfer knowledge from lossy, expensive context into cheap, mechanical constraints. Everything in that paper assumed a single agent per task.

That assumption is already breaking. Real tasks cross bounded contexts. A billing change triggers a shipping change triggers a notification change. A language cleanup touches every module. A large feature decomposes into a dozen sub-tasks, each independent, each blocked on a sequential queue of one agent. Teams are running agents in parallel anyway â€” in separate terminal sessions, on separate branches, hoping the merge goes cleanly. That is an ad hoc swarm with no orchestration, no shared protocol, and no verification beyond whatever CI catches at the end.

This paper makes the swarm explicit. It asks three questions:

1. **Preconfiguration.** How do you partition a task across agents so each one operates in the small, constrained context window the first paper worked so hard to create?
2. **Control.** How do you coordinate agents without a shared session â€” without the context overload that coordination usually brings?
3. **Verification.** How do you confirm that the aggregate output of a swarm is correct, consistent, and meets the definition of done â€” when no single agent saw the whole picture?

The answer, predictably, is the same determinism stack. The practices from the first paper are not just compatible with swarms â€” they are *prerequisites*. A swarm without enforced architecture, mechanical gates, executable specifications, and a shared language is not a productivity multiplier; it is a chaos multiplier. Everything that follows assumes you have the single-agent stack in place. If you do not, start there.

### What a swarm is (and is not)

A swarm, for the purposes of this paper, is **two or more agents working on related sub-tasks within the same codebase, concurrently, without sharing a session context.** Each agent has its own context window, its own tool access, and its own branch. They coordinate through artefacts â€” commits, contracts, test results, and structured messages â€” not through shared memory.

What a swarm is *not*:

- It is not one agent calling sub-agents within a single session. That is tool use, and the context stays unified.
- It is not agents on unrelated tasks that happen to commit to the same repository. That is just CI.
- It is not a pipeline (agent A's output is agent B's input). That is sequential composition, not concurrency.

The defining characteristic is **concurrent, coordinated, independent context.** That is what makes swarms powerful and what makes them dangerous: power because each agent's context is small and focused; danger because no single agent holds the whole picture, so coherence must be engineered at the boundaries.

---

## 2. Swarm Topologies

Not every task needs a swarm, and not every swarm needs the same shape. Three topologies cover the vast majority of real engineering work. Choosing the right one is the first act of preconfiguration.

### 2.1 Fan-out / fan-in

```
flowchart TB
    O[Orchestrator] -->|sub-task + contract| A1[Agent: billing]
    O -->|sub-task + contract| A2[Agent: shipping]
    O -->|sub-task + contract| A3[Agent: notifications]
    A1 -->|branch + result| V[Verifier]
    A2 -->|branch + result| V
    A3 -->|branch + result| V
    V -->|pass / fail + report| O
    O -->|integration PR| R[Repository]
```

**When to use it.** A task decomposes into independent sub-tasks scoped to different bounded contexts or modules, with well-defined integration contracts between them. The orchestrator decomposes, the leaf agents implement in isolation, the verifier checks aggregate consistency, and the orchestrator integrates.

**Example.** "When an invoice is settled, release held shipments and send a confirmation email." Three contexts, three agents, one published event schema as the integration contract.

### 2.2 Parallel inner loop

```
flowchart TB
    S[Spec agent] -->|failing acceptance test| S
    S -->|unit test + port interface| I1[Impl agent: Order]
    S -->|unit test + port interface| I2[Impl agent: Coupon]
    S -->|unit test + port interface| I3[Impl agent: Money]
    I1 -->|green signal| S
    I2 -->|green signal| S
    I3 -->|green signal| S
    S -->|wire + run outer test| S
    S -->|commit| R[Repository]
```

**When to use it.** A single feature within one bounded context has multiple collaborators that can be implemented concurrently. The spec agent owns the outer acceptance test and decomposes the inner loop; each implementation agent receives one red unit test and the port interface of its collaborator. They work in parallel; the spec agent wires and validates.

**Example.** The coupon checkout feature from the first paper: `Order`, `Coupon`, and `Money` are independent collaborators that can be built simultaneously against their interfaces.

### 2.3 Sweep

```
flowchart TB
    C[Coordinator] -->|rename plan + rules| W1[Worker: billing/]
    C -->|rename plan + rules| W2[Worker: catalog/]
    C -->|rename plan + rules| W3[Worker: shipping/]
    C -->|rename plan + rules| W4[Worker: identity/]
    W1 -->|branch| M[Merge agent]
    W2 -->|branch| M
    W3 -->|branch| M
    W4 -->|branch| M
    M -->|cross-boundary check + integrated PR| R[Repository]
```

**When to use it.** A mechanical transformation â€” rename, migration, dependency upgrade, lint-rule adoption â€” must be applied across the whole codebase, but each bounded context can be transformed independently and in parallel.

**Example.** Renaming `finalise` to `settle` across four bounded contexts after a language workshop, or migrating every adapter from callback style to async/await.

### Choosing the topology

The decision tree is short:

1. Does the task cross bounded contexts with integration contracts? â†’ **Fan-out / fan-in.**
2. Is the task within one context but has multiple independent collaborators? â†’ **Parallel inner loop.**
3. Is the task a mechanical transformation across the whole codebase? â†’ **Sweep.**
4. Is the task a single change in a single context with no parallelism? â†’ **Single agent.** You do not need a swarm.

If you find yourself unsure, default to a single agent. A swarm adds coordination overhead; the payoff comes only when the parallelism is real and the boundaries are clean. A task that *should* decompose but *cannot* â€” because the bounded contexts are tangled, or the integration contracts are implicit â€” is telling you the architecture needs work before the swarm will help.

---

## 3. Preconfiguration: Setting Up the Swarm Before It Runs

Preconfiguration is the work that happens before any agent writes a line of code. Its purpose is to ensure that when agents start, each one has a small, precise, self-sufficient context â€” and that the aggregate of their work will compose correctly. This is the swarm equivalent of writing a failing test before implementing: you define success structurally, then let the agents fill in the substance.

### 3.1 The dispatch protocol in AGENTS.md

The first paper's AGENTS.md is a contract for a single agent. For swarms, it gains a new section: the **dispatch protocol.** This tells the orchestrator how to decompose tasks and what each agent receives.

```markdown
# AGENTS.md â€” contract for automated contributors

## ... (existing sections from paper 1: architecture, change discipline, etc.)

## Swarm protocol

### Decomposition rules
- Tasks scoped to a single bounded context: single agent.
  Loads: root AGENTS.md + context AGENTS.md (if exists) + context src + context tests.
- Tasks crossing bounded contexts: fan-out / fan-in.
  Orchestrator decomposes into per-context sub-tasks.
  Each leaf agent loads: root AGENTS.md + its context only.
  Integration contract (event schema, port interface) is passed explicitly.
- Mechanical codebase-wide transforms: sweep.
  Coordinator generates the transformation plan.
  Each worker loads: root AGENTS.md + its context + the plan.
- Feature with multiple independent collaborators: parallel inner loop.
  Spec agent owns the acceptance test.
  Each impl agent loads: root AGENTS.md + one unit test + one port interface.

### Agent isolation rules
- Agents NEVER share a session context.
- Agents NEVER read another agent's working branch during implementation.
- Cross-agent data flows through artefacts only: committed code, test results,
  structured messages via the orchestrator.
- Each agent commits to its own branch: swarm/<task-id>/<context-name>.

### Branch naming
- swarm/<task-id>/<context> for leaf agents
- swarm/<task-id>/integration for the orchestrator's merge
- Format: swarm/TICKET-123/billing, swarm/TICKET-123/shipping

### Context budgets
- Leaf agents: root AGENTS.md + context code + context tests + task description.
  Target: under 15,000 tokens of source. If a context exceeds this,
  scope the task to the relevant aggregate.
- Orchestrator: root AGENTS.md + integration contracts + sub-task descriptions.
  Never loads implementation code.
- Verifier: root AGENTS.md + diffs from all leaf branches + integration contracts.
  Never loads full source; works from diffs and test results.
```

Note what this achieves: the decomposition rules, isolation rules, and context budgets are **checkable**. An orchestrator that routes a single-context task to a fan-out topology is violating the protocol. A leaf agent that loads code from another context is violating isolation. These are auditable, and in a mature setup, enforceable by the orchestration framework itself.

### 3.2 Task decomposition: the orchestrator's job

The orchestrator is not a coding agent. It never writes production code. Its job is decomposition, and it has a small, high-signal context: AGENTS.md, the task description, and the bounded-context map (which, per the first paper, is already encoded in the commitlint `scope-enum` and the directory structure).

A well-formed decomposition produces, for each leaf agent, a **task envelope**:

```typescript
interface TaskEnvelope {
  taskId: string;                   // e.g. "TICKET-123"
  subTaskId: string;                // e.g. "TICKET-123/billing"
  context: string;                  // bounded context name
  branch: string;                   // e.g. "swarm/TICKET-123/billing"
  description: string;             // what to implement, in domain language
  acceptanceTest?: string;         // path or inline: the red test, if pre-written
  integrationContract?: string;    // the port / event schema this agent must honour
  contextFiles: string[];          // explicit list of files/dirs to load
  doNotLoad: string[];             // explicit exclusions
  completionSignal: string;        // how to report done: e.g. "push branch + green CI"
}
```

The explicit `contextFiles` and `doNotLoad` lists are the preconfiguration payoff. Instead of each agent deciding what to read â€” the inference step that causes context overload â€” the orchestrator prescribes the reading list. This is the swarm-level equivalent of outside-in TDD: the scope is defined externally before work begins.

### 3.3 Integration contracts as shared artefacts

When a swarm crosses bounded contexts, the agents do not share code â€” they share **contracts.** These are the same integration mechanisms DDD prescribes: published event schemas, port interfaces, and anti-corruption layer specifications. The critical discipline is that the contract must exist *before* the swarm starts.

```typescript
// contracts/invoice-settled.event.ts â€” shared across billing and shipping agents
export interface InvoiceSettledEvent {
  readonly type: 'InvoiceSettled';
  readonly invoiceId: string;
  readonly orderId: string;
  readonly settledAt: ISO8601;
  readonly amountCents: number;
  readonly currency: CurrencyCode;
}
```

The billing agent publishes this event; the shipping agent consumes it. Neither agent needs to know the other's implementation. The contract is small (a handful of lines), stable (it changes only when the integration's semantics change), and precise (it is a type definition, not prose). This is the inter-agent equivalent of the port interface in hexagonal architecture: a small, stable surface that bounds what each agent must understand.

If the contract does not exist yet, writing it is the orchestrator's first act â€” before any leaf agent starts. If it already exists, the orchestrator verifies it is current and passes it to both sides. A swarm launched without explicit contracts is an ad hoc swarm, and its merge will reflect that.

### 3.4 Pre-written acceptance tests as swarm anchors

The most powerful preconfiguration artefact is a failing test. For a fan-out swarm, the orchestrator (or a QA-authored specification) writes an **integration acceptance test** that exercises the cross-context behaviour end-to-end, using in-memory fakes for all adapters. This test is red before the swarm starts. It goes green only when all the leaf agents' work is integrated and correct.

```typescript
// acceptance/invoice-settlement-releases-shipment.spec.ts
it('releases held shipments when an invoice is settled', async () => {
  // arrange: an order with a held shipment and an unsettled invoice
  const orders = new InMemoryOrderRepository();
  const shipments = new InMemoryShipmentRepository();
  const invoices = new InMemoryInvoiceRepository();
  const events = new InMemoryEventBus();

  await orders.save(anOrder({ id: 'o-1', status: 'CONFIRMED' }));
  await shipments.save(aShipment({ orderId: 'o-1', status: 'HELD' }));
  await invoices.save(anInvoice({ orderId: 'o-1', status: 'ISSUED' }));

  // wire the contexts via the event bus
  const billingService = new BillingService(invoices, events);
  const shippingHandler = new ShipmentReleaseHandler(shipments);
  events.subscribe('InvoiceSettled', shippingHandler);

  // act
  await billingService.settleInvoice('o-1');

  // assert
  const shipment = await shipments.findByOrderId('o-1');
  expect(shipment.status).toBe('RELEASED');
});
```

This test is the swarm's definition of done. No leaf agent can see it passing in isolation â€” it requires the integrated output of all of them. It is the outer loop of the double-loop TDD pattern, lifted to the swarm level.

---

## 4. Control: Coordinating Without Sharing Context

The central tension of a swarm is coordination without coupling. Agents that share context are back to the single-agent problem at larger scale. Agents that share nothing produce incoherent output. The control layer manages this tension through four mechanisms.

### 4.1 The orchestrator as a message router, not a manager

The orchestrator's operational model is **dispatch and collect**, not supervise. After decomposition, it:

1. Distributes task envelopes to leaf agents.
2. Waits for completion signals (branch pushed, CI green).
3. Collects results (branch references, test reports, structured summaries).
4. Passes results to the verifier.
5. Integrates (merges branches) or requests rework.

Critically, the orchestrator **does not monitor agents' intermediate steps.** It does not read their in-progress code. It does not inject mid-task guidance. Each leaf agent is autonomous within its context boundary â€” the constraints (AGENTS.md, hooks, tests) provide the guardrails, not the orchestrator. This is the same principle as the first paper's hexagonal architecture: the orchestrator depends on the *contract* (the task envelope and completion signal), not the *implementation* (what the agent does inside its context).

When an agent gets stuck â€” a hook it cannot satisfy, a test it cannot pass â€” the correct response is to **fail and report**, not to escalate to the orchestrator for help. The orchestrator is not a coding agent; it cannot debug a billing implementation. A stuck agent produces a structured failure report:

```json
{
  "subTaskId": "TICKET-123/billing",
  "status": "blocked",
  "reason": "CouponRepository port requires a method `findByOrderId` not in the current interface",
  "suggestedAction": "Update the CouponRepository port to include findByOrderId, or clarify whether coupon lookup should go through the OrderRepository",
  "filesRelevant": ["src/application/ports/CouponRepository.ts"]
}
```

The orchestrator can then: update the contract and redistribute, reassign to a more capable agent, or escalate to a human. The key is that the failure is *structured and specific*, not a vague "I'm having trouble." The task envelope's explicit scope makes this possible â€” the agent knows exactly what it was asked to do and exactly where it got stuck.

### 4.2 Branch isolation and the no-read rule

Leaf agents in a swarm **must not read each other's branches.** This is the isolation rule from Section 3.1, and it is the hardest discipline to enforce because it is the most tempting to break. An agent working on shipping wants to check how billing structured its event emission. If it reads billing's branch, it has just coupled its implementation to billing's in-progress work â€” work that may change, be reverted, or fail verification. The shipping agent's context is now polluted with unstable assumptions.

The alternative is the contract. The shipping agent knows the `InvoiceSettledEvent` schema. That is all it needs. How billing emits that event is billing's concern. This is dependency inversion at the swarm level: agents depend on abstractions (contracts), not on each other's implementations.

Branch isolation is enforceable:

- **Git permissions.** Leaf agents have write access only to their own branch and read access only to `main` (or the integration base). They physically cannot read `swarm/TICKET-123/billing` if they are the shipping agent.
- **Workspace isolation.** Each agent operates in its own workspace, cloned from the integration base. No shared filesystem.
- **Audit.** The orchestrator can verify, after the fact, that no leaf agent's commit references files outside its bounded context.

### 4.3 Structured inter-agent communication

When agents must exchange information mid-task (rare, and a sign the decomposition may be wrong), they do so through the orchestrator, using structured messages:

```json
{
  "from": "billing",
  "to": "orchestrator",
  "type": "contract_amendment",
  "payload": {
    "contract": "InvoiceSettledEvent",
    "change": "add field `paymentMethod: string`",
    "reason": "shipping needs payment method to select carrier"
  }
}
```

The orchestrator evaluates the amendment, updates the contract if approved, and notifies affected agents. This is the expandâ€“contract migration pattern from the first paper, applied to inter-agent contracts: the new field is added (expand), consumers are updated (migrate), and only then is the old shape deprecated (contract).

Unstructured communication â€” one agent sending prose to another â€” is banned. It reintroduces the context-overload problem: the receiving agent must interpret free-form text probabilistically, exactly the failure mode the whole stack is designed to eliminate.

### 4.4 Timeboxing and circuit-breaking

Swarms need deadlines. An agent that runs indefinitely â€” retrying a failing test in increasingly creative and destructive ways â€” is a single-agent problem that becomes a swarm-level problem when the orchestrator is waiting for its completion signal.

Two mechanisms:

**Timeboxing.** Each task envelope includes a time budget. When the budget expires, the agent stops, commits its current state (including failing tests), and reports a structured timeout. The orchestrator decides whether to extend, reassign, or abort.

```markdown
## Swarm protocol (addition to AGENTS.md)

### Timeboxing
- Leaf agents: 15 minutes per sub-task (configurable in task envelope).
- If not green after the timebox: commit current state, push branch,
  report structured timeout. Do not continue.
- Orchestrator may grant one extension. After two timeouts, the sub-task
  is escalated to a human.
```

**Circuit-breaking.** If a leaf agent's hook feedback loop exceeds a threshold â€” say, five consecutive hook failures on the same rule â€” the agent stops and reports. This prevents the degenerate loop where an agent "fixes" a lint error by introducing a worse one, which triggers a different error, ad infinitum. The circuit breaker treats repeated failure as a signal that the agent lacks the context or capability to solve the problem, not as a signal to try harder.

---

## 5. Verification: Confirming the Swarm's Aggregate Output

The single-agent determinism stack relies on tests and hooks to verify output. A swarm adds a new verification challenge: the aggregate output may be internally inconsistent even when each agent's output is individually correct. The billing agent's event emission may not match the shipping agent's event consumption. The naming conventions may drift between contexts. The overall diff may exceed the size budget even though each sub-diff is small.

Verification is the swarm layer that catches these failures.

### 5.1 The verifier agent

The verifier is a dedicated agent â€” distinct from the orchestrator and from every leaf agent â€” that runs after all leaf agents have completed. Its context is deliberately narrow:

- Root AGENTS.md (the contract).
- The diff from each leaf branch against the integration base.
- The integration contracts (event schemas, port interfaces).
- The integration acceptance test.
- Test results from each leaf agent's CI run.

The verifier **never loads the full source code.** It works from diffs and contracts, which keeps its context small and focused on the swarm's *boundaries*, not its internals. Its checks fall into four categories.

**Contract compliance.** Does each agent's diff conform to the integration contracts? If billing publishes `InvoiceSettledEvent`, does the type in its diff match the shared schema? If shipping subscribes to it, does its handler accept the same shape? Type-level checks can be automated; semantic checks (does billing actually emit the event in the right circumstances?) are verified by the integration acceptance test.

**Cross-agent naming consistency.** Do the agents use the same terms for the same concepts? If billing calls the field `settledAt` and shipping calls it `completedAt`, the contract is technically satisfied (if the schema says `settledAt`) but the code is drifting. The verifier flags naming divergence as a warning.

**Aggregate diff metrics.** Sum of all sub-diffs should respect the task's overall size budget. Three agents each producing 200-line diffs is a 600-line aggregate change â€” above the 300-line guideline from the first paper's AGENTS.md. The verifier reports this; the orchestrator decides whether to split the task further or accept the overage.

**Documentation completeness.** Per the first paper's rule â€” "if you change public behaviour, update the matching file under `docs/`" â€” the verifier checks that every context whose behaviour changed also has a docs update in its diff.

### 5.2 Independent verification vs. self-grading

A critical property of the verifier: **it has never seen the implementation agents' reasoning.** It has no access to their session context, their intermediate attempts, or their chain-of-thought. It sees only their committed output and the contracts they were given. This is deliberate. An agent that verifies its own work is influenced by its own reasoning â€” if it convinced itself that a certain approach was correct during implementation, it will be biased toward confirming that during verification. The verifier has no such bias; it is evaluating cold artefacts against explicit criteria.

This is the same principle as code review: the reviewer is valuable precisely because they did *not* write the code and are not anchored to its design decisions. The verifier agent operationalises this principle mechanically.

### 5.3 The integration acceptance test as the final gate

The integration acceptance test from Section 3.4 is the verifier's strongest tool. After all leaf branches are merged into the integration branch, the verifier runs this test. If it passes, the swarm's output satisfies the cross-context behaviour specification. If it fails, the verifier's job is to localise the failure: which context's contribution caused it?

Localisation is tractable because the test uses domain-language assertions (per the first paper's DDD guidance). `expect(shipment.status).toBe('RELEASED')` failing tells the verifier that the shipping context's handler did not do its job â€” even though billing's event emission may be correct. The verifier can report this specifically to the orchestrator, which can re-dispatch the shipping sub-task without disturbing billing.

### 5.4 A verification checklist

```markdown
## Swarm verification checklist (addition to AGENTS.md)

### Verifier runs after all leaf agents report completion.

#### Automated checks (CI-enforceable)
- [ ] Each leaf branch passes its own CI independently.
- [ ] Integration branch (merged leaves) passes full CI.
- [ ] Integration acceptance test passes on the integration branch.
- [ ] No leaf branch modifies files outside its bounded context.
- [ ] Aggregate diff size is within budget (default: 500 lines for swarms).
- [ ] Every context with behaviour changes has a docs/ update.

#### Contract checks (verifier agent)
- [ ] Published event schemas in diffs match the shared contract types.
- [ ] Port interface changes, if any, are reflected in all consuming contexts.
- [ ] No new cross-context imports introduced (architectural linting on integration branch).

#### Consistency checks (verifier agent, advisory)
- [ ] Naming across contexts is consistent (same terms for same concepts).
- [ ] Test names use the ubiquitous language.
- [ ] Commit messages follow Conventional Commits with correct context scopes.
```

---

## 6. The Extended Determinism Stack

The first paper defined four layers: constraints, gates, specifications, language. The swarm extends this with three additional layers that sit *above* the single-agent stack, wrapping it:

```
flowchart TB
    S1[Preconfiguration â€” dispatch protocol, task envelopes,\nintegration contracts, context budgets] --> S2
    S2[Orchestration â€” decomposition, routing,\nbranch isolation, structured messaging] --> S3
    S3[Verification â€” independent verifier, contract compliance,\naggregate metrics, integration acceptance test] --> S4
    S4[Constraints â€” AGENTS.md\narchitecture, change discipline] --> S5
    S5[Gates â€” hooks & CI\nmechanical enforcement] --> S6
    S6[Specifications â€” outside-in tests\nexecutable definition of done] --> S7
    S7[Language â€” DDD & fluent code\nmeaning embedded in structure]
```

The critical insight: **the swarm layers do not replace the single-agent layers; they depend on them.** Preconfiguration works because bounded contexts exist (language layer). Orchestration works because agents are constrained (constraints layer) and gates catch violations (gates layer). Verification works because there are executable specifications to verify against (specifications layer).

A team that tries to adopt swarms without the single-agent stack will find that:

- Without bounded contexts, there are no clean decomposition boundaries, and every sub-task leaks into every other.
- Without hooks, each agent's output must be manually reviewed for formatting, linting, and convention compliance â€” multiplied by the number of agents.
- Without executable specifications, the verifier has nothing to verify against except prose descriptions, which is where non-determinism lives.
- Without a ubiquitous language, the orchestrator's task descriptions and the agents' code will use different vocabularies, and cross-agent naming will drift silently.

### Extended summary table

| Layer | What it enforces | Context it eliminates | Determinism it adds |
| --- | --- | --- | --- |
| **Preconfiguration** | Task decomposition, context budgets, integration contracts, branch isolation | Agents deciding what to load; implicit cross-context dependencies | Every agent starts with a prescribed, minimal, self-sufficient context |
| **Orchestration** | Dispatch-and-collect, structured messaging, timeboxing, circuit-breaking | Ad hoc inter-agent communication; unbounded agent runtime | Coordination without shared context; bounded failure |
| **Verification** | Contract compliance, aggregate consistency, independent grading | Self-grading bias; cross-agent drift detected only in human review | Mechanical cross-agent coherence check before human review |
| **Constraints** *(paper 1)* | Architecture, change discipline, definition of done | Inferring rules from mixed-era code | Every session starts from the same explicit rules |
| **Gates** *(paper 1)* | Format, lint, arch rules, commit grammar, secrets | Style instructions; convention drift review | Identical feedback on every violation |
| **Specifications** *(paper 1)* | Executable spec precedes implementation | Prose ambiguity; "when am I done?" | Binary doneness; regressions caught in-loop |
| **Language** *(paper 1)* | One vocabulary across code, tests, docs, tasks | Tribal knowledge; cross-codebase inference | Task language matches code; contexts load in isolation |

---

## 7. Swarm Anti-Patterns

Swarms introduce failure modes that do not exist in single-agent work. Recognising them early saves significant rework.

### 7.1 The omniscient orchestrator

**Symptom.** The orchestrator loads the full codebase to "understand the task properly" before decomposing it. Its context is now as overloaded as a single agent's would be, and its decomposition decisions are as non-deterministic.

**Fix.** The orchestrator reads AGENTS.md, the bounded-context map, and the task description. It decomposes based on structure, not implementation. If it needs implementation details to decompose, the task is under-specified â€” fix the task, not the orchestrator's context.

### 7.2 The chatty swarm

**Symptom.** Agents exchange multiple rounds of unstructured messages through the orchestrator during implementation. Each message adds context to the receiving agent, progressively overloading it. By the third round, agents are responding to each other's reasoning rather than to the contracts.

**Fix.** If agents need to communicate more than once, the decomposition is wrong. Re-decompose the task so each sub-task is self-contained within its context boundary. The only legitimate mid-task communication is a contract amendment, which is rare and structured.

### 7.3 Context leakage through test fixtures

**Symptom.** Two agents in different contexts share a test fixture file (e.g. `test/helpers/factories.ts`) that contains builders for both contexts' domain objects. Agent A modifies the shared fixture for its needs and breaks Agent B's tests.

**Fix.** Test fixtures follow bounded-context boundaries. Each context has its own `test/billing/helpers/`, `test/shipping/helpers/`, etc. Shared factories for integration tests live in a dedicated `test/integration/` directory that neither leaf agent touches â€” only the verifier's integration acceptance test uses them.

### 7.4 Premature swarm adoption

**Symptom.** A team without architectural boundaries, without a test suite, and without an AGENTS.md launches a three-agent swarm. The agents produce three incompatible implementations that cannot be merged. The team concludes that "swarms don't work."

**Fix.** The prerequisite stack from the first paper is not optional. You need bounded contexts (to decompose), ports and adapters (to isolate), tests (to verify), and a contract file (to constrain). Build those first. You will find that many tasks that seemed to need a swarm are handled fine by a single well-constrained agent once the architecture supports it.

### 7.5 The verify-only-at-the-end trap

**Symptom.** All leaf agents complete, their branches are merged into the integration branch, and the integration acceptance test fails. The verifier reports the failure, but localising it across three agents' combined 500-line diff is nearly as hard as writing the feature from scratch. The swarm's output is discarded.

**Fix.** Progressive verification. The orchestrator merges branches incrementally â€” billing first, then shipping on top of billing, then notifications on top of both â€” running the integration test suite after each merge. The first failure is localised to the last merged branch. This is the same expandâ€“contract principle from the first paper, applied to branch integration: each step is independently verifiable and reversible.

---

## 8. Adoption Guide

As with the first paper, adopt incrementally. Each phase builds on the previous one.

### Prerequisites (from paper 1, non-negotiable)

- AGENTS.md under 80 lines, with architecture and change discipline.
- Pre-commit hooks mirrored in CI.
- At least one area practising test-first agent work.
- Bounded contexts identified and named (even if imperfectly separated).

### Phase 1 â€” Sweep swarms (lowest risk)

Start with the simplest topology. Pick a mechanical codebase-wide transformation: a rename, a dependency upgrade, a lint-rule adoption. Decompose by bounded context. Each worker agent applies the transformation within its context, protected by the existing test suite. A merge agent combines the branches. The total risk is low because the transformation is mechanical and the test suite catches semantic errors.

**What you learn:** Branch isolation discipline, context budget sizing, basic orchestration. Whether your bounded contexts are actually independent enough to work on in parallel.

**Metrics:** Wall-clock time vs. sequential single-agent (expect 2â€“4x speedup proportional to the number of contexts). Cross-branch merge conflict rate (expect near zero if contexts are properly separated; conflicts indicate entanglement).

### Phase 2 â€” Fan-out for cross-context features

Pick a small cross-context feature with a clear integration contract (ideally an existing event or port interface). Write the integration acceptance test as the swarm anchor. Decompose per context. Run the fan-out / fan-in topology with a human playing the orchestrator role (decomposing the task, distributing envelopes, reviewing the verifier's output).

**What you learn:** Contract authoring discipline, verifier setup, progressive integration. Whether your integration contracts are explicit enough to serve as inter-agent boundaries.

**Metrics:** Verifier pass rate on first integration attempt (target: >70%). Contract amendment frequency (target: <1 per swarm run; higher indicates under-specified contracts). Comparison of aggregate diff coherence vs. single-agent cross-context diffs.

### Phase 3 â€” Parallel inner loop

Apply the parallel inner loop topology to a feature with multiple independent collaborators within one context. The spec agent writes the acceptance test and fans out inner-loop unit tests. This requires the tightest discipline â€” the spec agent must produce precise, minimal port interfaces for each implementation agent.

**What you learn:** Whether your team's outside-in TDD practice is disciplined enough to support parallelisation. Whether your port interfaces are genuinely narrow (if an implementation agent needs to read more than the port to do its job, the port is too thin).

**Metrics:** Per-agent context size (target: under 5,000 tokens of source per implementation agent). Retry count per implementation agent vs. single-agent inner loop (expect equal or lower).

### Phase 4 â€” Automated orchestration

Replace the human orchestrator with an automated one. The orchestrator agent reads AGENTS.md, the task description, and the bounded-context map, and produces task envelopes without human intervention. The verifier runs automatically. Human review shifts from "did the agents do it right?" to "is this the right task decomposition?" and "does the integrated output meet requirements?"

**What you learn:** Whether your AGENTS.md and context structure are explicit enough to support fully automated decomposition. Where human judgment is still needed (likely: ambiguous task descriptions, novel integration patterns, business-logic edge cases that aren't covered by existing tests).

**Metrics:** Orchestrator decomposition accuracy (% of decompositions that a human reviewer would not change). End-to-end swarm completion rate without human intervention. Total wall-clock time from task to merged PR.

### Metrics to watch (swarm-specific)

All metrics from the first paper still apply (diff size, revert rate, review time, agent retry count). Add:

- **Swarm completion rate.** Percentage of swarm runs that produce a mergeable integration branch without human intervention.
- **Contract amendment frequency.** How often integration contracts need to change mid-swarm. Falling over time indicates improving contract authoring.
- **Cross-agent consistency score.** Verifier findings per swarm run: naming drift, documentation gaps, aggregate diff oversize. Should trend toward zero.
- **Merge conflict rate.** Conflicts between leaf branches during integration. Near zero indicates clean decomposition; rising indicates tangled contexts.
- **Verification pass rate.** Percentage of swarm outputs that pass the verifier's first check. Rising indicates improving preconfiguration.
- **Wall-clock speedup.** Ratio of swarm wall-clock time to estimated single-agent sequential time. The primary productivity metric.

---

## 9. Security and Trust Boundaries in Swarms

A single agent with repository access is one trust boundary. A swarm of agents is multiple trust boundaries, and the interactions between them create new attack surfaces.

### 9.1 Principle of least privilege per agent

Each leaf agent should have the minimum access required for its sub-task:

- **File access.** Read access to its bounded context and integration contracts only. Write access to its own branch only. No access to secrets, environment variables, or deployment configuration outside its context.
- **Tool access.** If the agent doesn't need network access, don't grant it. If it doesn't need database access, don't grant it. The task envelope should specify the tool set explicitly.
- **Duration.** The timebox from Section 4.4 is also a security control: a compromised or misbehaving agent is automatically stopped.

### 9.2 Treat inter-agent messages as untrusted input

The structured messages in Section 4.3 flow through the orchestrator, which should validate them against a schema before forwarding. A contract amendment from a leaf agent is a *request*, not an instruction â€” the orchestrator evaluates it against the original task and the existing contracts before distributing. This prevents a compromised or confused agent from manipulating another agent's behaviour through crafted messages.

### 9.3 Audit trail

Every swarm run should produce an audit log:

```json
{
  "swarmId": "TICKET-123",
  "topology": "fan-out",
  "orchestrator": { "model": "...", "startedAt": "...", "decomposition": [...] },
  "agents": [
    {
      "subTaskId": "TICKET-123/billing",
      "model": "...",
      "contextTokens": 12340,
      "hookFailures": 2,
      "completionStatus": "green",
      "wallClockSeconds": 480
    },
    ...
  ],
  "verifier": {
    "contractChecks": "pass",
    "consistencyWarnings": ["naming drift: settledAt vs completedAt"],
    "integrationTestResult": "pass",
    "aggregateDiffLines": 347
  }
}
```

This log is the swarm equivalent of clean git history: pre-compressed context for debugging, retrospectives, and future orchestrator tuning.

---

## Executive Summary

**Context.** The first paper â€” *Engineering for Determinism* â€” showed how four practices (contracts, gates, specifications, and language) transfer knowledge from lossy context into mechanical constraints, making single-agent output deterministic and reviewable. This companion paper extends those practices to multi-agent swarms.

**The problem.** Real engineering tasks cross bounded contexts, and running them through a single agent means either overloading its context or decomposing manually. Teams are already running informal swarms â€” multiple agents on separate branches â€” with no coordination, no shared protocol, and no aggregate verification. The results are inconsistent, hard to merge, and expensive to review.

**Three swarm questions, three answers.**

1. **Preconfiguration.** Extend AGENTS.md with a dispatch protocol that tells the orchestrator how to decompose tasks by topology (fan-out, parallel inner loop, sweep). Provide each agent with a task envelope: a prescribed, minimal context including only its bounded-context code, integration contracts, and (where applicable) a failing acceptance test. Define context budgets. Enforce branch isolation â€” agents never read each other's work.

2. **Control.** The orchestrator dispatches and collects; it does not supervise. Leaf agents are autonomous within their constraints. Inter-agent communication is structured, schema-validated, and rare â€” more than one exchange indicates a bad decomposition. Timeboxing and circuit-breaking bound failure: agents that cannot finish within budget stop and report, rather than spiralling.

3. **Verification.** An independent verifier agent â€” one that never saw the implementation agents' reasoning â€” checks the aggregate output: contract compliance, cross-agent naming consistency, aggregate diff size, documentation completeness. The integration acceptance test, written before the swarm starts, is the final gate. Progressive integration (merging branches one at a time, testing after each) localises failures to their source.

**The extended stack.** Three swarm layers â€” preconfiguration, orchestration, verification â€” sit above the four single-agent layers. They depend on them: without bounded contexts there are no decomposition boundaries; without hooks each agent's output needs manual review; without tests the verifier has nothing to verify; without a shared language the orchestrator's task descriptions and the agents' code talk past each other.

**Adoption.** Start with sweep swarms (mechanical transformations, lowest risk), progress to fan-out cross-context features, then parallel inner loops. Automate the orchestrator only after the contracts and verification are reliable. Watch swarm completion rate, contract amendment frequency, merge conflict rate, and wall-clock speedup.

**For leadership.** A well-engineered swarm trades one agent's wall-clock time for parallelism, without trading away determinism. The cost is coordination infrastructure â€” dispatch protocol, contracts, verification â€” but that infrastructure is DDD, hexagonal architecture, and CI extended to their logical conclusion. Teams that invested in the single-agent determinism stack are already most of the way there.

## External References

- [Engineering for Determinism](https://github.com/MaximumTrainer/white-papers/blob/main/Engineering-for-Determinism.md) â€” the prerequisite paper
- [Sensors for Coding Agents](https://martinfowler.com/articles/sensors-for-coding-agents.html)
- [Harness Engineering](https://martinfowler.com/articles/harness-engineering.html)
- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.dddcommunity.org/book/evans_2003/) â€” Eric Evans
- [Growing Object-Oriented Software, Guided by Tests](http://www.growing-object-oriented-software.com/) â€” Freeman & Pryce (outside-in TDD)
