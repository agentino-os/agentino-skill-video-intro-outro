# video-intro-outro

> **Prepend a branded intro and/or append an outro to an existing video with smooth crossfades. Auto-normalises mismatched resolution / fps / codec.**

An Agentino skill that slots a pre-rendered intro or outro (or both) around a main video and returns a single MP4. Unlike the raw ffmpeg `concat` demuxer, this skill re-encodes the inputs to a common spec — so you can keep a 10-second brand intro recorded once and reuse it across dozens of main videos with different formats.

Optional video+audio xfade at each join so there's no hard cut.

## Install

Requires [Agentino](https://github.com/dagoSte/agentino) ≥ `1.2.0-rc.1` and `ffmpeg`.

```bash
brew install ffmpeg                                                    # macOS
# sudo apt install ffmpeg                                              # Debian / Ubuntu

agentino marketplace install dagoSte/agentino-skill-video-intro-outro
agentino skill show video-intro-outro
```

## Use

### Intro + outro

```bash
agentino skill exec video-intro-outro \
  -i video_path=/tmp/talk.mp4 \
  -i intro_path=/tmp/brand_intro.mp4 \
  -i outro_path=/tmp/brand_outro.mp4 \
  -i output_path=/tmp/talk.branded.mp4 \
  -i fade_s=0.5
```

### Intro only

```bash
agentino skill exec video-intro-outro \
  -i video_path=/tmp/talk.mp4 \
  -i intro_path=/tmp/brand_intro.mp4 \
  -i fade_s=0.75
```

### Example output

```json
{
  "output_path": "/tmp/talk.branded.mp4",
  "intro_duration_s": 3.0,
  "main_duration_s": 613.2,
  "outro_duration_s": 5.0,
  "output_duration_s": 620.2,
  "resolution": "1920x1080",
  "fps": 30,
  "fade_s": 0.5,
  "intro_present": true,
  "outro_present": true
}
```

## Use with agentino run

```bash
agentino run "add /tmp/brand_intro.mp4 before and /tmp/brand_outro.mp4 after /tmp/talk.mp4 with a half-second crossfade, save to /tmp/talk.branded.mp4"
```

The LLM planner picks this skill because the prose mentions "before" + "after" + a target video.

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `video_path` | string | **required** | The main video. |
| `intro_path` | string | `""` | Optional clip to prepend. At least one of intro/outro must be set. |
| `outro_path` | string | `""` | Optional clip to append. |
| `output_path` | string | `<video>.branded.<ext>` | Destination MP4. |
| `fade_s` | number | `0.5` | Crossfade length at each join. `0` = hard cuts. |
| `resolution` | string | first clip's WxH | Output resolution. |
| `fps` | integer | first clip's fps | Output frame rate. |
| `bg_color` | string | `black` | Letterbox colour when aspect ratios mismatch. |

## Outputs

- **`output_path`** — absolute path to the branded MP4.
- **`intro_duration_s`** / **`main_duration_s`** / **`outro_duration_s`** — per-clip durations from `ffprobe` (0 if the clip wasn't provided).
- **`output_duration_s`** — final length = sum − (joins × fade_s).
- **`resolution`**, **`fps`**, **`fade_s`** — resolved spec.
- **`intro_present`**, **`outro_present`** — which joins actually happened.

## How it works

1. **Probe** the main video to derive default resolution + fps. Probe intro/outro for duration.
2. **Normalise every input** — `scale + pad + setsar + fps + setpts` to the common output spec. Audio likewise gets `aresample + asetpts`.
3. **Join** — if `fade_s == 0`, concat demuxer on all normalised streams; otherwise chain `xfade` + `acrossfade` for each pair with an offset tracking the running timeline.
4. **Render** in one ffmpeg pass — `libx264` CRF 20 + AAC 192 kbps + `+faststart`.
5. **Re-probe** the output for the final duration.

## Safety

- `sandbox_level: none` (ffmpeg discovery).
- `network_access: false`.
- `file_access: read-write`.
- `agentino skill exec skill-security-check -i path=skill.yaml` → zero findings.

## License

MIT — see [`LICENSE`](./LICENSE).
