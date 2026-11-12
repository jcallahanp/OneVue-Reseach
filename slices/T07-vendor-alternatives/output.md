# T07 Vendor Control Inventory

Scope: Which proctoring vendors have technical anti-cheat controls that bear on ONE threat —
a candidate running the exam AND an LLM/answer tool SIMULTANEOUSLY on ONE physical machine
(no external hardware, no helper, no second device). For this threat, **only software host-integrity
controls close the gap**; a second human camera does not. Capabilities documented only; no bypasses proposed.
Access date for all sources: **2026-11-12**.

Key architecture distinction used throughout:
- **Native host agent** (custom secure browser or desktop app = direct OS access: can enumerate processes, drivers, displays, overlay windows).
- **Browser extension / Electron tab** (sandboxed: sees its own tab/window, camera, focus, display count via permission — **cannot** see a VM, a remote-desktop session, or a capture-excluded overlay that lives *below* the browser) [S29][S30].

---

## Comparison matrix

Legend: Y=documented control / N=absent or unsupported / ~=partial, conditional, or heuristic-only / ?=undocumented (unknown).
"Host-agent level" = NATIVE (secure browser/desktop app) · EXT (browser extension) · EXT+APP (extension plus optional desktop companion).

| Vendor | VM-detect | Hypervisor/attestation (TPM/SecureBoot/Type-1) | Multi-monitor | Remote-desktop | 2nd-workspace (virtual desktop / overlay / process enum) | Screen-content monitoring | Host-agent level | Source# |
|---|---|---|---|---|---|---|---|---|
| **ProctorU / Meazure (Guardian Browser)** | Y (scans 400+ apps for VM) | N | Y (prompts disconnect) | Y (400+ incl TeamViewer/AnyDesk/Splashtop/CRD/RDP) | Y (logs ALL active processes; re-checks every 30s) | Y (record+review/live) | NATIVE | [S15][S16] |
| **Rosalyn / StableSight** | Y (deep process monitoring) | N | Y (no multiple displays) | Y (process analysis + typing-sound correlation) | Y (process monitoring + "screen visual detection" of other apps/start menu) | Y (screen record, AI+human) | NATIVE (desktop app) | [S25][S26][S27] |
| **Constructor / Examus** | Y (automatic w/ Secure Browser; flags "Virtualization is used") | N | Y (device detection) | Y (TeamViewer/Skype/RemoteAdmin; thin clients) | Y (blocks **overlays**; flags active-window change) | Y (100+ params, gaze) | NATIVE | [S23][S24][S28] |
| **Respondus LDB + Monitor** | Y (blocks incl Virtual Displays/Virtual Desktops) | N | Y (blocks extended desktop; blocking-window ALT+F4-closeable) | Y (blocks remote desktop/screenshare; TeamViewer caught, Win Remote Assistance NOT) | ~ (blocks task-switch/background apps at launch; no continuous deep enum; overlay/Cluely evades) | Y (optional "Webcam & Screen") | NATIVE | [S1][S2][S3][S20][S31] |
| **Kryterion / Webassessor** | Y (via Respondus LDB OEM) | N | Y (2nd monitor disabled by secure browser) | Y (via Respondus) | ~ (via Respondus) | Y (record+review; keystroke/eye-calib options) | NATIVE (Respondus OEM) | [S17][S18][S19] |
| **Prometric ProProctor** | Y (VM "not compatible"; also blocks Administrator-role machine) | N | Y (dual-monitor not allowed) | Y (controls desktop sharing; isolates to locked desktop) | ~ (terminates "unknown application running"; ProGuardian AI at workstation) | Y (desktop recording + 2nd camera) | NATIVE | [S10][S11][S12] |
| **PSI (Bridge Secure Browser)** | ~ (browser blocks VM at launch; 2020 doc: pre-check "unable to detect a virtual machine") | N | Y (multi-monitor error; dual not allowed) | Y (prevents remote desktop) | ~ (closes prohibited background apps at security check) | Y (records webcam + desktop) | NATIVE | [S7][S8][S9][S13] |
| **Talview (TSB + Alvy)** | Y (TSB "Virtual Desktop Detected" = VM/RDP/screen-share) | N | Y (screen-mirroring blocked) | Y (TSB detects RDP) | Y (flags virtual-desktop switch; but macOS overlay below capture evades) | Y (full-screen share; TSB blocks screen record) | NATIVE (TSB) or EXT (screenshare) | [S21][S22][S32] |
| **Mercer Mettl (MSB)** | Y (MSB "detects VM & does not allow proceed") | N | Y (multi-screen error; projection barred) | Y (blocks TeamViewer/screenshare; terminates if shared) | ~ (MSB shuts active apps; non-MSB = pixels only, no process enum) | Y (60s snapshots; full screenshare via ext) | NATIVE (MSB) or EXT (screenshare) | [S4][S5][S6] |
| **Pearson VUE OnVUE (BASELINE)** | Y (Browser Lock "blocks against the use of virtual machines") | N | Y (one display only; secondary must disconnect) | Y (blocks screen-sharing/mirroring tools) | ~ (Browser Lock names+closes disallowed processes at launch; user's deployment disables continuous screen/host monitoring) | ~ (capable; OFF in user's deployment) | NATIVE (Browser Lock) | [S33][S34][S35][S36] |
| **Honorlock** | N (extension; VM not documented) | N | ~ (detects window resize via chrome.system.display) | N (extension cannot) | N (extension; optional desktop app "detects apps you use" in limited release) | Y (chrome.desktopCapture, per-exam consent) | EXT (+APP limited release) | [S37][S38][S39] |
| **Proctorio** | ~ (extension VM detect weak/heuristic; "Advanced VM Detection" needs Companion App) | N | Y ("Only One Screen"; counts displays) | ~ (invisible if RDP app off-screen; study) | ~ (Advanced Program Detection needs Companion App; blocks Cluely by name) | Y (Record Screen optional, Chromium capture) | EXT (+APP for advanced) | [S40][S41][S42][S43] |
| **Examity** | ? (undocumented) | N | Y (dual monitors not allowed) | ? (live-proctor/record; not documented) | ? (undocumented) | Y (record + live proctor) | browser (+optional ext) | [S44] |
| **ProctorExam** | N (extension; cannot access files/processes) | N | ? | N (extension cannot) | N (pixels only via screenshare) | Y (screenshare + webcam + mobile 2nd cam) | EXT | [S45] |

