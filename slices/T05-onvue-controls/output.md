# T05 OnVUE Control Surface

Scope: What Pearson VUE **OnVUE** online-proctored delivery (secure browser = **Pearson VUE Browser Lock** / Pearson ITS Secure Browser; app process `proctorapp`) technically detects, blocks, and cannot observe. Defensive inventory only. All access dates **2026-11-12**. BLOCKS = prevents/stops exam; FLAGS = allowed but recorded/reviewed; UNDETECTED/UNKNOWN = not observed or not documented.

Primary caveat: OnVUE behaviour is configured per exam sponsor, so specifics vary. Pearson does **not** publish its detection internals; anything about *how* detection works is inferred from error strings, candidate reports, and third-party guides (all marked). In the defended deployment the **candidate screen is NOT shared/captured** — monitoring is webcam video + microphone audio + OnVUE client telemetry; this is corroborated below (§10).

## Summary table

| # | Control | Windows | macOS | Blocks / Flags / Undetected | Source (status) |
|---|---------|---------|-------|------------------------------|-----------------|
| 1 | System test / readiness check | Yes | Yes | Pre-exam gate (checks OS, bandwidth, cam/mic/speaker, downloads+launches secure browser, video stream) | Pearson PDF; CompTIA (primary) |
| 2 | Secure browser (Browser Lock) | Yes (`proctorapp`) | Yes (needs Camera, Mic, Automation, Input Monitoring perms) | BLOCKS other software + app-switch shortcuts during exam | Pearson PDF (primary) |
| 3 | Prohibited-process / app scan | Close all apps; Task Mgr Processes/Services | Close all apps | BLOCKS screen-capture/share/mirror, Photoshop; others must be closed | Pearson PDF (primary) |
| 4 | Virtual-machine detection | Detected; error "A virtual machine has been detected" | Presumed (not separately documented) | BLOCKS (prohibited; cannot continue) | Requirements page (primary); NashTech (3rd-party, method) |
| 5 | Hypervisor / VBS / Hyper-V | Hyper-V/VirtualMachinePlatform can trigger VM false-positive | n/a | BLOCKS via VM check; VBS/HVCI/nested = UNKNOWN | NashTech (3rd-party); vendor silent |
| 6 | Multiple monitors / displays | Single display only | Single display only | Required single; proctor FLAGS/terminates; launch-block mechanism unverified | Requirements page, PDF (primary) |
| 7 | Remote-desktop / screen-share | Blocked | Blocked | BLOCKS remote-control + screen-share tools | Open Group (3rd-party); Remote Utilities forum (anecdote) |
| 8 | Screen capture / overlay | Blocked | Blocked | BLOCKS capture/overlay/mirroring + app-switch keys; a11y allowed w/ accommodation | Pearson PDF (primary) |
| 9 | Virtual camera / virtual audio | Webcam *filters* banned; OBS-cam detection UNKNOWN | Apple "Reactions" banned | Filters = policy FLAG/prohibit; virtual-cam tech detection UNVERIFIED | Pearson PDF (filters); SETU report (3rd-party analog) |
| 10 | What proctor/system sees | Webcam + mic + telemetry; **screen not captured** | Same | Observational: live proctor + recording + analytics; facial-comparison biometrics | Privacy policy; CompTIA bans page (primary) |
| 11 | OS / platform support | Win 11 & 10 64-bit (no S Mode) | macOS 14+ (no beta) | Linux/Unix/ChromeOS UNSUPPORTED; Win7/8/XP/Vista unsupported | Pearson PDF, requirements page (primary) |

## Detailed findings

### 1. System test / readiness check
- System test should run on same device + network as exam; confirms system + bandwidth requirements. src: help.comptia.org/.../14344543746068 — 2026-11-12 — primary(sponsor-official).
- Checks OS, download speed, webcam/microphone/speaker function + permissions, secure-browser download/unzip/launch, live video-streaming connection, Pearson-server comms. src: ged.com/ifse.ca OnVUE Technical Requirements PDFs — 2026-11-12 — primary.
- Third-party guide: OnVUE "shuts down any prohibited processes it detects" during check-in. src: cciedump.spoto.net guides — 2026-11-12 — third-party/unverified.
- Required thresholds: >=6 Mbps down / 2 Mbps up (current PDF; older docs say 3 up), 4 GB RAM recommended, 1024x768 min. src: pearsonvue.com/.../onvue-technical-requirements.pdf (dated 2026-07-01) — 2026-11-12 — primary.

