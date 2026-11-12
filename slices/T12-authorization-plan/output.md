# T12 Test Execution Authorization Plan

**What this is.** The governance gate the certification owner must clear **before** running any of the
14 owner-run OnVUE detection tests designed in T09 (`T09-01`…`T09-12`, `T09-NC1`, `T09-NC2`). It is a
defensive QA / red-team governance document. **It authorizes nothing by itself** — it defines the
approvals, scoping, and safeguards that must exist first, and the checklist that must pass before each
run. Design/grounding from T09 (`/home/user/OneVue-Reseach/slices/T09-bypass-tests/output.md`, cited
T09-nn) and the OnVUE control surface from T05
(`/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md`, cited T05 §n). External sources
numbered in §10, access date **2026-11-12**.

> **Not legal advice.** The legal/ToS material in §3, §6, §7 is general informational guidance.
> Qualified counsel (and a privacy officer / DPO) must review and sign off before any run. Where a
> source could not be confirmed it is labeled **[unverified]**; no Pearson policy is fabricated.

---

## 1. Purpose & what this gates (ties to T09; authorizes nothing alone)

- **Gates:** execution of the T09 harness — a *detection-measurement* instrument that measures whether
  OnVUE detects each concealment strategy, run in a sandbox against a **practice / non-live** exam with
  **consenting internal testers** (T09 purpose; T09 Authorization statement). Every T09 block already
  carries "Design only — not run" and a per-test "Coordinate with Pearson/sponsor before any run."
  This plan is the procedure that turns that standing precondition into named approvals.
- **Does NOT gate / does NOT grant:** any probe of Pearson production, any live exam or booking, any
  real candidate's data, or any VM/remote-control/multi-display arm against a real delivery (T09-07/08,
  T09-10, T09-NC1, T09-NC2 exercise behaviors OnVUE policy **expressly prohibits** — T05 §4–7).
- **Decision produced:** a signed, scoped, time-bounded authorization (or a No-Go) covering a specific
  tester pool, exam form, OS set, and test tier. Absent all §8 gate items = **No-Go by default**.

## 2. Required inputs to confirm first (do not assume)

| # | Input | Why it gates | Status |
|---|-------|--------------|--------|
| I1 | **Owner's exact relationship to Pearson** — (A) already a **contracted test sponsor** delivering via OnVUE, vs (B) **evaluating / prospective**, no contract | Determines whether a contractual channel to request a sanctioned test/sandbox exists (§3 branch), and whether *any* Pearson-side contact is permitted. **UNKNOWN — confirm; do not assume.** | REQUIRED |
| I2 | **Specific exam form / sandbox** to be used | Must be the owner's **own practice / non-live** form or a Pearson-provided sandbox — never the live item pool (T09 validity note §8; T05 per-sponsor caveat) | REQUIRED |
| I3 | **OS set** in scope (Win 11/10 64-bit non-S-Mode; macOS 14+) and client version pinned | OnVUE behavior drifts by OS build, macOS floor (T05 §11 inconsistency 10.15 vs 14) and client version (3.27.x class, T05 §7); results invalid if unpinned | REQUIRED |
| I4 | **Tester pool** — named, internal, consenting adults; count; which are in BIPA/GDPR-style jurisdictions | Testers are **real people whose biometrics (facial geometry), ID images and room-scan video are captured** (T05 §10); consent + data duties attach per person (§5) | REQUIRED |
| I5 | **Sponsor/exam profile** (OnVUE config varies per sponsor — T05 Summary caveat) | Every result is valid only for the profile tested; record it | REQUIRED |
| I6 | **Isolated test environment** owner controls end-to-end | Scope-IN boundary (§4); keeps tests on the owner's own stack with least third-party exposure (§7) | REQUIRED |

## 3. Approval chain / sign-offs (who must say yes before Go)

All five required roles must sign for any run. **Pearson coordination is conditional on I1.**

