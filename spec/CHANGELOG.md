# Changelog

All notable changes to the Configurator File Type Specification will be documented in this file.

## [0.1.4] - 2026-04-09

### Added
- Three-layer geometry schema: Bounding Box (inline JSON, required), Model Geometry (URL reference, optional), and IFC (parallel URL field, optional, always IFC4.3)
- Drafting Geometry layer with named views: `plan` (required if `drafting_geometry` present), `elevation_front`, `elevation_back`, `elevation_left`, `elevation_right` (optional), and `sections` array (optional)
- Supported geometry formats: Model Geometry — `gltf`, `glb`, `obj`, `stl`, `3dm`; Drafting Geometry — `svg` (preferred), `dxf` (preferred), `dwg` (supported for compatibility). No proprietary formats.
- Unified Party Schema (Section 5.3) — common structure for all six party roles: manufacturer, logistics company, GC, hoisting company, pod installer, panel installer. Each party record includes name, structured address, website, and insurer (carrier, policy number, coverage type, expiration date)
- `parties` block at top level of `fulfillment_plan` consolidating all project party records
- Design Principle 2.8: Format-Agnostic Geometry
- Geometry Layer Summary table (Section 5.2)

### Changed
- `geometry` block in both `declares_interface` and Product Library Schema replaced with three-layer structure
- Connection direction naming updated from cardinal (north/south/east/west) to relative (front/back/left/right) throughout product schema and fulfillment plan
- Scattered party fields in `fulfillment_plan` (previously in `manufacturing_schedule.factory`, `delivery_manifest.shipments[].carrier`, `crane_lift_plan.crane`, `installation_schedule.general_contractor`, `crew_assignments[].provider`) consolidated into unified `parties` block
- Section 5.2 renamed from Category Values to Geometry Layer Summary; Category Values moved to Section 5.4; Design Line Values moved to Section 5.5; Interface Standard Conformance moved to Section 5.6

### Fixed
- (None)

## [0.1.3] - 2026-04-07

### Added
- `published_date` field on `declares_interface` — records when the manufacturer posted this version of the product file
- `thermal_performance` block on `declares_interface` — exposes U-factor, R-values, and climate zone compatibility to parent assemblies
- `price_quote` object on `placed_elements` — records live MSRP, ship date, lead time, quote expiration, and manufacturer contact retrieved from the manufacturer at configurator session time; distinct from the published price in `declares_interface.pricing`
- Note clarifying that multiple instances of the same product type are represented as separate `placed_elements` entries, each with a unique `instance_id` and independent `price_quote` and `instance_data`

### Changed
- `declares_interface` field table updated to include `product_version`, `published_date`, and `thermal_performance`
- `placed_elements` field table updated to include full `price_quote` schema

### Fixed
- (None)

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
