# Safety & Code Quality

[← Agenda](../README.md) · ZMF#5 · 2026-10-06 · Part I: 9:00 AM - 10:45 AM · Part II: 11:00 AM - 12:45 PM

## Key Speakers, Leads, and Required Attendees

- Tobias K, Inovex (Zephyr Safety Architect, lead host)
- Kate Stewart, Linux Foundation (co-host)
- Alberto Escolar Piedras, Nordic (co-host)
- Anas Nashif, Intel (co-host)
- Pieter De Gendt (co-host)
- Nicole Pappler, Alektometix (Safety Manager)
- Hanuobu Kurokawa, Renesas
- Alexandre Bailon, Baylibre

## Decision or Outcome Sought

Work towards

State in one or two sentences what the session should accomplish.

**By the end of this session, participants should have:**

- Agreed on:
  - [Decision or principle]
  - [Decision or principle]
- Identified:
  - [Open issue requiring follow-up]
- Assigned:
  - [Owner and next action]

Avoid outcomes such as “discuss the issue” or “raise awareness.” Describe what should be different after the session.

## Problem Statement

Describe the specific problem the session is intended to solve.

Include:

- What is not working today?
- Who is affected?
- What are the consequences if it is not addressed?
- Why does this require a cross-project discussion?

**Problem statement:**

Which standard configurations should we target? How document?

- Defaults need to be documented, and what retest if configuration changes.
- Which can be changed and which can not be changed.
- Symbols - changed to non compliant value. Leaving safety scope.

Integration Assumptions

- What are basic assumptions of use?
- Where do we store them (strictdoc, docs, ??)

How to extend to Network Stacks, USB, Missing Drivers, etc.

- Getting contributions of analysis - back to extend our scope?
- Maintainers sign off on requirements to be "follow our methodology" & tests contributed to prove it out.
- Paying others to extend beyond and contribute to upstream?

Isolation of certain component?

- time, memory, etc.
- FFI

New contributions best practices

- Where capture use cases
- Safety scope evolution

Tool qualification:

- What is our "[set of tools](https://docs.google.com/spreadsheets/d/109fU0XZ_yTD54x5dlS-ppZPpsGZodRbL00ZmqOoEalc/edit?usp=drive_link)" that we depend on.
- Toolchains - GNU/LLVM/IAR
- Scripts - west, twister, strictdoc,
- Need to write down expectations from ISO 20262 - Name, Version, Vendor, Usage Manual, known bugs, (Normal SBOM + Requirements + Test SPec & Results)
- need to be "evaluated"
  - Level of dependency
  - Artifacts for safety analysis that depend on it
  - Minimum: Name, Version, Vendor, etc (SBOM specific)

Companies are looking for:

Safety expansion and use

1. Kernel
2. Drivers
3. Network stacks
4. FS
5. Arch / Processor
6. Libraries
   1. Rust, etc.
7. Application
8. Compiler

Responsibility

- Creation --
- Maintainers -- signoff that it's
- Coordination

RASC - responsible/support/?

## Scope

### In scope

- [Area to be addressed]
- [Area to be addressed]
- [Area to be addressed]

### Out of scope

- [Related topic that will not be resolved in this session]
- [Implementation details to be handled later]
- [Topics requiring a separate session]

Clearly defining what is out of scope prevents the discussion from becoming too broad.

## Questions Requiring Agreement

Frame the session around a limited number of explicit questions.

### Decision 1: [Question]

**Decision required:**
[Specific question participants must answer]

**Possible outcomes:**

- [Option]
- [Option]
- [Option]

### Decision 2: [Question]

**Decision required:**
[Specific question]

**Possible outcomes:**

- [Option]
- [Option]
- [Option]

Repeat for no more than five or six primary decisions.

## Proposed Session Agenda

Not too strict agenda, but we need to manage the discussion and time box it, i.e. for a 2 hour session

- Present the problem / proposal (~20-30 minutes)
- Discuss (1 hours)
- Closing (30 minutes)

## Pre-Session Preparation

### Session owner

Before the event, the session owner should:

- Complete Sections above.
- Provide links to relevant issues, pull requests, and documentation.
- Identify two or three representative examples.
- Describe realistic options rather than only the preferred solution.
- Highlight decisions that may require TSC approval.

### Participants

Participants should:

- Read the document before the event.
- Add missing evidence or examples.
- Comment on the proposed scope.
- Identify constraints that may have been overlooked.
- Indicate strong objections before the session where possible.
