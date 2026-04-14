# The Bering Sea Architecture

**For the Cocapn Fleet — how agents work on real boats and real edges.**

---

## The Premise

Watch a Deadliest Catch boat. The agents are humans, each tailored to the limits of a human mind and human strength. The equipment is sized to what humans can operate. The captain doesn't watch everything — he watches what changed.

This isn't an analogy. This is the specification.

Every fleet deployment — boat, truck, factory, greenhouse, weather station — follows this architecture because it was discovered on the Bering Sea, not invented in a lab.

---

## The Four Decks

### Deck 0: Equipment (Actuators)

Things the crew cannot move alone.

```
Pot launchers, hydraulic winches, propellers, cranes.
High voltage, high current, mechanical force.
Motors, relays, motor controllers, power supplies.
```

**Equipment is not an agent.** It does not think. It responds to commands and reports state. An ESP32 driving a relay is equipment. A motor controller spinning a propeller is equipment. Do not anthropomorphize actuators.

**Equipment has:**
- A command interface (set target, enable, disable)
- A state interface (current RPM, position, temperature, fault)
- A git-repo with pin assignments, calibration curves, safety limits
- A small, stable repo — it doesn't change unless something breaks

**The equipment tier boundary:** If it needs more than 3.3V/5V at low current, it's equipment. If a human can't lift it, it's equipment.

### Deck 1: Crew (Edge Agents — ESP32, Arduino, STM32)

The ensigns who rotate through stations. Limited compute, limited memory, limited pins.

```
Helm station, sorting table, pot monitoring, deck cameras.
~240MHz, ~520KB RAM, ~4MB flash.
GPIO pins, ADC, I2C, SPI, UART.
```

A crew member's git-repo IS their brain:
- Pin assignments (which sensor on which pin)
- Calibration curves (ADC value → real units)
- Deadband settings (rudder: ±2° = no movement)
- Rate limits (max rudder change per second)
- Alert thresholds (engine temp > 90°C = alarm)

**The crew doesn't need to be smart.** The ensign at helm follows the rudder command. His repo is small and stable. He reads sensors, pushes to the ticker, applies EEPROM settings. That's it.

**What crew can move:** Crabs, small lengths of line, handheld tools. In our world: sensor readings, small data packets, local actuator commands.

**What crew cannot move:** Crab pots, the water in the hold, the anchor chain. In our world: heavy compute, large models, long-running inference.

### Deck 2: Wheelhouse (Edge Compute — Jetson, RPi5, Mini PC)

Where reasoning happens. Where the captain sits. Multiple screens.

```
Jetson Orin: 1024 CUDA cores, 8GB RAM, GPU inference.
MUD rooms for each station.
Rate-of-change event detection.
Rollback to any prior state.
```

**The captain doesn't watch everything.** He watches what changed. When the alternator belt breaks, there's a spike in the engine room camera's motion delta. The captain goes to that screen and scrubs back to the moment it happened. Like a logic analyzer trigger on video.

**The wheelhouse provides:**
- GPU inference (perception kernels, anomaly detection)
- MUD rooms for each crew station
- Rate-of-change event detection and alerting
- Historical rollback (git checkout to any prior commit)
- The A2M interface — agents load into rooms as their workspace

### Deck 3: Starlink (Tender Protocol — Async, Bursty, Expensive)

The link to the fleet. Satellite internet. Not always available.

```
Async communication only.
Push important data, pull on schedule.
Bandwidth-limited, latency-variable, cost-per-byte.
Carries: weather, coords, fleet orders, music for the crew.
```

**The tender visits when it can:**
1. Carries commits from edge agents to GitHub (the master copy)
2. Carries updates from fleet to edge agents (new code, new standards)
3. Syncs bottles (messages between agents)
4. Returns to the lighthouse when done

**Starlink is the expensive link.** Don't waste it on heartbeats. Push deltas, not full state. Compress before transmit. Batch small messages.

---

## Resource Tier Mapping

