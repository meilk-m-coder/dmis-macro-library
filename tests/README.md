# Test Matrix

This document defines reproducible test cases for the **DMIS Macro Library**.

The test matrix is intended to verify calculation logic, macro state handling, feature construction, tolerance generation, and output behavior.

> Passing these tests does not certify a macro or measurement strategy as safe for production use. Machine movements, probe configurations, collision risks, fixtures, workpieces, and measurement-system behavior must be validated separately on the target system.

## Test status

The following status values are used:

| Status | Meaning |
|---|---|
| `PASS` | Tested and result matched the expected behavior |
| `FAIL` | Tested and result did not match the expected behavior |
| `OPEN` | Test is defined but has not yet been executed |
| `N/A` | Test is not applicable to the current configuration |

---

# 1. POINT_GET_MAIN_AXES

These tests verify dominant-axis selection and the output-axis bit mask.

## 1.1 Dominant-axis selection

| Test | Nominal vector | Angle limit | Expected main axis | Expected mask | Status |
|---|---:|---:|---:|---:|---|
| AXIS-001 | `1, 0, 0` | `0°` | X = `1` | X = `1` | `OPEN` |
| AXIS-002 | `0, 1, 0` | `0°` | Y = `2` | Y = `2` | `OPEN` |
| AXIS-003 | `0, 0, 1` | `0°` | Z = `4` | Z = `4` | `OPEN` |
| AXIS-004 | `-1, 0, 0` | `0°` | X = `1` | X = `1` | `OPEN` |
| AXIS-005 | `0, -1, 0` | `0°` | Y = `2` | Y = `2` | `OPEN` |
| AXIS-006 | `0, 0, -1` | `0°` | Z = `4` | Z = `4` | `OPEN` |

### Expected behavior

The largest absolute normalized vector component determines `point_main_axis`.

Positive and negative vectors must select the same coordinate axis.

---

## 1.2 Secondary-axis selection

| Test | Nominal vector | Angle limit | Expected main axis | Expected mask | Status |
|---|---:|---:|---:|---:|---|
| AXIS-010 | `0.539, 0, 0.842` | `0°` | Z = `4` | Z = `4` | `OPEN` |
| AXIS-011 | `0.539, 0, 0.842` | `20°` | Z = `4` | X + Z = `5` | `OPEN` |
| AXIS-012 | `0, 0.539, 0.842` | `20°` | Z = `4` | Y + Z = `6` | `OPEN` |
| AXIS-013 | `0.539, 0.842, 0` | `20°` | Y = `2` | X + Y = `3` | `OPEN` |

### Expected behavior

The dominant axis is always selected.

When `angle_limit > 0`, every secondary absolute vector component that reaches the calculated component threshold is added to `point_axis_mask`.

---

# 2. POINT_MEASURE - Direct point measurement

These tests verify the direct point-measurement path:

```text
surface_point_count < 3
```

## 2.1 Basic point

| Test | Vector convention | Inverse | Thickness | Offset | Snap | References | Expected | Status |
|---|---:|---:|---:|---:|---:|---|---|---|
| MEAS-001 | `0` | `0` | `0.0` | `0.0` | `0` | none | Direct point measurement at nominal technical position | `PASS` |

Reference example:

```text
examples/point-basic.dmi
```

Screenshot:

```text
screenshots/point-basic.png
```

---

## 2.2 Measurement offset

| Test | Vector | `moffset` | Expected behavior | Status |
|---|---:|---:|---|---|
| MEAS-010 | `0,0,1` | `+1.0` | Probing position shifted +1 mm in active measurement-vector direction; technical offset removed from final result | `OPEN` |
| MEAS-011 | `0,0,1` | `-1.0` | Probing position shifted -1 mm in active measurement-vector direction; technical offset removed from final result | `OPEN` |

---

## 2.3 Material thickness

