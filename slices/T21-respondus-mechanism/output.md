# T21 Respondus (LockDown Browser + Monitor) — how it works

Scope: documentation synthesis from the PUBLIC technical record only (Respondus docs/KB, patents,
independent reverse-engineering write-ups, court filings, regulator decisions, privacy disclosures,
press). No binary reverse-engineering performed here. Every claim tagged **[Doc]** (documented in a
primary/secondary source) or **[Inf]** (inferred/analytical). All access dates 2026-11-12.
Descriptive, not an evasion guide.

---

## Architecture overview

Respondus sells two coupled products:

- **LockDown Browser (LDB)** — a native, locally-installed **custom browser** that renders the LMS
  exam and "locks down" the endpoint. Sold standalone as a cheating *deterrent*. **[Doc, S2, S4]**
- **Respondus Monitor** — a cloud-based **automated webcam/AI proctoring layer** built on top of LDB.
  Adds webcam+mic recording, optional screen recording, environment check, and ML "flagging." Licensed
  separately; LDB licenses include 200 free Monitor seats. **[Doc, S4, S13, S22]**

Key architectural facts:
- LDB is built on the **Chromium Embedded Framework (CEF)** on Windows and Mac; "renders content nearly
  identically to other browsers based on CEF (e.g. Chrome, Edge)." It is a full native app, **not** a
  browser extension — a distinction Respondus markets heavily because an extension is confined by the OS
  and cannot block VMs, remote desktop, multi-monitor, or background apps. **[Doc, S3 VPAT; S2; S42]**
- It is a **native application**, not Electron per se, but CEF-based (Chromium rendering embedded in a
  native host process). **[Doc, S3]**
- On Windows the client ships a **kernel-mode Windows filesystem minifilter driver** plus user-mode exe
  and a hook DLL (see §1, §2, §5). Students/faculty have publicly described it as **"Ring-0 software."**
  **[Doc, S29 Weaver; S33 EFF]**
