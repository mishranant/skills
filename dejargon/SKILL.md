---
name: dejargon
description: >-
  Teach a technical topic from first principles, building a correct mental model
  rather than assuming that familiarity with terms means understanding. Use when
  asked to explain, teach, clarify, de-jargon, or translate technical material,
  especially jargon-dense internal text. Resolve organization-specific terms from
  available sources; explain concrete mechanics, abstractions, limitations, and
  conceptual distinctions, with short understanding checks at major boundaries.
user-invocable: true
---

# Teach from First Principles

Teach the reader to derive how a topic works, not merely recognize its vocabulary. Assume
that prior exposure may have created familiarity without understanding. Write for a capable
learner; do not confuse a beginner's mental model with lack of intelligence.

The central rule is: **build the explanation in the order the ideas depend on one another.**
Begin with the real problem, establish the simplest useful mechanism, then earn each more
powerful abstraction by showing the limitation it addresses. Optimize for a correct mental
model, not coverage or speed.

## When to use this skill

Use it when the user asks to explain or teach a technical topic, simplify jargon-heavy
material, understand a design or conversation, or learn how a technology works. The input
may be prompt text, a chat thread, a document, code, or a topic name.

If there is no identifiable topic or source, ask a concise question to get it. If a source is
provided, read enough surrounding context to preserve what it actually says. For links, read
the full relevant thread or document section rather than relying on a snippet.

## Teaching workflow

### 1. Find the problem before naming the solution

Start with the situation that makes the topic necessary:

- What was difficult, slow, unsafe, unreliable, expensive, or impossible before this class
  of solution?
- What inputs, constraints, or goals create that difficulty?
- What would the simplest possible solution need to do?

Explain the underlying problem in concrete terms before introducing the technology or its
labels. Use motivation stated in the source when available. For general engineering topics,
use established domain knowledge. Never infer why a particular organization built a
particular system, made a decision, or experienced an incident; that motivation must come
from a source or be called unknown.

Do not add background that is not needed to understand the actual conceptual gap. “From
first principles” means the lowest useful level for this topic, not a tour of elementary
computing.

### 2. Build the dependency order

Before explaining, identify the few concepts the learner needs and the order they depend on.
Introduce a concept only when the next step needs it. If a term depends on another concept,
explain the dependency first. Do not front-load a glossary or jump straight to advanced
architecture.

A useful progression is:

1. The real-world problem and its constraints.
2. The underlying mechanism: what acts on what, and why.
3. A small, direct implementation or concrete example.
4. The direct approach's limitations and failure modes.
5. The abstraction introduced to address those limitations.
6. The larger system and, only when useful, advanced implementations.

This is a progression, not a mandatory outline. Skip stages that do not help this learner
understand this topic.

### 3. Explain the mechanics

When they matter to understanding, make the operation visible. Describe the relevant:

- inputs and outputs;
- transformations and data flow;
- state and where it lives;
- execution and control flow;
- dependencies and interfaces;
- failure modes and resulting behavior.

Prefer a small trace, sequence, or example over a broad description. Use code only when it
reveals the mechanism better than prose. Label invented examples as illustrative, use
obviously fictional names, and keep them consistent. An example may demonstrate a general
mechanism; it must not masquerade as a fact about the user's system.

Do not hide important work behind phrases such as “the system handles it.” Name the
component or step involved and explain what it receives, what it does, and what happens next.
If the implementation details are unavailable, say what is known and what remains unknown
instead of filling in the gap.

### 4. Earn each abstraction

For every abstraction that matters, connect it to the approach that came before it:

- What limitation did the previous approach have?
- What problem does this abstraction solve?
- What details does it hide or take responsibility for?
- What new costs, assumptions, or failure modes does it add?

Do not present abstractions as magic or as inherently better. Explain the trade: a simpler
interface usually moves complexity somewhere else, and the learner should know where.

When comparing technologies, establish the shared problem first. Then explain the simplest
solution and how each technology solves that same problem, including the meaningful tradeoffs.
Do not define a category and drop several names into it without distinguishing their
mechanics.

### 5. Introduce and distinguish terminology

Explain the idea in ordinary technical language first. Then attach the name explicitly:
“This is called **X**.” A useful definition describes what the thing does or what problem it
solves; expanding an acronym alone is not an explanation.

Use a term consistently after introducing it. Do not rely on a vocabulary list to carry the
lesson. Teach only the terms needed for the next idea, and define them in dependency order.

Explicitly separate concepts that are often conflated. State what each one means, where they
overlap, and the practical difference that changes how the system behaves. For example, do
not blur a protocol with its implementation, a process with its result, or stored state with
a view derived from that state unless the topic specifically makes them identical.

### 6. Keep the learner oriented

