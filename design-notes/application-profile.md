# Design Note: Application Profile (Layer 3)

*Extracted from HEAIN-CORE-1.1's internal design notes, for public reference alongside the published papers.*

## "Application Profile" vs. "Duty Profile" — two distinct concepts on two different layers, not the same thing under two names

**Duty Profile** (Layer 2, HEAIN Core Protocol) answers "what can this role slot's current occupant do, given its capacity" — a capability/capacity gate checked at promotion time.

**Application Profile** (Layer 3, Application AI) answers "what does this specific job need from this specific module — how should it be split, checkpointed, resumed, merged, verified for access, or by what AI method." Layer 2 never reads or reasons about an Application Profile; it only ever sees the Duty Profile side (a duty name + capacity requirement), the `recipient_key_encrypted` output-encryption-policy flag, and the opaque Swarm/ModelDelta payloads that Layer 3 modules send through it.

## Application Profile is a composite, extensible container — not embodied by a single module

```
ApplicationProfile { <module_name>: <that module's own facet schema>, ... }
```

This matches the same aggregation pattern already used for Duty Profile (many duties, from many modules, aggregated under one role-slot key) and for the Layer 3 module survey's "each module publishes its own Capability Catalog" principle.

`job` (orchestration: split-strategy, checkpoint, merge-strategy, resume-semantics — owned by the `heain-job` module) and `access` (recipient/verification-method/validity-window — owned by the `heain-access` module) are the two facets confirmed needed so far. Whether any other Layer 3 module needs its own Application Profile facet at all is decided per-module, later, not assumed — a module with nothing job-specific to configure needs no facet and is simply absent from a given job's `ApplicationProfile`.

## Each Layer 3 module is its own separate container

This is an implementation/deployment decision (containerization for independent scaling/deployment), not a statement about what belongs in a module's Application Profile facet.

## AI vs. deterministic code is a per-module choice — with one exception

Whether a given module (or a given facet within it) is implemented as genuine AI/ML or ordinary deterministic code remains a per-module Layer 3 implementation choice — not every Application Profile or every module needs to be AI.

**One confirmed exception: `heain-job` itself is required to be AI/ML, not merely permitted to be.** All modules' containers are provisioned Hybrid (GPU+CPU-capable), with `heain-job` deciding actual GPU-vs-CPU allocation per task at runtime — not a fixed compute profile per module. A fixed deterministic rule cannot capture "which of several hybrid modules' current tasks most need the limited GPU capacity available right now" the way a learned scheduler can — directly analogous to the core protocol's own dispatch layer, which already pairs an AI-assisted weighted score with a deterministic fallback one layer up from what `heain-job` does here.
