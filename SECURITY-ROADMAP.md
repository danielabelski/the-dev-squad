# Security Roadmap

This document is the implementation plan for moving The Dev Squad from "guardrails around a supervisor-led dev team" toward stronger user protection.

The current hook-based model is useful, but it is not a security boundary against an adversarial agent. The path forward is layered, not container-first.

## Current Stance

The Dev Squad is **not a sandbox.** The `PreToolUse` hook is a discipline guardrail, not OS-level isolation. Anyone who needs strong containment against a hostile or jailbreak-prone agent should run The Dev Squad inside a VM they own.

We are honest about this rather than promising containment we cannot reliably ship. Earlier versions of this roadmap promised a `v0.4` Docker-based sandbox runner. That work was attempted, the runner abstraction and DockerRunner code remain in the tree, but Claude Code subscription auth inside containers proved unreliable enough that we cannot make sandboxed execution the default. Until that upstream blocker changes, we are not going to keep advertising sandboxed execution as a near-term feature.

What we DO ship as practical safety today:

- per-agent role guardrails enforced by the hook
- strict mode for human-mediated Bash approval on Coder/Tester
- the optional Security Audit (Agent E) as a final read-only OWASP-class pass before deploy
- per-finding user-controlled fix loop instead of automatic fixes
- explicit user-gated deploy after the audit
- Claude Code's own `--permission-mode auto` safety classifier on every session

## Current Baseline

- Fast mode is the default execution mode and is optimized for speed.
- `pipeline/.claude/hooks/approval-gate.sh` constrains tool use, but it lives in an agent-writable tree and cannot fully control team members that still have Bash.
- `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1` mitigates `cd`-then-`Write` drift, but it does not make Bash read-only.
- The Security Audit (Agent E) is a read-only pass that flags OWASP-class issues. Findings go to the user, never auto-merged. The user decides per finding whether to send a scoped fix to Agent C, dismiss, or ignore.
- Remaining known gaps are documented in [SECURITY.md](SECURITY.md): Bash-mediated writes, indirect execution, hardlinks, TOCTOU, and accepted planning/review web egress.

## Target Modes

| Mode | Goal | User Experience | Residual Risk |
|------|------|-----------------|---------------|
| **Fast** | Keep the current autonomous workflow | Safe Bash stays automatic, obviously dangerous Bash asks for approval | Guardrails only, not strong containment |
| **Strict** | Put a human in the loop for all coder/tester Bash | Every Bash call from the Coder and Tester requires approval | Better practical safety, but still not a true sandbox |
| **Security Audit (E)** | Optional read-only OWASP-class audit before deploy | Toggle at build start; audit runs after testing; user reviews findings and chooses what to fix | Not isolation. Catches things review/test missed; user-gated. |

## Phase `v0.3`: Strict Mode (shipped)

### Goal

Make coder/tester Bash usage human-mediated instead of pattern-mediated.

### Status

Shipped in `v0.3.0`:

- Pipeline runs can now choose `Fast` or `Strict`
- Strict mode asks for approval on every coder/tester Bash call
- The selected mode is persisted in pipeline state and passed through spawned sessions
- Approval decisions are tied to explicit request records instead of "latest project wins"
- Approved Bash commands receive a one-time grant for the exact command that was approved

### What Shipped in `v0.3`

- Visible security mode selector in the viewer for pipeline runs: `Fast` or `Strict`
- Selected mode persisted in pipeline state so the UI, orchestrator, and approvals agree on the current policy
- Strict-mode flag passed through all pipeline Claude spawn paths
- In strict mode, every Bash call from the Coder and Tester requires approval
- Approval UI shows the pending command, agent, cwd, and project
- Approval requests and decisions are logged into `pipeline-events.json`

### Why This Mattered

Strict mode closes the biggest day-to-day bypass class without forcing a container project first. It fits the current product: the UI already has an approval surface, and the supervisor/team workflow already tracks project state.

