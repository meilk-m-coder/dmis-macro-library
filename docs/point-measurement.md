# Point Measurement

## Overview

The measurement layer of the **DMIS Macro Library** is responsible for measuring an existing nominal point feature and constructing the corresponding actual point feature.

The primary macro is:

```text
POINT_MEASURE
```

The supporting utility macro is:

```text
POINT_GET_MAIN_AXES
```

The calling DMIS program remains responsible for nominal feature definitions, probe selection, probe orientation, clearance positions, travel commands, and all part-specific parameter values.

> **Safety notice:** This macro can influence probing positions, active measurement vectors, and resulting machine movements. Every parameter combination must be validated in simulation and, where applicable, on the physical machine using appropriate collision-prevention measures and reduced machine speed.

## POINT_MEASURE

### Purpose

`POINT_MEASURE` performs the physical measurement of an existing nominal point feature.

The macro supports:

- direct single-point measurement
- standard and through-material vector conventions
- inverse measurement direction
- sheet-material thickness correction
- additional measurement offset
- referenced measurement using up to three reference features
- dominant-axis snapping
- surrounding-plane measurement
- actual surface-vector evaluation
- combinations of these strategies

The measured or constructed actual feature retains the original nominal point name.

## Interface

```dmis
CALL/M(POINT_MEASURE),$
  point_name,$
  vector_convention,$
  inverse_direction,$
  thickness,$
  moffset,$
  snap_to_axis,$
  surface_point_count,$
  surface_point_distance,$
  ref_x,$
  ref_y,$
  ref_z
```

## Parameters

### `point_name`

Name of an existing nominal point feature, passed as a character string.

The macro reads the nominal position and vector from:

```dmis
F(@point_name)
```

Expected `OBTAIN` indices for the nominal point are:

```text
3 = X
4 = Y
5 = Z
6 = I
7 = J
8 = K
```

### `vector_convention`

Controls the interpretation of the nominal point vector.

```text
0 = the nominal vector points away from the material
1 = the nominal vector points through the material
```

Convention `1` is intended for workflows in which a through-material vector convention is used.

### `inverse_direction`

Controls the side from which the point is measured.

```text
0 = measure from the default side
1 = measure from the opposite side
```

The resulting material correction and measurement-vector direction depend on the combination of `vector_convention` and `inverse_direction`.

### `thickness`

Sheet-material thickness used when the selected vector convention and measurement direction require a material-side correction.

Use `0.0` when no material-thickness correction is required.

### `moffset`

Additional measurement offset along the active measurement vector.

This offset changes the technical probing position. The corresponding technical displacement is removed from the measured result where required by the selected strategy.

### `snap_to_axis`

Controls whether secondary coordinates are kept as measured or snapped to the nominal point.

```text
0 = keep the measured result along the nominal point vector
1 = snap secondary coordinates using the dominant coordinate axis
```

The dominant axis is determined from the largest absolute component of the normalized nominal point vector.

### `surface_point_count`

Selects the physical measurement method.

```text
0 to 2 = direct single-point measurement
3 or more = surrounding-plane measurement
```

### `surface_point_distance`

Radial distance of the surrounding-plane probing points from the technical measurement position.

This parameter is only used when `surface_point_count` is at least `3`.

### `ref_x`, `ref_y`, `ref_z`

Optional reference-feature names for X, Y, and Z compensation.

An empty string disables compensation for the corresponding axis.

Example:

```text
ref_x = 'REFERENCE_X'
ref_y = ''
ref_z = 'REFERENCE_Z'
```

This configuration uses:

- X deviation from `REFERENCE_X`
- no Y compensation
- Z deviation from `REFERENCE_Z`

## Nominal-vector normalization

The nominal vector is read from the point feature and normalized:

```text
plan_vec = VUNIT(VCART(plan_i, plan_j, plan_k))
```

All subsequent vector operations use the normalized vector. This keeps direction-dependent calculations independent of the original vector magnitude.

## Reference-feature compensation

For each enabled reference axis, the macro calculates:

```text
reference offset = actual coordinate - nominal coordinate
```

The technical probing position is then calculated conceptually as:

```text
technical probing position =
    nominal point position
    + reference-feature coordinate deviation
    + material-side correction
    + measurement offset
```

Reference offsets affect the probing position only. They are intentionally not removed from the final result because they represent the detected displacement of the physical workpiece.

## Material-side correction

### Away-from-material convention

When `vector_convention = 0`, the nominal vector points away from the material.

- With `inverse_direction = 0`, no material-thickness correction is applied.
- With `inverse_direction = 1`, the probing position is shifted through the material and the active measurement vector is reversed.

### Through-material convention

When `vector_convention = 1`, the nominal vector points through the material.

- With `inverse_direction = 0`, the probing position is shifted by the material thickness in nominal-vector direction.
- With `inverse_direction = 1`, no material-thickness shift is applied and the active measurement vector is reversed.

