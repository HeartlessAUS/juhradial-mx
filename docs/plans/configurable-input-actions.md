# TODO: Configurable Input Actions

Branch: `feature/configurable-input-actions`

Status: planning only; implementation requires owner acceptance.

## Goal

Create a reusable, validated action model that the GUI can assign to button presses, wheel directions, taps, holds, and chords without adding a hardcoded daemon enum variant for every user shortcut.

## Current path to inspect

- `daemon/src/config.rs`: fixed `ButtonAction` enum and button configuration.
- `daemon/src/actions.rs`: general radial `ActionType`, shortcut parsing/injection, built-in button dispatcher.
- `daemon/src/presets.rs`: desktop-portable action resolution.
- `overlay/settings_constants.py`: duplicated action identifiers and display labels.
- `overlay/settings_dialog_button.py`: fixed grouped action list.
- `overlay/settings_dialog_radial.py` and `overlay/overlay_actions.py`: existing configurable radial action editing/execution.
- `overlay/settings_macro_actions.py`: reusable action-editing UI concepts.

## Analysis tasks

- [ ] Inventory every current action representation and execution path.
- [ ] Decide whether to extend radial `ActionType` or introduce a shared versioned `InputAction` type.
- [ ] Define a backward-compatible migration from string button actions.
- [ ] Define validation errors that can be returned to and displayed by the settings GUI.
- [ ] Specify shortcut token normalization, including comma, period, and layout-sensitive keys.
- [ ] Determine whether shortcut capture stores physical evdev codes, symbolic keys, or both.
- [ ] Define named integration hooks independently of their future transport.
- [ ] Separate action definition, validation, desktop-specific resolution, and execution into focused modules.
- [ ] Determine how per-application button overrides reference the shared action model.
- [ ] Document command-action trust boundaries and keep command entry explicit in the GUI.

## Proposed configuration shape to evaluate

```json
{
  "type": "shortcut",
  "keys": ["ctrl", "alt", "comma"]
}
```

Other candidate types: `builtin`, `mouse`, `command`, `dbus`, `integration`, and `none`. This is illustrative, not an accepted schema.

## GUI TODO

- [ ] Build a reusable action picker used by buttons and later gesture/wheel dialogs.
- [ ] Add a shortcut recorder with a readable normalized preview.
- [ ] Allow manual correction for shortcuts that cannot be captured reliably.
- [ ] Expose built-ins before advanced command/D-Bus actions.
- [ ] Show validation failures inline and prevent invalid saves.
- [ ] Add a safe “Test action” control only if action cancellation and side effects are clear.
- [ ] Preserve restore-default behavior.

## Test TODO

- [ ] Deserialize legacy string actions and serialize without losing intent.
- [ ] Round-trip every new action kind.
- [ ] Validate unknown types, missing fields, invalid keys, and empty actions.
- [ ] Test punctuation and modifier order/case normalization.
- [ ] Test Wayland evdev-code generation and X11 fallback behavior.
- [ ] Test settings loading, editing, saving, daemon reload, and restart persistence.
- [ ] Confirm existing button and radial actions behave unchanged.

## Acceptance questions

- Should shortcut assignments follow the active keyboard layout or physical key positions?
- Should arbitrary shell commands be available everywhere or remain an advanced action?
- Which stable interface should a named integration hook expose initially: logged no-op, D-Bus signal, or command placeholder?

## Exit criteria for planning

- Owner accepts the shared schema and migration strategy.
- GUI action-editing flow is agreed.
- Action execution ownership between daemon and overlay is explicit.
- Automated and manual test matrix is accepted.