Cross-cutting: **No vendor documents hardware attestation (TPM, Secure Boot, measured boot) or Type-1/bare-metal hypervisor detection.** Every "VM detection" is a software heuristic (CPUID vendor string, driver/registry/device-name lists, SMBIOS) that the evasion literature shows is defeatable by a tuned guest or a Type-1 hypervisor [S46][S47][S48].

---

## Per-vendor notes

### ProctorU / Meazure Learning — Guardian Browser (NATIVE) — strongest documented continuous host integrity
- Guardian is a proprietary **native secure browser**; it "blocks interaction with locally installed applications and checks for unpermitted software **every 30 seconds**" and "**logs all active processes** on a test-taker's machine for investigation" [S15].
- "Our proctoring system **scans for over 400 applications to detect virtual machines and remote access tools**. It also uses other methods to identify virtual security threats during an exam session" [S15].
- Denylist (published, non-exhaustive) covers remote access (TeamViewer, AnyDesk, Splashtop, **Chrome Remote Desktop / Remoting_Host.exe**, Microsoft Remote Desktop), screen capture/stream (Duet Display, GreenShot, ScreenPal, NVIDIA Broadcast), other browsers, and writing assistants (Grammarly); "monitors hundreds of additional processes" [S16].
- Multi-monitor: a detected secondary monitor prompts the test-taker to disconnect before proceeding [S15]. Concurrent-login block also present [S15].
- Relevance: continuous process enumeration is the single control that catches an LLM/answer **process** even when its window is hidden or capture-excluded. Residual gap: VM detection is still software-heuristic; no hardware attestation.

### Rosalyn / StableSight (NATIVE desktop app, AI + human-in-the-loop)
- Native desktop application (Windows/macOS), not an extension; "only runs if and when the test taker launches it," access limited to camera/mic/screen; ensures **no multiple displays** and that **all apps other than Rosalyn are closed** [S25][S26].
- StableSight "System Analysis": "**Deep process monitoring** detects sophisticated technical violations, **from virtual machines to unauthorized applications**" [S27].
- Remote-Access Detection: "AI-driven computer process analysis to continuously detect if unauthorized processes, such as a screen-sharing app, are running **or if the test is launched in a virtual machine**"; adds **typing-sound correlation** (typing with no keyboard tapping → flag) and "**screen visual detection** … identify if the start menu or **another app is accessed, even if it bypasses all the other processes running**" [S28].
- Relevance: the screen-visual-detection + process-monitoring combination is explicitly aimed at concealed second apps/overlays — directly on-point for the one-machine LLM-overlay vector. Residual gap: VM detection still heuristic; no attestation.

### Constructor / Examus (NATIVE Secure Browser + AI)
- Secure Browser "locks down the test environment, **blocks restricted apps and overlays**, and supports allow-listed resources" [S23].
- "**Virtual machine detection: Automatic whenever the Secure Browser is used**" [S23]; AI flag list includes "**Virtualization is used** — The proctoring application is running inside a virtual machine" and "**Active window is changed** — switched to another tab or application" [S24].
- Legacy Examus documentation: detects "the usage of **virtual machines and thin clients**," plus TeamViewer/Skype/RemoteAdmin "desktop broadcasting software" [S28-note / S49].
- Relevance: explicit **overlay** blocking + active-window-change detection is rare and directly targets the concealed-workspace vector. Residual gap: VM detection heuristic; overlay block relies on the secure browser being the active host.

