# TODO: Button Tap, Double-Tap, and Hold Gestures

Branch: `feature/button-tap-hold-gestures`

Status: planning only; implementation requires owner acceptance and should use the configurable-action foundation.

## Goal

Recognize configurable tap, double-tap, and hold interactions for the actions-ring/haptic side button. Double tap should initially invoke a named placeholder for the future focus/unfocus companion application while preserving the radial menu interaction.

## Current path to inspect

- `daemon/src/hidraw.rs`: diverted HID++ press/release state, active action, press time, and radial flow.
- `daemon/src/evdev.rs`: fallback button press/release timing and radial flow.
- `daemon/src/main.rs`: gesture events converted to menu signals or actions.
- `overlay/juhradial-overlay.py`: tap-to-stay-open and hold-drag behavior currently interpreted partly by the overlay.
- `daemon/src/config.rs`: single action per physical button.
- `overlay/settings_dialog_button.py`: assignment UI.

## Analysis tasks

- [ ] Trace complete HID++ and evdev event sequences for the physical actions-ring button.
- [ ] Confirm whether receiver, Bluetooth, reconnect fallback, and generic mode produce equivalent sequences.
- [ ] Design one explicit state machine for idle, pressed, awaiting-second-tap, second-pressed, held, chorded, and cancelled states.
- [ ] Decide whether timing recognition belongs entirely in the daemon and remove duplicated interpretation where necessary.
- [ ] Define configurable double-tap interval, hold threshold, and movement tolerance with safe ranges.
- [ ] Decide when the radial overlay appears during a possible double tap.
- [ ] Define cancellation on cursor movement, another button, wheel input, timeout, disconnect, config reload, and daemon shutdown.
- [ ] Prevent HID++ and evdev fallback paths from recognizing the same physical gesture twice.
- [ ] Define named integration-hook configuration and initial placeholder behavior.
- [ ] Reserve a stable future path for the focus/unfocus application without coupling gesture code to that application.
- [ ] Determine whether gesture mappings are global, per button, and/or per application.

## Placeholder requirements

- Suggested identifier: `focus_toggle`.
- It must be an ordinary configurable action, not a special case embedded in the button handler.
- Before the companion app exists, execution should be observable and harmless.
- Candidate initial behavior: structured log plus a D-Bus signal carrying the hook name.
- The final transport requires owner acceptance before implementation.

## GUI TODO

- [ ] Add interaction rows for tap, double tap, and hold.
- [ ] Reuse the shared action picker for each interaction.
- [ ] Add timing controls with defaults, valid ranges, explanations, and restore buttons.
- [ ] Clearly show conflicts between immediate single-tap behavior and double-tap recognition.
- [ ] Identify actions that are unavailable in generic mode or on unsupported hardware.
- [ ] Ensure the radial assignment remains understandable when attached to tap or hold.

## Test TODO

- [ ] Test events just below, at, and above every timing boundary.
- [ ] Test slow tap, rapid double tap, triple tap, bounce, long hold, and interrupted hold.
- [ ] Test cursor movement and wheel input during each state.
- [ ] Test config reload and device disconnect in every waiting/pressed state.
- [ ] Test exactly-once placeholder dispatch.
- [ ] Test radial show/hide/toggle behavior without flicker or stranded overlay state.
- [ ] Compare HID++, evdev fallback, and generic-mode behavior.
- [ ] Verify GUI persistence and invalid threshold handling.

## Acceptance questions

- Should the first tap open the radial immediately, or wait until the double-tap window expires?
- Should holding the button open the radial at the hold threshold, or continue opening on press?
- Should the placeholder emit a D-Bus signal, execute a configurable command, or only log until the companion app contract exists?
- Should double tap be available on every remappable button or only the actions-ring button initially?

## Exit criteria for planning

- State machine and cancellation table are accepted.
- Radial menu timing UX is accepted.
- Placeholder contract and safe initial behavior are accepted.
- Hardware/manual and automated test matrices are accepted.
