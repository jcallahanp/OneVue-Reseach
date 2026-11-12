# T22 Other vendors — how they work (mechanism dossiers)

Purpose: decision-grade technical dossiers of how the major OTHER online-proctoring
tools work, synthesized from the PUBLIC technical record only (vendor docs, security-research
teardowns, academic analyses, patents, privacy disclosures, court filings, observed behavior).
No binary reverse-engineering was performed for this slice; this is documentation synthesis.

Labelling convention used throughout:
- **[documented]** = stated in a primary vendor doc, a court filing, a published teardown, or peer-reviewed/academic work.
- **[inferred]** = reasoned from architecture or partial evidence; not directly stated.
- **[vendor-asserted]** = vendor marketing/claim, not independently verified.
- **[adversarial]** = source is a cheating-tool seller (self-interested, low trust); used only as a pointer, never as proof.
- Source reliability is called out inline. **Access date for every source: 2026-11-12.**

Relationship to prior slices: this EXTENDS T07 (control matrix for the one-machine
concealed-workspace threat) and T11 (claim re-verification). Where T07 already established a
control's presence/absence, this slice adds the *mechanism* (how the client is built, what the
pipeline does, how data flows, what research found). T07/T11 source numbers are referenced as
[T07 Sxx] / [T11 n] to avoid duplicating their URL lists; new URLs are numbered below.

---

## A. Proctorio (fullest) — browser-extension proctoring with optional desktop Companion App

Proctorio is the most thoroughly dissected proctoring product in the public record: at least
five independent teardowns, two reverse-engineering-driven controversies that became lawsuits,
a published UXSS vulnerability, and an academic efficacy study. This section is built on those,
cross-checked against Proctorio's own docs.

### A.1 Client architecture — what the extension is and what the sandbox lets it see

- **Deployment model:** a **Chromium/Edge browser extension** ("Proctorio Secure Exam Proctor"),
  distributed via the Chrome Web Store, integrated into the LMS (Canvas, Blackboard, Brightspace,
  Moodle, D2L, and assessment platforms) over **LTI** using the institution's single sign-on. No
  native install for the base product; the extension "only runs during an exam" and can be
  uninstalled after. [documented] [1][13][14]
- **Sandbox boundary (the central fact):** as a browser extension it is "mostly sandboxed and can
  only access things through the APIs exposed by the browser … it can not view any files on the
  computer or read any browser history or passwords. It also does not have low-level access to the
  hardware," only generic properties (device names for mic/webcam, number of cores, screen
  resolutions, disk size, power status). Independent security researcher Thom Wiggers establishes
  this directly from the extension's API surface. [documented] [1]
- **Two-part internal structure:** a **background page** (holds the extension's permissions) plus
  **content scripts** injected into the test page. The extension "inspects network traffic of the
  browser. When requests are observed for paths that match supported test-taking websites, it injects
  … content scripts," then queries the test site's API endpoints to confirm a Proctorio-enabled exam.
  [documented — Sector 7/Computest teardown] [4]
- **Permissions it requests (Chrome wording), and Proctorio's own stated purpose for each** [documented] [6][13]:
  - *Read and change all your data on the websites you visit* → monitor web traffic during the exam.
  - *Capture content of your screen* → screen recording / detect out-of-browser resources (when enabled).
  - *Access webcam and microphone* → video/audio recording (when enabled).
  - *Modify data you copy and paste* → overwrite the clipboard to block copy/paste of exam content.
  - *Manage your downloads* → block downloads during the exam.
  - *Identify and eject storage devices* → enumerate attached drives/USB (does not read files).
  - *Manage your apps, extensions, and themes* → the "Disable Extensions" setting; also used to
    **enumerate and disable other installed extensions** (e.g. AI side-panels / cheat helpers).
    [documented vendor + Fracture (adversarial) corroboration] [6][19]
  - *Change your privacy-related settings* → temporarily block the browser's password autofill; also
    used to **read/clear the browser's proxy settings** (the anti-proxy behavior researchers observed). [documented] [6][2]
  - *Display notifications* → alert test-taker in open-note exams.
- **What the sandbox CANNOT do (consequential for the certification owner):** it cannot see a VM it
  is NOT running inside, a remote-desktop session whose window is off-screen, running OS processes,
  or a capture-excluded overlay that lives outside the browser window. Wiggers: a remote-desktop
  helper is invisible "if the remote desktop application is not visible on the screen … Proctorio can
  not see running applications if they are not on the screen." VM detection is reduced to **heuristics**
  on the few readable properties and "can be defeated by tweaking some settings in the virtualisation
  software and running the VM at the full screen resolution." [documented] [1] (Consistent with [T07 S43].)

### A.2 The "abnormality" / behavioral-analysis engine — what signals, how scored

Proctorio does not run a single "cheating classifier." It computes a set of **per-frame and
per-session metrics**, each weighted by the exam administrator, and sums them into a relative
**"suspicion level."** The design is explicitly **administrator-configured and relative-to-cohort**,
not an autonomous verdict. [documented — Proctorio behavior-settings docs + efficacy study] [14][15][12]

- **Signals collected (grouped as Proctorio groups them)** [documented] [15][12]:
  - *Frame Metrics* (need Record Video / Record Audio on): **head movement** (looking away from the
    exam window), **multiple faces / "abnormal number of faces,"** **leaving the room** (no face),
    **audio levels** (ambient noise above room baseline), **navigating away** (new tab / focus loss),
    **copy/paste**, **browser resize**, **keystrokes**.
  - *Computer-Based Abnormalities* (relative to other test-takers): **mouse movement**, **clicking**,
    **scrolling**, **keystrokes**, **navigating away**, **copy/paste**, **browser resize** — each
    flagging a test-taker who did significantly MORE or LESS than the cohort.
  - *Environmental Abnormalities:* audio levels, head/eye movement, leaving the room, multi-face — again relative to cohort.
  - *Technical Abnormalities:* exam duration vs cohort, **start/end-time mismatch** between Proctorio
    and the LMS gradebook (flags lost connection or "attempted circumvention"), **exam collusion** (cross-attempt similarity).
- **Scoring mechanism:** each behavior flag carries a **weight 0–5**. 0 = off (not counted, not logged);
  5 = any single detection drives that frame to "high suspicion." The engine produces a **suspicion
  level and a cohort rank** per attempt, shown in the Proctorio Gradebook; reviewers see an incident
  log and the annotated recording. **Presets** ("Recommended," "Open Note," "Group Exam," "Custom")
  are bundles of these weights (e.g. Group Exam ignores audio/head-movement/multi-face; Open Note
  ignores new tabs, copy/paste, resize, head movement). [documented] [15][12][17]
- **Face/gaze specifics — what it actually is:** Proctorio's own FAQ states it uses **face DETECTION
  and gaze DETECTION, not facial RECOGNITION** — i.e. it detects the *presence/number* of faces and
  whether a face is oriented toward the screen, and does **not** identify or match the face to a
  database, and does **not** track precise eye movements. It says it does **not** use keystroke
  *fingerprinting* but does track **keystroke anomalies**. [documented — vendor] [14]
- **"No automated decision" framing:** Proctorio repeatedly states it "does not make automated
  decisions"; AI only flags, and a human (institution reviewer) decides. It says it does **not** train
  models on end-user exam video/recordings. [vendor-asserted] [14] This framing is contested by EPIC
  (see A.5) and is the pivot of the Twente efficacy critique (A.4).
- **Independent corroboration of the signal set:** the University of Twente efficacy study reproduced
  Proctorio's three profiles and tabulated the exact frame-metric weights (navigating away, keystrokes,
  copy/paste, browser resize, audio levels, head/eye movement, multi-face, leaving the room, plus the
  computer/environmental/technical groups) — confirming the documented settings match the deployed
  product. [academic] [12]

