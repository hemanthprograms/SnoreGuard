# Contributing

Thanks for taking a look. SnoreGuard is a small project, so the process is light.

## Setup

1. Install Xcode 16 or later and XcodeGen (`brew install xcodegen`).
2. Run `make project` and open `SnoreGuard.xcodeproj`. The project file is generated from
   `project.yml`, so change targets and settings there, not in Xcode.
3. Run `make test` before you start, to make sure everything passes on your machine.

## Making a change

- Branch from `main` and keep each pull request to one change.
- Put logic in plain, testable types (see `Detection/`, `Analysis/`), not in views, and add or update
  tests in `SnoreGuardTests/` for anything that changes behaviour.
- Detection thresholds live only in `Detection/DetectionConfig.swift`. If you change one, update the
  table in [docs/DETECTION.md](docs/DETECTION.md).
- Keep the privacy promises in [docs/PRIVACY.md](docs/PRIVACY.md): no network calls, no third-party
  dependencies, no raw audio on disk unless the user turns on clips.
- The build must stay free of warnings. CI fails on any warning or error in the project's sources.
- Use Dynamic Type text styles and give new controls and charts VoiceOver labels.

## Pull requests

Describe what changed and why, and how you tested it (simulator, device, imported recording). CI runs
the build and tests on every pull request, and a pull request merges once it's green and reviewed.

Add a line to the "Unreleased" section of [CHANGELOG.md](CHANGELOG.md) for anything a user would notice.
