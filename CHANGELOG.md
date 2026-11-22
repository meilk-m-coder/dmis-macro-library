# Changelog

All notable changes to this project will be documented in this file.

The project follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned

- Additional validation of point measurement strategies in WM | Quartis
- Extended example programs
- Documented test cases and expected results
- Further customer-specific strategy wrappers
- Review and stabilization of the public macro interfaces
- Additional reusable DMIS measurement strategies

## [0.1.0] - Initial Development Release

### Added

- Initial public version of the DMIS Macro Library
- Main library file `src/QuartisSystemMacros.dmi`
- Global state management for point tolerance and output processing

### Measurement

- `POINT_GET_MAIN_AXES`
  - dominant-axis detection from the nominal point vector
  - configurable angular limit for secondary-axis selection
  - bit-mask based axis selection

- `POINT_MEASURE`
  - direct single-point measurement
  - configurable vector convention
  - inverse measurement direction
  - material-thickness correction
  - additional measurement offset
  - optional X, Y and Z reference-feature compensation
  - dominant-axis snapping
  - surrounding-plane measurement
  - actual surface-vector evaluation
  - construction of the final result point from measured and theoretical geometry

### Tolerance generation

- `POINT_TOL_CORTOL`
  - explicit coordinate tolerances using an axis mask

- `POINT_TOL_MAIN_AXES`
  - coordinate tolerances derived from the dominant point-vector axis
  - optional secondary-axis selection using an angular limit
  - tolerance limits calculated from vector components and normal tolerance

- `POINT_TOL_VECTOR_CORTOL`
  - coordinate tolerances for all relevant point-vector components
  - epsilon filtering for very small vector components

- `POINT_TOL_NORMAL`
  - profile/normal tolerance generation
  - configurable naming and tolerance conventions

- `POINT_TOL_POSITION`
  - 3D position tolerance generation
  - configurable tolerance naming conventions

### Output

- `POINT_OUTPUT_RESET`
  - reset of the global point-output state

- `POINT_OUTPUT_WRITE`
  - separate or combined result output
  - nominal feature output
  - actual feature output
  - combined nominal and actual output

- `POINT_OUTPUT_COMBINED_F`
  - combined nominal-feature tolerance output

- `POINT_OUTPUT_COMBINED_FA`
  - combined actual-feature tolerance output

### Documentation

- Initial project README
- Project architecture description
- Safety and validation guidance
- Planned detailed documentation for:
  - point measurement
  - point tolerances
  - point output
  - customer conventions

### Project structure

- `src/` for the main DMIS macro library
- `docs/` for technical documentation
- `examples/` for neutral usage examples
- `experimental/` for experimental measurement strategies
- `tests/` for reproducible test scenarios
- `screenshots/` for project illustrations

### Licensing

- Added MIT License
- Added explicit safety guidance for use with physical measurement equipment

## Notes

Version `0.1.0` is an early development release.

The public macro interfaces, parameter conventions, measurement strategies, tolerance handling, and output behavior may change before version `1.0.0`.

All measurement strategies must be validated before production use, particularly when macros can influence machine movements, probe paths, feature construction, tolerances, or reported measurement results.