### A.3 Recording, encryption and data flow

- **Recording:** webcam video, microphone audio, and **screen** are captured via the browser's
  **screen-sharing / `chrome.desktopCapture`-type API** (originally built for video calls), plus
  room-scan and photo-ID capture at pre-check. Recording happens **only during the exam** and **only
  for the settings the administrator enabled** (the toolbar icon turns green while active). [documented] [1][13][14]
- **"Zero-Knowledge" encryption — and its documented weakness:** Proctorio markets AES-256
  "Zero-Knowledge Encryption" so only institution-approved reviewers hold keys; the extension layer
  uses AES-GCM. Independent reverse-engineering by "Proctor Ninja" (Lucy Satheesan) traced the key
  derivation and found the recording encryption key was derived as roughly
  `PBKDF2(password = MD5(quizID), salt = MD5(userID), count ≈ 12048)`. Because Canvas/Moodle **quiz IDs
  and user IDs are small incrementing integers**, the keyspace was brute-forceable "in a few hours on a
  commodity laptop" by an actor who already holds the encrypted data (e.g. rogue insider or
  post-compromise). Proctorio hardened the derivation ~1.5 weeks after disclosure (added constant noise,
  raised the iteration count). [documented — security research] [2]
- **Telemetry / server comms:** IP addresses are captured in **truncated** form; data is stored in a
  **regional data center** (disclosed at pre-check), AES-256 at rest, SOC-2 attested, retention set by
  the institution (as little as 7 days). [documented — vendor + court answer] [10][14] Independent
  observation (Kuketz, tcpdump/mitmproxy) found the installed extension makes **periodic TLS
  connections to Proctorio servers even outside exams**, contents not decryptable from the packet
  capture alone; and that the extension **overrides the browser/system proxy** and **ceases sending to
  Proctorio servers when it detects traffic being redirected to a proxy** (anti-interception behavior).
  [documented — independent security blog] [6]
- **Obfuscation:** teardowns found the extension is minified and **obfuscated with JScrambler**
  (variable masking, regex obfuscation) and ships XOR-obfuscated resources mislabeled `.7z`; crypto
  functions were renamed after intelligence agencies (e.g. aes→"cia"). Researchers note extension
  obfuscation appears to violate Chrome Web Store "code readability" policy. The facial-detection
  models are fetched as obfuscated `.7z` blobs (`FmuG8K.7z`, `nWZk8V.7z`, `rXqE9b.7z`, `sVAazF.7z`) and
  also served from `cdn.proctorauth.com`. [documented — security research] [2][3][9]

### A.4 The Companion App — the ONLY path to desktop-level visibility

- Base extension ≠ host visibility. Proctorio's **"Advanced Security"** settings —
  **Advanced Program Detection, Advanced VM Detection, Advanced Hardware Detection** — each **require
  the free desktop "Secure Companion App"** (Windows/macOS; a Mobile App on Chromebook) installed
  alongside the extension. Made generally available to customers April 2025. [documented — vendor] [16][17][18]
- **What the Companion App does** [documented — vendor] [16][17][18]:
  - *Advanced Program Detection:* "the Desktop App **closes unauthorized applications** before your exam"
    and **locks the device**; on macOS it uses **Apple's Assessment / Automatic Assessment Configuration**
    APIs. Candidates may be told to close listed background processes manually (via Task Manager / Activity
    Monitor). A later "Allow Apps" setting lets admins whitelist Word/Excel — which requires Advanced
    Program Detection first.
  - *Advanced VM Detection:* "ensure students cannot use a virtual machine to spoof audio, webcam video, or
    desktop screen sharing." A 2025 changelog adds that **Parallels Desktop** VMs are now flagged as
    "VM detected."
  - *Advanced Hardware Detection:* "deeper tracking of any webcam and microphone changes during the exam";
    exam PDFs can include **hardware-change, detected-speaking, and potential-mobile-device events**.
- **Implication for the certification owner:** Proctorio only reaches the host (processes/VM/hardware)
  when the Companion App is required AND installed; a managed/locked device may block installation. This
  is the difference between "mid-tier" and "extension-only / weak" in [T07 S40][T07 S43].

### A.5 Known security / privacy research and incidents (the rich part)

1. **OpenCV facial-detection racial-bias finding (2021).** Researcher **Lucy Satheesan** decompiled the
   Chrome extension and found the "proprietary" facial-detection files were **identical to OpenCV**
   models; tested against the **FairFace** dataset, they failed to detect **Black faces ~57% of the time**
   (detected <50%). Reproduced by Motherboard's consulted researcher and, independently, by **RTL Nieuws**
   (2023), which analyzed the four model files (`FmuG8K/nWZk8V/rXqE9b/sVAazF.7z`) against FairFace and found
   Black-face detection statistically significantly worse (p<0.01 on 22 of 24 comparisons; one tolerant
   model mis-classified chins/necks/clothing as faces). Proctorio conceded it "uses face detection (rather
   than facial recognition)" and licenses OpenCV but disputed the methodology. [documented — security
   research + press + national-broadcaster technical appendix] [7][8][9][11]
2. **Erik Johnson DMCA / free-speech lawsuit (2020–2022).** Miami University student/security researcher
   **Erik Johnson** tweeted annotated **excerpts of Proctorio's extension source** (language-string files
   revealing room-scan/ID strings and the **"Abnormal number of faces detected," "Multiple faces,"
   "Multi-Face," head-movement "e.g. look at notes"** metric labels) and showed the extension
   **self-destructs/uninstalls and redirects to a `/hacker` page when it detects DevTools open** — "always,
   not just during a test." Proctorio issued DMCA takedowns (Pastebin/GitHub); the EFF sued on Johnson's
   behalf (textbook fair-use). Settled 2022. Establishes, from the vendor's own strings, the behavior
   taxonomy and an anti-inspection self-uninstall. [documented — court complaint + EFF] [10]
3. **Universal Cross-Site Scripting vulnerability (Sector 7 / Computest, 2021).** The extension's in-exam
   **calculator** passed input to JavaScript **`eval()`** without validation; because the calculator is
   injected into the activating page's DOM, **any website** could auto-activate Proctorio (demo mode),
   open the calculator, execute code in the content script, then message the privileged background page —
   yielding **UXSS: any site could read/modify data on every site** for a victim with the extension
   installed. Reported 2021-06-18; fixed within a week. Illustrates that a broadly-permissioned proctoring
   extension is itself an attack surface. [documented — peer security research] [4]
4. **Obfuscation / Chrome-policy concerns (Proctor Ninja, 2020–21; Soatok, 2020).** See A.3; plus Soatok's
   analysis of the AES-GCM implementation (`dhs.mode.gcm`) flagging timing-leak concerns in the GMAC path.
   [documented — security research] [2][3][5]
