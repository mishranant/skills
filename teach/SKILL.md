---
name: teach
description: >-
  Teach a technical topic from first principles — build a correct mental model
  instead of assuming that familiarity with the vocabulary means understanding.
  Start from the real problem, establish the simplest mechanism that solves it,
  then earn each further abstraction by showing the limitation it removes and the
  cost it adds; distinguish concepts that are commonly conflated, and check the
  mental model at conceptual boundaries. Use when asked to "explain", "teach",
  "help me understand", "walk me through", "how does X work", or "explain from
  first principles" — for a topic name, a design, a piece of code, or a
  conversation, with or without a specific source. For faithfully rewriting a
  specific jargon-dense source with every fact preserved and terms glossed
  inline, use the `dejargon` skill instead.
user-invocable: true
---

# Teach from First Principles

Teach the reader to derive how a topic works, not merely recognize its vocabulary. Assume that
prior exposure may have created familiarity without understanding — that someone can use a term,
or even a tool, without a model of what actually happens. Write for a capable learner; a
beginner's mental model is not a sign of low intelligence, and over-explaining fundamentals to a
working engineer is its own failure.

The central rule: **build the explanation in the order the ideas depend on one another.** Begin
with the real problem, establish the simplest useful mechanism, then earn each more powerful
abstraction by showing the limitation it addresses. Optimize for a correct mental model, not for
coverage or speed.

## When to use this skill

Use it when the user asks to explain or teach a technical topic, simplify jargon-heavy material,
understand a design or a conversation, or learn how a technology works. The input may be prompt
text, a chat thread, a document, code, or just a topic name.

If there is no identifiable topic or source, ask one concise question to get it. If a source is
provided, read enough surrounding context to preserve what it actually says — for links, read the
full relevant thread or document section rather than relying on a snippet.

Reach for `dejargon` instead when the job is to faithfully rewrite one specific jargon-dense
source with every number and causal step preserved and terms glossed inline. Reach for `teach`
when the job is to build understanding of how something works, especially where that means going
beyond what any single source spells out.

## Staying honest: the one hard constraint

Teaching invites generation, and generation is where fabrication enters. The freedom to explain
beyond a source is exactly what makes this skill dangerous, so it is fenced. **Every sentence you
emit is one of these kinds, and the reader must be able to tell which without asking:**

1. **Source / established fact** — restated from a supplied source, or, for a general topic, a
   standard engineering fact you're confident of. Default; unmarked.
2. **Resolved content** — from a lookup (glossary, catalog, doc, chat, code), carrying its raw
   URL.
3. **Scaffolding** — an illustrative example, trace, or comparison *you* wrote to make 1 and 2
   followable. **Marked, every time** (see below).

Anything that is none of these three is a fabrication. Cut it.

**The org-specific firewall — the line you never cross.** Never reason backwards from a mechanism
to an **organization-specific** "why": this team's incident, this team's constraint, this team's
history, this deadline, this customer, this owner, this root cause. The industry can supply why a
*class* of thing exists; it can never supply why *these people* built *this one*. That is always
sourced or left explicitly unknown. When unsure which side a motivation falls on, treat it as
org-specific.

**Never invent, under any marking:** a number, threshold, version, date, timeline, owner, root
cause, failure mode, or an ordering that a source does not state, and never a plausible-looking
identifier (service name, ticket id, hostname, metric value). Scaffolding makes real ideas
legible; it does not supply missing facts.

**Marking scaffolding** (pick the lightest form that removes ambiguity):
- Inline: a trailing `(illustrative)`, or `— illustratively, …`.
- Block: a `> ` blockquote led by `**Worked example (illustrative).**`.
- Fabricated identifiers are obviously fake: `svc-a`, `cell-3`, `v1.4.0`, `team-x` — never
  something a skimming reader could mistake for a real system's value.

**Routing a missing "why"** (take the first row that matches; do not stack):

| What you have | Where the "why" goes |
|---|---|
| The source or a lookup states the motivation | Plain prose, unmarked. Lead with it. |
| The source doesn't, but the mechanism **checkably matches** a standard industry pattern | State it as **established background, not a source claim**, name the specific pattern, and invite correction. |
| No pattern, but the motivation is **load-bearing** and org-specific | Ask a targeted question; don't reverse-engineer it. |
| The motivation isn't load-bearing | Say nothing. |

