# Incident Response

> Architecture Reference // 2026
> Cloud-native principles for any SaaS team that runs production systems.
> **8 domains · 28 rules**
> Lifecycle: DETECT · DECLARE · MITIGATE · REVIEW

---

## 01 // Foundation
**What incident response is actually for**

In complex distributed systems, incidents are inevitable. The engineering quality of a team is not measured by whether incidents happen — it is measured by how fast and how well they respond when they do.

1. **Incidents are normal. Failure to respond is not.** — High-performing teams have more incidents than poor ones — because they instrument better and declare sooner. The goal is not zero incidents. The goal is fast detection, fast mitigation, and learning that prevents recurrence.
2. **Define "incident" before one occurs** — Without a shared definition, teams waste the first 10 minutes of every incident debating whether it is one. An incident is any unplanned event that impacts customers, violates an SLO, or requires coordination across more than one person to resolve.
3. **Blameless culture is not optional** — When engineers fear blame, they delay escalation, hide information, and write sanitized post-mortems. Blame produces compliance theater. Blameless culture produces the honest timelines and systemic fixes that prevent recurrence.

---

## 02 // Severity Classification
**Severity is determined by customer impact, not system state**

A database running at 100% CPU is not an incident if users are unaffected. A 2% failure rate on checkout affecting 500 paying customers is a SEV1. Customer impact is the only axis that matters. Attach concrete criteria and response SLAs to each level — vague thresholds cause delayed escalation.

| Level | Definition | Response |
|-------|-----------|----------|
| **SEV1** | Revenue-impacting, full outage, or data loss | All hands. Acknowledge within 5 min, IC assigned within 15 min. |
| **SEV2** | Degraded but functional — significant customer impact | Acknowledge within 15 min. SLO burning fast. Workaround may exist. |
| **SEV3** | Minor impact — workaround available, SLO not at risk | Acknowledge within 1 hour. Business hours response acceptable. |
| **SEV4** | No customer impact — internal or cosmetic | Normal sprint process. No on-call page. |

---

## 03 // Roles & Command
**A committee debugging together is not incident response**

Every incident needs clear, named roles before the incident starts. When roles are invented in the moment, coordination collapses, communication stops, and the most senior engineer ends up doing everything.

1. **IC — Incident Commander — coordinates, does not fix** — The IC owns the overall response: assigns tasks, drives communication, makes scope decisions, and ensures the right people are in the room. The IC must not be in the technical weeds. A single IC per incident — no exceptions.
2. **TL — Technical Lead — investigates, reports up to IC** — The TL digs into root cause, coordinates SMEs, and executes the mitigation plan. They report status to the IC at regular intervals. When the IC and TL are the same person, both roles degrade — the IC disappears into the problem and coordination stops.
3. **CL — Communications Lead — owns status page and stakeholders** — The CL handles all external and executive communication: status page updates, customer emails, and stakeholder briefings. This role frees the IC and TL from communication overhead at the moment they need full focus on mitigation.
4. **OB — Observers go to a read-only channel** — Every extra person in the active incident channel who is not contributing adds noise and slows decisions. Executives, stakeholders, and curious engineers observe in a separate read-only channel with regular IC updates. Keep the working channel small.

---

## 04 // Communication
**Update the status page before customers tweet**

Silence during an incident causes customers to file tickets, executives to escalate, and support queues to overflow — all of which add load at the worst moment. Communication is not a post-fix courtesy. It is an active mitigation tool.

1. **Status page within 15 minutes of a SEV1/SEV2** — Even if the message is only "We are investigating an issue affecting checkout." Customers who see a status page update stop filing tickets. Customers who see nothing assume you don't know and escalate to sales, executives, and social media.
2. **Cadenced internal updates — not ad hoc** — The IC posts a structured update every 15–30 minutes during a SEV1: current status, last action taken, next action, ETA if known. Cadence prevents the management question flood that derails technical work.
3. **Pre-write customer messaging templates** — During an incident is the worst time to write from scratch. Maintain templates for: "investigating," "cause identified, working on fix," "mitigation in place, monitoring," and "resolved." Fill in the blanks — don't draft new copy under pressure.
4. **Right fidelity for the right audience** — Engineering channel gets technical detail. Status page gets user-facing impact. Executive channel gets business impact and ETA only. Technical detail sent to executives wastes their time; vague impact sent to engineers wastes everyone's.

---

## 05 // Triage & Mitigation
**Restore service first. Understand why later.**

The priority during an active incident is returning customers to a working state — not finding root cause. Understanding comes in the post-mortem. A rollback you don't fully understand is better than a fix you're still designing while users are impacted.

