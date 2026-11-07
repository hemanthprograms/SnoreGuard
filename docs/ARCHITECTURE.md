# Architecture

MVVM-ish SwiftUI. Views read SwiftData with `@Query`; anything with logic lives in plain types that are
easy to test.

```
SnoreGuard/
├── App/          SnoreGuardApp, RootView + tabs, launch bootstrap, settings keys, Sensitivity
├── Models/       SwiftData: Night, SnoreEvent, Remedy, Plan
├── Detection/    SnoreDetecting protocol, LiveSnoreDetector, FileSnoreDetector,
│                 SoundClassifierPipeline, EventMerger, NightTotals, SnoreClipRecorder, DetectionConfig
├── Analysis/     RemedyAnalyzer, Statistics (permutation test), PlanEngine, ResultsBuilder
├── Services/     RecordingService, DemoData, CSVExporter, RemedyCatalog, NightStore, ClipStore, ClipPlayer
├── Views/        One folder per screen, plus Theme, Format and shared components
└── Resources/    Asset catalog, SampleSnoring.m4a
SnoreGuardTests/  XCTest unit tests
Tools/            Scripts that generate the sample recording and the app icon
```

- **`SnoreDetecting`** is the seam between audio and everything else. `start()` returns an
  `AsyncThrowingStream` of classified windows (plus paused/resumed signals); `stop()` ends it.
  - `LiveSnoreDetector`: `AVAudioEngine` input tap → `SoundClassifierPipeline`. Handles the audio session.
  - `FileSnoreDetector`: reads any audio file in chunks → the same `SoundClassifierPipeline`.
- **`RecordingService`** (`@MainActor @Observable`) owns one night: it consumes the stream, feeds an
  `EventMerger`, drives the live indicator, saves clips if enabled, checkpoints the night to SwiftData
  every 5 minutes and finalizes it on Stop.
- **`EventMerger`**, **`NightTotals`**, **`RemedyAnalyzer`**, **`Statistics`** and **`PlanEngine`** are pure
  value-type logic with no UI or persistence dependencies, and are unit tested.

## Data model

- `Night`: start and end time, remedy, listening time (`recordedDuration`), total snore minutes,
  **snore minutes per hour** (the main metric), event count, the threshold it was recorded with, its
  source (microphone, file, demo) and its events. No audio.
- `SnoreEvent`: start, duration, peak confidence, and an optional clip file name.
- `Remedy`: name, SF Symbol, a summary phrase ("Sleeping on your side"), built-in or custom. Exactly
  one built-in remedy is the baseline ("No remedy").
- `Plan`: the ordered remedies to test (baseline first) and nights per step (3).

