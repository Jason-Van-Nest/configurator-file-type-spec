# Configurator File Type Specification

A comprehensive, open-source specification for the **CTO (Configure-to-Order) File Format** — the standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products.

## Overview

The CTO file format enables seamless interoperability between Configure-to-Order building configurators, manufacturing execution systems, and project delivery platforms. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

### Key Features

- **Catalog-Constrained Design**: Every element references a product in a known catalog
- **Pre-Validated Connections**: All connections are validated against interface standards before saving
- **Complete Chain of Custody**: Tracks products from factory release through acceptance, documenting the legal transition from goods to real property across six defined party roles: manufacturer, logistics company, GC accepting shipping, hoisting company, pod installer, and panel installer
- **Price Transparency**: Deterministic pricing based on product composition
- **Self-Describing**: Contains all metadata necessary for rendering, validation, and pricing without external dependencies
- **End-to-End Traceability**: Encodes manufacturing schedules, delivery logistics, and installation sequencing
- **Nominal vs. Actual Dimensions**: Separates the grid footprint used by configurators from the physical shipping envelope, supporting chase-zone adjacency patterns
- **Sided Interface Roles**: Products declare which geometric face addresses which side of a CIS (Configure-to-Order Interface Standard) — enabling surface-specific connection validation
- **Multi-Story Support**: Floor level manager and stairwell conceptual objects for coordinated vertical stack configurations

## Current Status

- **Version**: 0.1.5 (Draft)
- **Status**: In Active Development
- **Last Updated**: April 12, 2026

## What's Included

- **[CTO File Format Specification v0.1.5](spec/v0.1/specification.md)** — Complete schema definition, validation rules, and examples
- **[Contributing Guidelines](CONTRIBUTING.md)** — How to propose changes and contribute
- **[Change Log](spec/CHANGELOG.md)** — Version history and updates

## Quick Links

| Resource | Link |
|----------|------|
| Full Specification | [spec/v0.1/specification.md](spec/v0.1/specification.md) |
| Contributing Guide | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Change Log | [spec/CHANGELOG.md](spec/CHANGELOG.md) |
| GitHub Repository | https://github.com/Jason-Van-Nest/configurator-file-type-spec |

## Getting Started

