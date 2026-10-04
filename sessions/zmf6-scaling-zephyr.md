# Scaling Zephyr (From Monorepo to Ecosystem: How Zephyr Scales to Thousands of Platforms)

[← Agenda](../README.md) · ZMF#6 · 2026-10-06 · 3:45 PM - 5:30 PM

## Key Speakers, Leads, and Required Attendees

- Anas Nashif, TSC Chair (Lead Organizer & Moderator)
- Johann Fischer, Nordic (co-host)
- Keith Short, Google
- \* [Maintainer/Core Developer Name], [Affiliation] (Build system / west / modules / repository architecture — critical for decision-making)
- \* [Maintainer/Core Developer Name], [Affiliation] (Hardware enablement / boards / SoCs / drivers — critical for decision-making)
- [Cross-subsystem Expert Name], [Affiliation] (CI / testing / infrastructure perspective)
- [Vendor or Downstream Representative], [Affiliation] (External hardware enablement / downstream perspective)

## Decision or Outcome Sought

The goal of this session is to agree on a direction for scaling Zephyr from a predominantly centralized, monorepo-style development model toward a broader ecosystem model that can support significantly more hardware, vendors, integrations, and downstream use cases without proportionally increasing central maintenance, review, CI, and infrastructure costs.

**By the end of this session, participants should have:**

- Agreed on:
  - The principles that determine what **must remain centrally maintained in Zephyr** versus what can be developed and maintained externally.
  - The minimum requirements for treating **external hardware enablement as a supported, first-class Zephyr workflow**, rather than as an exceptional or downstream-only model.
  - The role and boundaries of boards, SoCs, HALs, modules, samples, tests, overlays, and manifests in a more scalable ecosystem.
  - A direction for reducing unnecessary dependency leakage and allowing users to obtain only the components required for their platform or application.
- Identified:
  - Technical areas that require prototypes, RFCs, or further design work before implementation.
  - Existing policies or assumptions that need to change to support a decentralized ecosystem.
  - Any issues that require subsequent TSC decisions.
- Assigned:
  - Owners for follow-up proposals covering repository/module boundaries, manifests and dependency management, hardware enablement, and test/sample organization.

The session does not need to produce a complete architecture. It should establish enough common direction that concrete proposals can be developed without repeatedly reopening the fundamental questions.

## Problem Statement

Zephyr’s growth is a major success, but the current development and integration model increasingly places unrelated hardware, vendor dependencies, tests, samples, and platform-specific configuration into shared repositories, workflows, CI infrastructure, and maintainer queues.

The project continues to add boards, SoCs, drivers, HALs, modules, samples, tests, overlays, and vendor-specific integrations. Much of this content is integrated centrally because being “in-tree” currently provides important advantages: discoverability, documentation, CI coverage, compatibility testing, and the perception of being officially supported.

This creates several scaling problems:

- Central maintainers increasingly review hardware or vendor-specific changes for which they may have limited expertise.
- CI and infrastructure costs grow as platform and configuration combinations grow.
- Generic samples and tests accumulate hardware-specific overlays and platform-validation configurations.
- The Zephyr workspace contains or references dependencies that are irrelevant to many users.
- Vendor-specific implementation details and dependencies increasingly leak into the default Zephyr experience.
- Repository size and complexity grow even when individual users require only a very small subset of the ecosystem.
- External hardware enablement is possible today, but it is not always as discoverable, well-defined, testable, or well-supported as equivalent in-tree integrations.
- Vendors and contributors therefore have a strong incentive to move content into the main tree even when central ownership provides little technical benefit.
- Once content is accepted, there is no consistently applied lifecycle model for determining whether it should remain centrally maintained indefinitely.

The problem is broader than repository size. It affects maintainership, review latency, CI capacity, documentation, dependency management, testing, release processes, downstream compatibility, and the ability of vendors to innovate independently.

If the project continues scaling primarily by adding everything to central repositories and processes, the cost of adding the next platform or vendor will increasingly be paid by the entire project.

This requires a cross-project discussion because no single subsystem can solve it independently. Changes to repository boundaries affect west and manifests; changes to hardware integration affect boards, SoCs and drivers; changes to testing affect Twister and CI; and decentralization affects documentation, governance, maintenance expectations, compatibility, and the overall user experience.

## Scope

### In scope

- Define principles for what belongs in the main Zephyr tree versus external repositories or modules.
- Boards and board-specific enablement.
- SoCs and vendor/platform-specific hardware support.
- Drivers, HALs, vendor SDKs, and external dependencies.
- Zephyr modules and west manifests.
- Dependency selection and avoiding unnecessary downloads or initialization of unrelated components.
- Making external hardware enablement a first-class and documented workflow.
- Ownership and maintenance expectations for centrally and externally maintained components.
- Samples, tests, fixtures, snippets, and overlays associated with hardware enablement.
- Distinguishing API/behavior tests from board or vendor qualification.
- CI responsibilities and boundaries between the Zephyr Project and component owners.
- Discoverability and compatibility expectations for externally maintained Zephyr components.
- Lifecycle considerations: admission, maintenance, deprecation, and potentially moving components between central and external maintenance.

