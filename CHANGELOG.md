# Changelog

All notable changes to the AURIORA Hardware Design Guide are documented in this file. Released versions are tagged in version control.

## 0.5.0 - 2026-09-17

- Unattended operation hardware rules (new §14.1), following AES 0.8.0 `AES-MCI-006`/`AES-MCI-007` ([EDR-009](https://github.com/auriora-org/auriora-engineering-standard/blob/main/docs/edr/EDR-009-autonomous-module-operation-and-recovery.md)), for a Module that advertises unattended autonomous operation: an independent hardware watchdog and a supply supervisor holding reset below the valid range, verified against slow ramps and brownout oscillation; reset cause distinguishable by firmware (power-on, brownout, watchdog, software, external) and not cleared before it is read; nonvolatile storage that tolerates power loss mid-write, with endurance budgeted against checkpoint cadence and deployment life; an RTC with backup where segment timing matters, its drift documented and its backup loss observable, with optional provision for an external time reference; a power budget for the whole deployment duration including recovery peaks, with remaining energy observable where battery- or harvester-powered; and health sense points plus a local running / completed / refused indication readable without a host. Implementation guidance only — the protocol contract stays in AES.
- AURIORA Event Link (AEL) port and router hardware (§5.1), replacing the Module Synchronization Interface section after AES 0.8.0 superseded SYNC with AEL (`AES-AEL-001` to `AES-AEL-005`, `docs/interfaces/ael.md`, EDR-008). Ports: a parameter-specified 3.3 V transceiver class with separate driver and receiver pairs, a *true fail-safe* receiver established from the part's specification and a data rate with margin above the 1 Mbit/s AEL baseline (the normative electrical profile is still open in AES); termination at the receiver only; the fail-safe requirement restated against AEL's *logical idle*, which is UART mark (HIGH) — the receiver must land in mark with the cable unplugged, the far end unpowered or in reset, or the pair shorted, and must never present a start-bit edge during insertion or removal, verified with the receiving peripheral armed; a new rule that an AEL OUT transmitter holds idle through power-up, power-down, reset and firmware start, verified by cycling while a receiver counts frames; protection at the connector, no back-drive, coupled `AEL_P`/`AEL_N` routing and the `AEL_IN_*`/`AEL_OUT_*` net names; the frame path through a hardware serial peripheral with DMA and a hardware means of timestamping the first start-bit edge (capture mechanism not prescribed), never a bit-banged GPIO; the M8 cross-mating analysis carried over. Routers: the controller-less SYNC Hub guidance is **removed** — an AEL router has a controller in the event path by design and owes a characterized real-time path: one hardware receiver and transmitter per port with hardware timestamping; forwarding latency, jitter and fan-out skew measured with all ports active, management traffic running and at the documented maximum event rate; fan-out outputs launched together within the documented skew; deliberate saturation testing of queue overflow; per-port fault containment; route changes switched between frames only; idle outputs on power loss with no self-reinstated route table; Hub-to-Hub AEL links built as ordinary ports.

## 0.4.0 - 2026-09-11

- Module Synchronization Interface (SYNC) port hardware (§5.1): SYNC OUT and SYNC IN implemented with the transmitter and receiver of a parameter-specified 3.3 V full-duplex RS-422/RS-485-compatible transceiver class (no half-duplex device serving both ports, no part named); differential termination at the receiver only, nominally 120 Ω pending validation; fail-safe inactive state with the cable unplugged, the far end unpowered or the pair shorted, and no event on insertion or removal; TVS/clamp protection at the connector ahead of termination and transceiver; no back-drive of an unpowered board; coupled `SYNC_P`/`SYNC_N` routing and the `SYNC_IN_*`/`SYNC_OUT_*` net names; a hardware-timestamped capture path for SYNC IN and a hardware-timed output for SYNC OUT; a cross-mating analysis against every other M8 port on the product family while connector gender and keying are open; and the SYNC Hub as a controller-less receive → regenerate → distribute device with measured propagation delay, skew and jitter. The interface itself and its behavioral rules are defined by AES 0.6.0 (`AES-SYNC-001` to `AES-SYNC-004`, `docs/interfaces/sync.md`).

## 0.3.0 - 2026-09-10

- Unit Interface host power guidance (§6.1): port count is not additive — size the shared supply for the intended simultaneous load and document which port combinations are supported; the discovery-state current of every connected Unit is a real base load even with all functional domains disabled; Unit start-up peaks stack when several Units are enabled together, and staggering enables helps a transient limit only; shared versus per-port protection is a deliberate choice, and the physical protection — not the Unit's self-declared EEPROM metadata — is what contains a Unit drawing more than it declared.
- Component selection (§3): pointer to the AES default controller platform requirement (`AES-ARCH-001`), keeping this guide part-number-neutral rather than repeating the families.

## 0.2.0 - 2026-07-14

- Unit Interface power and readiness rules (§6.1): `UIF_PWR_EN` must switch the Unit's functional power domain locally, EEPROM discovery must work while that domain is disabled, disabled circuitry must not be back-powered through interface signals, `UIF_READY` must have a defined LOW state with no contention during boot/reset, hosts must verify EEPROM power metadata before asserting `UIF_PWR_EN`, no Unit-presence pin, and per-profile documentation of electrical/mechanical limits.

## 0.1.0 - 2026-07-13

First release.

- Design philosophy and a maturity-scaling model: safety and damage-prevention rules bind at every maturity level; production, DFM/DFT and documentation-package rules bind Released hardware; everything else is a recommended default.
- Schematic design, component selection and derating targets.
- PCB layout, EMC/EMI, power, high-speed, analog, RF and thermal design guidance.
- Mechanical integration, manufacturability (DFM), design for test (DFT) and reliability guidance.
- Documentation requirements for Released hardware.
- A one-screen prototype review checklist focused on failure prevention, plus release additions; self-review explicitly valid with independent review recommended for hazardous domains.
- Requirement language aligned with AES (MUST/SHOULD/MAY; a skipped SHOULD needs no documented exception).
- Component-agnostic policy: no specific part numbers or vendor recommendations.
- CC BY-SA 4.0 license.
