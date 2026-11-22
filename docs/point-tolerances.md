# Point Tolerances

## Overview

The tolerance-generation layer creates tolerance definitions for an existing point feature.

Tolerance generation is intentionally separated from result output. The tolerance macros create tolerance objects and update global output state, but they do not issue `OUTPUT` statements.

Current tolerance macros are:

```text
POINT_TOL_CORTOL
POINT_TOL_MAIN_AXES
POINT_TOL_VECTOR_CORTOL
POINT_TOL_NORMAL
POINT_TOL_POSITION
```

Before preparing tolerances for a new point, reset the output state:

```dmis
CALL/M(POINT_OUTPUT_RESET)
```

## Global tolerance state

Coordinate tolerances update:

```text
point_axis_mask
point_tol_x_name
point_tol_y_name
point_tol_z_name
```

Normal/profile tolerance generation updates:

```text
point_normal_active
point_tol_normal_name
```

Position tolerance generation updates:

```text
point_position_active
point_tol_position_name
```

The output layer uses this state to determine which results must be written.

## Axis-mask representation

Coordinate tolerances use a bit mask:

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

## POINT_TOL_CORTOL

### Purpose

Creates explicitly specified coordinate tolerances for selected axes.

### Interface

```dmis
CALL/M(POINT_TOL_CORTOL),$
  point_name,$
  axis_mask,$
  x_lower,$
  x_upper,$
  y_lower,$
  y_upper,$
  z_lower,$
  z_upper
```

### Parameters

- `point_name`: name of the point feature
- `axis_mask`: axes for which coordinate tolerances are generated
- `x_lower`, `x_upper`: X tolerance limits
- `y_lower`, `y_upper`: Y tolerance limits
- `z_lower`, `z_upper`: Z tolerance limits

### Generated names

```text
X tolerance: <point_name>X
Y tolerance: <point_name>Y
Z tolerance: <point_name>Z
```

### Behavior

Only axes enabled by `axis_mask` receive a `TOL/CORTOL` definition.

Example:

```dmis
CALL/M(POINT_TOL_CORTOL),$
  'POINT_1',$
  5,$
  -0.10,0.10,$
  0.0,0.0,$
  -0.20,0.20
```

Mask `5` generates X and Z coordinate tolerances.

## POINT_TOL_MAIN_AXES

### Purpose

Creates coordinate tolerances from the nominal point vector and a normal tolerance range.

The dominant axis is always selected. Additional axes can be selected using an angular limit.

### Interface

```dmis
CALL/M(POINT_TOL_MAIN_AXES),$
  point_name,$
  angle_limit,$
  normal_lower,$
  normal_upper
```

### Axis selection

The macro calls:

```text
POINT_GET_MAIN_AXES
```

The resulting axis mask is stored in:

```text
point_axis_mask
```

### Tolerance calculation

For every selected axis:

```text
value_1 = vector_component * normal_lower
value_2 = vector_component * normal_upper

lower = MIN(value_1, value_2)
upper = MAX(value_1, value_2)
```

Sorting the calculated values ensures that negative vector components do not interchange the lower and upper limits incorrectly.

### Example

```dmis
CALL/M(POINT_TOL_MAIN_AXES),$
  'POINT_1',$
  20.0,$
  -0.50,$
  0.50
```

The dominant axis is always included. Secondary axes are included when their absolute normalized vector components reach the threshold corresponding to `20` degrees.

## POINT_TOL_VECTOR_CORTOL

### Purpose

Creates coordinate tolerances for every relevant non-zero component of the nominal point vector.

Unlike `POINT_TOL_MAIN_AXES`, this macro applies:

- no dominant-axis restriction
- no angular-limit filtering

### Interface

```dmis
CALL/M(POINT_TOL_VECTOR_CORTOL),$
  point_name,$
  normal_lower,$
  normal_upper
```

### Component filtering

Very small vector components are ignored using an internal epsilon:

```text
epsilon = 0.000001
```

For each component above epsilon, the tolerance is calculated from the vector component and the supplied normal limits.

### Result

The macro updates `point_axis_mask` by adding:

```text
1 for X
2 for Y
4 for Z
```

## Choosing a coordinate-tolerance strategy

Use `POINT_TOL_CORTOL` when coordinate limits are known explicitly.

Use `POINT_TOL_MAIN_AXES` when coordinate limits should be derived from a normal tolerance while focusing on the dominant axis and optionally significant secondary axes.

Use `POINT_TOL_VECTOR_CORTOL` when every relevant vector component should receive a derived coordinate tolerance.

## POINT_TOL_NORMAL

### Purpose

Creates a point normal/profile tolerance and prepares the tolerance for later output.

### Interface

```dmis
CALL/M(POINT_TOL_NORMAL),$
  point_name,$
  output_convention,$
  normal_lower,$
  normal_upper
```

### Supported conventions

| Value | Convention | Tolerance name | Type |
|---:|---|---|---|
| 0 | Generic | `<point_name>N` | `PROFS` |
| 1 | Volkswagen / Audi | `<point_name>P` | `PROFP` |
| 2 | Mercedes-Benz | `<point_name>N` | `PROFS` |
| 3 | BMW | `N<point_name>` | `PROFS` |

Unknown values fall back to the generic convention.

### Output state

After creating the tolerance:

```text
point_normal_active = 1
```

The generated name is stored in:

```text
point_tol_normal_name
```

## POINT_TOL_POSITION

### Purpose

Creates a 3D position tolerance using:

```text
TOL/POS,3D
```

### Interface

```dmis
CALL/M(POINT_TOL_POSITION),$
  point_name,$
  output_convention,$
  position_tolerance
```

### Supported conventions

| Value | Convention | Tolerance name |
|---:|---|---|
| 0 | Generic | `<point_name>POS` |
| 1 | Volkswagen / Audi | `<point_name>P2` |
| 2 | Mercedes-Benz | `<point_name>N2` |
| 3 | BMW | `P<point_name>` |

Unknown values fall back to the generic convention.

### Output state

After creating the tolerance:

```text
point_position_active = 1
```

The generated name is stored in:

```text
point_tol_position_name
```

## Typical tolerance workflow

```dmis
CALL/M(POINT_OUTPUT_RESET)

CALL/M(POINT_TOL_MAIN_AXES),$
  'POINT_1',$
  20.0,$
  -0.50,$
  0.50

CALL/M(POINT_TOL_NORMAL),$
  'POINT_1',$
  0,$
  -0.50,$
  0.50

CALL/M(POINT_TOL_POSITION),$
  'POINT_1',$
  0,$
  1.00

CALL/M(POINT_OUTPUT_WRITE),$
  'POINT_1',$
  2,$
  0
```

## Important lifecycle rule

Tolerance state is global. Complete the tolerance preparation and output for one point before reusing the global state for another point.

Always reset the state before preparing the next point:

```dmis
CALL/M(POINT_OUTPUT_RESET)
```

## Validation checklist

Verify:

- correct point name
- correct axis mask
- correct lower and upper limits
- correct normal-vector components
- correct angular limit
- expected tolerance names
- expected tolerance type
- expected customer convention
- active tolerance flags
- resulting `point_axis_mask`
- compatibility of generated tolerances with the target Quartis version

## Related documentation

- [Architecture](architecture.md)
- [Point measurement](point-measurement.md)
- [Point output](point-output.md)
- [Customer conventions](customer-conventions.md)
