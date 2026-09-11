# Changelog

All notable changes to the AURIORA Hardware Design Guide are documented in this file. Released versions are tagged in version control.

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
