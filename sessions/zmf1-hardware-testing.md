# Hardware Testing

[← Agenda](../README.md) · ZMF#1 · 2026-10-06 · 1:45 PM - 3:30 PM

## Key Speakers, Leads, and Required Attendees

- Dan Kalowsky, Qualcomm (Lead Organizer & Moderator)
- Parthiban, Luminiz (co-organizer)

## Decision or Outcome Sought

State in one or two sentences what the session should accomplish.

**By the end of this session, participants should have:**

- Agreed on:
  - How tiered board support will be implemented
  - Where boards used for HIL will be housed
  - Where HIL tests will be utilized in the codebase
  - If no, how does the Zephyr project leverage the information from hardware vendors already collected for making PR and release determinations?
- Identified:
  - How shall the project handle board failures that are not included in the HIL?
  - What additional features shall be investigated?
    - Remote debugging
    - Snapshot of firmware+logs to replicate failures
    - Remote flashing
    - Support for twister fixtures
    - LLM support
  - What challenges exist around long term usage/running?
    - Board failures
    - Point of Contact
    - Priority of fixing on-site issues
    - ???
  - What is the effort required to implement?
  - [Open issue requiring follow-up]
- Assigned:
  - [Owner and next action]

## Problem Statement

At over 1000 officially supported boards, the Zephyr project has grown to the point beyond any developer being able to test all board configurations for a change. Currently, the Zephyr CI system depends heavily upon emulators (QEMU, FVP, etc) which have served well. Yet these solutions are proving not always suitable for testing entire classes of boards, devices, and code changes that Zephyr now encompasses. Pull Requests that refactor code or add new features are passing our CI, only to be (sometimes weeks) later reverted due to the discovery of an edge case on a silicon platform. As the Zephyr project continues to grow, ensuring the stability of the code and signaling more confidence to PR submitters and Release team members on changes will be essential.

Example of the issue:

- <https://github.com/zephyrproject-rtos/zephyr/pull/111679>
  `arch: arm: mpu: declare arm_core_mpu_enable/disable in header`
- Reverted by <https://github.com/zephyrproject-rtos/zephyr/pull/112024>
- Passed all CI tests
- Failure on NXP boards

Current state, as far as I can tell:

- Twister is not the solution.
  - There is some overlap.
  - It is seductive to extend twister into becoming a HIL test harness. Must resist this.
  - Any solution must work with Twister + west.
- Some vendors add results to <https://github.com/zephyrproject-rtos/test_results>
  - Weekly not per PR giving delay to discovery
