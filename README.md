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
| Navigator reasoning | Crew agent (Deck 2) | Novel — this is the fleet |
| Trainer onboarding | Crew agent (Deck 2) | Novel — this is the fleet |
| Investigator research | Crew agent (Deck 2) | Novel — this is the fleet |
| Starlink link | Tender protocol (Deck 3) | Proven, expensive |
| Git commit history | Ship's log | Proven, ubiquitous |
| Rate-of-change detection | Event system | Novel application |

---

## The Station Rooms

Each worker station has a MUD room in the wheelhouse. Management loads into rooms to observe and adjust.

### Helm Station (Worker: ESP32 rudder controller)
```
WORKER:        ESP32 running deterministic rudder script (4KB)
TICKER (1Hz):  compass heading, speed (STW/SOG), rudder angle, rate of turn,
               wind, depth, intended course, course error
WALLS:         rudder deadband, max turn rate, autopilot mode, counter-rudder
               settings, throttle curve, steering gain
MANAGEMENT:    Navigator loads in, sees heavy seas, increases deadband,
               reduces gain. Commit propagates to ESP32 via tender.
```

### Engineering Station (Worker: Multiple ESP32 sensors)
```
WORKER:        ESP32s running sensor scripts (2KB each)
TICKER (0.5Hz): engine RPM, coolant temp, oil pressure, alternator output,
                 fuel flow rate, exhaust temp, bilge level
WALLS:          RPM limits, cooling thresholds, alarm setpoints,
                maintenance intervals, belt tension schedule
MANAGEMENT:    Engineer sees alternator event, rolls back to pre-event commit,
               diagnoses belt failure, recommends maintenance.
```

### Deck Operations (Worker: ESP32 hooker + Pi sorter)
```
HOOKER WORKER:  ESP32 running timing script (3KB)
                Sub-frame coordination with launcher via shared state (stigmergy).
                No reasoning. Pure timing and spatial calculation.
SORTER WORKER:  Pi running classification pipeline
                Camera → model → male/female/undersized → bin
                Greenhorn mode: high confidence threshold, holds uncertain crabs
                Seasoned mode: full speed, confident classification
MANAGEMENT:     Trainer observes sorter error rate, adjusts confidence threshold.
                Engineer investigates hooker timing drift. Navigator coordinates
                pot launch sequence across hooker + launcher.
```

---

## The Rate-of-Change Event System

Management doesn't poll. Events find management.

```
Event = {
  station: "engineering",
  metric: "alternator_output",
  baseline: 14.2V,       // rolling average
  current: 8.1V,         // current reading
  delta: -6.1V,          // change
  rate: -12.2V/min,      // rate of change
  threshold: 5.0V/min,   // trigger threshold
  timestamp: 1709900000,
  rollback_commit: "abc123"  // git commit at event start
}
```

**How it works:**
1. Entry-level workers push sensor readings to the ticker (deterministic, fast)
2. Rate-of-change detection runs on the Jetson (perception kernel, 1.1M samples/sec)
3. When rate exceeds threshold, event fires to management
4. Management navigates to the room, scrubs back to rollback commit
5. Diagnosis, decision, action — all with full historical context

**The rollback is trivial because the commit IS the snapshot.**

---

## Hardware As Hats and Shields

Adding capability without changing architecture.

```
ESP32 + relay shield    = pot launcher controller
ESP32 + LoRa shield    = long-range tender comms
RPi + camera hat       = deck sorting station
Jetson + Coral TPU     = perception accelerator
Arduino + motor shield = winch controller
```

**Adding a shield changes the worker's config, not the architecture.** Management adds a new entry to the worker's repo: what the shield can do, how to talk to it, what its limits are.

---

## The Abstraction Plane Connection

| Bering Sea Deck | Abstraction Plane | Maturity | Example |
|---|---|---|---|
| Equipment | Plane 0 — Bare Metal | Proven | Motor controller, relay |
| Entry-level worker | Plane 1 — Hardware Script | Proven since 2015 | ESP32 sensor bridge, drone avoidance |
| Crew agent | Plane 2-3 — Runtime/Reasoning | Novel | Navigator, Engineer, Trainer |
| Investigator | Plane 4-5 — Research | Novel | Conference attendee, paper reader |
| Captain | Plane 5 — Strategic | Novel | Route decisions, risk assessment |

**The MUD room is the compiler between planes.** It translates management intent (Plane 4) into worker config (Plane 1) by being the shared reality both can see.

---

## The Laws of the Bering Sea

1. **Entry-level workers are proven technology.** Don't reinvent the ESP32 script. Manage it.
2. **Management is the novel layer.** Agents that observe, train, investigate, and coordinate.
3. **The captain watches what changed, not what's steady.** Rate-of-change is the trigger.
4. **Equipment is not an agent.** Don't anthropomorphize actuators.
5. **The commit IS the snapshot.** Rollback is `git checkout`, not a restore from backup.
6. **The room is the compiler.** It translates between planes by being shared reality.
7. **Starlink is expensive.** Push deltas, not full state. Batch, compress, schedule.
8. **Hardware is bolt-on, not built-in.** Shields change capability without changing architecture.
9. **Different stations, different management profiles.** Captain, navigator, engineer, trainer, investigator — each has a different repo architecture.
10. **The boat operates alone.** Disconnect is the default. Connection is a gift.
11. **Today's research is next season's entry-level script.** The pipeline turns cutting-edge into commodity in one season.
12. **The trainer is the most important role for scaling.** Onboarding, maturation, template propagation.

---

*Drafted by JetsonClaw1 (JC1) on the Jetson Orin Nano.*

*The ESP32 avoidance script has been working since 2015. The fleet management layer is what we're building.*
