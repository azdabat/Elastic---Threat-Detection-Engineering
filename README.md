# Elastic (Side Project) — Exploit Primitives → Telemetry Truth (Defensive-Only)

**Author:** Ala Dabat  
**Status:** Early-stage / slow-burn side project  
**Primary lane:** Microsoft Sentinel + MDE (production)  
**Secondary lane:** Elastic (research + primitive-led detection engineering)

This repo exists for one purpose:

> **Translate exploit primitives and malware mechanics into telemetry-first detections**  
> (without writing malware, without weaponisation, without “campaign label” chasing).

It complements my Microsoft-native portfolio by focusing on **mechanism detection**:
- exploit primitives
- loader behaviours
- protocol abuse
- memory/injection mechanics
- stealth & evasion patterns

---

## Why companies use Elastic (and why startups like it)

Elastic is often used because it can be a **fast, flexible research platform**:
- you can ingest **any telemetry** (endpoint, network, cloud, custom sensors)
- you can iterate quickly on **hypotheses** (new technique → new query → new dashboard)
- it’s great for **proto-detections** and “unknown unknowns” when you don’t yet know what the final rule should look like

This is why teams doing “0-day / exploit mechanics” style work often build:
- **primitive-led hunting** in Elastic
- then “promote” the highest-value truths into production platforms

*(My production platform remains Microsoft Sentinel/MDE. Elastic here is the research workbench.)*

---

## Non-Negotiable Safety / Defensive Boundary

This repository is **defensive-only**:
- No exploit code
- No weaponised PoCs
- No step-by-step intrusion workflows

The output is:
- threat modelling
- telemetry mapping
- detection logic
- validation harnesses
- investigation playbooks

---

## How this aligns to my framework

My core architecture remains unchanged:

### 1) Minimum Truth (Baseline Anchor)
One unavoidable behavioural truth must exist for the technique to be real.

### 2) Reinforcement (Confidence Builders)
Extra signals that raise confidence, but do not define the technique.

### 3) Convergence (Where noise becomes threat)
When multiple reinforcements intersect around the truth, we escalate.

### 4) Organisational Prevalence (Scaling + prioritisation)
Prevalence is never “truth”. It is a triage amplifier applied after truth.

This repo simply changes the *starting point* from “MITRE technique labels” to:

> **Exploit primitive → what must be true in telemetry → how to catch it at scale**

---

## Study spine: Practical Malware Analysis → Detection Engineering Output

I use chapters from **Practical Malware Analysis** as a structured way to learn primitives and translate them into detections. 2

### Output rule: every chapter produces 3 artifacts
1. **Telemetry Truth Map** (what MUST be true, per telemetry source)
2. **Detection Drafts** (Elastic queries + Microsoft equivalents where relevant)
3. **SOC Playbook Notes** (how to validate, scope, respond)

---

## Chapter → Detection Engineering Mapping (What matters for defenders)

Below is the defensive mapping of book chapters into detection engineering outcomes. 3

### Tier A — Core foundations (fast wins)
**Ch 1 — Basic Static Techniques**
- Defender output: prevalence, signer trust, import/section anomalies (as enrichment, not “truth”)

**Ch 3 — Basic Dynamic Analysis**
- Defender output: process/registry/file/network baselines and “what changed”

**Ch 11 — Malware Behavior**
- Defender output: persistence + C2 + credential access behaviours framed as “truth anchors”

**Ch 14 — Malware Focused Network Signatures**
- Defender output: protocol anomaly hunting, beacon features, domain/URI traits (without IOC dependence)

### Tier B — Evasion and stealth (high leverage)
**Ch 12 — Covert Malware Launching**
- Defender output: LOLBin proxies, service/task execution surfaces, parent-child trust breaks

**Ch 13 — Data Encoding**
- Defender output: encoded payload presence as reinforcement; staging patterns; “encoded intent” scoring

**Ch 15 — Anti-Disassembly / Ch 16 — Anti-Debugging / Ch 17 — Anti-VM**
- Defender output: sandbox evasion indicators as reinforcement (not a sole trigger)

### Tier C — Deep mechanics (primitives that map directly to “truth”)
**Ch 18 — Packers and Unpacking**
- Defender output: unpacking/runtime-write behaviours → memory protection changes, RWX, module anomalies

**Ch 19 — Shellcode Analysis**
- Defender output: shellcode-like execution signals → thread start anomalies, suspicious memory regions

**Ch 21 — 64-Bit Malware**
- Defender output: modern process injection + syscall patterns (telemetry-dependent)

### Optional skills (only as needed)
**Ch 4/5/6/8/9/10/20** are tooling depth (disassembly/debuggers) that helps you *understand* primitives,
but detections should still be expressed as telemetry truths.

---

## Repo structure (simple + sustainable)

/docs /primitives Primitive-01_ProcessInjection.md Primitive-02_ProcessHollowing.md Primitive-03_ReflectiveLoading.md Primitive-04_SignedProxyExecution.md Primitive-05_PersistenceSurfaces.md Primitive-06_C2_Channels.md /playbooks Playbook-Primitive-Triage.md Playbook-Validation-Checklist.md /detections /elastic primitive_process_injection.ndjson (or .md with query) primitive_signed_proxy_exec.ndjson primitive_named_pipe_c2.ndjson /microsoft primitive_process_injection.kql primitive_signed_proxy_exec.kql primitive_named_pipe_c2.kql /tests /datasets benign_baseline_samples.json synthetic_attack_like_samples.json /notes what_good_looks_like.md

