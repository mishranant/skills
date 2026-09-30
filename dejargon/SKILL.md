---
name: dejargon
description: >-
  Rewrite jargon-dense internal / organization-specific text into plain but
  still-technical engineering language that an engineer new to the industry (or
  new to the organization) can follow — without dumbing down the substance.
  Resolves internal acronyms, service and tool names, team names, and process
  terms using whatever resolution channels are available (the org's
  glossary/term-lookup, service catalog, internal docs/wiki, linked chat
  threads, code search), then produces a de-jargoned rewrite with each term
  glossed inline (no separate glossary table). Use when handed a specific
  jargon-heavy source to rewrite — asked to "dejargon", "de-acronym this",
  "translate this internal text", "make this thread/doc readable for a
  newcomer", or "rewrite this in plain terms". For teaching a topic from first
  principles when there is no specific source whose every fact must be
  preserved, use the `teach` skill instead.
user-invocable: true
---

# Dejargon

Turn insider-dense text into **plain but still-engineering** prose. The target reader is a
competent engineer who knows general software (Terraform, TLS/certs, CI/CD, Kubernetes,
SQL, networking) but has **zero organization-internal context** — no idea what your
internal platform, work-tracker, deployment cell, or in-house provisioning tool is.
De-jargon; do not dumb down. Every technical fact, number, and causal step from the source
must survive.

Two failure modes to avoid in equal measure. **Under-explaining:** expanding acronyms and
calling it done, leaving prose that parses but teaches nothing — including the subtler form
where every individual cog is glossed but the reader still can't see the machine the cogs sit
in (the pipeline, the request flow, the architecture the source assumes). **Over-reaching:**
inventing detail, motivation, or everyday-object analogies to make it feel explained. §5 exists
to get the teaching without the invention: use real teaching devices, keep every one of them
traceable, and route what you don't know instead of filling it in.

## Inputs

The text to rewrite comes from one or more of:
- **Inline text** in the prompt / `$ARGUMENTS`, or the message the user is pointing at.
- **A chat permalink** (Slack, Teams, etc.) → read the full thread and use it as the source.
- **A doc link** (Confluence, Notion, Quip, Google Doc, a wiki page) → fetch and parse it.

If no source is obvious, ask the user for the text or link before doing anything else.

## Workflow

### 1. Gather the source
Pull the raw text. For a chat link, read the whole thread (parent + replies) so context
isn't lost. For a doc link, fetch and parse the body.

### 2. Extract jargon candidates
Scan the source and list terms a newcomer would not know:
- **Acronyms** — internal initialisms, product/system short names, etc.
- **Internal service / tool / module names** — in-house platforms, provisioning tools,
  reusable modules/add-ons, monitoring and log systems referred to by codename.
- **Team / org / codename references** — scrum teams, org names, project codenames.
- **Internal process names** — release/promotion processes, change-approval processes,
  merge/deploy rituals.
- **Config / path idioms** — repo-specific file names, config-key paths, module ids.

Skip terms that are standard industry knowledge — don't gloss `TCP` or `git rebase`.

### 3. Take stock of resolution channels — and handle their absence UP FRONT

Before resolving anything, figure out **which resolution channels you actually have** in
this environment. Don't assume any internal tooling exists — a public/portable install of
this skill often has none of it.

**Always available (need no special tooling):**
- The **source text itself** and any threads/docs it links (fetch chat permalinks and doc
  URLs directly).
- `references/glossary.md` — the seed glossary in this skill (a fill-in template; may be
  empty).
- Your **general engineering knowledge** for genuinely industry-standard terms.

