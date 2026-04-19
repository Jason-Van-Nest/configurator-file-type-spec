# Configurator File Type Specification

This repository hosts the open specifications for two related file formats used in Configure-to-Order (CTO) building configurators:

- **The CTO File Format Specification** (`.cto` files) — building configurations and product definitions
- **The CIS File Format Specification** (`.cis` files) — connection interface standards

Both specifications are released under the Apache 2.0 license and developed by the [Center for Offsite Construction (CfOC) at NYIT](https://centerforoffsiteconstruction.org).

---

## Current versions

| Specification | Current Version | Location |
|---|---|---|
| CTO File Format | **v0.2.0** | [`spec/cto/v0.2/specification.md`](spec/cto/v0.2/specification.md) |
| CIS File Format | **v0.1.3** | [`spec/cis/v0.1/specification.md`](spec/cis/v0.1/specification.md) |

Released: April 19, 2026. See [CHANGELOG.md](spec/CHANGELOG.md) for version history.

---

## What these specifications do

### The CTO file format (`.cto`)

The CTO file format is an open standard for encoding building configurations composed from pre-designed, pre-engineered, and pre-certified offsite construction products. Unlike traditional CAD or BIM files that represent arbitrary geometry, a `.cto` file represents a **bounded design** — a composition of catalog products that has been validated against manufacturer constraints at the point of creation.

A `.cto` file:

- References products from a known catalog rather than defining arbitrary geometry
- Encodes spatial relationships, connection topology, and interface standard conformance
- Carries pricing, scheduling, and constraint metadata
- Tracks the complete chain of custody from fabrication to acceptance
- Documents the legal transition from goods (UCC) to real property (Common Law)
- Can be validated without human review

Four `.cto` file subtypes are defined: `assembly` (a complete or partial building configuration), `element` (a single product definition such as a kitchen pod), `template` (a pre-validated assembly used as a starting point), and `entourage` (non-structural visual elements like furniture layouts).

### The CIS file format (`.cis`)

The CIS file format is the companion specification to CTO. CIS files document **connection interface standards** as machine-readable artifacts — including connection plane sides, port geometry, port connection signatures, utility requirements, and structural handshakes.

A `.cis` file:

- Defines the two sides of a connection plane (e.g., `dwelling_unit_side` / `building_services_side`)
- Specifies port positions, tolerances, and connection signatures within each side
- Describes utility requirements (pipe specs, ASTM standards, pressure ranges) at each port
- Documents structural handshakes (bolts, alignment pins, load transfer)
- Distinguishes open standards (publicly registered) from proprietary catalog-internal standards

A `.cto` element file declares conformance to one or more `.cis` files via the `declares_interface` block. The CIS file is the source of truth for connection-plane specifics; the CTO file is the source of truth for product-specific geometry, lifecycle, and the rotation-lock binding between product faces and CIS sides.

---

## Why two specifications?

In v0.1.x, the CTO spec attempted to carry connection interface details directly within product definitions — port positions, gender assignments, MEP connection geometry, all in the product schema. This worked for simple cases but broke down as soon as real catalog authoring revealed:

- **Connection topology varies by material.** PEX uses barb-to-barb pairing with intermediate pipe; copper uses male-to-female threaded joints; PVC uses slip couplings or rubber boots. A single `gender` field can't express this richness.
- **Standards bodies and product manufacturers have different evolution cadences.** CfOC publishes interface standards on its own schedule; manufacturers update product catalogs on theirs. Coupling the two in a single spec creates artificial coordination overhead.
- **Open standards and proprietary standards coexist.** A pod might use an open CfOC standard for its building-services interface AND a proprietary catalog-internal standard for its pod-to-pod interface. Both deserve first-class machinery.

Splitting connection interface definitions into a separate `.cis` file format solves all three problems. The CTO spec becomes simpler (focused on products and assemblies); the CIS spec becomes possible (a place to publish and version connection standards independently); and the relationship between them is explicit.

---

## Repository structure

```
configurator-file-type-spec/
├── README.md                              ← this file
├── CONTRIBUTING.md                        ← how to contribute
├── LICENSE                                ← Apache 2.0
├── cto-file-format-intro.md               ← prose introduction to CTO
├── web-content-cto-file-format.md         ← FoD&D website content
├── spec/
│   ├── CHANGELOG.md                       ← version history (both specs)
│   ├── cto/
│   │   ├── v0.1/specification.md          ← CTO v0.1.6 (preserved)
│   │   └── v0.2/specification.md          ← CTO v0.2.0 (current)
│   └── cis/
│       ├── v0.1/specification.md          ← CIS v0.1.3 (current)
│       └── history/                       ← intermediate drafts
│           ├── v0.1.0/specification.md
│           ├── v0.1.1/specification.md
│           └── v0.1.2/specification.md
└── examples/
    └── cis/
        └── CfOC-ICC-1220-v0.2.0.cis      ← first example CIS file
```

The CTO and CIS specs live as siblings under `spec/`. Each spec uses per-MAJOR-version subfolders (`v0.1/`, `v0.2/`) holding the latest patch release of that minor version. Pre-release CIS drafts (v0.1.0 through v0.1.2) are preserved in `spec/cis/history/` as a historical artifact of the design conversation that produced v0.1.3.

---

## Getting started

**If you're a software developer building a CTO parser or configurator:**
Start with the CTO spec ([`spec/cto/v0.2/specification.md`](spec/cto/v0.2/specification.md)). Read §1.6 for the relationship to CIS, then §3-§7 for the schema and validation rules. The CIS spec ([`spec/cis/v0.1/specification.md`](spec/cis/v0.1/specification.md)) is normative for any CIS files referenced by CTO files; conformant parsers must be able to parse them.

**If you're a standards engineer authoring a connection standard:**
Read the CIS spec ([`spec/cis/v0.1/specification.md`](spec/cis/v0.1/specification.md)) end-to-end. The CIS spec §15 explains the relationship to CTO files. The example file at [`examples/cis/CfOC-ICC-1220-v0.2.0.cis`](examples/cis/CfOC-ICC-1220-v0.2.0.cis) is the canonical reference for what a fully-conformant CIS file looks like.

**If you're a manufacturer authoring an element file (a `.cto` file with `cto_type: "element"`):**
Read CTO spec §4.1a (cto_type), §4.2 (element_meta), §4.2a (declares_interface), and §5 (Product Library Schema). Then read CIS spec §15 (Relationship to the CTO File Format) for the division of labor between the two file types.

**If you're a decision-maker (manufacturer leadership, AHJ, building official) evaluating these formats for adoption:**
Read [`cto-file-format-intro.md`](cto-file-format-intro.md) for a prose overview that does not require parsing JSON schemas. Then skim CTO spec §1 and §2 (Introduction and Design Principles).

---

## Design philosophy

A new design principle was added in CTO v0.2.0 (§2.8) that we want to call out here: **specifications must speak to humans as well as to parsers**. Three audiences read these specs — software developers, standards engineers, and decision-makers — and schema definitions alone serve only the first.

Every section, block, and field benefits from prose that explains its purpose, scope, and explicit exclusions in plain language. Where a concept has counterintuitive scope — for example, `clearance_zones` describes *external* clearance only, not internal product operations — the spec makes this explicit. When in doubt, we write more prose, not less.

This principle applies to contributions: when proposing changes to either spec, please draft prose-first and JSON-second.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the contribution process. Issues and pull requests are welcome at:
https://github.com/Jason-Van-Nest/configurator-file-type-spec

---

## License

Both specifications are released under the [Apache License 2.0](LICENSE). Implementations may be commercially proprietary or open source; the specifications themselves are open.

---

## Editors and contributors

**Editors:**
- Jason Van Nest, Center for Offsite Construction
- Mathew Ford, Center for Offsite Construction

**Contributors:**
- Michael Nolan, CfOC BIM/VDC Research Fellow
- Steve DeWitt, CfOC Senior Research Fellow
- Sam Williams, CfOC Senior Research Fellow

**Acknowledgments:**
The v0.2.0 / v0.1.3 paired release was driven by the authoring of the first real CIS files (CfOC-ICC-1220 v0.2.0) and the first element file (LBS K01-UM kitchen pod), which surfaced 22 spec observations during a single intensive session. Approximately one-third of those observations are addressed in v0.2.0; the remainder are tracked for future releases.
