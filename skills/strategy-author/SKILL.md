---
name: strategy-author
description: Use when authoring, forking, or validating a Nexus strategy (preset bundle) — the bundle that declares capability routing and prompt templates for an orchestration loop. Covers bundle layout (preset.yaml / states / inner graphs / roles / templates), the two execution surfaces (integrator/Connect primary vs creator-daemon optional), the signals-not-delivered trap, and offline validation before shipping. Not for runtime operations, platform account work, or compute-module authoring (use compute-module-author).
---

# Strategy Authoring

This skill is the guided on-ramp for writing a Nexus **strategy**: a
plain directory — `preset.yaml` + a `templates/` directory — that
declares *capability routing and prompt templates* for an orchestration
loop: which lane a run enters, which prompts run in which order, and
what the run payload looks like. Nothing in a strategy is compiled into
any binary.

The canonical human reference is `docs/strategy-authoring.md` in the
Nexus repository (nexus42 / `42ch-dev/nexus`). This skill is an
on-ramp, **not a second spec**: every rule below is grounded in that
doc and in the preset manifest types
(`crates/nexus-contracts/src/local/orchestration/preset.rs`). When
the doc and this skill disagree, the doc wins.

## Scope

In scope: bundle layout, `preset.yaml` manifest sections (states,
inner_graphs, roles), prompt templates, run payload contract, the two
execution surfaces, the signals-not-delivered trap, and verifying a
bundle before shipping.

Out of scope: runtime operations, Connect SDK usage, compute-module
authoring (use the `compute-module-author` skill), platform account
work, and daemon administration.

## Load Order

1. Read `docs/strategy-authoring.md` (the human SSOT) before writing
   anything — this skill assumes you have.
2. Read the integrator walkthrough (`strategy-samples/README.md` in the
   repo) to see the end-to-end loop the bundle plugs into.
3. Fork a worked sample: `strategy-samples/game-narrative/` (trigger +
   scheduled lanes) or `strategy-samples/react-trpg-turn/` (turn loop
   with browse guard, settle-first mechanical lane, natural-language
   lane). Forking a real bundle is faster and safer than writing from
   memory.
4. Only then open the sections below that apply to your bundle.

## Workflow

1. **Decide the execution surface** (see Decision Rules — two
   surfaces). The integrator/Connect path is primary; the daemon path
   is optional. This changes where the bundle installs and how it is
   driven — write for the surface you target.
2. **Fork a sample** into a directory you own, or lay out a fresh
   bundle: `preset.yaml` + `templates/` with one template per prompt
   step.
3. **Write `preset.yaml`**: manifest header, `states` (the outer state
   machine), optional `inner_graphs`, optional `roles`, and the run
   payload contract in comments. Keep `preset.id` equal to the bundle
   directory name — the validator enforces it.
4. **Write the templates** in `templates/`, each self-contained (reads
   what it needs from `preset.input`) and referenced by relative path
   from the manifest.
5. **Verify before shipping** (Decision Rules — Validator verify):
   run the offline validator, fix every error, re-run until `valid:
   true` with an empty error list.
6. **Point readers at the human SSOT** — `docs/strategy-authoring.md`
   remains canonical; the skill is the on-ramp.

## Decision Rules

### Bundle layout

```
my-strategy/
├── preset.yaml            # manifest: lanes, states, inner graphs, run payload
└── templates/             # prompt templates, referenced by relative path
    ├── lane-route.md              # lane selector judge (GO/NOGO)
    ├── <lane-step>.md             # one template per prompt step
    └── ...
```

The bundle directory name is part of the contract: the validator
enforces `preset.id` == bundle directory name. There is no separate
`name` field — `id` is the identifier and `description` is the human
label.

### preset.yaml — manifest

Top-level keys (a subset of `PresetManifest`):

| Key | Meaning |
|-----|---------|
| `preset.id` | Identifier; `/^[a-z][a-z0-9._-]*$/` and **must equal the bundle directory name**. |
| `preset.version` | Schema version (`>= 1`; bump on breaking changes). |
| `preset.kind` | `creator` (user-facing) or `system` (internal). External strategies use `creator`. |
| `preset.description` | Human-readable description (<= 240 chars). |
| `preset.requires_capabilities` | Capabilities the strategy needs (`acp.prompt`, `creator.inject_prompt`, `judge.llm`, …). Loader rejects a preset whose capabilities are missing from the registry. |
| `preset.run_intents` | Declared run intents (e.g. `knowledge_ingest`); used for discovery/filtering. |
| `preset.initial` / `preset.terminal` | IDs of the initial and terminal states; both must exist in `states[].id`. |
| `preset.initial_action` | Optional action on run start (e.g. `kind: seed_direct` stores the schedule seed/input verbatim as `core_context` v0). |
| `signals` | Optional declared external-event bindings — schema-valid and loader-parsed, but **NOT delivered** (see the trap below). |
| `states` | Ordered state list — the outer state machine. |
| `inner_graphs` | Optional named node graphs referenced by `enter.kind: inner_graph`. |
| `roles` | Optional role-based agent definitions (see below). |

