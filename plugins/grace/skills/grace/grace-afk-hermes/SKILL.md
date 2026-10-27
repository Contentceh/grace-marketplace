---
name: grace-afk-hermes
description: Run an approved GRACE plan in a bounded Hermes turn.
version: 0.1.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [GRACE, AFK, Hermes, planning]
    related_skills: []
---

# GRACE AFK Hermes Skill

Run an already-approved GRACE plan for a bounded portion of the current Hermes turn. GRACE CLI manages session state and journaling; Hermes remains the executor. This is not background automation and does not survive a restart or turn end.

## When to Use

- The user explicitly authorizes continuing an existing GRACE plan in the current task.
- The user invokes `grace-afk-hermes` or `/grace-afk-hermes`.
- Don't use to initialize or migrate GRACE, invent/approve a plan, enable AFK on a project, or claim unattended operation.

## Prerequisites

- Use the project root and existing approved `.grace/` plan selected by the user/task. If missing or ambiguous, stop and clarify; never run init/migrate.
- Use `grace --help` and `grace afk --help` to confirm the available commands and flags, then read the project's applicable `AGENTS.md`/approved project rules and the approved plan's execution/review roles. Do not assume a separate `grace-cli` skill is installed. The normal `grace` command is supported; the dedicated fork CLI at `bun "$HOME/.local/share/grace-hermes/node_modules/@osovv/grace-cli/src/grace.ts"` remains an optional explicit alternative when available, not an additional installation prerequisite.
- At every CLI call set `GRACE_AFK_CONFIG` to the absolute path of this skill's `assets/no-telegram.json`, resolved beneath the active `$HERMES_HOME` (or the active profile home supplied by Hermes). Do not hardcode another profile or rely on project/global config lookup. The file contains `{}` intentionally: it prevents a project or global Telegram config from starting GRACE polling.
- The host's native `clarify` interaction is the only human-question route. Use its current-turn/session context; do not pass model-supplied chat IDs, impersonate an origin, or treat arbitrary chat text as a response.

## How to Run

Use Hermes `terminal` for each CLI call and explicitly override `GRACE_AFK_CONFIG` with this skill's profile-owned `assets/no-telegram.json`, resolved beneath the active `$HERMES_HOME` (or Hermes-provided active profile home). Invoke `grace afk <verb> --path <absolute-project-root>`; if the dedicated fork CLI is intentionally used instead, prefix the same environment override and pass arguments to its explicit `bun .../grace.ts` entrypoint. Keep the override on every call. Do not invoke `grace afk ask` or `check`: they can poll Telegram. Do not use a second bot or polling process.

## Procedure

1. **Confirm authorization and plan.** Preserve the approved plan, task scope, and designated writer/reviewer roles. Do not modify approved XML, broaden write scope, or repeat mode/plan choices already explicitly authorized. Choose any required execution mode once under the current authorization; ask only if it is genuinely missing or consequentially ambiguous.
2. **Start explicitly.** Run `grace afk start <hours> [budgetPercent] [--checkpoint <minutes>] --path <root>` using only user-authorized bounds. The CLI records lifecycle state; `budgetPercent` is telemetry, not an enforced Hermes token/spend cap. Do not invent a default duration or claim the CLI interrupts running tools.
3. **Loop within this turn.** Before each logical task, run `grace afk tick --path <root>`. Read the existing approved plan and select the first pending task whose dependencies are complete. Run its designated writer, then the designated Sol reviewer. Routine, in-scope progression to the next accepted task is authorized; don't ask the user after every routine step.
4. **Handle decisions narrowly.** Act only within approved scope and plan. For a meaningful unresolved tradeoff or action outside authorization, run `grace afk defer` and stop at the decision boundary; ask the user with native `clarify` only when their answer is needed to proceed. Present specific options and a recommendation. Only accept the native clarify result bound to this turn as a response. Do not infer approval from unrelated messages or silence. Any explicit approval applies only to the stated decision and does not authorize extra scope.
5. **Verify before advancing.** Require reviewer acceptance and applicable task checks before marking/recording acceptance or selecting the next task. Use `grace afk journal` and `grace afk increment` according to the CLI contract. If checks fail, reviewer rejects, scope is unclear, a command errors, tick indicates expired/stopped/no session, or the current-turn limit is reached, stop safely: record a checkpoint/defer where possible and do not continue. Never auto-commit, tag, reset, push, deploy, or rewrite the approved XML.
6. **Finish separately.** When the authorized work ends, run `grace afk report` and provide accepted work, verification evidence, deferred decisions, and remaining tasks. Final apply/integration remains a separate action unless separately authorized; do not interpret report completion as apply approval.

## Pitfalls

- Expiry is cooperative and checked at tick boundaries; it cannot interrupt a running command, model, or worker. On timeout, stop before starting another action and checkpoint/defer safely.
- `budgetPercent` is not enforced. Bound this work to the current turn; no background scheduler, durable Hermes resume, or restart recovery is promised.
- Native clarify is synchronous and turn/session-bound, not a durable decision queue. If the answer is unavailable or ambiguous, defer and stop rather than polling.
- Discord is the intended primary channel when this skill is invoked there. Telegram is optional only by invoking this same skill in the current Telegram chat; it is not automatically mirrored or polled.
- Do not assume `.grace-afk.json`, `~/.grace/afk.json`, or other Telegram settings are harmless. The explicit safe config override is mandatory for every CLI call.

## Verification

- Confirm the selected existing plan and task scope without changing approved XML.
- Confirm the explicit fork CLI and profile-owned `{}` config override were used; don't run any command that polls or contacts Telegram.
- Confirm every advanced task has writer acceptance, independent reviewer acceptance, and recorded verification; ensure deferred decisions remain unresolved.
- Confirm the report reflects the session and that no final apply or Git side effect occurred unless separately authorized.
