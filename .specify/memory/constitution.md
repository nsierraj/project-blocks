<!--
Sync Impact Report
Version change: template (unversioned) → 1.0.0
Modified principles: none (first ratification; all principles are new)
Added principles:
  - I. Original IP (NON-NEGOTIABLE)
  - II. Deterministic, Isolated Game Core
  - III. Test-First for Rules
  - IV. Performance and Input Budget
  - V. Accessible, Private, Resilient by Default
  - VI. Scope Discipline
Added sections: Platform Constraints; Development Workflow; Governance (filled)
Removed sections: none
Dependent templates: plan, spec and tasks templates and commands read this constitution at
  runtime; none were modified by this amendment.
Deferred items:
  - Project name is provisional ("Project Blocks") pending BRD open item O1. Setting the
    final name is a PATCH amendment.
  - Technology stack is intentionally unspecified (BRD open item O2). Choosing one during
    planning does not require an amendment unless it conflicts with a principle.
-->

# Project Blocks Constitution

## Core Principles

### I. Original IP (NON-NEGOTIABLE)

The project MUST NOT use any trademarked name, logo, trade dress or official asset of an
existing falling-block puzzle game. The public name, repo name, page title and app manifest
MUST NOT contain the genre-defining trademark or a confusingly similar name. Visuals and sound
MUST be original or royalty-free, with their source recorded. Piece appearance, board layout
and ghost and next-piece displays MUST follow an original visual style rather than a copy of
any licensed release.

Rationale: trademark and trade-dress claims are the highest-impact risk to a publicly hosted
portfolio piece (BRD §2, R1, acceptance criterion 6, F38).

### II. Deterministic, Isolated Game Core

Game rules (rotation, wall kicks, lock delay, scoring, randomizer, line clears) MUST live in
pure logic with no dependency on rendering, input devices, audio, storage or the DOM. The core
MUST advance on a fixed timestep that is independent of display refresh rate, so 60, 90 and
120 Hz screens play at the same speed. Rendering and input layers MUST only call into the
core's interface and MUST NOT implement rules themselves.

Rationale: separation makes the rules unit-testable and keeps timing identical across devices
(N6, N20, R4).

### III. Test-First for Rules

Automated tests for SRS rotation and wall kicks, T-spin detection, scoring and the 7-bag
randomizer MUST be written before or together with the code they cover. A failing rule test
MUST block merge. A change to a rule MUST include a test that fails without the change.

Rationale: these rules are well-defined by the genre guideline and easy to get subtly wrong
(N21, acceptance criterion 2).

### IV. Performance and Input Budget

The game MUST hold 60 fps on a mid-range phone from the last three years and on desktop.
Input events MUST be applied on the next frame after they occur. First load MUST complete in
under 2 seconds on a typical 4G connection, and total download size MUST stay under 500 KB.
A change that breaks any of these budgets MUST be fixed or justified in the plan's complexity
tracking before merge.

Rationale: responsiveness on both keyboard and touch is a stated objective (OB2, N5–N8).

### V. Accessible, Private, Resilient by Default

- Every piece MUST be distinguishable without color, using a pattern or shape mark, and the
  palette MUST be colorblind-safe.
- Text and UI contrast MUST meet WCAG 2.2 AA.
- The OS reduce-motion setting MUST suppress screen shake, flashes and particle bursts.
- Menus MUST be fully keyboard-usable with visible focus states and screen-reader labels.
- The game MUST pause automatically when the tab loses focus or the app is backgrounded.
- The game MUST NOT collect personal data or use third-party trackers or cookies.
- The game MUST run normally when browser storage is blocked; scores are then simply not
  saved.

Rationale: accessibility, privacy and graceful degradation are baseline quality, not polish
(N12–N18, F37, OB5).

### VI. Scope Discipline

The out-of-scope list in the BRD (online leaderboards, accounts, multiplayer, background
music, on-screen buttons, gamepad, monetization, tracking analytics, native store builds) is
binding. Work on any such item MUST NOT start until the BRD is revised to include it. Each
delivery phase MUST end with something playable before the next phase begins.

Rationale: a single developer with no deadline is at constant risk of scope creep (R6).

## Platform Constraints

- Hosting MUST be static (GitHub Pages) over HTTPS, with no server-side code.
- Supported browsers: latest two versions of Chrome, Edge, Firefox and Safari on desktop; and
  latest two versions of Safari on iOS/iPadOS, Chrome on Android and Samsung Internet (N1, N2).
- Layout MUST work from 360 px width up to desktop, in portrait and landscape, with portrait
  as the primary mobile layout (N3).
- The game MUST be installable as a PWA and fully playable offline after the first visit.
  Updates MUST be picked up on next launch without breaking saved scores or settings
  (N9–N11).
- The technology stack is chosen during planning and MUST satisfy the constraints in this
  section and the principles above.

## Development Workflow

- Work follows the BRD delivery phases in order; each phase MUST end playable.
- Commit history MUST be clean and readable, since the repo is a portfolio artifact (OB4).
- The README MUST cover how to run, build, test and deploy, and MUST be kept current with
  any change that affects those steps (N22).
- Every implementation plan MUST include a Constitution Check that verifies compliance with
  each principle above before design starts and again after design.
- Detailed rules, scoring tables, key maps and mode definitions belong in feature specs and
  the BRD, not in this constitution; specs and tasks MUST cite BRD requirement IDs.

## Governance

This constitution supersedes other practice documents for this project. The business
requirements document (BRD) in the repository root is the source for product requirements and
the guidance reference for runtime development; where the two conflict on governance, this
constitution wins.

**Amendment procedure**: Amendments are made by editing this file with a Sync Impact Report,
an updated version line and an updated Last Amended date. The owner approves every amendment.
An amendment that changes a principle MUST list any plans, specs or tasks that need follow-up.

**Versioning policy** (semantic):

- MAJOR: backward-incompatible principle removals or redefinitions.
- MINOR: a new principle or section, or materially expanded guidance.
- PATCH: clarifications, wording, typo fixes and other non-semantic refinements.

**Compliance review**: Every plan and code review MUST verify compliance with this
constitution. Any violation MUST be fixed or recorded with a written justification in the
plan's complexity tracking.

**Version**: 1.0.0 | **Ratified**: 2026-10-26 | **Last Amended**: 2026-10-26
