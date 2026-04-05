# CTO File Format

## The Configure-to-Order Standard for Offsite Construction

**Current Version:** 1.1.0  
**Status:** Draft for Public Comment  
**Specification:** [View Full Specification](/standards/cto-file-format-spec-v1.1.0)  
**GitHub:** [cfoc/cto-file-format](https://github.com/cfoc/cto-file-format)

---

## What is the CTO File Format?

The CTO (Configure-to-Order) file format is an open standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products.

Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

The CTO format embodies the core CfOC principle: **design as purchasing, not drafting**.

---

## Why a New File Format?

The architecture, engineering, and construction industry lacks a standard for Configure-to-Order building delivery. Existing formats (DWG, RVT, IFC) were designed for Engineer-to-Order workflows where every building is designed from scratch.

| Traditional Formats | CTO Format |
|---------------------|------------|
| Represent arbitrary geometry | Reference catalog products |
| Require human interpretation | Machine-validatable |
| Silent on pricing | Carries MSRP data |
| No supply chain integration | Tracks manufacturing and delivery |
| Legal status ambiguous | Documents goods-to-property transition |

The CTO format enables a new kind of building configurator — one where customers configure their homes from a catalog of pre-certified products, receive a firm price in real time, and generate a file that can flow directly into manufacturing execution systems.

---

## Key Features

### Catalog-Constrained Design

Every element in a CTO file references a `product_id` from a known catalog. The catalog — not the designer — defines what is possible. Invalid configurations cannot be saved.

### Interface Standard Conformance

Products declare the CfOC interface standards to which they conform (CfOC-ICC-1220 for modules, CfOC-ICC-1230 for panels). The configurator validates that connected products are compatible before allowing the connection.

### Real-Time Pricing

Every product carries an MSRP. Options and upgrades add predictably. The total price is always deterministic and transparent.

### End-to-End Traceability

The CTO format tracks each product instance from factory release to acceptance:

| Stage | Legal Status |
|-------|--------------|
| Factory Release | Goods (UCC) |
| Transport | Goods (UCC) |
| Delivery | Goods (UCC) |
| Staging | Goods (UCC) |
| Placement | Goods → Fixture |
| Connection | Fixture |
| Commissioning | Fixture |
| **Acceptance** | **Real Property** |

### Legal Mateline Documentation

The CTO format explicitly documents the transition from goods (governed by UCC Article 2) to real property (governed by Common Law). At each custody stage, the file records:

- **Who owns the product** (title holder)
- **Who bears the risk of loss** (risk owner)
- **Which insurance policy responds**
- **Legal status** (goods, fixture, real property)

This documentation supports the legal clarity called for in the CfOC whitepaper *From Handshake to Hardware*.

---

## File Structure

A CTO file is UTF-8 encoded JSON with the following top-level structure:

```json
{
  "cto_version": "1.1.0",
  "project_meta": { },
  "catalog_reference": { },
  "structural_grid": { },
  "floor_cartridges": [ ],
  "placed_elements": [ ],
  "roof_elements": [ ],
  "pricing_snapshot": { },
  "fulfillment_plan": { },
  "validation_state": { }
}
```

### Project Meta

Identifies the configuration: name, author, design line, climate zone, seismic design category, and jurisdiction.

### Catalog Reference

Points to the manufacturer catalog and declares which CfOC interface standards are used.

### Structural Grid

Derived from floor cartridge placement. The grid is an output of product placement, not an input to design.

### Placed Elements

Each element carries:
- Product reference
- Placement coordinates
- Rotation
- Connections (with interface standard references)
- Selected options
- Instance lifecycle data (serial number, chain of custody)

### Fulfillment Plan

Complete manufacturing, delivery, crane lift, and installation schedule — including chain of custody documentation at each stage.

### Validation State

Machine-readable validation results: errors, warnings, and interface compatibility status.

---

## Product Schema

Products in CTO-compatible catalogs carry comprehensive metadata:

### Identity & Lineage
- Product ID, version, design line, status

### Geometry
- Dimensions, weight, center of gravity, clearance zones

### Connectivity
- Connection points with interface standard conformance
- MEP connection locations and specifications

### Performance
- Structural: load capacity, transmitted loads, seismic compatibility
- Thermal: R-values, U-factors, climate zone compatibility
- Fire: ratings, smoke/flame indices

### Certifications
- ICC-ES, UL, ETL listings
- Code compliance (IBC, IRC, IECC, NFPA)
- State modular program approvals

### Installation
- Rigging requirements
- Connection sequence
- Commissioning checklist
- Crew requirements

### Warranty
- Coverage terms by system type
- Embedded product warranties

---

## Use Cases

### Building Configurators

The primary use case is the CTO building configurator: a web-based tool where customers configure homes from a kit of pre-certified parts and receive a firm price in real time.

The Logic Building Systems configurator (in development) is the reference implementation.

### Manufacturing Execution

CTO files can be ingested by factory MES systems to schedule production, allocate materials, and track work-in-process.

### Supply Chain Coordination

The fulfillment plan section coordinates multiple parties: factory, third-party inspectors, carriers, crane operators, installers, and AHJs.

### Legal Documentation

The chain of custody section provides evidentiary support for disputes, warranty claims, and insurance inquiries.

### Multi-Manufacturer Marketplaces

As the interface standards mature, CTO files will enable configurations that mix products from multiple manufacturers — all validated for compatibility.

---

## Relationship to CfOC Standards

The CTO file format is designed to reference and validate against CfOC interface standards:

| Standard | Scope | Status |
|----------|-------|--------|
| **CfOC-ICC-1220** | Module-to-module connectivity | In development |
| **CfOC-ICC-1230** | Panel-to-panel connectivity | In development |

As these standards are published, the CTO format will evolve to incorporate their requirements.

---

## Conformance Levels

| Level | Requirements |
|-------|--------------|
| **CTO Reader** | Can parse and validate CTO files |
| **CTO Writer** | Can produce valid CTO files |
| **CTO Configurator** | Full read/write with interactive editing |
| **CTO Fulfillment System** | Can execute fulfillment plans and chain of custody |

---

## Getting Started

### For Developers

1. **Read the specification:** [CTO File Format Specification v1.1.0](/standards/cto-file-format-spec-v1.1.0)
2. **Clone the repository:** `git clone https://github.com/cfoc/cto-file-format`
3. **Run the validator:** `npm install && npm run validate example.cto`

### For Manufacturers

1. **Review the product schema:** Understand what metadata your products need
2. **Map your catalog:** Identify which CfOC interface standards apply
3. **Contact CfOC:** Discuss certification and catalog publication

### For Researchers

1. **Read the whitepaper:** [CTO File Format: A Legal and Technical Foundation for Configure-to-Order Construction](/research/cto-whitepaper)
2. **Join the discussion:** Comment on the draft specification via GitHub Issues
3. **Propose extensions:** Submit pull requests for consideration

---

## Version History

| Version | Date | Summary |
|---------|------|---------|
| 1.0.0 | April 2026 | Initial release |
| 1.0.1 | April 2026 | Expanded fulfillment plan |
| **1.1.0** | April 2026 | Interface standards, chain of custody, legal mateline, performance attributes |

---

## Feedback & Contribution

The CTO file format is developed through CfOC's ANSI-accredited consensus process.

- **Comment Period:** Open through June 30, 2026
- **Submit Comments:** [GitHub Issues](https://github.com/cfoc/cto-file-format/issues)
- **Join the Standards Committee:** [Apply here](/get-involved/committees)

---

## Related Resources

- [From Handshake to Hardware: Questions & Pathways for Uniform Law & Commerce](/research/from-handshake-to-hardware) — The legal foundation for CTO commerce
- [Open-Source Interface Standards for Offsite Construction](/research/interface-standards) — The physical and digital interfaces that make CTO possible
- [The Future of Design & Delivery](/future-of-design-delivery) — The CfOC research roadmap

---

*The CTO File Format is published by the Center for Offsite Construction under the Apache 2.0 License.*
