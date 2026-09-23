# AURIORA Hardware Design Guide

**Document ID:** AHDG
**Version:** 0.7.0
**Status:** Normative
**Complements:** AURIORA Engineering Standard (AES)
**Language:** English

## About This Guide

This is the official hardware design guide for all AURIORA hardware. It complements the AURIORA Engineering Standard: AES defines *what* must be true about AURIORA engineering work (identity, interfaces, maturity levels, releases), while this guide defines *how* hardware should be designed to reach consistent quality across all modules.

It is an engineering handbook, not a tutorial. It describes design principles and proven practice, and it applies to every AURIORA hardware project regardless of function. It deliberately contains **no specific part numbers or vendor recommendations** — component choices age quickly, design rules do not.

Requirement language (aligned with AES):

- **MUST** — mandatory when applicable. Equivalent to `SHALL` in AES.
- **SHOULD** — recommended default; engineering judgment may justify another approach. Skipping a SHOULD requires no documented exception.
- **MAY** — optional improvement.

Where this guide and AES conflict, AES prevails. Deviating from an applicable MUST needs a concise note in the design notes; formal exception records are reserved for Released artifacts and platform-wide deviations ([AES-GOV-003](https://github.com/auriora-org/auriora-engineering-standard/blob/main/STANDARD.md#aes-gov-003-honest-deviations)).

### Maturity Scaling

Requirements scale with the [AES maturity level](https://github.com/auriora-org/auriora-engineering-standard/blob/main/STANDARD.md#3-maturity-model):

- **Always (all maturity levels):** the safety and damage-prevention rules — reverse-polarity and overcurrent protection on external power inputs, battery protection independent of software, creepage/clearance above SELV, safe default states, honest hazard notes. Electricity does not care that the board is a prototype.
- **Released only:** production design rules — full manufacturing package, alternate sourcing, DFM/DFT completeness, formal documentation packages. Sections 12, 13 and 15, and every rule that says "released", bind Released hardware.
- **Everything else** is the recommended default (SHOULD): follow it unless the specific board gives a real reason not to, and note non-obvious departures in the design notes. A two-layer low-speed PCB does not need high-speed design evidence; a board without calibration needs no calibration records; distinguish what the *design* needs from what the *checklist* lists.

---

## 1. Design Philosophy

AURIORA hardware is designed to be manufactured, repaired and extended for decades, by engineers who were not there when the original decisions were made.

- **Simplicity.** The simplest circuit that meets requirements with margin is the correct circuit. Justify extra complexity by a real requirement, not by anticipated needs. Prefer well-understood topologies over clever ones.
- **Reliability.** Designs SHOULD be dimensioned for worst case (supply tolerance, temperature, component tolerance, aging), not typical values; for Released hardware this is a MUST. Single points of failure that can cause damage are found in review and eliminated or consciously accepted. Faults SHOULD fail safely and visibly.
- **Maintainability.** A Released design MUST be understandable from its published documentation alone. Non-obvious decisions (odd component values, deliberate rule violations, layout-critical constraints) SHOULD be noted where they apply — a schematic annotation or design-notes line is enough.
- **Serviceability.** Parts that wear or fail first (connectors, fuses, protection parts, electrolytics) SHOULD be replaceable with standard rework equipment without disturbing unrelated circuitry.
- **Long lifetime.** Released designs MUST NOT design in parts already end-of-life or not-recommended-for-new-designs without a noted migration plan; prototypes may knowingly use them with a BOM note. Fast-aging technology (radios, MCUs, special sensors) SHOULD sit behind clean electrical interfaces so substitution changes one region, not the architecture.
- **Open hardware.** A Released design MUST be buildable from its published files: editable sources, full BOM, manufacturing outputs, assembly data ([AES-REL-001](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/07-maturity-and-release.md#aes-rel-001-release-completeness)). Avoid NDA-only parts; prefer tools and formats usable without commercial licensing.
- **Future expansion.** Interfaces on Released hardware MUST be declared and versioned per AES — no undocumented headers. Reserve power, pin and mechanical headroom only where a roadmap justifies it. Unpopulated option footprints MAY be used if named in the schematic and BOM variants.

---

## 2. Schematic Design

The schematic is read far more often than it is drawn — by reviewers, layout, test, firmware and future maintainers. Draw it for them.

- **Hierarchy.** Complex boards SHOULD use hierarchical sheets, one functional block per sheet, with a top sheet showing the board architecture: blocks, major buses, power rails, external connectors. A genuinely simple board is fine on one or two flat sheets — hierarchy is for readability, not ceremony.
- **Sheet organization.** Order sheets by signal flow (input/protection → processing → output; power sheets grouped). Give every sheet a descriptive title, project name, revision and sheet number. Split overcrowded sheets instead of shrinking them.
- **Naming.** All visible names MUST be English and follow the AES Naming Standard. Name by function, not implementation: `SENS_PWR_EN`, not `GPIO7`.
- **Reference designators.** Use conventional class letters (R, C, L, D, Q, U, J, TP, MH…), unique across the whole design. Once a revision is released, designators are frozen — never reuse a released designator for a different function.
- **Net naming.** Every power rail, bus, clock, reset and sheet-crossing net MUST have an explicit name. Power nets encode voltage and domain (`+3V3`, `+3V3_ANA`, `VBAT`); grounds are named per domain (`GND`, `AGND`, `PGND`) and joined at explicit net-tie points only. Active-low marking MUST use one project-wide convention (`_N`). Interface nets SHOULD carry their AES interface prefix (`UIF_`, `HIF_`).
- **Signal grouping.** Draw buses as buses, differential pairs adjacent and named `_P`/`_N`. Split large IC symbols by function, not physical pin order.
- **Power domains.** Every domain's source, voltage, consumers and enable control MUST be traceable in the schematic. Released designs with multiple power domains SHOULD additionally document a power tree or power-domain table (voltage, source, budget, protection, enable, sequencing) — for a single-rail prototype, clear schematic net names and a design-notes line are enough. Power sheets SHOULD state the budget: expected load, regulator limit, protection threshold.
- **Documentation in the schematic.** Annotate intent where it applies: value choices, DNP parts (marked in schematic, BOM and assembly outputs), current budgets, expected voltages at key nodes for bring-up, and all layout constraints (impedance, matching, keep-outs, kelvin connections). Layout requirements MUST be written down, never verbal.
- **Reusable blocks.** Proven circuits (protection front-ends, regulator stages, identity EEPROM) SHOULD live in a shared, versioned library and be reused, not redrawn. A reused block MUST still be re-verified in its new context, and fixes SHOULD flow back to the shared library.

---

## 3. Component Selection

Components are selected for a product lifetime of decades: availability and margin outweigh unit cost.

- **Controller platform.** AES names the Platform's default controller families per workload profile ([AES-ARCH-001](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-arch-001-default-controller-platform)); this guide stays part-number-neutral and does not repeat them. Another platform is allowed where the requirements warrant it, with the reasons in the design notes.
- **Packages.** Default passives to 0402, 0603 or 0805. Where board space permits, 0805 is a perfectly acceptable choice and SHOULD be preferred for serviceability — it is the easiest size to inspect and hand-rework. Smaller than 0402 MUST be justified by density. Prefer packages with inspectable, reworkable joints; BGA/QFN are acceptable but MUST be paired with the inspection and test measures of Sections 12–13. Minimize the number of distinct values and packages.
- **Availability.** For Released hardware, every BOM line SHOULD have at least two independent sources, and single-source parts MUST be flagged in the BOM with a substitution plan. During prototyping, alternate sourcing plans are not required — but avoid designing critical circuits around parts you already know are hard to buy. Generic parts (passives, standard logic) SHOULD be specified by parameters, not locked to one manufacturer.
- **Lifecycle.** Check lifecycle status at design time. Prefer industrial/automotive series with long production commitments over consumer series. For risky parts, choose footprints that accept known alternates so substitution does not force a respin.
- **Temperature grade.** Parts MUST cover the product's full operating range including self-heating and enclosure rise. Industrial grade (−40…+85 °C) SHOULD be the default. Evaluate temperature-sensitive parameters (ceramic capacitance loss, electrolytic lifetime, oscillator drift) at the extremes, not at 25 °C.
- **Derating.** Every part MUST stay within manufacturer ratings under worst case, and designs SHOULD meet these targets:

| Stress | Target (of rating) |
|---|---|
| Resistor power | ≤ 50 % |
| Ceramic capacitor DC voltage | ≤ 50–80 %; use capacitance-under-bias as the design value |
| Electrolytic / polymer voltage | ≤ 80 % |
| Tantalum voltage | ≤ 50 % |
| Inductor current | ≤ 80 % of I_sat at max temperature |
| Semiconductor junction temp | ≤ 110 °C or 80 % of max, whichever is lower |
| MOSFET / diode voltage | ≤ 80 % incl. transients |
| Connector contact current | ≤ 50–70 % |

  Voltage derating MUST be checked against real worst-case waveforms (startup overshoot, inductive kick), and current/power derating against real thermal conditions (enclosure, copper, airflow), not 25 °C free-air datasheet conditions. Electrolytics in long-life products MUST be lifetime-calculated at actual temperature and ripple.

---

## 4. PCB Layout

- **Placement first.** Good layout is 80 % placement. Place by functional block following the schematic architecture; keep each block's components together; place critical parts (decoupling, feedback, protection) before general routing. Protection components sit at the connector they protect, before anything else.
- **Routing strategy.** Route critical nets first (power, clocks, differential pairs, sensitive analog), then general signals. Avoid stubs on fast signals. Keep trace width consistent with current and impedance requirements; neck down only where necessary.
- **Return currents.** Every signal has a return path — design it. High-frequency return current flows directly under its trace; a signal MUST NOT cross a split or gap in its reference plane. If a signal changes reference layers, a stitching via (or stitching capacitor between different-voltage references) SHOULD be placed at the transition.
- **Grounding and planes.** Choose layer count from what the board actually needs — signal integrity, density, EMC environment — not from a blanket rule: a simple low-speed board is perfectly served by two layers with a disciplined ground pour, while switching converters, fast digital or sensitive analog usually earn four layers with a continuous ground plane. Two-layer boards keep the most continuous ground pour achievable. Do not split the ground plane by default — keep one solid GND and control return paths by *placement*. Separate ground domains (AGND, PGND, CHASSIS) only when there is a real reason, joined at a single defined point.
- **Power distribution.** Use planes or wide pours for power; size copper for current and thermal rise. Keep high-current loops small and off sensitive areas.
- **Analog/digital separation.** Separate by region, not by split planes: analog circuitry in one area over unbroken ground, digital in another, and no digital routing under or through the analog region.
- **Impedance control.** Controlled-impedance nets MUST be identified in the design files with target impedance and stackup, and the fab drawing MUST require the manufacturer to meet them.
- **Decoupling.** Every supply pin gets local decoupling; the smallest capacitor sits closest to the pin with the shortest possible loop to ground (via directly at the pad or next to it). Bulk capacitance per rail per board region. Decoupling placement is a placement-stage task, not a cleanup task.
- **Crystals.** Crystals and their load capacitors sit immediately at the IC, over solid ground, with no signals routed underneath and a guard of ground around the region. Keep the crystal away from board edges and heat sources.
- **Switching regulators.** Identify the hot loop (input cap – switch – diode/sync FET – back to input cap) and make it as small as physically possible on one layer. Keep the switch node small and away from feedback and sensitive signals. Feedback routes away from the inductor and switch node, sensed at the point of load. Follow the regulator vendor's layout guidance — it is the one place vendor documentation is normative.
- **Thermal.** Give power parts copper area and thermal vias per Section 10; check that neighboring components tolerate the local temperature rise.
- **Creepage and clearance.** Spacings for anything above SELV levels MUST follow the applicable safety standard (IPC-2221 as baseline) for the working voltage, pollution degree and material group, on both outer and inner layers. Mark required spacings as design rules, not by eye.
- **Mounting holes.** Mounting holes MUST match the mechanical design (Section 11), with keep-outs for screw heads and washers, and a defined grounding decision per hole (grounded to chassis, capacitively coupled, or isolated — chosen deliberately, see Section 5).
- **Connectors.** External connectors SHOULD be grouped on one board edge (noise and cable management); orientation, polarity and mechanical strain relief are placement decisions. Leave room for mating plugs, fingers and cable bend radius.

---

## 5. EMC / EMI Design

EMC is designed in, not tested in. The board that passes is the board where every fast current loop is small and every cable exit is filtered.

- **Emissions.** Minimize loop areas of fast signals and switching converters; keep clocks short and away from connectors and board edges; use the slowest edge rate that meets timing (series termination resistors, drive-strength settings). Solid ground planes and small hot loops do more than any later fix.
- **Immunity.** Assume cables bring interference in. Every signal entering or leaving the board SHOULD be filtered or protected at its connector; digital inputs from off-board SHOULD have defined static levels (pull resistors) and, where lines are long, RC filtering or hysteresis.
- **Filtering.** Filters belong at the board/cable boundary, referenced to a clean ground, with input and output routing kept apart so the filter is not bypassed by coupling. Common-mode chokes SHOULD be considered on cable interfaces that fail or risk failing conducted limits.
- **ESD protection.** Every externally accessible signal and connector shell MUST have an ESD strategy: TVS/clamp devices placed directly at the connector, with a short, low-inductance path to ground, before the protected line branches into the board. Match protection capacitance to the signal (low-capacitance devices on high-speed lines).
- **Surge protection.** Interfaces leaving the enclosure on long cables (power inputs, field wiring, Ethernet) SHOULD have surge-rated protection staged with the ESD/filtering elements, coordinated so the coarse protection operates before the fine protection dies.
- **Ferrite beads.** Use ferrites deliberately: to isolate a noisy domain from a quiet rail, chosen by impedance at the offending frequency and rated for the DC current. Do not scatter ferrites as a reflex — a ferrite plus decoupling forms a resonant circuit; damp it or verify it.
- **Shields and chassis.** Connector shells and cable shields SHOULD bond to chassis/shield ground at the point of entry, 360° where possible, not through a long pigtail to signal ground. The chassis-to-GND connection is a deliberate design decision (direct, capacitive, or single-point) and MUST be documented.
- **Cable routing and isolation.** Keep noisy cables (motor, switching power) and sensitive cables (analog sensors) physically separated and on separate connectors. On-board, keep noisy circuit regions away from I/O areas; if a noisy signal must cross the board, route it on inner layers between grounds.

### 5.1 AURIORA Event Link (AEL) Ports and Routers

These rules apply to any board that provides an AEL IN or AEL OUT link on its Module Port (§5.3), and to an AEL router — the event-routing part of a Module Hub, or a standalone AEL Hub. What AEL is — a Module-level, point-to-point differential link carrying a small typed event frame and no data — its topology, event identifiers, bindings, timing and routing rules are defined by AES ([Interfaces and Versioning §4](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/05-interfaces-and-versioning.md#4-auriora-event-link), [AEL specification](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/interfaces/ael.md)); this section states what the hardware MUST do to honor them. Connector, pinout, frame encoding and the open electrical items live in the specification and are not repeated here.

**Ports**

- **One transceiver class, two independent links.** Implement AEL OUT with the differential driver and AEL IN with the differential receiver of a 3.3 V-compatible transceiver class with separate driver and receiver pairs, a **true fail-safe receiver** — a defined mark output for open, shorted and idle inputs, established from the device's own specification rather than assumed of every RS-422/RS-485 part — and a rated data rate with margin above the 1 Mbit/s AEL baseline. The normative electrical profile (an RS-485-compatible driver/receiver contract is the intended direction) is still open in the specification; choose parts that will meet it either way and record the assumption. A half-duplex transceiver MUST NOT serve both ports through one device: the two ports are independent links, never a shared bus, and there is no direction control to drive. Specify the part by parameters (Section 3), not by one manufacturer.
- **Termination at the receiver only.** Each AEL IN carries its own differential termination (nominally 120 Ω, to be validated against the transceiver class and the cable). AEL OUT and router outputs MUST NOT be terminated as receivers. Place the termination at the receiver input, after the protection.
- **Idle is mark — make the hardware land in it.** AEL idle is the UART mark state, logic HIGH at the receiver output. An AEL IN MUST resolve to mark with the cable unplugged, with the far end unpowered or in reset, and with the pair shorted, and MUST NOT present a start-bit edge to the receiving peripheral during cable insertion or removal. A true fail-safe receiver does this by design; where the chosen part does not guarantee it, add biasing that does, and verify every case on the bench with the actual receiving peripheral armed — a framing error (a break) is acceptable and is counted; a valid-looking frame is not.
- **No frame from power or reset.** An AEL OUT transmitter MUST hold a defined idle line through power-up, power-down, reset and firmware start. Where the transceiver's driver-enable or the controller's pins are undefined during those transitions, gate the driver so the line stays idle (or undriven and therefore idle at the receiver) until firmware has configured the peripheral. Verify with power and reset cycling while a receiver counts frames: the count MUST stay at zero.
- **Protection first.** An AEL link leaves the board through the Module Port, an external, user-wired connector: TVS/clamp protection sits directly at that connector, before the termination and the transceiver, with a short low-inductance ground path (the ESD rules above). The transceiver's internal ESD rating is not the protection strategy. Where cables leave the enclosure for long runs, stage surge-rated protection ahead of it.
- **No back-drive.** An unpowered board MUST NOT be driven through, and MUST NOT source current into, its AEL pins; check the transceiver's unpowered bus-pin behavior and the protection network's leakage paths.
- **Layout and naming.** Route `AEL_P`/`AEL_N` as a coupled, symmetric pair from connector to transceiver (Section 7 differential-pair rules). Keep the protection → termination → transceiver chain short and in that order. Name the nets `AEL_IN_P`/`AEL_IN_N` and `AEL_OUT_P`/`AEL_OUT_N`.
- **Frame path through hardware, not through a task.** Connect the AEL IN receiver output to a controller peripheral that receives the 8N1 characters into DMA or a FIFO, and give the design a way to timestamp the frame reference instant — the first start-bit edge after the idle gap — in hardware: for example a timer input-capture channel or a programmable-I/O block on the same line, or a UART feature that captures the start edge. The specification fixes the reference instant, not the capture mechanism. Feed the AEL OUT driver from a hardware serial output whose transmission start can be timed against a timer or the Module's sample clock. A GPIO bit-banged from a task is not an AEL port on either side. The firmware side is in the [Firmware Style Guide](https://github.com/auriora-org/auriora-firmware-style-guide) §14.2.
- **Connector safety.** AEL has no connector of its own: the pairs leave the board through the Module Port, whose connector, cable and cross-mating rules are §5.3. Label nothing `AEL IN` or `AEL OUT` on the enclosure — those are link names; the connector is the Module Port.

**Routers**

- **A controller in the event path is expected — and it is a real-time subsystem.** An AEL router receives, validates and forwards frames in firmware or programmable logic; that is the design, not a compromise. What the hardware owes in return is a *characterized* path: every port's receive and transmit MUST go through hardware serial/DMA or programmable-I/O peripherals with hardware timestamping of the frame reference instant, so that forwarding latency depends on the router's event load and not on what its management firmware is doing. Choose the controller and its peripheral set so that every AEL port has its own hardware receiver and transmitter; multiplexing ports through one peripheral in software defeats the timing contract.
- **Measure under load, publish the envelope.** Forwarding latency (ingress reference instant to egress reference instant, minimum/typical/maximum), forwarding jitter and channel-to-channel skew of one fan-out MUST be measured with all ports active, with MCL management traffic running, and at the router's documented maximum event rate — not on an idle bench. The figures are part of the product's documentation for a Released router. A router is never described as zero-delay or load-independent.
- **Fan-out launches together.** Where one routing decision selects several outputs, the hardware MUST be able to start their transmissions within the skew the product documents — for example by arming several DMA-fed transmitters from one timer event rather than by writing to them in sequence from a loop.
- **Saturate it on purpose.** Test queue overflow: drive more events than the documented rate into several ingress ports selecting the same output, and confirm that the router drops what its documentation says it drops, counts every drop, and neither corrupts frames on the wire nor stalls other ports.
- **Per-port containment.** A short, an unpowered Module, a continuously toggling line or a cable removed mid-frame on one port MUST NOT disturb reception, transmission or timing on another. Each port has its own driver, receiver, termination and protection; shared supplies and shared logic are analyzed for the fault paths that would let one port pull down another.
- **Route changes are silent on the wire.** Committing a new route table, or aborting a deployment, MUST NOT glitch any link or emit a partial or spurious frame; transmit paths are switched between frames, never inside one.
- **Idle on loss of power.** Loss of router power or a router reset MUST leave every output in AEL idle — not toggling, not driving a stuck level that the receiving end could read as a frame — and the router does not reinstate an active route table on its own when power returns.
- **Hub-to-Hub links are ports.** A Hub-to-Hub AEL link is an AEL OUT on one router and an AEL IN on the other, built, protected and terminated exactly as the Module-facing ports above, on the same Module Port connector and joined by the same AURIORA Link Cable (§5.3) — one cable carries a link in each direction; label them `HUB LINK`. Their count is a product decision. The same cable from a downstream Hub's link port to an upstream Hub's Module Port also carries cascaded `MCL` on its third pair; nothing about the AEL hardware changes ([`AES-HUB-005`](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-hub-005-cascading-and-path-addressing)).

---

### 5.2 Module Hub Host-Facing Interface

These rules apply to the board of a Module Hub, or of any product that provides the host-facing interface AES requires of a Hub ([AES-HUB-003](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-hub-003-host-facing-interface-and-traffic-separation)) — the connection through which the system host reaches the Hub and the Modules behind it. AES defines the role; this section states what the hardware MUST do to honor it. The MCI binding, the USB class and the descriptors are not hardware decisions and are not repeated here.

- **A dedicated connector, and the Hub is the device.** The host-facing interface has its own connector, distinct from every Module Port and Hub-to-Hub link; it consumes no port. Where it is USB, the Hub implements the USB *device* side toward the computer — a device-role connector, or a dual-role connector configured as device — never a connector that reads as a USB host port on the Hub.
- **Bench ground meets computer ground here.** The host-facing connector is where a laptop's ground, its power adapter and whatever else it is connected to enter the bench. Analyze the common-mode path from that ground through the Hub to every Module Port's differential links and shield; decide deliberately whether the interface is galvanically isolated, and where it is not, provide the return path and protection that keep a ground shift or a host-side fault from disturbing `MCL` or AEL. Document the decision either way.
- **Signal integrity per Section 7.** Matched 90 Ω pair, low-capacitance ESD protection at the connector, common-mode choke and protection in the specified order, short and continuously referenced. The bulk traffic this link carries is the highest sustained data rate on the board and is routed as such.
- **Not the bench's power supply — and not the Modules' either.** The host-facing interface MUST NOT be the source of the power the Hub's AEL router and `MCL` concentrator need to meet their timing contracts; that comes from the Hub's own power input, for which a Module Power Interface input is the preferred form (§6.2). A Hub supplies no operating power to the Modules on its Module Ports ([AES-HUB-001](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-hub-001-module-hub-scope)): each Module has its own power input, and a Module Port carries signals only (§5.3). Whether the Hub's own logic may run from the host connection is a product decision; where it may, respect the USB current limits (Section 6), make the mode observable, and document what is and is not available in it — a Hub that meets its timing contract only on external power says so.
- **Host activity is not router activity.** USB traffic, enumeration, suspend and resume MUST NOT disturb the AEL router's supplies, clocks or peripherals; the §5.1 timing measurements are made with the host-facing link carrying bulk traffic at full rate.
- **Observable without the computer.** Provide link-state and activity indication for the host-facing interface so that a person at the bench can see that the Hub is connected and talking, independent of what the host shows.
- **Verify the roles on the bench.** Enumerate the Hub against at least two different host computers and confirm device-role behavior, inrush within limits, no back-drive of the Hub from the host connection when the Hub's own power is off unless that mode is designed and documented, and no change in AEL forwarding latency between an idle and a saturated host link.

### 5.3 Module Port and AURIORA Link Cable

These rules apply to any board that provides a Module Port — a Module, a Module Hub's Module Ports and its Hub-to-Hub link ports — and to anyone building an AURIORA Link Cable. What the port is and carries — one M8 8-position A-coded female connector with `MCL` (half duplex, one pair), AEL IN, AEL OUT, `GND` and one reserved contact, one contact assignment on every device, the crossover Link Cable, direct 1:1 operation without a Hub, no primary power — is defined by AES ([AES-MOD-005](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-mod-005-external-interfaces-and-power-separation), [Architecture §7.1](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#71-the-module-port), [Module Port specification](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/interfaces/module-port.md)); this section states what the hardware MUST do to honor it.

- **Female on the device, one assignment everywhere.** Fit the M8 8-position A-coded female connector on every Module Port, Module-side and Hub-side alike, wired to the same contact assignment; there is no mirrored Hub-side pinout — the crossover is in the cable. Position numbering is still open in the specification: build to the contact roles, and record the numbering assumption in the design notes, on the schematic and on the cable drawing, so that the Platform's assignment is a one-line change when it comes.
- **Three links, three transceivers.** `MCL`, AEL IN and AEL OUT each have their own driver and/or receiver, termination and protection. `MCL` is half duplex on one pair: a transceiver with driver enable, the Hub end driving as the polling master and the Module end answering; keep the direction-control line under firmware control with a defined idle (receive) state through reset and firmware start. AEL is two simplex links and follows §5.1 unchanged — never fold AEL IN and AEL OUT onto one half-duplex pair to save a transceiver; the pair is in the connector anyway and the collision it would allow has no recovery.
- **Every Hub port is the same hardware; the role is firmware.** A Module Port and a Hub-to-Hub link port on a Hub use identical `MCL` transceivers with driver and receiver enables under firmware control; the master role on Module Ports and the responder role on link ports ([`AES-HUB-005`](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-hub-005-cascading-and-path-addressing)) is a firmware property of the port index, and no jumper, strap or component difference distinguishes the two. Build all ports from one layout block. Label each port on the enclosure with the index the Hub declares over its management contract ([`AES-HUB-004`](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-hub-004-hub-identity-and-capability-discovery)) and its role; a *Downstream Hub* label on a Module Port is a label, not a different port, and a Module plugged into it works.
- **Two masters must not hurt anything.** Two Hubs' Module Ports cabled together put two `MCL` drivers on one pair. The transceiver choice and termination MUST survive that indefinitely — it is a special case of the wrong-cable rule above — so that the fault firmware reports is the only consequence.
- **Survive the wrong cable.** A generic straight-through M8 8-position cable puts two AEL drivers on one pair and two terminated receivers on another. Choose transceivers whose driver short-circuit and contention limits cover that indefinitely, verify it on the bench for the product's documented soak time, and confirm zero valid AEL frames while the cable is connected, plugged and unplugged. No cable-detection mechanism is added: the fault is visible as an `MCL` link that never sees its master and an AEL IN that never leaves idle.
- **`RESERVED` is open.** No pull, no ground, no test signal, no "temporary" use. Where the ESD network is a multi-channel part its channel may be populated; nothing else touches the contact.
- **Reference and shield are two conductors.** `GND` is the signal reference of all three pairs and carries no intentional current; the connector shell bonds the cable shield to chassis or shield ground at the point of entry (Section 5 shield rule) and is not the same conductor as `GND` in the cable. Record the chassis-to-`GND` decision.
- **Live insertion, every port.** A Module is connected and disconnected on a running bench. Protection, fail-safe biasing and driver gating MUST hold through insertion and removal with the bench armed; on a Hub, verify that plugging and unplugging one port disturbs no other port's links (§5.1 per-port containment).
- **Nothing powers anything.** No contact carries a Module's primary power and none is used as a power return; a Hub's Module Port draws nothing from and supplies nothing to the Module beyond transceiver bias. A design that needs power at the far end of a Link Cable has misidentified the interface — see §6.2.
- **Build the Link Cable to the specification, not to a catalogue.** Male-to-male M8 8-position; `AEL_OUT_P`/`AEL_OUT_N` of one end to `AEL_IN_P`/`AEL_IN_N` of the other and vice versa, polarity preserved; `MCL_P`/`MCL_N`, `GND` and `RESERVED` straight; one twisted pair per link; overall shield 360° to both shells; marked `AURIORA LINK`. Impedance, conductor size and maximum length are open in the specification — build to the AEL termination reference (120 Ω) and record the assumption. Test continuity and pair assignment against the specification's table before the cable meets a board, and confirm both orientations read identically.
- **Cross-mating check.** The 8-position insert does not mate with the 3- and 4-position M8 inserts; verify on the actual parts that a power plug does not enter a Module Port and a Link Cable does not enter the power input, and record it with the Section 11 assembly review.

---

## 6. Power Design

- **Power tree.** Every design MUST have a documented power tree: every rail, its source, voltage, current budget (typical and worst case), protection and enable control. Budgets include margin per Section 3 derating.
- **Regulator choice.** Use switching regulators for efficiency and thermal reasons, followed by LDO post-regulation where noise matters. Verify stability with the actual output capacitor type and value (ESR range), load range and input range — not just the datasheet typical circuit.
- **Sequencing.** Where rails have sequencing or ramp requirements (FPGAs, SoCs, mixed-voltage I/O), the sequence MUST be designed and documented, including behavior on power-down and brown-out, not only power-up.
- **Battery systems.** Battery-powered designs MUST define charge termination, over/under-voltage and over-current protection, and temperature limits appropriate to the chemistry; protection MUST exist independently of software. Design the quiescent-current budget explicitly — sleep current is a design requirement, not an outcome.
- **USB power.** USB-powered designs MUST respect inrush and current-draw limits of the USB specification they claim, tolerate the real voltage range at the connector after cable drop, and negotiate higher power before drawing it.
- **Reverse polarity.** Any user-wireable or user-pluggable power input MUST have reverse-polarity protection (P-FET preferred over series diode for efficiency; diode acceptable at low current).
- **Overcurrent.** Every board-level power input MUST have overcurrent protection (fuse, PTC or current-limited switch) rated so that a board fault cannot ignite the board or the upstream supply. Downstream domains that can be shorted by the user (expansion connectors, unit interfaces) SHOULD have their own current limiting.
- **Transients.** Power inputs MUST survive the transients of their environment: hot-plug inrush, inductive load dump, ESD. TVS on power entry SHOULD be default. Verify that input capacitance plus hot-plug does not violate connector or upstream limits.

### 6.1 Unit Interface Power and Readiness

These rules apply to any board implementing an AURIORA Unit Interface (`UIF_`), on the host side or the Unit side. The interface signal set, the discovery/activation sequence and `UIF_READY` semantics are defined by AES ([Interfaces and Versioning](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/05-interfaces-and-versioning.md), [AES-UNIT-007](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-unit-007-deterministic-discovery-and-activation-sequence)); this section states what the hardware MUST do to honor them.

- **Local power switching.** `UIF_PWR_EN` MUST control the Unit's functional power domain locally, through a load switch, regulator enable or equivalent on the Unit — not through a separate host-side power pin. The connector MUST NOT require separate discovery and functional power pins.
- **Discovery while disabled.** The Unit EEPROM and the minimum circuitry required for discovery MUST remain readable from `UIF_PWR_VIN` while the functional domain is disabled (`UIF_PWR_EN` LOW).
- **No back-powering.** Disabled Unit circuitry MUST NOT be back-powered through `UIF_I2C_*`, `UIF_SPI_*`, GPIO, interrupt, reset, synchronization or other interface signals while the functional domain is off. Check every signal that could source current into an unpowered rail.
- **Defined `UIF_READY` state.** `UIF_READY` is active-HIGH and MUST have a defined LOW state when the Unit is absent, disabled, starting or faulty. The host MUST provide that LOW state (pull-down or equivalent) when no Unit is connected or the Unit functional domain is disabled. A Passive Unit MAY derive `UIF_READY` from its switched functional power domain (pull-up or equivalent); a Managed Unit's controller drives it.
- **No `UIF_READY` contention.** A Managed Unit GPIO driving `UIF_READY` MUST NOT cause contention during reset, boot, unpowered or partially powered states — verify the pin is not driven high before its rail is valid.
- **Verify power before enabling.** The host MUST verify the Unit's EEPROM power metadata (required voltage, startup/operating/discovery currents) against its own capability before asserting `UIF_PWR_EN`; if the Unit exceeds host capability, `UIF_PWR_EN` stays LOW.
- **Ports are not additive.** The number of Unit ports a host provides is not a promise that all of them can draw their per-port maximum at once. Size the shared supply for the intended simultaneous load rather than for port count × per-port rating, and document which combinations the design supports ([AES-MOD-003](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-mod-003-unit-compatibility-matrix)). A Module that exposes more ports than it can drive together is a legitimate design; an undocumented one is not.
- **Discovery load is always present.** Every connected Unit draws its discovery-state current from `UIF_PWR_VIN` even while every functional domain is disabled. On a high-port-count host that sum is a real base load and belongs in the power tree — small is not zero.
- **Startup peaks overlap.** Unit start-up current can exceed operating current, and enabling several Units together stacks those peaks onto one supply. Either size for the overlap or stagger the enables; staggering helps a transient limit only, never a sustained shortfall.
- **Shared or per-port protection.** Choose deliberately. Shared upstream protection is often sufficient where a Unit fault only has to be contained; per-port current limiting earns its cost where one faulty Unit or damaged cable must not disturb the others, or where port faults must be individually diagnosable. Either way the physical protection is what contains a Unit that draws more than it declared — EEPROM power metadata is unauthenticated input, never a safety barrier ([AES-EEPROM-007](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/06-eeprom-metadata.md#aes-eeprom-007-metadata-trust-model)).
- **No presence pin.** Physical presence is determined by successful EEPROM discovery. Do not add a `UIF_PRESENT`/`UIF_PRESENT_N` or equivalent Unit-presence pin.
- **Document the electricals per profile.** Pull-up/pull-down values, series resistance, voltage thresholds, leakage current, startup timing and power sequencing MUST be documented in the applicable Unit Interface Profile specification, together with connector keying, pin numbering, voltage and current limits, ESD protection, hot-plug behavior and mechanical constraints. Downstream Unit-interface power domains SHOULD have their own current limiting per the general power rules above.

### 6.2 Module Power Interface and USB Power

These rules apply to any board that carries a Module's primary external power input, to a Module Hub's own power input, and to any source that provides Module Power Interface outputs. What the interface is — an M8 3-position A-coded **male** input on the Module at a nominal 12 V DC, pin 1 `MPI_VIN`, pin 3 `MPI_RTN`, pin 4 reserved, fed from a **female** source output over a male-to-female cable; every rail regulated locally; no primary power on the Module Port; USB power optional and declared — is defined by AES ([AES-MOD-005](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/03-architecture.md#aes-mod-005-external-interfaces-and-power-separation), [Module Power Interface specification](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/interfaces/module-power.md)); this section states what the hardware MUST do to honor it. The accepted input-voltage range and the current limit are open in the specification: design to a documented range and current, state them in the power tree, and expect the Platform to fix them.

- **Male on the Module — and never live from the inside.** The input is a panel-mount male M8 3-position connector, so an energized, unplugged cable end is female and exposes nothing. That property is only real if the Module never energizes its own input: the input path MUST be unidirectional toward the Module — an ideal-diode controller, a P-FET ORing stage or an equivalent power-path element, not a plain wire and not a Schottky whose leakage and drop have not been checked — so that `MPI_VIN` measures 0 V whenever the Module runs from USB, a battery or any other source with no cable in the input.
- **Positions 1, 3, 4 — the insert's own numbering.** Wire `MPI_VIN` to position 1 and `MPI_RTN` to position 3; leave position 4 open on the board (no pull, no ground, no sense) and tolerant of any voltage between `MPI_RTN` and `MPI_VIN`, because a future source may drive it. Do not renumber a standard insert, and do not put anything else on the M8 3-position A-coded form on the same product (below).
- **Every general power-input rule of Section 6 applies**, at every maturity level: reverse-polarity protection (P-FET preferred), overcurrent protection rated so that a board fault cannot ignite the board or the source's cable, TVS at power entry, and an inrush that hot-plugs into a live source without tripping it — hot-plug is a normal operation for this input. Verify that input capacitance plus hot-plug stays inside the source's and the connector's limits.
- **Regulate locally, budget honestly.** Every internal rail is generated on the Module from `MPI_VIN`; no source rail is used directly and no Module rail appears on the connector. The power tree states typical, maximum and start-up current at nominal voltage and the input-voltage range the regulators were designed to — those figures are what the Platform will fix, and what a power source's designer needs today.
- **Two sources, no back-feed either way.** A Module that accepts both the Module Power Interface and USB power MUST implement power-path protection in both directions: `MPI_VIN` never appears on `VBUS`, `VBUS` never appears on `MPI_VIN`, and the documented source wins when both are present without a glitch on the rails during the hand-over. Where USB powering is offered, the USB power rules above apply to that path — inrush, negotiated current, real cable-drop voltage — and the modes that work on USB alone are documented and observable; a Module that meets its full specification only on the Module Power Interface says so.
- **USB is not the deployed supply.** Design the Module to run its full function from the Module Power Interface alone. USB powering is a convenience for the bench, service and firmware work, and a product decision for low-power Modules — never the assumption a high-power Module's power tree is built on.
- **Sources: female, protected, per output.** A power source — a bench adapter, a future distributor — provides one female M8 3-position output per Module, protects each output against overcurrent and short circuit independently of the Module, and leaves position 4 open. Passive splitting of one output to several Modules is not a source. What an output does after a fault is a product decision; that it does not take the other outputs with it is not.
- **Keep other M8 3-position ports off the Module.** The power input is the only M8 3-position A-coded connector the Platform defines. Where a product family still carries that form on a sensor or electrode input, make mis-mating with the power input mechanically impossible or electrically harmless in both directions and record the analysis; the preferred answer is a different connector for the sensor.
- **Unit power is a different interface.** `UIF_PWR_VIN`/`UIF_PWR_EN` (§6.1) are the Module-to-Unit managed power inside a Module, derived from the Module's regulated rails — never `MPI_VIN` wired straight through, never presented on an external connector.
- **Verify on the bench.** Reverse polarity applied without damage; short and overcurrent contained; 100 hot-plugs into a live source without a trip or a reset of the source's other outputs; `MPI_VIN` at 0 V with the Module on USB; `VBUS` undriven with the Module on `MPI_VIN`; position 4 open and swept 0 V to `MPI_VIN` without effect; full function on `MPI_VIN` alone.

---

## 7. High-Speed Design

"High-speed" is defined by edge rate, not bit rate: when the edge is short relative to the trace, transmission-line rules apply.

- **Impedance.** High-speed interfaces MUST be routed at their specified impedance (differential pairs typically 90 Ω USB, 100 Ω Ethernet/LVDS/MIPI) over a continuous reference plane, with the stackup documented and communicated to the fab.
- **Differential pairs.** Route pairs coupled and symmetric: same layer, same length, same via count, symmetric around obstacles. Polarity swaps happen at the source (if the silicon supports lane inversion) or in the schematic — not by crossing traces mid-route.
- **Length matching.** Match within pairs first (intra-pair skew is what matters most), then between pairs/signals only as tightly as the interface actually requires. Meaningless picosecond-matching of slow buses adds serpentines and cost without benefit.
- **Return paths.** Never route a high-speed signal across a plane split or void. Place ground stitching vias next to every layer-change via. Connectors carrying high-speed signals need ground pins/vias adjacent to the signal pins.
- **USB.** Keep the pair short and matched, protect with low-capacitance ESD devices at the connector, and place any common-mode choke and protection in the specified order between connector and PHY.
- **Ethernet.** Maintain isolation spacing across the magnetics — no planes or signals crossing the isolation gap; terminate and reference the line side per the magnetics and PHY requirements.
- **SPI / SDIO / parallel buses.** These become high-speed at modern clock rates: keep them short, reference them continuously, series-terminate at the driver where edges ring, and route clock as the most protected member of the group.
- **MIPI and similar source-synchronous serial links.** Follow the lane impedance and intra-pair skew budget strictly; keep lanes on one layer where possible and minimize vias.

---

## 8. Analog Design

- **Sensor interfaces.** Define the full signal chain on paper first: sensor output range and impedance, gain, filtering, ADC input range, noise budget. Every stage's error contribution SHOULD be budgeted so accuracy claims are designed, not hoped for.
- **ADC/DAC layout.** Converters live on the analog side of the board over unbroken ground; their analog supply is filtered from the digital rail; reference and analog input routing is short and away from digital traffic. Drive ADC inputs from the impedance the converter requires (buffer or RC per datasheet), and place the sampling capacitor's charge reservoir close.
- **Instrumentation amplifiers.** For small differential signals (bridges, biosignals, current shunts), use an instrumentation amplifier topology with proper input bias paths, matched input filtering, and gain set by a quality resistor. Protect inputs against overvoltage without degrading CMRR.
- **Low-noise practice.** Bandwidth costs noise — limit it at every stage to what the signal needs. Choose the first-stage amplifier for the source impedance (voltage noise vs current noise). Keep bias currents' DC paths defined. Power the analog front-end from clean, post-regulated rails.
- **Guarding.** High-impedance nodes (electrometer inputs, pH/electrode interfaces, femtoamp bias paths) SHOULD be guarded: a driven guard ring at the node's potential surrounding the node, leakage-relevant solder-mask decisions made deliberately, and cleanliness requirements stated in manufacturing notes.
- **Reference routing.** Treat voltage references as analog signals, not power: star-route or kelvin-connect the reference to its consumers, decouple per the reference's stability requirements, and never share a reference trace with load current.

---

## 9. RF Design

- **Antenna placement.** Antennas need the keep-out and ground-plane geometry their design assumes: at a board edge or corner, with the specified clearance from metal, plastic, batteries and cables in all three dimensions. Antenna placement is decided at concept stage together with the enclosure — it cannot be fixed later.
- **Keep-outs.** Antenna keep-out zones MUST be drawn as explicit mechanical/electrical keep-out objects in the CAD data, covering all copper layers, components and the enclosure region, so they survive later edits.
- **Matching networks.** Every antenna feed SHOULD have a pi-network footprint (even if initially bypassed) between radio and antenna, placed in the 50 Ω line with solid ground underneath, because matching is tuned against the real enclosure at integration time.
- **RF ground.** RF sections need continuous, via-stitched ground: stitch the ground pour around RF traces and the module region at intervals short relative to the wavelength, and connect module ground pads with multiple vias.
- **Shielding.** Provide a shield-can footprint (or the option of one) over sensitive RF sections when the product's EMC environment is uncertain; a fence of ground vias under the can perimeter makes the shield effective.
- **Module integration.** When using certified wireless modules, follow the module's integration requirements exactly (ground plane size, antenna clearance, decoupling) — certification validity depends on it. Route the 50 Ω feed as a controlled-impedance line regardless of its length.

---

## 10. Thermal Design

- **Budget first.** Identify every component dissipating significant power, estimate worst-case dissipation, and verify junction temperatures against the derating limits of Section 3 using realistic ambient (inside the enclosure, hottest documented environment).
- **Copper is the heatsink.** On most boards, heat leaves through copper: give power parts generous connected copper on multiple layers. A thermal pad MUST connect to its copper area with an adequate array of thermal vias (filled or plugged where required by assembly).
- **Thermal vias.** Use enough vias under thermal pads to make via resistance small compared to the rest of the path; more small vias beat few large ones.
- **Placement.** Spread heat sources; keep them away from temperature-sensitive parts (oscillators, precision references, electrolytics, batteries) and away from connectors people touch.
- **Heatsinks and interfaces.** Where copper is not enough, plan the heatsink, its mounting force, and the thermal interface material as part of the mechanical design, with tolerances that guarantee contact.
- **Enclosure.** The enclosure is part of the thermal path: verify airflow or conduction paths, and re-check the thermal budget with the enclosure closed. A design verified open-air MUST be re-verified in its enclosure.

---

## 11. Mechanical Integration

- **Dimensions and datums.** PCB outline, mounting holes and connector positions MUST be defined against the mechanical design's datum system, with tolerances agreed with the enclosure design — not nominal-only.
- **Mounting.** Mounting points SHOULD be near board corners and near heavy components and connectors (they take the mechanical load). Keep component keep-outs around holes for screw heads, standoffs and tools.
- **Tolerances.** Stack up tolerances for anything that must align through the enclosure wall (connectors, LEDs, displays, buttons): PCB fab tolerance + assembly placement + enclosure molding. Design the alignment features so worst-case stack still assembles.
- **Connector accessibility.** Connectors MUST be usable in the assembled product: finger and tool clearance, cable bend radius, visible labeling, and no need to disassemble the unit for normal connections.
- **Enclosure compatibility.** Verify board-to-enclosure fit with a 3D model check (or physical check) before fabrication: component heights, cable volumes, antenna clearances, thermal contact points.
- **Assembly.** Design for unambiguous assembly: polarized connectors, asymmetric mounting patterns where orientation matters, and an assembly order that is physically possible with tools. Anything that can be plugged in backwards eventually will be — make it impossible or harmless.

---

## 12. Manufacturability (DFM)

- **Automated assembly.** Released boards MUST be assemblable by standard pick-and-place and reflow: standard footprint geometries with correct courtyard spacing, components on as few sides as practical, and hand-soldered parts minimized and listed explicitly.
- **Panelization.** Boards SHOULD be designed with panelization in mind: edge clearance for rails, defined breakaway method (V-cut or mouse bites) with component keep-back from board edges, and panel drawings included in manufacturing outputs when the design controls the panel.
- **Fiducials.** Provide global fiducials (at least two, preferably three, asymmetric) per board or panel, and local fiducials for fine-pitch parts.
- **Test points.** Provide test access per Section 13 early enough that DFM and DFT do not fight over the same board area.
- **Solderability.** Choose surface finish appropriate to the assembly process and pitch; avoid mixing extreme thermal masses on one pad pair (tombstoning); follow paste and mask guidance for bottom-terminated packages (windowed paste on thermal pads).
- **Inspection.** Every solder joint SHOULD be inspectable by AOI; where it cannot be (BGA), X-ray or electrical test MUST cover it. Keep designators readable and orientation marks visible after assembly.
- **Yield thinking.** Anything that depends on manual skill, undocumented tweaking or selection of parts is a yield and repeatability defect: eliminate select-on-test values where possible, or make them a documented calibration step.

---

## 13. Design for Test (DFT)

- **Debug access.** Every board with firmware MUST expose its debug interface (SWD/JTAG or equivalent) on a defined connector or documented test pads — including on production units, at least as pads.
- **Programming.** Production programming MUST be possible without hand-soldering: connector, tag-style pads or bed-of-nails points, defined in the manufacturing documentation together with the programming procedure.
- **Boundary scan.** Where the major digital devices support boundary scan, the chain SHOULD be routed and documented for manufacturing test use, even if the first production run tests functionally.
- **Measurement points.** Provide labeled test points for: every power rail, reset, critical clocks, interface identity buses and key analog nodes. A technician SHOULD be able to verify the power tree with a multimeter using only labeled points.
- **Automated test.** Design so a functional tester can control and observe the board: test points on a single side placed on a reasonable grid where fixture testing is expected, self-test hooks in firmware (per AES manufacturing/test requirements), and current-consumption limits documented per state so pass/fail criteria exist.

---

## 14. Reliability

- **Environment first.** Every design MUST document its intended environment (temperature, humidity, vibration, condensation, chemical exposure) and be reviewed against it; "indoor lab" is a valid environment but MUST be stated.
- **Vibration and shock.** Heavy components need mechanical support beyond their solder joints (adhesive, clips, mounting) in vibrating environments; connectors in such environments need positive latching. Avoid large components cantilevered on tall leads.
- **Humidity and condensation.** For humid or condensing environments, avoid high-impedance circuits exposed to surface leakage, choose moisture-tolerant parts, and consider conformal coating from the start (it affects test-point, connector and label placement).
- **Corrosion.** Match connector plating pairs, avoid galvanic mismatches at contact interfaces, and specify finish and cleaning (no-clean vs washed) with the environment in mind.
- **Conformal coating.** If coated: define masked areas (connectors, test points needed after coating, adjustment elements) in the assembly drawing, and verify coating compatibility with components and rework.
- **Connector durability.** Select connectors for the documented mating-cycle count of the use case; user-facing connectors need an order of magnitude more cycles than internal ones. Board-edge and high-wear interfaces SHOULD be designed so the wearing part is the cheap, replaceable side (cable, not board).
- **Aging.** Review the design for known aging mechanisms — electrolytic dry-out, flash retention, LED degradation, relay wear — against the intended product lifetime, and document expected service items.

### 14.1 Unattended Operation

These rules apply to a Module that advertises unattended autonomous operation — continuing a Session and, where declared, recovering after a reset with no host present ([AES-MCI-006](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/05-interfaces-and-versioning.md#aes-mci-006-autonomous-continuation-and-deployment-policy), [AES-MCI-007](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/05-interfaces-and-versioning.md#aes-mci-007-recovery-after-reset-and-run-segment-provenance)). The protocol contract, the lifecycle and the persistent-state content are AES's and the firmware guide's; this section states what the hardware MUST provide so that the firmware can honor them for weeks at a time.

- **Supervise the controller from outside it.** An independent hardware watchdog and a supply supervisor that holds the controller in reset below its valid operating range are required, so that a brownout produces a clean reset rather than a controller executing at an undefined voltage; an internal watchdog alone SHOULD NOT be relied on where the same fault can stall it. Verify that a slow supply ramp and a brownout oscillation both produce a defined reset and safe outputs, not a partially initialized state.
- **Expose the reset cause.** The design MUST let firmware distinguish, at minimum, power-on, brownout, watchdog, software and external reset — through the controller's reset-status register, the supervisor's flag, or a latched signal — and the reset path MUST NOT clear that information before firmware reads it. A recovery that cannot say why it happened is half a record.
- **Storage that survives losing power mid-write.** The nonvolatile memory holding deployment state and measurement data MUST tolerate power loss during a write without corrupting previously written data: hold-up energy sized to finish the write in flight, or a memory and layout whose partial writes are detectable and recoverable. Budget write endurance against the firmware's checkpoint cadence and the stated deployment life, with margin; a state record rewritten every ten minutes for a year is a real number of cycles.
- **Keep time across the gap.** Where segment timing matters, provide an RTC with a backup source sized for the longest outage the deployment must describe, and document its drift; where an external time reference (a Communication & Timing Unit or equivalent) may be fitted, provide for it without requiring it. Timekeeping status is something the firmware reports, so the hardware MUST let it know whether the backup was lost.
- **Energy margin for the whole deployment.** The power budget (Section 6) covers the stated deployment duration at the worst-case temperature, including storage write peaks, Unit start-up peaks after a recovery, and any radio — and documents the resulting runtime. A battery- or harvester-powered Module SHOULD make its remaining energy observable to firmware.
- **Observable without a host.** Provide the sense points — supply voltages, temperature, storage activity — that firmware needs for health telemetry, and a local indication (LED or equivalent) that distinguishes at least running, completed/stopped and refused-recovery states, so that a person at the enclosure can tell whether it is worth reconnecting.

---

## 15. Documentation Requirements

Documentation obligations are defined by AES ([AES-REL-001](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/07-maturity-and-release.md#aes-rel-001-release-completeness)); this section states what good hardware documentation contains. It binds **Released** hardware — a prototype needs only its sources, a README and honest design notes.

- **Released package.** A released hardware design MUST include: editable schematic and PCB sources, schematic PDF, manufacturing outputs (Gerbers, drill, pick-and-place), BOM, assembly drawing, stackup and impedance requirements where controlled, and a DRC report. Multi-domain designs include the power tree or power-domain table.
- **Schematic quality.** The schematic PDF is the primary review artifact: it MUST be complete, legible, and identical in content to the source at the released revision.
- **Assembly drawings.** Assembly drawings MUST show designators, polarity/orientation marks, DNP parts, variant differences, and any manual assembly or masking instructions.
- **BOM quality.** The BOM MUST identify every line by value/parameters, package, designators and quantity, mark single-source and DNP lines, and reflect variants explicitly. Parameter-specified generic lines (per Section 3) state acceptance criteria, not just one part.
- **Manufacturing outputs.** Generated outputs MUST match the released source revision exactly and be regenerated (never hand-edited) after any change. Fab notes state stackup, finish, impedance control, and acceptance class.
- **Revision management.** Board revisions follow the AES versioning rules; the physical board MUST carry its identity and revision (per AES identifier and marking rules), and the documentation package MUST make revision differences discoverable (changelog per revision).

---

## 16. Review Checklists

Self-review is valid ([AES-QA-001](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/07-maturity-and-release.md#aes-qa-001-proportionate-review)) — ideally after a day away from the design. Skip items that plainly don't apply; no N/A bookkeeping. Every real "no" is fixed or consciously accepted with a one-line design-notes entry. Independent review is strongly recommended for mains/hazardous voltage, high-energy batteries, RF compliance and production manufacturing.

### Prototype checklist

Use before *any* board order. This is deliberately one screen — it exists to prevent dead boards and damaged parts, not to prove process.

- [ ] Power rails and current paths checked: budgets, regulator stability, worst-case load
- [ ] Reverse-polarity, overcurrent and transient protection present on every external power input
- [ ] Component pinouts and footprints verified against datasheets (especially anything new)
- [ ] Connector pinouts verified, including mating-side orientation
- [ ] Module Port female M8 8P wired to the recorded contact assignment with `RESERVED` open; power input male M8 3P on positions 1/3/4 with position 4 open; `MPI_VIN` at 0 V when the board runs from USB
- [ ] Programming and boot paths checked (debug access, boot straps, reset)
- [ ] Protection and decoupling reviewed; decoupling loops minimal
- [ ] Return paths sane: no signal crossing a plane split; hot loops small
- [ ] Creepage/clearance rules set and DRC-checked where anything exceeds SELV
- [ ] ERC/DRC results reviewed
- [ ] Mechanical constraints checked (outline, holes, heights, connector access)
- [ ] Manufacturing outputs visually inspected (Gerber viewer pass)
- [ ] Known hazards and non-obvious decisions noted in README/design notes

### Release additions

Use additionally before a Release ([AES-REL-001](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/07-maturity-and-release.md#aes-rel-001-release-completeness)); this is where Sections 12–15 fully bind.

- [ ] Derating targets of Section 3 verified (voltage incl. transients, current incl. thermal, ceramic DC bias); electrolytic lifetime calculated where used
- [ ] No EOL/NRND parts without a noted migration plan; single-source parts flagged with substitution plan
- [ ] Sequencing (up, down, brown-out) designed where devices require it; battery/USB limits met independently of software
- [ ] ESD protection at every external connector; cable interfaces filtered; shield/chassis bonding decision documented
- [ ] High-speed/analog/RF rules of Sections 7–9 verified where applicable (impedance + stackup + fab note, isolation gaps, antenna keep-outs, signal-chain budget)
- [ ] Worst-case junction temperatures verified in enclosure conditions; thermal pads via-stitched
- [ ] 3D fit check done; nothing can be assembled or plugged incorrectly without being harmless
- [ ] Straight-through M8 8P cable soak on every Module Port with zero valid AEL frames; power-input hot-plug, reverse-polarity and two-way back-feed tests passed (§5.3, §6.2)
- [ ] DFM/DFT complete: standard assembly sufficient, fiducials, inspectability, production programming access, labeled test points, per-state current limits for test criteria
- [ ] Release package complete (Section 15) and regenerated from the tagged source revision
- [ ] Board carries identity and revision marking per AES; revision changelog present

---

*AURIORA Hardware Design Guide 0.6.0 — complements the AURIORA Engineering Standard. Licensed under CC BY-SA 4.0.*
