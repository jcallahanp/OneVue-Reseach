# T16 Hypervisor risk — HOME vs CENTER + OnVUE detection-stage timeline

Scope: DEFENSIVE decision-support for the certification owner choosing between **at-home** (candidate-owned/BYOD machine, 360° pre-check + live room camera, **no screen monitoring**) and **in-person** (center-owned machine) OnVUE delivery. This slice is **conclusions-level only**: it cites the attacker-capability conclusions already established in T05 / T06 / T14 and does **not** re-derive, build, configure, harden, or iterate any evasion or hypervisor method. Grounding: T05 (OnVUE control surface), T06 (single-machine red-team inventory), T14 (Type-1 hypervisor threat + prevention). All access dates **2026-11-12**.

Owner's question answered here (Goals D3): *"Is it just the install I'd catch them at? At what stage does OnVUE capture a VM/hypervisor? Does home make the hypervisor a bigger threat, and is it still secondary to methodology #1 at home?"*

---

## 1. OnVUE detection-stage timeline — where VM/hypervisor detection acts

Enumerated below are the stages a candidate passes through, what runs at each, whether VM/hypervisor detection is known to act there, and whether that detection is **point-in-time** or **continuous**. "Silent/UNKNOWN" marks a stage where the grounding files and public sourcing give no documented behavior.

| Stage | What happens | VM/hypervisor detection acts here? | Point-in-time vs continuous | Source |
|---|---|---|---|---|
| **S0 — Offline preparation** (home only) | Candidate prepares their own machine *before exam day*. No proctor, no OnVUE, no owner visibility. | **No** — nothing from the proctoring side observes this stage at all. | n/a (unobserved) | T14 §3 (pre-built worst case); Goals D3 |
| **S1 — System test** (same device + network, pre-exam) | Checks OS/bandwidth/cam/mic/speaker, downloads + launches secure browser, confirms video stream. | **Yes** — documented VM check fires here; "a virtual machine has been detected" hard-block (T05 §1, §4-5). Over-blocks Hyper-V/VBS/WSL (T05 §5). | **Point-in-time** (readiness snapshot) | T05 §1,§4,§5; GMAC system-test page |
| **S2 — Secure-browser launch / check-in** | Browser Lock (`proctorapp`) launches; prohibited-process scan; VM detection; display check; remote-control/screen-share block; ID photo + 360° room scan + facial-comparison biometrics. | **Yes** — this is the **naive-VM capture point**: the VM heuristic + process scan run as the exam is entered (T05 §3,§4). | **Point-in-time** (launch/check-in gate) | T05 §1,§3,§4,§6,§7,§10 |
| **S3 — During-exam monitoring** | Webcam video + microphone audio + OnVUE client telemetry + biometrics/analytics; Browser Lock foreground/app-switch lockdown. **Screen contents NOT captured or streamed.** | **UNKNOWN / silent** — whether the VM/hypervisor heuristic *re-runs continuously* mid-exam is undocumented. T05 §1 notes OnVUE "shuts down prohibited processes it detects" at check-in and *may* re-scan; no primary source confirms a continuous VM re-check. Third-party (Open Group) warns a silently-running VM/RAT *can* cause mid-exam termination, but this is unverified for a *hardened* guest. | **UNKNOWN** (mark silent) | T05 §1,§10; T06 #3; Open Group (3rd-party) |

**Where detection acts, in one line:** VM/hypervisor detection is **documented only as point-in-time, at S1 (system test) and S2 (launch/check-in)**. Continuous mid-exam VM re-checking (S3) is **UNKNOWN** and must not be assumed. The one decisive VM control (T05 §4 hard-block) fires at the *launch gate*, not continuously.

### Answer to "is it just the install I'd catch them at?"

No — and the framing itself needs correcting:

- **The install is never observed.** OnVUE has no visibility into S0. A home candidate installs and tunes the hypervisor off-site, days or weeks earlier (T14 §3); a center candidate would need boot/disk control to install at all (T14 §2a). There is **no "install event" the proctor or client ever sees**. The owner cannot "catch them at the install."
- **What OnVUE can catch is the runtime signature at launch — and only for a naive guest.** For a **naive** VM/guest, the S1/S2 heuristic is the capture point: it fires at launch and hard-blocks (T05 §4). That is the realistic automated catch, and it is a *launch-time signature check*, not a catch of the installation act.
- **For a hardened guest, no stage captures it.** Per the established conclusion in T06 #7 / T14, a hardened Type-1 guest can defeat the **launch-time signature heuristics** (timing-based detection is the unresolved hard part), and OnVUE running *inside* the guest has **no visibility across the VM boundary** (stated as given per the brief; no method elaborated here). So: S0 is unobserved, S1/S2 signatures are masked, and S3 provides no screen capture and no cross-boundary visibility (and any mid-exam VM re-check is UNKNOWN regardless). **No stage in the pipeline reliably captures a hardened guest.**