1. **Mitigate first, understand later** — Root cause analysis is a post-incident activity. During the incident, the only question is: what action restores service fastest? Understanding what caused the failure comes after customers are unblocked, never during.
2. **Rollback is the default recovery action** — If the last deployment correlates with the incident, roll it back immediately — before investigating. Feature flags, blue/green deploys, and canary releases exist specifically to make rollbacks fast and cheap. Maintain a deployment timeline inside every incident timeline.
3. **Kill switches for every major feature** — Feature flags that disable expensive or failing functionality in seconds are a tier-1 incident tool. When a payment provider is failing, you need to disable it without a deploy. Kill switches must be tested outside of incidents — they are useless if discovered broken during one.
4. **Time-box investigation before escalating** — If the on-call hasn't made progress toward mitigation within 30 minutes, they escalate — not continue debugging in isolation. Time-boxing forces escalation paths to be real, tested, and known. A single engineer blocking a SEV1 is an organizational failure, not a personal one.

---

## 06 // Runbooks & Playbooks
**Every alert is a question. The runbook is the answer.**

An alert without a runbook sends the on-call engineer into an incident with no map. Runbooks don't need to be long — they need to exist, be accurate, and be linked directly from the alert so they are found in under 10 seconds at 3am.

1. **Every alert links to a runbook — no exceptions** — A runbook must contain: what this alert means, the first 3 things to check, the rollback or kill switch procedure, escalation contact, and known false-positive conditions. It does not need to be comprehensive. It needs to exist and be correct.
2. **Runbooks are maintained like code** — A runbook 18 months out of date is worse than none — it sends the on-call in the wrong direction with false confidence. Runbooks go stale when systems change. Review them after every incident in which they were used, and quarterly otherwise.
3. **Game days make runbooks real** — Scheduled chaos exercises — injecting failures in a controlled environment — expose runbook gaps, broken escalation paths, and undocumented dependencies before a real incident does. Game days are not optional once you run production SaaS for paying customers.

**Every runbook must contain:** what this alert means · first 3 checks · rollback / kill switch · escalation contact · false positive criteria · related alerts

---

## 07 // Post-Mortem
**The incident ends at mitigation. The work begins after.**

A post-mortem that produces no action items is documentation theater. A post-mortem that names an individual as the root cause is counterproductive. The goal is systemic understanding that prevents recurrence — and learnings that the whole organization can use.

1. **Blameless by design** — People are never the root cause. Systems, processes, tooling, and conditions are. When engineers fear appearing in a post-mortem as the cause of an incident, they write sanitized timelines. Blameless culture is the only way to get the honest account needed to prevent recurrence.
2. **Reconstruct a complete timeline** — The timeline runs from the first observable anomaly to full resolution. It is built from logs, deploy records, alert timestamps, and chat history — not from memory. Gaps in the timeline are gaps in observability. Every gap is a follow-up action item.
3. **Every action item gets an owner and a deadline** — A post-mortem with no owners and no dates is a list of good intentions. Every contributing factor maps to a concrete remediation: a runbook update, a monitoring gap closed, a kill switch added, a process change. No owner and deadline means it does not happen.
4. **Share post-mortems across the organization** — The learnings from one team's incident are often directly relevant to three other teams running similar systems. Post-mortems should be indexed, searchable, and accessible company-wide. Tribal knowledge locked inside one team prevents the organization from learning.

---

## 08 // On-Call Health
**Sustainable on-call is an engineering requirement, not a culture choice**

A rotation that requires heroism to sustain will eventually fail. Alert fatigue, burnout, and turnover are direct, measurable costs of unsustainable on-call. On-call toil is a product defect — it belongs in the sprint backlog, not the "we'll get to it" list.

1. **Track MTTD, MTTR, and pages per engineer per week** — Mean Time to Detect and Mean Time to Resolve are the KPIs of your incident response program. Pages per engineer per week is the proxy metric for on-call health. Without tracking them, improvements are invisible and regressions go unnoticed. Publish to the engineering org quarterly.
2. **On-call rotation must be sustainable — not heroic** — Any engineer who gets paged more than 2–3 times outside business hours per week is accumulating a debt that compounds as fatigue. Rotations must be large enough to distribute load. An on-call that regularly pages at 3am is a system design problem that requires engineering, not endurance.
3. **Structured handoff at every shift boundary** — Every on-call handoff covers: active incidents and their state, recent deploys that might surface issues, known risks scheduled in the next shift, and any alerts currently suppressed or in a noisy state. An oral-only or skipped handoff is a gap that guarantees a cold-start the next time something breaks.
4. **On-call is a direct feedback loop into the product** — The engineer paged at 3am for a flaky integration has ground truth about product quality that no QA process replicates. On-call observations feed directly into sprint planning. Toil reduction is a product requirement — it improves reliability, reduces cost, and retains engineers.

---

## The mental model

> Declare early. Assign roles immediately.
> Mitigate before you understand.
> Communicate before customers ask.
> Post-mortems find system failures, not people failures.
> On-call toil is a bug. Fix it in the sprint.

---

*Daniel Brasileiro · [incident-response.w-b.dev](https://incident-response.w-b.dev)*