5. **Efficacy study — University of Twente / CSEDU 2021, "On the Efficacy of Online Proctoring using
   Proctorio."** 30 students, 6 instructed to cheat, 5 to act nervous-but-honest. Findings: VM use largely
   defeated screen recording (a Wikipedia lookup succeeded; the resized VM window was flagged only as an
   "odd resolution" abnormality, not as a VM per se); **virtualized webcam never passed the system check**;
   per Proctorio's own guidance only the **top 20% of suspicion ranks** get manual review, so honest-nervous
   students risk false flags while some cheating ranks below the review threshold. Conclusion: easy to
   work around, and ranking-based review trades false positives against missed cheating. [academic] [12]
6. **EPIC FTC/DC-AG complaint (Dec 2020).** EPIC named **Respondus, ProctorU, Proctorio, Examity, Honorlock**
   for excessive biometric/personal-data collection and "opaque, unproven, potentially biased AI." Proctorio
   and Honorlock specifically cited over facial-detection claims. (Cross-cutting — see §Cross-cutting.) [documented] [39]
7. **Linkletter litigation / broader chilling-effect context.** Proctorio's CEO publicly posted a student's
   support-chat log (Reddit, 2020) and sued UBC learning-technologist **Ian Linkletter** for linking to
   unlisted YouTube training videos — context establishing an adversarial posture toward public scrutiny
   (relevant to why so much of the record is researcher-vs-vendor). [documented — reported] [2]

### A.6 Documented limitations (Proctorio)

- Extension sandbox cannot see VM/remote-desktop/host processes/overlays unless the Companion App is used. [1][16]
- VM/webcam/display detection is **heuristic** at the extension layer; defeated by tuned VMs / full-screen resolution. [1][12]
- Facial detection is OpenCV-based and demonstrably **less accurate on darker skin** (false "no face"/"multi-face" flags). [7][9][11]
- The product is itself a security/privacy surface (UXSS history; proxy override; always-on server beacons; brute-forceable key derivation now patched). [2][4][6]
- "No automated decision" is a review-workflow claim, not an absence of scoring; the suspicion score IS an algorithmic ranking. [12][14][39]

---

## B. ProctorU / Meazure Learning — "Guardian" native secure browser + human-in-the-loop

ProctorU (owned by **Meazure Learning**) migrated from a browser extension to a **native secure
desktop browser, "Guardian,"** which became the mandatory connection mechanism in January 2023. This
is the architectural opposite of Proctorio: a native host agent with continuous checks, paired with
a live/AI **hybrid** human review model. (T07/T11 established the control set and re-verified the
30-second claim; this adds mechanism, the hybrid pipeline, and the breach.)

### B.1 Client architecture — native secure browser (host-integrity model)

- **Guardian is a proprietary native application** (not an extension) downloaded from a Meazure source,
  installed once and reused for later exams. Meazure's own rationale: test-takers "download the Guardian
  Browser **so Meazure Learning can monitor their machines from a technical perspective**" — explicitly a
  native path to host visibility the old extension lacked. [documented — vendor] [23][T07 S15]
- It is the connector for all three service lines — **Live+, Review+, Record+** — and "locks down each
  test-taker's computer according to settings configured by exam administrators." Security functions
  (disable screenshots, block copy/paste, lock the virtual environment) are **in effect only while a
  ProctorU session is active**; Guardian clears history/cache/cookies after use and stores no PII itself
  (PII lives in the ProctorU Platform). [documented — vendor] [24][22]

### B.2 Detection mechanisms — process enumeration, ~30s re-check, VM/remote scan

- **Periodic re-enumeration (the load-bearing control):** "the Guardian Browser **blocks interaction
  with locally installed applications and checks for unpermitted software every 30 seconds**." It also
  **"logs all active processes"** on the machine for investigation and runs a **"continuous script"** to
  detect/deny multiple monitors, virtual machines, and unpermitted applications **during the exam**.
  [documented — vendor; independently re-verified in T11 as periodic ~30s + all-process logging, HIGH that
  documented / MODERATE that it performs exactly as described, no third-party teardown of Guardian exists] [20][21][T11 1-4]
- **Blocklist scope:** Guardian "**detects over 500 computer processes and applications**" total, of which
  "**over 400 applications**" specifically target **virtual machines and remote-access tools** (nested
  scopes, both vendor figures). Published denylist (non-exhaustive) covers remote access (TeamViewer,
  AnyDesk, Splashtop, Chrome Remote Desktop/`remoting_host.exe`, MS Remote Desktop), screen capture/stream
  (Duet Display, GreenShot, ScreenPal, NVIDIA Broadcast), other browsers, and writing assistants (Grammarly);
  "monitors hundreds of additional processes." [documented — vendor] [21][25][T07 S16]
- **Multi-monitor / concurrent login:** a detected second display prompts disconnect before proceeding;
  concurrent logins are blocked. [documented] [T07 S15]
- **VM detection is still software-heuristic** (app-signature scan), not hardware attestation — the residual
  gap T07 flagged. [documented/inferred] [T07 S37-S48 cross-cutting]

### B.3 Monitoring / AI pipeline — hybrid, human-decides

- **Hybrid model, by design and as a marketing stance:** Meazure positions itself **against
  "software-only proctoring."** AI is used "for automating repetitive tasks" but **"we do not allow AI to
  make … decisions like whether or not someone cheated."** Human, trained proctors assess **every minute**
  of a session (live in Live+, or full-video review in Review+/Record+ — not only AI-flagged segments).
  [vendor-asserted] [22 (myth-busters)][20]
- **Three lines:** *Live+* = synchronous human proctor who can call in an **intervention specialist** to
  stop suspected cheating in real time; *Review+* and *Record+* = asynchronous full-video human review.
  Incident reports (timestamped evidence, end-to-end recording) delivered ~8h average, up to 24h (Live+) /
  72h (Review+/Record+). [documented — vendor] [24][20]
- **Telemetry:** "extensive telemetry data generated by the platform itself" plus multifactor ID validation;
  Guardian captures "a host of telemetry … to identify suspicious setups." [documented — vendor] [20]

### B.4 Known security / privacy research and incidents

- **2020 ShinyHunters data breach (~444,000 records).** In late July 2020 the threat actor **ShinyHunters**
  dumped a ProctorU database (≈444k users) among 18 companies' data (386M+ records) on a hacker forum.
  Exposed: usernames, legal names, full residential addresses, phone, institutional email, and **password
  digests (reported bcrypt; some reports said unencrypted/weakly-protected)**. ProctorU confirmed a breach
  but said the data was **legacy (registered on/before ~2014, nothing newer than 2016) and not from active
  production servers**; it reset affected passwords, raised complexity requirements, and re-established
  internal permissions. Multiple Australian universities (Sydney, UNSW, Melbourne, Queensland, etc.)
  investigated. [documented — BleepingComputer, AUSCERT, Infosecurity Mag, iTnews, SBS, Avast + ProctorU
  statement] [26][27][28 group][29]
- **EPIC complaint (Dec 2020):** named alongside the other four for excessive data collection and opaque AI.
  [documented] [39]
- **No independent technical teardown of Guardian exists** (unlike Proctorio). The 30s/400+/500+/continuous
  claims are **entirely first-party**; treat as vendor-asserted but internally consistent across 4+ Meazure
  pages (T11 verdict). [T11 1-4]

### B.5 Documented limitations (ProctorU/Guardian)

- "Continuous" is really **30-second polling** + all-process logging; a helper opening and closing entirely
  inside a <30s gap could slip the re-scan (the all-process log narrows this). [T11 1]