### 2. Secure browser / lockdown client ("Browser Lock")
- Secure browser named **Pearson VUE Browser Lock** or **Pearson ITS Secure Browser**; candidate downloads + launches it (app `proctorapp`). src: pearsonvue onvue-technical-requirements.pdf — 2026-11-12 — primary.
- "Browser Lock blocks other software during the exam... It also blocks keyboard shortcuts that let candidates switch apps or use software that is not allowed." src: same PDF — 2026-11-12 — primary (BLOCKS).
- macOS: OnVUE needs user grants for **Camera, Microphone, Automation, and Input Monitoring** (System Settings > Privacy). src: same PDF, "Mac special note" — 2026-11-12 — primary.
- Privilege: Pearson docs do **not** explicitly state admin rights. Third-party advisory claims "OnVUE requires full local administrator rights." src: help.opengroup.org/.../34238069858706 — 2026-11-12 — third-party/unverified-by-vendor.
- Windows must pass Genuine Windows validation; macOS beta versions excluded. src: pearson PDF — 2026-11-12 — primary.

### 3. Prohibited-process / application scanning
- Minimum requirement: "Ability to close all applications except OnVUE." src: pearsonvue.com/us/en/onvue/requirements.html — 2026-11-12 — primary.
- Browser Lock "blocks other software during the exam, including Photoshop, screen capture tools, screen-sharing tools, and screen-mirroring tools." src: pearson PDF — 2026-11-12 — primary (BLOCKS named categories).
- Windows: "close most software in Task Manager > Processes. Some software may appear under Task Manager > Services." src: pearson PDF — 2026-11-12 — primary.
- Enumeration method (how it lists/kills processes) is **not documented**. status: UNKNOWN.
- Endpoint-protection/DLP/behaviour agents (CrowdStrike, Defender ATP, Sophos) can block launch or trigger termination. src: help.opengroup.org/.../34238069858706 — 2026-11-12 — third-party.

### 4. Virtual-machine detection
- Prohibited item: "Virtual machines or beta operating systems." src: pearsonvue.com/us/en/onvue/requirements.html — 2026-11-12 — primary.
- Blocking error string: "A virtual machine has been detected. Please close all virtual machine software before continuing." (candidate cannot continue). src: blog.nashtechglobal.com/fixing-onvue-virtual-machine-detected-error... (2026-03-19) — 2026-11-12 — third-party (BLOCK).
- CompTIA: "Disable any Virtual Private Networks (VPNs) or Virtual Machines before testing." src: help.comptia.org/.../14344543746068 — 2026-11-12 — primary(sponsor).
- Open Group: "OnVUE actively blocks ... Virtual machines. Even if these are running silently in the background, the system check can fail." src: help.opengroup.org/.../34238069858706 — 2026-11-12 — third-party.
- Detection method itself = **not documented**; false positives reported from leftover VM drivers (Device Manager). src: NashTech blog — 2026-11-12 — third-party/unverified.

### 5. Hypervisor / virtualization-based detection
- Windows virtualization features (Hyper-V, VirtualMachinePlatform), running WSL, Docker/container services reported to trigger the "VM detected" block even with no active VM. src: blog.nashtechglobal.com/fixing-onvue-virtual-machine-detected... — 2026-11-12 — third-party/anecdotal.
- Reported workaround trigger-set: `wsl --shutdown`, DISM-disable VirtualMachinePlatform + Microsoft-Hyper-V-All, detach Dev Drive, end WSL processes, reboot. src: same — 2026-11-12 — third-party (indicates these surfaces are what the check reacts to).
- VBS / Memory Integrity (HVCI) / nested virtualization / Type-1 bare-metal hypervisor handling: **UNKNOWN / not documented** by Pearson or sources. status: UNKNOWN.

