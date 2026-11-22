# Point Output

## Overview

The output layer writes previously prepared point tolerances and feature results.

Output is deliberately separated from measurement and tolerance generation.

The output macros do not calculate tolerance limits and do not decide how a feature is measured. They read the global output state created by the tolerance macros.

Current output macros are:

```text
POINT_OUTPUT_RESET
POINT_OUTPUT_WRITE
POINT_OUTPUT_COMBINED_F
POINT_OUTPUT_COMBINED_FA
```

## Global output state

The output layer uses:

```text
point_main_axis
point_axis_mask

point_normal_active
point_position_active

point_tol_x_name
point_tol_y_name
point_tol_z_name
point_tol_normal_name
point_tol_position_name
```

## POINT_OUTPUT_RESET

### Purpose

Clears all prepared point-tolerance state and names.

### Interface

```dmis
CALL/M(POINT_OUTPUT_RESET)
```

### Reset values

```text
point_main_axis = 0
point_axis_mask = 0

point_normal_active = 0
point_position_active = 0

point_tol_x_name = ''
point_tol_y_name = ''
point_tol_z_name = ''
point_tol_normal_name = ''
point_tol_position_name = ''
```

### Required use

Call `POINT_OUTPUT_RESET` before preparing tolerances for each new point.

Failure to reset the state can cause tolerance names or active flags from a previously processed point to be reused.

## POINT_OUTPUT_WRITE

### Purpose

Writes all previously prepared tolerance results for a point.

### Interface

```dmis
CALL/M(POINT_OUTPUT_WRITE),$
  point_name,$
  feature_output,$
  output_style
```

## Parameters

### `point_name`

Name of the point feature whose results are written.

### `feature_output`

Controls nominal and actual feature output.

```text
0 = nominal feature only
1 = actual feature only
2 = nominal and actual feature
```

Nominal output uses:

```text
F(@point_name)
T(@tolerance_name)
```

Actual output uses:

```text
FA(@point_name)
TA(@tolerance_name)
```

### `output_style`

Controls separate or combined output.

```text
0 = separate OUTPUT statement for every tolerance
1 = one combined OUTPUT statement per selected feature
```

## Separate output

Separate output writes each prepared tolerance independently.

Possible outputs include:

- X coordinate tolerance
- Y coordinate tolerance
- Z coordinate tolerance
- normal/profile tolerance
- 3D position tolerance

Example conceptual result:

```dmis
OUTPUT/F(@point_name),T(@point_tol_x_name)
OUTPUT/F(@point_name),T(@point_tol_normal_name)
```

For actual-feature output:

```dmis
OUTPUT/FA(@point_name),TA(@point_tol_x_name)
OUTPUT/FA(@point_name),TA(@point_tol_normal_name)
```

Separate output is the preferred mode for the current Quartis-targeted implementation.

## Combined output

Combined output writes every prepared tolerance in one `OUTPUT` statement per selected feature.

Nominal combined output is delegated to:

```text
POINT_OUTPUT_COMBINED_F
```

Actual combined output is delegated to:

```text
POINT_OUTPUT_COMBINED_FA
```

## Criteria mode

The helper macros combine the two optional tolerance flags into one internal mode:

```text
criteria_mode = point_normal_active + 2 * point_position_active
```

The resulting values are:

```text
0 = neither normal nor position tolerance
1 = normal tolerance only
2 = position tolerance only
3 = normal and position tolerance
```

Together with `point_axis_mask`, this determines the complete tolerance list.

## Coordinate-mask handling

Supported masks are:

```text
1 = X
2 = Y
3 = X + Y
4 = Z
5 = X + Z
6 = Y + Z
7 = X + Y + Z
```

A default case handles mask `0` or an invalid mask. In that case, only active normal and position tolerances are written.

## POINT_OUTPUT_COMBINED_F

### Purpose

Writes one combined nominal-feature output statement.

### Interface

```dmis
CALL/M(POINT_OUTPUT_COMBINED_F),point_name
```

### Example combinations

X coordinate only:

```dmis
OUTPUT/F(@point_name),$
  T(@point_tol_x_name)
```

X coordinate and normal tolerance:

```dmis
OUTPUT/F(@point_name),$
  T(@point_tol_x_name),$
  T(@point_tol_normal_name)
```

X, Y, Z, normal, and position tolerances:

```dmis
OUTPUT/F(@point_name),$
  T(@point_tol_x_name),$
  T(@point_tol_y_name),$
  T(@point_tol_z_name),$
  T(@point_tol_normal_name),$
  T(@point_tol_position_name)
```

## POINT_OUTPUT_COMBINED_FA

### Purpose

Writes one combined actual-feature output statement.

### Interface

```dmis
CALL/M(POINT_OUTPUT_COMBINED_FA),point_name
```

The logic mirrors `POINT_OUTPUT_COMBINED_F`, but uses actual features and actual tolerances:

```text
FA(...)
TA(...)
```

## Typical output workflow

```dmis
CALL/M(POINT_OUTPUT_RESET)

CALL/M(POINT_TOL_CORTOL),$
  'POINT_1',$
  4,$
  0.0,0.0,$
  0.0,0.0,$
  -0.20,0.20

CALL/M(POINT_TOL_NORMAL),$
  'POINT_1',$
  0,$
  -0.20,$
  0.20

CALL/M(POINT_OUTPUT_WRITE),$
  'POINT_1',$
  2,$
  0
```

This example:

- resets previous state
- creates a Z coordinate tolerance
- creates a generic normal tolerance
- writes nominal and actual results
- uses separate output statements

## Combined-output example

```dmis
CALL/M(POINT_OUTPUT_WRITE),$
  'POINT_1',$
  2,$
  1
```

This requests one combined nominal output and one combined actual output.

## Output-state responsibilities

Tolerance macros are responsible for:

- creating tolerance definitions
- storing generated names
- setting active flags
- updating `point_axis_mask`

Output macros are responsible for:

- choosing nominal and/or actual feature output
- selecting separate or combined output
- assembling the tolerance list
- issuing `OUTPUT` statements

The calling program is responsible for:

- resetting state for each new point
- calling tolerance macros in the intended sequence
- selecting an output style supported by the target environment
- validating generated reports and results

## Limitations

The current combined-output implementation explicitly handles every axis-mask and criteria-mode combination.

This is verbose but predictable. Any additional tolerance category will require corresponding changes to the combined-output helper macros.

Because the state is global, interleaved output preparation for multiple points is not supported without careful state management.

## Validation checklist

Verify:

- `POINT_OUTPUT_RESET` is called for every new point
- `point_axis_mask` matches the intended axes
- normal and position flags are correct
- generated tolerance names exist
- nominal and actual feature selection is correct
- separate output is accepted by the target environment
- combined output is accepted by the target environment
- output order is suitable for downstream reporting
- no stale state from another point is present
- generated reports contain the expected result values

## Related documentation

- [Architecture](architecture.md)
- [Point measurement](point-measurement.md)
- [Point tolerances](point-tolerances.md)
- [Customer conventions](customer-conventions.md)
