# T18 — Decision memo: at-home vs in-person delivery for a high-stakes technical certification

**Audience:** certification owner (current OnVUE sponsor) choosing a delivery model for a high-stakes technical credential taken by sophisticated candidates.
**Level:** defensive / governance only. This memo states where assurance comes from and where it caps out. It contains **no attacker methodology** — techniques are cited to the grounding slices, not re-derived here.
**Two models compared:**
- **HOME (proposed):** 360° pre-check + **live room camera**, **no screen monitoring**, **candidate-owned machine**.
- **IN-PERSON:** **owner-controlled machines**, firmware/boot lockdown + endpoint controls available.

Grounding slices (all 2026-11-12): T16 (home-vs-center detection stages / assurance ceiling), T17 (clipboard/paste is not an OnVUE control), T13 (local-tooling feasibility + the three center controls), T14 (firmware/boot lockdown stack), T08 (BYOD defense stack + residual gaps). External best-practice sources are numbered in Sources and accessed 2026-11-12.

---

## 1. Bottom line up front

**At-home delivery cannot reach high assurance for this credential. For certainty against a sophisticated candidate, deliver the high-stakes exam in person on owner-locked machines.**

One-sentence reason: at home the owner loses the only layer that decisively beats the two hardest threats — the firmware/boot/endpoint/imaging control of an owned machine — and in this proposed model also forgoes the screen, so the strongest controls that remain (OnVUE's launch-time host checks) are a vendor backstop that is **unverified against a hardened guest** (T16 §2, §4; T14 §4) and sit beside threats OnVUE does not technically control at all (clipboard/paste — T17 §1; passive local tooling on an unobserved screen — T13 Q4, T16 §3).

Direct recommendation:
- **High-stakes capstone / pass-fail credential decision → in-person on locked machines.** This is the only configuration that collapses the Type-1-hypervisor and local-tooling threats to "defeat firmware the owner controls" (T14 §6).
- **Home is a risk-*reduction* posture, not a closure posture.** It is defensible for lower-stakes forms and as a convenience channel, but it should not carry the weight of the credential's highest-stakes decision against this candidate population. Record-and-review remote proctoring is itself documented in the credentialing literature as a *deterrence and identity* tool, "**not a sufficient method for the detection**" of determined cheating [S3], and the proposed model is weaker still because it drops screen monitoring.

---

## 2. Why home caps out

At home the owner is defending a machine it does not own, with a sensor set that cannot see the screen. Four structural losses stack:

1. **The owner loses the endpoint.** Every firmware/boot/imaging control that makes the in-person model work — BIOS admin password, Secure Boot, disabled external/USB/PXE boot, locked boot order, **disabled hardware virtualization (VT-x/VT-d)**, standard-user + application allow-listing, golden-image reset between sessions, supervised cold boot — **requires machine ownership and does not exist at home** (T14 §5, every row marked "needs ownership = Yes"; T16 §2 table). The candidate owns the firmware, the OS (full admin/root), and the disk, and has **unlimited offline prep time on the exact machine they will test on** — a stage OnVUE never observes (T16 §1, stage S0 "unobserved"). There is no "install event" the proctor or client ever sees; the only automated catch point is the **launch-time** VM signature, and that is reliable **only against a naive, unhardened guest** (T16 §1).

2. **The screen is unobserved.** In the proposed model monitoring is camera + mic only; the candidate's display is never captured. A concealed same-machine second workspace (a local answer engine surfaced on the candidate's own primary display) is invisible to this configuration (T08 §1, central weakness; T16 §3). Gaze/head-pose analytics cannot close this — an overlay on the *primary* display keeps gaze natural on camera (T08 §2, RR5).

3. **OnVUE's host checks are a backstop, unverified against a hardened guest.** OnVUE's one documented decisive host-integrity control is the launch-time "a virtual machine has been detected" hard block (T14 §4). Whether it catches a **well-hardened Type-1 guest**, and whether it performs **timing-based** hypervisor detection (the only reliable catch for a hardened guest), are **UNKNOWN / unverified** in any primary source (T14 §4; T16 §1 stage S3). Independent testing shows proctoring vendors vary **wildly** on VM detection and that none reliably detects virtual-camera injection without hardware attestation (T16 §4A, SETU report — single unreviewed source). No proctoring vendor documents hardware attestation or Type-1 detection (T08 §4, §5 RR1). So the owner would be relying, for the hardest threat, on an undocumented vendor heuristic it cannot tune or verify.