| Role | Sign-off covers | Blocking? |
|------|-----------------|-----------|
| **Program owner** (certification owner/leader) | Final Go/No-Go; scope, budget, tester pool; accepts residual risk | Yes — final authority |
| **Legal counsel** | CFAA / "authorized access", DMCA §1201 anti-circumvention, ToS & Candidate-Rules / sponsor-agreement exposure (§6, §7); confirms practice-only, no production | Yes |
| **Privacy / DPO** | Biometric & PII consent, BIPA written consent + retention/destruction policy, GDPR basis, data-handling & deletion plan (§5) | Yes |
| **Security lead** | Rules of Engagement, environment isolation, evidence integrity, stop conditions, no live content leakage (§4) | Yes |
| **Pearson VUE / exam-sponsor coordination** | Written acknowledgment/authorization for anything touching Pearson systems or exercising policy-prohibited surfaces | **Branch-dependent (below)** |

**Branch A — owner IS a contracted sponsor (I1=A).** A contractual relationship exists. Pearson VUE's
test-owner security function is sponsor-facing: the **Secure Testing Framework™**, "develop a strategy
to fight [threats]," and "data analytics and forensics … Investigate and reconstruct fraudulent
incidents," with a "Partner with our team" contact route [src 3, 4]. **Required:** raise the
detection-testing intent through the owner's Pearson program/account manager or that test-owner channel,
obtain **written** acknowledgment, and request a Pearson-sanctioned practice/sandbox form. Pearson
coordination is **mandatory** before any Tier-2 (VM / remote-control / client-environment) arm.

**Branch B — owner is prospective / evaluating (I1=B).** No contractual channel to authorize testing
and **no permission to probe Pearson production**. Pearson publishes **no OnVUE-client penetration-test
or reverse-engineering authorization path** (confirmed: see §6). Route questions through Pearson VUE
sales / test-owner contact [src 4]; route any incidental client-level security finding to Pearson's
**Responsible Security Disclosure** address `responsible.disclosure@pearson.com` [src 1] — **not** the
test-security line, which is for testing-violation reports, not software vulnerabilities [src 2].
**Until a relationship/authorization exists: run Tier-1 only, on the owner's own stack, with
practice/placeholder content; defer all Tier-2 arms.**

## 4. Rules of Engagement

**Scope IN**
- The owner's **own practice / non-live exam form** or a Pearson-provided sandbox (I2).
- **Consenting, named internal testers** only (I4); no third party, no real candidate.
- **Isolated hardware/network** the owner controls end-to-end (I6); pinned OS + client version (I3).
- **Benign / placeholder content** in every treatment arm (overlays, inference, captures, injection use
  placeholder data — never the live item pool) (T09 validity note §8).

**Scope OUT (hard prohibitions)**
- Any **live candidate session**, live exam, or live booking.
- Any **live exam content** / real item pool.
- **Probing Pearson production** infrastructure or telemetry hosts (`*.onvue.com`, `*.proctorcam.com`,
  `*.wowza.com`, `*.ably.io`, `*.twilio.com`; T05 §10) — observe only what the owner's own sanctioned
  session exposes; do not attack, flood, or enumerate Pearson systems.
- Any **real candidate's data**, credentials, or account.
- **Reverse-engineering, patching, or tampering with the OnVUE client binary** unless explicitly
  cleared by counsel + (Branch A) Pearson (§6).

**Stop conditions (halt the run, preserve logs, escalate to Security lead + Program owner)**
- Any test unexpectedly reaches a live form, real candidate, or Pearson production path.
- A negative control (T09-NC1/NC2) fails to fire → harness mis-instrumented; treatment FAILs untrusted
  (T09 validity note §2) — stop and re-baseline.
- Tester withdraws consent, or biometric/PII capture occurs outside the consented scope.
- Legal/privacy sign-off lapses, client/OS version drifts, or sponsor coordination is withdrawn.

## 5. Environment & consent

- **Non-live only:** practice / sandbox exam form (I2); isolated, owner-controlled hardware (I6); no
  connection to any production delivery or real booking.
- **Testers are real people whose biometrics are captured.** OnVUE check-in takes a self-photo,
  front/back government-ID images, a room/workspace scan, and **facial-geometry biometric comparison**;
  the full session is webcam + mic recorded (T05 §10). Each tester therefore needs **written, informed,
  revocable consent** specific to: biometric (facial-geometry) capture, ID imaging, room-scan video, and
  audio/video recording — before any session.
