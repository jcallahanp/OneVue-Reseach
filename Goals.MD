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