So the honest answer to the owner: *you would not catch the install at all; your only automated catch point is the launch-time VM signature at S1/S2, and that catch is reliable only against a naive, unhardened guest.*

---

## 2. HOME vs CENTER — structural risk difference (what the owner CAN and CANNOT control)

This contrast is framed as **loss of owner controls**, not as attacker steps. Every control below is one T14 §5 marks "needs machine ownership = Yes."

| Control dimension | CENTER (owner-owned machine) | HOME (candidate-owned / BYOD) |
|---|---|---|
| **Firmware / boot** (BIOS admin password, Secure Boot, disable USB/PXE boot, lock boot order) | **Owner controls** — can lock all of it (T14 controls 1-4) | **None** — owner has zero firmware control; the candidate *owns* the firmware |
| **Hardware virtualization toggle** (VT-x/AMD-V, VT-d) | **Owner can disable + lock** — the decisive lever: no hardware Type-1 hypervisor or passthrough can run (T14 control 6) | **None** — candidate sets this freely |
| **Endpoint software / OS image** (disable Hyper-V/VBS/WSL, standard-user account, WDAC/AppLocker) | **Owner images + locks** (T14 controls 7,11) | **None** — candidate has full admin/root over the OS |
| **Machine imaging / reset between sessions** (verified golden image, reset to known-clean) | **Owner reimages** each session (T14 control 9) — the defining owned-vs-BYOD advantage | **None** — the exam runs on the candidate's own persistent, pre-prepared machine |
| **Install prevention** (can a concealing hypervisor be installed/booted?) | **Denied at firmware layer** — reimage + USB-boot both need boot/disk control that lockdown removes (T14 §2a) | **Not preventable** — candidate owns the machine and all offline access to it |
| **Offline prep time on the *actual* exam machine** | **Effectively none** — candidate cannot pre-stage the owner's box; the pre-built-USB path still needs external boot *enabled* (T14 §3) | **Unlimited** — candidate builds + tests a hardened guest on the very machine they will test on, before exam day |
| **Supervised cold boot into a verified image** | **Possible** (T14 control 12) | **Impossible** — no trusted boot path to observe |
| **Physical / tamper controls** (chassis lock, port blockers, seals) | **Available** (T14 control 14) | **None** |
| **Screen visibility during exam** | **Not captured** (same OnVUE deployment; T05 §10) | **Not captured** — shared blind spot in *both* models |
| **OnVUE's own VM heuristic (vendor backstop)** | Available — but T14 §4: backstop only, catches naive guests; hardened-guest behavior UNKNOWN | Available — identical backstop, identical UNKNOWN limit |

### Conclusion: how much does hypervisor risk rise at home, and why

**It rises from "closable before OnVUE even runs" to "reliant on a vendor backstop that is unverified against a hardened guest."** The reason is purely **loss of owner controls**:

- At the **center**, the hypervisor threat is decisively beatable *without depending on OnVUE's undocumented internals*: firmware boot lockdown + disabled VT-x/VT-d + supervised boot into a verified golden image collapse the attack to "defeating firmware the owner controls" (T14 §6). The concealing routes are closed **before OnVUE runs**.
- At **home**, **every one of those controls disappears.** The owner cannot touch firmware, cannot prevent the install, cannot image or reset the machine, and cannot observe the (off-site, days-earlier) preparation. What remains is only (a) OnVUE's own launch-time VM heuristic — which T14 §4 and the vendor-variance research (§4 below) show reliably catches **naive guests only** — plus (b) the camera/mic. The candidate is simultaneously the machine owner, so T14's "pre-built USB worst case" is no longer an edge case requiring external boot to be enabled; at home it is simply the candidate's normal, fully-controlled environment with **unlimited prep time on the real exam machine**.

Net: the hypervisor risk is **materially higher at home**, and the increase is entirely attributable to the owner losing the firmware/boot/endpoint/imaging layer that is the center's decisive defense.

---

## 3. At home: hypervisor vs methodology #1 (local tooling) — which is the bigger practical concern

