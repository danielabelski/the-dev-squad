# TODO

## Next Up

- Finish Squad View polish:
  - tighten the left rail and team list
  - reduce badge noise in the center header
  - keep improving the chat transcript so it feels closer to a real terminal/chat hybrid
- Improve supervisor-native recovery:
  - make approval and failure flows read more like manager guidance
  - keep reducing cases where users need to inspect raw pipeline details
- Security Audit polish (post-`v0.4.0`):
  - manage the case where the orchestrator crashes mid-action and `auditActionInFlight` stays true (stale-lock detection or manual reset)
  - decide whether the supervisor should nudge the user if all findings are handled but Deploy hasn't been clicked
  - true rollback (e.g. git stash) when a scoped fix breaks tests, instead of just reverting the finding status

## v0.5 Gate (no ship date)

- Host-owned policy service so policy and approvals don't live in agent-writable workspace
- Stable approval IDs that survive project rename/copy
- Tamper-resistant audit log for command/cwd/agent/decision/timestamp
- Allowlisted execution path for common install/build/test ops (reduce raw Bash dependence)

## Keep Honest

- Fast and Strict are the public supported modes today
- The optional Security Audit (Agent E) is live in `v0.4.0` — read-only OWASP-class pass with severity ranking, user-controlled fix loop, and explicit deploy gate
- Sandboxed/isolated execution is **not** an active roadmap item. The Docker runner code remains for narrow cases but cannot be the default until Claude Code subscription auth in containers is reliable. If you need OS-level isolation, run The Dev Squad in a VM you own.
- Manual mode still relies on Claude permission prompts rather than pipeline guardrails