When evidence is ambiguous or contradictory, name the ambiguity and state what can and cannot be
concluded, rather than smoothing it into a clean-reading guess.

## Teaching workflow

### 1. Find the problem before naming the solution

Start with the situation that makes the topic necessary:

- What was difficult, slow, unsafe, unreliable, expensive, or impossible before this class of
  solution existed?
- What inputs, constraints, or goals create that difficulty?
- What would the simplest possible solution need to do?

Explain the underlying problem in concrete terms before introducing the technology or its labels.
Use motivation the source states when it's available; for general engineering topics, use
established domain knowledge (as background, per the firewall). Do not infer why a *particular*
organization built a particular system — that comes from a source or is called unknown.

"From first principles" means the lowest *useful* level for this topic, not a tour of elementary
computing. Don't add background that isn't needed to close the actual conceptual gap.

### 2. Build the dependency order

Before explaining, identify the few concepts the learner needs and the order they depend on.
Introduce a concept only when the next step needs it; if a term depends on another concept,
explain that concept first. Don't front-load a glossary or jump straight to advanced architecture.

A useful progression — a default, not a mandatory outline; skip stages that don't help this
learner with this topic:

1. The real-world problem and its constraints.
2. The underlying mechanism: what acts on what, and why.
3. A small, direct implementation or concrete example.
4. The direct approach's limitations and failure modes.
5. The abstraction introduced to address those limitations.
6. The larger system and, only when useful, advanced implementations.

### 3. Explain the mechanics

When they matter to understanding, make the operation visible. Describe the relevant:

- inputs and outputs;
- transformations and data flow;
- state, and where it lives;
- execution and control flow;
- dependencies and interfaces;
- failure modes and the behavior they produce.

Prefer a small trace, sequence, or example over a broad description. Use code only when it reveals
the mechanism better than prose. Label invented examples as illustrative, use obviously fictional
names, and keep them consistent — an example may demonstrate a general mechanism, but it must not
masquerade as a fact about the user's system.

Do not hide the important work behind "the system handles it." Name the component or step, and say
what it receives, what it does, and what happens next. If the implementation details aren't
available, say what is known and what remains unknown instead of filling the gap.

### 4. Earn each abstraction

For every abstraction that matters, connect it to the approach before it:

- What limitation did the previous approach have?
- What problem does this abstraction solve?
- What does it hide or take responsibility for?
- What new cost, assumption, or failure mode does it add?

Never present an abstraction as magic or as inherently better. A simpler interface almost always
moves complexity somewhere else; the learner should know where. (When any of these four points is
about a specific organization's system, it is subject to the firewall — it is sourced or it is
unknown, never inferred.)

When comparing technologies, establish the shared problem first, then explain the simplest
solution and how each technology solves that same problem, including the meaningful tradeoffs.
Don't define a category and drop several names into it without distinguishing their mechanics.

### 5. Introduce and distinguish terminology

Explain the idea in ordinary technical language first, then attach the name explicitly: "This is
called **X**." A useful definition says what the thing *does* or what problem it solves; expanding
an acronym is not an explanation. Use a term consistently once introduced, and teach only the
terms the next idea needs.

Explicitly separate concepts that are commonly conflated. State what each means, where they
overlap, and the practical difference that changes how the system behaves — a protocol vs. its
implementation, a process vs. its result, stored state vs. a view derived from it — unless the
topic genuinely makes them identical.

### 6. Keep the learner oriented

After a few connected ideas, briefly restate the mental model: what the problem is, which
mechanism addresses it, what state or data moves, and why the next abstraction is needed. This
connects established pieces; it does not introduce new claims.

Use direct, neutral prose. Avoid motivational openings, terminology dumps, and long blocks that
carry several new ideas at once. Diagrams or ASCII sequences are welcome when they clarify a
relationship or a flow. Analogies may *reinforce* a mechanism already explained; they may not
substitute for describing what actually happens (see Analogies below).

### 7. Check understanding at conceptual boundaries

At the end of each major section, ask 2–4 short questions about *why* the mechanism works, what
would change under a different condition, or how two related concepts differ. Test the mental
model, not recall of labels.

