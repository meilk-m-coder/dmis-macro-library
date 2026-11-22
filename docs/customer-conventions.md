# Customer Conventions

## Overview

The library supports configurable naming, tolerance, and vector conventions for different measurement environments.

Current convention identifiers are:

```text
0 = Generic
1 = Volkswagen / Audi
2 = Mercedes-Benz
3 = BMW
```

These identifiers are implementation helpers used by the macro library. They do not represent official libraries, specifications, or endorsements from the referenced manufacturers.

> The calling program must verify that the selected convention matches the applicable customer requirements, project rules, and target-system behavior.

## Scope

Customer conventions currently affect:

- nominal-vector interpretation during point measurement
- material-thickness correction
- normal/profile tolerance naming
- normal/profile tolerance type
- 3D position tolerance naming

Other customer-specific rules can be implemented later using wrapper macros while keeping the generic core macros stable.

## Generic convention

Convention value:

```text
0
```

### Vector convention

The nominal point vector is interpreted as pointing away from the material.

### Normal tolerance

```text
Name: <point_name>N
Type: PROFS
```

### Position tolerance

```text
Name: <point_name>POS
Type: POS, 3D
```

### Example

For:

```text
point_name = POINT_1
```

Generated names are:

```text
Normal tolerance: POINT_1N
Position tolerance: POINT_1POS
```

## Volkswagen / Audi convention

Convention value:

```text
1
```

### Normal tolerance

```text
Name: <point_name>P
Type: PROFP
```

### Position tolerance

```text
Name: <point_name>P2
Type: POS, 3D
```

### Example

For:

```text
point_name = POINT_1
```

Generated names are:

```text
Normal tolerance: POINT_1P
Position tolerance: POINT_1P2
```

## Mercedes-Benz convention

Convention value:

```text
2
```

### Vector convention

The point-measurement interface can use a through-material nominal-vector interpretation:

```text
vector_convention = 1
```

This influences material-thickness correction and inverse measurement direction in `POINT_MEASURE`.

### Normal tolerance

```text
Name: <point_name>N
Type: PROFS
```

### Position tolerance

```text
Name: <point_name>N2
Type: POS, 3D
```

### Example

For:

```text
point_name = POINT_1
```

Generated names are:

```text
Normal tolerance: POINT_1N
Position tolerance: POINT_1N2
```

### Measurement example

```dmis
CALL/M(POINT_MEASURE),$
  'POINT_1',$
  1,$
  0,$
  0.8,$
  0.0,$
  0,$
  1,$
  0.0,$
  '',$
  '',$
  ''
```

The example uses:

- through-material vector convention
- default measurement side
- material thickness of `0.8`
- direct single-point measurement

The values are illustrative only and must be validated for the actual project.

## BMW convention

Convention value:

```text
3
```

### Normal tolerance

```text
Name: N<point_name>
Type: PROFS
```

### Position tolerance

```text
Name: P<point_name>
Type: POS, 3D
```

### Example

For:

```text
point_name = POINT_1
```

Generated names are:

```text
Normal tolerance: NPOINT_1
Position tolerance: PPOINT_1
```

## Convention summary

| Value | Convention | Normal name | Normal type | Position name |
|---:|---|---|---|---|
| 0 | Generic | `<point_name>N` | `PROFS` | `<point_name>POS` |
| 1 | Volkswagen / Audi | `<point_name>P` | `PROFP` | `<point_name>P2` |
| 2 | Mercedes-Benz | `<point_name>N` | `PROFS` | `<point_name>N2` |
| 3 | BMW | `N<point_name>` | `PROFS` | `P<point_name>` |

## Coordinate tolerances

Coordinate tolerance names currently use a shared convention:

```text
X tolerance: <point_name>X
Y tolerance: <point_name>Y
Z tolerance: <point_name>Z
```

Example:

```text
POINT_1X
POINT_1Y
POINT_1Z
```

If a customer requires different coordinate-tolerance naming, that behavior should be added through an explicit convention or wrapper rather than hidden inside the generic implementation.

## Strategy wrappers

The library contains a section reserved for customer strategy wrappers.

Wrapper macros can simplify repeated calls by supplying fixed convention values and project defaults.

Conceptual example:

```dmis
M(CUSTOMER_POINT_TOLERANCES)=MACRO/point_name,$
                                   normal_lower,$
                                   normal_upper

  CALL/M(POINT_OUTPUT_RESET)

  CALL/M(POINT_TOL_MAIN_AXES),$
    point_name,$
    20.0,$
    normal_lower,$
    normal_upper

  CALL/M(POINT_TOL_NORMAL),$
    point_name,$
    1,$
    normal_lower,$
    normal_upper

ENDMAC
```

This is an architectural example only. Project-specific defaults must be verified before implementation.

## Fallback behavior

For unknown `output_convention` values, the current tolerance macros fall back to the generic naming convention.

This avoids leaving tolerance names undefined, but an unexpected fallback should still be treated as a configuration error in the calling program.

The calling program should pass only documented convention values.

## Separation of concerns

Customer conventions should describe naming and interpretation rules.

Customer conventions should not silently define:

- probe selection
- probe orientation
- clearance positions
- machine travel
- fixture-specific movements
- part-specific nominal geometry

These responsibilities remain in the calling measurement program.

## Neutral public examples

Public examples should use neutral identifiers such as:

```text
POINT_1
REFERENCE_X
REFERENCE_Z
```

Avoid publishing:

- confidential part numbers
- customer program names
- vehicle-project identifiers
- production measurement data
- internal naming specifications

## Validation checklist

For every convention, verify:

- nominal-vector interpretation
- material-thickness sign
- inverse-direction behavior
- normal tolerance type
- normal tolerance name
- position tolerance name
- coordinate tolerance names
- nominal and actual output behavior
- compatibility with customer requirements
- compatibility with the installed Quartis version

## Disclaimer

The convention names in this document are descriptive labels for implementation behavior.

This project is independent and is not affiliated with, endorsed by, or maintained by Volkswagen, Audi, Mercedes-Benz, BMW, WENZEL Group, or WENZEL Metromec AG.

All product names, company names, and trademarks belong to their respective owners.

## Related documentation

- [Architecture](architecture.md)
- [Point measurement](point-measurement.md)
- [Point tolerances](point-tolerances.md)
- [Point output](point-output.md)
