# HeartlessAUS Feature Pinboard

This pinboard is the shared planning surface for the HeartlessAUS fork. It records current capabilities, requested features, dependencies, open decisions, and validation requirements without treating an idea as approved implementation work.

Status vocabulary:

- **Current**: present in the inspected codebase.
- **Proposed**: requested, but not yet accepted for implementation.
- **Researching**: architecture or platform behavior still needs confirmation.
- **Ready**: plan and acceptance criteria are accepted; implementation may begin when requested.
- **Implementing**: code work is active on its feature branch.
- **Testing**: implementation exists and is undergoing automated/manual validation.
- **Blocked**: an owner decision or external dependency prevents useful progress.
- **Complete**: merged into the fork's main branch and documented.

## Current product capabilities

| Area | Status | Current behavior and notes |
| --- | --- | --- |
| Radial menu | Current | Eight configurable slices, themes, animations, tap-to-stay-open, and hold-drag selection. |
| Logitech HID++ | Current | Device discovery, diverted controls, thumb-wheel reporting, DPI, scrolling modes, haptics, battery, and Easy-Switch support. |
| Generic mouse fallback | Current | evdev-based radial trigger and button/macro input without Logitech-only hardware controls. |
| Button assignment | Current | A fixed enum of built-in actions is exposed in the GTK settings application. `Custom` is visible but its daemon dispatcher is a placeholder. |
| Thumb wheel | Current | Off/native horizontal scroll, volume, and zoom modes with inversion and repeat-speed controls. Only volume and zoom are diverted. |
| Shortcut injection | Current | xdotool on X11 and ydotool/uinput for mapped shortcuts on Wayland. The internal Wayland map does not yet cover punctuation such as comma and period. |
| Per-application profiles | Current | Hardware profiles are supported on KDE, Hyprland, and X11; GNOME Wayland lacks active-window identity support. |
| Macros | Current | Recording, editing, repeat modes, and button triggers. Keyboard-named macro triggers are parsed but not implemented by the evdev layer. |
| Gaming mode | Current | Gaming profiles, DPI cycling, macro integration, and radial-overlay suppression. |
| JuhFlow | Current | Encrypted Linux/macOS cursor and clipboard handoff. Windows remains planned. |
| Settings GUI | Current | GTK4/libadwaita pages edit config and use D-Bus for live device operations and reloads. |
| Compositor support | Current | KDE Plasma 6, GNOME, Hyprland, COSMIC, Sway/wlroots, niri, and X11 placement paths; Wayland overlay placement still depends on XWayland. |

## Requested feature branches

### Configurable input actions

- **Branch:** `feature/configurable-input-actions`
- **Status:** Ready; plan accepted for implementation on 2026-07-19
- **Purpose:** Provide one reusable action representation for buttons, wheel directions, taps, holds, and chords.
- **Required action kinds:** keyboard shortcut, mouse action, built-in action, command, D-Bus call, named integration hook, and disabled/no-op.
- **GUI requirement:** action picker plus shortcut capture/editor; raw configuration editing must not be required.
- **Compatibility:** existing snake-case button action values must continue loading.
- **Dependency role:** foundation for all three input features below.

### Configurable horizontal-wheel directions

- **Branch:** `feature/thumbwheel-custom-directions`
- **Status:** Proposed; depends on configurable input actions
- **Requested default:** map the two directions to `Ctrl+Alt+,` and `Ctrl+Alt+.` respectively.
- **GUI requirement:** independent left/right selectors with shortcut capture, inversion, and rate/sensitivity controls.
- **Important constraint:** custom direction actions require diverted thumb-wheel notifications; the current native-scroll path cannot transform each direction into arbitrary actions.
- **Open decision:** confirm which physical direction receives comma and which receives period before implementation acceptance.

### Actions-ring button double tap

- **Branch:** `feature/button-tap-hold-gestures`
- **Status:** Proposed; depends on configurable input actions
- **Requested behavior:** double-tapping the large actions-ring/haptic side button invokes a placeholder for the future focus/unfocus companion application.
- **GUI requirement:** configurable double-tap action and timing threshold. The placeholder must remain replaceable by another action.
- **Integration shape:** prefer a named hook that can later resolve to D-Bus or another stable IPC contract without rewriting gesture recognition.
- **Open decision:** choose whether single-tap radial opening waits for the double-tap timeout or opens immediately and is cancelled/replaced when the second tap arrives.

