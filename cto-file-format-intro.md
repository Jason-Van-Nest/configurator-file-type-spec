# CTO File Format

**A Legal and Technical Foundation for Configure-to-Order Construction**

Version 0.1.0 | April 2026

**Center for Offsite Construction**  
School of Architecture & Design  
New York Institute of Technology  
[centerforoffsiteconstruction.org](https://centerforoffsiteconstruction.org)

---

## Abstract

The CTO (Configure-to-Order) file format is an open standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a bounded design — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

This whitepaper presents the rationale, structure, and adoption pathway for the CTO file format. It explains how the format supports the transition from Engineer-to-Order (ETO) to Configure-to-Order (CTO) delivery, documents the legal transition from goods to real property, and provides the foundation for a national marketplace of interoperable building products.

Version 0.1.0 introduces comprehensive product performance attributes, interface standard conformance tracking, and a complete chain-of-custody framework that documents the legal transition from goods (governed by UCC Article 2) to real property (governed by Common Law).

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [The Problem: Why Existing Formats Fall Short](#2-the-problem-why-existing-formats-fall-short)
3. [The Solution: A Configure-to-Order File Format](#3-the-solution-a-configure-to-order-file-format)
4. [File Structure Overview](#4-file-structure-overview)
5. [Product Schema](#5-product-schema)
6. [Chain of Custody and Legal Mateline](#6-chain-of-custody-and-legal-mateline)
7. [Interface Standard Conformance](#7-interface-standard-conformance)
8. [Validation and Constraints](#8-validation-and-constraints)
9. [Use Cases](#9-use-cases)
10. [Path to Adoption](#10-path-to-adoption)
11. [Conclusion](#11-conclusion)

---

## 1. Introduction

The architecture, engineering, and construction (AEC) industry is undergoing a fundamental transformation. After a century of Engineer-to-Order (ETO) delivery — where every building is designed from scratch through bespoke professional services — the industry is moving toward Configure-to-Order (CTO) production, where buildings are assembled from pre-designed, pre-certified, interoperable products.

This transformation mirrors what happened in automotive, aerospace, and electronics manufacturing decades ago. In those industries, the shift from craft production to platform-based manufacturing unlocked exponential productivity gains, predictable quality, and scalable supply chains. Construction is the last major industry to make this transition.

The CTO file format is the digital foundation for this transformation. It provides a standard way to represent, validate, price, and track building configurations composed from catalog products. It embodies the core principle of Configure-to-Order: **design as purchasing, not drafting.**

### 1.1 Purpose of This Document

This whitepaper serves three audiences:

- **Software developers** building configurators, manufacturing execution systems, or supply chain platforms who need to understand the CTO schema
- **Manufacturers** producing offsite construction products who want their catalogs to be CTO-compatible
- **Legal and policy professionals** examining how commercial law (UCC) and property law (Common Law) intersect in offsite construction

---

## 2. The Problem: Why Existing Formats Fall Short

The AEC industry has developed sophisticated digital standards for representing buildings: DWG, RVT, IFC, gbXML, and others. These formats serve their purposes well, but they were designed for an Engineer-to-Order world. They assume that every building is a one-off design, that geometry is authored by professionals, and that the file represents an intent rather than a validated configuration.

### 2.1 Limitations of Traditional Formats

| Aspect | Traditional Formats | CTO Format |
|---|---|---|
| Content | Arbitrary geometry | Catalog product references |
| Validation | Requires human interpretation | Machine-validatable |
| Pricing | Not represented | MSRP embedded |
| Supply Chain | Not integrated | Manufacturing & delivery tracking |
| Legal Status | Ambiguous | Documents goods-to-property transition |

Traditional formats assume that design will be engineered, priced, and scheduled through separate downstream processes. The CTO format assumes that these concerns are resolved at the point of product design, not project design.

### 2.2 The Legal Gap

Perhaps most critically, existing file formats are silent on the legal status of the components they represent. A bathroom pod in a BIM model has geometry, materials, and cost estimates — but nothing indicates whether it is currently a good (governed by UCC Article 2), a fixture, or real property (governed by state property law).

This ambiguity matters because offsite products move through multiple legal regimes before becoming part of a building. They are manufactured as goods, shipped under commercial law, delivered to a jobsite, installed as fixtures, and ultimately become real property. At each transition, questions arise: Who owns the product? Who bears the risk of loss? Which insurance policy responds? When do warranties activate?

The CfOC whitepaper *From Handshake to Hardware* documents these legal challenges in detail. The CTO file format is designed to address them.

---

## 3. The Solution: A Configure-to-Order File Format

The CTO file format is a JSON-based standard that represents building configurations as compositions of catalog products. It is designed to be:

- **Catalog-constrained:** Every element references a `product_id` from a known catalog. Invalid configurations cannot be saved.
- **Machine-validatable:** Constraint satisfaction, interface compatibility, and structural/thermal performance can be verified without human interpretation.
- **Price-transparent:** Every product carries an MSRP. Options add predictably. The total price is always deterministic.
- **Supply-chain-integrated:** The fulfillment plan section tracks manufacturing, delivery, crane lifts, and installation.
- **Legally explicit:** The chain of custody documents ownership, risk, insurance, and legal status at each stage.

### 3.1 Design as Purchasing

The fundamental shift from ETO to CTO is captured in a simple phrase: **design as purchasing, not drafting.**

In an ETO workflow, design is an act of invention. Professionals draw custom geometry, specify custom assemblies, and engineer custom solutions. The result is a set of documents (drawings, specifications, schedules) that must be interpreted by downstream parties.

In a CTO workflow, design is an act of selection. Customers configure their homes from a catalog of pre-designed products. The result is a validated configuration that can flow directly into manufacturing execution.

The CTO file format embodies this shift. It does not represent arbitrary geometry; it represents selections from a bounded design space.

---

## 4. File Structure Overview

A CTO file is UTF-8 encoded JSON with the following top-level structure:

| Section | Purpose |
|---|---|
| `cto_version` | Specification version for compatibility checking |
| `project_meta` | Project identity, author, climate zone, jurisdiction |
| `catalog_reference` | Manufacturer, catalog version, interface standards used |
| `structural_grid` | Coordinate system derived from floor cartridge placement |
| `floor_cartridges` | Structural floor assemblies defining the grid |
| `placed_elements` | Pods, panels, and other products with connections |
| `roof_elements` | Roof assemblies |
| `pricing_snapshot` | Line items, subtotals, and total MSRP |
| `fulfillment_plan` | Manufacturing, delivery, crane lift, installation schedules |
| `validation_state` | Errors, warnings, interface compatibility status |

Each section serves a distinct purpose in the CTO workflow. The design-time sections (`project_meta` through `roof_elements`) represent the configuration itself. The commerce sections (`pricing_snapshot` and `fulfillment_plan`) represent the business and logistics aspects. The `validation_state` section provides machine-readable quality assurance.

---

## 5. Product Schema

Products in CTO-compatible catalogs carry comprehensive metadata organized into several domains:

### 5.1 Identity & Catalog Lineage

Each product has a unique `product_id`, `manufacturer_id`, `catalog_version`, and `product_version`. The `design_line` attribute (e.g., Urban Minimalist, Rustic Retreat, Coastal Breeze) groups products into aesthetic families.

### 5.2 Geometry

Dimensions, weight, center of gravity, bounding box, and clearance zones enable collision detection, structural analysis, and rigging planning.

### 5.3 Connectivity

Connection points declare position, direction, interface standard conformance, and gender. MEP connections specify electrical capacity, plumbing sizes, and HVAC requirements. This enables the configurator to validate that connected products are compatible.

### 5.4 Performance

Structural performance attributes include load capacity, transmitted loads, seismic design category compatibility, and foundation requirements. Thermal performance attributes include R-values, U-factors, air leakage rates, and climate zone compatibility. These enable code compliance validation.

### 5.5 Certifications

Products carry arrays of certifications (ICC-ES, UL, ETL), code compliance declarations (IBC, IRC, IECC, NFPA), fire ratings, accessibility compliance, and state modular program approvals. These attestations travel with the product.

### 5.6 Installation & Warranty

Rigging requirements, installation manuals, commissioning checklists, and warranty terms are embedded in the product schema. This information flows into the fulfillment plan when the product is ordered.

---

## 6. Chain of Custody and Legal Mateline

The chain of custody system is the CTO format's response to the legal challenges documented in *From Handshake to Hardware*. It tracks each product instance from factory release to acceptance, documenting the legal transition from goods to real property.

### 6.1 The Legal Mateline

The **legal mateline** is the boundary at which a product transitions from goods (governed by UCC Article 2) to real property (governed by Common Law). The CTO format explicitly documents this transition through the chain of custody.

| Stage | Legal Status | Description |
|---|---|---|
| Factory Release | Goods | Product cleared from manufacturing floor |
| Transport | Goods | Moving between facilities |
| Delivery | Goods | Arrived at jobsite or staging yard |
| Staging | Goods | Awaiting installation |
| Placement | Goods → Fixture | Set on foundation/structure |
| Connection | Fixture | Attached to adjacent elements |
| Commissioning | Fixture | Systems tested and verified |
| **Acceptance** | **Real Property** | **Mateline crossing; warranties activate** |

### 6.2 What the Chain Documents

At each custody stage, the CTO file records:

- **Timestamp:** When the stage occurred
- **Responsible party:** Who took custody
- **Witness:** Who verified the handoff
- **Condition:** Status, notes, photos
- **Risk owner:** Who bears the risk of loss
- **Title holder:** Who owns the asset
- **Insurance policy:** Which policy responds
- **Legal status:** Goods, fixture, or real property

This documentation provides evidentiary support for disputes, warranty claims, and insurance inquiries.

---

## 7. Interface Standard Conformance

Products in a CTO ecosystem must be interoperable. This requires shared interface standards that define how products connect physically, structurally, and thermally.

### 7.1 CfOC Interface Standards

The Center for Offsite Construction, as an ANSI-accredited standards developer, is developing interface standards for offsite construction:

- **CfOC-ICC-1220:** Module-to-module connectivity
- **CfOC-ICC-1230:** Panel-to-panel connectivity

Products declare which interface standards they conform to at each connection point. The configurator validates that connected products share at least one compatible interface version.

### 7.2 Validation Rules

A connection is valid if and only if:

1. Both products declare the same `interface_standard`
2. The `interface_versions` arrays have at least one version in common
3. The `gender` values are complementary (`male`/`female`) or both `neutral`
4. The `connection_type` values match

---

## 8. Validation and Constraints

The CTO format is designed to be machine-validatable. A conforming parser enforces multiple categories of validation rules:

**Referential Integrity**  
Every `product_id` must exist in the referenced catalog. Every connection must reference a valid `instance_id`.

**Grid Consistency**  
Grid lines must be monotonically increasing. Floor cartridges must tile completely without gaps.

**Constraint Satisfaction**  
Products may only connect at declared connection points. Placement constraints (`min_adjacent_width`, `max_stack_height`) must be satisfied.

**Interface Compatibility**  
Connected products must share compatible interface standards and versions.

**Structural Performance**  
Transmitted loads at each connection must not exceed the receiving element's capacity.

**Thermal Performance**  
All exterior elements must be compatible with the project's climate zone.

**Chain of Custody Consistency**  
Dates must be internally consistent. Legal status must progress correctly from goods to fixture to real property.

---

## 9. Use Cases

### 9.1 Building Configurators

The primary use case is the CTO building configurator: a web-based tool where customers configure homes from a kit of pre-certified parts. The IKEA kitchen planner is the UX precedent. The configurator validates constraints in real time, displays pricing, and generates a `.cto` file that can flow into manufacturing.

### 9.2 Manufacturing Execution

CTO files can be ingested by factory MES systems to schedule production, allocate materials, assign serial numbers, and track work-in-process. The `production_slots` array in the fulfillment plan integrates directly with factory scheduling.

### 9.3 Supply Chain Coordination

The fulfillment plan coordinates multiple parties: factory, third-party inspectors, carriers, crane operators, installers, and AHJs. Each party can read the sections relevant to their role while respecting the chain of custody.

### 9.4 Legal Documentation

The chain of custody section provides evidentiary support for disputes, warranty claims, and insurance inquiries. It answers the key questions: Who owned the product? Who bore the risk? Which policy responded?

### 9.5 Multi-Manufacturer Marketplaces

As interface standards mature, CTO files will enable configurations that mix products from multiple manufacturers — all validated for compatibility through shared interface standard conformance.

---

## 10. Path to Adoption

The CTO file format is being developed through CfOC's ANSI-accredited consensus process. Adoption will proceed through several phases:

### 10.1 Reference Implementation

Logic Building Systems is developing a reference implementation: a web-based configurator that generates v1.1.0-compliant CTO files for single-family residential configurations.

### 10.2 Validation Tools

CfOC will publish open-source validation tools (JSON Schema, reference validator, test fixtures) to help developers verify conformance.

### 10.3 Manufacturer Onboarding

CfOC will work with pod, panel, and module manufacturers to develop CTO-compatible product catalogs. The goal is to demonstrate interoperability across multiple suppliers.

### 10.4 IANA Registration

CfOC will register the MIME type (`application/vnd.cfoc.cto+json`) with IANA to formalize the file format in internet standards.

### 10.5 Integration Partners

CfOC will partner with MES vendors, ERP systems, and project delivery platforms to demonstrate end-to-end workflows from configuration to acceptance.

---

## 11. Conclusion

The CTO file format represents a new foundation for the built environment — one designed for Configure-to-Order rather than Engineer-to-Order delivery.

By encoding configurations as compositions of catalog products, the CTO format enables machine validation, real-time pricing, supply chain integration, and legal traceability. By documenting the chain of custody from factory to acceptance, it bridges the gap between commercial law (UCC) and property law (Common Law) that has long created ambiguity in offsite construction.

Version 1.1.0 introduces the interface standard conformance framework that will enable multi-manufacturer interoperability, and the chain of custody system that documents the legal mateline crossing from goods to real property.

The CfOC invites software developers, manufacturers, and legal professionals to engage with this draft specification. Together, we can build the digital infrastructure for a Configure-to-Order building economy.

---

## Document Information

**Specification Version:** 0.1.0  
**Whitepaper Date:** April 5, 2026  
**Comment Period:** Open through June 30, 2026  
**License:** Apache 2.0

### Editors

Jason Van Nest, Center for Offsite Construction  
Mathew Ford, Center for Offsite Construction

### Contributors

Michael Nolan, CfOC BIM/VDC Research Fellow  
Steve DeWitt, CfOC Senior Research Fellow  
Sam Williams, CfOC Senior Research Fellow

### Contact

Center for Offsite Construction  
School of Architecture & Design  
New York Institute of Technology  
1855 Broadway, New York, NY 10023  
mford@centerforoffsiteconstruction.org
