# T09 OnVUE Detection-Measurement Test Design (Owner QA / Red-Team Harness)

**Purpose.** For the certification OWNER: a *design-only* harness that measures whether OnVUE
detects each top-ranked concealment strategy from T06's ranked inventory. These are the owner's
own red-team / QA tests, executed in a sandbox against a **PRACTICE / non-live** exam with
**consenting testers**. This is a *detection-measurement* instrument, not a cheating tutorial —
every block stays at the design/procedure level (OS feature state + measurement steps), never an
operational build-and-cheat recipe. **Nothing here is run; running any of it requires explicit
authorization (see final section).**

**Grounding.** All facts cite T05 (`/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md`,
cited as T05 §n) and T06 (`/home/user/OneVue-Reseach/slices/T06-persona-a-redteam/output.md`,
cited as T06 #n). Where T05 marks a control UNKNOWN/UNVERIFIED, the test's job is precisely to
*measure* that unknown; any expectation I give is **labeled `[prediction]`** and never asserted as
observed OnVUE behavior. Access date **2026-11-12**.

**PASS/FAIL convention (applies to every block).**
- **PASS = OnVUE detects or blocks the strategy** (the owner's control works): a documented
  detection signal appears (launch/readiness block, VM error string, mid-exam close-prompt,
  session flag/termination, or a proctor-review hit).
- **FAIL = the strategy evades** (a control gap): the technique provably ran (validity-confirmed,
  see Measurement-Validity note) **and no** detection signal appeared.
- For **treatment** strategies a FAIL prediction means "we expect OnVUE to miss it" (the finding
  the owner most needs). For **negative controls** a PASS prediction means "we expect OnVUE to
  catch it" — they calibrate the harness and prove the detection signals are observable.

---

## Test matrix

| ID | Strategy (T06 #) | Platform(s) | Expected PASS/FAIL `[prediction]` | Primary detection signal (T05) |
|----|------------------|-------------|-----------------------------------|--------------------------------|
| T09-01 | Non-activating on-screen overlay (#1) | Win (strong), macOS | **FAIL** (evades) on windowed; PASS if exclusive-fullscreen covers it | Focus-loss / app-switch flag; proctor-review hit (§2/§8/§10) |
| T09-02 | Fully local offline LLM + answer bank (#2) | Win, macOS | **FAIL** (evades) | Process-scan kill / mid-exam prompt; host-load telemetry (§3/§1) |
| T09-03 | Pre-exam staging / memory-resident helper (#3) | Win, macOS | **FAIL** (evades) vs snapshot gate; PASS if continuous re-scan | "Close all applications" gate; check-in process shutdown (§1/§3) |
| T09-04 | Passive foreground-preservation (#4) | Win, macOS | **FAIL** (evades) | Focus-loss flag (§2/§8) |
| T09-05 | Accessibility / UI-Automation read (#5) | Win (UIA), macOS (AX) | **FAIL** (evades) on Win; macOS may PASS via TCC grant visibility | A11y-client-attach flag; macOS permission gate (§8/§2) |
| T09-06 | Local OCR of framebuffer + local LLM (#6) | Win (DXGI), macOS (CGDisplayStream) | **split** `[prediction]`: PASS if capture-API class blocked, else FAIL | Capture/screen-share category block; mid-exam prompt (§3/§8) |
| T09-07 | Stealthed Type-1 hypervisor, OnVUE as guest (#7) | Win guest (macOS harder) | **PASS** (caught) if timing/driver check fires; FAIL if signatures-only | "A virtual machine has been detected" block (§4/§5) |
| T09-08 | GPU/device passthrough near-native guest (#8) | Win (DDA)/Linux host (VFIO) | **PASS** (caught) `[prediction]` via VM check; signatures may FAIL | VM-detected block (§4) |
| T09-09 | Fast-user-switching / 2nd concurrent session (#9) | Win (macOS FUS analog) | **unknown→measure**; `[prediction]` FAIL if scan is single-session | Cross-session process scan; proctor-review hit (§3/§10) |
| T09-10 | Virtual display adapter / IddCx (#10) | Win (IDD); macOS no clean equiv | **PASS** (caught) `[prediction]` via multi-display flag | Multi-monitor flag / launch warning (§6) |
| T09-11 | macOS platform asymmetry (TCC/Spaces) (#11) | macOS only | **split**: PASS if launch refuses w/o grants; FAIL if grant-scoping evades | macOS Privacy grant gate; parity vs Win (§2/§11) |
| T09-12 | Local answer-injection agent (#12) | Win, macOS | **FAIL** (evades), display-only strongest; injection may PASS | Injected-input flag / input-cadence review (§7/§10) |
| T09-NC1 | **[neg ctrl]** Casual VM / WSL / Hyper-V helper (T06 closed) | Win; macOS weaker | **PASS** (caught) | "A virtual machine has been detected…" block (§4/§5) |
| T09-NC2 | **[neg ctrl]** Loopback remote-desktop to hidden session (T06 closed) | Win, macOS | **PASS** (caught) | Remote-control/screen-share block; mid-exam close-prompt (§7) |

*(T06's third cleanly-closed path — audio/TTS delivery, #20 — is folded into the Measurement-Validity
note as an always-on-mic calibration check rather than a full block.)*

---

## Per-strategy test blocks

### T09-01 — Non-activating on-screen overlay (T06 #1; Family B/D)
- **Hypothesis.** OnVUE **misses** a topmost, layered, no-activate, click-through window over the
  exam on the primary display, because the lockdown models app-switch *keys* and focus loss, not a
  non-activating window (T05 §2/§8; screen unobserved §10). `[prediction]` Detection only if the
  exam surface runs exclusive-fullscreen or asserts top z-order.
- **Setup / preconditions.** Sandbox host, practice exam. Treatment arm has an overlay window
  present at a documented window level/flags (Win: topmost + no-activate + click-through; macOS:
  high `NSWindow` level + non-activating collection behavior). Control arm: identical host, no
  overlay. Vary the exam surface state across runs: **windowed/borderless vs exclusive-fullscreen**.
- **Method.** 2 arms × 2 surface states × N≥5 trials. Tester starts the practice exam normally in
  both arms. In treatment, the overlay displays benign placeholder text (e.g., a fixed watermark)
  so no real answer content is used. Record every OnVUE event. Owner-side instrumentation
  (separate from the OnVUE client) logs the overlay's actual z-order/visibility each trial to
  confirm it was really on top (validity).
- **Observable outcome = detection signal (T05).** Any of: a focus-loss / app-switch flag raised,
  a mid-exam prompt, a session flag/termination, or a proctor-review hit (§2/§8/§10). Absence of
  all = no detection.
- **PASS/FAIL.** **PASS** if OnVUE raises any signal attributable to the overlay. **FAIL** if the
  overlay is confirmed topmost-and-visible yet OnVUE raises none.
- **Platform(s) / Win vs macOS.** Windows strongest for the technique; macOS via high window level.
  `[prediction]` Similar FAIL on both when windowed; macOS compositor/Spaces behavior may differ
  under fullscreen — run both platforms to compare.
- **Safety / ethics.** Sandbox only; practice/non-live exam; consenting tester; placeholder
  (non-answer) overlay content; no live candidate. Coordinate with Pearson/sponsor before any run
  (ToS/legal). Do not retain exam content.
- **Expected result `[prediction]`.** FAIL (evades) on windowed/borderless surfaces; PASS only if
  `proctorapp` is confirmed exclusive-fullscreen or top-z-order (T06 #1 residual). This is T06's
  single highest-feasibility family — treat a FAIL here as a priority gap.

### T09-02 — Fully local offline LLM + indexed answer bank (T06 #2; Family C)
- **Hypothesis.** A quantized local model + local index produces **no exam-time network content**,
  so the telemetry/stream hosts see nothing anomalous (T05 §10) and there is no VM to detect
  (§4). `[prediction]` Residual catch route is the §3 process scan or a host-load/thermal tell.
- **Setup / preconditions.** Sandbox host with a local-inference runtime installed and the machine
  otherwise offline for exam content. Two load profiles: idle-baseline vs sustained-inference.
- **Method.** Control = exam with runtime absent. Treatment = exam with the runtime resident and
  running benign inference (no real item content). Owner instruments CPU/GPU utilization, fan/thermal,
  and the local process list independently. Capture any OnVUE process-scan action and all client
  telemetry fields the owner can observe. Vary: runtime named normally vs renamed/benign.
- **Observable outcome = detection signal (T05).** Process-scan kill or a mid-exam prompt to close
  software (§3; §1 "shuts down prohibited processes it detects"); or a readiness-gate block (§1).
  Host-load anomaly surfaced in client telemetry would be a *new* signal, not a documented one —
  log it as candidate.
- **PASS/FAIL.** **PASS** if OnVUE kills/prompts/blocks the runtime. **FAIL** if the runtime is
  confirmed running throughout and OnVUE takes no action.
- **Platform(s) / Win vs macOS.** Both; `[prediction]` equivalent. macOS process visibility to
  `proctorapp` is sandbox-bounded (§2/§11) — compare.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; benign inference only (no real
  answer bank keyed to the live pool). Pearson/sponsor coordination; ToS/legal caution.
- **Expected result `[prediction]`.** FAIL (evades) — no documented control covers passive local
  inference; the only live question is whether host-load telemetry gives the owner a usable tell.

### T09-03 — Pre-exam staging / memory-resident helper (T06 #3; Family G)
- **Hypothesis.** A helper launched and hidden **before** `proctorapp` and before the room/tech
  scan passes a *point-in-time* "close all applications" gate (T05 §1/§3). `[prediction]` Caught
  only if OnVUE re-scans continuously during the exam.
- **Setup / preconditions.** Sandbox host; a benign marker process is made resident before client
  launch and surfaces (e.g., writes a timestamped local log) only after the exam starts.
- **Method.** Control = clean machine. Treatment = marker pre-staged. Key variable: **timing of
  OnVUE's scan** — does the check-in gate catch it, and does any mid-exam re-scan catch it later?
  Owner logs the marker's liveness continuously and correlates with OnVUE events. N≥5.
- **Observable outcome = detection signal (T05).** Check-in shutdown of the process, a "close all
  applications" failure to proceed (§1/§3), or a later mid-exam prompt/flag.
- **PASS/FAIL.** **PASS** if the pre-staged marker is detected/killed at any point. **FAIL** if it
  survives from before-launch through exam end with no OnVUE action.
- **Platform(s) / Win vs macOS.** Both; `[prediction]` equivalent.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; benign marker (no answer logic);
  Pearson/sponsor coordination; ToS/legal caution.
- **Expected result `[prediction]`.** FAIL against a snapshot gate; the discriminating measurement
  is **continuous vs check-in-only** scanning (T06 #3 signal) — the harness directly tests which.

### T09-04 — Passive foreground-preservation (T06 #4; Family D)
- **Hypothesis.** A helper whose every interaction is non-activating (background render/read/control)
  never changes `GetForegroundWindow`/focus, so the §2/§8 focus-loss flag never fires.
- **Setup / preconditions.** Sandbox host; helper components configured to never take foreground.
  Include a variant that installs a global input hook (for a hotkey) vs one that does not.
- **Method.** Control = foregrounding helper (activates a window) as a *positive* reference that
  *should* trip focus-loss. Treatment = non-activating helper. Owner instruments foreground-window
  identity and any global input hooks independently. Compare whether OnVUE flags the activating
  reference but not the passive one. N≥5.
- **Observable outcome = detection signal (T05).** Focus-loss / app-switch flag; mid-exam prompt;
  proctor-review hit (§2/§8/§10).
- **PASS/FAIL.** **PASS** if the passive helper triggers a focus/app-switch signal. **FAIL** if the
  activating reference trips a signal but the passive helper (confirmed never-foreground) does not.
- **Platform(s) / Win vs macOS.** Both; `[prediction]` equivalent. On Windows also test whether a
  global input hook alone (no focus change) draws a signal.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; benign helper; Pearson/sponsor
  coordination; ToS/legal caution.
- **Expected result `[prediction]`.** FAIL (evades) for the pure non-activating case; the
  activating reference arm should PASS and thereby validate the focus-loss signal is observable.

### T09-05 — Accessibility / UI-Automation read channel (T06 #5; Family C)
- **Hypothesis.** A UIA (Win `TextPattern`) / AX (macOS `AXUIElement`) client can read the exam's
  exposed UI text passively — no capture, no focus change — and OnVUE does not flag the attach
  (T05 §8 shows a11y is *tolerated with accommodation*). `[prediction]` macOS is weaker for the
  attacker because the AX/Screen-Recording grant is visible in Privacy settings (§2).
- **Setup / preconditions.** Sandbox host; an assistive-tech-style client attaches to `proctorapp`
  and reads exposed text (benign logging only). macOS arm: Accessibility (and, if required,
  Screen-Recording) permission granted.
- **Method.** Control = no a11y client. Treatment = a11y client attached and reading. Variable:
  whether the exam renders in a text-exposing control vs a canvas/custom control (which forces the
  degrade-to-OCR path, cross-ref T09-06). Owner instruments the UIA/AX attach
  (`UiaClientsAreListening` / `AXObserver`) and whether any text was actually readable (validity).
- **Observable outcome = detection signal (T05).** A flag for an assistive-tech client attaching
  without an accommodation on record; or, macOS, the permission gate itself. Absence = no detection.
- **PASS/FAIL.** **PASS** if OnVUE flags/blocks the unexpected a11y client. **FAIL** if the client
  is confirmed attached-and-reading and OnVUE raises nothing.
- **Platform(s) / Win vs macOS.** Win (UIA) vs macOS (AX). `[prediction]` Win FAIL (silent attach);
  macOS may PASS indirectly because granting AX is user-visible and auditable (§2/§11). Run both.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; read benign placeholder content,
  not live items; Pearson/sponsor coordination; ToS/legal caution.
- **Expected result `[prediction]`.** FAIL on Windows (silent UIA read); macOS split — the grant
  visibility is the owner's leverage (T06 #5/#11 signals).

### T09-06 — Local OCR of the exam framebuffer + local LLM (T06 #6; Family C)
- **Hypothesis.** `[prediction]` This is the highest-risk C-family path: desktop-duplication/capture
  (Win DXGI `IDXGIOutputDuplication`, macOS `CGDisplayStream`) is **exactly** the "screen-capture
  tools" category Browser Lock blocks (T05 §3/§8). The bet is that a renamed agent dodges
  *signature-based* blocking; the test measures whether OnVUE blocks the *API class* or only
  named tools.
- **Setup / preconditions.** Sandbox host; a benign capture agent uses the duplication API against
  the exam region (captures to a local checksum, not stored imagery). Variant A: agent named like a
  known capture tool. Variant B: renamed/benign.
- **Method.** Control = no capture. Treatment A/B as above. Owner instruments duplication-API calls
  independently and confirms frames were actually captured (validity). Record OnVUE behavior:
  does capture fail/deny, does the client prompt, does the session flag?
- **Observable outcome = detection signal (T05).** Capture denied/hooked, a screen-capture category
  block, a mid-exam prompt to close the tool (analogous to §7 Remote Utilities close-prompt), or a
  session flag (§3/§8).
- **PASS/FAIL.** **PASS** if OnVUE denies/blocks/flags the capture (either variant). **FAIL** if a
  renamed agent captures the exam framebuffer with no OnVUE response.
- **Platform(s) / Win vs macOS.** Win (DXGI) vs macOS (CGDisplayStream). macOS additionally gates
  screen capture behind a Screen-Recording TCC grant (§2/§11) — `[prediction]` macOS more likely
  PASS via that gate; Windows is the open question.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; checksum-only capture (no exam
  imagery retained); Pearson/sponsor coordination; ToS/legal caution.
- **Expected result `[prediction]`.** Split: PASS if OnVUE blocks the duplication API class
  (likely, since §3/§8 names this category); FAIL for a renamed agent if blocking is signature-only.
  This is the test most likely to catch a treatment strategy among the C family.

### T09-07 — Stealthed Type-1 hypervisor, OnVUE as hardened guest (T06 #7; Family A/E)
- **Hypothesis.** OnVUE's **only documented hard-BLOCK** is VM detection (T05 §4). The question is
  *how* it detects: signature tells (CPUID hypervisor bit, paravirt drivers, SMBIOS/MAC strings)
  are maskable, but **timing-based** detection (CPUID VM-exit latency, RDTSC) is the open, hard
  part (T06 #7). `[prediction]` A signature-masked guest may evade; a timing/driver check would
  catch it.
- **Setup / preconditions.** Sandbox: OnVUE running inside a guest on a bare-metal hypervisor, with
  the guest in escalating hardening states. **This is a measurement of the detection boundary, not
  a build guide** — states are described as feature levels: (S0) stock guest, (S1) signature-masked
  guest, (S2) signature-masked + timing-smoothed guest. Bare-metal control host for baseline.
- **Method.** Launch OnVUE readiness check + practice exam in each state S0–S2 and on bare metal.
  For each, record whether the VM-detected block fires. The progression localizes *which* class of
  tell OnVUE relies on (string/driver vs timing). Owner runs an independent timing red-pill probe
  (CPUID/RDTSC VM-exit latency vs bare-metal baseline) to characterize the environment (validity).
- **Observable outcome = detection signal (T05).** The exact block string **"A virtual machine has
  been detected. Please close all virtual machine software before continuing."** and inability to
  continue (§4); or a readiness-gate failure (§1); or VM false-positive triggers (§5).
- **PASS/FAIL.** **PASS** if OnVUE blocks in a given state. **FAIL** if a hardened state (S1/S2) runs
  the exam to completion with the hypervisor confirmed present (via the owner's red-pill probe).
- **Platform(s) / Win vs macOS.** Windows guest primary; macOS guest is harder/licence-constrained
  (T06 #7) — `[prediction]` run Windows first; note macOS as a secondary, lower-feasibility arm.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; no live exam; Pearson/sponsor
  coordination strongly advised (VM use is explicitly prohibited by policy, §4 — do not attempt on
  a live booking); ToS/legal caution is highest here.
- **Expected result `[prediction]`.** PASS at S0 and likely S1 via the documented block (§4);
  the decisive unknown is S2 — if OnVUE relies on signatures only, S2 FAILs (evades), which is why
  T06 recommends the owner add a **timing red-pill** (the control this test tells them whether they
  need).

### T09-08 — GPU/device passthrough for near-native guest (T06 #8; Family E)
- **Hypothesis.** Passthrough (Win DDA host / Linux VFIO host) makes graphics and render timing look
  native, targeting OnVUE's *signature* VM heuristics (T05 §4). `[prediction]` Timing side channels
  and missing physical-sensor coherence (battery/thermal) remain tells.
- **Setup / preconditions.** Sandbox: guest with a passed-through GPU, SMBIOS/serial/MAC at
  bare-metal values, guest-additions suppressed (feature-level description, not a build script).
  Bare-metal control for sensor-coherence baseline.
- **Method.** Run readiness + practice exam in the passthrough guest and on bare metal. Record VM
  detection outcome. Owner independently checks TPM-measured-boot attestation and device-sensor
  coherence (battery discharge curve, thermal) to characterize what a passthrough guest cannot
  forge (this is the candidate *new* control, per T06 #8 signal).
- **Observable outcome = detection signal (T05).** VM-detected block (§4); readiness-gate failure
  (§1). No block = no detection.
- **PASS/FAIL.** **PASS** if OnVUE blocks. **FAIL** if the passthrough guest completes with the
  hypervisor confirmed present.
- **Platform(s).** Win (DDA) / Linux host (VFIO); guest is Windows. macOS n/a as host here.
- **Safety / ethics.** As T09-07 (VM policy-prohibited; sandbox/practice only; Pearson/sponsor
  coordination; highest ToS/legal caution).
- **Expected result `[prediction]`.** PASS via the §4 block if OnVUE's check includes any timing or
  hypervisor-present element; FAIL against pure signature heuristics. Pairs with T09-07 to tell the
  owner whether signature-only detection is sufficient (prediction: it is not).

### T09-09 — Fast user switching / second concurrent session (T06 #9; Family B)
- **Hypothesis.** If OnVUE's process scan enumerates only the current interactive session (scope
  undocumented, T05 §3), a helper in a second user session or a Session-0 service is outside its
  view. `[prediction]` FAIL if single-session scope; PASS if it enumerates all sessions.
- **Setup / preconditions.** Sandbox host with a second logged-in session (Win FUS; macOS FUS
  analog) hosting a benign marker process; OnVUE in the primary interactive session. The tester
  does **not** switch sessions during the exam (that would be an app-switch/focus event, §2/§8).
- **Method.** Control = single session. Treatment = benign marker in a second session / Session-0
  service. Owner enumerates processes across all sessions independently (validity). Record whether
  OnVUE detects the secondary session or its process, or flags the multi-session state.
- **Observable outcome = detection signal (T05).** A process-scan hit on the cross-session marker
  (§3), a readiness block, or a proctor-review/flag for a secondary interactive session (§10).
- **PASS/FAIL.** **PASS** if OnVUE detects the second session or its process. **FAIL** if the
  cross-session marker is confirmed running and OnVUE never sees it.
- **Platform(s) / Win vs macOS.** Win (FUS) primary; macOS FUS analogous — `[prediction]` compare,
  as session-enumeration behavior may differ by platform.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; benign marker; Pearson/sponsor
  coordination; ToS/legal caution.
- **Expected result `[prediction]`.** Unknown→measure; T06 ranks this **Med** with `[prediction]`
  FAIL if the scan is single-session. The matrix flags this as the test whose result most directly
  scopes OnVUE's process-scan reach.

### T09-10 — Virtual display adapter (IddCx) as off-camera surface (T06 #10; Family E)
- **Hypothesis.** A Windows Indirect Display Driver (IddCx) adds a user-mode virtual monitor; the
  attacker hopes OnVUE counts only physical outputs (T05 §6). `[prediction]` A virtual head usually
  **adds** an enumerated display → trips the very multi-monitor flag it hoped to dodge.
- **Setup / preconditions.** Sandbox Windows host with a virtual display adapter present (IddCx).
  Control: single physical display only.
- **Method.** Control = one physical display. Treatment = virtual head added. Run readiness +
  practice exam. Owner enumerates all adapters/monitors including indirect/virtual and non-PnP
  (validity — confirm the virtual head is live). Record OnVUE's display-count behavior.
- **Observable outcome = detection signal (T05).** Multiple-monitor flag, the launch-time "extra
  monitor" warning (§6, reported but mechanism UNVERIFIED), a single-display readiness block, or a
  proctor-review/termination (§6/§10).
- **PASS/FAIL.** **PASS** if OnVUE counts the virtual head and flags/blocks multi-display. **FAIL**
  if the virtual head is confirmed active and OnVUE still reports a single display.
- **Platform(s) / Win vs macOS.** Windows (IDD) only; macOS has **no clean equivalent** (T06 #10)
  — mark macOS N/A for this technique and note it as a platform asymmetry (attacker tool is
  Windows-only here).
- **Safety / ethics.** Sandbox; practice exam; consenting tester; Pearson/sponsor coordination;
  ToS/legal caution.
- **Expected result `[prediction]`.** PASS (caught) via the §6 multi-monitor flag **if** OnVUE's
  enumeration includes indirect/virtual adapters; FAIL only if it counts physical panels exclusively
  (T06 #10 signal — the owner should verify enumeration covers IddCx/non-PnP).

### T09-11 — macOS platform asymmetry (TCC / Spaces / sandbox) (T06 #11; Family F)
- **Hypothesis.** macOS enforcement is bounded by user-granted TCC permissions (Camera, Mic,
  Automation, Input Monitoring; T05 §2) and sandbox limits on what `proctorapp` can learn about
  other apps. `[prediction]` Either OnVUE refuses to launch without the full grant set (PASS), or a
  candidate can scope/withhold a grant or exploit full-screen-as-own-Space separation (FAIL).
- **Setup / preconditions.** Sandbox macOS 14+ host (§11 floor). Arms vary the grant set: (G0) all
  four grants + Screen-Recording; (G1) one non-essential grant withheld; (G2) Spaces/Mission-Control
  separation exercised. Windows host as the parity reference.
- **Method.** For each grant state, attempt readiness + practice exam. Record whether OnVUE
  launches, what it demands, and whether a withheld grant or Spaces separation opens a gap a
  Windows run would not have. Run the **same** treatment tests (T09-01/04/05) on macOS vs Windows
  to measure whether the weaker platform is a soft target (T06 #11 signal: parity testing).
- **Observable outcome = detection signal (T05).** Launch refusal / permission gate until grants
  are complete (§2); a readiness block; or, conversely, successful exam with a grant withheld =
  a scoping gap. Compare the macOS-vs-Windows detection delta on shared treatment tests.
- **PASS/FAIL.** **PASS** if OnVUE enforces the full grant set and matches Windows detection on the
  shared tests. **FAIL** if a withheld grant / Spaces separation lets a treatment technique evade on
  macOS that Windows catches (a platform-asymmetry gap).
- **Platform(s).** macOS only (with Windows as the parity baseline).
- **Safety / ethics.** Sandbox; practice exam; consenting tester; Pearson/sponsor coordination;
  ToS/legal caution.
- **Expected result `[prediction]`.** Split: PASS on launch-grant enforcement (OnVUE likely refuses
  without grants, §2); FAIL risk is the **parity delta** — the harness's real output is a
  side-by-side table of which treatment tests macOS misses that Windows catches (or vice-versa).

### T09-12 — Fully local answer-injection agent (T06 #12; Family C)
- **Hypothesis.** A local agent that selects answers via synthetic input (`SendInput`/CGEvent) is a
  *local* action, not a remote-control session, so §7 remote-control detection does not model it
  (T05 §7). `[prediction]` Residual tells: inhuman input cadence/precision and (Win) the injected-
  input flag; display-only (human clicks) removes most risk.
- **Setup / preconditions.** Sandbox host. Arm A: display-only (agent shows a benign marker; the
  human clicks). Arm B: synthetic-input (agent injects benign navigation into non-scored fields,
  e.g., moving between practice items — **no answer logic against a live pool**).
- **Method.** Control = human-only input. Treatment A/B. Owner instruments injected-input flags
  (Win `LLKHF_INJECTED`) and input-timing cadence independently. Record whether OnVUE flags the
  injection, prompts, or treats it as a remote-control session.
- **Observable outcome = detection signal (T05).** A remote-control/screen-share style block or
  mid-exam prompt (§7); an injected-input flag surfaced by the client; or a proctor-review hit on
  input-cadence analytics (§10). Absence = no detection.
- **PASS/FAIL.** **PASS** if OnVUE flags/blocks the synthetic input. **FAIL** if injected input is
  confirmed delivered to the exam with no OnVUE response (display-only arm is expected trivially to
  evade, since it is ordinary human input).
- **Platform(s) / Win vs macOS.** Both; Windows exposes `LLKHF_INJECTED` (a candidate owner signal).
  `[prediction]` equivalent miss on display-only; injection arm is the discriminating case.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; injection only into non-scored
  navigation, never a real answer bank; Pearson/sponsor coordination; ToS/legal caution.
- **Expected result `[prediction]`.** FAIL for display-only (High feasibility in T06); synthetic
  input may PASS if OnVUE surfaces the injected-input flag or cadence analytics — the test tells the
  owner whether injection is a usable tell they already have.

---

## Negative controls (expected: OnVUE catches — calibrate the harness)

### T09-NC1 — Casual VM / WSL / Hyper-V helper **[negative control]** (T06 closed path; T05 §4–5)
- **Hypothesis.** OnVUE **catches** a candidate running OnVUE inside, or alongside, a stock VM /
  WSL / Hyper-V / VirtualMachinePlatform — it over-blocks these even in the background (T05 §5;
  T06 "What is NOT the answer").
- **Setup / preconditions.** Sandbox host in states: (a) stock VM guest running OnVUE; (b) bare
  metal with Hyper-V/VirtualMachinePlatform/WSL enabled but idle; (c) clean bare-metal control.
- **Method.** Attempt readiness + practice exam in each state. Record the block and its exact
  string. This confirms the VM-detected signal is observable (calibration for T09-07/08).
- **Observable outcome = detection signal (T05).** **"A virtual machine has been detected. Please
  close all virtual machine software before continuing."** and inability to continue (§4); or the
  documented false-positive block from idle Hyper-V/WSL surfaces (§5).
- **PASS/FAIL.** **PASS** (expected) if OnVUE blocks. A **FAIL** here would mean the documented
  §4/§5 control is weaker than T05 records — itself a high-value finding.
- **Platform(s) / Win vs macOS.** Windows: documented block (§4/§5). macOS: VM detection is
  "presumed, not separately documented" (T05 §4) — `[prediction]` run macOS as a **sub-measurement**:
  if macOS does **not** block a stock VM, that is a platform-asymmetry gap, not an expected pass.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; VM use is policy-prohibited — do
  not run on a live booking; Pearson/sponsor coordination; ToS/legal caution.
- **Expected result `[prediction]`.** PASS on Windows (confirms the one documented hard-BLOCK);
  macOS result is itself the measurement (T05 flags it UNVERIFIED).

### T09-NC2 — Loopback remote-desktop to a hidden session **[negative control]** (T06 closed path; T05 §7)
- **Hypothesis.** OnVUE **catches** remote-desktop / screen-share to a hidden local session — the
  remote-control/screen-share category block (T05 §7; §3/§8), including loopback RDP/VNC (T06
  "Still catches").
- **Setup / preconditions.** Sandbox host running a remote-control/screen-share tool to a loopback
  or LAN session while OnVUE runs. Control: tool absent.
- **Method.** Attempt readiness + practice exam with the tool active. Record whether OnVUE blocks at
  launch or prompts mid-exam (the §7 Remote Utilities anecdote: candidate was prompted to close the
  tool mid-exam). This calibrates the mid-exam close-prompt signal.
- **Observable outcome = detection signal (T05).** Launch block of remote-control/screen-share
  category (§7), or a **mid-exam prompt to close the tool** (§7 anecdote), or a session flag.
- **PASS/FAIL.** **PASS** (expected) if OnVUE blocks or prompts. A FAIL would contradict the
  documented §7 block — high-value if observed.
- **Platform(s) / Win vs macOS.** Both; `[prediction]` both blocked per §7.
- **Safety / ethics.** Sandbox; practice exam; consenting tester; Pearson/sponsor coordination;
  ToS/legal caution.
- **Expected result `[prediction]`.** PASS on both platforms (confirms the §7 category block and the
  mid-exam close-prompt signal are live).

---

## Measurement-validity note (avoiding false negatives and confounds)

A **false negative** here is the worst outcome: concluding "OnVUE misses X" when X never actually
ran. Controls to prevent it:

1. **Confirm the technique ran (independent of OnVUE).** Every treatment block requires owner-side
   instrumentation separate from the OnVUE client that *proves the technique was live during the
   exam window* — overlay confirmed topmost-and-visible (T09-01), runtime confirmed executing
   (T09-02), marker confirmed resident from before launch (T09-03), never-foreground confirmed
   (T09-04), UIA/AX confirmed attached-and-reading actual text (T09-05), frames confirmed captured
   (T09-06), hypervisor confirmed present via red-pill probe (T09-07/08), cross-session process
   confirmed running (T09-09), virtual head confirmed enumerable (T09-10), grant state confirmed
   (T09-11), injected input confirmed delivered (T09-12). A "FAIL (evades)" is only valid when this
   liveness check passed in the same trial.
2. **Positive-control / calibration arms.** The two negative controls (NC1 VM, NC2 remote-desktop)
   and the activating-helper reference in T09-04 prove the detection signals (VM string, close-prompt,
   focus-loss flag) are observable on this harness. If a negative control fails to fire, the harness
   is mis-instrumented — do not trust that session's treatment FAILs. The always-on-mic / audio path
   (T06 #20) can be used as a further calibration: a benign TTS beep should produce an audio/voice
   flag (§10); if it does not, mic-side signals are not being captured.
3. **Per-exam-sponsor configuration confound.** T05's primary caveat: OnVUE behaviour is
   **configured per exam sponsor** (§ Summary caveat). A result is only valid for the specific
   sponsor/exam profile tested. Record the sponsor, exam form, OnVUE client version (e.g., the
   3.27.23-class version in §7), and date for every trial; do not generalize across sponsors.
4. **Version / platform drift.** OnVUE client version, OS build, and macOS floor (docs disagree
   10.15 vs 14, T05 §11) all shift behaviour. Pin and log versions; re-baseline when any changes.
5. **Signature-vs-class confound (T09-06, T09-07/08, NC1).** Distinguish "named-tool signature block"
   from "capability/API-class block" by running named vs renamed variants (T09-06) and stock vs
   masked guest states (T09-07/08). A single arm cannot tell these apart.
6. **Screen-unobserved assumption.** The whole design rests on T05 §10 (screen not captured in the
   defended deployment). Re-verify this holds for the tested sponsor before interpreting any
   "answer-on-screen" FAIL; a sponsor profile that *does* stream the screen invalidates the overlay/
   OCR/read family's premise.
7. **Order / learning effects and proctor variance.** Human-proctor flags (§10) vary by proctor.
   Randomize arm order, use multiple proctors/sessions, and treat a single proctor-review hit as
   weak evidence — require N≥5 and report rates, not single trials. Separate *automated* client
   signals (reproducible) from *human* review hits (variable) in the results.
8. **Do not use live content.** Using the live item pool would both invalidate results (sponsor may
   react to content exposure, not technique) and breach policy. All arms use practice/placeholder
   content so the only variable is the *technique*, not the *answers*.

---

## Authorization and legal statement (required before any run)

**This document is a design only. Running ANY test here requires the user's explicit approval and
sponsor/vendor authorization.** Executing these procedures may implicate **Pearson VUE's terms of
service and the exam-sponsor agreements**, and several arms exercise behaviours those agreements
and OnVUE policy **expressly prohibit** (virtual machines and VM-ish surfaces, T05 §4–5; remote-
control/screen-share tools, §7; multiple displays, §6). Specific requirements before execution:

- Obtain written authorization from the certification owner/sponsor **and** coordinate with Pearson
  VUE; do not run against any **live** exam or booking under any circumstances — **practice / non-
  live exam only**, in an isolated sandbox, with **consenting testers** and **no live candidate**.
- Treat VM-based arms (T09-07, T09-08, T09-NC1) and the remote-desktop control (T09-NC2) as the
  highest ToS/legal exposure; they must not touch a real candidate account or live delivery.
- Retain no live exam content; log only technique-liveness and OnVUE-signal data.
- If Pearson/sponsor authorization is not granted, the harness remains a paper design and is not
  executed. The owner should get authorization and legal review before scheduling any run.

---

## Source grounding

- **T05** — OnVUE control surface: `/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md`
  (cited as T05 §n). Key anchors used: §1 readiness gate / check-in process shutdown; §2 Browser
  Lock + macOS grants; §3 process scan (enumeration method UNDOCUMENTED); §4 VM-detected block
  string; §5 hypervisor/VBS UNKNOWN + false-positive trigger set; §6 single-display flag
  (launch-block mechanism UNVERIFIED); §7 remote-control/screen-share block + mid-exam close-prompt
  anecdote; §8 capture/overlay block + a11y carve-out; §9 virtual-cam detection UNKNOWN; §10 screen
  NOT captured, webcam/mic/telemetry/biometrics; §11 platform support (Linux gap, macOS floor
  inconsistency). Primary caveat: per-sponsor configuration.
- **T06** — Persona A red-team inventory:
  `/home/user/OneVue-Reseach/slices/T06-persona-a-redteam/output.md` (cited as T06 #n). Treatment
  strategies drawn from the ranked inventory (#1–#12) at High / Med-High / Med feasibility; negative
  controls drawn from T06's "What is NOT the answer (closed paths)" and "Still catches" sections;
  per-strategy detection signals adopted from T06's "single defender detection signal to add" column.

All expectations above are labeled `[prediction]` and derived from T05/T06; no OnVUE behaviour is
asserted as observed. **Design only — not run.**