### Respondus LockDown Browser + Monitor (NATIVE custom browser) — strong on VM, weaker on overlay
- Native custom browser (not an extension): "native applications have direct access to the operating system … whereas a plugin is blocked from accessing key areas of the OS" [S3].
- Blocks VMs and refuses to run when a VM is detected — explicitly including "**Virtual Displays / Virtual Desktops**," VMware/VirtualBox, thin apps (ThinApp/App-V), Windows emulators (WINE/CrossOver), and virtual hard drives [S1]. Also blocks remote desktop, screenshare, extended desktop/multiple monitors, background comms apps, task switching [S2][S3]. Monitor adds optional "Webcam & Screen" screen recording [S2].
- Documented evasions (academic/community): student-version LDB **fails to detect Windows Remote Assistance** running as a process (while TeamViewer IS caught); the purple secondary-display **blocking window can be closed with ALT+F4** to reveal the second monitor [S20]. VM detection is defeated by registry-key edits + fixed-size VHD + ≥4 GB RAM on a tuned guest [S31], and by a Windows-Sandbox hooking tool (cat-and-mouse; Respondus began detecting it ~Feb 2025) [S50].
- Does NOT reach a capture-excluded AI overlay (Cluely-type) that runs as a separate OS process outside the single browser window [S32]. Residual gap: single-window lockdown model + heuristic VM detection.

### Kryterion / Webassessor (NATIVE — uses Respondus LDB OEM)
- Online-proctored exams require the **Respondus LockDown Browser (OEM)** secure browser; it creates a locked-down environment and "disables plug-ins and applications (such as recording software and Skype)" [S17].
- "**Dual monitors are detected by the secure browser, and the second monitor is disabled**" [S17]. Virtual machines "are not allowed"; candidates must close VMware/Bootcamp/TeamViewer/screen-sharing before launch [S18][S19]. Optional keystroke verification, facial recognition, and eye-calibration biometrics [S19].
- Inherits Respondus's VM/remote-desktop/multi-monitor detection and its limitations (above).

