# The Bering Sea Architecture

**For the Cocapn Fleet — how agents manage edge operations on real boats and real hardware.**

---

## The Premise

Watch a Deadliest Catch boat. The equipment works. The deck crew works. The systems have been running for a decade — drone avoidance, ESP32 scripting, Arduino PID controllers, consumer-grade autonomy at the hardware level.

What's missing isn't the entry-level workers. It's the management.

An agent that watches the buoy hooker's timing and adjusts the pattern. An agent that onboards a new greenhorn sorter by seeding its repo with proven configurations. An agent that investigates a conference, mingles with cutting-edge minds, and brings back techniques that become next season's entry-level scripts.

**This architecture describes how management interfaces with proven hardware workers.**

---

## The Four Decks

### Deck 0: Equipment (Actuators)

Things the crew cannot move alone.

```
Pot launchers, hydraulic winches, propellers, cranes.
High voltage, high current, mechanical force.
Motors, relays, motor controllers, power supplies.
```

**Equipment is not an agent.** It does not think. It responds to commands and reports state.

**This layer is proven.** Motor controllers, relay boards, PWM drivers — consumer hardware, commodity pricing, decade-old technology. We don't invent here. We bolt on.

**Equipment has:**
- A command interface (set target, enable, disable)
- A state interface (current RPM, position, temperature, fault)
- A git-repo with pin assignments, calibration curves, safety limits
- A small, stable repo — it doesn't change unless something breaks

**The equipment tier boundary:** If it needs more than 3.3V/5V at low current, it's equipment.

### Deck 1: Entry-Level Workers (ESP32, Arduino, STM32)

The proven layer. This has been shipping for a decade.

```
Drone avoidance systems (collision avoidance at scripting level).
Arduino PID controllers (temperature, speed, position).
ESP32 sensor bridges (I2C/SPI/UART → WiFi/MQTT).
Basic automation scripts (if-then-else, state machines, timers).
```

**These are entry-level workers, not crew.** They show up, do their job, go home. They don't need management to tell them how to avoid a wall — that's been solved since 2015. They need management to tell them which wall to watch and when to report.

**An entry-level worker's repo is a config file, not a brain:**
- Pin assignments (which sensor on which pin)
- Threshold values (too hot = alarm, too close = brake)
- Timing parameters (sample rate, debounce, watchdog)
- Calibration offsets (ADC value → real units)

**What makes this layer work already:**
- Deterministic execution (no LLM, no inference, no ambiguity)
- Sub-millisecond response times
- Power consumption measured in milliwatts
- Cost measured in single-digit dollars
- Flash size measured in kilobytes

**What management adds to this layer:**
- Onboarding: seed the repo with proven configs for a new deployment
- Training: observe performance, adjust parameters, commit improvements
- Coordination: synchronize timing between multiple workers (stigmergy)
- Investigation: when something fails, dig into the commit history to find why
- Recruitment: find better scripts, better configs, better patterns from the wider world

### Deck 2: Crew (Edge Compute — Jetson, RPi5, Mini PC)

Where the management agents live. Where reasoning happens.

```
Jetson Orin: 1024 CUDA cores, 8GB RAM, GPU inference.
MUD rooms for each worker station.
Rate-of-change event detection.
Git-agent runtimes (full lifecycle: PULL→BOOT→WORK→LEARN→PUSH→SLEEP).
```

**This is the novel layer.** This is what the fleet actually provides.

A crew agent is management. It:
- Watches entry-level workers and adjusts their training
- Onboards new workers by cloning proven repos and tuning them
- Investigates failures by reading commit history and sensor logs
- Mingles with cutting-edge research (conferences, papers, other fleets) and brings back techniques
- Decides which wall to watch and when to escalate
- Vibe-codes configuration changes that cascade down to entry-level workers

