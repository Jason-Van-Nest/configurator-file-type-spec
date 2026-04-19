# Introducing the CTO File Format

**A prose introduction to the Configure-to-Order file format ecosystem**
*Updated for v0.2.0 — April 19, 2026*

---

## The short version

The CTO file format is an open standard for representing buildings as **compositions of pre-validated catalog products**, rather than as arbitrary geometry the way CAD or BIM files do. A `.cto` file describes a complete or partial building by referencing products that have already been engineered, certified, and priced — and by declaring how those products connect.

As of v0.2.0, the CTO format has a companion: the **CIS file format**, which separately documents the connection interface standards that products conform to. Together, the two file formats let manufacturers, software vendors, and AHJs work with a shared, machine-readable language for offsite construction.

If you've used the IKEA kitchen planner, the conceptual model is similar: you don't draw arbitrary cabinets, you place catalog cabinets. The CTO format takes that idea and applies it to whole buildings — pods, panels, cartridges, couplers — with the rigor needed for real-world construction (structural validation, code compliance, chain of custody, legal status tracking from manufacturing to acceptance).

---

## The two file formats

### `.cto` files describe **what gets built**

A `.cto` file has one of four `cto_type` values:

- **`assembly`** — a complete or partial building configuration. This is what a designer creates when they sit down at a configurator and arrange products to form a building. An assembly file references product IDs from a manufacturer's catalog and describes how the products are placed and connected.
- **`element`** — a single product definition. This is what a manufacturer authors when they want to publish a product (a kitchen pod, a wall panel) in a way that other tools can validate and reference.
- **`template`** — a pre-validated assembly used as a starting point. Think of templates like "starter floor plans" — they're real assemblies that designers can open and modify.
- **`entourage`** — non-structural visual elements like furniture layouts. Entourage files render in the configurator for scale and context but don't contribute to pricing, scheduling, or chain of custody.

### `.cis` files describe **how products connect**

A `.cis` file documents one connection interface standard. Connection standards specify what happens at the plane between two products — port positions, gendering, utility requirements (pipe sizes, electrical specs), structural handshakes (bolts, alignment pins), and version compatibility.

Two examples make this concrete:

**`CfOC-ICC-1220`** is an open standard governing the core-utility handshake between any volumetric module and the building services it plugs into. It specifies a 6"×92.5" connection envelope with cold water at one position, sanitary drain at another, primary electrical at a third, and so on. Any manufacturer can adopt CfOC-ICC-1220 in their products; the standard belongs to the industry.

**`LBS-INT-UTIL-KIT`** is a proprietary catalog-internal standard owned by Logic Building Systems. It governs how Logic kitchen pods connect to Logic utility pods. Other manufacturers' products can't use this standard, but Logic's own catalog uses it consistently.

Both kinds of standards — open and proprietary — use the same `.cis` file format. The `registry_scope` field distinguishes them.

---

## The relationship between CTO and CIS

The architectural design is straightforward: **products declare which standards they conform to; standards specify what those declarations mean.**

A kitchen pod's `.cto` element file declares: *"My back face addresses the dwelling-unit side of CfOC-ICC-1220, and my left or right face addresses the kitchen-pod side of LBS-INT-UTIL-KIT."* That's two interfaces, expressed compactly in the element file's `declares_interface` block.

The element file does NOT include the port positions, pipe specs, or bolt diameters of either interface. Those live in the referenced `.cis` files. The element file just says *which* standards apply and *which face of the product addresses which side of each standard*.

This separation has three benefits:

1. **Standards bodies can publish and version independently of product catalogs.** When CfOC publishes a new version of `CfOC-ICC-1220`, manufacturers don't have to re-author their product files — they just bump the version reference.
2. **Element files stay compact and focused.** A kitchen pod file is about 200-300 lines describing the kitchen pod, not 1000+ lines re-documenting the connection standard.
3. **Multiple standards on one product is natural.** A bath pod that uses CfOC-ICC-1220 on its top face and LBS-INT-BATH-UTIL on its bottom face just declares two interfaces. No special schema accommodation needed.

---

## A small but important constraint: the rotation lock

One concept in the CTO spec deserves explicit explanation because it's easy to miss and important to get right: the **rotation-lock validation rule** in the `interface_roles` block.

