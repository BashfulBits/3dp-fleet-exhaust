# Prototype Rework Log

> Every hardware change made by hand to a prototype board: a cut track, a bodge wire, a changed value, a lifted part. One row per change, per board. A rework that fixes a design fault becomes a change to the design (a `D` row and the next board revision); a rework that only enables a test (a fault injected on purpose) says so and is undone after the test.

## How to log a rework

1. Give it the next number (`RK-n`) and name the board serial and revision.
2. Say why: the failing test (`VAL-…`) and what it showed, or the fault being injected.
3. Say exactly what was changed, with reference designators and values, and a photo in `results/`.
4. Re-run the failing test and every test in its group (README §4) and record the result.
5. Close it one of three ways: **design change** (name the `D` row and the revision that carries it), **test only** (undone, with the date), or **board fault** (an assembly defect; the board is retired or repaired, no design change).

## Log

| # | Board (serial, rev) | Test | Why | Change made | Result | Closed as |
|---|---|---|---|---|---|---|
| — | — | — | No prototype boards yet | — | — | — |

## Design changes from validation

Reworks closed as **design change**, by revision, so the next layout picks them all up.

| Rev | Rework | Design change | D row |
|---|---|---|---|
| — | — | — | — |
