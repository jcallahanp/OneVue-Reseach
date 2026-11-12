# T08 Persona B Defensive Synthesis

Scope: The exam owner's (Persona B's) answer — the rules, proctoring procedures, and vendor selection that make the current OnVUE deployment's detection holes materially harder for a sophisticated candidate (Persona A) to circumvent. Defensive posture only; no bypass instructions. Grounded in T05 (OnVUE control surface), T06 (Persona A red-team inventory), T07 (vendor control comparison). All slice access dates 2026-11-12. No additional web research was needed — the three slices establish every sourced fact used here; claims resting only on third-party/anecdotal slice sources are labeled.

**Fixed context (not contradicted):** high-stakes technical cert; sophisticated candidates; current defended deployment = 360° front camera + manual pre-exam check for confederates/unapproved tech, then **NO screen monitoring and NO computer monitoring during the exam** — only camera video, mic audio, OnVUE client telemetry/biometrics. Central weakness: the **screen is unobserved**, so only OnVUE's launch-time client-side checks guard against a concealed same-machine second workspace.

**Interpretation note — "G1 gaps":** I read the Goal-1 gap as the central weakness above — a concealed same-machine second workspace (local LLM/answer engine surfaced on the candidate's own primary display) that the screen-blind configuration cannot see (T06 Executive answer; T07 bottom line). The roadmap (§6) ranks by impact on closing that gap.

---

## 1. PERSONA B DIRECT ANSWER

**The single biggest change** is to stop operating in the screen-blind, launch-only configuration. Concretely, in descending order of what actually moves the needle:

- **If the "no screen/computer monitoring" constraint can be lifted (biggest lever overall):** reinstate **continuous screen capture/recording for the whole session** *and* pair it with a **native continuous host agent** (see §4). OnVUE's product is already capable of screen capture; this deployment simply has it OFF (T07 S35; T07 OnVUE note). Screen capture alone is **not sufficient** — a capture-excluded overlay (Windows `WDA_EXCLUDEFROMCAPTURE`) sits below naive screen/tab capture and is only caught by OS-level overlay/top-most-window + process + GPU/network enumeration (T07 S32). So the biggest change is "screen capture **plus** native overlay/process enumeration," not either alone.

- **If the constraint is immovable (biggest in-constraint change):** convert host integrity from **point-in-time at launch** (what OnVUE does today — names and closes disallowed processes at launch, VM-blocks at launch; T05 §1/§3; T07 OnVUE note) to **continuous, all-session process + overlay + display enumeration** by selecting a vendor whose native agent runs throughout the exam. This is the one control that catches a **pre-staged helper process** and a **concealed LLM** even while the screen stays unobserved (T06 #2/#3; T07 §Bottom line). It does not need the screen to be watched.

**The layered rest** (each detailed in §2–§4), applied as defense-in-depth because no single control closes the Type-1-hypervisor and capture-excluded-overlay residuals (§5):
1. Native continuous host agent (vendor change) — continuous process enum across **all sessions + Session 0**, overlay/z-order audit, all-adapter display enumeration. Closes T06 families **G, B, C, E**.
2. Pre-exam **witnessed clean reboot** + continuous (not launch-only) scanning — cheaply defeats the memory-resident pre-stage (T06 #3, family G).
3. **Assistive-tech-attach detection** + **IME/text-service lock** + **host-load/GPU telemetry** — closes the passive read + local-LLM channels (T06 #2/#5/#18, family C).
4. **Hardware attestation (TPM/Secure Boot/measured boot) + a timing-based hypervisor probe** — the *only* path that closes the Type-1/passthrough residual (T06 #7/#8; T07 cross-cutting). No proctoring vendor ships this today; it is added tooling / a constraint change.
5. Keep the sensors that already work: **mic voice/TTS analytics** (closes audio delivery, T06 #20) and **camera biometrics + random liveness** (closes virtual-cam/liveness fraud, T06 #16). Do **not** over-invest in gaze analytics against the primary threat — an overlay on the **primary** display keeps gaze natural on camera (T06 Executive answer).

**One-line program:** *Move off the screen-blind OnVUE config to a native, continuously-running host agent (ProctorU/Guardian class) that enumerates processes + overlay windows + displays all session long; re-enable screen capture where policy permits; add attestation for the top tier; and keep mic/camera analytics for the sensors the screen-blindness does not relax.*

---

## 2. RULE SET (each mapped to the T06 family it blunts)

Enforceability key: **[OnVUE-now]** current feature set can enforce in this deployment · **[OnVUE-capable, OFF]** OnVUE product can do it but this deployment disabled it (= constraint change) · **[vendor]** needs a different vendor's native continuous agent · **[tooling]** needs added tooling beyond any current proctoring product.

| # | Rule / policy | T06 family / technique blunted | OnVUE control defeated (T05) | Enforceability |
|---|---------------|-------------------------------|------------------------------|----------------|
| R1 | **Require continuous screen capture/recording for the full session** (the "view-barrier" rule) | B (#1 overlay), C (#2/#6 local LLM display, #5 read), the "must VIEW answers" barrier (T06 Exec) | Screen-unobserved gap §10 | **[OnVUE-capable, OFF]** — constraint change. Pair with R3 (capture-excluded overlays beat capture alone, T07 S32) |
| R2 | **Require a native continuous host agent; ban extension-only/pixels-only proctoring** | All same-machine families; structurally B/C/D/G | Launch-only scan §1/§3 → continuous | **[vendor]** — browser-extension products cannot see VM/overlay/process (T07 S29/S32) |
| R3 | **Continuous, all-session process enumeration** (across every logged-in session + Session 0), baseline-diff vs pre-launch state | G (#3 pre-stage), C (#2 local LLM process, #6 OCR agent), B (#9 second-session) | §3 process scan is point-in-time + single-session; §1 | **[vendor]** (ProctorU re-checks every 30s, logs all processes — T07 S15) |
| R4 | **Top-of-z-order / overlay-window audit:** flag non-exam layered-topmost / high-level / PiP / click-through windows during the exam | B/D (#1 overlay, #4 passive foreground, #15 PiP) | Focus/app-switch lockdown §2/§8 (models keys & focus loss, not non-activating windows) | **[vendor]** (Constructor blocks overlays; Rosalyn screen-visual detection — T07 S23/S28); **[tooling]** to add to OnVUE |
| R5 | **Single-display attestation by full adapter/monitor enumeration** — count indirect/virtual (IddCx), non-PnP, and mirrored heads, not just physical panels; hard-block launch on >1 | E (#10 virtual display), B | Single-display check §6 (launch-block mechanism unverified; virtual head may be miscounted) | **[vendor]** (Respondus blocks virtual displays/desktops — T07 S1; Proctorio counts displays — S42); **[tooling]** |
| R6 | **Boot-state / OS lockdown:** require a clean, non-virtualized, **consumer (non-managed)** OS; ban/verify hypervisor features (Hyper-V, VBS/HVCI, WSL, Docker, nested virt) with a read of the hypervisor-present flag | A/D (#7 Type-1, #13 VBS/nested) | VM detection §4; undocumented hypervisor-state handling §5 | **[OnVUE-now]** partially — OnVUE already over-blocks WSL/Hyper-V/VBS surfaces (T05 §5), at the cost of false positives; explicit VBS/HVCI verification is **[tooling]** |
| R7 | **Hardware attestation requirement:** TPM 2.0 + Secure Boot + measured-boot attestation of a known-good, non-virtualized boot chain; refuse bare-metal/Type-1 hypervisor | A/E (#7 Type-1, #8 GPU passthrough) — the decisive residual | VM detection §4–5 (only heuristic today) | **[tooling]** — **no vendor documents this** (T07 cross-cutting, S47 negative result). The only control that closes R1-of-§5 |
| R8 | **Assistive-tech governance:** lock text-input services/IME to a known-good set; require any screen-reader/magnifier accommodation to be **pre-approved and allow-listed**, and flag unexpected UIA/AX clients attaching to `proctorapp` | C (#5 a11y read, #18 IME) | a11y carve-out §8 (tolerated → abused) | **[tooling]** (UIA/AX attach detection, IME lock); accommodation allow-list is **[OnVUE-now]** as policy |
| R9 | **Injected-input & input-cadence policy:** treat synthetic input (`LLKHF_INJECTED`) and inhuman answer-cadence as violations | C (#12 answer-injection) | Remote-control detection §7 does not model local `SendInput` | **[vendor]/[tooling]** — post-exam analyzable from telemetry |
| R10 | **Audio rule (keep):** any speech/sound = violation; run voice/TTS detection on the mic stream | C (#20 TTS) | Already closed by always-on mic §10 | **[OnVUE-now]** |
| R11 | **Camera-authenticity rule (keep + extend):** ban webcam filters/virtual cameras; enforce random liveness challenges and virtual-camera device-path detection | cam (#16 virtual-cam loop) | Virtual-cam detection UNKNOWN §9; biometrics §10 | **[OnVUE-now]** (filter ban, biometrics) + **[tooling]** (device-path detection) |

**Rules that look appealing but are weak against the *primary* threat (state honestly):** gaze/head-pose analytics (the overlay keeps gaze on the primary display — T06 Exec); virtual-desktop/Space-switch hooks (the switch is already an app-switch event the lockdown catches — T06 #17, largely redundant). Keep them cheap; don't rely on them.

---

## 3. PROCTORING PROCEDURES (operational detection the software lacks)

Each step adds human/operational detection beyond the client. Mapped to T06 family, with realistic trade-offs.

### Check-in (before launch)
- **Witnessed clean reboot immediately before the exam.** Forces any memory-resident helper to re-stage *after* the clean gate, where continuous scanning (R3) can catch it. Blunts **G (#3)**. *Trade-off:* +3–5 min friction; defeated if the candidate automates re-staging, so it must pair with continuous (not point-in-time) scanning.
- **Live Task Manager / full-process + display-enumeration walk-through on camera** (candidate screen-shares the process list and display config to the proctor once). Blunts **G/C/E** at a basic level. *Trade-off:* a renamed/signed benign-looking helper (T06 #3/#6) survives; catches only the careless. Low cost.
- **Capture-exclusion probe:** have the candidate drag a proctor-supplied marker window across the screen during the screen-share handshake; a capture-excluded overlay region reveals itself as a seam/gap. Blunts **B (#1)**. *Trade-off:* requires screen-share enabled (R1, constraint change); false positives from compositor quirks.
- **Attestation read-out** (if R7 adopted): proctor confirms TPM/Secure-Boot/measured-boot pass before admission. Blunts **A/E**. *Trade-off:* excludes legitimately non-TPM or Linux hardware; higher support load.

### During the exam
- **Continuous host-agent telemetry monitored** (process/overlay/display/host-load deltas surfaced to proctor or AI). Blunts **G, B, C, E**. *Trade-off:* needs the vendor agent (§4); VM false-positives from leftover virtual NIC/disk drivers are common (T05 §5), raising benign-flag rates.
- **Random liveness challenges** (camera) + mic voice/TTS monitoring. Blunts **#16, #20**. *Trade-off:* interrupts flow; privacy-sensitive; biometric retention caps apply (T05 §10, Illinois 3-yr).
- **Host-load / GPU-utilization watch:** sustained inference-class load during a light exam is a tell for a local LLM. Blunts **C (#2)**. *Trade-off:* background OS/AV tasks cause false positives; needs a per-candidate baseline.

### Post-exam review (forensics the live proctor misses)
- **Review host-agent logs** (process baseline-diff, overlay/top-most window appearances, all-session enumeration, display changes). Blunts **G, B, C, E**. *Trade-off:* not real-time; deterrence + invalidation after the fact.
- **Input-cadence & injected-input analysis** (R9) and **answer-latency vs item-difficulty** anomaly scan. Blunts **C (#12)**. *Trade-off:* human-paced reading of an overlay + normal clicking leaves little signal (honest limit — see §5 R6).
- **Statistical / item-exposure forensics:** response-pattern similarity to known LLM outputs; pool-rotation exposure analytics. *Trade-off:* probabilistic, not proof; best as deterrence and for serial offenders.

**Cross-cutting trade-offs:** candidate friction (reboot, screen-share, Task-Manager walk, attestation) raises abandonment and support cost; **privacy** (continuous screen capture + biometrics) raises consent/legal burden (T05 §10); **false positives** concentrate on VM-driver leftovers (T05 §5), legitimate accessibility users (T05 §8), and background host-load; **cost** is native-agent licensing + human-review hours + optional second camera.

---

## 4. VENDOR SELECTION (from T07)

**Decision rule (T07 §Bottom line):** pick a vendor with a **native, continuously-running host agent that enumerates processes + overlay/top-most windows + virtual displays throughout the exam** — that is what closes the concealed-workspace gap. Rank by documented native-agent depth.

**Primary pick — ProctorU / Meazure (Guardian Browser).** Native agent; **logs all active processes, re-checks for unpermitted software every 30s, scans 400+ apps for VMs and remote-access tools**, prompts on a second monitor, blocks concurrent login (T07 S15/S16).
- *Gap it closes:* **G (#3 pre-staged helper)** and **C (#2 local LLM)** — continuous process enumeration catches the helper **process** even when its window is hidden or capture-excluded; **B (#9 second session)** via concurrent-login block and all-process logging.
- *Residual:* VM detection is still software-heuristic (no attestation); overlay-**window** audit is less explicitly documented than Rosalyn/Constructor — pair with R4.

**Strong complement for the overlay vector — Rosalyn / StableSight.** Native desktop app; **deep process monitoring (VM → unauthorized apps)**, remote-access detection, **typing-sound correlation**, and explicit **"screen-visual detection" of another app/start menu even if it bypasses the process checks** (T07 S27/S28).
- *Gap it closes:* most directly engineered against **B (#1 concealed overlay / #6 another app)** and **C** — the screen-visual layer is the backstop when a helper evades process enumeration.

**Also on-point — Constructor / Examus.** Native secure browser with **automatic VM detection, explicit overlay blocking, and active-window-change flags** (T07 S23/S24).
- *Gap it closes:* **B/D (#1 overlay, active-window change)** — overlay blocking is rare and squarely on the top-ranked T06 attack.

**Acceptable native alternates:** Prometric ProProctor (sole-running-app lockdown, **Administrator-role machine blocked**, "unknown application running" terminated — T07 S10/S11/S12; strong posture, less public overlay-enum detail) and Respondus/Kryterion (blocks **virtual displays/virtual desktops**, remote desktop, multi-monitor — T07 S1/S17; but documented evasions: Windows Remote Assistance undetected, ALT+F4 on the monitor-blocker, registry-tuned VM — S20/S31 — and a **single-window model that does not reach a capture-excluded separate-process overlay** — S32).

**Exclude for this threat:** Honorlock (extension; host visibility only via limited-release app), Proctorio (needs Companion App even to reach mid-tier; extension-only VM detection is weak — T07 S40/S43), Examity (host integrity undocumented — S44), ProctorExam (pixels-only, structurally blind — S45). A browser tab/extension **cannot** see a VM, a remote session, or a capture-excluded overlay (T07 S29/S32).

**Where NO vendor closes the gap (state candidly):**
- **Bare-metal / Type-1 hypervisor + tuned guest (T06 #7/#8):** *no vendor documents hardware attestation (TPM/Secure Boot/measured boot) or Type-1/bare-metal hypervisor detection.* Every "VM detection" is a software heuristic (CPUID string, driver/registry/device lists, SMBIOS) defeatable by a tuned guest or a Type-1 hypervisor (T07 cross-cutting; S46/S47/S48/S51). Additionally, **timing-based** hypervisor detection — the one tell a stealthed guest struggles to hide — is academically an open problem (T06 #7 refs). **Residual must be covered by rule R7 + the §3 attestation procedure, not by any product.**
- **Capture-excluded overlay (`WDA_EXCLUDEFROMCAPTURE`) vs screen capture:** reinstating screen capture (R1) does **not** by itself catch it; only OS-level **process + top-most/overlay-window + GPU/network** enumeration does (T07 S32). So even with screen-share ON, the native overlay/z-order audit (R4) is mandatory — otherwise this is unclosable.

---

## 5. RESIDUAL RISK REGISTER

What remains exploitable **after** best-practice rules + procedures + vendor, and the only things that close it. Items requiring a change outside the current "no screen/computer monitoring" constraint are **[CONSTRAINT CHANGE]**; items needing capability no proctoring product ships are **[NEW TOOLING]**.

| ID | Residual (T06 ref) | Why it survives best practice | What actually closes it |
|----|--------------------|-------------------------------|-------------------------|
| RR1 | **Bare-metal Type-1 hypervisor + GPU passthrough, OnVUE as hardened guest; helper outside the guest** (#7/#8, A/E) | Heuristic VM detection is defeatable; helper is invisible across the VM boundary to any in-guest agent; timing detection is unsolved | **[NEW TOOLING]** Hardware attestation — TPM 2.0 + Secure Boot + **measured-boot** attestation refusing a hypervisor-present boot (T07 S47) + a **timing red-pill probe** (CPUID/RDTSC VM-exit latency vs bare-metal baseline, T06 #7). No vendor offers either |
| RR2 | **Capture-excluded local-LLM overlay** (#1/#2, B/C) | Beats screen capture (sits below it) and launch-only scans; stays on the primary display so gaze is natural | **[vendor]** native **overlay/top-most-window + continuous process** enumeration (R3+R4). If the config stays OnVUE-only, unclosable |
| RR3 | **Signed/renamed memory-resident pre-staged helper** (#3/#6, G/C) | A benign-looking, signed process can pass signature-based continuous enumeration | **[NEW TOOLING]** deny-by-default **application allow-listing** (only known-good signed processes may run) + witnessed clean reboot (§3). Very high friction |
| RR4 | **Passive UIA/AX read by a process posing as granted accessibility tech** (#5, C) | a11y is tolerated (T05 §8); the read is passive, no capture, no focus change | **[NEW TOOLING/vendor]** assistive-tech-attach detection + a **pre-approved accommodation allow-list** (R8) |
| RR5 | **Gaze-natural reading from a primary-display surface** (T06 Exec) | Camera/gaze analytics fundamentally cannot distinguish reading an overlay from reading the exam | **[CONSTRAINT CHANGE]** screen capture (R1) **+** R4; camera alone never closes it |
| RR6 | **Human-paced display-only use** (read overlay, click normally) (#12 display-only) | Input-cadence biometrics catch synthetic injection, not a human clicking | Same as RR2/RR5 — detection must be at the source surface, not at input timing |
| RR7 | **Residual value of any single breach** | Even a caught attempt may have exposed items | Item-pool rotation / exposure controls + statistical forensics (defense-in-depth, deterrence — §3) |

**The three things that genuinely close the central gap** (all beyond the current constraint, labeled): **[CONSTRAINT CHANGE]** continuous screen capture (OnVUE-capable, OFF today — T07 S35) *with* R4 overlay enumeration; **[NEW TOOLING]** hardware attestation + timing hypervisor probe (RR1); and — the ultimate backstop for the highest-stakes forms — **[CONSTRAINT CHANGE]** in-person/test-center delivery, which removes the single-machine same-OS premise entirely.

---

## 6. PRIORITIZED ROADMAP

Ranked by **impact on closing the G1 (concealed same-machine workspace) gap** vs **effort/friction**. Do the top band first.

**Band 1 — High impact, moderate effort (do first):**
1. **Adopt a native continuous-agent vendor** (ProctorU/Guardian primary; Rosalyn or Constructor as overlay-focused alternates). *Impact:* closes T06 **G, C, and #9** (pre-stage, local LLM, second session) — the core of the gap. *Effort:* procurement/migration + native-install friction. **Highest impact per unit effort.** (§4)
2. **Turn on the overlay/z-order audit + all-adapter display enumeration** that the Band-1 vendors provide (R4, R5). *Impact:* closes the **#1 overlay** attack — the highest-feasibility T06 technique. *Effort:* low marginal once on #1.
3. **Witnessed clean reboot + continuous (not launch-only) scanning** (R3, §3). *Impact:* closes **G (#3)** cheaply. *Effort:* low cost, moderate check-in friction.

**Band 2 — High impact, higher effort or a constraint change:**
4. **[CONSTRAINT CHANGE] Reinstate continuous screen capture** (R1), paired with R4. *Impact:* closes the "view" barrier for non-capture-excluded surfaces (RR5). *Effort:* config is trivial (OnVUE-capable); the cost is privacy/consent/legal and the fact that it still needs R4 for capture-excluded overlays.
5. **Assistive-tech-attach detection + IME/text-service lock + host-load/GPU telemetry** (R8, R3). *Impact:* closes **C read + local-LLM** channels (#5/#18/#2). *Effort:* moderate tooling.
6. **Random liveness + virtual-camera device-path detection** (R11). *Impact:* closes **#16**. *Effort:* low, largely off-the-shelf.

**Band 3 — High impact, very high effort (reserve for the top-stakes tier):**
7. **[NEW TOOLING + CONSTRAINT CHANGE] Hardware attestation (TPM/Secure Boot/measured boot) + timing hypervisor probe** (R7). *Impact:* the **only** control that closes **RR1 (Type-1/passthrough)**. *Effort:* very high — no vendor ships it; likely custom agent; excludes some legitimate hardware.
8. **[CONSTRAINT CHANGE] In-person/test-center delivery for the highest-stakes forms.** *Impact:* removes the single-machine premise entirely — ultimate backstop. *Effort:* highest operational cost; reserve for the forms that warrant it.

**Band 4 — Keep but don't over-invest (low marginal impact on G1):**
9. Mic voice/TTS analytics (R10) — already closes audio (#20); keep, low cost.
10. Ban managed/corporate machines, keep VM/hyperviser-feature blocks (R6) — marginal to this threat but cheap and reduces the injection/hook surface.
11. Gaze/head-pose analytics and virtual-desktop-switch hooks — weak against the primary threat (overlay keeps gaze natural; switch already caught); keep cheap, do not rely on them.

**Sequencing logic:** Bands 1 and 2 convert OnVUE's launch-only, screen-blind posture into a continuously-enumerated, overlay-aware one — this closes the *feasible-today* attacks (overlay, local LLM, pre-stage, second session) at acceptable cost. Band 3 is what remains after that: the Type-1 hypervisor and (for capture-excluded surfaces) attestation residuals, which cost real money and friction and are reserved for the stakes that justify them.

---

## Sources

Primary grounding (all 2026-11-12):
- **T05** OnVUE control surface — `/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md` (cited as T05 §n; underlying Pearson/CompTIA primary + NashTech/Open Group/SETU third-party sources enumerated in T05 Sources 1–16).
- **T06** Persona A red-team inventory — `/home/user/OneVue-Reseach/slices/T06-persona-a-redteam/output.md` (cited as T06 #n / family; VM-evasion, overlay, UIA/AX, IDD, local-LLM sources enumerated in T06 Sources 1–18).
- **T07** Vendor control inventory — `/home/user/OneVue-Reseach/slices/T07-vendor-alternatives/output.md` (cited as T07 S-n / vendor; vendor-doc and evasion-research sources enumerated in T07 Sources S1–S51).

Key source IDs load-bearing in this synthesis: T07 S15/S16 (ProctorU/Guardian continuous process enum, 30s re-check, 400+ VM scan); S27/S28 (Rosalyn deep process monitoring + screen-visual detection + typing-sound correlation); S23/S24 (Constructor overlay blocking + active-window-change + automatic VM detect); S32 (capture-excluded `WDA_EXCLUDEFROMCAPTURE` overlays only caught by OS-level process/overlay/GPU signals); S46/S47/S48/S51 (VM detection is heuristic; no vendor uses hardware attestation/Type-1 detection); S35 (OnVUE screen-capture capability, disabled in this deployment); S29 (browser-extension products structurally blind to VM/remote/overlay). T06 #1/#2/#3/#5/#7/#16/#20 and Defender's-leverage summary for the attack-to-control mapping.

No additional web research was performed; the three slices established every fact used. Claims that rest only on third-party/anecdotal slice sources (e.g., Respondus ALT+F4 / Windows Remote Assistance evasions — T07 S20; OBS virtual-cam analog — T05 §9/Source 14) carry their original third-party labeling.
