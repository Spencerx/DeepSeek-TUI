# v0.8.67 Constitution Setup Current-Build Evidence

This note records current-build text/render evidence for the v0.8.67
constitution-first setup lane. It complements
`docs/evidence/v0867-constitution-setup-qa-matrix.md`; it is not a release
tag, artifact, or publish record.

Representative guided output examples, including a GLM-5.2-oriented profile,
are recorded in `docs/evidence/v0867-guided-constitution-examples.md`. As of
this snapshot the wizard also offers model-assisted drafting: `A` on the
Constitution step asks the first configured model to draft from the guided
answers, gated through `UserConstitution::from_untrusted_json`
(parse/sanitize/bound) and the same ratification preview + explicit `G` save.

- Date: 2026-07-02T03:54:02Z
- Branch: `claude/v0.8.67-constitution-setup-174rj9`
- Head: `fa7c4b055`
- Workspace version observed in `Cargo.toml`: `0.8.66`

## Covered Surfaces

These checks cover the text-snapshot side of the #3412 release-docs request:

- `/setup` constitution step at blocker terminal sizes `80x24`, `100x30`,
  `120x32`, and `160x40`.
- `/setup` provider/model readiness, runtime posture, constitution choice,
  guided preview/save, update checkpoint, skip/retry, verification report, and
  zh-Hans checkpoint copy.
- `/constitution` manager, preview, edit/repair/bundled/repo/explain/posture
  help paths, including zh-Hans manager/preview copy.
- Prompt injection for the user-global
  `<codewhale_user_constitution>` block and suppression for bundled/deferred or
  expert-override choices.
- `codewhale doctor --json` setup state derivation and persisted-state readback.
- Context report constitution/WHALE.md migration diagnostics without loading
  legacy `WHALE.md` bodies.
- Locale JSON validity for the shipped setup locale files.

## Commands Run

```sh
cargo test -p codewhale-tui --bin codewhale-tui --locked setup_wizard_is_usable_and_opaque_at_blocker_sizes -- --nocapture
```

Result: 1 passed, 0 failed.

```sh
cargo test -p codewhale-tui --bin codewhale-tui --locked constitution -- --nocapture
```

Result: 42 passed, 0 failed.

```sh
cargo test -p codewhale-tui --bin codewhale-tui --locked setup -- --nocapture
```

Result: 127 passed, 0 failed (includes the model-draft request/ingestion,
ratification, discard-on-tune, and authoring-provenance tests).

```sh
cargo test -p codewhale-tui --bin codewhale-tui --locked context_report -- --nocapture
```

Result: 10 passed, 0 failed.

```sh
cargo test -p codewhale-tui --bin codewhale-tui --locked doctor_setup -- --nocapture
```

Result: 5 passed, 0 failed.

```sh
cargo test -p codewhale-config --lib
```

Result: 342 passed, 0 failed (includes the `untrusted_draft_*` ingestion-gate
tests and the `constitution_authoring` round-trip/legacy-load tests).

```sh
cargo test -p codewhale-tui --bin codewhale-tui --locked verification_report -- --nocapture
```

Result: 2 passed, 0 failed.

```sh
jq empty crates/tui/locales/en.json crates/tui/locales/es-419.json crates/tui/locales/ja.json crates/tui/locales/pt-BR.json crates/tui/locales/vi.json crates/tui/locales/zh-Hans.json crates/tui/locales/zh-Hant.json
```

Result: passed.

## Localization Coverage and Fallback (zh-Hant)

The v0.8.67 setup/constitution surfaces are fully localized for `en` and
`zh-Hans` (545+ message keys each). `zh-Hant` ships a partial catalog
(~162 keys) that does not yet include the setup/constitution strings.

Documented fallback behavior for the strings zh-Hant does not carry: the
runtime message loader is initialized with `i18n!("locales", fallback =
["en"])` (`crates/tui/src/main.rs`), so untranslated zh-Hant keys render in
**English**. The `LocaleSpec { fallback: "zh-Hans" }` entry in
`crates/tui/src/localization.rs` is descriptive metadata only and is not
wired into message resolution; a per-locale zh-Hant → zh-Hans chain would be
a behavior change and is deliberately left out of v0.8.67. This satisfies
the #3412/#3794 acceptance alternative of "documented fallback" for
zh-Hant; full zh-Hant coverage (or a zh-Hans fallback chain) remains open
as follow-up localization work.

## Remaining Manual Evidence

Before the release is called ready, keep the final manual pass from the QA
matrix: open a current TUI build and visually confirm the same flow through
`/setup`, `/constitution`, `/setup report`, `doctor --json`, and
`doctor --context-json`. This file records automated current-build coverage,
not a human visual acceptance pass.
