# Windows Toolchain Forensics

A forensic agent skill for diagnosing fragmented Windows development environments where installed state, shell behavior, executable resolution, package-manager claims, editor behavior, or prior-agent changes no longer agree.

The skill is built for cases where ordinary "reinstall it" troubleshooting is too destructive or too shallow. It starts read-only, separates observed evidence from inference, finds the earliest broken layer, and only moves into remediation after the proposed change and rollback path are explicit.

## When to use it

Use Windows Toolchain Forensics when the environment is inconsistent rather than simply missing one known dependency. Typical signals include:

- a command works in one shell or editor but not another;
- `PATH`, shims, wrappers, or executable precedence are ambiguous;
- an installer or package manager reports success but the runtime still fails;
- multiple runtimes or toolchains are interfering with one another;
- a previous agent or repair attempt claims success that current machine behavior does not support;
- shell profiles, registry policy, editor injection, WSL boundaries, or project-local environments may be changing behavior.

Casual descriptions such as “my PATH is cursed,” “VS Code works but the terminal doesn’t,” or “I don’t know what is actually installed anymore” are intentional trigger cases.

## Operating model

The core skill uses three explicit states:

1. **INSPECTION** — read-only evidence collection.
2. **GUIDED** — concrete remediation is proposed with rollback information.
3. **EXECUTION** — state-changing commands are permitted only after explicit approval.

A failure, timeout, ambiguity, or cancellation returns the workflow to inspection rather than allowing a partially verified repair to drift forward.

Findings are labeled as **Observed**, **Strong inference**, **Weak inference**, or **Unknown** so inaccessible state is not silently treated as absent.

## Diagnostic layers

The forensic workflow works from upstream constraints toward downstream symptoms:

1. policy and security blockers;
2. `PATH` and environment precedence;
3. package-manager, shim, and wrapper integrity;
4. runtime and dependency coherence;
5. shell, profile, editor, and registry injection;
6. project-local isolation such as virtual environments, version managers, repository configuration, and WSL boundaries;
7. prior-agent drift and unverifiable changes.

The skill explicitly distinguishes Windows PowerShell 5.1 from PowerShell 7+, checks CMD `Command Processor\\AutoRun` state, and treats capability probes as stronger evidence than metadata alone when validating runtimes or GPU/toolchain behavior.

## Verification discipline

A successful command is not enough to close a repair. Environment-policy changes are checked across three surfaces when relevant:

- the underlying registry or user-policy state;
- a fresh external PowerShell or CMD process;
- the current long-lived host, such as an editor or agent session that may have inherited stale environment state.

The repaired command or workflow is then re-run in every affected context, with residual risks and intentionally untouched state kept explicit.

## Repository layout

- [`SKILL.md`](./SKILL.md) — trigger conditions, operating rules, state transitions, workflow, and output contract.
- [`references/PLAYBOOK.md`](./references/PLAYBOOK.md) — full staged forensic procedure.
- [`references/RED-FLAG-INDEX.md`](./references/RED-FLAG-INDEX.md) — symptom-to-root-cause lookup and high-priority branches.
- [`references/BASELINE-ARTIFACT-TEMPLATE.md`](./references/BASELINE-ARTIFACT-TEMPLATE.md) — post-repair environment baseline template.
- [`references/NOTES.md`](./references/NOTES.md) — deployment and host-capability guidance.
- `mcp-main/` — a separate Microsoft Learn MCP Server source tree present in this repository. The root forensic skill does not depend on it for its operating workflow.

## Output contract

The skill reports work in a stable structure:

1. Situation Snapshot
2. Verified Facts
3. Likely Fragmentation Points
4. Root-Cause Ranking
5. Next Safe Actions
6. Remaining Unknowns
7. Definition of Done

This keeps diagnosis, proposed mutation, uncertainty, and closure evidence distinguishable instead of collapsing them into a generic troubleshooting narrative.
