# mk96-ai-claude-yt-plugin

Claude Code plugin (`mk96-yt`) with YouTube tools. Each tool is an independent *feature*.

| Feature | Command | What it does |
|---|---|---|
| shorts | `/mk96-yt:shorts <url or path>` | Turns a long video into vertical 9:16 shorts |
| shorts | `/mk96-yt:shorts-style` | Creates and manages styles: a full-screen frame with a video window and a caption area |

## Install
```
claude plugin marketplace add mk96-ai/mk96-ai-claude-yt-plugin
claude plugin install mk96-yt@mk96-ai-claude-yt-plugin
```
To try it without installing: `claude --plugin-dir <path to this repository>`.

## First run on a machine
```
python core/mk96yt.py setup        # Python environment + models (--cpu if there is no NVIDIA GPU)
python core/mk96yt.py doctor       # checks ffmpeg, node, NVENC, dependencies and models
```
Requirements: Python 3.10+, ffmpeg/ffprobe on the PATH, Node.js (YouTube downloads only).

## Architecture
```
.claude-plugin/          plugin.json, marketplace.json
core/                    shared infrastructure (standard library only, no feature logic)
  mk96yt.py              entry point: setup | doctor | features | <feature> <command>
  mk96yt_core/           paths, jsonio, config (layers), proc, features (discovery), setup, doctor, cli
skills/<feature>/        one folder per feature; the core discovers it through its feature.json
  SKILL.md               workflow Claude follows (defines the /mk96-yt:<feature> command)
  feature.json           dependencies, models and system tools it needs
  defaults.json          default settings
  src/mk96yt_<feature>/
    commands/            one command per file (entry layer: arguments and output)
    services/            logic with side effects (ffmpeg, audio, detection, transcription...)
    domain/              pure rules (language, timeline, transcript, canvas)
    settings.py          layered settings loading
skills/shorts-style/     SKILL.md only: shorts command to manage styles (the code lives in shorts)
```
Dependencies: `commands` → `services` → `domain`, and everything → `mk96yt_core`. Nothing depends upwards.

### Adding a new feature (e.g. publish)
1. Create `skills/publish/` with `SKILL.md` (shows up as `/mk96-yt:publish`), `feature.json` and `src/mk96yt_publish/commands/<command>.py`.
2. Done: `mk96yt setup`, `mk96yt doctor` and `mk96yt publish <command>` pick it up without touching `core/`.

## Where files are stored (optional environment variables)
| Variable | Contents | Default |
|---|---|---|
| `MK96YT_HOME` | Python environment, models, `<feature>.json` settings | `~/.mk96-yt` |
| `MK96YT_WORKSPACE` | `input/` and `work/<id>/` | `<HOME>/workspace` |
| `MK96YT_OUT` | delivery: `<id>/short_<n>.mp4`, `publicar.md`, `panel.md` | `<WORKSPACE>/output` |
| `MK96YT_PYTHON` | ready-made interpreter to use instead of the plugin's own environment | (unset) |
