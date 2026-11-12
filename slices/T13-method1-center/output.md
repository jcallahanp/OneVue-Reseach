# T13 Methodology #1 feasibility — center-owned, low-compute, 4h model

Scope: DEFENSIVE threat assessment for the certification owner. "Methodology #1" = **local answer-tooling** (T06 Family C/B "no-VM local tooling": a local answer engine — indexed cert bank and/or small local LLM — that reads the exam in OnVUE's screen blind spot and surfaces answers on the same primary display; T06 entries #1 overlay, #2 local LLM+bank, #3 pre-staging, #5 a11y read, #6 local OCR). Analysis/countermeasure level only — no configs, commands, or operational build steps. All web access dates **2026-11-12**.

**Clarified deployment (fixed):** test center, **center-owned** Windows machines (macOS differences flagged), **low compute**, endpoints **currently poorly locked down but owner-controlled**, 4-hour on-site exam, one proctor for many screens (loose), candidate **cannot pre-stage the primary machine** but **may bring USB**, OnVUE is the secure client (controls per T05).

**One-line finding:** The two constraints the user flagged — low compute and the 4h limit — are **weak barriers** and do not meaningfully reduce feasibility. The real determinant is **endpoint hardening**, which the owner controls. As currently (poorly) locked down: feasibility **High**. Fully hardened: **Low**.

---

## Q1 — Pre-engineering off-site vs on-site build: does the 4h limit constrain a prepared candidate?

**Conclusion: the 4h limit is near-irrelevant to a prepared candidate. Methodology #1 is a fully PRE-BUILT, BRING-AND-RUN attack, not an on-site engineering effort.**

- Nothing in methodology #1 requires on-site development. Every heavy component is built off-site and reduced to a portable artifact carried on USB (the candidate "MAY bring removable media" — the stated delivery channel):
  - **Pre-indexed answer bank**: the cert corpus plus a full-text/retrieval index, built off-site, is a static data set. On-site cost = plug in, open, query. Zero build time. (T06 #2)
  - **Pre-quantized small model**: a 1–3B model quantized off-site to a single weight file (~0.9 GB at 4-bit for 1B, ~2 GB for 3B — Q3) plus a portable runtime. On-site cost = launch. (T06 #2)
  - **Portable read/overlay helper**: the non-activating overlay (T06 #1) and a11y/OCR read channel (T06 #5/#6) are prebuilt executables; pre-exam staging (T06 #3) is designed to run before the secure client and the room scan.
- On-site, the candidate does only three mechanical things: deliver the payload (USB), launch it (before `proctorapp`), and read. Those cost minutes, not hours.
- **What the 4h actually governs** is answering throughput, not attack construction. Even a slow per-question lookup/inference (seconds — Q3) fits comfortably inside a 4h window for a normal item count. The clock is a convenience, not a gate.
- **Therefore the user's time intuition is inverted:** 4h does not "make it hard"; a generous, loosely-monitored 4h window is permissive. The binding constraints are **delivery** (can a payload reach the machine?) and **execution** (will it run?) — both owner-controllable (Q5), neither time-bound.

Residual nuance: the candidate cannot pre-stage *this* machine, so a machine reset to a clean, hardened image (Q5-D) forces every attempt to re-deliver on the day — which is exactly where USB/allowlisting/egress controls bite. The 4h limit is not that choke point; delivery and execution are.

---

## Q2 — "Run the LLM through a VM" (OnVUE on host, answer tool in a Type-2 hosted VM on the same machine): detected?

**Conclusion: DETECTED / BLOCKED. This route fails against OnVUE's one documented hard-BLOCK, and it is additionally compute-prohibitive on a low-compute machine. Do not treat the VM as a viable escape.**

Reasoning (ties to T05 §4–5):
- Here OnVUE runs on the **host**, and OnVUE scans **the host it runs on** for virtualization. A Type-2 hypervisor (VirtualBox / VMware Workstation / Hyper-V) installed on that host plants exactly the artifacts T05 §4–5 documents as triggers: kernel drivers/services, virtual NIC OUIs, SMBIOS/device strings, and — for Hyper-V/WSL/VirtualMachinePlatform — the Windows hypervisor itself enabled under the OS.
- T05 §4: prohibited item "Virtual machines"; hard-BLOCK error "A virtual machine has been detected. Please close all virtual machine software before continuing." This is OnVUE's **only documented decisive host-integrity control**.
- T05 §5 is the exact answer to the user's question: background/silent VM platforms **still trip the block** — Open Group: "Even if these are running silently in the background, the system check can fail." Hyper-V/VirtualMachinePlatform/WSL/Docker are called out specifically. A Type-2 VM hosting the answer tool is precisely "VM software running in the background," so OnVUE detects the hypervisor surface regardless of whether OnVUE itself is inside or outside the guest.
- **Compounding factor:** running a Type-2 VM *plus* an in-VM LLM on top of OnVUE, on a low-compute machine, is the heaviest possible footprint — doubly dead (detected AND infeasible on weak hardware).
- **macOS flag:** OnVUE's VM detection on macOS is *presumed, not separately documented* (T05 §4). A macOS Type-2 VM (Fusion/Parallels/UTM/VirtualBox) plants host kernel extensions and virtual devices analogous to Windows, so the same logic applies, but the detection is **under-documented** — treat as likely-detected, verify on the owner's macOS build if macOS is in scope.
- **Type-1 bare-metal hypervisor (OnVUE as a hardened guest) is a different case and is handled in T14** — not covered here. Note only: the Type-2 answer-in-VM question the user asked is closed; the hard, unsolved part lives in the Type-1 analysis.

---

## Q3 — Does low compute actually kill methodology #1? Three escapes.

**Conclusion: the compute limit is NOT a real barrier. It is sidesteppable two independent ways (3a near-zero compute; 3c offload), and even the on-device-model path (3b) is marginally feasible. Low compute does not kill methodology #1.**

### (3a) Pre-indexed answer bank / full-text retrieval — SURVIVES weak hardware (decisively)
- A full-text/keyword/retrieval index over the cert corpus is a lookup, not inference: memory-resident or SSD index, millisecond-scale queries, no model math. Compute cost ≈ that of a desktop search. A low-compute machine runs it trivially.
- This is the strongest compute-independent escape. Its real limiters are **not** hardware: (i) getting the corpus onto a non-pre-stageable machine (USB — Q5-A), (ii) a process/surface allowed to run (Q5-B), and (iii) item design — heavily reworded/scenario/compute-the-answer items lower hit rate vs recall-style items. Hardware is not among its constraints.
- **Verdict: compute is irrelevant to 3a.**

### (3b) Tiny quantized local model (1–3B) on a weak CPU — MARGINALLY feasible; not killed, but weakest
- **Fits in RAM:** 1B at 4-bit ≈ 0.9 GB, 3B ≈ 2 GB [R6-jacar]; OnVUE's own floor is ~4 GB RAM (T05 §1). A 1B/3B weight file fits alongside OnVUE on a low-end machine.
- **Latency on CPU:** throughput is **memory-bandwidth-bound, not core-count-bound** [R6-itsfoss], so low-end machines with slow RAM trail. Reported ranges: ~10–30 tok/s for small Q4 models on a *modern* laptop CPU; a constrained-edge arXiv test measured **3.07 tok/s for a 0.5B model** [R6-arxiv]; single-digit tok/s "technically works but feels painfully slow" [R6-itsfoss]. Label: specific tok/s figures are third-party/benchmark, **unverified** for the owner's exact hardware.
- **Why it is still usable for an exam despite weak hardware:** multiple-choice answering needs **very few output tokens** (a letter, maybe a short rationale). Even at single-digit tok/s, emitting "C" costs ~1 s; the cost is prefill (reading the question), and cert items are short-to-moderate. So latency is tolerable for recall/MCQ items even on weak CPUs, degrading on long scenario items.
- **The real ceiling for 3b is accuracy, not speed:** a 1–3B model's correctness on professional cert content is modest and may underperform a plain answer bank. So 3b is the *least attractive* of the three — but "low compute" does not categorically kill it.
- **Verdict: feasible-but-marginal; compute slows it, accuracy limits it; not a hard stop.**

### (3c) Network offload to a remote LLM/API over the center network — SURVIVES weak hardware (fully sidesteps compute)
- Offloading inference to a remote API moves **all** compute off the machine. Local footprint collapses to a thin HTTPS client plus the read/overlay channel — well within any low-compute machine.
- OnVUE does not close this on its own: T05 §7 blocks **remote-control/screen-share tools**, but there is **no evidence** OnVUE inspects or blocks arbitrary outbound HTTPS to third-party hosts (OnVUE itself needs egress to `*.onvue.com`, `*.twilio.com`, `*.wowza.com`, `*.ably.io`, port 1935 — T05 §10; general egress is a network-layer decision, not an OnVUE control).
- So 3c's gate is **network egress policy**, not compute and not OnVUE. On the *currently poorly-guarded* network it works. It is the escape that most cleanly defeats the user's compute worry — and the one the owner closes only with egress control (Q5-C), which is the *sole* control that closes it.
- **Verdict: compute is fully sidestepped; feasibility reduces to "does the network permit the call?"**

**Q3 overall:** Compute is not the barrier the user hoped. 3a ignores it, 3c removes it, 3b survives it marginally. The user's "low compute kills methodology #1" hypothesis is **rejected**.

---

## Q4 — Net feasibility verdict for methodology #1 in THIS center model

**Verdict: HIGH as currently deployed (poorly locked down) → LOW once binding controls are deployed. The outcome is governed by endpoint hardening, NOT by the compute or 4h factors the user flagged — both of which are weak barriers.**

Explicitly testing the user's hypothesis ("low compute + 4h makes it hard"): **both legs fail.**
- **4h (Q1):** the attack is pre-built off-site and merely run on-site; 4h is permissive, not constraining.
- **Low compute (Q3):** sidestepped by the answer bank (near-zero compute) and by network offload (compute moved off-box); even the on-device model is marginally usable for MCQ.
- The factors that **do** decide feasibility are delivery (USB) and execution (what's allowed to run) and egress (can it phone home) — the exact things the owner controls.

Why HIGH now: the candidate can't pre-stage the machine, but on a loosely-controlled center PC they can deliver a pre-built toolkit by USB and run the **answer-bank** path (3a, near-zero compute, in OnVUE's screen blind spot T05 §10) or the **network-offload** path (3c). Loose one-proctor-many-screens monitoring (camera/mic only) does not observe the screen, matching T06's core finding. The clarified machine ownership does **not** by itself help the defender until hardening is applied.

Why LOW after hardening: each binding control (Q5) independently removes a necessary link — delivery, execution, or egress — and the center-owned machine is precisely where those controls are enforceable (unlike T06's candidate-owned machine, where the owner had no such leverage). This machine-ownership is the owner's decisive advantage, currently unused.

Net rating for the verdict box: **High (current) / Low (hardened)** — compute+4h hypothesis **not** supported.

---

## Q5 — Binding controls the owner should deploy, each mapped to the escape it breaks, ranked by impact

Escape legend: **1** = deliver+run a pre-built portable toolkit (USB vector); **2** = Type-2 answer-in-VM; **3a** = local answer bank; **3b** = tiny local model; **3c** = network offload.

| Rank | Control | Breaks | Mechanism (defensive) |
|------|---------|--------|-----------------------|
| **1 (tie)** | **Application execution allow-listing** — WDAC signed policy (preferred; **cannot be disabled even by admin/SYSTEM**), AppLocker where per-user rules are needed [R1] | **1, 2, 3a-runner, 3b** | Only signed, enumerated binaries run. A portable helper, VM software, a model runtime (llama.cpp/ollama-class), and a custom index/search binary all fail to execute. **Caveat:** allowlisting is only as strong as its handling of built-in script hosts/LOLBINs (PowerShell, wscript, built-in search) and allowed interpreters — these must be constrained, or 3a can ride a native allowlisted tool. |
| **1 (tie)** | **Removable-media / USB control** — block USB mass-storage (allow HID only) via GPO Removable Storage Access / Device Installation Restrictions, or Intune/Defender device control [R2][R3] | **1, 3a-delivery, 3b-delivery** | USB is the *only* stated way to get a pre-built payload onto a machine the candidate cannot pre-stage. Blocking mass storage removes delivery of the portable exe, the answer bank, and the model file at the source. Pairs with allow-listing: together they are the decisive pair (delivery + execution). |
| **2** | **Network egress control / isolation** — default-deny outbound; allow only OnVUE's documented hosts/ports (`*.onvue.com`, `*.proctorcam.com`, `*.wowza.com`, `*.ably.io`, `*.twilio.com`, 1935; T05 §10). Isolate the test LAN [R4][R5][R7] | **3c (sole closer)** + blocks internet payload/model fetch | The **only** control that closes network offload — the escape that fully defeats the compute limit. A default-deny allowlist also denies pulling a model/payload from the internet if USB is blocked, closing the backup delivery route. Critical precisely because no other control touches 3c. |
| **3** | **Machine imaging & reset between sessions** — reimage from a known-good hardened golden image each session; no persistence | persistence of **1/3a/3b**; stage-in-session-1-use-in-session-2; insider tamper | Guarantees each session starts from the hardened baseline, so nothing planted survives and every attempt must re-deliver on the day — where USB/allow-listing/egress then bite. Hygiene backstop; low direct effect on a single live session, high systemic value. |
| **4** | **OnVUE's own client checks as backstop** — VM detection (T05 §4–5), continuous process scan (§3), focus/overlay lockdown (§2/§8) | **2 (decisively)**; raises cost on 3a/3b runner + overlay | VM detection already kills the Q2 route outright. Process/overlay checks add defense-in-depth but are undocumented, per-sponsor-configured, and operate on a machine the owner otherwise controls better directly — so OnVUE is a **backstop**, not the primary line. Do not rely on it in place of endpoint hardening. |

**Ranking logic:** allow-listing + USB control are the co-highest because together they sever both **execution** and **delivery** of escapes 1/3a/3b on a machine the candidate cannot pre-stage. Egress control is #2 because it is the *unique* closer of 3c (the compute-defeating escape). Imaging/reset is the persistence backstop. OnVUE's own checks are last as a backstop — though they are the component that already answers the user's VM question (Q2).

---

## Verdict box

- **Net feasibility of methodology #1 in this center model: HIGH as currently deployed (poorly locked down) → LOW once binding controls are in place.**
- **User's hypothesis ("low compute + 4h makes it hard") — REJECTED.** 4h is permissive (attack is pre-built off-site, bring-and-run — Q1). Low compute is sidesteppable: the answer bank is near-zero compute (3a) and network offload removes compute entirely (3c); even a tiny local model is marginally usable for MCQ (3b). Neither flagged factor is the barrier.
- **The VM route the user asked about (Q2) is DETECTED/BLOCKED** by OnVUE's VM detection (T05 §4–5) and is also compute-prohibitive — not viable.
- **The 2–3 controls that actually close it (deploy together):**
  1. **Application execution allow-listing (WDAC signed)** — breaks execution of the portable toolkit, VM software, model runtime, and index binary (escapes 1, 2, 3a, 3b); tamper-resistant even against SYSTEM.
  2. **Removable-media / USB blocking** — breaks the sole delivery vector onto a non-pre-stageable machine (escapes 1, 3a, 3b).
  3. **Network egress default-deny allowlist** — breaks network offload (escape 3c), the only control that does, and denies internet payload fetch.
- With all three deployed, every escape (1, 2, 3a, 3b, 3c) loses a necessary link (delivery, execution, or egress); OnVUE's VM detection + reset-to-golden-image are defense-in-depth. The center-owned machine is the owner's decisive, currently-unused advantage.

---

## Sources (numbered URLs + dates)

Internal grounding:
- **T05** — OnVUE control surface: `/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md` (cited as T05 §n) — 2026-11-12.
- **T06** — Persona A red-team inventory: `/home/user/OneVue-Reseach/slices/T06-persona-a-redteam/output.md` (methodology #1 = Family C/B; entries #1/#2/#3/#5/#6) — 2026-11-12.
- **T14** — Type-1 bare-metal hypervisor case (deferred from Q2): `/home/user/OneVue-Reseach/slices/T14-type1-hypervisor/` — 2026-11-12.

Application control (Q5-1):
1. Microsoft Learn — "WDAC and AppLocker overview" (App Control is Microsoft's recommended path; AppLocker for per-user rules on shared/kiosk devices). https://learn.microsoft.com/windows/security/threat-protection/windows-defender-application-control/wdac-and-applocker-overview — accessed 2026-11-12 — PRIMARY (vendor). Supporting (claim that a signed WDAC policy cannot be disabled even at SYSTEM, AppLocker user-mode bypassable with admin): improsec.com/tech-blog/one-thousand-and-one-application-blocks; petri.com/what-is-windows-defender-application-control — THIRD-PARTY, label **unverified**.

Removable-media / USB control (Q5-2):
2. Microsoft Learn — "Device control in Microsoft Defender for Endpoint" (granular removable-storage control; "not all USB devices are removable media devices"). https://learn.microsoft.com/defender-endpoint/device-control-overview — accessed 2026-11-12 — PRIMARY (vendor).
3. GPO removable-storage method (Computer Config > Admin Templates > System > Removable Storage Access "All Removable Storage classes: Deny all access"; Device Installation Restrictions to allow HID while blocking storage; caveat: pre-installed devices persist). ninjaone.com/blog/how-to-disable-usb-drives ; practical365.com/using-device-control-in-defender-for-endpoint-to-block-usb-drives ; petri.com/disable-access-to-removable-storage-in-windows — accessed 2026-11-12 — THIRD-PARTY how-to.

Network egress / isolation (Q5-3, Q3-3c):
4. Pearson — PVTC Technical Requirements (test centers using a firewall/web filter must allowlist Pearson domains + IP ranges). https://www.pearson.com/content/dam/pearson-sites/shared/pvtc-technical-requirements.pdf — accessed 2026-11-12 — PRIMARY (vendor).
5. ETS — CTAS Technical Configuration Requirements (isolate test LAN switch from WAN router; delivery machines on same subnet). https://www.fr.ets.org/content/dam/ets-org/pdfs/ctas/ctas-technical-configuration-requirements-guidelines.pdf — accessed 2026-11-12 — PRIMARY (vendor).
6. ProctorU — Ports and Allow-List Requirements (egress allowlist model for proctored delivery; blocked ports cause failures). https://support.proctoru.com/hc/en-us/articles/26201105201293 — accessed 2026-11-12 — THIRD-PARTY (vendor analog).

Small local LLM on weak hardware (Q3-3b):
7. arXiv 2505.15030 — llama.cpp edge-inference study (constrained-hardware throughput, e.g. ~3.07 tok/s for a 0.5B model). https://arxiv.org/abs/2505.15030 — accessed 2026-11-12 — PRIMARY (research); label specific figures **unverified** for the owner's hardware.
8. It's FOSS — "Can You Run LLMs Locally Without a GPU? I Tested 8 Models" (throughput is memory-bandwidth-bound; single-digit tok/s "painfully slow," ~15–30 tok/s "responsive"). https://itsfoss.com/testing-local-llms-without-gpu — accessed 2026-11-12 — THIRD-PARTY, **unverified**.
9. jacar.es — "Llama 3.2 at the edge" (1B@4-bit ≈ 0.9 GB, 3B ≈ 2 GB; runs on phone-class/edge). https://jacar.es/en/llama-3-2-at-the-edge-meta-bets-on-small/ — accessed 2026-11-12 — THIRD-PARTY, **unverified**.
10. Supporting tok/s ranges (small Q4 models ~10–30 tok/s on modern laptop CPUs). arcee.ai/blog/is-running-language-models-on-cpu-really-viable ; insiderllm.com/guides/cpu-only-llms-what-actually-works/ — accessed 2026-11-12 — THIRD-PARTY, **unverified**.
