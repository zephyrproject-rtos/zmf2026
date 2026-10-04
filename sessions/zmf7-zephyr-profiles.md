# Zephyr Profiles — Scaling from Tiny Devices to Rich Embedded Systems

[← Agenda](../README.md) · ZMF#7 · 2026-10-06 · 1:45 PM - 3:30 PM

## Key Speakers, Leads, and Required Attendees

- Anas Nashif, TSC Chair (Lead Organizer & Moderator)
- Fabio Baltieri, Google (co-host)
- Måns Ansgariusson (invitee)
- Sylvio Alves, Espressif (invitee)
- [Kernel / Architecture Maintainer Name], [Affiliation] (Critical for execution model, memory protection, userspace, and system architecture)
- [POSIX / Userspace / llext Maintainer Name], [Affiliation] (Critical for portability layers, processes, extensions, and richer system services)
- [Subsystem Maintainer Name], [Affiliation] (Cross-subsystem applicability)
- [Safety / Security Expert Name], [Affiliation] (Safety- and security-oriented requirements)
- [Hardware / SoC Expert Name], [Affiliation] (Complex and high-end hardware perspective)

## Decision or Outcome Sought

The goal of this session is to define how Zephyr can support a much broader range of systems and hardware capabilities while preserving its core strengths of low footprint, configurability, determinism, portability, and suitability for deeply embedded systems.

Zephyr today spans systems ranging from tiny MCUs with tens of kilobytes of memory to powerful multi-core SoCs with MMUs, large amounts of RAM, external storage, rich networking, and application environments increasingly resembling those found on larger operating systems.

The project needs a clearer architectural model for supporting these very different use cases without making simple systems pay for complexity they do not need, and without artificially constraining capable hardware to the programming and execution model of the smallest devices.

**By the end of this session, participants should have:**

- Agreed on:
  - What a Zephyr “profile” represents and whether profiles should be formal architectural configurations, collections of capabilities, or documented usage models.
  - Which capabilities should remain part of a minimal Zephyr core and which should be optional higher-level system services.
  - Principles for introducing richer capabilities without increasing baseline footprint or forcing dependencies on smaller systems.
  - How portability layers such as POSIX should relate to native Zephyr APIs and semantics.
  - How Zephyr should approach richer execution models including protected applications, processes, loadable extensions, dynamic components, and application lifecycle management.
  - How Linux-like facilities such as /proc, /dev, system introspection, filesystem-backed interfaces, and device discovery can be supported where useful without making them fundamental Zephyr abstractions.
  - How safety, security, diagnostics, validation, and defensive programming fit into these profiles.
- Identified:
  - Architectural areas where current Zephyr abstractions do not scale well from minimal to feature-rich systems.
  - Existing features that need to be consolidated into a coherent model.
  - Areas requiring prototypes, RFCs, API work, or TSC-level policy decisions.
- Assigned:
  - Owners for follow-up work on profiles, richer execution environments, POSIX integration, process/application models, runtime introspection, and profile-specific guarantees.

The session should establish a model that allows Zephyr to become more capable without changing what makes Zephyr attractive for constrained embedded systems.

## Problem Statement

Zephyr was originally designed around deeply embedded systems where low footprint, static configuration, deterministic behavior, and tight control over resources are fundamental requirements.

That model remains critical and must continue to be a first-class Zephyr use case.

At the same time, the hardware and products using Zephyr are becoming significantly more capable.

Modern platforms may include:

- Multiple processor cores.
- MMUs or sophisticated MPUs.
- Megabytes or gigabytes of memory.
- External storage and filesystems.
- High-performance networking.
- Dynamic application workloads.
- Accelerators and heterogeneous processors.
- Rich device enumeration and discovery.
- Multiple isolated applications.
- Runtime-loaded components.
- Complex debugging and observability requirements.
- POSIX-oriented middleware and application frameworks.

These platforms create demand for capabilities traditionally associated with richer operating systems, including:

- POSIX APIs.
- Process-like execution and isolation.
- Application lifecycle management.
- Dynamic or loadable components.
- Runtime extension mechanisms.
- Device and system introspection.
- Filesystem-oriented interfaces such as /dev or /proc.
- Richer shell and administrative facilities.
- Standardized application environments.
- Dynamic resource management.
- Stronger isolation and security boundaries.