### Prometric ProProctor (NATIVE lock-down desktop + ProGuardian AI)
- Proprietary lock-down browser "isolat[es] the exam session to a **locked-down desktop**, taking **complete control over the computer environment**," making ProProctor "the only application running," and controls access to email/IM/**desktop sharing** — "without making modifications to the host system" [S11].
- VMs, VPNs, firewalls, and "**machines running in Administrator Role**" are "**not compatible**" with ProProctor [S10] (admin-role block is a notable anti-tamper posture). Dual-monitor not allowed [S10].
- "**ProGuardian** brings AI-assisted detection to the testing **workstation**," backed by a 24/7 Security Command Center; desktop recording + second camera are available; "detecting if there is an unknown application running … the application can quickly be terminated" [S12][S11].
- Residual gap: public docs don't quantify process/overlay-enumeration depth; VM handled as "incompatible" rather than described as continuous detection.

### PSI (Bridge / PSI Secure Browser) (NATIVE)
- Native downloadable secure lockdown browser; the "Record and Review platform … prevents users from copying, pasting, taking screen grabs, **using remote desktop and virtual machines**, using instant messaging or other applications" [S7]. No remote-proctor-access model (differentiator) [S9].
- Multi-monitor detected → must disconnect; "Dual screens are not allowed" [S8][S13]. A security check prompts closing prohibited background processes (e.g., MS Word) [S7-note]. Records webcam + desktop [S13].
- Important nuance: a 2020 setup guide states "you cannot take an exam using a virtual machine even though the compatibility check **may not display any issues**. The compatibility check is **unable to detect a virtual machine**" — i.e., VM enforcement is policy + secure-browser-launch, with a documented pre-check gap at that time [S14].

### Talview (Talview Secure Browser "TSB" + Alvy AI)
- TSB (native) raises a "**Virtual Desktop Detected**" error when it identifies the device "connected to another system through **Remote Desktop, Virtual Machine (VM), or screen-sharing software**, or if you try to switch to a **virtual desktop** during the exam" [S21]. TSB also restricts screen recording, app/tab switching, alt-tab, browsing, and keyboard shortcuts [S22].
- Alvy AI "names applications and devices, inside a secure browser"; liveness + face match rules out a virtual camera (US Patent 12,361,115 B1) [S32-brand]. Non-TSB mode is Chrome/Edge full-screen share (pixels only).
- Limitation: third-party overlay tooling reports that a macOS overlay window sits **below** Talview's WebRTC/tab-capture path and is not seen; TSB "VM checks rely on browser-exposed system fingerprints and timing heuristics" [S32].

### Mercer Mettl (Mettl Secure Browser "MSB" + SecureProctor extension)
- MSB (native) markets: "**Prohibits virtual machines — Detects virtual machine and does not allow the test taker to proceed**," bars screen projection to external screens, and blocks screen-sharing apps (TeamViewer), VMs, and remote desktops; disables external ports to prevent screen duplication/extension [S4][S5].
- Screen/camera snapshots every 60s; in non-MSB mode a Chrome screen-sharing extension is required and the test "**terminates if the candidate shares a screen after starting MSB**"; multi-screen raises an error [S5][S6].
- Caveat: Mettl's *published* 15-item anti-cheat list does not itself name VM detection, and non-MSB (extension) capture is "pixels, not processes" — it "cannot list the applications running on your machine" [S6]. Strength depends entirely on MSB being enforced.

### Pearson VUE OnVUE (BASELINE — Browser Lock secure browser + camera)
- Browser Lock is a native secure browser: it "**blocks against the use of virtual machines**" and requires all disallowed applications closed before launch, **naming the specific applications/processes** that must be closed [S33][S34]. It blocks Photoshop, screen-capture, screen-sharing, and screen-mirroring tools and app-switch keyboard shortcuts; screen-capture software cannot run inside it [S35][S36].
- Policy layer: one display only; virtual machines, secondary/touchscreen displays, VPNs, corporate networks all prohibited; "close all applications except OnVUE" [S33]. Full candidate agreement authorizes real-time monitoring of "face, voice, desk, and workspace" [S36-agr].
- **User's specific deployment (per Goals.MD)** disables screen monitoring and candidate-computer monitoring, leaving the 360° camera + OnVUE's launch-time Browser Lock checks and telemetry. So OnVUE's *product* capability (VM block + process-close at launch) is mid-tier-strong, but **this deployment removes the continuous host/screen layer** that would catch a second workspace opened *after* launch — the precise gap T07 is asked to close.

### Honorlock (Chrome EXTENSION; optional desktop app in limited release) — weak host integrity
- Mass-market product is a **Chrome extension only**: "Honorlock does not have the ability to control your computer, read your passwords or download any files" [S38][S39]. Screen recording via `chrome.desktopCapture` with per-exam consent; `chrome.system.display` used to detect window resizing [S37].
- A **new limited-release desktop Application** (installed alongside the extension) is "designed to **detect apps on your device that you use (or attempt to use)** during the exam" — this is the only Honorlock path with host-level process visibility [S38][S39].
- Secondary-Device Detection / Search & Destroy (audio watermark, traces phone searches) targets a **separate physical device**, not a second workspace on the same machine — out of scope for this threat [S37-sd]. VM detection not documented.

### Proctorio (Chrome/Edge EXTENSION; optional Companion App) — conditional
- Runs as a browser extension with limited OS access ("cannot access personal files or change system settings") [S41]. Lockdown "Only One Screen" blocks multi-monitor and the number of connected displays is recorded [S42].
- **Advanced VM Detection** and **Advanced Program Detection** (program lockdown / Office-app allow) **require Proctorio's Companion App**, a separate desktop application [S40]. Without it, independent studies show the extension's VM detection is weak heuristic (screen-resolution/cores/disk) and that students defeated VM, virtual-webcam, and virtual-desktop use and googled answers; remote desktop is "invisible if not on screen" [S43]. Proctorio now blocks Cluely-type overlays "by name" at launch [S32-cluely].

### Examity — live-proctor / ID-centric (thin host integrity for this threat)
- Dual monitors, phones, tablets, headphones not allowed; multi-layered authentication (facial match, challenge questions, biometric keystroke/typing match); browser Back disabled [S44].
- VM detection, remote-desktop detection, and process enumeration are **not documented** in the candidate guide — host integrity for the one-machine threat is **undocumented/unverified**; its strength has historically rested on live-proctor observation and record-and-review, which a concealed on-machine overlay evades.

### ProctorExam — EXTENSION + screenshare (weak)
- Chrome browser extension for screen-share; "We cannot access any file on your computer at anytime," only browsing history during the exam [S45]. Options: screenshare only / + webcam / + mobile phone second camera.
- Pixels-only; no VM, remote-desktop, process, or overlay detection documented — structurally unable to see a second workspace on the same machine.

---

## Vendors strongest against the one-machine concealed-workspace threat (ranked)

The threat decomposes into two software-only sub-vectors on one machine:
- **(a) VM split** — run the exam in a guest and research on the host (or vice-versa). Needs VM/hypervisor detection or attestation.
- **(b) Concealed host workspace** — a virtual desktop (Task View), a background LLM app, or an **AI "answer overlay"** (Cluely/OpenCluely/Pluely/Nistaro-type) that sets an OS screen-capture-exclusion flag (Windows `WDA_EXCLUDEFROMCAPTURE`) and runs as a **separate process outside** the exam window. A browser tab/extension and even a single-window lockdown browser structurally **cannot** see this; only a native agent enumerating **processes + top-most/overlay windows + virtual displays (+ GPU/network)** can [S29][S32][S51].

Ranking reflects **documented native host-agent depth** (continuous process + overlay/virtual-display enumeration), which is what actually closes sub-vector (b), plus VM handling for (a).

1. **ProctorU / Meazure (Guardian Browser).** Native agent; **logs all active processes, re-checks every 30 s, scans 400+ apps for VMs and remote-access tools**, prompts on second monitor [S15][S16]. The continuous process enumeration catches a hidden LLM **process** even when its window is capture-excluded. Best documented coverage of (b); partial on (a) (heuristic VM).
2. **Rosalyn / StableSight.** Native desktop app; **deep process monitoring (VM + unauthorized apps), remote-access detection, typing-sound correlation, and explicit "screen-visual detection" of another app/start menu even if it bypasses process checks** [S27][S28]. Most directly engineered against the concealed-overlay vector.
3. **Constructor / Examus.** Native secure browser with **automatic VM detection, explicit overlay blocking, and active-window-change flags** [S23][S24]. Overlay blocking is rare and squarely on-point for (b).
4. **Prometric ProProctor.** Native locked-down **desktop** (sole running app), VM + **Administrator-role machine** blocked, "unknown application running" terminated, ProGuardian AI on the workstation [S10][S11][S12]. Strong posture; less public detail on overlay-enumeration depth.
5. **Respondus LDB (and Kryterion, which licenses it).** Native; blocks VMs including **virtual displays/virtual desktops**, remote desktop, extended desktop, task switching [S1][S2][S17]. But documented evasions (Windows Remote Assistance undetected, ALT+F4 on the monitor-blocker, registry-tuned VM) and a single-window model that **does not reach a capture-excluded overlay** cap it at mid-strong [S20][S31][S32].
6. **Talview (TSB).** Native secure browser detects VM/RDP/screen-share and **virtual-desktop switching** [S21][S22]; weakened by overlay-below-capture on macOS [S32].
7. **PSI Bridge.** Native secure browser blocks VM/remote-desktop/other apps and multi-monitor, but a documented pre-check VM-detection gap and limited continuous-enumeration detail [S7][S8][S14].
8. **Mercer Mettl (MSB).** Native MSB detects/blocks VMs, bars screen projection, terminates on screen-share; collapses to pixels-only when only the extension (non-MSB) is used [S4][S5][S6].
9. **Pearson VUE OnVUE (baseline).** Browser Lock blocks VMs and names/closes disallowed processes **at launch** and blocks screen-share/mirror tools [S33][S34][S35] — mid-tier as a product, but **the user's deployment turns off continuous screen/host monitoring**, so a workspace opened after launch is the open gap this task targets.
10. **Proctorio** (needs Companion App to reach mid-tier; extension-only is weak) [S40][S43]; **Honorlock** (extension; host visibility only via limited-release desktop app) [S38][S39]; **Examity** (undocumented host integrity) [S44]; **ProctorExam** (extension/pixels-only, weakest) [S45].

**Bottom line for the exam owner.** To close the one-machine concealed-workspace gap, select a vendor with a **native, continuously-running host agent that enumerates processes and overlay/top-most windows and virtual displays throughout the exam** — in descending documented strength: **ProctorU/Guardian, Rosalyn/StableSight, Constructor/Examus**, then **Prometric ProProctor** and **Respondus/Kryterion**. Even these only **raise cost**, not eliminate the threat: (i) **no vendor offers hardware attestation (TPM/Secure Boot/measured boot) or Type-1 hypervisor resistance**, so a tuned guest or bare-metal hypervisor still defeats the VM sub-vector [S46][S47][S48][S51]; and (ii) capture-excluded AI overlays are only reachable by OS-level process/overlay/GPU/network signals, which a browser-extension product cannot provide [S32][S51]. Any deployment (like the user's current OnVUE configuration) that disables continuous screen/host monitoring forfeits the single layer that catches a post-launch second workspace.

---

## Sources (numbered URLs + access dates)

Access date for every source: 2026-11-12.

- [S1] Respondus Support — "The browser can't be used in virtual machine software…" (VM warning; lists Virtual Displays/Virtual Desktops, ThinApp/App-V, WINE/CrossOver, vHDs). https://support.respondus.com/hc/en-us/articles/4409604116123
- [S2] Respondus — LockDown Browser product page (blocks VMs, remote desktops, screensharing, screen recording, task switching). https://www.respondus.com/products/lockdown-browser
- [S3] Respondus — "LockDown Browser vs. 'locked browser' extensions" (native app = direct OS access; lists VM/remote-desktop/extended-desktop/background-app exploits blockable only by native app). https://web.respondus.com/lockdown-browser-vs-locked-browser-extensions
- [S4] Mercer | Mettl — "Safe & Secure Exam Browser" (Prohibits virtual machines — detects VM and does not allow proceed; bars screen projection; blocks screenshare apps). https://mettl.com/en-gb/safe-and-secure-exam-browser
- [S5] Mercer | Mettl — Security Document PDF (VMs/remote desktops/screen-sharing blocked; external ports disabled to prevent screen duplication/extension). https://mettl.com/downloads/wp-content/uploads/sites/12/2020/05/MM_Security-Document_V2.pdf
- [S6] TryControl — "Does Mettl Record Your Screen?" / InterviewMan — "Does Mercer Mettl Detect AI Tools?" (60s snapshots; terminates on screen-share; non-MSB capture is pixels, cannot list processes; VM not on published list). https://trycontrol.app/blog/does-mettl-record-your-screen ; https://interviewman.com/blog/does-mercer-mettl-detect-ai-tools
- [S7] PSI Exams — "Secure online proctoring: the threat of remote access" (Record & Review secure browser prevents remote desktop and virtual machines, screen grabs, IM, other apps). https://www.psiexams.com/knowledge-hub/secure-online-proctoring-for-education-the-threat-of-remote-access/
- [S8] PSI/TCN Helpdesk — "[PSI Bridge] Multiple Monitors Detected - Windows OS." https://helpdesk.psionline.com/hc/en-gb/articles/4403236819476
- [S9] PSI — "Online Proctoring with PSI Bridge" / "PSI Bridge for Education" (secure lockdown browser, no remote access). https://content.psionline.com/hubfs/PSI_Bridge_for_Education.pdf
- [S10] Prometric — Remote Exam Experience FAQ ("VPNs, firewalls, Virtual Machines (VMs), and machines running in Administrator Role … are not compatible"; dual-monitor not allowed). https://www.prometric.com/remote-exam-experience
- [S11] Prometric — "Combating Proxy Testing in the Digital Age" (lock-down browser isolates to locked-down desktop, complete control, only app running; controls desktop sharing; terminates unknown app). https://www.prometric.com/blog/combating-proxy-testing-in-the-digital-age
- [S12] Prometric — Security page (ProGuardian AI-assisted detection at the workstation; Security Command Center; desktop recording + second camera). https://www.prometric.com/security
- [S13] PSI — "A Guide to Taking Your Test with PSI" / Bridge NCARB guide (records webcam + desktop; only one monitor; closes prohibited programs). https://www.psiexams.com/test-takers/resources ; https://content.psionline.com/hubfs/PSI%20Bridge%20Full%20Guide%20NCARB%20-%20UPDATES%203.14.22.pdf
- [S14] PSI — Online Proctored Exam Set Up Guide (2020): "The compatibility check is unable to detect a virtual machine." https://wac-cdn.atlassian.com/dam/jcr:9b7a1db5-524d-46d1-83e1-3a264f3872bc/PSI-Online-Proctored-Exam-Setup-Guide
- [S15] Meazure Learning — "Proctoring in Focus, Part 1" (Guardian checks every 30 s, logs all active processes, scans 400+ apps for VMs and remote-access tools; multi-monitor prompt). https://www.meazurelearning.com/resources/proctoring-in-focus-part-1-online-exam-security
- [S16] ProctorU Support — "Unpermitted Applications" (Guardian denylist: remote access, screen capture/stream, other browsers, writing assistants; monitors hundreds of processes). https://support.proctoru.com/hc/en-us/articles/46589198199565-Unpermitted-Applications
- [S17] Kryterion Webassessor — Online Proctoring Testing Standards (secure browser = Respondus LockDown Browser; dual monitors detected and 2nd disabled). https://guide.webassessor.com/userguide/online-proctoring-testing-standards ; -comprehensive version https://guide.webassessor.com/userguide/online-proctoring-testing-standards-comprehensive
- [S18] MS Council / Kryterion — "Online Testing Requirements" (no virtual machines; LockDown Browser install needs admin). https://www.mscouncil.org.nz/assets_mlsb/Uploads/Online-Testing-Requirements-Webassessor.pdf
- [S19] Kryterion Support — "Accessibility and How to Install the Respondus Secure Browser" / "How to Launch an OLP Exam" (disable VMware/Bootcamp/TeamViewer/screen-share; keystroke + facial + eye calibration). https://support.kryterion.com/en_us/accessibility-and-how-to-install-the-respondus-secure-browser-ByivBLks1e ; https://support.kryterion.com/en_us/how-to-launch-an-online-proctored-(olp)-exam-Byr5BLJi1e
- [S20] IAFOR IICE2023 — "Automated Proctoring Solutions: Modern Techniques to…" (Respondus fails to detect Windows Remote Assistance process; ALT+F4 closes secondary-display blocking window). https://papers.iafor.org/wp-content/uploads/papers/iice2023/IICE2023_66044.pdf
- [S21] Talview Docs — "Why Am I Seeing a 'Virtual Desktop Detected' Error in the Secure Browser?" (fires on Remote Desktop, VM, screen-sharing, or switching to a virtual desktop). https://docs.talview.com/candidates/system-diagnostics/talview-secure-browser-tsb-compatibility/why-am-i-seeing-a-virtual-desktop-detected-error-in-the-secure-browser
- [S22] Talview — Talview Secure Browser datasheet (no app/tab switching, blocks alt-tab, prevents screen recording/capture). https://info.talview.com/hubfs/Talview-Secure-Browser.pdf
- [S23] Constructor Tech — Proctor product page (Secure Browser blocks restricted apps and overlays; "Virtual machine detection: Automatic whenever the Secure Browser is used"). https://constructor.tech/products/learning/proctor
- [S24] Constructor Tech — Proctor Guide PDF (AI flags: "Virtualization is used — running inside a virtual machine"; "Active window is changed"). https://docs.constructor.tech/documentation/pdfs/en/proctor-proctor-guide.pdf
- [S25] Rosalyn — Admin FAQ (locked-down browser; no multiple displays; ensures all apps other than Rosalyn closed; not a browser extension — app runs only when launched). https://chat.prod.rosalyn.ai/hc/admin-help-center/articles/1774307834-rosalyn-faq
- [S26] Synap — "Taking a Rosalyn-proctored exam" / Rosalyn Examinee Help (desktop application for Mac/Windows; records screen/webcam/mic). https://synap.ac/candidates/candidate-rosalyn-proctoring ; https://support.rosalyn.ai/en/articles/6085549-rosalyn-proctoring
- [S27] Rosalyn — "Platform · Multi Modal Detection" (StableSight System Analysis: deep process monitoring detects VMs to unauthorized applications; remote-access prevention; system integrity verification). https://www.rosalyn.ai/platform-multi-modal-detection
- [S28] Rosalyn — "Unveiling Rosalyn.ai's Advanced Proctoring Features" (Remote Access Detection via process analysis incl. VM launch; typing-sound correlation; screen-visual detection of start menu/another app). https://www.rosalyn.ai/blog/unveiling-rosalyn-ais-advanced-proctoring-features-reinforcing-examination-integrity-through-cutting-edge-ai
- [S29] ProctorLink — "What Browser-Based Proctoring Can and Cannot Detect" (a VM or remote-desktop session is invisible to any tab; VM/process inspection is a Tier-3 desktop-agent capability). https://proctorlink.com/knowledge/what-browser-proctoring-can-detect
- [S30] (architecture, same as S29/S3).
- [S31] Simko et al. (CCS 2024) — "'Modern problems require modern solutions': Community-Developed Techniques for Online Exam Proctoring Evasion" (VM registry-key edits, fixed-size VHD, ≥4 GB RAM to evade Respondus VM detection). https://www.lucysimko.com/publications/simko-exam-proctoring-evasion.pdf
- [S32] Aiseptor — "Can Cluely Be Detected? Why Lockdown Browsers Miss It" (overlays set WDA_EXCLUDEFROMCAPTURE / macOS window-privacy flag; run outside the lockdown browser's one window; only OS-level top-most/overlay-window + GPU VRAM + process/network signals catch them; Honorlock/Proctorio block Cluely by name). https://aiseptor.com/detect-cluely  ·  LDBypass — "AI Overlay for Talview" (overlay sits below Talview's WebRTC/tab-capture; Talview VM checks = browser fingerprints/timing heuristics). http://ldbypass.com/guides/ai-overlay-talview
- [S33] Pearson VUE — OnVUE testing requirements (one display only; prohibited: virtual machines/beta OS, secondary/touchscreen displays, VPNs/corporate networks; close all apps except OnVUE). https://www.pearsonvue.com/us/en/onvue/requirements.html
- [S34] Pearson VUE — Client-Proctored Client Support FAQ (Browser Lock blocks against use of virtual machines; names disallowed applications/processes that must be closed; screen-capture software cannot run in secure browser). https://www.pearsonvue.com/content/dam/VUE/vue/en/documents/client-proctored/Client-Proctored-Client-Support-FAQ.pdf
- [S35] Pearson VUE — OnVUE Candidate Technical Requirements PDF (Browser Lock blocks Photoshop, screen-capture, screen-sharing, screen-mirroring tools and app-switch shortcuts; macOS Automation + Input Monitoring). https://www.pearsonvue.com/content/dam/VUE/vue/en/documents/tech-specs/online-proctored/onvue-technical-requirements.pdf
- [S36] Pearson VUE — "Traveling the online testing route" guide (shut down all other apps; OnVUE will attempt to close them) ; OnVUE online exam agreement (real-time monitoring of face/voice/desk/workspace). https://www.pearsonvue.com/content/dam/VUE/vue/en/documents/onvue/pearson-vue-online-testing-guide-en.pdf
- [S37] Honorlock Help Center — Extension Permissions and Functions (chrome.desktopCapture screen record; chrome.system.display window-resize detection; chrome.management) ; "How Honorlock Monitors Secondary Devices" (Search & Destroy / audio watermark = separate device). https://honorlock.kb.help/honorlock-extension-permissions-and-functions/ ; https://honorlock.kb.help/how-honorlock-monitors-secondary-devices
- [S38] Honorlock — Faculty & Student FAQ ("widely available products use only a Chrome browser extension"; limited-release desktop app "to detect apps on your device you use or attempt to use"). http://honorlock.com/faculty-student-faq
- [S39] Honorlock — Student Data Privacy & Security (no ability to control your computer; optional downloaded Application detects prohibited programs). https://honorlock.com/studentprivacy
- [S40] Oregon State Ecampus — "Proctorio Recommended Settings" PDF (Advanced VM Detection and Advanced Program Detection require Proctorio's Companion App; "Only One Screen"). https://ecampus.oregonstate.edu/faculty/exam-proctoring/Proctorio%20Recommended%20Settings.pdf
- [S41] Proctorio — FAQ / Secure Exam Proctor extension article (browser extension; cannot access files or change system settings; gaze/face detection only). https://proctorio.com/faq ; https://proctorio.com/support/hc/articles/1752083369562
- [S42] CSUF IT — Proctorio FAQ (records number of display screens connected; detects monitors; "Only One Screen"). https://www.fullerton.edu/it/services/software/proctorio.html
- [S43] Thom Wiggers — "Dissecting Proctorio" ; Univ. Twente — "On the Efficacy of Online Proctoring using Proctorio" (extension VM detection weak; students used VM/virtual-desktop/virtual-webcam and googled answers; remote desktop invisible if off-screen). https://thomwiggers.nl/blog/proctorio/ ; https://ris.utwente.nl/ws/portalfiles/portal/275927505/3e2a9e5b2fad237a3d35f36fa2c5f44552f2.pdf
- [S44] Okta/Examity — Examity Users Guide (dual monitors not allowed; facial match + challenge questions + biometric typing; VM/process detection not documented). https://www.okta.com/sites/default/files/2023-02/Examity_Use_Guide.pdf
- [S45] ProctorExam — Privacy for test takers (Chrome extension for screen-share; cannot access any file; only views browsing history during exam; optional mobile 2nd camera). https://proctorexam.com/privacy-for-test-takers
- [S46] Mohamed et al., arXiv:2205.03009 — "An Empirical Study of Proctoring Software" (VM detection = CPUID vendor-string check / driver-list search; "easily bypassed"; Chromium can be recompiled to report as explorer.exe). https://arxiv.org/pdf/2205.03009
- [S47] Microsoft Learn — "TPM attestation overview for Azure" (what hardware attestation/measured boot is; no proctoring vendor was found using it — negative-result reference). https://learn.microsoft.com/en-us/azure/attestation/tpm-attestation-concepts
- [S48] gucci-on-fleek/lockdown-browser (GitHub) — runs Respondus LDB in Windows Sandbox, bypassing VM detection; Respondus began detecting the tool ~Feb 2025 (illustrates heuristic VM cat-and-mouse). https://github.com/gucci-on-fleek/lockdown-browser
- [S49] Examus (now Constructor) — "Online proctoring. How it works?" (detects virtual machines and thin clients; TeamViewer/Skype/RemoteAdmin). https://medium.com/examus-official-blog/online-proctoring-how-it-works-ebe9a0813abe
- [S50] (same tool as S48; detection-timeline note.)
- [S51] GekkoQuest/fairview (GitHub) — hobby host-agent documenting the OS-level signals a strong agent uses (click-through/overlay detection, virtual-display via QueryDisplayConfig, Microsoft-Hv CPUID on OEM SMBIOS = nested-VM heuristic); also states client-side heuristics cannot see an LLM inside a whitelisted browser tab — illustrates technique, not a commercial vendor. https://github.com/GekkoQuest/fairview
