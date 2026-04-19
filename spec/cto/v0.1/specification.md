# CTO File Format Specification

**Version:** 0.1.6
**Status:** Draft
**Published by:** Center for Offsite Construction (CfOC) at NYIT
**Specification URL:** https://centerforoffsiteconstruction.org/offsite-product-configurator-file-type/
**GitHub Repository:** https://github.com/Jason-Van-Nest/configurator-file-type-spec
**License:** Apache 2.0
**Last Updated:** April 17, 2026

---

## Abstract

The CTO (Configure-to-Order) file format is an open standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

This specification defines the JSON schema, validation rules, and interchange requirements for CTO files. Version 0.1.6 introduces three major extensions: (1) the Entourage product type for non-structural CAD block elements such as furniture layouts that aid scale comprehension without contributing to pricing or scheduling; (2) the Stairwell Bounding Volume, upgrading the v0.1.5 conceptual stairwell object to a first-class hosting element for prefab staircases, interior furring wall cartridges, and shortened floor cartridges; and (3) the Level Datum Architecture with Roof Prism, establishing levels as first-class abstract hosts independent of floor plates, with a geometric roof prism whose sloped faces host roof panels.

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
- Commercial configurations (planned for v0.2.0)

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
| **Interface Standard** | A CfOC-published specification defining connection geometry and performance (e.g., CfOC-ICC-1220, CfOC-ICC-1230) |
| **CIS File** | A Configure-to-Order Interface Standard file (`.cis`) — a machine-readable definition of a connection standard published by CfOC. CTO files reference CIS files by ID and version via URL rather than embedding engineering data |
| **Fulfillment Plan** | The complete manufacturing, delivery, and installation schedule for a configuration |
| **Chain of Custody** | The documented sequence of handoffs tracking ownership, risk, and legal status across all parties from manufacturer to owner |
| **Legal Mateline** | The boundary at which a product transitions from goods (UCC) to real property (Common Law) |
| **Acceptance** | The event at which the buyer formally accepts an installed product, activating warranties and completing the goods-to-property transition |
| **Floor Cartridge** | A horizontal structural floor assembly. The term "cartridge" is reserved exclusively for interior horizontal assemblies that may carry finish options. Floor cartridges are the structural baseplates on which pods are placed |
| **Wall Panel** | An exterior vertical assembly (solid, window, door, or mixed). The term "panel" denotes exterior products — wall panels and roof panels |
| **Nominal Dimensions** | The grid footprint dimensions used by the configurator for snapping, placement, and spatial reservation. Nominal dimensions include chase zone allowances and installation tolerances |
| **Actual Dimensions** | The physical shipping envelope of a product as manufactured. Represented by the `bounding_box` in the product geometry block. Actual dimensions are smaller than nominal dimensions when chase zones are present |
| **Chase Zone** | A protected volume on the exterior face of a pod where MEP services (plumbing, electrical, HVAC) are mounted for factory inspection access. When two pods are placed adjacently, their opposing chase zones form a service chase without additional framing |
| **Sided Interface** | A connection interface that distinguishes between two named faces — for example, the dwelling-unit side and the building-services side of a Logic Coupler. Products declare which side of an interface standard they address, and on which geometric face |
| **Component Display ID** | A human-readable unique identifier assigned to each placed component instance (e.g., `FP-001`, `KP-001`). Used to cross-reference BOM line items with tagged export drawings |
| **SWB Origin** | South-West-Bottom origin. The mandatory registration point for all product geometry files. The south-west corner of the product base is placed at world coordinates `[0, 0, 0]`. Product depth grows along the negative Z-axis. Height grows along the positive Y-axis (GLB/web convention) |

### 1.4 Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### 1.5 Related Standards

This specification references or is designed to interoperate with:

| Standard | Organization | Relevance |
|----------|--------------|-----------|
| CfOC-ICC-1220 | CfOC/ICC | Module-to-module connectivity |
| CfOC-ICC-1230 | CfOC/ICC | Panel-to-panel connectivity |
| ICC/MBI-1200 | ICC/MBI | Off-site construction requirements |
| ICC/MBI-1205 | ICC/MBI | Off-site inspection and compliance |
| UCC Article 2 | ULC | Sale of goods |
| IFC 4.3 | buildingSMART | Building information model exchange |

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

A CTO file contains all metadata necessary for a conforming parser to render, validate, and price the configuration without external dependencies (beyond the referenced catalog).

### 2.6 End-to-End Traceability

A CTO file encodes not only what will be built, but how, when, and by whom. The chain of custody captures the complete lifecycle from factory release to acceptance, enabling coordination between manufacturers, logistics providers, installers, and owners.

### 2.7 Legal Clarity

A CTO file explicitly documents the legal status of each product instance — whether it is currently a good (governed by UCC Article 2), a fixture, or real property. The acceptance event marks the legal mateline crossing, activating warranties and completing the title transfer.

---

## 3. File Structure

### 3.1 File Extension

CTO files use the `.cto` extension.

### 3.2 Encoding

CTO files MUST be encoded as UTF-8 JSON without BOM.

### 3.3 Compression

CTO files MAY be compressed using gzip. Compressed files SHOULD use the `.cto.gz` extension.

### 3.4 Top-Level Structure

A CTO file is a JSON object with the following top-level keys:

