# T11 Vendor claim re-verification

Scope: targeted re-check of three load-bearing claims used in the Persona B
defensive synthesis (T08). Capabilities only; no bypass content. All access dates
**2026-11-12**. Source reliability is labelled inline: **[vendor]** =
first-party marketing/docs, **[peer-reviewed]**, **[academic-3P]** = third-party
academic, **[adversarial]** = cheating-tool seller (self-interested, low trust).

---

## Claim 1 (Guardian continuous vs launch-only) — VERDICT + evidence + corrected label

**VERDICT: CONTINUOUS / PERIODIC during the exam — NOT launch-only. Confirmed in the
vendor primary source the task named; interval IS documented (every ~30s); both the
"400+" and "500+" figures are correct but refer to different scopes.**
Confidence: **HIGH that the vendor documents periodic re-enumeration; MODERATE that it
performs exactly as described** (every source is first-party Meazure/ProctorU; no
independent technical teardown of Guardian was found — see "ALSO" section).

Primary source (the one the task asked for) — Meazure Learning, *"Proctoring in Focus,
Part 1"* (2024-05-29) [vendor], verbatim:
- "the Guardian Browser blocks interaction with locally installed applications and
  **checks for unpermitted software every 30 seconds**."
- "It also **logs all active processes** on a test-taker's machine for investigation if
  a potentially threatening program is detected... Guardian will recognize it, log what
  was attempted within the session timeline, and disallow the activity."
- "Our proctoring system **scans for over 400 applications to detect virtual machines
  and remote access tools**."

Corroborating first-party sources (consistent across 4+ pages):
- *"A Closer Look at the Guardian Browser"* (2023-04-25) [vendor]: "Guardian detects
  **over 500 computer processes and applications** and stops them from running during an
  exam"; "It runs a **continuous script** to detect and deny... multiple monitors...
  virtual machines, and... unpermitted applications **during an exam**"; "These
  processes and applications are **only blocked when a test-taker is taking an exam**,
  not while the browser is open... outside of the testing process."
