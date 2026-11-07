# SnoreGuard

[![CI](https://github.com/hemanthprograms/SnoreGuard/actions/workflows/ci.yml/badge.svg)](https://github.com/hemanthprograms/SnoreGuard/actions/workflows/ci.yml)
![iOS 17+](https://img.shields.io/badge/iOS-17%2B-blue)
![Swift](https://img.shields.io/badge/Swift-5.10%2B-orange)

SnoreGuard is an iPhone app that helps you find out **which remedy actually reduces your snoring**.
Each night you pick one remedy (or "No remedy" for a baseline night), leave your iPhone by the bed, and
the app measures snoring on the device. After a few nights it compares the remedies and tells you
honestly whether any of them helped, or that it's too early to say.

> SnoreGuard is a wellness tool, not a medical device. If you're worried about your sleep or
> breathing, talk to a doctor.

## Highlights

- **On-device sound classification** with Apple's SoundAnalysis model: 1.5 s windows, merged into snore
  events, normalized to *snore minutes per hour* of listening.
- **Honest statistics.** Remedies are compared against baseline nights with a one-sided permutation test,
  and the app says "Not enough data yet" rather than overclaim.
- **Built for a whole night.** Background audio, automatic recovery from calls, route changes and
  media-services resets, and a checkpoint every 5 minutes so a crash loses almost nothing.
- **Private by design.** No network calls, no accounts, no analytics, no third-party dependencies.
- SwiftUI, SwiftData, Swift Charts, AVFoundation, XcodeGen, unit tests and CI on GitHub Actions.

## Getting started

You need a Mac with Xcode 16 or later and [XcodeGen](https://github.com/yonaskolb/XcodeGen)
(the Xcode project is generated from `project.yml` and isn't checked in).

```sh
brew install xcodegen
make project               # or: xcodegen generate
open SnoreGuard.xcodeproj  # pick an iPhone simulator and press Run
```

From the command line:

```sh
make build   # xcodegen + xcodebuild build
make test    # xcodegen + xcodebuild test
```

**Demo mode.** Debug builds running in the Simulator turn demo mode on the first time they launch,
which fills the app with 12 realistic nights, so every screen has something to show without anyone
sleeping. Switch it on or off any time in Settings → Demo mode; switching it off removes only the demo
nights.

**On a device.** Set your team under Signing & Capabilities, run on an iPhone, allow the microphone,
plug the phone in and tap Start night.

## Screens

| Screen | What it does |
| --- | --- |
| Onboarding | Three pages: what the app does (with the disclaimer), microphone permission, pick up to 3 remedies to test. |
| Tonight (home) | Tonight's suggested remedy, plan progress ("Night 5 of 12"), last night's summary, Start night. |
| Start night | Choose exactly one remedy (defaults to the plan's suggestion), bedtime tips, test with an audio file. |
| Recording | Near-black screen with elapsed time, a live "Snoring detected" indicator, event count and a large Stop button. Fades almost to black after 20 s; tap to wake. Prompts you to plug in when on battery. |
| Night detail | Snore minutes per hour, totals, a timeline of events, snore minutes by hour, the event list and clip playback when clips are on. |
| Results | A one-sentence summary, a bar chart (mean ± 1 SD per remedy, dashed baseline line) and a row per remedy with its label. |
| History | Every night, newest first; swipe to delete. |
| Settings | Save snore clips, detection sensitivity, manage remedies, start a new plan, CSV export, delete all data, demo mode, the disclaimer. |

Every list and chart has an empty state that says what to do next. Text uses Dynamic Type styles and
the interactive elements and charts have VoiceOver labels.

## Documentation

| Doc | What's in it |
| --- | --- |
| [Architecture](docs/ARCHITECTURE.md) | Code layout, the `SnoreDetecting` seam, `RecordingService` and the data model |
| [Detection and analysis](docs/DETECTION.md) | The audio pipeline, every threshold, overnight recording, and how remedies are compared |
| [Testing](docs/TESTING.md) | What the unit tests cover, CI, and testing detection by hand |
| [Privacy](docs/PRIVACY.md) | What is and isn't stored, and how to export or delete it |
| [Contributing](CONTRIBUTING.md) | How to set up, make a change and open a pull request |
| [Changelog](CHANGELOG.md) | What changed in each version |

## Repository layout

```
SnoreGuard/        App sources (App, Models, Detection, Analysis, Services, Views, Resources)
SnoreGuardTests/   XCTest unit tests
Config/            Info.plist
Tools/             Scripts for the sample recording, the app icon and the CI warnings check
docs/              Architecture, detection, testing and privacy docs
project.yml        XcodeGen spec
Makefile           project / build / test / open / assets / clean
```

## Known limitations

- **One microphone can't tell whose snoring it hears.** A partner's snoring counts as yours. Sleeping
  alone gives the cleanest results.
- **The phone must stay plugged in.** Listening all night uses a lot of battery, and iOS may end the
  session if the battery runs low.
- **iOS decides what runs in the background.** Recording continues with the screen locked, but if
  another app takes over audio while SnoreGuard is in the background, iOS may not let it resume until
  you return to the app (it retries automatically, and resumes as soon as the app is opened).
- **Apple's classifier isn't perfect.** Breathing, a fan or a TV can occasionally score as snoring, and
  quiet snoring can be missed. Sensitivity trades one for the other.
- **The bundled sample is synthetic.** It was generated by `Tools/make_sample_audio.py` (no real
  recording was available). Apple's model does score it as snoring (peak confidence 0.84, both snoring
  stretches found as events), but a real night is the honest test: import one.
- **Small numbers.** A few nights can't prove much; the labels say so rather than overclaim.
- iPhone only, portrait only, English only.
