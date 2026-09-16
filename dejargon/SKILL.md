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
  glossed inline (no separate glossary table). Use when asked to "dejargon",
  "translate this", "explain in plain engineering terms", "de-acronym this",
  "make this readable for a new grad / newcomer", or when handed a jargon-heavy
  chat thread, doc, or writeup to simplify.
user-invocable: true
---

# Dejargon

Turn insider-dense text into **plain but still-engineering** prose. The target reader is a
competent engineer who knows general software (Terraform, TLS/certs, CI/CD, Kubernetes,
SQL, networking) but has **zero organization-internal context** — no idea what your
internal platform, work-tracker, deployment cell, or in-house provisioning tool is.
De-jargon; do not dumb down. Every technical fact, number, and causal step from the source
must survive.

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

### 5. Rewrite
**Gloss each term exactly once, inline, the first time it appears — then reuse the short
form.** Do not re-explain it, and do not also collect the terms into a separate glossary
table; the definitions live in the prose. One explanation per term, in place.

- Inline-gloss format: `DC (deployment cell — one region/environment instance of a
  service)`, then just `DC` afterward. For a noun: `the secrets-provisioning step (writes
  credentials into the secrets manager)`.
- **Lead with the finding / conclusion, not background.** State what happened or what's being
  asked in the first sentence or two; introduce terms as they come up, not in an upfront
  "background on the terms" dump.
- **No run-in bold headers** like `**The problem.**` mixed into prose. Use plain sentences
  (`The bug:` / `Why:` as ordinary lead-ins are fine), or real `##`/`###` subheadings when the
  source is long. Don't decorate — the output should read as a rewrite, not a marked-up copy.
- Keep every technical fact, number, ordering, and cause→effect chain from the source.
  Simplify vocabulary, not substance.
- Preserve the source's own section/step structure where it aids reading.
- Neutral, factual tone. No new claims that aren't in the source or a cited lookup.
- If simplifying a term would make it *wrong*, keep the precise term and add a gloss instead.
- **No analogies, metaphors, or "think of it like…" framing.** The reader speaks engineering —
  explain the actual mechanism in literal technical terms. No mailrooms, no restaurants, no ELI5.

### 6. Output
Emit the rewrite under a single `## Rewrite` heading — clean prose with terms glossed inline
(no separate glossary table). Only if some term couldn't be resolved, add a short `## Unresolved`
section after it:

```
## Rewrite
<the rewritten text, terms glossed inline on first use>

## Unresolved
- <term> — couldn't resolve confidently; left as-is. Would resolve with: <one-line meaning / glossary link / doc / chat thread>.   (omit this section entirely if empty)
```

## Guardrails
- **Never fabricate a definition.** An honest "unresolved" — or an up-front, pinpointed ask —
  beats a confident guess.
- **Handle missing tooling explicitly.** If you lack the channels to resolve the core terms,
  ask the user up front to either connect a source (glossary / doc-search / chat-search MCP)
  or supply the specific definitions — don't quietly ship a guessy rewrite.
- **No analogies or metaphors — ever.** De-jargon by expanding and glossing the real terms, not
  by mapping them onto everyday objects. Literal mechanism only.
- Cite authoritative internal sources; paste **raw URLs** (bare chat permalinks, full PR/doc
  URLs — no markdown-wrapped or bare `#number` references).
- Don't alter technical meaning. When in doubt, keep the term + gloss.
- Don't add opinions, recommendations, or next steps unless the source contained them.
