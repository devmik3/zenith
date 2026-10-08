# Using Zenith from Amp

This manual is for Amp threads developing another repository. The application
repository is the target; this Zenith checkout supplies documentation and,
after approved setup, the runtime. Do not develop the application inside Zenith.

## Compatibility status

Source reviewed: Zenith a8d9b5786f81e70e73cce7bb164f5f83da84a4b5, 2026-10-08.

The proposed starting arrangement is Amp as the MCP orchestrator, with supported
Claude, Codex, or Hermes ACP sessions doing implementation, validation, and
closure review. This is a mixed-provider system, not an all-Amp agent team.

Amp documents MCP support. This repository does not register Amp as a provider,
and this arrangement has not yet passed a live integration pilot. Do not claim
that attaching this repository makes Zenith ready to use.

For small changes, or while integration remains unverified, use the target's
normal Amp workflow. Borrowing Zenith's planning and review methods without
running its runtime is a Zenith-inspired workflow, not a Zenith mission.

## Start here in a target project

Ask the primary Amp thread:

> Read AMP.md in the Zenith additional repository. Keep this repository as the
> target and preserve its existing instructions and tools. Check Zenith
> readiness without installing, initializing, or launching anything. Resume
> existing mission state if available; otherwise propose the smallest setup
> and bounded task for approval.

In an Orb, additional repositories are checked out under `../repos/` relative
to the primary repository. Discover the actual Zenith path; do not assume its
directory name. Only the primary repository's `.agents/setup` runs automatically.
An additional repository does not automatically load this manual, configure MCP,
install adapters, or provide provider credentials.

Locally, keep Zenith in a separate checkout and open Amp in the target repository.

## Readiness and setup

Before proposing execution:

1. Read the target's AGENTS.md, relevant skills, project conventions, current
   branch, and worktree status. Preserve existing work and configuration.
2. Identify the target directory, Zenith checkout and revision, runtime
   environment, provider adapter/version, model choices, and state directory.
3. Find any existing mission pointer and confirm which thread owns it.
4. Check tool availability and authentication status without printing secrets
   or invoking paid agents.
5. Report what is ready, what is missing, and the setup's side effects and cost.
   Obtain approval before installing, configuring, or initializing anything.

Requirements include Python 3.11+, Zenith's Python dependencies, a supported ACP
adapter and its prerequisites, and that provider's authentication. Amp
authentication does not authenticate Claude/Codex/Hermes workers. Confirm billing
and account terms separately; do not assume an Amp subscription covers workers.

After setup approval, configure a project-scoped stdio MCP connection in the
target's `.amp/settings.json`, merging with existing settings. The installed
`zenith-server` entry point accepts:

    --mode orchestrator --transport stdio

Use an absolute executable path from the pinned runtime environment. Amp owns
this stdio process; do not start it as an unrelated background daemon.

Configure a separate absolute `ZENITH_HOME` for each target. Check
`ZENITH_PROJECTS_DIR` too: it overrides the default projects location. Set
`ZENITH_MAX_PARALLEL_NODES=1` initially. Explicitly review the worker, validator,
and terminal-reviewer provider/command settings in `config.py`.

`ZENITH_ORCHESTRATOR_PROVIDER` accepts only registered provider names. It is
provider metadata used by Zenith, not a declaration that Amp is supported.
Do not set it to `amp`, run `zenith init --agent amp`, alias Amp as another
provider, or replace an ACP command with plain `amp`.

Read the bundled orchestrator prompt and relevant playbook as source material.
Adapt them to the approved workflow; do not let their installation, delegation,
or continuation instructions override the target's rules or user checkpoints.
Do not execute the README's automatic-installation prompt for Amp.

Start with direct project MCP configuration. A reusable Amp skill can later
carry this procedure and MCP configuration after the pilot passes. Keep
machine paths, provider choices, and secrets project-specific; do not create a
skill or copy another provider's configuration automatically.

## Keep runtime, target, and state separate

Example local layout:

    ~/tools/zenith/                     pinned Zenith checkout
      zenith/                          Python package/runtime environment
    ~/src/example-app/                 target repository
    ~/.local/state/zenith/example-app/  target-specific ZENITH_HOME
      projects/<project-id>/
        .zenith/                       brief, memory, contracts, evidence
        .zenith-runtime/               state, tasks, structured handoffs

Use the paths returned by Zenith rather than constructing mission paths from
memory. Keep one authoritative approved plan, normally `.zenith/mission.md`.
Contracts and runtime tasks implement that plan; an Amp checklist must not become
a competing plan.

Keep a non-secret resume pointer in the target's existing project-status
location. Record the target path, state location, project/mission IDs, plan path,
owning Amp thread, runtime/adapter versions, latest evidence, and next checkpoint.

`start_project` is not read-only: it creates state and may add skill directories,
copies, and discovery symlinks in the target. Inspect these effects and preserve
existing guidance. Do not commit machine-specific symlinks or private runtime
evidence blindly.

Use a pinned runtime revision and reproducible dependency/adapter versions.
Upgrade between missions, review relevant changes, and repeat the pilot checks.
Do not update a shared runtime beneath active projects.

## New and existing projects

For a new project, establish its repository, conventions, and test commands
normally before introducing a mission.

For an existing project, retain its instructions, skills, MCP settings, models,
and normal build/test workflow. Start with one bounded feature, not a whole-repo
migration. Review existing uncommitted changes before assigning file ownership.

In both cases, use one active orchestrator and one active mission per target
checkout. Separate state directories do not isolate filesystem access.

## Run the approved workflow

Research → Plan → Approve → Build → Verify → User Test → Ship.