**States.** Each state has: `id` (unique); optional `description`;
`enter` (optional actions on entry: `kind: inner_graph` runs a named
`inner_graphs` entry, `kind: capability` invokes a capability, e.g.
`creator.inject_prompt` with `prompt_file`); `exit_when` (exit
condition: `kind: llm_judge` with judge prompt in `template_file` and
optional `min_interval` throttle, `kind: graph_complete`, or
`kind: manual` — park, `NextAction::WaitForInput`); `next` (edges to
the next state — linear target, `go`/`nogo` targets valid only on
`llm_judge` states, or `branches` with `when: "<expr>"` + `default`);
and `terminal: true` on the terminal state.

**Inner graphs.** Named node DAGs entered from a state. Nodes are
sequenced with `depends_on`. Node kinds: `acp_prompt` (run a prompt
template). `output_binding` exports one node's output under a key for
the caller. Each node template is self-contained (reads
`preset.input`) per the embedded-preset convention. Real example
(`game-narrative` trigger lane):

```yaml
inner_graphs:
  trigger_graph:
    nodes:
      - id: trigger_parse
        kind: acp_prompt
        template_file: templates/trigger-game-event.md
        tool_policy: auto_grant_read_only
      - id: extract_worldview
        kind: acp_prompt
        depends_on: [trigger_parse]
        template_file: templates/import-worldview.md
        tool_policy: auto_grant_read_only
      - id: extract_characters
        kind: acp_prompt
        depends_on: [extract_worldview]
        template_file: templates/import-character-sheet.md
        tool_policy: auto_grant_read_only
    output_binding: extract_characters.text
```

**Roles (optional).** Multi-agent presets declare `roles` — one
`PresetRoleDefinition` per role: `id` (unique within the preset),
`description`, `system_prompt_file` (relative to the bundle root), and
`recommended_skills` (ordered skill slugs; first = primary). The
loader validates the slug format and requires at least one
recommended skill per role; presets without `roles` run in
single-agent mode. On the daemon path at runtime, the daemon injects
each skill into the role's ACP session using the priority resolution
order (`CLI` > user config > `recommended_skills`); the
integrator/Connect path runs the LLM step on the backend and does not
receive skill injection from the runtime.

```yaml
roles:
  - id: writer
    description: "Primary content writer"
    system_prompt_file: prompts/writer-system.md
    recommended_skills:
      - "novel-writing-assistant"
```

### Two execution surfaces

A strategy does **not** execute on the runtime. It declares routing
and prompts; the executing side is one of two surfaces:

1. **Integrator path (primary).** Your backend runs the LLM step
   using the templates, performs settlement (the `compute` op over
   Connect, for turn strategies), and writes the resulting
   KnowledgeEntry drafts into the World over Connect. The runtime owns
   the World KB, the write/read ops, and host-local compute modules;
   it intentionally ships without the daemon HTTP router and without
   schedule supervision. On this path the run payload (`preset.input`)
   is supplied by your backend; the schedule-input path is the primary
   lane entry (see the run-payload section below).
2. **Creator-daemon path (optional).** Install the bundle under
   `~/.nexus42/presets/<id>/` (3-tier resolution: `~/.nexus42/presets/`
   overrides embedded presets with the same id) and drive it through
   the daemon's schedule API; preset lanes then execute on the
   creator-facing daemon's orchestration engine. Snippets marked
   *daemon path* apply only there.

### Run payload (`preset.input`)

The run payload is the strategy's input contract. Templates render
dotted keys — `{{preset.input.mode}}` works inside a template. The
expression engine that evaluates branch conditions resolves
`_context.<path>` against a **flat** context map, so
`_context.preset.input.mode` can never be read; the only
runtime-populated discriminator expressions can see is
`_context._judge_result`, written by an `llm_judge` exit **before**
branch evaluation. Consequently:

> Lane selection is always a tiny judge (`llm_judge` exit whose prompt
> reads `preset.input.<selector>` and answers GO/NOGO), never a branch
> expression.

