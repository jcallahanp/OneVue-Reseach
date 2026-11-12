# Goal
See Goals.MD

# Plan
- [x] T01 Create CLAUDE.md ≤50 lines | check: wc -l CLAUDE.md ≤ 50 | evidence: wc -l → 45; reviewer gaps fixed
- [x] T02 Create Goals.MD, Progress.MD, Research.MD, slices/ | check: files exist | evidence: ls → all present
- [x] T03 User supplies specific concerns | check: Goals.MD Concerns populated | evidence: C1-C3 + threat model set
- [x] T04 User defines done-conditions G1–G3 | check: no "TBD" | evidence: done-whens + Persona A/B set
- [x] T05 OnVUE control surface (grounding) | check: slices/T05/output.md, sourced | evidence: 129 lines, 16 sources, merged to Research.MD
- [ ] T06 Persona A red-team inventory (first slice) | check: slices/T06/output.md | agent: running (needs T05 ✓)
- [ ] T07 Vendor alternatives control inventory | check: slices/T07/output.md, sourced | agent: running
- [ ] T08 Persona B synthesis (rules+procedures+vendor) | check: slices/T08/output.md | needs: T06,T07
- [ ] T09 G3 bypass-test designs for top strategies | check: slices/T09/output.md | needs: T06

# Findings
- Research findings live in Research.MD, not here.
- T05 pivot: proctor never sees screen → only OnVUE client-side checks guard the second workspace. This is the central G1 result.

# Decisions
- Subagents write only to own slice; orchestrator merges into Research.MD | why: avoid parallel write conflicts
- Bypass tests: design only; execution needs user approval | why: user asked to design; no assumed intent
- Done-conditions left TBD | why: not stated by user

# Blockers
- Specific concerns not provided | needs: user
- Done-conditions for G1–G3 undefined | needs: user
- Target product name ("OneVue" vs "OnVUE") not confirmed | needs: user
- Which exam/platform is "my exam" | needs: user
- Kind of alternatives to examine not specified | needs: user
