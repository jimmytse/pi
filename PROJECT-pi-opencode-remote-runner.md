# PROJECT-pi-opencode-remote-runner.md

## 1. Project Title

**Pi Remote Task Runner for OpenCode — Telegram-to-Home-Server Workflow**

## 2. One-sentence Pitch

Let the user submit a Claude-written OpenCode task from a phone through Telegram, have a constrained Pi agent on the Ubuntu Server validate and dispatch it to OpenCode, and receive progress, results, and approval requests without manually relaying messages between tools.

## 3. Problem Statement

The current workflow requires manually copying task instructions between Claude, Telegram, and OpenCode running on a home Ubuntu Server LTS machine. While away from home, the user cannot reliably initiate, monitor, or resume work without this relay.

Pi is being evaluated as a lightweight agent that can run with a weak local model and follow strict rules/playbooks. Pi should be the controlled workflow/orchestration layer, not a second unrestricted coding agent or a replacement for OpenCode. OpenCode remains the coding executor and retains its existing model/provider setup.

The project must reduce manual message handling while preserving predictable behavior, resumability, security, and visibility into what actually ran.

## 4. Target Customer (ICP)

Initial user: a solo developer/product builder running OpenCode on a personally managed Ubuntu Server LTS machine, using a phone while away from home, and relaying task instructions from a stronger assistant through Telegram and GitHub.

Future ICP: solo developers and small teams running coding agents on self-hosted machines who need a mobile-friendly, auditable, policy-constrained task queue.

## 5. Proposed Solution

Build a small, modular Pi-based workflow that receives a task envelope through Telegram, validates it against a strict playbook, stores its state durably, and invokes OpenCode through a narrowly defined adapter. The system sends progress, completion summaries, errors, and approval requests back through Telegram. GitHub remains the source of truth for code and task artifacts where appropriate.

### Intended end-to-end workflow

1. The user obtains a task message from Claude and sends/forwards it to the private Telegram bot. The first version may require one deliberate paste; it must eliminate repeated copying between Telegram and the home server.
2. The bot accepts messages only from configured Telegram user IDs and creates a task record with a unique ID. Message content is task data and cannot override system policy.
3. Pi parses the request into a small task envelope: task ID, repository/workspace, objective, acceptance criteria, allowed actions, and approval requirements. If required information is missing or ambiguous, ask a concise follow-up instead of guessing.
4. A deterministic policy/playbook validates the task, repository allowlist, working-tree state, command category, time/resource limits, and whether human approval is required.
5. Pi queues the task durably and dispatches it through an OpenCode adapter. The technical lead must verify the installed OpenCode CLI's supported non-interactive interface and exit/status behavior; do not assume undocumented flags.
6. Pi reports accepted, queued, running, blocked-for-approval, succeeded, or failed status to Telegram. Long-running work must not depend on the Telegram request remaining open.
7. On completion, Pi returns a concise summary, validation/test results, changed-file/commit/PR references when available, and the path to full logs. Distinguish verified results from model claims.
8. The user can query status, request a safe stop, approve/reject gated actions, or resume/retry a failed task by task ID. Retries must not silently repeat non-idempotent actions.

### Operating principles

- **Pi coordinates; OpenCode codes.** Keep responsibilities explicit.
- **Deterministic controls before model judgment.** The weak local model may classify, clarify, and summarize, but it is not the security boundary.
- **Playbook-first execution.** Every task uses a named, versioned workflow with explicit allowed and forbidden actions.
- **Fail closed.** Invalid identity, malformed task, unknown workspace, failed validation, or uncertain execution state stops safely.
- **Modular and replaceable.** Keep Telegram intake, task store/queue, policy/playbooks, Pi runtime, OpenCode adapter, and notifications behind clear boundaries. Claude chooses implementation details after inspecting the repository and host constraints.
- **No paid cloud-model dependency for the basic control loop.** Validate with a weak local model; use deterministic parsing/checks wherever possible.

## 6. Monetization Model

