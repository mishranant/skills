---
name: inside-the-vm-debug
description: >-
  Structured methodology for debugging a live issue you cannot reproduce
  locally, where the evidence lives on a system the agent cannot touch directly
  — a VM / host / pod / cluster node. Works in TWO access modes: (A) interactive
  shell, when an operator can SSH in (a non-prod box, or prod with an approved
  break-glass) and pastes command output; (B) telemetry-only, when SSH is
  forbidden (typical for prod) and the agent itself drives read-only
  observability — metrics, logs, dashboards, on-call/incident tooling, and
  config read at the deployed commit/SHA. Same spine either way: exhaustive
  read-only evidence collection, root-cause hypothesis as pointwise claims, a
  written plan, then PRE-COMMITTED tracks (shell-command tracks in Mode A, query
  tracks in Mode B) where each step is bound in advance to its expected results
  and the branch taken per result. Use for connectivity failures,
  DNS/tunnel/dataplane faults, service-down, packet loss, "control plane healthy
  but data plane black-holed", or any live incident. Triggers: "debug this
  live", "I'm on the host/box", "ssh'd into the vm", "can't ssh into prod",
  "availability metric is 0", "tunnel is up but", "give me commands / queries to
  run", "collect info first".
user-invocable: true
---

# Inside-the-VM Debug — diagnose a live issue you can't reproduce locally

You are driving an investigation on a system you cannot reach. Your job is to
extract the maximum signal per step, converge on a root cause, and hand over a
plan executable with confidence — never a guessing loop.

The method is the same regardless of access. What changes is the **evidence
channel**. Resolve that first — pick the access mode (§0.5) and confirm you can
actually reach the evidence (§0.6) — then run the same four phases:
**Collect → Diagnose → Plan → Guide**, iterated until the system tells you the
answer. Do not skip ahead to a fix.

---

## 0. Safety model — read this first

- **Read-only until the fix is agreed.** Phases 1–3 collect and reason only —
  no restarts, config writes, `iptables`, or deploys. Mutations happen only in
  Phase 4, after the plan is confirmed.
- **Never fabricate.** Every IP, commit SHA, hostname, counter, metric value,
  or interface name in your reasoning must come from a real result — pasted
  command output (Mode A) or a tool result produced this run (Mode B). If you
  need a value you don't have, go get it; do not assume it.
- **The live system is the source of truth.** Docs and dashboards describe
  intent; only a metric, log line, or command output proves current state. A
  claim is "confirmed" only when a result proves it.
- **Never read at main/HEAD as if deployed.** In Mode B, config claims must be
  read at the *deployed* commit/image discovered from telemetry. A read of the
  main branch only distinguishes "merged but not deployed" from "never
  created".
- **Treat every prior conclusion as a lead until proven.** Don't let an early
  guess harden into an assumption; re-test it if later evidence conflicts.
- **Handle missing tooling explicitly (§0.6).** If you can't reach the evidence
  channel a step needs, ask the operator up front to either connect the tool/MCP
  or relay the exact values — never invent a value to fill the gap.

---

## 0.5 Access gate — pick the evidence channel BEFORE collecting

Determine which mode you are in. When unsure, ask the operator one question:
"Can you get an approved shell on this box, or is this prod / no-SSH?"

| | **Mode A — Interactive shell** | **Mode B — Telemetry-only** |
|---|---|---|
| **When** | Non-prod, lab, or prod with approved break-glass | Prod / restricted; SSH not allowed |
| **Who executes** | The **operator** pastes command output | The **agent** runs read-only tools itself |
| **Evidence** | `ipsec status`, `dig`, `ip route`, `ovs-appctl`, counters | Metrics, logs, dashboards, on-call/incident tooling, config@commit, git |
| **Phase 4 shape** | Shell-command tracks | Query tracks |
| **Wall** | none within read-only | on-box-only actions (capture, live trace) → **escalate**, don't SSH |

**Prod default is Mode B.** Do not ask the operator to SSH into a prod box.
If the investigation hits something only obtainable on the box (packet capture,
`ofproto/trace`, a live `tcpdump`), that is the **escalation boundary**: either
(a) hand the owning team an evidence package to run it, or (b) if break-glass is
genuinely justified and grantable, request it explicitly and only then switch to
Mode A for that single step. Never quietly fall back to SSH on prod.

---

## 0.6 Tooling gate — confirm you can actually pull the evidence

Picking the mode is not enough — you need the **channel** to be wired up. Don't
assume any observability tooling exists; a portable install of this skill often
has none of it connected. Confirm your evidence channel BEFORE you start
collecting, and if it's missing, resolve that up front instead of stalling
mid-investigation.

**Mode A** needs only an operator who can run read-only commands and paste
output — no agent-side tooling. If you have that, proceed.