| Test | Vector convention | Inverse | Thickness | Expected behavior | Status |
|---|---:|---:|---:|---|---|
| MEAS-020 | `0` | `0` | `1.0` | No material shift | `OPEN` |
| MEAS-021 | `0` | `1` | `1.0` | Shift through material and reverse measurement vector | `OPEN` |
| MEAS-022 | `1` | `0` | `1.0` | Shift by thickness in nominal-vector direction | `OPEN` |
| MEAS-023 | `1` | `1` | `1.0` | No thickness shift; reverse active measurement vector | `OPEN` |

---

# 3. Reference compensation

These tests verify reference-feature coordinate compensation.

Reference offset:

```text
actual reference coordinate - nominal reference coordinate
```

## 3.1 No reference deviation

| Test | X reference | Y reference | Z reference | Expected target deviation | Status |
|---|---|---|---|---|---|
| REF-001 | none | none | `REFERENCE_Z` | No additional Z deviation when reference nominal and actual coordinates are equal | `PASS` |

Reference example:

```text
examples/point-referenced.dmi
```

Screenshot:

```text
screenshots/point-referenced.png
```

---

## 3.2 Simulated positive Z deviation

| Test | Reference nominal Z | Reference actual Z | Expected reference offset | Target nominal Z | Expected target actual Z | Status |
|---|---:|---:|---:|---:|---:|---|
| REF-010 | `83.4` | `86.4` | `+3.0` | `82.8` | `85.8` | `PASS` |

Reference example:

```text
examples/point-referenced-with-deviation.dmi
```

Screenshot:

```text
screenshots/point-referenced-with-deviation.png
```

### Expected behavior

The `+3.0 mm` reference deviation shifts the technical target probing position by `+3.0 mm` in Z.

The reference component remains part of the final measured result.

---

## 3.3 Reference-axis independence

| Test | ref_x | ref_y | ref_z | Expected affected axes | Status |
|---|---|---|---|---|---|
| REF-020 | active | empty | empty | X only | `OPEN` |
| REF-021 | empty | active | empty | Y only | `OPEN` |
| REF-022 | empty | empty | active | Z only | `PASS` |
| REF-023 | active | active | active | X + Y + Z | `OPEN` |

---

# 4. Surrounding-plane measurement

These tests verify:

```text
surface_point_count >= 3
```

## 4.1 Five-point surrounding plane

| Test | Point count | Radius | Snap | Expected behavior | Status |
|---|---:|---:|---:|---|---|
| PLANE-001 | `5` | `2.5 mm` | `0` | Five surrounding points measured; result constructed from actual plane and nominal-vector line | `PASS` |

Reference example:

```text
examples/point-surface-plane.dmi
```

Screenshot:

```text
screenshots/point-surface-plane.png
```

### Expected behavior

The auxiliary plane:

```text
<point_name>_PLN
```

is measured from the surrounding point pattern.

The final actual point is constructed from:

```text
corrected actual plane
+
theoretical line through the original nominal point
```

---

## 4.2 Minimum surrounding-plane count

| Test | Point count | Expected measurement mode | Status |
|---|---:|---|---|
| PLANE-010 | `2` | Direct single-point measurement | `OPEN` |
| PLANE-011 | `3` | Surrounding-plane measurement | `OPEN` |

This verifies the strategy boundary between direct and surrounding-plane measurement.

---

## 4.3 Surrounding plane with axis snapping

| Test | Point count | Snap | Expected result line | Status |
|---|---:|---:|---|---|
| PLANE-020 | `5` | `0` | Nominal point vector | `PASS` |
| PLANE-021 | `5` | `1` | Dominant coordinate axis | `OPEN` |

---

# 5. POINT_TOL_CORTOL

These tests verify explicit coordinate-tolerance masks.

| Test | Mask | Expected tolerance axes | Status |
|---|---:|---|---|
| CORTOL-001 | `0` | none | `OPEN` |
| CORTOL-002 | `1` | X | `OPEN` |
| CORTOL-003 | `2` | Y | `OPEN` |
| CORTOL-004 | `3` | X + Y | `PASS` |
| CORTOL-005 | `4` | Z | `PASS` |
| CORTOL-006 | `5` | X + Z | `OPEN` |
| CORTOL-007 | `6` | Y + Z | `OPEN` |
| CORTOL-008 | `7` | X + Y + Z | `PASS` |

