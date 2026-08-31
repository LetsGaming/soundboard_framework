# Soundboard Framework

A reusable framework for cosplay/character soundboards: a Flask server
plays sounds (e.g. on a Raspberry Pi hidden in a costume, connected to a
speaker), and any phone on the same network is a remote control at
`http://<server-ip>:5000`. Playback is non-blocking, sample-accurate
gapless looping via `sounddevice`/`soundfile`, and the frontend is an
installable PWA.

A "character" is a small project — a `character.toml` config file, an
`icons/` folder, and a `sounds/` folder of `.ogg` files — that consumes
this package. See [LetsGaming/viego_sound_player](https://github.com/LetsGaming/viego_sound_player)
for a real example.

## Install

```bash
pip install git+https://github.com/LetsGaming/soundboard_framework.git
```

## Creating a character project

```bash
soundboard-new "My Character"
cd my_character
# edit character.toml (theme colors, category labels, fade timings)
# drop icons into icons/ (favicon.ico, icon-192.png, icon-512.png)
# drop .ogg files into sounds/<language>/<category>/, or:
soundboard-fetch --character-dir . --language en   # if [voice_scraper] is configured
python run.py
```

New languages and categories are auto-discovered from the `sounds/`
folder structure — drop a new language folder in and call
`POST /api/reload` (or restart) to pick it up.

## `character.toml`

```toml
[character]
name = "My Character"
short_name = "My Character"
description = "Remote control panel for my character's sound player."
filename_prefix_to_strip = ""

[audio]
fade_in_ms = 400
fade_out_ms = 300
language_independent_categories = ["music"]

[categories]
order = ["general", "move", "attack", "kill", "death", "music"]

[categories.labels]
general = "General"
move = "Move"
attack = "Attack"
kill = "Kill"
death = "Death"
music = "Music & SFX"

[theme]
# Leave empty to use the framework's default palette, or override any subset:
# abyss = "#0a0f12"
# glow = "#35e0b8"

[voice_scraper]
# url = "https://example.com/my-character/Audio"
# [voice_scraper.category_map]
# Joke = "general"
```

A character project can also drop an optional `theme.css` file next to
`character.toml` for bespoke CSS beyond the theme tokens (animations,
one-off rules) — it's appended after the framework's base styles.

## Environment variables

`SOUNDBOARD_HOST` (default `0.0.0.0`), `SOUNDBOARD_PORT` (default `5000`)
— or pass `--host`/`--port` to `soundboard-serve`.

## Raspberry Pi note

`sounddevice` needs the PortAudio system library:

```bash
sudo apt install libportaudio2
```

`soundfile` ships with libsndfile bundled in its wheel — no extra packages.

## API

| Method | Route          | Body / params                  | Purpose                          |
|--------|----------------|---------------------------------|-----------------------------------|
| GET    | `/api/library` | —                              | Full catalog (languages, categories, sounds with durations) |
| GET    | `/api/status`  | —                              | Now playing, position, loop, volume |
| POST   | `/api/play`    | `{"key": "en/kill/…", "loop": false}` | Play a sound (replaces current) |
| POST   | `/api/stop`    | —                              | Fade out and stop                |
| POST   | `/api/volume`  | `{"volume": 0.0–1.0}`          | Set server volume                |
| POST   | `/api/loop`    | `{"loop": true}`               | Toggle looping (never interrupts playback)  |
| POST   | `/api/reload`  | —                              | Rescan the sounds folder         |

## Development

```bash
pip install -e ".[dev]"
pytest tests/
```