**Mode B** needs the agent to reach read-only observability itself:
- a **metrics** source (Prometheus/Grafana, Datadog, CloudWatch, or equivalent),
- a **logs** source (Splunk, ELK, Loki, cloud logging, …),
- **dashboards**,
- **incident / on-call** tooling (PagerDuty, Opsgenie, an incident tracker),
- **config at the deployed commit** (a repo/git or code-search reader).

Check which of these you actually have (typically as connected MCP servers or
tools). **If the channels you'd need are missing, decide up front — don't
silently guess or fabricate values:**

1. **Map each critical-path layer to a channel you have.** If a layer's only
   evidence lives behind a channel you lack, that's a gap to resolve now.
2. **If core channels are missing, stop and give the operator a concrete
   either/or:**
   - **Connect the tool** — "I can drive this myself if you connect a metrics /
     logs / incident MCP (or point me at the dashboard/query API). Want to set
     that up?", **or**
   - **Relay the telemetry** — a single **pinpointed ask**: the exact queries /
     dashboard panels / log searches for the operator to run and paste back
     (this turns Mode B into an operator-relayed telemetry flow — the human is
     the transport, the method is unchanged). Give literal expressions, not
     "check the dashboard".
3. **If only a peripheral channel is missing**, proceed with what you have,
   label that layer **untested**, and name in the plan exactly what would close
   the gap (which tool to connect, or which value to paste).

Make any ask **specific and batched** — one message naming every value/query you
need and where it comes from. Never send the operator on a vague "send me the
metrics" round trip, and never invent a value to avoid asking.

---

## 1. Collect — maximum information before any theory

Build a complete picture of the system's state BEFORE forming a hypothesis. A
wrong theory on thin data wastes everyone's time.

**Common to both modes:**
1. **Pin the context.** Establish invariants: which box (hostname, cloud/region,
   role), the exact symptom, what changed and when (deploy, config push, image
   bump), and what "healthy" looks like. Get a known-good peer for differential
   diagnosis — comparing to a working peer is faster than reasoning in the
   absolute.
2. **Fan out, don't drill.** Sample every layer independently
   (process → socket → route → tunnel → dataplane → resolver → upstream) before
   deep-diving one. You don't yet know which layer is broken.
3. **Label every result by the box / direction it came from.** In cross-box
   problems (A → B), tag each result with which end and which direction. Never
   mix them — and for tunnels, check **both directions** (return path can be
   dead while forward is fine).
4. **Sample rate-based signals twice.** Packet/drop counters, flow hits,
   metric series: read, wait/probe under load, read again — a delta of zero
   under active traffic is itself evidence.
5. **Record raw results verbatim** into a findings scratch file; you'll cite
   them in Phase 2. Paraphrase loses signal.

**Mode A adds:** issue the fan-out as a batch of read-only commands for the
operator to paste.

**Mode B adds:** the agent runs the fan-out itself across observability:
- **Metrics** (Prometheus/Grafana, Datadog, CloudWatch, or equivalent) — the
  failing metric and its *siblings*. For a synthetic-ping failure, don't stop at
  the top-line availability metric; pull the whole watchdog family (DNS-resolve
  success, tunnel/BFD state, end-to-end pinger status) to stage-isolate. Query
  the **reverse direction** as its own series.
- **Logs** (Splunk, ELK, Loki, cloud logging, or equivalent) — the deployed
  version/image log line (this gives you the commit for config reads), plus
  error/timeout lines around the onset timestamp.
- **Dashboards** — the operator's link is a lead; reconstruct the underlying
  query expression and run it directly so you have the raw values, not a
  picture.
- **Incident / on-call tooling** (PagerDuty, Opsgenie, an incident tracker) — is
  there an open incident? onset time? blast radius (one node vs a whole
  region/path)?
- **Config at the deployed commit** — read the manifest/policy/records the box
  actually runs, via the repo/git at that commit — never main-branch-as-deployed.

Exit criterion (both modes): for each layer in the critical path you can say
healthy / broken / untested — no "unknown" gaps.

---

## 2. Diagnose — pointwise claims backed by evidence

Convert raw evidence into a small set of **claims**, each with the exact result
that supports it. This is what you reason from and what a reviewer audits:

```
Claim N: <one-sentence assertion>
Evidence — <command OR query/metric that produced it>:
  <verbatim output / metric value / log line>
Therefore: <what this rules in / rules out>
```

Rules:
- **Separate confirmed-healthy from confirmed-broken, explicitly.** "Tunnel +
  BFD + DNS process healthy; cross-region data plane black-holed, return counter
  = 0" is far stronger than "ping is broken".
- **Localize, don't hand-wave.** Push toward the single drop point / failing
  component. If current evidence can't localize it, name the exact result that
  WOULD and go get it (back to Phase 1).
- **Bottom line in one paragraph**: root cause (or best-supported hypothesis +
  confidence), scope/blast radius, and what's already ruled out.