When a product declares `interface_roles`, it's not just providing informational metadata. It's saying: *"Only this specific face of mine is permitted to mate with this CIS plane. If you (the configurator) try to rotate me so a different face is at the connection plane, you've broken the rule."*

Why does this matter? Because a CIS file knows what the connection plane looks like, but it doesn't and can't know which face of any specific product is the face that addresses it. A bath pod has six faces; only one of them carries the 1" PEX male coupling that a CfOC-ICC-1220 plane expects. If a configurator blindly rotated the pod 90°, it would try to connect the side face to the building services — and there's no PEX coupling on that side face. Manufacturing impossibility.

The `interface_roles` block prevents this. It's the binding between *product geometry* (which face is which) and *interface standards* (what connections happen where). Both are needed. Neither alone is sufficient.

A subtle but important policy choice: rotation violations produce **warnings, not errors**. At design time, users explore — they may want to rotate a product to see what it would look like, even if the rotation breaks an interface. The warning informs them. Manufacturing-ready export requires resolving all warnings; design-time exploration permits them.

---

## What's new in v0.2.0

The CTO spec graduated from v0.1.6 to v0.2.0 in coordination with the first publication of the CIS spec (v0.1.3). The major changes:

- **Companion specification introduced.** The CIS file format is published as a sibling to CTO in the same repository. CTO spec §1.6 explains the relationship.
- **`declares_interface` block formalized.** Previously a forward-pointer ambiguity in the CTO spec, this block is now a fully-specified top-level array. See CTO §4.2a.
- **`element_meta` block added.** Element files (`cto_type: "element"`) get their own metadata block, distinct from the `project_meta` block used by user assemblies. See CTO §4.2.
- **`interface_roles` reframed as a rotation-lock validation rule.** The block was always meant to enforce rotation constraints; the v0.2.0 prose makes this explicit, with a worked bath-pod example. See CTO §5.7.
- **Eight in-scope clarifications.** Bounding box coordinates, face label semantics, clearance zone scope, and several other ambiguities surfaced during real catalog authoring are now resolved in prose. See CTO Appendix D.
- **A new design principle: specifications speak to humans.** Schema definitions alone serve only software developers. The spec now states explicitly that prose is required for two other audiences (standards engineers and decision-makers), with extra prose preferred to less. See CTO §2.8.

The CIS spec is at v0.1.3, having gone through three iterations during a single intensive authoring session that surfaced 22 spec observations. Approximately one-third of those observations are addressed in the v0.2.0 / v0.1.3 paired release; the rest are tracked for future versions.

---

## Where to go next

**Read the specs:** [CTO v0.2.0](spec/cto/v0.2/specification.md) | [CIS v0.1.3](spec/cis/v0.1/specification.md)

**See an example:** [CfOC-ICC-1220 v0.2.0 CIS file](examples/cis/CfOC-ICC-1220-v0.2.0.cis) — a fully-conformant CIS file documenting the core-utility handshake between modules and building services

**Track the version history:** [CHANGELOG.md](spec/CHANGELOG.md)

**Contribute:** Issues and pull requests are welcome at the [GitHub repo](https://github.com/Jason-Van-Nest/configurator-file-type-spec). Please draft prose-first and JSON-second, per the v0.2.0 design principle.

---

## A note on the project context

The CTO/CIS file format ecosystem is a project of the [Center for Offsite Construction (CfOC)](https://centerforoffsiteconstruction.org) at NYIT's School of Architecture and Design. CfOC's broader research thesis is that the U.S. construction industry should transition from Engineer-to-Order (ETO) workflows — where every building is designed from scratch and engineered by hand — to Configure-to-Order (CTO) workflows, where buildings are composed from validated catalogs of pre-engineered products.

The CTO and CIS file formats are the **digital infrastructure** for that transition. They make it possible to express buildings in machine-readable terms that catalogs, configurators, manufacturers, and AHJs can all work with. They are not, by themselves, a complete CTO ecosystem — the catalogs and configurators that *use* these file formats are separate work — but they are the language those tools must share.

This work is published under Apache 2.0 because the file format ecosystem only delivers value if anyone can adopt it. We invite manufacturers, software vendors, and standards bodies to use, extend, and improve these specifications.

---

*For the full repository structure, version history, and contribution guidelines, see [README.md](README.md).*