- VM detection = app/heuristic scan; no hardware attestation / Type-1 hypervisor resistance. [cross-cutting]
- All host-integrity claims are vendor-asserted, un-audited by any third party. [T11]
- Legacy data-handling weakness demonstrated by the 2020 breach (plaintext/weak PII at rest historically). [26][29]

---

## C. Honorlock — Chrome extension + "BrowserGuard," cloud AI, "Search & Destroy," phone detection

Honorlock's mass-market product is **a Chrome extension**, backed by **cloud AI (AWS) + on-demand live
proctors ("Live Pop-In")**. Its distinctive mechanisms are **BrowserGuard** (in-extension lockdown),
**Search & Destroy** (leaked-content takedown), and **Multi-Device Detection** (audio-watermark phone
detection). A newer **Desktop Application** adds limited host visibility.

### C.1 Client architecture — extension-first, optional desktop app

- **Base product = Chrome extension only**, one-click install/remove, active only during an
  Honorlock-enabled exam in the LMS; "Honorlock does not have the ability to control your computer, access
  your passwords, or download any files." Data at rest/in transit encrypted in **Honorlock's AWS private
  cloud**; standard **365-day retention**. [documented — vendor + university IT] [36][37][38]
- **Extension permissions and what each is for** [documented — Honorlock KB] [30]:
  - *Read/change all data on all websites* + *read/change browsing history* → identify enabled exam URLs;
    if none, "the extension does nothing"; during the exam, read browsing history (time-boxed to exam window only).
  - *`chrome.management`* → **query installed extensions and disable unauthorized ones** after consent.
  - *`chrome.tabs`* → close/block tabs and keep the proctoring window open (BrowserGuard / allowed-URL).
  - *`chrome.desktopCapture`* → initialize **desktop screen recording** (only if "Screen Record" enabled).
  - *`chrome.system.display`* → **detect window resizing** (ensure single-screen focus).
  - *content settings / `chrome.storage`* → read JavaScript/plugin/mic/camera status; store proctoring settings.
  - Facial **detection only** (presence/▲multiple faces), no identity match, no facial data stored. [36]
- **BrowserGuard (in-extension lockdown):** one-click toggle that "blocks and flags unauthorized websites,
  browser tabs, applications, and keyboard shortcuts while recording the desktop"; removes toolbar/menu,
  **blocks/flags minimizing** the window, and can **allowlist specific URLs/apps**. It is a browser-level
  lockdown, not an OS lockdown. [documented — vendor] [31][35]
- **Honorlock Desktop Application (newer, "for heightened security"):** a separate download that "launches
  Chrome in a secure state," **monitors desktop activity to block unauthorized external applications and
  native AI features**, and is "designed to **detect any apps on your device that you use (or attempt to
  use)** during the exam." This is the ONLY Honorlock path with host-level process visibility; otherwise
  Honorlock is pixels-only. [documented — vendor] [35][36][38]

### C.2 Detection mechanisms and pipeline

- **Cloud AI + live hybrid:** AI monitors the session; if it "senses something is wrong, it triggers a
  **live proctor pop-in**." AI flags: **no face / multiple faces, voice detected** (Honorlock generates a
  **written transcript whenever a voice is detected** and high-priority-flags trigger words), and
  **Apple Hand-Off icon detection** (infers a nearby iPhone being used). Human proctors review/intervene.
  [documented — vendor] [32][38]
