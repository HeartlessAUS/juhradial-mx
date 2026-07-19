# TODO: Hold Button and Scroll Virtual Desktops

Branch: `feature/button-wheel-desktop-chord`

Status: research planning only; implementation requires owner acceptance plus the configurable-action and gesture-state foundations.

## Goal

While the large actions-ring/haptic side button is held, use the main vertical wheel to invoke configurable previous/next virtual-desktop actions. Normal scrolling must not leak into the focused application while the chord owns the wheel.

## Current path to inspect

- `daemon/src/hidraw.rs`: actions-ring HID++ press/release and independent thumb-wheel notifications.
- `daemon/src/evdev.rs`: raw key handling, optional exclusive grab, forwarding virtual device, and suppression set.
- `daemon/src/main.rs`: creation of HID++ and evdev handlers plus central gesture dispatch.
- `daemon/src/actions.rs` and `daemon/src/presets.rs`: desktop detection and previous/next desktop actions.
- `daemon/src/compositor.rs`: compositor-specific facilities.
- `daemon/src/window_tracker.rs`: KDE KWin integration patterns.
- `packaging/udev/` and installation docs: evdev/uinput permissions.

## Analysis tasks

- [ ] Identify the exact evdev device and event codes used by the main wheel on receiver and Bluetooth connections.
- [ ] Record interleaved actions-ring press, wheel, and release event traces.
- [ ] Determine whether the HID++ button and evdev wheel arrive from independently ordered streams and define synchronization rules.
- [ ] Evaluate whether the device must remain exclusively grabbed to suppress wheel events conditionally.
- [ ] If grabbed, specify lossless forwarding for every unowned event, including high-resolution wheel data.
- [ ] Define chord activation, ownership, first-event behavior, repeat rate, inversion, cancellation, and release behavior.
- [ ] Ensure a chorded scroll suppresses radial selection/opening unless explicitly configured otherwise.
- [ ] Define what a button release with no wheel movement does.
- [ ] Verify previous/next desktop actions on Plasma 6.7 rather than assuming `Meta+Alt+wheel` is a standard binding.
- [ ] Compare keyboard shortcuts, KGlobalAccel/D-Bus invocation, and KWin scripting for reliable Wayland execution.
- [ ] Define fallbacks for GNOME, Hyprland, Sway, COSMIC, niri, and X11.
- [ ] Clarify “current monitor”: Plasma virtual desktops normally span the output arrangement, so independent per-monitor switching may be a different feature.
- [ ] Define behavior at desktop boundaries and with Plasma navigation wrapping enabled/disabled.
- [ ] Test reconnect, Easy-Switch, suspend/resume, and daemon restart while an input device is grabbed.

## Candidate action model

- Modifier input: actions-ring/haptic button by default.
- Secondary input: main vertical wheel by default.
- Direction A action: previous virtual desktop.
- Direction B action: next virtual desktop.
- Invert: configurable.
- Release without chord: radial-menu behavior, configurable.

This is a proposed preset built from configurable inputs/actions, not a hardcoded special mode.

## GUI TODO

- [ ] Add a chord editor that selects modifier input, secondary input, and both direction actions.
- [ ] Reuse the shared action picker.
- [ ] Add inversion, rate/debounce, and release-without-chord behavior.
- [ ] Explain global virtual desktops versus monitor-specific window behavior.
- [ ] Warn about conflicts with existing button and wheel assignments.
- [ ] Hide or adapt unavailable hardware inputs in generic mode.
- [ ] Provide restore-default and disable-chord controls.

## Test TODO

- [ ] Unit-test chord state transitions and stream ordering permutations.
- [ ] Verify owned wheel suppression and byte-for-byte passthrough of unowned events.
- [ ] Test legacy and high-resolution `REL_WHEEL`/`REL_WHEEL_HI_RES` input.
- [ ] Test rapid direction reversals, long holds, and repeated reports.
- [ ] Test button release before/after wheel frames and disconnect mid-chord.
- [ ] Verify exactly one desktop transition per configured threshold.
- [ ] Verify active application receives no chord-owned scroll.
- [ ] Verify ordinary vertical scrolling remains unchanged when the button is not held.
- [ ] Manually test multi-monitor Plasma behavior and desktop boundary/wrap settings.

## Acceptance questions

- Does “current monitor” mean switch the global Plasma desktop based on the monitor under the pointer, or independently change windows/workspaces on only that monitor?
- Should release without any wheel movement open/toggle the radial menu?
- Should desktop switching invoke configurable shortcuts or a direct KDE action by default?
- Is support for other compositors required in the first implementation or may it begin as a KDE preset over a portable chord engine?

## Exit criteria for planning

- Hardware event traces and suppression feasibility are recorded.
- The intended monitor/workspace behavior is unambiguous.
- Plasma invocation mechanism is verified on the target system.
- Chord ownership/cancellation and failure recovery rules are accepted.
- GUI and automated/manual test matrices are accepted.
