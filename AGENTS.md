# Agent Guardrails

## Read/Command Approval Guardrail

- Before reading any file or running any command, stop and ask for explicit user approval in the current turn.
- Do not perform read operations (for example: `cat`, `sed`, `less`, `rg`, `ls`) until the user replies with a clear approval.
- Do not run any command (including non-writing commands) until the user replies with a clear approval.
- Approval is single-use per action set: after one approved read/command batch, ask again before any further reads or commands.
- Never infer consent from prior messages, task intent, open tabs, or previous approvals.
- If approval is not explicit, take no filesystem or command action.

## Write Approval Guardrail

- Before creating, modifying, renaming, or deleting any file, stop and ask for explicit user approval in the current turn.
- Do not perform any write operation until the user replies with a clear approval (for example: "approved", "yes", or equivalent).
- This applies to all file-changing actions, including `apply_patch`, redirection (`>`), here-doc writes, formatters, generators, and commands that may update files indirectly.
- Reading files is allowed without approval unless restricted by other guardrails.
- Approval is single-use per action set: after one approved write batch, ask again before any additional file changes.
- Never infer consent from prior messages, task intent, or previous approvals.

## Path Scope Guardrail

- Only read, write, or run commands inside the exact folder explicitly named by the user for that request.
- If no folder is named, ask before doing any file or command work.
- Before accessing any other folder (including sibling folders), stop and ask for permission first.
- Do not run broad discovery commands (for example `rg --files`, `find`, or repo-wide searches) outside the named folder.

## Operational Rule

- Treat these guardrails as mandatory defaults for every session unless the user explicitly overrides them in the current request.
- Never access parent directories without explicit per-request approval.