### Out of scope

- Deciding during this session which existing individual boards, SoCs, drivers, or HALs should be moved out of tree.
- Designing the complete implementation of a package registry, module registry, or replacement for west.
- Selecting specific hosting infrastructure for external repositories.
- Solving individual vendor integration disagreements.
- Detailed implementation of new build-system or manifest features.
- Defining a Zephyr certification or trademark program.
- Redesigning Zephyr as a collection of completely independent repositories.
- Producing migration patches during the session.

The session should establish principles and direction first. Detailed implementation should follow through focused proposals and prototypes.

## Questions Requiring Agreement

### Decision 1: What must remain centralized in Zephyr?

**Decision required:**
What functionality has sufficient cross-project importance that it should remain under central Zephyr Project governance and maintenance?

Possible categories include:

- Kernel and architecture infrastructure.
- Public subsystem APIs and common abstractions.
- Generic driver models.
- Common protocol stacks and services.
- Build, configuration, tooling, and core testing infrastructure.
- Vendor-neutral documentation and reference samples.
- Conformance and API-level tests.
- Selected reference implementations or platforms.

**Possible outcomes:**

- Define a relatively small **Zephyr core/platform layer** that is expected to remain central.
- Continue the existing broad in-tree model but apply stronger admission criteria.
- Use a hybrid model where some hardware remains central while vendor/product-specific enablement becomes increasingly external.

The desired result is not a list of individual files or boards, but an agreed set of principles for determining central ownership.

### Decision 2: What can safely live outside the main Zephyr tree?

**Decision required:**
Which classes of content can be maintained externally without fragmenting the Zephyr ecosystem or reducing compatibility?

Potential candidates include:

- Product-specific boards.
- Vendor evaluation or reference boards.
- Vendor-specific SoC enablement.
- Vendor SDK/HAL integration.
- Specialized or optional middleware.
- Product validation tests.
- Hardware-specific samples.
- Platform-specific overlays and configuration.
- Experimental hardware support.
- Components with a single vendor or maintainer community.

**Possible outcomes:**

- Allow most hardware enablement to live externally when standard Zephyr interfaces are respected.
- Use explicit criteria to determine when hardware belongs centrally versus externally.
- Introduce an incubation or lifecycle model where integrations can begin externally and move centrally when broader project value is demonstrated.

A key principle to examine is whether Zephyr should care **where an implementation lives or how many implementations exist**, provided that they conform to defined interfaces and integration contracts and have clear ownership.

### Decision 3: What makes external hardware enablement a first-class Zephyr workflow?

**Decision required:**
What guarantees, tooling, metadata, documentation, and project support are required so that “out-of-tree” does not mean “unsupported” or “second class”?

Potential requirements include:

- A standard module/package structure.
- Clearly defined interfaces and integration contracts.
- Compatibility metadata.
- Zephyr version compatibility information.
- Maintainer and ownership metadata.
- Documentation integration.
- Discoverability.
- Reproducible dependency resolution.
- Standard CI/test expectations.
- Security contact and maintenance expectations.
- Standard workflows for adding an external platform to a Zephyr workspace.
- A path for externally maintained integrations to become centrally maintained where appropriate.

**Possible outcomes:**

- Extend the existing Zephyr module model to become the primary mechanism.
- Define a new hardware/package concept layered on top of modules.
- Introduce an ecosystem registry or catalog while retaining repositories independently.
- Define the requirements now and defer the exact technical implementation to a follow-up design proposal.

### Decision 4: How should manifests and dependencies scale?

**Decision required:**
How should Zephyr evolve so users obtain only the repositories and dependencies needed for their selected platforms and use cases?

Questions include:

- Should a default Zephyr workspace continue to reference all vendor HALs and modules?
- Should dependencies be selectable by vendor, platform, feature, or package?
- Should modules or hardware packages be allowed to contribute manifest fragments?
- How do we retain reproducibility when dependency sets become dynamic or selective?
- How should documentation and CI operate when different users have different dependency sets?
- What should happen when a board requires dependencies that are not part of the default Zephyr workspace?

**Possible outcomes:**

- Retain the current manifest model with improved filtering.
- Introduce optional or conditional module groups.
- Allow hardware packages/modules to declare their own dependencies.
- Move toward a minimal core manifest plus explicitly selected ecosystem packages.

The goal is to establish requirements and identify approaches worth prototyping, not to design west syntax during the session.

### Decision 5: What belongs in samples and tests?

**Decision required:**
Where should Zephyr draw the boundary between demonstrating or validating Zephyr behavior and validating individual platforms or vendor products?

The session should distinguish between:

- API/example samples.
- Zephyr regression tests.
- Specification or conformance tests.
- Hardware qualification tests.
- Vendor product-validation tests.
- Hardware-specific examples.
- Platform-specific overlays and fixtures.

**Possible outcomes:**

- Generic samples demonstrate intended behavior and should avoid becoming platform-validation matrices.
- Generic tests validate Zephyr interfaces and behavior rather than every possible hardware configuration.
- Platform qualification should reuse Zephyr test infrastructure but may maintain its configurations and overlays with the platform itself.
- Hardware-specific examples should generally be maintained together with the hardware they demonstrate.

