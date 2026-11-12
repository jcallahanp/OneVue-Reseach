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