### For Specification Readers
1. Start with the [Introduction](spec/v0.1/specification.md#1-introduction) and review the Terminology table — several key terms (Floor Cartridge, Wall Panel, Nominal Dimensions, Chase Zone, Sided Interface) are defined there
2. Review the [Design Principles](spec/v0.1/specification.md#2-design-principles)
3. Explore the [Schema Definition](spec/v0.1/specification.md#4-schema-definition)

### For Implementers
1. Review the complete [Schema Definition](spec/v0.1/specification.md#4-schema-definition) and [Product Library Schema](spec/v0.1/specification.md#5-product-library-schema)
2. Note the [Unified Party Schema](spec/v0.1/specification.md#43a-unified-party-schema-reference-definition) (Section 4.3a) — all chain-of-custody actor blocks share this structure
3. Implement [Validation Rules](spec/v0.1/specification.md#7-validation-rules)
4. Test against the [Validation Suite](https://github.com/cfoc/cto-validator) (external)

### For Contributors
See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes, report issues, and submit improvements.

## Key Concepts

### Configure-to-Order (CTO)
A design approach where buildings are composed from catalog products rather than the arbitrary geometry of Engineer-to-Order methods. A CTO configurator constrains design to what is pre-designed, pre-certified, manufacturable, and available. The governing principle: **design as purchasing, not drafting**.

### Catalog-Constrained Design
Every element in a CTO file must reference a `product_id` that exists in a known catalog. The catalog — not the designer — defines what is possible.

### Nominal vs. Actual Dimensions
Products declare both `nominal_dimensions` (the grid footprint the configurator reserves for snapping and placement, including chase zone allowances) and `bounding_box` (the physical shipping envelope as manufactured). This distinction is essential for factory-finished pods that carry MEP services on their exterior faces.

### Chase Zones
A protected volume on the exterior face of a pod where MEP services are mounted for factory inspection access. When two pods are placed adjacently with opposing chase zones, the combined gap forms a service chase without additional framing. Chase zones are declared in the product `geometry` block.

### Chain of Custody
The documented sequence of handoffs tracking a product instance from factory release through acceptance. Six party types are defined:

| Party Type | Role |
|---|---|
| `manufacturer` | Fabricates and releases from factory |
| `logistics_company` | Transports to jobsite |
| `gc_accepting_shipping` | Receives and signs for delivery |
| `hoisting_company` | Crane and rigging services |
| `pod_installer` | Installs volumetric pod units (MEP trades) |
| `panel_installer` | Installs structural panels (structural trades) |

Each party block carries name, address, website, contact, license number, and a structured `insurer` object (carrier name, policy number, coverage type, expiration date).

### Interface Standards and CIS Files
CfOC-published specifications (e.g., CfOC-ICC-1220, CfOC-ICC-1230) define connection geometry and performance requirements. In v0.1.5, products reference these standards via `interface_roles` — declaring which geometric face of the product addresses which named side of the standard (e.g., `dwelling_unit_side` vs. `building_services_side`). The engineering details live in external `.cis` files referenced by URL, keeping element files lightweight.

### Floor Levels and Stairwell Objects
Multi-story configurations use a `floor_levels` array to define abstract vertical planes (replacing bare `story` integers). `conceptual_objects` of type `stairwell_volume` span multiple levels and impose void-alignment constraints on adjacent floor cartridges.

### Legal Mateline
The boundary at which a product transitions from goods (governed by UCC Article 2) to real property (governed by Common Law). This transition occurs at acceptance and is explicitly documented in the chain of custody.

## Technical Details

### File Format
- **Extension**: `.cto`
- **Encoding**: UTF-8 JSON
- **Compression**: Optional gzip (`.cto.gz`)
- **MIME Type**: `application/vnd.cfoc.cto+json`

### 3D Asset Standard
GLB (Binary GLTF) files associated with `.cto` elements MUST:
- Be scaled to meters (1 unit = 1 meter, per glTF 2.0)
- Use Y-Up orientation
- Register the South-West-Bottom (SWB) corner of the product at world origin `[0, 0, 0]`
- Project depth along the negative Z-axis

### Scope
The current specification covers:
- Single-family residential configurations (v0.1+)
- Multi-family residential configurations (v0.1.1+)
- Commercial configurations (planned for v0.2.0)

### Validation
A CTO file is valid if it:
- Conforms to the schema definition
- Passes all validation rules
- Has referential integrity (all references exist)
- Satisfies all constraints (spatial, structural, interface, orientation)
- Maintains chain of custody consistency (if fulfillment plan is present)

## Standards & References

The CTO specification references and is designed to interoperate with:

| Standard | Organization | Relevance |
|----------|--------------|-----------|
| CfOC-ICC-1220 | CfOC/ICC | Module-to-module connectivity |
| CfOC-ICC-1230 | CfOC/ICC | Panel-to-panel connectivity |
| ICC/MBI-1200 | ICC/MBI | Off-site construction requirements |
| ICC/MBI-1205 | ICC/MBI | Off-site inspection and compliance |
| UCC Article 2 | ULC | Sale of goods |
| IFC 4.3 | buildingSMART | Building information model exchange |

## How to Contribute

We welcome contributions from:
- **Manufacturers** proposing product enhancements
- **Software vendors** implementing the specification
- **Researchers** studying off-site construction
- **Industry professionals** with implementation feedback

### Ways to Contribute
1. **Open an Issue** — Discuss a proposed change or report a problem
2. **Submit a Pull Request** — Propose edits to the specification
3. **Share Examples** — Contribute example CTO files
4. **Provide Feedback** — Help us improve clarity and usability

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## License

This specification is released under the **MIT license**.

See [LICENSE](LICENSE) for details.

## Contact & Support

- **Issues & Discussions**: Use [GitHub Issues](https://github.com/Jason-Van-Nest/configurator-file-type-spec/issues) to report problems or propose changes
- **Questions**: Open a [GitHub Discussion](https://github.com/Jason-Van-Nest/configurator-file-type-spec/discussions) (if enabled)
- **Direct Contact**: [Submit feedback](https://github.com/Jason-Van-Nest/configurator-file-type-spec/issues/new)

## Roadmap

| Milestone | Status | Target Date |
|-----------|--------|-------------|
| v0.1.5 Release | ✅ Complete | April 2026 |
| Seed Library (LBS Urban Minimalist) | ✅ Complete | April 2026 |
| Validation Suite | 🔄 In Progress | 2026 |
| CIS File Type Specification | 🔄 In Progress | 2026 |
| Reference Implementation | 📋 Planned | 2026 |
| v0.2.0 (Multi-Family + Commercial Scope) | 📋 Planned | 2027 |

## Related Projects

- **[cto-validator](https://github.com/cfoc/cto-validator)** — Reference validation suite
- **CfOC Standards** — https://centerforoffsiteconstruction.org/standards/
- **Logic Building Systems** — https://buildwithlogic.com

## Document Information

**Specification Authors:**
- Jason Van Nest, Center for Offsite Construction
- Mathew Ford, Center for Offsite Construction

**Contributors:**
- Michael Nolan, CfOC BIM/VDC Research Fellow
- Steve DeWitt, CfOC Senior Research Fellow
- Sam Williams, CfOC Senior Research Fellow
- Connor Baily, CfOC Senior Research Fellow, DPR Construction
- Edward Palka, CfOC Senior Research Fellow, CLAE

**Copyright:**
© 2026 Center for Offsite Construction

---

**Last Updated:** April 12, 2026

For the latest updates, visit: https://github.com/Jason-Van-Nest/configurator-file-type-spec