This keeps Elastic work “clean” and prevents random one-off rules.

---

## Primitive-led detection: the “Truth → Reinforcement” templates

### Template format (applies to BOTH Elastic & Microsoft)
Each primitive detection has:

- **Truth Anchor**
- **Reinforcement Signals**
- **Noise Suppression**
- **Prevalence hook**
- **SOC directives**

---

## Starter pack (what I’m building first)

### 1) Named Pipe C2 (High value / high skill)
**Why it matters:** named pipes are common for local IPC, and also heavily abused by implants for stealthy C2 or operator channels.
Your edge is not “pipe strings” — it’s **enterprise noise control + convergence**.

**Minimum Truth (choose ONE per rule):**
- **Truth A (endpoint):** suspicious process interacting with a named pipe *in an anomalous context*
- **Truth B (cross-surface):** pipe activity + adjacent execution surface (lateral/service/task) in a tight window

**Reinforcement:**
- rare pipe name prevalence (org-wide)
- suspicious parent lineage (Office/script host → pipe user)
- pipe user also makes outbound connections soon after
- pipe user drops/loads artifacts in writable paths
- pipe user executes “operator-like” tooling primitives

**Noise suppression principles:**
- safe OS/service pipe allowlist **by pattern**, not endless exact names
- suppress known vendor agents (but never suppress if convergence is strong)
- treat “pipe-only, no reinforcement” as investigation/low priority rather than “critical”

**Deliverables:**
- `primitive_named_pipe_c2` (Elastic query + Microsoft KQL cousin)
- a validation guide: “what normal looks like in big estates”

---

### 2) Process Injection (mechanism > malware family)
**Truth anchors (pick one per rule):**
- suspicious remote thread creation / cross-process memory write (telemetry-dependent)
- anomalous memory protection transitions (RW → RX / RWX) (telemetry-dependent)

**Reinforcement:**
- unsigned/rare injector
- suspicious target processes (LSASS, browser, Office children, service hosts)
- concurrent LOLBin proxy execution / script staging
- immediate suspicious network behaviour after injection

---

### 3) Signed Proxy Execution (LOLBin “execution surfaces”)
**Truth anchors:**
- trusted binary executing with anomalous arguments / from anomalous parent
- trusted binary loading from user-writable locations

**Reinforcement:**
- child process tree anomalies
- encoded/script primitives
- external network retrieval
- prevalence rarity

---

## How to build “Exploit primitive → telemetry truth” without touching malware

You do NOT need weaponised samples to build strong detections.

Use this workflow:

1. **Define the primitive**
   - e.g., “remote thread injection”, “named pipe operator channel”, “silent scheduled task persistence”

2. **Write the Minimum Truth**
   - the one unavoidable telemetry condition if the primitive exists

3. **List reinforcements**
   - evidence that raises confidence (not required for truth)

4. **Design suppression**
   - what is normal in enterprises for this primitive-adjacent surface?

5. **Build a validation harness**
   - compare against baseline data (benign)
   - add synthetic “attack-like” rows (NOT exploit code) that emulate the *telemetry shape*

6. **Promote only after**
   - stable signal
   - stable telemetry availability
   - understandable SOC output

---

## Testing strategy (practical, SOC-real)

### Phase 1 — Baseline first (benign reality)
- run detections against normal fleet data
- measure:
  - alert volume per day
  - top noisy processes
  - common pipe patterns
  - common parents/children

### Phase 2 — Synthetic “attack-like” telemetry (safe)
- create **non-executable** synthetic events that mimic:
  - suspicious parentage
  - rare pipe naming
  - correlated inbound SMB → scheduled execution (for cousins)
  - encoded/URL-bearing command lines

### Phase 3 — Promotion gates
A rule only moves from “research” to “portfolio-ready” when:
- truth anchor is stable
- suppression is explicit
- output is explainable in 60 seconds
- you have at least one investigation playbook note

---

## Relationship to my Microsoft portfolio

- My production-ready composites live in Microsoft-native repos (Sentinel/MDE).
- This Elastic repo is:
  - primitive-led research
  - technique modelling
  - detection experiments
  - slow-burn skill expansion

If a primitive proves valuable here, it becomes a **Microsoft-native composite** using:
- Minimum Truth → Reinforcement → Convergence
- org prevalence weighting
- SOC directives embedded in output

---

## Roadmap (slow but real)

**Week 1–2:** Named Pipe C2 baseline + prevalence model  
**Week 3–4:** Pipe convergence (pipe + lateral cousin)  
**Week 5–6:** Injection primitive truth mapping  
**Week 7–8:** Signed proxy execution primitive pack  
**Ongoing:** one new primitive every 2 weeks (sustainable)

---

## References
- Practical Malware Analysis chapter spine (chapter list) 4
- My Microsoft-native framework and composites live in my main GitHub profile and pinned repos.
