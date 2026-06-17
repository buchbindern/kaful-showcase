# Technical approach

## Why physics-first?

Most predictive maintenance approaches are data-driven: train a model on historical sensor readings and failure events, then predict future failures. This works well when you have years of labeled failure data from a large fleet.

It breaks down in exactly the situations manufacturers actually face: a new machine with no failure history, a one-off production line, or a facility that can't afford unplanned downtime while waiting years to accumulate training data.

The physics-first approach inverts this. OEM manuals already contain the degradation physics — service intervals, tolerance thresholds, failure mode descriptions — expressed as engineering knowledge. Kaful extracts that knowledge and uses it to construct a model that can produce meaningful RUL forecasts from day one.

## Pipeline design decisions

### RAG over manuals (not fine-tuning)

Kaful uses retrieval-augmented generation to extract component specs and failure modes from OEM PDFs. Fine-tuning was considered and rejected: the relevant information is sparse and machine-specific, retrieval keeps the source document as the authority, and it generalizes to new machine types without retraining.

### Particle filter (not a point estimate)

The prognosis stage uses a particle filter over the physics simulation rather than producing a single RUL number. This was a deliberate choice: a single number implies false precision. A particle filter produces a probability distribution over future states, which means the output is a calibrated uncertainty range — actionable for a maintenance engineer in a way that a point estimate isn't.

Ensemble propagation was added to address particle filter degeneracy, a known failure mode where the ensemble collapses to a single trajectory under heavy censoring.

### Agentic pipeline (not a monolithic model)

The seven-stage pipeline is structured as an agentic system where each stage's output becomes the next stage's input, with LLM reasoning driving the comprehension, triage, and codegen stages. This decomposition makes it possible to inspect and debug each stage independently, which matters when the input is an OEM PDF and the output is a physics model.

## Known limitations

- RUL forecasts are only as good as the physics encoded in the OEM manual. Underdocumented failure modes produce wider uncertainty ranges.
- The particle filter requires calibration per machine type (handled by the phase 6.5 calibration module) to avoid degradation timescales triggering too early.
- Current support is for rotating industrial equipment; HVAC, hydraulic, and electrical systems require additional physics primitives.