Examples providing coverage include:

```text
point-basic.dmi
point-surface-plane.dmi
tolerance-output.dmi
```

---

# 6. POINT_TOL_MAIN_AXES

These tests verify coordinate tolerances derived from a normal tolerance and selected vector axes.

## 6.1 Positive component

| Test | Vector component | Normal lower | Normal upper | Expected lower | Expected upper | Status |
|---|---:|---:|---:|---:|---:|---|
| MAIN-001 | `+1.0` | `-0.5` | `+1.0` | `-0.5` | `+1.0` | `PASS` |
| MAIN-002 | `+0.5` | `-0.5` | `+1.0` | `-0.25` | `+0.5` | `OPEN` |

## 6.2 Negative component

| Test | Vector component | Normal lower | Normal upper | Raw values | Expected sorted limits | Status |
|---|---:|---:|---:|---|---|---|
| MAIN-010 | `-1.0` | `-0.5` | `+1.0` | `+0.5`, `-1.0` | `-1.0`, `+0.5` | `OPEN` |
| MAIN-011 | `-0.5` | `-0.5` | `+1.0` | `+0.25`, `-0.5` | `-0.5`, `+0.25` | `OPEN` |

These tests specifically verify the `MN` / `MX` sorting of calculated tolerance limits.

---

# 7. POINT_TOL_VECTOR_CORTOL

These tests verify vector-based coordinate-tolerance generation.

| Test | Normalized vector | Expected mask | Status |
|---|---:|---:|---|
| VECTOR-001 | `1,0,0` | X = `1` | `OPEN` |
| VECTOR-002 | `0,1,0` | Y = `2` | `OPEN` |
| VECTOR-003 | `0,0,1` | Z = `4` | `OPEN` |
| VECTOR-004 | `0.7071,0.7071,0` | X + Y = `3` | `OPEN` |
| VECTOR-005 | `0.57735,0.57735,0.57735` | X + Y + Z = `7` | `OPEN` |

## Epsilon handling

Internal epsilon:

```text
0.000001
```

| Test | Vector component | Expected | Status |
|---|---:|---|---|
| VECTOR-010 | `0.0000001` | Component ignored | `OPEN` |
| VECTOR-011 | `0.00001` | Component included | `OPEN` |

---

# 8. Normal/profile tolerance conventions

These tests verify `POINT_TOL_NORMAL`.

| Test | Convention | Expected name | Expected type | Status |
|---|---:|---|---|---|
| NORMAL-001 | `0` Generic | `<point_name>N` | `PROFS` | `PASS` |
| NORMAL-002 | `1` Volkswagen / Audi | `<point_name>P` | `PROFP` | `PASS` |
| NORMAL-003 | `2` Mercedes-Benz | `<point_name>N` | `PROFS` | `OPEN` |
| NORMAL-004 | `3` BMW | `N<point_name>` | `PROFS` | `OPEN` |

---

# 9. Position tolerance conventions

These tests verify `POINT_TOL_POSITION`.

| Test | Convention | Expected name | Expected type | Status |
|---|---:|---|---|---|
| POS-001 | `0` Generic | `<point_name>POS` | `POS,3D` | `PASS` |
| POS-002 | `1` Volkswagen / Audi | `<point_name>P2` | `POS,3D` | `PASS` |
| POS-003 | `2` Mercedes-Benz | `<point_name>N2` | `POS,3D` | `OPEN` |
| POS-004 | `3` BMW | `P<point_name>` | `POS,3D` | `OPEN` |

---

# 10. Output-state reset

These tests verify `POINT_OUTPUT_RESET`.

| Test | Prepared state before reset | Expected state after reset | Status |
|---|---|---|---|
| STATE-001 | Coordinate mask active | `point_axis_mask = 0` | `PASS` |
| STATE-002 | Normal tolerance active | `point_normal_active = 0` | `PASS` |
| STATE-003 | Position tolerance active | `point_position_active = 0` | `PASS` |
| STATE-004 | Tolerance names populated | All tolerance-name strings empty | `PASS` |

