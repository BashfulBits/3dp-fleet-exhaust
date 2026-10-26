# Contributing

Thanks for helping. This project is a design framework for a shared exhaust, filtration and chamber-heating system on a small fleet of enclosed 3D printers. It is in Phase 1 (design baseline), so the most useful contributions today are corrections, reviews, measurements and printer packs.

## What to open an issue for

- **A mistake in a doc**: wrong value, broken reasoning, a claim that needs a source. Include the doc and section, and say whether you have a measurement or a primary source ([V] verified, [E] estimate, [U] unverified: the evidence key in doc 01).
- **A measurement from real hardware**: it can turn an [U] into a [V], or show an [E] is wrong. These are the most valuable reports.
- **A safety concern**: use the same issue form, with "safety" in the title. Nothing in this project may kill a print except the mains cut on a corroborated fire or a verified heater runaway, so say which safety layer (doc 05) is affected.

## Adding a printer

1. Read the [porting guide](docs/12_porting_guide.md) and its checklist (§7).
2. Copy `printers/_template/` to `printers/<vendor>_<model>/`, fill in `profile.yaml` and the README, and run through the checklist.
3. A profile can only make the core limits stricter, never looser (doc 12 §6).
4. Link the pack's printable parts in doc 08's pack index and its lines in the BOM scaling section (doc 03 §1.2).

## Changing the design

- A design change that applies to any build gets a new `D<n>` row in [design_decisions.md](docs/design_decisions.md), saying what it amends or supersedes. Update the docs it affects in the same pull request, including the BOM total.
- **Safety changes** (doc 05, the safety contract in doc 12 §6, or anything that touches the mains cut, heater limits or watchdog) need a matching commissioning test in doc 05 and an explanation of the failure it prevents. Fewer false positives on the mains cut is a requirement, not a nice-to-have.
- Keep the evidence key and cross-reference style of the existing docs. Mermaid diagrams should stay under about 700 px wide.
- Prefer existing printable models over custom parts (doc 08), and link models rather than copying them. CC BY-NC models are linked, never copied.

## Licences

- Firmware, scripts and the klippy extra: MIT. Hardware designs: CERN-OHL-P-2.0. Docs and printed-part CAD: CC BY 4.0. See [doc 12 §8](docs/12_porting_guide.md#8-licensing-and-contributions).
- The repo follows the [REUSE](https://reuse.software/) spec: new source files carry an SPDX licence header naming MIT, CERN-OHL-P-2.0 or CC-BY-4.0, and licence texts live in `LICENSES/`. The CERN-OHL-P-2.0 text is added with the first hardware files.
- **Do not copy code from GPL or AGPL projects** (FLOOK32, Nevermore Controller and similar). Ideas are fine; code is not.
- By contributing you agree your work is released under the licence for its area.

## Pull requests

Keep them small and focused, describe what changed and why, and say how you checked it. Commits are signed off by your own judgment, not by a bot: if you used an AI assistant, check its output against the sources.