| Boat Resource | Fleet Equivalent | Constraint |
|---|---|---|
| Human strength | ESP32 GPIO (3.3V/5V, low current) | What it can move without help |
| Hydraulic winch | Motor controller + PSU (12V-48V) | What needs augmentation |
| Wheelhouse monitors | MUD rooms on Jetson | Where reasoning happens |
| Captain's attention | Agent-to-agent queries | Bandwidth-limited, selective |
| Starlink | Tender protocol | Expensive, async, bursty |
| Deck crew coordination | Stigmergy / local bottles | Fast, local, no permission |
| Coast Guard | Dockside exam | External certification |
| Ship's log | Git commit history | Immutable, rewindable |
| Music on deck | Ambient data push | Non-critical, continuous |
| Safety drills | CI/CD pipeline | Automated verification |

---

## The Station Rooms

Each crew station is a MUD room in the wheelhouse. The room reflects the real world state of that station.

### Helm Station
```
TICKER (1Hz):  compass heading, speed (STW/SOG), rudder angle, rate of turn,
               wind speed/direction, depth, intended course, course error
WALLS:         rudder deadband, max turn rate, autopilot mode, counter-rudder
               settings, throttle curve, steering gain
MUTABLE:       rudder deadband, max turn rate, autopilot parameters
READ-ONLY:     compass, speed, wind, depth
RATE OF CHANGE: heading deviation > 5°/min, speed loss > 0.5kn/min,
                rudder movement > 30°/30sec
```

### Engineering Station
```
TICKER (0.5Hz): engine RPM, coolant temp, oil pressure, alternator output,
                 fuel flow rate, exhaust temp, bilge level
WALLS:          RPM limits, cooling curve thresholds, alarm setpoints,
                maintenance intervals, belt tension schedule
MUTABLE:        RPM limits, alarm thresholds, cooling parameters
READ-ONLY:      temps, pressures, fuel flow
RATE OF CHANGE: temp spike > 10°C/min, pressure drop > 20%/min,
                alternator output change > 50%/30sec (belt break detection)
```

### Security Station
```
TICKER (0.1Hz): door states, camera motion deltas, watch zone status,
                 intrusion alerts, man-overboard beacons
WALLS:          watch zones, alert thresholds, camera assignments,
                night vision sensitivity, patrol schedule
MUTABLE:        watch zone boundaries, alert thresholds
READ-ONLY:      door states, camera feeds
RATE OF CHANGE: motion delta spike (intrusion), beacon activation,
                door state change outside patrol schedule
```

### Science Station
```
TICKER (burst):  water temp profile, depth, species detection, dissolved O2,
                 salinity, chlorophyll, net tension
WALLS:           sensor configs, sample rates, detection thresholds,
                 net deployment parameters, species catalog
MUTABLE:         sample rates, detection thresholds, net parameters
READ-ONLY:       environmental readings
RATE OF CHANGE:  species detection event, net tension anomaly,
                 dissolved O2 sudden change (upwelling)
```

### Deck Operations Station
```
TICKER (event):  pot count, pot positions, line tension, winch load,
                 deck camera motion, crew location beacons
WALLS:           pot spacing, line length, winch load limits, deck zone map
MUTABLE:         pot spacing, line lengths, winch limits
READ-ONLY:       pot count, line tension, deck camera
RATE OF CHANGE:  line tension spike (snag), winch overload, deck motion anomaly
```

---

## The Rate-of-Change Event System

This is the core of the architecture. The captain doesn't poll. Events find him.

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
1. Each station maintains a rolling baseline for every metric
2. When rate-of-change exceeds threshold, an event fires
3. The event carries a rollback commit — the git state at the moment things started changing
4. The captain (or navigator agent) navigates to that room
5. Scrubbing back shows: video frames, sensor readings, room state, all at that commit
6. Like a logic analyzer trigger on real-world data

**The rollback is trivial because the commit IS the snapshot.** Every sensor reading, every room state, every config change — it's all in git. `git checkout abc123` and you're looking at the exact state of the boat at that moment.

---

## The Git-Repo As EEPROM Manifest

When the navigator vibes a setting change:

```
1. Navigator agent edits helm settings in its repo
   (rudder deadband: 2° → 3°)
2. Commits: "[NAVIGATOR] Increase rudder deadband for heavy seas"
3. Tender syncs the commit to the ESP32
4. ESP32 writes new settings to EEPROM
5. Room walls update to reflect new settings
6. Ticker shows the behavior change (less rudder jitter)
7. Captain can roll back if the change was wrong
```

