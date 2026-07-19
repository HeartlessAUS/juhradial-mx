# TODO: Configurable Thumb-Wheel Directions

Branch: `feature/thumbwheel-custom-directions`

Status: planning only; implementation requires owner acceptance and should use the configurable-action foundation.

## Goal

Allow each horizontal thumb-wheel direction to invoke any supported action. The requested initial mapping is `Ctrl+Alt+,` in one direction and `Ctrl+Alt+.` in the other, selected and editable through the GUI.

## Current path to inspect

- `daemon/src/config.rs`: `ThumbwheelMode`, `ThumbwheelConfig`, direction resolution, inversion, and repeat count.
- `daemon/src/hidraw.rs`: ThumbWheel notification decoding and action event emission.
- `daemon/src/main.rs`: HID++ diversion application and thumb-wheel event dispatch.
- `daemon/src/hidpp/device.rs` and `daemon/src/hidpp/manager.rs`: feature discovery and reporting controls.
- `daemon/src/actions.rs`: shortcut parsing and injection.
- `overlay/settings_page_scroll.py`: fixed Off/Volume/Horizontal scroll/Zoom selector.
- `daemon/src/profiles.rs`: per-application thumb-wheel mode overrides.

## Analysis tasks

- [ ] Capture representative HID++ deltas from both physical directions.
- [ ] Confirm direction naming so GUI labels match what the user feels.
- [ ] Verify the current thumb-wheel divert behavior over receiver and Bluetooth.
- [ ] Design independent `left_action` and `right_action` configuration using shared actions.
- [ ] Retain a native-scroll option that disables diversion when both directions are native.
- [ ] Define behavior when one direction is native and the other custom; HID++ diversion may require software reinjection for both.
- [ ] Decide whether speed means repeats, accumulated delta threshold, rate, or separate controls.
- [ ] Add rate limiting/coalescing so high-resolution reports do not flood shortcuts.
- [ ] Preserve invert semantics without swapping stored user assignments unexpectedly.
- [ ] Define per-application override migration from the existing mode string.
- [ ] Confirm reload, reconnect, and Easy-Switch reapply diversion correctly.

## Requested default to confirm

- Physical direction A: `Ctrl+Alt+,`
- Physical direction B: `Ctrl+Alt+.`

The final left/right assignment must be confirmed from hardware testing before it becomes a default.

## GUI TODO

- [ ] Replace or extend the fixed mode selector with two direction-action rows.
- [ ] Reuse the shared action picker and shortcut recorder.
- [ ] Retain clear presets for Off, Native horizontal scroll, Volume, and Zoom.
- [ ] Explain when custom mappings require software diversion.
- [ ] Keep inversion and response controls visible only when applicable.
- [ ] Show per-application overrides consistently with the base configuration.
- [ ] Provide restore-default behavior.

## Test TODO

- [ ] Resolve positive, negative, zero, inverted, and large deltas.
- [ ] Test comma and period shortcut injection on KDE Wayland.
- [ ] Test native/native, custom/custom, and mixed native/custom mappings.
- [ ] Verify rate limiting under rapid and high-resolution wheel movement.
- [ ] Verify no duplicate native plus injected horizontal scroll.
- [ ] Verify config migration from off, scroll, volume, and zoom modes.
- [ ] Verify GUI save/reload/restart persistence.
- [ ] Manually test receiver, Bluetooth, reconnect, and Easy-Switch transitions.

## Acceptance questions

- Which physical direction should send comma, and which should send period?
- Should both mappings be populated as the new fork default or applied only to the owner's profile?
- Should speed remain action repeats, or become a wheel-delta threshold/rate control?
- Is mixed native/custom direction behavior required for the first implementation?

## Exit criteria for planning

- Hardware direction and HID++ delta evidence are recorded.
- Diversion and reinjection behavior is agreed for every mapping combination.
- Migration and per-application behavior are specified.
- GUI layout and test matrix are accepted.
