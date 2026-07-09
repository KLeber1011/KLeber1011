SD-WAN Reference Architecture for a Regional Retail Chain


Reference architecture / design exercise.
This is a greenfield architecture study, not a client deliverable. The scenario
is modeled on the public profile of a regional São Paulo supermarket chain
(Barbosa Supermercados — ~35 stores across 14 municipalities, three store
formats) used purely as a realistic sizing reference. No internal data,
topology, or confidential information from any company is used. All numbers are
illustrative, derived from public information and standard market practice.




1. The problem

A regional supermarket chain operates ~35 stores across the São Paulo
metropolitan area and interior, plus a central administrative site and
distribution center. The chain is growing toward ~45 stores. Each store today is,
in practice, a small edge datacenter: point-of-sale (POS) terminals,
self-checkout, card acquiring, in-store ERP client, IP surveillance, digital
signage, and back-office.

The recurring pain in this segment is well documented in the market: stores are
connected with commodity broadband contracted per-store on price alone, with no
redundancy strategy, no traffic prioritization, and no central visibility. When a
link degrades, POS and card acquiring — the transactional heart of the store —
compete for bandwidth with surveillance and digital signage. Five minutes of POS
downtime in a busy 24h store is lost revenue that no SLA on paper recovers.

The architecture question: how do you connect ~35 heterogeneous stores so
that transactional traffic is always protected, cost stays proportional to store
criticality, and the whole estate is centrally visible and scales cleanly to 45?


2. Design principles

These principles govern every decision that follows. Where a decision record
(ADR) applies, it is linked.


Criticality drives spend, not uniformity. A 1,000 m² neighborhood store
and a 2,300 m² 24h flagship should not receive identical connectivity. The
estate is segmented into tiers. → ADR-0002
Transaction traffic is sacred. POS and card acquiring always take priority
and always have a fallback path. Everything else can degrade gracefully.
→ ADR-0003
Hybrid transport, not single-vendor lock-in. The design mixes dedicated
links, broadband, and 4G/5G, and uses SD-WAN as the intelligence layer that
selects the path per application in real time.
→ ADR-0001
Diverse last mile is real redundancy. Two links from the same physical
infrastructure (same provider, same pole) are not redundancy. Path diversity
is verified, not assumed. → ADR-0004
Central visibility is a first-class requirement. The chain must see every
store's link health, SLA compliance, and security posture from one pane of
glass — this is what turns "connectivity" into "infrastructure as a managed
asset."



3. Current state (As-Is)

The typical starting point for a chain of this size, based on common market
practice:

AspectTypical As-IsTransportCommodity broadband, one link per store, contracted per-store on priceRedundancyNone, or informal (a 4G router someone plugs in during an outage)Traffic policyNone — POS, acquiring, CCTV and signage share one flat pipeWAN managementPer-store, no central console; failures reported by the store calling ITSecurityProvider-grade CPE, no consistent segmentation between POS and guest/otherVisibilityReactive: IT learns of a problem when the store stops selling

This As-Is is not a criticism of the operator — it is the rational outcome of
growing store-by-store. The architecture's job is to impose a model on an estate
that grew organically.


4. Target state (To-Be)

An SD-WAN overlay unifies the estate. Each store runs an SD-WAN edge
appliance that terminates two or more transport links and applies centrally
defined policy. The administrative site / distribution center hosts the
concentration point and the central ERP; store traffic to the ERP and to card
acquirers is steered per-application.

See the topology diagram: diagrams/topology.md

Traffic steering (summary):


POS / card acquiring → primary path (dedicated or best broadband), automatic
failover to secondary, then to 4G/5G. Never allowed to be starved.
In-store ERP / inventory → primary path, failover to secondary.
IP surveillance → secondary/broadband path, bandwidth-capped so it can never
crowd out transactions.
Digital signage / guest Wi-Fi → lowest priority, on broadband, isolated in
its own segment.



