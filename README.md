# HyperEngine Game Turbo Pro

By **nc**. MIT licensed.

An Android game-booster and in-game HUD: a Game Space library, a side-swipe overlay with performance tools, a one-tap
**Wild Boost** (also a Quick Settings tile), per-game profiles, and a set of low-latency and capture tools called **HyperLab**.
Kotlin, no layout XML (the UI is built in code), custom vector icons throughout.

> **Not affiliated with NVIDIA.** This is independent software. It is not made by, endorsed by or licensed from NVIDIA,
> AMD, Google, Xiaomi or any game publisher. Some features are inspired by desktop GPU technologies but are original, far
> smaller approximations built on public Android APIs. See [NOTICE.md](NOTICE.md).

> **Status: pre-release.** The code has been statically checked (manifest, resources, references) but has **not been
> compiled or run on a device by its author's tooling**. Expect to fix build errors on first sync, and treat every
> "works on ROM X" claim below as unverified until you have tried it on your own phone.

> **UI redesign in progress** (HyperOS Game Turbo x RedMagic landscape console).

## What it does

| Area | Feature |
| --- | --- |
| Wild Boost | Fixed-performance mode, OEM throttler freeze, RAM clean, background-data block, game mode PERFORMANCE, refresh-rate lock, optional DEX compile. Also a **Quick Settings tile** and a 2.2 s ignition animation. |
| Refresh rate | Five-layer override and a watchdog that re-applies the lock when a game regains focus. The panel is the ceiling (144 on a 120 Hz screen becomes 120). |
| HyperBoost | Render-scale profiles (Ultra Quality 0.85x, Quality 0.75x, Balanced 0.67x, Ultra Performance 0.55x) through `cmd game` / `device_config` plus SurfaceFlinger pacing properties. |
| HyperLab | **Low-latency mode** (live latency estimate), **Clip buffer** (rolling RAM recording, "Save last clip"), **Game colour filters** (per game: clarity, vibrance, HDR toning, night, colourblind assist) and **Refresh-rate matching**. |
| Virtual-display upscaler | The game runs on a private low-resolution virtual display; its picture is upscaled (bicubic) and sharpened (contrast-adaptive) to your native screen on the GPU. Touches are forwarded to the virtual display. Needs Shizuku. Card in HyperLab. |
| Voice studio | Voice effects with a self-recovering capture loop, hardware sample-rate negotiation and mic-source fallback. |
| Landscape HUD | Full-screen console HUD in landscape (status bar, quick-toggle rail, Dashboard / All tools tabs, 4-face Energy Cube, brightness/volume, gamepad key hints), switchable in Console settings. Gamepad / keyboard navigation. |
| Console tools | Competitive Mode Quick Settings tile (slider sound + haptic, landscape lock, full Wild Boost), Volume Up + Down chord, all-apps freeform launcher, screenshot, built-in screen recorder, cast / ADB helper. |
| Calls and input | In-game call banner with hands-free answer on speaker, keyboard and mouse mapper (WASD joystick, key taps). |
| Accessibility | Auto tapper and a self-healer that re-enables and re-binds the accessibility service (needs Shizuku, root or the secure-settings grant). |
| HUD | Edge trigger, sidebar with live CPU / temperature / RAM dials and a spinning fan, crosshair, pointer, macro, danmaku ticker, AFK lock. |
| Slide bar | HyperOS Game Turbo style edge tab and dock: all apps and chat apps in floating windows, mini browser, quick notes, calculator and your pinned apps. |

## Requirements

- Android 8.0+ (`minSdk 26`). Several features need Android 10-12+ and say so in the UI.
- **Shizuku** (or root) for nearly everything that changes system behaviour. Without it the app still opens, shows the
  library and the HUD, and reports "needs Shizuku" per feature instead of failing silently.
- Android Studio with AGP 8.1.4, Kotlin 1.9.22, `compileSdk 34`, JDK 8 target.

## Build

1. Open the project folder in Android Studio (JDK 17 for Gradle) and let Gradle sync. Android Studio creates the Gradle wrapper if it is missing.
2. Run the `app` configuration on a device (an emulator has no Shizuku and no real panel modes).
3. Start Shizuku, open the app, and use **Permissions and engine diagnostics** (hub banner / menu): each requirement has a
   one-tap fix button.

To change an icon, edit geometry or colours in `tools/gen_icons.py` and run
`python3 tools/gen_icons.py app/src/main/res/drawable`. Do not edit the generated XML by hand.

## What it cannot do