---

# 11. POINT_OUTPUT_WRITE

These tests verify feature and output-style selection.

## 11.1 Feature selection

| Test | feature_output | Expected feature output | Status |
|---|---:|---|---|
| OUTPUT-001 | `0` | Nominal `F` only | `OPEN` |
| OUTPUT-002 | `1` | Actual `FA` only | `OPEN` |
| OUTPUT-003 | `2` | Nominal `F` and actual `FA` | `PASS` |

## 11.2 Output style

| Test | output_style | Expected behavior | Status |
|---|---:|---|---|
| OUTPUT-010 | `0` | Separate OUTPUT statement per prepared tolerance | `PASS` |
| OUTPUT-011 | `1` | Combined OUTPUT statement per selected feature | `PASS` |

Reference example:

```text
examples/tolerance-output.dmi
```

Screenshot:

```text
screenshots/tolerance-output.png
```

---

# 12. Combined output combinations

The combined-output helpers should support every axis mask together with the optional normal and position criteria.

`criteria_mode` is:

```text
0 = no normal and no position tolerance
1 = normal only
2 = position only
3 = normal + position
```

## Required matrix

Each axis mask should ultimately be tested with each criteria mode.

| Axis mask | None | Normal | Position | Normal + Position |
|---:|---|---|---|---|
| `0` | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `1` X | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `2` Y | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `3` X+Y | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `4` Z | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `5` X+Z | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `6` Y+Z | `OPEN` | `OPEN` | `OPEN` | `OPEN` |
| `7` X+Y+Z | `OPEN` | `OPEN` | `OPEN` | `PASS` |

The `PASS` entry for mask `7` with normal and position tolerances is covered by `tolerance-output.dmi`.

---

# 13. Current integration-test summary

| Example | Primary coverage | Result |
|---|---|---|
| `point-basic.dmi` | Direct point measurement, explicit coordinate tolerances, combined output | `PASS` |
| `point-referenced.dmi` | Reference-feature measurement without simulated deviation | `PASS` |
| `point-referenced-with-deviation.dmi` | Z reference with simulated `+3.0 mm` deviation | `PASS` |
| `point-surface-plane.dmi` | Five-point surrounding-plane measurement | `PASS` |
| `tolerance-output.dmi` | All coordinate axes, normal tolerance, position tolerance, separate vs. combined output | `PASS` |

Associated screenshots are available under:

```text
screenshots/
```

---

# 14. Environment record

For every formal test run, record at least:

```text
Date:
Library version:
Library commit/tag:
WM | Quartis version:
DMIS version:
Machine / simulation mode:
Sensor configuration:
Example or test program:
Expected result:
Actual result:
Status:
Notes:
```

Example:

```text
Date: YYYY-MM-DD
Library version: 0.1.0
Library commit/tag: <git commit or tag>
WM | Quartis version: <version>
DMIS version: 5.2
Machine / simulation mode: <simulation / physical machine>
Sensor configuration: <configuration>
Example or test program: point-basic.dmi
Expected result: Direct point measurement, X+Y tolerances, combined F/FA output
Actual result: <documented result>
Status: PASS
Notes: <optional notes>
```

---

# 15. Release criteria

Before promoting the library to version `1.0.0`, the following areas should have defined and executed tests:

- dominant-axis selection
- angular secondary-axis selection
- positive and negative vector components
- direct point measurement
- material-thickness conventions
- inverse measurement
- measurement offset
- X/Y/Z reference compensation
- surrounding-plane measurement
- surrounding-plane axis snapping
- all coordinate masks
- all customer tolerance conventions
- output-state reset
- nominal-only output
- actual-only output
- nominal + actual output
- separate output
- combined output
- combined-output mask/criteria combinations

Production suitability additionally requires machine-specific safety and measurement validation outside the scope of this matrix.