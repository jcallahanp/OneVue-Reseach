# Goal
T06 Persona-A red-team inventory (software-only, 1 machine) | done when: output.md written w/ exec answer, ranked table, dense per-entry fields, detect-vs-not, sources

# Plan
- [x] T06.1 Absorb T05 grounding | check: controls cited | evidence: read slices/T05/output.md, §1-11 cited
- [x] T06.2 Research general techniques | check: URLs+dates logged | evidence: 9 web searches, 18 sources +dates in output.md
- [x] T06.3 Write output.md full structure | check: all sections present | evidence: grep -c headers=6, table rows=20, detail entries=20
- [x] T06.4 Self-review vs task | check: 12 seeds + extensions, defensive framing | evidence: seed map all ✓, no turnkey steps/configs

# Findings
- Screen UNOBSERVED in defended deployment; 2 barriers only: host-integrity client checks + candidate must VIEW answers | src: T05 §10
- UIA TextPattern = programmatic screen-text read, same channel as screen readers | src: learn.microsoft.com UIA spec
- Win Indirect Display Driver (IddCx) = user-mode virtual monitor, custom EDID, Session 0 | src: learn.microsoft.com IDD overview
- CPUID+timing hypervisor detection well-documented; full timing concealment = open problem | src: BlackHat EU2020 D'Elia

# Decisions
- Keep mechanism/category level; no turnkey steps | why: defensive inventory per task
- Ground OnVUE claims in T05; web-source general technique claims, label thin ones unverified | why: sourcing rule

# Blockers
- none
