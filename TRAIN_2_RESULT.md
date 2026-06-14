# Train 2 Result — Permissions + Shell Nonblocking

Branch: `codex/v0.8.61-train-2`  
Final status: clean worktree; no push, PR, tag, release, or publish.

## #3211 + #1186 — Typed Permission/Shell Profile

Status: implemented and committed (`aa2950eb3`).

Files:
- `crates/tui/src/worker_profile.rs`
- `crates/tui/src/tools/spec.rs`
- `crates/tui/src/tools/registry.rs`
- `crates/tui/src/core/engine/tool_setup.rs`
- `crates/tui/src/core/engine.rs`
- `crates/tui/src/prompts.rs`
- `crates/tui/src/tools/shell.rs`
- `crates/tui/src/tools/shell/tests.rs`
- `crates/tui/src/core/engine/tests.rs`

Tests:
- `cargo test -p codewhale-tui --bin codewhale-tui build_tool_context_uses_typed_shell_policy_per_mode` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui read_only_shell_policy` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui agent_tools_with_shell_policy_readonly_includes_shell_tools` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui runtime_policy_reference_is_included_in_full_prompt` — passed.

Risks:
- `ShellPolicy::ReadOnly` relies on the existing read-only shell classifier; it is a typed policy gate, not an OS sandbox.
- Legacy `allow_shell` still feeds policy derivation for compatibility.

## #3212 — Background Shell + Verifier Defaults

Status: implemented and committed (`15aa2ca80`).

Files:
- `crates/tui/src/core/engine/turn_loop.rs`
- `crates/tui/src/tools/shell.rs`
- `crates/tui/src/tools/shell/tests.rs`
- `crates/tui/src/tools/tasks.rs`
- `crates/tui/src/tools/verifier.rs`

Tests:
- `cargo test -p codewhale-tui --bin codewhale-tui exec_shell_wait_schema_defaults_to_nonblocking_snapshot` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui exec_shell_wait_without_wait_arg_returns_snapshot` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui background_start_advertises_auto_notify` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui drain_finished_jobs_reports_once` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui shell_completion_handoff_is_internal_user_message` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui run_verifiers_background_starts_shell_jobs_and_returns_task_ids` — passed.
- `cargo test -p codewhale-tui --bin codewhale-tui run_verifiers_background_advertises_detached_start` — passed.

Risks:
- Background completion notices are injected by turn-loop drain points; an idle runtime still depends on those loop passes to surface completions.

## #1791 — Cancellable Synchronous Tools

Status: already implemented in the branch foundations; verified, no new commit.

Files inspected:
- `crates/tui/src/tools/file_search.rs`
- `crates/tui/src/tools/search.rs`
- `crates/tui/src/tools/file.rs`

Tests:
- `cargo test -p codewhale-tui --bin codewhale-tui respects_cancel_token` — passed, 3 tests.

Risks:
- The blocking work still runs in `spawn_blocking`; cancellation is cooperative through checked cancel tokens inside the search/list loops.

## #1737 + #1786 — Failed Shell Stuck State + PID/Queue Hangs

Status: implemented and committed (`b38f0ae7d`).

Files:
- `crates/tui/src/core/engine/tool_execution.rs`
- `crates/tui/src/core/engine/turn_loop.rs`
- `crates/tui/src/core/events.rs`
- `crates/tui/src/tools/shell.rs`
- `crates/tui/src/tools/shell/tests.rs`
- `crates/tui/src/tui/app.rs`
- `crates/tui/src/tui/shell_job_routing.rs`
- `crates/tui/src/tui/sidebar.rs`
- `crates/tui/src/tui/subagent_routing.rs`
- `crates/tui/src/tui/ui.rs`
- `crates/tui/src/tui/ui/tests.rs`

Tests:
- `cargo test -p codewhale-tui --bin codewhale-tui tool_heartbeat` — passed, 3 tests.
- `cargo test -p codewhale-tui --bin codewhale-tui turn_liveness` — passed, 8 tests.
- `cargo test -p codewhale-tui --bin codewhale-tui running_job_snapshot` — passed, 2 tests.
- `cargo test -p codewhale-tui --bin codewhale-tui stale_background_job` — passed, 1 test.
- `cargo test -p codewhale-tui --bin codewhale-tui repeated_shell_waits` — passed, 2 tests.
- `cargo test -p codewhale-tui --bin codewhale-tui list_shows_controls_and_stale_state` — passed, 1 test.

Risks:
- UI recovery after the 15m tool-hang watchdog is non-destructive; an underlying OS child may still be running and must be canceled separately.
- Live shell jobs are marked stale after 60s without observed output; long quiet commands will show the warning even if they are still healthy.

## #2475 — YOLO/MCP Prompt Interruption

Status: implemented and committed (`7f988806e`).

Files:
- `crates/tui/src/mcp.rs`

Tests:
- `cargo test -p codewhale-tui --bin codewhale-tui mcp_response` — passed, 1 test.
- `cargo test -p codewhale-tui --bin codewhale-tui call_method_invalid_json_includes_server_output_preview` — passed, 1 test.
- `cargo test -p codewhale-tui --bin codewhale-tui call_method_times_out_while_waiting_for_response` — passed, 1 test.

Risks:
- MCP receive timeout honors the effective configured read timeout; a zero-second read timeout fails immediately.
- This turns prompt-blocked stdio into a clear MCP error/disconnect; it does not auto-answer third-party proxy prompts.

## Test Hardening — Env-Mutating Config Tests

Status: implemented and committed (`5d58c090a`).

Files:
- `crates/tui/src/config.rs`

Tests:
- `cargo test -p codewhale-tui --bin codewhale-tui save_api_key_for -- --test-threads=8` — passed, 2 tests.
- `cargo test -p codewhale-tui --bin codewhale-tui route -- --test-threads=8` — passed, 100 tests.

Risks:
- This hardens the named save-key/env path and the route-filtered parallel run; broader env-mutating tests outside that filter may still need similar cleanup if exposed by a larger parallel suite.

## Final Verification

- `cargo fmt --all` — passed before each commit series.
- `git status --short --branch` — clean on `codex/v0.8.61-train-2`.