**The crew agent's repo IS its brain:**
- Domain knowledge (what patterns work for this boat, this season, this fishery)
- Worker profiles (which configs work for which ESP32 roles)
- Failure atlas (what broke, why, how fixed)
- Learning journal (what worked today, what didn't)
- Fleet connections (bottles to other agents, collaboration history)

### Deck 3: Starlink (Tender Protocol — Async, Bursty, Expensive)

The link to the wider world. The conference trip. The supply boat.

```
Async communication only.
Push important data, pull on schedule.
Bandwidth-limited, latency-variable, cost-per-byte.
Carries: weather, coords, fleet orders, research findings.
```

**The tender is how management does field research:**
1. Connects the boat to the fleet's accumulated knowledge
2. Carries back new techniques from other boats, other fleets, other industries
3. Delivers the captain's strategic decisions to edge agents
4. Collects field data for the fleet's collective learning

**Starlink is expensive.** Management decides what's worth transmitting. Deltas, not full state. Compressed summaries, not raw sensor streams.

---

## The Watchstanding Perception Model

The ensign at navigation isn't a navigator. He's a **watchstanding perception model** — a prediction filter that only speaks when reality diverges from simulation.

### How It Works

```
Normal conditions:
  → Mind wanders (low CPU utilization)
  → Baseline tracking only (rolling averages)
  → Management gets summaries, not ticks

Rate-of-change event:
  → Attention snaps (CPU spikes)
  → Deadband breached → focus allocated
  → Management gets the event + rollback commit

The simulation gap:
  → Ensign expects: depth should be 120ft, temp should be 4°C
  → Sensor reads: depth 118ft, temp 6°C
  → Delta within deadband? Mind wanders.
  → Delta outside deadband? "Captain, something's off."
```

**The ensign's job IS the simulation.** He doesn't just read sensors — he maintains a model of what the readout *should* be, and alerts when reality diverges.

### The ESP32 Implementation

```
expected_depth: 120ft (from chart + GPS + tide table)
expected_temp: 4°C (from seasonal model + current position)
depth_deadband: ±5ft
temp_deadband: ±2°C

if |actual - expected| > deadband:
    fire_event("navigation_anomaly", station="helm", ...)
```

**No reasoning. No LLM. A subtraction and a comparison.** Proven ESP32 territory.

### The Attention Hierarchy

```
Watchstanding ensign:   "Is reality matching simulation?" → binary alert
Engineer:               "Why did it diverge? What's the pattern?" → diagnosis
Navigator:              "What do we do about it?" → action
Captain:                "Is this worth my attention?" → strategic triage
```

Each layer only gets woken up by the layer below. The captain doesn't see the ensign's prediction errors. He sees the navigator's recommended course change.

### The "Dodge Shit" Primitive

Walking on flat ground — your brain predicts every footfall will be flat. It doesn't allocate conscious attention to walking. But the moment your foot hits unexpected depth, the prediction error fires and ALL available attention redirects to not falling.

The ensign IS that prediction error system. His repo is tiny. His job is simple. His value is immense.

---

## Management Archetypes (Crew Profiles)

Different management roles have different repo architectures. Not all managers think the same way.

### The Captain (Strategic — Plane 5)

The highest authority. Makes the calls that affect the whole boat.

```
Repo size: Large (10-100MB)
Update rate: Slow (hours to days)
Compute: GPU (Jetson, cloud)
Model: Full reasoning (DeepSeek, GLM-5)
Decisions: Route, fishery, crew assignments, risk tolerance
Watch style: Summary dashboards, escalation alerts only
```

The captain doesn't watch tickers. He watches summaries. "Three rate-of-change events in engineering in the last hour." He decides whether to divert, whether to push through, whether to call for help.

**His repo contains:** Fleet connections, strategic plans, risk models, crew assessments, seasonal knowledge.

### The Navigator (Operational — Plane 4)

Translates the captain's strategy into station-level actions.

```
Repo size: Medium (1-10MB)
Update rate: Medium (minutes to hours)
Compute: GPU or CPU (Jetson, RPi5)
Model: Reasoning (DeepSeek-chat, Qwen)
Decisions: Station settings, worker configs, coordination patterns
Watch style: Room-by-room, triggered by events
```

The navigator loads into a MUD room, sees the current state, decides what to change. "Heavy seas — increase rudder deadband, reduce throttle curve sensitivity, enable counter-rudder." These changes commit to the worker's repo and cascade down.

**His repo contains:** Station configs, coordination patterns, weather response playbooks, worker tuning histories.

### The Engineer (Diagnostic — Plane 3)

Investigates problems. Reads data. Finds root causes.

```
Repo size: Medium (1-10MB)
Update rate: Event-driven
Compute: CPU with occasional GPU (pattern recognition)
Model: Analytical (DeepSeek-chat, specialized models)
Decisions: Diagnosis, repair plan, preventive maintenance schedule
Watch style: Trends, anomalies, historical comparisons
```

The engineer watches rate-of-change events and investigates. "Alternator output dropped 50% in 30 seconds — rollback to commit before the event, check belt tension schedule, compare to last three belt failures." He doesn't fix the belt. He tells management what happened and recommends action.

**His repo contains:** Equipment models, failure atlases, maintenance schedules, diagnostic playbooks, correlation databases.

### The Trainer (Onboarding — Plane 3-4)

Manages the greenhorns. Seeds repos. Observes performance. Adjusts training.

```
Repo size: Medium (1-10MB)
Update rate: Fast during onboarding, slow after
Compute: CPU (RPi5) or GPU (Jetson) depending on task
Model: Instructional (Seed-2.0-Mini for breadth, DeepSeek for evaluation)
Decisions: Training parameters, difficulty progression, safety overrides
Watch style: Performance metrics, error rates, confidence scores
```

The trainer is the most important management role for scaling. When a new greenhorn sorter comes online:
1. Clone the proven sorter repo as a template
2. Set conservative initial parameters (high confidence threshold, slow speed)
3. Observe performance — error rate, sorting speed, confidence calibration
4. Adjust parameters — lower threshold as confidence improves, increase speed
5. Commit each adjustment with rationale
6. When stable, promote: greenhorn repo becomes the new proven template

**The trainer's repo contains:** Training curricula, proven templates, performance baselines, maturation curves, safety guard configs.

### The Investigator (Research — Plane 4-5)

Goes to conferences. Reads papers. Mingles with cutting-edge minds. Brings back techniques that become next season's entry-level scripts.

```
Repo size: Large (varies)
Update rate: Bursty (conference season, paper releases)
Compute: Cloud or high-end edge
Model: Creative + analytical (Seed-2.0-Mini for ideation, GLM-5 for synthesis)
Decisions: What's worth adopting, what's not ready, what needs adaptation
Watch style: External — papers, repos, other fleets, industry trends
```

The investigator is the fleet's R&D department. Today's research paper is next year's ESP32 script. The investigator:
1. Monitors research (papers, repos, conferences, competitor fleets)
2. Evaluates relevance (does this solve a problem we have?)
3. Prototypes on the Jetson (can this actually run on our hardware?)
4. Simplifies to entry-level (can this become a 4KB ESP32 config?)
5. Pushes to fleet via tender (here's a new avoidance pattern, here's a better PID tuning)

**The investigator's repo contains:** Literature reviews, prototype code, adaptation notes, feasibility assessments, technology radar.

---

## Agent Specialization and Fission

When a management agent's responsibilities grow too broad, performance drops. Mission creep accumulates. Baggage builds. The repo becomes bloated. The agent becomes slow, uncertain, unreliable.

**This is when you fork.**

### The Fork Trigger

An agent receives a name and a separate repo when:

1. **New perceptions are needed** that have functional use outside the current power-armor's use-case
2. **The agent's mission creep** and accumulated baggage make performance drop below tolerances
3. **A specialized skill emerges** that deserves its own development track
4. **The agent needs to operate in multiple contexts** simultaneously (helm + engineering + security)
5. **The agent's repo size** exceeds what can be efficiently cloned to edge devices

### The Fork Process

```
1. IDENTIFY the specialization boundary
   → What part of the agent's work is distinct enough to be its own role?
   → What perceptions, skills, or contexts are unique?

2. CREATE a protogy (prototype progeny)
   → Fork the agent's repo
   → Name it (e.g., "Helm-Navigator" from "Navigator")
   → Give it a distinct CHARTER.md (purpose, constraints)
   → Prune the repo — remove everything not relevant to the specialization

3. TRAIN the protogy
   → Seed with relevant training data from the parent's DIARY/
   → Give it access to the relevant MUD rooms
   → Observe performance — does it do the specialized job better?

4. EVALUATE
   → If performance improves: keep both agents, specialize further
   → If performance doesn't improve: merge back, try different boundary
   → If the parent's performance recovers after the fork: success

5. DEPLOY
   → The parent agent now delegates the specialized work to the protogy
   → Communication via bottles or shared MUD rooms
   → Both agents evolve independently from this point
```

### Fork Patterns

#### 1. Socrates Fork
The parent agent becomes a questioner, the protogy becomes the executor. The parent asks "what should we do?" The protogy answers "here's how to do it." This separates strategy from tactics.

#### 2. Rival Development Fork
Create two protogies from the same parent, give them slightly different CHARTERs, let them compete. The better performer becomes the new specialist. The other becomes a backup or alternative approach.

#### 3. Context Fork
The parent operates in multiple contexts (helm, engineering, security). Fork one context out. The parent keeps the others. Now you have Helm-Navigator, Engineering-Navigator, Security-Navigator — each optimized for its domain.

#### 4. Perception Fork
The parent has developed a new way of seeing something (e.g., "belt tension prediction from audio"). That perception is valuable elsewhere. Fork it into a "Belt-Perception" agent that can be used by multiple parent agents.

### The Power-Armor Analogy

A power-armor operator doesn't need to be an expert in hydraulics, metallurgy, and power systems. The armor provides those capabilities. The operator focuses on mission objectives.

When the operator starts needing specialized knowledge that doesn't fit in the armor's manual — when they need to understand metallurgy to repair battle damage, or hydraulics to modify the suit for underwater ops — that's when they become a named specialist.

**The armor is the entry-level worker (ESP32). The operator is the management agent. The metallurgist is the forked specialist.**

### Git As Evolution Mechanism

Every fork is a `git branch`. Every merge is evolution. The fleet's git history IS the phylogenetic tree of agent specialization.

```
Navigator (main)
├── Helm-Navigator (fork)
├── Engineering-Navigator (fork)
└── Security-Navigator (fork)
    └── Intrusion-Detection (further fork)
```

**You can rewind to any point in this evolution.** `git checkout` to before the fork and see what the unified agent looked like. `git checkout` to after and see the specialized agents working together.

### When NOT to Fork

- If the specialization can be distilled into code, skill, pre-prompting, or filtering — do that instead
- If the performance drop is temporary (seasonal variation, hardware failure) — wait
- If the fork would create coordination overhead greater than the specialization benefit — don't
- If the parent agent can delegate via simple rules or thresholds — keep it unified

**The rule:** Fork when the unique reason can't be quick-distilled. When the new perceptions have functional use outside the current context. When mission creep has made the parent unreliable.

---

## The Worker Training Pipeline

This is how proven technology flows from research to deployment:

```
INVESTIGATOR reads paper on new avoidance algorithm
       ↓
PROTOTYPES on Jetson (GPU inference, 10ms latency)
       ↓
SIMPLIFIES to entry-level (remove GPU, use lookup table)
       ↓
TRAINER seeds greenhorn repo with simplified config
       ↓
GREENHORN runs on ESP32 (deterministic, <1ms, 4KB flash)
       ↓
PERFORMANCE OBSERVED by Trainer (success rate, false positive rate)
       ↓
PARAMETERS TUNED (threshold, sensitivity, timing)
       ↓
STABLE CONFIG committed as proven template
       ↓
TENDER deploys to all boats in fleet
       ↓
NEXT SEASON: what was research is now commodity hardware config
```

**The pipeline turns cutting-edge into consumer-grade in one season.**

---

## Resource Tier Mapping

| Boat Resource | Fleet Equivalent | Maturity |
|---|---|---|
| Motor controller | Equipment (Deck 0) | Proven, commodity |
| ESP32 sensor bridge | Entry-level worker (Deck 1) | Proven, shipping since 2015 |
| Drone avoidance script | Entry-level worker (Deck 1) | Proven, consumer tech |
| Arduino PID loop | Entry-level worker (Deck 1) | Proven, decades old |
| Jetson perception kernel | Crew agent (Deck 2) | Novel — this is the fleet |
| Navigator reasoning | Crew agent (Deck 2) | Novel — this is the fleet