The challenge is not whether Zephyr should become Linux.

It should not.

The challenge is whether Zephyr can support **Linux-like capabilities where they make sense**, while retaining a fundamentally configurable embedded architecture where those capabilities disappear completely when they are not needed.

Today, Zephyr does not always have a clear model for doing this.

As richer functionality is introduced, there is a risk that:

- Core APIs become more complex to support high-end use cases.
- Small systems inherit dependencies, state, indirection, or runtime checks they do not require.
- POSIX compatibility drives native APIs toward semantics inappropriate for an RTOS.
- Higher-level abstractions get added directly into subsystems instead of being layered.
- Configuration becomes a growing collection of unrelated feature switches.
- Individual subsystems invent their own approaches to dynamic objects, introspection, namespaces, or runtime discovery.
- Rich-system functionality becomes fragmented across userspace, POSIX, llext, memory domains, filesystems, shells, and subsystem-specific mechanisms.
- Developers cannot easily determine what functionality belongs to “core Zephyr” versus a richer operating environment built on top of it.

Conversely, if Zephyr optimizes exclusively for very small systems, it risks forcing capable hardware into an unnecessarily restrictive execution model and causing vendors and users to build incompatible extensions outside the project.

The project therefore needs an explicit architecture for **scaling upward without bloating downward**.

## Scope

### In scope

- Definition of Zephyr profiles or capability classes.
- Minimal/constrained systems.
- General embedded production systems.
- Rich connected embedded systems.
- Security-hardened systems.
- Safety-oriented systems.
- High-performance and complex SoCs.
- POSIX compatibility and portability layers.
- Relationship between native Zephyr APIs and POSIX APIs.
- Userspace and application isolation.
- Memory domains and protection models.
- Process-like abstractions.
- Application lifecycle and execution contexts.
- Loadable extensions and dynamic components.
- llext and related runtime-loading mechanisms.
- Dynamic resource and object management.
- Runtime device discovery and enumeration.
- System and device introspection.
- Linux-like concepts such as /dev, /proc, and system information interfaces where appropriate.
- Filesystem-backed administrative interfaces.
- Logging, tracing, diagnostics, assertions, and defensive programming.
- Configuration and dependency boundaries between profiles.
- Ensuring optional capabilities have zero or minimal cost when disabled.
- How APIs and subsystems should behave across profiles.
- Test and CI expectations for representative profiles.

### Out of scope

- Turning Zephyr into a general-purpose Linux replacement.
- Implementing complete Linux ABI compatibility.
- Requiring POSIX APIs for native Zephyr applications.
- Mandating processes or dynamic loading for all systems.
- Designing a complete /proc or /dev implementation during the session.
- Selecting exact filesystem namespace layouts.
- Replacing native Zephyr APIs with POSIX.
- Defining complete functional-safety certification requirements.
- Redesigning every existing subsystem during the session.
- Implementing profile support during the forum.

The goal is to define architectural boundaries and principles, not detailed implementations.

## Questions Requiring Agreement

### Decision 1: What is a Zephyr profile?

**Decision required:**
Should Zephyr explicitly describe a small number of system profiles that represent different capability and policy expectations?

Potential examples:

- **Minimal / Deeply Embedded**
  - Static system composition.
  - Very small footprint.
  - Native Zephyr APIs.
  - Minimal runtime infrastructure.
  - No unnecessary dynamic services.
- **Embedded Production**
  - Standard Zephyr kernel and subsystem model.
  - Controlled diagnostics.
  - MPU-based protection where needed.
  - Filesystems, networking, and richer middleware as selected.
- **Rich Embedded**
  - POSIX compatibility.
  - MMU/MPU-backed isolation.
  - Process-like applications.
  - Runtime-loaded extensions.
  - Rich filesystem and device services.
  - System introspection and administration.
- **Security-Hardened**
  - Strong validation.
  - Isolation.
  - Restricted capabilities.
  - Auditing and defensive runtime checks.
