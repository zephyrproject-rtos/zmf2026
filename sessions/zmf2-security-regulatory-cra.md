# Security, Regulatory and CRA Discussion

[← Agenda](../README.md) · ZMF#2 · 2026-10-06 · 3:45 PM - 5:30 PM

## Key Speakers, Leads, and Required Attendees

- Martin Jäger, A Labs & Libre Solar (Lead Organizer & Moderator) \*
- Kate Stewart, Linux Foundation (Co-host) \*
- David Brown, Linaro (Security Chair)
- Flavio Ceolin, Hubble Network (Security Architect)

## Decision or Outcome Sought

**By the end of this session, participants should have:**

- Agreed on:
  - Process how vulnerabilities can be automatically matched with an application based on Zephyr
  - To what extent the project shall support product developers with necessary steps for certification
- Identified:
  - Gaps in documentation
  - Helpful/necessary additional tooling (e.g. as part of west)
- Assigned:
  - [Owner and next action]

## Problem Statement

The EU Cyber Resilience Act imposes obligations on manufacturers placing connected products with digital elements on the EU market, with vulnerability-reporting obligations applying from September 2026 and the main essential requirements from December 2027.

Zephyr already has tools e.g. for SBOM generation in place. However, becoming (and staying) CRA compliant requires much more:

- Risk Assessment considering possible threats and attack vectors of a particular product
- Continuous monitoring of published vulnerabilities and matching with used software components (based on the SBOM)
- Reporting, fixing of vulnerabilities and deploying the fixes to devices in the field

Within this workshop, we would like to discuss security and regulatory best practices (with special focus on CRA) with the Zephyr community.

Many parts of the overall compliance process are out of scope of the Zephyr project itself. However, it could still be helpful to give pointers, discuss existing and potential new helpful tools that integrate well with Zephyr projects, and guide Zephyr users where possible.

Relevant issues and PRs:

- **Documentation**
  - RED Cybersecurity Requirements (EN 18031-x)
    Issue: <https://github.com/zephyrproject-rtos/zephyr/issues/85687>
    PR: <https://github.com/zephyrproject-rtos/zephyr/pull/120713>
  - WIP Harmonized Standards for CRA (EN 40000-1-x)
    PR: <https://github.com/zephyrproject-rtos/zephyr/pull/120742>
- **SBOM, SPDX and Metadata**
  - Vulnerabilities/CVE in Zephyr modules
    <https://github.com/zephyrproject-rtos/zephyr/issues/53479>
- **Automated Vulnerability Tracking**
  - Yocto project cve-check equivalent for vulnerability check at build time
    <https://github.com/zephyrproject-rtos/zephyr/issues/85570>
- … ?

## Scope

### In scope

- Security and regulatory best practices for Zephyr-based products (incl. vulnerability monitoring)
- Available and necessary tools specifically for the Zephyr ecosystem
- Identifying gaps in documentation and tooling

### Out of scope

- Generic risk assessment tools
- Any product-specific processes and tools

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

- Regulatory Frameworks (15 min)
- Processes and Tooling (15 min)
- Discussion (1 hours)
- Wrap-up (15 minutes)