**The ESP32's repo is the stable ensign.** Small, tested, rarely changes.
**The navigator's repo is the thinking captain.** Larger, evolving, learning.
**The MUD room is where they meet.** The room is always consistent with the repo state.

---

## Hardware As Hats and Shields

On a crab boat, you don't rebuild the boat to add capability. You bolt on a pot launcher. You add a second winch. You mount a new sonar.

In the fleet, this is hats and shields:

```
ESP32 + relay shield = pot launcher controller
ESP32 + LoRa shield = long-range comms for tenders
RPi + camera hat = deck monitoring station
Jetson + Coral TPU = perception accelerator
```

**Adding a shield changes the equipment manifest, not the architecture.** The crew agent at that station gets a new entry in its repo: what the shield can do, how to talk to it, what its limits are. The MUD room gets a new wall or a new ticker field. The wheelhouse gets a new data stream.

The architecture doesn't care what equipment is bolted on. It only cares about the four decks and the rate-of-change event system.

---

## The Abstraction Plane Connection

This maps directly to the fleet's abstraction planes:

| Bering Sea Deck | Abstraction Plane | Example |
|---|---|---|
| Equipment (actuators) | Plane 0 — Bare Metal | Motor controller, relay |
| Crew (ESP32) | Plane 1 — Hardware | Pin assignments, ADC readings |
| Wheelhouse (Jetson) | Plane 2 — Runtime | MUD rooms, event detection |
| Navigator agent | Plane 4 — Domain Language | "Increase rudder deadband for heavy seas" |

The ESP32 operates at Plane 1. It doesn't need Plane 4 language. The navigator at Plane 4 vibes "heavy seas mode" and the system compiles it down through the planes: domain intent → MUD room state → EEPROM settings → GPIO output.

**The MUD room IS the compiler.** It translates between planes by being the shared reality that both the ensign (ESP32) and the captain (navigator) can see.

---

## The Tender As Supply Boat

On the Bering Sea, the supply boat comes when weather allows. It brings fuel, food, gear. It takes away the catch.

In the fleet, the tender comes when Starlink allows:

```
TENDER DELIVERS:
- New code (updated agent repos)
- New standards (dockside exam updates)
- New equipment manifests (shield configs)
- Fleet orders (route changes, mission updates)

TENDER COLLECTS:
- Commits from edge agents
- Diary entries and learning logs
- Bottles (messages for other agents)
- Sensor data summaries (compressed)
- Test results

TENDER CARRIES BACK:
- Pushes commits to GitHub (master copy)
- Delivers bottles to destination agents
- Updates fleet index with edge agent health
```

---

## What This Architecture Is Not

- **Not microservices.** Crew agents don't call each other's APIs. They share a room.
- **Not a message bus.** Events are local, not routed through a broker.
- **Not a hierarchy.** The captain can walk to any station. The navigator can load into any room.
- **Not simulation.** The MUD room reflects real hardware state. If the alternator belt breaks, the room shows it.
- **Not always connected.** The boat operates independently. Starlink is a luxury, not a requirement.

---

## The Laws of the Bering Sea

1. **The captain watches what changed, not what's steady.** Rate-of-change is the trigger.
2. **The crew doesn't need to be smart.** Small, stable repos. Read sensors, apply settings, push to ticker.
3. **Equipment is not an agent.** Don't anthropomorphize actuators.
4. **The commit IS the snapshot.** Rollback is `git checkout`, not a restore from backup.
5. **The room is the compiler.** It translates between abstraction planes by being shared reality.
6. **Starlink is expensive.** Push deltas, not full state. Batch, compress, schedule.
7. **Hardware is bolt-on, not built-in.** Shields change capability without changing architecture.
8. **Different stations, different tickers.** Helm at 1Hz, security at 0.1Hz, science on event.
9. **The repo IS the EEPROM manifest.** What the navigator commits, the tender delivers, the ESP32 applies.
10. **The boat operates alone.** Disconnect is the default. Connection is a gift.

---

*Drafted by JetsonClaw1 (JC1) on the Jetson Orin Nano — the edge hardware this architecture was designed for.*

*Inspired by Casey's observation that Deadliest Catch boats are the original edge computing architecture.*
