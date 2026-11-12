# Research Log
All research findings. Orchestrator appends from slice outputs. Never edit past entries; append corrections.

Entry format:
- [YYYY-MM-DD] [T<NN>] <finding ≤15 words> | src: <URL or file:line> | status: verified|unverified

## G1 OneVue feasibility
- [2026-11-12] [T02] Pearson VUE online proctoring is branded "OnVUE" (user-confirmed), not "OneVue" | src: user | status: verified

### T05 OnVUE control surface (full: slices/T05-onvue-controls/output.md)
- [2026-11-12] [T05] Secure browser = Pearson VUE Browser Lock (app `proctorapp`); blocks other software + app-switch keys during exam | src: pearsonvue onvue-technical-requirements.pdf | status: verified(primary)
- [2026-11-12] [T05] VM detected → BLOCKS; error "A virtual machine has been detected"; detection METHOD undocumented | src: pearsonvue requirements.html; nashtech blog | status: verified(primary)/method-unknown
- [2026-11-12] [T05] Hyper-V/VirtualMachinePlatform/WSL/Docker can trigger VM block even with no active VM | src: blog.nashtechglobal.com | status: third-party/anecdotal
- [2026-11-12] [T05] VBS/HVCI/nested-virtualization/Type-1 bare-metal hypervisor handling NOT documented | src: vendor silent | status: UNKNOWN
- [2026-11-12] [T05] Single display required; launch-time hard-block vs proctor visual catch UNVERIFIED | src: pearsonvue requirements.html/PDF | status: primary/mechanism-unverified
- [2026-11-12] [T05] Remote-control + screen-share/mirror tools BLOCKED; per-protocol RDP/VNC detection undocumented | src: pearson PDF; opengroup help | status: primary+third-party
- [2026-11-12] [T05] Screen capture/overlay blocked; accessibility apps allowed ONLY with granted accommodation | src: pearson PDF | status: verified(primary)
- [2026-11-12] [T05] Virtual-camera/virtual-audio technical detection UNKNOWN; policy bans "filters" only; analog research missed OBS vcam | src: pearson PDF; SETU showcase.itcarlow.ie | status: UNKNOWN/third-party
- [2026-11-12] [T05] KEY: proctor monitors webcam video + mic audio + telemetry only; candidate SCREEN NOT captured/streamed | src: pearsonvue privacy policy; comptia bans page | status: verified(primary)
- [2026-11-12] [T05] Linux/Unix/ChromeOS UNSUPPORTED; Win 10/11 64-bit (no S Mode), macOS 14+ only | src: pearson PDF | status: verified(primary)
- [2026-11-12] [T05] Admin-rights requirement claimed by third party, absent from Pearson docs | src: help.opengroup.org | status: UNVERIFIED
- [2026-11-12] [T05] No independent binary teardown of OnVUE client found; internals rest on vendor text + candidate reports | src: T05 §gaps | status: limitation-noted

