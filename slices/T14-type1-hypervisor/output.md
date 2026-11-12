# T14 Type-1 bare-metal hypervisor — threat + prevention (center-owned machines)

Scope: DEFENSIVE deepening of the "stealthed Type-1 / bare-metal hypervisor, OnVUE as guest" strategy (T06 #7–8) under the clarified deployment: a **test center with center-owned Windows machines** (macOS differences flagged), likely **low compute**, endpoints currently poorly locked down **but owned and hardenable** by the certification owner; a **4-hour on-site exam**, **one proctor for many screens**, candidate may bring **USB media**. Concern model: a Type-1 hypervisor runs first, OnVUE runs as a **guest**, the answer tool runs in the **host/sibling partition**, invisible to OnVUE across the VM boundary. Owner's PRIMARY need is **PREVENTION**. This document is recognition + feasibility + countermeasure level: it names publicly-known tools so the defender can recognize and block them. It is **not** an install-and-evade tutorial and contains no configs or commands for running OnVUE as a concealed guest. All access dates **2026-11-12**. Grounding: T05 (OnVUE control surface) and T06 (red-team inventory).

---

## The attack chain (so each control maps to a step)

A concealed-guest attack must complete **all** of these; break any one and it fails:

- **S1 — Gain boot/disk control.** Either (a) **reimage the disk** (needs to boot an installer, normally from USB or PXE), or (b) **boot a live/pre-built hypervisor from external USB**, or (c) **enable a built-in Windows hypervisor** (Hyper-V / VBS) that needs no boot control.
- **S2 — Run a hypervisor beneath the OS** so OnVUE can be placed in a guest.
- **S3 — Place OnVUE in the guest and the answer tool in the host or a sibling partition** (the isolation that defeats OnVUE's in-guest process scan, T05 §3).
- **S4 — Harden the guest** so OnVUE's VM detection (T05 §4–5) does not fire.
- **(S0 — Staging/media):** bring tooling/answer-bank on USB; pre-stage a helper before OnVUE launches (T06 #3).

Key asymmetry from T05 §5: route **S1(c)** — the *easiest* route, enabling Hyper-V/VBS with no boot control — is **self-defeating**, because OnVUE over-blocks Hyper-V / VirtualMachinePlatform / WSL and trips the "virtual machine detected" hard block even with no active VM. So a *concealed* guest realistically needs **S1(a) reimage or S1(b) USB-boot**, both of which require **boot/disk control that firmware lockdown denies**. That is the owner's decisive lever.

---

## 1. Available Type-1 hypervisors

"Install model" is given at the level a defender needs to recognize it. Overhead is relative to a **low-compute center desktop** (assume ~8 GB RAM, integrated GPU). "OnVUE-as-guest realistic?" judges the *full* concealed-guest goal (guest + host-side answer tool, hardened against VM detection) on that hardware.

| Tool | Install model | Approx. setup time (from scratch, on-site) | Overhead on low-compute | OnVUE-as-guest realistic? |
|---|---|---|---|---|
| **Microsoft Hyper-V** (built into Win Pro/Ent/Edu) | **Enable a built-in feature** — no boot media, no reimage; Windows becomes the root partition under the MS hypervisor at ring −1 [S9,S10] | Minutes (enable + reboot) | Low to enable; a *nested guest* Windows adds full guest cost | **No** — self-defeating: OnVUE over-blocks Hyper-V/VBS (T05 §5); and in the default model OnVUE lands in the **root** partition, not isolated from a sibling, so concealment needs a detected child-VM anyway |
| **VMware ESXi** (free tier historically) | **Reimage / dedicate the disk**, boot installer from USB; thin Type-1, 8 GB boot RAM minimum, 8 GB min to run VMs [S3,S4,S5] | ~20–45 min + guest install | High: 8 GB floor just to boot the hypervisor leaves ~nothing for a Win guest + answer tool on an 8 GB box | **Marginal/No** on low compute; ESXi bootloader is **signed** (Secure Boot may *permit* it — a double-edged residual) |
| **Proxmox VE** (KVM-based, Debian) | **Reimage / dedicate the disk**, boot ISO from USB; needs VT-x/AMD-V in firmware; ~2 GB + guest RAM [S1,S2] | ~20–40 min + guest install | Medium-high once a Win11 guest (MS floor 4 GB/64 GB) is added | **Marginal** on low compute; viable only with real prep and more RAM |
| **Xen Project / QubesOS** (Xen-based) | Xen loads via a **Linux dom0** (reimage/dedicate disk) [S7]; **QubesOS** is a bare-metal Xen install requiring **VT-x+EPT and VT-d** [S8] | ~30–60 min + guest install | High; Qubes explicitly wants VT-d and a fast SSD — rarely present on low-end office desktops | **Marginal/No** on low compute; strongest *isolation* model but heaviest to stand up |
| **KVM on a Linux host** (Proxmox is the packaged form) | Install Linux + enable KVM (reimage/dedicate disk); needs VT-x/AMD-V | ~30–60 min | Medium-high with a Win guest | **Marginal** — same as Proxmox |
| **bhyve** (FreeBSD) | Install FreeBSD host, run bhyve (reimage/dedicate disk); Windows-guest support is newer/niche — *general knowledge, unverified for this exact use* | ~45–90 min | High | **No, practically** — niche Windows-guest path, weak tooling for this scenario |
| **Bareflank-class / "from-scratch" academic (e.g. Bareflank)** | A thin hypervisor that boots the existing OS into a guest *in place*; Bareflank README: **64-bit, no BIOS support, UEFI Type-1 was a 2016–2017 roadmap item** — research scaffolding, not a turnkey product [S11] | Days–weeks of engineering (not an on-site task) | Low runtime, but enormous build/prep cost | **No for a walk-in**; **Yes only as pre-built R&D** — this is the class that could, in principle, hoist a running Windows+OnVUE into a guest while masking signatures. The realistic worst case is pre-built (Section 3) |
| **Jailhouse** (Siemens, partitioning) | Loaded by a booted Linux via a kernel module; statically partitions cores/devices; "mixture of Type-1 and Type-2" [S12,S13] | ~30–60 min + heavy config | Low-overhead by design, but real-time/embedded-oriented, not for hiding a Windows desktop guest | **No** — wrong tool; no general Windows-guest desktop story |

Reading of the table: the hypervisors that are **easy to stand up** (Hyper-V/VBS built-in) are exactly the ones **OnVUE catches**; the ones that give **clean isolation** (Xen/Qubes/ESXi/Proxmox) **require boot/disk control and real RAM**, which both firmware lockdown and low-compute hardware deny; and the one class that could truly conceal a hardened guest (**Bareflank-class, pre-built**) is **not a walk-in build** — it is prior R&D delivered on media.

---

## 2. Feasibility verdict on "install within ~1h on an unlocked machine" (bounded, honest)

**Verdict: the *bare* install is plausible in ~1 h on an unlocked machine; the *concealed, OnVUE-evading* install is NOT a within-the-hour walk-in task. The realistic threat is a *pre-built* image (Section 3), not on-site construction.** Bounded by the three gates the user asked about:

**(a) Does install require boot/disk control that firmware lockdown would deny? — YES, for any concealing route.**
- Reimage (ESXi/Proxmox/Xen/Qubes) needs to **boot an installer** (USB or PXE) and **write the disk**. Booting a **pre-built hypervisor from USB** needs **external boot enabled**. Both are denied by firmware lockdown (admin password + disable USB/PXE boot + locked boot order; Section 5, controls 1–4).
- The only no-boot-control route is **enabling Hyper-V/VBS** — but that is **self-defeating** (OnVUE over-blocks it; T05 §5) and does not isolate OnVUE from a sibling by default. So **on an unlocked machine the attack is feasible; on a firmware-locked machine the concealing routes are closed before they start.** This is the single most important finding for the owner.

**(b) Is a naive Type-1 still CAUGHT by OnVUE, so only a HARDENED guest evades — and is hardening a within-the-hour task? — naive is CAUGHT; hardening is an ARMS RACE, not a 1-hour task.**
- A **naive** Type-1 guest still presents VM tells: CPUID hypervisor-present bit, SMBIOS/ACPI OEM strings, paravirtual drivers, virtual NIC OUI / disk vendor strings — exactly the trigger set behind OnVUE's documented "a virtual machine has been detected" hard block (T05 §4). **So a quick naive install does not achieve concealment.**
- Defeating **signature** tells is achievable for a prepared attacker, but T06 #7 and the literature treat **timing-based** detection (CPUID VM-exit latency ≈ hundreds of cycles native vs thousands under a VM; RDTSC smoothing) as the **hard, unsolved part** [T06 S1 D'Elia BH-EU 2020; T06 S2 Rossow DIMVA]. That hardening is **prior R&D on a dedicated prepared build**, not something assembled on-site in the first hour of the exam.

**(c) Compute overhead on low-end machines — a natural barrier.**
- ESXi needs **8 GB just to boot** (and 8 GB min to run VMs) [S3,S4]; a Windows 11 guest's own Microsoft floor is **4 GB RAM / 64 GB disk**; the **host-side answer tool** (a local LLM is heavy — T06 #2 flags sustained CPU/GPU/thermal load as a tell) wants more still. On an ~8 GB integrated-GPU office desktop, a smooth **4-hour** exam in a nested Windows guest **plus** host-side inference is **marginal to infeasible**, and GPU passthrough (T06 #8) needs **VT-d/IOMMU** and typically a spare/second GPU rarely present on such hardware. *This paragraph is bounded analysis from the sourced minimums, not a vendor claim.*

**Net:** "student walks in and has a concealed Type-1 up in ~1 hour, not resource-heavy" is **overstated** for an on-site build: it is gated by boot control (closable by the owner), by an unsolved VM-detection arms race, and by compute. The honest worst case is **a prepared student with a pre-built image** — addressed next.

---

## 3. Pre-built-USB worst case

**This is the realistic worst case the owner should design against.** A prepared candidate does all the hard work at home — build a hardened hypervisor + Windows-guest image, tune it against known VM tells, embed the answer tool in the host/sibling partition — then on-site simply **boots that image from USB in minutes**. The 1-hour "install" collapses to a few-minute "boot," because nothing is being built on-site.

Assessment:
- **Enabler:** this works **only if external boot is permitted** (USB boot enabled, boot order not locked, and Secure Boot not blocking the image's bootloader). Remove external boot and this entire worst case is denied at the firmware layer (Section 5, controls 1–4), regardless of how good the image is.
- **Residual even if it boots:** (i) it still faces OnVUE's **undocumented** VM detection — unknown whether OnVUE's specific checks (possibly timing; T05 §5 UNKNOWN) catch that particular hardened build; (ii) **low-compute** center hardware may not run guest+host-tool acceptably for 4 hours; (iii) a **supervised cold boot** into the center's verified image (Section 5, control 12) under observation denies the chance to boot foreign media at all.
- **Signed-image nuance:** a pre-built **ESXi** image carries a **signed** bootloader, and some Linux shims are Microsoft-signed; with Secure Boot on but **no admin-password lock on firmware**, a candidate could enroll their own key (MOK) or the signed loader could be *permitted*. So Secure Boot alone is **necessary but not sufficient** — it must be paired with the firmware admin password and disabled external boot (controls 1–3) [S14,S15].

Bottom line: the pre-built USB is the scenario to beat, and **it is beaten decisively by owner-controlled firmware boot lockdown plus a supervised reboot into a verified image** — none of which a BYOD proctoring deployment could enforce.

---

## 4. Would OnVUE detect it? (catches vs unknown vs residual)

From T05/T06, strictly bounded:

**CATCHES (documented / corroborated):**
- **Naive VM / hypervisor signatures** — the "**a virtual machine has been detected**" hard **BLOCK** (T05 §4): the one documented, decisive host-integrity control. Triggered by CPUID hypervisor bit, paravirt drivers, SMBIOS/device strings, leftover virtual NIC/disk artifacts.
- **Built-in Windows virtualization surfaces** — OnVUE **over-blocks** Hyper-V / VirtualMachinePlatform / WSL / Docker, tripping the VM block even with no active VM (T05 §5). This paradoxically closes the easy S1(c) route.

**UNKNOWN (undocumented; no primary source):**
- Whether a **well-hardened Type-1 guest** (all signature tells masked) is detected at all.
- Whether OnVUE performs **timing-based** hypervisor detection (the only reliable catch for a well-hardened guest; T06 #7). **Not verified** in any source.
- **VBS / HVCI / nested-virtualization** handling (T05 §5 UNKNOWN).

**RESIDUAL (what slips through / why not to rely on OnVUE):**
- A signature-clean, timing-smoothed guest **could** evade OnVUE's VM block; we **cannot verify** OnVUE closes this. Therefore **do not treat OnVUE's VM detection as the primary control** — it is a **backstop**. Because the center **owns the machines**, the load should sit on **owner-side firmware/boot lockdown** (Section 5), which does not depend on OnVUE's undocumented internals and closes the attack **before** OnVUE ever runs.

---

## 5. PREVENTION checklist (control → attack step broken → residual → needs-ownership)

Priority order is roughly top-down. "Needs ownership" is the decisive point: **every control below requires owning/managing the machine** — a BYOD proctoring model (candidate's own PC) can enforce almost none of them, which is exactly why a center-owned deployment is far more defensible.

| # | Control | Attack step broken | Residual gap | Needs machine ownership? |
|---|---|---|---|---|
| 1 | **BIOS/UEFI admin (supervisor) password** [S6] | Protects S1/S2/S4 by locking firmware: boot order, external-boot toggles, VT-x/VT-d toggles can't be changed | On some models (documented on Dell) an admin password did **not** block the **one-time UEFI boot menu** — booting USB still worked; **test per model** [S6b]. CMOS-reset/jumper clears it → control 14 | **Yes** — only the owner sets firmware passwords |
| 2 | **Enable Secure Boot** [S14,S15] | S1(b)/S2: unsigned live-USB/hypervisor bootloaders refuse to boot | **Signed** images (ESXi, MS-signed shims) may still be *permitted*; MOK enrollment possible **unless** firmware is locked (control 1). Doesn't stop a reimage with a signed OS | **Yes** |
| 3 | **Disable USB boot + PXE/network boot** (all external boot) [S6] | **S1(a)/S1(b): the single most important lever** — denies the reimage installer **and** the pre-built-USB worst case (Section 3) | Only durable if locked behind control 1; internal reimage by other means still conceivable (controls 5,9,11) | **Yes** |
| 4 | **Lock / password the boot order** [S6] | Reinforces 3: candidate can't reorder to external media | Same admin-password dependency as 1; per-model boot-menu caveat (control 1 residual) | **Yes** |
| 5 | **Full-disk encryption (BitLocker + TPM)** [S16] | S1(a): can't quietly reimage-in-place or alter the OS offline without triggering recovery; protects the verified image at rest | Protects at-rest/measured-boot integrity, **not** a clean external boot of a *different* disk/USB (that's 2/3/6). Default PCR profile (**4,7,11** since Jul 2024) may **not** catch a pure boot-order change — add **TPM+PIN** and rely on controls 1/3 [S16]; recovery-key ops overhead | **Yes** |
| 6 | **Disable hardware virtualization in firmware (VT-x/AMD-V + VT-d)** where the exam doesn't need it [S17,S18] | **S2/S4 directly:** no VT-x ⇒ no hardware-assisted Type-1 hypervisor can run; no VT-d ⇒ no GPU/device passthrough (T06 #8). OnVUE itself needs none of it | Must be locked behind control 1 or it's re-enabled; Windows VBS/Credential Guard wants VT-x — but VBS is itself a hypervisor OnVUE over-blocks (T05 §5), so disabling is consistent | **Yes — a decisive owner-only lever; BYOD cannot do this** |
| 7 | **Disable Hyper-V / VirtualMachinePlatform / WSL / Sandbox in the OS image** [S9,S10] | S1(c)/S2: removes the built-in hypervisor route and the VBS ambiguity; also removes the components that cause OnVUE VM false-positives (T05 §5) | Doesn't stop an **external** hypervisor (2/3/6); needs standard-user enforcement (control 11) so it can't be re-enabled | **Yes** (control the image) |
| 8 | **USB port control / device allow-listing** (block USB mass storage; allow only keyboard/mouse/approved webcam) [S19,S20] | S0: no runtime load of a staged helper/answer-bank from USB; reinforces 3 against media the candidate brings | OS-level block doesn't stop **pre-OS** USB boot (needs firmware control 3); VID/PID/serial allow-listing is fiddly/buggy in practice; **HID-emulating "BadUSB"** bypasses a mass-storage block | **Yes** |
| 9 | **Verified golden imaging + reset between sessions** (reimage / snapshot / Deep-Freeze) [S21,S23] | S0/persistence: wipes any hypervisor install or staged helper a prior candidate left; guarantees known-clean state each session | Doesn't stop a **live USB boot this session** (control 3) or an **in-memory** helper staged this session (needs continuous process scan — T06 #3 signal); reset cadence/compute cost | **Yes — the defining owned-vs-BYOD advantage** |
| 10 | **TPM measured boot / attestation** [S16,S22] | Detects S1/S2 tampering: firmware/bootloader/early-OS changes shift PCRs; with BitLocker a tampered chain fails to unseal | **Detects, does not prevent**; default PCR set may miss boot-order-only changes; remote-attestation infra is heavier to stand up; OnVUE does not consume host TPM attestation (not documented) | **Yes** |
| 11 | **Standard-user exam account + app allow-listing (WDAC/AppLocker)** | S2/S3: can't install a hypervisor, enable Windows features, or run an unsigned answer tool inside the OS; reinforces 7/8 | Doesn't stop the **external-boot** route (below the OS entirely → 2/3/6); kernel-exploit edge cases | **Yes** |
| 12 | **Supervised reboot into the verified image before exam** [S21,S24] | S0/S1/S2: ensures the machine enters the exam from the trusted boot path under observation — defeats pre-staged helpers and a hidden boot of foreign media | **One proctor for many screens** limits observation; a transparent hypervisor can look identical — pair with measured boot/attestation (10) for assurance | **Yes + process** |
| 13 | **Keep OnVUE's own VM block enabled (backstop)** (T05 §4) | S4: catches **naive** VMs — defense-in-depth | **Large residual:** hardened-Type-1 handling UNKNOWN, timing detection unverified (Section 4). Vendor-controlled, not owner-tunable — **do not rely on it** | **No** (vendor control — which is why owner-side 1–12 must carry the load) |
| 14 | **Physical / BIOS tamper controls** (chassis lock, port blockers, tamper-evident seals, block CMOS-reset access) [S6] | Protects controls 1–6: prevents clearing the admin password or physically adding a boot device | A determined physical attacker with a long, lightly-supervised **4-hour** window is a real gap; raises effort/visibility rather than eliminating | **Yes** |

**macOS differences (center-owned Macs; Windows is primary).** The Type-1-beneath-OS concern is **structurally weaker** on macOS and the controls map differently — *the following is general platform knowledge, label unverified where no source is cited*:
- Apple Silicon has **no VT-x/AMD-V**; virtualization uses Apple's Hypervisor/Virtualization.framework (hosted) and can host only **ARM** Windows, which OnVUE does not target — so the "pre-built Windows guest on a Mac" worst case largely does not exist.
- Boot lockdown is via **Startup Security Utility** (Intel T2 Macs: set **Full Security** + **disallow booting from external media**) or Apple-Silicon recovery security policy, not a BIOS menu; use **FileVault** for FDE and a **firmware password** on Intel Macs; enforce via **Apple MDM / Supervised** mode.
- OnVUE runs **natively** on macOS (T05 §11), so keep controls 8–12 (device control, golden image, standard user, supervised reboot) and run **parity tests** so the Mac fleet is not the soft target (T06 #11).

---

## 6. If you do only 3 things

1. **Lock the firmware and kill external boot.** Set a **BIOS/UEFI admin password** (controls 1,4), **disable USB boot + PXE/network boot** and **lock the boot order** (control 3), and **enable Secure Boot** (control 2). This alone closes both realistic sub-1-hour paths — the **reimage installer** and the **pre-built-USB worst case** (Section 3). **Test the per-model UEFI boot-menu caveat** (control 1 residual) on your exact hardware.
2. **Turn off virtualization the exam doesn't need.** Disable **VT-x/AMD-V + VT-d** in firmware (control 6, locked behind the admin password) and **Hyper-V/VBS/VirtualMachinePlatform/WSL** in the OS image (control 7). Without hardware virtualization **no hardware Type-1 hypervisor or GPU passthrough can run at all** — an owner-only lever a BYOD exam can never pull.
3. **Boot every session from a verified golden image under a supervised reboot,** on a **standard-user** account with app allow-listing, and **reset between candidates** (controls 9,11,12). This guarantees a known-clean state, blocks in-OS install/staging, and is the defining **owned-vs-BYOD** advantage.

These three collapse the threat from "feasible with preparation" to "requires defeating firmware the owner controls," and correctly relegate OnVUE's (undocumented, unreliable-against-hardened-guests) VM detection to a **backstop** rather than the primary control.

---

## Sources (numbered)

Internal grounding:
- **T05** — OnVUE control surface: `/home/user/OneVue-Reseach/slices/T05-onvue-controls/output.md` (cited as T05 §n) — 2026-11-12.
- **T06** — Persona A red-team inventory: `/home/user/OneVue-Reseach/slices/T06-persona-a-redteam/output.md` (cited as T06 #n; its VM-evasion sources S1 D'Elia BH-EU 2020, S2 Rossow DIMVA) — 2026-11-12.

Hypervisor projects (install model / requirements):
1. Proxmox VE bare-metal install (CPU needs Intel VT / AMD-V; ~2 GB + guest RAM; ISO to USB). IONOS — https://www.ionos.com/digitalguide/server/configuration/proxmox-bare-metal-installation/ — 2026-11-12 — THIRD-PARTY (official Proxmox docs not directly fetched; treat as secondary).
2. Proxmox install beginners guide (USB media, checksum, installer flow). VirtualizationHowto — https://www.virtualizationhowto.com/2024/05/proxmox-install-beginners-guide — 2026-11-12 — THIRD-PARTY.
3. vSphere/ESXi 8 hardware requirements (boot RAM raised 4→8 GB; 8 GB min to run VMs). Vladan.fr — https://www.vladan.fr/vcp-dcv-on-vsphere-8-x-identify-the-pre-requisites-and-components-for-a-vsphere-implementation — 2026-11-12 — THIRD-PARTY.
4. ESXi 8 boot-device size + SD/USB guidance (KB 85685; ~8 GB USB/SD, 32 GB other). Broadcom KB — https://knowledge.broadcom.com/external/article/313156/guidance-on-minimum-boot-device-size-req.html — 2026-11-12 — PRIMARY (vendor).
5. Install vSphere ESXi 8 on bare-metal (USB boot, installer steps). ComputingForGeeks — https://computingforgeeks.com/install-vsphere-esxi-8-on-bare-metal-servers/ — 2026-11-12 — THIRD-PARTY.
6. NSA/CISA — UEFI Lockdown / UEFI Secure Boot Customization guidance (admin/user password, disable unused boot options + ports, Secure Boot, TPM, block non-admin changes, verify boot order). https://media.defense.gov/2019/Jul/16/2002158050/-1/-1/0/CSI-UEFI-LOCKDOWN.PDF — 2026-11-12 — PRIMARY (government).
   - 6b. Dell community thread — admin password on boot menu applied only in Legacy mode; in UEFI mode USB boot still possible (per-model caveat). https://www.dell.com/community/Desktops-General-Read-Only/Optiplex-790-with-BIOS-A13-UEFI-boot-security-issue/m-p/3872438 — 2026-11-12 — THIRD-PARTY/anecdote — label: unverified-per-model.
7. Xen Project handbook — Getting Started (Xen installed via a Linux distro; loaded by the bootloader; Linux boots as dom0; ≥4 GB total). https://handbook.xenproject.org/users/getting-started.html — 2026-11-12 — PRIMARY (vendor).
8. Qubes OS — System requirements (bare-metal Xen; requires 64-bit CPU with VT-x+EPT and VT-d; installing Qubes in a VM not recommended). https://www.qubes-os.org/doc/system-requirements — 2026-11-12 — PRIMARY (vendor).
9. TechTarget — What is Hyper-V on Windows 11 (Type-1/bare-metal; enabling makes Windows the root partition; enable via Optional Features + reboot; needs firmware virtualization). https://www.techtarget.com/enterprise-software/tip/What-is-Hyper-V-on-Windows-11-and-what-can-it-do — 2026-11-12 — THIRD-PARTY.
10. Petri — Understanding the Architecture of the Hyper-V Hypervisor (hardware → MS hypervisor → root/child partitions; VBS runs the hypervisor beneath Windows at ring −1 even with no VM). https://petri.com/hyper-v-hypervisor-architecture — 2026-11-12 — THIRD-PARTY.
11. Bareflank Hypervisor (README: C++ scaffolding to build any hypervisor type; 64-bit, no BIOS, LGPL; UEFI Type-1 was a 2016–2017 roadmap item — dated). https://github.com/Bareflank/hypervisor — 2026-11-12 — PRIMARY (project) — label: dated/roadmap.
12. Jailhouse (Siemens) — partitioning hypervisor based on Linux; statically partitioning, non-scheduling. https://github.com/siemens/jailhouse — 2026-11-12 — PRIMARY (project).
13. "Look Mum, no VM Exits! (Almost)" — Jailhouse as a mixture of Type-1/Type-2 (Linux loads it, kernel module enables it). arXiv 1705.06932 — https://arxiv.org/abs/1705.06932 — 2026-11-12 — PRIMARY (research).

Secure Boot / boot-media control:
14. Kicksecure — Dev/Secure Boot (Secure Boot refuses unsigned/unauthorized bootloaders; unsigned live media won't boot natively; MOK enrollment path). https://www.kicksecure.com/wiki/Dev/Secure_Boot — 2026-11-12 — THIRD-PARTY.
15. liveng — UEFI Secure Boot (unsigned live USB blocked; some ISOs lack signed bootloader). https://liveng.readthedocs.io/en/latest/secure-boot.html — 2026-11-12 — THIRD-PARTY.

BitLocker / TPM measured boot:
16. Microsoft Learn — BitLocker countermeasures (Secure Boot integrity via TPM PCR[7]; default UEFI profile changed to PCRs 4,7,11 from Jul 2024; TPM+PIN recommended for advanced attacks; measured boot detects, not prevents). https://learn.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-countermeasures — 2026-11-12 — PRIMARY (vendor).

Disable virtualization in firmware:
17. ElevenForum — Enable/Disable CPU Virtualization in UEFI BIOS (VT-x / Intel Virtualization Technology; SVM Mode / AMD-V; set Disabled; re-check after BIOS reset). https://www.elevenforum.com/t/enable-or-disable-cpu-virtualization-in-uefi-bios-firmware-settings-on-windows-pc.4928/ — 2026-11-12 — THIRD-PARTY.
18. GeekRewind — Enable/Disable CPU virtualization in UEFI BIOS (VT-x vs VT-d distinction; VT-d = I/O virtualization / passthrough). https://geekrewind.com/how-to-enable-or-disable-cpu-virtualization-in-uefi-bios-on-windows-pc/ — 2026-11-12 — THIRD-PARTY.

USB device control:
19. Microsoft Learn (Intune Q&A) — Block removable USB storage via Intune (Removable storage = Block; already-used devices caveat). https://learn.microsoft.com/en-us/answers/questions/1157448/block-removeable-usb-storage-via-intune — 2026-11-12 — PRIMARY (vendor).
20. Practical365 — Device Control in Defender for Endpoint to block USB drives (Removable Storage Access = Block; VID/PID/serial exclusions). https://practical365.com/using-device-control-in-defender-for-endpoint-to-block-usb-drives/ — 2026-11-12 — THIRD-PARTY.

Exam-station / kiosk hardening + reset:
21. OPSWAT MetaDefender Kiosk hardening (BIOS passwords, TPM + Secure Boot, BitLocker, disable autorun). https://www.opswat.com/docs/mdkiosk/v4.8.5/hardened-image/kiosk-hardening — 2026-11-12 — THIRD-PARTY (vendor docs).
22. Cyberus — NixOS for programming-exam environments (used **Secure Boot to stop student-supplied OSes from booting**; lanzaboote/MOK). https://cyberus-technology.de/post/nixos-for-programming-exam-environments — 2026-11-12 — THIRD-PARTY.
23. GSE — Exam computer lab speed reset / control (restore from system image, freeze states/snapshots, Deep Freeze between sessions). https://www.gse.kz/en/blog/exam-computer-lab-speed-reset-control — 2026-11-12 — THIRD-PARTY.
24. Prolydian SEAL — bootable-USB OS that turns a PC into a test kiosk without installing on the machine (illustrates both the attack-analog and a reset model). https://www.prolydian.com/node/41 — 2026-11-12 — THIRD-PARTY (vendor).
