# CTO File Format Specification

**Version:** 0.2.0
**Status:** Draft
**Published by:** Center for Offsite Construction (CfOC) at NYIT
**Specification URL:** https://centerforoffsiteconstruction.org/offsite-product-configurator-file-type/
**GitHub Repository:** https://github.com/Jason-Van-Nest/configurator-file-type-spec
**License:** Apache 2.0
**Last Updated:** April 19, 2026

---

## Abstract

The CTO (Configure-to-Order) file format is an open standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

This specification defines the JSON schema, validation rules, and interchange requirements for CTO files. Version 0.2.0 introduces the CIS (Configure-to-Order Interface Standard) file format as a companion specification, formalizes how CTO element files declare conformance to CIS-defined connection standards, and adopts a stated design principle that specifications must speak to humans as well as to parsers.

Major additions in v0.2.0 over v0.1.6:

- **CIS file format coexistence**: a companion `.cis` file format now lives in the same repository, governing connection interface standards. Element files reference CIS files for connection geometry, gendering, and utility specifications rather than embedding them
- **The `declares_interface` block**: a formal top-level block in element files listing the CIS standards each product conforms to
- **Rotation-lock framing for `interface_roles`**: the `interface_roles` block is now explicitly the primary mechanism preventing invalid product rotations at the configurator
- **The `element_meta` block**: element files now have their own metadata block, distinct from the `project_meta` block used by user assemblies
- **Eight clarifications** addressing real ambiguities surfaced during authoring of the first CIS files and the first element file (K01-UM kitchen pod)
- **A new design principle (§2.8)** stating that CTO and CIS specifications must speak to three audiences: software developers, standards engineers, and decision-makers

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [File Structure](#3-file-structure)
4. [Schema Definition](#4-schema-definition)
5. [Product Library Schema](#5-product-library-schema)
6. [Chain of Custody](#6-chain-of-custody)
7. [Validation Rules](#7-validation-rules)
8. [Versioning](#8-versioning)
9. [MIME Type](#9-mime-type)
10. [Examples](#10-examples)
11. [Conformance](#11-conformance)
12. [Appendices](#12-appendices)

---

## 1. Introduction

### 1.1 Purpose

The CTO file format enables interoperability between Configure-to-Order building configurators, manufacturing execution systems, and project delivery platforms. It provides a machine-readable representation of a building design that:

- References products from a known catalog rather than defining arbitrary geometry
- Encodes spatial relationships, connection topology, and interface standard conformance
- Carries pricing, scheduling, and constraint metadata
- Tracks the complete chain of custody from fabrication to acceptance
- Documents the legal transition from goods (UCC) to real property (Common Law)
- Can be validated without human review

### 1.2 Scope

This specification covers:

- Single-family residential configurations (v0.1+)
- Multi-family residential configurations (v0.1.1+)
- Commercial configurations (planned for v0.3.0)

### 1.3 Terminology

| Term | Definition |
|------|------------|
| **Configuration** | A complete building design expressed as a composition of catalog products |
| **Catalog** | A collection of products from a single manufacturer conforming to the Product Library Schema |
| **Product** | A discrete, manufacturable building component (pod, panel, cartridge) with known properties |
| **Instance** | A specific unit of a product, with serial number and lifecycle tracking |
| **Template** | A pre-validated configuration used as a starting point for user modification |
| **Constraint** | A rule governing valid product placement or connection |
| **Grid** | The structural coordinate system derived from floor cartridge placement |
| **Interface Standard** | A published specification defining connection geometry and performance, documented as a CIS file. Examples: CfOC-ICC-1220 (module-to-building-services utility handshake), CfOC-ICC-1230 (panel-to-panel) |
| **CIS File** | A Configure-to-Order Interface Standard file (`.cis`) — a machine-readable definition of a connection standard. The CIS file format is documented in the companion specification (`/spec/cis/specification.md`). CTO element files reference CIS files by ID and version via URL rather than embedding engineering data |
| **Connection Plane** | The abstract surface at which two products meet, governed by a CIS file. The CIS file describes what happens at the plane; the CTO element file declares which face of the product addresses which side of the plane |
| **Fulfillment Plan** | The complete manufacturing, delivery, and installation schedule for a configuration |
| **Chain of Custody** | The documented sequence of handoffs tracking ownership, risk, and legal status across all parties from manufacturer to owner |
| **Legal Mateline** | The boundary at which a product transitions from goods (UCC) to real property (Common Law) |
| **Acceptance** | The event at which the buyer formally accepts an installed product, activating warranties and completing the goods-to-property transition |
| **Floor Cartridge** | A horizontal structural floor assembly. The term "cartridge" is reserved exclusively for interior horizontal assemblies that may carry finish options. Floor cartridges are the structural baseplates on which pods are placed |
| **Wall Panel** | An exterior vertical assembly (solid, window, door, or mixed). The term "panel" denotes exterior products — wall panels and roof panels |
| **Nominal Dimensions** | The grid footprint dimensions used by the configurator for snapping, placement, and spatial reservation. Nominal dimensions include chase zone allowances and installation tolerances |
| **Actual Dimensions** | The physical shipping envelope of a product as manufactured. Represented by the `bounding_box` in the product geometry block. Actual dimensions are smaller than nominal dimensions when chase zones are present |
| **Chase Zone** | A protected volume on the exterior face of a pod where MEP services (plumbing, electrical, HVAC) are mounted for factory inspection access. When two pods are placed adjacently, their opposing chase zones form a service chase without additional framing |
| **Sided Interface** | A connection interface that distinguishes between two named faces of a CIS plane — for example, the dwelling-unit side and the building-services side of CfOC-ICC-1220. Products declare which side of an interface standard they address, and on which geometric face. Sides belong to the CIS file, not the product. See §5.7 |
| **Component Display ID** | A human-readable unique identifier assigned to each placed component instance (e.g., `FP-001`, `KP-001`). Used to cross-reference BOM line items with tagged export drawings |
| **SWB Origin** | South-West-Bottom origin. The mandatory registration point for all product geometry files. The south-west corner of the product base is placed at world coordinates `[0, 0, 0]`. Product depth grows along the negative Z-axis. Height grows along the positive Y-axis (GLB/web convention) |
| **Product-Local Face Labels** | The labels `front`, `back`, `left`, `right`, `top`, `bottom` used in product geometry refer to faces of the product as oriented at SWB origin with no rotation. They are NOT real-world compass directions. A product placed in an assembly may be rotated such that its `front` face points west; the label `front` is preserved as a property of the product, not the placement |
| **Connection Plane** (also see above) | Sides of a connection plane belong to the CIS file. A product's face *addresses* a side by reaching toward the plane from that side. See §5.7 and the CIS spec §2.2 |

### 1.4 Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### 1.5 Related Standards

This specification references or is designed to interoperate with:

| Standard | Organization | Relevance |
|----------|--------------|-----------|
| CIS File Format | CfOC | Companion specification governing connection interface files referenced by CTO element files via `declares_interface` |
| CfOC-ICC-1220 | CfOC/ICC | Open-standard CIS for module-to-building-services core utility handshake |
| CfOC-ICC-1220-S | CfOC/ICC | Open-standard CIS for module-to-building-services structural handshake (separate from utility) |
| CfOC-ICC-1230 | CfOC/ICC | Open-standard CIS for panel-to-panel connectivity |
| ICC/MBI-1200 | ICC/MBI | Off-site construction requirements |
| ICC/MBI-1205 | ICC/MBI | Off-site inspection and compliance |
| UCC Article 2 | ULC | Sale of goods |
| IFC 4.3 | buildingSMART | Building information model exchange |

### 1.6 Companion Specifications

As of v0.2.0, the `configurator-file-type-spec` repository contains two related specifications:

**The CTO File Format Specification (this document, `/spec/cto/specification.md`)** governs `.cto` files: complete building configurations, product definitions (element files), templates, and entourage layouts. CTO files are *what gets built* — both the products themselves and the assemblies of products.

**The CIS File Format Specification (`/spec/cis/specification.md`)** governs `.cis` files: connection interface standards. CIS files are *the language of how products connect* — their port geometry, gendering, utility requirements, structural handshake, and version compatibility.

The two specifications are designed to coexist. A CTO element file references one or more CIS files via the `declares_interface` block (§4.2a). The CIS file is the source of truth for connection-plane specifics; the CTO file is the source of truth for product-specific geometry, lifecycle, and the rotation-lock binding between product faces and CIS sides.

This separation enables:
- Standards bodies to publish and version connection standards independently of product catalogs
- Manufacturers to author element files that remain compact and focused on product-specific data
- Configurators to perform validation by referencing well-defined CIS files rather than guessing at intent from product schemas
- Open-standard interfaces (published by CfOC) and proprietary catalog-internal interfaces (owned by manufacturers) to coexist using the same machinery

The CIS specification is normative for any CIS file referenced by a CTO file; conformant CTO parsers MUST be capable of parsing referenced CIS files.

---

## 2. Design Principles

### 2.1 Catalog-Constrained Design

A CTO file does not describe arbitrary geometry. Every element in a configuration MUST reference a `product_id` that exists in a known catalog. The catalog — not the designer — defines what is possible. This is the foundational principle of Configure-to-Order: **design as purchasing, not drafting**.

### 2.2 Cartridge-Driven Grids

The structural grid is derived from floor cartridge placement, not defined independently. Valid grid configurations are those for which a complete floor cartridge assembly exists. The grid is an output of product placement, not an input to design.

### 2.3 Pre-Validated Connections

Products declare their connection points and the interface standards to which they conform. A valid configuration is one in which all connections are satisfied according to product constraints and interface compatibility. Invalid configurations MUST NOT be savable as `.cto` files.

### 2.4 Price Transparency

Every product carries an MSRP. A configuration's total price is deterministically calculable from its composition. Options and upgrades add to the base price in a predictable manner.

### 2.5 Self-Describing

A CTO file contains all metadata necessary for a conforming parser to render, validate, and price the configuration without external dependencies (beyond the referenced catalog and any referenced CIS files).

### 2.6 End-to-End Traceability

A CTO file encodes not only what will be built, but how, when, and by whom. The chain of custody captures the complete lifecycle from factory release to acceptance, enabling coordination between manufacturers, logistics providers, installers, and owners.

### 2.7 Legal Clarity

A CTO file explicitly documents the legal status of each product instance — whether it is currently a good (governed by UCC Article 2), a fixture, or real property. The acceptance event marks the legal mateline crossing, activating warranties and completing the title transfer.

### 2.8 Specifications Speak to Humans

A `.cto` specification is read by three audiences: software engineers building parsers and configurators, standards engineers writing schemas and CIS files, and decision-makers (manufacturers, AHJs, building officials) evaluating the format for adoption. Schema definitions alone serve only the first audience.

Every section, block, and field benefits from prose that explains its purpose, scope, and explicit exclusions in plain language. Where a concept has counterintuitive scope — for example, `clearance_zones` describes *external* clearance only, not internal product operations — the spec MUST state this explicitly. When in doubt, write more prose, not less.

This principle is a hard requirement for new sections and a strong preference for revisions to existing sections. Future contributors are asked to draft prose-first and JSON-second.

---

## 3. File Structure

### 3.1 File Extension

CTO files use the `.cto` extension.

### 3.2 Encoding

CTO files MUST be encoded as UTF-8 JSON without BOM.

### 3.3 Compression

CTO files MAY be compressed using gzip. Compressed files SHOULD use the `.cto.gz` extension.

### 3.4 Top-Level Structure

A CTO file is a JSON object with the following top-level keys. Which keys are present depends on the `cto_type` (see §4.1a):

```json
{
  "cto_version": "0.2.0",
  "cto_type": "assembly",
  "project_meta": { },
  "element_meta": { },
  "declares_interface": [ ],
  "catalog_reference": { },
  "structural_grid": { },
  "floor_levels": [ ],
  "level_datums": [ ],
  "floor_cartridges": [ ],
  "placed_elements": [ ],
  "entourage_elements": [ ],
  "roof_prism": { },
  "roof_elements": [ ],
  "stairwell_volumes": [ ],
  "conceptual_objects": [ ],
  "pricing_snapshot": { },
  "fulfillment_plan": { },
  "validation_state": { }
}
```

**By `cto_type`:**

- `assembly` files use `project_meta`, `catalog_reference`, the placement-related keys, and `pricing_snapshot` / `fulfillment_plan` / `validation_state`. They do NOT use `element_meta` or `declares_interface`.
- `element` files use `element_meta`, `declares_interface`, `catalog_reference`, and the geometry / connectivity / constraints sections of the Product Library Schema (§5). They do NOT use the placement-related keys (`placed_elements`, `floor_cartridges`, etc.) or `fulfillment_plan`.
- `template` files use the same keys as `assembly` files plus `is_template: true` in `project_meta`.
- `entourage` files use `entourage_meta` (§4.4c) and `sub_elements`.

---

## 4. Schema Definition

### 4.1 `cto_version` (REQUIRED)

```json
"cto_version": "0.2.0"
```

The version of the CTO specification this file conforms to. Parsers MUST reject files with a major version they do not support.

### 4.1a `cto_type` (REQUIRED)

```json
"cto_type": "assembly"
```

Declares the intended role of this `.cto` file, analogous to part vs. assembly files in parametric CAD tools.

| Value | Description |
|-------|-------------|
| `element` | A single product definition exposing its interfaces to parent assemblies. Element files include an `element_meta` block (§4.2) and a `declares_interface` block (§4.2a). They describe what a product *is* — its geometry, constraints, and which CIS standards it conforms to |
| `assembly` | A composition of multiple elements forming a complete or partial building configuration. This is the most common type for user-created projects. Assembly files include a `project_meta` block (§4.3) |
| `template` | A pre-validated assembly used as a starting point for user modification. Templates have `is_template: true` in `project_meta` |
| `entourage` | A non-structural visual element (furniture layout, equipment placeholder) that renders in 2D/3D but does not contribute to pricing, scheduling, or chain of custody. Entourage files include an `entourage_meta` block (§4.4c) |

Parsers MUST reject files with an unrecognized `cto_type` value.

### 4.2 `element_meta` (REQUIRED for element files)

The `element_meta` block carries metadata for `cto_type: "element"` files. It is analogous to but distinct from `project_meta` (which describes user-created assemblies). An element file is a *product definition* authored by the manufacturer; it has different metadata needs than a project file authored by a user.

```json
"element_meta": {
  "id": "lbs-pod-k01-um",
  "name": "Kitchen Pod \u2014 K01-UM",
  "category": "pod",
  "subcategory": "kitchen",
  "design_line": "urban_minimalist",
  "product_version": "0.1.0",
  "status": "active",
  "created_at": "2026-04-19T14:00:00Z",
  "modified_at": "2026-04-19T14:30:00Z",
  "manufacturer": {
    "name": "Logic Building Systems",
    "address": "456 Industrial Park Dr, Brattleboro, VT 05301",
    "website": "https://buildwithlogic.com",
    "contact": {
      "name": "Jason Van Nest",
      "email": "jason@buildwithlogic.com",
      "role": "founder"
    }
  },
  "description": "Fully finished kitchen module with appliances, cabinetry, and countertops. Open-plan kitchen with single MEP chase on back face. Connects to Logic utility pods on left or right face via LBS-INT-UTIL-KIT proprietary interface.",
  "notes": "First production version; version 1.0.0 will follow once LBS engineering data (loads, lift points, certifications) is finalized."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique product identifier matching `product_id` references in assembly files |
| `name` | string | Yes | Human-readable product name |
| `category` | enum | Yes | Product category: `pod`, `wall_panel`, `roof_panel`, `floor_cartridge`, `coupler`, `structural_support` |
| `subcategory` | string | No | Subcategory per §5.3 |
| `design_line` | enum | No | One of: `urban_minimalist`, `rustic_retreat`, `coastal_breeze` (LBS-specific; other manufacturers may define their own enum) |
| `product_version` | semver string | Yes | The product's own version, distinct from `cto_version` |
| `status` | enum | Yes | One of: `draft`, `active`, `deprecated`, `withdrawn` |
| `created_at` | ISO 8601 timestamp | Yes | When this product definition was first authored |
| `modified_at` | ISO 8601 timestamp | Yes | Last modification timestamp |
| `manufacturer` | object | Yes | Manufacturer party block per §4.3a |
| `description` | string | Yes | Prose description of the product, including counterintuitive scope notes per §2.8 |
| `notes` | string | No | Free-form notes |

### 4.2a `declares_interface` (REQUIRED for element files)

The `declares_interface` block is the single most important block in an element file. It is the formal statement of which CIS standards this product conforms to — answering the question "what connection planes does this product touch?"

This is a top-level array; each entry references one CIS file by identifier and version. A product may declare conformance to multiple CIS standards if it has multiple interface faces (for example, a kitchen pod that uses `CfOC-ICC-1220` on its back face for chase services AND `LBS-INT-UTIL-KIT` on its left/right faces for pod-to-pod mating).

The `declares_interface` block answers *which* standards are conformed to. The `connectivity.interface_roles` block (§5.7) answers *which face of the product addresses which side of each declared standard.* Both are required for full interface conformance; `declares_interface` alone is not sufficient.

```json
"declares_interface": [
  {
    "cis_id": "CfOC-ICC-1220",
    "cis_version": "0.2.0",
    "cis_registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v0.2.0.cis",
    "scope": "module_to_building_services",
    "registry_scope": "open_standard",
    "description": "This product conforms to the open CfOC-ICC-1220 module-to-building-services core utility interface. See connectivity.interface_roles for which face addresses this standard."
  },
  {
    "cis_id": "LBS-INT-UTIL-KIT",
    "cis_version": "0.1.0",
    "cis_registry_url": null,
    "scope": "kitchen_to_utility",
    "registry_scope": "catalog_internal",
    "description": "This product conforms to the Logic Building Systems proprietary kitchen-to-utility interface. Catalog-internal; only mates with other LBS products declaring the same standard."
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cis_id` | string | Yes | The CIS file identifier (e.g., `CfOC-ICC-1220`, `LBS-INT-UTIL-KIT`) |
| `cis_version` | semver string | Yes | The version of the CIS standard this product conforms to |
| `cis_registry_url` | URL string or null | Yes | URL to the canonical CIS file. Null for catalog-internal proprietary standards that are not publicly registered |
| `scope` | string | Yes | Brief scope identifier (e.g., `module_to_building_services`, `kitchen_to_utility`) |
| `registry_scope` | enum | Yes | One of: `open_standard`, `catalog_internal`. Per CIS spec §10 |
| `description` | string | Yes | Prose explanation of why this product declares this interface, per §2.8 |

#### Multi-Standard Elements

It is common for a product to declare conformance to multiple CIS standards. Examples:

- **Kitchen pod (K01-UM)**: declares both `CfOC-ICC-1220` (for back-face chase services to building) AND `LBS-INT-UTIL-KIT` (for left/right face mating to utility pod). When placed in an assembly, the configurator validates each interface independently against its respective CIS file
- **Logic Coupler**: declares `CfOC-ICC-1220` twice — once on its top face addressing the `building_services_side`, once on its bottom face addressing the `dwelling_unit_side`. (See §5.7 for how this is expressed in `interface_roles`)
- **Bath pod (B01-UM)**: declares `CfOC-ICC-1220` on its top face AND `LBS-INT-BATH-UTIL` on its bottom face

Multi-standard declarations are not a special case in the schema — they are simply multiple entries in the `declares_interface` array. The configurator's job is to check, for each declared interface, that a corresponding `interface_roles` entry exists declaring which face addresses which side of that CIS plane.

### 4.3 `project_meta` (REQUIRED for assembly and template files)

```json
"project_meta": {
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Smith Residence",
  "created_at": "2026-04-04T14:30:00Z",
  "modified_at": "2026-04-04T16:45:00Z",
  "template_id": "template-sf-standard-001",
  "design_line": "urban_minimalist",
  "is_template": false,
  "author": {
    "name": "Jane Smith",
    "email": "jane@example.com",
    "organization": "Smith Development LLC",
    "role": "owner"
  },
  "project_type": "single_family_residential",
  "climate_zone": "5A",
  "seismic_design_category": "B",
  "jurisdiction": {
    "state": "VT",
    "county": "Windham",
    "municipality": "Brattleboro",
    "ahj_name": "Brattleboro Building Department",
    "ahj_contact": "building@brattleboro.org"
  },
  "notes": "Initial design for client review"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | UUID string | Yes | Unique identifier for this configuration |
| `name` | string | Yes | Human-readable project name |
| `created_at` | ISO 8601 timestamp | Yes | Creation timestamp |
| `modified_at` | ISO 8601 timestamp | Yes | Last modification timestamp |
| `template_id` | string or null | No | ID of the template this configuration was derived from |
| `design_line` | enum | Yes | One of: `urban_minimalist`, `rustic_retreat`, `coastal_breeze` |
| `is_template` | boolean | Yes | If true, this file is a template, not a user project |
| `author` | object | No | Author information |
| `project_type` | enum | Yes | `single_family_residential`, `multi_family_residential`, `commercial` |
| `climate_zone` | string | No | IECC climate zone (1-8 with moisture indicator A/B/C) |
| `seismic_design_category` | string | No | SDC A through F |
| `jurisdiction` | object | No | Authority having jurisdiction details |
| `notes` | string | No | Free-form project notes |

### 4.3a Unified Party Schema (REFERENCE DEFINITION)

The following party block structure is used throughout this specification wherever a responsible party is declared — in `chain_of_custody`, `fulfillment_plan`, `element_meta.manufacturer`, and all chain-of-custody actor fields.

```json
{
  "name": "string \u2014 legal entity name",
  "address": "string \u2014 full mailing address",
  "website": "string (URL) \u2014 company website",
  "contact": {
    "name": "string",
    "email": "string",
    "phone": "string",
    "role": "string"
  },
  "license_number": "string \u2014 applicable trade or contractor license",
  "insurer": {
    "carrier_name": "string \u2014 insurance carrier legal name",
    "policy_number": "string \u2014 policy identifier",
    "coverage_type": "string \u2014 e.g., general_liability, builders_risk, cargo, workers_comp",
    "expiration_date": "ISO 8601 date string \u2014 e.g., 2027-06-30"
  }
}
```

The six chain-of-custody party types (used in `chain_of_custody` blocks) are unchanged from v0.1.6:

| Party Type | Role in Chain of Custody |
|---|---|
| `manufacturer` | Fabricates and releases the product from the factory |
| `logistics_company` | Transports the product from factory to jobsite or staging yard |
| `gc_accepting_shipping` | General contractor who receives and signs for delivery at the jobsite |
| `hoisting_company` | Provides crane and rigging services; may also accept delivery on smaller projects |
| `pod_installer` | Licensed trade contractor who installs volumetric pod units |
| `panel_installer` | Licensed trade contractor who installs structural panels |

### 4.4 `catalog_reference` (REQUIRED for assembly and element files)

```json
"catalog_reference": {
  "manufacturer": "Logic Building Systems",
  "manufacturer_address": "456 Industrial Park Dr, Brattleboro, VT 05301",
  "manufacturer_website": "https://buildwithlogic.com",
  "catalog_id": "lbs-2026-q2",
  "catalog_edition": "2026 Spring Edition",
  "catalog_version": "2.1.0",
  "catalog_url": "https://buildwithlogic.com/catalog/lbs-2026-q2.json",
  "product_url": "https://buildwithlogic.com/products/pod-k01-um",
  "catalog_checksum": "sha256:a1b2c3d4e5f6...",
  "interface_standards": [
    {
      "standard_id": "CfOC-ICC-1220",
      "version": "0.2.0",
      "scope": "module_to_building_services",
      "cis_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v0.2.0.cis"
    },
    {
      "standard_id": "CfOC-ICC-1230",
      "version": "1.0.0",
      "scope": "panel_to_panel"
    }
  ]
}
```

The `catalog_reference` block remains unchanged from v0.1.6 except that `interface_standards` entries now SHOULD include a `cis_url` pointing at the canonical CIS file when one is available.

### 4.4a `floor_levels`, 4.4b `conceptual_objects`, 4.4c `entourage_elements`, 4.4d `stairwell_volumes`, 4.4e `level_datums`

These sections are unchanged from v0.1.6. See the v0.1.6 specification for full schemas. Future revisions may consolidate `conceptual_objects` (legacy) and `stairwell_volumes` (current).

### 4.5 `floor_cartridges`, 4.6 `placed_elements`, 4.7 `roof_elements`, 4.8 `pricing_snapshot`, 4.9 `fulfillment_plan`, 4.10 `validation_state`

These sections are unchanged from v0.1.6. The `placed_elements[].instance_data.chain_of_custody` and `fulfillment_plan` sections continue to use the Unified Party Schema (§4.3a).

---

## 5. Product Library Schema

Product catalogs MUST conform to the following schema. This is the schema for individual products that are referenced by `product_id` in CTO files. As of v0.2.0, several substantive updates have been made to align with the CIS specification:

- **Removed:** `gender` field on `connection_points` (now lives in CIS files via `connection_signature`)
- **Added:** `cis_port_id_ref` field on `connection_points` to reference the corresponding port in the relevant CIS file
- **Updated:** `interface_roles` framing to clarify it is a rotation-lock validation rule (see §5.7)
- **Clarified:** `bounding_box.min/max` coordinate convention (§5.1)
- **Clarified:** `clearance_zones` describes external clearance only, not internal product operations (§5.1)

### 5.1 Complete Product Schema

The product schema is read by manufacturers writing element files, by parsers validating those files, and by AHJ inspectors reviewing them. Where a field has counterintuitive scope, the prose makes it explicit. Consult §2.8 for the design principle motivating this verbosity.

```json
{
  "id": "lbs-pod-k01-um",
  "category": "pod",
  "subcategory": "kitchen",
  "name": "Kitchen Pod \u2014 K01-UM",
  "description": "Fully finished kitchen module with appliances, cabinetry, and countertops",
  "design_line": "urban_minimalist",
  "product_version": "0.1.0",
  "status": "active",

  "geometry": {
    "nominal_dimensions": {
      "width_ft": 10.75,
      "length_ft": 9.0,
      "description": "Grid footprint used by the configurator for snapping and spatial reservation. Includes chase zone allowances. The configurator reserves this footprint on the floor plate."
    },
    "bounding_box": {
      "width_ft": 10.25,
      "length_ft": 8.75,
      "height_ft": 8.0,
      "weight_lbs": 8400,
      "min": { "x": 0, "y": 0, "z": 0 },
      "max": { "x": 10.25, "y": 8.0, "z": 8.75 },
      "description": "Physical shipping envelope as manufactured. Always smaller than or equal to nominal dimensions when chase zones are present. The min/max coordinates are expressed in product-local feet, with origin at the SWB corner. Width grows along positive X, height grows along positive Y, depth grows along positive Z (note: this is product-local positive Z, distinct from the GLB coordinate system which uses negative Z for depth growth per the glTF Y-up convention; see glb_coordinate_system block below)"
    },
    "chase_zones": [
      {
        "side": "back",
        "depth_ft": 0.25,
        "purpose": "mep_handshake_clearance",
        "description": "Exterior-mounted MEP service volume. When two pods are placed with opposing chase zones touching, the combined gap forms a service chase without additional framing. The chase zone is sealed and inaccessible after installation \u2014 it is NOT a serviceable space"
      }
    ],
    "opening_geometry": null,
    "center_of_gravity": {
      "x_ft": 5.125,
      "y_ft": 6.0,
      "z_ft": 3.5
    },
    "clearance_zones": [
      {
        "purpose": "door_swing",
        "min": { "x": -3, "y": 5, "z": 0 },
        "max": { "x": 0, "y": 8, "z": 7 },
        "description": "Clearance zones describe EXTERNAL clearance requirements \u2014 space outside the product envelope that other catalog products MUST NOT occupy and that the configurator should reserve. They are NOT for documenting internal product operations such as appliance door swings within a kitchen pod, dishwasher pulls, or oven door openings; those happen within the product's own bounding volume and are not the configurator's concern. Use this field only when the product physically requires reserved space adjacent to its envelope"
      }
    ],
    "model_glb_url": "https://buildwithlogic.com/models/lbs-pod-k01-um.glb",
    "glb_coordinate_system": {
      "unit": "meters",
      "up_axis": "Y",
      "origin": "SWB \u2014 South-West-Bottom corner of product at world [0, 0, 0]",
      "depth_direction": "Negative Z-axis",
      "note": "GLB assets MUST be scaled to meters (1 unit = 1 meter per glTF 2.0 standard). Note that the GLB coordinate convention (Y-up, negative Z for depth) differs from the product-local feet coordinates used in the bounding_box.min/max block above"
    }
  },

  "connectivity": {
    "connection_points": [
      {
        "id": "cp-back-1220-cold-water",
        "face": "back",
        "position_ft": { "x": 0.125, "y": 7.5, "z": 8.75 },
        "cis_id_ref": "CfOC-ICC-1220",
        "cis_version_ref": "0.2.0",
        "cis_port_id_ref": "cold_water_supply",
        "description": "Cold water supply port on the back face, addressing the dwelling_unit_side of CfOC-ICC-1220. The connection signature, gender, and utility specification are defined in the CIS file (port id 'cold_water_supply') and are not duplicated here. Position is expressed in product-local feet"
      }
    ],
    "interface_roles": [
      {
        "cis_id": "CfOC-ICC-1220",
        "cis_version": "0.2.0",
        "cis_registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v0.2.0.cis",
        "face": "back",
        "side_addressed": "dwelling_unit_side",
        "description": "The back face of K01-UM addresses the dwelling-unit side of the CfOC-ICC-1220 standard. Per §5.7, this is a rotation-lock declaration: only the back face of K01-UM is permitted to mate with a 1220 plane. Other rotations are configurator-time warnings"
      }
    ],
    "mep_connections_legacy": null,
    "mep_connections_note": "In v0.1.6 and earlier, MEP connection details (electrical service entry, plumbing inlets, HVAC ducts) lived in this product schema. As of v0.2.0, MEP details are documented in the CIS files referenced via declares_interface (utility_connections array per CIS spec §7). The product file contains only the connection_points array referencing CIS port ids. Existing v0.1.6 product files MAY still include the legacy mep_connections block; v0.2.0 conformant parsers SHOULD ignore it in favor of the CIS-resolved data"
  },

  "structural_performance": {
    "load_capacity": {
      "dead_load_psf": 15,
      "live_load_psf": 40,
      "snow_load_psf": 30,
      "wind_load_psf": 25
    },
    "transmitted_loads_legacy_note": "In v0.1.6, transmitted_loads was a per-direction object on the product. As of v0.2.0, structural transmitted loads are described in the structural_connections array of the relevant CIS file (e.g., CfOC-ICC-1220-S). The product schema's transmitted_loads field is retained for backward compatibility but should be considered informational; CIS-defined structural capacities take precedence in validation",
    "lateral_system_participation": true,
    "deflection_limit": "L/360",
    "seismic_design_categories": ["A", "B", "C", "D0", "D1"],
    "foundation_requirements": {
      "point_loads": [
        { "position_ft": { "x": 0, "y": 0 }, "load_lbs": 4200 },
        { "position_ft": { "x": 8, "y": 0 }, "load_lbs": 4200 },
        { "position_ft": { "x": 0, "y": 12 }, "load_lbs": 4200 },
        { "position_ft": { "x": 8, "y": 12 }, "load_lbs": 4200 }
      ],
      "min_bearing_capacity_psf": 1500
    }
  },

  "thermal_performance": {
    "r_value_walls": 21,
    "r_value_ceiling": 38,
    "r_value_floor": 19,
    "u_factor_assembly": 0.048,
    "air_leakage_rate_cfm_per_sf": 0.25,
    "climate_zone_compatibility": ["4A", "4B", "4C", "5A", "5B", "5C", "6A", "6B", "7", "8"],
    "condensation_resistance_factor": 65,
    "shgc": null
  },

  "certifications": { /* unchanged from v0.1.6 */ },
  "constraints": { /* unchanged from v0.1.6 */ },
  "rigging": { /* unchanged from v0.1.6 */ },
  "installation": { /* unchanged from v0.1.6 */ },
  "warranty": { /* unchanged from v0.1.6 */ },
  "embedded_products": [ /* unchanged from v0.1.6 */ ],
  "options": [ /* unchanged from v0.1.6 */ ],
  "pricing": { /* unchanged from v0.1.6 */ },
  "documentation": { /* unchanged from v0.1.6 */ }
}
```

### 5.2 Category Values (unchanged from v0.1.6)

### 5.3 Subcategory Values (unchanged from v0.1.6, with one addition)

A new category, `coupler`, is added in v0.2.0 with subcategories:

| Category | Subcategory | Description |
|----------|-------------|-------------|
| `coupler` | `logic_coupler` | Logic Coupler product addressing both sides of CfOC-ICC-1220 |
| `coupler` | `panel_embedded_coupler` | A wall panel that includes an integrated coupler on one face |
| `coupler` | `site_built_stub` | Reference to a site-built utility stub (the product is a placeholder for what the GC builds) |

### 5.4 Opening Geometry (unchanged from v0.1.6)

### 5.5 Permitted Orientation (unchanged from v0.1.6)

### 5.6 Design Line Values (unchanged from v0.1.6, manufacturer-specific)

### 5.7 Interface Standard Conformance and Sided Interface Roles

Products MUST declare the interface standards to which they conform via the top-level `declares_interface` block (§4.2a). The `connectivity.interface_roles` array within the Product Library Schema is a complementary block that answers a different question:

**`declares_interface` answers:** Which CIS standards does this product conform to?
**`connectivity.interface_roles` answers:** Which face of this product addresses which side of each declared CIS standard?

Both are required for full interface conformance.

#### Why `interface_roles` Exists: The Rotation-Lock Validation Rule

The `interface_roles` block is the **primary mechanism preventing invalid product rotations** at the configurator. It is not merely informational — it is a product-specific geometric constraint declaring that *only* the listed face(s) of the product carry the CIS-conformant port cluster.

**Worked example.** Consider a bath pod that addresses the dwelling-unit side of CfOC-ICC-1220 on its top face. The CIS file defines what happens at the 1220 plane: a 6"×92.5" envelope with cold water at (1.5, 90), hot water at (3.0, 90), sanitary drain at (1.5, 2.0), and so forth. The bath pod's `connectivity.interface_roles` declares: "the top face of this pod addresses the `dwelling_unit_side` of `CfOC-ICC-1220`."

Without `interface_roles`, a configurator could blindly rotate the bath pod 90° and attempt to mate its side face to a Logic Coupler. This would be a manufacturing impossibility because the 1" PEX male coupling on the bath pod is only physically present on the top face. The `interface_roles` declaration is the rotation constraint that prevents this.

Per the CIS spec §15.3, violations of `interface_roles` produce **warnings**, not errors. A user designing in exploratory mode may rotate a product to see how it would look in different orientations; the warning informs them that the rotation breaks an interface. Manufacturing-ready export requires resolving all such warnings.

#### Schema

```json
"connectivity": {
  "interface_roles": [
    {
      "cis_id": "CfOC-ICC-1220",
      "cis_version": "0.2.0",
      "cis_registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v0.2.0.cis",
      "face": "top",
      "side_addressed": "dwelling_unit_side",
      "description": "The top face of this bath pod addresses the dwelling-unit side of CfOC-ICC-1220. Rotation to use any other face for this interface is invalid"
    }
  ],
  "connection_points": [
    {
      "id": "cp-top-1220-cold-water",
      "face": "top",
      "position_ft": { "x": 0.125, "y": 7.5, "z": 8.0 },
      "cis_id_ref": "CfOC-ICC-1220",
      "cis_version_ref": "0.2.0",
      "cis_port_id_ref": "cold_water_supply"
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cis_id` | string | Yes | Must match a `cis_id` in the file's `declares_interface` array |
| `cis_version` | semver string | Yes | The version of the CIS standard this face addresses |
| `cis_registry_url` | URL string or null | No | URL to the CIS file (null for catalog-internal) |
| `face` | enum | Yes | Product-local face label: `front`, `back`, `left`, `right`, `top`, `bottom`. Per §1.3, these are product-local labels at SWB origin, NOT real-world compass directions |
| `side_addressed` | string | Yes | Which side of the CIS plane this face addresses (e.g., `dwelling_unit_side`, `building_services_side`, `kitchen_pod_side`) |
| `description` | string | Yes | Prose explanation, per §2.8 |

#### Symmetric / Mirrored Interfaces

When a product can mate from multiple symmetric faces (e.g., a kitchen pod that connects to a utility pod on either its left OR right face), declare two separate `interface_roles` entries with the same `cis_id` and `cis_version` but different `face` values. The configurator selects which face is actually used based on assembly-time placement; both entries are valid declarations of capability.

```json
"interface_roles": [
  {
    "cis_id": "LBS-INT-UTIL-KIT",
    "cis_version": "0.1.0",
    "face": "left",
    "side_addressed": "kitchen_pod_side",
    "description": "Left face may address the kitchen-pod-side of LBS-INT-UTIL-KIT"
  },
  {
    "cis_id": "LBS-INT-UTIL-KIT",
    "cis_version": "0.1.0",
    "face": "right",
    "side_addressed": "kitchen_pod_side",
    "description": "Right face may address the kitchen-pod-side of LBS-INT-UTIL-KIT (mirror of left). The configurator picks one based on adjacent utility pod placement"
  }
]
```

#### Connection Validity

A connection between two products is valid if and only if:

1. Both products' `interface_roles` declare the same `cis_id`
2. Their declared `cis_version` values have at least one common entry in each other's CIS `compatible_with` arrays (negotiated per CIS spec §9.3)
3. Their `side_addressed` values are complementary (one addresses each side of the CIS plane)
4. The mated faces' connection signatures pair correctly per CIS spec §6.5
5. Where an `interface_role` declares a `face`, the configurator's placement of the product orients that face toward the connection plane

Failures of rules 1–4 are validation errors. Failures of rule 5 (rotation mismatch) are warnings per §5.7's rotation-lock framing.

---

## 6. Chain of Custody (unchanged from v0.1.6)

The chain of custody system tracks each product instance from factory release to acceptance, documenting the legal transition from goods (UCC Article 2) to real property (Common Law). All blocks remain unchanged from v0.1.6 and continue to use the Unified Party Schema (§4.3a).

---

## 7. Validation Rules (updates noted)

A conforming CTO parser MUST enforce the following validation rules. v0.2.0 adds rules related to CIS conformance.

### 7.1 Referential Integrity (updates)

- Every `product_id` MUST exist in the referenced catalog
- Every `connected_to` instance_id MUST exist
- The `template_id` (if present) MUST reference a valid template
- Every `instance_id` in `fulfillment_plan` MUST exist in the configuration
- Every `level_id` in `floor_cartridges` and `placed_elements` MUST reference an entry in `floor_levels`
- **NEW:** Every `cis_id` in `connectivity.interface_roles` MUST also appear in the file's top-level `declares_interface` block
- **NEW:** Every `cis_port_id_ref` in `connectivity.connection_points` MUST reference a `port_id` defined in the corresponding CIS file at the declared version

### 7.2 through 7.7 (unchanged from v0.1.6)

### 7.8 Interface Compatibility (rewritten)

For each connection between two products at a CIS plane, the validator MUST:

1. Resolve both products' `declares_interface` and `connectivity.interface_roles` entries for the relevant face pair
2. Confirm both products reference the same `cis_id`
3. Confirm version negotiation succeeds per CIS spec §9.3
4. Confirm `side_addressed` values are complementary
5. Confirm port-pair signature compatibility per CIS spec §6.5 (looking up the connection signature registry in CIS spec Appendix F)
6. Check the rotation-lock rule per §5.7 — if a product's actual placement orients a non-declared face toward the connection plane, generate a warning (not an error)

### 7.9 Chain of Custody Consistency (unchanged from v0.1.6)

---

## 8. Versioning (updates)

### 8.1 Semantic Versioning (unchanged)

The CTO specification follows Semantic Versioning 2.0.0:
- **MAJOR**: Incompatible schema changes
- **MINOR**: Backward-compatible additions
- **PATCH**: Clarifications and corrections

### 8.2 Compatibility (unchanged)

### 8.3 Migration (updates)

For migration from v0.1.6 to v0.2.0:

- Existing assembly files (`cto_type: "assembly"`) require no changes; v0.2.0 parsers accept v0.1.6 assembly files unchanged
- Existing element files (`cto_type: "element"`) SHOULD be migrated to add the new `element_meta` block (replacing `project_meta`) and `declares_interface` block
- Existing element files with `connectivity.connection_points[].gender` fields SHOULD migrate to the CIS-referenced model: remove `gender`, add `cis_id_ref` / `cis_version_ref` / `cis_port_id_ref`. The CIS file is the source of truth for gender (now expressed as `connection_signature` per CIS spec §6)
- Existing element files SHOULD remove the `connectivity.mep_connections` block; MEP details now live in the referenced CIS files
- The `cto_version` field MUST be updated from `"0.1.6"` to `"0.2.0"`

For files that do not migrate, v0.2.0 parsers SHOULD accept v0.1.6 element files in legacy mode, treating absent `declares_interface` blocks as a warning rather than an error.

### 8.4 Independent Versioning of Companion Specifications

The CTO spec and the CIS spec version independently. A v0.2.0 CTO file references CIS files at whatever version those CIS files declare; the CTO version does not constrain CIS versions. See CIS spec §9.2 for parallel guidance.

---

## 9. MIME Type (unchanged from v0.1.6, with version parameter updated to 0.2.0)

```
Type name: application
Subtype name: vnd.cfoc.cto+json
Required parameters: none
Optional parameters: version (e.g., "0.2.0")
Encoding considerations: UTF-8
Security considerations: Standard JSON security considerations apply
Published specification: https://centerforoffsiteconstruction.org/standards/cto-file-format
Contact: standards@centerforoffsiteconstruction.org
```

---

## 10. Examples

### 10.1 Minimal Valid Configuration (unchanged from v0.1.6)

### 10.2 Single-Family Starter Template (see Appendix A)

### 10.3 Multi-Standard Element File (NEW in v0.2.0)

A worked example of an element file declaring conformance to multiple CIS standards. This example uses the K01-UM kitchen pod, which connects to building services indirectly (through a utility pod) and to an adjacent utility pod via a proprietary interface.

```json
{
  "cto_version": "0.2.0",
  "cto_type": "element",
  "element_meta": {
    "id": "lbs-pod-k01-um",
    "name": "Kitchen Pod \u2014 K01-UM",
    "category": "pod",
    "subcategory": "kitchen",
    "design_line": "urban_minimalist",
    "product_version": "0.1.0",
    "status": "active",
    "manufacturer": { "name": "Logic Building Systems", "...": "..." }
  },
  "declares_interface": [
    {
      "cis_id": "LBS-INT-UTIL-KIT",
      "cis_version": "0.1.0",
      "cis_registry_url": null,
      "scope": "kitchen_to_utility",
      "registry_scope": "catalog_internal",
      "description": "Kitchen pod conforms to the LBS-internal kitchen-to-utility interface. Mates only with Logic utility pods"
    }
  ],
  "catalog_reference": { "...": "..." },
  "geometry": { "...": "..." },
  "connectivity": {
    "interface_roles": [
      {
        "cis_id": "LBS-INT-UTIL-KIT",
        "cis_version": "0.1.0",
        "face": "left",
        "side_addressed": "kitchen_pod_side",
        "description": "Left face addresses the kitchen-pod-side of LBS-INT-UTIL-KIT"
      },
      {
        "cis_id": "LBS-INT-UTIL-KIT",
        "cis_version": "0.1.0",
        "face": "right",
        "side_addressed": "kitchen_pod_side",
        "description": "Right face also addresses the kitchen-pod-side. K01-UM may mate to a utility pod on either left or right face; the configurator selects based on adjacent placement"
      }
    ],
    "connection_points": [ "..." ]
  },
  "structural_performance": { "...": "..." },
  "thermal_performance": { "...": "..." }
}
```

The K01-UM file declares one CIS standard but two `interface_roles` entries (one for each candidate face). At assembly time, the configurator picks whichever face has an adjacent utility pod.

A bath pod (B01-UM) by contrast would declare two CIS standards — `CfOC-ICC-1220` for its top face addressing building services, AND `LBS-INT-BATH-UTIL` for its bottom face addressing the utility pod beneath it — with one `interface_roles` entry per CIS.

---

## 11. Conformance (unchanged from v0.1.6)

CTO Reader / CTO Writer / CTO Configurator / CTO Fulfillment System levels remain as in v0.1.6. As of v0.2.0, all conformance levels MUST also be capable of resolving and parsing CIS files referenced via `declares_interface`.

---

## 12. Appendices

### Appendix A: Complete Single-Family Template Example

*[To be added: full JSON example of a Standard One-Story 48'×20' template]*

### Appendix B: JSON Schema (machine-readable)

*[To be added in v0.2.1: formal JSON Schema for automated validation]*

### Appendix C: Interface Standard Quick Reference

| Standard | Version | Scope | Key Parameters |
|----------|---------|-------|----------------|
| CfOC-ICC-1220 | 0.2.0 | Module-to-building-services core utility handshake | Cold water, optional hot water, sanitary, vent, electrical, optional data, dryer/bath exhaust |
| CfOC-ICC-1220-S | TBD | Module-to-building-services structural handshake | Bolts, alignment pins, load transfer (planned) |
| CfOC-ICC-1230 | 1.0.0 | Panel-to-panel | Structural fastening, thermal break, weather seal |
| LBS-INT-BATH-UTIL | 0.1.0 (planned) | Bath pod to utility pod (Logic-internal) | TBD |
| LBS-INT-UTIL-KIT | 0.1.0 (planned) | Utility pod to kitchen pod (Logic-internal) | TBD |

### Appendix D: Changelog

#### v0.2.0 (April 19, 2026) — CIS Companion Specification

**Major additions:**
- §1.6 — New section introducing the CIS companion specification
- §2.8 — New design principle: "Specifications Speak to Humans"
- §4.2 — New `element_meta` block for element files (distinct from `project_meta`)
- §4.2a — New top-level `declares_interface` block formalizing CIS conformance declarations
- §5.1 — Removed `gender` from `connection_points`; added `cis_port_id_ref`; clarified `bounding_box.min/max` coordinate convention; clarified that `clearance_zones` describes external clearance only
- §5.7 — Substantially rewritten to frame `interface_roles` as a rotation-lock validation rule, with worked example
- §10.3 — New worked example: multi-standard element file (K01-UM)
- §1.3 — Added terminology entries for `CIS File`, `Connection Plane`, and `Product-Local Face Labels`
- §1.5 — Added CIS File Format and CfOC-ICC-1220-S to Related Standards table

**In-scope spec observations addressed (numbered per session log):**
- #1 — `declares_interface` formalized (§4.2a)
- #2 — `bounding_box.min/max` coordinate clarification (§5.1)
- #3 — Face labels clarified as product-local (§1.3, §5.7)
- #4 — `clearance_zones` clarified as external-only (§5.1)
- #8 — `element_meta` block added (§4.2)
- #11 — Multi-standard element example added (§10.3)
- #15 — `interface_roles` rotation-lock framing (§5.7)
- #16 — `gender` removed from connection_points; CIS-referenced (§5.1, §5.7)

**Companion spec context:** v0.2.0 is published alongside CIS mini-spec v0.1.3. Together they form the v1 of the configurator-file-type-spec ecosystem.

**Deferred to v0.2.1+:**
- Coupler product class formalization (observation #7)
- Services chain validation (observation #12)
- Interface-carrier products documentation (observation #13)
- `field_built_element` parent abstraction
- Base/foundation and rigging views in Drafting Geometry
- Stairwell/conceptual_objects reconciliation
- Formal JSON Schema (Appendix B)

#### v0.1.6 (April 17, 2026)

(See `spec/CHANGELOG.md` for the full v0.1.6 changelog.)

### Appendix E: Acknowledgments

This specification was developed by the Center for Offsite Construction in collaboration with Logic Building Systems and the CfOC Standards Committee. The v0.2.0 release in particular was driven by the authoring of the first CIS files (CfOC-ICC-1220 v0.2.0) and the first element file (K01-UM kitchen pod), which surfaced 22 spec observations during a single intensive session. Approximately one-third of those observations are addressed in v0.2.0; the remainder are tracked for future releases.

---

## Document Information

**Editors:**
- Jason Van Nest, Center for Offsite Construction
- Mathew Ford, Center for Offsite Construction

**Contributors:**
- Michael Nolan, CfOC BIM/VDC Research Fellow
- Steve DeWitt, CfOC Senior Research Fellow
- Sam Williams, CfOC Senior Research Fellow

**Feedback:**
Submit issues and pull requests to https://github.com/Jason-Van-Nest/configurator-file-type-spec

**Copyright:**
© 2026 Center for Offsite Construction. This specification is released under the Apache 2.0 License.
