# Changelog

All notable changes to the Configurator File Type Specification will be documented in this file.

## [0.1.5] - 2026-04-12

### Added

- **Unified Party Schema**: Structured blocks for all chain-of-custody actors beyond the manufacturer. New party types: `logistics_company`, `gc_accepting_shipping`, `hoisting_company`, `pod_installer`, `panel_installer`. Each party block carries: `name`, `address`, `website`, and a structured `insurer` object.
- **Structured Insurer Block**: Replaces informal insurance fields throughout the spec. Fields: `carrier_name`, `policy_number`, `coverage_type`, `expiration_date`.
- **Manufacturer Party Expansion**: `catalog_reference` now includes `manufacturer_address`, `manufacturer_website`, `catalog_edition`, `catalog_url`, and `product_url`.
- **Nominal vs. Actual Dimensions**: Products now declare both `nominal_dimensions` (the grid footprint used by the configurator for snapping and placement) and `bounding_box` (the actual physical shipping envelope). This distinction supports the LBS Chase-Generating Adjacency pattern.
- **Chase Zones**: New `chase_zones` array in product geometry. Each zone declares `side`, `depth_ft`, and `purpose`. Chase zones represent the exterior-mounted MEP service volumes that generate service chases when two pods are placed adjacently.
- **Installation Orientation Constraints**: New `permitted_orientation` field in product constraints. Values: `horizontal`, `vertical`, `sloped`. Prevents illegal rotations at placement time. Floor cartridges declare `horizontal`, wall panels declare `vertical`, roof panels declare `sloped` with an optional `pitch_range`.
- **Roof Pitch Constraints**: New `pitch_range` object in product constraints for roof panels. Fields: `min_pitch` and `max_pitch` (expressed as rise/run strings, e.g., `"3/12"`). Fixed-pitch panels declare `fixed_pitch` instead.
- **Subcategory Enum Expansion**: `wall_panel` subcategory now includes `solid`, `window`, `door`, `mixed`. `floor_cartridge` subcategory now includes `standard`, `stairwell`.
- **Opening Geometry**: New `opening_geometry` block in product geometry for wall panels with subcategory `window`, `door`, or `mixed`. Fields: `type`, `width_ft`, `height_ft`, `offset_x_ft`, `offset_z_ft`.
- **Cartridge vs. Panel Terminology**: Codified in Section 1.3. `floor_cartridge` is reserved for interior horizontal assemblies that may carry finish options. `wall_panel` and `roof_panel` denote exterior products.
- **Floor Level Manager**: New `floor_levels` array in assembly files. Each level declares `level_id`, `name`, `z_origin_ft`, `ceiling_height_ft`, and `is_foundation`. Replaces bare `story` integers for multi-story coordination. `story` integer is retained for backward compatibility.
- **Stairwell Conceptual Object**: New `conceptual_objects` array in assembly files. Supports `type: "stairwell_volume"` with `level_range`, `nominal_dimensions`, and `behavior_rules` (`force_void_alignment`, `magnetic_snap_edges`).
- **Sided Interface Roles**: New `interface_roles` array in product connectivity. Each role declares `cis_id`, `version`, `face` (geometric face of the product: `front`, `back`, `left`, `right`, `top`, `bottom`), and `side_addressed` (the named side from the referenced CIS standard). Enables surface-specific interface mapping without embedding engineering data.
- **CIS Registry Reference**: Products reference external Configure-to-Order Interface Standard (`.cis`) files by `cis_id` and `version` via URL link rather than embedding engineering data. This keeps element files lightweight while maintaining traceability.
- **Component Display ID**: New `component_display_id` field in placed elements. A human-readable unique ID per placed component (e.g., `FP-001` for floor panel 1, `KP-001` for kitchen pod 1) used to cross-reference BOM line items with tagged export drawings.
- **3D Asset URL**: New `model_glb_url` field in product documentation block. Points to a GLB (Binary GLTF) file for Three.js rendering. Companion to the existing `model_3d_url` (GLTF) and `model_bim_url` (IFC) fields.
- **GLB Coordinate System Standard**: GLB assets MUST be scaled to meters (1 unit = 1 meter, per glTF 2.0 standard), use Y-Up orientation, and register the South-West-Bottom (SWB) corner of the product at world origin `[0, 0, 0]`. Product depth grows along the negative Z-axis.

### Changed

- `catalog_reference` block expanded with new manufacturer and catalog fields (backward compatible — new fields are optional).
- Insurance fields throughout `chain_of_custody` standardized to the new structured `insurer` block.
- `story` integer field in `floor_cartridges` and `placed_elements` retained but supplemented by the new `level_id` string reference to the `floor_levels` array.

### Fixed

- Version labeling inconsistency: spec header now reads `0.1.5` to match internal `cto_version` field usage pattern.

## [0.1] - 2026-04-05

### Added
- Initial draft specification
- Contributing guidelines
- Example files structure

### Changed
- (None yet)

### Fixed
- (None yet)

---

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