- **Safety-Oriented**
  - Predictable configuration.
  - Restricted dynamism where appropriate.
  - Strong fault detection.
  - Traceability and controlled failure behavior.

**Possible outcomes:**

- Define formal named profiles.
- Define profiles only as documented reference configurations.
- Define orthogonal capabilities instead of fixed profiles.
- Use profiles as combinations of capability sets and policy sets.

A likely useful distinction is between **capabilities** and **policies**.

For example, POSIX or process support is a capability, while aggressive runtime checking or safety restrictions are policies. Profiles may be combinations of both rather than monolithic configurations.

### Decision 2: What belongs in the minimal Zephyr core?

**Decision required:**
Which abstractions must remain universal across Zephyr, and which should be layered so that constrained systems do not pay for richer functionality?

Questions include:

- What is the smallest architectural contract shared across all Zephyr systems?
- Which APIs must remain lightweight and static?
- Should runtime enumeration, namespaces, object registries, process models, or POSIX abstractions ever leak into core kernel interfaces?
- How do we enforce zero-cost or near-zero-cost exclusion of higher-level functionality?

**Possible outcomes:**

- Explicitly define a minimal Zephyr system model.
- Require higher-level functionality to layer over core primitives.
- Introduce architectural boundaries between core kernel mechanisms and richer system services.
- Establish a rule that optional higher-level features must not change baseline semantics or cost when disabled.

The key principle to test is:

**More capable Zephyr configurations should be built by adding layers, not by making the lowest layer progressively heavier.**

### Decision 3: How should POSIX fit into Zephyr?

**Decision required:**
Define the role of POSIX as Zephyr expands into richer application environments.

Questions include:

- Is POSIX primarily a portability layer or an alternative application API?
- Should POSIX behavior ever dictate native Zephyr API design?
- Which POSIX functionality belongs in the C library versus Zephyr itself?
- How complete should POSIX support become for rich profiles?
- Can richer profiles support POSIX concepts such as processes, file descriptors, signals, or process-local state without imposing them on minimal systems?

**Possible outcomes:**

- Treat POSIX as an optional compatibility/application layer built on native Zephyr primitives.
- Expand POSIX significantly for richer configurations while preserving native interfaces independently.
- Define explicit boundaries between native and POSIX semantics.
- Establish that native Zephyr APIs should not imitate POSIX where the semantics do not naturally fit an RTOS.

The desired outcome is to enable software portability without forcing Zephyr itself to behave like a Unix kernel internally.

### Decision 4: Does Zephyr need a coherent application/process model?

**Decision required:**
Determine whether existing mechanisms such as userspace, memory domains, threads, executable loading, and llext should be consolidated into a broader application or process abstraction.

Questions include:

- What constitutes an “application” inside Zephyr?
- Should an application own threads, memory, resources, handles, or capabilities?
- Should applications be independently startable and stoppable?
- Should they have private address spaces where MMU hardware permits?
- How should MPU-only systems map onto the same abstraction?
- How should static applications and dynamically loaded applications relate?
- Should llext evolve into part of a broader executable/application model?
- How much Unix process semantics are useful versus unnecessary?

**Possible outcomes:**

- Keep current mechanisms independent.
- Define a lightweight Zephyr-native application abstraction.
- Introduce an optional process model for capable systems.
- Build a common execution model that can map from static MPU-based applications to MMU-backed processes and dynamically loaded extensions.

The goal should not necessarily be POSIX processes, but a coherent Zephyr model that can support comparable use cases.

### Decision 5: How should richer systems expose devices and system state?

**Decision required:**
Determine whether Zephyr needs optional standardized mechanisms for system introspection, runtime discovery, and administrative interfaces.

Examples include:

- Device enumeration.
- Runtime driver state.
- Memory and thread information.
- Process/application information.
- Network state.
- Power-management information.
- Kernel statistics.
- Resource usage.

Linux commonly exposes such information through mechanisms such as /dev, /proc, and /sys.

Zephyr currently exposes similar information through a mixture of:

- Device structures.
- Shell commands.
- APIs.
- Logging.
- tracing.
- filesystem interfaces.
- subsystem-specific mechanisms.

**Possible outcomes:**