### 6. Multiple-monitor / display detection
- "Multiple monitors are not allowed." / "One display screen only (multi-monitor setups are not permitted)." src: pearson PDF + requirements.html — 2026-11-12 — primary.
- "Secondary or touchscreen displays (disconnect and cover if not removable)"; external monitor requires closing the laptop lid + external keyboard/mouse/webcam. src: requirements.html, pearson PDF — 2026-11-12 — primary.
- Vendor: failing requirements on exam day "can result in immediate exam cancellation and forfeiture." src: pearsonvue requirements page — 2026-11-12 — primary (proctor-side FLAG/terminate).
- Candidate reports of a launch-time "extra monitor" warning exist, but Pearson does **not** document whether the client enumerates displays (EDID/OS) and hard-blocks launch, vs proctor visual catch. status: BLOCK-mechanism UNVERIFIED; mirroring/docks may still report two active displays. src: techbloat / constructor.tech analog — 2026-11-12 — third-party.

### 7. Remote-desktop / screen-sharing detection
- "OnVUE actively blocks: Remote-control software, Screen-sharing tools, Virtual machines." src: help.opengroup.org/.../34238069858706 — 2026-11-12 — third-party.
- Browser Lock blocks "screen-sharing tools, and screen-mirroring tools." src: pearson PDF — 2026-11-12 — primary (BLOCKS).
- Candidate running Remote Utilities was prompted mid-exam to close it (OnVUE 3.27.23). src: remoteutilities.com forum thread 1623 — 2026-11-12 — anecdote.
- RDP/VNC/TeamViewer named individually: **not** in Pearson docs; covered by generic "remote-control / screen-sharing." status: inferred BLOCK, specific protocols UNKNOWN.

### 8. Screen capture / recording / overlay detection
- Browser Lock blocks screen-capture tools, screen-share, screen-mirroring, and Photoshop (cited as overlay/capture example); blocks app-switch keyboard shortcuts. src: pearson PDF — 2026-11-12 — primary (BLOCKS).
- Accessibility software (screen readers, magnifiers) may still run **only** when an accommodation is granted. src: pearson PDF — 2026-11-12 — primary (a11y-API carve-out).
- Transparent/overlay windows, virtual desktops: **not named** by Pearson. Analog vendor (Talview) shows a "Virtual Desktop Detected" error for RDP/VM/screen-share. src: docs.talview.com — 2026-11-12 — third-party analog (not OnVUE).

### 9. Virtual-camera / virtual-audio device detection
- "Webcam filters are not allowed, including Apple 'Reactions.'" src: pearson onvue-technical-requirements.pdf — 2026-11-12 — primary (policy prohibition / FLAG).
- "Mobile phones cannot be used as a webcam for exam delivery." src: same PDF — 2026-11-12 — primary.
- Whether OnVUE technically detects/blocks a software virtual camera (OBS Virtual Cam, ManyCam, Snap): **not documented**. status: UNKNOWN/UNVERIFIED.
- Analog security research: an anonymized commercial proctoring tool did not differentiate physical vs virtual camera at DirectShow API level; OBS virtual cam substituted a loop undetected (not confirmed as OnVUE). src: showcase.itcarlow.ie/C00284314 (SETU 2026) — 2026-11-12 — third-party/anonymized/educated-guess.
- Virtual-audio device detection: **no source**. status: UNKNOWN.

### 10. What the human proctor and automated system see (screen NOT shared)
- "You will be monitored over the Internet through your computer via your webcam and microphone." src: pearsonvue.com/us/en/legal/privacy-and-cookies-policy.html — 2026-11-12 — primary.
- "Your face, voice, desk and workspace will be captured and a recording will be made ... monitored during your entire testing session in real-time." src: same privacy policy — 2026-11-12 — primary.
- CompTIA: "remote proctored exams are proctored via a laptop webcam and laptop audio." src: comptia.org/.../permanent-onvue-bans — 2026-11-12 — primary(sponsor) — corroborates **no screen stream** to proctor.
- No OnVUE/privacy text describes capturing or streaming candidate **screen contents** in online proctoring; screen-side enforcement is local (Browser Lock) not observed by proctor. status: SCREEN UNOBSERVED (consistent with defended deployment).
- Check-in: self-photo, front/back government ID photos, room/workspace scan, facial-comparison biometrics (facial geometry) vs ID. src: privacy policy + cciedump guides — 2026-11-12 — primary + third-party.
- Automated security analytics: "monitoring of testing events using comprehensive analytics tools that analyze ... video from testing sessions." src: privacy policy — 2026-11-12 — primary.
- Human-detected FLAGS (terminate/invalidate): leaving webcam view, talking/mumbling, third party entering, bathroom break, wearing a watch. src: help.comptia.org/.../12874612989332 — 2026-11-12 — primary(sponsor).
- Retention: biometric data kept "as long as deemed necessary by the test sponsor" (Illinois cap 3 yrs); session audio/video supplied to sponsor. src: privacy policy — 2026-11-12 — primary. Third-party FAQ claims 30-day session retention. src: iapp.org/about/certification-privacy-faq — 2026-11-12 — third-party/possibly-stale.
- Telemetry/streaming infra (defender-relevant hosts): video over port 1935; domains `*.onvue.com`, `*.proctorcam.com`, `*.wowza.com` (stream), `*.ably.io`/`*.ably-realtime.com` (realtime), `*.twilio.com` (comms). src: pearson PDF advanced reqs — 2026-11-12 — primary.