- **BIPA / privacy duties (informational; DPO + counsel confirm):** Illinois BIPA requires **informed
  written consent** and a **written retention & destruction policy** before collecting biometric
  identifiers; exam-proctoring tools have drawn BIPA class actions (Respondus, Examity, ProctorU), with
  per-scan violation theories and statutory damages [src 8]. Treat GDPR/CCPA-style duties analogously
  for testers in those jurisdictions. Reference T05 §10 (Pearson retains biometric data "as long as
  deemed necessary by the test sponsor"; Illinois 3-yr cap noted).
- **Data-handling & deletion plan (pre-registered before Go):**
  - Log only **technique-liveness** and **OnVUE-signal** data; **retain no live exam content** (T09
    Authorization statement; validity note §8).
  - Minimize tester PII; store session recordings/biometrics encrypted, access-controlled, segregated.
  - Define retention limit and a **documented deletion step** executed at project close; give testers a
    withdrawal + deletion path.
  - Note: biometrics captured by OnVUE itself flow to Pearson under its policy (T05 §10) and are **not**
    in the owner's sole control — a factor counsel/DPO must weigh, and a reason to minimize real tester
    exposure and prefer the fewest sessions that yield N≥5 (T09 validity note §7).

## 6. Legal & ToS review step (informational, not legal advice — counsel sign-off required)

- **No published Pearson OnVUE security-testing / reverse-engineering authorization path found.**
  Pearson's **Responsible Security Disclosure Policy** is the only disclosure route located; it is scoped
  to "Pearson products and services which have a security.txt file in their root," routes to
  `responsible.disclosure@pearson.com`, and **prohibits** testing out-of-scope systems, **disabling or
  altering security controls**, accessing anyone's data, exfiltration, DoS, brute-force, and social
  engineering; researchers must "stop … after … the first security issue and request permission to
  continue," and it "does not give you permission to act in any manner … inconsistent with legal and
  regulatory compliance" [src 1]. It does **not** mention reverse engineering, and it is **not** an
  authorization to run detection probes against OnVUE. **Fallback = general best practice:** obtain
  explicit written authorization (Branch A Pearson coordination; Branch B none for production), stay on
  the owner's own stack, and have counsel review before any client-level manipulation.
- **Candidate Rules / sponsor agreements prohibit tampering** with the exam/computer; a proctor who
  suspects tampering with exam software ends the session [src 5]. The **exact reverse-engineering clause
  wording is [unverified]** (full current Candidate Rules Agreement text not retrieved) — counsel should
  pull the live agreement for the owner's sponsor.
- **CFAA / "authorized access":** *Van Buren v. United States* (2021) narrowed "exceeds authorized
  access" to entering areas one is **not entitled to reach**, rejecting the theory that a mere
  terms-of-service violation is a federal crime [src 6] — generally favorable to good-faith testing on
  one's **own** systems/accounts. Limits remain: the "without authorization" prong was not decided, and
  a cease-and-desist can revoke authorization. **Distinguish:** (a) tests on the owner's **own
  practice/sandbox form with consenting internal testers** — low CFAA exposure; vs (b) **probing
  Pearson's live production platform** — materially higher, and out of scope here (§4).
- **DMCA §1201 anti-circumvention (client-tampering / environment-manipulation tier):** arms that mask
  or manipulate the client's runtime environment — **T09-07** (stealthed hypervisor), **T09-08**
  (GPU-passthrough guest), **T09-10** (virtual display adapter), and the VM / remote-desktop negative
  controls **T09-NC1 / T09-NC2** (this is the "client-tampering" test class the task flags as
  "#14-type"; T09 has no single test literally numbered 14 — it is the 12 treatments + 2 negative
  controls) — can implicate §1201 if they circumvent a technological protection measure on the OnVUE
  client. A **good-faith security-research exemption** exists at **37 C.F.R. §201.40(b)(7)**, carried
  forward unchanged in the Copyright Office's **ninth triennial rule (effective Oct 28 2024)**, but it
  is **narrow and conditional** (several limiting requirements) and not a blanket license [src 7].
  Counsel must map each Tier-2 arm against it **before** execution.
- **ToS violation risk:** running OnVUE in a VM, with remote-control/screen-share tools, or on multiple
  displays violates stated OnVUE policy (T05 §4–7) even in a sandbox; doing so on a real account/booking
  risks ban/forfeiture (T05 §6; [src 5]). Keep Tier-2 off any real account entirely.
- **Required artifact:** a counsel memo per planned tier, naming the owner's relationship (I1), the
  exemptions relied on, and the scope limits — attached to the §9 sign-off record.

## 7. Per-test authorization tiering (map T09 → risk tier)

**Tier 1 — lower exposure (owner's OWN stack, passive host-side, no VM, no client-binary tampering).**
Runnable with least third-party exposure once §3 required roles sign (Branch B: Tier-1 is the ceiling).

| T09 | Technique | Tier-1 rationale |
|-----|-----------|------------------|
| T09-01 | Non-activating overlay | Passive host window; placeholder content; no VM/client tamper |
| T09-02 | Local offline LLM + answer bank | Local inference, benign content; no network/VM |
| T09-03 | Pre-exam staging / memory-resident helper | Benign marker process; host-side |
| T09-04 | Passive foreground-preservation | Non-activating helper; host-side |
| T09-05 | Accessibility / UI-Automation read | A11y client read of placeholder text; host-side |
| T09-06 | Local OCR of framebuffer + local LLM | Capture-API probe, checksum-only; host-side (no client tamper) |
| T09-09 | Fast-user-switching / 2nd session | OS session feature; benign marker |
| T09-11 | macOS TCC/Spaces grant enforcement | Permission-grant measurement; host-side |
| T09-12 | Local answer-injection (display-only arm) | Human clicks; ordinary input. *(synthetic-input arm → mid; local only, non-scored fields)* |

**Tier 2 — higher exposure (VM / virtualization / client-environment manipulation; DMCA §1201 + ToS
engaged; policy-prohibited surfaces). Needs extra sign-off: counsel memo per arm AND Branch-A Pearson
coordination; prohibited for Branch-B owners until a relationship/authorization exists.**

| T09 | Technique | Why Tier-2 |
|-----|-----------|-----------|
| T09-07 | Stealthed Type-1 hypervisor, OnVUE as guest | VM — policy-prohibited (T05 §4–5); §1201 circumvention concern |
| T09-08 | GPU/device-passthrough near-native guest | VM/passthrough — same |
| T09-10 | Virtual display adapter (IddCx) | Manipulates display enumeration (T05 §6 multi-display prohibition) |
| T09-NC1 | **[neg ctrl]** casual VM / WSL / Hyper-V | VM surfaces — policy-prohibited (T05 §4–5) |
| T09-NC2 | **[neg ctrl]** loopback remote-desktop | Remote-control/screen-share — policy-prohibited (T05 §7) |

**Least-third-party-exposure path:** Tier-1 arms on the owner's own isolated hardware and own
practice/sandbox form, with the owner's own tester accounts, do not require touching Pearson production
and (Branch B) are the only arms runnable without a Pearson relationship. Tier-2 always needs the extra
sign-off above.

## 8. Go/No-Go checklist (binary gate — ALL must be YES before each run; any NO = No-Go)

1. [ ] I1–I6 inputs (§2) confirmed in writing (relationship, exam form, OS/version, tester pool,
   sponsor profile, isolated environment).
2. [ ] Five required sign-offs present and current (§3): program owner, legal counsel, privacy/DPO,
   security lead.
3. [ ] If run touches Pearson systems **or** is any Tier-2 arm → Pearson/sponsor coordination obtained
   (Branch A); **or** run is Tier-1-only on owner's own stack (Branch B).
4. [ ] Exam form is **practice / non-live / sandbox** (never live pool); confirmed not production.
5. [ ] Written, informed, revocable tester consent on file for biometric + ID + room-scan + recording
   (§5); BIPA/GDPR duties cleared by DPO.
6. [ ] Data-handling & deletion plan registered; "retain no live content, log signal/liveness only."
7. [ ] Scope-OUT prohibitions (§4) acknowledged; stop conditions briefed to operators.
8. [ ] Counsel memo attached for each Tier-2 arm (CFAA/§1201/ToS mapping).
9. [ ] Negative controls scheduled first to validate the harness (T09-NC1/NC2); treatment FAILs void if
   a control fails to fire.
10. [ ] Client version + OS build pinned and recorded; sponsor/exam profile recorded for the run.

## 9. Sign-off record template

| Approver role | Name | Date | Scope approved (tests / tier / OS / exam form / tester pool) | Conditions / limits |
|---------------|------|------|--------------------------------------------------------------|---------------------|
| Program owner |  |  |  |  |
| Legal counsel |  |  |  | CFAA/§1201/ToS memo ref: |
| Privacy / DPO |  |  |  | Consent + retention/deletion ref: |
| Security lead |  |  |  | RoE + stop conditions ref: |
| Pearson VUE / exam-sponsor coord. (if I1=A or Tier-2) |  |  |  | Written ack ref: |

*One record per authorized run/tier. No row blank for a required role = No-Go.*

## 10. Sources (numbered)

1. Pearson — Responsible Security Disclosure Policy. https://www.pearson.com/legal-information/our-policies/responsible-security-disclosure-policy.html — accessed 2026-11-12 — PRIMARY (scope, `responsible.disclosure@pearson.com`, prohibited actions; no reverse-engineering clause).
2. Pearson VUE — Contact test security (testing-violation reports, not software vulnerabilities). https://www.pearsonvue.com/us/en/contact-us/test-security.html — accessed 2026-11-12 — PRIMARY.
3. Pearson VUE — Test owners / Secure (Secure Testing Framework™; data analytics & forensics; sponsor partnership). https://www.pearsonvue.com/us/en/test-owners/secure.html — accessed 2026-11-12 — PRIMARY.
4. Pearson VUE — Test owners / Contact ("Partner with our team"). https://www.pearsonvue.com/us/en/test-owners/contact.html — accessed 2026-11-12 — PRIMARY (referenced; page content **[unverified]**).
5. Pearson VUE Candidate Rules Agreement (tampering prohibition; proctor ends session on suspected tampering). https://form.jotform.com/231414303339043 — accessed 2026-11-12 — THIRD-PARTY mirror; **exact reverse-engineering clause [unverified]**.
6. *Van Buren v. United States* (2021), CFAA "exceeds authorized access" narrowed. https://en.wikipedia.org/wiki/Van_Buren_v._United_States ; analysis: https://www.cooley.com/news/insight/2021/2021-06-09-us-supreme-court-computer-fraud-abuse-act-van-buren — accessed 2026-11-12 — SECONDARY (legal commentary).
7. DMCA §1201 good-faith security-research exemption (37 C.F.R. §201.40(b)(7)); ninth triennial rule effective Oct 28 2024. https://www.authorsalliance.org/2024/11/08/the-dmca-1201-rulemaking-summary-key-takeaways-and-other-items-of-interest/ ; https://www.mondaq.com/unitedstates/copyright/1538214/final-rule-issued-in-the-us-copyright-offices-ninth-triennial-section-1201-proceeding — accessed 2026-11-12 — SECONDARY (law-firm/advocacy commentary; exemption limits **[verify against 37 C.F.R. §201.40 text]**).
8. Exam-proctoring BIPA class actions (written consent + retention policy; Respondus/Examity/ProctorU). https://www.faegredrinker.com/en/insights/publications/2021/3/exam-proctoring-software-targeted-in-new-wave-of-bipa-class-action-litigation — accessed 2026-11-12 — SECONDARY (law-firm commentary).
9. **T05** OnVUE Control Surface — `/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md` (cited T05 §n; esp. §4–7 policy-prohibited surfaces, §10 biometrics/recording/telemetry hosts, §11 platform/version, per-sponsor caveat).
10. **T09** OnVUE Detection-Measurement Test Design — `/home/user/OneVue-Reseach/slices/T09-bypass-tests/output.md` (cited T09-nn; 14 test designs, safety/ethics + Authorization statement, measurement-validity note).

---
*Defensive governance only. This plan authorizes nothing; it defines the approvals, scoping, and
safeguards that must exist before any T09 test runs. Legal/ToS content is informational, not legal
advice — counsel and DPO review required. Design only — not run.*
