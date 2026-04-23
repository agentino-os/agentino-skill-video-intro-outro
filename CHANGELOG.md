# Changelog

All notable changes to the `video-intro-outro` Agentino skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this skill adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-04-23

First public release for the Agentino marketplace.

### Added

- `agentino skill exec video-intro-outro` prepends an intro and/or appends an outro to a main video:
  - Accepts mismatched resolution / fps / codec and normalises all clips to a common output spec.
  - Configurable video+audio crossfade at every join (`fade_s=0` = hard cuts).
  - Letterboxes aspect-ratio mismatches with configurable `bg_color`.
  - At least one of `intro_path` / `outro_path` required — refusing no-op calls.
  - Single `filter_complex` pass with `libx264` CRF 20 + AAC 192 kbps + `+faststart`.

### Tested

- 2 s intro + 13.57 s main + 2.5 s outro + `fade_s=0.5` → 17.1 s 1080p MP4 (expected ≈ 17.07 s accounting for 2 joins × 0.5 s fade overlap).
- Zero findings from `agentino skill exec skill-security-check -i path=skill.yaml` (fail-on = high).

### Requires

- Agentino ≥ 1.2.0-rc.1
- `ffmpeg` (and `ffprobe`) on `PATH`
