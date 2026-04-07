# CTO File Format Specification

**Version:** 0.1.0  
**Status:** Draft  
**Published by:** Center for Offsite Construction (CfOC) at NYIT 
**Specification URL:** https://centerforoffsiteconstruction.org/offsite-product-configurator-file-type/ 
**GitHub Repository:** https://github.com/cfoc/cto-file-format  
**License:** MIT 
**Last Updated:** April 5, 2026  

---

## Abstract

The CTO (Configure-to-Order) file format is an open standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

This specification defines the JSON schema, validation rules, and interchange requirements for CTO files. Version 0.1.0 introduces comprehensive product performance attributes, interface standard conformance tracking, and a complete chain-of-custody framework that documents the legal transition from goods to real property.

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

- Single-family residential configurations (v1.0+)
- Multi-family residential configurations (v1.1+)
- Commercial configurations (planned for v2.0)

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
| **Fulfillment Plan** | The complete manufacturing, delivery, and installation schedule for a configuration |
| **Chain of Custody** | The documented sequence of handoffs tracking ownership, risk, and legal status |
| **Legal Mateline** | The boundary at which a product transitions from goods (UCC) to real property (Common Law) |
| **Acceptance** | The event at which the buyer formally accepts an installed product, activating warranties and completing the goods-to-property transition |

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



markdown## 3. File Structure

### 3.1 File Extension

CTO files use the `.cto` extension.

### 3.2 Encoding

CTO files MUST be encoded as UTF-8 JSON without BOM.

### 3.3 Compression

CTO files MAY be compressed using gzip. Compressed files SHOULD use the `.cto.gz` extension.

### 3.4 Top-Level Structure

A CTO file is a JSON object with the following top-level keys:
```json
{
  "cto_version": "0.1.0",
  "cto_type": "assembly",
  "project_meta": { },
  "catalog_reference": { },
  "declares_interface": { },
  "structural_grid": { },
  "floor_cartridges": [ ],
  "placed_elements": [ ],
  "roof_elements": [ ],
  "pricing_snapshot": { },
  "fulfillment_plan": { },
  "validation_state": { }
}
```

The `cto_type` field declares the intended role of this file — whether it is a discrete design element authored by a manufacturer, or a composition of elements authored by a designer. The `declares_interface` section exposes the public-facing interface of this file when it is referenced by a parent assembly; it is required for `element` files and ignored for `assembly` files. All other sections serve their existing purposes.

---

## 4. Schema Definition

### 4.1 `cto_type` (OPTIONAL)
```json
"cto_type": "assembly"
```

Declares the intended role of this file. Parsers MUST NOT reject a file based on `cto_type` alone — it is advisory, not enforced. If absent, parsers MUST treat the file as `"assembly"`.

| Value | Description |
|-------|-------------|
| `"element"` | A discrete design unit authored by a manufacturer — a pod, panel, cartridge, or sub-assembly. SHOULD include a `declares_interface` section. |
| `"assembly"` | A composition of elements — a home design, a building module, or a multi-unit configuration. |
| `"template"` | A pre-validated starting-point configuration intended for user modification. Equivalent to `is_template: true` in `project_meta`. |

### 4.2 `cto_version` (REQUIRED)
```json
"cto_version": "0.1.0"
```

The version of the CTO specification this file conforms to. Parsers MUST reject files with a major version they do not support.

### 4.3 `project_meta` (REQUIRED)
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