### T06 Persona A red-team inventory (full + 20-technique table: slices/T06-persona-a-redteam/output.md)
- [2026-11-12] [T06] ANSWER (Persona A primary): no-VM local tooling — offline local LLM + indexed bank, accessibility/UIA or OCR read of exam, non-activating topmost overlay on PRIMARY display, staged into memory pre-launch | src: slices/T06 | status: analysis
- [2026-11-12] [T06] Why it wins: sidesteps VM detection (OnVUE's only hard-BLOCK), no network, gaze stays on-screen, lives in 3 blind spots (screen unobserved, process-scan internals undocumented, focus-only lockdown) | src: slices/T06 §exec | status: analysis
- [2026-11-12] [T06] Binding constraints a defender should target: (a) helper surviving the process scan, (b) overlay beating exclusive-fullscreen / client z-order | src: slices/T06 | status: analysis
- [2026-11-12] [T06] Secondary (higher skill): stealthed Type-1 hypervisor + GPU passthrough, OnVUE as hardened guest; signature VM-tells beatable, TIMING red-pills remain the unsolved hard part | src: slices/T06 #7-8; BlackHat EU2020 D'Elia | status: analysis+research
- [2026-11-12] [T06] CLOSED paths: casual VM/WSL/Hyper-V (over-blocked), audio/TTS delivery (always-on mic), loopback remote-desktop (category block), 2nd display/device (out of scope + flagged) | src: slices/T06 | status: analysis
- [2026-11-12] [T06] Top defender signals: (1) timing-based hypervisor probe, (2) continuous all-session process + capture-API scan, (3) top-of-z-order overlay audit, (4) assistive-tech client-attach detection + host-load telemetry | src: slices/T06 §defender | status: analysis
- [2026-11-12] [T06] Caveat: no OnVUE binary teardown exists; virtual-cam detection, process-scan method, VBS/HVCI handling, proctorapp fullscreen mode remain UNKNOWN and drive feasibility ratings | src: slices/T06; T05 §gaps | status: limitation

## G2 Alternatives
### T07 Vendor control inventory (full + 51 sources: slices/T07-vendor-alternatives/output.md)
- [2026-11-12] [T07] Decisive factor = architecture: a lockdown-browser/extension structurally cannot see a VM, remote session, or capture-excluded overlay; only a native host agent with continuous process+overlay+display enumeration closes the gap | src: slices/T07 [S29][S32] | status: analysis
- [2026-11-12] [T07] Strongest native host-integrity: ProctorU/Meazure Guardian (logs all processes, re-checks ~30s, scans 400+ apps for VM/remote tools) | src: slices/T07 [S15][S16] | status: verified(vendor)
- [2026-11-12] [T07] Rosalyn/StableSight: native app, deep process monitoring + typing-sound correlation + explicit screen-visual detection of another app | src: slices/T07 [S27][S28] | status: verified(vendor)
- [2026-11-12] [T07] Constructor/Examus: secure browser with auto VM detection + explicit overlay blocking + active-window-change flag | src: slices/T07 [S23][S24] | status: verified(vendor)
- [2026-11-12] [T07] Prometric ProProctor blocks VM + Administrator-role machine; Respondus/Kryterion block VMs incl virtual displays but have documented evasions | src: slices/T07 [S10][S11][S20][S31][S32] | status: verified+evasions-noted
- [2026-11-12] [T07] Weakest for this threat: Proctorio (needs Companion App), Honorlock (extension), Examity (host integrity undocumented), ProctorExam (pixels-only) | src: slices/T07 | status: verified(vendor)
- [2026-11-12] [T07] RESIDUAL GAP (a): NO vendor uses hardware attestation (TPM/Secure Boot/measured boot) or Type-1 bare-metal hypervisor detection; all VM detection is software heuristic, defeatable by tuned/bare-metal guest | src: slices/T07 [S46][S31] | status: verified(absence)
- [2026-11-12] [T07] RESIDUAL GAP (b): capture-excluded AI overlays (WDA_EXCLUDEFROMCAPTURE / macOS window-privacy) reachable only via OS process/overlay/GPU/network signals; strongest vendors reduce but do not eliminate | src: slices/T07 [S32] | status: verified
- [2026-11-12] [T07] OnVUE baseline = native secure browser (mid-tier) but the user's deployment disables continuous screen/host monitoring — forfeits the one layer that catches a workspace opened AFTER launch | src: slices/T07 [S33][S34] + Goals.MD | status: analysis

## G3 Bypass test design
### T09 Owner-run detection test designs (full: slices/T09-bypass-tests/output.md) — DESIGN ONLY, not run
- [2026-11-12] [T09] 14 tests: 12 treatments (T06 High/Med items) + 2 negative controls (casual VM, loopback RDP, predicted caught) | src: slices/T09 | status: design
- [2026-11-12] [T09] Each test: hypothesis, setup, control-vs-treatment method, detection signal tied to T05 behavior, explicit PASS(=detects)/FAIL(=evades), Win/macOS delta, ethics guardrails, labeled prediction | src: slices/T09 | status: design
- [2026-11-12] [T09] Predictions: VM/display-adapter families → PASS(caught); overlay/local-LLM/pre-staging/a11y-read → FAIL(evade) — matching the T05 blind spots | src: slices/T09 | status: prediction (nothing run)
- [2026-11-12] [T09] Execution requires user approval + practice/non-live exam + consenting testers + Pearson/sponsor coordination + ToS/legal review | src: slices/T09 | status: guardrail