### 11. Platform / OS version support and gaps
- Current PDF: "Windows 11 and 10 (64-bit) – (excluding 'S Mode'). macOS 14 and above – (excluding beta versions)." src: pearson onvue-technical-requirements.pdf (2026-07-01) — 2026-11-12 — primary.
- Unsupported: "Windows 8/8.1, Windows 7, Windows XP, and Windows Vista"; "Linux/Unix and Chrome-based operating systems." src: same PDF — 2026-11-12 — primary (Linux = hard gap).
- Version inconsistency: 2026 CompTIA + 2022 IFSE spec still cite **macOS 10.15+**, while current Pearson PDF + requirements page say **macOS 14+**. src: help.comptia.org/.../14344543746068 vs pearson PDF — 2026-11-12 — label unverified which floor each sponsor enforces.
- Tablets prohibited unless physical keyboard + meet OS reqs; touch screens not allowed. src: pearson PDF — 2026-11-12 — primary.
- Mobile phone (check-in only, not during exam): Android 11+ (Chrome) or iOS 15+ (Safari). src: pearson PDF — 2026-11-12 — primary.
- Corporate/managed machines strongly discouraged: MDM, always-on VPN, proxy, DLP, endpoint agents conflict; BIOS locks can break webcam/monitor/peripheral checks. src: help.opengroup.org/.../34238069858706 — 2026-11-12 — third-party.
- Restart cap: proctor may allow max 2 restarts per session before termination (older guide). src: nicet.org OnVUE Reconnection Guide (2020) — 2026-11-12 — primary-mirror/dated.

## Gaps / undocumented areas (could NOT confirm)
- Exact VM/hypervisor detection method (registry, CPUID, MAC OUI, driver/device enumeration, timing). Only error strings + false-positive triggers known. UNKNOWN.
- Whether VBS / Memory Integrity (HVCI) / nested virtualization / Type-1 bare-metal hypervisors are detected or conflict. No source. UNKNOWN.
- Whether the client hard-blocks at launch on a second display via OS display enumeration, vs relying on proctor visual room scan. UNVERIFIED.
- Whether software virtual cameras (OBS/ManyCam/Snap) and virtual audio devices are technically detected/blocked (vs policy prohibition of "filters"). UNKNOWN; analog research suggests some proctoring tools miss virtual cams.
- Precise RDP/VNC/TeamViewer per-protocol detection (only generic "remote-control/screen-sharing" documented). UNKNOWN.
- Definitive statement that OnVUE requires local admin rights (claimed by third parties; absent from Pearson docs). UNVERIFIED.
- Current authoritative session-recording retention period (privacy policy defers to sponsor; 30-day figure is third-party/possibly stale). PARTIAL.
- macOS minimum version actually enforced (docs disagree: 10.15 vs 14). INCONSISTENT.
- Keystroke logging: no source indicates OnVUE logs keystrokes. Not documented. UNKNOWN.
- No independent/primary technical teardown of the OnVUE client binary was found in sourcing; internals rely on vendor text + candidate reports.