### 4.4 `catalog_reference` (REQUIRED)
```json
"catalog_reference": {
  "manufacturer": "Logic Building Systems",
  "catalog_id": "lbs-2026-q2",
  "catalog_version": "2.1.0",
  "catalog_url": "https://buildwithlogic.com/catalog/lbs-2026-q2.json",
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
| `manufacturer` | string | Yes | Manufacturer name |
| `catalog_id` | string | Yes | Unique catalog identifier |
| `catalog_version` | semver string | Yes | Catalog version |
| `catalog_url` | URL string | No | Location to fetch the catalog |
| `catalog_checksum` | string | No | Integrity hash of the catalog file |
| `interface_standards` | array | No | CfOC interface standards used by products in this catalog |

### 4.5 `declares_interface` (REQUIRED for `element` files; omit for `assembly` files)

The `declares_interface` section is the public-facing contract of an `element` file. When a parent assembly references this file via `product_source`, the parent parser reads only this section — the interior of the file is opaque. It exposes exactly the information a parent needs to place, connect, price, and validate the element without knowing how it is built internally.
```json
"declares_interface": {
  "display_name": "Kitchen Pod — Compact",
  "category": "pod",
  "design_line": "urban_minimalist",
  "geometry": {
    "width_ft": 8,
    "length_ft": 12,
    "height_ft": 9,
    "weight_lbs": 8400,
    "bounding_box": {
      "min": { "x": 0, "y": 0, "z": 0 },
      "max": { "x": 8, "y": 12, "z": 9 }
    }
  },
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
  "mep_connections": {
    "electrical": { "voltage": 240, "amperage": 50, "phases": 1, "circuits": 4 },
    "plumbing": { "water_inlet_size_in": 0.75, "drain_size_in": 2 },
    "hvac": { "duct_size_in": 8, "cfm_required": 150 }
  },
  "constraints": {
    "min_adjacent_width_ft": 8,
    "max_stack_height": 1,
    "requires_floor_cartridge": true,
    "compatible_roof_types": ["flat", "shed"]
  },
  "pricing": {
    "base_msrp": 48000,
    "currency": "USD",
    "lead_time_days": 42,
    "pricing_valid_until": "2026-12-31"
  },
  "documentation": {
    "thumbnail_url": "https://buildwithlogic.com/images/pod-001-thumb.jpg",
    "spec_sheet_url": "https://buildwithlogic.com/specs/pod-001.pdf"
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `display_name` | string | Yes | Human-readable name shown in parent configurator |
| `category` | enum | Yes | Product category (see Section 5.2) |
| `design_line` | enum | Yes | Aesthetic family |
| `geometry` | object | Yes | Outer dimensions, weight, and bounding box |
| `connection_points` | array | Yes | All connection points exposed to parent assemblies |
| `mep_connections` | object | No | MEP interface summary (capacities only, not internal routing) |
| `constraints` | object | No | Placement rules the parent configurator must enforce |
| `pricing` | object | Yes | Base MSRP and lead time |
| `documentation` | object | No | Thumbnail and spec sheet for display in parent configurator |

### 4.6 `structural_grid` (REQUIRED)
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
| `derived_from` | enum | Yes | Must be `floor_cartridges` in v1.x |
| `grid_lines_x_ft` | array of numbers | Yes | X-axis grid line positions |
| `grid_lines_y_ft` | array of numbers | Yes | Y-axis grid line positions |
| `unit` | enum | Yes | Unit of measurement: `ft`, `m`, or `mm` |
| `origin` | object | No | Coordinate system origin metadata |

### 4.7 `floor_cartridges` (REQUIRED)
```json
"floor_cartridges": [
  {
    "instance_id": "fc-instance-001",
    "product_id": "fc-001",
    "story": 1,
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
| `story` | integer | Yes | Floor level (1 = ground floor) |
| `grid_origin` | object | Yes | Grid cell indices for cartridge origin |
| `span_x` | integer | Yes | Number of grid cells spanned in X |
| `span_y` | integer | Yes | Number of grid cells spanned in Y |
| `rotation_deg` | number | Yes | Rotation in degrees (0, 90, 180, 270) |
| `requires_mid_support` | boolean | Yes | Whether this cartridge needs intermediate support |
| `implied_supports` | array | Yes | Structural supports generated by this cartridge |
| `instance_data` | object | No | Lifecycle data for this specific instance |

### 4.8 `placed_elements` (REQUIRED)
```json
"placed_elements": [
  {
    "instance_id": "elem-001",
    "product_id": "pod-001",
    "product_source": {
      "type": "cto_file",
      "url": "https://buildwithlogic.com/products/kitchen-pod-compact.cto",
      "cto_version": "0.1.0",
      "checksum": "sha256:abc123..."
    },
    "category": "pod",
    "story": 1,
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
| `product_id` | string | Yes | Human-readable product identifier |
| `product_source` | object | No | If present, resolves this element from a referenced `.cto` file rather than a flat catalog entry |
| `product_source.type` | enum | Yes (if `product_source` present) | Must be `"cto_file"` in v0.1 |
| `product_source.url` | URL string | Yes (if `product_source` present) | Location of the referenced `.cto` element file |
| `product_source.cto_version` | semver string | No | Expected version of the referenced file |
| `product_source.checksum` | string | No | Integrity hash of the referenced file |
| `category` | enum | Yes | Product category: `pod`, `wall_panel`, `floor_cartridge`, etc. |
| `story` | integer | Yes | Floor level |
| `placement` | object | Yes | Positioning information (see below) |
| `rotation_deg` | number | Yes | Rotation in degrees |
| `connections` | array | Yes | Connected elements with interface standard references |
| `options` | object | No | Product options/upgrades |
| `instance_data` | object | No | Lifecycle data for this specific instance |

When `product_source` is present, the parser MUST fetch and validate the referenced `.cto` file and treat its `declares_interface` section as the product definition. The interior of the referenced file is opaque to the parent — only the declared interface (connection points, dimensions, pricing, weight) is visible. `product_id` SHOULD still be populated as a human-readable identifier but is not used for catalog resolution when `product_source` is present.

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

### 4.9 `roof_elements` (REQUIRED)
```json
"roof_elements": [
  {
    "instance_id": "roof-001",
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

### 4.10 `pricing_snapshot` (REQUIRED)
```json
"pricing_snapshot": {
  "calculated_at": "2026-04-04T16:45:00Z",
  "currency": "USD",
  "line_items": [
    {
      "instance_id": "elem-001",
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

### 4.11 `fulfillment_plan` (OPTIONAL)

The fulfillment plan captures the complete production-to-installation timeline. It is OPTIONAL because configurations may be saved before scheduling is complete, but a configuration is not considered "order-ready" until the fulfillment plan is populated and validated.

See [Section 6: Chain of Custody](#6-chain-of-custody) for complete documentation of the fulfillment plan structure.

### 4.12 `validation_state` (REQUIRED)
```json
"validation_state": {
  "is_valid": true,
  "validated_at": "2026-04-04T16:45:00Z",
  "validator_version": "0.1.0",
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
    "dimensions": {
      "width_ft": 8,
      "length_ft": 12,
      "height_ft": 9
    },
    "weight_lbs": 8400,
    "center_of_gravity": {
      "x_ft": 4.0,
      "y_ft": 6.0,
      "z_ft": 4.2
    },
    "bounding_box": {
      "min": { "x": 0, "y": 0, "z": 0 },
      "max": { "x": 8, "y": 12, "z": 9 }
    },
    "clearance_zones": [
      {
        "purpose": "door_swing",
        "min": { "x": -3, "y": 5, "z": 0 },
        "max": { "x": 0, "y": 8, "z": 7 }
      }
    ]
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
      { "id": "lp-1", "position_ft": { "x": 1, "y": 1, "z": 9 }, "capacity_lbs": 5000 },
      { "id": "lp-2", "position_ft": { "x": 7, "y": 1, "z": 9 }, "capacity_lbs": 5000 },
      { "id": "lp-3", "position_ft": { "x": 1, "y": 11, "z": 9 }, "capacity_lbs": 5000 },
      { "id": "lp-4", "position_ft": { "x": 7, "y": 11, "z": 9 }, "capacity_lbs": 5000 }
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
| `wall_panel` | Envelope panel (exterior, interior, partition) |
| `roof_panel` | Roof assembly (flat, shed, gable) |
| `floor_cartridge` | Structural floor assembly |
| `structural_support` | Columns, beams, connections |

### 5.3 Design Line Values

| Value | Description |
|-------|-------------|
| `urban_minimalist` | Clean lines, industrial materials, monochromatic |
| `rustic_retreat` | Natural materials, warm tones, textured surfaces |
| `coastal_breeze` | Light colors, open feel, weather-resistant |

### 5.4 Interface Standard Conformance

Products MUST declare the interface standards to which their connection points conform. This enables the configurator to validate that connected products are compatible.

```json
"connection_points": [
  {
    "id": "cp-north",
    "direction": "north",
    "interface_standard": "CfOC-ICC-1220",
    "interface_versions": ["0.0.1", "0.1.0"],
    "connection_type": "logic_coupler",
    "gender": "male"
  }
]
```

A connection is valid if and only if:
1. Both products declare the same `interface_standard`
2. The `interface_versions` arrays have at least one version in common
3. The `gender` values are complementary (male/female) or both are `neutral`
4. The `connection_type` values match

---

## 6. Chain of Custody

The chain of custody system tracks each product instance from factory release to acceptance, documenting the legal transition from goods (UCC Article 2) to real property (Common Law).

### 6.1 Legal Framework

The chain of custody addresses the key questions identified in the CfOC whitepaper "From Handshake to Hardware":

| Question | Documentation Point |
|----------|---------------------|
| Who owns the product? | `title_holder` at each stage |
| Who bears the risk of loss? | `risk_owner` at each stage |
| Which insurance policy responds? | `insurance_policy` at each stage |
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

```json
"chain_of_custody": [
  {
    "stage": "factory_release",
    "timestamp": "2026-05-08T18:00:00Z",
    "responsible_party": {
      "name": "Logic Building Systems",
      "role": "manufacturer",
      "license_number": "VT-MFG-2026-001",
      "contact": {
        "name": "Factory Operations",
        "email": "factory-ops@buildwithlogic.com",
        "phone": "+1-802-555-0100"
      }
    },
    "witness": {
      "name": "John Smith",
      "organization": "ICC Evaluation Service",
      "role": "third_party_inspector",
      "license_number": "ICC-INSP-12345"
    },
    "condition": {
      "status": "passed_qc",
      "notes": "No defects observed; all commissioning checks passed in factory",
      "photo_urls": [
        "https://buildwithlogic.com/qc/pod-001-sn-12345-release-1.jpg"
      ],
      "qc_report_url": "https://buildwithlogic.com/qc/pod-001-sn-12345-report.pdf"
    },
    "legal_status": "goods",
    "risk_owner": {
      "party": "manufacturer",
      "name": "Logic Building Systems"
    },
    "title_holder": {
      "party": "manufacturer",
      "name": "Logic Building Systems"
    },
    "insurance": {
      "carrier": "Hartford",
      "policy_number": "HFD-PROD-123456",
      "coverage_type": "products_liability",
      "coverage_limit": 5000000
    },
    "documents": [
      {
        "type": "qc_report",
        "url": "https://buildwithlogic.com/qc/pod-001-sn-12345-report.pdf",
        "hash": "sha256:abc123..."
      },
      {
        "type": "third_party_certification",
        "url": "https://buildwithlogic.com/cert/pod-001-sn-12345-icc.pdf",
        "hash": "sha256:def456..."
      }
    ]
  },
  {
    "stage": "delivery",
    "timestamp": "2026-06-18T08:00:00Z",
    "responsible_party": {
      "name": "Northeast Heavy Haul",
      "role": "carrier",
      "mc_number": "MC-123456",
      "contact": {
        "name": "Dispatch",
        "phone": "+1-802-555-0456"
      }
    },
    "witness": {
      "name": "Mike Torres",
      "organization": "Smith Development LLC",
      "role": "site_superintendent"
    },
    "condition": {
      "status": "intact",
      "notes": "Visual inspection passed; no transit damage observed",
      "photo_urls": [
        "https://smithdev.com/projects/smith-residence/delivery-1.jpg"
      ]
    },
    "legal_status": "goods",
    "risk_owner": {
      "party": "buyer",
      "name": "Smith Development LLC",
      "transfer_event": "FOB_destination"
    },
    "title_holder": {
      "party": "buyer",
      "name": "Smith Development LLC",
      "transfer_event": "upon_delivery"
    },
    "insurance": {
      "carrier": "Builder's Risk Mutual",
      "policy_number": "BRM-789012",
      "coverage_type": "builders_risk",
      "coverage_limit": 2000000
    },
    "shipping_documents": {
      "bill_of_lading": "BOL-2026-06-18-001",
      "delivery_receipt_signed": true,
      "delivery_receipt_url": "https://smithdev.com/docs/delivery-receipt-001.pdf"
    }
  },
  {
    "stage": "acceptance",
    "timestamp": "2026-06-20T16:00:00Z",
    "responsible_party": {
      "name": "Smith Development LLC",
      "role": "owner",
      "contact": {
        "name": "Jane Smith",
        "email": "jane@smithdev.com",
        "phone": "+1-802-555-0100"
      }
    },
    "witness": {
      "name": "Building Inspector Johnson",
      "organization": "Town of Brattleboro",
      "role": "authority_having_jurisdiction",
      "license_number": "VT-BI-789"
    },
    "condition": {
      "status": "commissioned_and_accepted",
      "notes": "All systems tested and operational; CO issued",
      "photo_urls": [],
      "commissioning_report_url": "https://smithdev.com/docs/commissioning-001.pdf"
    },
    "legal_status": "real_property",
    "risk_owner": {
      "party": "owner",
      "name": "Smith Development LLC"
    },
    "title_holder": {
      "party": "owner",
      "name": "Smith Development LLC"
    },
    "insurance": {
      "carrier": "Vermont Property Insurance",
      "policy_number": "VPI-345678",
      "coverage_type": "property",
      "coverage_limit": 500000
    },
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
    },
    "embedded_warranty_transfers": [
      {
        "product": "GE Refrigerator GTS18GTHWW",
        "serial_number": "GE-REF-987654",
        "warranty_transferred_to": "Smith Development LLC",
        "warranty_expiration": "2027-06-20"
      }
    ]
  }
]
```

### 6.4 Fulfillment Plan Structure

The complete fulfillment plan integrates chain of custody with manufacturing, delivery, and installation scheduling:

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
        "product_id": "pod-001",
        "product_name": "Kitchen Pod — Compact",
        "serial_number": "LBS-POD-2026-001234",
        "production_start": "2026-05-01T06:00:00Z",
        "production_end": "2026-05-08T18:00:00Z",
        "production_days": 6,
        "station_assignments": [
          { "station": "framing", "start": "2026-05-01", "end": "2026-05-02" },
          { "station": "mep_rough", "start": "2026-05-03", "end": "2026-05-04" },
          { "station": "insulation", "start": "2026-05-05", "end": "2026-05-05" },
          { "station": "finishes", "start": "2026-05-06", "end": "2026-05-07" },
          { "station": "qc_staging", "start": "2026-05-08", "end": "2026-05-08" }
        ],
        "qc_status": "passed",
        "qc_date": "2026-05-08",
        "third_party_inspection_date": "2026-05-08",
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
        "carrier": {
          "name": "Northeast Heavy Haul",
          "mc_number": "MC-123456",
          "dot_number": "DOT-789012",
          "contact_phone": "+1-802-555-0456",
          "insurance_carrier": "Trucking Insurance Co",
          "insurance_policy": "TIC-456789"
        },
        "truck": {
          "type": "flatbed",
          "length_ft": 48,
          "max_weight_lbs": 48000,
          "license_plate": "VT-123456"
        },
        "load_manifest": [
          {
            "instance_id": "elem-001",
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
        "shipping_terms": "FOB_destination",
        "title_transfer_event": "upon_delivery",
        "risk_transfer_event": "upon_delivery"
      }
    ]
  },

  "crane_lift_plan": {
    "crane": {
      "type": "mobile_hydraulic",
      "capacity_tons": 100,
      "boom_length_ft": 140,
      "provider": "Green Mountain Crane Services",
      "contact_phone": "+1-802-555-0789",
      "operator_certification": "NCCCO-12345"
    },
    "setup": {
      "arrival_date": "2026-06-18",
      "arrival_time": "06:00",
      "setup_duration_hours": 2,
      "position": {
        "description": "Southeast corner of lot",
        "coordinates": { "latitude": 42.8508, "longitude": -72.5575 }
      },
      "ground_preparation": "timber_mats",
      "outrigger_spread_ft": 24,
      "swing_radius_clear": true
    },
    "lift_operations": [
      {
        "lift_id": "lift-001",
        "sequence_order": 1,
        "instance_id": "elem-001",
        "serial_number": "LBS-POD-2026-001234",
        "product_name": "Kitchen Pod — Compact",
        "weight_lbs": 8400,
        "rigging_config": {
          "spreader_bar": true,
          "spreader_length_ft": 10,
          "sling_count": 4,
          "sling_angle_deg": 60,
          "sling_capacity_lbs": 6000
        },
        "scheduled_date": "2026-06-18",
        "scheduled_time": "10:00",
        "estimated_duration_minutes": 60,
        "placement_coordinates": { "x_ft": 2, "y_ft": 4, "z_ft": 0 },
        "rotation_deg": 0,
        "dependencies": [],
        "safety_requirements": [
          "Tag lines required",
          "Clear swing radius before lift",
          "Spotter at each corner"
        ],
        "status": "scheduled"
      }
    ],
    "total_lift_days": 1,
    "demobilization_date": "2026-06-18",
    "demobilization_time": "16:00"
  },

  "installation_schedule": {
    "general_contractor": {
      "company": "Smith Development LLC",
      "contact_name": "Jane Smith",
      "contact_phone": "+1-802-555-0100",
      "license_number": "VT-GC-12345"
    },
    "crew_assignments": [
      {
        "role": "crane_operator",
        "provider": "Green Mountain Crane Services",
        "personnel_count": 1,
        "dates": ["2026-06-18"],
        "shift": { "start": "06:00", "end": "18:00" },
        "certifications_required": ["NCCCO"],
        "hourly_rate": 150,
        "total_hours": 12
      },
      {
        "role": "rigger",
        "provider": "Green Mountain Crane Services",
        "personnel_count": 2,
        "dates": ["2026-06-18"],
        "shift": { "start": "06:00", "end": "18:00" },
        "certifications_required": ["NCCCO Rigging"],
        "hourly_rate": 85,
        "total_hours": 24
      },
      {
        "role": "mep_connector",
        "provider": "Logic Building Systems",
        "personnel_count": 2,
        "dates": ["2026-06-18", "2026-06-19"],
        "shift": { "start": "07:00", "end": "17:00" },
        "certifications_required": ["Licensed Plumber", "Licensed Electrician"],
        "hourly_rate": 95,
        "total_hours": 40
      }
    ],
    "installation_tasks": [
      {
        "task_id": "task-001",
        "description": "Set kitchen pod and level",
        "instance_ids": ["elem-001"],
        "assigned_roles": ["crane_operator", "rigger"],
        "scheduled_date": "2026-06-18",
        "scheduled_start": "10:00",
        "estimated_duration_hours": 1,
        "dependencies": [],
        "status": "scheduled"
      },
      {
        "task_id": "task-002",
        "description": "Connect MEP systems",
        "instance_ids": ["elem-001"],
        "assigned_roles": ["mep_connector"],
        "scheduled_date": "2026-06-18",
        "scheduled_start": "11:00",
        "estimated_duration_hours": 4,
        "dependencies": ["task-001"],
        "status": "scheduled"
      },
      {
        "task_id": "task-003",
        "description": "Commission and test systems",
        "instance_ids": ["elem-001"],
        "assigned_roles": ["mep_connector"],
        "scheduled_date": "2026-06-19",
        "scheduled_start": "07:00",
        "estimated_duration_hours": 4,
        "dependencies": ["task-002"],
        "status": "scheduled"
      }
    ],
    "installation_start_date": "2026-06-18",
    "installation_end_date": "2026-06-19",
    "total_installation_days": 2
  },

  "commissioning": {
    "commissioning_agent": {
      "company": "Logic Building Systems",
      "contact_name": "Technical Services",
      "contact_phone": "+1-802-555-0150"
    },
    "scheduled_date": "2026-06-19",
    "checklist_version": "3.2.0",
    "ahj_inspection": {
      "scheduled_date": "2026-06-20",
      "inspector_assigned": false,
      "inspection_type": "final"
    }
  },

  "contingencies": {
    "weather_hold_days": 2,
    "buffer_days": 1,
    "weather_criteria": {
      "max_wind_mph": 25,
      "no_precipitation": true,
      "min_temp_f": 32
    },
    "rain_delay_protocol": "Hold crane ops if winds exceed 25 mph or lightning within 10 miles",
    "reschedule_contact": {
      "name": "Mike Torres",
      "phone": "+1-802-555-0123"
    }
  },

  "critical_path": {
    "milestone_dates": {
      "production_start": "2026-05-01",
      "production_complete": "2026-05-12",
      "ship_ready": "2026-05-09",
      "first_delivery": "2026-06-18",
      "installation_start": "2026-06-18",
      "installation_complete": "2026-06-19",
      "commissioning_complete": "2026-06-19",
      "ahj_inspection": "2026-06-20",
      "acceptance": "2026-06-20"
    },
    "critical_dependencies": [
      {
        "from": "production_complete",
        "to": "first_delivery",
        "slack_days": 36,
        "notes": "Buffer for shipping coordination"
      },
      {
        "from": "installation_complete",
        "to": "commissioning_complete",
        "slack_days": 0,
        "notes": "Same-day commissioning"
      }
    ],
    "total_project_duration_days": 50
  }
}
```

---

markdown## 7. Validation Rules

A conforming CTO parser MUST enforce the following validation rules before accepting a file as valid.

### 7.1 Referential Integrity

- Every `product_id` MUST exist in the referenced catalog, OR the element MUST have a valid `product_source` object
- Every `connected_to` instance_id MUST exist in `placed_elements`, `floor_cartridges`, or `roof_elements`
- The `template_id` (if present) MUST reference a valid template
- Every `instance_id` in `fulfillment_plan` MUST exist in the configuration

### 7.2 Nested Element Resolution

When a `placed_element` contains a `product_source` object:

- The parser MUST fetch the referenced `.cto` file at `product_source.url`
- The referenced file MUST be a valid `.cto` file with `cto_type: "element"`
- The referenced file MUST contain a `declares_interface` section
- If `product_source.checksum` is present, the parser MUST verify the file integrity before use
- If `product_source.cto_version` is present, the parser MUST verify the referenced file's `cto_version` matches
- The parser MUST use only the `declares_interface` section of the referenced file for validation — the interior is opaque
- If the referenced file cannot be fetched or fails validation, the parent file MUST be marked invalid

### 7.3 Grid Consistency

- `grid_lines_x_ft` and `grid_lines_y_ft` MUST be monotonically increasing
- All `grid_anchor` references MUST fall within valid grid index bounds
- Floor cartridges MUST tile completely without gaps or overlaps

### 7.4 Constraint Satisfaction

- Products MUST only connect at declared `connection_points`
- `min_adjacent_width_ft` constraints MUST be satisfied
- `max_stack_height` MUST not be exceeded
- `requires_floor_cartridge` products MUST be placed above valid floor cartridges
- `compatible_roof_types` constraints MUST be satisfied

### 7.5 Interface Compatibility

- Connected products MUST declare the same `interface_standard`
- Connected products MUST have at least one common version in `interface_versions`
- Connected products MUST have complementary `gender` values (male/female) or both `neutral`
- Connected products MUST have matching `connection_type` values
- When one or both connected elements are resolved from a `product_source`, interface compatibility MUST be validated against their respective `declares_interface.connection_points`

### 7.6 Spatial Validity

- No two elements MAY occupy the same space (collision detection)
- Elements MUST NOT extend beyond the building footprint
- Multi-story elements MUST have valid structural support
- Clearance zones MUST NOT overlap unless explicitly permitted

### 7.7 Structural Performance

- Total transmitted loads at each connection MUST NOT exceed the receiving element's capacity
- Foundation point loads MUST NOT exceed bearing capacity
- Seismic design category of all elements MUST be compatible with project SDC

### 7.8 Thermal Performance

- All exterior elements MUST be compatible with the project's climate zone
- U-factor of the assembly MUST meet IECC requirements for the climate zone

### 7.9 Chain of Custody Consistency

When `fulfillment_plan` is present:

- Every element in the configuration MUST appear in `manufacturing_schedule.production_slots`
- Every element MUST appear in exactly one `delivery_manifest.shipments[].load_manifest`
- Every element requiring crane placement MUST appear in `crane_lift_plan.lift_operations`
- Dates MUST be internally consistent (production → shipping → delivery → installation → commissioning → acceptance)
- `chain_of_custody` events MUST be in chronological order
- `legal_status` MUST progress correctly: `goods` → `fixture` → `real_property`


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
Optional parameters: version (e.g., "0.1.0")
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
  "cto_version": "0.1.0",
  "project_meta": {
    "id": "example-001",
    "name": "Minimal Example",
    "created_at": "2026-04-04T12:00:00Z",
    "modified_at": "2026-04-04T12:00:00Z",
    "design_line": "urban_minimalist",
    "is_template": false,
    "project_type": "single_family_residential"
  },
  "catalog_reference": {
    "manufacturer": "Logic Building Systems",
    "catalog_id": "lbs-2026-q2",
    "catalog_version": "2.1.0"
  },
  "structural_grid": {
    "derived_from": "floor_cartridges",
    "grid_lines_x_ft": [0, 24],
    "grid_lines_y_ft": [0, 32],
    "unit": "ft"
  },
  "floor_cartridges": [
    {
      "instance_id": "fc-001",
      "product_id": "fc-standard-24x32",
      "story": 1,
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
  "pricing_snapshot": {
    "calculated_at": "2026-04-04T12:00:00Z",
    "currency": "USD",
    "line_items": [],
    "subtotal_products": 0,
    "total_msrp": 0
  },
  "validation_state": {
    "is_valid": true,
    "validated_at": "2026-04-04T12:00:00Z",
    "validator_version": "0.1.0",
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

*[To be added: Full JSON example of a Standard One-Story template with all elements]*

### Appendix B: JSON Schema (machine-readable)

*[To be added: Formal JSON Schema for automated validation]*

### Appendix C: Interface Standard Quick Reference

| Standard | Version | Scope | Key Parameters |
|----------|---------|-------|----------------|
| CfOC-ICC-1220 | 1.0.0 | Module-to-module | Structural coupling, MEP pass-through, air/water barrier |
| CfOC-ICC-1230 | 1.0.0 | Panel-to-panel | Structural fastening, thermal break, weather seal |

### Appendix D: Changelog

| Version | Date | Changes |
|---------|------|---------|
| 0.0.0 | 2026-04-04 | Initial release |
| 0.0.1 | 2026-04-04 | Expanded `delivery_schedule` to `fulfillment_plan` |
| 0.1.0 | 2026-04-04 | Added interface standard conformance, chain of custody, legal mateline tracking, expanded product schema with structural/thermal performance |
| 0.2.0 | 2026-04-07 | Added `cto_type` field enabling element/assembly distinction; added `declares_interface` section for opaque nesting of element files within assembly files; added `product_source` field on `placed_elements` enabling resolution from referenced `.cto` files; added Section 7.2 validation rules for nested element resolution |

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
- Connor Baily, CfOC Senior Research Fellow, DPR Construction
- Edward Palka, CfOC Senior Research Fellow, CLAE

**Feedback:**
Submit issues and pull requests to https://github.com/cfoc/cto-file-format

**Copyright:**
© 2026 Center for Offsite Construction. This specification is released under the Apache 2.0 License.
