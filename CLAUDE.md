# CLAUDE.md — OneVue Research

## Mission
- Assess Pearson OneVue feasibility as a proctoring service against the user's specific concerns.
- Examine alternatives.
- Run deep research. Design tests to determine whether strategies exist to bypass the user's exam.
- Scope and concerns are defined only in Goals.MD. Do not infer them.

## State Files (nothing lives in session memory)
- Goals.MD — all goals and done-conditions. Edited only with user approval.
- Progress.MD — plan, task status, decisions, blockers toward each goal.
- Research.MD — every research finding, with source. Nothing researched goes unlogged.
- Session start: read all three + git log. Update after every finding, decision, or task.

## Roles
- Orchestrator: exactly one, assigned by the user. Coordinates only; does not do task work.
- Orchestrator splits goals into small atomic tasks: one question or change, one check.
- One subagent per atomic task. Subagents execute directly; they do not spawn agents.
- Subagent brief contains only: task, check, slice folder path, required inputs. No other context.

## Lanes and Slices
- Each task gets its own lane and slice folder: slices/T<NN>-<short-name>/
- A subagent reads and writes only inside its own slice folder.
- Subagent output: slices/T<NN>-<short-name>/output.md (findings, sources, evidence).
- Orchestrator merges slice output into Research.MD and Progress.MD.

## Research Rules
- Every claim cites a source URL or file:line, with access date.
- Unchecked claims are labeled "unverified". Never fabricate sources, quotes, or results.
- Prefer primary sources: vendor docs, policies, specs, advisories, peer-reviewed work.

## Bypass Test Design
- Tests target only the user's own exam and setup.
- Each test states: hypothesis, method, pass/fail criterion.
- Design only. Running any test requires explicit user approval.

## Intent
- Do not make assumptions about the user's intent.
- Unclear scope, concern, or criterion: ask the user; log it under Blockers in Progress.MD.

## Communication
- Respond to the user concisely: `Current: <state>` / `Need: <action from user>`.

## Git
- Commit after each completed task. Push only to the assigned dev branch.
