# FROM APPLICATION SILOS TO A COMMON SERVICE LAYER
## A Capability Platform Strategy

**Purpose:** Make the case for a strategic shift in how the SDO builds software: from every application team constructing its own service layer to a common enterprise service layer delivered by capability platforms and consumed by all applications. Define what changes, why it must change, what the end state looks like, and how we get there.

**Owning authority:** [SDO Technical Director]
**Status:** Draft for internal review
**Version:** 0.2
**Date:** [DD Month YYYY]

---

## 1.0 Where we are today

Every application we build today carries its own service layer. When a team needs identity, storage, search, document processing, translation, or task management, the default answer is to build it, wrap it, or integrate it locally, inside the application, on the application's timeline, to the application's standard.

This is not because teams are undisciplined. It is the rational response to the environment they operate in. There is no catalog to shop. Capabilities that do exist elsewhere are not adoptable without a negotiation: find the owning team, get a meeting, get credentials, reverse-engineer an undocumented interface, and accept a dependency with no SLO, no deprecation policy, and no support channel. Faced with that, a team with a delivery date builds their own. Every time. And each time they do, they make the same rational choice easier for the next team, because the enterprise has one more incompatible local implementation and still no common service.

The result is a structural condition, not a series of isolated decisions:

- **We pay for the same capability many times.** Identity handling, storage access, document parsing, and pipeline plumbing exist in near-every application, implemented differently in each. With [N] applications each carrying [M] common capabilities, we fund on the order of N times M builds where the enterprise needs M products.
- **Every new application starts from zero.** Nothing built before it was built to be consumed, so nothing accelerates it. Application timelines are dominated not by mission logic but by rebuilding plumbing that already exists somewhere we cannot reach.
- **Quick-reaction is structurally impossible.** A quick-reaction application is, by definition, assembled from things that already work. In the current state there is nothing to assemble from, so "quick-reaction" means a compressed version of the same ground-up build.
- **Our vendors deliver demos that pass acceptance.** An OTA can deliver a service that functions in a demonstration and satisfies every technical requirement, while no application team can adopt it without a month of integration. We accept the delivery and inherit the integration cost, per consumer, forever.
- **Divergence compounds into risk.** Five identity implementations means five places to get classification handling wrong, five audit surfaces, five patch cycles. In our mission space, duplicated plumbing is not just waste. It is attack surface and compliance exposure multiplied by the number of copies.

None of this is fixed by building more services. We have built services before. They were not adopted, because functioning and being consumable are different properties, and we have only ever required the first.

## 2.0 The strategic shift

The shift is from **applications that own their capabilities** to **applications that assemble from capabilities the enterprise owns**.

That sentence implies more than an architecture. It is a change in what we build, what we buy, how we accept vendor deliveries, how teams are measured, and where engineering investment compounds. Concretely, five things invert:

| | Today | Under this strategy |
|---|---|---|
| **Unit of investment** | The application. Capabilities are a byproduct, trapped inside whichever app built them | The capability platform. Applications are thin assemblies of mission logic over shared services |
| **Default behavior** | Build it locally. Reuse is the exception requiring effort | Consume from the catalog. Building locally is the exception requiring justification |
| **Definition of done** | The service functions | The service is a product: a stranger can adopt it in production without a meeting |
| **Integration model** | Per-consumer negotiation. Every adoption is a meeting, a ticket chain, and a bespoke integration project | Self-service. Contract, credentials, quickstart, and reference client are the product |
| **What compounds** | Nothing. Each application's investment dies with the application | Every productized service makes the next application cheaper. The platform layer appreciates |

The heart of the shift is the third row. Today we treat a working capability as a finished capability. Under this strategy, a capability that works but cannot be adopted without the owning team's involvement is unfinished, no matter how well it functions. **Reusable is a property we will now require, define, measure, and enforce, not a property we hope emerges.**

## 3.0 Why we must make this shift

**The math does not work any other way.** Application demand is growing faster than the engineering base. We cannot hire our way to N times M. The only way a fixed-size organization delivers a growing application portfolio is if each application costs less than the last, and that happens only when applications share a service layer instead of each rebuilding it. Platform leverage is not an efficiency initiative. It is the only scaling model available to us.

**Quick-reaction is now a mission requirement, not an aspiration.** The demand signal we receive is increasingly for applications in weeks, not quarters. That timeline is only achievable by assembly. A common service layer is the precondition for every quick-reaction commitment we want to make.

**We are at a structural window.** The platform OTAs [AIAS, LPS, and the Foundation platform to follow] are being structured now. Acceptance requirements written today decide whether vendors deliver products or components for the life of those agreements. This leverage exists exactly once per OTA, at drafting time. Retrofitting productization into a signed agreement means paying for it twice.

**AI-assisted development raises the stakes in both directions.** Assisted teams generate application code faster than ever, which means they generate duplicated service layers faster than ever if the default does not change. The same tools make catalog-based assembly dramatically faster when there is a catalog worth assembling from. AI acceleration amplifies whichever default we set. We should set the right one.

