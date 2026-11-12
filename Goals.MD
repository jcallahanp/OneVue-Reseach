# Goals
User-defined. Agents edit only with user approval.

## Core Threat Model (fixed by user)
- Target: Pearson VUE **OnVUE** online proctoring (confirmed).
- Attacker objective: run the OnVUE exam in one window AND research answers
  (LLM / answer bank) on the **same single physical machine**, undetected.
- Deployment being defended: 360° front camera; confirm no confederates / no
  unapproved tech; then **no screen monitoring and no monitoring of the
  candidate's computer** during the exam.
- Industry: highly technical certification; exploitation skills are commonplace
  among candidates.
- IN SCOPE: software-only methods on one machine.
- OUT OF SCOPE: hardware methods, confederates, proxy test-takers, second
  physical devices.
- Deliverable posture: defensive threat model for the exam owner (Persona B).
  Analytical level — technique categories, detection-gap rationale,
  countermeasures. Not a turnkey operational cheating playbook.

## Deployment Clarification (user-supplied 2026-11-12 — refines the model above)
- User is a **current OnVUE sponsor/customer** (has used it in production) now in an
  **evaluation phase for alternative solutions**. (Corrected 2026-11-12; earlier
  "evaluating, not yet a sponsor" was a misstatement.) → Authorization = Branch A.
- Real environment = **test CENTER with center-OWNED machines**, NOT home BYOD.
  - Center machines are **low compute** (likely cannot run a full local LLM).
  - Endpoints currently **poorly locked down** ("not guarded very well") — but the
    center OWNS them, so firmware/boot/endpoint hardening is available to the owner.
  - **4-hour** on-site exam; **one proctor per many screens**, loose monitoring.
  - Candidate cannot pre-stage their own primary machine; may bring removable media.
- Implication: owner controls the endpoint → firmware/boot lockdown + USB + network
  egress control become the dominant defenses (unlike the BYOD Persona B analysis).
- User's two priority deepenings:
  - D1 (→T13): Is methodology #1 (local tooling) feasible here given low compute +
    4h + must-pre-engineer? Can they run the LLM via a VM, or is it detected?
  - D2 (→T14): Type-1 bare-metal hypervisor — what open-source tools exist, could a
    student install one within ~1h on a poorly-locked machine, is it resource-heavy,
    would OnVUE detect it, and the best preventions?

## Home-vs-Center reconsideration (user-supplied 2026-11-12)
- User is weighing ALLOWING AT-HOME testing: 360° pre-check + a LIVE room camera,
  but NO screen monitoring (hard to enforce). vs. in-person on center-owned machines.
- New depth required (research slice):
  - D3 (→T16): Hypervisor — home vs center discrepancy + OnVUE DETECTION STAGES.
    "Is it just the install I'd catch them at?" At what stage does OnVUE capture a
    VM/hypervisor? Does home (BYOD, pre-install, no firmware lockdown, unlimited prep)
    make the hypervisor a bigger threat, and could a prepared candidate stand it up
    quickly ahead of time? Is it still secondary to methodology #1 at home?
  - D4 (→T17): Copy-paste/clipboard — can OnVUE see/log/block copy-paste in-exam? If
    the LLM is on the HOST and OnVUE runs in a Type-1 GUEST, can OnVUE capture clipboard
    transfer across the VM boundary?
  - D5 (→T18): Best approach to guardrailing HOME testing — or is in-person on locked
    machines the only path to high assurance? Honest home-vs-in-person assurance call.

## G1 OnVUE feasibility (core question)
Thorough inventory, far beyond typical strategies, of ANY software-only method
to run the exam + simultaneous answer research on one machine without OnVUE
flagging it; each entry annotated with the specific OnVUE control it defeats or
evades and the residual detection risk.
| done when: inventory complete AND Persona A + B answerable (below).

## G2 Alternatives
Identify proctoring vendors whose technical controls close the gaps found in G1,
stating which specific gap each control closes.
| done when: vendors mapped to the G1 gaps they would catch.

## G3 Bypass test design
Design (not run) tests to confirm whether each top-ranked G1 strategy evades
OnVUE on the user's own setup. Design only; execution needs user approval.
| done when: each top strategy has a test (hypothesis, method, pass/fail).

## Acceptance tests (done when both are fully answerable)
- PERSONA A: advanced test-taker, multiple home computers, years of experience,
  no external hardware. "What strategy do I use to run OnVUE + an LLM/answer
  bank on my own machine without being detected?"
- PERSONA B: certification owner who knows Persona A exists. "What rules and
  proctoring procedures do I put in place, and what vendor do I select, to make
  the current detection holes far harder to circumvent?"

## Concerns (user-supplied)
- C1: OnVUE cannot see the candidate's screen or computer in this deployment;
  only a front 360° camera + OnVUE's own software telemetry.
- C2: Candidates are technically sophisticated (exploitation common).
- C3: Specifically worried about advanced software methods to run a concealed
  second browser/LLM workspace on the same machine undetected.