1. Research the bounded feature and define observable acceptance criteria,
   non-goals, file ownership, evidence, provider/model choices, and budget.
2. After initialization approval, use `start_project` with the target's absolute
   `workspace_dir`. Record the returned identifiers immediately.
3. Write the authoritative plan and contract assertions. Define coherent work
   tasks, independent validation, and gates. Show the plan for build approval.
4. Only after approval, call `submit_plan`, then `advance_project`. Give workers
   room to finish meaningful tasks within approved scope. Do not require repeated
   permission for routine implementation.
5. Inspect actual diffs and recorded handoffs. Workers and validators must call
   `end_node`; final prose alone is not completion. Validators must check the
   assigned assertions independently, not accept the worker's claim.
6. Resolve attention with evidence. Retry transient failures within the agreed
   limit; patch genuine planning or implementation failures. Do not use
   `continue` to conceal failed acceptance criteria. Seek approval when scope,
   cost, risk, or visible behavior materially changes.
7. Once work and gates are complete, call `end_mission` for independent closure
   review. Require the reviewer's structured `submit_terminal_review` result.
8. Present evidence, remaining limitations, and focused manual-test steps.
   Pause for the user's testing before publication.
9. Commit, push, merge, or deploy only within the user's explicit authorization
   for that action and destination. Zenith's `done` is not shipping approval.

Use existing project test conventions and proportional checks; do not impose
TDD. Include real user-facing behavior where relevant, not only successful
commands. A validator can use the same provider as a worker, but must execute
in a separate session and collect its own evidence.

## Permissions, parallelism, and stopping

Workers share the target working tree; Zenith does not provide isolated
worktrees or automatic integration. Serialize overlapping files and shared
resources. After serial operation is proven, use explicit dependencies and
disjoint ownership for a small parallel batch. The orchestrator owns integration.

Start with one runnable worker or validator at a time. The concurrency setting
alone is insufficient: gate validation can dispatch a batch separately. Avoid
nested worker-created agent swarms.

Claude execution requests permission bypass; Codex requests unrestricted sandbox
access and no approvals. Validators are instructed to be read-only but are not
technically restricted to it. Processes inherit environment credentials and can
access more than the target directory. Use a disposable environment without
production credentials or unrestricted publication access.

`advance_project` blocks and may take minutes. `max_steps=1` limits scheduler
steps, not elapsed time. A disconnected or cancelled Amp tool call is not proof
that execution stopped. `abort_project` records an abort; it is not a reliable
process-kill mechanism and may wait behind an active call.

After interruption, inspect state and actual processes before retrying. Confirm
old execution has stopped, preserve evidence, and reconcile recorded handoffs.
Do not blindly repeat a mutating call or start a replacement mission.

Time, spend, session, and retry limits require operating discipline and, where
available, external enforcement. Stop on exhausted limits, unexplained extra
processes, unsafe permissions, repeated missing handoffs, or unresolved ownership.

## Resume in another Amp thread or Orb

Read the resume pointer and approved plan, establish sole ownership, then call
`inspect_project(project_id)`. Resume the recorded state; do not call
`start_project` merely because this is a new conversation.

A new Orb does not inherit another Orb's uncommitted code, installed runtime,
credentials, state, or running processes. Prefer continuing the existing Orb.
A transfer needs the exact code and state, verified paths and discovery links,
and confirmation that the former execution stopped. Do not hand-edit runtime
cursors to make a transfer appear successful.

The runtime, target, adapters, credentials, and loopback handoff services must
all be reachable in the execution environment. Local success does not prove Orb
compatibility. Do not introduce a remote server or public portal for Zenith
without a demonstrated need and a separate security review.

Terminal projects are not automatically reusable for a new feature. Plan the new
lifecycle explicitly and inspect discovery links before creating another project.

## Required first pilot — propose, do not auto-run

Use a disposable repository/environment and one already-approved supported
provider. Agree exact model and spending limits before execution.

Suggested limits: at most two product/test files, serial execution, one transient
retry, one repair cycle, eight paid executor sessions, ten minutes per session,
sixty minutes total, and a $10 usage ceiling. These are proposed limits, not
Zenith-enforced controls. If cost cannot be bounded adequately, stop before launch.

Establish all of the following:

- Amp discovers the seven orchestrator tools.
- A real worker receives the right task and changes only allowed product files,
  apart from explicitly approved initialization artifacts.
- Zenith records a valid structured handoff.
- A fresh validator rejects a deliberately introduced failure.
- Repair and independent closure succeed.
- Interruption and resumption do not duplicate execution.
- Only then, two disjoint tasks pass a small parallel-work check.

Record exact versions, commands, evidence, costs, failures, and environment.
Repeat the necessary lifecycle checks in an Orb before claiming Orb readiness.
Mocked tests or a successfully launched process are not integration proof.

## Sources

- [Providers](zenith/src/zenith_harness/providers.py)
- [Configuration](zenith/src/zenith_harness/config.py)
- [MCP tools](zenith/src/zenith_harness/server.py)
- [Controller](zenith/src/zenith_harness/controller.py)
- [Scheduling](zenith/src/zenith_harness/coordinator.py)
- [ACP execution](zenith/src/zenith_harness/acp_runner.py)
- [Storage and discovery links](zenith/src/zenith_harness/storage.py)
- [Bundled orchestrator prompt](zenith/src/zenith_harness/bundled/prompts/orchestrator/system_prompt.md)
- [Amp MCP](https://ampcode.com/docs/customize/mcp)
- [Amp skills](https://ampcode.com/docs/customize/skills)
- [Orb setup and additional repositories](https://ampcode.com/docs/orbs/customizing)