## Phase `v0.4`: Optional Security Audit (Agent E) (shipped)

### Goal

Add a final OWASP-class pass over the finished, tested code before deploy. Surface concrete findings to the user with severity rankings. Let the user decide what to fix, what to dismiss, and when to deploy.

### Status

Shipped in `v0.4.0`:

- Build-start toggle: "Security Audit: Off / On" (default Off)
- New Agent E (Security Auditor): read-only, no Bash, no Write/Edit, no WebFetch/WebSearch, no Agent tool
- Pipeline ordering when ON: A → B → C → D (review) → D (testing) → **E (audit) → user-gated review → optional scoped fix loop with C+D+E re-audit → user-gated Deploy → commit + open**
- Severity ranking on every finding (`critical` / `high` / `medium` / `low`) calibrated by exploitability and prerequisites
- Per-finding user actions: Send to C (scoped fix → D verifies tests → E re-audits that one finding) or Dismiss (logged)
- Dedicated Security Audit panel in the Office View with findings list, chat with E, and gated Deploy button
- Deploy is user-explicit (modal confirms before commit)

### What This Is NOT

- Not isolation
- Not a sandbox
- Not an automatic fix loop (intentionally — fix loops without user input were rejected)
- Not a replacement for human code review on sensitive changes

### What This Is

A calibrated final-pass advisory the user can act on without leaving the team workflow.

## Phase `v0.5`: Host-Owned Policy Service (planned, no ship date)

### Goal

Move trust out of the repo so protection does not depend on files the agent can edit, rename, or replace.

### Deliverables

- Replace repo-local policy ownership with a host-owned policy service or wrapper
- Store approvals, denials, and audit records outside the project directory
- Identify sessions and approvals by stable IDs instead of path scanning
- Keep a tamper-resistant audit log of:
  - command
  - cwd
  - agent
  - decision
  - timestamp
- Add an allowlisted execution path for common actions like install, build, and test so fewer tasks need raw Bash

### Files Likely To Change

- [pipeline/orchestrator.ts](/Users/johnknopf/Projects/the-dev-squad/pipeline/orchestrator.ts) — delegate policy and approval state to the host-owned layer
- [src/app/api/pending/route.ts](/Users/johnknopf/Projects/the-dev-squad/src/app/api/pending/route.ts) — read from a stable policy store
- [src/app/api/approve/route.ts](/Users/johnknopf/Projects/the-dev-squad/src/app/api/approve/route.ts) — write decisions to the same store
- [pipeline/.claude/hooks/approval-gate.sh](/Users/johnknopf/Projects/the-dev-squad/pipeline/.claude/hooks/approval-gate.sh) — either shrink to a thin adapter or remove it from the trust boundary entirely

### Acceptance Criteria

- Renaming or deleting `.claude/` inside a project does not disable policy enforcement
- Approvals survive project rename/copy operations because they are not keyed by "latest project"
- A user can inspect approval history and active requests per project
- The repo can truthfully claim that the main enforcement boundary lives outside the agent-writable workspace

## Container/Sandbox Status

The runner abstraction is in place (`pipeline/runner.ts`) and includes both a `HostRunner` and a `DockerRunner`. The Docker code path works in narrow conditions but cannot be made the default because Claude Code subscription auth inside containers is not reliable enough.

We are not deleting the runner code. We are deleting the *promise* that sandboxed execution is the next major user-facing feature. If upstream auth-in-container becomes reliable later, the abstraction is ready and we can revisit.

If you need OS-level isolation today, the recommended approach is to run The Dev Squad inside a VM you own.

## Non-Goals

These are not realistic promises for the current architecture:

- Perfect protection with unrestricted Bash and no OS isolation
- Perfect detection of every indirect execution trick with grep or regexes
- Perfect human review of every command
- Sandboxed execution as a default mode without solving subscription auth in containers

## Reporting

If you find a security issue, please open a private issue or contact the maintainer directly.
