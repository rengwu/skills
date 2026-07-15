---
name: wayfinder-prune
description: Sweep a wayfinder map — mid-charting or already charted — and settle the open questions not worth a human's attention on the model's own judgment, behind a single hard gate where the human keeps or prunes every candidate at once.
disable-model-invocation: true
---

wayfinder charts a map, and later works it, by grilling every open question along the way. wayfinder-prune is what you reach for once that starts costing more attention than it's worth — not a starting choice, but a **sweep** you run over whatever currently exists. It sorts every open question — the settled ones it can answer itself, the genuine forks and taste- or stakes-based calls it can't — and surfaces every settled one, all at once, for a single keep-or-prune review before anything is written. What you leave pruned is recorded on the model's judgment; what you pull back stays a live grilling ticket. Fewer open nodes, fewer tokens spent walking the rest — not by ceding the nitty-gritty to the model unseen, but by letting it triage and signing off on the whole cut list in one pass.

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

## The review gate

Classification produces a set of **prune candidates** — some already open tickets, some not yet written. None is recorded as resolved yet: the whole set goes to the human in **one review**, and the human rules on all of them together. Exempt and genuine-fork tickets never appear here — they are grilled live as wayfinder always grills them.

**One interruption, never a stream.** Pruning exists to spend the human's attention once — on a batch they skim — instead of one grilling exchange at a time across sessions. So the gate asks **exactly once**, every candidate listed together; asking per candidate rebuilds the very cost the sweep exists to remove.

**What each row carries:** the decision, the answer the model would settle on, and the one-breath defense for it. That is enough to fire a "no — grill this one" without re-litigating anything; it is a keep-or-prune skim, not a second grilling.

**Rank the marginal calls to the top.** A flat list invites approve-all, and an approve-all reflex turns the gate into ceremony. Order the candidates so the shakiest lead — the ones that barely cleared the one-breath test, or that lean on context the model isn't sure it has. Human attention then lands where substitution is riskiest, and the genuinely-obvious tail can be kept in bulk without pretending each got scrutiny.

**The verdict, per candidate:** *keep as pruned* — the model's answer stands, and it is recorded per the next section — or *retain* — it goes back to being an ordinary open grilling ticket, resolved live later exactly as un-pruned wayfinder would. Retaining degrades gracefully to plain wayfinder for that node.

**Nothing writes before the gate.** This is a hard pre-write step, not a courtesy heads-up: no `## Answer`, no Decisions-so-far line, no graduated fog for any candidate until the human has ruled. That is what stops a wrong prune from hardening into a premise other tickets are built on before anyone saw it.

**When the gate fires.** Sweeping a map that already exists is one batch, one gate — collect every candidate, ask once. When the sweep runs *forward* during charting, candidates **accumulate** rather than gating each as it appears; the gate fires once at a checkpoint — the end of the charting session — over the whole buffered set.

## Recording a pruned decision

Only a candidate the gate kept as pruned is recorded here; one the human pulled back becomes an ordinary open grilling ticket instead, resolved live later. For a kept candidate not yet ticketed, its `## Question` and `## Answer` are written together, in one pass. A ticket that already exists and is still open gets its `## Answer` added now — the same act that resolves any ticket, just arrived at without a live exchange, and without ever being claimed first. Either way the ticket ends up `resolved` per the adapter's derived-status rule, and never sits on the frontier. Keep `type: grilling`: it's still a grilling-shaped question, just one resolved without the live half of HITL. No new frontmatter field and no new type — the adapter stays exactly as wayfinder defines it.

Mark provenance in prose, since status and type carry no signal of *how* a ticket was resolved: open the `## Answer` with `(pruned)` before the reasoning, and suffix the ticket's line in the map's Decisions-so-far with the same tag. A session skimming the map, or you skimming it, can tell at a glance which decisions were grilled live and which the model settled for the human to ratify at the gate — and reverse any of the latter the same way an undermined decision gets reopened. Ratifying a prune at the gate is not grilling it: the human signed off on the model's call without weighing it live, so the tag stays on to record that the model authored the decision, not that the review has reopened it.

## Pre-noted sub-questions

A ticket that doesn't fully clear the test can still mix threads — one genuinely open question tangled with others that would have cleared the one-breath test on their own. Note those the moment wayfinder-prune runs, whether the ticket is being written fresh or already exists: attach the model's recommendation directly to the relevant sub-question in the ticket's `## Question`, the same recommendation a live grilling session would have offered anyway, just computed early instead of in the room.

The exemption cascades. If the ticket's own topic is itself exempt — design, tech-stack, and so on — no sub-question inside it gets pre-noted or skipped; grill the whole ticket live.

At resolution, a pre-noted sub-question is answered without asking — folded straight into the ticket's `## Answer` — **only if it's still sound**: scan the map's Decisions-so-far for anything resolved since the note was written that could undermine its premise. Nothing does — accept it, no question asked. Something might — ask it live, exactly as an un-noted sub-question would be. This is the same judgment `wayfinder` already applies when marking a resolved ticket undermined; a pre-note is just checked one step earlier, before it's ever answered rather than after.

A ticket carrying pre-noted sub-questions survived the prune test, so it never appears at the review gate — it is grilled live, and its pre-notes ride along to be answered at that live resolution. The gate rules only on whole tickets settled without grilling; a pre-note is a recommendation inside a ticket still headed for a live exchange.