**The security and governance case is independent of the economics.** One classification-propagation service, one export boundary, one audit spine is a governable enterprise. Fifteen local variants is not. Even if the duplication cost were free, the risk concentration argument alone would justify the shift.

## 4.0 Capability platforms are the substrate agentic work runs on

There is a second consumer of the service layer arriving now, and it raises the stakes on everything above: AI agents. An agentic workflow is a system that decomposes a task, selects capabilities, invokes them, evaluates results, and iterates, at machine speed, without a human negotiating each step. Everything we are standing up under AIAS, the capability registry, the MCP gateway, agent orchestration, assurance and evaluation, presumes that agents have capabilities worth invoking. The common service layer is that substrate. Without it, the agent fabric is an engine with nothing to drive.

**An agent is the ultimate stranger.** The adoption test asks whether a team that has never contacted the owning team can adopt a service without a meeting. An agent is that consumer taken to its logical limit. It cannot attend a meeting, file a ticket to a person, ask a colleague what the undocumented field means, or absorb tribal knowledge over coffee. It can only consume what is explicit: a machine-readable contract, self-service credentials under a governed authorization model, documented errors, and a discoverable registry entry. A service that fails the adoption test for humans is simply invisible to agents. The standard we wrote for human strangers is the minimum bar for machine consumers, which means every dollar spent productizing is simultaneously a dollar spent on agent readiness. They are not two initiatives.

**The productization standard is, requirement by requirement, the agent-consumability standard.** This is not a loose analogy. Each property maps directly:

| Productization requirement | What it means for an agent |
|---|---|
| Machine-readable contract (OpenAPI, protobuf) | The contract is the tool definition. It is what gets registered, surfaced through the MCP gateway, and reasoned over during planning |
| Documented idempotency and retry semantics | The difference between an agent that can safely retry autonomously and one whose retry duplicates a transaction. For autonomous invocation this is a safety property, not documentation hygiene |
| Stable, versioned error contract | Agents recover from failures by reasoning over them. Undocumented or shifting errors turn recoverable failures into hallucinated handling |
| Published SLOs and status signals | What an orchestrator plans against: timeout budgets, fallback selection, routing around a degraded dependency instead of discovering it through silence |
| Semantic versioning and deprecation policy | Agent workflows are built against contracts and break silently when contracts drift. The notice period protects fleets of workflows, not just consuming teams |
| Registry discoverability | The registry is the agent's capability catalog. For a human it aids discovery; for an agent it is the entire universe of what exists |

**Agents make the duplication problem ungovernable, not just expensive.** With human teams, fifteen local identity implementations is waste and risk. With agents in the loop, it is worse: there is no coherent way to bound what an agent may do, audit what it did, or propagate classification through what it touched when the same capability exists in fifteen bespoke variants. Governable agentic work requires exactly what this strategy delivers: one common service layer, consumed through one mediated gateway, with identity, classification propagation, and the accountability chain implemented once. The THEMIS accountability model assumes this shape. A common service layer is not merely helpful for agent governance; it is the precondition for it.

**The default we set gets amplified either way.** Agents are also prolific builders. Pointed at a task with no catalog to draw from, an agentic development workflow will generate its own local service layer faster than any human team ever duplicated one, multiplying today's problem at machine speed. Pointed at a productized catalog, the same agent assembles instead of building, exactly as we want application teams to. Agents do not change the strategic choice; they raise its stakes in both directions and shorten the time we have to make it.

**And the payoff runs forward.** The quick-reaction application assembled by a human team in three weeks is the intermediate state. The end state this substrate enables is workflows assembled at task time: an agent that composes ingest, translation, entity resolution, and publication into a mission answer in minutes, under governance, because every one of those capabilities was a product with a contract it could discover, invoke, and trust. That end state cannot be retrofitted onto a portfolio of application-locked capabilities. It can only be built on a service layer that was productized from the start.

## 5.0 What the end state looks like

**For an application team, day one looks like this.** The team shops the service registry: every enterprise capability listed with its owner, contract, maturity, and status. They obtain credentials through a self-service flow the same day. They make their first successful call against each service they need within [target] minutes of opening its documentation, starting from a maintained reference client rather than a blank file. They stand up their skeleton from the shared app shell with identity and classification already wired. Their first weeks are spent on mission logic, because the plumbing was someone's product, not their project. A three-month build becomes a three-week assembly. That is not a metaphor; it is the target.

**For a platform team, the deliverable changes.** They ship products, not components. A capability is done when it meets the productization standard for its tier: stable contract, self-service onboarding, reference implementation, operational SLOs, named ownership, and an honest maturity declaration. Their roadmap is driven by consumer demand visible in the registry, their success is measured in adoptions and time-to-first-call, and their consumers are teams they have never met, by design.

**For the enterprise, the portfolio inverts.** Capability platforms [Foundation, AIAS, LPS] own the common service layer and carry the majority of durable engineering investment. Applications, flagship and quick-reaction, become thin, fast, and numerous, because everything below the mission logic line is consumed rather than built. Vendor deliveries are accepted against the productization standard, so what an OTA produces is adoptable on arrival. The catalog is simultaneously the acquisition structure and the application builder's menu.