Initially this is an internal productivity project, not a validated standalone business. If it proves reliable, possible later paths include a paid self-hosted setup service, support/maintenance, or a productized agent-runner package for small engineering teams. Do not build billing, multi-tenancy, or SaaS infrastructure in the MVP.

## 7. Why This Can Make Money

The immediate value is reduced context switching and faster remote task execution. A reusable, secure workflow for controlling coding agents on self-hosted machines may later serve developers and small teams who want automation without handing repositories to a hosted platform. Commercial potential is unvalidated; prioritize solving the user's workflow and collecting reliability evidence before productizing.

## 8. Core Features (MVP) – 5–8 max

1. **Private Telegram intake and identity allowlist** — accept commands only from explicitly configured Telegram user IDs; reject all others.
2. **Structured task envelope and clarification** — capture objective, repository, acceptance criteria, limits, and approval requirements.
3. **Strict playbooks and policy checks** — start with one coding-task playbook; enforce workspace allowlists, permitted command categories, timeouts, and approval gates outside the LLM.
4. **Durable queue and task state** — unique task IDs, deduplication, explicit state transitions, recoverable state after restart, and protection against duplicate execution.
5. **OpenCode execution adapter** — invoke only a verified, supported OpenCode interface; capture exit status, output, timeout, cancellation, and execution identity.
6. **Telegram status, approvals, and recovery controls** — acknowledge receipt, report milestones/final result, request approval where required, and support status/stop/retry by task ID.
7. **Audit trail and verification report** — record task source, policy decision, timestamps, adapter metadata, exit status, tests run, and relevant GitHub artifact links; redact secrets.

## 9. Differentiation

- Designed around a deliberately weak local model rather than assuming a powerful model can reason through every task.
- Separates probabilistic model assistance from deterministic policy enforcement.
- Uses the existing OpenCode setup and Telegram habit instead of replacing the coding tool or requiring a web dashboard.
- Uses small, inspectable playbooks and auditable task records rather than unrestricted autonomy.

This is a workflow integration, not unique underlying technology. Telegram bots, coding-agent CLIs, queues, and agent frameworks already exist; differentiation depends on reliability, low resource use, strict controls, and easy self-hosting.

## 10. Go-to-Market Outline

Not applicable to the first internal MVP. If reliability is demonstrated, document installation and threat model, publish a repeatable demo, recruit a small number of self-hosting developers for feedback, and test willingness to pay for setup/support before building a commercial product.

## 11. Success Metrics

MVP acceptance targets:

- At least 9 of 10 valid test tasks are received and assigned exactly one task ID.
- No task from an unapproved Telegram user reaches execution.
- Restarting Pi does not lose queued task state or silently re-run a task whose execution outcome is uncertain.
- A normal task can be submitted, executed, and reported without SSH access or manually copying messages to the server.
- Every task ends in an explicit state: succeeded, failed, cancelled, or blocked; no silent disappearance.
- Completion reports include actual process exit status and validation evidence when available.
- Dangerous/out-of-scope actions are rejected or held for approval in all defined negative tests.
- Track median time to acknowledgement, completion-notification delivery, retry rate, and human interventions required.

## 12. Risks & Assumptions

- **OpenCode interface uncertainty:** inspect the installed version and documented CLI/headless capabilities first. If reliable non-interactive execution is unsupported, stop and propose the smallest viable alternative before implementing.
- **Weak-model reliability:** never rely on the local model for policy enforcement or deciding whether a security rule can be ignored.
- **Prompt injection:** Claude task text, repository files, tool output, and Telegram messages are untrusted data. None may override system rules or expand the workspace allowlist.
- **Remote command exposure:** Telegram is a control channel, not a general-purpose shell. Do not implement arbitrary shell commands from chat.
- **Credential leakage:** never return tokens, environment variables, private keys, or raw secrets in Telegram or logs. Keep bot credentials out of Git.
- **Unsafe mutations:** begin with a designated workspace and explicit approval rules for destructive operations, dependency changes, pushes, merges, deployments, and actions outside that workspace.
- **Concurrent/duplicate tasks:** enforce one active task per workspace initially; use durable deduplication and reconcile uncertain execution before retry.
- **Network/host downtime:** queue or reject predictably and recover after restart.
- **Long-running tasks:** use a persistent worker/process lifecycle and polling or events, not one long-lived Telegram request.
- **Repository fit:** this is the upstream Pi monorepo. Keep the integration separate and avoid changing Pi core unless inspection proves a core change is necessary. Inspect the separate `pi-chat` integration mentioned in the upstream README before building Telegram transport from scratch.
- **GitHub scope:** do not commit secrets, local state, generated logs, or private task contents into a public repository.