4. **Clipboard/paste is not an OnVUE control, and local tooling needs nothing the candidate doesn't own.**
   - **Clipboard:** Pearson documents a *policy* ban on copy/paste but **no technical clipboard monitoring, no paste/copy-event logging, and no keystroke logging** in OnVUE — all UNKNOWN/undocumented (T17 §1). With the screen unshared, the proctor **cannot see a paste happen** (T17 §2). A real paste block lives in the *sponsor exam-UI* layer, not Browser Lock, and is trivially defeated by retyping (T17 §4). (One genuine piece of defender leverage survives here: automatic host↔guest clipboard needs hypervisor guest tools, which are themselves a VM signature — so a *stealth* guest and a *convenient* cross-boundary clipboard are close to mutually exclusive (T17 §3b). This raises the attacker's effort; it does not give the owner a detection.)
   - **Local tooling:** the strongest single-machine path (a passive local answer bank/LLM read in the screen blind spot) sidesteps OnVUE's only hard block (VM detection) entirely because it uses **no VM**, and at home it runs with no standard-user/WDAC/AppLocker constraint, no owner process baseline, no firmware change — **nothing the candidate does not already fully own** (T16 §3; T13 Q3, Q4). The constraints that make this path feasible at a center (USB delivery, execution allow-listing, network egress) are exactly the owner-controlled levers that **do not exist on a candidate-owned machine** (T13 Q5).

**Residual threats that remain OPEN at home even with best effort:**
- **R-A — Bare-metal / Type-1 hypervisor with a hardened guest; answer tool outside the guest.** Unaddressable at home: no firmware lockdown, hardened-guest detection UNKNOWN, timing detection unsolved, no attestation (T14 §4; T08 RR1; T16 §4 ceiling).
- **R-B — Passive local tooling on the unobserved primary display** (answer bank / small local LLM / capture-excluded overlay / passive accessibility read). Camera + mic cannot see it; screen is not captured (T16 §3; T08 RR2/RR4/RR6).
- **R-C — Retyped (not pasted) answer insertion.** No deterministic OnVUE control; only probabilistic, post-hoc keystroke-cadence analytics, which are not evidenced in OnVUE and are evadable by human-paced use (T17 §4).
- **R-D — Pre-staged helper booted/loaded before the secure client,** on a persistent machine the owner cannot reset to a known-clean image (T13 Q1; T14 §3).

These four are precisely the threats the in-person model closes and the home model cannot. That is the cap.

---

## 3. Best achievable home guardrail stack (if home is still chosen)

This is **risk reduction, not closure.** Ranked by impact-per-effort on the open residuals above. Each entry: what it reduces → its residual/limit. Do the top band first.

**Band 1 — highest impact, do first**

1. **Make exam design the primary control (lookup-resistant items + forensics).** The most durable home lever because it is **method- and device-independent** — it reduces the *value* of any outside assistance however delivered (T16 §4A). Concretely: large, frequently refreshed **item banks** with per-item **exposure control** and scheduled refreshment [S5, S6]; **scenario / performance / judgment items** that resist recall-style lookup (predominantly scenario-based authoring, discrete-option multiple choice, embedded verification and "Trojan-horse" items) [S6]; **randomized item selection** and shuffled options so no two sittings overlap much [S6, S10]; **time limits set near the observed average** and **one-question-at-a-time** presentation to make real-time outside lookup impractical [S10]; and **data forensics** on response patterns and **response-time preknowledge** detection, including "security items" / differential person functioning to flag candidates who ace exposed content but not uncompromised content [S5, S7, S8, S9].
   - *Residual/limit:* probabilistic and largely post-hoc; a patient candidate who paces themselves defeats response-time flags [S7]; forensics is deterrence and invalidation after the fact, not a gate. It reduces payoff; it does not stop a concealed assistant in the moment.

2. **Require a stronger host-integrity vendor than the current screen-blind OnVUE config — a native, continuously-running host agent (record-and-review class).** Convert host integrity from **point-in-time at launch** to **continuous, all-session process + overlay-window + display enumeration** (T08 §1, §4, Band 1). This is the one control that can catch a **pre-staged helper process** or a **concealed local LLM** even while the screen stays unobserved (closes much of R-B behavioral surface, R-D). Pair with a **witnessed clean reboot** immediately before the exam so a memory-resident helper must re-stage after the clean gate (T08 §3).
   - *Residual/limit:* VM detection stays a software heuristic — **does not close R-A** (hardened Type-1); a signed/renamed benign-looking helper can survive signature-based enumeration (T08 RR3); on a candidate-owned machine the owner cannot force the OS image, so agent coverage depends on candidate compliance.

3. **Full-session record + AI/behavioral post-review, with human adjudication.** Record webcam + mic (and screen where policy permits) for the whole session; use AI to **flag** and **trained human reviewers to confirm** — the documented safer model for credentialing [S1, S4]. Raises catch probability for behavioral tells (gaze drift, off-screen glances, typing bursts after looking away, lip movement, a second device).
   - *Residual/limit:* record-and-review is a deterrence/identity control, explicitly "**not a sufficient method for the detection**" of determined cheating [S3]; it catches *behavior*, not the hidden mechanism; false-positive-prone and privacy-heavy; a human-paced overlay read on the primary display leaves little behavioral signal (T08 RR5/RR6).

**Band 2 — high impact, more effort or a constraint change**

4. **Live proctoring at a low candidate-to-proctor ratio (not record-and-review alone) for the home high-stakes tier.** Literature and state policy converge on a **maximum 1:10** proctor-to-candidate ratio for remote high-stakes testing, with live proctoring the highest-security remote tier, typically **1:8–15** and justified specifically for professional certifications [S1, S2, S3]. A live proctor can intervene on camera-visible correlates in real time.
   - *Residual/limit:* a live proctor with no screen feed still "**cannot see a paste happen**" (T17 §2) and cannot see a passive overlay; raises cost sharply; catches the careless, not the prepared.

5. **Second-camera / room-camera showing hands, screen edge and desk from the side.** Adds a view that can reveal a second surface, a second device, or an off-screen reference, and partially addresses the camera's blind spot behind the candidate.
   - *Residual/limit:* does not see the candidate's own primary display content; defeated by on-screen (same-display) assistance, which is the primary threat (T08 §2).

6. **Disable paste in the exam config + input-cadence analytics as a review flag (sponsor exam-UI layer).** A field-level `onpaste` block stops the paste *action*; pair with keystroke-cadence / copy-typing analytics to flag unnaturally clean insertion bursts (T17 §4, §5 rows 1–2).
   - *Residual/limit:* blocking paste only **forces retyping**, which the paste block cannot see (R-C); cadence analytics is a probabilistic, length-gated, human-adjudicated flag, "**should not be used alone**," and is **not evidenced in OnVUE** — it is an added layer (T17 §4).

7. **Clear integrity agreement + attestation, candidate environment rules, and a documented, equally-applied incident process.** Explicit AI-use policy and consequences, ban on managed/corporate machines and webcam filters/virtual cameras, consent for recording, and consistent procedures with incident records and evidence that remote and in-person candidates are treated equally [S1, S3, S4] (T08 R6/R11).
   - *Residual/limit:* deterrence and legal defensibility, not prevention; a sophisticated, motivated candidate is the population least deterred by attestation.

**Honest framing of the stack:** Bands 1–2 convert the proposed screen-blind, launch-only posture into a continuously-enumerated, forensics-backed, lower-lookup-value posture. That measurably **reduces** exposure to the careless and the moderately prepared. It does **not close R-A (hardened Type-1), R-B (passive on-screen tooling on the unobserved display), or R-C (retyped insertion)** — those need either a constraint change (continuous screen capture *plus* overlay enumeration) or capability no proctoring product ships (hardware attestation + a timing hypervisor probe), and even then the candidate-owned machine denies the owner the firmware layer (T08 §5; T16 §4 ceiling). Treat the home stack as deter-and-probabilistically-catch, with residual exposure to the prepared technical candidate accepted by design.

---

## 4. In-person control stack (if certainty is needed)

On owner-controlled machines the decisive controls become available, and they close the threats home cannot. The stack (T13 + T14):

- **The three center controls (T13 Q5), deployed together:**
  1. **Application execution allow-listing (WDAC signed policy; tamper-resistant even against admin/SYSTEM)** — only enumerated signed binaries run, so a portable helper, VM software, a model runtime, or a custom index binary cannot execute (breaks R-B/R-D execution).
  2. **Removable-media / USB control (block mass storage, allow HID only)** — removes the sole delivery vector for a pre-built toolkit onto a non-pre-stageable machine (breaks R-B/R-D delivery).
  3. **Network egress default-deny allowlist (only OnVUE's documented hosts/ports; isolate the test LAN)** — the *only* control that closes network offload of inference, and denies internet payload/model fetch.
- **Firmware / boot lockdown (T14 §5–6):** BIOS/UEFI **admin password**, **Secure Boot**, **disable USB/PXE/external boot + lock boot order**, and — the decisive owner-only lever — **disable hardware virtualization (VT-x/AMD-V + VT-d)** so **no hardware Type-1 hypervisor or GPU passthrough can run at all** (breaks R-A at the firmware layer, before OnVUE runs). Disable Hyper-V/VBS/WSL in the image.
- **Strip guest tools from the golden image + verified imaging with reset between sessions** — guarantees a known-clean baseline each session, removes the guest-tool clipboard channel at the source (the T17 §3b leverage, operationalized), and forces every attempt to re-deliver on the day where USB/allow-listing/egress bite (T14 control 9; T17 §5 row 6).
- **Supervised cold boot into the verified image** under observation — denies a hidden boot of foreign media and the pre-built-USB worst case (T14 control 12, §3). Pair with standard-user accounts, device/port control, and physical tamper controls (chassis lock, port blockers, seals).

**Assurance difference vs home (the core of the memo):** these controls **collapse the whole threat class to "defeating firmware the owner controls"** (T14 §6) and relegate OnVUE's undocumented, hardened-guest-unverified VM check to a **backstop rather than the primary line** (T14 §4). At home, **every one of these controls disappears**, OnVUE's backstop becomes the primary (and only automated) line against the hardened guest, and the screen is unobserved on top of that. The gap is not incremental: it is the difference between a threat **closed before the secure client even runs** and a threat that is **structurally open and relies on a vendor heuristic the owner cannot verify** (T16 §2 conclusion). This is why certainty requires in-person on locked machines.

---

## 5. Decision criterion

Tie the delivery choice to **consequence of a false pass** and **acceptable cheating rate**, judged against **candidate sophistication** (here: high — "exploitation skills commonplace," per the owner's own framing, T16 §4 ceiling).

Rubric:

| Stakes of the credential decision | Acceptable residual cheating rate | Candidate sophistication | Recommended model |
|---|---|---|---|
| **High** — pass gates practice/employment, hard to revoke, legally challengeable, limited attempts, high incentive to cheat | **Near-zero required** | High | **In-person on owner-locked machines** (Section 4). Home cannot reach this; the open residuals R-A/R-B are unacceptable here. |
| **Medium** — meaningful but recoverable; moderate incentive | Low, some residual tolerable with strong post-hoc forensics | Mixed | **Home with the full Band 1–2 stack** (continuous host agent + record/AI review + lookup-resistant item design + forensics + low proctor ratio), accepting documented residual exposure. |
| **Low** — formative, practice, CE credit, micro-credential | Residual tolerable; deterrence sufficient | Any | **Home / automated proctoring** is appropriate and proportionate [S3, S4]. |

Decision rules:
- **The acceptable-cheating-rate test is decisive.** If the owner cannot defend a false-pass rate that includes "a prepared candidate used undetectable on-screen assistance on their own machine," then home is out for that form, regardless of convenience or cost — because that exact scenario is an **open residual** at home (Section 2, R-A/R-B).
- **Match security to stakes** — the credentialing literature is explicit that high-stakes entry-to-practice exams warrant the **highest available security and the lowest candidate-to-proctor ratio**, and recommends **continued access to in-person testing** alongside any remote program [S1, S3].

**Hybrid option (recommended default):** run a **multi-modal** program.
- **Home (Band 1–2 stack)** for low- and medium-stakes forms, practice/CE, and re-tests — capturing the reach and convenience benefits that are real and documented [S1, S4], while keeping lookup-resistant item design and forensics on every form.
- **In-person on locked machines** for the **high-stakes capstone / the pass-fail credential decision**, where the firmware/endpoint controls of Section 4 provide the certainty the credential's consequence demands.
- Keep **item banks, exposure control and data forensics common to both modalities** so cross-modal statistical comparison (home vs center score/time patterns) is itself a security signal [S5, S9], and so the two channels are demonstrably treated equally for defensibility [S1].

---

## 6. Candid caveats

Do not over-trust any single control; several key facts are genuinely unknown:

1. **OnVUE internals are UNKNOWN.** Pearson publishes no telemetry schema. It is **undocumented** whether OnVUE re-checks for VMs continuously mid-exam, whether it performs timing-based hypervisor detection, whether it logs keystrokes or distinguishes pasted from typed text, and whether its VM check catches a hardened guest at all (T16 §1 stage S3; T14 §4; T17 §1). Absence of documentation is not proof of absence — but it is equally not a basis for relying on these as controls. Treat them as backstops, never as the primary line.
2. **Vendor security claims are vendor-asserted.** The continuous-host-agent capabilities that the Band-1 vendor recommendation rests on (process/overlay/display enumeration, "400+ app" VM scans, 30-second re-checks) are **vendor documentation, not independently verified** (T08 §4). Independent testing that does exist shows VM detection varies wildly between products and that virtual-camera injection went undetected without hardware attestation (T16 §4A) — and that is a single unreviewed student report. Pilot and test the chosen vendor against your own threat model before trusting it.
3. **No independent teardown exists.** The entire assessment rests on published docs, third-party reports, and structural reasoning — not on a security audit of OnVUE or of the candidate's machine. The hardened-Type-1 residual (R-A) is a reasoned worst case, not a demonstrated exploit in this deployment; equally, nothing in the record lets the owner rule it out.
4. **The home residuals are structural, not fixable by tuning.** R-A (hardened hypervisor), R-B (passive on-screen tooling on an unobserved display), and R-C (retyped insertion) are open at home because of **what the owner does not control** (firmware, the screen, the input path), not because a setting is off. More cameras, stricter policy, or a better vendor reduce probability; they do not change the structure. Only machine ownership (in-person) or capability no vendor ships (attestation + timing probe) changes it.
5. **Guardrails can create false confidence.** A rich home stack *looks* like high assurance and will catch most careless candidates, which can mask that it does not catch the prepared technical candidate — exactly the population this credential attracts. State the residual explicitly in the program's risk register so the credential decision is made with eyes open.

---

## Sources (numbered)

Internal grounding (all 2026-11-12):
- **T16** — Hypervisor risk HOME vs CENTER + OnVUE detection-stage timeline: `/home/user/OneVue-Reseach/slices/T16-hypervisor-home-vs-center/output.md`
- **T17** — Copy-paste / clipboard detection on OnVUE: `/home/user/OneVue-Reseach/slices/T17-copypaste-clipboard/output.md`
- **T13** — Methodology #1 (local tooling) feasibility, center model + three controls: `/home/user/OneVue-Reseach/slices/T13-method1-center/output.md`
- **T14** — Type-1 bare-metal hypervisor threat + prevention (firmware/boot lockdown stack): `/home/user/OneVue-Reseach/slices/T14-type1-hypervisor/output.md`
- **T08** — Persona B defensive synthesis (BYOD defense stack + residual risk register + vendor selection): `/home/user/OneVue-Reseach/slices/T08-persona-b-synthesis/output.md`

External best-practice (web, accessed 2026-11-12):
1. REL Southeast / IES — "Proctoring Strategies for Remote Statewide Exams: A Summary of the Evidence" (max **1:10** proctor-to-student ratio; live remote proctoring, proctor training/certification, small testing groups, controlled environment; recommends **continued access to in-person testing**; AI mitigation = clear policy + multiple versions). https://ies.ed.gov/sites/default/files/rel-southeast/document/2026/09/6.2.3.18%20Proctoring%20Remote%20Exams.pdf — PRIMARY (government research).
2. TestWe — "Assessment with Proctoring: Organising Secure Remote Exams" (browser lockdown defeated by a second device; AI proctoring "flags anomalies, does not prove"; **live proctoring 1:8–15**, justified for high-stakes certifications; hybrid AI+human). https://testwe.eu/en/blog/evaluation-avec-proctoring-en — THIRD-PARTY (vendor).
3. CNAR — "An Introduction to Remote Proctoring" (white paper) (high-stakes entry-to-practice exams need the **highest security and lowest candidate-to-proctor ratio**; **record-and-review is deterrence/identity, "not a sufficient method for the detection"** of cheating; match proctoring level to stakes). https://portal.cnar-rcor.ca/wp-content/uploads/2024/08/CNAR-Remote-Proctoring-White-Paper-2021.pdf — THIRD-PARTY (credentialing body).
4. ProctorFree — "Remote Proctoring for Certification and Licensure Exams: A Security Guide" (a remote program must be **at least as secure as a test center**; **human-assisted (AI flags + trained human review) is the safer choice for credentialing**; fully automated fits practice/CE/micro-credentials; psychometrics/data forensics for score anomalies). https://www.proctorfree.com/news/online-proctoring-for-certification-exams — THIRD-PARTY (vendor).
5. Prometric — "Considerations for Computer-Based Test Item Development" (**large item banks + scheduled item refreshment** to mitigate overexposure; **forensic data** on exposure, average item time, response change over exposure). https://www.prometric.com/test-owners/resources/considerations-for-computer-based-test-item-development — THIRD-PARTY (vendor).
6. Credentialing Insights — "Taking Test Security to the Next Level Through Design" (**discrete-option multiple choice**, embedded verification questions, **Trojan-horse items**, show only a subset of content to prevent harvesting; data forensics — gain scores, similar responses, brain-dump key errors). https://www.credentialinginsights.org/Article/taking-test-security-to-the-next-level-through-design-1 — THIRD-PARTY (ICE/industry).
7. Alpine Testing (Smith & Davis-Becker) — "Detecting Suspect Examinees: Differential Person Functioning" (**security items** presumed uncompromised, refreshed frequently; flag candidates high on the exam form but low on security items → preknowledge). https://assets.alpinetesting.com/detecting_suspect_examinees_an_application_of_differential_person_functioning_analysis_7573dfce42.pdf — PRIMARY (research).
8. Qian et al. 2016, *Educational Measurement: Issues and Practice* (NCSBN) — "Using Response Time to Detect Item Preknowledge in CBT" (response-time residual model; large pool + randomesque exposure control + web patrolling; detection rate depends on how fast cheaters answer). https://www.ncsbn.org/public-files/Qian_et_al-2016-Educational_Measurement-_Issues_and_Practice.pdf — PRIMARY (research).
9. van der Linden 2023, *Journal of Educational Measurement* — "A Statistical Test for the Detection of Item Compromise Combining Responses and Response Times" (frequent pool refreshment + randomized exposure control + **data forensics combining responses and response times**; organized/industrialized cheating pacing to evade). https://onlinelibrary.wiley.com/doi/full/10.1111/jedm.12346 — PRIMARY (research).
10. University of Pittsburgh CGS — "Technical Canvas Settings to Reinforce AI-Resistant Exams" (**set time limit near observed average**, **one question at a time + lock after answering**, **shuffle answer choices** to make outside/AI assistance impractical). https://www.cgs.pitt.edu/teaching-toolbox/technical-canvas-settings — THIRD-PARTY (university).

Note: external best-practice sources inform Section 3 (home guardrails) and Section 5 (decision criterion) only. The core structural conclusions (Sections 1, 2, 4, and the residual register) rest on the internal grounding slices T13/T14/T16/T17/T08, which in turn carry their own primary Pearson/OnVUE and research sourcing and their explicit UNKNOWN labels. OnVUE internals remain UNKNOWN; vendor capabilities are vendor-asserted; no independent teardown exists (Section 6).