- Keep the existing API/shell-based approach.
- Define common introspection APIs and let frontends expose them through shell, filesystem, RPC, or other mechanisms.
- Introduce optional virtual filesystems similar in spirit to devfs or procfs.
- Define a richer system-services layer that provides namespaces and introspection without making them kernel fundamentals.

An important architectural principle is that /proc or /dev should potentially be **views over common system information**, rather than becoming the underlying kernel architecture.

### Decision 6: How do we prevent richer profiles from creating codebase fragmentation?

**Decision required:**
Establish how optional capabilities should be implemented so that Zephyr does not devolve into profile-specific forks inside the same source tree.

Problems to avoid include:

- Large numbers of `#ifdef CONFIG_RICH_PROFILE`.
- Different APIs for different classes of hardware.
- Profile-specific behavior hidden inside generic subsystems.
- Multiple implementations of the same policy spread across the tree.
- Features that are technically optional but still increase baseline complexity.

**Possible outcomes:**

- Define stable layering rules.
- Use common interfaces with optional implementations.
- Keep policy above mechanism where possible.
- Prefer capability detection over profile-specific conditionals.
- Use reusable infrastructure for namespaces, handles, isolation, introspection, and runtime loading.
- Require new rich-system features to demonstrate that disabled configurations incur no meaningful footprint or runtime cost.

Profiles should organize capabilities, not partition the codebase.

### Decision 7: How do safety, security, performance, and diagnostics fit into this model?

**Decision required:**
Determine how behavioral policies interact with capability profiles.

A powerful system may want:

- Maximum diagnostics during development.
- Strong security checks in production.
- Dynamic applications.
- POSIX APIs.

A safety-oriented system may instead want:

- Strong checks.
- Static configuration.
- No runtime loading.
- Restricted resource creation.
- Highly predictable execution.

A tiny device may want:

- Minimal diagnostics.
- No userspace.
- No dynamic allocation.
- No filesystem.
- Native APIs only.

These requirements are not a single linear progression from “small” to “large.”

**Possible outcomes:**

- Separate **capability profiles** from **policy profiles**.
- Allow combinations such as:
  - Rich + security hardened.
  - Minimal + safety oriented.
  - Rich + development.
  - Minimal + production.
- Define common policy mechanisms for assertions, validation, logging, defensive checks, dynamic allocation, and runtime loading.
- Avoid assuming that more capable hardware automatically means less determinism or weaker safety constraints.

This may lead to a profile model based on several independent dimensions rather than five predefined configurations.

## Proposed Session Agenda

Not too strict agenda, but we need to manage the discussion and time box it, i.e. for a 2 hour session

### Present the problem / proposal (~20–30 minutes)

Start with the range Zephyr now needs to support.

At one extreme:

- Tiny MCU.
- Tens of kilobytes of RAM.
- Static image.
- Native APIs.
- No userspace.
- No filesystem.
- Minimal diagnostics.

At the other:

- Multi-core SoC.
- MMU.
- Hundreds of megabytes or more of RAM.
- Persistent storage.
- Networking.
- Multiple applications.
- POSIX-oriented software.
- Dynamic components.
- Rich observability and security requirements.

Present the central question:

**How does Zephyr scale upward in capability without scaling upward in baseline cost?**

Show examples of existing mechanisms that already point toward richer profiles:

- POSIX.
- Userspace.
- Memory domains.
- llext.
- Filesystems.
- Dynamic memory.
- shell.
- tracing.
- runtime statistics.
- device model.
- SMP.
- MMU/MPU support.

The argument should be that Zephyr already contains many of the pieces, but lacks a coherent architectural model describing how they fit together.

### Discuss (~60 minutes)

Organize around major architectural questions.

**What is core Zephyr?**

- What should every Zephyr system have?
- What must remain extremely small?

**How do capabilities layer?**

- POSIX.
- Filesystems.
- process/application model.
- dynamic loading.
- runtime introspection.
- namespaces and device views.

**What does capable hardware need?**

- Are current thread/userspace abstractions enough?
- What becomes difficult on MMU-based high-end systems?
- Where are vendors already building private solutions?