## 13. Handoff Notes for Technical Lead

Claude should begin with repository and environment discovery, then implement small, independently testable milestones. Do not begin by building a general-purpose autonomous agent.

### Milestone 0 — Discovery and design checkpoint

- Inspect the Pi monorepo structure, contribution rules, agent instructions, package conventions, existing Telegram/chat integrations (including the separate `pi-chat` repository referenced by the README), durable-state facilities, and examples.
- Confirm the Ubuntu Server LTS environment, Node.js/Pi versions, local model/provider, Telegram bot setup, and installed OpenCode version/interface. Ask only for information that cannot be safely discovered.
- Check whether existing chat integration can be reused.
- Write a brief design decision note listing the integration boundary, constraints, and blockers. Do not invent architecture before this inspection.
- Work in a feature branch/fork; do not push implementation directly to upstream main.

### Milestone 1 — Contract and deterministic core

- Define a versioned task envelope and explicit state machine.
- Define the first playbook: a bounded coding task in one configured repository, with acceptance criteria and a defined verification command set.
- Implement validation, policy decisions, task IDs, deduplication, state persistence, and redaction without requiring an LLM.
- Test valid/invalid envelopes, unauthorized users, duplicate submissions, state transitions, restarts, and policy-denied actions.

### Milestone 2 — OpenCode adapter proof

- Prove one minimal task can be launched through the installed and documented OpenCode interface in a controlled test workspace.
- Capture stdout/stderr safely, exit status, timeout, cancellation, and process lifecycle.
- Do not assume command flags; document the exact verified invocation and version.
- Add integration tests with a stub/fake executor so most tests do not require a live model or real repository mutations.

### Milestone 3 — Telegram intake and status

- Connect authenticated Telegram intake to the task contract and durable queue.
- Implement acknowledgement, status, progress, completion/failure report, stop, and approval/rejection flows.
- Enforce allowlisted user IDs and private-chat checks; validate callback/task ownership.
- Test restarts, network interruptions, repeated message delivery, malformed messages, unauthorized senders, and long-running tasks.

### Milestone 4 — Weak-local-model and playbook validation

- Run the workflow with the intended weak local model.
- Keep model responsibilities narrow: parse/clarify intent and summarize outcomes. Deterministic code owns permissions, transitions, and command constraints.
- Test adversarial task text, injected instructions in repository content, ambiguous requests, context limits, malformed model output, and model unavailability.
- If parsing fails, ask for clarification or reject safely; never fall back to unrestricted execution.

### Milestone 5 — Pilot, operations, and handoff

- Provide a safe setup guide with environment-variable names/placeholders only, Telegram bot setup, service lifecycle, logs, backup/recovery, upgrades, and uninstall steps.
- Add dry-run mode and a visible kill switch.
- Run an end-to-end phone pilot: submit a low-risk task, inspect status, restart the service with a queued task, verify the result, and confirm no duplicate execution.
- Deliver tests, limitations, example task messages, a threat model, and a short operator runbook.
- Keep modules replaceable without changing the task contract.

### Definition of done

The user can send a Claude-written task to the private Telegram bot while away from home; the home server validates and queues it; Pi follows the selected playbook and invokes OpenCode through a verified interface; the user receives truthful status and a result report; restarts and duplicate messages are handled safely; and policy/security tests pass. The MVP does not need a web dashboard, multi-agent collaboration, autonomous GitHub merging, SaaS hosting, or general-purpose remote shell access.
