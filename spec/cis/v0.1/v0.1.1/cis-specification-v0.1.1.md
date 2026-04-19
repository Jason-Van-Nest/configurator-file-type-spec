# CIS File Format Specification

**Version:** 0.1.1
**Status:** Draft
**Published by:** Center for Offsite Construction (CfOC) at NYIT
**Specification URL:** https://centerforoffsiteconstruction.org/offsite-product-configurator-file-type/
**GitHub Repository:** https://github.com/Jason-Van-Nest/configurator-file-type-spec
**License:** Apache 2.0
**Last Updated:** April 18, 2026

---

## Abstract

The CIS (Configure-to-Order Interface Standard) file format is an open standard for encoding connection interface definitions used in offsite construction. A `.cis` file describes a complete connection standard — including its connection plane, the two sides of that plane, port geometry and gendering, utility requirements, structural handshake, and version compatibility — as a machine-readable document that can be referenced by `.cto` element files.

CIS files are the companion format to the CTO file format. Where `.cto` files describe *what a product is and which face addresses which connection standard*, `.cis` files describe *what happens at the connection plane between two products*. A `.cto` element file declares conformance to one or more CIS files; the CIS file carries the detailed connection specification so that element files can remain compact and focused on product-specific data.

Version 0.1.1 refines v0.1.0 by reframing sides as properties of the connection plane (not the product), establishing port-level gender, codifying the upstream-to-downstream naming convention for proprietary standards, and separating the roles of CIS files (defining the standard) from CTO `interface_roles` blocks (enforcing product-specific rotation constraints).

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [File Structure](#3-file-structure)
4. [Schema Definition](#4-schema-definition)
5. [Connection Planes and Sides](#5-connection-planes-and-sides)
6. [Port Geometry and Gender](#6-port-geometry-and-gender)
7. [Utility Connections](#7-utility-connections)
8. [Structural Connections](#8-structural-connections)
9. [Versioning and Compatibility](#9-versioning-and-compatibility)
10. [Registry Scope and Naming](#10-registry-scope-and-naming)
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

- A canonical definition of a connection plane's two sides, including port layouts and gendering
- Geometric coordinates that enable automated validation of connection alignment
- Version compatibility rules that govern which products can mate with which
- A distinction between open (publicly registered) and proprietary (catalog-internal) standards
- A reference target for `.cto` element files declaring interface conformance

### 1.2 Scope

This specification covers the structure of individual CIS files. It does not cover the CIS registry, the submission process for new open standards, or the governance of standard versioning at CfOC. Those are addressed in separate documents.

### 1.3 Terminology

| Term | Definition |
|------|------------|
| **Connection Plane** | The abstract surface at which two products meet. A connection standard describes what happens at this plane; it exists in the space *between* products, not within either of them |
| **Connection Standard** | A specification defining what a connection plane looks like — its port layout, utility requirements, structural handshake, and geometric envelope |
| **CIS File** | A `.cis` file that documents a connection standard in machine-readable form |
| **Side** | One of the two faces of a connection plane. A product's face *addresses* a side by reaching toward the plane from that side. Sides are properties of the CIS file (not the product) |
| **Port** | A discrete connection point within a side — a single pipe, cable, bolt, or alignment feature, with position, tolerance, and gender |
| **Port Cluster** | The complete set of ports on one side of a connection plane, together forming the geometric pattern that the mating side must match |
| **Gender** | The physical directional orientation of a port: `male` (protruding feature), `female` (receiving feature), `neutral` (flange, bearing surface, butt joint), or `androgynous` (hermaphroditic couplings) |
| **Registry Scope** | Whether a CIS file is `open_standard` (publicly registered with a central authority such as CfOC) or `catalog_internal` (proprietary to a single manufacturer's catalog) |
| **Publisher** | The entity that authored and maintains the standard |
| **Mating** | The act of joining two products whose faces address complementary sides of the same CIS at a shared connection plane |
| **Upstream / Downstream** | In a services chain (building services → pod → pod → pod), "upstream" means closer to the building services; "downstream" means farther. Used in proprietary standard naming conventions (see §10.3) |

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

A CIS file is the authoritative description of a connection standard. Element `.cto` files MUST NOT duplicate the interface's port geometry, utility requirements, gender assignments, or structural handshake. They reference the CIS file and trust its contents.

### 2.2 Sides Belong to the Connection Plane, Not the Product

A connection standard exists at the plane between two products. The CIS file describes that plane — its envelope, its port layout, and what is required of each of its two sides. Products do not "own" sides; they address them. A product's face *reaches toward* a connection plane from one side; the complementary product's face reaches toward it from the other side. The CIS file is complete and standalone as a description of the plane.

### 2.3 Geometry and Gender Enable Validation

Port positions, tolerances, and genders are specified in the CIS file. This enables the configurator to validate — not just that two products claim compatibility, but that their ports actually align when mated and that male ports meet female ports (or neutral meets neutral).

### 2.4 Product-Specific Constraints Live in CTO Files

A CIS file does not know which face of a bath pod is the face that addresses its dwelling-unit-side ports. That is product-specific geometry belonging in the bath pod's `.cto` element file, via the `interface_roles` block. This block is the **rotation constraint** that prevents the configurator from rotating the product to address the CIS plane from the wrong face. See §15.2.

### 2.5 Open and Proprietary Standards Coexist

A CIS file can describe an open standard published by CfOC or a proprietary standard owned by a single manufacturer. The file format is the same; the `registry_scope` field distinguishes them.

### 2.6 Versioning Enables Evolution

Connection standards evolve. The CIS format encodes version history, backward-compatibility rules, and the superseded-by relationship so that element files can reason about which versions they support. CIS standards version independently from both the CTO spec and the CIS mini-spec. See §9.

### 2.7 Utilities and Structure Are Both In Scope

A real connection standard governs both the utility handshake (pipes, cables, ducts) and the structural handshake (bolts, alignment pins, load transfer surfaces). Both are first-class in CIS.

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
  "cis_format_version": "0.1.1",
  "id": "CfOC-ICC-1220",
  "standard_version": "0.2.0",
  "name": "Module-to-Building-Services Utility Interface",
  "description": "Standard for volumetric module handshaking with site-supplied or panel-embedded utility couplers",
  "publisher": { },
  "published_date": "2026-04-18",
  "status": "draft",
  "registry_scope": "open_standard",
  "registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v0.2.0.cis",
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

The `cis_format_version` field tracks which version of *this specification* the file conforms to (analogous to `cto_version` in CTO files). The `standard_version` field tracks the version of the connection standard itself. These two fields version independently (see §9).

---

## 4. Schema Definition

### 4.1 `cis_format_version` (REQUIRED)

```json
"cis_format_version": "0.1.1"
```

The version of the CIS specification this file conforms to. Parsers MUST reject files with a major version they do not support.

### 4.2 `id` (REQUIRED)

```json
"id": "CfOC-ICC-1220"
```

A unique, stable identifier for the connection standard. For open standards published through CfOC, the format is `CfOC-{organization}-{number}` (e.g., `CfOC-ICC-1220`). For catalog-internal proprietary standards, the format is `{manufacturer}-INT-{upstream}-{downstream}` (e.g., `LBS-INT-BATH-UTIL` for the Logic Building Systems internal bath-to-utility standard).

See §10 for registry scope and naming conventions.

### 4.3 `standard_version` (REQUIRED)

```json
"standard_version": "0.2.0"
```

The version of the connection standard itself, following Semantic Versioning 2.0.0. Versions independently of the CTO spec and the CIS mini-spec (see §9).

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

See §10 for details.

### 4.10 `registry_url` (CONDITIONAL)

REQUIRED when `registry_scope` is `open_standard`. OPTIONAL (and typically null) when `catalog_internal`. The canonical URL where this exact version of the CIS file can be fetched.

### 4.11 `license` (REQUIRED)

SPDX license identifier (e.g., `Apache-2.0`, `CC-BY-4.0`, `Proprietary`).

### 4.12 `checksum` (RECOMMENDED)

A content hash (e.g., `sha256:...`) that CTO files can use to verify they are referencing an exact, unmodified version of the standard.

### 4.13 `supersedes` / `superseded_by` (OPTIONAL)

References to previous or successor versions as CIS ID strings including version (e.g., `"CfOC-ICC-1220@0.1.1"`).

### 4.14 `compatibility` (REQUIRED)

```json
"compatibility": {
  "compatible_with": ["0.2.0", "0.2.1"],
  "breaking_changes_from": "0.1.1",
  "forward_compatible": false,
  "description": "This version is wire-compatible with 0.2.x. Version 1.0.0 will introduce breaking changes and is not backward-compatible."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `compatible_with` | array of semver strings | Yes | Which other versions of this standard products conforming to this file can mate with |
| `breaking_changes_from` | semver string | No | The earliest version before which there are known breaking incompatibilities |
| `forward_compatible` | boolean | Yes | Whether products conforming to this version can safely mate with future minor versions |

---

## 5. Connection Planes and Sides

### 5.1 The Connection Plane

A CIS file describes one connection plane. At this plane, two products meet by reaching toward it with their respective faces. The plane has two sides — each with its own required envelope, port layout, and gendering. A product's face addresses exactly one side of the plane.

Sides are properties of the plane, not of products. Two products at the same plane address opposite sides of it.

### 5.2 The `sides` Block (REQUIRED)

Every CIS file MUST define exactly two sides.

```json
"sides": {
  "dwelling_unit_side": {
    "description": "Side of the plane that is addressed by a volumetric dwelling module (pod). Ports on this side face into the dwelling.",
    "envelope": {
      "nominal_width_in": 6.0,
      "nominal_height_in": 92.5,
      "clearance_depth_in": 14.0,
      "description": "The face envelope is the rectangular zone on the side within which all ports must reside. Clearance depth is the volume behind the side that must be kept free of obstructions for installation."
    },
    "mirror_of": null,
    "notes": "Port positions on this side are authoritative; the building_services_side mirrors them (see §6.3)."
  },
  "building_services_side": {
    "description": "Side of the plane that is addressed by building services \u2014 either via a site-built utility stub or via a Logic Coupler product.",
    "envelope": {
      "nominal_width_in": 6.0,
      "nominal_height_in": 92.5,
      "clearance_depth_in": 14.0
    },
    "mirror_of": "dwelling_unit_side"
  }
}
```

### 5.3 Side Naming Conventions

Side names are defined by the standard and scoped to the CIS file. Three common patterns:

- **Role-asymmetric** (one side is a services "producer," the other a "consumer"): name sides by role, e.g., `dwelling_unit_side` / `building_services_side`. Common for open standards bridging pods to building infrastructure.
- **Product-specific** (sides named for the product types that address them): e.g., `bath_pod_side` / `utility_pod_side`. Common for proprietary standards that exist to govern a specific pod-to-pod connection.
- **Role-symmetric** (either side can play either role): use `side_a` / `side_b`. Rare; used when the interface is genuinely reversible.
- **Physically gendered at the side level** (all ports on one side are male, all on the other female): use `male` / `female`. Also rare; typically gender varies per-port, not per-side.

Side names MUST be stable across versions of a given standard. Renaming a side constitutes a breaking change.

### 5.4 Side Names Are Scoped to the CIS File

Side names are unique only within a single CIS file, not globally. Two different CIS files may both declare a `bath_pod_side` and they mean different things — one describes the bath pod's interface to a 1220 plane, the other describes the bath pod's interface to a BATH-UTIL plane. When a product addresses two different CIS files with similarly named sides, the `cis_id` in the `interface_roles` entry disambiguates; there is no collision.

This is why CTO files reference both `cis_id` AND `side_addressed` in every `interface_roles` entry. Neither alone is sufficient.

### 5.5 The `mirror_of` Field

When one side's port layout is the geometric mirror image of the other side (typical for pipe-to-pipe connections where two opposing faces must have matching ports in reversed gender), the mirrored side's `mirror_of` field points at the authoritative side. Ports defined on the authoritative side are automatically mirrored onto the `mirror_of` side during validation, **with complementary gender**: a male port at (x, y) on the authoritative side becomes a female port at (W - x, y) on the mirrored side, and vice versa. See §6.4.

When sides have independent port layouts (typical for structural connections where, e.g., one side has bolts and the other has bolt-receiving holes that are not geometrically mirror-symmetric), `mirror_of` is null and each side declares its own ports.

---

## 6. Port Geometry and Gender

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
    "gender": "female",
    "description": "Cold water supply port on the dwelling unit side. Position is measured from the bottom-left corner of the side envelope."
  },
  {
    "port_id": "structural_bolt_upper_left",
    "side": "dwelling_unit_side",
    "position_in": { "x": 0.5, "y": 90.0 },
    "tolerance_in": { "x": 0.125, "y": 0.125 },
    "category": "structural",
    "structural_connection_ref": "primary_shear_bolts",
    "gender": "female",
    "description": "Upper-left structural connection \u2014 receiving hole for 3/4 inch A325 bolt."
  }
]
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `port_id` | string | Yes | Unique identifier within this CIS file |
| `side` | string | Yes | Which side of the connection plane this port is on |
| `position_in` | object | Yes | Position in inches within the side's envelope, measured from the bottom-left corner |
| `tolerance_in` | object | Yes | Acceptable deviation in inches |
| `category` | enum | Yes | `utility`, `structural`, `alignment`, `data` |
| `utility_connection_ref` | string | Conditional | REQUIRED when `category: "utility"` — points to an entry in `utility_connections[]` |
| `structural_connection_ref` | string | Conditional | REQUIRED when `category: "structural"` — points to an entry in `structural_connections[]` |
| `direction` | enum | Conditional | For utility ports: `ingress`, `egress`, `bidirectional` — flow direction from the perspective of this side |
| `gender` | enum | Yes | `male`, `female`, `neutral`, or `androgynous` (see §6.2) |

### 6.2 Gender Semantics

Gender describes the physical directional orientation of a port and determines what can mate with it:

| Value | Physical Meaning | Mating Rule | Examples |
|-------|-----------------|-------------|----------|
| `male` | Protruding feature | Mates with `female` or `androgynous` | Bolt, pin, plug, barb fitting, PEX barb |
| `female` | Receiving feature | Mates with `male` or `androgynous` | Threaded receiver, socket, jack, hub fitting, sanitary hub |
| `neutral` | Non-directional mating feature | Mates with `neutral` only | Flange-to-flange, bearing surface, butt joint |
| `androgynous` | Hermaphroditic feature | Mates with any gender | Specialized industrial couplings (rare) |

Gender is determined by the physical reality of the connection and is authored into the CIS file by the standard writer. It is not product-specific; a CIS file fully defines port gender, and products simply inherit it by reference.

### 6.3 Coordinate System

Port positions are specified in **product-local inches** with the origin at the bottom-left corner of the side envelope, as viewed by someone facing the side. X increases rightward, Y increases upward.

### 6.4 Mirror Geometry and Gender

When a side has `mirror_of: "other_side_name"`, its ports are derived by mirroring the other side's ports across the vertical centerline of the envelope, with complementary gender. If the envelope width is W:

- A port at `(x, y)` with gender `male` on the authoritative side appears at `(W - x, y)` with gender `female` on the mirrored side
- A port at `(x, y)` with gender `female` on the authoritative side appears at `(W - x, y)` with gender `male` on the mirrored side
- A port at `(x, y)` with gender `neutral` on the authoritative side appears at `(W - x, y)` with gender `neutral` on the mirrored side
- A port at `(x, y)` with gender `androgynous` on the authoritative side appears at `(W - x, y)` with gender `androgynous` on the mirrored side

The CIS file SHOULD NOT re-declare mirrored ports; validators compute them automatically.

### 6.5 Port-Pair Mating Rules

When two products are mated at a connection plane, the configurator validates each port pair independently:

1. **Port correspondence**: each port on one side MUST correspond to exactly one port on the other side, based on position within tolerance
2. **Gender compatibility**: the two paired ports MUST have compatible genders per §6.2
3. **Utility consistency**: if both ports reference a utility connection, the connections MUST match (same `utility_id`)
4. **Category consistency**: paired ports MUST have the same `category` (a structural port cannot mate with a utility port)

A configuration in which any port pair fails these rules is invalid at manufacturing-ready status, though it MAY be allowed during design-time exploration as a warning (see §15.3).

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

- **MAJOR**: Breaking changes — port positions moved, sides renamed, utilities removed, gender assignments reversed, dimensions changed incompatibly
- **MINOR**: Backward-compatible additions — new optional ports, expanded tolerance ranges, additional utility types
- **PATCH**: Clarifications and corrections — description text, typo fixes, clarifying notes

### 9.2 Independent Versioning

CIS standards version independently of both the CTO spec and the CIS mini-spec. Each standard begins at `1.0.0` when first published as `status: "published"`, with pre-release drafts using `0.y.z`. Versioning is internal to each standard and has no coordination requirement with any other standard or spec.

This independence exists because:
- CTO spec and CIS standards evolve on different cadences and for different reasons
- Different standards have different publishers and governance processes
- Breaking changes in a CTO spec (file structure) and in a CIS standard (physical reality) affect different consumers

Each CIS file carries two version numbers — `cis_format_version` (which version of the CIS mini-spec the file conforms to) and `standard_version` (the version of the standard itself). These are independent.

### 9.3 Compatibility Negotiation

When a CTO file references a CIS file at version X, it is declaring conformance to **any version in X's `compatible_with` array**. The configurator performs compatibility negotiation by checking whether the versions declared by the two mating products have a common entry in their respective `compatible_with` arrays.

### 9.4 Deprecation

A standard marked `status: "deprecated"` MAY still be used but SHOULD be replaced by the version listed in `superseded_by`. Configurators SHOULD emit warnings when a configuration uses deprecated standards.

---

## 10. Registry Scope and Naming

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
- MUST use the `{manufacturer}-INT-{upstream}-{downstream}` naming convention (see §10.3)
- MAY be licensed `Proprietary`
- MUST still be parseable by any CIS-conformant parser — the file format is the same regardless of scope
- Products declaring catalog-internal standards can only mate with other products from the same catalog

### 10.3 Naming Conventions for Proprietary Standards

Proprietary standards use the pattern `{manufacturer}-INT-{upstream}-{downstream}`, where "upstream" means the product type closer to the building services and "downstream" means the product type farther from the building services in a services-handoff chain.

**Example services chain:**
```
Building services
  → Logic Coupler (1220)
  → Bath Pod
  → Utility Pod
  → Kitchen Pod
```

In this chain, the plane between Bath Pod and Utility Pod is governed by `LBS-INT-BATH-UTIL` (Bath upstream, Utility downstream). The plane between Utility Pod and Kitchen Pod is governed by `LBS-INT-UTIL-KIT` (Utility upstream, Kitchen downstream).

This upstream-to-downstream ordering carries semantic weight: it tells readers of the standard name which direction the services flow. The naming is a documentation convention, not a parser requirement, but catalogs SHOULD follow it consistently.

**Recommended patterns:**

| Pattern | Meaning | Example |
|---------|---------|---------|
| `{MFG}-INT-{upstream}-{downstream}` | Interface between two specific product types in a chain | `LBS-INT-BATH-UTIL`, `LBS-INT-UTIL-KIT` |
| `{MFG}-INT-{scope}` | Scope-wide internal standard (no clear chain direction) | `LBS-INT-STACK` (pod stacking) |
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

### 11.4 Gender Consistency

- Every port MUST have a declared gender
- When a side has `mirror_of: "other"`, gender inversion MUST be respected (see §6.4)
- Port-pair mating rules (§6.5) MUST be enforced during mating validation

### 11.5 Registry Scope Consistency

- `registry_scope: "open_standard"` MUST have a non-null `registry_url`
- `registry_scope: "catalog_internal"` SHOULD have a null `registry_url` unless the manufacturer chooses to publish
- Proprietary standard IDs SHOULD follow the `{MFG}-INT-{upstream}-{downstream}` naming convention

---

## 12. MIME Type

### 12.1 Registered Type

```
Type name: application
Subtype name: vnd.cfoc.cis+json
Required parameters: none
Optional parameters: version (e.g., "0.1.1")
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
  "cis_format_version": "0.1.1",
  "id": "LBS-INT-EXAMPLE",
  "standard_version": "0.1.0",
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
    "compatible_with": ["0.1.0"],
    "forward_compatible": false
  },
  "sides": {
    "side_a": {
      "description": "First face of the connection plane",
      "envelope": { "nominal_width_in": 12.0, "nominal_height_in": 12.0, "clearance_depth_in": 6.0 },
      "mirror_of": null
    },
    "side_b": {
      "description": "Second face of the connection plane",
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

See `examples/CfOC-ICC-1220-v0.2.0.cis` in the repository for a complete open-standard example with utility, structural, and gender-assigned ports.

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

A CIS file is the source of truth for everything about a connection plane — its sides, ports, gender, utilities, and structural handshake. CTO element files reference CIS files; they do not duplicate CIS content.

### 15.1 How CTO Files Reference CIS Files

A `.cto` element file declares interface conformance through two blocks:

**`declares_interface`** (top-level, required for element files): A list of CIS standards the product conforms to. Each entry names a CIS file by `cis_id`, `cis_version`, and `registry_url`. This answers the question "what connection planes does this product touch?"

**`connectivity.interface_roles`** (within `connectivity`): For each declared interface, which face of the product addresses which side of the CIS plane. This answers the question "which specific face of mine reaches toward this plane, and from which side?" A product with a mirrored interface (e.g., a mid-row kitchen pod that can mate from either its left or right face) declares two entries with the same `cis_id` and different `face` values.

### 15.2 `interface_roles` as a Rotation-Lock Validation Rule

The `interface_roles` block in a `.cto` file is the **primary mechanism preventing invalid product rotations** at the configurator. It is not merely informational — it is a product-specific geometric constraint declaring that *only* the listed face(s) of the product carry the CIS-conformant port cluster.

**Why this matters:** A CIS file describes what happens at a connection plane. It does not and cannot know which face of a specific bath pod is the face that addresses the dwelling-unit-side port cluster. That is product-specific geometry. Without `interface_roles`, a configurator could blindly rotate a bath pod 90° and attempt to mate its side face to a Logic Coupler — which would be a manufacturing impossibility because the 1" PEX male coupling on the bath pod is only physically present on the top face. The `interface_roles` block is the rotation constraint that prevents this.

**Division of labor:**

| Data | Location | Owner |
|------|----------|-------|
| Port positions and geometry | **CIS file** | Standard writers |
| Port gender (male/female/neutral/androgynous) | **CIS file** | Standard writers |
| Utility requirements (pressure, pipe spec, ASTM refs) | **CIS file** | Standard writers |
| Structural bolt specs and capacities | **CIS file** | Standard writers |
| Which CIS standards a product conforms to | **CTO file** (`declares_interface`) | Product designer |
| Which face of a product addresses which side of a CIS | **CTO file** (`interface_roles`) | Product designer |
| Product-specific overrides or extensions | **CTO file** (`connectivity.product_extensions`, if needed) | Product designer |

### 15.3 Warnings vs. Errors at Configurator Time

`interface_roles` violations — cases where the configurator observes a user attempting to mate a face that is *not* declared in the product's `interface_roles` to a CIS plane — produce **warnings**, not errors. This reflects the configurator's design philosophy: at design time, users explore possibilities; at manufacturing-ready time, violations must be resolved.

A configuration containing `interface_roles` warnings is a valid `.cto` file and can be saved. It cannot be exported to manufacturing without resolving the warnings (either by rotating the product to use the declared face, or by the user explicitly overriding with a documented rationale).

This warning-not-error philosophy extends to other geometric constraints as well — clearance zone violations, structural load overages, etc. — and aligns with the CTO spec's `validation_state.errors` vs. `validation_state.warnings` distinction (CTO spec §4.10).

### 15.4 Version Negotiation at Mating

When the configurator validates a proposed connection between two products, it:
1. Identifies the CIS each product declares for the proposed face pair
2. Confirms both products reference the same `cis_id`
3. Confirms their declared versions have at least one common entry in each other's `compatible_with` arrays
4. Confirms their `side_addressed` values are complementary (one side per side of the CIS plane)
5. Confirms the computed port alignment is within tolerance AND gender compatibility holds per §6.5

---

## 16. Appendices

### Appendix A: Complete CfOC-ICC-1220 v0.2.0 CIS File

*[Referenced at `examples/CfOC-ICC-1220-v0.2.0.cis`]*

### Appendix B: JSON Schema (machine-readable)

*[To be added in v0.2.0: formal JSON Schema for automated validation]*

### Appendix C: Migration Path from Pre-Spec CIS Files

CIS files authored before this specification existed (such as early `CfOC-ICC-1220-v0.1.1.cis` files that pre-date the CIS mini-spec) can be migrated to v0.1.1 structure. See `docs/cis-migration-v0.1.1.md` for a step-by-step migration guide.

### Appendix D: Changelog

See `spec/cis/CHANGELOG.md` for the full version history. Summary of changes from v0.1.0:

- §2.2, §5: reframed sides as properties of the connection plane, not the product
- §5.4: added explicit rule that side names are scoped to their CIS file
- §6.1, §6.2: added `gender` field on ports with four-value enum
- §6.4: updated mirror-geometry rules to account for gender inversion
- §6.5: added port-pair mating rules including gender compatibility
- §9.2: added the independent-versioning rule
- §10.3: codified the `{MFG}-INT-{upstream}-{downstream}` naming convention with chain-order semantics
- §15.2: reframed `interface_roles` as a rotation-lock validation rule, not just an informational mapping
- §15.3: added warnings-vs-errors philosophy

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