**And critically, honesty is built into the model.** Services declare their maturity plainly: Experimental, Beta, Productized, or Sunset. A consuming team reads the label and decides with open eyes. A catalog that overstates readiness would collapse trust in the whole strategy; a catalog that is honestly incomplete still accelerates every team that uses it.

## 6.0 The mechanism: the productization standard

The shift is enforced by one standard, governed by one test:

**A team that has never contacted the owning team can adopt the service in production without a meeting.**

Everything else is mechanics in service of that test. The standard [defined in full in the companion Productization Standard document] requires six properties of every shared capability: a stable versioned contract with a deprecation policy, self-service onboarding, a maintained reference implementation, an operational contract with published SLOs and degraded-mode behavior, named ownership with a support channel, and an honest maturity declaration in the registry.

Two design choices keep the standard from becoming bureaucracy:

- **Proportional weight.** Not everything reusable is a running service. Three tiers, productized service, shared library, and reference pattern, fulfill the reuse rule at three costs. The owner picks the tier by how much genuinely varies between consumers. Gold-plating a two-consumer library is the same failure as under-productizing a broadly consumed service.
- **The test outranks the checklist.** Where a genuine constraint blocks a specific requirement, an air-gapped enclave, a classified sandbox, the owner records a public waiver with a compensating measure. Waivers exist for real constraints, not for skipping the work.

And the rule that puts the standard into force: **if more than one application may need a capability, or any quick-reaction application may need it, it shall be delivered as a product.** Building it is not sufficient. Productizing it is the requirement.

## 7.0 How we get there

This is a transition, not a flag day. Three moves, in order of leverage:

**Move 1: Change the acquisition boundary now.** Insert the productization acceptance requirement into every platform OTA at drafting. From that point, vendor deliveries either meet the standard or are accepted only as declared Beta with gaps enumerated. This is the cheapest move on the list and the most consequential, because it redirects money already being spent.

**Move 2: Productize what already exists before building anything new.** Several foundation capabilities [identity, classification propagation, retention, storage, search] function today. They are components, not products. Wrapping them in the standard, one contract surface, registry presence, self-service onboarding, is mostly contract discipline and integration, not new construction, and it seeds the catalog with the services every application needs first. The registry itself [the Interop effort is the candidate] stands up in this move, because a catalog nobody can find is the old world with extra steps.

**Move 3: Change the application default.** Once the catalog has real products in it, the build-vs-consume default flips. New applications justify building locally rather than justifying consuming. The first two or three applications assembled under the new model are run deliberately as proof points, instrumented for assembly time, and their friction reports feed directly back into the standard. The shared app shell and UI component library land in this move, because the fastest backend catalog still leaves teams rebuilding frontend plumbing without them.

**What we deliberately do not do:** pause application delivery for a platform re-architecture, mandate migration of existing applications onto the service layer on day one, or build speculative services ahead of demonstrated demand. Existing applications migrate opportunistically, at natural touchpoints. The catalog grows by extraction and productization of things applications actually need, proven by the fact that multiple applications built them.

## 8.0 How we will know it is working

The strategy stands or falls on a small number of measures, all consumer-side:

- **Time-to-first-call:** from opening a service's documentation to a successful production-path call, per service. Target: under [X] minutes. This is the direct measure of the adoption test.
- **Application assembly time:** concept to fielded for quick-reaction applications. The three-month-to-three-week trajectory is the headline claim; we instrument it and report it honestly.
- **Adoption without contact:** the fraction of new service adoptions completed with zero meetings with the owning team. This is the adoption test measured in the wild.
- **Duplication trend:** count of local reimplementations of catalog capabilities in new applications. It should approach zero for Productized-maturity services and its exceptions should each carry a recorded justification.
- **Catalog honesty:** consumer-reported gaps between declared maturity and actual experience. A rising number here is a five-alarm signal, because the entire model runs on catalog trust.

Vanity measures we will not use: number of services in the catalog, lines of platform code, or number of OTA deliverables accepted. A large catalog of unadoptable services is the failure state, counted proudly.

## 9.0 What we are asking leadership to decide

1. **Adopt the rule and the standard** as SDO policy: shared capabilities are delivered as products, to the productization standard, at the declared tier.
2. **Approve insertion of the acceptance requirement** into the platform OTAs at drafting, before signature.
3. **Charter the Foundation platform** as the home for the substrate services no existing OTA can own, and direct the incorporation of the existing foundation services under one product surface.
4. **Set the numeric targets once, centrally:** onboarding time, first-call time, deprecation notice period, concurrently supported versions.
5. **Name an owner for the registry and for the shared UI component library,** the two highest-leverage currently-unowned items in the strategy.

---

*This is a draft internal strategy document for review. The productization standard referenced throughout is defined in full in the companion document, and the platform and service catalog assignments are maintained separately.*
