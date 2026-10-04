# Long Term Topics w/o Resolution (Clock Management focus)

[← Agenda](../README.md) · ZMF#3 · 2026-10-06 · 11:00 AM - 12:45 PM

## Key Speakers, Leads, and Required Attendees

- Daniel De Grasse, (Lead Organizer & Moderator)
- Carles Cufi (Co-organizer)
- Eric Hay (Co-organizer)
- Erwan Gouriou (ST, key stakeholder for clock management requirements)
- Tom Burdick (Infineon, key stakeholder for clock management requirements)
- Bjarki Andreasen (Nordic, key stakeholder for clock management requirements)

## Key Background Information

Original Clock Management PR, with key discussion and framework evolution:
<https://github.com/zephyrproject-rtos/zephyr/pull/72102>

Current Clock Management PR, with ongoing discussion (includes implementation for LPC55S69):
<https://github.com/zephyrproject-rtos/zephyr/pull/89124>

Implementation of Clock Management for ADI MAX32:
[github.com/danieldegrasse/zephyr/pull/6](http://github.com/danieldegrasse/zephyr/pull/6)

Alternative approaches for Clock Management that rework clock control:

- <https://github.com/zephyrproject-rtos/zephyr/pull/96315>
- <https://github.com/zephyrproject-rtos/zephyr/pull/119332>

Issue tracking Clock Management use cases and requirements:
<https://github.com/zephyrproject-rtos/zephyr/issues/111406>

Clock Management Talks:

- <https://youtu.be/Gb6EWbJvZMo?si=bHrYZRP0YFf9M_9p> (2025)
- <https://youtu.be/G4-71BuSJb8?si=yT_8kXtFIFgRCJds> (2024)

## Decision or Outcome Sought

This session aims to resolve long term discussions within the Zephyr community that haven’t reached a resolution via the normal working channels. The primary focus will be on clock management, with additional time allocated for other topics if allowed

**By the end of this session, participants should have:**

- Agreed on:
  - A portable devicetree description for clocks in Zephyr
  - An approach for how to handle clock reconfiguration events in Zephyr
  - A set of requirements for the clock management framework
  - A target timeline for completion of the initial clock framework
- Identified:
  - A migration path for existing vendor drivers to move to clock management
  - A migration plan for customers using Zephyr to move to clock management
- Assigned:
  - Stakeholders for porting key SoC vendor drivers to clock management
  - Key reviewers for clock management framework who need to approve the changes prior to merge

## Problem Statement

This session aims to solve clock management shortcomings in Zephyr. Currently, there is no standardized way to reconfigure clocks within Zephyr, and no standardized interface for setting up clocks at boot time. This limits the reusability of vendor agnostic drivers, and restricts power management possibilities for SoCs with advanced clock trees.

If these issues are not addressed, vendors will likely seek to implement their own custom solutions for managing clock trees and power states. This will lead to fragmentation and code duplication across vendor drivers, as well as maintenance difficulties in vendor agnostic drivers

This issue requires a broader discussion in the project because clocking is fundamental to SoC bringup, and any modification to the support in Zephyr will need to be supported and accepted by all major vendors engaged with the project.

## Scope

### In scope

- Devicetree formatting for Clock data
- Handling clock change events in Zephyr
- Migration plan for vendors to use clock management

### Out of scope

- Power management support around clocks
- Vendor-provided tooling for clock tree validation/generation

Clearly defining what is out of scope prevents the discussion from becoming too broad.

## Questions Requiring Agreement

Frame the session around a limited number of explicit questions.

### Decision 1: How can we effectively represent clock settings at build time?

**Decision required:**
In order to represent multiple potential clock configurations statically, we need to describe them at build time. How can we do this in a way that minimizes build footprint?

**Possible outcomes:**

- SOC-wide devicetree based states
- Per peripheral partial device-based states
- YAML or other format

### Decision 2: How can we support runtime clock reconfiguration?

**Decision required:**
In order to support power management states, we need to enable SoCs to reconfigure their clocks at runtime. How do we perform this in a manner that allows clock consumers to respond to clock frequency or property changes?

**Possible outcomes:**

- SOC wide state transitions, that notify all consumers in the system of a new rate
- Per peripheral clock state transitions, that only notify consumers when their clock is affected

### Decision 3: How should we transition to clock management?

**Decision required:**
Clock management has been designed to be able to coexist with clock control, to enable a seamless transition between the frameworks. The expectation currently is that we will eventually deprecate clock control code entirely. How should we enable this transition?

**Possible outcomes:**

- Require all SoC vendors to transition to clock management. This should provide a more consistent user experience but will require much more work from vendors.
- Implement clock control on top of clock management, reducing the API churn for users while enabling SoC vendors to transition at their own pace

### Decision 4: Do we need static validation of clock settings?

**Decision required:**
Static clock states are currently defined by the user, and can apply invalid settings. Since clock settings are defined per node, it is difficult to validate the clock tree at the system level. Should we implement support for this feature?

**Possible outcomes:**

- Implement support for static validation. Will likely require additional vendor code to run build time checks, which will be very SoC specific.
- Rely on user knowledge and vendor tooling to handle validation of clock state definitions

### Decision 5: Should we continue to invest effort in clock control?

**Decision required:**
Several proposals have been put forward for improving clock control without significant API rework. These proposals do not enable runtime clock reconfiguration, but they do enable generic interaction with the clock control framework (which is not currently possible). Should we work towards merging one of these solutions?

**Possible outcomes:**

- Work towards improving clock control while clock management continues to evolve
- Keep clock management API stable to reduce churn for users, only perform one transition to clock management

### Decision 6: Should we make clock reconfiguration opt-in?

**Decision required:**
Clock reconfiguration *notifications* are currently opt-in, but the operation of reconfiguring clocks is always available. Should we enable this option in all cases, or make it opt-in to avoid users potentially performing dangerous clock changes?

**Possible outcomes:**

- Allow users to change clocks, even if notification features are disabled. Dangerous but enables more user control
- Prevent users from reconfiguring clocks unless notifications are enabled. Requires users to take footprint hit to enable clock reconfiguration, even if they know their reconfiguration is safe to apply without notifications

## Proposed Session Agenda

Not too strict agenda, but we need to manage the discussion and time box it, i.e. for a 2 hour session

- Background slides on the current state of clock management, and the required issues to solve (20 minutes)
- Discuss devicetree representation (40 minutes)
- Discuss runtime notifications (20 minutes)
- Discuss framework transition plan (10 minutes)
- Closing/next steps (30 minutes)

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
