# Architecture

## Overview

The **DMIS Macro Library** is structured around a separation of measurement, tolerance generation, and result output.

The main design goal is to keep reusable measurement logic inside the library while leaving part-specific definitions and machine-specific execution decisions in the calling DMIS program.

The library therefore does not attempt to replace a complete measurement program.

The calling program remains responsible for tasks such as:

- defining nominal features
- selecting probes and probe orientations
- defining machine movements and clearance positions
- supplying part-specific measurement parameters
- selecting the required measurement strategy
- selecting the required tolerance strategy
- validating all machine movements and measurement results

The current core API focuses on point-like features.

---

## Core design

Point processing is divided into three functional layers:

```text
Measurement
    |
    v
Tolerance generation
    |
    v
Output
```

These layers are intentionally separated.

A measurement strategy can therefore be changed without necessarily changing tolerance-generation logic, while output formatting can evolve independently of the physical measurement strategy.

---

## Processing workflow

A typical point-processing sequence is:

```text
Nominal point feature
        |
        v
POINT_MEASURE
        |
        v
Actual point feature
        |
        v
POINT_OUTPUT_RESET
        |
        +--------------------------+
        |                          |
        v                          v
Coordinate tolerances       Additional tolerances
POINT_TOL_*                 POINT_TOL_NORMAL
                            POINT_TOL_POSITION
        |                          |
        +------------+-------------+
                     |
                     v
             POINT_OUTPUT_WRITE
                     |
              +------+------+
              |             |
              v             v
        Separate output   Combined output
```

The nominal feature is defined by the calling program before the library is invoked.

The library then performs measurement, prepares the required tolerance definitions, and finally writes the requested output.

---

# 1. Measurement layer

The measurement layer determines how a nominal point feature is physically measured and how the resulting actual point is constructed.

The primary macro is:

```text
POINT_MEASURE
```

Supporting utility:

```text
POINT_GET_MAIN_AXES
```

## Responsibilities

The measurement layer can handle:

- reading the nominal point position
- reading the nominal point vector
- vector normalization
- measurement-direction selection
- material-thickness correction
- additional measurement offset
- optional reference-feature compensation
- direct single-point measurement
- surrounding-plane measurement
- actual surface-vector evaluation
- dominant-axis determination
- coordinate-axis snapping
- construction of the final actual point

The measurement layer is responsible for the resulting actual feature.

It is not responsible for creating tolerances or writing tolerance output.

---

## Nominal and actual features

The library assumes that the nominal point feature already exists.

Conceptually:

```text
F(POINT_NAME)
```

represents the nominal feature.

After measurement and processing:

```text
FA(POINT_NAME)
```

represents the corresponding actual feature.

The library attempts to preserve the original feature name so that the calling program can continue working with the same logical point identifier.

Auxiliary features may be created internally where required by a measurement strategy.

Examples include:

```text
<point_name>_PLN
<point_name>_LIN
```

These auxiliary features support operations such as surrounding-plane evaluation and coordinate-axis snapping.

---

# 2. Tolerance-generation layer

Tolerance generation is deliberately separated from physical measurement.

The tolerance macros determine:

- which tolerances exist
- which coordinate axes are relevant
- how tolerance limits are calculated
- how tolerance names are generated

Current tolerance macros include:

```text
POINT_TOL_CORTOL
POINT_TOL_MAIN_AXES
POINT_TOL_VECTOR_CORTOL
POINT_TOL_NORMAL
POINT_TOL_POSITION
```

No result output is generated directly by these macros.

Instead, the macros create the required tolerance objects and update the library's global output state.

---

## Coordinate-axis mask

Coordinate tolerances use a bit-mask representation.

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

This mask is stored in:

```text
point_axis_mask
```

The representation allows the output layer to determine which coordinate tolerances have been prepared without needing to reproduce the original tolerance-selection logic.

---

## Dominant axis

The dominant nominal point-vector axis is stored in:

```text
point_main_axis
```

Values are:

```text
1 = X
2 = Y
4 = Z
```

The dominant axis is determined from the largest absolute component of the normalized nominal point vector.

Using absolute components means positive and negative vector directions use the same axis-selection logic.

---

# 3. Output layer

The output layer is responsible only for writing already prepared measurement and tolerance results.

Primary macros are:

```text
POINT_OUTPUT_RESET
POINT_OUTPUT_WRITE
POINT_OUTPUT_COMBINED_F
POINT_OUTPUT_COMBINED_FA
```

The output layer does not decide how a feature is measured and does not calculate tolerance limits.

---

## Output state

The library maintains global state describing which tolerances have been prepared for the current point.

Current state variables include:

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

These variables form the interface between tolerance generation and result output.

---

## State lifecycle

The output state belongs logically to one point-processing operation.

Before preparing tolerances for a new point, the state should be cleared:

```dmis
CALL/M(POINT_OUTPUT_RESET)
```

The intended lifecycle is:

```text
POINT_OUTPUT_RESET
        |
        v
Create coordinate tolerances
        |
        v
Optionally create normal tolerance
        |
        v
Optionally create position tolerance
        |
        v
POINT_OUTPUT_WRITE
```

Reusing stale output state from a previously processed point must be avoided.

---

## Active tolerance flags

Two independent flags indicate whether additional tolerance types have been prepared.

### Normal tolerance

```text
point_normal_active
```

Values:

```text
0 = no normal tolerance prepared
1 = normal tolerance prepared
```

### Position tolerance

```text
point_position_active
```

Values:

```text
0 = no position tolerance prepared
1 = position tolerance prepared
```

Together with `point_axis_mask`, these flags allow the output layer to determine the set of tolerance results that must be written.

---

# 4. Separate and combined output

