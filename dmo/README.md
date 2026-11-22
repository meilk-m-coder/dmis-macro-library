# DMO Test Results

This directory contains DMO output files generated while testing the example programs with WM | Quartis.

The files are stored as test artifacts and are intended to make the generated DMIS result output reproducible and reviewable.

| Example | DMO result |
|---|---|
| `point-basic.dmi` | `point-basic__M1.dmo` |
| `point-referenced.dmi` | `point-referenced__M1.dmo` |
| `point-referenced-with-deviation.dmi` | `point-referenced-with-deviation__M1.dmo` |
| `point-surface-plane.dmi` | `point-surface-plane__M1.dmo` |
| `tolerance-output.dmi` | `tolerance-output__M1.dmo` |
| `hexagon-experimental.dmi` | `hexagon-experimental__M1.dmo` |

## Purpose

The DMO files provide generated output corresponding to the tested example programs.

They can be used to review:

- nominal feature output
- actual feature output
- generated tolerance names
- coordinate tolerances
- profile/normal tolerances
- position tolerances
- separate output
- combined output

## Important

These files document individual test runs.

They are not reference values certified for other machines, workpieces, sensor configurations, or WM | Quartis installations.

A changed library version may intentionally produce different output.