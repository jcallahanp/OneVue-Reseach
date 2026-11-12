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

## G3 Bypass test design