These rules must be checked against the conventions used by the calling project.

## Direct single-point measurement

Direct measurement is selected when:

```text
surface_point_count < 3
```

The original nominal feature is measured directly:

```dmis
MEAS/POINT,F(@point_name),1
  PTMEAS/CART,$
    measure_x,measure_y,measure_z,$
    measure_i,measure_j,measure_k
ENDMES
```

The technical probing position includes optional reference offsets, material-side correction, and measurement offset.

After measurement, technical displacements caused by material thickness and `moffset` are removed where required. Reference-feature deviations remain part of the actual result.

## Axis snapping for direct measurement

When `snap_to_axis = 1`, the macro:

1. corrects material-thickness and measurement-offset displacement
2. reads the corrected actual point coordinates
3. creates an actual plane through the corrected point using the nominal point vector
4. creates a theoretical axis-parallel line through the original nominal point
5. constructs the final point from the intersection of the actual plane and theoretical line

This retains the primary measured deviation while returning secondary coordinates to the theoretical axis relationship.

## Surrounding-plane measurement

Surrounding-plane measurement is selected when:

```text
surface_point_count >= 3
```

The macro:

1. selects a reference vector that is not parallel to the active measurement vector
2. creates two orthogonal unit vectors in the measurement surface
3. distributes probing points around the technical measurement position
4. measures an actual plane
5. reads the actual plane position and vector
6. removes material-thickness and measurement-offset displacement
7. retains reference-feature displacement
8. constructs the final actual point from the corrected plane and a theoretical line

The probing pattern follows all technical position corrections automatically because every probing point is generated relative to the calculated plane center.

## Surrounding-plane result line

The theoretical line used to construct the final point depends on `snap_to_axis`.

```text
snap_to_axis = 0 -> result line follows the nominal point vector
snap_to_axis = 1 -> result line follows the dominant coordinate axis
```

The final point is constructed from the intersection of:

```text
corrected actual plane
+
theoretical result line through the original nominal point
```

## POINT_GET_MAIN_AXES

### Purpose

`POINT_GET_MAIN_AXES` determines:

- the dominant coordinate axis of a nominal point vector
- the coordinate axes relevant for tolerance output

### Interface

```dmis
CALL/M(POINT_GET_MAIN_AXES),$
  point_name,$
  angle_limit
```

### Results

The macro writes to global state:

```text
point_main_axis
point_axis_mask
```

`point_main_axis` values:

```text
1 = X
2 = Y
4 = Z
```

`point_axis_mask` values:

```text
0 = no coordinate tolerance
1 = X
2 = Y
3 = X + Y
4 = Z
5 = X + Z
6 = Y + Z
7 = X + Y + Z
```

### Angular limit

```text
0.0 = dominant axis only
> 0 = add secondary axes that reach the angular component limit
```

The component threshold is calculated from the angular limit:

```text
component_limit = sin(angle_limit)
```

with the angle converted from degrees to radians.

Absolute vector components are used, so positive and negative vector directions are treated identically for axis selection.

## Basic example

```dmis
F(POINT_1)=FEAT/POINT,CART,$
  100.0,50.0,25.0,$
  0.0,0.0,1.0

CALL/M(POINT_MEASURE),$
  'POINT_1',$
  0,$
  0,$
  0.0,$
  0.0,$
  0,$
  1,$
  0.0,$
  '',$
  '',$
  ''
```

The calling program must select the correct probe and establish safe travel before calling the macro.

## Referenced measurement example

```dmis
CALL/M(POINT_MEASURE),$
  'POINT_2',$
  0,$
  0,$
  0.0,$
  0.0,$
  0,$
  1,$
  0.0,$
  'REFERENCE_X',$
  '',$
  'REFERENCE_Z'
```

This uses X and Z deviations from previously measured reference features while leaving Y uncompensated.

## Surrounding-plane example

```dmis
CALL/M(POINT_MEASURE),$
  'POINT_3',$
  0,$
  0,$
  0.0,$
  0.0,$
  1,$
  5,$
  2.5,$
  '',$
  '',$
  ''
```

This measures five points around the technical point position and constructs the result using the actual measured surface and the dominant coordinate axis.

## Validation checklist

Before production use, verify:

- nominal point coordinates and vector
- selected vector convention
- inverse-direction behavior
- material thickness and sign
- measurement offset and sign
- reference-feature availability
- reference-feature axis assignment
- probe orientation
- clearance and travel paths
- `surface_point_count`
- `surface_point_distance`
- auxiliary-feature naming
- final actual point coordinates
- actual surface-vector behavior
- axis-snapping result

## Related documentation

- [Architecture](architecture.md)
- [Point tolerances](point-tolerances.md)
- [Point output](point-output.md)
- [Customer conventions](customer-conventions.md)