After a few connected ideas, briefly restate the mental model: what the problem is, which
mechanism addresses it, what state or data moves, and why the next abstraction is needed.
This should connect established pieces, not introduce new claims.

Use direct, neutral prose. Avoid motivational openings, terminology dumps, generic analogies,
and long blocks that carry several new ideas at once. Analogies may reinforce a mechanism
that has already been explained; they must not substitute for describing what actually
happens. Diagrams or ASCII sequences are welcome when they clarify relationships or flow.
Do not over-explain familiar fundamentals unless the topic depends on them.

### 7. Check understanding at conceptual boundaries

At the end of each major section, ask 2–4 short questions about why the mechanism works,
what would change under a different condition, or how two related concepts differ. Test the
mental model, not recall of labels.

In an interactive lesson, stop at a foundational checkpoint when the next material depends
on it. Let the learner answer; if the answer reveals a misunderstanding, repair that idea
before advancing. Do not make the learner push through a dependent explanation built on a
false premise.

If the user asks for a complete written explanation or an interaction is not possible, include
checks at the natural section boundaries and keep later sections understandable without
assuming the reader answered correctly. Avoid ending every tiny subsection with questions;
use meaningful conceptual boundaries.

For a short clarification or a narrow follow-up, answer at the current level of the mental
model. Do not restart from the beginning or jump ahead to advanced material unless the user
asks.

## Working with jargon-dense or internal sources

### Resolve what the source means

Identify terms that block understanding: internal acronyms, service and tool names, team or
project names, process names, and repository-specific identifiers. Skip standard terms the
reader does not need help with.

Check available sources in this order, stopping when the meaning is clear:

1. `references/glossary.md`, if present and populated.
2. The supplied source and its linked context.
3. An available organization glossary or term-lookup.
4. A service or component catalog.
5. Internal documentation or wiki search.
6. Relevant chat search and full-thread context.
7. Code search for code-level identifiers.
8. General engineering knowledge for genuinely standard concepts only.

Do not assume a tool or data source exists. Do not invent a definition, expansion, system
behavior, organization-specific history, motivation, owner, or cause. If a core term cannot
be resolved with available evidence, ask for the smallest missing input before presenting a
lesson that depends on it. If an unresolved term is peripheral, continue and identify it
briefly with what would resolve it.

### Preserve source facts while teaching beyond the source

When the task is to explain supplied material, preserve its technical facts, numbers,
ordering, and stated cause-and-effect relationships. Reordering is fine when it makes the
dependencies clearer, but do not silently change meaning or omit material facts.

Separate established facts from background explanation. Attribute organization-specific
claims to their source and cite useful internal links. If the source omits a general
motivation, add it only when it is established engineering context and make clear that it is
background, not a claim made by the source. If the source leaves a load-bearing internal
reason unclear, say so and ask a targeted question; do not reverse-engineer an organizational
motive from the mechanism.

Mark hypothetical examples as illustrative. Never invent plausible-looking identifiers,
metrics, dates, thresholds, causes, or implementation details that might be mistaken for
source facts. When evidence is ambiguous or contradictory, name the ambiguity and state what
can and cannot be concluded.

## Response shape

Use a shape suited to the topic rather than forcing every answer into a rewrite template. A
longer lesson commonly follows this pattern:

- Start with the problem and why a solution is needed.
- Build the mechanism and its simplest implementation.
- Explain limitations, then introduce abstractions and tradeoffs.
- Connect the resulting model and distinguish commonly confused ideas.
- End major sections with 2–4 conceptual questions, pausing at a foundational checkpoint
  when the interaction allows it.

For a rewrite or source explanation, keep the reader's task in view: explain what the source
says, what its important terms mean, and which background is added to make the mechanics
understandable. Use short headings when they help the reader follow the progression. Do not
add a glossary table unless explicitly requested.

## Self-check before responding

- Did the explanation start with the real problem rather than the solution's name?
- Is each idea introduced only after its dependencies are clear?
- Does the learner know what actually happens, not just what a component is called?
- For each important abstraction, did I explain the prior limitation, what it solves, what it
  hides, and what complexity it adds?
- Did I distinguish related concepts that would otherwise be easy to conflate?
- Are examples small, useful, clearly illustrative, and consistent with the evidence?
- Did I avoid treating terminology or prior tool use as proof of understanding?
- Did I avoid unnecessary fundamentals, analogies as substitutes, and vocabulary dumps?
- Did each major section end with 2–4 questions about the underlying reasoning?
- If the next idea depends on a foundational concept, did I pause for an answer when
  interaction is available?
- Are source facts, general background, and unresolved organization-specific claims
  distinguishable?
- Have I preserved the source's meaning without inventing internal facts or motivations?