**Available only if the environment provides them (check, don't assume):**
- An **org glossary / term-lookup** tool or MCP.
- A **service / component catalog** (definition lookups).
- **Internal doc / wiki search** (Confluence, Notion, …).
- **Chat search** (Slack, Teams, …).
- **Code search** for code-level identifiers.

**If the channels you'd need are missing, decide up front — do not silently guess:**
1. **Triage the candidate list against what you have.** Resolve everything the available
   channels + the source + the seed glossary + common knowledge can cover.
2. **If the *core* terms (the ones the text can't be understood without) are unreachable**
   with the channels you have, **stop and ask up front** rather than producing a
   half-resolved rewrite. Give the user a concrete, either/or choice:
   - **Connect a source** — "I can resolve these automatically if you connect an org
     glossary / doc-search / chat-search MCP (or point me at your glossary URL). Want to
     set that up?", **or**
   - **Supply the definitions** — a single **pinpointed ask** that lists exactly the terms
     blocking the rewrite and, for each, the smallest thing that would resolve it (a
     one-line meaning, a glossary link, a doc, or the relevant chat thread).
3. **If only a few peripheral terms are unreachable**, don't block: produce the rewrite,
   leave those terms as-is, and list them under `## Unresolved` with the same pinpointed
   "here's what would resolve each" framing.

Make the ask **specific and batched** — one message naming every term you need, each with
what would unblock it. Never send the user on a vague "can you explain the jargon" round
trip, and never invent a meaning to avoid asking.

### 4. Resolve each candidate (confidence over completeness)
Check `references/glossary.md` first for stable, common terms. For anything not there or
ambiguous, resolve using the available channels in this order and **stop at the first
confident hit**:
1. **Org glossary / term-lookup** — the fastest source for internal acronyms/terms, if you
   have one.
2. **Service / component / domain definitions** — for a named service or component, look up
   its definition in your org's service catalog / registry.
3. **Internal docs / wiki** — actively **search the internal doc corpus** for the term,
   then parse the top hit for its definition.
4. **Chat search** — actively **search team chat** for the term to find where it's defined
   or used in context; and if a chat thread was supplied as input, read it in full and use
   its surrounding context.
5. **Code search** — for code-level identifiers, as a last resort.

Record each resolution as *(expansion, one-line plain meaning, source URL)*. **If nothing
resolves it confidently, mark it unresolved — never invent a meaning.** (See §3 for what to
do about unresolved terms.)

### 4.5 Calibrate scope: cogs vs. the machine

Glossing terms is necessary; it is not always sufficient. Internal terms are usually *cogs in
a machine the reader can't see* — a stage in a pipeline, a call in a protocol, a role in a
service topology, a state in a state machine. Glossing each cog in isolation can leave a reader
who now knows every word and still cannot picture how the thing works. Decide, before
rewriting, which case you're in:

- **Self-contained** — the source's own text carries enough structure that inline glosses make
  it followable. Go straight to the rewrite. Default for short text.
- **Cogs-in-a-machine** — the terms only resolve against an enclosing system the source assumes
  but never lays out. Here, term-by-term glossing under-serves the reader by design.

For the cogs-in-a-machine case, do one of two things — never silently default to bare glossing:

1. **Build the frame from resolved sources.** Produce the bigger picture — a diagram, an ordered
   stage list, a request/data flow, a topology — pulled from docs / code / definitions and cited
   as resolved content per §5a (kind 2). Lead with the frame, then place the glossed terms inside
   it. Anything you infer about structure the sources don't state is marked illustrative per §5a.
2. **Offer it** when building the frame is heavy, needs tools the user may not want spent, or you
   are unsure they want that altitude. Ask in one line — "do you want me to work out the bigger
   picture these terms sit in (the pipeline / flow / architecture) from the docs and code, or
   just gloss the terms as they appear?" — then follow their answer. Don't disappear into deep
   research the user didn't ask for, and don't bury them in cogs when they wanted the machine.

The frame is governed by the whole provenance regime exactly like the prose is: it is resolved
content or marked scaffolding, never invented structure. **A fabricated architecture or a made-up
stage ordering is the most authoritative-looking hallucination this skill can emit** — a diagram
reads as ground truth. Ground every box and edge, or mark it illustrative, or leave it out.

### 5. Rewrite

Teaching devices — worked examples, problem-ladders, question-beats, comparisons — are
**allowed and encouraged**, because a rewrite that is merely de-acronymed is still
unreadable. They are also the only place hallucination can enter. So every one of them is
governed by §5a. Read §5a before using any device in §5c–§5d.

#### 5a. The provenance rule (governs everything below)

Every sentence in the output is exactly one of four kinds, and the reader must be able to
tell which without asking:

1. **Source content** — restated from the source text. Default; unmarked.
2. **Resolved content** — from a lookup (glossary, catalog, doc, chat, code), carrying its
   raw URL.
3. **Scaffolding** — a device *you* wrote to make 1 and 2 followable: a worked example, a
   comparison, a framing question. **Visibly marked, every time.**
4. **Industry context** — a standard, non-organization-specific engineering fact that
   supplies motivation the source omits. **Quarantined in a labeled block; never inline.**
   Governed by §5a-2.

**The scaffolding test — apply to every device before you keep it:**

> If the reader took this sentence as a literal statement about the system, would they now
> believe anything the source and lookups do not establish?

- **No** → it's safe; keep it.
- **Yes** → mark it (kind 3), quarantine it as industry context (kind 4), or cut it. Never a
  fifth option.

**Marking conventions** (pick the lightest one that removes the ambiguity):
- Inline: `— illustratively, <device>` or a trailing `(illustrative)`.
- Block: a `> ` blockquote led by `**Worked example (illustrative).**` or
  `**Comparison (illustrative).**`.
- Fabricated identifiers are obviously fake: `svc-a`, `cell-3`, `v1.4.0`, `team-x`. Never a
  plausible-looking real service name, ticket id, hostname, date, or metric value — a reader
  who skims must not be able to mistake invented values for real ones.

**Never scaffold these, under any marking:** a number, a threshold, a version, a timeline, a
root cause, an owner, a failure mode, or an ordering the source does not state. Scaffolding
makes existing facts legible. It does not supply missing ones.

#### 5a-1. Routing the missing "why"

A mechanism explained without its motivation does not stick, and most sources are
mechanism-only. So the "why" has a route rather than a ban. **Take the first row that
matches — do not use a lower row when a higher one is available, and do not stack them.**

| What you have | Where the "why" goes |
|---|---|
| The source or a lookup states the motivation | Plain prose, unmarked. Lead the section with it (§5c). |
| The source doesn't, but the mechanism it describes **checkably matches** a standard industry pattern | **Industry context** block (§5a-2). |
| No identifiable pattern, but the motivation is **load-bearing** — the reader can't act on the text without it | **Targeted hypothesis** in `## Unresolved` (§5a-3). |
| The motivation isn't load-bearing | **Nothing.** State the mechanism and move on. |

That last row is load-bearing itself. Without it every unmotivated mechanism in the document
collects a hedged block and the output becomes a swamp of caveats. Silence is the correct
output more often than not.

**The surviving ban, scoped:** never reason backwards from a mechanism to an
**organization-specific** why — this team's incident, this team's constraint, this team's
history, this deadline, this customer. The industry can supply why a *class* of thing
exists. It can never supply why *these people* built *this one*. That distinction is the
whole firewall; when unsure which side a motivation falls on, it is org-specific.

#### 5a-2. Industry context blocks

Permitted when the source describes a mechanism but omits its motivation, **and** that
mechanism is a recognizable instance of a standard industry pattern — circuit breaker, dead
letter queue, API gateway, rate limiter, write-ahead log, read replica, bulkhead, cache
aside, and so on.

**The recognition threshold — the part that actually fails.** The block quarantines the
*motivation*. It does not quarantine the *pattern identification*, and misidentification is
the real failure mode: a labeled, well-formatted, confidently-sourced explanation pointing
the wrong way is still wrong, and reads as more trustworthy than a guess. So:

- Claim the pattern **only from behavior the source itself describes**, in enough detail
  that a reviewer can check the mapping against the source's own words. A name that merely
  sounds like a pattern is not a match.
- **Name the specific pattern.** Never hedge into unfalsifiable generic-plural — "proxies of
  this type are typically introduced to…" cannot be checked or rejected, so it launders a
  guess into prose. Name it so a reviewer can say "no, it isn't that."
- **State the identification as a claim and ask for confirmation.** The mapping is the thing
  being asserted; make it rejectable.
- One block. Never more than a few sentences. Never the spine of a section.

Format:

> **Industry context (not from the source).** The source doesn't say why `Gorgon` exists.
> The behavior it does describe — per-client request budgets, shed on breach — matches a
> rate-limiting proxy, which is standardly introduced to stop one aggressive client from
> cascading failure into shared downstreams. Confirm the mapping.

#### 5a-3. Targeted hypotheses

When motivation is load-bearing and no pattern matches, do not invent one and do not leave a
passive hole. Log it in `## Unresolved` as a question aimed at someone who can answer it.

- **Question form with named candidates**, never a bare declarative. A declarative hypothesis
  sitting in a list gets skimmed once and absorbed as fact a week later. A two-or-three-way
  choice cannot be misread as settled and forces a decision.
- **Carry the resolution path**, matching the convention §3 already uses for unresolved
  terms: the smallest thing that would settle it.
- Only for motivations the text actually depends on. Curiosity is not load-bearing.

```
- Motivation for the nightly cell rebalance — the source doesn't state why it's needed.
  Confirm which: mitigating memory growth in long-lived nodes, or rebalancing regional
  spend? Would resolve with: the design doc, or whoever owns the rebalance job.
```

**If this skill is ever run non-interactively** (batch, pipeline, no addressee), targeted
hypotheses have no one to answer them and become noise. In that case suppress them and list
the motivation as plainly unresolved instead.

#### 5b. How to gloss a term

**Gloss each term exactly once, inline, the first time it appears — then reuse the short
form.** Do not re-explain it, and do not also collect the terms into a separate glossary
table; the definitions live in the prose. One explanation per term, in place.

- **Gloss by function, not by expansion.** Expanding the letters is not a definition. The
  gloss must say what the thing *does* or *what problem it exists to solve* — that is what
  makes the surrounding sentence parseable.
  - Weak: `BOM (bill of materials)` — the reader is no better off.
  - Strong: `BOM (bill of materials — the pinned set of module versions promoted together,
    so every cell in a wave gets identical code)`.
  - For a process: name its input and its effect — `the secrets-provisioning step (reads
    the module's declared secret paths and writes credentials into the secrets manager)`.
- **Earn the introduction: define at first load-bearing use, in dependency order.** If term
  B's gloss depends on term A, A must already be glossed. Walk the source's own causal
  chain and introduce each term at the point where the reader first needs it to follow the
  next sentence — never in an upfront "background on the terms" dump, and never as a
  forward reference ("explained below").
- **One name per thing, for the whole document.** Pick the short form at first gloss and use
  that exact string every time after. Never synonym-drift (`DC` → "the cell" → "the
  instance" → "the regional deployment"). If the source itself drifts, normalize to one name
  and say so once.
- **Disambiguate lookalikes.** When two resolved terms are easily confused — similar names,
  overlapping scope, or the source uses them near each other — add one clause distinguishing
  them at the point the second one appears (`deployment cell` = one running copy;
  `deployment wave` = the ordered group of cells promoted together). Only from resolved
  sources, never invented.
- **Don't gloss what isn't load-bearing.** If an internal name is mentioned in passing and
  the reader does not need it to follow the text, leave it and mark it as skippable rather
  than derailing into a definition: `Rivet (an internal tool; not needed to follow this)`.
  Spend the reader's attention on terms the argument actually rests on.

#### 5c. How to structure the explanation

**Build in dependency order.** Before writing, list the few concepts the reader actually needs
and the order they depend on, then introduce each one only at the point the next sentence needs
it. If concept B can't be understood without concept A, A comes first. This is the §5b term rule
("define at first load-bearing use, in dependency order") applied to the whole explanation, not
just to single glosses: no front-loaded background dump, no forward references ("explained
below"), and when the dependency chain and the source's own paragraph order disagree, let the
dependency chain set the sequence (keeping every fact — §5e).

**Why before what.** A mechanism explained before its motivation does not stick. Lead a
section with the problem the thing exists to solve, then the thing. Where the motivation is
allowed to come from, and what to do when the source omits it, is the routing table in
§5a-1 — follow it rather than deciding case by case.

**Problem-ladders are a re-ordering, not an addition.** When a source presents a chain of
capabilities flatly, restructure it as: each pain the source states → the mechanism the
source says addresses it → the new pain that creates. This is the most effective device
available here *and* it is near-fidelity-safe, because every rung is source content moved,
not content invented.

A ladder asserts more than its rungs, though. It asserts *this is the shape of the story* —
so an invented rung inherits that authority even when marked, and inline marking degrades
badly inside flowing prose. Hence one exemption, tightly scoped:

- **The entry rung may borrow.** The opening "why does this class of thing exist at all" is
  usually a universal engineering pain — plaintext credentials spread across repos leak;
  hand-rolled retries stampede a recovering dependency. If the source omits it, supply it
  per §5a-1 and mark it. Cheap to correct if wrong, and it is the rung the reader most needs.
- **Interior rungs may not.** "And *that* created *this* new problem, so they built *that*"
  is a claim about **this organization's specific evolution**. No industry knowledge can
  supply it. A plausible invented middle reads as institutional history and is expensive to
  unpick later.
- **Every rung's resolution is the source's own**, not one you know from the industry.
- If an interior rung would need a pain the source never mentions, **the ladder stops
  there.** A three-rung ladder is fine. A fourth invented rung is not.

**Earn each abstraction.** The ladder is at its most useful when the "mechanisms" are the
source's successive abstractions. For each abstraction the source introduces, make four things
explicit *where the source states them*: the limitation of the approach before it, the problem
it solves, what it now hides or takes responsibility for, and the new cost, assumption, or
failure mode it adds. Never present an abstraction as simply "better" or as magic — a simpler
interface almost always moves complexity somewhere else, and the reader should be told where.
This is still the ladder, so the firewall is unchanged: each of those four is source content or
a cited lookup, and the interior-rung ban applies — if naming what an abstraction "hides" or
"costs" would require a limitation the source never states, leave it out rather than invent it.

**One running example, preferably the source's own.** If the source already works a concrete
case, adopt it and carry it through the whole rewrite rather than introducing fresh cases
per section — re-anchoring costs the reader more than it gives. If the source has none and
the material is abstract enough to need one, you may introduce **exactly one**, marked at
introduction per §5a and using obviously-fake identifiers. It stays consistent to the end.
It illustrates only chains the source asserts. It never grows a detail the source lacks.

**Question-beats at structural pivots.** State the reader's actual blocking question as a
question, then answer it immediately from source or lookup content. `Where does the wave
controller fit in? It owns the ordering: ...` Constraints:
- A question is not a claim, so this device carries no fabrication risk **provided the
  answer is sourced**. An answered-from-nowhere question is a fabrication with a question
  mark in front of it.
- Never rhetorical, never left hanging, never answered later.
- At most one per section, at genuine pivots only. Do not convert the document into a
  Socratic dialogue.
- Ban the content-free kind: `So what is X? Let's find out!` The question must be one a
  reader would actually be stuck on.

**Compress at the end of a long section.** One sentence a reader could repeat back, built
only from what the section already established. Same rule as `## In one line` in §6.

#### 5d. Anchor comparisons (the only allowed analogies)

A comparison is allowed **only when it maps an unfamiliar internal thing onto a standard
industry technology the target reader is already stipulated to know** — the list at the top
of this file: Terraform, TLS/certs, CI/CD, Kubernetes, SQL, networking, git. That is the
donor domain. Nothing else is.

- **Allowed:** `the promotion wave works like a Kubernetes rolling update — one batch at a
  time, halting the rest on failure.`
- **Allowed:** `a cell manifest is the Terraform state for one cell: the recorded current
  shape, diffed against the desired shape before anything is applied.`
- **Banned — household / physical-world mappings.** No mailrooms, no restaurants, no filing
  cabinets, no libraries, no airport security, no "think of it like a post office." The
  reader is a working engineer. Mapping a system they can already reason about onto a
  kitchen insults them and transfers nothing, because the donor domain has no mechanism to
  borrow.
- **Banned — explaining the known via the simpler.** No comparison whose donor is something
  the reader understands *less* precisely than the target. If both sides are unfamiliar, it
  is decoration.
- **Banned — ELI5 register.** No "basically it's just", no "think of it like", no "imagine
  you're a…".

Form constraints:
- **One sentence.** Never extended, never returned to, never used as a section's spine.
- **At most one per major concept**, and only for concepts that are genuinely unfamiliar —
  if the literal mechanism is already clear from §5b's functional gloss, add nothing.
- **State the break where it matters.** If the comparison misleads on a point the text
  depends on, name that in the same sentence or the next: `…like a rolling update, except
  there is no automatic rollback.` An unqualified comparison that breaks silently is worse
  than no comparison.
- **It is scaffolding.** Mark it per §5a whenever it could be read as a factual claim about
  the internal system's implementation.

#### 5e. How to write the prose

- **Lead with the finding / conclusion, not background.** State what happened or what's being
  asked in the first sentence or two.
- **One idea per sentence.** Keep a flat, predictable information rate: no clause-stacking, no
  sentence that introduces a new term *and* a new causal step *and* a caveat at once. Split it.
  The reader should never have to re-read a sentence to find where the new information was.
- **Hold one altitude per section, and announce every change of altitude.** Don't jump between
  architecture-level and line-of-config-level mid-paragraph. When the source forces a zoom,
  say so in a plain lead-in sentence.
- **Name the confusion instead of smoothing it over.** If the source uses one term for two
  different things, uses it loosely, or contradicts itself, say that plainly in one sentence
  and state which reading you used. A smoothed-over rewrite that reads cleanly but hides an
  ambiguity is worse than one that flags it.
- **No dangling threads.** Every term, pronoun, and "this" resolves in the same pass. Nothing
  is left hanging for a reader to reconstruct.
- **No run-in bold headers** like `**The problem.**` mixed into prose. Use plain sentences
  (`The bug:` / `Why:` as ordinary lead-ins are fine), or real `##`/`###` subheadings when the
  source is long. Don't decorate — the output should read as a rewrite, not a marked-up copy.
- Keep every technical fact, number, ordering, and cause→effect chain from the source.
  Simplify vocabulary, not substance.
- Preserve the source's own section/step structure where it aids reading — unless §5c's
  ladder gives a strictly clearer order, in which case re-order and keep every fact.
- Neutral, factual tone. Never condescending. The reader is a competent engineer missing
  organization-internal context, not missing engineering ability — write for someone who
  will be annoyed by being over-explained.
- If simplifying a term would make it *wrong*, keep the precise term and add a gloss instead.

### 6. Output
Emit the rewrite under a single `## Rewrite` heading — clean prose with terms glossed inline
(no separate glossary table). Only if some term couldn't be resolved, add a short `## Unresolved`
section after it:

```
## In one line
<one sentence, source-derived — include only for a long or multi-section source>

## Rewrite
<the rewritten text, terms glossed inline on first use>

## Unresolved
- <term> — couldn't resolve confidently; left as-is. Would resolve with: <one-line meaning / glossary link / doc / chat thread>.
- Motivation for <mechanism> — the source doesn't state why. Confirm which: <candidate A>, or <candidate B>? Would resolve with: <design doc / owner / thread>.
   (omit this section entirely if empty)
```

`## Unresolved` holds two kinds of entry: **terms** that couldn't be resolved (§3), and
**targeted hypotheses** about missing motivation (§5a-3). Both carry a resolution path. Both
are omitted entirely when empty.

**`## In one line`** is a compression of the source, not a summary you author. Include it only
when the source is long enough that the reader needs an anchor before the detail — a multi-section
doc, a long thread. Skip it for anything short. It must be a single sentence the reader could
repeat back, it must be derivable from the source alone, and it must not contain any claim the
source doesn't make. If you can't write it without adding something, omit the section.

## Self-check before emitting

Run the rewrite against these. Any failure means fix it, not ship it.

**Provenance (§5a) — the important pass. Do this one sentence by sentence.**
- For every sentence: is it source, a cited lookup, marked scaffolding, or a quarantined
  industry-context block? Anything that is none of the four is a fabrication — cut it.
- Could any unmarked sentence be read as a factual claim about the system that the source
  and lookups do not establish?
- Does any invented identifier look real enough to be mistaken for real?
- Did any number, threshold, version, date, owner, root cause, or ordering enter the output
  that the source did not state?
- Is any supplied "why" **organization-specific** — this team's incident, constraint,
  history, deadline, or customer? That is the firewall; it can only ever be sourced.
- For each industry-context block: is the pattern **named**, not hedged into generic-plural?
  Is the mapping checkable against behavior the source itself describes? Is it flagged for
  confirmation?
- Did the missing "why" take the **first matching row** of §5a-1, with no stacking — and did
  non-load-bearing motivations correctly get silence?
- Does any ladder borrow a pain at an **interior** rung rather than only the entry rung?

**Craft**
- Did you calibrate scope (§4.5)? If the terms are cogs in a machine the source never lays out,
  did you either build the enclosing frame from resolved sources or offer to — rather than
  silently shipping bare glosses? Is every box/edge in any frame grounded or marked illustrative?
- Could a newcomer act on each gloss, or does it only expand letters?
- Is every term defined before the sentence that depends on it — no forward references?
- Does each thing keep one name throughout?
- Does any sentence carry more than one new idea?
- Does every ladder rung's pain come from the source?
- For any earned abstraction, do its "hides" and "costs" come from the source, not from what
  you'd expect an abstraction of that kind to do?
- Is every question answered immediately, from sourced content?
- Does every comparison's donor side come from the reader's stipulated engineering
  vocabulary — and would a competent engineer find it useful rather than patronizing?
- Is every technical fact, number, ordering, and cause→effect chain from the source still
  present?

## Guardrails
- **Never fabricate a definition.** An honest "unresolved" — or an up-front, pinpointed ask —
  beats a confident guess.
- **Don't ship cogs when the reader needs the machine (§4.5).** When terms only make sense
  against an enclosing system the source assumes, build that frame from grounded sources or
  offer to — don't dutifully de-acronym parts in isolation. Any frame (diagram, stage order,
  flow) is resolved content or marked illustrative, never invented structure.
- **Handle missing tooling explicitly.** If you lack the channels to resolve the core terms,
  ask the user up front to either connect a source (glossary / doc-search / chat-search MCP)
  or supply the specific definitions — don't quietly ship a guessy rewrite.
- **Nothing unmarked that isn't source or lookup.** Teaching devices are encouraged, but every
  sentence must be traceable to the source, to a cited lookup, or to a visible scaffolding
  marker (§5a). There is no fourth category.
- **Scaffolding illustrates; it never supplies.** A worked example may make a stated chain
  concrete. It may not introduce a number, threshold, version, date, owner, root cause,
  failure mode, or ordering the source lacks.
- **Comparisons borrow from engineering, never from the kitchen.** The donor side must be a
  standard technology the reader already knows (§5d). One sentence, break stated, never
  extended. No household analogies, no ELI5 register, no "think of it like".
- **A missing "why" is routed, never guessed and never silently dropped.** Follow §5a-1:
  sourced motivation in prose, standard-pattern motivation in a quarantined industry-context
  block, load-bearing unknowns as targeted hypotheses, everything else omitted.
- **Never supply an organization-specific motivation.** The industry can explain why a class
  of mechanism exists. It can never explain why these people built this one — that is always
  sourced or unresolved. When unsure which side a motivation falls on, treat it as
  org-specific.
- Cite authoritative internal sources; paste **raw URLs** (bare chat permalinks, full PR/doc
  URLs — no markdown-wrapped or bare `#number` references).
- Don't alter technical meaning. When in doubt, keep the term + gloss.
- Don't add opinions, recommendations, or next steps unless the source contained them.
