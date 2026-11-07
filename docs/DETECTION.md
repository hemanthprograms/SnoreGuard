# Detection and analysis

How SnoreGuard turns microphone audio into snore events, and how it decides whether a remedy helped.

## Detection pipeline

```
microphone ─► AVAudioEngine tap (8192 frames) ─┐
                                               ├─► SNAudioStreamAnalyzer + SNClassifySoundRequest(.version1)
audio file ─► AVAudioFile, 16384-frame chunks ─┘          │  1.5 s windows, 50% overlap
                                                          ▼
                                     "snoring" confidence per window (DetectionWindow)
                                                          ▼
                              EventMerger: window counts if confidence ≥ threshold,
                              merge when gap < 10 s, drop events < 3 s
                                                          ▼
                       Night: events, total minutes, minutes per hour of listening
```

All thresholds live in `Detection/DetectionConfig.swift`:

| Constant | Value | Meaning |
| --- | --- | --- |
| `windowDuration` | 1.5 s | Preferred classifier window (clamped to what the classifier allows) |
| `windowOverlap` | 0.5 | 50% overlap, so a new window every 0.75 s |
| `defaultThreshold` | 0.6 | Window counts as snoring at or above this confidence |
| Sensitivity | Low 0.7 / Normal 0.6 / High 0.5 | User-facing threshold (Settings) |
| `mergeGap` | 10 s | Snoring windows closer than this join one event |
| `minimumEventDuration` | 3 s | Shorter events are dropped |
| `minimumNightDuration` | 30 min | Shorter nights are kept but excluded from results and plan progress |
| `maxClipsPerNight` / `clipDuration` | 10 / 10 s | Only when "Save snore clips" is on |
| `checkpointInterval` | 5 min | How often an in-progress night is saved |

**Overnight recording.** The app declares the `audio` background mode and uses an `AVAudioSession`
with category `.record` and mode `.measurement`. On an interruption (a call, Siri) listening pauses and
then resumes automatically when it ends, whether or not the system suggests resuming. Route changes,
engine configuration changes and media-services resets rebuild the engine with the new input format. If
a restart fails it retries every 15 seconds and again when the app returns to the foreground. Time spent
paused is excluded from the listening time, so per-hour numbers stay honest. While the phone is
charging the screen stays on (dark); on battery it may lock as usual and listening continues in the
background.

**Crash safety.** A night in progress is saved every 5 minutes. If the app is killed before Stop, the
next launch keeps everything up to the last save and marks the night "Recovered".

## How remedies are compared

For each remedy with at least one counted night, SnoreGuard shows the mean snore minutes per hour, the
number of nights and the change against the baseline mean. Labels come from `RemedyAnalyzer`:

| Label | Rule |
| --- | --- |
| Not enough data yet | Fewer than 3 nights for the remedy or the baseline (shows how many more are needed) |
| Likely helps | At least 20% less snoring **and** a one-sided permutation test p < 0.1 |
| Might help | At least 10% less snoring |
| Seems worse | At least 10% more snoring |
| No clear effect | Anything else |

The permutation test asks: if remedy and baseline nights were shuffled at random, how often would the
remedy look at least this good? Small groups are enumerated exactly; larger groups use 10,000 seeded
shuffles so results are repeatable. With 3 nights against 3, "Likely helps" needs every remedy night to
beat every baseline night (p = 1/20), which is a deliberately high bar for so little data.

The summary sentence names the clearest winner, for example "Sleeping on your side cut your snoring by
about 40% over 4 nights.", and stays cautious otherwise ("…might help…, but it's too early to be sure",
"None of the remedies you've tried has made a clear difference yet").

**The plan.** 3 baseline nights, then 3 nights each of up to 3 remedies picked in onboarding (up to 12
nights). Nights count toward the step that matches their remedy, so an off-plan night is never wasted.
The home screen always suggests the first unfinished step.