At home, **methodology #1 (local tooling) remains the bigger practical concern**, with the hypervisor a secondary, higher-sophistication path. T06's executive answer already ranks "no-VM, in-the-blind-spot local tooling" (a fully local answer engine reading the exam passively and surfacing answers on the same unobserved primary display, staged before the secure client launches) as the **strongest family for a single-machine candidate**, precisely because it never touches virtualization and so entirely sidesteps OnVUE's *only documented hard-block* — VM detection (T05 §4) — the very control the hypervisor family must fight. Removing owner endpoint controls (the home condition) strengthens local tooling further: it runs with no standard-user/WDAC/AppLocker constraint, no owner process baseline, no firmware change, no boot control, and no VM at all — nothing the candidate does not already fully own. The hypervisor (T06 #7, Family A) offers cleaner *isolation* but carries the VM-detection arms race (timing the unresolved part) and higher compute/skill overhead; it becomes *more viable* at home than at the center (firmware lockdown gone, prep time unlimited), yet it is still not the lowest-effort route a rational candidate reaches for. Both paths converge on the same owner conclusion: with the screen unobserved (T05 §10) and no endpoint/firmware control, the passive local-tooling path is unaddressable by OnVUE **regardless of virtualization** — so at home the owner should treat local tooling as the primary concern and the hypervisor as a secondary one whose home-side growth is a symptom of the same lost-control problem.

---

## 4. Countermeasures by model

### 4A. What meaningfully reduces hypervisor (and local-tooling) risk AT HOME — with an honest ceiling

These are the only levers available when the owner does **not** control the machine. Each is **probabilistic or value-reducing**, not a reliable block of a hardened guest.

- **Record-and-review** (full session recording + human and/or AI post-hoc review). Raises catch probability for *behavioral* tells (gaze drift, unnatural timing, lip movement). Does **not** detect a hardened guest or a passive on-screen overlay directly — it catches behavior, not the mechanism.
- **Stronger host-integrity lockdown-browser vendors.** Some proctoring stacks advertise deeper VM/RAT scanning (e.g. "400+ application" signature scans). Independent testing shows vendors vary **wildly** — one detected VMs reliably, another failed VM detection in every test, and *none* reliably detected virtual-camera injection without hardware attestation (SETU 2026 student report; treat as single unreviewed source). A better host-integrity client raises the **naive-VM** bar; **hardened-guest + timing detection remains unverified across vendors**, so this narrows but does not close the gap.
- **Exam design** (large randomized item banks, tight time pressure, scenario/performance items hard to answer by lookup, high cut scores, and statistical forensics on answer patterns + response times). **Vendor-independent** and method-independent: it reduces the *value* of any assistance however it is delivered. The most durable home lever.
- **AI / behavioral analytics** (keystroke/mouse cadence, gaze/head-pose, response-time anomalies, cheating-ring/copy-typing detection; cf. 2025 keystroke-forensics work). Probabilistic and largely post-hoc; catches *patterns*, not the delivery mechanism, and is evadable by display-only, human-paced use.
- **Limiting top-stakes content to in-person.** The honest high-assurance fallback for the material that most needs protection.

### 4B. Firmware-layer controls that ONLY work on owned machines (center, not home)

All of these are the T14 §5 stack, every row of which is marked "needs machine ownership = Yes" and **none of which exist at home**:

- **BIOS/UEFI admin password + Secure Boot + disable USB/PXE boot + lock boot order** (T14 controls 1-4) — closes the reimage installer and the pre-built-USB worst case before OnVUE runs.
- **Disable VT-x/AMD-V + VT-d in firmware** (T14 control 6) — *the decisive owner-only lever*: with no hardware virtualization, no hardware Type-1 hypervisor or GPU passthrough can run at all.
- **Disable Hyper-V/VBS/WSL in the image; standard-user exam account + WDAC/AppLocker; verified golden image + reset between sessions; supervised reboot into the verified image** (T14 controls 7,9,11,12).

These relegate OnVUE's own (undocumented, hardened-guest-unverified) VM check to a **backstop** and put the load on controls the owner fully governs (T14 §4, §6).

### Assurance ceiling at home — explicit and honest