- There is at least one vendor trying to add HIL results on a PR basis (<https://github.com/zephyrproject-rtos/zephyr/pulls?q=is%3Apr+%22Hardware-in-the-Loop+Test+Results%22+is%3Aclosed>).
  - An example output can be seen at <https://github.com/zephyrproject-rtos/zephyr/pull/101608#issuecomment-3760063560>.
- A search through incoming PRs for “HIL” results proving users are doing some type.
  - The project would benefit by unifying the reporting format for PRs and discussions.

## Scope

### In scope

- Which twister fixtures would benefit most from hardware testing.
- Which sections of Zephyr code would benefit most from hardware testing.
- Updating current infrastructure and designs to support boards.

### Out of scope

- Addressing PR and code review short falls. There are opportunities to improve both, which are open for discussion in the Process Working Group.
- Simplifying processes, such as SDK updates, release, PR notifications, etc. There are opportunities to improve here that should be brought before the Process Working Group.
- Budgets - from emulation resources to infrastructure. While important, let us first focus on our ideal scenario then work on reducing to what we can accomplish.
- Negative commentary on vendors participation.
- Removing the use of twister.

## Questions Requiring Agreement

### D1: Hardware Based Testing

**Decision required:** Does the project still desire hardware in the loop testing in CI/CD?

Some background:

- First identified as an issue by the TSC in 2017 (<https://lists.zephyrproject.org/g/tsc/message/72>) the TSC identified trying to create a service similar to [kernelci.org](http://kernelci.org).
- The 2025-04-02 Joint Zephyr TSC/Board Steering meeting brought up the need again for hardware testing (<https://drive.google.com/file/d/1UI44uW6r9hgQzzs1uoJGCw8lvhUQZFxN/view?usp=sharing> Slide 7)
- The 2025-07-16 TSC meeting expresses a dislike towards the project becoming the place to determine if hardware and boards are supported (<https://drive.google.com/file/d/1UI44uW6r9hgQzzs1uoJGCw8lvhUQZFxN/view?usp=sharing>). Preferring the vendors be the source. This appears to be counter to the idea of a [kernelci.org](http://kernelci.org) like service.
- The 2025-08-28 Zephyr TSC meeting made it clear the community really wants hardware in the loop testing (<https://drive.google.com/file/d/1UI44uW6r9hgQzzs1uoJGCw8lvhUQZFxN/view?usp=sharing> see multiple bullet points in Lunch topic).

**Possible outcomes:**

- Yes
- No

### D2: Tier Revival

**Decision required:** Does the project still desire to use the previously proposed hardware tier system in CI/CD?

Some Background:

- The Process Working Group began exploring a priority tiered hardware system in 2021 (<https://github.com/zephyrproject-rtos/zephyr/issues/38566>), with the goal of enabling Tier 1 boards as a hardware run requirement.
- The Process Working Group concept was brought to a TSC vote in September of 2022 (<https://lists.zephyrproject.org/g/tsc/message/1464>) and passed on a vote. Unfortunately there appears to be limited further efforts.

**Possible outcomes:**

- Yes
- No

### D3: Tier Expectations

**Decision required:** What roles do each tier encompass?

**Outcome Matrix:** (based upon previous Process Working Group discussions)

| Tier | PR | Nightly / Weekly | RC and Release | No support? |
| :-: | :-: | :-: | :-: | :-: |
| Tier 1 | X | X | X | |
| Tier 2 | | X | X | |
| Tier 3 | | | X | |
| Tier 4 | | | | X |

### D4: Test Suites in Tiers

**Decision required:** Which Zephyr tests suites belong to which tier on a per PR basis? Which areas of the code base would benefit the most from Hardware in the Loop testing for CI?

| Top Level | Tier | Detailed List |
| :-: | :-: | :-: |
| **arch** | | |
| **boards** | | |
| **drivers** | | |
| **dts** | | |
| **kernel** | | |
| **lib** | | |
| **modules** | | |
| **snippets** | | |
| **soc** | | |
| **subsys** | | |
| **tests** | | |

### D5: Board Hosting

**Decision required:** Where shall boards for the HIL CI/CD be hosted?

Where the test boards are hosted introduces several challenges. For example, ensuring uptime of scheduler host and workers, regular upgrade/replacement of boards, and consistent network and power speeds to hosts all need to be considered.

What about boards that need to have rework to support HIL testing?

Usage in various scenarios also introduces engagement requirements. For example, board support testing for an LTS must encompass the lifetime of the LTS.

**Outcomes Matrix:**

| Location | Tier 1 | Tier 2 | Tier 3 | No |
| :-: | :-: | :-: | :-: | :-: |
| Project Member | | | | |
| Vendors | | | | |
| Zephyr Project | | | | |
| Individuals | | | | |

### D6: Scheduler Framework

**Decision required:** Does the project care to have a scheduler framework for CI/CD?

In broad strokes, the process for submitting to a HIL process will likely take on the form of:

1. PR submission
2. Zephyr CI checks what tests and if any boards are needed for the PR through `scripts/ci/test_plan_v2.py`.
3. Twister builds the test artifacts for each board.
4. The twister built artifacts are submitted for testing to a test scheduler.
5. Test Scheduler distributes the job to a worker with the correct devices attached to it.
6. Test worker runs twister with the pre-built artifacts.
7. The PR is gated until the HIL test is completed and results collected.

Two options exist for following this pattern.

**Option 1** - the Zephyr project decides upon a framework and hosts a scheduler that distributes jobs to worker clients connected to it. An alphabetical list of (currently known) possible frameworks:

- LabGrid
- Latchport
- Linaro LAVA

There are talks at this conference (example: <https://osselceu2026.sched.com/event/2RaYr/demystifying-board-farms-build-a-mini-desktop-board-farm-with-standard-tools-and-hardware-francesco-cervigni-neoncomputing?iframe=no&w=&sidebar=yes&bg=no>) giving more detail about all of these options. Please attend one if you’re curious about details.

Advantages:

- Clear way to document and ramp up new workers as needed.
- The Zephyr project can easily disable/enable boards when needed.
- Clear performance monitoring knobs.
- Upgrades to workflow are easier.

Disadvantages:

- Sites with already established HIL infrastructure may not want to convert/support any alternative framework.
- Another service that Zephyr infrastructure must host.
- May require vendors to have two HIL systems, external public and internal private.
- Zephyr CI may still be in the path and very busy.

**Option 2** - The Zephyr project stays framework agnostic and only focuses on the resulting twister run outputs.

Golioth has shown it is possible 3 years ago:

- Zephyr Tech Talk 1 - <https://www.youtube.com/live/940O1CUgh4Q>
- Blog Post - <https://blog.golioth.io/golioth-hil-testing-part1/>

Advantages:

- Already established HIL infrastructures can be used provided all accept a GitHub API based submission.
- The Zephyr CI is already good at parsing twister output.
- Can reduce the Zephyr CI load by having build on downstream consumers.

Disadvantages:

- Limited to no insight into infrastructure failures.
- Will require coordination on GitHub access tokens to submit work.
- Difficult to update workflows.

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
