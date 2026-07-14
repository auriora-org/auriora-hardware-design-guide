# Changelog

All notable changes to the AURIORA Hardware Design Guide are documented in this file. Released versions are tagged in version control.

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