**How do we avoid becoming Linux?**

- Which Linux concepts are useful?
- Which are implementation details Zephyr should avoid?
- Can familiar interfaces be provided as optional frontends over Zephyr-native mechanisms?

**How do profiles combine?**

- Minimal vs rich.
- development vs production.
- safety vs dynamic.
- security vs performance.

**How do we keep features zero-cost when unused?**

- Build-time elimination.
- dependency isolation.
- layering.
- avoiding global registries or indirection in minimal configurations.

For each topic capture:

- Agreed architectural principle.
- Current mechanism that can be reused.
- Gap requiring new infrastructure.
- Areas requiring prototypes.
- Questions requiring TSC decisions.

### Closing (~30 minutes)

Review conclusions and classify each as:

- **Agreement reached**
- **Direction agreed, details required**
- **Requires prototype/data**
- **Requires TSC decision**
- **No agreement**

The session should ideally produce a small set of principles such as:

1. Zephyr must remain viable for deeply constrained systems.
2. Rich-system capabilities should be additive layers rather than changes to the baseline architecture.
3. Native Zephyr APIs remain independent of POSIX compatibility requirements.
4. POSIX should be capable of becoming substantially richer where hardware allows.
5. Userspace, memory protection, runtime loading, and application isolation should evolve toward a coherent execution model.
6. System introspection should use common internal interfaces with optional frontends such as shell, RPC, or virtual filesystems.
7. Capabilities and behavioral policies should be treated as independent dimensions.
8. Optional functionality should incur negligible cost when disabled.
9. Rich hardware should not be artificially constrained to the capabilities of the smallest supported MCU.
10. Supporting richer systems must not compromise Zephyr's identity as a configurable embedded RTOS.

These are starting points for discussion, not predetermined decisions.

Expected follow-up work may include:

- Formal definition of Zephyr capability profiles.
- Definition of policy profiles or policy dimensions.
- Minimal/core Zephyr architecture document.
- POSIX roadmap for rich systems.
- Unified application/process/userspace model proposal.
- Integration plan for userspace, memory domains, and llext.
- Runtime introspection architecture.
- Optional devfs/procfs-style frontend prototype.
- Profile-oriented CI configurations.
- Guidance for subsystem maintainers on layering and optional capabilities.
- Analysis of zero-cost behavior for disabled rich-system features.

For each follow-up item assign:

- An owner.
- Expected artifact: RFC, architecture document, prototype, API proposal, or policy document.
- Review venue.
- Target milestone or TSC meeting.

## Pre-Session Preparation

### Session owner

Before the event, the session owner should:

- Complete Sections above.
- Provide links to relevant issues, pull requests, and documentation.
- Identify two or three representative examples.
- Describe realistic options rather than only the preferred solution.
- Highlight decisions that may require TSC approval.
- Prepare contrasting examples of:
  - A highly constrained Zephyr system.
  - A current mainstream Zephyr system.
  - A high-end platform where the existing system model becomes restrictive.
- Identify existing building blocks such as POSIX, userspace, memory domains, llext, filesystems, SMP, MMU support, shell, tracing, and runtime statistics.
- Identify cases where richer functionality currently leaks complexity into unrelated configurations.
- Identify cases where vendors or users have built Linux-like or process-like functionality outside Zephyr because existing abstractions were insufficient.
- Bring at least one example where a familiar Linux abstraction could be useful but should clearly remain optional and layered.

### Participants

Participants should:

- Read the document before the event.
- Add missing evidence or examples.
- Comment on the proposed scope.
- Identify constraints that may have been overlooked.
- Indicate strong objections before the session where possible.
- Consider which assumptions in their subsystem are driven by historically small hardware rather than fundamental Zephyr architecture.
- Identify functionality their subsystem would need in a richer execution environment.
- Consider whether that functionality belongs:
  - In core Zephyr.
  - In a reusable optional system layer.
  - In POSIX.
  - In an application framework.
  - Outside the Zephyr project entirely.
- Come prepared to answer the central architectural question:
  **What should a developer running Zephyr on a powerful SoC be able to do that they cannot reasonably do today — and how do we add that without costing a 64 KB MCU anything?**
