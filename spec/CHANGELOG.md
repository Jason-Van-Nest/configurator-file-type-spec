# CHANGELOG

This file tracks the version history of both the CTO File Format Specification and the CIS (Configure-to-Order Interface Standard) File Format Specification. Both specs are hosted in this repository (`configurator-file-type-spec`) and version independently. Entries are ordered most-recent-first, with `[CTO]` and `[CIS]` prefixes indicating which spec each entry documents.

For full version history of files predating this combined CHANGELOG, see the spec subfolders (`spec/cto/v0.1/specification.md` for CTO v0.1.6 and earlier, and `spec/cis/history/` for CIS pre-release drafts).

---

## [CTO] v0.2.0 — April 19, 2026

**CIS Companion Specification.** Major release introducing the CIS file format as a companion specification, formalizing how CTO element files declare conformance to CIS-defined connection standards.

### Added

- **§1.6 Companion Specifications** — explains the CIS file format, the architectural relationship between CTO and CIS, and the design principle that they are siblings (not parent/child)
- **§2.8 Specifications Speak to Humans** — new design principle: specifications must serve software developers, standards engineers, AND decision-makers (manufacturers, AHJs, building officials). Schema definitions alone serve only the first audience
- **§4.2 `element_meta` block** — distinct metadata block for `cto_type:"element"` files, separate from `project_meta` (which documents user-created assemblies)
- **§4.2a `declares_interface` block** — top-level array of CIS standard references, formalizing what was previously a forward-pointer ambiguity. Includes a Multi-Standard Elements subsection documenting common patterns
- **§10.3 Multi-Standard Element worked example** — the K01-UM kitchen pod, which declares conformance to both `CfOC-ICC-1220` (back face) and `LBS-INT-UTIL-KIT` (left/right faces)
- **§1.3 Terminology** — new entries for CIS File, Connection Plane, and Product-Local Face Labels

### Changed

- **§5.1 Product Library Schema** — removed `gender` from `connection_points` (gender now lives in CIS files via `connection_signature` per CIS spec §6); added `cis_port_id_ref`; substantially rewrote prose throughout to clarify ambiguous fields per the new §2.8 design principle
- **§5.7 Interface Standard Conformance** — substantially rewritten to frame `interface_roles` as a rotation-lock validation rule (not just informational metadata), with a worked bath-pod-rotation example
- **§7.8 Interface Compatibility validation** — rewritten to incorporate CIS-aware validation (port-pair signature compatibility, version negotiation per CIS spec §9.3)
- **§8.3 Migration** — added migration guidance from v0.1.6 to v0.2.0 for both assembly and element files
- **§1.5 Related Standards** — added CIS File Format and CfOC-ICC-1220-S to the table

### Clarified (8 in-scope spec observations from the authoring session)

- `bounding_box.min/max` coordinates are product-local feet, distinct from GLB Y-up coordinates (observation #2)
- Face labels (`front`, `back`, `left`, `right`, `top`, `bottom`) are product-local, NOT real-world compass directions (observation #3)
- `clearance_zones` describes EXTERNAL clearance requirements only — space outside the product envelope reserved for operation or access. NOT for documenting internal product operations like appliance door swings (observation #4)
- `interface_roles` is the primary mechanism preventing invalid product rotations at the configurator (observation #15)

### Deferred to v0.2.1 or later

- Coupler product class formalization (observation #7)
- Services chain validation (observation #12)
- Interface-carrier products documentation pattern (observation #13)
- `field_built_element` parent abstraction
- Base/foundation and rigging views in Drafting Geometry layer
- Stairwell / `conceptual_objects` reconciliation
- Formal JSON Schema (Appendix B)

---

## [CIS] v0.1.3 — April 19, 2026

**Optional ports.** Adds the `required: boolean` field to ports, enabling optional ports within a CIS standard. Optional ports may be present on one side of a connection plane and absent on the other; the configurator silently treats unmated optional ports as terminated at the manufacturing stage. This addition supports the common case where a single CIS standard governs both fully-equipped products (e.g., a kitchen pod with hot water) and reduced-feature products (e.g., a closet pod without hot water).

### Added

- §6.1 — `required: boolean` field on port schema, default `true`
- §6.5 rule 4 — optional-port presence asymmetry handling
- §6.5.1 — manufacturing implications of permissive optional ports
- §9.6 — adding `required: false` ports is a backward-compatible MINOR bump

---

## [CIS] v0.1.2 — April 18, 2026

**Connection signature registry.** Replaces the abstract `gender` field on ports with a material-aware `connection_signature` field drawn from a fixed registry (Appendix F). This change reflects the physical reality that connection topology (PEX barb-to-barb via intermediate pipe, copper male-to-female direct thread, PVC slip joints, flange-to-flange via gasket) varies by material and cannot be captured by a single male/female axis.

### Added

- Appendix F — Connection Signature Registry, with 39+ signatures across 7 categories (water supply, DWV, gas, electrical, data, HVAC, structural)
- §6.1 — `connection_signature` field replacing `gender`
- §6.2 — three signature patterns: same-with-intermediate, same-direct, gendered-direct
- §6.5 — port-pair mating rules including registry table lookup
- §7.1 — `intermediate_connector` block on utility connections (required when signatures are same-to-same-with-intermediate)
- §9.5 — registry governance: new signatures via MINOR bump, removed via MAJOR

### Changed

- §6.4 — mirror geometry rules updated for signature complementarity
- §11.4 — validation rules updated for signatures

---

## [CIS] v0.1.1 — April 18, 2026

**Connection plane reframing.** Sides are properties of the connection plane (defined in the CIS file), not properties of the product. A product's face *addresses* a side by reaching toward the plane from that side.

### Added

- §1.3 — Terminology entries for "Connection Plane," "Sides," "Upstream / Downstream"
- §5.4 — explicit rule that side names are scoped to their CIS file (not globally unique)
- §10.3 — `{MFG}-INT-{upstream}-{downstream}` naming convention for proprietary standards, with chain-order semantics

### Changed

- §2.2 and §5 — language reworked to describe sides as properties of the connection plane, not the product
- §15.2 — `interface_roles` framed as a rotation-lock validation rule, not just an informational mapping
- §15.3 — added warnings-vs-errors philosophy for design-time vs. manufacturing-ready

---

## [CIS] v0.1.0 — April 18, 2026

**Initial publication.** First version of the CIS file format specification. Establishes the baseline structure: top-level metadata, sides, ports with geometry, utility connections, structural connections, versioning, and the open-vs-proprietary distinction.

### Added

- Initial 16-section specification covering file structure, schema, sidedness, port geometry, utility/structural connections, versioning, MIME type, validation, and conformance
- Companion to the CTO file format specification (released in coordination)

---

## [CTO] v0.1.6 — April 17, 2026

(See `spec/cto/v0.1/specification.md` for the v0.1.6 CTO spec. Pre-v0.2.0 CHANGELOG entries are preserved in the file's git history; future entries appear here.)

### Added (summary)

- Entourage product type (`cto_type: "entourage"`)
- Stairwell Bounding Volume — first-class hosting element for prefab staircases
- Level Datum Architecture with Roof Prism — levels as first-class abstract hosts independent of floor plates

### Changed

- Unified Party Schema standardized across `chain_of_custody`, `fulfillment_plan`, and chain-of-custody actor fields
- Sided Interface Roles introduced (foundation for v0.2.0's full CIS coexistence work)

---

## Earlier versions

For CTO versions earlier than v0.1.6, see the git history of `spec/cto/v0.1/specification.md`. The CIS specification did not exist before v0.1.0 (April 18, 2026).
