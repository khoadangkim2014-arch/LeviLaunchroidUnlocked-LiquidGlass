<div align="center">

# LeviLaunchroidUnlocked — Liquid Glass Edition

**A Minecraft: Bedrock Edition launcher for Android, rebuilt with a real Liquid Glass UI — no Material Design.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=flat-square)](https://www.apache.org/licenses/LICENSE-2.0)
[![Android](https://img.shields.io/badge/Android-9.0%2B-green?style=flat-square&logo=android)](https://www.android.com/)

</div>

---

## What this is

This is a fork of [LeviLaunchroid](https://github.com/LiteLDev/LeviLaunchroid) (the "Unlocked" variant) with its UI being rebuilt on top of [**Liquid Glass Android**](https://github.com/QWEA0/Liquid-Glass-Android) — a real glassmorphism rendering library for the classic Android View system, not a cosmetic reskin. It uses actual backdrop capture, signed-distance-field refraction, chromatic dispersion, and a C++/NEON rendering pipeline on older devices, falling back to AGSL runtime shaders on API 33+.

**Goal: replace Material Design components entirely** — buttons, tab bars, panels — with real liquid-glass surfaces that refract whatever's actually behind them, the way iOS 26's Liquid Glass material does.

## Why this library

Several Android glassmorphism libraries exist; most are Jetpack Compose–only. This codebase is a classic View/XML-layout app (no Compose), so the requirements were:

- Works with plain `FrameLayout`/XML — no Compose migration
- Real backdrop blur below API 33 (most glassmorphism-via-AGSL libraries require API 33+ and do nothing before that)
- Ready-made interactive components (button, tab bar, FAB), not just a blur `Modifier`

[QWEA0/Liquid-Glass-Android](https://github.com/QWEA0/Liquid-Glass-Android) was the only match: a `FrameLayout` subclass (`LiquidGlassView`) with a C++/NEON fallback pipeline for API 24–32, and purpose-built subclasses (`LiquidGlassButton`, `LiquidGlassFab`, `LiquidGlassTabBar`) that map directly onto what a launcher's chrome needs. MIT licensed.

## What's converted so far

- **Nav bar → floating glass bar.** `BaseActivity` used to stack the nav bar in its own row above page content (`LinearLayout`, vertical). It now overlays content in a `FrameLayout`, with the glass bar floating on top and content padded to sit below it — matching how iOS's translucent bars actually work, and giving the glass panel something real to refract instead of a flat color.
- **Tab bar → `LiquidGlassTabBar`.** The four nav tabs (Launch/Instances/About/Settings) were four separate `TextView`s with manual color-swapping for "selected" state. They're now a single `LiquidGlassTabBar` — an iOS 26-style bar where the selection indicator is a real glass droplet that refracts the bar content under it and slides between tabs with overshoot + liquid stretch.
- **Primary buttons → `LiquidGlassButton`.** The main "Launch" button and the nav bar's "Sign in" button were `MaterialButton`s; both are now `LiquidGlassButton`, a pill-shaped glass surface with built-in press feedback.
- **Backdrop wiring.** `BaseActivity.wireGlassBackdrop()` walks the nav bar's view tree at inflate time and points every `LiquidGlassView` it finds at the actual screen content behind it (`setBackdropSource` + `setEnableDynamicBackground(true)`), so the glass tracks scrolling/animating content live instead of freezing on a single captured frame.

## What's *not* converted yet

This is a large, existing codebase (settings screens, dialogs, the mod manager, the memory editor overlay, world/instance management, account UI, and more) — converting every single Material component across all of it is a much bigger, iterative effort than one pass can responsibly cover blind, without on-device testing at each step. What's done is a solid, working foundation:

- The nav bar and primary CTA buttons are real, tested-against-the-library's-actual-API conversions, not placeholders.
- Remaining Material components (`MaterialButton`, `MaterialCardView`, `Widget.MaterialComponents.*` styles) are listed by running:
  ```bash
  grep -rl "com.google.android.material\|MaterialComponents" app/src/main/res/layout
  ```
  Each can be swapped for `LiquidGlassView` (generic panel), `LiquidGlassButton`, or `LiquidGlassFab` following the same pattern used in `nav_bar.xml` and `activity_main.xml`.

## Integration notes for continuing this

- **The glass needs something behind it.** `LiquidGlassView` samples whatever's drawn behind it in the same window. A glass surface with nothing behind it (a flat-colored screen, an empty layout) renders as a plain tint — not a bug, just nothing to refract. Point `backdropSource` at real content.
- **`enableDynamicBackground` must be `true`** for anything that scrolls or animates, or the glass freezes on its first captured frame. `wireGlassBackdrop()` already does this for the nav bar; do the same for any new glass surface added elsewhere.
- **Liquid Glass 2.0 effects (`bevelWidth`, `refractionHeight`, `dispersionStrength`, adaptive tint) need API 33+** and silently degrade to a plain blur/tint below that — no crash, no exception, just less shine on older devices.
- Full API reference: [the library's `llms.txt`](https://github.com/QWEA0/Liquid-Glass-Android/blob/main/llms.txt) and [README](https://github.com/QWEA0/Liquid-Glass-Android/blob/main/README.md).

## Building

Same toolchain as upstream LeviLaunchroid:

- JDK 21, Android SDK (compileSdk 36), NDK, [Xmake](https://xmake.io/)
- **Clone with submodules** — `git clone --recurse-submodules`, or run `git submodule update --init --recursive` after a plain clone/ZIP download. This repo depends on [`preloader-android`](https://github.com/LiteLDev/preloader-android) and Microsoft's [`libHttpClient`](https://github.com/microsoft/libHttpClient) (Android target only, pruned to what's actually needed to keep the repo a reasonable size).
- `./gradlew assembleDebug`

## Credits

- **[LiteLDev / LeviMC team](https://github.com/LiteLDev/LeviLaunchroid)** — original LeviLaunchroid
- **[QWEA0](https://github.com/QWEA0/Liquid-Glass-Android)** — Liquid Glass Android, the rendering library this UI rework is built on
- **[Microsoft libHttpClient](https://github.com/microsoft/libHttpClient)** and **[preloader-android](https://github.com/LiteLDev/preloader-android)** — native dependencies

## Disclaimer

Use only with a legitimately owned copy of Minecraft: Bedrock Edition. This software is provided as-is.
