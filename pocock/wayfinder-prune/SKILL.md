---
name: wayfinder-prune
description: Sweep a wayfinder map — mid-charting or already charted — and prune it down to the decisions genuinely worth a human's attention, deciding the rest by the model's own settled judgment.
disable-model-invocation: true
---

wayfinder charts a map, and later works it, by grilling every open question along the way. wayfinder-prune is what you reach for once that starts costing more attention than it's worth — not a starting choice, but a **sweep** you run over whatever currently exists. It decides the settled tickets outright and records them, leaving only genuine forks and taste- or stakes-based calls for you. Fewer open nodes, fewer tokens spent walking the rest, at the cost of ceding the nitty-gritty implementation calls to the model's default judgment.

Read [wayfinder/SKILL.md](../wayfinder/SKILL.md) and its adapter first — map format, ticket types, fog of war, undermined decisions, and naming the destination all stay exactly as wayfinder defines them. wayfinder-prune touches only two things: which **open** tickets get a live grilling exchange versus a settled-on-the-spot answer, and — for a ticket that survives with a pre-noted sub-question inside it — whether resolution still has to ask.

## Run this on a map that already exists

Chart with plain `wayfinder`, same as always — a map's shape isn't visible until charting is underway, so there's nothing to prune before then. Reach for wayfinder-prune the moment the map (or the grilling session still charting it) starts feeling overwhelmingly big: that can be mid-charting, right after the map is committed, or later still, once you're partway through resolving it and the remaining frontier is what feels like too much.

Once invoked, it applies from that point forward to every ticket currently **open** — frontier and blocked alike, whichever stage of the process they're in — skipping only tickets already `claimed` (a session is mid-flight on those; not this sweep's business) and, naturally, anything already `resolved` or `out_of_scope`. If charting itself is still underway, the same test also governs every new candidate the rest of the session would otherwise grill.

Pruning only ever intercepts what is, or would become, a **Grilling** ticket. Research, Prototype, and Task tickets are untouched — Prototype is HITL by definition already, and Research/Task don't cost a live exchange in the first place, so there's nothing for pruning to save.

## The prune test

For each open Grilling ticket — existing or not yet created — classify:

1. **Exempt — always grill.** Matters of taste or genuine stakes, where the model substituting its own answer would mean guessing at yours: design, prototyping, tech-stack choices, material or hard-to-reverse cost (recurring spend, an infra choice expensive to walk back — not any decision with *some* footprint), and user-facing product behavior (what the product does or how it behaves for a user — not the internal mechanics of how it gets built). Just to name a few — the class is open, anchored by these, not limited to them.
2. **Genuine fork — grill.** Not exempt, but defending the choice means weighing real, competing tradeoffs pulling in different directions, with no clear winner.
3. **Settled — prune.** Everything else. The **one-breath test**: could you, right now, name the choice and defend it in a single sentence with no serious counter-argument? If yes, and it isn't exempt, prune it.

Exemption always wins: a one-breath answer inside an exempt category still gets grilled.

## Recording a pruned decision

A candidate not yet ticketed gets its `## Question` and `## Answer` written together, in one pass. A ticket that already exists and is still open gets its `## Answer` added now — the same act that resolves any ticket, just arrived at without a live exchange, and without ever being claimed first. Either way the ticket ends up `resolved` per the adapter's derived-status rule, and never sits on the frontier. Keep `type: grilling`: it's still a grilling-shaped question, just one resolved without the live half of HITL. No new frontmatter field and no new type — the adapter stays exactly as wayfinder defines it.

Mark provenance in prose, since status and type carry no signal of *how* a ticket was resolved: open the `## Answer` with `(pruned)` before the reasoning, and suffix the ticket's line in the map's Decisions-so-far with the same tag. A session skimming the map, or you skimming it, can tell at a glance which decisions were grilled and which were the model's own call — and reverse any of them the same way an undermined decision gets reopened.

## Pre-noted sub-questions

A ticket that doesn't fully clear the test can still mix threads — one genuinely open question tangled with others that would have cleared the one-breath test on their own. Note those the moment wayfinder-prune runs, whether the ticket is being written fresh or already exists: attach the model's recommendation directly to the relevant sub-question in the ticket's `## Question`, the same recommendation a live grilling session would have offered anyway, just computed early instead of in the room.

The exemption cascades. If the ticket's own topic is itself exempt — design, tech-stack, and so on — no sub-question inside it gets pre-noted or skipped; grill the whole ticket live.

At resolution, a pre-noted sub-question is answered without asking — folded straight into the ticket's `## Answer` — **only if it's still sound**: scan the map's Decisions-so-far for anything resolved since the note was written that could undermine its premise. Nothing does — accept it, no question asked. Something might — ask it live, exactly as an un-noted sub-question would be. This is the same judgment `wayfinder` already applies when marking a resolved ticket undermined; a pre-note is just checked one step earlier, before it's ever answered rather than after.
