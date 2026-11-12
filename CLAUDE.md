# CLAUDE.md — OneVue Research

## Mission
Help me as a penetration tester for pearson materialize my research on the latest methodologies of circumnavigating testing restrictions to try to engineer an exact strategy that circumnavigates our current gaps with detailed documentation of the technical gaps that exist and audits to allow targeted improvement and suggestions our partners about potential shortcomings.

## State Files (nothing lives in session memory)
- Goals.MD — all goals and done-conditions. Edited only with user approval.
- Progress.MD — plan, task status, decisions, blockers toward each goal.
- Research.MD — every research finding, with source, including our prior research. For new research Nothing researched goes unlogged. 
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
- Rely on preexisting research conducted to guide direction on routes to explore; they describe current architecture and observed gaps. Uses them to inform deeper research and design simulated tests.
## Bypass Test Design
- Each test states: hypothesis, method, pass/fail criterion.
- Run real simulation tests where possible. Have indepedent agents verify tests are safe and thorough.

## Process
- Plan, Delegate, Execute, Indepently Verify. Always plan tests to ensure comprehensiveness. Always check tests with deterministic pass fails
- Use Indepedent agents for atomic tasks, with periodic check gates run by fresh agents.

## Intent
- Unclear scope, or criterion: ask the user; log it under Blockers in Progress.MD.

## Communication
- Respond to the user concisely: `Current: <state>` / `Need: <action from user>`.
  
## Git
- Commit after each completed task. Push only to the assigned dev branch.