- **Multi-Device Detection / "Search & Destroy" are two different things — do not conflate:**
  - **Multi-Device Detection (the phone-catching mechanism):** when a test-taker searches a test question on
    a **secondary device**, Honorlock plays an **inaudible "audio watermark" through the secondary device**
    (triggered when the phone loads a page hosting Honorlock's beacon), which is **picked up by the exam
    computer's microphone** ("proximity sensor"). Honorlock then "knows what device, what browser, and what
    website" and captures a screen recording of the secondary device. Crucially, Honorlock states it does
    **NOT** scan the home network or access the phone directly — detection is **acoustic + web-beacon**, not
    network snooping. [documented — vendor; mechanism via patent] [32][37]
  - **Search & Destroy™ (leaked-content protection, not a device sensor):** on enabling an exam, Honorlock
    **searches the public web for copies of the exam's questions** (verbatim/near-verbatim), scores and
    buckets each question as **at-risk / common / unique** (deterministic rule: matches on supported sites
    with ≤5 total links = at-risk; >5 or unsupported-only = common; no match = unique; questions must be
    ≥6 words), and offers **one-click DMCA takedown** against supported sites (Chegg, Quizlet, Course Hero
    type). This is content-leak defense, independent of any individual session. [documented — vendor +
    public API docs] [33][34]
- **Connection-quality monitoring** of the test device only (to document stability), explicitly not of other
  network devices. [documented] [37]

### C.3 Known research / incidents and limitations

- **EPIC complaint (Dec 2020)** named Honorlock (with Proctorio specifically) over facial detection and
  data-collection claims. [documented] [39]
- **No published binary/extension teardown of Honorlock** of the depth of the Proctorio corpus was found —
  the record is vendor docs + university privacy pages + the EPIC filing. (Gap worth noting for an evaluator.)
- **Documented limitations:** extension sandbox cannot see a VM it isn't in, an off-screen remote-desktop
  session, or host processes/overlays (unless the Desktop App is used — limited release); **no VM detection
  documented** for the extension; Multi-Device Detection depends on the phone loading a beacon page and on
  mic proximity/quiet room; Search & Destroy only helps for **≥6-word, supported-site** leaks. [documented/inferred] [30][32][33][T07 S37-S39]

---

## D. Brief dossiers (architecture type · core detection model · public research)

### D.1 Examity — browser-based, extension-optional, live/auto tiered, AI + human auditor

- **Architecture:** **browser-based (Chrome) with a lightweight Examity browser extension** installed per
  session (and usually self-removed after). An **optional "Lockdown Browser" feature runs *through the
  existing extension*** (no extra download) and can close extra tabs, block new tabs, and disable copy/paste
  and print — only when the exam is inside the LMS. So: extension-class, NOT a native secure browser; host
  visibility is shallow. [documented — vendor guides] [44][46]
- **Core detection model:** a **tiered** product — *Auto ID / Mobile ID / Live ID* authentication (Mobile ID
  uses phone fingerprint/face/voice biometrics; profile setup includes a **biometric keystroke signature** +
  3 challenge questions + photo-ID + facial scan) → *Auto Standard / Auto Premium / Live Standard / Live
  Premium* proctoring. **Real-time video analysis runs in-browser via TensorFlow.js**; flags fire on
  off-domain navigation, loud/aberrant audio (audio-signature), **no-face / multi-face**, **posture/face-
  direction away from screen** (by degree and duration), and finishing too fast. A **human auditor** second-
  reviews and applies a **four-color flag system** (green/yellow/red/blue); test-takers are never told their
  flags. Retention ~60 days (1 year if a red flag). [documented — vendor] [43][45][47][48]
- **Public research/incidents:** named in the **EPIC Dec-2020 complaint** [39]. No independent technical
  teardown found; VM / remote-desktop / process enumeration are **undocumented** (consistent with T07's
  "undocumented host integrity" verdict [T07 S44]). Strength historically rests on live-proctor observation +
  human audit, which a concealed on-machine overlay evades.

### D.2 PSI (Bridge Secure Browser) and Prometric (ProProctor) — native secure browsers / locked desktops

Grouped because both are **native lock-down desktop clients** aimed at professional/certification testing
(the closest architectural peers to Pearson VUE OnVUE's Browser Lock). Mechanism detail and the full control
set are in T07; summarized here with the architecture framing.

- **PSI Bridge Secure Browser — architecture:** a **native downloadable secure lockdown browser**
  ("Record & Review") that "prevents users from copying, pasting, taking screen grabs, **using remote desktop
  and virtual machines**, using instant messaging or other applications," records webcam + desktop, and
  blocks multi-monitor (dual screens not allowed). Differentiator: **no remote-proctor-access model**.
  Documented nuance: a 2020 setup guide stated the **compatibility pre-check could not itself detect a VM**
  (enforcement rested on policy + secure-browser launch). [documented] [T07 S7-S9][T07 S13][T07 S14]
- **Prometric ProProctor — architecture:** a proprietary **lock-down browser that isolates the exam to a
  "locked-down desktop," taking "complete control of the computer environment"** so ProProctor is "the only
  application running," controlling email/IM/**desktop sharing** "without modifying the host system."
  **VMs, VPNs, firewalls, and machines in Administrator Role are "not compatible"** (admin-role block is a
  notable anti-tamper posture); dual-monitor not allowed. **ProGuardian AI** runs "at the testing
  workstation," backed by a 24/7 Security Command Center; "unknown application running" can be terminated;
  desktop recording + second camera available. [documented] [T07 S10-S12]
- **Public research:** no independent teardown of either found; posture is strong but **public docs don't
  quantify process/overlay-enumeration depth** (VM handled as "incompatible," not described as continuous
  detection). [T07]

### D.3 Talview — Talview Secure Browser (TSB) + "Alvy" AI; patent on contextual detection

- **Architecture:** dual-mode. **TSB = native secure browser** (lockdown; blocks app/tab switching, alt-tab,
  screen recording, shortcuts) OR a **Chrome/Edge full-screen-share mode** (pixels-only) when TSB isn't used.
  Positioned as **"nothing to install"** for the browser mode; auxiliary smartphone **secondary camera** for
  desk/room view. [documented — vendor] [49][T07 S21-S22]
- **Core detection model ("Alvy"):** **continuous liveness + face authentication** (not one-time) to catch a
  post-login swap; **ID-document authenticity + face match with liveness to rule out a photo or virtual
  camera**; "names applications and devices inside a secure browser"; TSB raises a **"Virtual Desktop
  Detected"** error on Remote Desktop / VM / screen-share / switching to a virtual desktop. Automated,
  record-and-review, or live, with Alvy as "copilot" and a **human making the final call**. Talview itself
  candidly states standard gaze/face proctoring "has nothing to say about … an AI model answering in real
  time." [documented — vendor] [49][T07 S21]
- **Public research/IP:** **US Patent 12,361,115 B1** ("contextual, real-time session detection") is the
  cited primary technical reference. An **[adversarial]** bypass-seller claims a macOS overlay sits *below*
  Talview's WebRTC/tab-capture path and that TSB's VM checks rely on browser-exposed fingerprints/timing
  heuristics — pointer only, not proof [T07 S32]. No independent academic teardown found.

### D.4 Constructor / Examus — native Secure Browser + AI "cheating score"; best-documented API

- **Architecture:** a **web application (runs in Chrome/Edge, no install)** OR a **standalone native
  "Secure Browser"** (Windows/macOS; no tabs, cannot open other pages, blocks shortcuts, auto-closes
  block-listed apps like Zoom/Skype/Telegram, and in Medium/High modes blocks app use; High mode also blocks
  minimize/close). Three modes: **basic / medium / high** (`secureBrowserLevel`). Lineage: Constructor
  acquired/rebranded **Examus**. [documented — vendor docs] [50][51][52]
- **Core detection model (unusually transparent, via public developer docs):** a per-examinee **AI "cheating
  score" 0–100%**. Documented warning signals include: **`virtual_environment`/"Virtualization is used"**
  (VM), **virtual-camera detection** (blocks faked feeds incl. NVIDIA Broadcast), **extra/mirrored display**,
  **USB-device add/remove**, **phone / phone-screen detected**, **`substitution_user`** (face swap mid-exam),
  **`account_collision`** (one person, multiple accounts), **gaze/`avert_eyes`/`look_away`** (measured via
  **head-pose angle**), **`active window is changed`** (tab/app switch), **non-verbal communication**, and
  **human-voice-in-audio**. "Blended proctoring" = AI alerts + human review. [documented — vendor dev/proctor
  guides] [51][52][50]
- **Public research:** independently described in an **EDM 2025 academic short paper** (Examus "AI-driven
  analysis of webcam, screen activity, and user behavior … detects application switches," citing Migut et al.
  frame-similarity) — a third-party *description* that corroborates the capability is real/documented, not that
  it is robust [T11 10]. Note the **T11 correction**: Constructor's "overlays" in its own docs are **full-screen
  violation-WARNING overlays shown to the examinee**, NOT a capability to detect a third-party capture-excluded
  overlay — don't read "overlay blocking" as overlay-window auditing. [T11 2c][T11 8]

### D.5 Mercer | Mettl — native Mettl Secure Browser (MSB) + "Secure Exam Proctor" Chrome extension

- **Architecture:** dual. **MSB = native web-based lockdown browser** (disables right-click/copy/paste/print/
  screenshot/shortcuts; **"detects virtual machine and does not allow the test taker to proceed"**; bars screen
  projection to external screens; blocks screen-capture/virtual-meeting apps; disables external ports to
  prevent screen duplication). Separately, a **Chrome-based "Secure Exam Proctor" extension** for Moodle/Canvas
  (blocks right-click/devtools, flags navigate-away) — pixels-only, cannot enumerate host processes. Data on
  **AWS**, encrypted in transit/at rest. [documented — vendor] [53][54][55]
- **Core detection model:** AI auto-proctoring with a defined **flag enum** — `FACE_NOT_VISIBLE`,
  `ANOTHER_PERSON`, `AWAY_FROM_WINDOW`, `PARTICIPANT_LOOKING_AWAY`, `PARTICIPANT_FACE_PARTIALLY_VISIBLE`,
  `PARTICIPANT_NOT_PRESENT`, `MOBILE_PHONE_DETECTED` — plus **audio proctoring** (detects human speech and
  number of speakers), **screen + camera snapshots every 60 seconds**, mandatory **360° room scan**, 3-point
  auth (email + ID + OTP), IP-range restriction, and a **human-AI tiered review** (AI flags → human proctoring
  shield → live proctoring). Marketing claims "95% cheating detection" (vendor-asserted). [documented — vendor] [53][54][55]
- **Public research:** no independent teardown found; the **published 15-item anti-cheat list historically did
  not itself name VM detection** (now stated on the MSB page), and non-MSB extension capture is **pixels, not
  processes** — strength depends on MSB being enforced. [documented] [53][54][T07 S4-S6]

---

## Architecture-type summary table

Legend — Client type: **EXT**=browser extension · **EXT+APP**=extension plus optional desktop companion ·
**NATIVE**=native secure browser / locked desktop · **WEB/EXT+NATIVE**=offers both a web/extension mode and a
native secure-browser mode. "Host-visibility depth" = how deep into the OS the client can see (process/VM/
overlay), to the extent public sources show.

| Vendor | Client type | Core detection model | Host-visibility depth | Notable research / incidents |
|---|---|---|---|---|
| **Proctorio** | EXT (+APP for "Advanced Security") | Admin-weighted per-frame/session abnormality metrics → relative suspicion score; OpenCV face+gaze DETECTION; human reviews, no auto-decision | Extension = browser sandbox only (no host processes/VM/overlay); **Companion App** adds process-close + VM + hardware detection (macOS Assessment API) | Richest public corpus: OpenCV racial-bias teardown (Satheesan/RTL) [7][9][11]; Johnson DMCA/EFF suit [10]; Sector 7 **UXSS** [4]; brute-forceable "zero-knowledge" key derivation [2]; Twente efficacy study [12]; EPIC [39] |
| **ProctorU / Guardian (Meazure)** | NATIVE secure browser | **~30s re-check + all-process logging**; 500+ app blocklist (400+ = VM/remote-access); live+AI **hybrid**, human decides | **Deepest documented** continuous host enumeration (processes, monitors, VM heuristic) — but all vendor-asserted, no third-party teardown | **2020 ShinyHunters breach ~444k records** (legacy PII, weak at rest) [26][29]; EPIC [39]; no independent Guardian teardown [T11] |
| **Honorlock** | EXT (+ newer Desktop App) | Cloud AI (AWS) face/voice detection + **Live Pop-In**; BrowserGuard lockdown; **Multi-Device Detection** (audio watermark→mic); **Search & Destroy** web-leak DMCA | Extension = sandbox (no VM/host processes/overlay); **Desktop App** adds app-use detection (limited release) | EPIC (named with Proctorio on facial detection) [39]; patent-based phone detection [32]; no deep public teardown found |
| **Examity** | EXT (optional in-ext "Lockdown Browser") | Tiered Auto/Live; TensorFlow.js in-browser video analysis; face/posture/audio flags; keystroke-biometric auth; human auditor + 4-color flags | Shallow (extension-class); **VM/remote-desktop/process enumeration undocumented** | EPIC [39]; no independent teardown; host integrity undocumented [T07 S44] |
| **PSI (Bridge Secure Browser)** | NATIVE secure browser | Record&Review lockdown: blocks VM/remote-desktop/other apps, multi-monitor; records webcam+desktop; **no remote-proctor access** | Native (host-level launch checks); documented **pre-check could not itself detect a VM** (policy-enforced) | No independent teardown; 2020 setup-guide VM-detection gap [T07 S14] |
| **Prometric ProProctor** | NATIVE locked desktop | Isolates to a locked-down desktop (sole app); blocks VM/VPN/**Admin-role machine**/dual-monitor; **ProGuardian AI** at workstation; terminates unknown app | Native, "complete control"; overlay/process-enumeration depth not quantified publicly | No independent teardown; posture strong, mechanism under-documented [T07 S10-S12] |
| **Talview (TSB + Alvy)** | WEB/EXT **+ NATIVE** (TSB) | Continuous liveness + face auth; ID authenticity; virtual-camera rule-out; TSB "Virtual Desktop Detected" (VM/RDP/screen-share); human final call | TSB native = VM/RDP/virtual-desktop detect; browser mode = pixels-only | **US Patent 12,361,115 B1**; [adversarial] overlay-below-capture claim on macOS [49][T07 S32] |
| **Constructor / Examus** | WEB/EXT **+ NATIVE** (Secure Browser, 3 modes) | AI **cheating score 0–100%**; VM + **virtual-camera** + extra/mirror display + phone + face-swap + account-collision + head-pose gaze + active-window-change | Native Secure Browser = app block-list + VM/virtual-cam/display; web mode = browser sandbox | EDM 2025 academic description [T11 10]; **T11 correction**: "overlays" are warning overlays, not 3rd-party-overlay detection [T11 2c] |
| **Mercer | Mettl** | WEB/EXT **+ NATIVE** (MSB) | AI flag enum (face/person/phone/away) + audio speaker-count; **60s snapshots**; 360° scan; 3-point auth; human-AI tiers; MSB "detects VM, blocks proceed" | MSB native = VM/screen-projection/port lockdown; extension = pixels-only | No independent teardown; VM historically absent from published list; AWS-hosted [53][54] |
| *(baseline) Pearson VUE OnVUE* | NATIVE (Browser Lock) | VM block + name/close disallowed processes **at launch**; one display; capable of continuous monitoring but **OFF in user's deployment** | Native launch-time checks; continuous host/screen layer disabled in user's config | See T07/T08 — the post-launch second-workspace gap this project targets [T07 S33-S36] |

Cross-cutting (reaffirmed from T07): **no vendor documents hardware attestation (TPM / Secure Boot /
measured boot) or Type-1 / bare-metal hypervisor detection.** Every "VM detection" above is a **software
heuristic** (CPUID/vendor-string where a native agent can read it, driver/registry/device-name lists,
SMBIOS, app-signature scans). The native secure-browser vendors (Guardian, PSI, Prometric, MSB, TSB,
Constructor) can see host processes/VM; the extension vendors (Proctorio base, Honorlock base, Examity)
cannot, absent their optional desktop companion.

---

## Cross-cutting privacy / legal record (applies to the whole field)

- **EPIC "In re Online Test Proctoring Companies" (filed 2020-12-09, DC AG / FTC Act):** named
  **Respondus, ProctorU, Proctorio, Examity, Honorlock** for (A) excessive biometric/personal-data
  collection, (B) opaque/unproven/biased AI, (C) Proctorio- and Honorlock-specific facial-technology
  claims, (D) deceptive reliability claims; demanded data-minimization, trustworthy-AI compliance, and
  **annual third-party audits**. [documented] [39]
- **Respondus Monitor BIPA class action (Powell v. DePaul, N.D. Ill. 2021–22):** alleged Respondus Monitor
  "captures, uses, and stores … facial-recognition data, facial-detection data, recorded patterns of
  keystrokes, eye-monitoring data, gaze-monitoring data, and camera/microphone recordings," violating
  Illinois BIPA — a court-record enumeration of the behavioral/biometric signal classes the field collects.
  [documented] [41]
- **ExamSoft (adjacent, exam-delivery not proctoring):** 2020 PA-bar-applicant complaint to the Bureau of
  Consumer Protection alleged a data breach and intrusive device access — illustrates the sector's breach
  exposure. [documented] [42]
- **Academic synthesis:** a 2025 systematic-narrative review (ERIC EJ1481131) catalogs the field's data
  classes (physiological + behavioral biometrics: face/voice, typing patterns, mouse patterns, eye movement)
  and the underlying CV techniques named across vendors (face detection, face alignment, head-pose
  estimation, eye-gaze estimation; Viola-Jones / SURF / SIFT lineage), and calls for model-card-level
  transparency the vendors do not provide. [academic] [40]

---

## Sources (numbered URLs + dates; access date 2026-11-12)

New sources for this slice (T07/T11 sources are cited inline as [T07 Sxx]/[T11 n] and not repeated):

Proctorio:
1. Thom Wiggers — "Dissecting Proctorio" (2020-09-30) [independent security research]. https://thomwiggers.nl/blog/proctorio/
2. Proctor Ninja (Lucy Satheesan) — "Wave Rake: Proctorio" (2020-12-15) [security research; zero-knowledge key-derivation flaw]. https://proctor.ninja/wave-rake-proctorio
3. Proctor Ninja — "The duality of obfuscation: feat. Proctorio" (2021-03-18) [security research; JScrambler/XOR obfuscation]. https://proctor.ninja/the-duality-of-obfuscation-feat
4. Sector 7 / Computest — "Proctorio Chrome extension Universal Cross-Site Scripting" (2021-12-14) [peer security research; CVE-class UXSS]. https://sector7.computest.nl/post/2021-12-proctorio/ (mirror: https://defion.security/en/research-labs/proctorio-chrome-extension-universal-cross-site-scripting/)
5. Soatok (Dhole Moments) — "EduTech Spyware is Still Spyware: Proctorio Edition" (2020-09-12) [security research; AES-GCM impl]. https://soatok.blog/2020/09/12/edutech-spyware-is-still-spyware-proctorio-edition/
6. Kuketz IT-Security Blog — "Proctorio: Proctoring-Software auf Augenhöhe mit Spyware" (2021-03-11) [independent; proxy override, always-on beacons, tcpdump]. https://www.kuketz-blog.de/proctorio-proctoring-software-auf-augenhoehe-mit-spyware/
7. VICE/Motherboard (Todd Feathers) — "Proctorio Is Using Racist Algorithms to Detect Faces" (2021-04-08) [press, OpenCV finding]. https://www.vice.com/en/article/proctorio-is-using-racist-algorithms-to-detect-faces
8. The Verge (Mitchell Clark) — "Students of color are getting flagged … Proctorio … OpenCV" (2021-04-08) [press]. https://www.theverge.com/2021/4/8/22374386/proctorio-racial-bias-issues-opencv-facial-detection-schools-tests-remote-learning
9. RTL Nieuws — "Verantwoording onderzoek … naar Proctorio" (2023-10-07) [broadcaster technical appendix; FairFace test of 4 model files]. https://www.rtlnieuws.nl/sites/default/files/content/documents/2023/10/07/Verantwoording%20onderzoek%20RTL%20Nieuws%20naar%20Proctorio.pdf
10. Erik Johnson v. Proctorio Inc. — Complaint (2021-04-21) [court filing + EFF; metric strings, DevTools self-uninstall]. https://www.eff.org/files/2021/04/21/2021-04-21_johnson-proctorio_-_complaint_-_as_efiled1.pdf (docket: https://www.courtlistener.com/docket/59843781/johnson-v-proctorio-incorporated)
11. AIAAIC Repository — "Proctorio racist facial detection" (AIAAIC0574) [incident catalog]. https://www.aiaaic.org/aiaaic-repository/ai-algorithmic-and-automation-incidents/proctorio-racist-facial-detection
12. University of Twente — "On the Efficacy of Online Proctoring using Proctorio," CSEDU 2021 (2021-05-02) [academic efficacy study]. https://ris.utwente.nl/ws/portalfiles/portal/275927505/3e2a9e5b2fad237a3d35f36fa2c5f44552f2.pdf
13. Proctorio — "Why Proctorio requests certain browser permissions" (2020-10-06) [vendor]. https://proctorio.com/blog/why-proctorio-requests-certain-browser-permissions
14. Proctorio — FAQ (face vs gaze detection, no auto-decision, retention, no model training on exam data) [vendor]. https://proctorio.com/faq ; https://proctorio.com/support/hc/faq ; https://proctorio.com/frequently-asked-questions
15. Proctorio Behavior/Frame-Metrics settings documentation (weights 0–5; presets; abnormality groups) [vendor, via inscribe.education LMS resource]. https://inscribe.education/ (Proctorio Behavior Settings resource)
16. Proctorio — "Proctorio Secure Companion App" support article (Advanced Security; closes apps; macOS Assessment API) [vendor]. https://proctorio.com/support/hc/articles/1752083784489-proctorio-secure-companion-app
17. Oregon State Ecampus — "Proctorio Automated Proctoring – Recommended Settings" (upd. 2025-12) [institutional; Advanced VM/Program/Hardware Detection require Companion App; frame-metric weighting]. https://ecampus.oregonstate.edu/faculty/exam-proctoring/Proctorio%20Recommended%20Settings.pdf
18. Proctorio changelog (changes.proctorio.com) — Companion App flow; Parallels VM flag; updated facial-detection model; "Allow Apps" needs Advanced Program Detection [vendor]. https://changes.proctorio.com/ ; https://changes.proctorio.com/april-25-2025-320204 ; https://changes.proctorio.com/october-3-2025-324807
19. Fracture Security Research — "How Proctorio Detects Browser Extensions" **[adversarial/low-trust; pointer only]** (extension-enumeration via chrome.management; cannot enumerate macOS apps). https://www.fracturebrowser.com/guides/how-does-proctorio-detect-extensions

ProctorU / Meazure (Guardian):
20. Meazure Learning — "Proctoring in Focus, Part 1" (2024-05-29) [vendor; 30s check, logs all processes, 400+ VM/remote scan, telemetry]. https://www.meazurelearning.com/resources/proctoring-in-focus-part-1-online-exam-security
21. Meazure Learning — "A Closer Look at the Guardian Browser" (2023-04-25) [vendor; 500+ processes, continuous script]. https://www.meazurelearning.com/resources/a-closer-look-at-the-guardian-browser-benefits-functions-and-future-plans
22. Meazure Learning — "ProctorU Platform Myth Busters: 4 Ways We DON'T Invade … Privacy" (2023-05-08) [vendor; hybrid, human-decides, no PII in Guardian, anti-software-only stance]. https://www.meazurelearning.com/resources/proctoru-myth-busters-4-ways-we-dont-invade-our-test-takers-privacy
23. Meazure Learning — "The Guardian Secure Proctoring Browser" / Guardian Browser (2022-08-26 & 2023-03-09) [vendor; VM detection, app/process blocklist, "download two apps," replaces extension Jan 2023]. https://www.meazurelearning.com/guardian-browser ; https://www.meazurelearning.com/exam-technology/proctoru-online-proctoring/guardian-browser
24. Meazure Learning — "Live+" service line (synchronous proctor + intervention specialist; connects via Guardian) [vendor]. https://www.meazurelearning.com/exam-technology/proctoru-online-proctoring/live-plus
25. ProctorU Support — "Unpermitted Applications" (Guardian denylist; monitors hundreds of processes) [vendor]. https://support.proctoru.com/hc/en-us/articles/46589198199565-Unpermitted-Applications
26. BleepingComputer — "Hacker leaks 386 million user records from 18 companies for free" (2020-07-28) [press; ProctorU 444k among ShinyHunters dump]. https://www.bleepingcomputer.com/news/security/hacker-leaks-386-million-user-records-from-18-companies-for-free
27. AUSCERT — "AUSCERT mailout: ProctorU breach" (2020-08-06) [CERT; bcrypt digests, ≤2016 data]. https://auscert.org.au/blogs/2020-08-06-auscert-mailout-proctoru-breach
28. Infosecurity Magazine / iTnews / SBS / Avast — ProctorU breach coverage (2020-08) [press; usernames, names, addresses, hashed passwords; Australian universities]. https://www.infosecurity-magazine.com/news/online-exam-tool-suffers-data ; https://www.itnews.com.au/news/australian-universities-investigate-online-exam-tool-data-breach-551373 ; https://blog.avast.com/proctoru-breach-learnings-avast
29. ProctorU — official "Security Update" statement (2020-08-05) [vendor; legacy directory data, hashed+salted, password resets]. https://www.proctoru.com/security-update

Honorlock:
30. Honorlock Help Center — "Honorlock Extension Permissions and Functions" (upd. 2025-07) [vendor; chrome.management/tabs/desktopCapture/system.display]. https://honorlock.kb.help/honorlock-extension-permissions-and-functions/
31. Honorlock — "BrowserGuard" product page (2023-12-17) [vendor; in-extension lockdown]. https://honorlock.com/browserguard
32. Honorlock Help Center — "How Honorlock Monitors Secondary Devices" (upd. 2025-09) [vendor; audio watermark + mic proximity; Apple Hand-Off; voice transcript]. https://honorlock.kb.help/how-honorlock-monitors-secondary-devices/
33. Honorlock API Docs — "Search and Destroy" [vendor technical; verbatim web search, at-risk/common/unique rules, ≥6-word, DMCA]. https://api-docs.honorlock.com/search-and-destroy
34. Honorlock — "Search and Destroy" product (2023-12-18) [vendor]. https://honorlock.com/search-and-destroy
35. Honorlock Help Center — "Browser Guard and Allowed Site URLs FAQ" / Honorlock Desktop Application (upd. 2026-09) [vendor; desktop app blocks external apps/native AI]. https://honorlock.kb.help/browser-guard-and-allowed-site-urls-faq/
36. Honorlock — "Student Data Privacy & Security" (2026-01-27) [vendor; AWS, 365-day retention, face detection only, optional Application detects prohibited programs]. https://honorlock.com/studentprivacy/
37. ASU Teaching & Learning Center — "Honorlock: Security and Privacy Information" (2025-11-07) + WCU of PA Honorlock page [institutional; extension-only, no network scan, patent pointer]. https://tlc.sols.asu.edu/technology/honorlock-security-and-privacy-information ; https://updates.wcupa.edu/sciences-mathematics/mathematics/honorlock.aspx
38. Honorlock — "Faculty & Student FAQ" (2023-12-17) [vendor; AI → live pop-in; limited-release desktop app]. http://honorlock.com/faculty-student-faq

Cross-cutting / academic:
39. EPIC — "In re Online Test Proctoring Companies" complaint + letters (2020-12-09) [advocacy/legal; names all five]. https://epic.org/documents/in-re-online-test-proctoring-companies ; complaint PDF https://epic.org/wp-content/uploads/privacy/dccppa/online-test-proctoring/EPIC-complaint-in-re-online-test-proctoring-companies-12-09-20.pdf
40. ERIC EJ1481131 — "A Systematic-Narrative Review of Online Proctoring Systems and a Case for Open Standards" (2025-08-08) [academic; data classes, CV algorithms, transparency gap]. https://files.eric.ed.gov/fulltext/EJ1481131.pdf
41. Powell v. DePaul University, No. 1:2021cv03001 (N.D. Ill., 2022-11-04) [court; Respondus Monitor BIPA signal enumeration]. https://law.justia.com/cases/federal/district-courts/illinois/ilndce/1:2021cv03001/404241/50
42. JURIST — "Pennsylvania bar applicants request investigation after ExamSoft data breach" (2020-09-10) [press; sector breach exposure]. https://www.jurist.org/news/2020/09/pennsylvania-bar-applicants-request-investigation-after-exam-software-data-breach

Examity:
43. Examity — "What does our AI do?" (via Chest.ac.uk, 2020) [vendor; TensorFlow.js in-browser real-time video analysis; flag list]. https://www.chest.ac.uk/media/3643/examity-ai-overview-2020.pdf
44. Examity — "Lockdown Browser Feature Guide" (2020-10-08) [vendor; lockdown runs through the existing extension, no extra download]. https://itle.okstate.edu/site-files/documents/examity_lockdown_browser_feature_guide.pdf
45. Examity — "Examity Levels" (2020-08-18) [vendor; Auto ID/Mobile ID/Live ID biometrics; Auto Standard/Premium]. https://www.mtsu.edu/online/wp-content/uploads/sites/72/2024/07/Examity_Levels_Explanations.pdf
46. Examity — "Automated Proctoring Test-taker Guide" (2022) [vendor; install Examity extension per session, self-removes, challenge question]. https://itle.okstate.edu/site-files/documents/examity_auto_proctoring_test_taker_guide_2022.pdf
47. Examity — "Flag System" / reporting + retention (60 days / 1 year on red) [vendor]. https://www.examity.com/flag-system
48. Queen's University Registrar — "Examity" (biometric keystroke signature; facial scan + photo-ID + 3 challenge questions; US-hosted) [institutional]. https://www.queensu.ca/registrar/academic-info/exams/remote-proctoring/examity

Talview:
49. Talview — homepage + Alvy AI Proctoring (continuous liveness/face auth, virtual-camera rule-out, secure browser, auxiliary camera; **US Patent 12,361,115 B1**) [vendor]. https://www.talview.com/en/

Constructor / Examus:
50. Constructor Tech — "Secure browser" product page (VM/USB/mirror/virtual-camera/NVIDIA-Broadcast detection; device control; blended proctoring) [vendor]. https://constructor.tech/products/learning/proctor/secure-browser
51. Constructor Tech — Proctor Developer Guide / API parameters (warning enums: virtual_environment, substitution_user, account_collision, phone, avert_eyes, non-verbal, USB; secureBrowserLevel basic/medium/high; streamsPreset) [vendor technical]. https://docs.constructor.tech/documentation/en/proctor-developer-guide/constructor-proctor-api/api-parameters ; https://docs.constructor.tech/documentation/pdfs/en/proctor-developer-guide.pdf
52. Constructor Tech — Proctor Guide (cheating score 0–100%; gaze via head-pose angle; active-window change; Secure Browser modes) [vendor]. https://docs.constructor.tech/documentation/pdfs/en/proctor-proctor-guide.pdf

Mercer | Mettl:
53. Mercer|Mettl Support — "Anti-Cheating Measures in Mercer|Mettl System" (2020-06-23) [vendor; MSB; AI flag enum incl. MOBILE_PHONE_DETECTED; 60s snapshots; audio speaker-count; 360° scan; OTP/IP]. https://support.mettl.com/portal/en/kb/articles/proctoring-basics-23-6-2020
54. Mercer|Mettl — "Safe & Secure Exam Browser" (MSB detects VM & blocks proceed; bars screen projection; three-point auth; human-AI tiers) [vendor]. https://mettl.com/en-gb/safe-and-secure-exam-browser/
55. Mercer|Mettl — "Secure Exam Proctor" Chrome extension (Moodle/Canvas; blocks devtools/right-click; AWS hosting; "95% cheating detection") [vendor]. https://mettl.com/en/secure-proctor

Prior-slice sources reused (not re-listed): PSI Bridge [T07 S7-S9, S13, S14]; Prometric ProProctor
[T07 S10-S12]; Talview TSB [T07 S21-S22, S32]; Constructor/Examus matrix [T07 S23-S24, S28, S49] and
[T11 8-10]; Mercer Mettl matrix [T07 S4-S6]; ProctorU/Guardian matrix [T07 S15-S16] and [T11 1-4];
Honorlock matrix [T07 S37-S39]; Examity [T07 S44]; OnVUE baseline [T07 S33-S36]; VM-heuristic/attestation
cross-cutting [T07 S46-S48, S51].