5. Store tiering and sizing

The estate is not 35 identical sites — it is three archetypes plus the hub. This
is the core of the design. Full rationale in
ADR-0002.

TierStore profileApprox. countPrimary transportSecondaryTertiaryTarget availabilityHubAdmin site + distribution center1Dedicated (redundant)Dedicated (diverse)4G/5G99.9%T1 — FlagshipLarge, 24h, ~2,300 m², ~29 POS + self-checkout~6Dedicated linkBroadband (diverse ISP)4G/5G99.9%T2 — StandardMedium, 1,000–1,600 m²~18Business broadbandBroadband (diverse ISP)4G/5G99.5%T3 — NeighborhoodSmall, <1,000 m²~10Business broadband4G/5G—99.0%

Counts are illustrative and sum to ~35, scaling to ~45 as the chain grows.

Why tiering matters commercially: applying T1 redundancy to all 35 stores
would waste budget on low-traffic sites; applying T3 economics to a 24h flagship
would risk revenue on the busiest sites. Tiering aligns cost to risk — the
argument a Head of IT can take to a CFO.


6. Conceptual bill of materials (BOM)

Vendor-neutral, at the capability level. A real project would map these to
specific SKUs after a proof of concept.

ComponentRoleNotesSD-WAN edge (per store)Terminates links, applies policySized per tier; T1 with higher throughput / dual-WAN + LTESD-WAN edge (hub, HA pair)Concentration + policy enforcementHigh-availability pair at the admin/DC siteCentral orchestrator / controllerSingle pane of glass, policy, telemetryCloud-hosted or on-prem per operator preference4G/5G modulesTertiary failoverOn all tiers except T3-optionalSegmentation policyIsolate POS / ERP / CCTV / guestEnforced at the edge, defined centrally


7. Rollout approach

A 35-store estate is not migrated at once. The sequence protects revenue:


Hub first. Stand up the HA pair and orchestrator at the admin/DC site.
One pilot per tier. Prove the T1, T2 and T3 templates on one store each
before scaling — this is where the design is validated against reality.
Wave migration by tier, T3 → T2 → T1, keeping the old link live during
cutover and validating POS + acquiring before decommissioning.
Template the new-store build. Every one of the ~10 stores still to open
(35→45) ships with the tier template pre-configured — the model scales without
redesign.



8. Risks and mitigations

Full register: RISKS.md. The three that most shape the design:


False redundancy (shared last mile). Two links on the same physical path
give no protection. Mitigation: verified path diversity per store, 4G/5G as
independent tertiary. → ADR-0004
POS starvation under congestion. Without QoS, a flat pipe lets non-critical
traffic crowd out transactions. Mitigation: application-aware steering with POS
guaranteed. → ADR-0003
Provisioning lead time. Dedicated circuits can take weeks. Mitigation:
broadband + 4G/5G as day-one connectivity, dedicated added where the tier
justifies it, never blocking store opening.



9. What this exercise demonstrates


Segmenting a heterogeneous estate by criticality rather than treating it as
uniform.
Translating a business constraint (protect revenue / control cost) into a
technical policy (application-aware steering + tiered redundancy).
Documenting decisions as ADRs, so the reasoning — not just the result — is
reviewable.
Designing for scale (35 → 45) from the start.



Repository contents

sdwan-retail-reference-architecture/
├── README.md                  ← this file
├── RISKS.md                   ← full risk register
├── diagrams/
│   └── topology.md            ← network topology (Mermaid)
└── decisions/
    ├── ADR-0001-transport-sdwan-vs-mpls.md
    ├── ADR-0002-store-tiering.md
    ├── ADR-0003-qos-and-application-priority.md
    └── ADR-0004-last-mile-diversity.md


Author: Kleber Nascimento — Infrastructure & Solutions Architect.
Reference architecture for portfolio purposes. Scenario inspired by the public
profile of a regional retail chain; contains no confidential or client data.
