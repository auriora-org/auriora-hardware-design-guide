# AURIORA Hardware Design Guide

The AURIORA Hardware Design Guide (AHDG) is the official design guide for all AURIORA hardware projects. It complements the AURIORA Engineering Standard (AES): AES defines *what* must be true about AURIORA engineering work, AHDG defines *how* hardware should be designed to achieve consistent quality across all AURIORA modules.

Requirements scale with the AES maturity level: safety and damage-prevention rules apply always (protection, battery safety, creepage/clearance, safe defaults), production and documentation-package rules bind Released hardware only, and everything else is a recommended default that engineering judgment may override without exception paperwork.

Read the guide: [GUIDE.md](./GUIDE.md)

## Scope

- Design philosophy, schematic design, component selection, PCB layout
- EMC/EMI, power, high-speed, analog, RF and thermal design
- Mechanical integration, manufacturability, design for test, reliability
- Documentation requirements for Released hardware
- A one-screen prototype review checklist plus release additions

The guide is technology-independent and contains no specific part numbers or vendor recommendations.

## Requirement Language

Aligned with AES:

- `MUST` — mandatory when applicable (equivalent to `SHALL` in AES)
- `SHOULD` — recommended default; engineering judgment may justify another approach, no documented exception needed
- `MAY` — optional improvement

Where AHDG and AES conflict, AES prevails.

## Versioning and Changes

Released versions are recorded in the [CHANGELOG](./CHANGELOG.md) and tagged in version control.

## License

This documentation is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](./LICENSE).