**For a technically sophisticated candidate on their own machine, no at-home countermeasure provides high assurance against the hypervisor-isolation or the local-tooling path.** The owner cannot control firmware/boot, cannot prevent a hypervisor install, cannot image or reset the machine, and cannot observe the off-site, days-earlier preparation. OnVUE's own VM check is a backstop that reliably catches **naive** guests only; hardened-guest and timing detection are **UNKNOWN/unverified** (T14 §4), and any mid-exam VM re-check is undocumented (§1 above). Because the screen is unobserved in this deployment (T05 §10), a **passive local-tooling path is unaddressable regardless of virtualization**. The realistic home ceiling is therefore: **deter and probabilistically catch the careless candidate and behavioral tells, while accepting residual exposure to the prepared, technical candidate** — exactly the population the owner's own notes flag as commonplace in this certification (Goals: "exploitation skills are commonplace"). The only path to **high assurance** against this threat class is **in-person on owner-locked machines**, where firmware boot lockdown + disabled virtualization + a supervised boot into a verified image collapse the threat to "defeating firmware the owner controls" (T14 §6). This is consistent with the owner's D5 framing: at-home guardrails reduce but cannot close the gap; in-person on locked machines is the high-assurance option.

---

## Sources (numbered)

Internal grounding:
1. **T05** — OnVUE control surface: `/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md` (cited T05 §n) — 2026-11-12.
2. **T06** — Persona A single-machine red-team inventory: `/home/user/OneVue-Reseach/slices/T06-persona-a-redteam/output.md` (cited T06 #n) — 2026-11-12.
3. **T14** — Type-1 hypervisor threat + prevention (center-owned machines): `/home/user/OneVue-Reseach/slices/T14-type1-hypervisor/output.md` (cited T14 §n / control n) — 2026-11-12.

External (confirming OnVUE stage behavior + general proctoring approaches to VMs at home; no evasion method sourced):
4. The Open Group Help — "Do Not Use a Corporate or Managed Laptop for OnVUE Online Exams" (OnVUE actively blocks VMs/remote-control/screen-share; silently-running tools can fail the system check or cause mid-exam termination). https://help.opengroup.org/hc/en-us/articles/34238069858706 — accessed 2026-11-12 — THIRD-PARTY (sponsor-help). Used for S3 mid-exam-termination caveat; the *continuous VM re-check* claim is NOT established by it.
5. NashTech blog — "Fixing OnVUE 'Virtual Machine Detected' Error When You're Not Using a VM" (VM check fires at system test; background Hyper-V/VBS/WSL/Docker trigger false positives). https://blog.nashtechglobal.com/fixing-onvue-virtual-machine-detected-error-when-youre-not-using-a-vm/ — accessed 2026-11-12 — THIRD-PARTY. Supports S1/S2 point-in-time detection.
6. GMAC — "What does the OnVUE System Test Check?" (system test = pre-exam readiness gate: OS, connection, cam/mic/speaker, secure-browser launch). https://www.gmac.com/executive-assessment/about/frequently-asked-questions/what-does-the-onvue-system-test-check — accessed 2026-11-12 — PRIMARY (sponsor). Supports S1 as a point-in-time readiness stage.
7. SETU / IT Carlow student project (2026) — "Vulnerabilities in Modern Proctoring Software" (vendor VM-detection varied widely — one reliable, one failed every test; virtual-camera injection undetected by both; concludes no platform reliably detects camera substitution without hardware attestation / active liveness). https://showcase.itcarlow.ie/C00284314/index.html — accessed 2026-11-12 — THIRD-PARTY / single unreviewed student report (label: treat with caution). Used for §4A vendor-variance ceiling.
8. "How Proctoring Platforms Detect and Prevent Cheating?" — WeCreateProblems (typical VM detection = CPU-manufacturer / hardware-string signature check; logged + prompted, i.e. point-in-time). https://www.wecreateproblems.com/blog/how-proctoring-platforms-detect-and-prevent-cheating — accessed 2026-11-12 — THIRD-PARTY (vendor blog). Used for general point-in-time VM-detection characterization.
9. AIME 2025 (ACL Anthology) — keystroke/mouse behavioral forensics for detecting copy-typing / cheating rings. https://aclanthology.org/2025.aimecon-wip.13.pdf — accessed 2026-11-12 — PRIMARY (research). Supports §4A behavioral-analytics as probabilistic/post-hoc.
10. Meazure Learning — "Proctoring in Focus, Part 1: Online Exam Security Q&A" (record-and-review + live + analytics as layered remote-proctoring approaches). https://www.meazurelearning.com/resources/proctoring-in-focus-part-1-online-exam-security — accessed 2026-11-12 — THIRD-PARTY (vendor). Used for §4A record-and-review framing.

Note: Pearson VUE's own documentation does **not** state whether OnVUE re-checks for VMs continuously during the exam; S3 continuous detection is therefore marked UNKNOWN (unverified) rather than asserted.