`POINT_OUTPUT_WRITE` supports two output concepts.

## Separate output

Each tolerance is written using an individual `OUTPUT` statement.

Conceptually:

```text
Feature + X tolerance
Feature + Y tolerance
Feature + normal tolerance
...
```

This mode keeps each output operation independent.

## Combined output

All prepared tolerances for a feature are written in one combined `OUTPUT` statement.

The helper macros:

```text
POINT_OUTPUT_COMBINED_F
POINT_OUTPUT_COMBINED_FA
```

handle nominal and actual features separately.

The combined-output implementation evaluates:

```text
point_axis_mask
point_normal_active
point_position_active
```

to determine the required output combination.

---

# 5. Measurement reference compensation

`POINT_MEASURE` can optionally use previously measured reference features.

Three independent reference-feature parameters are available:

```text
ref_x
ref_y
ref_z
```

Each reference corresponds to one coordinate axis.

An empty feature name disables compensation for that axis.

The reference deviation is conceptually calculated as:

```text
reference deviation =
    actual reference coordinate
    -
    nominal reference coordinate
```

The deviation modifies the technical probing position.

Conceptually:

```text
technical probing position =
    nominal point position
    + reference deviation
    + material-side correction
    + measurement offset
```

Reference compensation is intended to help locate the expected physical workpiece position.

The reference deviation is therefore not automatically removed from the final measured result.

---

# 6. Material-side and vector conventions

The measurement layer supports different interpretations of the nominal point vector.

The selected convention determines how material thickness and inverse measurement direction affect the technical measurement position and active measurement vector.

This logic is intentionally encapsulated inside `POINT_MEASURE`.

The calling program supplies the required convention through macro parameters rather than duplicating the underlying vector logic.

Customer-specific conventions are documented separately in:

```text
docs/customer-conventions.md
```

---

# 7. Surrounding-plane strategy

A point can optionally be evaluated using a surrounding measured plane.

When the configured number of surface points reaches the surrounding-plane threshold, the library:

1. determines the technical measurement position
2. creates two orthogonal vectors in the measurement surface
3. distributes probing points around the technical point
4. measures an actual plane
5. evaluates the actual plane position and surface vector
6. corrects technical measurement displacement where required
7. constructs the final point from the measured plane and a theoretical line

The theoretical result line can follow either:

- the nominal point vector, or
- the dominant coordinate axis

depending on the selected snapping strategy.

This allows surface evaluation and coordinate-axis snapping to be combined.

---

# 8. Customer conventions

Customer-specific behavior should be kept separate from generic geometric and measurement logic wherever practical.

Current development considers naming and tolerance conventions for:

```text
Generic
Volkswagen / Audi
Mercedes-Benz
BMW
```

These conventions primarily affect areas such as:

- tolerance naming
- tolerance type selection
- vector interpretation
- output strategy

They do not imply that this project is an official library of any referenced manufacturer.

Customer-convention details are documented in:

```text
docs/customer-conventions.md
```

---

# 9. Generic core vs. specialized strategies

The `POINT_*` macros form the generic core of the library.

Specialized or experimental measurement strategies should not silently change the behavior of this core API.

The repository therefore separates experimental code under:

```text
experimental/
```

A specialized strategy may eventually be promoted into the main library after:

- its interface is stable
- its assumptions are documented
- expected results are defined
- behavior has been sufficiently validated

---

# 10. Library state and reentrancy

The current design uses global DMIS variables for communication between tolerance-generation and output macros.

This makes the output workflow straightforward but also means the output state is shared library state.

Consequences include:

- `POINT_OUTPUT_RESET` should be called before preparing a new point's tolerances
- tolerance preparation and output for one point should normally be completed before processing another point's output state
- nested or interleaved tolerance-output workflows require special care

Future versions may refine state handling while preserving the measurement/tolerance/output separation.

---

# 11. Calling-program responsibilities

The library intentionally does not control the complete measurement process.

The calling DMIS program remains responsible for:

```text
Nominal feature definition
Probe selection
Probe orientation
Safe machine travel
Clearance positions
Part-specific parameter values
Calling sequence
Measurement validation
```

This prevents reusable library functions from making assumptions about machine configuration, fixture layout, collision conditions, or part-specific access.

---

# 12. Safety boundary

The architecture separates reusable geometric logic from machine- and part-specific responsibility, but this separation does not guarantee safe machine operation.

Library macros may influence:

- probing positions
- measurement vectors
- constructed features
- technical offsets
- machine movements triggered by measurement statements
- tolerance definitions
- reported results

All library functions must therefore be validated in the target environment before production use.

See the safety section in the project `README.md` for the project's general safety and liability guidance.

---

# 13. Architectural principles

The library follows these core principles:

## Separate measurement from evaluation

Physical probing logic should not be mixed with result-output formatting.

## Separate tolerance definition from output

Tolerance macros prepare tolerance objects and state. Output macros decide how those prepared results are written.

## Preserve nominal feature identity

Where practical, the final actual feature uses the original nominal point name.

## Keep part-specific behavior outside the library

Probe selection, travel paths, clearance positions, and nominal feature definitions belong to the calling program.

## Make conventions explicit

Customer- or application-specific conventions should be selected using explicit parameters or wrapper macros rather than being hidden in generic functions.

## Prefer reusable geometry

Complex result construction should use reusable geometric relationships rather than hard-coded coordinate assumptions where practical.

## Keep experimental strategies separate

Experimental measurement approaches should be isolated until their interfaces and behavior are sufficiently validated.

---

# Related documentation

- [Point measurement](point-measurement.md)
- [Point tolerances](point-tolerances.md)
- [Point output](point-output.md)
- [Customer conventions](customer-conventions.md)