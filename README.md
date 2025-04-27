# CineDual

A simple solution for playing a video with two different audio tracks simultaneously on separate output devices. Perfect for multilingual viewers or when one person needs audio description.

## Quick Start

```bash
# Install required packages
sudo apt install mpv socat fzf pulseaudio-utils  # Debian/Ubuntu
# or
sudo pacman -S mpv socat fzf pipewire-pulse      # Arch Linux

# Start playback (defaults to tracks 1 & 2 if not specified)
./cinedual play "movie.mkv"
# Or specify audio tracks
./cinedual play "movie.mkv" 1 2

# Control playback from another terminal
./cinedual control
```

## Requirements

- `mpv`: Media player
- `socat`: IPC communication utility
- `fzf`: Interactive selection tool
- `pactl`: PulseAudio/PipeWire utility

## Features

- Play video with different audio tracks on two audio devices
- Interactive sink selection with fuzzy search
- Synchronization controls to fine-tune audio timing
- Remembers delay settings between sessions

## Usage Details

### Starting Playback

```bash
./cinedual play "movie.mkv" [<video_audio_id> <second_audio_id>]
```

If audio tracks aren't specified, defaults to tracks 1 and 2. You'll be prompted to select output devices for each audio stream.

### Controlling Playback

Interactive mode:
```bash
./cinedual control
```

Direct commands:
```bash
./cinedual control play               # Resume playback
./cinedual control pause              # Pause playback
./cinedual control seek 300           # Jump to 5 minutes position
./cinedual control set_delay 0.3      # Set 300ms audio sync delay
./cinedual control increase_delay     # Increase audio sync delay by small step
./cinedual control decrease_delay     # Decrease audio sync delay by small step
./cinedual control quit               # Exit players
```

### Getting Help

```bash
./cinedual help
```

## Audio Synchronization

After seeking, audio tracks may need synchronization:

- **Adjust delay:** Use `set_delay` command (e.g., `set_delay 0.3`)
- **Fine-tune:** Use `increase_delay` and `decrease_delay` commands

The delay value refers to the audio-only player relative to the video player:
- Positive value: Audio-only player is delayed
- Negative value: Audio-only player starts earlier

While `increase_delay`, `decrease_delay`, and `set_delay` work in most cases, the most reliable way to realign tracks is to:
1. Use `seek` to a nearby position
2. Let the system apply the current delay setting automatically

For optimal lip synchronization, it's recommended to:
- Place the original language track on the video player
- Place the dubbed/translated track on the audio-only player

Delay settings are automatically saved between sessions.

## How It Works

CineDual launches two MPV instances:
1. First plays video + primary audio track
2. Second plays only the secondary audio track

The instances are synchronized through IPC sockets using precise timing calculations to ensure both audio tracks remain aligned throughout playback.

Synchronization preferences are stored in `~/.config/cinedual/settings.conf`.