An expected principle is that adding a new platform should not normally require modifying large numbers of unrelated tests and samples merely to describe how that platform should run them.

### Decision 6: What ownership and lifecycle model should apply?

**Decision required:**
What responsibilities must accompany the addition of hardware or vendor-specific functionality, and what happens when those responsibilities are no longer met?

Questions include:

- Who owns hardware-specific drivers and integrations?
- What responsibility does a subsystem maintainer have for vendor-maintained implementations?
- Should subsystem maintainers primarily protect the API and architecture rather than become maintainers of every implementation?
- What maintenance commitment is required for centrally hosted hardware?
- What happens when maintainers disappear?
- Can content move from central to external maintenance without being considered deprecated?
- Can external integrations move centrally when they gain broad project relevance?

**Possible outcomes:**

- Vendor/component maintainers own implementations while subsystem maintainers own the common interface and architecture.
- Central integration requires an explicit ongoing maintenance commitment.
- Repository placement becomes a lifecycle decision rather than a permanent status.
- Establish defined states for incubation, supported external integration, central integration, deprecated/unmaintained content, and retirement.

## Proposed Session Agenda

Not too strict agenda, but we need to manage the discussion and time box it, i.e. for a 2 hour session

The agenda should remain flexible enough for technical discussion, but the moderator should actively move the group toward the explicit decisions above.

### Present the problem / proposal (~20–30 minutes)

- Briefly establish how Zephyr got to the current model.
- Present representative scaling problems rather than attempting an exhaustive list.
- Show concrete examples involving:
  - Hardware/vendor dependencies.
  - Modules and manifests.
  - Samples/tests/overlays.
  - Review or CI burden.
  - Successful and unsuccessful out-of-tree integrations.
- Present the central question:
  **How do we allow the Zephyr ecosystem to grow significantly faster than the centrally maintained Zephyr project?**
- Confirm that the problem statement and scope are accepted before moving into solutions.

### Discuss (~60 minutes)

Structure the discussion around the decision questions rather than allowing it to become a general conversation about repository size.

Suggested grouping:

**Centralization boundaries**

- What must remain central?
- What can move outside the main tree?
- What interfaces or contracts preserve ecosystem coherence?

**First-class ecosystem model**

- What does supported external hardware enablement require?
- How should ownership work?
- How do users discover and consume external integrations?

**Dependencies and manifests**

- How do we prevent unrelated vendor dependencies from becoming part of every Zephyr workspace?
- What capabilities are missing from the existing module/manifest model?

**Tests and samples**

- Where does Zephyr validation end and platform/vendor qualification begin?
- Where should the associated configuration and overlays live?

For each area, explicitly capture:

- Areas of agreement.
- Strong objections.
- Questions requiring additional data.
- Ideas requiring prototypes.
- Issues that require TSC policy decisions.

Do not spend significant time attempting to design detailed implementation syntax or repository layouts.

### Closing (~30 minutes)

Review the decisions and direction captured during the discussion.

For each major question classify the outcome as:

- **Agreement reached**
- **Direction agreed, details required**
- **Requires prototype/data**
- **Requires TSC decision**
- **No agreement**

Then identify concrete follow-up work.

Expected follow-up items may include:

- Repository/content placement policy.
- Hardware enablement lifecycle proposal.
- Definition of a first-class external hardware/module model.
- Manifest/dependency-selection prototype.
- Samples/tests/overlays policy update.
- CI responsibility model.
- Discovery/catalog/registry requirements.

For every follow-up item, assign:

- An owner.
- The expected artifact: RFC, policy change, prototype, documentation PR, etc.
- Where the result will be reviewed.
- A target milestone or TSC meeting where appropriate.

The session should finish with a concise statement of the agreed direction, even if implementation details remain unresolved.

## Pre-Session Preparation

### Session owner

Before the event, the session owner should:

- Complete Sections above.
- Provide links to relevant issues, pull requests, and documentation.
- Identify two or three representative examples.
- Describe realistic options rather than only the preferred solution.
- Highlight decisions that may require TSC approval.
- Provide a few concrete examples of:
  - Content that clearly belongs centrally.
  - Content that arguably does not need to be central.
  - An external integration that works well today.
  - A case where current manifests/dependencies create unnecessary coupling.
  - A sample or test where platform-specific configuration has become difficult to maintain.
- Avoid attempting to solve every implementation detail in advance; preparation should provide enough context for participants to make the architectural and policy decisions.

### Participants

Participants should:

- Read the document before the event.
- Add missing evidence or examples.
- Comment on the proposed scope.
- Identify constraints that may have been overlooked.
- Indicate strong objections before the session where possible.
- Consider the problem from the perspective of the overall Zephyr ecosystem, not only the subsystem, organization, or hardware they currently maintain.
- Come prepared to distinguish between:
  - What **must** be centrally controlled to preserve Zephyr compatibility.
  - What is centralized today primarily because the project does not yet provide a good alternative.