On the daemon path the payload is provided when creating the schedule
(`POST /v1/daemon/orchestration/schedules` with
`{ "creator_id", "preset_id", "input": { ... run payload ... } }`).
The CLI's `nexus42 daemon schedule add --preset <id> --creator <id>
--seed <text>` only stores seed text as `core_context` v0 — it does
**not** populate `preset.input.*`.

### The signals-not-delivered trap (hard rule)

A `signals:` section in `preset.yaml` is **schema-valid and parsed by
the loader** — but **no runtime component delivers declared signals
yet**. The daemon's `/schedules/{id}/signal` endpoint only handles
lifecycle signals `start|pause|resume|cancel|advance` and rejects
preset-declared names (an unknown name returns an error; the endpoint
message is "unknown signal '…'; expected
start|pause|resume|cancel|advance").

Consequences:

- Do **not** promise that a declared signal will be delivered.
- Treat signal delivery as a host customization point; the
  schedule-input path is the primary lane entry.
- A `signals:` block is for the schema and the loader — it does not
  make the event arrive.

### Validator verify (before shipping)

Verify the bundle with the real validator core, in-process, no daemon
required:

```bash
bash strategy-samples/validate.sh my-strategy      # from the repo root
# or directly:
nexus42 system preset validate --offline my-strategy
# machine-readable verdict:
nexus42 system preset validate --offline --json my-strategy
# → { "errors": [], "id": "my-strategy", "state_count": 5, "valid": true, "version": 1 }
```

Requires the `nexus42` CLI on `PATH` (build once: `cargo build --bin
nexus42`). Exit status: `0` when clean, non-zero otherwise (`127`
when the CLI is missing). `--offline` mirrors the daemon's
`POST /v1/daemon/presets:validate` composition in-process. On
oversize / parse / depth failures the offline path exits non-zero with
no JSON body (error on stderr) — treat non-zero as "could not
validate".

**Before you ship, an agent MUST:**

1. Run the offline validator (either form above) on the exact bundle
   directory.
2. Confirm `valid: true` and an empty `errors` array (or exit 0 on the
   non-JSON form). Non-zero exit = fix first.
3. Confirm the verdict's `id` equals the bundle directory name, and
   every template the manifest references exists inside the bundle.
4. Re-run after every edit — the validator proves schema, graph shape,
   and referenced assets; it does **not** prove behavior.

**What the validator proves:** manifest compatibility (required
capabilities exist in the registry; `initial` / `terminal` and every
`next` target reference existing states; `go`/`nogo` only on
`llm_judge` states); semantic checks (initial→terminal reachability,
terminal-marker consistency, bundle-id match, inner-graph references,
labeled-edge duplicates, merge-node integrity, capability-arg
compatibility, run intents, CLI args); path safety (every
`template_file` / `prompt_file` / `system_prompt_file` reference is a
safe relative path — no `..`, no absolute paths, no backslashes, no
null bytes / control characters); assets in bundle (`preset.id` equals
the bundle directory name, every referenced template exists, no
symlink escapes); and size limits (YAML capped at 1 MiB and depth 10).

**What it does not prove:** idempotency guarantees (uniqueness of
`turnId` / `operationId`, retry dedup, atomic commit, client unlock)
belong to the client/runtime boundary — your backend generates stable
ids, persists raw input + receipts + final output in its own turn
ledger, and applies the world-aware CAS rules when committing. YAML
alone does not provide a global idempotency ledger.

### Honesty constraints (never promise)

- No agent skill ships in the nexus repo — the skill lives in the
  external `42ch-dev/agent-toolkit` repository, never embedded in
  nexus.
- No `SKILL_MANIFEST.json` / preset `recommended_skills` change is
  made for this skill; a strategy bundle is consumed through the same
  preset bundle format as embedded presets.
- Do not promise that declared signals are delivered (see the trap).
- Do not promise that the daemon path is required — the integrator
  path is primary and needs no daemon HTTP router.
- This skill does not replace the human authoring doc.

## Evidence

- Verdict is `valid: true` with an empty `errors` array from the
  offline validator (JSON form), or exit code 0 from
  `strategy-samples/validate.sh`.
- Templates resolve: validator asset checks pass (every referenced
  file exists in the bundle, no symlink escapes).
- For a shipped turn strategy: the backend ledger shows stable ids,
  deduped retries, atomic commits, and CAS-compliant writes over
  Connect.

## References

- **Human SSOT:** `docs/strategy-authoring.md` in `42ch-dev/nexus`
  — bundle layout, manifest reference, trigger/scheduled lanes,
  validator contract, fork flow.
- **Walkthrough:** `strategy-samples/README.md` in `42ch-dev/nexus` —
  the end-to-end integrator loop (runtime boot, Connect SDK,
  N-C1/N-C2 ops, compute, fork + validate).
- **Worked examples:** `strategy-samples/game-narrative/` (trigger +
  scheduled lanes) and `strategy-samples/react-trpg-turn/` (turn loop;
  its README carries the browse-guard contract).
- **Manifest types (normative):** `crates/nexus-contracts/src/local/
  orchestration/preset.rs` in `42ch-dev/nexus` — `PresetManifest`,
  `PresetRoleDefinition`, `SignalBinding`.