These limits come from Android, not from missing effort. The app labels them in the UI where they matter.

- **A game cannot hear the changed voice.** Android gives apps no way to feed audio into another app's microphone. The voice
  studio is a monitor and recording tool. The permission and focus handling only keeps *our* mic open.
- **No pixel-level sharpening of a game on the phone screen itself.** An overlay window can only draw its own pixels. "Clarity"
  in the colour filters is contrast around mid-grey, and "vibrance" is saturation. Colour filters use a SurfaceFlinger colour
  matrix (Shizuku or root). The one route that does touch pixels is the **virtual-display upscaler**: the game runs on a separate
  display and we draw its picture ourselves. It is a spatial filter (bicubic + contrast-adaptive sharpening, in the spirit of
  FSR 1 / CAS), not DLSS-style AI upscaling, and it has its own limits (60 Hz virtual display, black for `FLAG_SECURE` games,
  games that refuse a second display). See `docs/FIX_VIRTUAL_UPSCALER_NOTES.md`.
- **No variable refresh rate API exists for apps.** Refresh-rate matching reads the game's real FPS and moves the panel to the
  smallest supported mode that is a whole multiple of it. It needs FPS to be readable on the device.
- **Render scale depends on the game and ROM.** The GameManager downscale needs Android 12+ and a game that honours it; the
  `debug.sf.*` and `debug.hwui.*` properties are ROM dependent. Each step reports its own result. The "estimated gain" labels
  are for GPU-bound games, not promises.
- **The clip buffer records video only** (no game audio), records black for `FLAG_SECURE` games, fixes its capture size at start,
  and costs some GPU/CPU while it runs. Nothing touches the disk until you save.
- **Low-latency mode cannot flush input queues or change input-dispatcher priority.** It sets what a shell can set, and the
  latency figure is an estimate (frame time + half a refresh + a typical touch wait), not a measurement.
- Some ROMs refuse to bind a background accessibility service whatever is written to settings, and some ignore the Back action
  that collapses the shade after a tile tap.

## Privacy

- No account, no analytics, no ads, no cloud. Profiles and clips stay on the device (`allowBackup` is off).
- `INTERNET` is used only for the latency readout: a TCP connect to a public endpoint (three AWS DynamoDB regional hosts by default;
  the profile picks which one). No game data is sent.
- The accessibility service declares `canRetrieveWindowContent="false"`. It listens for touch start / end and window changes
  (package name only) so the auto tapper and refresh-rate watchdog work. It does not read what is on screen.
- The notification listener is used only by the optional danmaku ticker.

## Permissions

Every permission in `AndroidManifest.xml` has a caller in the code. landscape HUD and console features. Three deserve attention before you publish:

- `QUERY_ALL_PACKAGES`: needed to list installed games. **Google Play restricts this permission** and requires a declaration;
  a GitHub / F-Droid build is not affected.
- `WRITE_SECURE_SETTINGS`, `DUMP`, `READ_LOGS`: protected permissions that are only usable after a one-time grant through
  Shizuku or ADB. Declaring them does nothing by itself.
- The Android 14+ screen-capture consent dialog appears again for every clip-buffer session.

## Repository layout

```
app/src/main/java/com/hyperengine/gameturbo/
  core/      engines: boost, refresh rate, HyperBoost, GamingSuiteManager, ClipBufferEngine, VirtualDisplayUpscaler + UpscaleRenderer, voice, self-healer
  service/   game detector, Quick Settings tile, voice foreground service, HUD controller
  ui/        HUD, sidebar, overlays, AnimationController, LabPanel, VIcon, Brand (all HyperLab names)
  hub/       Game Space library, carousel, setup, HyperLab page
  data/      profiles and persistence
app/src/main/res/drawable/   custom vector icon pack (generated by tools/gen_icons.py)
docs/        phase notes: what was built, how, and the known limits of each part
```

## Development notes

- **Part 9:** ROM Lab tab in Game Space (live Hz / 1% low / temperature, AOT compile, AnTuTu prep, panel lock).
- **Part 10:** `GameSessionGuard` locks the panel and starts the frame smoother when a game comes forward, and restores both afterwards.
- **Part 11:** 12 bug fixes and lag reductions (clip buffer crash, main-thread stalls, per-frame allocations).

## Contributing

Issues and pull requests are welcome. Say which device and ROM you tested on; most behaviour here is ROM dependent.

## Licence

[MIT](LICENSE) (c) 2026 nc. Third-party libraries keep their own licences: see [NOTICE.md](NOTICE.md).
