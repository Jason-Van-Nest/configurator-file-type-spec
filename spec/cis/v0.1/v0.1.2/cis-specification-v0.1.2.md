# CIS File Format Specification

**Version:** 0.1.2
**Status:** Draft
**Published by:** Center for Offsite Construction (CfOC) at NYIT
**Specification URL:** https://centerforoffsiteconstruction.org/offsite-product-configurator-file-type/
**GitHub Repository:** https://github.com/Jason-Van-Nest/configurator-file-type-spec
**License:** Apache 2.0
**Last Updated:** April 18, 2026

---

## Abstract

The CIS (Configure-to-Order Interface Standard) file format is an open standard for encoding connection interface definitions used in offsite construction. A `.cis` file describes a complete connection standard — including its connection plane, the two sides of that plane, port geometry, port connection signatures, utility requirements, structural handshake, and version compatibility — as a machine-readable document that can be referenced by `.cto` element files.

CIS files are the companion format to the CTO file format. Where `.cto` files describe *what a product is and which face addresses which connection standard*, `.cis` files describe *what happens at the connection plane between two products*. A `.cto` element file declares conformance to one or more CIS files; the CIS file carries the detailed connection specification so that element files can remain compact and focused on product-specific data.

Version 0.1.2 replaces the abstract `gender` field on ports with a material-aware `connection_signature` field drawn from a fixed registry (Appendix F). This change reflects the physical reality that connection topology (PEX barb-to-barb via intermediate pipe, copper male-to-female direct thread, PVC slip joints, flange-to-flange via gasket) varies by material and cannot be captured by a single male/female axis. The fixed registry approach forces ecosystem coherence: catalogs cannot invent new signatures; new types must be added to the mini-spec.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [File Structure](#3-file-structure)
4. [Schema Definition](#4-schema-definition)
5. [Connection Planes and Sides](#5-connection-planes-and-sides)
6. [Port Geometry and Connection Signatures](#6-port-geometry-and-connection-signatures)
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

- A canonical definition of a connection plane's two sides, including port layouts and connection signatures
- Geometric coordinates that enable automated validation of connection alignment
- Material-aware connection signatures that match physical reality (PEX, copper, PVC, electrical, data, etc.)
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
| **Port** | A discrete connection point within a side — a single pipe, cable, bolt, or alignment feature, with position, tolerance, and connection signature |
| **Port Cluster** | The complete set of ports on one side of a connection plane, together forming the geometric pattern that the mating side must match |
| **Connection Signature** | A material-aware identifier describing the physical form of a port. Signatures determine what other ports can mate, whether an intermediate connector is required, and whether the connection is gendered (male/female) or non-gendered (barb-to-barb, flange-to-flange). Defined in Appendix F. |
| **Intermediate Connector** | A field-installed piece (PEX pipe, rubber boot, gasket, wire nut) that joins two ports whose connection signature requires it. Specified at the utility connection level |
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
| CfOC-ICC-1220 | CfOC/ICC | Example open-standard CIS for module-to-building-services utility handshaking |
| CfOC-ICC-1220-S | CfOC/ICC | Example open-standard CIS for module-to-building-services structural handshaking (separate from utility) |
| CfOC-ICC-1230 | CfOC/ICC | Example open-standard CIS for panel-to-panel connectivity |

---

## 2. Design Principles

### 2.1 Single Source of Truth for Connection Data

A CIS file is the authoritative description of a connection standard. Element `.cto` files MUST NOT duplicate the interface's port geometry, utility requirements, connection signatures, or structural handshake. They reference the CIS file and trust its contents.

### 2.2 Sides Belong to the Connection Plane, Not the Product

A connection standard exists at the plane between two products. The CIS file describes that plane — its envelope, its port layout, and what is required of each of its two sides. Products do not "own" sides; they address them. A product's face *reaches toward* a connection plane from one side; the complementary product's face reaches toward it from the other side. The CIS file is complete and standalone as a description of the plane.

### 2.3 Geometry and Signatures Enable Validation

Port positions, tolerances, and connection signatures are specified in the CIS file. This enables the configurator to validate — not just that two products claim compatibility, but that their ports actually align when mated and that signatures pair correctly per the registry mating table (Appendix F).

### 2.4 Product-Specific Constraints Live in CTO Files

A CIS file does not know which face of a bath pod is the face that addresses its dwelling-unit-side ports. That is product-specific geometry belonging in the bath pod's `.cto` element file, via the `interface_roles` block. This block is the **rotation constraint** that prevents the configurator from rotating the product to address the CIS plane from the wrong face. See §15.2.

### 2.5 Material-Aware Signatures Reflect Physical Reality

Connection topology varies by material. PEX uses barb-to-barb pairing with an intermediate pipe; copper uses male-to-female threaded or sweated joints; PVC uses slip couplings or rubber boots with band clamps; electrical uses wire nuts, lugs, conduit threads, or plug-and-jack interfaces. The `connection_signature` field captures these distinctions explicitly rather than forcing them through a generic male/female abstraction.

### 2.6 Fixed Registry Forces Ecosystem Coherence

The set of valid `connection_signature` values is fixed in this specification (Appendix F). CIS files MUST NOT use unrecognized signatures. New signatures are added through CIS mini-spec MINOR version bumps; signatures are removed through MAJOR version bumps. This discipline prevents catalog fragmentation where each manufacturer invents its own naming for the same physical connections.

### 2.7 Open and Proprietary Standards Coexist

A CIS file can describe an open standard published by CfOC or a proprietary standard owned by a single manufacturer. The file format is the same; the `registry_scope` field distinguishes them.

### 2.8 Versioning Enables Evolution

Connection standards evolve. The CIS format encodes version history, backward-compatibility rules, and the superseded-by relationship so that element files can reason about which versions they support. CIS standards version independently from both the CTO spec and the CIS mini-spec. See §9.

### 2.9 Utilities and Structure Are Both In Scope

A real connection standard governs the utility handshake (pipes, cables, ducts) and/or the structural handshake (bolts, alignment pins, load transfer surfaces). A given CIS may cover one, the other, or both. Splitting utility and structural concerns into separate CIS files is permitted and often preferable; see §1.5 for the CfOC-ICC-1220 / CfOC-ICC-1220-S example.

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
  "cis_format_version": "0.1.2",
  "id": "CfOC-ICC-1220",
  "standard_version": "0.2.0",
  "name": "Module-to-Building-Services Utility Interface",
  "description": "Standard for volumetric module utility handshaking with site-supplied or panel-embedded utility couplers. Structural handshake handled separately by CfOC-ICC-1220-S.",
  "publisher": { },
  "published_date": "2026-04-18",
  "status": "draft",
  "registry_scope": "open_standard",
  "registry_url": "https://centerforoffsiteconstruction.org/standards/CfOC-ICC-1220-v0.2.0.cis",
  "license": "Apache-2.0",
  "checksum": "sha256:...",
  "supersedes": "CfOC-ICC-1220@0.1.1",
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
"cis_format_version": "0.1.2"
```

The version of the CIS specification this file conforms to. Parsers MUST reject files with a major version they do not support. Files declaring a `cis_format_version` of `0.1.2` or later use the `connection_signature` model (§6); files declaring `0.1.0` or `0.1.1` use the deprecated `gender` model.

### 4.2 `id` (REQUIRED)

```json
"id": "CfOC-ICC-1220"
```

A unique, stable identifier for the connection standard. For open standards published through CfOC, the format is `CfOC-{organization}-{number}` (e.g., `CfOC-ICC-1220`). For catalog-internal proprietary standards, the format is `{manufacturer}-INT-{upstream}-{downstream}` (e.g., `LBS-INT-BATH-UTIL`).

See §10 for registry scope and naming conventions.

### 4.3 `standard_version` (REQUIRED)

```json
"standard_version": "0.2.0"
```

The version of the connection standard itself, following Semantic Versioning 2.0.0. Versions independently of the CTO spec and the CIS mini-spec (see §9).

### 4.4 `name` (REQUIRED)

A human-readable name for the standard.

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

### 4.7 `published_date` (REQUIRED)

ISO 8601 date string for when this version was published.

### 4.8 `status` (REQUIRED)

One of: `draft`, `published`, `deprecated`, `withdrawn`.

### 4.9 `registry_scope` (REQUIRED)

One of: `open_standard`, `catalog_internal`. See §10.

### 4.10 `registry_url` (CONDITIONAL)

REQUIRED when `registry_scope` is `open_standard`. The canonical URL where this exact version of the CIS file can be fetched.

### 4.11 `license` (REQUIRED)

SPDX license identifier (e.g., `Apache-2.0`, `CC-BY-4.0`, `Proprietary`).

### 4.12 `checksum` (RECOMMENDED)

A content hash (e.g., `sha256:...`) that CTO files can use to verify they are referencing an exact, unmodified version of the standard.

### 4.13 `supersedes` / `superseded_by` (OPTIONAL)

References to previous or successor versions as CIS ID strings including version (e.g., `"CfOC-ICC-1220@0.1.1"`).

### 4.14 `compatibility` (REQUIRED)

```json
"compatibility": {
  "compatible_with": ["0.2.0"],
  "breaking_changes_from": "0.1.1",
  "forward_compatible": false,
  "description": "v0.2.0 is the first version conforming to CIS mini-spec v0.1.2 (connection_signature model). Not backward-compatible with 0.1.x files using the deprecated gender model."
}
```

---

## 5. Connection Planes and Sides

### 5.1 The Connection Plane

A CIS file describes one connection plane. At this plane, two products meet by reaching toward it with their respective faces. The plane has two sides — each with its own required envelope and port layout. A product's face addresses exactly one side of the plane.

Sides are properties of the plane, not of products. Two products at the same plane address opposite sides of it.

### 5.2 The `sides` Block (REQUIRED)

Every CIS file MUST define exactly two sides.

```json
"sides": {
  "dwelling_unit_side": {
    "description": "Side of the plane addressed by a volumetric dwelling module (pod). Ports on this side face into the dwelling.",
    "envelope": {
      "nominal_width_in": 6.0,
      "nominal_height_in": 92.5,
      "clearance_depth_in": 14.0,
      "description": "The face envelope is the rectangular zone on the side within which all ports must reside. Clearance depth is the volume behind the side that must be kept free of obstructions for installation."
    },
    "mirror_of": null,
    "notes": "Port positions on this side are authoritative; the building_services_side mirrors them per §6.4."
  },
  "building_services_side": {
    "description": "Side of the plane addressed by building services \u2014 either via a site-built utility stub or via a Logic Coupler product.",
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

Side names are defined by the standard and scoped to the CIS file. Common patterns:

- **Role-asymmetric**: `dwelling_unit_side` / `building_services_side`
- **Product-specific**: `bath_pod_side` / `utility_pod_side`
- **Role-symmetric**: `side_a` / `side_b`

Side names MUST be stable across versions of a given standard. Renaming a side constitutes a breaking change.

### 5.4 Side Names Are Scoped to the CIS File

Side names are unique only within a single CIS file, not globally. Two different CIS files may both declare a `bath_pod_side` and they mean different things. CTO files reference both `cis_id` AND `side_addressed` in every `interface_roles` entry to disambiguate.

### 5.5 The `mirror_of` Field

When one side's port layout is the geometric mirror image of the other side, the mirrored side's `mirror_of` field points at the authoritative side. Ports defined on the authoritative side are automatically mirrored onto the `mirror_of` side during validation, with **complementary connection signatures** per §6.4.

When sides have independent port layouts, `mirror_of` is null and each side declares its own ports.

---

## 6. Port Geometry and Connection Signatures

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
    "connection_signature": "pex_barb",
    "description": "Cold water supply port on the dwelling unit side. Position is measured from the bottom-left corner of the side envelope."
  },
  {
    "port_id": "structural_bolt_upper_left",
    "side": "dwelling_unit_side",
    "position_in": { "x": 0.5, "y": 90.0 },
    "tolerance_in": { "x": 0.125, "y": 0.125 },
    "category": "structural",
    "structural_connection_ref": "primary_shear_bolts",
    "connection_signature": "machine_bolt_through_F",
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
| `connection_signature` | string | Yes | A value from the registry in Appendix F. Determines what other ports may mate with this one (see §6.5) |

### 6.2 Connection Signature Semantics

A connection signature describes the physical form of a port. Signatures fall into three patterns:

**Same-to-same with intermediate** (e.g., `pex_barb`, `pvc_rubber_boot`, `electrical_wire_nut`)
Both ports have the same signature. They mate via a field-installed intermediate piece (PEX pipe, rubber boot, wire nut). The intermediate piece is described in the utility connection's `intermediate_connector` field.

**Same-to-same direct** (e.g., `flange`)
Both ports have the same signature. They mate directly with the help of bolts and a gasket (specified at the utility or structural connection level), with no flexible intermediate.

**Gendered direct mating** (e.g., `copper_thread_M` / `copper_thread_F`, `rj45_plug` / `rj45_jack`)
Ports have paired signatures with `_M`/`_F` suffix or named gendered variants. Male mates with female directly.

The signature alone determines what can mate. A `pex_barb` port can ONLY mate with another `pex_barb` port. A `copper_thread_M` port can ONLY mate with a `copper_thread_F` port. Mating compatibility is read from the registry mating table (Appendix F), not inferred.

### 6.3 Coordinate System

Port positions are specified in **product-local inches** with the origin at the bottom-left corner of the side envelope, as viewed by someone facing the side. X increases rightward, Y increases upward.

### 6.4 Mirror Geometry and Signature

When a side has `mirror_of: "other_side_name"`, its ports are derived by mirroring the other side's ports across the vertical centerline of the envelope, with appropriately complementary signatures. If the envelope width is W:

- **Same-to-same signatures** (e.g., `pex_barb`, `flange`, `pvc_rubber_boot`): a port at `(x, y)` with signature `S` on the authoritative side appears at `(W - x, y)` with signature `S` on the mirrored side
- **Gendered direct signatures** (e.g., `copper_thread_M`, `rj45_plug`): a port at `(x, y)` with signature `X_M` on the authoritative side appears at `(W - x, y)` with signature `X_F` on the mirrored side, and vice versa

The CIS file SHOULD NOT re-declare mirrored ports; validators compute them automatically from the registry mating table.

### 6.5 Port-Pair Mating Rules

When two products are mated at a connection plane, the configurator validates each port pair independently:

1. **Port correspondence**: each port on one side MUST correspond to exactly one port on the other side, based on position within tolerance
2. **Signature compatibility**: the two paired ports MUST have a registered mating per Appendix F. Same-to-same signatures must match exactly; gendered signatures must pair `_M` with `_F`
3. **Utility consistency**: if both ports reference a utility connection, the connections MUST match (same `utility_id`)
4. **Category consistency**: paired ports MUST have the same `category`

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
    },
    "intermediate_connector": {
      "type": "pex_pipe",
      "spec": "1 inch PEX B",
      "spec_standard": "ASTM F876",
      "length_variable": true,
      "length_max_in": 24,
      "termination_method": "crimp ring at each end",
      "description": "A length of 1 inch PEX B pipe joins the two pex_barb ports. Length is field-determined based on the gap between the two products' faces."
    }
  }
]
```

Each utility connection is referenced by one or more ports via the `utility_connection_ref` field. The `intermediate_connector` field is REQUIRED when the utility's ports use a same-to-same-with-intermediate signature (per §6.2 and Appendix F); OPTIONAL otherwise.

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
    "quantity_per_connection": 4
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
- **MAJOR**: Breaking changes — port positions moved, sides renamed, utilities removed, connection signatures changed incompatibly
- **MINOR**: Backward-compatible additions
- **PATCH**: Clarifications and corrections

### 9.2 Independent Versioning

CIS standards version independently of both the CTO spec and the CIS mini-spec. Each CIS file carries two version numbers — `cis_format_version` (which version of the CIS mini-spec the file conforms to) and `standard_version` (the version of the standard itself). These are independent.

### 9.3 Compatibility Negotiation

When a CTO file references a CIS file at version X, it is declaring conformance to **any version in X's `compatible_with` array**. The configurator performs compatibility negotiation by checking whether the versions declared by the two mating products have a common entry in their respective `compatible_with` arrays.

### 9.4 Deprecation

A standard marked `status: "deprecated"` MAY still be used but SHOULD be replaced by the version listed in `superseded_by`. Configurators SHOULD emit warnings when a configuration uses deprecated standards.

### 9.5 CIS Mini-Spec Versioning Governs the Connection Signature Registry

The fixed registry of connection signatures (Appendix F) is governed by CIS mini-spec versioning:

- Adding a new signature: MINOR bump (backward-compatible)
- Removing a signature: MAJOR bump (breaking)
- Changing a signature's mating rule: MAJOR bump (breaking)
- Clarifying a signature's description or examples: PATCH bump

CIS files SHOULD use the latest mini-spec version available at authoring time to gain access to the broadest set of registered signatures.

---

## 10. Registry Scope and Naming

### 10.1 Open Standards

`registry_scope: "open_standard"` indicates a standard published through a central authority (typically CfOC) and available for any manufacturer to adopt. Open standards:

- MUST have a populated `registry_url`
- MUST have a public, fetchable canonical version
- MUST use the `CfOC-{organization}-{number}` naming convention
- SHOULD be licensed under a permissive license

### 10.2 Catalog-Internal Standards

`registry_scope: "catalog_internal"` indicates a standard owned by a single manufacturer. Proprietary standards:

- MAY have a null `registry_url`
- MUST use the `{manufacturer}-INT-{upstream}-{downstream}` naming convention
- MAY be licensed `Proprietary`
- MUST still be parseable by any CIS-conformant parser
- Products declaring catalog-internal standards can only mate with other products from the same catalog

### 10.3 Naming Conventions for Proprietary Standards

Proprietary standards use `{manufacturer}-INT-{upstream}-{downstream}`, where "upstream" means the product type closer to the building services and "downstream" means the product type farther from the building services in a services-handoff chain.

Example: `LBS-INT-BATH-UTIL` (Bath upstream, Utility downstream); `LBS-INT-UTIL-KIT` (Utility upstream, Kitchen downstream).

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
- Mirrored sides MUST NOT re-declare ports that would conflict with computed mirror positions
- Tolerance values MUST be non-negative

### 11.3 Sidedness Consistency

- A CIS file MUST declare exactly two sides
- Side names MUST NOT be reused for different roles across versions of the same standard
- Ports MUST be assigned to a declared side

### 11.4 Connection Signature Consistency

- Every port MUST have a `connection_signature` value drawn from the registry in Appendix F. Unrecognized signatures are validation errors
- When a side has `mirror_of: "other"`, signature complementarity MUST be respected per §6.4
- Port-pair mating rules (§6.5) MUST be enforced during mating validation
- Utility connections whose ports use same-to-same-with-intermediate signatures MUST include a populated `intermediate_connector` block

### 11.5 Registry Scope Consistency

- `registry_scope: "open_standard"` MUST have a non-null `registry_url`
- `registry_scope: "catalog_internal"` SHOULD have a null `registry_url` unless the manufacturer chooses to publish
- Proprietary standard IDs SHOULD follow the `{MFG}-INT-{upstream}-{downstream}` naming convention

---

## 12. MIME Type

```
Type name: application
Subtype name: vnd.cfoc.cis+json
Optional parameters: version (e.g., "0.1.2")
Encoding considerations: UTF-8
File extension: .cis
```

---

## 13. Examples

### 13.1 Minimal CIS File

```json
{
  "cis_format_version": "0.1.2",
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

See `examples/CfOC-ICC-1220-v0.2.0.cis` in the repository.

---

## 14. Conformance

| Level | Requirements |
|-------|--------------|
| **CIS Reader** | Can parse and validate CIS files against this specification, including the connection signature registry |
| **CIS Writer** | Can produce valid CIS files using only registered signatures |
| **CIS Registry** | Can host and serve CIS files with integrity verification |

CfOC will provide a reference validation suite at https://github.com/cfoc/cis-validator.

---

## 15. Relationship to the CTO File Format

A CIS file is the source of truth for everything about a connection plane. CTO element files reference CIS files; they do not duplicate CIS content.

### 15.1 How CTO Files Reference CIS Files

A `.cto` element file declares interface conformance through two blocks:

**`declares_interface`** (top-level, required for element files): A list of CIS standards the product conforms to.

**`connectivity.interface_roles`** (within `connectivity`): For each declared interface, which face of the product addresses which side of the CIS plane.

### 15.2 `interface_roles` as a Rotation-Lock Validation Rule

The `interface_roles` block in a `.cto` file is the **primary mechanism preventing invalid product rotations**. A CIS file knows what happens at the connection plane; it does not and cannot know which face of a specific bath pod is the face that addresses the dwelling-unit-side ports. That is product-specific geometry.

**Division of labor:**

| Data | Location | Owner |
|------|----------|-------|
| Port positions and geometry | **CIS file** | Standard writers |
| Port connection signatures | **CIS file** | Standard writers |
| Utility requirements (pressure, pipe spec, ASTM refs) | **CIS file** | Standard writers |
| Intermediate connector specifications | **CIS file** (utility connection level) | Standard writers |
| Structural bolt specs and capacities | **CIS file** | Standard writers |
| Which CIS standards a product conforms to | **CTO file** (`declares_interface`) | Product designer |
| Which face of a product addresses which side of a CIS | **CTO file** (`interface_roles`) | Product designer |

### 15.3 Warnings vs. Errors at Configurator Time

`interface_roles` violations produce **warnings**, not errors, at design time. A configuration containing such warnings is a valid `.cto` file and can be saved. It cannot be exported to manufacturing without resolving the warnings. This aligns with the CTO spec's `validation_state.errors` vs. `validation_state.warnings` distinction.

### 15.4 Version Negotiation at Mating

When the configurator validates a proposed connection between two products, it:
1. Identifies the CIS each product declares for the proposed face pair
2. Confirms both products reference the same `cis_id`
3. Confirms their declared versions have at least one common entry in each other's `compatible_with` arrays
4. Confirms their `side_addressed` values are complementary
5. Confirms port alignment is within tolerance AND signatures mate per Appendix F

---

## 16. Appendices

### Appendix A: Complete CfOC-ICC-1220 v0.2.0 CIS File

*[Referenced at `examples/CfOC-ICC-1220-v0.2.0.cis`]*

### Appendix B: JSON Schema (machine-readable)

*[To be added in v0.2.0: formal JSON Schema for automated validation]*

### Appendix C: Migration Path from v0.1.x to v0.1.2

CIS files authored against mini-spec v0.1.0 or v0.1.1 used the `gender` field, which v0.1.2 replaces with `connection_signature`. Migration steps:

1. For each port, identify its physical material and connection style
2. Look up the appropriate signature in the Appendix F registry
3. Replace the `gender` field with `connection_signature`
4. If the port previously had `gender: "male"` for a barb-style fitting, the new signature is likely `pex_barb` (NOT `pex_M`); confirm via the registry
5. If the port previously had `gender: "neutral"`, the new signature is likely `flange` or a similar same-to-same-direct
6. Update `cis_format_version` to `"0.1.2"`
7. For utilities whose new signatures are same-to-same-with-intermediate, add an `intermediate_connector` block

### Appendix D: Changelog

Summary of changes from v0.1.1:

- §1.3: replaced "Gender" terminology with "Connection Signature" and "Intermediate Connector"
- §2.5, §2.6: added principles on material-aware signatures and fixed-registry coherence
- §6.1: replaced `gender` field with `connection_signature` field
- §6.2: rewrote semantics around three signature patterns (same-with-intermediate, same-direct, gendered-direct)
- §6.4: updated mirror rules to use signature complementarity per the registry
- §6.5: updated mating rules to look up the registry table
- §7.1: added `intermediate_connector` block on utility connections
- §9.5: added rule that signature registry is governed by CIS mini-spec versioning
- §11.4: replaced gender-consistency rules with connection-signature-consistency rules
- §15.2: updated division-of-labor table to reflect signatures
- Appendix F: added the connection signature registry

### Appendix E: Acknowledgments

This specification was developed by the Center for Offsite Construction in collaboration with Logic Building Systems during the authoring of the first CIS files for real-world catalog use.

### Appendix F: Connection Signature Registry

This is the authoritative registry of `connection_signature` values. CIS files MUST use only values listed here. New signatures are added through MINOR version bumps of this mini-spec; signatures are removed through MAJOR version bumps.

#### F.1 Plumbing — Water Supply

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `pex_barb` | `pex_barb` | Yes — PEX pipe with crimp ring at each end | PEX water supply via barbed brass fitting (ASTM F1807) |
| `copper_thread_M` | `copper_thread_F` | No — direct thread | Male NPT or NPS threaded copper supply fitting |
| `copper_thread_F` | `copper_thread_M` | No — direct thread | Female NPT or NPS threaded copper supply fitting |
| `copper_sweat_M` | `copper_sweat_F` | No — soldered | Male copper supply requiring sweat soldering |
| `copper_sweat_F` | `copper_sweat_M` | No — soldered | Female copper supply socket requiring sweat soldering |
| `cpvc_thread_M` | `cpvc_thread_F` | No — direct thread (PTFE tape recommended) | Male threaded CPVC supply fitting |
| `cpvc_thread_F` | `cpvc_thread_M` | No — direct thread | Female threaded CPVC supply fitting |

#### F.2 Plumbing — DWV (Drain, Waste, Vent)

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `pvc_slip_M` | `pvc_slip_F` | No — solvent cement | Male PVC drain pipe end for slip joint |
| `pvc_slip_F` | `pvc_slip_M` | No — solvent cement | Female PVC drain hub for slip joint |
| `pvc_rubber_boot` | `pvc_rubber_boot` | Yes — rubber fitting with stainless band clamps at each end | PVC drain pipe end designed for rubber boot connection (Fernco-style) |
| `cast_iron_hub_M` | `cast_iron_hub_F` | No — lead/oakum or compression gasket | Cast iron drain pipe spigot for hub-and-spigot connection |
| `cast_iron_hub_F` | `cast_iron_hub_M` | No — lead/oakum or compression gasket | Cast iron drain hub for hub-and-spigot connection |
| `pvc_thread_M` | `pvc_thread_F` | No — direct thread (PTFE tape recommended) | Male threaded PVC drain fitting |
| `pvc_thread_F` | `pvc_thread_M` | No — direct thread | Female threaded PVC drain fitting |

#### F.3 Gas

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `gas_thread_M` | `gas_thread_F` | No — direct thread with gas-rated thread sealant | Male NPT gas supply fitting (black iron or galvanized) |
| `gas_thread_F` | `gas_thread_M` | No — direct thread with gas-rated thread sealant | Female NPT gas supply fitting |
| `csst_fitting_M` | `csst_fitting_F` | No — manufacturer-specific compression | Male corrugated stainless steel tubing fitting |
| `csst_fitting_F` | `csst_fitting_M` | No — manufacturer-specific compression | Female corrugated stainless steel tubing fitting |

#### F.4 Electrical

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `electrical_lug_M` | `electrical_lug_F` | No — terminal lug | Male electrical terminal lug |
| `electrical_lug_F` | `electrical_lug_M` | No — terminal lug | Female electrical terminal lug |
| `electrical_wire_nut` | `electrical_wire_nut` | Yes — wire nut or Wago lever connector | Bare wire end designed for twist-on or lever-action splice |
| `electrical_conduit_M` | `electrical_conduit_F` | No — threaded conduit (EMT, rigid, IMC) | Male threaded conduit end |
| `electrical_conduit_F` | `electrical_conduit_M` | No — threaded conduit | Female threaded conduit coupling |
| `nema_5-15_M` | `nema_5-15_F` | No — direct plug | Standard 120V 15A residential plug |
| `nema_5-15_F` | `nema_5-15_M` | No — direct plug | Standard 120V 15A residential receptacle |
| `nema_l14-30_M` | `nema_l14-30_F` | No — direct twist-lock plug | 240V 30A twist-lock plug |
| `nema_l14-30_F` | `nema_l14-30_M` | No — direct twist-lock plug | 240V 30A twist-lock receptacle |

#### F.5 Data and Communications

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `rj45_jack` | `rj45_plug` | No — direct plug | RJ45 Ethernet jack |
| `rj45_plug` | `rj45_jack` | No — direct plug | RJ45 Ethernet plug |
| `rj11_jack` | `rj11_plug` | No — direct plug | RJ11 telephone jack |
| `rj11_plug` | `rj11_jack` | No — direct plug | RJ11 telephone plug |
| `coax_f_M` | `coax_f_F` | No — direct thread | Male F-type coaxial connector |
| `coax_f_F` | `coax_f_M` | No — direct thread | Female F-type coaxial connector |

#### F.6 HVAC

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `hvac_duct_collar_M` | `hvac_duct_collar_F` | No — slip with foil tape and screws | Male round duct collar |
| `hvac_duct_collar_F` | `hvac_duct_collar_M` | No — slip with foil tape and screws | Female round duct collar |
| `hvac_flex_clamp` | `hvac_flex_clamp` | Yes — length of flex duct with worm-gear clamps | Rigid duct stub designed for flex duct connection |
| `hvac_flange` | `hvac_flange` | Yes — gasket and sheet metal screws | Rectangular duct flange |

#### F.7 Structural

| Signature | Mates With | Intermediate Required | Description |
|-----------|------------|-----------------------|-------------|
| `flange` | `flange` | Yes — gasket and bolts (specified at structural connection level) | Generic flange-to-flange mating with gasket |
| `machine_bolt_through_M` | `machine_bolt_through_F` | No — direct insertion (washer and nut on opposite side) | Through-bolt designed for nut on the far side |
| `machine_bolt_through_F` | `machine_bolt_through_M` | No — direct insertion | Through-hole designed to receive a through-bolt |
| `machine_bolt_threaded_M` | `machine_bolt_threaded_F` | No — direct thread (Loctite or torque-to-spec) | Male machine bolt for threaded receiver |
| `machine_bolt_threaded_F` | `machine_bolt_threaded_M` | No — direct thread | Female threaded receiver for machine bolt |
| `bearing_surface` | `bearing_surface` | No — direct contact under load | Flat bearing surface for compression load transfer |
| `alignment_pin_M` | `alignment_pin_F` | No — direct insertion (slip fit) | Male alignment pin |
| `alignment_pin_F` | `alignment_pin_M` | No — direct insertion | Female alignment hole |
| `welded_joint` | `welded_joint` | Yes — weld bead per AWS specification | Surfaces designed for field welding |

#### F.8 Notes on Using This Registry

- Every signature listed mates with the value in its "Mates With" column. No other mating combinations are valid
- "Intermediate Required: Yes" means the utility connection's `intermediate_connector` block MUST be populated
- For signatures not listed in this registry: the signature is invalid in v0.1.2. Submit a registry expansion request for inclusion in a future MINOR version of the mini-spec
- This registry will expand over time. Authors SHOULD use the latest mini-spec version available at authoring time

---

## Document Information

**Editors:**
- Jason Van Nest, Center for Offsite Construction
- Mathew Ford, Center for Offsite Construction

**Feedback:**
Submit issues and pull requests to https://github.com/Jason-Van-Nest/configurator-file-type-spec

**Copyright:**
© 2026 Center for Offsite Construction. This specification is released under the Apache 2.0 License.
