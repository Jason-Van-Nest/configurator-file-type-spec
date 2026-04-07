# Configurator File Type Specification

A comprehensive, open-source specification for the **CTO (Configure-to-Order) File Format** — the standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products.

## Overview

The CTO file format enables seamless interoperability between Configure-to-Order building configurators, manufacturing execution systems, and project delivery platforms. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

### Key Features

- **Catalog-Constrained Design**: Every element references a product in a known catalog
- **Pre-Validated Connections**: All connections are validated against interface standards before saving
- **Complete Chain of Custody**: Tracks products from factory release through acceptance, documenting the legal transition from goods to real property
- **Price Transparency**: Deterministic pricing based on product composition
- **Self-Describing**: Contains all metadata necessary for rendering, validation, and pricing without external dependencies
- **End-to-End Traceability**: Encodes manufacturing schedules, delivery logistics, and installation sequencing

## Current Status

- **Version**: 0.1.2 (Draft)
- **Status**: In Active Development
- **Last Updated**: April 7, 2026

## What's Included

- **[CTO File Format Specification v0.1.0](spec/v0.1/specification.md)** — Complete schema definition, validation rules, and examples
- **[Contributing Guidelines](CONTRIBUTING.md)** — How to propose changes and contribute
- **[Examples](examples/)** — Sample CTO files and templates
- **[Change Log](spec/CHANGELOG.md)** — Version history and updates

## Quick Links

| Resource | Link |
|----------|------|
| Full Specification | [spec/v0.1/specification.md](spec/v0.1/cto-file-format-spec-v0.1.0.md) |
| Contributing Guide | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Change Log | [spec/CHANGELOG.md](spec/CHANGELOG.md) |
| GitHub Repository | https://github.com/Jason-Van-Nest/configurator-file-type-spec |

## Getting Started

### For Specification Readers
1. Start with the [Overview section](spec/v0.1/specification.md#1-introduction) of the specification
2. Review the [Design Principles](spec/v0.1/specification.md#2-design-principles)
3. Explore the [Schema Definition](spec/v0.1/specification.md#4-schema-definition)

### For Implementers
1. Review the complete [Schema Definition](spec/v0.1/specification.md#4-schema-definition) and [Product Library Schema](spec/v0.1/specification.md#5-product-library-schema)
2. Implement [Validation Rules](spec/v0.1/specification.md#7-validation-rules)
3. Test against the [Validation Suite](https://github.com/cfoc/cto-validator) (external)
4. Check your implementation against the [Examples](spec/v0.1/specification.md#10-examples)

### For Contributors
See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes, report issues, and submit improvements.

## Key Concepts

### Configure-to-Order (CTO)
A design approach where buildings are composed from catalog products rather than the arbitrary geometry of Engineer-to-Order methods of architects and engineers. A CTO configurator constrains design to what's pre-designed, pre-certified, manufactureable, and available.

### Catalog-Constrained Design
Every element in a CTO file must reference a `product_id` that exists in a known catalog. The catalog — not the designer — defines what is possible.

### Chain of Custody
The documented sequence of handoffs tracking a product instance from factory release through acceptance. It captures:
- **Who** owns and bears risk at each stage
- **What** the legal status is (goods, fixture, or real property)
- **When** the legal mateline crossing occurs (at acceptance)
- **Which** insurance policy responds at each stage

### Interface Standards
CfOC-published specifications (e.g., CfOC-ICC-1220, CfOC-ICC-1230) that define connection geometry and performance requirements. All connections in a valid CTO file must conform to declared interface standards.

## Technical Details

### File Format
- **Extension**: `.cto`
- **Encoding**: UTF-8 JSON
- **Compression**: Optional gzip (`.cto.gz`)
- **MIME Type**: `application/vnd.cfoc.cto+json`

### Scope
The current specification covers:
- Single-family residential configurations (v0.0+)
- Multi-family residential configurations (v0.1+)
- Commercial configurations (planned for v2.0)

### Validation
A CTO file is valid if it:
- Conforms to the schema definition
- Passes all validation rules
- Has referential integrity (all references exist)
- Satisfies all constraints (spatial, structural, interface)
- Maintains chain of custody consistency (if fulfillment plan is present)

## Standards & References

The CTO specification references and is designed to interoperate with:

| Standard | Organization | Relevance |
|----------|--------------|-----------|
| CfOC-ICC-1220 | CfOC/ICC | Module-to-module connectivity |
| CfOC-ICC-1230 | CfOC/ICC | Panel-to-panel connectivity |
| ICC/MBI-1200 | ICC/MBI | Off-site construction requirements |
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

- ## Roadmap

| Milestone | Status | Target Date |
|-----------|--------|-------------|
| v0.1.0 Initial Release | ✅ Complete | April 5, 2026 |
| v0.1.2 Element/Assembly Nesting | ✅ Complete | April 7, 2026 |
| Validation Suite | 🔄 In Progress | 2026 |
| Reference Implementation | 🔄 In Progress | 2026 |
| Commercial Support | 📋 Planned | 2027 |
| v2.0 (Commercial Scope) | 📋 Planned | 2027 |

## Related Projects

- **[cto-validator](https://github.com/cfoc/cto-validator)** — Reference validation suite
- **CfOC Standards** — https://centerforoffsiteconstruction.org/standards/

## Document Information

**Specification Authors:**
- Jason Van Nest, Center for Offsite Construction
- Mathew Ford, Center for Offsite Construction

**Contributors:**
- Michael Nolan, CfOC BIM/VDC Research Fellow
- Steve DeWitt, CfOC Senior Research Fellow
- Sam Williams, CfOC Senior Research Fellow

**Copyright:**
© 2026 Center for Offsite Construction

---

**Last Updated:** April 7, 2026

For the latest updates, visit: https://github.com/Jason-Van-Nest/configurator-file-type-spec


For the latest updates, visit: https://github.com/Jason-Van-Nest/configurator-file-type-spec
