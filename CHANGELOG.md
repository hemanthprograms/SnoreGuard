# Changelog

All notable changes to SnoreGuard are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Changed
- Moved SnoreGuard out of the `hemanthprograms/hemanthprograms` profile repository into its own
  repository, with the app at the root.
- Split the README into docs for architecture, detection, testing and privacy, and added this
  changelog and a contributing guide.
- CI now lives in `.github/workflows/ci.yml` and runs on every pull request and on pushes to `main`.

## [1.0.0] - 2026-10-22

### Added
- iPhone MVP: onboarding, Tonight, Start night, Recording, Night detail, Results, History and Settings.
- On-device snore detection with SoundAnalysis, from the microphone or an imported audio file.
- Event merging, snore minutes per hour, and a remedy comparison with a permutation test.
- A 12-night test plan (3 baseline nights, then 3 nights for each of up to 3 remedies).
- Overnight recording with interruption and route-change recovery, and crash-safe checkpoints.
- Optional snore clips, CSV export, delete all data, and demo mode.
- Unit tests and GitHub Actions CI (simulator build and test, plus an Xcode 16 build).