## Sources (numbered)
1. Pearson VUE — OnVUE Candidate Technical Requirements (current PDF, dated 2026-07-01). https://www.pearsonvue.com/content/dam/VUE/vue/en/documents/tech-specs/online-proctored/onvue-technical-requirements.pdf — accessed 2026-11-12 — PRIMARY.
2. Pearson VUE — OnVUE "Online testing requirements" page. https://www.pearsonvue.com/us/en/onvue/requirements.html — accessed 2026-11-12 — PRIMARY.
3. Pearson VUE — Privacy and Cookies Policy (incl. Online Proctoring + Biometric Data sections). https://www.pearsonvue.com/us/en/legal/privacy-and-cookies-policy.html — accessed 2026-11-12 — PRIMARY.
4. IFSE mirror — OnVUE Candidate Technical Requirements (v.22.12.1 / Dec 2022, OnVUE OP 3.0; Advanced Networking section). https://www.ifse.ca/wp-content/uploads/2023/07/OnVUE-Technical-Requirements.pdf — accessed 2026-11-12 — PRIMARY (dated mirror).
5. CompTIA Help — What Are the Technical Requirements for OnVUE Online Testing? (updated 2026-03-31). https://help.comptia.org/hc/en-us/articles/14344543746068-What-Are-the-Technical-Requirements-for-OnVUE-Online-Testing — accessed 2026-11-12 — PRIMARY (sponsor).
6. CompTIA Help — What Are the Most Common OnVUE Policy Violations to Avoid? (2026-03-30). https://help.comptia.org/hc/en-us/articles/12874612989332-What-Are-the-Most-Common-OnVUE-Policy-Violations-to-Avoid — accessed 2026-11-12 — PRIMARY (sponsor).
7. CompTIA — Permanent OnVUE Bans ("proctored via a laptop webcam and laptop audio"). https://www.comptia.org/en-eu/resources/test-policies/permanent-onvue-bans/ — accessed 2026-11-12 — PRIMARY (sponsor).
8. The Open Group Help — Do Not Use a Corporate or Managed Laptop for OnVUE Online Exams (2026-03-23). https://help.opengroup.org/hc/en-us/articles/34238069858706-Do-Not-Use-a-Corporate-or-Managed-Laptop-for-OnVUE-Online-Exams — accessed 2026-11-12 — THIRD-PARTY (sponsor-help).
9. NashTech blog — Fixing OnVUE "Virtual Machine Detected" Error When You're Not Using a VM (2026-03-19). https://blog.nashtechglobal.com/fixing-onvue-virtual-machine-detected-error-when-youre-not-using-a-vm/ — accessed 2026-11-12 — THIRD-PARTY.
10. Remote Utilities forum — "Pearson VUE / OnVUE 3.27.23" (asked to close remote tool mid-exam). https://www.remoteutilities.com/support/forums/forum1/1623-pearson-vue-_-onvue_3.27.23 — accessed 2026-11-12 — ANECDOTE.
11. NICET — OnVUE Reconnection Guide (2020; max 2 restarts). https://www.nicet.org/nicetorg/assets/File/public/OnVUE-Reconnection-Guide-kim-8-27-20-reference.pdf — accessed 2026-11-12 — PRIMARY-mirror (dated).
12. IAPP — Certification Privacy FAQ (claims 30-day session-recording retention). https://iapp.org/about/certification-privacy-faq — accessed 2026-11-12 — THIRD-PARTY (possibly stale).
13. Talview docs — "Virtual Desktop Detected" error (RDP/VM/screen-share) — analog vendor. https://docs.talview.com/candidates/system-diagnostics/talview-secure-browser-tsb-compatibility/why-am-i-seeing-a-virtual-desktop-detected-error-in-the-secure-browser — accessed 2026-11-12 — THIRD-PARTY analog (not OnVUE).
14. SETU/IT Carlow student project (2026) — proctoring bypass testing; OBS virtual cam undetected on anonymized platform. https://showcase.itcarlow.ie/C00284314/index.html — accessed 2026-11-12 — THIRD-PARTY / anonymized / educated-guess.
15. UiPath community forum — proctor could not see candidate webcam feed (anecdote). https://forum.uipath.com/t/video-issue-with-uipath-certification-exam-on-pearson-onvue-system/311228 — accessed 2026-11-12 — ANECDOTE.
16. GED mirror — OnVUE Technical Requirements PDF (returned 404 at fetch time; listed for completeness). https://www.ged.com/educators-admins/wp-content/uploads/sites/3/OnVUE-Technical-Requirements.pdf — accessed 2026-11-12 — PRIMARY-mirror (unreachable).