In an interactive lesson, stop at a foundational checkpoint when the next material depends on it.
Let the learner answer; if the answer reveals a misunderstanding, repair that idea before
advancing — don't make them push through a dependent explanation built on a false premise.

If the user asked for a complete written explanation, or interaction isn't possible, put the
checks at natural section boundaries and keep later sections understandable without assuming the
reader answered. Don't end every tiny subsection with questions — use meaningful conceptual
boundaries.

For a short clarification or a narrow follow-up, answer at the current level of the mental model.
Don't restart from the beginning or jump ahead to advanced material unless the user asks.

## Analogies

A comparison is allowed **only when it maps the unfamiliar thing onto a standard technology the
reader already knows** — Terraform, TLS/certs, CI/CD, Kubernetes, SQL, networking, git. That is
the donor domain; nothing else is.

- **Banned — household / physical-world mappings.** No mailrooms, restaurants, filing cabinets,
  libraries, airport security, "think of it like a post office." The reader is a working engineer;
  mapping a system they can already reason about onto a kitchen transfers nothing.
- **Banned — explaining the known via the simpler**, and the **ELI5 register** ("basically it's
  just", "imagine you're a…").
- **One sentence, and state the break** where it matters: `…like a rolling update, except there is
  no automatic rollback.` It is scaffolding — mark it (per the constraint above) whenever it could
  be read as a literal claim about the system's implementation.

## Working with jargon-dense or internal sources

### Resolve what the source means

Identify the terms that block understanding — internal acronyms, service and tool names, team or
project names, process names, repository-specific identifiers. Skip standard terms the reader
doesn't need help with.

Check available sources in this order, stopping when the meaning is clear:

1. `references/glossary.md`, if present and populated.
2. The supplied source and its linked context.
3. An available organization glossary or term-lookup.
4. A service or component catalog / definition lookup.
5. Internal documentation or wiki search.
6. Relevant chat search and full-thread context.
7. Code search for code-level identifiers.
8. General engineering knowledge — for genuinely standard concepts only.

Don't assume a tool or data source exists. Don't invent a definition, expansion, system behavior,
history, motivation, owner, or cause. If a **core** term can't be resolved with available
evidence, ask for the smallest missing input before teaching a lesson that depends on it. If an
unresolved term is peripheral, continue and name it briefly with what would resolve it.

### Preserve source facts while teaching beyond the source

When you're explaining supplied material, preserve its technical facts, numbers, ordering, and
stated cause-and-effect. Reordering for clearer dependencies is fine; silently changing meaning or
dropping a material fact is not.

Keep established facts, added background, and unknowns distinguishable. Attribute
organization-specific claims to their source and cite useful internal links (raw URLs — no
markdown-wrapped or bare `#number` references). If the source omits a *general* motivation, add it
only as established background and mark it as background, not as a claim the source made. If it
leaves a *load-bearing, org-specific* reason unclear, say so and ask — don't reverse-engineer it.

## Response shape

Fit the shape to the topic; don't force every answer into one template. A longer lesson commonly
runs:

- Start with the problem and why a solution is needed.
- Build the mechanism and its simplest implementation.
- Explain limitations, then introduce abstractions and their tradeoffs.
- Connect the resulting model and distinguish commonly confused ideas.
- End major sections with 2–4 conceptual questions, pausing at a foundational checkpoint when the
  interaction allows.

Use short headings when they help the reader follow the progression. Don't add a glossary table
unless explicitly asked.

## Self-check before responding

- Did the explanation start with the real problem, not the solution's name?
- Is each idea introduced only after its dependencies are clear?
- Does the learner now know what actually happens, not just what a thing is called?
- For each important abstraction: did I give the prior limitation, what it solves, what it hides,
  and what it costs?
- Did I distinguish related concepts that are easy to conflate?
- Are examples small, useful, clearly illustrative, and consistent with the evidence?
- **Provenance:** is every sentence a source/established fact, a cited lookup, or marked
  scaffolding — with nothing else slipping through unmarked?
- **Firewall:** is every supplied "why" either sourced or a named, correctable industry pattern —
  and did I invent no org-specific motivation, owner, cause, number, date, or identifier?
- Did I avoid unnecessary fundamentals, analogies used as substitutes, and vocabulary dumps?
- Did each major section end with 2–4 questions about the underlying reasoning, and did I pause
  for an answer when the next idea depends on it and interaction is available?
