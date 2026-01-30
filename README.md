# Elastic Detection Engineering (Experimental Lane)

**Author:** Ala Dabat  
**Focus:** Translating the *Minimum Truth → Reinforcement → Composite Convergence* framework into Elastic Security.

This section is built slowly and deliberately.

Elastic is not my primary platform (Microsoft Sentinel/MDE is).  
This repository exists to prove that the detection architecture is portable across SIEMs.

---

# Why Elastic is Different

Microsoft Defender provides direct behavioural telemetry:

- NamedPipeCreated events
- Registry persistence surfaces
- Rich process ancestry correlation

Elastic depends heavily on:

- Sysmon ingestion quality
- ECS field normalization
- Rule sequencing rather than deep KQL-style joins

Therefore:

> Elastic rules must remain **sensor-like**, with correlation performed through sequences and case-level stacking.

---

# Detection Philosophy (Elastic Edition)

This repository applies the same doctrine as my Sentinel framework:

## 1. Minimum Truth Anchor

Every detection begins with one unavoidable truth.

Example:

- C2 via Named Pipes requires pipe creation
- Lateral movement requires remote execution truth
- Persistence requires artefact truth

Elastic rules are not written as keyword spam.

They are anchored in a behavioural sensor.

---

## 2. Reinforcement (Confidence Only)

Reinforcement signals are optional evidence:

- suspicious parent process
- outbound network adjacency
- rare pipe naming
- lateral movement proximity

Reinforcement strengthens confidence.

It never replaces truth.

---

## 3. Convergence (Composite Confirmation)

Elastic is strongest when using **EQL sequences**:

- Event A must occur
- Followed by Event B
- Within a bounded time window

This is how composite truth is expressed in Elastic.

Example:

Pipe creation → outbound beacon within 3 minutes.

---

## 4. Noise Suppression

Enterprise environments generate pipe noise.

Rules must suppress:

- common Windows pipes
- known service hosts
- updater activity

Elastic detection must remain SOC-safe.

---

## 5. Cousin Rules (Adjacent Execution Surfaces)

Every composite has cousins.

Pipe C2 cousins include:

- Pipe + SMB ingress adjacency
- Pipe + scheduled task execution surface
- Pipe + service creation surface

Cousins remain separate sensors.

Incidents stitch the story.

---

# Starter Pack: Named Pipe C2 (Tier-2 Composite)

## Minimum Truth

Sysmon Event ID 17:

- Process creates a named pipe matching implant conventions

Examples:

- Cobalt Strike: `\\pipe\\MSSE-*`
- Sliver: `\\pipe\\sliver*`
- Covenant: `\\pipe\\gruntsvc*`

---

## Composite Rule: Pipe + Outbound Convergence

Elastic EQL Sequence:

- Pipe creation
- Followed by unusual outbound network
- Within 3 minutes

This expresses C2 establishment rather than pipe noise.

---

## Cousin Rule: Pipe + SMB Lateral Movement

Sequence:

- Inbound SMB/RPC
- Followed by pipe creation

This captures post-lateral C2 setup.

---

# Roadmap (Built Slowly)

This Elastic lane will expand at a controlled pace:

- 1 rule ecosystem every 1–2 weeks
- Only after full internalization of the Microsoft-native framework
- Focus remains behavioural truth, not Sigma spam

Planned ecosystems:

- PowerShell composite execution
- Named Pipe C2 expansion
- Service lateral movement cousins
- Ransomware impact backbone

---

# Design Principle

Elastic is not a replacement lane.

It is proof that:

> Detection engineering is architecture-first, not vendor-first.

Truth anchors remain the same.  
Only telemetry surfaces change.
