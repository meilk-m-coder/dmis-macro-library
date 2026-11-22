# Changelog

All notable changes to this project will be documented in this file.

The project follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned

- Additional validation of measurement strategies in WM | Quartis
- Additional reusable DMIS measurement strategies
- Further customer-specific strategy wrappers
- Review and stabilization of the public macro interfaces
- Extended automated and manual test coverage
- Generic tolerance and output functions for additional feature types

## [0.1.0] - Initial Development Release

### Added

- Initial public version of the **DMIS Macro Library**
- Main library file `src/QuartisSystemMacros.dmi`
- Global state management for point tolerance and output processing
- Reusable point measurement, tolerance and output API
- Experimental hexagon measurement strategy

### Measurement

#### `POINT_GET_MAIN_AXES`

- Dominant-axis detection from the nominal point vector
- Configurable angular limit for secondary-axis selection
- Bit-mask based axis selection

#### `POINT_MEASURE`

- Direct single-point measurement
- Configurable vector convention
- Inverse measurement direction
- Material-thickness correction
- Additional measurement offset
- Optional X, Y and Z reference-feature compensation
- Dominant-axis snapping
- Surrounding-plane measurement
- Actual surface-vector evaluation
- Construction of the final result point from measured and theoretical geometry

### Tolerance generation

#### `POINT_TOL_CORTOL`

- Explicit coordinate tolerances using an axis mask

#### `POINT_TOL_MAIN_AXES`

- Coordinate tolerances derived from the dominant point-vector axis
- Optional secondary-axis selection using an angular limit
- Tolerance limits calculated from vector components and normal tolerance
- Correct lower/upper ordering for negative vector components

#### `POINT_TOL_VECTOR_CORTOL`

- Coordinate tolerances for all relevant point-vector components
- Epsilon filtering for very small vector components

#### `POINT_TOL_NORMAL`

- Profile/normal tolerance generation
- Configurable naming and tolerance conventions

#### `POINT_TOL_POSITION`

- 3D position tolerance generation
- Configurable tolerance naming conventions

### Output

#### `POINT_OUTPUT_RESET`

- Reset of the global point-output state

#### `POINT_OUTPUT_WRITE`

- Separate or combined result output
- Nominal feature output
- Actual feature output
- Combined nominal and actual output

#### `POINT_OUTPUT_COMBINED_F`

- Combined nominal-feature tolerance output

#### `POINT_OUTPUT_COMBINED_FA`

- Combined actual-feature tolerance output

### Reference compensation

- Independent optional X, Y and Z reference features
- Reference deviation calculated from actual minus nominal reference coordinates
- Reference deviation applied to the technical probing position
- Reference displacement intentionally retained in the measured result
- Added test case with simulated `+3.0 mm` Z reference deviation

### Surrounding-plane measurement

- Multi-point measurement around the technical point position
- Construction of orthogonal surface basis vectors
- Circular distribution of surrounding probing points
- Actual plane evaluation
- Correction of technical measurement displacement
- Construction of the final result point from actual plane and theoretical result line
- Optional combination with dominant-axis snapping

### Experimental

- Added experimental `HEX_BASE` strategy
- Added experimental `HEX_WITH_LINES` strategy
- Added experimental `HEX_MEASURE` strategy
- Added hexagon measurement example for an internal hexagonal feature
- Added explicit diameter and 3D position tolerance output
- Added explicit X, Y and Z coordinate tolerance output
- Experimental `HEX_*` macros remain part of `src/QuartisSystemMacros.dmi`
- Generic tolerance/output helpers for circle- or hole-like features are not yet implemented

### Customer conventions

- Added generic tolerance naming convention
- Added Volkswagen / Audi tolerance convention
- Added Mercedes-Benz tolerance convention
- Added BMW tolerance convention
- Added configurable normal/profile tolerance naming
- Added configurable 3D position tolerance naming
- Added configurable point-vector interpretation

### Examples

Added tested example programs for:

- `point-basic.dmi`
- `point-referenced.dmi`
- `point-referenced-with-deviation.dmi`
- `point-surface-plane.dmi`
- `tolerance-output.dmi`
- `hexagon-experimental.dmi`

### Testing

- Added reproducible test matrix in `tests/README.md`
- Added integration tests executed in WM | Quartis
- Added screenshots documenting tested measurement workflows
- Added DMO result artifacts for tested examples
- Added direct point-measurement test
- Added reference-compensation test without simulated deviation
- Added reference-compensation test with simulated `+3.0 mm` Z deviation
- Added surrounding-plane integration test
- Added explicit coordinate-tolerance tests
- Added normal/profile tolerance tests
- Added 3D position tolerance tests
- Added separate-output test
- Added combined-output test
- Added experimental hexagon integration test

### Documentation

Added project and technical documentation:

- `README.md`
- `docs/architecture.md`
- `docs/point-measurement.md`
- `docs/point-tolerances.md`
- `docs/point-output.md`
- `docs/customer-conventions.md`
- `tests/README.md`
- `dmo/README.md`

Documentation includes:

- Library architecture and processing layers
- Macro interfaces and parameters
- Global output-state lifecycle
- Reference compensation
- Surrounding-plane measurement
- Tolerance strategies
- Output strategies
- Customer conventions
- Test matrix and expected behavior
- Safety and validation guidance

### Test artifacts

Added DMO artifacts generated from tested example programs:

- `dmo/point-basic__M1.dmo`
- `dmo/point-referenced__M1.dmo`
- `dmo/point-referenced-with-deviation__M1.dmo`
- `dmo/point-surface-plane__M1.dmo`
- `dmo/tolerance-output__M1.dmo`
- `dmo/hexagon-experimental__M1.dmo`

Added screenshots documenting tested workflows:

- `screenshots/point-basic.png`
- `screenshots/point-referenced.png`
- `screenshots/point-referenced-with-deviation.png`
- `screenshots/point-surface-plane.png`
- `screenshots/point-surface-plane-result.png`
- `screenshots/tolerance-output.png`
- `screenshots/hexagon-experimental.png`

### Project structure

Established project structure for:

- `src/` - main DMIS macro library
- `docs/` - technical documentation
- `examples/` - neutral integration examples
- `tests/` - reproducible test matrix
- `dmo/` - generated DMO test artifacts
- `screenshots/` - visual documentation of tested workflows

### Licensing and safety

- Added MIT License
- Added explicit safety guidance for physical measurement equipment
- Added collision-risk warning
- Added guidance for validating probe configuration and machine movements
- Added disclaimer for measurement results, tolerances and production use
- Clarified that vendor-supplied software, macros, documentation and the DMIS standard are not part of the project license

## Notes

Version `0.1.0` is an early development release.

The public macro interfaces, parameter conventions, measurement strategies, tolerance handling, and output behavior may change before version `1.0.0`.

The current core API focuses primarily on point measurement and evaluation. Experimental strategies for additional feature types may evolve independently.

All measurement strategies must be validated before production use, particularly when macros can influence machine movements, probe paths, feature construction, tolerances, or reported measurement results.