- Guardian Browser product page (2023-03-09) [vendor]: "**performs continuous checks
  against those settings throughout the entire exam**"; contrasts the old extension
  ("as a step in the exam launch process, the system checks and terminates") against
  Guardian ("the system **will continuously check** which applications and background
  programs are running and terminate any that are not allowed"). This explicitly frames
  the launch-only→continuous change as the product's differentiator.
- ProctorU Support, *"Unpermitted Applications"* [vendor]: "Guardian Browser **monitors
  hundreds of additional processes and applications**; the full denylist is updated over
  time."

**Reconciling 400+ vs 500+ (the discrepancy is resolved, not an error):**
- **500+** = total process/application blocklist ("detects over 500 computer processes
  and applications").
- **400+** = the subset specifically targeting **VM + remote-access** detection ("scans
  for over 400 applications to detect virtual machines and remote access tools").
- They are nested scopes, not conflicting claims. Both are vendor figures, neither
  independently audited.

**Corrected label to apply:** replace any "launch-only / ambiguous 400+" wording with:
*"Guardian performs **periodic re-enumeration every ~30s plus continuous all-process
logging** throughout the exam (vendor-documented, not independently verified). Blocklist
is **500+ processes/apps total**, of which **400+** specifically target VMs/remote-access
tools."*

**Caveat that matters for the recommendation:** "continuous" is really **30-second
polling**, not instantaneous streaming. A persistently-running helper opened *after*
exam start is caught within one poll cycle (≤30s) — so the T08 rationale (catch a
helper/second workspace opened after launch) **holds** for any helper that stays
resident. A helper that opens and closes entirely inside a <30s gap between polls could
slip the re-scan, though the "logs all active processes" line narrows that window. State
the mechanism as "periodic (30s) re-enumeration," not literally "continuous."

---

## Claim 2 (Rosalyn / Constructor) — per-vendor corroboration status

First, a naming correction: **StableSight is Rosalyn's own detection engine** (launched
2023-11-09, BusinessWire [vendor PR]) — "Rosalyn/StableSight" is **one vendor**, not two.

**2a. Rosalyn / StableSight — "screen-visual detection that catches another app/overlay
even if it bypasses process checks": VENDOR-ASSERTED ONLY.**
- Sole source is Rosalyn's own marketing blog [vendor]: "**screen visual detection
  capabilities allow us to identify if the start menu or another app is accessed, even
  if it bypasses all the other processes running**." Also: "Remote Access Detection...
  employs AI-driven computer process analysis to **continuously detect if unauthorized
  processes**... are running."
- StableSight product page [vendor] repeats it at marketing altitude ("System Analysis:
  Deep process monitoring detects... virtual machines to unauthorized applications").
- No independent (non-Rosalyn) technical corroboration found. Third-party integration
  docs (Synap) describe Rosalyn's lockdown + AI-screening generically but do **not**
  confirm the "catches an app even if it bypasses process checks" claim.
- Status: **vendor-asserted-only.**

**2b. Constructor / Examus — "active-window-change flag": CORROBORATED (vendor docs +
independent academic reference).**
- Constructor official product docs, *"Full-screen overlays"* (updated 2026-10-15)
  [vendor-docs]: lists "**Exam page lost focus** | The mouse cursor is outside the area
  with the test" and "**Extra display detected**" as detected violations.
- Examus official blog (2020) [vendor]: auto-detects "**Change of the windows and tabs
  on the desktop**, i.e. if the user decided to switch from the test window... or open a
  third-party application," each with a timestamp.
- Independent academic description — EDM 2025 short paper [academic-3P]: describes Examus
  as using "AI-driven analysis of webcam feeds, **screen activity**, and user
  behavior... detects **application switches**," and cites Migut et al.'s screen-content-
  change (frame-similarity) approach. This is a third-party *description* of the claimed
  capability (not an efficacy test), so it corroborates that the capability is real and
  documented, not that it is robust.
- Status: **independently corroborated as a documented capability** (strongest of the
  three sub-claims).

**2c. Constructor / Examus — "overlay blocking": MISCHARACTERIZED / vendor-asserted.**
- The Constructor "overlays" in the cited doc are **full-screen WARNING overlays the
  system displays TO the test-taker** on a violation (window not full-screen, focus lost,
  extra display) — i.e. an enforcement/UX layer that covers the exam. This is **not** a
  documented capability to **detect or block a third-party capture-excluded OVERLAY
  cheating window**. The T07/T08 shorthand "blocks overlays" conflates the two.
- No vendor doc found showing Constructor/Examus detects a top-most/click-through
  third-party overlay window.
- Contra-indication [adversarial, low trust]: a bypass seller (LDBypass, 2026) claims
  Examus on macOS relies entirely on browser screen-capture that honors window-privacy
  and that "**None** [of its suspicion signals] **observe the screen window list**."
  Self-interested and unverified, but it points the same way: Examus's screen observation
  is browser-sandbox-based, not an OS-level overlay-window audit.
- Status: **vendor-asserted / mischaracterized** — "overlay blocking" should be downgraded
  to "displays full-screen violation overlays + flags active-window/focus change"; there
  is **no** substantiated third-party-overlay-window detection.

---

## Claim 3 (Simko source type) — characterization

**It is a peer-reviewed QUALITATIVE study of community-reported methods — NOT a technical
evaluation of evasion efficacy.** [peer-reviewed]

- Full cite: L. Simko, A. Hutchinson, A. Isaac, E. Fries, M. Sherr, A. J. Aviv,
  *"'Modern problems require modern solutions': Community-Developed Techniques for Online
  Exam Proctoring Evasion,"* **ACM CCS 2024**, pp. 2681–2695, DOI 10.1145/3658644.3691638
  (Barnard / George Washington Univ. / Georgetown). Presented 2024-10-14. CCS is a
  flagship, rigorously peer-reviewed security venue — so the study itself is rigorous
  **as a qualitative HCI/usable-security study**.
- Method (verbatim): "**qualitative analysis** of social media videos (n=137) and
  comments (n=4,297) on YouTube (n=17) and TikTok (n=120)." Confirmed independently by
  Barnard College news (2024-10-18) [academic-3P] and the Georgetown SecLab project page
  ("iterative analysis of thousands of TikTok and YouTube videos").
- The specific VM-evasion details (**≥4GB RAM, fixed-size VHD, registry edits**) are
  **direct quotes/paraphrases of what video creators claim**, reported by the authors —
  e.g. a creator saying "*it's possible Respondus detects the virtual machine if you have
  2G of RAM*" and "*if we use dynamically allocated disk storage space, the lockdown
  browser might detect we have a weird hard drive.*" The authors **document** these as
  community beliefs; they **did not empirically test** whether the techniques defeat
  Respondus.
- **Certainty the paper supports:** HIGH that these techniques are *being shared and
  attempted* by an organized community; **LOW** that any specific technique *actually
  defeats* Respondus VM detection. Do **not** cite Simko et al. as proof that
  registry-edit / fixed-VHD / 4GB-RAM evasion works — that is an over-read of a
  descriptive study.
- For *demonstrated* (not merely reported) Respondus evasion, the separate, weaker-venue
  works are Solis, Shashidhar & Varol (IAFOR 2023) [independent] and the DSU 2024 dynamic-
  analysis study — neither is Simko, and neither is a top-tier efficacy proof either.

---

## Does the Persona B recommendation survive? (one short paragraph)

**Yes — the core recommendation survives, with two label corrections.** The load-bearing
leg (R3: pick a native agent that **re-enumerates processes all session long**, Guardian
as primary pick) is backed by genuine vendor documentation: Guardian's **30-second
re-check + all-active-process logging** is explicitly stated across four Meazure/ProctorU
pages, and the 400+/500+ figures are both legitimate (nested scopes), so the "catch a
helper opened after launch" rationale holds for any resident helper (≤30s to detection).
Two corrections: (1) state it as **periodic (~30s) re-enumeration, vendor-asserted and
not independently verified**, not literally "continuous" and not independently proven;
(2) the **overlay-WINDOW audit leg (R4) is the weak one** — Guardian doesn't document
z-order/top-most-window auditing, Constructor's "overlay" is a *warning overlay shown to
the examinee* (not third-party-overlay detection), and Rosalyn's "screen-visual
detection" is **vendor-asserted-only**. So R4 should rest on the **general security
principle** (launch-only point-in-time checks cannot see a post-launch capture-excluded
overlay; only OS-level overlay/z-order + process + screen-capture enumeration can) paired
with re-enabled screen capture — not on any single verified product capability.

---

## Corrected labels to apply (old claim -> corrected wording/label)

- **Guardian "continuous process re-scanning (400+ or 500+?)"** -> *"Periodic
  re-enumeration **every ~30s** + continuous all-process **logging** during exam
  (vendor-documented, not independently verified). Blocklist **500+ total**, of which
  **400+** target VMs/remote-access. Confidence HIGH that documented, MODERATE that it
  performs as described."*
- **Guardian framed as possibly "launch-only"** -> *"Explicitly **not** launch-only; the
  launch-only→continuous move is the product's stated differentiator vs the old browser
  extension."* [vendor]
- **"Rosalyn / StableSight" (two vendors)** -> *"StableSight is **Rosalyn's** detection
  engine — one vendor."*
- **Rosalyn/StableSight "screen-visual detection catches an app even if it bypasses
  process checks"** -> *"**Vendor-asserted only** (sole source: Rosalyn marketing blog);
  no independent corroboration."*
- **Constructor/Examus "blocks overlays"** -> *"**Mischaracterized.** Documented behavior
  is **full-screen violation warning overlays shown to the examinee** + an
  **active-window/focus-change flag** ('Exam page lost focus'). **No** substantiated
  third-party capture-excluded-overlay detection."*
- **Constructor/Examus "active-window-change flag"** -> *"**Corroborated** — official
  Constructor docs + Examus blog + referenced in EDM 2025 academic paper (strongest
  sub-claim of Claim 2)."*
- **Simko et al. cited as evidence that VM edits "defeat" Respondus** -> *"**Qualitative,
  peer-reviewed study of community-REPORTED methods** (137 YouTube/TikTok videos, 4,297
  comments). Reports, does **not test**, the 4GB-RAM / fixed-VHD / registry-edit claims.
  Supports 'these techniques are shared/attempted,' **not** 'they work.'"*
- **"Continuous native enumeration" presented as independently proven** -> *"**Vendor-
  asserted across the board; no independent technical verification found.** Rest the
  recommendation on the general principle + Guardian as the most specifically documented
  vendor claim (named 30s interval + process logging), labelled vendor-asserted."*

---

## Sources (numbered URLs + dates; access date 2026-11-12)

1. Meazure Learning — "Proctoring in Focus, Part 1: Online Exam Security Q&A" (pub.
   2024-05-29) [vendor, PRIMARY for 30s + 400+].
   https://www.meazurelearning.com/resources/proctoring-in-focus-part-1-online-exam-security
2. Meazure Learning — "A Closer Look at the Guardian Browser: Benefits, Functions, and
   Future Plans" (pub. 2023-04-25) [vendor, 500+ + continuous script].
   https://www.meazurelearning.com/resources/a-closer-look-at-the-guardian-browser-benefits-functions-and-future-plans
3. Meazure Learning — "The Guardian Secure Proctoring Browser" product page (pub.
   2023-03-09) [vendor, "continuous checks throughout the entire exam"].
   https://www.meazurelearning.com/exam-technology/proctoru-online-proctoring/guardian-browser
4. ProctorU Support — "Unpermitted Applications" [vendor, "monitors hundreds of
   additional processes"].
   https://support.proctoru.com/hc/en-us/articles/46589198199565-Unpermitted-Applications
5. Rosalyn — "Unveiling Rosalyn.ai's Advanced Proctoring Features" [vendor, "screen
   visual detection... even if it bypasses all the other processes running"].
   https://www.rosalyn.ai/blog/unveiling-rosalyn-ais-advanced-proctoring-features-reinforcing-examination-integrity-through-cutting-edge-ai
6. Rosalyn — "Platform · Multi Modal Detection" (StableSight) [vendor].
   https://www.rosalyn.ai/platform-multi-modal-detection
7. BusinessWire — "Rosalyn Launches StableSight..." (2023-11-09) [vendor PR, StableSight =
   Rosalyn engine].
   https://www.businesswire.com/news/home/20231109705435/en/Rosalyn-Launches-StableSight-an-Exam-Proctoring-AI-That-Counters-Generative-AI-Use-and-Cheating-Rings
8. Constructor Tech — Proctor Guide, "Full-screen overlays" (doc updated 2026-10-15)
   [vendor-docs, warning overlays + "Exam page lost focus" + "Extra display detected"].
   https://docs.constructor.tech/en/proctor-proctor-guide/ai-detected-warnings-displayed-to-test-takers/full-screen-overlays
9. Examus official blog (Medium) — "Online proctoring: how it works" (2020-03-25)
   [vendor, window/tab-change detection].
   https://medium.com/examus-official-blog/online-proctoring-how-it-works-ebe9a0813abe
10. EDM 2025 short paper — "Assistant for the detection of potential cheating behavior in
    synchronous online programming exams" [academic-3P; describes Examus screen-activity /
    app-switch detection + Migut et al. screen-content-change].
    https://educationaldatamining.org/EDM2025/proceedings/2025.EDM.short-papers.20/2025.EDM.short-papers.20.pdf
11. Simko, Hutchinson, Isaac, Fries, Sherr, Aviv — "'Modern problems require modern
    solutions': Community-Developed Techniques for Online Exam Proctoring Evasion," ACM
    CCS 2024, pp. 2681–2695, DOI 10.1145/3658644.3691638 [peer-reviewed, qualitative].
    https://dl.acm.org/doi/10.1145/3658644.3691638 ; author PDF
    https://www.lucysimko.com/publications/simko-exam-proctoring-evasion.pdf
12. Barnard College News — "Professor Lucy Simko Publishes New Research..." (2024-10-18)
    [academic-3P, confirms qualitative YouTube/TikTok methodology].
    https://barnard.edu/news/professor-lucy-simko-publishes-new-research-academic-dishonesty-online
13. Georgetown SecLab (Micah Sherr) — project page [author lab, "iterative analysis of
    thousands of TikTok and YouTube videos"].
    https://seclab.cs.georgetown.edu/msherr/
14. Solis, Shashidhar, Varol — "Automated Proctoring Solutions: Modern Techniques to
    Evade & Lure Computerized Proctoring Systems," IAFOR 2023 [independent, weaker venue;
    DEMONSTRATED Respondus evasion, distinct from Simko].
    https://papers.iafor.org/submission66044
15. LDBypass — "AI Overlay for Examus Proctoring on Mac" (2026-03-01) **[ADVERSARIAL /
    low-trust; used only as contra-indication, never as corroboration]**: claims Examus
    macOS capture honors window-privacy and "None observe the screen window list."
    https://ldbypass.com/guides/ai-overlay-examus
