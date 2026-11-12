# Goal
See Goals.MD

# Plan
- [x] T01 Create CLAUDE.md ≤50 lines | check: wc -l CLAUDE.md ≤ 50 | evidence: wc -l → 45; reviewer gaps fixed
- [x] T02 Create Goals.MD, Progress.MD, Research.MD, slices/ | check: files exist | evidence: ls → all present
- [x] T03 User supplies specific concerns | check: Goals.MD Concerns populated | evidence: C1-C3 + threat model set
- [x] T04 User defines done-conditions G1–G3 | check: no "TBD" | evidence: done-whens + Persona A/B set
- [x] T05 OnVUE control surface (grounding) | check: slices/T05/output.md, sourced | evidence: 129 lines, 16 sources, merged to Research.MD
- [x] T06 Persona A red-team inventory (first slice) | check: slices/T06/output.md | evidence: 160 lines, 20 techniques ranked, 18 web srcs + T05, merged to Research.MD
- [x] T07 Vendor alternatives control inventory | check: slices/T07/output.md, sourced | evidence: 195 lines, 51 sources, 13 vendors, merged to Research.MD
- [x] T08 Persona B synthesis (rules+procedures+vendor) | check: slices/T08/output.md | evidence: 154 lines, 6 sections, 11 rules + vendor pick + roadmap, merged to Research.MD
- [x] T09 G3 bypass-test designs for top strategies | check: slices/T09/output.md | evidence: 14 tests (12 treatment+2 control), design-only, merged to Research.MD
- [x] T10 Final verification vs done-conditions | check: reviewer PASS | evidence: PASS-WITH-GAPS; G1-G3 + Persona A/B all PASS; 3 sourcing gaps flagged
- [x] T11 Re-verify load-bearing vendor claims | check: slices/T11/output.md | evidence: Guardian ~30s re-scan CONFIRMED (vendor-asserted); overlay-detect leg corrected; Simko relabeled; corrections appended to Research.MD
- DONE: G1-G3 met; Persona A+B answerable; verification PASS; sourcing gaps fixed. Deliverables in slices/ + Research.MD.
- [x] T12 Test execution authorization plan (user-requested) | check: slices/T12/output.md | evidence: 10 sections, no Pearson pentest path found, Branch A/B, tiering, merged to Research.MD
- [ ] T13 Deepen methodology #1 under center/low-compute/4h model | check: slices/T13/output.md | agent: running
- [ ] T14 Deepen Type-1 hypervisor: tools, feasibility, prevention | check: slices/T14/output.md | agent: running

# Findings
- Research findings live in Research.MD, not here.
- T05 pivot: proctor never sees screen → only OnVUE client-side checks guard the second workspace. This is the central G1 result.

# Decisions
- T10 reviewer: PASS-WITH-GAPS. Fixing the 1 load-bearing correctness gap (Guardian continuous-scan) via T11 before final done | why: user rule = fix correctness gaps before done

- Subagents write only to own slice; orchestrator merges into Research.MD | why: avoid parallel write conflicts
- Bypass tests: design only; execution needs user approval | why: user asked to design; no assumed intent
- Done-conditions left TBD | why: not stated by user

# Blockers
- (all prior blockers resolved by user's 2026-11-12 scope message)
- OPEN: user's exact exam platform/sponsor + OS mix unknown | needs: user (affects T09 test realism; not blocking design)
- NOTE: T06 subagent claimed it edited Progress.MD; it did not (lane held). No corruption.
