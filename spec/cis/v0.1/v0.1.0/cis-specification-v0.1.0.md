# CIS File Format Specification

**Version:** 0.1.0
**Status:** Draft
**Published by:** Center for Offsite Construction (CfOC) at NYIT
**Specification URL:** https://centerforoffsiteconstruction.org/offsite-product-configurator-file-type/
**GitHub Repository:** https://github.com/Jason-Van-Nest/configurator-file-type-spec
**License:** Apache 2.0
**Last Updated:** April 18, 2026

---

## Abstract

The CIS (Configure-to-Order Interface Standard) file format is an open standard for encoding connection interface definitions used in offsite construction. A `.cis` file describes a complete connection standard — including its sides, port geometry, utility requirements, structural handshake, and version compatibility — as a machine-readable document that can be referenced by `.cto` element files.

CIS files are the companion format to the CTO file format. Where `.cto` files describe *what a product is*, `.cis` files describe *how products connect*. A `.cto` element file declares conformance to one or more CIS files; the CIS file carries the detailed connection specification so that element files can remain compact and focused on product-specific data.

Version 0.1.0 establishes the baseline CIS structure: sides, port clusters with geometry, utility connections, structural connections, versioning, and the distinction between open standards and catalog-internal proprietary standards.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [File Structure](#3-file-structure)
4. [Schema Definition](#4-schema-definition)
5. [Sides and Sidedness](#5-sides-and-sidedness)
6. [Port Geometry](#6-port-geometry)
7. [Utility Connections](#7-utility-connections)
8. [Structural Connections](#8-structural-connections)
9. [Versioning and Compatibility](#9-versioning-and-compatibility)
10. [Registry Scope](#10-registry-scope)
11. [Validation Rules](#11-validation-rules)
12. [MIME Type](#12-mime-type)
13. [Examples](#13-examples)
14. [Conformance](#14-conformance)
15. [Relationship to the CTO File Format](#15-relationship-to-the-cto-file-format)
16. [Appendices](#16-appendices)

---

## 1. Introduction

### 1.1 Purpose

The CIS file format enables connection interface standards to be published, versioned, and referenced by product catalogs as first-class documents. A CIS file provides:

- A canonical definition of a connection standard's sides, ports, and utility carriage
- Geometric coordinates that enable automated validation of connection alignment
- Version compatibility rules that govern which products can mate with which
- A distinction between open (publicly registered) and proprietary (catalog-internal) standards
- A reference target for `.cto` element files declaring interface conformance

### 1.2 Scope

This specification covers the structure of individual CIS files. It does not cover the CIS registry, the submission process for new open standards, or the governance of standard versioning at CfOC. Those are addressed in separate documents.

### 1.3 Terminology

| Term | Definition |
|------|------------|
| **Connection Standard** | A specification defining how two products can be physically and functionally joined |
| **CIS File** | A `.cis` file that documents a connection standard in machine-readable form |
| **Side** | One of the two faces of an interface, each addressed by a different product. For example, a module-to-services coupler has a "dwelling unit side" and a "building services side" |
| **Port** | A discrete connection point within a side — a single pipe, cable, bolt, or alignment feature |
| **Port Cluster** | The complete set of ports on one side of an interface, together forming the geometric pattern that the mating side must match |
| **Registry Scope** | Whether a CIS file is `open_standard` (publicly registered with a central authority such as CfOC) or `catalog_internal` (proprietary to a single manufacturer's catalog) |
| **Publisher** | The entity that authored and maintains the standard |
| **Mating** | The act of joining two products whose faces address complementary sides of the same CIS |

### 1.4 Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### 1.5 Related Specifications

| Standard | Organization | Relevance |
|----------|--------------|-----------|
| CTO File Format | CfOC | Element files reference CIS files via `declares_interface` and `interface_roles` |
| CfOC-ICC-1220 | CfOC/ICC | Example open-standard CIS for module-to-building-services handshaking |
| CfOC-ICC-1230 | CfOC/ICC | Example open-standard CIS for panel-to-panel connectivity |

---

## 2. Design Principles

### 2.1 Single Source of Truth for Connection Data

A CIS file is the authoritative description of a connection standard. Element `.cto` files MUST NOT duplicate the interface's port geometry, utility requirements, or structural handshake; they reference the CIS file and trust its contents.

### 2.2 Sides Are First-Class

Every connection standard has two sides. The CIS file names them, describes their envelopes, and assigns ports to them. Products always address exactly one side.

### 2.3 Geometry Enables Validation

Port positions within a side are specified as coordinates. This enables the configurator to validate — not just that two products claim compatibility, but that their ports actually align when mated.

### 2.4 Open and Proprietary Standards Coexist

A CIS file can describe an open standard published by CfOC or a proprietary standard owned by a single manufacturer. The file format is the same; the `registry_scope` field distinguishes them.

### 2.5 Versioning Enables Evolution

Connection standards evolve. The CIS format encodes version history, backward-compatibility rules, and the superseded-by relationship so that element files can reason about which versions they support.

### 2.6 Utilities and Structure Are Both In Scope

A real connection standard governs both the utility handshake (pipes, cables, ducts) and the structural handshake (bolts, alignment pins, load transfer surfaces). Both are first-class in CIS v0.1.0.

---

## 3. File Structure

### 3.1 File Extension

CIS files use the `.cis` extension.

### 3.2 Encoding

CIS files MUST be encoded as UTF-8 JSON without BOM.

### 3.3 Compression

CIS files MAY be compressed using gzip. Compressed files SHOULD use the `.cis.gz` extension.

### 3.4 Top-Level Structure

A CIS file is a JSON object with the following top-level keys:

```json
{
  "cis_format_version": "0.1.0",
  "id": "CfOC-ICC-1220",
  "standard_version": "1.0.0",
  "name": "Module-to-Building-Services Utility Interface",
  "description": "Standard for volumetric module handshaking with site-supplied or panel-embedded utility couplers",
  "publisher": { },
  "published_date": "2026-04-18",
  "status": "draft",
  "registry_scope": "open_standard",
  "registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v1.0.0.cis",
  "license": "Apache-2.0",
  "checksum": "sha256:...",
  "supersedes": null,
  "superseded_by": null,
  "compatibility": { },
  "sides": { },
  "ports": [ ],
  "utility_connections": [ ],
  "structural_connections": [ ],
  "tolerances": { }
}
```

The `cis_format_version` field tracks which version of *this specification* the file conforms to (analogous to `cto_version`). The `standard_version` field tracks the version of the connection standard itself.

---

## 4. Schema Definition

### 4.1 `cis_format_version` (REQUIRED)

```json
"cis_format_version": "0.1.0"
```

The version of the CIS specification this file conforms to. Parsers MUST reject files with a major version they do not support.

### 4.2 `id` (REQUIRED)

```json
"id": "CfOC-ICC-1220"
```

A unique, stable identifier for the connection standard. For open standards published through CfOC, the format is `CfOC-{organization}-{number}` (e.g., `CfOC-ICC-1220`). For catalog-internal proprietary standards, the format is `{manufacturer}-INT-{scope}` (e.g., `LBS-INT-KTU-UTL` for the Logic Building Systems internal kitchen-to-utility standard).

See Section 10 for registry scope conventions.

### 4.3 `standard_version` (REQUIRED)

```json
"standard_version": "1.0.0"
```

The version of the connection standard itself, following Semantic Versioning 2.0.0.

### 4.4 `name` (REQUIRED)

A human-readable name for the standard. SHOULD be descriptive of the standard's scope rather than its identifier (e.g., "Module-to-Building-Services Utility Interface", not "CfOC-ICC-1220").

### 4.5 `description` (REQUIRED)

A brief prose description of what the standard governs.

### 4.6 `publisher` (REQUIRED)

```json
"publisher": {
  "name": "Center for Offsite Construction",
  "organization_type": "standards_body",
  "website": "https://centerforoffsiteconstruction.org",
  "contact": {
    "name": "Standards Committee",
    "email": "standards@centerforoffsiteconstruction.org"
  }
}
```

The entity that authored and maintains the standard. For proprietary standards, this is the manufacturer.

### 4.7 `published_date` (REQUIRED)

ISO 8601 date string for when this version was published.

### 4.8 `status` (REQUIRED)

One of:
- `draft` — under active development, not stable
- `published` — stable, available for use
- `deprecated` — still usable but discouraged; see `superseded_by`
- `withdrawn` — no longer valid

### 4.9 `registry_scope` (REQUIRED)

One of:
- `open_standard` — publicly registered, available for any manufacturer to adopt
- `catalog_internal` — proprietary to a specific manufacturer's catalog; may not be usable by outside products

See Section 10 for details.

### 4.10 `registry_url` (CONDITIONAL)

REQUIRED when `registry_scope` is `open_standard`. OPTIONAL (and typically null) when `catalog_internal`. The canonical URL where this exact version of the CIS file can be fetched.

### 4.11 `license` (REQUIRED)

SPDX license identifier (e.g., `Apache-2.0`, `CC-BY-4.0`, `Proprietary`).

### 4.12 `checksum` (RECOMMENDED)

A content hash (e.g., `sha256:...`) that CTO files can use to verify they are referencing an exact, unmodified version of the standard.

### 4.13 `supersedes` / `superseded_by` (OPTIONAL)

References to previous or successor versions as CIS ID strings including version (e.g., `"CfOC-ICC-1220@0.9.0"`).

### 4.14 `compatibility` (REQUIRED)

```json
"compatibility": {
  "compatible_with": ["1.0.0", "1.0.1", "1.1.0"],
  "breaking_changes_from": "0.9.0",
  "forward_compatible": false,
  "description": "This version is wire-compatible with 1.0.x and 1.1.0. Version 2.0.0 will introduce breaking changes to port geometry and is not backward-compatible."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `compatible_with` | array of semver strings | Yes | Which other versions of this standard products conforming to this file can mate with |
| `breaking_changes_from` | semver string | No | The earliest version before which there are known breaking incompatibilities |
| `forward_compatible` | boolean | Yes | Whether products conforming to this version can safely mate with future minor versions |

---

## 5. Sides and Sidedness

Every CIS file MUST define exactly two sides. The sides have fixed semantic names defined by the standard itself — `dwelling_unit_side` / `building_services_side`, `kitchen_side` / `utility_side`, `male` / `female`, etc.

### 5.1 `sides` Block (REQUIRED)

```json
"sides": {
  "dwelling_unit_side": {
    "description": "Face that mates with a Logic Pod or other volumetric module",
    "envelope": {
      "nominal_width_in": 6.0,
      "nominal_height_in": 92.5,
      "clearance_depth_in": 14.0,
      "description": "The face envelope is the rectangular zone on the mating face within which all ports must reside. Clearance depth is the volume behind the face that must be kept free of obstructions."
    },
    "mirror_of": null,
    "notes": "Ports are laid out in product-local coordinates where (0,0) is the bottom-left of the envelope from the perspective of someone facing the side."
  },
  "building_services_side": {
    "description": "Face that mates with site-built utility stubs or panel-embedded couplers",
    "envelope": {
      "nominal_width_in": 6.0,
      "nominal_height_in": 92.5,
      "clearance_depth_in": 14.0
    },
    "mirror_of": "dwelling_unit_side",
    "notes": "Port positions on this side are mirror images of the dwelling_unit_side ports along the vertical centerline of the envelope. See Section 6.3."
  }
}
```

### 5.2 Side Naming

Side names are defined by the standard. Conventions:
- **Asymmetric standards** (one side is a "producer," the other a "consumer"): name sides by role, e.g., `dwelling_unit_side` / `building_services_side`, `kitchen_side` / `utility_side`
- **Symmetric standards** (either side can play either role): use `side_a` / `side_b`
- **Gendered standards** (physical male/female): use `male` / `female`

Side names MUST be stable across versions of a given standard. Renaming a side constitutes a breaking change.

### 5.3 The `mirror_of` Field

When one side's port layout is the geometric mirror image of the other side (typical for pipe-to-pipe connections where two opposing faces must have matching ports), the mirrored side's `mirror_of` field points at the authoritative side. Ports defined on the authoritative side are automatically mirrored onto the `mirror_of` side during validation.

When sides have independent port layouts (typical for structural connections where, e.g., one side has bolts and the other has bolt-receiving holes), `mirror_of` is null and each side declares its own ports.

---

## 6. Port Geometry

### 6.1 The `ports` Array (REQUIRED)

```json
"ports": [
  {
    "port_id": "cold_water_supply",
    "side": "dwelling_unit_side",
    "position_in": { "x": 1.5, "y": 24.0 },
    "tolerance_in": { "x": 0.25, "y": 0.25 },
    "category": "utility",
    "utility_connection_ref": "cold_water_supply_utility",
    "direction": "ingress",
    "description": "Cold water supply port on the dwelling unit side. Position is measured from the bottom-left corner of the side envelope."
  },
  {
    "port_id": "structural_bolt_upper_left",
    "side": "dwelling_unit_side",
    "position_in": { "x": 0.5, "y": 90.0 },
    "tolerance_in": { "x": 0.125, "y": 0.125 },
    "category": "structural",
    "structural_connection_ref": "primary_shear_bolts",
    "description": "Upper-left structural alignment bolt."
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `port_id` | string | Yes | Unique identifier within this CIS file |
| `side` | string | Yes | Which side this port is on |
| `position_in` | object | Yes | Position in inches within the side's envelope, measured from the bottom-left corner |
| `tolerance_in` | object | Yes | Acceptable deviation in inches |
| `category` | enum | Yes | `utility`, `structural`, `alignment`, `data` |
| `utility_connection_ref` | string | Conditional | REQUIRED when `category: "utility"` — points to an entry in `utility_connections[]` |
| `structural_connection_ref` | string | Conditional | REQUIRED when `category: "structural"` — points to an entry in `structural_connections[]` |
| `direction` | enum | Conditional | For utility ports: `ingress`, `egress`, `bidirectional` — describes flow direction from the perspective of this side |

### 6.2 Coordinate System

Port positions are specified in **product-local inches** with the origin at the bottom-left corner of the side envelope, as viewed by someone facing the side. X increases rightward, Y increases upward.

### 6.3 Mirror Geometry

When a side has `mirror_of: "other_side_name"`, its ports are derived by mirroring the other side's ports across the vertical centerline of the envelope. If the envelope width is W, a port at `(x, y)` on the authoritative side appears at `(W - x, y)` on the mirrored side. The CIS file SHOULD NOT re-declare mirrored ports; validators compute them.

---

## 7. Utility Connections

### 7.1 The `utility_connections` Array (OPTIONAL — REQUIRED for utility-carrying standards)

```json
"utility_connections": [
  {
    "utility_id": "cold_water_supply_utility",
    "type": "cold_water",
    "requirement": {
      "pressure_psi_min": 45,
      "pressure_psi_max": 60,
      "pipe_spec": "1 inch PEX B",
      "pipe_standard": "ASTM F876",
      "fitting": "Brass Barbed",
      "fitting_standard": "ASTM F1807",
      "method": "Clamp & Crimp",
      "method_standard": "ASTM F1960"
    }
  },
  {
    "utility_id": "primary_electrical_feed",
    "type": "electrical",
    "requirement": {
      "supply": "120V AC",
      "amperage_amps": 100,
      "cable_type": "SOOW Heavy Duty",
      "code_standard": "NEC / CEC C22.1"
    }
  },
  {
    "utility_id": "sanitary_drain",
    "type": "sanitary",
    "requirement": {
      "pipe_spec": "4.5 inch PVC Schedule 40",
      "pipe_standard": "ASTM D2665",
      "fitting": "Rubber Fitting w/ Band Clamp",
      "slope_required": "1/4 inch per foot minimum"
    }
  }
]
```

Each utility connection is referenced by one or more ports via the `utility_connection_ref` field on the port. This separation allows a single utility specification (e.g., "1 inch PEX cold water at 45-60 PSI") to be shared across multiple port instances if the standard has redundant supply points.

### 7.2 Utility Types

Recommended values for `type`:

| Value | Description |
|-------|-------------|
| `cold_water` | Cold water supply |
| `hot_water` | Hot water supply |
| `sanitary` | Sanitary drain / wastewater |
| `vent` | Plumbing vent |
| `gas` | Natural gas or propane |
| `electrical` | Electrical supply |
| `data` | Data / network cabling |
| `hvac_supply` | HVAC supply air |
| `hvac_return` | HVAC return air |
| `hvac_exhaust` | HVAC exhaust |
| `fire_suppression` | Sprinkler or suppression line |
| `custom` | Manufacturer-specific utility; `type_extension` field required |

---

## 8. Structural Connections

### 8.1 The `structural_connections` Array (OPTIONAL — REQUIRED for structurally active standards)

```json
"structural_connections": [
  {
    "structural_id": "primary_shear_bolts",
    "type": "bolted_connection",
    "bolt_spec": "3/4 inch A325 Grade 5",
    "bolt_standard": "ASTM F3125",
    "torque_ft_lbs": 125,
    "shear_capacity_lbs": 5000,
    "tension_capacity_lbs": 3500,
    "quantity_per_connection": 4,
    "description": "Four 3/4-inch A325 bolts at the corners of the side envelope, providing primary shear load transfer between mated products."
  },
  {
    "structural_id": "alignment_pins",
    "type": "alignment_feature",
    "pin_spec": "1/2 inch hardened steel, 2 inch exposure",
    "purpose": "Registers the two sides during placement; carries no structural load",
    "quantity_per_connection": 2
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `structural_id` | string | Yes | Unique identifier within this CIS file |
| `type` | enum | Yes | `bolted_connection`, `welded_connection`, `alignment_feature`, `bearing_surface`, `custom` |
| Remaining fields | varies | varies | Depend on connection type |

---

## 9. Versioning and Compatibility

### 9.1 Semantic Versioning

CIS `standard_version` follows Semantic Versioning 2.0.0:

- **MAJOR**: Breaking changes — port positions moved, sides renamed, utilities removed, dimensions changed incompatibly
- **MINOR**: Backward-compatible additions — new optional ports, expanded tolerance ranges, additional utility types
- **PATCH**: Clarifications and corrections — description text, typo fixes, clarifying notes

### 9.2 Compatibility Negotiation

When a CTO file references a CIS file at version X, it is declaring conformance to **any version in X's `compatible_with` array**. The configurator performs compatibility negotiation by checking whether the versions declared by the two mating products have a common entry in their respective `compatible_with` arrays.

### 9.3 Deprecation

A standard marked `status: "deprecated"` MAY still be used but SHOULD be replaced by the version listed in `superseded_by`. Configurators SHOULD emit warnings when a configuration uses deprecated standards.

---

## 10. Registry Scope

### 10.1 Open Standards

`registry_scope: "open_standard"` indicates a standard published through a central authority (typically CfOC) and available for any manufacturer to adopt. Open standards:

- MUST have a populated `registry_url`
- MUST have a public, fetchable canonical version
- MUST use the `CfOC-{organization}-{number}` naming convention or equivalent
- SHOULD be licensed under a permissive license (Apache-2.0, CC-BY-4.0, or similar)
- SHOULD be published with a checksum for integrity verification

### 10.2 Catalog-Internal Standards

`registry_scope: "catalog_internal"` indicates a standard owned by a single manufacturer and used only within that manufacturer's catalog. Proprietary standards:

- MAY have a null `registry_url` (the standard may not be publicly accessible)
- MUST use the `{manufacturer}-INT-{scope}` naming convention (e.g., `LBS-INT-KTU-UTL`)
- MAY be licensed `Proprietary`
- MUST still be parseable by any CIS-conformant parser — the file format is the same regardless of scope
- Products declaring catalog-internal standards can only mate with other products from the same catalog

### 10.3 Naming Conventions for Proprietary Standards

The `{manufacturer}-INT-{scope}` convention encourages descriptive scope names. Recommended patterns:

| Pattern | Meaning | Example |
|---------|---------|---------|
| `{MFG}-INT-{product_a}-{product_b}` | Interface between two specific product types | `LBS-INT-KTU-UTL` (kitchen-to-utility) |
| `{MFG}-INT-{scope}` | Scope-wide internal standard | `LBS-INT-STACK` (any pod stacking) |
| `{MFG}-INT-{system}` | System-level internal standard | `LBS-INT-SENSOR-BUS` |

---

## 11. Validation Rules

A conforming CIS parser MUST enforce the following.

### 11.1 Referential Integrity

- Every `port.utility_connection_ref` MUST reference an entry in `utility_connections[]`
- Every `port.structural_connection_ref` MUST reference an entry in `structural_connections[]`
- Every `sides[X].mirror_of` MUST reference another key in `sides[]`
- `supersedes` and `superseded_by` MUST reference valid CIS identifier + version strings

### 11.2 Geometric Validity

- Port positions MUST fall within the envelope of their declared side
- Mirrored sides MUST NOT re-declare ports that would conflict with the computed mirror positions
- Tolerance values MUST be non-negative

### 11.3 Sidedness Consistency

- A CIS file MUST declare exactly two sides
- Side names MUST NOT be reused for different roles across versions of the same standard
- Ports MUST be assigned to a declared side

### 11.4 Registry Scope Consistency

- `registry_scope: "open_standard"` MUST have a non-null `registry_url`
- `registry_scope: "catalog_internal"` SHOULD have a null `registry_url` unless the manufacturer chooses to publish

---

## 12. MIME Type

### 12.1 Registered Type

```
Type name: application
Subtype name: vnd.cfoc.cis+json
Required parameters: none
Optional parameters: version (e.g., "0.1.0")
Encoding considerations: UTF-8
Security considerations: Standard JSON security considerations apply
Published specification: https://centerforoffsiteconstruction.org/standards/cis-file-format
Contact: standards@centerforoffsiteconstruction.org
```

### 12.2 File Extension

The `.cis` extension is associated with this MIME type.

---

## 13. Examples

### 13.1 Minimal CIS File

```json
{
  "cis_format_version": "0.1.0",
  "id": "LBS-INT-EXAMPLE",
  "standard_version": "1.0.0",
  "name": "Example Minimal Proprietary Standard",
  "description": "A minimal valid CIS file for documentation purposes.",
  "publisher": {
    "name": "Example Manufacturer",
    "organization_type": "manufacturer",
    "website": "https://example.com"
  },
  "published_date": "2026-04-18",
  "status": "draft",
  "registry_scope": "catalog_internal",
  "registry_url": null,
  "license": "Proprietary",
  "compatibility": {
    "compatible_with": ["1.0.0"],
    "forward_compatible": false
  },
  "sides": {
    "side_a": {
      "description": "First face",
      "envelope": { "nominal_width_in": 12.0, "nominal_height_in": 12.0, "clearance_depth_in": 6.0 },
      "mirror_of": null
    },
    "side_b": {
      "description": "Second face",
      "envelope": { "nominal_width_in": 12.0, "nominal_height_in": 12.0, "clearance_depth_in": 6.0 },
      "mirror_of": "side_a"
    }
  },
  "ports": [],
  "utility_connections": [],
  "structural_connections": []
}
```

### 13.2 Full Open-Standard Example

See `examples/CfOC-ICC-1220-v1.0.0.cis` in the repository for a complete open-standard example with utility and structural connections.

---

## 14. Conformance

### 14.1 Conformance Levels

| Level | Requirements |
|-------|--------------|
| **CIS Reader** | Can parse and validate CIS files against this specification |
| **CIS Writer** | Can produce valid CIS files that pass validation |
| **CIS Registry** | Can host and serve CIS files with integrity verification |

### 14.2 Validation Suite

CfOC will provide a reference validation suite at:
https://github.com/cfoc/cis-validator

---

## 15. Relationship to the CTO File Format

A CIS file is the source of truth for everything about a connection — its sides, ports, utilities, and structural handshake. CTO element files reference CIS files; they do not duplicate CIS content.

### 15.1 How CTO Files Reference CIS Files

A `.cto` element file declares interface conformance through two blocks:

**`declares_interface`** (top-level, required for element files): A list of CIS standards the product conforms to. Each entry names a CIS file by `cis_id`, `cis_version`, and `registry_url`.

**`connectivity.interface_roles`** (within `connectivity`): For each declared interface, which face of the product addresses which side of the CIS, and where within that face the port cluster begins. A product that mirrors an interface across two faces (e.g., a mid-row kitchen pod) declares two entries with the same `cis_id` and different `face` values.

### 15.2 What Stays in the CTO File, What Moves to the CIS File

| Data | Location |
|------|----------|
| Port positions and geometry | **CIS file** |
| Utility requirements (pressure, pipe spec) | **CIS file** |
| Structural bolt specs and capacities | **CIS file** |
| Which face of a product addresses which side of a CIS | **CTO file** (`interface_roles`) |
| Which CIS standards a product conforms to | **CTO file** (`declares_interface`) |
| Product-specific overrides or extensions | **CTO file** (`connectivity.product_extensions`, if needed) |

### 15.3 Version Negotiation at Mating

When the configurator validates a proposed connection between two products, it:
1. Identifies the CIS each product declares for the proposed face pair
2. Confirms both products reference the same `cis_id`
3. Confirms their declared versions have at least one common entry in each other's `compatible_with` arrays
4. Confirms their `side_addressed` values are complementary (one side per side of the CIS)
5. Confirms the computed port alignment is within tolerance

---

## 16. Appendices

### Appendix A: Complete CfOC-ICC-1220 v1.0.0 CIS File

*[Referenced at `examples/CfOC-ICC-1220-v1.0.0.cis`]*

### Appendix B: JSON Schema (machine-readable)

*[To be added in v0.1.1: formal JSON Schema for automated validation]*

### Appendix C: Migration Path from Pre-Spec CIS Files

CIS files authored before this specification existed (such as early `CfOC-ICC-1220-v0.1.1.cis` files) can be migrated to v0.1.0 structure. See `docs/cis-migration-v0.1.0.md` for a step-by-step migration guide.

### Appendix D: Changelog

See `spec/cis/CHANGELOG.md` for the full version history.

### Appendix E: Acknowledgments

This specification was developed by the Center for Offsite Construction in collaboration with Logic Building Systems during the authoring of the first CIS files for real-world catalog use.

---

## Document Information

**Editors:**
- Jason Van Nest, Center for Offsite Construction
- Mathew Ford, Center for Offsite Construction

**Feedback:**
Submit issues and pull requests to https://github.com/Jason-Van-Nest/configurator-file-type-spec

**Copyright:**
© 2026 Center for Offsite Construction. This specification is released under the Apache 2.0 License.
