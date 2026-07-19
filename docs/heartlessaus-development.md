# HeartlessAUS Fork: Repository and Development Guide

This document records what is currently in the fork and how work should be performed. It supplements the original project documentation; it does not replace it.

## Purpose

JuhRadial MX is a Linux control suite for Logitech MX Master mice, with a generic-mouse fallback. Its goal is to provide Logitech-style advanced mouse features on Linux, especially Wayland desktops: a radial gesture menu, button and wheel remapping, hardware configuration, macros, per-application profiles, and peer-to-peer cursor/clipboard sharing.

The running application has three main parts:

1. A Rust/Tokio daemon handles HID++, hidraw and evdev input, uinput action injection, configuration, profiles, macros, and the D-Bus API.
2. A PyQt6 overlay draws and positions the radial menu.
3. A Python GTK4/libadwaita settings application edits the configuration and asks the daemon to reload it.

The default user configuration is stored at `~/.config/juhradial/config.json`. See [Architecture](architecture.md) and [Configuration](configuration.md) for the detailed data flow and schema.

## What is currently included

- Radial menu with eight configurable slices, themes, animations, tap and hold-drag interaction.
- Logitech MX Master HID++ support for device discovery, haptics, DPI, SmartShift/HiRes scrolling, battery state, diverted buttons, thumb wheel, and Easy-Switch.
- Generic evdev mouse mode for radial-menu triggering and button/macro input without Logitech-only hardware controls.
- Button actions, desktop-portable presets, thumb-wheel volume/zoom/horizontal scrolling, and uinput-based action injection.
- Per-application hardware profiles on KDE, Hyprland, and X11.
- Gaming profiles and a macro editor/engine with recording and repeat modes.
- JuhFlow peer discovery, encrypted Linux/macOS cursor handoff, and clipboard sharing, plus a bundled macOS companion application.
- KDE Plasma 6, GNOME, Hyprland, COSMIC, Sway/wlroots, niri, and X11 positioning paths. The Wayland overlay currently uses XWayland.
- Installation and packaging material for Arch-family distributions, Fedora, Debian/Ubuntu, openSUSE, NixOS, Flatpak, and RPM, plus systemd and udev integration.
- Rust unit tests and benchmarks, Python placement/startup/Flow tests, launcher tests, distro-container smoke tests, and GitHub security/documentation workflows.
- User and contributor documentation under `docs/`, built with MkDocs.

## Known boundaries and incomplete areas

This is a snapshot of the repository as inspected on 2026-07-19, not a promised roadmap.

- JuhFlow supports Linux and macOS; Windows support is documented as planned, not implemented.
- Per-application profiles do not work on GNOME Wayland because an active-window identity source is not implemented there.
- niri positioning is interim and requires `xwayland-satellite`; a native layer-shell overlay is only planned.
- Wayland overlay placement depends on XWayland on every supported compositor.
- The `Smartshift` and `Custom` variants in the daemon's direct button-action dispatcher currently log that they are not implemented. Other SmartShift configuration paths exist, so this limitation is specifically about those button-action variants.
- Keyboard-named macro triggers (`key:<name>`) are parsed but are not supported by the evdev trigger layer.
- The Arch `PKGBUILD` appears behind the current application: it declares version `0.2.5` while the daemon and main documentation declare `0.4.0`, references an old udev-rule filename, and omits some currently documented Python UI dependencies. Treat it as needing verification before packaging.
- GitHub CI builds the daemon and invokes Rust tests, but the Rust test step currently has `continue-on-error: true`; a test failure therefore does not fail that job.
- Several features require real mouse hardware, raw-input permissions, a user D-Bus session, a compositor, or two paired computers. The automated suite cannot fully validate those paths.

When proposed work touches one of these boundaries, confirm the intended behavior and scope with the repository owner before designing the change.

## Fork safety and branch workflow

All GitHub operations must stay under `HeartlessAUS/`. Never push to the original project. Before network operations, verify the remotes:

```bash
git remote -v
```

Every feature, fix, or documentation idea starts on its own branch:

```bash
git switch master
git pull --ff-only origin master
git switch -c feature/short-descriptive-name
```

Use `fix/…` or `docs/…` when those better describe the work. Push only after confirming that `origin` is `HeartlessAUS/juhradial-mx`:

```bash
git push -u origin feature/short-descriptive-name
```

Keep commits focused and use the repository's Conventional Commit style. Avoid unrelated formatting or refactoring so a change can later be proposed upstream with a small, understandable diff.

## Arch Linux and KDE Plasma 6.7 setup

These steps prepare a development checkout on the primary supported environment.

1. Install build and runtime dependencies:

   ```bash
   sudo pacman -S --needed \
     rust python python-pip python-pytest python-pillow python-numpy \
     python-pyqt6 qt6-svg python-gobject gtk4 libadwaita \
     gtk4-layer-shell python-cryptography dbus systemd-libs \
     libevdev hidapi ydotool git make base-devel
   ```

2. Build the Rust daemon:

   ```bash
   make build
   ```

3. For real-device testing, install the repository's udev rules and ensure the user has the device permissions described in [Installation](installation.md). Log out and back in if group membership changes.

4. Start the development checkout:

   ```bash
   ./scripts/juhradial-mx.sh
   ```

5. To inspect daemon behavior separately, stop the combined launcher and run:

   ```bash
   ./daemon/target/release/juhradiald --verbose
   ```

Do not run the application as root. Raw-device access should be provided through udev and the `input` group.

## Testing changes

Run the smallest relevant checks while iterating, followed by the broader suite before proposing a merge.

1. Format and statically check Rust code:

   ```bash
   cd daemon
   cargo fmt --check
   cargo clippy --all-targets --all-features -- -D warnings
   ```

   These verify standard Rust formatting and catch suspicious or non-idiomatic code across the daemon, tests, and benchmark targets.

2. Build and test the daemon:

   ```bash
   cd daemon
   cargo build --release
   cargo test --all-targets
   ```

   These verify the optimized binary builds and exercise the Rust unit/integration targets.

3. Run the Python regression tests from the repository root:

   ```bash
   python -m pytest tests/test_kde_placement.py \
     tests/test_overlay_placement_dpr.py \
     tests/test_settings_startup.py \
     tests/test_startup_latency.py \
     overlay/flow/test_marconi.py \
     overlay/flow/test_juhflow_bridge.py
   ```

   These cover KDE and fractional-scale placement math, settings/startup regressions, and JuhFlow protocol/bridge behavior. The image-measurement scripts in `tests/test_measure_segments.py` and `tests/test_placement.py` are visual diagnostics and may create output; use them when changing radial artwork or geometry.

4. Test launcher path handling:

   ```bash
   bash tests/test_launcher.sh
   ```

   This verifies both development and installed layouts and confirms a missing daemon fails clearly.

5. When changing installation or distribution support, run the disposable distro smoke tests with Podman (or set `RUNTIME=docker`):

   ```bash
   tests/distro_build_test.sh
   ```

   This downloads container images and toolchains, so it is slower and requires network access. It verifies daemon builds on the supported distribution families and checks known installer regressions.

6. Manually test hardware/UI behavior that applies to the change:

   - Confirm the radial menu opens at the pointer on KDE Plasma 6.7 Wayland.
   - Check every connected monitor, including fractional scaling and different monitor origins.
   - Exercise the changed mouse button or wheel with both press and release paths.
   - Confirm the settings UI saves, reloads, and survives a daemon or UI restart.
   - Reconnect or Easy-Switch the mouse when changing HID++ diversion, because device state is volatile.
   - Confirm default behavior still works when the feature is disabled.

Record the exact automated commands and manual environment in the merge request or handoff, including any checks that could not be run and why.

## Repeated workflows

Automation is appropriate when a task is repeated, deterministic, reviewable, and does not hide destructive or remote operations. Good candidates include a local aggregate check command, documentation validation, or packaging consistency checks. Git pushes, merges, releases, hardware actions, and changes requiring owner decisions should remain explicit unless the owner approves a narrowly scoped workflow.