{
  "cto_version": "0.1.6",
  "project_meta": { },
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

---

## 4. Schema Definition

### 4.1 `cto_version` (REQUIRED)

```json
"cto_version": "0.1.5"
```
The version of the CTO specification this file conforms to. Parsers MUST reject files with a major version they do not support.

### 4.1a `cto_type` (REQUIRED)

```json
"cto_type": "assembly"
```

Declares the intended role of this `.cto` file, analogous to part vs. assembly files in parametric CAD tools.

| Value | Description |
|-------|-------------|
| `element` | A single product definition exposing its interfaces to parent assemblies. Element files include a `declares_interface` block (see Section 4.2a) |
| `assembly` | A composition of multiple elements forming a complete or partial building configuration. This is the most common type for user-created projects |
| `template` | A pre-validated assembly used as a starting point for user modification. Templates have `is_template: true` in `project_meta` |
| `entourage` | A non-structural visual element (furniture layout, equipment placeholder) that renders in 2D/3D but does not contribute to pricing, scheduling, or chain of custody. Entourage files include an `entourage_meta` block (see Section 4.4c). New in v0.1.6 |

Parsers MUST reject files with an unrecognized `cto_type` value.

### 4.2 `project_meta` (REQUIRED)

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

### 4.3 `catalog_reference` (REQUIRED)

```json
"catalog_reference": {
  "manufacturer": "Logic Building Systems",
  "manufacturer_address": "456 Industrial Park Dr, Brattleboro, VT 05301",
  "manufacturer_website": "https://buildwithlogic.com",
  "catalog_id": "lbs-2026-q2",
  "catalog_edition": "2026 Spring Edition",
  "catalog_version": "2.1.0",
  "catalog_url": "https://buildwithlogic.com/catalog/lbs-2026-q2.json",
  "product_url": "https://buildwithlogic.com/products/pod-001",
  "catalog_checksum": "sha256:a1b2c3d4e5f6...",
  "interface_standards": [
    {
      "standard_id": "CfOC-ICC-1220",
      "version": "1.0.0",
      "scope": "module_to_module"
    },
    {
      "standard_id": "CfOC-ICC-1230",
      "version": "1.0.0",
      "scope": "panel_to_panel"
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `manufacturer` | string | Yes | Manufacturer legal name |
| `manufacturer_address` | string | No | Manufacturer full mailing address |
| `manufacturer_website` | URL string | No | Manufacturer website |
| `catalog_id` | string | Yes | Unique catalog identifier |
| `catalog_edition` | string | No | Human-readable catalog edition name (e.g., "2026 Spring Edition") |
| `catalog_version` | semver string | Yes | Catalog version |
| `catalog_url` | URL string | No | Location to fetch the full catalog file |
| `product_url` | URL string | No | Direct URL to the specific product page for this element |
| `catalog_checksum` | string | No | Integrity hash of the catalog file |
| `interface_standards` | array | No | CfOC interface standards used by products in this catalog |

### 4.3a Unified Party Schema (REFERENCE DEFINITION)

The following party block structure is used throughout this specification wherever a responsible party is declared — in `chain_of_custody`, `fulfillment_plan`, and all chain-of-custody actor fields. All party blocks share this structure:

```json
{
  "name": "string — legal entity name",
  "address": "string — full mailing address",
  "website": "string (URL) — company website",
  "contact": {
    "name": "string",
    "email": "string",
    "phone": "string",
    "role": "string"
  },
  "license_number": "string — applicable trade or contractor license",
  "insurer": {
    "carrier_name": "string — insurance carrier legal name",
    "policy_number": "string — policy identifier",
    "coverage_type": "string — e.g., general_liability, builders_risk, cargo, workers_comp",
    "expiration_date": "ISO 8601 date string — e.g., 2027-06-30"
  }
}
```

The six chain-of-custody party types and their roles:

| Party Type | Role in Chain of Custody |
|---|---|
| `manufacturer` | Fabricates and releases the product from the factory |
| `logistics_company` | Transports the product from factory to jobsite or staging yard |
| `gc_accepting_shipping` | General contractor who receives and signs for delivery at the jobsite |
| `hoisting_company` | Provides crane and rigging services; may also accept delivery on smaller projects |
| `pod_installer` | Licensed trade contractor who installs volumetric pod units — MEP-intensive work requiring plumbing and electrical licenses |
| `panel_installer` | Licensed trade contractor who installs structural panels — different trade license, insurance requirements, and liability window from pod installer |

Note: The GC and hoisting company may be the same legal entity on smaller projects. The spec models them as discrete records because liability transfers differently depending on who is physically receiving and lifting. Pod and panel installers are kept separate because they represent different trade licenses, insurance requirements, and liability windows.

The structured `insurer` block replaces all informal insurance text fields used in earlier versions of this specification. Every party block SHOULD include a populated `insurer` object whenever insurance information is known.

### 4.4 `structural_grid` (REQUIRED)

```json
"structural_grid": {
  "derived_from": "floor_cartridges",
  "grid_lines_x_ft": [0, 12, 24, 32],
  "grid_lines_y_ft": [0, 14, 28, 40],
  "unit": "ft",
  "origin": {
    "description": "Southwest corner of building footprint",
    "latitude": 42.8509,
    "longitude": -72.5579,
    "elevation_ft": 320
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `derived_from` | enum | Yes | Must be `floor_cartridges` in v0.1.x |
| `grid_lines_x_ft` | array of numbers | Yes | X-axis grid line positions |
| `grid_lines_y_ft` | array of numbers | Yes | Y-axis grid line positions |
| `unit` | enum | Yes | Unit of measurement: `ft`, `m`, or `mm` |
| `origin` | object | No | Coordinate system origin metadata |

### 4.4a `floor_levels` (OPTIONAL — REQUIRED for multi-story configurations)

The `floor_levels` array defines the abstract vertical planes on which floor cartridges and placed elements are organized. It replaces the bare `story` integer for multi-story coordination. The `story` integer is retained in `floor_cartridges` and `placed_elements` for backward compatibility but SHOULD be supplemented with a `level_id` reference when `floor_levels` is present.

```json
"floor_levels": [
  {
    "level_id": "L1",
    "name": "Ground Floor",
    "z_origin_ft": 0.0,
    "ceiling_height_ft": 9.0,
    "is_foundation": true
  },
  {
    "level_id": "L2",
    "name": "Second Floor",
    "z_origin_ft": 9.5,
    "ceiling_height_ft": 9.0,
    "is_foundation": false
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `level_id` | string | Yes | Unique identifier for this level (e.g., `L1`, `L2`) |
| `name` | string | Yes | Human-readable level name |
| `z_origin_ft` | decimal | Yes | Vertical offset from ground in feet. Ground floor is always `0.0` |
| `ceiling_height_ft` | decimal | Yes | Top-plate height for this level. Used to position wall panels and calculate the z_origin of the level above |
| `is_foundation` | boolean | Yes | If true, this level hosts foundation-bearing elements |

### 4.4b `conceptual_objects` (OPTIONAL)

Conceptual objects are non-physical planning volumes that impose constraints on placed elements without being catalog products themselves. In v0.1.5, the only supported type is `stairwell_volume`.

```json
"conceptual_objects": [
  {
    "type": "stairwell_volume",
    "instance_id": "main-stair-01",
    "level_range": ["L1", "L2"],
    "nominal_dimensions": {
      "width_ft": 8.0,
      "length_ft": 10.0
    },
    "position": { "x": 16, "y": 0 },
    "behavior_rules": {
      "force_void_alignment": true,
      "magnetic_snap_edges": true
    }
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | enum | Yes | `stairwell_volume` in v0.1.5. `elevator_shaft` planned for v0.2.0 |
| `instance_id` | string | Yes | Unique identifier for this conceptual object |
| `level_range` | array of level_id strings | Yes | The floor levels this object spans |
| `nominal_dimensions` | object | Yes | Width and length footprint of the volume in feet |
| `position` | object | Yes | X/Y grid position of the SW corner of the volume |
| `behavior_rules.force_void_alignment` | boolean | Yes | If true, floor cartridges at these coordinates must carry `opening_geometry` of type `void` |
| `behavior_rules.magnetic_snap_edges` | boolean | Yes | If true, shorter floor cartridges snap to the stairwell perimeter edges |

### 4.4c `entourage_elements` (OPTIONAL)

Entourage elements are non-structural, non-priced visual elements placed in a configuration to aid scale comprehension and spatial planning. They render in both 2D (as architectural linework) and 3D (as furniture models) but do NOT contribute to the pricing snapshot, BOM, fulfillment plan, or delivery schedule. They are excluded from chain-of-custody tracking.

Entourage elements are defined as `.cto` files with `cto_type: "entourage"`. Each entourage file describes a pre-composed group of sub-elements (e.g., a "King Bedroom" entourage contains a king bed, two nightstands, and a dresser positioned relative to each other).

```json
"entourage_elements": [
  {
    "instance_id": "ent-001",
    "entourage_id": "king-bedroom-01",
    "name": "King Bedroom Layout",
    "entourage_source": {
      "cto_file_url": "https://buildwithlogic.com/entourage/king-bedroom-01.cto",
      "cto_version": "0.1.6",
      "checksum": "sha256:abc123..."
    },
    "placement": {
      "method": "grid_anchor",
      "offset_ft": { "x": 8, "y": 4, "z": 0 }
    },
    "rotation_deg": 0,
    "level_id": "L1",
    "sub_elements": [
      {
        "sub_id": "bed",
        "name": "King Bed",
        "relative_offset_ft": { "x": 0, "y": 0, "z": 0 },
        "dimensions_ft": { "width": 6.67, "length": 6.67, "height": 2.0 },
        "svg_url": "https://buildwithlogic.com/entourage/svg/king-bed.svg",
        "glb_url": "https://buildwithlogic.com/entourage/glb/king-bed.glb"
      },
      {
        "sub_id": "nightstand-left",
        "name": "Nightstand",
        "relative_offset_ft": { "x": -1.75, "y": 0.5, "z": 0 },
        "dimensions_ft": { "width": 1.5, "length": 1.5, "height": 2.0 },
        "svg_url": "https://buildwithlogic.com/entourage/svg/nightstand-18.svg",
        "glb_url": "https://buildwithlogic.com/entourage/glb/nightstand-18.glb"
      },
      {
        "sub_id": "nightstand-right",
        "name": "Nightstand",
        "relative_offset_ft": { "x": 7.17, "y": 0.5, "z": 0 },
        "dimensions_ft": { "width": 1.5, "length": 1.5, "height": 2.0 },
        "svg_url": "https://buildwithlogic.com/entourage/svg/nightstand-18.svg",
        "glb_url": "https://buildwithlogic.com/entourage/glb/nightstand-18.glb"
      }
    ]
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `instance_id` | UUID string | Yes | Unique identifier for this placed entourage group |
| `entourage_id` | string | Yes | Reference to the entourage definition (from `.cto` file or catalog) |
| `name` | string | Yes | Human-readable name for display in palette and tooltips |
| `entourage_source` | object | No | Reference to the `.cto` entourage file. If absent, the entourage definition is resolved from hardcoded data |
| `placement` | object | Yes | Positioning of the group origin, using the same placement methods as `placed_elements` |
| `rotation_deg` | number | Yes | Rotation of the entire group in degrees (0, 90, 180, 270) |
| `level_id` | string | No | Reference to the `floor_levels` entry this entourage is hosted on |
| `sub_elements` | array | Yes | The individual furniture pieces within this group, positioned relative to the group origin |

#### Sub-Element Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `sub_id` | string | Yes | Unique identifier within this group |
| `name` | string | Yes | Human-readable name |
| `relative_offset_ft` | object | Yes | Position relative to the group origin `{ x, y, z }` |
| `dimensions_ft` | object | Yes | Bounding box `{ width, length, height }` in feet |
| `svg_url` | URL string | Yes | Reference to the SVG file for 2D architectural linework rendering |
| `glb_url` | URL string | No | Reference to the GLB file for 3D furniture model rendering. If absent, a labeled bounding box is rendered |

#### Entourage CTO File Schema

When an entourage is defined as a `.cto` file, the file MUST include:

```json
{
  "cto_version": "0.1.6",
  "cto_type": "entourage",
  "entourage_meta": {
    "id": "king-bedroom-01",
    "name": "King Bedroom Layout",
    "category": "bedroom",
    "description": "King bed centered between two 18-inch nightstands",
    "overall_dimensions_ft": { "width": 10.17, "length": 7.17, "height": 2.0 },
    "contributes_to_msrp": false,
    "contributes_to_schedule": false,
    "contributes_to_bom": false
  },
  "sub_elements": [ ]
}
```

The `cto_type: "entourage"` value is new in v0.1.6. It joins the existing values `"element"`, `"assembly"`, and `"template"`.

#### 2D Rendering Rules for Entourage

Entourage elements render in 2D as thin solid medium-gray (`#9CA3AF`) linework using the referenced SVG files. The SVG files contain architectural-convention furniture symbols (bed outlines, table shapes, chair arcs). Entourage linework MUST be visually distinct from structural elements (walls, pods, cartridges) to prevent user confusion.

#### 3D Rendering Rules for Entourage

Entourage elements render in 3D using the referenced GLB files. If no GLB file is provided for a sub-element, a transparent bounding box with a text label is rendered. GLB files for entourage follow the same coordinate conventions as product GLB files (meters, Y-up, SWB origin).

#### Validation Rules for Entourage

- Entourage elements MUST be hosted to a `level_id` (they sit on a floor level datum)
- The entire footprint of an entourage group MUST be within floor cartridge coverage (partial overlap is invalid)
- Entourage elements do NOT participate in collision detection with structural elements (pods, walls, cartridges)
- Entourage elements DO participate in collision detection with other entourage elements on the same level
- Entourage elements MUST NOT appear in `pricing_snapshot.line_items`, `fulfillment_plan`, or `chain_of_custody`

### 4.4d `stairwell_volumes` (OPTIONAL — replaces `conceptual_objects` stairwell_volume)

In v0.1.6, the stairwell is upgraded from a conceptual planning volume to a first-class hosting element. A stairwell volume defines a vertical void through the floor plate that can host a prefab staircase product, interior furring wall cartridges along its perimeter, and shortened floor cartridges that dead-end against its edges.

The `conceptual_objects` array with `type: "stairwell_volume"` (introduced in v0.1.5) is deprecated but remains valid for backward compatibility. Parsers encountering both `stairwell_volumes` and a `conceptual_objects` stairwell at the same position SHOULD prefer `stairwell_volumes`.

```json
"stairwell_volumes": [
  {
    "instance_id": "stair-001",
    "component_display_id": "SW-001",
    "name": "Main Stairwell",
    "level_range": ["L1", "L2"],
    "position": { "x": 16, "y": 0 },
    "nominal_dimensions": {
      "width_ft": 8.0,
      "length_ft": 10.0,
      "description": "The stairwell occupies a rectangular void in the floor plate. Width is measured along the X-axis, length along the Y-axis."
    },
    "rotation_deg": 0,
    "hosted_staircase": {
      "product_id": "stair-prefab-001",
      "product_source": "https://buildwithlogic.com/products/stair-prefab-001.cto",
      "description": "Optional. If a prefab staircase product is placed within this volume, it is referenced here. The staircase product is a catalog item with its own pricing, scheduling, and chain of custody."
    },
    "hosted_furring_walls": [
      {
        "instance_id": "fur-001",
        "edge": "left",
        "wall_thickness_ft": 0.375,
        "description": "Interior furring wall cartridge along the left edge of the stairwell. Provides a finished surface against the stairwell void."
      },
      {
        "instance_id": "fur-002",
        "edge": "right",
        "wall_thickness_ft": 0.375
      },
      {
        "instance_id": "fur-003",
        "edge": "back",
        "wall_thickness_ft": 0.375
      }
    ],
    "behavior_rules": {
      "force_void_alignment": true,
      "magnetic_snap_edges": true,
      "allow_shortened_cartridges": true,
      "min_cartridge_stub_ft": 2.0,
      "description": "Floor cartridges adjacent to the stairwell may be shortened to dead-end against the stairwell perimeter. The minimum stub length prevents structural instability."
    }
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `instance_id` | UUID string | Yes | Unique identifier |
| `component_display_id` | string | No | Human-readable ID for BOM/drawing cross-reference |
| `name` | string | Yes | Human-readable name |
| `level_range` | array of level_id strings | Yes | The floor levels this stairwell spans. Must reference entries in `floor_levels` |
| `position` | object | Yes | X/Y grid position of the SW corner of the volume |
| `nominal_dimensions` | object | Yes | Width and length footprint in feet |
| `rotation_deg` | number | Yes | Rotation in degrees (0, 90, 180, 270) |
| `hosted_staircase` | object | No | Reference to a prefab staircase product placed within the volume |
| `hosted_furring_walls` | array | No | Interior furring wall cartridges along the stairwell perimeter edges |
| `behavior_rules` | object | Yes | Constraint rules governing adjacent floor cartridge behavior |

#### Hosted Furring Wall Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `instance_id` | string | Yes | Unique identifier for this furring wall instance |
| `edge` | enum | Yes | Which edge of the stairwell: `front`, `back`, `left`, `right` |
| `wall_thickness_ft` | decimal | Yes | Thickness of the furring wall cartridge |

#### Behavior Rules

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `force_void_alignment` | boolean | Yes | If true, floor cartridges at these coordinates must carry `opening_geometry` of type `void` |
| `magnetic_snap_edges` | boolean | Yes | If true, shorter floor cartridges snap to the stairwell perimeter edges |
| `allow_shortened_cartridges` | boolean | Yes | If true, floor cartridges adjacent to the stairwell may be trimmed to dead-end against the volume |
| `min_cartridge_stub_ft` | decimal | No | Minimum length of a shortened floor cartridge stub (prevents structural instability). Default: 2.0 ft |


markdown### 4.4e Level Datum Architecture (v0.1.6)

In v0.1.6, levels are elevated from an organizational convenience (`floor_levels`) to a first-class abstract hosting system. The `level_datums` array defines the vertical datum planes to which all elements are hosted. Levels are independent of floor plates — a level exists as an abstract plane whether or not floor cartridges have been placed on it.

#### Hierarchy
Level Datum
└── hosts Floor Cartridges
└── hosts Wall Panels (exterior, standing on this datum)
└── hosts Pods (sitting on floor cartridges at this datum)
└── hosts Entourage Elements
└── hosts Interior Walls (non-panelized partitions)
└── hosts Stairwell Volumes (spanning this datum to the next)

#### Level Datum Schema

```json
"level_datums": [
  {
    "datum_id": "GF",
    "name": "Ground Floor",
    "type": "floor",
    "z_origin_ft": 0.0,
    "ceiling_height_ft": 9.0,
    "is_ground": true,
    "above_grade": true,
    "auto_derived": false,
    "notes": "Ground floor datum. Z-origin is the top of the foundation/slab."
  },
  {
    "datum_id": "L2",
    "name": "Second Floor",
    "type": "floor",
    "z_origin_ft": 9.833,
    "ceiling_height_ft": 9.0,
    "is_ground": false,
    "above_grade": true,
    "auto_derived": true,
    "derived_from": {
      "datum_below": "GF",
      "formula": "GF.z_origin_ft + GF.ceiling_height_ft + cartridge_thickness_ft",
      "cartridge_thickness_ft": 0.833,
      "description": "Second level z_origin auto-derives from the ground floor ceiling height plus the floor cartridge structural thickness. User may override with a manual value; a red conflict warning appears if the override creates an impossible gap or overlap."
    },
    "notes": null
  },
  {
    "datum_id": "RF",
    "name": "Roof",
    "type": "roof",
    "z_origin_ft": 19.666,
    "is_ground": false,
    "above_grade": true,
    "auto_derived": true,
    "derived_from": {
      "datum_below": "L2",
      "formula": "L2.z_origin_ft + L2.ceiling_height_ft + cartridge_thickness_ft"
    },
    "roof_prism": "roof-prism-001",
    "notes": "Roof datum hosts the roof prism geometry."
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `datum_id` | string | Yes | Unique identifier for this datum (e.g., `GF`, `L2`, `RF`) |
| `name` | string | Yes | Human-readable name |
| `type` | enum | Yes | `floor` or `roof` |
| `z_origin_ft` | decimal | Yes | Vertical offset from project origin in feet |
| `ceiling_height_ft` | decimal | Conditional | Required for `type: "floor"`. Top-plate height for this level |
| `is_ground` | boolean | Yes | True for the ground floor datum |
| `above_grade` | boolean | Yes | True if this datum is above finished grade |
| `auto_derived` | boolean | Yes | If true, `z_origin_ft` was calculated from the datum below |
| `derived_from` | object | No | Derivation formula when `auto_derived: true` |
| `roof_prism` | string | No | Reference to a `roof_prism` instance_id. Only valid for `type: "roof"` |

#### Relationship to `floor_levels`

The `floor_levels` array (introduced in v0.1.5) is retained for backward compatibility. When both `floor_levels` and `level_datums` are present, `level_datums` takes precedence. Parsers SHOULD migrate `floor_levels` entries to `level_datums` entries when writing v0.1.6 files. The key difference: `level_datums` supports auto-derivation, roof datums, and the hosting hierarchy.

#### Conflict Warnings

When a user manually overrides an auto-derived `z_origin_ft`, the configurator MUST check for geometric conflicts:

- If the manual value creates a gap smaller than the floor cartridge thickness, a red warning is displayed
- If the manual value creates an overlap (datum below ceiling + cartridge extends above this datum), a red warning is displayed
- Conflicts are advisory (warnings, not errors) to allow intentional non-standard configurations

### Roof Prism (v0.1.6)

The roof prism defines the three-dimensional geometry of the roof as a prismatic solid hosted to a roof-type level datum. The prism is an inverted triangular extrusion: the hypotenuse face points downward (toward the building interior), and two sloped faces rise upward from the eaves to the ridge. Roof panels are hosted to the sloped faces.

```json
"roof_prism": {
  "instance_id": "roof-prism-001",
  "datum_id": "RF",
  "name": "Main Roof",
  "prism_geometry": {
    "ridge_height_ft": 4.0,
    "ridge_offset_x_ft": 24.0,
    "building_width_ft": 48.0,
    "building_length_ft": 20.0,
    "eave_overhang_ft": 1.5,
    "gable_overhang_ft": 0.0,
    "description": "The prism spans the full building length. The ridge runs parallel to the building length axis at ridge_offset_x_ft from the left edge. Two sloped faces descend from the ridge to the eave lines. Ridge height is measured from the roof datum z_origin."
  },
  "slope_left": {
    "rise_ft": 4.0,
    "run_ft": 24.0,
    "pitch": "2/12",
    "hosted_panels": ["roof-001", "roof-002", "roof-003"],
    "description": "Left slope descends from ridge to left eave. Panels hosted to this face must have compatible pitch."
  },
  "slope_right": {
    "rise_ft": 4.0,
    "run_ft": 24.0,
    "pitch": "2/12",
    "hosted_panels": ["roof-004", "roof-005", "roof-006"]
  },
  "ridge": {
    "position_ft": { "x": 24.0, "z": 4.0 },
    "length_ft": 20.0,
    "cap_product_id": null,
    "description": "Ridge line where the two sloped faces meet. Optional ridge cap product."
  },
  "gable_ends": {
    "handling": "wall_panels",
    "description": "Gable end triangles are handled by wall panels, not by the roof system. Wall panels at gable end positions are trimmed or custom-shaped to fill the triangular area above the top plate."
  },
  "eave_extensions": {
    "panels_may_overhang": true,
    "max_overhang_ft": 2.0,
    "description": "Roof panels hosted to sloped faces may extend beyond the eave line to create overhangs. The overhang is measured horizontally from the exterior wall face."
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `instance_id` | string | Yes | Unique identifier for this roof prism |
| `datum_id` | string | Yes | Reference to the roof-type level datum |
| `name` | string | Yes | Human-readable name |
| `prism_geometry` | object | Yes | Overall prism dimensions |
| `slope_left` | object | Yes | Left slope face definition with pitch and hosted panel references |
| `slope_right` | object | Yes | Right slope face definition |
| `ridge` | object | Yes | Ridge line geometry |
| `gable_ends` | object | Yes | How gable end triangles are handled |
| `eave_extensions` | object | No | Rules for panel overhang beyond the eave line |

#### Roof Panel Hosting

Roof panels referenced in `slope_left.hosted_panels` and `slope_right.hosted_panels` MUST:

1. Be present in the `roof_elements` array
2. Have `permitted_orientation: "sloped"` in their product constraints
3. Have a declared `fixed_pitch` or `pitch_range` compatible with the slope's pitch
4. Not extend beyond the ridge line or past the maximum overhang distance from the eave

#### Relationship to `roof_elements`

The `roof_elements` array (Section 4.7) continues to hold the individual roof panel instances. The roof prism provides the geometric context that determines where and at what angle those panels are placed. Think of the prism as the "datum surface" and the panels as the "hosted elements."

#### Gable Ends

Gable end triangles — the triangular wall area above the top plate at each end of the building — are NOT part of the roof prism geometry. They are handled by wall panels. This keeps the roof system and wall system independent: wall panels below the roof host to their own level datum, and gable end panels are simply wall panels shaped or positioned to fill the triangular area.

### 4.5 `floor_cartridges` (REQUIRED)

```json
"floor_cartridges": [
  {
    "instance_id": "fc-instance-001",
    "product_id": "fc-001",
    "story": 1,
    "level_id": "L1",
    "grid_origin": {
      "x_index": 0,
      "y_index": 0
    },
    "span_x": 2,
    "span_y": 2,
    "rotation_deg": 0,
    "requires_mid_support": false,
    "implied_supports": [],
    "instance_data": {
      "serial_number": null,
      "manufacturing_status": "not_started"
    }
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `instance_id` | UUID string | Yes | Unique identifier for this placed instance |
| `product_id` | string | Yes | Reference to catalog product |
| `story` | integer | Yes | Floor level integer (1 = ground floor). Retained for backward compatibility |
| `level_id` | string | No | Reference to a `floor_levels` entry. SHOULD be present when `floor_levels` is defined |
| `grid_origin` | object | Yes | Grid cell indices for cartridge origin |
| `span_x` | integer | Yes | Number of grid cells spanned in X |
| `span_y` | integer | Yes | Number of grid cells spanned in Y |
| `rotation_deg` | number | Yes | Rotation in degrees (0, 90, 180, 270) |
| `requires_mid_support` | boolean | Yes | Whether this cartridge needs intermediate support |
| `implied_supports` | array | Yes | Structural supports generated by this cartridge |
| `instance_data` | object | No | Lifecycle data for this specific instance |

### 4.6 `placed_elements` (REQUIRED)

```json
"placed_elements": [
  {
    "instance_id": "elem-001",
    "component_display_id": "KP-001",
    "product_id": "pod-001",
    "category": "pod",
    "story": 1,
    "level_id": "L1",
    "placement": {
      "method": "grid_anchor",
      "grid_anchor": { "x_index": 0, "y_index": 0 },
      "offset_ft": { "x": 2, "y": 4, "z": 0 }
    },
    "rotation_deg": 0,
    "connections": [
      {
        "direction": "north",
        "connected_to": "elem-002",
        "connection_type": "logic_coupler",
        "interface_standard": "CfOC-ICC-1220",
        "interface_version": "1.0.0"
      }
    ],
    "options": {
      "finish_package": "premium",
      "sensor_suite": "standard"
    },
    "instance_data": {
      "serial_number": null,
      "manufacturing_status": "not_started",
      "chain_of_custody": []
    }
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `instance_id` | UUID string | Yes | Unique identifier for this placed instance |
| `component_display_id` | string | No | Human-readable ID for BOM cross-referencing (e.g., `KP-001`, `FP-003`). Used to tag elements in exported drawings |
| `product_id` | string | Yes | Reference to catalog product |
| `category` | enum | Yes | Product category: `pod`, `wall_panel`, `floor_cartridge`, `roof_panel` |
| `story` | integer | Yes | Floor level integer. Retained for backward compatibility |
| `level_id` | string | No | Reference to a `floor_levels` entry. SHOULD be present when `floor_levels` is defined |
| `placement` | object | Yes | Positioning information (see below) |
| `rotation_deg` | number | Yes | Rotation in degrees |
| `connections` | array | Yes | Connected elements with interface standard references |
| `options` | object | No | Product options/upgrades |
| `instance_data` | object | No | Lifecycle data for this specific instance |

#### Placement Methods

**Grid Anchor** (for pods and freestanding elements):
```json
"placement": {
  "method": "grid_anchor",
  "grid_anchor": { "x_index": 0, "y_index": 0 },
  "offset_ft": { "x": 2, "y": 4, "z": 0 }
}
```

**Edge Anchor** (for panels):
```json
"placement": {
  "method": "edge_anchor",
  "grid_anchor": { "x_index": 0, "y_index": 0 },
  "edge": "north",
  "position_along_edge_ft": 0
}
```

#### Connection Objects

```json
{
  "direction": "north",
  "connected_to": "elem-002",
  "connection_type": "logic_coupler",
  "interface_standard": "CfOC-ICC-1220",
  "interface_version": "1.0.0",
  "transmitted_loads": {
    "axial_lbs": 5000,
    "shear_lbs": 1200,
    "moment_ft_lbs": 800
  }
}
```

### 4.7 `roof_elements` (REQUIRED)

```json
"roof_elements": [
  {
    "instance_id": "roof-001",
    "component_display_id": "RP-001",
    "product_id": "rp-001",
    "placement": {
      "grid_anchor": { "x_index": 0, "y_index": 0 },
      "spans_to": { "x_index": 2, "y_index": 2 }
    },
    "rotation_deg": 0,
    "connections": [],
    "instance_data": {
      "serial_number": null,
      "manufacturing_status": "not_started"
    }
  }
]
```

### 4.8 `pricing_snapshot` (REQUIRED)

```json
"pricing_snapshot": {
  "calculated_at": "2026-04-04T16:45:00Z",
  "currency": "USD",
  "line_items": [
    {
      "instance_id": "elem-001",
      "component_display_id": "KP-001",
      "product_id": "pod-001",
      "product_name": "Kitchen Pod — Compact",
      "quantity": 1,
      "unit_msrp": 48000,
      "options": [
        {
          "option_id": "finish-premium",
          "option_name": "Premium Finish Package",
          "msrp_add": 2500
        }
      ],
      "options_subtotal": 2500,
      "line_total": 50500
    }
  ],
  "subtotal_products": 142000,
  "subtotal_options": 8500,
  "subtotal_delivery": 12000,
  "subtotal_installation": 18000,
  "subtotal_crane": 8500,
  "tax_rate": 0.06,
  "tax_amount": 11340,
  "total_msrp": 200340,
  "pricing_valid_until": "2026-06-30"
}
```

### 4.9 `fulfillment_plan` (OPTIONAL)

The fulfillment plan captures the complete production-to-installation timeline. It is OPTIONAL because configurations may be saved before scheduling is complete, but a configuration is not considered "order-ready" until the fulfillment plan is populated and validated.

See [Section 6: Chain of Custody](#6-chain-of-custody) for complete documentation of the fulfillment plan structure.

### 4.10 `validation_state` (REQUIRED)

```json
"validation_state": {
  "is_valid": true,
  "validated_at": "2026-04-04T16:45:00Z",
  "validator_version": "0.1.5",
  "errors": [],
  "warnings": [
    {
      "code": "W001",
      "severity": "warning",
      "message": "Window panel adjacent to utility pod may reduce natural light",
      "element_ids": ["elem-003", "elem-007"],
      "remediation": "Consider relocating window panel or adding skylight"
    }
  ],
  "interface_compatibility": {
    "all_connections_validated": true,
    "interface_standards_used": ["CfOC-ICC-1220", "CfOC-ICC-1230"],
    "incompatible_connections": []
  }
}
```

---

## 5. Product Library Schema

Product catalogs MUST conform to the following schema. This is the schema for individual products that are referenced by `product_id` in CTO files.

### 5.1 Complete Product Schema

```json
{
  "id": "pod-001",
  "category": "pod",
  "subcategory": "kitchen",
  "name": "Kitchen Pod — Compact",
  "description": "Fully finished kitchen module with appliances, cabinetry, and countertops",
  "design_line": "urban_minimalist",
  "product_version": "2.3.0",
  "status": "active",

  "geometry": {
    "nominal_dimensions": {
      "width_ft": 10.75,
      "length_ft": 9.0,
      "description": "Grid footprint used by the configurator for snapping and spatial reservation. Includes chase zone allowances. The configurator reserves this footprint on the floor plate."
    },
    "bounding_box": {
      "width_ft": 10.25,
      "length_ft": 8.5,
      "height_ft": 8.0,
      "weight_lbs": 8400,
      "min": { "x": 0, "y": 0, "z": 0 },
      "max": { "x": 10.25, "y": 8.5, "z": 8.0 },
      "description": "Physical shipping envelope as manufactured. Always smaller than or equal to nominal dimensions when chase zones are present."
    },
    "chase_zones": [
      {
        "side": "back",
        "depth_ft": 0.5,
        "purpose": "mep_handshake_clearance",
        "description": "Exterior-mounted MEP service volume. When two pods are placed with opposing chase zones touching, the combined gap forms a service chase without additional framing."
      }
    ],
    "opening_geometry": null,
    "center_of_gravity": {
      "x_ft": 5.125,
      "y_ft": 4.25,
      "z_ft": 4.0
    },
    "clearance_zones": [
      {
        "purpose": "door_swing",
        "min": { "x": -3, "y": 5, "z": 0 },
        "max": { "x": 0, "y": 8, "z": 7 }
      }
    ],
    "model_glb_url": "https://buildwithlogic.com/models/pod-001.glb",
    "glb_coordinate_system": {
      "unit": "meters",
      "up_axis": "Y",
      "origin": "SWB — South-West-Bottom corner of product at world [0, 0, 0]",
      "depth_direction": "Negative Z-axis",
      "note": "GLB assets MUST be scaled to meters (1 unit = 1 meter per glTF 2.0 standard)"
    }
  },

  "connectivity": {
    "connection_points": [
      {
        "id": "cp-north",
        "direction": "north",
        "position_ft": { "x": 4, "y": 12, "z": 4.5 },
        "interface_standard": "CfOC-ICC-1220",
        "interface_versions": ["1.0.0"],
        "connection_type": "logic_coupler",
        "gender": "male",
        "required": false
      },
      {
        "id": "cp-south",
        "direction": "south",
        "position_ft": { "x": 4, "y": 0, "z": 4.5 },
        "interface_standard": "CfOC-ICC-1220",
        "interface_versions": ["1.0.0"],
        "connection_type": "logic_coupler",
        "gender": "female",
        "required": true
      }
    ],
    "interface_roles": [
      {
        "cis_id": "CfOC-ICC-1220",
        "cis_version": "1.0.0",
        "cis_registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v1.0.0.cis",
        "face": "back",
        "side_addressed": "dwelling_unit_side",
        "description": "The back face of this pod addresses the dwelling-unit side of the CfOC-ICC-1220 standard. Products do not embed engineering data — they reference the CIS registry by ID and version."
      }
    ],
    "mep_connections": {
      "electrical": {
        "service_entry": { "x": 0, "y": 6, "z": 8 },
        "voltage": 240,
        "amperage": 50,
        "phases": 1,
        "circuits": 4
      },
      "plumbing": {
        "water_inlet": { "x": 0, "y": 3, "z": 1 },
        "water_outlet": { "x": 0, "y": 9, "z": 1 },
        "drain": { "x": 4, "y": 6, "z": 0 },
        "vent": { "x": 4, "y": 6, "z": 9 },
        "water_inlet_size_in": 0.75,
        "drain_size_in": 2
      },
      "hvac": {
        "supply_duct": { "x": 4, "y": 0, "z": 8.5 },
        "return_duct": { "x": 4, "y": 12, "z": 8.5 },
        "duct_size_in": 8,
        "cfm_required": 150
      }
    }
  },

  "structural_performance": {
    "load_capacity": {
      "dead_load_psf": 15,
      "live_load_psf": 40,
      "snow_load_psf": 30,
      "wind_load_psf": 25
    },
    "transmitted_loads": {
      "north": { "axial_lbs": 5000, "shear_lbs": 1200, "moment_ft_lbs": 800 },
      "south": { "axial_lbs": 5000, "shear_lbs": 1200, "moment_ft_lbs": 800 },
      "east": { "axial_lbs": 3000, "shear_lbs": 800, "moment_ft_lbs": 500 },
      "west": { "axial_lbs": 3000, "shear_lbs": 800, "moment_ft_lbs": 500 }
    },
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

  "certifications": {
    "listings": [
      {
        "agency": "ICC-ES",
        "report_number": "ESR-4567",
        "expiration": "2027-12-31",
        "scope": "Structural and fire resistance"
      },
      {
        "agency": "UL",
        "report_number": "UL-123456",
        "expiration": "2027-06-30",
        "scope": "Electrical safety"
      }
    ],
    "code_compliance": {
      "ibc_version": "2021",
      "irc_version": "2021",
      "iecc_version": "2021",
      "nfpa_versions": ["NFPA 70-2020", "NFPA 13-2022"]
    },
    "fire_rating": {
      "walls": "1-hour",
      "ceiling": "1-hour",
      "floor": "1-hour"
    },
    "smoke_development_index": 25,
    "flame_spread_index": 15,
    "accessibility": {
      "ada_compliant": true,
      "fha_compliant": true
    },
    "modular_program_approvals": [
      { "state": "VT", "approval_number": "VT-MOD-2026-001" },
      { "state": "MA", "approval_number": "MA-MOD-2026-234" }
    ]
  },

  "constraints": {
    "permitted_orientation": "horizontal",
    "pitch_range": null,
    "fixed_pitch": null,
    "min_adjacent_width_ft": 8,
    "max_stack_height": 1,
    "requires_floor_cartridge": true,
    "compatible_floor_cartridges": ["fc-001", "fc-002", "fc-003"],
    "compatible_roof_types": ["flat", "shed"],
    "excluded_adjacencies": [],
    "required_adjacencies": [],
    "max_cantilever_ft": 0,
    "tolerance_class": "standard"
  },

  "rigging": {
    "lift_points": [
      { "id": "lp-1", "position_ft": { "x": 0.5, "y": 8.0, "z": -0.5 }, "capacity_lbs": 5000 },
      { "id": "lp-2", "position_ft": { "x": 9.75, "y": 8.0, "z": -0.5 }, "capacity_lbs": 5000 },
      { "id": "lp-3", "position_ft": { "x": 0.5, "y": 8.0, "z": -8.0 }, "capacity_lbs": 5000 },
      { "id": "lp-4", "position_ft": { "x": 9.75, "y": 8.0, "z": -8.0 }, "capacity_lbs": 5000 }
    ],
    "spreader_bar_required": true,
    "min_spreader_length_ft": 10,
    "max_sling_angle_deg": 60,
    "certified_rigger_required": true
  },

  "installation": {
    "installation_manual_url": "https://buildwithlogic.com/manuals/pod-001-install.pdf",
    "installation_manual_version": "3.2.0",
    "staging_requirements": {
      "ground_prep": "Level to within 1/4 inch",
      "blocking_pattern": "4-point",
      "weather_protection": "Cover if precipitation expected within 24 hours"
    },
    "connection_sequence": [
      { "step": 1, "action": "Position on foundation", "duration_minutes": 30 },
      { "step": 2, "action": "Level and shim", "duration_minutes": 20 },
      { "step": 3, "action": "Bolt to foundation", "duration_minutes": 45 },
      { "step": 4, "action": "Connect structural interfaces", "duration_minutes": 30 },
      { "step": 5, "action": "Connect plumbing", "duration_minutes": 60 },
      { "step": 6, "action": "Connect electrical", "duration_minutes": 45 },
      { "step": 7, "action": "Connect HVAC", "duration_minutes": 30 }
    ],
    "commissioning_checklist": [
      { "item": "Plumbing pressure test", "acceptance_criteria": "Hold 60 PSI for 30 minutes" },
      { "item": "Electrical circuit test", "acceptance_criteria": "All circuits functional, GFCI trips" },
      { "item": "HVAC airflow test", "acceptance_criteria": "CFM within 10% of design" },
      { "item": "Appliance function test", "acceptance_criteria": "All appliances operational" },
      { "item": "Visual inspection", "acceptance_criteria": "No visible damage or defects" }
    ],
    "estimated_installation_hours": 4.5,
    "crew_requirements": {
      "crane_operator": 1,
      "riggers": 2,
      "mep_connectors": 2,
      "general_labor": 2
    }
  },

  "warranty": {
    "structural": { "duration_years": 10, "transferable": true },
    "finish": { "duration_years": 2, "transferable": true },
    "mep_systems": { "duration_years": 5, "transferable": true },
    "appliances": { "duration_years": 1, "transferable": false, "note": "Manufacturer warranty applies" },
    "warranty_document_url": "https://buildwithlogic.com/warranty/pod-001.pdf"
  },

  "embedded_products": [
    {
      "manufacturer": "GE Appliances",
      "model": "GTS18GTHWW",
      "description": "Top-freezer refrigerator",
      "warranty_years": 1,
      "serial_prefix": "GE-REF-"
    },
    {
      "manufacturer": "Bosch",
      "model": "SHPM88Z75N",
      "description": "Dishwasher",
      "warranty_years": 1,
      "serial_prefix": "BOSCH-DW-"
    }
  ],

  "options": [
    {
      "option_id": "finish-premium",
      "name": "Premium Finish Package",
      "description": "Upgraded countertops, cabinet hardware, and fixtures",
      "msrp_add": 2500,
      "affects_weight_lbs": 50,
      "compatible_with": ["sensor-standard", "sensor-advanced"]
    },
    {
      "option_id": "sensor-standard",
      "name": "Standard Sensor Suite",
      "description": "Water leak detection, smoke/CO detection",
      "msrp_add": 800,
      "compatible_with": ["finish-standard", "finish-premium"]
    }
  ],

  "pricing": {
    "base_msrp": 48000,
    "lead_time_days": 42,
    "pricing_effective_date": "2026-01-01",
    "pricing_expiration_date": "2026-12-31"
  },

  "documentation": {
    "thumbnail_url": "https://buildwithlogic.com/images/pod-001-thumb.jpg",
    "gallery_urls": [
      "https://buildwithlogic.com/images/pod-001-1.jpg",
      "https://buildwithlogic.com/images/pod-001-2.jpg"
    ],
    "model_glb_url": "https://buildwithlogic.com/models/pod-001.glb",
    "model_3d_url": "https://buildwithlogic.com/models/pod-001.gltf",
    "model_bim_url": "https://buildwithlogic.com/models/pod-001.ifc",
    "spec_sheet_url": "https://buildwithlogic.com/specs/pod-001.pdf",
    "cad_details_url": "https://buildwithlogic.com/details/pod-001.dwg"
  }
}
```

### 5.2 Category Values

| Category | Description |
|----------|-------------|
| `pod` | Fully finished volumetric module (kitchen, bath, utility) |
| `wall_panel` | Exterior envelope panel — vertical orientation required |
| `roof_panel` | Roof assembly — sloped orientation with declared pitch |
| `floor_cartridge` | Structural floor assembly — horizontal orientation required |
| `structural_support` | Columns, beams, connections |

### 5.3 Subcategory Values

| Category | Subcategory | Description |
|----------|-------------|-------------|
| `pod` | `kitchen` | Fully finished kitchen module |
| `pod` | `bath` | Fully finished bathroom module |
| `pod` | `utility` | Mechanical/electrical service module |
| `wall_panel` | `solid` | No openings |
| `wall_panel` | `window` | Contains window opening defined by `opening_geometry` |
| `wall_panel` | `door` | Contains door opening defined by `opening_geometry` |
| `wall_panel` | `mixed` | Contains multiple openings defined by `opening_geometry` |
| `floor_cartridge` | `standard` | Standard rectangular floor assembly |
| `floor_cartridge` | `stairwell` | Floor assembly with structural void for stair run |
| `roof_panel` | `solid` | Solid roof panel at declared pitch |

### 5.4 Opening Geometry

Wall panels with subcategory `window`, `door`, or `mixed` MUST include an `opening_geometry` block defining the subtractive void. This block is used by the configurator for 2D plan rendering, 3D visualization, and rough-opening scheduling.

```json
"opening_geometry": {
  "type": "door",
  "width_ft": 3.0,
  "height_ft": 7.0,
  "offset_x_ft": 2.5,
  "offset_z_ft": 0.0,
  "description": "offset_x_ft is the distance from the left edge of the panel to the left edge of the opening. offset_z_ft is the distance from the bottom of the panel to the bottom of the opening."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | enum | Yes | `window`, `door`, or `void` |
| `width_ft` | decimal | Yes | Opening width in feet |
| `height_ft` | decimal | Yes | Opening height in feet |
| `offset_x_ft` | decimal | Yes | Distance from left edge of panel to left edge of opening |
| `offset_z_ft` | decimal | Yes | Distance from bottom of panel to bottom of opening. For doors, this is typically `0.0` |

### 5.5 Permitted Orientation

All products MUST declare their `permitted_orientation` in the `constraints` block. This prevents illegal rotations at placement time.

| Value | Applies To | Description |
|-------|-----------|-------------|
| `horizontal` | Pods, floor cartridges | Product lies flat; Z-axis is height |
| `vertical` | Wall panels | Product stands upright |
| `sloped` | Roof panels | Product is inclined; requires `pitch_range` or `fixed_pitch` |

For roof panels with `permitted_orientation: "sloped"`, the `constraints` block MUST include either `fixed_pitch` (a single rise/run string, e.g., `"5/12"`) or `pitch_range` (an object with `min_pitch` and `max_pitch`).

```json
"constraints": {
  "permitted_orientation": "sloped",
  "fixed_pitch": "5/12",
  "pitch_range": null
}
```

### 5.6 Design Line Values

| Value | Description |
|-------|-------------|
| `urban_minimalist` | Clean lines, industrial materials, monochromatic |
| `rustic_retreat` | Natural materials, warm tones, textured surfaces |
| `coastal_breeze` | Light colors, open feel, weather-resistant |

### 5.7 Interface Standard Conformance and Sided Interface Roles

Products MUST declare the interface standards to which their connection points conform. In v0.1.5, products additionally declare `interface_roles` — mapping specific geometric faces to specific sides of a CIS (Configure-to-Order Interface Standard) file. This enables sided interface validation: the configurator checks not only that two products share a standard, but that the correct faces are mated.

```json
"connectivity": {
  "connection_points": [
    {
      "id": "cp-north",
      "direction": "north",
      "interface_standard": "CfOC-ICC-1220",
      "interface_versions": ["1.0.0", "1.1.0"],
      "connection_type": "logic_coupler",
      "gender": "male"
    }
  ],
  "interface_roles": [
    {
      "cis_id": "CfOC-ICC-1220",
      "cis_version": "1.0.0",
      "cis_registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v1.0.0.cis",
      "face": "back",
      "side_addressed": "dwelling_unit_side"
    }
  ]
}
```

Valid face values: `front`, `back`, `left`, `right`, `top`, `bottom`.

A connection is valid if and only if:
1. Both products declare the same `interface_standard`
2. The `interface_versions` arrays have at least one version in common
3. The `gender` values are complementary (male/female) or both are `neutral`
4. The `connection_type` values match
5. Where `interface_roles` are declared, the mated faces address complementary sides of the same CIS standard

---

## 6. Chain of Custody

The chain of custody system tracks each product instance from factory release to acceptance, documenting the legal transition from goods (UCC Article 2) to real property (Common Law).

### 6.1 Legal Framework

The chain of custody addresses the key questions identified in the CfOC whitepaper "From Handshake to Hardware":

| Question | Documentation Point |
|----------|---------------------|
| Who owns the product? | `title_holder` at each stage |
| Who bears the risk of loss? | `risk_owner` at each stage |
| Which insurance policy responds? | `insurer` block at each stage |
| What is the legal status? | `legal_status`: `goods`, `fixture`, `real_property` |
| When does warranty activate? | `acceptance` stage timestamp |

### 6.2 Custody Stages

| Stage | Legal Status | Description |
|-------|--------------|-------------|
| `factory_release` | goods | Product cleared from manufacturing floor |
| `pickup` | goods | Loaded onto transport vehicle |
| `in_transit` | goods | Moving between facilities |
| `delivery` | goods | Arrived at jobsite or staging yard |
| `staging` | goods | Placed in staging area awaiting installation |
| `rigging` | goods | Attached to crane/lifting apparatus |
| `hoisting` | goods | In the air, being positioned |
| `placement` | goods → fixture | Set down on foundation/structure |
| `connection` | fixture | Physically attached to adjacent elements |
| `commissioning` | fixture | Systems tested and verified |
| `acceptance` | fixture → real_property | **Legal mateline crossing** — buyer accepts, warranties activate |

### 6.3 Chain of Custody Schema

All `responsible_party` blocks in the chain of custody conform to the Unified Party Schema defined in Section 4.3a. The `insurer` block within each party replaces the informal `insurance` fields used in earlier drafts.

```json
"chain_of_custody": [
  {
    "stage": "factory_release",
    "timestamp": "2026-05-08T18:00:00Z",
    "responsible_party": {
      "name": "Logic Building Systems",
      "address": "456 Industrial Park Dr, Brattleboro, VT 05301",
      "website": "https://buildwithlogic.com",
      "contact": {
        "name": "Factory Operations",
        "email": "factory-ops@buildwithlogic.com",
        "phone": "+1-802-555-0100",
        "role": "manufacturer"
      },
      "license_number": "VT-MFG-2026-001",
      "insurer": {
        "carrier_name": "Hartford Fire Insurance",
        "policy_number": "HFD-PROD-123456",
        "coverage_type": "products_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "legal_status": "goods",
    "risk_owner": "manufacturer",
    "title_holder": "manufacturer"
  },
  {
    "stage": "delivery",
    "timestamp": "2026-06-18T08:00:00Z",
    "responsible_party": {
      "name": "Northeast Heavy Haul",
      "address": "789 Transport Way, Springfield, VT 05156",
      "website": "https://neheavyhaul.com",
      "contact": {
        "name": "Dispatch",
        "phone": "+1-802-555-0456",
        "role": "logistics_company"
      },
      "license_number": "MC-123456",
      "insurer": {
        "carrier_name": "Trucking Insurance Co",
        "policy_number": "TIC-456789",
        "coverage_type": "cargo",
        "expiration_date": "2027-01-31"
      }
    },
    "gc_accepting_shipping": {
      "name": "Smith Development LLC",
      "address": "123 Main Street, Brattleboro, VT 05301",
      "website": "https://smithdev.com",
      "contact": {
        "name": "Mike Torres",
        "phone": "+1-802-555-0123",
        "role": "site_superintendent"
      },
      "license_number": "VT-GC-12345",
      "insurer": {
        "carrier_name": "Builder's Risk Mutual",
        "policy_number": "BRM-789012",
        "coverage_type": "builders_risk",
        "expiration_date": "2027-03-31"
      }
    },
    "legal_status": "goods",
    "risk_owner": "buyer",
    "title_holder": "buyer",
    "shipping_terms": "FOB_destination"
  },
  {
    "stage": "hoisting",
    "timestamp": "2026-06-18T10:00:00Z",
    "responsible_party": {
      "name": "Green Mountain Crane Services",
      "address": "321 Crane Ave, Bellows Falls, VT 05101",
      "website": "https://gmcrane.com",
      "contact": {
        "name": "Crane Dispatch",
        "phone": "+1-802-555-0789",
        "role": "hoisting_company"
      },
      "license_number": "NCCCO-12345",
      "insurer": {
        "carrier_name": "Crane Operators Insurance Group",
        "policy_number": "COIG-321654",
        "coverage_type": "general_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "legal_status": "goods",
    "risk_owner": "buyer",
    "title_holder": "buyer"
  },
  {
    "stage": "acceptance",
    "timestamp": "2026-06-20T16:00:00Z",
    "responsible_party": {
      "name": "Smith Development LLC",
      "address": "123 Main Street, Brattleboro, VT 05301",
      "website": "https://smithdev.com",
      "contact": {
        "name": "Jane Smith",
        "email": "jane@smithdev.com",
        "phone": "+1-802-555-0100",
        "role": "owner"
      },
      "license_number": null,
      "insurer": {
        "carrier_name": "Vermont Property Insurance",
        "policy_number": "VPI-345678",
        "coverage_type": "property",
        "expiration_date": "2027-06-20"
      }
    },
    "pod_installer": {
      "name": "Green Mountain MEP Services",
      "address": "555 Plumber Lane, Brattleboro, VT 05301",
      "website": "https://gmmep.com",
      "contact": {
        "name": "Lead Installer",
        "phone": "+1-802-555-0200",
        "role": "pod_installer"
      },
      "license_number": "VT-PLB-7890",
      "insurer": {
        "carrier_name": "Contractor Liability Insurance",
        "policy_number": "CLI-111222",
        "coverage_type": "general_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "panel_installer": {
      "name": "Structural Solutions VT",
      "address": "444 Panel Way, Brattleboro, VT 05301",
      "website": "https://structvt.com",
      "contact": {
        "name": "Foreman",
        "phone": "+1-802-555-0300",
        "role": "panel_installer"
      },
      "license_number": "VT-STR-4567",
      "insurer": {
        "carrier_name": "Structural Contractors Mutual",
        "policy_number": "SCM-333444",
        "coverage_type": "general_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "legal_status": "real_property",
    "risk_owner": "owner",
    "title_holder": "owner",
    "legal_mateline_crossing": {
      "occurred": true,
      "timestamp": "2026-06-20T16:00:00Z",
      "certificate_of_occupancy": "CO-2026-0456",
      "ahj_approval": true
    },
    "warranty_activation": {
      "activated": true,
      "activation_date": "2026-06-20",
      "warranties": [
        { "type": "structural", "expiration_date": "2036-06-20" },
        { "type": "finish", "expiration_date": "2028-06-20" },
        { "type": "mep_systems", "expiration_date": "2031-06-20" }
      ]
    }
  }
]
```

### 6.4 Fulfillment Plan Structure

The complete fulfillment plan integrates chain of custody with manufacturing, delivery, and installation scheduling. All party blocks in the fulfillment plan conform to the Unified Party Schema defined in Section 4.3a.

```json
"fulfillment_plan": {
  "plan_id": "fp-550e8400",
  "plan_status": "confirmed",
  "created_at": "2026-04-04T16:45:00Z",
  "confirmed_at": "2026-04-05T10:00:00Z",

  "jobsite": {
    "address": {
      "street": "123 Main Street",
      "city": "Brattleboro",
      "state": "VT",
      "zip": "05301",
      "country": "USA"
    },
    "coordinates": {
      "latitude": 42.8509,
      "longitude": -72.5579
    },
    "site_contact": {
      "name": "Mike Torres",
      "phone": "+1-802-555-0123",
      "email": "mike@smithdev.com",
      "role": "site_superintendent"
    },
    "access_constraints": {
      "crane_access": true,
      "max_truck_length_ft": 53,
      "max_truck_weight_lbs": 80000,
      "delivery_hours": { "start": "07:00", "end": "17:00" },
      "requires_police_escort": false,
      "route_restrictions": "No left turn from Route 9",
      "permit_ids": ["BP-2026-0456"]
    },
    "ready_date": "2026-06-15"
  },

  "manufacturing_schedule": {
    "factory": {
      "facility_id": "lbs-brattleboro-01",
      "name": "Logic Building Systems — Brattleboro",
      "address": "456 Industrial Park Dr, Brattleboro, VT 05301",
      "third_party_inspector": {
        "agency": "ICC Evaluation Service",
        "inspector_name": "John Smith",
        "license_number": "ICC-INSP-12345"
      }
    },
    "production_slots": [
      {
        "instance_id": "elem-001",
        "component_display_id": "KP-001",
        "product_id": "pod-001",
        "product_name": "Kitchen Pod — Compact",
        "serial_number": "LBS-POD-2026-001234",
        "production_start": "2026-05-01T06:00:00Z",
        "production_end": "2026-05-08T18:00:00Z",
        "production_days": 6,
        "qc_status": "passed",
        "qc_date": "2026-05-08",
        "ship_ready_date": "2026-05-09",
        "chain_of_custody": []
      }
    ],
    "total_production_days": 12,
    "production_complete_date": "2026-05-12"
  },

  "delivery_manifest": {
    "shipments": [
      {
        "shipment_id": "ship-001",
        "logistics_company": {
          "name": "Northeast Heavy Haul",
          "address": "789 Transport Way, Springfield, VT 05156",
          "website": "https://neheavyhaul.com",
          "contact": {
            "name": "Dispatch",
            "phone": "+1-802-555-0456",
            "role": "logistics_company"
          },
          "license_number": "MC-123456",
          "insurer": {
            "carrier_name": "Trucking Insurance Co",
            "policy_number": "TIC-456789",
            "coverage_type": "cargo",
            "expiration_date": "2027-01-31"
          }
        },
        "load_manifest": [
          {
            "instance_id": "elem-001",
            "component_display_id": "KP-001",
            "serial_number": "LBS-POD-2026-001234",
            "product_name": "Kitchen Pod — Compact",
            "weight_lbs": 8400,
            "position_on_truck": 1,
            "blocking_required": true,
            "rigging_points": 4,
            "special_handling": "Keep upright; do not stack"
          }
        ],
        "total_weight_lbs": 8400,
        "pickup": {
          "facility_id": "lbs-brattleboro-01",
          "scheduled_date": "2026-06-18",
          "scheduled_time": "06:00",
          "dock_assignment": "Dock A"
        },
        "delivery": {
          "scheduled_date": "2026-06-18",
          "scheduled_time": "08:00",
          "estimated_transit_hours": 2,
          "staging_area": "North lot"
        },
        "status": "scheduled",
        "shipping_terms": "FOB_destination"
      }
    ]
  },

  "crane_lift_plan": {
    "hoisting_company": {
      "name": "Green Mountain Crane Services",
      "address": "321 Crane Ave, Bellows Falls, VT 05101",
      "website": "https://gmcrane.com",
      "contact": {
        "name": "Crane Dispatch",
        "phone": "+1-802-555-0789",
        "role": "hoisting_company"
      },
      "license_number": "NCCCO-12345",
      "insurer": {
        "carrier_name": "Crane Operators Insurance Group",
        "policy_number": "COIG-321654",
        "coverage_type": "general_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "crane": {
      "type": "mobile_hydraulic",
      "capacity_tons": 100,
      "boom_length_ft": 140
    },
    "lift_operations": [
      {
        "lift_id": "lift-001",
        "sequence_order": 1,
        "instance_id": "elem-001",
        "component_display_id": "KP-001",
        "weight_lbs": 8400,
        "scheduled_date": "2026-06-18",
        "scheduled_time": "10:00",
        "estimated_duration_minutes": 60,
        "status": "scheduled"
      }
    ],
    "total_lift_days": 1
  },

  "installation_schedule": {
    "gc_accepting_shipping": {
      "name": "Smith Development LLC",
      "address": "123 Main Street, Brattleboro, VT 05301",
      "website": "https://smithdev.com",
      "contact": {
        "name": "Jane Smith",
        "phone": "+1-802-555-0100",
        "role": "gc_accepting_shipping"
      },
      "license_number": "VT-GC-12345",
      "insurer": {
        "carrier_name": "Builder's Risk Mutual",
        "policy_number": "BRM-789012",
        "coverage_type": "builders_risk",
        "expiration_date": "2027-03-31"
      }
    },
    "pod_installer": {
      "name": "Green Mountain MEP Services",
      "address": "555 Plumber Lane, Brattleboro, VT 05301",
      "website": "https://gmmep.com",
      "contact": {
        "name": "Lead Installer",
        "phone": "+1-802-555-0200",
        "role": "pod_installer"
      },
      "license_number": "VT-PLB-7890",
      "insurer": {
        "carrier_name": "Contractor Liability Insurance",
        "policy_number": "CLI-111222",
        "coverage_type": "general_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "panel_installer": {
      "name": "Structural Solutions VT",
      "address": "444 Panel Way, Brattleboro, VT 05301",
      "website": "https://structvt.com",
      "contact": {
        "name": "Foreman",
        "phone": "+1-802-555-0300",
        "role": "panel_installer"
      },
      "license_number": "VT-STR-4567",
      "insurer": {
        "carrier_name": "Structural Contractors Mutual",
        "policy_number": "SCM-333444",
        "coverage_type": "general_liability",
        "expiration_date": "2027-06-30"
      }
    },
    "installation_start_date": "2026-06-18",
    "installation_end_date": "2026-06-19",
    "total_installation_days": 2
  },

  "critical_path": {
    "milestone_dates": {
      "production_start": "2026-05-01",
      "production_complete": "2026-05-12",
      "first_delivery": "2026-06-18",
      "installation_start": "2026-06-18",
      "installation_complete": "2026-06-19",
      "commissioning_complete": "2026-06-19",
      "ahj_inspection": "2026-06-20",
      "acceptance": "2026-06-20"
    },
    "total_project_duration_days": 50
  }
}
```

---

## 7. Validation Rules

A conforming CTO parser MUST enforce the following validation rules before accepting a file as valid.

### 7.1 Referential Integrity

- Every `product_id` MUST exist in the referenced catalog
- Every `connected_to` instance_id MUST exist in `placed_elements`, `floor_cartridges`, or `roof_elements`
- The `template_id` (if present) MUST reference a valid template
- Every `instance_id` in `fulfillment_plan` MUST exist in the configuration
- Every `level_id` in `floor_cartridges` and `placed_elements` MUST reference an entry in `floor_levels`

### 7.2 Grid Consistency

- `grid_lines_x_ft` and `grid_lines_y_ft` MUST be monotonically increasing
- All `grid_anchor` references MUST fall within valid grid index bounds
- Floor cartridges MUST tile completely without gaps or overlaps

### 7.3 Constraint Satisfaction

- Products MUST only connect at declared `connection_points`
- `min_adjacent_width_ft` constraints MUST be satisfied
- `max_stack_height` MUST not be exceeded
- `requires_floor_cartridge` products MUST be placed above valid floor cartridges
- `compatible_roof_types` constraints MUST be satisfied
- Products with `permitted_orientation: "horizontal"` MUST NOT be placed vertically
- Products with `permitted_orientation: "vertical"` MUST NOT be placed horizontally
- Products with `permitted_orientation: "sloped"` MUST declare `fixed_pitch` or `pitch_range`
- Wall panels with subcategory `window`, `door`, or `mixed` MUST include `opening_geometry`
- Floor cartridges with subcategory `stairwell` MUST include `opening_geometry` of type `void`

### 7.4 Interface Compatibility

- Connected products MUST declare the same `interface_standard`
- Connected products MUST have at least one common version in `interface_versions`
- Connected products MUST have complementary `gender` values (male/female) or both `neutral`
- Connected products MUST have matching `connection_type` values
- Where `interface_roles` are declared, mated faces MUST address complementary sides of the same CIS standard

### 7.5 Spatial Validity

- No two elements MAY occupy the same space (collision detection uses nominal dimensions)
- Elements MUST NOT extend beyond the building footprint
- Multi-story elements MUST have valid structural support on the level below
- Clearance zones MUST NOT overlap unless explicitly permitted
- Pods MUST be fully contained within floor cartridge coverage (partial overlap is not sufficient)
- Wall panels MUST touch an outer perimeter edge of the floor plate (interior placement is invalid)
- Stairwell conceptual objects with `force_void_alignment: true` MUST be matched by floor cartridges with `opening_geometry` of type `void` at the same position

### 7.6 Structural Performance

- Total transmitted loads at each connection MUST NOT exceed the receiving element's capacity
- Foundation point loads MUST NOT exceed bearing capacity
- Seismic design category of all elements MUST be compatible with project SDC

### 7.7 Thermal Performance

- All exterior elements MUST be compatible with the project's climate zone
- U-factor of the assembly MUST meet IECC requirements for the climate zone

### 7.8 Chain of Custody Consistency

When `fulfillment_plan` is present:

- Every element in the configuration MUST appear in `manufacturing_schedule.production_slots`
- Every element MUST appear in exactly one `delivery_manifest.shipments[].load_manifest`
- Dates MUST be internally consistent (production → delivery → installation → commissioning → acceptance)
- `chain_of_custody` events MUST be in chronological order
- `legal_status` MUST progress correctly: `goods` → `fixture` → `real_property`
- All party blocks MUST conform to the Unified Party Schema (Section 4.3a)

---

## 8. Versioning

### 8.1 Semantic Versioning

The CTO specification follows Semantic Versioning 2.0.0:

- **MAJOR**: Incompatible schema changes
- **MINOR**: Backward-compatible additions
- **PATCH**: Clarifications and corrections

### 8.2 Compatibility

Parsers MUST:
- Accept files with the same MAJOR version
- Accept files with a lower MINOR version (ignoring unknown fields)
- Reject files with a higher MAJOR version

### 8.3 Migration

When MAJOR version changes occur, CfOC will publish migration guides and, where possible, automated migration tools.

---

## 9. MIME Type

### 9.1 Registered Type

```
Type name: application
Subtype name: vnd.cfoc.cto+json
Required parameters: none
Optional parameters: version (e.g., "0.1.5")
Encoding considerations: UTF-8
Security considerations: Standard JSON security considerations apply
Published specification: https://centerforoffsiteconstruction.org/standards/cto-file-format
Contact: standards@centerforoffsiteconstruction.org
```

### 9.2 File Extension

The `.cto` extension is associated with this MIME type.

---

## 10. Examples

### 10.1 Minimal Valid Configuration

```json
{
  "cto_version": "0.1.5",
  "project_meta": {
    "id": "example-001",
    "name": "Minimal Example",
    "created_at": "2026-04-12T12:00:00Z",
    "modified_at": "2026-04-12T12:00:00Z",
    "design_line": "urban_minimalist",
    "is_template": false,
    "project_type": "single_family_residential"
  },
  "catalog_reference": {
    "manufacturer": "Logic Building Systems",
    "manufacturer_address": "456 Industrial Park Dr, Brattleboro, VT 05301",
    "manufacturer_website": "https://buildwithlogic.com",
    "catalog_id": "lbs-2026-q2",
    "catalog_version": "2.1.0"
  },
  "structural_grid": {
    "derived_from": "floor_cartridges",
    "grid_lines_x_ft": [0, 8, 16, 24, 32, 40, 48],
    "grid_lines_y_ft": [0, 20],
    "unit": "ft"
  },
  "floor_levels": [
    {
      "level_id": "L1",
      "name": "Ground Floor",
      "z_origin_ft": 0.0,
      "ceiling_height_ft": 9.0,
      "is_foundation": true
    }
  ],
  "floor_cartridges": [
    {
      "instance_id": "fc-001",
      "component_display_id": "FP-001",
      "product_id": "lbs-fc-h-8x20",
      "story": 1,
      "level_id": "L1",
      "grid_origin": { "x_index": 0, "y_index": 0 },
      "span_x": 1,
      "span_y": 1,
      "rotation_deg": 0,
      "requires_mid_support": false,
      "implied_supports": []
    }
  ],
  "placed_elements": [],
  "roof_elements": [],
  "conceptual_objects": [],
  "pricing_snapshot": {
    "calculated_at": "2026-04-12T12:00:00Z",
    "currency": "USD",
    "line_items": [],
    "subtotal_products": 0,
    "total_msrp": 0
  },
  "validation_state": {
    "is_valid": true,
    "validated_at": "2026-04-12T12:00:00Z",
    "validator_version": "0.1.5",
    "errors": [],
    "warnings": []
  }
}
```

### 10.2 Single-Family Starter Template

See Appendix A for a complete single-family template example.

---

## 11. Conformance

### 11.1 Conformance Levels

| Level | Requirements |
|-------|--------------|
| **CTO Reader** | Can parse and validate CTO files against this specification |
| **CTO Writer** | Can produce valid CTO files that pass validation |
| **CTO Configurator** | Full read/write with interactive editing and real-time validation |
| **CTO Fulfillment System** | Can populate and execute `fulfillment_plan` section including chain of custody |

### 11.2 Validation Suite

CfOC provides a reference validation suite at:
https://github.com/cfoc/cto-validator

Implementations MUST pass all tests in the validation suite to claim conformance.

### 11.3 Certification

Manufacturers and software vendors may apply for CfOC certification by demonstrating conformance to this specification.

---

## 12. Appendices

### Appendix A: Complete Single-Family Template Example

*[To be added: Full JSON example of a Standard One-Story 48'×20' template with all elements including floor cartridges, wall panels, and pods]*

### Appendix B: JSON Schema (machine-readable)

*[To be added: Formal JSON Schema for automated validation]*

### Appendix C: Interface Standard Quick Reference

| Standard | Version | Scope | Key Parameters |
|----------|---------|-------|----------------|
| CfOC-ICC-1220 | 1.0.0 | Module-to-module | Structural coupling, MEP pass-through, air/water barrier |
| CfOC-ICC-1230 | 1.0.0 | Panel-to-panel | Structural fastening, thermal break, weather seal |

### Appendix D: Changelog

See `spec/CHANGELOG.md` for the full version history.

### Appendix E: Acknowledgments

This specification was developed by the Center for Offsite Construction in collaboration with Logic Building Systems and the CfOC Standards Committee.

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
