# Changelog

All notable changes to the Configurator File Type Specification will be documented in this file.

## [0.1.2] - 2026-04-07

### Added
- `cto_type` top-level field (`"element"`, `"assembly"`, `"template"`) enabling files to declare their intended role, analogous to part vs. assembly files in parametric CAD tools
- `declares_interface` top-level section for `element` files, exposing connection points, geometry, MEP interfaces, constraints, and pricing to parent assemblies while keeping internals opaque
- `product_source` field on `placed_elements`, enabling elements to be resolved from a referenced `.cto` file rather than a flat catalog entry
- Section 7.2 validation rules for nested element resolution, including checksum verification, version matching, and opaque interface enforcement

### Changed
- Section 3.4 top-level structure updated to include `cto_type` and `declares_interface`
- Section 4 renumbered to accommodate new `cto_type` (4.1) and `declares_interface` (4.5) subsections
- Section 7.1 Referential Integrity updated to allow `product_source` as an alternative to catalog resolution

### Fixed
- (None)

## [0.1.1] - 2026-04-05

### Added
- Interface standard conformance tracking
- Chain of custody framework and legal mateline documentation
- Expanded product schema with structural and thermal performance attributes
- `fulfillment_plan` replacing `delivery_schedule`

### Changed
- (None)

### Fixed
- (None)

## [0.1.0] - 2026-04-05

### Added
- Initial draft specification
- Contributing guidelines
- Example files structure

### Changed
- (None)

### Fixed
- (None)
