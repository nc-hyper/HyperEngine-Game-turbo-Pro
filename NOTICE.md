# Notice

## Not affiliated

HyperEngine Game Turbo Pro is independent, community software. It is **not made by, endorsed by, sponsored by,
licensed from or affiliated with NVIDIA Corporation, Advanced Micro Devices (AMD), Google, Xiaomi, Shizuku's authors,
or any game publisher or device maker.**

Several features are inspired by well-known desktop GPU technologies (render-scale upscaling, low-latency modes, instant
clip capture, screen filters, adaptive refresh). They are original, much smaller software approximations that work through
public Android APIs and, where the user allows it, a Shizuku or root shell. They do not use, contain or emulate any vendor
code, driver or algorithm, and they do not deliver what the vendor products deliver. Their names in this project are
descriptive on purpose (HyperLab, Low-latency mode, Clip buffer, Game colour filters, Refresh-rate matching, HyperBoost
render scale).

## Trademarks

All product names, logos and brands mentioned anywhere in this repository (including NVIDIA, GeForce, DLSS, Reflex,
ShadowPlay, Freestyle and G-SYNC, which are trademarks of NVIDIA Corporation; Android and Google Play, trademarks of Google
LLC; HyperOS and Xiaomi, trademarks of Xiaomi) are the property of their respective owners. They are mentioned only to say
what this project is not, or to describe compatibility. Please do not present forks or builds of this project as official
products of any of those companies.

## Third-party software

The app is built with the libraries below. Each keeps its own licence, which applies to that library only.

| Library | Used for | Licence |
| --- | --- | --- |
| Shizuku API (`dev.rikka.shizuku:api`, `provider`) | privileged shell / binder access | MIT |
| AndroidHiddenApiBypass (`org.lsposed.hiddenapibypass`) | reflecting on a hidden input API | Apache-2.0 |
| kotlinx.coroutines | background work | Apache-2.0 |
| Gson | profile persistence | Apache-2.0 |
| AndroidX (core-ktx, dynamicanimation) | window insets, spring physics | Apache-2.0 |

The Shizuku licence column was confirmed from the published Maven metadata; the others are listed from the
projects' public documentation. Check the licence file of each exact pinned version (see `app/build.gradle.kts`) before
you publish a release build, and ship their notices with it where the licence requires that.

The Shizuku manager app has its own terms (for example, apps must not use "Shizuku" as their name or declare
`moe.shizuku.manager.permission.*` permissions). This project only *uses* the Shizuku API and does neither.