- **Name what is not yet localized.** Honesty about the gap prevents a
  confident-wrong conclusion — and in Mode B usually defines the escalation.

---

## 3. Plan — write it down before touching anything

A plan readable in 30 seconds:
- **Root cause** (or leading hypothesis) in one line.
- **Confirmed healthy / confirmed broken** bullets.
- **Recommended next action** — the single most informative next step. If
  diagnostic, say what each result would mean; if a fix, say exactly what
  changes and the expected effect.
- **Escalation path** — who owns the fix if it's outside reach, plus the
  evidence package to hand them. In Mode B this is often the *primary* output:
  you've localized as far as telemetry allows and the on-box step belongs to
  the owning team.

One decisive step at a time — not five speculative fixes.

---

## 4. Guide — pre-committed tracks

The core of the method: never fire a step and wait to see what comes back with
no plan for the result. Commit IN ADVANCE to the step, the results you expect,
and the branch per result. This turns a back-and-forth guessing chat into a
decision tree.

**Mode A — shell-command track** (operator executes):
```
Step N — <what this determines>
Run (on <which box>):
    <exact command, copy-pasteable, correct privilege/wrappers>
Expected & branches:
  • If <output A>      → <meaning>. Next: <Step X / done / fix>.
  • If <output B>      → <meaning>. Next: <Step Y>.
  • If <error/empty>   → <meaning>. Next: <Step Z>.
```

**Mode B — query track** (agent executes, then reports the branch taken):
```
Step N — <what this determines>
Query: <metric expression / log search / config path @ deployed commit>
Expected & branches:
  • If series/value is <A>  → <meaning>. Next: <Step X / escalate>.
  • If <B>                  → <meaning>. Next: <Step Y>.
  • If empty / no data      → <meaning: gap vs genuine zero>. Next: <Step Z>.
```

Rules for tracks (both modes):
- **Pre-commit the branches.** Enumerate plausible results and bind each to a
  next action before running. No waiting on you to interpret a result you could
  have anticipated.
- **One step at a time** unless several are truly independent reads (then batch
  and label them).
- **Exact and executable.** Mode A: correct box, `sudo`, container `exec`
  wrappers, full paths. Mode B: the literal metric/log query expression, not a
  description — and distinguish "empty because no data / wrong query" from a
  genuine zero (they mean opposite things).
- **Prefer authoritative signals over passive ones.** A tool tracing its own
  decision (`ovs-appctl ofproto/trace`, `ip route get`, resolver matrix
  `@local` vs `@upstream`; in Mode B a metric emitted *by the failing node*
  toward the peer) beats passive capture that can lie — kernel-bypass
  dataplanes return empty tcpdumps even when traffic flows.
- **Converge.** Each track should halve the remaining hypothesis space. Stop
  when the root cause is pinned or handed off.

---

## Worked example — prod, no SSH (Mode B)

Symptom: a synthetic pinger from region A to region B
(`regionA-host-* → regionB-host-*`) shows `PingAvailability=0`, but "the tunnel
exists". SSH into prod is forbidden → **Mode B**.

- **Collect (agent runs metric queries):** query the failing metric *and its
  siblings*. Availability per remote peer (start from the operator's dashboard
  expression, grouped by remote host). Then pull the DNS-resolve success metric
  for the synthetic ping, the tunnel/BFD state metric, and the end-to-end pinger
  status for the same source; and query the **reverse** (region B → region A) as
  its own series. Logs: deployed image line + errors around onset. Incident
  tooling: open incident? onset time? one remote or all?
- **Diagnose (stage-isolate):** if the tunnel metric is healthy but the
  DNS-resolve-for-ping metric is 0, the synthetic resolve dies before the ping
  stage → not a transport fault. If ping is attempted but 0 in one direction
  only, suspect a return-path / dataplane black-hole. Write each as a Claim with
  the metric value as evidence; separate healthy (tunnel/BFD) from broken
  (ping/return).
- **Plan / escalate:** telemetry can localize to "control plane healthy, this
  cross-region path black-holed, correlated with <deploy at onset>". The final
  localizer (per-hop packet trace on the box) is the **escalation boundary** —
  package the metric evidence + onset + blast radius for the dataplane/network
  owner rather than SSHing.

---

## Loop back

If a track's result contradicts the current hypothesis, return to Phase 2,
demote the busted claim to a lead, and re-diagnose from the fuller evidence.
Don't force the old theory. Collect → Diagnose → Plan → Guide, iterated until
the system tells you the answer.

## Output discipline

- Keep prose tight; the operator is often mid-incident.
- Every result labeled by source (box/direction, or metric/query). Every value
  traceable to a real result — never invented.
- End each turn with one clear next action (a command, a query, or an
  escalation), never an open question with nothing attached.