- Monitor is explicitly a **cloud service** ("Respondus Monitor is a cloud-based service … and
  software") hosted on **Amazon Web Services (AWS)** as sub-processor; no video stored at respondus.com.
  **[Doc, S22, S24, S25]**
- **Pearson relevance:** Respondus publishes **Pearson-specific Monitor Terms of Use**
  (tou-monitor-pearson-student / -instructor), and markets LDB as "licensed by dozens of the largest
  publishers, **testing centers, certification organizations**, and assessment platforms." So Respondus
  is a *peer/competitor to OnVUE that Pearson itself also integrates with.* **[Doc, S21, S42]**

---

## 1. Architecture: client type, install, privilege, OS support, environment locking

**Client type** — Custom CEF/Chromium native browser; students do NOT need Chrome installed and an
existing Chrome install is unaffected. **[Doc, S2, S3]**

**OS support (current, 2026):** **[Doc, S1 computer-requirements; S20 Monitor requirements]**
- Windows 11 and 10 (build 19045+). x86-64 and ARM64 (via x86 emulation, incl. Copilot+ PC). **Windows
  "S mode" and "SE" are NOT supported** and LDB is deliberately not on the Microsoft Store.
- macOS 12+ (ARM + Intel).
- ChromeOS / managed Chromebook (LTS channel minimum). Note: some campus labs report Chromebook not
  supported for their config. **[Doc, S1; GSU note]**
- iPadOS 15+ (iPad only; requires compatible LMS integration; not Sakai). App Store app.
- Footprint tiny: ~75–300 MB disk, 2 GB RAM (4 GB when webcam/Monitor used).

**Install & privilege:**
- Installation requires **local administrator privileges** (documented repeatedly across institutions).
  Students install an **institution-unique** build (per-institution installer URL/ID). **[Doc, S44; many
  campus KBs]**
- Windows runtime components (from independent RE, LDB 2.1.5.01): **[Doc, S29 Weaver]**
  - `LockDownBrowser.exe` — primary user-mode app.
  - `LockDownBrowser.dll` — UI, **input, and hook-related** behavior (keyboard/mouse hooks).
  - `LockDownService215.sys` — **Windows filesystem minifilter driver** that registers kernel callbacks
    for **process creation, thread creation, and image loading**, with user-mode↔kernel-mode comms.
  - → **[Inf]** This is how LDB can see and kill prohibited processes and resist tampering at a level an
    ordinary browser cannot. Privilege at runtime is effectively **kernel-level (Ring-0)** on Windows.
- LDB "does not run in any capacity when closed" and does not install persistent tracking; services it
  deactivates (e.g. Teams, Discord) are restored on reboot. **[Doc, S43 UDM CETL]**

**How it locks the environment** (the canonical list, consistent across Respondus + institutions):
**[Doc, S2, S4, S10, S11, S12]**
- Assessment shown **full-screen; cannot be minimized, resized, or exited** until submitted.
- **Modified toolbar** — only Forward, Back, Refresh, Stop. URL bar removed; users cannot type URLs.
- **Printing disabled; screen capture / print-screen disabled.**
- **Copy and paste to/from the assessment prevented** (verify note in §6).
- **Right-click menus, function keys, keyboard shortcuts, and task-switching disabled.**
- **Start button, system tray, and menu bars removed** (Windows). "Source code" view disabled; pages
  not cached/stored in history.
- **Blocks access to other applications**, including messaging, screen-sharing, **virtual machines, and
  remote desktops**. External links open in a locked secondary window that can't browse beyond.
- At launch, LDB **kills / refuses to start alongside hundreds of named applications** — screen capture,
  messaging, screen-sharing, network monitors ("You must close the following program…"). Can auto-close
  them. **[Doc, S9]**
- Marketing claim of scope: "blocks hundreds of common and advanced methods of digital cheating, such as
  the use of Virtual Machines, remote desktops, screensharing, instant messaging, screen recording,
  keystroke combinations, launching applications with timers/alerts, browser cache exploits." **[Doc, S4]**

---

## 2. Detection mechanisms (how each works, to the extent public)

### 2a. Virtual-machine detection ("Yikes! A VM has been detected" / "Virtual Moose")
- **[Doc, S5, S6]** On a detected VM, LDB shows a VM warning (student-facing "Virtual Moose" cartoon),
  states **"Your instructor is being notified and your exam session will be terminated,"** and refuses to
  run. Respondus's doc enumerates what counts: VM host apps (VMware, VirtualBox), thin-app virtualization
  (VMware ThinApp, Microsoft App-V), Windows emulators (WINE, CodeWeavers CrossOver), and other
  virtualization ("Virtual Hard Drives/vHDs," **"Virtual Displays"/"Virtual Desktops"**).
- **Mechanism (documented via independent RE, LDB 2.1.5.01): [Doc, S29 Weaver]**
  - **"There was no single VM check."** Environment classification is distributed across: hardware
    identity, **Windows device inventory (SetupAPI)**, **BIOS and registry values**, system-manufacturer
    data, CPU identity, **process/module paths**, sessions, runtime artifacts, and kernel-visible activity.
  - A recovered **26-entry device-prefix table** matches indicators for **Parallels, VMware, VirtualBox,
    QEMU, Xen, VirtIO, AWS, and Wine**, plus checks on device descriptions, friendly names, and **COM
    device names**.
  - Internal state flag `rldbvm`: `=1` broad hardware/platform classification; `=2` written from username,
    process/module path, and periodically-refreshed runtime-artifact paths; `=3` an **expected-device
    consistency failure involving display-like hardware and a PCI Express root** (→ **[Inf]** this is the
    virtual-display / missing-GPU heuristic). A separate `rldbdetect` flag records broader hack/tamper
    conditions.
- **Corroboration that detection is signature/driver-based (hence heuristic): [Doc, S5, S7-TAMU]**
  Respondus's own KB says false positives arise from (a) **antivirus that spins up small sandbox VMs**,
  and (b) **leftover VM device drivers** from a prior VM install or a **machine imaged from a VM image** —
  i.e. detection keys on **device-driver/registry/BIOS artifacts**, not on true hypervisor presence.
- **[Inf]** Because detection is artifact/signature enumeration in user+kernel mode, it is structurally a
  heuristic: it catches off-the-shelf **Type-2** hypervisors and their drivers, but its completeness
  depends on the signature list. A **Type-1 (bare-metal) hypervisor** presenting clean/native-looking
  device identity, or a guest with VM artifacts scrubbed, is the documented weak point — though Weaver
  explicitly produced **no working bypass** and could not confirm "the complete policy transition from
  classification to enforcement." Mark any "defeat" as unproven in the public record. (Cross-ref T14/T16.)

### 2b. Virtual-display / second-monitor detection
- **[Doc, S7, S8]** LDB is "intentionally designed not to support multiple monitors"; it **blacks out /
  disables the secondary display** and opens only on the primary. Students are told to disconnect extra
  monitors first. "Virtual Displays/Virtual Desktops" are lumped into the VM-detection path (§2a).
- **[Inf]** Mechanism = enumerate attached displays (and the `rldbvm=3` display/PCIe-root consistency
  check); extended desktop and screencast "virtual displays" register as added display devices or as the
  virtual-display drivers in the signature table.

### 2c. Prohibited-process / running-app detection
- **[Doc, S9, S12]** At launch and continuously, LDB checks the running-process list against a large
  blocklist (screen capture, messaging, screen-sharing, network monitors, recorders, apps with
  timers/alerts). It can terminate them automatically.
- **[Inf]** Enforced via the kernel minifilter's **process-creation / image-load callbacks** plus
  user-mode ToolHelp/PSAPI enumeration (Weaver confirms those APIs and the driver callbacks). This lets
  LDB block an app **started before** the browser and intercept one launched during the exam.
- **[Doc, S45]** Webcam-stealing identity managers (Intel/McAfee TrueKey `McAfee.TrueKey.Service.exe`,
  SensibleVision FastAccess) are singled out as interfering — confirming process-name matching.

### 2d. Screen-capture / remote-access blocking
- **[Doc, S4, S42]** Print-screen and screen-capture are disabled; screen-recording, screen-sharing, and
  remote-desktop apps are in the block/terminate set. Respondus's comparison paper lists as exploits
  "possible with a locked-browser extension but NOT with LockDown Browser": **VM/Sandbox/Safe-Mode,
  remote desktop & screenshare, extended-desktop/multi-monitor/screencast, background IM/video recording,
  task-switching hacks.**

### 2e. Other
- **[Doc, S2, S11]** Disabled: function keys, keyboard-shortcut/keystroke combinations, right-click,
  Start/tray/menu bar, URL entry, view-source, caching/history.
- **[Doc, S43]** Any head/eye tracking is **not** done on the device — it is done **by Respondus on their
  servers, only while Monitor is running** (see §3). LDB alone does no webcam work.

---

## 3. Respondus Monitor — the webcam AI layer

**Startup sequence (pre-exam):** Terms of Use → **Webcam & Microphone Check** → optional **Environment
Check** (video pan of room) → **Facial Detection Check** (look into camera) → optional **Photo ID**
capture → optional "Show Your Work." Instructor chooses which steps are required. **[Doc, S13, S19, S30]**

**What it records: [Doc, S19, S25]** webcam video + audio of the student; optionally a **screen
recording** (added feature — lets instructors see on-screen activity alongside the student); thumbnails;
photo ID; log files of LDB↔LMS interaction; "education data" (exam duration, time per question, flags).
Monitor can also run **instructor Live Proctoring** (real-time remote observation). **[Doc, S43]**

**The computer-vision engine — "Respondus Vision": [Doc, S15]**
- Respondus trains its **own** CV/ML models on proctoring data rather than using off-the-shelf models,
  arguing this suits indoor lighting, webcam-quality video, and keeps models **small enough for low
  bandwidth and underpowered computers**. (→ **[Inf]** relevant to low-compute test-center hardware.)
- **Division of labor (documented): [Doc, S19, S26 patent, S43]** A **parallel facial-detection system
  runs real-time on the client** solely to *alert the student* ("face can't be detected," "tilt camera
  down," "fix lighting"). The **substantive analysis and flagging happen server-side** after upload;
  "head and eye tracking is done by Respondus on their servers." Per patent, the client face-detector is
  an optional instructor setting and warns ~30–60 s after signal loss.

**Flags (events that "might" indicate a problem) and Milestones: [Doc, S13, S18]**
- Flags: **Missing from Frame**; **Different person in Frame**; **Multiple persons in Frame**;
  **Partially Missing from Frame** (Strict sensitivity only); **Internet interruption**; **Failed Facial
  Detection Check** (pre-exam); **Student alerted to failed face detection**; optional **Handheld Device**
  (a calculator/phone detected). Per patent also: **webcam covered** (when/how long), **audio-level
  changes**, **overall motion** of subject and background.
- Milestones (neutral events): Pre-Exam, Exam Start, End of Exam, question answered, webcam disconnected.
- **MISSING rule (from patent US11645935B1): [Doc, S26]** face not detected for ~**15–25 s with no
  motion** → MISSING event. Short absences/hidden-webcam get different weight.

**How flags become a ranking — "Review Priority": [Doc, S13, S17, S18, S26]**
- Each session gets an overall **Review Priority** = **Low / Medium / High**, shown with a
  green→red bar, ranking which sessions warrant closer human review.
- **Computation (patent):** each flagged event is assigned a **point value**, then **adjusted by a
  weight derived from data other than the mere occurrence** (e.g. video quality, baseline across all
  analyzed videos, duration, context). Low-quality video can itself be surfaced so poor recordings aren't
  silently trusted. **[Doc, S26, S27, S28]**
- **Sensitivity levels — Strict / Standard / Relaxed** change which events flag and recompute Review
  Priority. Crucially, the instructor can **change sensitivity AFTER the exam** and the scores/rankings
  update instantly (analysis data is retained, not just a verdict). **[Doc, S14]**
- Results surface in the LMS **LockDown Browser Dashboard → Class Results**, ~**10–15 min** after
  submission, with a synchronized **Timeline** (red flags / blue milestones), video playback,
  thumbnails; instructors can mark reviewed / add comments. **[Doc, S17, S18]**

**Human-review model (important): [Doc, S17, S19]**
- "**Respondus staff do not watch or review student videos.**" Only instructors/TAs/admins **enrolled in
  that course** can view, via LMS authentication.
- Respondus states its CV "**exclusively generates event flags and alerts and does not make any
  determinations, automated decisions, or recommendations**." The **instructor is the sole
  decision-maker**; "flags aren't cheating." This posture is also its legal/ethical defense. **[Doc, S19]**
- **[Inf]** Net: Monitor is a *triage/deterrence* system — it compresses hours of video into a ranked
  queue + jump-to-flag timeline. Detection quality rides entirely on face-in-frame computer vision.

---

## 4. Data flow & telemetry

- **Transport/storage:** webcam video is **streamed with per-session encryption** to a server that
  "only accepts streaming connections"; the **video itself is stored on a separate AWS server — no video
  stored at respondus.com.** Data **encrypted at rest**. **AWS is the named sub-processor.** **[Doc, S24,
  S25]**
- **Access control:** instructor must authenticate **through the institution's LMS**; videos reached via
  the Respondus dashboard using an **institution-unique secret key**; **video URLs are one-time use** and
  break if copied. **[Doc, S24; S-student-terms]**
- **Pre-exam check data:** the **webcam & microphone check** video/audio is held in **temporary cache and
  auto-deleted in ~1 hour.** **[Doc, S21]**
- **Facial-feature template (the biometric used to decide "same person"):** a **temporary template** is
  built during processing; it is **not saved to the device, DB, or cloud storage** and is **cleared from
  server memory after processing — no later than 2 days** post-exam. **[Doc, S21, S49]**
- **Retention (operator-configurable): [Doc, S22, S23]**
  - **Media files (recordings, thumbnails): 30 days to 5 years** (Pearson student terms default = 2
    years; some institution terms = 1 year + up to 4 more; support says videos stored **5 years** total).
  - **PII: 18 months to 5 years.** Course must remain active/restored for links to resolve.
  - Institution is the data **controller**; Respondus is the **processor**. Deletion at end of retention,
    except legal holds / backups. **[Doc, S25]**
- **Third parties / processors:** **AWS** (hosting + video + biometric transit); **customer-support
  providers** for Help Center data (retained ≤1 year); **de-identified recordings may go to contracted
  researchers/biometric experts** (name/course/institution/ID stripped, under confidentiality). **[Doc,
  S19, S25, S31]**
- **Compliance posture:** FERPA, GDPR, CCPA, EDUCAUSE **HECVAT**, **SOC 2 Type 1**, PCI scanning,
  third-party penetration testing, monthly vulnerability scanning; AWS-certified engineers. **[Doc, S24]**
- **Data categories collected:** identifiers (LMS id, name), **biometric** (facial-geometry scans; a
  "voiceprint" per litigation), **sensory** (video/audio/photos), internet/network activity, and
  **education data** (exam duration, time-per-question, violation flags). **[Doc, S19, S25, S30]**

---

## 5. Known security/privacy research & incidents

**Independent reverse-engineering (headline technical analysis):**
- **Wyatt Weaver, "Respondus LockDown Browser Research" (Sept 2026)** — concluded RE study of LDB
  2.1.5.01 using Ghidra + Frida. **Conclusions level:** mapped the 3-component architecture (exe + hook
  DLL + **kernel minifilter driver** with process/thread/image-load callbacks); showed **VM detection is
  distributed artifact-enumeration** (26 device prefixes + BIOS/registry/CPU/COM checks; `rldbvm`
  states); documented heavy **anti-analysis obfuscation** (overlapping instruction streams, RET gadgets
  inside other bytes, synthetic return stacks, opaque predicates, computed dispatch) and
  **instrumentation-sensitive** behavior (Frida attach kills the process). **Explicitly produced NO
  bypass** and did **not** confirm how classification maps to enforcement. Builds on earlier
  offset-mapping by `arcticdev00`. **[Doc, S29 + GitHub UwUGreed/...]**
- **[Inf]** Takeaway for an evaluator: LDB's defenses are real and kernel-deep, but its VM/environment
  model is **signature-driven and obfuscation-protected**, i.e. an arms race, not a cryptographic
  guarantee. Public record shows capable researchers mapping it but not publishing a working defeat.

**No formally assigned CVE** for LDB/Monitor was found in this search. Student petitions (e.g. **UMass
Lowell**) argued the **"Ring-0 software … creates massive security vulnerabilities and attack vectors,"**
but that is advocacy, not a disclosed exploit. **[Doc, S33 EFF]** (Mark "CVE: none found" as
unverified-absence, not proof of none.)

**Privacy litigation & regulatory actions (biometric is the live risk):**
- **Bridges v. Respondus** (N.D. Ill., consol. w/ Patterson, Wu, Veiga), **Illinois BIPA.** Alleges
  Monitor captured **facial-geometry scans and a voiceprint** without informed written consent / public
  retention schedule. **23 Mar 2022: Judge Pallmeyer refused to enforce the Washington choice-of-law
  clause and declined to dismiss** the facial-geometry claims. **[Doc, S30, S31, S32]**
- **Italy (Garante):** **Bocconi University fined €200,000 (Sept 2021)** for using Respondus — insufficient
  notice and **processing biometric data without a legal basis**; decision implicates other EU users.
  **[Doc, S37 EDRi]**
- **Ontario IPC privacy complaint report (2024, McMaster):** found **LDB** collection "necessary and
  proportional," and **Monitor** lawfully authorized for proctoring, but criticized transparency
  (students must consult scattered pages). A nuanced *pro-necessity* data point. **[Doc, S38]**
- **EPIC FTC complaint (Dec 2020)** named Respondus (+ ProctorU, Proctorio, Examity, Honorlock) as unfair
  surveillance of biometric/personal data. **[Doc, S35]**

**Advocacy / press / institutional pushback:**
- **EFF**: Kelley & Oliver, *"Proctoring Apps Subject Students to Unnecessary Surveillance"* (Aug 2020);
  Kelley, *"Students Are Pushing Back…"* (Sep 2020) — flags the Ring-0 access specifically. **[Doc, S33,
  S34]**
- **Reuters** (Nov 2020) "global student revolt"; **Washington Post** (Nov 2020) "students revolt."
  Industry rebuttal: tools "do not directly identify cheaters, but simply flag suspicious behavior."
  **[Doc, S36, S40 fn1]**
- **University of Guelph** withdrew Monitor (PACAR anti-racism committee: facial detection "perpetuates
  inequality"; reports of mis-flagging students of colour, trans students, disabled students) — but
  **kept LockDown Browser.** **University of Victoria** does **not** approve Monitor/Proctorio but
  **supports LDB.** A recurring institutional split: *the lockdown browser is tolerated; the webcam AI is
  the lightning rod.* **[Doc, S39, S40]**
- Academic: **Shioji et al., IEEE S&P 2025** — institutions perceive lockdown browsers as collecting far
  less data than webcam AI, but notes proctoring software generally is "highly privileged" and performs
  "sweeping surveillance." **[Doc, S41]**
- **Respondus's own fairness study** (self-published, 2022/2025) uses Meta's **Casual Conversations**
  dataset and states it **prioritizes minimizing false positives over maximizing true positives** across
  age/gender/skin-tone. Vendor-authored, not independent. **[Doc, S20]**

**Patents (primary, Respondus-assigned):** family by inventor **David Smetters** — **US11645935B1**
(2023), **US11915616B2** (2024), **US12154457B2** (2024), apps US2023/0245581, US2024/0169855 — "Systems
and methods for assessing data collected by automated proctoring": point-value+weighted risk scoring,
MISSING timing, webcam-covered/audio/motion signals, and the **client-side real-time face detector that
only alerts** the student. **[Doc, S26, S27, S28]** (Competitor **Constructor Technology** app
US2026/0154011 enumerates the same threat surface — **virtual camera, VM, remote desktop, external
display, screen-share, muted mic** — confirming these are the industry-recognized blind spots. **[Doc,
S47]**)

---

## 6. Documented limitations

**What it structurally CANNOT see: [Inf unless noted]**
- **Anything off-camera / on a second physical device.** Monitor sees only the webcam cone; a phone,
  tablet, or notes outside frame are invisible unless they enter the frame (Handheld-Device flag is
  optional and frame-bound). Loyola's own guidance: **"Monitor will not flag audio such as the student
  asking somebody nearby for help."** **[Doc, S46]**
- **Behavior under the lockdown that doesn't reach the webcam or a prohibited-process signature.**
  Respondus itself admits (re locked-browser extensions, but the logic generalizes): **"Even if the
  computer screen is recorded, many exploits won't appear in the screen recording video. For example, a
  video capture application started before the exam can run quietly in the background."** **[Doc, S42]**
- **A clean host beneath the client.** LDB inspects the OS it runs in; it cannot introspect a Type-1
  hypervisor host presenting native-looking hardware, nor content running on a device it was never
  installed on. (Cross-ref T14/T16.) **[Inf]**

**Copy/paste — claim vs reality:**
- **Documented behavior:** LDB **prevents copying and pasting anything to or from an assessment** and
  disables the clipboard keystroke combinations and right-click. This is enforced **within the LDB
  window**. **[Doc, S2, S4, S10]**
- **[Inf / verify boundary]** The block is on the exam surface and the OS clipboard shortcuts *inside
  LDB*; it does not imply the host OS clipboard is wiped for processes outside LDB's reach. Cross-ref
  **T17 (clipboard)** for whether clipboard transfer is observable across a VM boundary. Instructors can
  optionally **re-enable Print and a Calculator** in the toolbar, and **allow a whitelist of web
  domains** during the exam — each an operator-chosen hole. **[Doc, S12]**

**VM detection is heuristic/defeatable in principle:**
- Confirmed signature/artifact-based (§2a, §5): triggers on **leftover VM drivers, VM-imaged machines,
  and AV sandboxes** (false positives), which means it reasons from device/registry/BIOS artifacts, not
  from a hypervisor-proof root of trust. **[Doc, S5, S7]**
- Public RE maps the signature tables and obfuscation but yields **no published working bypass**; the
  classification→enforcement step is unproven in public. So "defeatable" is **plausible but
  undemonstrated** in the record — treat claims of an easy defeat as unverified. **[Doc, S29]**

**Other operational limits:** no Linux support; Windows S-mode/SE excluded; needs admin to install
(blocks some locked-down/work machines); webcam false-positives from AV, identity-manager webcam apps,
poor lighting, caps, glasses, backlight, looking away; internet drop can freeze the exam. **[Doc, S1,
S13, S45, S-webcam]**

---

## 7. Relevance to the one-machine concealed-workspace threat (vs. OnVUE)

User's threat (Goals C1–C3): run the exam in one window **and** research answers (LLM/answer bank) on the
**same single physical machine**, undetected, in a deployment where OnVUE has **no screen/computer
monitoring — only a front 360° camera + the proctoring client's telemetry.**

How Respondus's model compares:

- **Same two-layer architecture as OnVUE** (secure lockdown client + webcam/AI proctoring), but
  Respondus's **LockDown Browser is a more aggressive endpoint enforcer** than a browser-only lockdown:
  it ships a **kernel minifilter driver**, **kills prohibited processes at launch and during the exam**,
  **blocks VMs/remote-desktop/screen-share**, and **disables secondary monitors and task-switching**.
  These are exactly the controls that **close the "concealed second workspace on the same machine" gap**
  that the user's OnVUE deployment leaves open by not monitoring the computer. **[Doc, S4, S9, S29]**
- **Concrete gap-closures vs the user's OnVUE posture:** **[Inf, grounded in §1–§2]**
  1. **Concealed second browser/app on the same box** → LDB's full-screen lock + no task-switch + process
     kill-list directly targets this (the single biggest hole in the user's model).
  2. **Type-2 VM to sandbox a research workspace** → LDB's VM warning + session termination (OnVUE in the
     described deployment is not watching the OS at all).
  3. **Second monitor / extended desktop / screencast** → LDB blacks out secondary displays.
  4. **Screen recorders / remote-assist tools pre-started in background** → blocked/terminated by name.
- **Gaps Respondus does NOT close (shared with OnVUE):** **[Inf, grounded in §6]**
  - A **Type-1 bare-metal hypervisor** that presents clean hardware and runs LDB in a guest while the LLM
    runs on the host is the structural blind spot — LDB's VM detection is signature-based, and it cannot
    see a host it was never installed in. (Weaver: no published defeat, but detection is heuristic.)
  - **Off-camera second device / confederate / paper** — out of the webcam's cone; Monitor adds a camera
    but not omniscience (and the user's deployment already has a 360° camera, which is Monitor's job).
  - **Audio collusion** is not reliably flagged.
- **Net assessment for the evaluator:** **[Inf]** For the *one-machine software-concealment* threat
  specifically, **Respondus LockDown Browser's endpoint lockdown is materially stronger than an
  OnVUE-style client that is not monitoring the computer** — it converts "the computer is unmonitored"
  into "the computer is actively locked, VM-screened, and process-controlled." But it **does not add a
  hardware root of trust**, so the **residual risk collapses onto the same Type-1-hypervisor / clean-host
  scenario** that is the crux of T14/T16. Its webcam-AI layer (Monitor) is a *deterrent/triage* tool with
  the same frame-bound limits as OnVUE's camera — plus heavier biometric/privacy and bias liabilities
  (§5), which matter for a certification owner's vendor selection. On **center-owned, low-compute
  machines** (user's other deployment), LDB's kernel-level process/VM/monitor controls are a strong fit;
  the owner's own firmware/boot/USB hardening remains the thing that closes the Type-1 residual.

---

## Confirmed vs opaque

**Confirmed (documented):**
- CEF/Chromium native client; Win/Mac/iPad/Chromebook support; admin install; Windows kernel minifilter
  driver + user-mode exe + hook DLL. [S1,S3,S29,S44]
- Full lockdown feature set (print/copy/paste/right-click/function-keys/task-switch/screen-capture/
  multi-monitor/VM/remote-desktop/process-kill). [S2,S4,S10,S12]
- VM detection is distributed artifact/signature enumeration; VM warning terminates session + notifies
  instructor; secondary displays blacked out. [S5,S6,S7,S29]
- Monitor pipeline: pre-exam checks; client-side alert-only face detector; server-side CV ("Respondus
  Vision") flagging; flag set; MISSING ~15–25 s; Review Priority Low/Med/High via weighted point scores;
  Strict/Standard/Relaxed sensitivity recomputable post-exam; ~10–15 min processing; LMS dashboard +
  timeline; instructor-only viewing; Respondus staff don't watch. [S13–S19,S26]
- Data flow: AWS storage, per-session-encrypted stream, one-time URLs, encrypted at rest; facial template
  cleared ≤2 days; webcam-check video deleted ~1 hr; media retention 30 d–5 y, PII 18 mo–5 y; researcher
  sharing of de-identified video. [S21–S25]
- Biometric litigation (BIPA), Italian Garante fine, EFF/EPIC/press criticism, Guelph withdrawal; Smetters
  patent family. [S26–S41]

**Opaque / unverified:**
- The exact **classification→enforcement** logic and which `rldbvm`/`rldbdetect` states hard-terminate vs
  log (Weaver left open). [S29]
- The **complete VM/environment signature list** and the driver's IOCTL/minifilter protocol. [S29]
- **No CVE found** — absence not proven.
- Precise **server-side model internals** (architecture, thresholds) of Respondus Vision — proprietary.
- Whether current builds detect **virtual cameras / fed video loops** beyond prompting the student to pick
  the physical camera — not documented by Respondus (competitor patent treats virtual-camera as an open
  problem). [S47, S-webcam]
- iPad/Chromebook internal lockdown mechanics (OS-sandbox-bound; less privileged than Windows kernel
  driver) — inferred, not documented in detail.
- Exact clipboard handling across a VM boundary — deferred to T17.

---

## Sources (accessed 2026-11-12)

S1. Respondus Support — Computer requirements for LockDown Browser — https://support.respondus.com/hc/en-us/articles/4409607170459
S2. Respondus — LockDown Browser Resources (Chromium; feature list) — https://web.respondus.com/he/lockdownbrowser/resources
S3. Respondus — VPAT 2.5, Oct 2024 (CEF statement) — https://web.respondus.com/wp-content/uploads/2025/07/Respondus_VPAT2.5_Oct2024.pdf
S4. Respondus — LockDown Browser product page (blocks VMs/remote desktop/etc.) — https://web.respondus.com/he/lockdownbrowser
S5. Respondus Support — VM warning (VMware/VirtualBox/ThinApp/App-V/WINE/CrossOver/vHD/Virtual Displays) — https://support.respondus.com/hc/en-us/articles/4409604116123
S6. CSUDH — "Yikes! A VM has been detected" (Virtual Moose; instructor notified, session terminated) — https://at.csudh.edu/canvas/knowledge-base/rldb-troubleshooting/virtual-moose-yikes-a-vm-has-been-detected
S7. Texas A&M — "A VM has been detected" (AV sandboxes/leftover drivers false positives) — https://service.tamu.edu/TDClient/36/Portal/KB/Article/637
S8. GVSU — dual monitors: LDB blacks out/disables second screen, by design — https://services.gvsu.edu/TDClient/60/Portal/KB/Article/25297
S9. Respondus Support — "You must close the following program before starting the browser" — https://support.respondus.com/hc/en-us/articles/4409595453339
S10. Delta College — full lockdown list (VMs, remote desktops, screen-sharing, task-switch) — https://students.delta.edu/online-learning/respondus.html
S11. UIUC — LockDown Browser disabled controls/toolbar/screen-grab/function keys — https://help.uillinois.edu/TDClient/38/uis/KB/ArticleDet?ID=2575
S12. Respondus — HE Instructor Quick Start Guide 2026 (hundreds of apps blocked; screen recording; optional Calculator/Print/domain allow) — https://web.respondus.com/wp-content/uploads/2026/05/HE-Instructor-QSG.pdf
S13. Respondus — Understanding Proctoring Results (flags, milestones, facial detection) — https://web.respondus.com/monitor-review/
S14. Respondus — Sensitivity Levels (Strict/Standard/Relaxed; post-exam recompute) — https://web.respondus.com/sensitivity-levels-in-respondus-monitor/
S15. Respondus — Why We Do Our Own Machine Learning (Respondus Vision; small models) — https://web.respondus.com/why-we-do-our-own-machine-learning/
S16. Respondus — Enhanced Flagging (resolution, tilt-down, bad-lighting prompts) — https://web.respondus.com/enhanced-flagging-respondus-monitor/
S17. Respondus — Rapid Rollout / Instructors (Review Priority; "Respondus staff do not watch videos"; 10–15 min) — https://web.respondus.com/rapid-rollout-instructors
S18. UMBC — How to view Monitor videos (Review Priority Low/Med/High; Timeline) — https://umbc.atlassian.net/wiki/spaces/faq/pages/30755898
S19. Respondus — Additional Privacy Information: Monitor (CV generates flags only, no decisions; purposes) — https://web.respondus.com/privacy-center-overview/privacy-additional-monitor
S20. Respondus — Algorithm Fairness study (Casual Conversations; minimize false positives) — https://web.respondus.com/wp-content/uploads/2025/06/Algorithm_Fairness_06.25.2025.pdf
S21. Respondus — Monitor Terms of Use (Pearson – Student): template cleared ≤2 days; webcam-check deleted ~1 hr; reverse-engineering prohibited — https://web.respondus.com/tou-monitor-pearson-student
S22. Respondus — Monitor Institution Terms of Use (cloud service; media 30 d–5 y, PII 18 mo–5 y) — https://web.respondus.com/tou-monitor-admin
S23. Respondus Support — How long is video kept? (5 years) — https://support.respondus.com/hc/en-us/articles/4409595425307
S24. GVSU — Student Privacy & Security (AWS storage separate from respondus.com; per-session encryption; one-time; compliance list) — http://services.gvsu.edu/TDClient/60/Portal/KB/PrintArticle?ID=18017
S25. Respondus — Data Processing Agreement (US), Aug 2023 (AWS sub-processor; encrypted at rest; education-data categories) — https://web.respondus.com/wp-content/uploads/2023/08/Data-Processing-Agreement-US-DPAUS-082523.pdf
S26. Patent US11645935B1 — Smetters/Respondus (risk assessment; MISSING 15–25 s; client face detector) — https://patents.google.com/patent/US11645935B1/en
S27. Patent US11915616B2 — Smetters/Respondus (point value + weight) — https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/11915616
S28. Patent US12154457B2 — Smetters/Respondus (biometric data; low-quality video handling) — https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/12154457
S29. Wyatt Weaver — Respondus LockDown Browser 2.1.5.01 RE study (components; distributed VM detection; rldbvm; minifilter; obfuscation; no bypass) — https://wyattweaver.org/projects/respondus-lockdown-browser-research (repo: https://github.com/UwUGreed/respondus-lockdown-browser-research-2.1.5.01)
S30. Bridges v. Respondus — N.D. Ill. opinion (facial geometry + voiceprint; LDB description) — https://law.justia.com/cases/federal/district-courts/illinois/ilndce/1:2021cv01785/397487/43
S31. ClassAction.org — Respondus BIPA class action (facial/keystroke/eye/gaze data; "spyware") — https://www.classaction.org/news/respondus-hit-with-privacy-class-action-over-collection-of-illinois-test-takers-facial-features-voices
S32. Legal Newsline — Judge Pallmeyer refuses to dismiss BIPA / voids WA choice-of-law (Mar 2022) — https://www.legalnewsline.com/cook-county-record/judge-says-online-college-test-proctor-respondus-cant-ditch-il-biometrics-class-actions-over-student/article_21599c41-c773-51b4-9150-91ce70676bc2.html
S33. EFF — Students Are Pushing Back Against Proctoring Surveillance Apps (Ring-0; UMass Lowell) — https://www.eff.org/deeplinks/2020/09/students-are-pushing-back-against-proctoring-surveillance-apps
S34. EFF — Proctoring Apps Subject Students to Unnecessary Surveillance (Aug 2020) — https://www.eff.org/deeplinks/2020/08/proctoring-apps-subject-students-unnecessary-surveillance
S35. EPIC — FTC complaint re online test proctoring companies (Dec 2020) — https://epic.org/wp-content/uploads/privacy/dccppa/online-test-proctoring/EPIC-complaint-in-re-online-test-proctoring-companies-12-09-20.pdf
S36. Reuters — 'Unfair surveillance'? Online exam software sparks global student revolt (Nov 2020) — https://www.reuters.com/world/unfair-surveillance-online-exam-software-sparks-global-student-revolt--trfn-2020-11-10
S37. EDRi — Italian DPA fines Bocconi €200,000 for Respondus (biometric, no legal basis) — https://edri.org/our-work/no-biometric-surveillance-for-italian-students-during-exams
S38. Ontario IPC — Privacy Complaint Report (McMaster; LDB necessary/proportional; Monitor authorized) — https://decisions.ipc.on.ca/ipc-cipvp/privacy/en/521580/1/document.do
S39. Guelph Today — U of G steps away from Respondus Monitor (keeps LockDown Browser) — https://www.guelphtoday.com/local-news/u-of-g-to-step-away-from-use-of-controversial-web-cam-watching-6223858
S40. EDUCAUSE Review — Proctoring Software in Higher Ed: Prevalence and Patterns (Respondus 65%; U Victoria stance) — https://er.educause.edu/articles/2021/2/proctoring-software-in-higher-ed-prevalence-and-patterns
S41. Shioji et al. — "It's been lovely watching you," IEEE S&P 2025 (institutional decision-making; lockdown vs AI data perceptions) — https://www.lucysimko.com/publications/shioji-proctoring-ieee-sp-25.pdf
S42. Respondus — LockDown Browser vs "Locked Browser Extensions" (native-app rationale; exploits blocked; background-capture admission) — https://web.respondus.com/wp-content/uploads/2024/11/LDB_vs_Extensions_11.09.2024.pdf
S43. Univ. of Detroit Mercy CETL — The Truth about LockDown Browser (head/eye tracking server-side; deactivates Teams/Discord; screen recording; Live Proctoring) — https://cetl.udmercy.edu/the-truth-about-lockdown-browser-the-inside-scoop
S44. Respondus Support — How do I install LockDown Browser? (local admin required) — https://support.respondus.com/hc/en-us/articles/4409607231003 (corroborated: South Plains, UWyo, Loyola, SC4, AUD, GSU KBs)
S45. Respondus Support — Identity-management webcam facial-recognition programs interfere (process-name matching) — https://support.respondus.com/hc/en-us/articles/4409604259611
S46. Loyola University New Orleans — Lockdown/Monitor (Monitor will not flag audio such as asking someone nearby) — https://ctrl.loyno.edu/lockdown
S47. Patent app US2026/0154011A1 — Constructor Technology (competitor; enumerates virtual camera/VM/remote desktop/external display as proctoring problems) — https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/20260154011
S48. Respondus Monitor Terms of Use (Student, general) — webcam/mic check storage; facial template handling — https://web.respondus.com/tou-monitor-student/