### Hold actions-ring button and scroll desktops

- **Branch:** `feature/button-wheel-desktop-chord`
- **Status:** Researching; depends on configurable input actions and tap/hold gesture state
- **Requested behavior:** while the large actions-ring/haptic side button is held, vertical-wheel movement selects the previous or next virtual desktop.
- **GUI requirement:** configurable modifier button, input wheel, direction actions, inversion, activation threshold, and whether release opens the radial menu when no chord occurred.
- **Input requirement:** observe and consume vertical-wheel events only while the chord owns them, preventing accidental scrolling in the focused application.
- **KDE note:** Plasma documents desktop/pager wheel switching and configurable previous/next desktop shortcuts. `Meta+Alt+wheel` is not yet confirmed as a standard virtual-desktop binding and must be tested on the target Plasma 6.7 system.
- **Monitor note:** Plasma virtual desktops normally represent the workspace across outputs. “Current monitor only” needs a precise behavior definition and may require a KWin-specific window workflow rather than ordinary virtual-desktop switching.

## Dependency order

1. Accept and implement `feature/configurable-input-actions`.
2. Implement `feature/thumbwheel-custom-directions` on the shared action model.
3. Implement `feature/button-tap-hold-gestures` and its configurable timing state machine.
4. Implement `feature/button-wheel-desktop-chord` using the action and gesture layers.

The branches are separate review units, but later branches may be rebased onto accepted prerequisite work before implementation.

## Shared design requirements

- Preserve existing user configuration and native defaults.
- Keep shortcut choices, direction mappings, timing thresholds, and integration hooks configurable.
- Separate physical input recognition from action execution.
- Define press, release, repeat, cancellation, double-tap timeout, and device-disconnect behavior explicitly.
- Avoid invoking actions twice when HID++ and evdev observe the same physical control.
- Reapply volatile HID++ diversion after reload, reconnect, and Easy-Switch transitions.
- Keep generic-mouse behavior functional, but hide or explain controls that require Logitech HID++ hardware.
- Provide safe validation and clear GUI errors for invalid shortcut or command definitions.
- Maintain KDE Plasma 6.7 Wayland as the primary target while preserving other supported desktops where practical.

## Shared validation checklist

Automated coverage should include:

- backward-compatible configuration deserialization and round trips;
- action validation and dispatch for every supported action kind;
- punctuation and modifier shortcut parsing;
- direction inversion and repeat/rate limiting;
- tap, double-tap, hold, chord, cancellation, and boundary timing cases;
- suppression of owned wheel events and passthrough of unowned events;
- reconnect/reload behavior and duplicate-event prevention;
- settings load, edit, save, reload, and restart persistence.

Manual validation on Arch Linux, KDE Plasma 6.7, Wayland, and the target mouse should include:

- every changed button and wheel direction;
- fast and slow double taps around the configured threshold;
- hold-and-scroll without scroll leakage into the active application;
- radial menu behavior after a tap, hold, cancelled gesture, and chord;
- desktop switching at the first/last desktop, including wrap settings;
- multiple monitors and mixed/fractional scaling;
- mouse reconnect and Easy-Switch away/back;
- action changes made entirely through the GUI.

## Additional customization candidates

These are ideas only; they do not have branches or implementation approval.

- Tap, double-tap, hold, and chord assignments for every remappable button.
- Alternate action layers while Ctrl, Alt, Shift, Meta, or another mouse button is held.
- Wheel acceleration curves, repeat throttling, and per-action debounce controls.
- Per-application gesture layers in addition to existing hardware profiles.
- A GUI “Test action” control that previews an assignment before saving.
- Importable and exportable control profiles.
- Optional on-screen feedback for desktop, volume, DPI, and profile changes.
- User-defined named D-Bus hooks for companion applications.
- Conflict warnings when multiple gestures compete for the same physical input.

## Decision log

| Date | Decision |
| --- | --- |
| 2026-07-19 | Planning is split into four branches with a shared configurable-action foundation. No feature implementation is authorized by the initial TODO commits. |
| 2026-07-19 | No remote pushes are permitted until the repository owner reviews and accepts the local branch/commit set. |
| 2026-07-19 | The repository owner accepted the branch plans and authorized implementation to begin on `feature/configurable-input-actions`. The dependent feature branches remain planning-only until their prerequisite is available and their implementation is requested. |
