# degenview-eframe

> **This is a fork.** It is [`eframe`](https://github.com/emilk/egui/tree/main/crates/eframe) 0.36.2
> from the [egui](https://github.com/emilk/egui) project (Emil Ernerfeldt and contributors) with one
> small Wayland fix, published for [DegenView](https://github.com/degenview/degenview). It is not
> affiliated with or endorsed by the egui project. If you do not need the fix, use
> [`eframe`](https://crates.io/crates/eframe).

## What it changes

On Wayland, a viewport the app hid is no longer painted or presented.

Wayland cannot hide a toplevel window. Upstream eframe presents every viewport on every pass, which
maps a viewport the app asked to hide, and winit then gates that window's redraws on frame callbacks
that compositors withhold from a window nobody sees (minimized, or on another virtual desktop). A
hidden root viewport that drives other viewports then stalls until the user makes it visible again:
dialogs that should close stay open, new windows do not open. This is
[degenview/degenview#622](https://github.com/degenview/degenview/issues/622).

With this fork, `Viewport::hidden` records that the app asked for the viewport to be hidden (through
`NativeOptions::viewport` or the viewport builder, and then through every
`ViewportCommand::Visible`). On Wayland such a viewport still runs its ui pass, so `App::update` and
the viewports it creates keep working, but it is not tessellated, painted or presented. Its surface
never gets a buffer and stays unmapped. Its texture deltas wait for the next painted viewport, as
they already do for a minimized one. Other platforms and visible viewports behave as in upstream.

The whole change is the second commit of this repository, in `src/native/wgpu_integration.rs` and
`src/native/glow_integration.rs` (same shape in both renderers). Everything else in `src/` is
byte-identical to upstream.

## Base

| | |
| --- | --- |
| Upstream | `eframe` 0.36.2, `crates/eframe` of <https://github.com/emilk/egui> (commit `49682f8baa058bf49e011035cfbd6e825f88a5ef`, from the crate's `.cargo_vcs_info.json`) |
| Imported from | <https://static.crates.io/crates/eframe/eframe-0.36.2.crate> |
| sha256 of that file | `55ad41fac6e3149abb4866e93df37dbd33baa95db525c3368c944584e95a5dc2` |
| Version here | `0.36.2`, the same number, so a dependency on this crate names the eframe release it is based on |

The first commit of this repository is the extracted crate, unmodified; `git diff` from it shows
every change. Besides the source fix, the package metadata in `Cargo.toml` is ours (name,
description, repository, keywords, `include`), `Cargo.toml.orig`, `.cargo_vcs_info.json` and
`Cargo.lock` of the tarball are not kept (the first two describe upstream's workspace, the lock file
is not used by dependents), and the license texts are added (the crates.io tarball does not ship
them).

## Use

The library is still called `eframe`, so `use eframe::...` keeps working:

```toml
[dependencies]
eframe = { package = "degenview-eframe", version = "=0.36.2", features = ["persistence"] }
```

Features are those of eframe 0.36.2. Pin the version exactly: this crate follows DegenView's needs,
not upstream's release schedule.

## Retirement

The crate goes away once egui ships an equivalent fix; DegenView then depends on `eframe` again. It
will not be updated for new egui releases unless DegenView still needs it. Other changes belong in
egui, not here.

## License

`MIT OR Apache-2.0`, as upstream. See [LICENSE-MIT](LICENSE-MIT) and
[LICENSE-APACHE](LICENSE-APACHE), copied from <https://github.com/emilk/egui>. The code is
copyrighted by Emil Ernerfeldt and the egui contributors; the fix is contributed under the same
terms.
